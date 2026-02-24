# High-Level Design (HLD) -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. System Overview

ERP-Church-Management is a multi-tenant, microservices-based Church Management System supporting 12 domain services, a Go API gateway, React/Next.js web frontend, and Flutter mobile applications. The system manages the complete lifecycle of church members from first-time visitor to fully assimilated, serving member through an event-driven architecture.

---

## 2. High-Level System Diagram

```mermaid
flowchart TB
    subgraph Presentation_Tier["Presentation Tier"]
        WEB["Web Application\n(React/Next.js)"]
        MOBILE["Mobile App\n(Flutter)"]
        KIOSK["Check-in Kiosk\n(React Kiosk Mode)"]
    end

    subgraph Application_Tier["Application Tier"]
        LB["Load Balancer / Ingress"]
        GW["API Gateway (Go)\nJWT + Tenant + Entitlements"]

        subgraph Service_Mesh["12 Domain Services"]
            MS["Member"]
            VS["Visitor"]
            FS["Followup"]
            GS["Giving"]
            ES["Event"]
            GRS["Group"]
            DS["Discipleship"]
            WS["Welfare"]
            CS["Communication"]
            KS["KPI"]
            VOS["Volunteer"]
            FAS["Facility"]
        end
    end

    subgraph Data_Tier["Data Tier"]
        PG[("PostgreSQL 16\nPrimary + Read Replica")]
        RD[("Redis 7\nCache Cluster")]
        RP["Redpanda\nEvent Stream"]
        S3["Object Storage\n(Profile photos, documents)"]
    end

    subgraph External_Tier["External Systems"]
        IAM["ERP-IAM"]
        PLT["ERP-Platform"]
        FIN["ERP-Finance"]
        BII["ERP-BI"]
        TWI["Twilio"]
        WHA["WhatsApp API"]
        TG["Telegram API"]
    end

    WEB & MOBILE & KIOSK --> LB
    LB --> GW
    GW --> Service_Mesh
    Service_Mesh --> PG & RD & RP
    CS --> TWI & WHA & TG
    GW --> IAM & PLT
    GS --> FIN
    KS --> BII
    Service_Mesh --> S3
```

---

## 3. Component Design

### 3.1 API Gateway

| Aspect | Design Decision |
|---|---|
| Language | Go (stdlib net/http) |
| Routing | Path-prefix based: `/v1/{service}/...` |
| Auth | JWT Bearer token validation |
| Multi-tenancy | `X-Tenant-ID` header enforcement |
| Entitlements | REST call to ERP-Platform with graceful degradation |
| Proxying | `httputil.NewSingleHostReverseProxy` |
| Correlation | Auto-generated `X-Correlation-ID` |
| Port | 8090 (mapped to 8093 externally) |

### 3.2 Service Design Principles

Each of the 12 microservices follows these principles:

1. **Single Responsibility**: Each service owns one bounded context
2. **Database per Service (future)**: Phase 1 uses shared DB; Phase 2 splits
3. **Event-Driven Communication**: Async communication via Kafka topics
4. **Stateless**: No in-process state; all state in PostgreSQL/Redis
5. **Health Check**: Every service exposes `GET /healthz`
6. **Idempotent Writes**: All mutations are idempotent with deduplication keys

### 3.3 Core Services Deep Design

#### member-service

```mermaid
flowchart TD
    subgraph member_service["member-service"]
        API["REST API"]
        SVC["Member Service"]
        REPO["Member Repository"]
        PUB["Event Publisher"]

        API --> SVC
        SVC --> REPO
        SVC --> PUB

        subgraph Endpoints
            E1["POST   /members"]
            E2["GET    /members"]
            E3["GET    /members/:id"]
            E4["PUT    /members/:id"]
            E5["DELETE /members/:id"]
            E6["GET    /members/search"]
            E7["GET    /members/statistics"]
            E8["GET    /members/absentees"]
            E9["GET    /members/:id/attendance"]
            E10["GET   /members/:id/donations"]
            E11["GET   /members/:id/groups"]
            E12["PUT   /members/:id/natural-group"]
            E13["POST  /members/:id/account-officer"]
            E14["PUT   /members/:id/status"]
        end
    end
```

#### visitor-service

