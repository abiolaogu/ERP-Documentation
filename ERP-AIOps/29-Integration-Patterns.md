# ERP-AIOps Integration Patterns

> **Document ID:** ERP-AIOPS-INT-029
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [23-Webhook-Specifications.md](./23-Webhook-Specifications.md), [12-High-Level-Design.md](./12-High-Level-Design.md)

---

## 1. OTel Collector Federation with All ERP Modules

### Federation Architecture

Every ERP module runs a local OTel Collector sidecar that ships telemetry to the AIOps central OTel Collector gateway. This federation pattern ensures each module's telemetry is enriched with module-specific metadata before reaching AIOps.

```
┌──────────────────────────────────────────────────────────────────────┐
│  ERP Module (e.g., ERP-CRM)                                        │
│                                                                      │
│  ┌─────────────┐    ┌──────────────────────────────────────┐        │
│  │ Application  │───>│ Local OTel Collector (Sidecar)       │        │
│  │ (OTel SDK)   │    │                                      │        │
│  │              │    │ Receivers:  otlp (gRPC :4317)        │        │
│  │ Metrics      │    │ Processors: batch, resource,         │        │
│  │ Logs         │    │             attributes (module=crm)  │        │
│  │ Traces       │    │ Exporters:  otlp (to AIOps gateway)  │        │
│  └─────────────┘    └──────────────────┬───────────────────┘        │
└─────────────────────────────────────────┼────────────────────────────┘
                                          │
                                          │ OTLP gRPC
                                          v
┌──────────────────────────────────────────────────────────────────────┐
│  AIOps OTel Gateway Collector                                       │
│                                                                      │
│  Receivers:  otlp (gRPC :4317, HTTP :4318)                          │
│  Processors:                                                         │
│    - batch (200ms window, 1024 batch size)                           │
│    - memory_limiter (limit 2GB, spike 512MB)                         │
│    - resource (add tenant_id from auth)                              │
│    - filter (drop debug/verbose)                                     │
│    - transform (normalize metric names)                              │
│  Exporters:                                                          │
│    - otlp → AIOps Rust Ingestion Pipeline                           │
│    - prometheusremotewrite → VictoriaMetrics                        │
│    - otlp → Quickwit (logs)                                         │
│    - otlp → Tempo (traces)                                          │
└──────────────────────────────────────────────────────────────────────┘
```

### Module-Specific Collector Configuration

Each module's collector adds module identification attributes:

```yaml
# otel-collector-config.yaml (per-module sidecar)
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"

processors:
  resource:
    attributes:
      - key: erp.module
        value: "crm"
        action: upsert
      - key: erp.version
        value: "${MODULE_VERSION}"
        action: upsert
  batch:
    send_batch_size: 512
    timeout: 200ms

exporters:
  otlp:
    endpoint: "aiops-otel-gateway:4317"
    tls:
      insecure: false
      cert_file: /certs/client.crt
      key_file: /certs/client.key

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [resource, batch]
      exporters: [otlp]
    logs:
      receivers: [otlp]
      processors: [resource, batch]
      exporters: [otlp]
    traces:
      receivers: [otlp]
      processors: [resource, batch]
      exporters: [otlp]
```

### Standard Metrics Expected from Each Module

| Metric | Type | Description |
|--------|------|-------------|
| `http_request_duration_seconds` | Histogram | Request latency distribution |
| `http_requests_total` | Counter | Total request count by status code |
| `process_cpu_seconds_total` | Counter | CPU usage |
| `process_resident_memory_bytes` | Gauge | Memory usage |
| `db_connections_active` | Gauge | Database connection pool usage |
| `db_query_duration_seconds` | Histogram | Database query latency |

---

## 2. VictoriaMetrics Metric Queries

AIOps queries VictoriaMetrics (part of ERP-Observability) for historical metrics used in anomaly detection, RCA, and forecasting.

### Connection Configuration

```yaml
observability:
  victoriametrics:
    url: "http://victoriametrics.observability.svc:8428"
    auth:
      type: "bearer"
      token: "${VM_TOKEN}"
    timeout: 30s
    max_concurrent_queries: 10
```

### Common Query Patterns

**Baseline Computation (7-day average):**

```promql
avg_over_time(
  http_request_duration_seconds{
    service="erp-crm",
    quantile="0.99"
  }[7d]
)
```

**Anomaly Context (metrics around incident time):**

```promql
{
  service="erp-crm",
  __name__=~"http_requests_total|process_cpu_seconds_total|db_connections_active"
}[1h] offset 0s  # Centered around incident time
```

