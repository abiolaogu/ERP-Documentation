# ERP-BI Data Flow Documentation

| Field | Value |
|---|---|
| Module | ERP-BI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. End-to-End Data Flow

```mermaid
graph TB
    subgraph "Source Systems (ERP Modules)"
        FIN[ERP-Finance<br/>Invoices, Payments, GL]
        CRM[ERP-CRM<br/>Leads, Opportunities, Accounts]
        HCM[ERP-HCM<br/>Employees, Payroll, Time]
        SCM[ERP-SCM<br/>POs, Inventory, Suppliers]
        COM[ERP-Commerce<br/>Orders, Products, Customers]
        MFG[ERP-BI Internal<br/>WorkOrders, Quality, BOM]
    end

    subgraph "Event Transport"
        NATS[NATS JetStream<br/>CDC Events]
    end

    subgraph "Ingestion Layer"
        CDC[CDC Consumer<br/>Data Warehouse Service]
        VAL[Validator<br/>Schema + Quality]
        DLQ[Dead Letter Queue]
    end

    subgraph "Storage Layer"
        STG[(Staging Tables<br/>ClickHouse)]
        FACT[(Fact Tables<br/>ClickHouse)]
        DIM[(Dimension Tables<br/>ClickHouse)]
        MV[(Materialized Views<br/>ClickHouse)]
        META[(Metadata<br/>PostgreSQL)]
    end

    subgraph "Semantic Layer"
        SEM[Semantic Models<br/>Data Modeling Service]
        RLS[RLS Policies<br/>ERP-IAM Bound]
    end

    subgraph "Query Layer"
        QE[Query Engine<br/>SQL Generation]
        CACHE[Redis Cache<br/>L1 + L2]
    end

    subgraph "Consumption Layer"
        DASH[Dashboards]
        RPT[Reports]
        NLQ_S[NLQ Answers]
        ALT[Alerts]
        EMBED[Embedded Analytics]
        API_C[API Consumers]
    end

    FIN --> NATS
    CRM --> NATS
    HCM --> NATS
    SCM --> NATS
    COM --> NATS
    MFG --> NATS

    NATS --> CDC
    CDC --> VAL
    VAL -->|Pass| STG
    VAL -->|Fail| DLQ
    STG --> FACT
    STG --> DIM
    FACT --> MV

    SEM --> QE
    RLS --> QE
    QE --> CACHE
    QE --> FACT
    QE --> DIM
    QE --> MV

    CACHE --> DASH
    CACHE --> RPT
    CACHE --> NLQ_S
    QE --> ALT
    QE --> EMBED
    QE --> API_C
```

---

## 2. CDC Event Flow

### 2.1 Event Structure

Each CDC event from an ERP module follows the canonical format:

```json
{
  "event_id": "evt_abc123",
  "event_type": "erp.finance.invoice.created",
  "tenant_id": "tenant_001",
  "timestamp": "2026-02-23T10:00:00Z",
  "version": "1.0",
  "payload": {
    "id": "inv_789",
    "amount": 15000.00,
    "currency": "USD",
    "customer_id": "cust_456",
    "line_items": [...]
  },
  "metadata": {
    "source_module": "ERP-Finance",
    "source_entity": "invoice",
    "action": "created",
    "correlation_id": "corr_xyz"
  }
}
```

### 2.2 Event Processing Pipeline

```mermaid
sequenceDiagram
    participant SRC as Source Module
    participant NATS as NATS JetStream
    participant DWS as Data Warehouse Service
    participant STG as ClickHouse Staging
    participant TGT as ClickHouse Target
    participant LIN as Lineage Tracker
    participant QM as Quality Monitor

    SRC->>NATS: Publish CDC event
    Note over NATS: Stream: erp.cdc.*<br/>Consumer: bi-ingestion
    NATS->>DWS: Deliver (ack required)

    DWS->>DWS: Deserialize & validate schema
    DWS->>STG: INSERT INTO staging_buffer

    loop Micro-batch (every 5s)
        DWS->>STG: SELECT from staging_buffer
        DWS->>DWS: Transform (column mapping,<br/>type conversion, derivations)
        DWS->>TGT: INSERT INTO fact/dim table
        DWS->>LIN: Record lineage edge
        DWS->>QM: Run quality checks
        QM-->>DWS: Quality score
    end

    DWS->>NATS: ACK batch
    DWS->>NATS: Publish erp.bi.ingestion.completed
```

---

## 3. Query Data Flow

### 3.1 Dashboard Query Flow

