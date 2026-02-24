# ERP-Observability Software Architecture

## 1. Architecture Style

ERP-Observability employs a microservice architecture with three distinct API services -- a Go gateway for routing and authentication, a Node.js observability-api for telemetry query orchestration, and a Go tenant-api for tenant lifecycle management. The platform follows Domain-Driven Design principles with bounded contexts for metrics, logs, traces, alerts, infrastructure, and tenants.

```mermaid
graph TB
    subgraph "API Gateway Layer"
        GW["Go Gateway (:8090)<br/>Routing, Auth, Tenant Extraction"]
    end

    subgraph "Service Layer"
        subgraph "Observability API (Node.js :3000)"
            METRIC_SVC["Metric Service<br/>(PromQL Proxy)"]
            LOG_SVC["Log Service<br/>(Quickwit Query)"]
            TRACE_SVC["Trace Service<br/>(Quickwit Trace)"]
            ALERT_SVC["Alert Service<br/>(Alertmanager Proxy)"]
            DASH_SVC["Dashboard Service<br/>(Grafana Proxy)"]
            INFRA_SVC["Infrastructure Service<br/>(Zabbix + OpenNMS)"]
            SEARCH_SVC["Unified Search Service<br/>(Cross-Signal)"]
        end

        subgraph "Tenant API (Go :8080)"
            TENANT_SVC["Tenant CRUD"]
            PROVISION_SVC["Provisioning Service"]
            CONFIG_SVC["Configuration Service"]
            USAGE_SVC["Usage Metering"]
        end
    end

    subgraph "Backend Layer"
        VM["VictoriaMetrics Cluster"]
        QW["Quickwit Cluster"]
        AM["Alertmanager HA"]
        GRAF["Grafana"]
        ZBX["Zabbix Server"]
        ONMS["OpenNMS"]
        DF["DragonflyDB"]
        YB["YugabyteDB"]
    end

    GW --> METRIC_SVC & LOG_SVC & TRACE_SVC & ALERT_SVC & DASH_SVC & INFRA_SVC & SEARCH_SVC
    GW --> TENANT_SVC & PROVISION_SVC & CONFIG_SVC & USAGE_SVC

    METRIC_SVC --> VM & DF
    LOG_SVC --> QW & DF
    TRACE_SVC --> QW
    ALERT_SVC --> AM & VM
    DASH_SVC --> GRAF
    INFRA_SVC --> ZBX & ONMS
    SEARCH_SVC --> VM & QW

    TENANT_SVC --> YB
    PROVISION_SVC --> GRAF & VM & QW & ZBX
    CONFIG_SVC --> YB & DF
    USAGE_SVC --> VM & YB
```

## 2. Component Architecture

### 2.1 Go Gateway (`cmd/server/main.go`)

The Go gateway is the single entry point for all API requests, handling authentication, tenant extraction, rate limiting, and request routing.

```mermaid
graph TB
    subgraph "Go Gateway (:8090)"
        ROUTER["HTTP Router<br/>(net/http)"]

        subgraph "Middleware"
            AUTH_MW["JWT Auth Middleware<br/>(ERP-IAM validation)"]
            TENANT_MW["Tenant Middleware<br/>(X-Tenant-ID extraction)"]
            RATE_MW["Rate Limiter<br/>(per-tenant limits)"]
            CORS_MW["CORS Middleware"]
            LOG_MW["Request Logger"]
            TRACE_MW["OTel Trace Middleware"]
        end

        subgraph "Route Groups"
            HEALTH["Health Routes<br/>/health, /ready"]
            METRICS_R["Metric Routes<br/>/api/v1/metrics/*"]
            LOGS_R["Log Routes<br/>/api/v1/logs/*"]
            TRACES_R["Trace Routes<br/>/api/v1/traces/*"]
            ALERTS_R["Alert Routes<br/>/api/v1/alerts/*"]
            DASH_R["Dashboard Routes<br/>/api/v1/dashboards/*"]
            INFRA_R["Infrastructure Routes<br/>/api/v1/infrastructure/*"]
            EVENTS_R["Event Routes<br/>/api/v1/events/*"]
            TENANTS_R["Tenant Routes<br/>/api/v1/tenants/*"]
            SEARCH_R["Search Routes<br/>/api/v1/search/*"]
        end

        subgraph "Reverse Proxy"
            OBS_PROXY["observability-api proxy<br/>:3000"]
            TENANT_PROXY["tenant-api proxy<br/>:8080"]
        end
    end

    ROUTER --> AUTH_MW --> TENANT_MW --> RATE_MW
    RATE_MW --> HEALTH & METRICS_R & LOGS_R & TRACES_R & ALERTS_R & DASH_R & INFRA_R & EVENTS_R & TENANTS_R & SEARCH_R
    METRICS_R & LOGS_R & TRACES_R & ALERTS_R & DASH_R & INFRA_R & EVENTS_R & SEARCH_R --> OBS_PROXY
    TENANTS_R --> TENANT_PROXY
```

