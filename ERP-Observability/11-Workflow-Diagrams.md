# ERP-Observability Workflow Diagrams

## 1. Alert Lifecycle Workflow

The complete lifecycle of an alert from threshold breach through resolution.

```mermaid
flowchart TB
    START([Metric Threshold<br/>Breached]) --> EVAL["vmalert Evaluates<br/>Alert Rule (PromQL)"]
    EVAL --> PENDING{Duration<br/>Met?}

    PENDING -->|No, recovered| CLEAR["Alert Cleared<br/>(no notification)"]
    PENDING -->|Yes| FIRE["Alert Fires<br/>Status: Firing"]

    FIRE --> AM["Alertmanager<br/>Receives Alert"]
    AM --> GROUP["Group Alerts<br/>(by module, severity)"]
    GROUP --> DEDUP["Deduplicate<br/>(same alert fingerprint)"]
    DEDUP --> INHIBIT{Inhibition<br/>Rule Match?}

    INHIBIT -->|Yes| SUPPRESS["Alert Suppressed<br/>(higher severity active)"]
    INHIBIT -->|No| SILENCE{Silence<br/>Active?}

    SILENCE -->|Yes| SILENCED["Alert Silenced<br/>(maintenance window)"]
    SILENCE -->|No| ROUTE["Route to Receiver<br/>(based on labels)"]

    ROUTE --> NOTIFY_CRITICAL["Critical:<br/>PagerDuty/OpsGenie Page"]
    ROUTE --> NOTIFY_WARNING["Warning:<br/>Slack Channel"]
    ROUTE --> NOTIFY_INFO["Info:<br/>Email Digest"]

    NOTIFY_CRITICAL & NOTIFY_WARNING & NOTIFY_INFO --> ACKNOWLEDGE["SRE Acknowledges<br/>Alert"]
    ACKNOWLEDGE --> INVESTIGATE2["Investigate Root Cause<br/>(Metrics/Logs/Traces)"]
    INVESTIGATE2 --> FIX2["Apply Remediation"]
    FIX2 --> RECOVER{Metric<br/>Recovers?}

    RECOVER -->|Yes| RESOLVED["Alert Resolved<br/>Status: Resolved"]
    RECOVER -->|No| ESCALATE3["Escalate to<br/>Next On-Call"]
    ESCALATE3 --> INVESTIGATE2

    RESOLVED --> HISTORY["Record in<br/>Alert History<br/>(Quickwit)"]
    HISTORY --> POSTMORTEM["Postmortem<br/>(if severity >= Warning)"]

    style FIRE fill:#ef4444,color:#fff
    style RESOLVED fill:#22c55e,color:#fff
    style SILENCED fill:#f59e0b,color:#fff
    style SUPPRESS fill:#6b7280,color:#fff
```

## 2. Log Ingestion Pipeline Workflow

From application log emission through storage and search.

```mermaid
flowchart TB
    subgraph "Application Layer"
        APP1["ERP Module 1<br/>(OTel SDK)"]
        APP2["ERP Module 2<br/>(OTel SDK)"]
        APP3["ERP Module N<br/>(OTel SDK)"]
        SYSLOG["Infrastructure<br/>(Syslog)"]
    end

    subgraph "Collection Layer"
        RECV["OTel Collector<br/>Receivers"]
        RECV_OTLP["OTLP Receiver<br/>gRPC :4317 / HTTP :4318"]
        RECV_SYSLOG["Syslog Receiver<br/>TCP :514"]
    end

    subgraph "Processing Layer"
        MEMLIMIT["Memory Limiter<br/>(80% check interval)"]
        ATTR["Attributes Processor<br/>(Inject tenant_id)"]
        FILTER["Filter Processor<br/>(Drop DEBUG in prod)"]
        RESOURCE["Resource Processor<br/>(Standardize attributes)"]
        BATCH["Batch Processor<br/>(200ms timeout, 8192 items)"]
    end

    subgraph "Export Layer"
        EXPORT["OTLP Exporter<br/>(Quickwit)"]
    end

    subgraph "Storage Layer"
        INDEXER["Quickwit Indexer<br/>(Parse, tokenize, index)"]
        SPLITS["Index Splits<br/>(Time-partitioned)"]
        RUSTFS["RustFS S3<br/>(Long-term storage)"]
    end

    subgraph "Query Layer"
        SEARCHER["Quickwit Searcher<br/>(Distributed search)"]
        CACHE["DragonflyDB<br/>(Query cache)"]
        API3["Observability API<br/>(Search proxy)"]
    end

    APP1 & APP2 & APP3 -->|OTLP| RECV_OTLP
    SYSLOG --> RECV_SYSLOG
    RECV_OTLP & RECV_SYSLOG --> MEMLIMIT
    MEMLIMIT --> ATTR --> FILTER --> RESOURCE --> BATCH
    BATCH --> EXPORT
    EXPORT --> INDEXER
    INDEXER --> SPLITS
    SPLITS --> RUSTFS
    SEARCHER --> SPLITS & CACHE
    API3 --> SEARCHER

    style MEMLIMIT fill:#f59e0b,color:#fff
    style BATCH fill:#3b82f6,color:#fff
```

