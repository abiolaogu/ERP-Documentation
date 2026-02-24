# ERP-Observability High-Level Design

## 1. System Context

```mermaid
C4Context
    title ERP-Observability System Context Diagram

    Person(sre, "SRE", "Monitors SLOs, responds to incidents")
    Person(devops, "DevOps Engineer", "Monitors infrastructure, deployments")
    Person(admin, "Platform Admin", "Manages tenants, configuration")
    Person(developer, "Module Developer", "Debugs applications, analyzes logs")

    System(obs, "ERP-Observability", "Unified Observability Platform")

    System_Ext(iam, "ERP-IAM", "Identity & Access Management")
    System_Ext(platform, "ERP-Platform", "Control Plane & Licensing")
    System_Ext(modules, "ERP Modules (20+)", "Telemetry Sources")
    System_Ext(infra, "Infrastructure", "Hosts, Networks, Devices")

    Rel(sre, obs, "Monitors, investigates", "HTTPS")
    Rel(devops, obs, "Monitors, configures", "HTTPS")
    Rel(admin, obs, "Manages tenants", "HTTPS")
    Rel(developer, obs, "Searches, debugs", "HTTPS")

    Rel(obs, iam, "Authenticates", "OIDC/JWT")
    Rel(obs, platform, "License check", "REST")
    Rel(modules, obs, "Emit telemetry", "OTLP gRPC/HTTP")
    Rel(infra, obs, "Infrastructure metrics", "Zabbix/SNMP")
```

## 2. Container Architecture

```mermaid
graph TB
    subgraph "Client Tier"
        WEB3["React Web App<br/>Refine + AntD (#059669)"]
        GRAFANA3["Grafana Web UI<br/>(Embedded + Standalone)"]
    end

    subgraph "API Tier"
        GW5["Go Gateway<br/>:8090<br/>(Auth, Routing, Rate Limit)"]
        OBS_API2["Node.js observability-api<br/>:3000<br/>(Query Orchestration)"]
        TENANT_API2["Go tenant-api<br/>:8080<br/>(Tenant Lifecycle)"]
    end

    subgraph "Telemetry Tier"
        OTEL4["OTel Collector<br/>(DaemonSet)<br/>(Receive, Process, Export)"]
        VMALERT2["vmalert<br/>(Alert Rule Evaluation)"]
        AM4["Alertmanager<br/>(3-node HA)<br/>(Route, Group, Notify)"]
    end

    subgraph "Metrics Tier"
        VMINSERT2["vminsert<br/>(Write Ingestion)"]
        VMSELECT2["vmselect<br/>(PromQL Query)"]
        VMSTORAGE2["vmstorage<br/>(Time Series Data)"]
        VMAUTH2["vmauth<br/>(Tenant Routing)"]
    end

    subgraph "Search Tier"
        QW_INDEXER2["Quickwit Indexer<br/>(Log + Trace Indexing)"]
        QW_SEARCHER2["Quickwit Searcher<br/>(Distributed Search)"]
    end

    subgraph "Infrastructure Tier"
        ZBX_SERVER["Zabbix Server<br/>(Host Monitoring)"]
        ONMS_SERVER["OpenNMS<br/>(Event Correlation)"]
    end

    subgraph "Data Tier"
        DF3["DragonflyDB<br/>(Query Cache)"]
        YB3["YugabyteDB<br/>(Config + State)"]
        RFS3["RustFS<br/>(S3 Object Storage)"]
    end

    WEB3 & GRAFANA3 --> GW5
    GW5 --> OBS_API2 & TENANT_API2
    OBS_API2 --> VMAUTH2 & QW_SEARCHER2 & AM4 & GRAFANA3 & ZBX_SERVER & ONMS_SERVER & DF3
    TENANT_API2 --> YB3 & DF3 & GRAFANA3 & VMAUTH2 & QW_INDEXER2 & ZBX_SERVER

    OTEL4 --> VMINSERT2 & QW_INDEXER2
    VMINSERT2 --> VMSTORAGE2
    VMSELECT2 --> VMSTORAGE2
    VMAUTH2 --> VMINSERT2 & VMSELECT2
    VMALERT2 --> VMSELECT2 & AM4

    QW_INDEXER2 --> RFS3
    QW_SEARCHER2 --> RFS3
```

## 3. Component Design

### 3.1 Go Gateway Components

