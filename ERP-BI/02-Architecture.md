# ERP-BI Architecture Document

| Field | Value |
|---|---|
| Module | ERP-BI (Business Intelligence & Analytics) |
| Version | 1.0.0 |
| Status | Approved |
| Last Updated | 2026-02-23 |

---

## 1. Architecture Overview

ERP-BI follows a microservices architecture with seven distinct services, a Next.js frontend, and ClickHouse as the primary OLAP engine. The module integrates with the broader ERP platform through NATS event backbone, ERP-IAM for identity/authorization, and ERP-Platform for subscription entitlements.

```mermaid
graph TB
    subgraph "Client Tier"
        Browser[Browser / Mobile]
        Embed[Embedded iFrame]
        API_Client[API Consumer]
    end

    subgraph "Edge"
        CDN[CDN / Edge Cache]
        LB[Load Balancer]
    end

    subgraph "Presentation Tier"
        NextJS[Next.js App<br/>TypeScript + Tailwind CSS<br/>React 18 / Recharts]
    end

    subgraph "Service Tier"
        direction TB
        GW[API Gateway]
        DS[Dashboard Service<br/>Go :8080]
        RS[Report Service<br/>Go :8081]
        DMS[Data Modeling Service<br/>Go :8082]
        QE[Query Engine<br/>Go :8083]
        DWS[Data Warehouse Service<br/>Go :8084]
        AS[Alert Service<br/>Go :8085]
        NLQ[NLQ Service<br/>Go :8086]
    end

    subgraph "Data Tier"
        CH[(ClickHouse Cluster<br/>OLAP)]
        PG[(PostgreSQL<br/>Metadata)]
        Redis[(Redis Cluster<br/>Cache)]
        S3[(Object Storage<br/>Exports)]
    end

    subgraph "Platform Tier"
        NATS[NATS JetStream<br/>Event Backbone]
        IAM[ERP-IAM<br/>Identity & Access]
        PLAT[ERP-Platform<br/>Entitlements]
        AI[ERP-AI<br/>Claude / ML]
    end

    Browser --> CDN --> LB --> NextJS
    Embed --> LB
    API_Client --> LB
    NextJS --> GW
    GW --> DS
    GW --> RS
    GW --> DMS
    GW --> QE
    GW --> DWS
    GW --> AS
    GW --> NLQ

    DS --> QE
    RS --> QE
    NLQ --> QE
    NLQ --> AI
    DMS --> PG
    QE --> CH
    QE --> Redis
    DWS --> CH
    DWS --> PG
    AS --> QE
    RS --> S3

    DS --> NATS
    RS --> NATS
    DWS --> NATS
    AS --> NATS
    GW --> IAM
    GW --> PLAT
```

---

## 2. Service Decomposition

### 2.1 Dashboard Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/dashboard` |
| Port | 8080 |
| Health Check | `/healthz` |
| Event Topics | `erp.bi.dashboard.{created,updated,deleted,listed,read}` |

**Responsibilities**: Dashboard CRUD, widget layout management, real-time subscription management, template library, embedding token generation, white-label configuration.

**Key Endpoints**:
- `GET /v1/dashboard` - List dashboards (paginated, filtered by tenant)
- `POST /v1/dashboard` - Create dashboard
- `GET /v1/dashboard/{id}` - Get dashboard by ID
- `PUT /v1/dashboard/{id}` - Update dashboard
- `DELETE /v1/dashboard/{id}` - Delete dashboard

### 2.2 Report Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/report` |
| Port | 8081 |
| Event Topics | `erp.bi.report.{created,updated,deleted,listed,read}` |

**Responsibilities**: Report definition CRUD, report execution, scheduling, delivery (email/Slack/webhook), export rendering (PDF/Excel/CSV/PowerPoint), sub-report resolution, parameter injection.

### 2.3 Data Modeling Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/data-modeling` |
| Port | 8082 |
| Event Topics | `erp.bi.data-modeling.{created,updated,deleted,listed,read}` |

**Responsibilities**: Semantic model management, calculated field definitions, measure/dimension registry, hierarchy configuration, data blending rules, RLS policy binding.

