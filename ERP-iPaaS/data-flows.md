# Data Flows -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Overview

This document maps all data flows within ERP-iPaaS, covering ingress/egress paths, inter-service communication, event propagation, ETL pipelines, and data lineage tracking.

## 2. System-Level Data Flow

```mermaid
graph TB
    subgraph "External Sources"
        ERP[ERP Modules]
        SAAS[External SaaS]
        WH_IN[Incoming Webhooks]
        API_CL[API Clients]
    end

    subgraph "Ingress"
        TRF[Traefik Gateway<br/>TLS termination]
    end

    subgraph "Processing"
        WE[Workflow Engine]
        CF[Connector Framework]
        EB[Event Backbone]
        AM[API Management]
        ETL[ETL Service]
        WHM[Webhook Manager]
    end

    subgraph "Runtimes"
        AP[Activepieces]
        TMP[Temporal]
    end

    subgraph "Data Stores"
        PG[(PostgreSQL<br/>Operational)]
        CH[(ClickHouse<br/>Analytics)]
        RP[Redpanda<br/>Events]
        MIO[(MinIO<br/>Objects)]
        DF[Dragonfly<br/>Cache]
    end

    subgraph "Egress"
        EXT_API[External APIs]
        EXT_WH[Outgoing Webhooks]
        NOTIFY[Notifications<br/>Slack/Email]
    end

    ERP --> TRF
    SAAS --> TRF
    WH_IN --> TRF
    API_CL --> TRF

    TRF --> WE
    TRF --> CF
    TRF --> EB
    TRF --> AM
    TRF --> ETL
    TRF --> WHM

    WE --> AP
    WE --> TMP
    WE --> PG
    WE --> RP
    WE --> CH

    CF --> PG
    CF --> RP
    EB --> RP
    EB --> CH
    ETL --> PG
    ETL --> CH
    ETL --> MIO
    WHM --> PG
    WHM --> RP

    AP --> DF
    TMP --> PG
    AP --> EXT_API
    TMP --> EXT_API
    WHM --> EXT_WH
    AP --> NOTIFY
```

## 3. Workflow Execution Data Flow

### 3.1 Activepieces Execution

```mermaid
sequenceDiagram
    participant TRG as Trigger Source
    participant TRF as Traefik
    participant WE as Workflow Engine
    participant AP as Activepieces
    participant PIECE as Custom Piece
    participant RP as Redpanda
    participant CH as ClickHouse
    participant PG as PostgreSQL

    TRG->>TRF: Trigger event (webhook/schedule)
    TRF->>WE: Route to workflow engine
    WE->>PG: Load workflow definition
    WE->>AP: Execute Activepieces flow

    loop For each piece in flow
        AP->>PIECE: Execute action
        PIECE->>PIECE: Process data
        PIECE-->>AP: Action result
        AP->>RP: Emit step event
    end

    AP-->>WE: Flow completed
    WE->>CH: INSERT INTO runs
    WE->>RP: Publish workflow.completed
```

### 3.2 Temporal Execution

```mermaid
sequenceDiagram
    participant TRG as Trigger
    participant WE as Workflow Engine
    participant TMP as Temporal Server
    participant WRK as Worker
    participant ACT as Activity
    participant PG as PostgreSQL
    participant CH as ClickHouse

    TRG->>WE: Start workflow request
    WE->>TMP: StartWorkflowExecution
    TMP->>PG: Persist workflow state
    TMP->>WRK: Schedule task

    loop For each activity
        WRK->>ACT: Execute activity
        ACT->>ACT: Business logic
        ACT-->>WRK: Result
        WRK->>TMP: Report completion
        TMP->>PG: Update execution state
    end

    TMP->>WE: Workflow completed
    WE->>CH: INSERT INTO runs
```

## 4. Event Data Flow

### 4.1 Event Production Pipeline

```mermaid
graph LR
    subgraph "Producers"
        P1[Service Layer]
        P2[Workflow Runtime]
        P3[External Webhook]
    end

    subgraph "Schema Validation"
        SR[Redpanda Schema Registry]
    end

    subgraph "Redpanda Cluster"
        T1[tenant.{id}.workflow.events]
        T2[tenant.{id}.connector.events]
        T3[tenant.{id}.etl.events]
        T4[tenant.{id}.webhook.events]
    end

    P1 --> SR
    P2 --> SR
    P3 --> SR
    SR --> T1
    SR --> T2
    SR --> T3
    SR --> T4
```

### 4.2 Event Consumption Pipeline

```mermaid
graph LR
    subgraph "Topics"
        T[tenant.{id}.*]
    end

    subgraph "Consumer Groups"
        CG1[clickhouse-sink<br/>Analytics ingestion]
        CG2[workflow-trigger<br/>Reactive workflows]
        CG3[alert-evaluator<br/>Prometheus alerting]
        CG4[audit-logger<br/>Compliance logging]
        CG5[notification-dispatcher<br/>Slack/Email]
    end

    T --> CG1
    T --> CG2
    T --> CG3
    T --> CG4
    T --> CG5
```

### 4.3 Dead Letter Queue Flow