## 3. Metric Collection Flow

From application instrumentation through VictoriaMetrics storage.

```mermaid
flowchart TB
    subgraph "Instrumentation"
        SDK["OTel SDK<br/>(Counter, Gauge, Histogram)"]
        PROM_EP["Prometheus /metrics<br/>(Legacy endpoints)"]
        ZBX_AGENT2["Zabbix Agent<br/>(Host metrics)"]
    end

    subgraph "OTel Collector"
        OTLP_RECV["OTLP Receiver"]
        PROM_RECV["Prometheus Receiver<br/>(Scrape config)"]
        PROC["Processors<br/>(Memory limit, batch,<br/>attributes, filter)"]
        RW_EXPORT["Remote Write Exporter"]
    end

    subgraph "VictoriaMetrics Cluster"
        VMAUTH["vmauth<br/>(Tenant routing proxy)"]
        VMINSERT["vminsert<br/>(Write path)"]
        VMSTORAGE["vmstorage<br/>(Time series storage)"]
        VMSELECT["vmselect<br/>(Read path, PromQL)"]
        VMALERT["vmalert<br/>(Alert rule evaluation)"]
    end

    subgraph "Alerting"
        AM2["Alertmanager<br/>(Alert routing)"]
    end

    subgraph "Visualization"
        GRAFANA2["Grafana<br/>(PromQL datasource)"]
        FRONTEND["React Frontend<br/>(Metric Explorer)"]
    end

    SDK -->|OTLP| OTLP_RECV
    PROM_EP -->|Scrape| PROM_RECV
    ZBX_AGENT2 -->|Zabbix Protocol| VMINSERT

    OTLP_RECV & PROM_RECV --> PROC
    PROC --> RW_EXPORT
    RW_EXPORT -->|Remote Write| VMAUTH
    VMAUTH --> VMINSERT
    VMINSERT --> VMSTORAGE
    VMSELECT --> VMSTORAGE
    VMALERT --> VMSELECT
    VMALERT --> AM2

    GRAFANA2 -->|PromQL| VMAUTH --> VMSELECT
    FRONTEND -->|API| VMAUTH

    style VMSTORAGE fill:#059669,color:#fff
    style VMINSERT fill:#3b82f6,color:#fff
    style VMSELECT fill:#8b5cf6,color:#fff
```

## 4. Trace Propagation Workflow

How trace context propagates across ERP services.

