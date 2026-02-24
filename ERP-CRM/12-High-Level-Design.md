# ERP-CRM High-Level Design

## 1. System Context

```mermaid
C4Context
    title ERP-CRM System Context Diagram

    Person(salesRep, "Sales Representative", "Manages contacts, deals, pipeline")
    Person(supportAgent, "Support Agent", "Handles tickets, KB articles")
    Person(admin, "CRM Administrator", "Configures system, automation")
    Person(customer, "Customer", "Self-service, chat, forms")

    System(crm, "ERP-CRM", "Customer Relationship Management Module")

    System_Ext(iam, "ERP-IAM", "Identity & Access Management")
    System_Ext(platform, "ERP-Platform", "Control Plane & Licensing")
    System_Ext(directory, "ERP-Directory", "User Provisioning")
    System_Ext(email, "Email System", "SMTP/IMAP Integration")

    Rel(salesRep, crm, "Uses", "HTTPS/GraphQL")
    Rel(supportAgent, crm, "Uses", "HTTPS/GraphQL")
    Rel(admin, crm, "Configures", "HTTPS")
    Rel(customer, crm, "Self-service", "HTTPS/WebSocket")

    Rel(crm, iam, "Authenticates", "OIDC/JWT")
    Rel(crm, platform, "License check", "REST")
    Rel(crm, directory, "User sync", "REST")
    Rel(crm, email, "Send/Receive", "SMTP/IMAP")
```

## 2. Container Architecture

```mermaid
graph TB
    subgraph "Client Tier"
        WEB2["React Web App<br/>Refine + AntD + React Query"]
        FLUTTER2["Flutter Mobile App<br/>Ferry + Riverpod"]
        ANDROID2["Android Native<br/>Compose + Apollo + Hilt"]
        IOS2["iOS Native<br/>SwiftUI + Apollo + TCA"]
    end

    subgraph "API Tier"
        GW3["API Gateway<br/>Nginx / Envoy"]
        HASURA2["Hasura GraphQL<br/>Auto-generated from PostgreSQL"]
        CORE2["CRM Core API<br/>Rust (axum) on port 8081"]
    end

    subgraph "Microservices Tier"
        direction LR
        MS_CONTACT["contact-service<br/>Go :8080"]
        MS_LEAD["lead-service<br/>Go :8080"]
        MS_PIPELINE["pipeline-service<br/>Go :8080"]
        MS_OPP["opportunity-service<br/>Go :8080"]
        MS_ACTIVITY["activity-service<br/>Go :8080"]
        MS_HELPDESK["helpdesk-service<br/>Go :8080"]
        MS_KB["knowledge-base-service<br/>Go :8080"]
        MS_FORM["form-builder-service<br/>Go :8080"]
        MS_CHAT["chat-service<br/>Go :8080"]
        MS_AUTO["automation-service<br/>Go :8080"]
        MS_REPORT["reporting-service<br/>Go :8080"]
        MS_TERRITORY["territory-service<br/>Go :8080"]
    end

    subgraph "Data Tier"
        PG3["PostgreSQL 16<br/>Primary data store"]
        NATS3["NATS JetStream<br/>Real-time events"]
        PULSAR3["Apache Pulsar<br/>Durable event streaming"]
        QW3["Quickwit<br/>Log search & analytics"]
    end

    WEB2 & FLUTTER2 & ANDROID2 & IOS2 --> GW3
    GW3 --> HASURA2
    GW3 --> CORE2
    GW3 --> MS_CONTACT & MS_LEAD & MS_PIPELINE & MS_OPP & MS_ACTIVITY & MS_HELPDESK
    GW3 --> MS_KB & MS_FORM & MS_CHAT & MS_AUTO & MS_REPORT & MS_TERRITORY

    HASURA2 --> PG3
    CORE2 --> PG3
    CORE2 --> NATS3
    MS_CONTACT & MS_LEAD & MS_PIPELINE --> PULSAR3
    PULSAR3 --> QW3
```

## 3. Component Design

### 3.1 CRM Core Components