```mermaid
graph TB
    subgraph "Go Gateway (:8090)"
        subgraph "Middleware Stack"
            TRACE_MW2["OTel Trace Middleware"]
            LOG_MW2["Request Logger"]
            CORS2["CORS Middleware"]
            AUTH2["JWT Validation<br/>(ERP-IAM JWKS)"]
            TENANT3["Tenant Extraction<br/>(X-Tenant-ID -> X-Scope-OrgID)"]
            RATE2["Rate Limiter<br/>(per-tenant, DragonflyDB-backed)"]
        end

        subgraph "Route Groups"
            HEALTH4["/health, /ready"]
            METRICS4["/api/v1/metrics/*"]
            LOGS4["/api/v1/logs/*"]
            TRACES4["/api/v1/traces/*"]
            ALERTS4["/api/v1/alerts/*"]
            DASHBOARDS4["/api/v1/dashboards/*"]
            INFRA4["/api/v1/infrastructure/*"]
            EVENTS4["/api/v1/events/*"]
            TENANTS4["/api/v1/tenants/*"]
            SEARCH4["/api/v1/search/*"]
        end

        subgraph "Proxies"
            OBS_PROXY2["Reverse Proxy -> :3000"]
            TENANT_PROXY2["Reverse Proxy -> :8080"]
        end
    end

    TRACE_MW2 --> LOG_MW2 --> CORS2 --> AUTH2 --> TENANT3 --> RATE2
    RATE2 --> HEALTH4 & METRICS4 & LOGS4 & TRACES4 & ALERTS4 & DASHBOARDS4 & INFRA4 & EVENTS4 & TENANTS4 & SEARCH4
    METRICS4 & LOGS4 & TRACES4 & ALERTS4 & DASHBOARDS4 & INFRA4 & EVENTS4 & SEARCH4 --> OBS_PROXY2
    TENANTS4 --> TENANT_PROXY2
```

### 3.2 VictoriaMetrics Cluster Components

```mermaid
graph TB
    subgraph "VictoriaMetrics Cluster"
        subgraph "Write Path"
            VMAUTH_W["vmauth (write)<br/>Tenant -> accountID mapping"]
            INSERT1["vminsert-1"]
            INSERT2["vminsert-2"]
        end

        subgraph "Storage"
            STORE1["vmstorage-1<br/>(Shard 1)"]
            STORE2["vmstorage-2<br/>(Shard 2)"]
            STORE3["vmstorage-3<br/>(Shard 3)"]
        end

        subgraph "Read Path"
            VMAUTH_R["vmauth (read)<br/>Tenant -> accountID mapping"]
            SELECT1["vmselect-1"]
            SELECT2["vmselect-2"]
        end

        subgraph "Alert Engine"
            ALERT_ENGINE["vmalert<br/>Rule evaluation<br/>Recording rules"]
        end
    end

    VMAUTH_W --> INSERT1 & INSERT2
    INSERT1 & INSERT2 --> STORE1 & STORE2 & STORE3
    SELECT1 & SELECT2 --> STORE1 & STORE2 & STORE3
    VMAUTH_R --> SELECT1 & SELECT2
    ALERT_ENGINE --> VMAUTH_R
```

## 4. Data Flow Design

### 4.1 Metric Read Path

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as Go Gateway (:8090)
    participant OBS as Observability API (:3000)
    participant Cache as DragonflyDB
    participant VM as VictoriaMetrics (vmselect)

    Client->>Gateway: GET /api/v1/metrics/query?query=...
    Gateway->>Gateway: Validate JWT, Extract tenant_id
    Gateway->>OBS: Forward + X-Scope-OrgID header
    OBS->>Cache: Check query cache (key: hash(tenant+query+time))

    alt Cache Hit
        Cache-->>OBS: Cached result
        OBS-->>Gateway: PromQL result
    else Cache Miss
        OBS->>VM: PromQL query via vmauth (tenant-scoped)
        VM-->>OBS: Time series result
        OBS->>Cache: Cache result (TTL: 15s)
        OBS-->>Gateway: PromQL result
    end

    Gateway-->>Client: JSON Response
```

### 4.2 Log Write Path

```mermaid
sequenceDiagram
    participant App as ERP Module
    participant Collector as OTel Collector
    participant QW as Quickwit (Indexer)
    participant RFS as RustFS (S3)

    App->>Collector: OTLP Log Export (gRPC)
    Collector->>Collector: Memory limit check
    Collector->>Collector: Inject tenant_id from context
    Collector->>Collector: Filter (drop DEBUG)
    Collector->>Collector: Batch (200ms timeout)
    Collector->>QW: OTLP Log Export (HTTP)
    QW->>QW: Parse log records
    QW->>QW: Tokenize and index
    QW->>QW: Create index split (time-partitioned)
    QW->>RFS: Upload split to S3 storage
    Note over QW,RFS: Splits are immutable once written