```mermaid
sequenceDiagram
    participant Client as Browser/Mobile
    participant GW as API Gateway
    participant CRM as ERP-CRM
    participant IAM as ERP-IAM
    participant DB as YugabyteDB
    participant Collector as OTel Collector
    participant QW as Quickwit

    Note over Client,QW: Trace Context: W3C traceparent header

    Client->>GW: GET /api/v1/contacts<br/>traceparent: 00-{traceId}-{spanId}-01
    activate GW
    GW->>GW: Create Span: "gateway.route"

    GW->>CRM: Forward Request<br/>traceparent: 00-{traceId}-{gwSpanId}-01
    activate CRM
    CRM->>CRM: Create Span: "crm.list_contacts"

    CRM->>IAM: Validate JWT<br/>traceparent: 00-{traceId}-{crmSpanId}-01
    activate IAM
    IAM->>IAM: Create Span: "iam.validate_token"
    IAM-->>CRM: Token Valid
    deactivate IAM

    CRM->>DB: SELECT * FROM contacts<br/>traceparent: 00-{traceId}-{crmSpanId2}-01
    activate DB
    DB->>DB: Create Span: "db.query"
    DB-->>CRM: Result Set
    deactivate DB

    CRM-->>GW: JSON Response
    deactivate CRM

    GW-->>Client: 200 OK + Contacts
    deactivate GW

    Note over GW,QW: Spans exported asynchronously

    GW->>Collector: Export Spans (OTLP)
    CRM->>Collector: Export Spans (OTLP)
    IAM->>Collector: Export Spans (OTLP)
    DB->>Collector: Export Spans (OTLP)

    Collector->>Collector: Batch + Process + Enrich
    Collector->>QW: Export Trace (OTLP)
    QW->>QW: Index Trace<br/>(all spans with same traceId)
```

## 5. Tenant Provisioning Workflow

Automated stack provisioning when a new tenant is onboarded.

```mermaid
flowchart TB
    START4([Admin Creates<br/>New Tenant]) --> VALIDATE2["Validate Tenant<br/>Information"]
    VALIDATE2 --> STORE["Store Tenant Record<br/>in YugabyteDB"]
    STORE --> PROVISION2["Start Parallel<br/>Provisioning"]

    PROVISION2 --> P1["Create Grafana Org<br/>+ Admin User<br/>+ Datasources"]
    PROVISION2 --> P2["Create VM Namespace<br/>+ vmauth Route<br/>+ Recording Rules"]
    PROVISION2 --> P3["Create Quickwit Indexes<br/>logs-{tenant_id}<br/>traces-{tenant_id}"]
    PROVISION2 --> P4["Create Zabbix<br/>Host Group<br/>+ Templates"]
    PROVISION2 --> P5["Create Alertmanager<br/>Route + Default Rules"]

    P1 --> CHECK1{Success?}
    P2 --> CHECK2{Success?}
    P3 --> CHECK3{Success?}
    P4 --> CHECK4{Success?}
    P5 --> CHECK5{Success?}

    CHECK1 -->|Yes| D1["Provision Default<br/>Dashboards"]
    CHECK2 -->|Yes| D2["Create Default<br/>Recording Rules"]
    CHECK3 -->|Yes| D3["Configure<br/>Retention Policies"]
    CHECK4 -->|Yes| D4["Assign Default<br/>Monitoring Templates"]
    CHECK5 -->|Yes| D5["Create Default<br/>Alert Rules"]

    CHECK1 & CHECK2 & CHECK3 & CHECK4 & CHECK5 -->|No| RETRY["Retry with<br/>Exponential Backoff"]
    RETRY --> FAIL{Max Retries<br/>Exceeded?}
    FAIL -->|Yes| MANUAL["Mark for Manual<br/>Intervention"]
    FAIL -->|No| PROVISION2

    D1 & D2 & D3 & D4 & D5 --> ACTIVATE2["Set Tenant Status:<br/>Active"]
    ACTIVATE2 --> NOTIFY2["Send Onboarding<br/>Guide to Admin"]
    NOTIFY2 --> AUDIT3["Log Provisioning<br/>to Audit Trail"]

    MANUAL --> AUDIT3

    style ACTIVATE2 fill:#22c55e,color:#fff
    style MANUAL fill:#ef4444,color:#fff
```

## 6. Dashboard Creation Workflow

From user intent to interactive Grafana dashboard.