```mermaid
flowchart TD
    subgraph visitor_service["visitor-service"]
        API["REST API"]
        SVC["Visitor Service"]
        REPO["Visitor Repository"]
        PUB["Event Publisher"]
        CONV["Conversion Engine"]

        API --> SVC
        SVC --> REPO
        SVC --> PUB
        SVC --> CONV

        subgraph Endpoints
            E1["POST   /visitors"]
            E2["GET    /visitors"]
            E3["GET    /visitors/:id"]
            E4["PUT    /visitors/:id"]
            E5["DELETE /visitors/:id"]
            E6["GET    /visitors/first-timers"]
            E7["GET    /visitors/pending-followup"]
            E8["GET    /visitors/72-hour-status"]
            E9["POST   /visitors/:id/follow-up"]
            E10["POST  /visitors/:id/convert"]
            E11["POST  /visitors/:id/welcome-gift"]
            E12["POST  /visitors/:id/72-hour-contact"]
            E13["GET   /visitors/:id/72-hour-report"]
        end
    end
```

#### followup-service

```mermaid
flowchart TD
    subgraph followup_service["followup-service"]
        API["REST API"]
        CON["Kafka Consumer"]
        SVC["Followup Service"]
        ROUTER["Directorate Router"]
        ASSIGN["Assignment Engine"]

        API --> SVC
        CON --> SVC
        SVC --> ROUTER
        SVC --> ASSIGN

        subgraph Directorates
            D1["1st Timer Directorate"]
            D2["Further Follow-up"]
            D3["Counselling"]
            D4["Natural Group"]
            D5["Development & Structuring"]
            D6["Welfare & Finance"]
        end

        ROUTER --> D1 & D2 & D3 & D4 & D5 & D6
    end
```

---

## 4. Data Flow Design

### 4.1 Read Path (Query)

```mermaid
sequenceDiagram
    participant C as Client
    participant GW as Gateway
    participant SVC as Service
    participant RD as Redis
    participant PG as PostgreSQL

    C->>GW: GET /v1/member/123
    GW->>GW: Validate JWT + Tenant
    GW->>SVC: Forward request
    SVC->>RD: Check cache
    alt Cache hit
        RD-->>SVC: Cached response
    else Cache miss
        SVC->>PG: SELECT * FROM members WHERE id = 123 AND tenant_id = ?
        PG-->>SVC: Row data
        SVC->>RD: SET cache key (TTL: 5 min)
    end
    SVC-->>GW: JSON response
    GW-->>C: JSON response
```

### 4.2 Write Path (Command)

```mermaid
sequenceDiagram
    participant C as Client
    participant GW as Gateway
    participant SVC as Service
    participant PG as PostgreSQL
    participant RD as Redis
    participant RP as Redpanda

    C->>GW: POST /v1/visitor (body)
    GW->>GW: Validate JWT + Tenant
    GW->>SVC: Forward request
    SVC->>PG: INSERT INTO visitors (...) VALUES (...)
    PG-->>SVC: Created row
    SVC->>RD: Invalidate related cache keys
    SVC->>RP: Publish visitor.created event
    SVC-->>GW: 201 Created + JSON
    GW-->>C: 201 Created + JSON

    Note over RP: Async consumers
    RP->>FU: followup-service consumes
    RP->>COM: communication-service consumes
    RP->>KPI: kpi-service consumes
```

---

## 5. Scalability Design

### 5.1 Horizontal Scaling

| Component | Scaling Strategy | Min Replicas | Max Replicas | Trigger |
|---|---|---|---|---|
| Gateway | HPA | 2 | 10 | CPU > 70% |
| member-service | HPA | 2 | 8 | CPU > 70% or RPS > 500 |
| visitor-service | HPA | 1 | 6 | CPU > 70% |
| event-service | HPA | 2 | 10 | CPU > 70% (Sunday spike) |
| communication-service | HPA | 2 | 12 | Queue depth > 1000 |
| kpi-service | HPA | 1 | 4 | CPU > 80% |
| Other services | HPA | 1 | 4 | CPU > 70% |

### 5.2 Database Scaling

```mermaid
flowchart LR
    subgraph Write_Path
        SVC["Services (writes)"] --> PGB["PgBouncer\n(Connection Pool)"]
        PGB --> PG_P["PostgreSQL Primary"]
    end
    subgraph Read_Path
        SVC2["Services (reads)"] --> PGB2["PgBouncer\n(Connection Pool)"]
        PGB2 --> PG_R1["Read Replica 1"]
        PGB2 --> PG_R2["Read Replica 2"]
    end
    PG_P -->|Streaming Replication| PG_R1 & PG_R2
```

---

## 6. Security Design

### 6.1 Authentication Flow

```mermaid
flowchart TD
    USER["User"] --> LOGIN["Login via ERP-IAM"]
    LOGIN --> TOKEN["JWT Access Token\n(Claims: sub, role, tenant_id, exp)"]
    TOKEN --> GW["Gateway receives request"]
    GW --> VALIDATE["Validate JWT signature\n(JWKS endpoint)"]
    VALIDATE --> EXTRACT["Extract claims"]
    EXTRACT --> CHECK_ROLE["Check role authorization"]
    CHECK_ROLE --> INJECT["Inject X-Tenant-ID\nfrom token claims"]
    INJECT --> SVC["Forward to service"]
    SVC --> QUERY["Query with WHERE tenant_id = ?"]
```

