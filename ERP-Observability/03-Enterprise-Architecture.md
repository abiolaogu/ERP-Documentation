# ERP-Observability Enterprise Architecture

## 1. Business Architecture

### 1.1 Business Capability Model

```mermaid
graph TB
    subgraph "ERP-Observability Business Capabilities"
        subgraph "Metric Intelligence"
            MI1["Metric Collection & Ingestion"]
            MI2["PromQL Query Engine"]
            MI3["Time Series Analytics"]
            MI4["SLO/SLI Management"]
            MI5["Capacity Planning"]
            MI6["Cost Attribution"]
        end
        subgraph "Log Management"
            LM1["Centralized Log Aggregation"]
            LM2["Structured Log Search"]
            LM3["Log Pattern Analysis"]
            LM4["Real-Time Log Tailing"]
            LM5["Log-Based Alerting"]
            LM6["Compliance Log Retention"]
        end
        subgraph "Distributed Tracing"
            DT1["Trace Collection"]
            DT2["Trace Search & Filter"]
            DT3["Trace Waterfall Visualization"]
            DT4["Service Map Generation"]
            DT5["Latency Analysis"]
            DT6["Error Propagation Tracking"]
        end
        subgraph "Alert & Incident Management"
            AI1["Alert Rule Evaluation"]
            AI2["Alert Routing & Grouping"]
            AI3["Multi-Channel Notification"]
            AI4["Escalation Policies"]
            AI5["Silence & Maintenance Windows"]
            AI6["Incident Correlation"]
        end
        subgraph "Infrastructure Monitoring"
            IM1["Host Health Monitoring"]
            IM2["Network Device Monitoring"]
            IM3["Process & Service Monitoring"]
            IM4["Auto-Discovery"]
            IM5["Event Correlation"]
            IM6["Topology Visualization"]
        end
        subgraph "Platform Administration"
            PA1["Tenant Provisioning"]
            PA2["Dashboard Management"]
            PA3["Retention Policy Management"]
            PA4["Access Control"]
            PA5["Usage Metering"]
            PA6["Self-Monitoring"]
        end
    end
```

### 1.2 Value Stream Mapping

```mermaid
graph LR
    E["Emit Telemetry<br/>(OTel SDK in Modules)"] --> C["Collect & Process<br/>(OTel Collector)"]
    C --> S["Store & Index<br/>(VM + Quickwit)"]
    S --> V["Visualize & Explore<br/>(Grafana + Frontend)"]
    V --> A["Alert & Notify<br/>(Alertmanager)"]
    A --> R["Respond & Resolve<br/>(SRE / DevOps)"]
    R --> I["Improve & Optimize<br/>(SLO Reviews)"]
    I --> E
```

### 1.3 Stakeholder Map

| Stakeholder | Role | Primary Concerns |
|------------|------|-----------------|
| Site Reliability Engineers (SREs) | Primary User | SLO compliance, incident response, error budgets, capacity planning |
| DevOps Engineers | Primary User | Pipeline health, deployment monitoring, infrastructure automation |
| Platform Engineers | Primary User | Multi-tenant management, self-service observability, cost optimization |
| Module Developers | End User | Application debugging, log analysis, trace exploration |
| Security Engineers | End User | Audit log review, anomaly detection, compliance reporting |
| IT Operations | Technical | Infrastructure health, network monitoring, Zabbix/OpenNMS management |
| Executive Leadership | Sponsor | Platform reliability KPIs, cost reduction, SLA compliance |
| Tenant Administrators | Admin | Tenant-specific dashboards, alert configuration, access management |

### 1.4 Observability-as-a-Service for ERP Modules

ERP-Observability functions as an internal observability platform, providing self-service monitoring capabilities to all 20+ ERP modules:

```mermaid
graph TB
    subgraph "ERP Suite (Consumers)"
        CRM["ERP-CRM"]
        IAM["ERP-IAM"]
        ACC["ERP-Accounting"]
        INV["ERP-Inventory"]
        HRM["ERP-HRM"]
        SCM["ERP-Supply-Chain"]
        PROJ["ERP-Projects"]
        PLAT["ERP-Platform"]
        DIR["ERP-Directory"]
        COMM["ERP-Communication"]
        WH["ERP-Warehouse"]
        MFG["ERP-Manufacturing"]
        PROC["ERP-Procurement"]
        PAY["ERP-Payroll"]
        BI["ERP-Analytics"]
        DOC["ERP-Documents"]
        QUAL["ERP-Quality"]
        MAINT["ERP-Maintenance"]
        FLEET["ERP-Fleet"]
        ASSET["ERP-Assets"]
    end

    subgraph "ERP-Observability (Provider)"
        METRICS["Metrics Service<br/>(VictoriaMetrics)"]
        LOGS["Log Service<br/>(Quickwit)"]
        TRACES["Trace Service<br/>(Quickwit)"]
        ALERTS["Alert Service<br/>(Alertmanager)"]
        INFRA["Infrastructure Service<br/>(Zabbix + OpenNMS)"]
        DASH["Dashboard Service<br/>(Grafana)"]
    end

    CRM & IAM & ACC & INV & HRM & SCM --> METRICS & LOGS & TRACES
    PROJ & PLAT & DIR & COMM & WH & MFG --> METRICS & LOGS & TRACES
    PROC & PAY & BI & DOC & QUAL & MAINT --> METRICS & LOGS & TRACES
    FLEET & ASSET --> METRICS & LOGS & TRACES
    METRICS & LOGS & TRACES --> ALERTS
    METRICS & LOGS & TRACES --> DASH
    ALERTS --> DASH
    INFRA --> ALERTS & DASH
```

## 2. Application Architecture

### 2.1 Application Portfolio

```mermaid
graph TB
    subgraph "ERP Suite"
        IAM2["ERP-IAM<br/>(Identity & Access)"]
        PLAT2["ERP-Platform<br/>(Control Plane)"]
        DIR2["ERP-Directory<br/>(User Provisioning)"]
        OBS["ERP-Observability<br/>(This Module)"]
    end

    subgraph "ERP-Observability Internal"
        GW["Go Gateway (:8090)"]
        API["Node.js observability-api (:3000)"]
        TAPI["Go tenant-api (:8080)"]
        WEB2["React Frontend"]
        GRAF["Grafana"]
    end

    IAM2 -->|OIDC/JWT| OBS
    PLAT2 -->|Entitlements| OBS
    DIR2 -->|Provisioning| OBS
    OBS --> GW
    OBS --> API
    OBS --> TAPI
    OBS --> WEB2
    OBS --> GRAF
```

### 2.2 Integration Architecture

```mermaid
flowchart TB
    subgraph "Telemetry Sources"
        OTEL_SDK["OTel SDKs<br/>(All ERP Modules)"]
        PROM_EP["Prometheus Endpoints<br/>(Legacy Metrics)"]
        SYSLOG["Syslog<br/>(Infrastructure)"]
        ZBX_AGENT["Zabbix Agents<br/>(Host Monitoring)"]
        ONMS_POLLER["OpenNMS Pollers<br/>(Service Availability)"]
    end

    subgraph "ERP-Observability"
        COLLECTOR["OTel Collector"]
        GW2["API Gateway"]
        HOOK["Webhook Engine"]
    end

    subgraph "ERP Platform"
        IAM3["ERP-IAM"]
        PLAT3["ERP-Platform"]
        PULSAR2["Apache Pulsar"]
        NATS2["NATS JetStream"]
    end

    subgraph "Notification Targets"
        EMAIL["Email (SMTP)"]
        SLACK["Slack"]
        PD["PagerDuty"]
        OG["OpsGenie"]
        WH2["Custom Webhooks"]
    end

    OTEL_SDK -->|OTLP gRPC/HTTP| COLLECTOR
    PROM_EP -->|Scrape| COLLECTOR
    SYSLOG -->|Syslog Receiver| COLLECTOR
    ZBX_AGENT -->|Zabbix Protocol| GW2
    ONMS_POLLER -->|REST/Events| GW2

    IAM3 -->|JWT Validation| GW2
    PLAT3 <-->|License/Feature Flags| GW2
    PULSAR2 <-->|Observability Events| GW2
    NATS2 <-->|Real-time Alerts| GW2

    GW2 --> HOOK
    HOOK --> EMAIL & SLACK & PD & OG & WH2
```

### 2.3 Application Decomposition