```mermaid
flowchart TB
    USER([User Opens<br/>Dashboard Builder]) --> TEMPLATE{Start From<br/>Template?}

    TEMPLATE -->|Yes| SELECT["Select Template<br/>(Module Health, SLO, Custom)"]
    TEMPLATE -->|No| BLANK["Create Blank<br/>Dashboard"]

    SELECT --> CUSTOMIZE["Customize Template<br/>Variables + Panels"]
    BLANK --> ADD_PANEL["Add First Panel"]

    CUSTOMIZE --> EDIT["Edit Panels"]
    ADD_PANEL --> EDIT

    EDIT --> CHOOSE_TYPE{Panel<br/>Type?}
    CHOOSE_TYPE -->|Time Series| PROMQL_EDIT["Write PromQL Query<br/>Configure Axes + Legend"]
    CHOOSE_TYPE -->|Log Panel| QW_EDIT["Write Quickwit Query<br/>Configure Fields"]
    CHOOSE_TYPE -->|Stat| STAT_EDIT["Write Instant Query<br/>Configure Thresholds"]
    CHOOSE_TYPE -->|Table| TABLE_EDIT["Write Range Query<br/>Configure Columns"]
    CHOOSE_TYPE -->|Zabbix| ZBX_EDIT["Select Host + Item<br/>Configure Display"]

    PROMQL_EDIT & QW_EDIT & STAT_EDIT & TABLE_EDIT & ZBX_EDIT --> PREVIEW["Preview Panel<br/>(Live Data)"]
    PREVIEW --> ADJUST{Looks<br/>Good?}
    ADJUST -->|No| EDIT
    ADJUST -->|Yes| MORE{More<br/>Panels?}

    MORE -->|Yes| ADD_PANEL
    MORE -->|No| VARIABLES["Add Variables<br/>(Module, Environment)"]

    VARIABLES --> ARRANGE["Arrange Panel<br/>Layout (Grid)"]
    ARRANGE --> SAVE["Save Dashboard<br/>(Name, Folder, Tags)"]
    SAVE --> SHARE["Share Dashboard<br/>(URL, Embed, Export)"]

    style SAVE fill:#22c55e,color:#fff
```

## 7. Incident Response Workflow

End-to-end incident response using the observability platform.

```mermaid
flowchart TB
    ALERT_FIRE([Alert Fires:<br/>Critical Severity]) --> PAGE["PagerDuty Pages<br/>On-Call SRE"]
    PAGE --> ACK["SRE Acknowledges<br/>Alert (< 5 min)"]

    ACK --> TRIAGE["Triage:<br/>Check Main Dashboard"]
    TRIAGE --> SCOPE2{Scope of<br/>Impact?}

    SCOPE2 -->|Single Module| SINGLE2["Open Module<br/>Dashboard"]
    SCOPE2 -->|Multiple Modules| CROSS2["Open Cross-Module<br/>Correlation View"]
    SCOPE2 -->|Infrastructure| INFRA_CHECK["Open Zabbix<br/>Host Overview"]

    SINGLE2 --> METRICS3["Analyze Metrics<br/>(RED: Rate, Error, Duration)"]
    CROSS2 --> SERVICE_MAP["Review Service Map<br/>(Identify Common Root)"]
    INFRA_CHECK --> ZBX_TRIGGERS["Review Active<br/>Zabbix Triggers"]

    METRICS3 --> LOGS2["Search Error Logs<br/>(Quickwit)"]
    SERVICE_MAP --> LOGS2
    ZBX_TRIGGERS --> LOGS2

    LOGS2 --> TRACES2["Follow Traces<br/>(Waterfall View)"]
    TRACES2 --> ROOT["Identify<br/>Root Cause"]

    ROOT --> COMMUNICATE["Communicate Impact<br/>(Slack Incident Channel)"]
    COMMUNICATE --> MITIGATE{Mitigation<br/>Available?}

    MITIGATE -->|Rollback| ROLLBACK["Rollback Deployment"]
    MITIGATE -->|Config Change| CONFIG_FIX["Apply Config Fix"]
    MITIGATE -->|Scale| SCALE_UP["Scale Infrastructure"]
    MITIGATE -->|Code Fix| HOTFIX["Deploy Hotfix"]

    ROLLBACK & CONFIG_FIX & SCALE_UP & HOTFIX --> VERIFY3["Verify Recovery<br/>(Metrics Normal)"]
    VERIFY3 --> RECOVERED{Metrics<br/>Recovered?}

    RECOVERED -->|Yes| RESOLVE2["Resolve Alert<br/>+ Remove Silence"]
    RECOVERED -->|No| ESCALATE4["Escalate to<br/>Next On-Call / Expert"]
    ESCALATE4 --> ROOT

    RESOLVE2 --> ANNOTATE["Add Grafana Annotations<br/>(Incident Start + End)"]
    ANNOTATE --> DOCUMENT["Write Postmortem<br/>(Links to Observability Data)"]
    DOCUMENT --> IMPROVE2["Create Action Items<br/>(Better Alerts, Runbooks)"]

    style ALERT_FIRE fill:#ef4444,color:#fff
    style RESOLVE2 fill:#22c55e,color:#fff
    style ESCALATE4 fill:#f59e0b,color:#fff
```