**Forecasting Input (90-day export):**

```
GET /api/v1/export?match[]={service="erp-finance",__name__="disk_usage_bytes"}&start=-90d
```

**Cross-Service Comparison:**

```promql
topk(10,
  rate(http_requests_total{status=~"5.."}[5m])
  / rate(http_requests_total[5m])
) > 0.01
```

---

## 3. Quickwit Log Queries for RCA

AIOps queries Quickwit (part of ERP-Observability) for log evidence during Root Cause Analysis.

### Connection Configuration

```yaml
observability:
  quickwit:
    url: "http://quickwit.observability.svc:7280"
    index: "erp-logs"
    auth:
      type: "bearer"
      token: "${QW_TOKEN}"
    timeout: 30s
```

### RCA Log Query Patterns

**Error logs around incident time:**

```json
{
  "query": "service:erp-iam AND level:ERROR",
  "start_timestamp": 1708769400,
  "end_timestamp": 1708773000,
  "max_hits": 100,
  "sort_by": "timestamp"
}
```

**Deployment/change events:**

```json
{
  "query": "event_type:deployment AND service:erp-iam",
  "start_timestamp": 1708762200,
  "end_timestamp": 1708773000,
  "max_hits": 20
}
```

**Configuration changes:**

```json
{
  "query": "event_type:config_change AND service:erp-iam",
  "start_timestamp": 1708762200,
  "end_timestamp": 1708773000,
  "max_hits": 20
}
```

**Full-text search across all modules:**

```json
{
  "query": "\"connection refused\" OR \"timeout\" OR \"pool exhausted\"",
  "start_timestamp": 1708769400,
  "end_timestamp": 1708773000,
  "max_hits": 200,
  "sort_by": "timestamp"
}
```

---

## 4. GitHub Incident Sync

### Configuration

```yaml
integrations:
  github:
    enabled: true
    app_id: "${GITHUB_APP_ID}"
    private_key_path: "/secrets/github-app.pem"
    installation_id: "${GITHUB_INSTALLATION_ID}"
    repository: "org/erp-operations"
    auto_create_for: ["P1", "P2"]
    label_mapping:
      P1: ["critical", "aiops-incident"]
      P2: ["high-priority", "aiops-incident"]
      P3: ["medium-priority", "aiops-incident"]
    sync_direction: "bidirectional"
```

### Sync Flow

```
AIOps Incident Created (P1/P2)
       │
       v
GitHub Issue Created ──────────────────────────────────────────┐
  - Title: [P{severity}] {incident_title}                     │
  - Body: Incident details, RCA summary, link to AIOps        │
  - Labels: Per mapping                                        │
  - Assignee: Mapped from on-call engineer                     │
       │                                                        │
       ├──── AIOps comment added ──> GitHub comment added      │
       ├──── AIOps state change ──> GitHub label updated       │
       ├──── GitHub comment added ──> AIOps activity posted    │
       ├──── GitHub label added ──> AIOps tag added            │
       │                                                        │
AIOps Incident Resolved                                        │
       │                                                        │
       v                                                        │
GitHub Issue Closed                                             │
  - Resolution comment with summary                             │
  - Post-mortem link attached                                   │
```

### API Usage

AIOps uses the GitHub App API (not personal access tokens) for enhanced security and higher rate limits.

```python
# Creating a GitHub issue from an incident
async def create_github_issue(incident: Incident) -> int:
    headers = {
        "Authorization": f"Bearer {generate_jwt()}",
        "Accept": "application/vnd.github+json",
    }

    body = {
        "title": f"[{incident.severity}] {incident.title}",
        "body": format_incident_body(incident),
        "labels": get_labels(incident.severity),
        "assignees": [map_assignee(incident.assignee)],
    }

    response = await httpx.post(
        f"https://api.github.com/repos/{REPO}/issues",
        json=body,
        headers=headers,
    )

    return response.json()["number"]
```

---

## 5. Slack/Teams Notifications

### Slack Integration

**Configuration:**

```yaml
integrations:
  slack:
    enabled: true
    bot_token: "${SLACK_BOT_TOKEN}"
    channels:
      incidents: "#aiops-incidents"
      anomalies: "#aiops-anomalies"
      remediation: "#aiops-remediation"
    notification_rules:
      - events: ["incident.created"]
        severity: ["P1", "P2"]
        channel: "#aiops-incidents"
      - events: ["remediation.approval_required"]
        channel: "#aiops-remediation"
    interactive: true  # Enable button actions (acknowledge, approve)
```

**Interactive Actions:**