### 6.2 Data Encryption

| Data State | Encryption Method |
|---|---|
| In Transit | TLS 1.3 (all connections) |
| At Rest (DB) | PostgreSQL TDE / Volume encryption |
| At Rest (Cache) | Redis TLS + encrypted volumes |
| At Rest (Events) | Redpanda TLS + encrypted storage |
| Sensitive Fields | AES-256 application-level encryption (PII) |
| Passwords | bcrypt (cost factor 12) |

---

## 7. Deployment Design

### 7.1 Container Topology

```mermaid
flowchart TB
    subgraph K8s_Cluster["Kubernetes Cluster"]
        subgraph NS_Church["Namespace: erp-church-management"]
            GW_D["gateway\n(Deployment: 2 replicas)"]
            MS_D["member-service\n(Deployment: 2 replicas)"]
            VS_D["visitor-service\n(Deployment: 1 replica)"]
            FS_D["followup-service\n(Deployment: 1 replica)"]
            GS_D["giving-service\n(Deployment: 1 replica)"]
            ES_D["event-service\n(Deployment: 2 replicas)"]
            GRS_D["group-service\n(Deployment: 1 replica)"]
            DS_D["discipleship-service\n(Deployment: 1 replica)"]
            WS_D["welfare-service\n(Deployment: 1 replica)"]
            CS_D["communication-service\n(Deployment: 2 replicas)"]
            KS_D["kpi-service\n(Deployment: 1 replica)"]
            VOS_D["volunteer-service\n(Deployment: 1 replica)"]
            FAS_D["facility-service\n(Deployment: 1 replica)"]
        end

        subgraph NS_Data["Namespace: erp-data"]
            PG_ST["PostgreSQL\n(StatefulSet)"]
            RD_ST["Redis\n(StatefulSet)"]
            RP_ST["Redpanda\n(StatefulSet)"]
        end
    end

    ING["Ingress Controller\n(nginx/traefik)"] --> GW_D
```

### 7.2 Resource Requirements (Per Pod)

| Service | CPU Request | CPU Limit | Memory Request | Memory Limit |
|---|---|---|---|---|
| gateway | 100m | 500m | 64Mi | 256Mi |
| member-service | 100m | 500m | 128Mi | 512Mi |
| visitor-service | 100m | 500m | 128Mi | 512Mi |
| followup-service | 100m | 500m | 128Mi | 512Mi |
| giving-service | 100m | 500m | 128Mi | 512Mi |
| event-service | 200m | 1000m | 256Mi | 1Gi |
| group-service | 100m | 500m | 128Mi | 512Mi |
| discipleship-service | 100m | 500m | 128Mi | 512Mi |
| welfare-service | 100m | 500m | 128Mi | 512Mi |
| communication-service | 200m | 1000m | 256Mi | 1Gi |
| kpi-service | 200m | 1000m | 256Mi | 1Gi |
| volunteer-service | 100m | 500m | 128Mi | 512Mi |
| facility-service | 100m | 500m | 128Mi | 512Mi |

---

## 8. Monitoring Design

```mermaid
flowchart LR
    subgraph Services
        S["All 12 Services\n+ Gateway"]
    end

    subgraph Metrics
        PROM["Prometheus\n(Scrape /metrics)"]
    end

    subgraph Logging
        LOKI["Loki\n(Log aggregation)"]
    end

    subgraph Tracing
        OTEL["OpenTelemetry\nCollector"]
        JAEGER["Jaeger\n(Trace storage)"]
    end

    subgraph Visualization
        GRAF["Grafana\n(Dashboards + Alerts)"]
    end

    S -->|metrics| PROM
    S -->|logs| LOKI
    S -->|traces| OTEL --> JAEGER
    PROM --> GRAF
    LOKI --> GRAF
    JAEGER --> GRAF
```

### 8.1 Key Alerts

| Alert | Condition | Severity |
|---|---|---|
| Service Down | healthz returns non-200 for > 30s | Critical |
| High Latency | p95 > 500ms for > 5 min | Warning |
| Error Rate | 5xx rate > 1% for > 2 min | Critical |
| DB Connection Pool Exhausted | Available connections < 5 | Critical |
| Kafka Consumer Lag | Lag > 10,000 messages | Warning |
| 72-Hour SLA Breach | Uncontacted visitors > 72 hours | Business Critical |
| Disk Usage > 80% | PostgreSQL/Redpanda disk | Warning |
