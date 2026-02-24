# ERP-CRM Software Architecture

## 1. Architecture Style

ERP-CRM employs a hybrid architecture combining a Domain-Driven Design (DDD) Rust monolith for core CRM logic with twelve Go microservices for domain-specific CRUD operations. The system follows hexagonal (ports and adapters) architecture internally and communicates across service boundaries via event-driven messaging.

```mermaid
graph TB
    subgraph "Hexagonal Architecture (Rust Core)"
        subgraph "Inbound Ports"
            IP1["ContactUseCases"]
            IP2["DealUseCases"]
        end

        subgraph "Application Layer"
            APP["ContactService<br/>DealService<br/>Use Case Orchestration"]
        end

        subgraph "Domain Layer"
            DOM["Aggregates: Contact, Deal, Account<br/>Value Objects: Email, Money, Phone, Address<br/>Domain Events: ContactEvent, DealEvent, AccountEvent<br/>Domain Services: LeadScoring, Forecast, Merge"]
        end

        subgraph "Outbound Ports"
            OP1["ContactRepository"]
            OP2["DealRepository"]
            OP3["EventPublisher"]
        end

        subgraph "Infrastructure Adapters"
            INF1["PostgresContactRepo"]
            INF2["PostgresDealRepo"]
            INF3["NatsEventPublisher"]
            INF4["PulsarEventPublisher"]
        end

        IP1 & IP2 --> APP
        APP --> DOM
        APP --> OP1 & OP2 & OP3
        OP1 --> INF1
        OP2 --> INF2
        OP3 --> INF3 & INF4
    end
```

## 2. Component Architecture

### 2.1 Rust Core Monolith (`src/`)

The Rust core is the authoritative system of record for contacts, companies, deals, activities, and pipelines. It exposes REST API endpoints via axum and publishes domain events via NATS.

```mermaid
graph LR
    subgraph "src/"
        MAIN["main.rs<br/>(Entry Point, Router, Handlers)"]
        CONFIG["config.rs<br/>(Environment Configuration)"]
        LIB["lib.rs<br/>(Module Registry, Re-exports)"]

        subgraph "domain/"
            AGG["aggregates/<br/>contact.rs, deal.rs"]
            VO["value_objects/<br/>email.rs, money.rs, phone.rs, address.rs"]
            EVT["events/<br/>DomainEvent, ContactEvent, DealEvent, AccountEvent"]
            SVC["services/<br/>LeadScoringService, ForecastService, ContactMergeService"]
        end

        subgraph "application/"
            CMD["commands/<br/>ContactService, DealService"]
            QRY["queries/"]
            DTO["dto/<br/>CreateContactCommand, PipelineView, ForecastView"]
        end

        subgraph "ports/"
            INB["inbound/<br/>ContactUseCases, DealUseCases, UseCaseError"]
            OUT["outbound/<br/>ContactRepository, DealRepository, EventPublisher, RepositoryError"]
        end

        subgraph "infrastructure/"
            PER["persistence/<br/>PostgreSQL implementations"]
        end
    end

    MAIN --> LIB
    LIB --> AGG & VO & EVT & SVC
    LIB --> CMD & QRY & DTO
    LIB --> INB & OUT
    LIB --> PER
```

### 2.2 Go Microservices (`services/`)

Each Go microservice follows an identical structure with health check, CRUD endpoints, tenant isolation, and CloudEvents topic emission.

```mermaid
graph TB
    subgraph "Go Microservice Pattern"
        HEALTH["/healthz<br/>Health Check"]
        LIST["GET /v1/{entity}<br/>List (event: listed)"]
        CREATE["POST /v1/{entity}<br/>Create (event: created)"]
        READ["GET /v1/{entity}/{id}<br/>Read (event: read)"]
        UPDATE["PUT /v1/{entity}/{id}<br/>Update (event: updated)"]
        DELETE["DELETE /v1/{entity}/{id}<br/>Delete (event: deleted)"]
    end

    subgraph "Middleware"
        TENANT["X-Tenant-ID Validation"]
        JSON["JSON Response Writer"]
    end

    TENANT --> LIST & CREATE & READ & UPDATE & DELETE
    JSON --> LIST & CREATE & READ & UPDATE & DELETE
```