### 2.2 Node.js Observability API (`observability-api/`)

```mermaid
graph TB
    subgraph "Node.js Observability API (:3000)"
        EXPRESS["Express.js Server"]

        subgraph "Controllers"
            METRIC_CTRL["MetricController<br/>PromQL query proxy"]
            LOG_CTRL["LogController<br/>Quickwit search proxy"]
            TRACE_CTRL["TraceController<br/>Trace search/detail"]
            ALERT_CTRL["AlertController<br/>Rule + silence management"]
            DASH_CTRL["DashboardController<br/>Grafana API proxy"]
            INFRA_CTRL["InfraController<br/>Zabbix + OpenNMS proxy"]
            SEARCH_CTRL["SearchController<br/>Cross-signal unified search"]
        end

        subgraph "Services"
            VM_CLIENT["VictoriaMetrics Client<br/>(HTTP, PromQL)"]
            QW_CLIENT["Quickwit Client<br/>(HTTP, Search API)"]
            AM_CLIENT["Alertmanager Client<br/>(HTTP API)"]
            GRAF_CLIENT["Grafana Client<br/>(HTTP API)"]
            ZBX_CLIENT["Zabbix Client<br/>(JSON-RPC API)"]
            ONMS_CLIENT["OpenNMS Client<br/>(REST API)"]
            CACHE_CLIENT["DragonflyDB Client<br/>(Redis Protocol)"]
        end

        subgraph "Middleware"
            SCOPE_MW["X-Scope-OrgID Injector"]
            CACHE_MW["Response Cache<br/>(DragonflyDB)"]
            ERROR_MW["Error Handler"]
        end
    end

    EXPRESS --> METRIC_CTRL & LOG_CTRL & TRACE_CTRL & ALERT_CTRL & DASH_CTRL & INFRA_CTRL & SEARCH_CTRL
    METRIC_CTRL --> VM_CLIENT
    LOG_CTRL --> QW_CLIENT
    TRACE_CTRL --> QW_CLIENT
    ALERT_CTRL --> AM_CLIENT & VM_CLIENT
    DASH_CTRL --> GRAF_CLIENT
    INFRA_CTRL --> ZBX_CLIENT & ONMS_CLIENT
    SEARCH_CTRL --> VM_CLIENT & QW_CLIENT
    VM_CLIENT & QW_CLIENT --> CACHE_CLIENT
```

### 2.3 Go Tenant API (`tenant-api/`)

```mermaid
graph TB
    subgraph "Go Tenant API (:8080)"
        MUX["HTTP Router"]

        subgraph "Handlers"
            TENANT_H["Tenant CRUD<br/>Create/Read/Update/Delete"]
            PROVISION_H["Provisioning Handler<br/>Stack setup per tenant"]
            CONFIG_H["Configuration Handler<br/>Retention, alerts, dashboards"]
            USAGE_H["Usage Handler<br/>Metric counts, log volume"]
            RBAC_H["RBAC Handler<br/>Role assignment per tenant"]
        end

        subgraph "Services"
            TENANT_REPO["Tenant Repository<br/>(YugabyteDB)"]
            GRAF_PROV["Grafana Provisioner<br/>(Org + datasources)"]
            VM_PROV["VM Provisioner<br/>(Tenant namespaces)"]
            QW_PROV["Quickwit Provisioner<br/>(Index creation)"]
            ZBX_PROV["Zabbix Provisioner<br/>(Host groups)"]
        end
    end

    MUX --> TENANT_H & PROVISION_H & CONFIG_H & USAGE_H & RBAC_H
    TENANT_H --> TENANT_REPO
    PROVISION_H --> GRAF_PROV & VM_PROV & QW_PROV & ZBX_PROV
    CONFIG_H --> TENANT_REPO
    USAGE_H --> TENANT_REPO
```

### 2.4 Web Frontend (`web/`)

