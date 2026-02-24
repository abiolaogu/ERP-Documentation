# ERP-Marketing -- Software Architecture

## 1. Overview

ERP-Marketing follows a hybrid architecture combining a Rust monolith API gateway with domain-driven Go microservices. The Rust core owns the database schema, API routing, and domain aggregates. The Go services handle domain-specific processing with tenant-aware HTTP handlers. Communication between components is both synchronous (REST/gRPC) and asynchronous (Apache Pulsar events).

## 2. Layered Architecture

```mermaid
flowchart TB
    subgraph "Presentation Layer"
        WEB[React + Ant Design + Refine SPA]
        MOBILE[Flutter / Android / iOS]
        API_GW[REST API Gateway]
    end
    subgraph "Application Layer"
        CMD_HANDLERS[Command Handlers]
        QUERY_HANDLERS[Query Handlers]
        AIDD[AIDD Guardrail Engine]
    end
    subgraph "Domain Layer"
        AGG[Aggregates: Campaign, Automation]
        VO[Value Objects: CampaignType, Segment, FilterCondition]
        EVT[Domain Events: CampaignEvent, AutomationEvent]
    end
    subgraph "Infrastructure Layer"
        DB[PostgreSQL via sqlx]
        MQ[Apache Pulsar Client]
        CACHE[DashMap In-Memory Cache]
        LOG[Quickwit Logger]
    end
    WEB --> API_GW
    MOBILE --> API_GW
    API_GW --> CMD_HANDLERS & QUERY_HANDLERS
    CMD_HANDLERS --> AIDD --> AGG
    QUERY_HANDLERS --> AGG
    AGG --> EVT
    AGG --> DB
    EVT --> MQ
    MQ --> LOG
```

## 3. Domain-Driven Design Structure

### 3.1 Module Organization

```
src/
  domain/
    aggregates/
      campaign.rs      -- Campaign aggregate root
      automation.rs     -- Automation/workflow aggregate root
      mod.rs            -- Public exports
    value_objects/
      mod.rs            -- CampaignType, Segment, SegmentFilter, FilterCondition, FilterLogic
    events/
      mod.rs            -- DomainEvent, CampaignEvent, AutomationEvent
    mod.rs              -- Domain module root
  lib.rs                -- Public API surface
  main.rs               -- axum HTTP server, route definitions, handler functions
```

### 3.2 Aggregate Invariants

**Campaign Aggregate:**
- A campaign cannot be sent or scheduled without content
- Sending transitions status from Draft/Scheduled to Sending
- Completion records final stats and emits a CampaignSent domain event
- Cancellation is terminal

**Automation Aggregate:**
- Steps are added sequentially
- Activation requires at least a trigger and name
- Pausing suspends enrollment but preserves state

### 3.3 State Machines

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Scheduled: schedule(at)
    Draft --> Sending: send()
    Scheduled --> Sending: trigger at time
    Sending --> Sent: complete(stats)
    Draft --> Cancelled: cancel()
    Scheduled --> Cancelled: cancel()
    Sending --> Paused: pause()
    Paused --> Sending: resume()
```

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Active: activate()
    Active --> Paused: pause()
    Paused --> Active: activate()
    Active --> Archived: archive()
    Paused --> Archived: archive()
```

## 4. API Architecture

### 4.1 Rust API Gateway Routes

| Method | Path | Handler | Description |
|---|---|---|---|
| GET | `/health` | inline | Health check |
| GET | `/api/v1/campaigns` | `list_campaigns` | List all campaigns |
| POST | `/api/v1/campaigns` | `create_campaign` | Create campaign |
| GET | `/api/v1/campaigns/:id` | `get_campaign` | Get campaign by ID |
| POST | `/api/v1/campaigns/:id/send` | `send_campaign` | Trigger campaign send |
| POST | `/api/v1/campaigns/:id/launch` | AIDD-gated | Launch with guardrail |
| POST | `/api/v1/campaigns/:id/pause` | `pause_campaign` | Pause active campaign |
| GET | `/api/v1/audiences` | `list_audiences` | List audiences |
| POST | `/api/v1/audiences` | `create_audience` | Create audience |
| GET | `/api/v1/templates` | `list_templates` | List email templates |
| POST | `/api/v1/templates` | `create_template` | Create email template |
| GET | `/api/v1/contacts` | `fetchContacts` | List contacts |
| POST | `/api/v1/contacts` | `createContact` | Create contact |
| POST | `/api/v1/contacts/:id/score` | AIDD-gated | Score contact |
| GET | `/api/v1/segments` | `fetchSegments` | List segments |
| GET | `/api/v1/journeys` | `fetchJourneys` | List journeys |
| POST | `/api/v1/journeys/:id/activate` | AIDD-gated | Activate journey |
| GET | `/api/v1/dashboard/summary` | `fetchDashboardSummary` | Dashboard KPIs |
| GET | `/api/v1/dashboard/attribution` | `fetchAttributionSummary` | Attribution by channel |
| GET | `/api/v1/recommendations` | `fetchRecommendations` | AI recommendations |
| GET | `/api/v1/audit/guardrails` | `fetchGuardrailEvents` | Guardrail audit log |

