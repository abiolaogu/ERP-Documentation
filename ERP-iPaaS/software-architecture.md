# Software Architecture -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This document describes the software architecture of ERP-iPaaS at the component and module level, detailing service internals, package structure, communication patterns, and the technology choices behind each layer.

## 2. Component Architecture

```mermaid
graph TB
    subgraph "Presentation Layer"
        WEB[Web UI - React/Next.js]
        PORTAL[Developer Portal - Static]
        MOBILE[Mobile - Flutter/React Native]
    end

    subgraph "API Layer"
        GW[Traefik API Gateway]
        GQL[GraphQL Schema]
        REST[REST OpenAPI]
    end

    subgraph "Service Layer"
        direction TB
        WE[Workflow Engine Service]
        CF[Connector Framework Service]
        EB[Event Backbone Service]
        AM[API Management Service]
        ETL[ETL Pipeline Service]
        WH[Webhook Management Service]
    end

    subgraph "Runtime Layer"
        AP[Activepieces Runtime]
        TMP[Temporal Runtime]
        NF[Nexum Flow Engine]
        MCP[MCP Host - AI Gateway]
    end

    subgraph "SDK Layer"
        TS_SDK[TypeScript SDK<br/>packages/integration-layer-ts]
        GO_SDK[Go SDK<br/>packages/integration-layer-go]
        CLI[Connector CLI<br/>packages/connector-cli]
        LLM[LLM Utilities<br/>packages/llm-utils]
    end

    subgraph "Data Access Layer"
        PG_DAL[PostgreSQL DAL]
        CH_DAL[ClickHouse DAL]
        RP_DAL[Redpanda Producer/Consumer]
        REDIS_DAL[Redis/Dragonfly Cache]
        S3_DAL[MinIO S3 Client]
    end

    WEB --> GW
    PORTAL --> GW
    MOBILE --> GW
    GW --> WE
    GW --> CF
    GW --> EB
    GW --> AM
    GW --> ETL
    GW --> WH

    WE --> AP
    WE --> TMP
    WE --> NF
    NF --> MCP

    WE --> PG_DAL
    WE --> CH_DAL
    WE --> RP_DAL
    CF --> PG_DAL
    CF --> RP_DAL
    EB --> RP_DAL
    EB --> CH_DAL
    ETL --> PG_DAL
    ETL --> CH_DAL
    ETL --> S3_DAL
    WH --> PG_DAL
    WH --> RP_DAL
    AP --> REDIS_DAL
    TMP --> PG_DAL
    NF --> REDIS_DAL
```

## 3. Package Structure

### 3.1 Repository Layout

```
ERP-iPaaS/
  services/                    # Go microservices
    workflow-engine/           # Main workflow orchestration service
    connector-framework/       # Connector lifecycle management
    event-backbone/            # Event streaming management
    api-management-service/    # API gateway management
    etl-service/               # ETL pipeline management
    webhook-service/           # Webhook management
  packages/                    # TypeScript/Go packages
    activepieces/              # Activepieces custom pieces
    connector-cli/             # Connector CLI tool
    integration-layer-ts/      # TypeScript SDK
    integration-layer-go/      # Go SDK
    llm-utils/                 # LLM utility functions
    mcp-host/                  # AI MCP gateway
    nexum-flow/                # Visual DAG engine
    temporal/                  # Temporal SDK wrapper
  activepieces/                # Activepieces config and pieces
    config/                    # Multitenancy, PII guards
    pieces/                    # Custom pieces (ClickHouse, ERPNext, CRM)
    templates/                 # 16 workflow templates
  temporal/                    # Temporal config and workflows
    src/workflows/             # Workflow definitions
    src/activities/            # Activity implementations
    workers/                   # Worker processes
    templates/                 # Temporal templates
  src/                         # Shared source code
    activepieces/pieces/       # Piece implementations
    ai-agent/                  # AI agent service
    lib/interops/              # Interop utilities
    lib/llm/                   # LLM prompts, redaction, retry
    temporal/workflows/        # Temporal workflow implementations
    temporal/workers/          # Worker implementations
  config/                      # Configuration
    clickhouse/                # DDL scripts
    grafana/                   # Dashboard definitions
    kafka/schemas/             # Avro schemas
    prometheus/rules/          # Alert rules
    security/                  # RLS policies, MinIO policies
  infra/                       # Infrastructure as Code
    helm/                      # 16 Helm charts
    terraform/                 # Terraform modules
    argocd/                    # ArgoCD applications
```