```mermaid
graph TB
    subgraph "CRM Core (Rust)"
        subgraph "API Layer"
            ROUTER["Router<br/>axum::Router"]
            HEALTH2["Health Handlers<br/>/health, /ready, /metrics"]
            CONTACT_H["Contact Handlers<br/>CRUD + Search"]
            COMPANY_H["Company Handlers<br/>CRUD"]
            DEAL_H["Deal Handlers<br/>CRUD + Pipeline"]
            ACTIVITY_H["Activity Handlers<br/>CRUD"]
            DASH_H["Dashboard Handler<br/>Aggregated Stats"]
        end

        subgraph "Domain Layer"
            CONTACT_AGG["Contact Aggregate<br/>Lifecycle, Scoring, Tags"]
            DEAL_AGG["Deal Aggregate<br/>Stages, Products, Competitors"]
            LEAD_SCORE_SVC["LeadScoringService<br/>AI Score Calculation"]
            FORECAST_SVC["ForecastService<br/>Pipeline Analytics"]
            MERGE_SVC["ContactMergeService<br/>Deduplication"]
        end

        subgraph "Infrastructure Layer"
            PG_REPO["PostgreSQL Repository<br/>sqlx queries"]
            NATS_PUB["NATS Publisher<br/>Domain events"]
            CONFIG2["Configuration<br/>Environment variables"]
        end
    end

    ROUTER --> HEALTH2 & CONTACT_H & COMPANY_H & DEAL_H & ACTIVITY_H & DASH_H
    CONTACT_H --> CONTACT_AGG
    DEAL_H --> DEAL_AGG
    CONTACT_AGG --> LEAD_SCORE_SVC
    DEAL_AGG --> FORECAST_SVC
    CONTACT_AGG --> MERGE_SVC
    CONTACT_AGG & DEAL_AGG --> PG_REPO & NATS_PUB
```

### 3.2 Microservice Component (Generic)

Each Go microservice follows an identical internal pattern:

```mermaid
graph TB
    subgraph "Go Microservice"
        MUX["HTTP Multiplexer<br/>net/http"]
        HEALTH3["/healthz Handler"]
        LIST_H["List Handler<br/>GET /v1/{entity}"]
        CREATE_H["Create Handler<br/>POST /v1/{entity}"]
        READ_H["Read Handler<br/>GET /v1/{entity}/{id}"]
        UPDATE_H["Update Handler<br/>PUT /v1/{entity}/{id}"]
        DELETE_H["Delete Handler<br/>DELETE /v1/{entity}/{id}"]
        TENANT_MW["Tenant Middleware<br/>X-Tenant-ID validation"]
        JSON_W["JSON Writer<br/>Response serialization"]
    end

    MUX --> HEALTH3
    MUX --> TENANT_MW
    TENANT_MW --> LIST_H & CREATE_H & READ_H & UPDATE_H & DELETE_H
    LIST_H & CREATE_H & READ_H & UPDATE_H & DELETE_H --> JSON_W
```

## 4. Data Flow Design

### 4.1 Read Path

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant Hasura as Hasura GraphQL
    participant PG as PostgreSQL

    Client->>Gateway: GraphQL Query
    Gateway->>Hasura: Forward Query
    Hasura->>PG: SQL Query (auto-generated)
    PG-->>Hasura: Result Set
    Hasura-->>Gateway: GraphQL Response
    Gateway-->>Client: JSON Response
```

### 4.2 Write Path

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant Core as CRM Core (Rust)
    participant Domain as Domain Aggregate
    participant PG as PostgreSQL
    participant NATS as NATS JetStream
    participant Pulsar as Apache Pulsar

    Client->>Gateway: REST POST/PUT/DELETE
    Gateway->>Core: Forward Request
    Core->>Core: Validate Input (validator)
    Core->>Domain: Create/Update Aggregate
    Domain->>Domain: Apply Business Rules
    Domain->>Domain: Raise Domain Events
    Core->>PG: Persist Aggregate
    Core->>NATS: Publish Domain Events
    NATS->>Pulsar: Relay to Durable Store
    Core-->>Gateway: Response
    Gateway-->>Client: JSON Response
```

## 5. Deployment Architecture