## 8. OTel Collector Pipeline Configuration Workflow

How OTel Collector processes telemetry data.

```mermaid
flowchart LR
    subgraph "Receivers"
        direction TB
        R1["otlp (gRPC :4317)"]
        R2["otlp (HTTP :4318)"]
        R3["prometheus (scrape)"]
        R4["syslog (TCP :514)"]
        R5["hostmetrics (system)"]
    end

    subgraph "Processors"
        direction TB
        P1_2["memory_limiter<br/>check_interval: 1s<br/>limit_mib: 4096<br/>spike_limit_mib: 512"]
        P2_2["attributes<br/>actions:<br/>  - key: tenant_id<br/>    from_context: X-Tenant-ID<br/>    action: upsert"]
        P3_2["filter<br/>error_mode: ignore<br/>logs:<br/>  exclude:<br/>    severity: DEBUG"]
        P4_2["resource<br/>attributes:<br/>  - key: deployment.environment<br/>    value: production<br/>    action: upsert"]
        P5_2["batch<br/>timeout: 200ms<br/>send_batch_size: 8192<br/>send_batch_max_size: 16384"]
        P6_2["tail_sampling<br/>policies:<br/>  - always_sample (errors)<br/>  - probabilistic (10%)"]
    end

    subgraph "Exporters"
        direction TB
        E1_2["prometheusremotewrite<br/>endpoint: http://vminsert:8480<br/>headers:<br/>  X-Scope-OrgID: ${tenant_id}"]
        E2_2["otlp (logs)<br/>endpoint: http://quickwit:7281<br/>tls: insecure"]
        E3_2["otlp (traces)<br/>endpoint: http://quickwit:7281<br/>tls: insecure"]
    end

    R1 & R2 & R3 & R4 & R5 --> P1_2
    P1_2 --> P2_2 --> P3_2 --> P4_2 --> P5_2
    P5_2 --> E1_2 & E2_2
    P4_2 --> P6_2 --> E3_2
```

## 9. Self-Monitoring Pipeline Workflow

How the observability platform monitors itself.

```mermaid
flowchart TB
    subgraph "Platform Components"
        VM3["VictoriaMetrics<br/>Internal Metrics"]
        QW3["Quickwit<br/>Internal Metrics"]
        OTEL3["OTel Collector<br/>Internal Metrics"]
        AM3["Alertmanager<br/>Internal Metrics"]
        GW4["Gateway<br/>Internal Metrics"]
        GRAF3["Grafana<br/>Internal Metrics"]
    end

    subgraph "Self-Monitoring OTel Collector"
        SELF_RECV["Prometheus Receiver<br/>(Scrape internal endpoints)"]
        SELF_PROC["Process + Batch"]
        SELF_EXPORT["Export to VM<br/>(Self-monitoring namespace)"]
    end

    subgraph "Self-Monitoring Alerts"
        DEAD_MAN["Dead Man's Switch<br/>(Always-firing alert)"]
        INGEST_LAG["Ingestion Lag Alert<br/>(Queue backlog)"]
        STORAGE_FULL["Storage Alert<br/>(Capacity warning)"]
        QUERY_SLOW["Query Slowness Alert<br/>(p99 > threshold)"]
    end

    subgraph "Self-Monitoring Dashboard"
        HEALTH_DASH["Platform Health Dashboard<br/>Component status, ingestion rates,<br/>query latency, storage usage"]
    end

    VM3 & QW3 & OTEL3 & AM3 & GW4 & GRAF3 --> SELF_RECV
    SELF_RECV --> SELF_PROC --> SELF_EXPORT
    SELF_EXPORT --> DEAD_MAN & INGEST_LAG & STORAGE_FULL & QUERY_SLOW
    SELF_EXPORT --> HEALTH_DASH

    style DEAD_MAN fill:#22c55e,color:#fff
    style STORAGE_FULL fill:#f59e0b,color:#fff
```