### 4.2 Go Microservice Pattern

Each Go microservice follows an identical pattern:

```mermaid
flowchart LR
    REQ[HTTP Request] --> TENANT{X-Tenant-ID?}
    TENANT -->|Missing| ERR[400 Bad Request]
    TENANT -->|Present| METHOD{HTTP Method}
    METHOD -->|GET /base| LIST[List items + emit listed event]
    METHOD -->|POST /base| CREATE[Create item + emit created event]
    METHOD -->|GET /base/:id| READ[Read item + emit read event]
    METHOD -->|PUT/PATCH /base/:id| UPDATE[Update item + emit updated event]
    METHOD -->|DELETE /base/:id| DELETE[Delete item + emit deleted event]
```

## 5. Data Access Pattern

The Rust core uses sqlx with compile-time checked SQL queries against PostgreSQL. All database interactions use the connection pool managed by `PgPoolOptions` with a configurable `max_connections` (default 10).

```mermaid
flowchart LR
    Handler[axum Handler] -->|State extraction| Pool[PgPool]
    Pool --> Query[sqlx::query_as]
    Query --> PG[(PostgreSQL)]
    PG --> Row[Deserialized Struct]
    Row --> JSON[Json Response]
```

Key patterns:
- UUID v7 for time-ordered primary keys (`Uuid::now_v7()`)
- JSONB columns for flexible schema data (filters, tags, traits, engagement, steps)
- `RETURNING *` for atomic insert-and-return
- Indexed columns for hot query paths (status, lifecycle stage, lead score, timestamps)

## 6. Frontend Architecture

```mermaid
flowchart TB
    subgraph "React SPA"
        REFINE[Refine Framework]
        ANTD[Ant Design Components]
        RQ[TanStack React Query]
        GQL[GraphQL Codegen Types]
    end
    subgraph "Data Layer"
        DP[Data Provider<br/>REST + GraphQL]
        AP[Auth Provider<br/>OAuth/OIDC]
        API_CLIENT[marketingApi.ts<br/>Typed REST Client]
    end
    REFINE --> ANTD
    REFINE --> DP & AP
    DP --> API_CLIENT
    API_CLIENT --> BACKEND[Backend API]
```

The frontend uses a typed API client (`marketingApi.ts`) that handles snake_case-to-camelCase transformation, error handling, and type-safe request/response mapping for all 25+ API endpoints.

## 7. Concurrency Model

- **Rust async**: tokio multi-threaded runtime with work-stealing scheduler
- **Connection pooling**: sqlx PgPool with bounded connections
- **In-memory cache**: DashMap for concurrent read-heavy workloads
- **Event processing**: Pulsar consumers with configurable parallelism per partition

## 8. Error Handling

- Domain errors: `CampaignError` enum with `NoContent`, `NoSegment` variants
- API errors: HTTP status codes mapped from domain errors via `(StatusCode, String)` tuples
- Client errors: `ApiClientError` class with status code and message
- Infrastructure errors: anyhow for contextual error propagation

## 9. Testing Strategy

- **Unit tests**: Rust `#[cfg(test)]` modules per aggregate (e.g., `test_campaign`)
- **Frontend tests**: Vitest with React Testing Library
- **Integration tests**: CI pipeline with PostgreSQL service container
- **Quality gates**: `cargo fmt`, `cargo clippy -D warnings`, `cargo test --all-features`