```mermaid
sequenceDiagram
    participant Browser
    participant UI as Next.js Frontend
    participant DS as Dashboard Service
    participant QE as Query Engine
    participant L1 as L1 Cache
    participant L2 as Redis
    participant CH as ClickHouse
    participant IAM as ERP-IAM

    Browser->>UI: Load dashboard
    UI->>DS: GET /v1/dashboard/{id}
    DS->>IAM: Validate JWT + tenant
    IAM-->>DS: User context + RLS policies
    DS-->>UI: Dashboard definition (widgets)

    loop For each widget
        UI->>QE: POST /v1/query-engine
        QE->>QE: Build cache key (model+dims+measures+filters+tenant)
        QE->>L1: GET cache_key
        alt L1 hit
            L1-->>QE: Cached result
        else L1 miss
            QE->>L2: GET cache_key
            alt L2 hit
                L2-->>QE: Cached result
                QE->>L1: SET (TTL 30s)
            else L2 miss
                QE->>QE: Generate SQL from semantic model
                QE->>QE: Inject RLS WHERE clause
                QE->>CH: Execute SQL
                CH-->>QE: Result set
                QE->>L2: SET (TTL 5min)
                QE->>L1: SET (TTL 30s)
            end
        end
        QE-->>UI: Query result
    end

    UI-->>Browser: Render charts
```

### 3.2 NLQ Data Flow

```mermaid
sequenceDiagram
    participant User
    participant UI as Next.js Frontend
    participant NLQ as NLQ Service
    participant AI as ERP-AI (Claude)
    participant QE as Query Engine
    participant CH as ClickHouse

    User->>UI: "What were top products last month?"
    UI->>NLQ: POST /v1/nlq
    NLQ->>NLQ: Parse intent + extract entities
    NLQ->>NLQ: Assemble schema context
    NLQ->>AI: Prompt with schema + question
    AI-->>NLQ: Generated SQL
    NLQ->>NLQ: Validate SQL (read-only, no injection)
    NLQ->>QE: Execute validated SQL
    QE->>CH: Run query
    CH-->>QE: Result set
    QE-->>NLQ: Results
    NLQ->>NLQ: Select chart type based on result shape
    NLQ-->>UI: SQL + results + chart suggestion
    UI-->>User: Rendered visualization
```

---

## 4. Alert Data Flow

```mermaid
sequenceDiagram
    participant CRON as Cron Scheduler
    participant AS as Alert Service
    participant QE as Query Engine
    participant AI as Anomaly Detector
    participant NATS as NATS
    participant EMAIL as Email Service
    participant SLACK as Slack API
    participant WH as Webhook

    CRON->>AS: Trigger evaluation cycle
    AS->>AS: Load active alert rules

    loop For each rule
        AS->>QE: Execute metric query
        QE-->>AS: Current metric value

        alt Threshold Alert
            AS->>AS: Compare value vs threshold
        else Anomaly Alert
            AS->>AI: Run anomaly detection
            AI-->>AS: Anomaly score + type
        else Trend Alert
            AS->>AS: Calculate trend over window
        end

        alt Alert triggered
            AS->>NATS: Publish erp.bi.alert.triggered
            AS->>EMAIL: Send notification
            AS->>SLACK: Post message
            AS->>WH: POST payload
            AS->>AS: Start escalation timer
        end
    end
```

---

## 5. Report Delivery Data Flow

```mermaid
sequenceDiagram
    participant SCHED as Scheduler
    participant RS as Report Service
    participant QE as Query Engine
    participant RENDER as Renderer
    participant S3 as Object Storage
    participant DELIVER as Delivery Engine
    participant USER as User

    SCHED->>RS: Trigger scheduled report
    RS->>RS: Resolve parameters
    RS->>RS: Expand sub-reports
    RS->>QE: Execute data queries
    QE-->>RS: Result sets

    RS->>RENDER: Render to target format
    alt PDF
        RENDER->>RENDER: HTML to PDF (headless Chrome)
    else Excel
        RENDER->>RENDER: XLSX generation
    else CSV
        RENDER->>RENDER: CSV serialization
    else PowerPoint
        RENDER->>RENDER: PPTX generation
    end

    RENDER->>S3: Upload rendered file
    S3-->>RS: File URL

    RS->>DELIVER: Dispatch deliveries
    DELIVER->>USER: Email with attachment/link
    DELIVER->>USER: Slack message with link
    DELIVER->>USER: Webhook POST with URL
```

---

## 6. Data Lineage Tracking

```mermaid
graph LR
    subgraph "Source"
        S1[ERP-Finance.invoice]
        S2[ERP-CRM.opportunity]
    end

    subgraph "Staging"
        ST1[stg_invoice]
        ST2[stg_opportunity]
    end

    subgraph "Target"
        F1[fact_revenue]
        D1[dim_customer]
    end

    subgraph "Semantic"
        M1[Revenue Model]
    end

    subgraph "Consumption"
        DA[Sales Dashboard]
        RP[Revenue Report]
    end

    S1 --> ST1 --> F1
    S2 --> ST2 --> F1
    S2 --> ST2 --> D1
    F1 --> M1
    D1 --> M1
    M1 --> DA
    M1 --> RP
```

Every transformation step records a lineage edge in the metadata database, enabling full provenance tracking from source ERP record to final dashboard widget.