### 3.2 Service Internal Architecture

Each Go microservice follows a consistent pattern:

```mermaid
graph TB
    subgraph "Go Microservice Pattern"
        MAIN[main.go]
        MUX[http.ServeMux]
        HEALTH[/healthz handler]
        LIST[GET handler]
        CREATE[POST handler]
        READ[GET /:id handler]
        UPDATE[PATCH /:id handler]
        DELETE[DELETE /:id handler]
    end

    subgraph "Cross-Cutting Middleware"
        TENANT[X-Tenant-ID Validation]
        JSON[JSON Serialization]
        EVENT[Event Topic Tagging]
    end

    MAIN --> MUX
    MUX --> HEALTH
    MUX --> LIST
    MUX --> CREATE
    MUX --> READ
    MUX --> UPDATE
    MUX --> DELETE

    LIST --> TENANT
    CREATE --> TENANT
    READ --> TENANT
    UPDATE --> TENANT
    DELETE --> TENANT

    CREATE --> EVENT
    UPDATE --> EVENT
    DELETE --> EVENT

    LIST --> JSON
    CREATE --> JSON
    READ --> JSON
```

## 4. Workflow Runtime Architecture

### 4.1 Activepieces Runtime

```mermaid
graph TB
    subgraph "Activepieces Architecture"
        UI[Visual Builder UI]
        CTRL[Controller Pod]
        WRK[Worker Pod]
        PIECES[Custom Pieces]

        UI --> CTRL
        CTRL --> WRK
        WRK --> PIECES

        subgraph "Custom Pieces"
            CHP[ClickHouse Piece]
            ERPP[ERPNext Piece]
            CRMP[Internal CRM Piece]
        end

        PIECES --> CHP
        PIECES --> ERPP
        PIECES --> CRMP
    end

    subgraph "Shared Services"
        HC[HTTP Client<br/>_shared/http-client.ts]
        TC[Tenant Context<br/>_shared/tenant-context.ts]
    end

    CHP --> HC
    ERPP --> HC
    CRMP --> HC
    CHP --> TC
    ERPP --> TC
    CRMP --> TC
```

### 4.2 Temporal Runtime

```mermaid
graph TB
    subgraph "Temporal Architecture"
        FE[Frontend Service]
        HIST[History Service]
        MATCH[Matching Service]
        WRK[Worker Service]
    end

    subgraph "Workflows"
        LI[Lead Intake]
        ST[Support Triage]
        BTS[Batch to Stream]
        HAP[Human Approval]
    end

    subgraph "Activities"
        CRM_A[CRM Activities]
        LLM_A[LLM Activities]
        NOTIF[Notification Activities]
        THROT[Throttling Activities]
    end

    FE --> HIST
    FE --> MATCH
    MATCH --> WRK
    WRK --> LI
    WRK --> ST
    WRK --> BTS
    WRK --> HAP

    LI --> CRM_A
    LI --> LLM_A
    LI --> NOTIF
    ST --> LLM_A
    ST --> NOTIF
    HAP --> NOTIF
    BTS --> THROT
```

### 4.3 Nexum Flow DAG Engine