```mermaid
graph TD
    MSG[Event Message] --> CON{Consumer<br/>Processing}
    CON -->|Success| ACK[Commit Offset]
    CON -->|Failure| RT1[Retry 1<br/>1s delay]
    RT1 --> CON
    CON -->|Failure| RT2[Retry 2<br/>5s delay]
    RT2 --> CON
    CON -->|Failure| DLQ[tenant.{id}.dlq]
    DLQ --> MON[Monitoring Alert]
    DLQ --> RPL[Manual Replay]
    RPL --> MSG
```

## 5. ETL Data Flow

### 5.1 Batch ETL Pipeline

```mermaid
graph LR
    subgraph "Extract"
        SRC_DB[(Source DB<br/>PostgreSQL/MySQL)]
        SRC_API[Source API<br/>REST/GraphQL]
        SRC_FILE[Source File<br/>CSV/JSON/Parquet]
    end

    subgraph "Transform"
        MAP[Field Mapping]
        FIL[Filtering]
        AGG[Aggregation]
        ENR[Enrichment]
        DED[Deduplication]
    end

    subgraph "Load"
        TGT_DB[(Target DB<br/>ClickHouse)]
        TGT_API[Target API]
        TGT_FILE[(Target File<br/>MinIO)]
    end

    SRC_DB --> MAP
    SRC_API --> MAP
    SRC_FILE --> MAP

    MAP --> FIL --> AGG --> ENR --> DED

    DED --> TGT_DB
    DED --> TGT_API
    DED --> TGT_FILE
```

### 5.2 CDC (Streaming) Pipeline

```mermaid
graph LR
    PG[(PostgreSQL<br/>Source)] --> DBZ[Debezium<br/>CDC Connector]
    DBZ --> RP[Redpanda<br/>Change Events]
    RP --> XFORM[Stream Processor<br/>Transform]
    XFORM --> CH[(ClickHouse<br/>Target)]
    XFORM --> NOTIFY[Downstream<br/>Notifications]
```

## 6. Connector Data Flow

### 6.1 Connector Lifecycle

```mermaid
graph TD
    DEV[Developer] --> |"scaffold"| CLI[Connector CLI]
    CLI --> |"validate"| SR[Schema Registry]
    CLI --> |"publish"| MKT[(Marketplace<br/>ClickHouse)]
    MKT --> |"install"| AP[Activepieces Runtime]
    AP --> |"execute"| EXT[External System]
    EXT --> |"response"| AP
    AP --> |"metrics"| CH[(ClickHouse<br/>connector_latency)]
```

### 6.2 Connector Authentication Flow

```mermaid
sequenceDiagram
    participant WF as Workflow
    participant CON as Connector
    participant SEC as Secrets API
    participant EXT as External API

    WF->>CON: Execute action
    CON->>SEC: Fetch credentials (encrypted)
    SEC-->>CON: Decrypted credentials

    alt OAuth2
        CON->>EXT: Token refresh if expired
        EXT-->>CON: New access token
        CON->>EXT: API call with Bearer token
    else API Key
        CON->>EXT: API call with X-API-Key header
    else Basic Auth
        CON->>EXT: API call with Authorization: Basic
    end

    EXT-->>CON: Response
    CON-->>WF: Action result
```

## 7. Webhook Data Flow

### 7.1 Incoming Webhook

```mermaid
sequenceDiagram
    participant EXT as External System
    participant TRF as Traefik
    participant WH as Webhook Service
    participant SIG as Signature Verifier
    participant WE as Workflow Engine
    participant LOG as Delivery Log

    EXT->>TRF: POST /webhook/{endpoint-id}
    TRF->>WH: Forward request
    WH->>SIG: Verify HMAC-SHA256 signature
    SIG-->>WH: Signature valid
    WH->>LOG: Log delivery (request body, headers)
    WH->>WE: Trigger associated workflow
    WH-->>EXT: 200 OK
```

### 7.2 Outgoing Webhook

```mermaid
sequenceDiagram
    participant WF as Workflow
    participant WH as Webhook Service
    participant SIGN as Signer
    participant EXT as External Endpoint
    participant DLQ as Retry Queue

    WF->>WH: Dispatch webhook
    WH->>SIGN: Sign payload (HMAC-SHA256)
    SIGN-->>WH: Signature header
    WH->>EXT: POST with signature header

    alt Success (2xx)
        EXT-->>WH: 200 OK
        WH->>WH: Log successful delivery
    else Failure (4xx/5xx/timeout)
        EXT-->>WH: Error
        WH->>DLQ: Enqueue for retry
        Note over DLQ: Retry 1: 1s, Retry 2: 5s, Retry 3: 25s, Retry 4: 125s, Retry 5: DLQ
    end
```

## 8. Data Lineage

```mermaid
graph LR
    subgraph "Lineage Tracking"
        SRC[Source System] --> |"trace_id"| EVT[Event]
        EVT --> |"trace_id"| WF[Workflow Run]
        WF --> |"trace_id"| ACT[Activity]
        ACT --> |"trace_id"| TGT[Target System]
    end

    subgraph "Correlation"
        ALL[All records share trace_id<br/>via OpenTelemetry propagation]
    end
```

Every data movement carries an OpenTelemetry `trace_id` that enables end-to-end lineage tracking from source event to final destination. Traces are stored in Tempo and correlated with Loki logs and Prometheus metrics in Grafana.