### 2.4 Query Engine

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/query-engine` |
| Port | 8083 |
| Event Topics | `erp.bi.query-engine.{created,updated,deleted,listed,read}` |

**Responsibilities**: SQL generation from semantic models, ClickHouse query execution, materialized view management, pre-aggregation scheduling, query caching (L1/L2), governor enforcement, query plan optimization.

### 2.5 Data Warehouse Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/data-warehouse` |
| Port | 8084 |
| Event Topics | `erp.bi.data-warehouse.{created,updated,deleted,listed,read}` |

**Responsibilities**: CDC event consumption from all ERP modules, ETL pipeline orchestration, ClickHouse schema management, data lineage tracking, data quality monitoring, incremental ingestion.

### 2.6 Alert Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/alert` |
| Port | 8085 |
| Event Topics | `erp.bi.alert.{created,updated,deleted,listed,read}` |

**Responsibilities**: Alert rule CRUD, threshold evaluation, AI anomaly detection integration, trend analysis, scheduled/event-driven evaluation, notification dispatch, escalation management.

### 2.7 NLQ Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/nlq` |
| Port | 8086 |
| Event Topics | `erp.bi.nlq.{created,updated,deleted,listed,read}` |

**Responsibilities**: Natural language parsing, intent classification, text-to-SQL translation (via Claude), chart type recommendation, query history, SQL validation and sanitization.

---

## 3. Data Architecture

### 3.1 ClickHouse Schema Strategy

```mermaid
graph LR
    subgraph "Star Schema - Sales Analytics"
        FS[fact_sales<br/>order_id, date_key, product_key,<br/>customer_key, amount, qty]
        DD[dim_date<br/>date_key, year, quarter,<br/>month, week, day]
        DP[dim_product<br/>product_key, name, category,<br/>subcategory, brand]
        DC[dim_customer<br/>customer_key, name,<br/>segment, region]
    end

    FS --> DD
    FS --> DP
    FS --> DC
```

**Storage Engine**: `MergeTree` family with `ReplacingMergeTree` for dimension tables and `SummingMergeTree` for pre-aggregation tables.

**Partitioning**: Monthly partitions on fact tables by date column.

**Sharding**: Distributed tables across ClickHouse cluster nodes for horizontal scaling.

### 3.2 CDC Pipeline

```mermaid
sequenceDiagram
    participant ERP as ERP Module
    participant NATS as NATS JetStream
    participant DWS as Data Warehouse Service
    participant CH as ClickHouse
    participant QM as Quality Monitor

    ERP->>NATS: Publish CDC event<br/>(erp.{module}.{entity}.{action})
    NATS->>DWS: Deliver event (JetStream)
    DWS->>DWS: Transform & validate
    DWS->>CH: Upsert into staging table
    DWS->>CH: Merge into fact/dim table
    DWS->>QM: Quality check
    QM-->>DWS: Quality score
    DWS->>NATS: Publish lineage event
```

### 3.3 Query Caching Strategy

| Cache Tier | Technology | TTL | Scope |
|---|---|---|---|
| L1 - In-Process | Go map + sync.RWMutex | 30 seconds | Per-service instance |
| L2 - Distributed | Redis Cluster | 5 minutes | Cross-instance |
| L3 - Materialized | ClickHouse MV | Configured refresh | Global |

---

## 4. Frontend Architecture

### 4.1 Technology Stack

| Technology | Purpose |
|---|---|
| Next.js 14 | React framework with App Router |
| TypeScript 5.5 | Type safety |
| Tailwind CSS 3.4 | Utility-first styling |
| Recharts 2.12 | Chart rendering |
| TanStack Query 5.45 | Server state management |
| Zustand 4.5 | Client state management |
| Prisma 5.15 | ORM for PostgreSQL metadata |
| Zod 3.23 | Schema validation |
| Lucide React | Icon library |

### 4.2 Component Hierarchy

```mermaid
graph TD
    RootLayout[RootLayout<br/>layout.tsx]
    Providers[Providers<br/>QueryClient + Zustand]
    Sidebar[Sidebar Navigation]
    Header[Header with Search]

    RootLayout --> Providers
    Providers --> Sidebar
    Providers --> Header
    Providers --> PageContent

    subgraph "Page Content"
        PageContent[Dynamic Page]
        Dashboard[Dashboard Page]
        Products[Products Page]
        WorkOrders[Work Orders Page]
        Quality[Quality Page]
        AIInsights[AI Insights Page]
        BOM[BOM Page]
        ShopFloor[Shop Floor Page]
        Capacity[Capacity Page]
        Settings[Settings Page]
    end

    subgraph "UI Components"
        KPICard[KPI Card]
        DataTable[Data Table]
        StatusBadge[Status Badge]
        AIInsightCard[AI Insight Card]
        ProgressBar[Progress Bar]
        Modal[Modal]
    end

    Dashboard --> KPICard
    Dashboard --> AIInsightCard
    Dashboard --> ProgressBar
    WorkOrders --> DataTable
    WorkOrders --> StatusBadge
```

