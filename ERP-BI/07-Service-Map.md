# ERP-BI Service Map

| Field | Value |
|---|---|
| Module | ERP-BI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Service Topology

```mermaid
graph TB
    subgraph "ERP-BI Service Mesh"
        DS[Dashboard Service<br/>:8080<br/>Go 1.22]
        RS[Report Service<br/>:8081<br/>Go 1.22]
        DMS[Data Modeling Service<br/>:8082<br/>Go 1.22]
        QE[Query Engine<br/>:8083<br/>Go 1.22]
        DWS[Data Warehouse Service<br/>:8084<br/>Go 1.22]
        AS[Alert Service<br/>:8085<br/>Go 1.22]
        NLQ[NLQ Service<br/>:8086<br/>Go 1.22]
    end

    subgraph "Frontend"
        UI[Next.js Application<br/>:3000<br/>TypeScript]
    end

    subgraph "Data Stores"
        CH[(ClickHouse<br/>:8123 HTTP / :9000 Native)]
        PG[(PostgreSQL<br/>:5432)]
        Redis[(Redis Cluster<br/>:6379)]
        S3[(Object Storage)]
    end

    subgraph "Platform Services"
        NATS[NATS JetStream<br/>:4222]
        IAM[ERP-IAM]
        PLAT[ERP-Platform]
        AI[ERP-AI]
    end

    UI --> DS & RS & NLQ
    DS --> QE
    RS --> QE
    NLQ --> QE & AI
    DMS --> PG
    QE --> CH & Redis
    DWS --> CH & PG & NATS
    AS --> QE & NATS
    RS --> S3
    DS & RS & AS --> NATS
```

---

## 2. Service Registry

| Service | Port | Base Path | Health | Language | Replicas |
|---|---|---|---|---|---|
| dashboard-service | 8080 | /v1/dashboard | /healthz | Go 1.22 | 3 |
| report-service | 8081 | /v1/report | /healthz | Go 1.22 | 2 |
| data-modeling-service | 8082 | /v1/data-modeling | /healthz | Go 1.22 | 2 |
| query-engine | 8083 | /v1/query-engine | /healthz | Go 1.22 | 4 |
| data-warehouse-service | 8084 | /v1/data-warehouse | /healthz | Go 1.22 | 2 |
| alert-service | 8085 | /v1/alert | /healthz | Go 1.22 | 2 |
| nlq-service | 8086 | /v1/nlq | /healthz | Go 1.22 | 2 |
| nextjs-ui | 3000 | / | /api/health | TypeScript | 3 |

---

## 3. Service Dependencies

```mermaid
graph LR
    DS[Dashboard] --> QE[Query Engine]
    RS[Report] --> QE
    NLQ[NLQ] --> QE
    NLQ --> AI[ERP-AI]
    AS[Alert] --> QE
    QE --> CH[(ClickHouse)]
    QE --> Redis[(Redis)]
    DMS[Data Modeling] --> PG[(PostgreSQL)]
    DWS[Data Warehouse] --> CH
    DWS --> PG
    DWS --> NATS[NATS]

    style QE fill:#f9f,stroke:#333,stroke-width:2px
    style CH fill:#ff9,stroke:#333,stroke-width:2px
```

**Critical path**: Query Engine is the most critical service -- all consumer services depend on it. ClickHouse is the single OLAP data store.

---

## 4. Inter-Service Communication

| From | To | Protocol | Pattern |
|---|---|---|---|
| Dashboard Service | Query Engine | HTTP/REST | Synchronous |
| Report Service | Query Engine | HTTP/REST | Synchronous |
| NLQ Service | Query Engine | HTTP/REST | Synchronous |
| NLQ Service | ERP-AI | HTTP/REST | Synchronous |
| Alert Service | Query Engine | HTTP/REST | Synchronous |
| Data Warehouse Service | NATS | NATS protocol | Async (consumer) |
| Dashboard Service | NATS | NATS protocol | Async (publisher) |
| Report Service | NATS | NATS protocol | Async (publisher) |
| Alert Service | NATS | NATS protocol | Async (pub/sub) |
| All services | ERP-IAM | HTTP/REST | Synchronous (auth) |

---

## 5. Event Topics

| Topic | Publisher | Subscribers | Description |
|---|---|---|---|
| erp.bi.dashboard.created | Dashboard Service | Audit, Analytics | Dashboard created |
| erp.bi.dashboard.updated | Dashboard Service | Audit, Cache invalidation | Dashboard modified |
| erp.bi.dashboard.deleted | Dashboard Service | Audit, Cleanup | Dashboard removed |
| erp.bi.report.created | Report Service | Audit, Scheduler | Report definition created |
| erp.bi.report.executed | Report Service | Audit, Analytics | Report execution completed |
| erp.bi.alert.triggered | Alert Service | Notification dispatch | Alert condition met |
| erp.bi.data-warehouse.sync.completed | Data Warehouse Service | Cache invalidation, Reports | ETL batch completed |
| erp.bi.nlq.created | NLQ Service | Audit, Analytics | NLQ query executed |
| erp.{module}.{entity}.{action} | ERP Modules | Data Warehouse Service | CDC events |

---

## 6. Failure Domains

```mermaid
graph TB
    subgraph "Domain 1: Query Path"
        QE_F[Query Engine Failure]
        CH_F[ClickHouse Failure]
        Redis_F[Redis Failure]
    end

    subgraph "Domain 2: Ingestion Path"
        DWS_F[Data Warehouse Failure]
        NATS_F[NATS Failure]
    end

    subgraph "Domain 3: Presentation"
        UI_F[UI Failure]
        DS_F[Dashboard Service Failure]
    end

    QE_F -->|Impact| DS_F
    QE_F -->|Impact| RS_F[Report Service]
    QE_F -->|Impact| NLQ_F[NLQ Service]
    CH_F -->|Impact| QE_F
    Redis_F -->|Degraded| QE_F
    NATS_F -->|Stale data| DWS_F
```

| Failure | Impact | Mitigation |
|---|---|---|
| ClickHouse down | All queries fail | Read replicas, circuit breaker, cached results |
| Redis down | Cache miss, higher latency | Bypass cache, direct ClickHouse query |
| NATS down | No new data ingestion | Buffer in DWS, replay on recovery |
| Query Engine down | Dashboards/reports/NLQ unavailable | Auto-scale, health-based routing |
| NLQ Service down | NLQ unavailable | Graceful degradation, manual query mode |