```mermaid
graph TB
    subgraph "React Frontend (Refine + AntD)"
        ENTRY["main.tsx<br/>(App Entry)"]
        APP["App.tsx<br/>(Refine Provider, Routes)"]
        AUTH["authProvider.ts<br/>(JWT Authentication)"]
        DATA["dataProvider.ts<br/>(REST Data Fetching)"]

        subgraph "Pages"
            P1["Dashboard Page"]
            P2["Metric Explorer Page"]
            P3["Log Search Page"]
            P4["Trace Explorer Page"]
            P5["Alert Management Page"]
            P6["Infrastructure Page"]
            P7["Tenant Admin Page"]
            P8["Settings Page"]
        end

        subgraph "Components"
            C1["KPI Cards"]
            C2["Time Series Chart"]
            C3["Log Table"]
            C4["Trace Waterfall"]
            C5["Alert Timeline"]
            C6["Service Map"]
            C7["PromQL Editor"]
            C8["Grafana Embed"]
        end

        subgraph "Theme"
            THEME["Green Theme<br/>#059669 primary<br/>Ant Design tokens"]
        end
    end

    ENTRY --> APP
    APP --> AUTH & DATA & THEME
    APP --> P1 & P2 & P3 & P4 & P5 & P6 & P7 & P8
    P1 --> C1 & C2 & C8
    P2 --> C2 & C7
    P3 --> C3
    P4 --> C4 & C6
    P5 --> C5
```

## 3. Data Model

### 3.1 Tenant Configuration (YugabyteDB)

```mermaid
erDiagram
    tenants ||--|{ tenant_configs : "has"
    tenants ||--|{ tenant_dashboards : "has"
    tenants ||--|{ tenant_alert_rules : "has"
    tenants ||--|{ tenant_notification_channels : "has"
    tenants ||--|{ tenant_slos : "has"
    tenants ||--o{ tenant_users : "has"
    tenant_alert_rules ||--o{ alert_history : "generates"
    tenant_notification_channels ||--|{ channel_templates : "uses"

    tenants {
        uuid id PK
        text tenant_id UK
        text name
        text status
        jsonb settings
        timestamp created_at
        timestamp updated_at
    }

    tenant_configs {
        uuid id PK
        uuid tenant_id FK
        text config_key
        jsonb config_value
        timestamp updated_at
    }

    tenant_dashboards {
        uuid id PK
        uuid tenant_id FK
        text grafana_org_id
        text dashboard_uid
        text title
        jsonb definition
        timestamp created_at
    }

    tenant_alert_rules {
        uuid id PK
        uuid tenant_id FK
        text name
        text expr
        text severity
        text duration
        jsonb labels
        jsonb annotations
        boolean enabled
        timestamp created_at
    }

    tenant_notification_channels {
        uuid id PK
        uuid tenant_id FK
        text name
        text type
        jsonb settings
        boolean is_default
        timestamp created_at
    }

    tenant_slos {
        uuid id PK
        uuid tenant_id FK
        text name
        text service
        float target
        text indicator_type
        text query
        text window
        timestamp created_at
    }
```

### 3.2 Quickwit Log Index Schema

```json
{
  "version": "0.7",
  "index_id": "logs-{tenant_id}",
  "doc_mapping": {
    "field_mappings": [
      {"name": "timestamp", "type": "datetime", "fast": true, "input_formats": ["rfc3339"]},
      {"name": "tenant_id", "type": "text", "tokenizer": "raw", "fast": true},
      {"name": "service_name", "type": "text", "tokenizer": "raw", "fast": true},
      {"name": "severity", "type": "text", "tokenizer": "raw", "fast": true},
      {"name": "body", "type": "text", "tokenizer": "default"},
      {"name": "trace_id", "type": "text", "tokenizer": "raw", "fast": true},
      {"name": "span_id", "type": "text", "tokenizer": "raw", "fast": true},
      {"name": "resource_attributes", "type": "json"},
      {"name": "log_attributes", "type": "json"}
    ],
    "timestamp_field": "timestamp"
  },
  "search_settings": {
    "default_search_fields": ["body", "service_name"]
  },
  "retention": {
    "period": "90 days",
    "schedule": "daily"
  }
}
```

### 3.3 VictoriaMetrics Metric Model

Metrics follow the Prometheus data model with multi-tenant isolation:

```
# Metric naming convention
{module}_{component}_{metric_name}_{unit}

# Examples
erp_crm_http_requests_total{method="GET", status="200", tenant_id="acme"}
erp_iam_auth_latency_seconds{quantile="0.99", tenant_id="acme"}
erp_accounting_invoice_processing_duration_seconds_bucket{le="0.5", tenant_id="acme"}

# Tenant isolation via vmauth
# Request: GET /select/{tenant_id}/prometheus/api/v1/query?query=...
# X-Scope-OrgID header propagated to all queries
```