### 2.3 Web Frontend (`web/`)

```mermaid
graph TB
    subgraph "React Frontend (Refine + AntD)"
        ENTRY["main.tsx<br/>(App Entry)"]
        APP["App.tsx<br/>(Refine Provider, Routes)"]
        AUTH["authProvider.ts<br/>(JWT Authentication)"]
        DATA["dataProvider.ts<br/>(GraphQL Data Fetching)"]
        GQL["graphql/entities.graphql<br/>(Operations)"]
        CODEGEN["codegen.ts<br/>(GraphQL Code Generation)"]
    end

    ENTRY --> APP
    APP --> AUTH & DATA
    DATA --> GQL
    GQL --> CODEGEN
```

## 3. Domain Model

### 3.1 Contact Aggregate

The Contact aggregate is the richest domain object, encapsulating lead lifecycle management, scoring, qualification, and ownership.

```mermaid
statechart-v2
    [*] --> New
    New --> Contacted: Record Activity
    Contacted --> Qualified: qualify()
    Contacted --> Unqualified: disqualify()
    Qualified --> Nurturing: Needs Nurture
    Nurturing --> Qualified: Re-qualify
    Qualified --> Converted: convert_to_customer()
    Unqualified --> [*]: Archived

    state Qualified {
        [*] --> SalesQualifiedLead
        SalesQualifiedLead --> Opportunity: Deal Created
    }

    state Converted {
        [*] --> Customer
        Customer --> Evangelist: High NPS
    }
```

**Key Business Rules:**
- Cannot qualify an already-unqualified contact
- Cannot qualify an already-qualified contact
- Cannot convert to customer if already a customer
- Lead score changes of 10+ points emit LeadScoreChanged event
- Ownership transfer emits OwnershipTransferred event

### 3.2 Deal Aggregate

```mermaid
statechart-v2
    [*] --> Open
    Open --> Open: move_to_stage()
    Open --> Won: close_won()
    Open --> Lost: close_lost()
    Won --> Open: reopen()
    Lost --> Open: reopen()
    Won --> [*]: Archived
    Lost --> [*]: Archived

    state Open {
        [*] --> Lead
        Lead --> Qualified: 25%
        Qualified --> Proposal: 50%
        Proposal --> Negotiation: 75%
        Negotiation --> ClosedWon: 100%
    }
```

**Key Business Rules:**
- Closed deals cannot have stage changes or amount updates
- Currency mismatch on amount updates raises error
- Product addition/removal triggers amount recalculation
- Weighted value = amount x (probability / 100)
- Stage history is immutable and append-only

### 3.3 Ticket Aggregate (Support)

```mermaid
statechart-v2
    [*] --> New
    New --> Open: assign()
    Open --> Pending: Awaiting Customer
    Open --> OnHold: Paused
    Pending --> Open: Customer Reply
    OnHold --> Open: Resume
    Open --> Solved: solve()
    Solved --> Closed: close()
    Solved --> Open: reopen()
    Closed --> Open: reopen()
```

## 4. Event Architecture

### 4.1 Domain Events

```mermaid
graph LR
    subgraph "Contact Events"
        CE1["contact.created"]
        CE2["contact.qualified"]
        CE3["contact.converted_to_customer"]
        CE4["contact.lead_score_changed"]
        CE5["contact.ownership_transferred"]
        CE6["contact.merged"]
    end

    subgraph "Deal Events"
        DE1["deal.created"]
        DE2["deal.stage_changed"]
        DE3["deal.won"]
        DE4["deal.lost"]
        DE5["deal.amount_changed"]
    end

    subgraph "Account Events"
        AE1["account.created"]
        AE2["account.contact_linked"]
        AE3["account.deal_linked"]
    end

    subgraph "Consumers"
        AUTO2["Automation Engine"]
        AUDIT["Audit Logger"]
        NOTIFY["Notification Service"]
        ANALYTICS["Analytics Pipeline"]
    end

    CE1 & CE2 & CE3 & CE4 --> AUTO2 & AUDIT & ANALYTICS
    DE1 & DE2 & DE3 & DE4 --> AUTO2 & AUDIT & NOTIFY & ANALYTICS
    AE1 & AE2 & AE3 --> AUDIT & ANALYTICS
```