Slack messages include action buttons that call back to the AIOps API:

| Button | Action | AIOps API Call |
|--------|--------|----------------|
| Acknowledge | Transition incident to Acknowledged | `POST /incidents/:id/transition {state: "acknowledged"}` |
| Investigate | Transition to Investigating | `POST /incidents/:id/transition {state: "investigating"}` |
| Approve Remediation | Approve pending action | `POST /remediation/approve/:id` |
| Reject Remediation | Reject pending action | `POST /remediation/reject/:id` |

### Microsoft Teams Integration

```yaml
integrations:
  teams:
    enabled: true
    webhook_url: "${TEAMS_WEBHOOK_URL}"
    notification_rules:
      - events: ["incident.created"]
        severity: ["P1", "P2"]
```

---

## 6. PagerDuty Integration

### Configuration

```yaml
integrations:
  pagerduty:
    enabled: true
    api_key: "${PAGERDUTY_API_KEY}"
    service_mapping:
      default: "PAIOPS_SERVICE_ID"
      P1: "PAIOPS_P1_ESCALATION"
      P2: "PAIOPS_P2_ESCALATION"
    auto_trigger_for: ["P1", "P2"]
    auto_resolve: true
```

### Event Mapping

| AIOps Event | PagerDuty Action |
|-------------|-----------------|
| Incident created (P1/P2) | Trigger alert |
| Incident acknowledged | Acknowledge alert |
| Incident resolved | Resolve alert |
| Incident severity upgraded | Update alert severity |
| Incident severity downgraded | Update alert severity |

### PagerDuty Event Payload

```json
{
  "routing_key": "INTEGRATION_KEY",
  "event_action": "trigger",
  "dedup_key": "aiops-inc-550e8400",
  "payload": {
    "summary": "[P1] ERP-IAM Authentication Cascade Failure",
    "severity": "critical",
    "source": "ERP-AIOps",
    "component": "ERP-IAM",
    "group": "Authentication",
    "custom_details": {
      "incident_id": "inc-550e8400",
      "affected_services": ["erp-iam", "erp-crm", "erp-finance"],
      "anomaly_count": 8,
      "aiops_url": "https://aiops.erp.internal/incidents/inc-550e8400"
    }
  },
  "links": [
    {
      "href": "https://aiops.erp.internal/incidents/inc-550e8400",
      "text": "View in AIOps"
    }
  ]
}
```

---

## 7. Observability Module Bidirectional Integration

ERP-AIOps and ERP-Observability have a deep bidirectional relationship. Observability provides the data infrastructure; AIOps provides the intelligence layer.

### Data Flow: Observability to AIOps

| Data | Source | Protocol | Purpose |
|------|--------|----------|---------|
| Metrics | VictoriaMetrics | PromQL HTTP API | Anomaly detection baselines, forecasting input |
| Logs | Quickwit | Search API | RCA evidence, change detection |
| Traces | Tempo | TraceQL API | Topology discovery, latency analysis |
| Alerts | Alertmanager | Webhook | External alert ingestion |

### Data Flow: AIOps to Observability

| Data | Destination | Protocol | Purpose |
|------|-------------|----------|---------|
| Anomaly scores | VictoriaMetrics | Remote Write | Store anomaly scores as metrics for Grafana dashboards |
| Incident annotations | Grafana | Annotation API | Show incident markers on metric charts |
| Topology data | Grafana | Data source plugin | Display service map in Grafana |
| Health scores | VictoriaMetrics | Remote Write | Per-module health scores for dashboards |

### Grafana Dashboard Integration

AIOps publishes key metrics back to VictoriaMetrics, making them available in Grafana dashboards:

```promql
# AIOps-generated metrics available in Grafana
aiops_health_score{module="erp-crm", tenant="tenant-001"}
aiops_anomaly_score{service="erp-crm", metric="cpu_usage"}
aiops_incident_count{severity="P1", state="active"}
aiops_remediation_success_rate{playbook="auto-scale"}
aiops_cost_savings_monthly{tenant="tenant-001"}
```

### Cross-Platform Linking

Grafana dashboards include deep links to AIOps:
- Metric panel context menu: "Analyze in AIOps" opens the anomaly detail page.
- Alert notification: Includes link to corresponding AIOps incident.
- Service map: Clicking a node links to AIOps topology view for that service.

AIOps dashboards include deep links to Grafana:
- Incident timeline: "View in Grafana" opens the time-correlated dashboard.
- Anomaly detail: "Explore metric" opens the metric in Grafana Explore.
- RCA evidence: Log links open Quickwit search in Grafana.