## 4. Event Architecture

### 4.1 Observability Events

```mermaid
graph LR
    subgraph "Alert Events"
        AE1["alert.firing"]
        AE2["alert.resolved"]
        AE3["alert.silenced"]
        AE4["alert.escalated"]
    end

    subgraph "Tenant Events"
        TE1["tenant.created"]
        TE2["tenant.provisioned"]
        TE3["tenant.updated"]
        TE4["tenant.decommissioned"]
    end

    subgraph "Infrastructure Events"
        IE1["host.down"]
        IE2["host.recovered"]
        IE3["trigger.fired"]
        IE4["trigger.resolved"]
    end

    subgraph "Consumers"
        NOTIF["Notification Service"]
        AUDIT["Audit Logger"]
        ESCALATE["Escalation Engine"]
        DASHBOARD["Dashboard Refresh"]
    end

    AE1 & AE2 & AE3 & AE4 --> NOTIF & AUDIT & ESCALATE
    TE1 & TE2 & TE3 & TE4 --> AUDIT & DASHBOARD
    IE1 & IE2 & IE3 & IE4 --> NOTIF & AUDIT & ESCALATE
```

### 4.2 Event Format (CloudEvents)

```json
{
  "specversion": "1.0",
  "type": "com.opensase.observability.alert.firing",
  "source": "opensase-observability",
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "time": "2026-02-24T10:00:00Z",
  "data": {
    "alert_name": "HighErrorRate",
    "tenant_id": "acme-corp",
    "severity": "critical",
    "module": "erp-crm",
    "value": 0.15,
    "threshold": 0.05
  }
}
```

## 5. OTel Collector Pipeline Design

### 5.1 Pipeline Configuration

```mermaid
graph LR
    subgraph "Receivers"
        R1["otlp<br/>(gRPC :4317)"]
        R2["otlp<br/>(HTTP :4318)"]
        R3["prometheus<br/>(scrape config)"]
        R4["syslog<br/>(TCP :514)"]
    end

    subgraph "Processors"
        P1["batch<br/>(200ms, 8192)"]
        P2["memory_limiter<br/>(80% limit)"]
        P3["attributes<br/>(tenant_id injection)"]
        P4["filter<br/>(drop debug)"]
        P5["resource<br/>(standardize)"]
        P6["tail_sampling<br/>(error-biased)"]
    end

    subgraph "Exporters"
        E1["prometheusremotewrite<br/>(VictoriaMetrics)"]
        E2["otlp<br/>(Quickwit logs)"]
        E3["otlp<br/>(Quickwit traces)"]
    end

    R1 & R2 --> P2 --> P3 --> P4 --> P5 --> P1
    R3 --> P2 --> P1
    R4 --> P2 --> P3 --> P1
    P1 --> E1 & E2 & E3
    P5 --> P6 --> E3
```

## 6. Security Architecture

```mermaid
flowchart TB
    CLIENT["Client Request"] --> GW3["API Gateway (:8090)"]
    GW3 --> JWT["JWT Validation<br/>(ERP-IAM)"]
    JWT --> TENANT3["Tenant Extraction<br/>(X-Tenant-ID)"]
    TENANT3 --> RBAC["RBAC Check<br/>(Viewer/Editor/Admin)"]
    RBAC --> SCOPE["Inject X-Scope-OrgID<br/>(= tenant_id)"]
    SCOPE --> BACKEND["Backend Service<br/>(VM/Quickwit/Grafana)"]
    BACKEND --> ISOLATE["Tenant-Scoped Response<br/>(only tenant data returned)"]
    SCOPE --> AUDIT2["Audit Event<br/>(Quickwit Immutable Index)"]
```

## 7. Concurrency Model

The Go gateway and tenant-api use Go's goroutine-based concurrency:

- **Gateway**: One goroutine per HTTP request, `http.ReverseProxy` for backend forwarding
- **Tenant API**: Connection pool to YugabyteDB via `pgx`, DragonflyDB connection pool via `go-redis`
- **Node.js API**: Event loop with async/await for non-blocking I/O to all backends
- **DragonflyDB**: Multi-threaded shared-nothing architecture for cache operations
- **VictoriaMetrics**: Concurrent query execution with per-tenant query limits
- **Quickwit**: Search parallelism across index splits