### 4.2 Event Format (CloudEvents)

```json
{
  "specversion": "1.0",
  "type": "com.opensase.crm.contact.created",
  "source": "opensase-crm",
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "time": "2026-02-23T10:00:00Z",
  "data": {
    "contact_id": "019503a2-...",
    "email": "john@example.com"
  }
}
```

## 5. Data Architecture

### 5.1 Database Schema Overview

```mermaid
erDiagram
    contacts ||--o{ deals : "has"
    contacts ||--o{ activities : "has"
    contacts ||--o{ notes : "has"
    companies ||--o{ contacts : "employs"
    companies ||--o{ deals : "has"
    companies ||--o{ activities : "has"
    companies ||--o{ notes : "has"
    pipelines ||--|{ pipeline_stages : "contains"
    pipeline_stages ||--o{ deals : "contains"
    deals ||--o{ activities : "has"
    deals ||--o{ notes : "has"
    tickets ||--|{ ticket_comments : "has"
    kb_categories ||--o{ kb_articles : "contains"
    forms ||--|{ submissions : "receives"
```

### 5.2 Query Patterns

| Pattern | Implementation | Index Strategy |
|---------|---------------|---------------|
| Contact by email | `SELECT * FROM contacts WHERE email = $1` | B-tree on `email` |
| Contacts by company | `SELECT * FROM contacts WHERE company_id = $1` | B-tree on `company_id` |
| Contacts by lifecycle | `SELECT * FROM contacts WHERE lifecycle_stage = $1` | B-tree on `lifecycle_stage` |
| Deals by pipeline | `SELECT * FROM deals WHERE pipeline_id = $1` | B-tree on `pipeline_id` |
| Deals by stage | `SELECT * FROM deals WHERE stage_id = $1` | B-tree on `stage_id` |
| Activities by contact | `SELECT * FROM activities WHERE contact_id = $1` | B-tree on `contact_id` |
| Activities by due date | `SELECT * FROM activities WHERE due_date < $1` | B-tree on `due_date` |
| Tickets by status | `SELECT * FROM tickets WHERE status = $1` | B-tree on `status` |
| Tickets by customer | `SELECT * FROM tickets WHERE customer_email = $1` | B-tree on `customer_email` |
| Form submissions | `SELECT * FROM submissions WHERE form_id = $1` | B-tree on `form_id` |

## 6. Security Architecture

```mermaid
flowchart TB
    CLIENT["Client Request"] --> GW2["API Gateway"]
    GW2 --> JWT["JWT Validation<br/>(ERP-IAM)"]
    JWT --> TENANT2["Tenant Extraction<br/>(X-Tenant-ID)"]
    TENANT2 --> RBAC["RBAC Check<br/>(Role Permissions)"]
    RBAC --> HANDLER["Request Handler"]
    HANDLER --> REPO["Repository<br/>(Tenant-Scoped Queries)"]
    REPO --> DB2["PostgreSQL<br/>(Row-Level Security)"]
    HANDLER --> AUDIT2["Audit Event<br/>(Pulsar Immutable Topic)"]
```

## 7. Concurrency Model

The Rust core uses Tokio's async runtime for non-blocking I/O:

- **Database pool**: sqlx PgPoolOptions with configurable max connections (default: 10)
- **Request handling**: axum handlers are `async fn` with shared `AppState` via `Arc`
- **Event publishing**: Non-blocking NATS publish with fire-and-forget semantics
- **In-memory caching**: `DashMap` for concurrent read/write access to hot data
- **Connection management**: Graceful degradation when NATS is unavailable (optional integration)