```mermaid
graph LR
    subgraph "Nexum Flow"
        ENG[NexumEngine]
        EXC[Execution Context]
        REC[Execution Record]
    end

    subgraph "Node Executors"
        HTTP[HTTP Node]
        LLM[LLM Node]
        MCP_N[MCP Node]
        TMP_N[Temporal Node]
    end

    subgraph "State"
        REDIS[Redis/Dragonfly]
        EVENTS[Event Emitter]
    end

    ENG --> EXC
    ENG --> REC
    ENG --> HTTP
    ENG --> LLM
    ENG --> MCP_N
    ENG --> TMP_N

    ENG --> REDIS
    ENG --> EVENTS
```

## 5. Communication Patterns

### 5.1 Synchronous Communication

- **REST API**: All service-to-client communication via OpenAPI-defined REST endpoints
- **gRPC**: Temporal client-to-server communication
- **GraphQL**: Schema defined in `schema.graphql` for query aggregation

### 5.2 Asynchronous Communication

- **Kafka/Redpanda**: All cross-service events via tenant-scoped topics
- **CloudEvents**: Standard event envelope format
- **Avro Schema Registry**: Schema validation on produce

### 5.3 Event Flow

```mermaid
sequenceDiagram
    participant WE as Workflow Engine
    participant RP as Redpanda
    participant SR as Schema Registry
    participant CH as ClickHouse
    participant AM as Alert Manager

    WE->>SR: Validate schema
    SR-->>WE: Schema valid
    WE->>RP: Produce event (tenant.123.workflow.created)
    RP->>CH: Consume + insert (runs table)
    RP->>AM: Consume + evaluate alert rules
    AM-->>AM: Check Prometheus rules
```

## 6. Error Handling Strategy

### 6.1 Retry Policies

| Component | Strategy | Max Attempts | Backoff |
|-----------|----------|-------------|---------|
| Temporal Workflows | Exponential backoff with jitter | 5 | 2x coefficient |
| Activepieces Actions | Configurable retry | 3 | Linear |
| Kafka Consumers | Retry topic + DLQ | 3 | Fixed interval |
| HTTP Clients | Circuit breaker | 3 | Exponential |
| Webhook Delivery | Exponential backoff | 5 | 2x with jitter |

### 6.2 Circuit Breaker Pattern

```mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open: Failure threshold exceeded
    Open --> HalfOpen: Timeout elapsed
    HalfOpen --> Closed: Probe succeeds
    HalfOpen --> Open: Probe fails
```

The circuit breaker is implemented in `src/temporal/workers/circuitBreaker.ts` and applied to all external HTTP calls.

## 7. Configuration Management

### 7.1 Configuration Hierarchy

```
Environment Variables (highest priority)
  -> Helm Values (per-chart)
    -> Kustomize Overlays (dev/prod)
      -> Default Values (lowest priority)
```

### 7.2 Key Configuration Sources

| Config | Location | Format |
|--------|----------|--------|
| Activepieces multitenancy | `activepieces/config/multitenancy.ts` | TypeScript |
| PII guards | `activepieces/config/pii-guards.ts` | TypeScript |
| ClickHouse DDL | `config/clickhouse/ddl.sql` | SQL |
| Kafka schemas | `config/kafka/schemas/` | Avro |
| Prometheus alerts | `config/prometheus/rules/` | YAML |
| Grafana dashboards | `config/grafana/dashboards/` | JSON |
| OPA constraints | `config/opa/` | YAML |
| Security policies | `config/security/` | SQL/JSON |

## 8. Dependency Injection and Modularity

The platform uses a modular architecture where each package is independently publishable:

```mermaid
graph TB
    subgraph "Published Packages"
        ILT[integration-layer-ts<br/>npm]
        ILG[integration-layer-go<br/>go module]
        CLI[connector-cli<br/>npm]
        NF[nexum-flow<br/>npm]
        MCP[mcp-host<br/>npm]
    end

    subgraph "Internal Packages"
        LLM[llm-utils]
        AP_PKG[activepieces]
        TMP_PKG[temporal]
    end

    NF --> LLM
    MCP --> NF
    CLI --> ILT
    AP_PKG --> LLM
    TMP_PKG --> LLM
```