---

## 5. Integration Architecture

### 5.1 Multi-Tenant Isolation

Every API request requires `X-Tenant-ID` header. The service validates tenant context against ERP-IAM tokens and enforces:

1. **Schema-level isolation**: Separate ClickHouse databases per tenant for enterprise tier
2. **Row-level isolation**: Tenant ID column filter for shared-schema deployments
3. **Cache-level isolation**: Redis key namespacing by tenant ID

### 5.2 Event Topology

| Event Pattern | Example | Purpose |
|---|---|---|
| `erp.bi.{service}.created` | `erp.bi.dashboard.created` | Resource lifecycle events |
| `erp.bi.{service}.updated` | `erp.bi.report.updated` | Audit trail |
| `erp.{module}.{entity}.{action}` | `erp.finance.invoice.created` | CDC ingestion trigger |
| `erp.bi.alert.triggered` | - | Alert notification dispatch |

### 5.3 Platform Integration

```mermaid
graph LR
    BI[ERP-BI] --> |Authentication| IAM[ERP-IAM]
    BI --> |Entitlements| PLAT[ERP-Platform]
    BI --> |NLQ / Anomaly| AI[ERP-AI]
    BI --> |Events| NATS[NATS JetStream]

    subgraph "CDC Sources"
        FIN[ERP-Finance]
        CRM[ERP-CRM]
        HCM[ERP-HCM]
        SCM[ERP-SCM]
        COM[ERP-Commerce]
    end

    FIN --> NATS
    CRM --> NATS
    HCM --> NATS
    SCM --> NATS
    COM --> NATS
    NATS --> BI
```

---

## 6. Deployment Architecture

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "BI Namespace"
            DS_D[Dashboard Service<br/>Deployment x3]
            RS_D[Report Service<br/>Deployment x2]
            DMS_D[Data Modeling Service<br/>Deployment x2]
            QE_D[Query Engine<br/>Deployment x4]
            DWS_D[Data Warehouse Service<br/>Deployment x2]
            AS_D[Alert Service<br/>Deployment x2]
            NLQ_D[NLQ Service<br/>Deployment x2]
            UI_D[Next.js UI<br/>Deployment x3]
        end

        subgraph "Data Namespace"
            CH_S[ClickHouse<br/>StatefulSet x3]
            PG_S[PostgreSQL<br/>StatefulSet x2]
            Redis_S[Redis Cluster<br/>StatefulSet x3]
        end
    end

    subgraph "External"
        S3_E[S3 Object Storage]
        SMTP_E[SMTP Relay]
    end
```

**Container Registry**: Harbor / ECR
**CI/CD**: GitHub Actions with Makefile targets (`make test`, `make test-integration`, `make test-e2e`)
**Observability**: OpenTelemetry traces, Prometheus metrics, Loki logs

---

## 7. Security Architecture

| Layer | Mechanism |
|---|---|
| Network | mTLS between services, NetworkPolicies |
| Authentication | JWT tokens from ERP-IAM, X-Tenant-ID validation |
| Authorization | RBAC + ABAC policies, RLS enforcement |
| Data | AES-256 encryption at rest, TLS 1.3 in transit |
| Query | Read-only mode for NLQ, SQL injection prevention |
| Audit | Full audit trail via NATS events |
| Compliance | AIDD guardrails enforcement (erp/aidd.guardrails.yaml) |

---

## 8. Scalability Strategy

| Dimension | Approach |
|---|---|
| Read throughput | Horizontal scaling of Query Engine, ClickHouse read replicas |
| Write throughput | CDC batching, ClickHouse async inserts |
| Storage | ClickHouse partitioning, S3 cold storage tiering |
| Concurrency | Governor limits, connection pooling, request queuing |
| Cache | Redis Cluster with consistent hashing |