```mermaid
graph TB
    subgraph "Production Kubernetes Cluster"
        subgraph "Namespace: crm"
            CRM_DEPLOY["crm-core<br/>Deployment (2 replicas)"]
            SVC_DEPLOY["microservices<br/>12 Deployments (1+ replicas)"]
            WEB_DEPLOY["web-frontend<br/>Deployment (2 replicas)"]
            HASURA_DEPLOY["hasura<br/>Deployment (2 replicas)"]
        end

        subgraph "Namespace: data"
            PG_SS["postgresql<br/>StatefulSet (1 primary + 1 replica)"]
            NATS_SS["nats<br/>StatefulSet (3 nodes)"]
        end

        subgraph "Namespace: platform"
            PULSAR_SS["pulsar<br/>StatefulSet (3+ bookies)"]
            QW_DEPLOY2["quickwit<br/>Deployment (searcher + indexer)"]
        end

        subgraph "Ingress"
            INGRESS2["Ingress Controller<br/>TLS termination"]
        end
    end

    INGRESS2 --> CRM_DEPLOY & SVC_DEPLOY & WEB_DEPLOY & HASURA_DEPLOY
    CRM_DEPLOY --> PG_SS & NATS_SS
    HASURA_DEPLOY --> PG_SS
    NATS_SS --> PULSAR_SS
    PULSAR_SS --> QW_DEPLOY2
```

## 6. Security Design

### 6.1 Authentication and Authorization Flow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Gateway as API Gateway
    participant IAM as ERP-IAM
    participant CRM as CRM Service

    User->>Browser: Access CRM
    Browser->>Gateway: Request (no token)
    Gateway->>Browser: Redirect to IAM
    Browser->>IAM: OIDC Login
    IAM-->>Browser: JWT Token
    Browser->>Gateway: Request + Bearer Token
    Gateway->>IAM: Validate JWT
    IAM-->>Gateway: Token Valid + Claims
    Gateway->>CRM: Request + X-Tenant-ID + User Claims
    CRM->>CRM: RBAC Check
    CRM-->>Gateway: Response
    Gateway-->>Browser: Response
```

### 6.2 Tenant Isolation

All data access is scoped by tenant:

```mermaid
graph LR
    REQ["API Request"] --> EXTRACT["Extract<br/>X-Tenant-ID Header"]
    EXTRACT --> VALIDATE4["Validate Tenant<br/>Exists in IAM"]
    VALIDATE4 --> SCOPE["Scope All Queries<br/>WHERE tenant_id = $1"]
    SCOPE --> EXECUTE["Execute Query<br/>on Tenant Data Only"]
```

## 7. Scalability Design

### 7.1 Horizontal Scaling Strategy

| Component | Scaling Method | Scaling Trigger | Max Scale |
|-----------|---------------|----------------|-----------|
| CRM Core | HPA (CPU 70%) | Request rate | 10 replicas |
| Go Microservices | HPA (CPU 60%) | Request rate | 5 replicas each |
| Web Frontend | HPA (CPU 50%) | Connection count | 5 replicas |
| Hasura | HPA (CPU 70%) | Query rate | 5 replicas |
| PostgreSQL | Vertical + Read Replicas | Query latency | 1 primary + 3 replicas |
| NATS | StatefulSet | Message backlog | 5 nodes |

### 7.2 Caching Strategy

```mermaid
graph LR
    REQUEST["API Request"] --> CACHE_CHECK{In Cache?}
    CACHE_CHECK -->|Yes| RETURN_CACHED["Return Cached<br/>Response"]
    CACHE_CHECK -->|No| DB_QUERY["Query<br/>PostgreSQL"]
    DB_QUERY --> CACHE_STORE["Store in<br/>DashMap Cache"]
    CACHE_STORE --> RETURN_FRESH["Return Fresh<br/>Response"]

    WRITE["Write Operation"] --> INVALIDATE["Invalidate<br/>Cache Entry"]
    INVALIDATE --> PERSIST["Persist to<br/>PostgreSQL"]
```

## 8. Reliability Design

### 8.1 Failure Handling

```mermaid
graph TB
    subgraph "Failure Modes"
        F1["Database<br/>Unavailable"]
        F2["NATS<br/>Disconnected"]
        F3["Pulsar<br/>Down"]
        F4["Microservice<br/>Crashed"]
    end

    subgraph "Handling"
        H1["Readiness probe fails<br/>Traffic diverted"]
        H2["Graceful degradation<br/>Events queued locally"]
        H3["NATS retains events<br/>Replay on recovery"]
        H4["K8s auto-restart<br/>Health check loop"]
    end

    F1 --> H1
    F2 --> H2
    F3 --> H3
    F4 --> H4
```

The CRM core handles NATS disconnection gracefully -- the connection is optional, and the system continues to function without event publishing. This is implemented in `main.rs` with a fallback that logs a warning and sets `nats = None`.