| Application Component | Technology | Deployment | Scaling Strategy |
|-----------------------|-----------|------------|-----------------|
| API Gateway | Go (net/http) | Docker/K8s | Horizontal (stateless) |
| Observability API | Node.js (Express) | Docker/K8s | Horizontal (stateless) |
| Tenant API | Go (net/http) | Docker/K8s | Horizontal (stateless) |
| VictoriaMetrics (vmselect) | Go | Docker/K8s | Horizontal (query parallelism) |
| VictoriaMetrics (vminsert) | Go | Docker/K8s | Horizontal (write throughput) |
| VictoriaMetrics (vmstorage) | Go | Docker/K8s (StatefulSet) | Vertical + horizontal sharding |
| Quickwit (Searcher) | Rust | Docker/K8s | Horizontal (search parallelism) |
| Quickwit (Indexer) | Rust | Docker/K8s | Horizontal (indexing throughput) |
| Grafana | Go | Docker/K8s | Horizontal (session affinity) |
| OTel Collector | Go | Docker/K8s (DaemonSet) | One per node |
| Alertmanager | Go | Docker/K8s | HA cluster (gossip) |
| Zabbix Server | C | Docker/K8s | Vertical |
| Zabbix Proxy | C | Docker/K8s | Per-location deployment |
| OpenNMS | Java | Docker/K8s | Vertical |
| DragonflyDB | C++ | Docker/K8s (StatefulSet) | Vertical + replication |
| YugabyteDB | C++ | Docker/K8s (StatefulSet) | Horizontal (automatic sharding) |
| RustFS | Rust | Docker/K8s (StatefulSet) | Horizontal (erasure coding) |
| Web Frontend | React (static) | CDN/Nginx | CDN edge caching |

## 3. Data Architecture

### 3.1 Data Flow Architecture

```mermaid
flowchart LR
    subgraph "Sources"
        APP["Application Telemetry<br/>(OTLP)"]
        INFRA2["Infrastructure Metrics<br/>(Zabbix/SNMP)"]
        NET["Network Events<br/>(OpenNMS)"]
        AUDIT["Audit Events<br/>(Pulsar)"]
    end

    subgraph "Ingestion"
        OTEL2["OTel Collector<br/>(Process + Route)"]
        ZBX2["Zabbix Server<br/>(Aggregate)"]
        ONMS2["OpenNMS<br/>(Correlate)"]
    end

    subgraph "Storage"
        VM2["VictoriaMetrics<br/>(Metrics TSDB)"]
        QW2["Quickwit<br/>(Logs + Traces Index)"]
        DF2["DragonflyDB<br/>(Query Cache)"]
        YB2["YugabyteDB<br/>(Config + State)"]
        RFS2["RustFS<br/>(Long-term Archive)"]
    end

    subgraph "Consumption"
        GRAF2["Grafana<br/>(Visualization)"]
        API2["Observability API<br/>(Programmatic)"]
        AM2["Alertmanager<br/>(Alerting)"]
    end

    APP --> OTEL2
    INFRA2 --> ZBX2
    NET --> ONMS2
    AUDIT --> OTEL2

    OTEL2 --> VM2 & QW2
    ZBX2 --> VM2
    ONMS2 --> QW2

    VM2 & QW2 --> GRAF2 & API2
    VM2 --> AM2
    DF2 --> API2
    YB2 --> API2
    QW2 --> RFS2
```

### 3.2 Data Classification

| Data Category | Sensitivity | Retention | Encryption |
|--------------|------------|-----------|-----------|
| Application Metrics | Low | 30 days (default), up to 5 years | TLS in transit |
| Application Logs | Medium (may contain PII) | 90 days (default) | AES-256 at rest, TLS in transit |
| Distributed Traces | Low | 7 days (default) | TLS in transit |
| Infrastructure Metrics (Zabbix) | Low | 30 days | TLS in transit |
| Network Events (OpenNMS) | Low | 30 days | TLS in transit |
| Alert History | Medium | 1 year | TLS in transit |
| Audit Events | High | 7 years (regulatory) | AES-256 at rest, immutable |
| Tenant Configuration | High | Indefinite | AES-256 at rest, TLS in transit |
| Dashboard Definitions | Low | Indefinite (Git-tracked) | TLS in transit |
| SLO Definitions | Medium | Indefinite | TLS in transit |

## 4. Technology Architecture

### 4.1 Infrastructure Topology