```

### 4.3 Trace Write and Read Path

```mermaid
sequenceDiagram
    participant App as ERP Module
    participant Collector as OTel Collector
    participant QW as Quickwit
    participant API as Observability API
    participant Client

    Note over App,Client: Write Path
    App->>Collector: OTLP Trace Export (gRPC)
    Collector->>Collector: Tail sampling (error-biased)
    Collector->>Collector: Batch + enrich
    Collector->>QW: OTLP Trace Export
    QW->>QW: Index trace spans

    Note over App,Client: Read Path
    Client->>API: GET /api/v1/traces/:traceId
    API->>QW: Search by trace_id
    QW-->>API: All spans for trace
    API->>API: Reconstruct trace tree
    API-->>Client: Trace waterfall data
```

## 5. Deployment Architecture

```mermaid
graph TB
    subgraph "Production Kubernetes Cluster (Harvester HCI)"
        subgraph "Namespace: observability-api"
            GW_DEPLOY["gateway<br/>Deployment (2 replicas)"]
            API_DEPLOY["observability-api<br/>Deployment (2 replicas)"]
            TAPI_DEPLOY["tenant-api<br/>Deployment (2 replicas)"]
            WEB_DEPLOY2["web-frontend<br/>Deployment (2 replicas)"]
        end

        subgraph "Namespace: observability-metrics"
            VM_INSERT_SS["vminsert<br/>Deployment (2 replicas)"]
            VM_SELECT_SS["vmselect<br/>Deployment (2 replicas)"]
            VM_STORAGE_SS["vmstorage<br/>StatefulSet (3 nodes)"]
            VM_AUTH_DEPLOY["vmauth<br/>Deployment (2 replicas)"]
            VM_ALERT_DEPLOY["vmalert<br/>Deployment (2 replicas)"]
        end

        subgraph "Namespace: observability-search"
            QW_INDEXER_DEPLOY["quickwit-indexer<br/>Deployment (2 replicas)"]
            QW_SEARCHER_DEPLOY["quickwit-searcher<br/>Deployment (2 replicas)"]
        end

        subgraph "Namespace: observability-infra"
            ZBX_DEPLOY["zabbix-server<br/>Deployment (1 replica)"]
            ONMS_DEPLOY["opennms<br/>Deployment (1 replica)"]
        end

        subgraph "Namespace: observability-telemetry"
            OTEL_DS2["otel-collector<br/>DaemonSet (1 per node)"]
            AM_SS["alertmanager<br/>StatefulSet (3 nodes)"]
        end

        subgraph "Namespace: observability-data"
            DF_SS["dragonflydb<br/>StatefulSet (1 primary + 1 replica)"]
            YB_SS["yugabytedb<br/>StatefulSet (3 nodes)"]
            RFS_SS["rustfs<br/>StatefulSet (4 nodes)"]
        end

        subgraph "Namespace: observability-viz"
            GRAF_DEPLOY2["grafana<br/>Deployment (2 replicas)"]
        end

        subgraph "Ingress"
            INGRESS3["Ingress Controller<br/>TLS Termination"]
        end
    end

    INGRESS3 --> GW_DEPLOY
    GW_DEPLOY --> API_DEPLOY & TAPI_DEPLOY
    OTEL_DS2 --> VM_INSERT_SS & QW_INDEXER_DEPLOY