```mermaid
graph TB
    subgraph "Kubernetes Cluster (Harvester HCI)"
        subgraph "Ingress"
            LB["Load Balancer"]
            INGRESS["Nginx Ingress Controller"]
        end

        subgraph "Application Pods"
            GW_POD["Gateway<br/>(2+ replicas)"]
            API_POD["Observability API<br/>(2+ replicas)"]
            TAPI_POD["Tenant API<br/>(2+ replicas)"]
            WEB_POD["Web Frontend<br/>(Nginx)"]
            GRAF_POD["Grafana<br/>(2+ replicas)"]
        end

        subgraph "Telemetry Pods"
            OTEL_DS["OTel Collector<br/>(DaemonSet)"]
            AM_POD["Alertmanager<br/>(3-node HA)"]
        end

        subgraph "Metrics Pods"
            VM_INSERT["vminsert<br/>(2+ replicas)"]
            VM_SELECT["vmselect<br/>(2+ replicas)"]
            VM_STORAGE["vmstorage<br/>(3+ StatefulSet)"]
        end

        subgraph "Search Pods"
            QW_INDEX["Quickwit Indexer<br/>(2+ replicas)"]
            QW_SEARCH["Quickwit Searcher<br/>(2+ replicas)"]
        end

        subgraph "Infrastructure Pods"
            ZBX_POD["Zabbix Server"]
            ONMS_POD["OpenNMS"]
        end

        subgraph "Data Pods"
            DF_POD["DragonflyDB<br/>(Primary + Replica)"]
            YB_POD["YugabyteDB<br/>(3-node cluster)"]
            RFS_POD["RustFS<br/>(4+ nodes, erasure coding)"]
        end

        subgraph "Storage"
            MAYASTOR["Mayastor/Vitastor<br/>Persistent Volumes"]
        end
    end

    LB --> INGRESS
    INGRESS --> GW_POD & API_POD & TAPI_POD & WEB_POD & GRAF_POD
    OTEL_DS --> VM_INSERT & QW_INDEX
    VM_INSERT --> VM_STORAGE
    VM_STORAGE --> MAYASTOR
    QW_INDEX --> RFS_POD
    DF_POD & YB_POD --> MAYASTOR
```

### 4.2 Technology Standards

| Standard | Implementation |
|----------|---------------|
| Authentication | OIDC/JWT via ERP-IAM |
| Authorization | RBAC with tenant isolation (X-Scope-OrgID) |
| API Style | REST (gateway) + PromQL (metrics) + Quickwit Query (logs) |
| Telemetry Protocol | OTLP (gRPC + HTTP) |
| Metric Query | PromQL (VictoriaMetrics-compatible) |
| Log Query | Quickwit Query Language |
| Observability | Self-monitoring via internal OTel pipeline |
| Storage | Mayastor/Vitastor on Harvester HCI |
| Object Storage | RustFS (S3-compatible) |
| CI/CD | GitHub Actions |
| Containerization | Docker (multi-stage) |
| Orchestration | Kubernetes + Helm |
| Event Format | CloudEvents v1.0 |

## 5. Governance

### 5.1 Architecture Principles

1. **AIDD Compliance**: All technology choices must comply with AIDD standards -- VictoriaMetrics (not Prometheus), Quickwit (not Elasticsearch), DragonflyDB (not Redis), YugabyteDB (not PostgreSQL)
2. **OTel-native**: All telemetry must flow through OpenTelemetry, no proprietary agent lock-in
3. **Multi-tenant by design**: All data scoped by tenant at every layer via X-Scope-OrgID
4. **Self-service**: Module teams can create dashboards, alerts, and SLOs without platform team intervention
5. **Sovereign-first**: Self-hosted, no external SaaS dependencies for core observability
6. **Immutable audit**: All configuration changes and alerts are immutably logged
7. **Cost-aware**: Storage tiering with automatic archival to RustFS for long-term retention

### 5.2 Architecture Review Checklist

- [ ] All new telemetry sources use OTel SDKs or OTel Collector receivers
- [ ] Tenant isolation verified at API gateway and storage layers
- [ ] Alert rules reviewed for noise and actionability
- [ ] Dashboard templates provisioned for new modules
- [ ] Retention policies configured per data classification
- [ ] Performance benchmarks established for query latency targets
- [ ] AIDD compliance verified (no banned technologies)
- [ ] Architecture Decision Record created for significant changes