```

## 6. External Interfaces

### 6.1 Inbound Interfaces

| Interface | Protocol | Port | Purpose |
|-----------|----------|------|---------|
| OTLP gRPC | gRPC | 4317 | Telemetry ingestion from OTel SDKs |
| OTLP HTTP | HTTP | 4318 | Telemetry ingestion from OTel SDKs |
| Prometheus Scrape | HTTP | varies | Legacy metric scraping |
| Syslog | TCP | 514 | Infrastructure log ingestion |
| Zabbix Agent | Zabbix Protocol | 10050/10051 | Host metric collection |
| SNMP | UDP | 161/162 | Network device monitoring |
| API Gateway | HTTPS | 443 (8090 internal) | User and API access |

### 6.2 Outbound Interfaces

| Interface | Protocol | Purpose |
|-----------|----------|---------|
| SMTP | TCP/587 | Email alert notifications |
| Slack Webhook | HTTPS | Slack alert notifications |
| PagerDuty API | HTTPS | PagerDuty incident creation |
| OpsGenie API | HTTPS | OpsGenie alert creation |
| Custom Webhooks | HTTPS | Custom notification integrations |
| Pulsar | Native | Observability event streaming |
| NATS JetStream | Native | Real-time alert notifications |

### 6.3 Query Interfaces

| Interface | Query Language | Backend |
|-----------|---------------|---------|
| PromQL | `rate(metric[5m])` | VictoriaMetrics |
| Quickwit Query | `service_name:erp-crm AND severity:ERROR` | Quickwit |
| Grafana Explore | PromQL + Quickwit | Grafana |
| Zabbix API | JSON-RPC | Zabbix Server |
| OpenNMS REST | REST API | OpenNMS |

## 7. Scalability Design

### 7.1 Horizontal Scaling Strategy

| Component | Scaling Method | Scaling Trigger | Max Scale |
|-----------|---------------|----------------|-----------|
| Go Gateway | HPA (CPU 70%) | Request rate | 10 replicas |
| Observability API | HPA (CPU 70%) | Request rate | 10 replicas |
| Tenant API | HPA (CPU 60%) | Request rate | 5 replicas |
| vminsert | HPA (CPU 70%) | Ingestion rate | 10 replicas |
| vmselect | HPA (CPU 70%) | Query rate | 10 replicas |
| vmstorage | Manual scaling | Storage capacity | 20 nodes |
| Quickwit Indexer | HPA (CPU 70%) | Ingestion rate | 10 replicas |
| Quickwit Searcher | HPA (CPU 70%) | Search rate | 10 replicas |
| OTel Collector | DaemonSet | Node count | 1 per node |
| Alertmanager | StatefulSet | N/A (3-node HA) | 5 nodes |
| Grafana | HPA (Memory 80%) | Connection count | 5 replicas |
| DragonflyDB | Vertical + replication | Cache hit ratio | 1 primary + 3 replicas |
| YugabyteDB | Horizontal | Storage + query load | 9 nodes |
| RustFS | Horizontal | Storage capacity | 16 nodes |

### 7.2 Caching Strategy

```mermaid
graph LR
    QUERY["PromQL / Search Query"] --> CACHE_CHECK2{In DragonflyDB<br/>Cache?}
    CACHE_CHECK2 -->|Yes, TTL valid| RETURN_CACHED2["Return Cached<br/>Response (<1ms)"]
    CACHE_CHECK2 -->|No| BACKEND["Query Backend<br/>(VM / Quickwit)"]
    BACKEND --> CACHE_STORE2["Store in<br/>DragonflyDB (TTL: 15s)"]
    CACHE_STORE2 --> RETURN_FRESH2["Return Fresh<br/>Response"]

    WRITE2["Write / Config Change"] --> INVALIDATE2["Invalidate<br/>Related Cache Keys"]
```

## 8. Reliability Design

### 8.1 Failure Handling

```mermaid
graph TB
    subgraph "Failure Modes"
        F1["VictoriaMetrics<br/>Storage Down"]
        F2["Quickwit<br/>Indexer Down"]
        F3["OTel Collector<br/>Down"]
        F4["Alertmanager<br/>Down"]
        F5["DragonflyDB<br/>Down"]
        F6["Grafana<br/>Down"]
    end

    subgraph "Handling"
        H1["vminsert buffers writes<br/>vmstorage auto-rebalances"]
        H2["OTel Collector queues logs<br/>Replay on recovery"]
        H3["K8s DaemonSet auto-restart<br/>Applications retry OTLP export"]
        H4["HA cluster (3 nodes)<br/>Gossip protocol failover"]
        H5["Bypass cache, direct backend<br/>Graceful degradation"]
        H6["React frontend remains functional<br/>Embedded panels fallback to API"]
    end

    F1 --> H1
    F2 --> H2
    F3 --> H3
    F4 --> H4
    F5 --> H5
    F6 --> H6
```

### 8.2 Data Durability

| Data Type | Durability Mechanism | RPO |
|-----------|---------------------|-----|
| Metrics (VictoriaMetrics) | Replication factor 2, Mayastor PV | < 1 minute |
| Logs (Quickwit) | Immutable splits on RustFS (erasure coded) | < 5 minutes |
| Traces (Quickwit) | Immutable splits on RustFS | < 5 minutes |
| Configuration (YugabyteDB) | Raft consensus (3 replicas) | 0 (synchronous) |
| Cache (DragonflyDB) | Ephemeral (rebuild from backends) | N/A |
| Dashboards (Grafana) | YugabyteDB backend + Git versioning | 0 |
