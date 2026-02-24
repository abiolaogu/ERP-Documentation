# ERP-Observability Technical Specifications

## 1. System Requirements

### 1.1 Hardware Requirements

| Component | Minimum | Recommended | Enterprise |
|-----------|---------|-------------|-----------|
| CPU (total cluster) | 16 cores | 64 cores | 128+ cores |
| RAM (total cluster) | 32 GB | 128 GB | 256+ GB |
| Storage (metrics) | 100 GB NVMe | 1 TB NVMe | 10 TB+ NVMe |
| Storage (logs) | 500 GB SSD | 5 TB SSD | 100 TB+ (RustFS) |
| Storage (config DB) | 20 GB SSD | 100 GB SSD | 500 GB SSD |
| Network | 1 Gbps | 10 Gbps | 25 Gbps |

### 1.2 Software Requirements

| Software | Version | Required |
|----------|---------|----------|
| Go | 1.21+ | Yes (gateway, tenant-api) |
| Node.js | 18+ | Yes (observability-api) |
| VictoriaMetrics | latest | Yes (metrics store) |
| Quickwit | latest | Yes (log/trace store) |
| Grafana | latest | Yes (visualization) |
| OTel Collector | latest | Yes (telemetry pipeline) |
| Alertmanager | latest | Yes (alert routing) |
| Zabbix | latest | Yes (infrastructure monitoring) |
| OpenNMS | latest | Yes (event correlation) |
| DragonflyDB | latest | Yes (caching) |
| YugabyteDB | latest | Yes (config/state storage) |
| RustFS | latest | Yes (S3 object storage) |
| Docker | 24+ | Recommended |
| Kubernetes | 1.28+ | Production |

## 2. API Specifications

### 2.1 Base URLs

```
Production:  https://observe.{domain}/api/v1
Development: http://localhost:8090/api/v1
```

### 2.2 Authentication

All business endpoints require JWT authentication via ERP-IAM:

```
Authorization: Bearer <jwt_token>
X-Tenant-ID: <tenant_id>
```

The gateway automatically injects `X-Scope-OrgID` from the tenant ID.

### 2.3 Common Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | Bearer JWT token |
| `X-Tenant-ID` | Yes | Tenant identifier for data scoping |
| `X-Scope-OrgID` | Auto-injected | VictoriaMetrics tenant scope (set by gateway) |
| `Content-Type` | Yes (POST/PUT) | `application/json` |
| `Accept` | Optional | `application/json` (default) |

### 2.4 Metric API Endpoints

#### Instant Query

```
GET /api/v1/metrics/query
POST /api/v1/metrics/query

Parameters:
  query    (required)  PromQL expression
  time     (optional)  Evaluation timestamp (RFC3339 or Unix)
  timeout  (optional)  Query timeout (default: 30s)
```

Response:
```json
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {"__name__": "erp_http_requests_total", "module": "erp-crm"},
        "value": [1708786800, "42.5"]
      }
    ]
  }
}
```

#### Range Query

```
GET /api/v1/metrics/query_range
POST /api/v1/metrics/query_range

Parameters:
  query    (required)  PromQL expression
  start    (required)  Start timestamp
  end      (required)  End timestamp
  step     (required)  Query resolution step (e.g., 15s, 1m, 5m)
  timeout  (optional)  Query timeout (default: 30s)
```

#### Series Metadata

```
GET /api/v1/metrics/series
Parameters: match[] (required, repeated), start, end

GET /api/v1/metrics/labels
Parameters: start, end

GET /api/v1/metrics/label/{name}/values
Parameters: start, end
```

### 2.5 Log API Endpoints

#### Log Search

```
POST /api/v1/logs/search

Body:
{
  "query": "service_name:erp-crm AND severity:ERROR",
  "start_timestamp": "2026-02-24T00:00:00Z",
  "end_timestamp": "2026-02-24T23:59:59Z",
  "max_hits": 100,
  "sort_by_field": "timestamp",
  "sort_order": "Desc",
  "aggregations": {
    "severity_count": {
      "terms": {"field": "severity_text"}
    },
    "volume_over_time": {
      "date_histogram": {"field": "timestamp", "fixed_interval": "1h"}
    }
  }
}
```

Response:
```json
{
  "num_hits": 1523,
  "hits": [
    {
      "timestamp": "2026-02-24T14:32:15Z",
      "service_name": "erp-crm",
      "severity_text": "ERROR",
      "body": "database connection pool exhausted",
      "trace_id": "abc123def456",
      "span_id": "789ghi",
      "resource": {"service.version": "1.0.0", "k8s.pod.name": "erp-crm-7d4f8b-x2k9s"}
    }
  ],
  "aggregations": {
    "severity_count": {"buckets": [{"key": "ERROR", "doc_count": 1523}]},
    "volume_over_time": {"buckets": [{"key": "2026-02-24T14:00:00Z", "doc_count": 892}]}
  }
}
```

#### Real-Time Log Tailing

```
WebSocket /api/v1/logs/tail

Connection:
  ws://observe.{domain}/api/v1/logs/tail?query=severity:ERROR&service=erp-crm

Messages (server -> client):
{
  "type": "log",
  "data": {
    "timestamp": "2026-02-24T14:32:15.123Z",
    "service_name": "erp-crm",
    "severity_text": "ERROR",
    "body": "..."
  }
}
```

### 2.6 Trace API Endpoints

#### Trace Search

```
POST /api/v1/traces/search

Body:
{
  "service_name": "erp-crm",
  "operation_name": "HTTP GET /api/v1/contacts",
  "min_duration_ms": 500,
  "max_duration_ms": null,
  "status": "ERROR",
  "start_timestamp": "2026-02-24T00:00:00Z",
  "end_timestamp": "2026-02-24T23:59:59Z",
  "limit": 50,
  "tags": {"http.method": "GET"}
}
```

#### Get Trace by ID

```
GET /api/v1/traces/{traceId}

Response:
{
  "traceId": "abc123def456",
  "spans": [
    {
      "spanId": "root-span-1",
      "parentSpanId": null,
      "serviceName": "api-gateway",
      "operationName": "HTTP GET /api/v1/contacts",
      "startTime": 1708786800000000,
      "duration": 1200000,
      "status": "OK",
      "attributes": {"http.method": "GET", "http.status_code": 200},
      "events": [],
      "children": ["child-span-1", "child-span-2"]
    },
    {
      "spanId": "child-span-1",
      "parentSpanId": "root-span-1",
      "serviceName": "erp-crm",
      "operationName": "crm.list_contacts",
      "startTime": 1708786800050000,
      "duration": 1100000,
      "status": "OK",
      "attributes": {},
      "children": ["child-span-3"]
    }
  ]
}
```

### 2.7 Alert API Endpoints

```
GET    /api/v1/alerts/rules               List alert rules
POST   /api/v1/alerts/rules               Create alert rule
PUT    /api/v1/alerts/rules/:id           Update alert rule
DELETE /api/v1/alerts/rules/:id           Delete alert rule
GET    /api/v1/alerts/active              List active (firing) alerts
GET    /api/v1/alerts/history             Alert state change history
GET    /api/v1/alerts/silences            List silences
POST   /api/v1/alerts/silences            Create silence
DELETE /api/v1/alerts/silences/:id        Delete silence
```

### 2.8 Tenant API Endpoints

```
GET    /api/v1/tenants                    List tenants
POST   /api/v1/tenants                    Create tenant
GET    /api/v1/tenants/:id                Get tenant
PUT    /api/v1/tenants/:id                Update tenant
DELETE /api/v1/tenants/:id                Delete tenant
POST   /api/v1/tenants/:id/provision      Provision observability stack
GET    /api/v1/tenants/:id/usage          Get usage metrics
GET    /api/v1/tenants/:id/config         Get configuration
PUT    /api/v1/tenants/:id/config         Update configuration
```

### 2.9 Infrastructure API Endpoints

```
GET    /api/v1/infrastructure/hosts       List Zabbix hosts
GET    /api/v1/infrastructure/hosts/:id   Get host details
GET    /api/v1/infrastructure/triggers    List active triggers
GET    /api/v1/events                     List OpenNMS events
POST   /api/v1/events/:id/acknowledge     Acknowledge event
POST   /api/v1/events/correlate           Cross-module event correlation
GET    /api/v1/infrastructure/topology    Get network topology
```

### 2.10 Health Endpoints

```
GET /health     Service health check (no auth required)
GET /ready      Backend connectivity check (no auth required)
```

Health response:
```json
{
  "status": "healthy",
  "service": "opensase-observability-gateway",
  "version": "1.0.0",
  "backends": {
    "victoriametrics": "healthy",
    "quickwit": "healthy",
    "yugabytedb": "healthy",
    "dragonflydb": "healthy",
    "alertmanager": "healthy",
    "grafana": "healthy"
  }
}
```

## 3. Performance Specifications

### 3.1 Ingestion Throughput

| Data Type | Target Throughput | Measurement |
|-----------|------------------|-------------|
| Metrics | 1,000,000 data points/sec | VictoriaMetrics vminsert benchmark |
| Logs | 500,000 log lines/sec | Quickwit indexer benchmark |
| Traces | 100,000 spans/sec | Quickwit indexer benchmark |
| Infrastructure metrics | 50,000 items/sec | Zabbix server capacity |

### 3.2 Query Latency Targets

| Query Type | p50 | p95 | p99 |
|-----------|-----|-----|-----|
| PromQL instant (1h range) | 10ms | 50ms | 100ms |
| PromQL range (24h, 15s step) | 50ms | 200ms | 500ms |
| PromQL range (7d, 1m step) | 100ms | 500ms | 2000ms |
| Log keyword search (100TB) | 50ms | 100ms | 200ms |
| Log structured search | 20ms | 80ms | 150ms |
| Trace by ID | 5ms | 20ms | 50ms |
| Trace search (filtered) | 50ms | 150ms | 300ms |
| Dashboard load (20 panels) | 500ms | 1500ms | 2000ms |
| Alert rule evaluation | N/A | N/A | < 10s interval |

### 3.3 Storage Specifications

| Data Type | Compression Ratio | Storage Per 1M Series/Day | Retention Options |
|-----------|------------------|--------------------------|-------------------|
| Metrics (VictoriaMetrics) | 10x vs. Prometheus | ~1 GB/day per 1M active series | 7d, 30d, 90d, 1y, 5y |
| Logs (Quickwit) | 5-10x (columnar) | ~100 GB/day per 1TB raw logs | 30d, 90d, 180d, 1y, 2y |
| Traces (Quickwit) | 5-10x (columnar) | ~50 GB/day per 100K spans/sec | 3d, 7d, 14d, 30d |
| Config (YugabyteDB) | Standard SQL | Negligible | Indefinite |
| Cache (DragonflyDB) | None (in-memory) | 1-4 GB typical | Ephemeral (TTL-based) |

### 3.4 Capacity Limits

| Resource | Limit |
|----------|-------|
| Active time series per tenant | 10,000,000 |
| Log ingestion rate per tenant | 100,000 lines/sec |
| Trace ingestion rate per tenant | 50,000 spans/sec |
| PromQL query duration max | 30 seconds |
| PromQL concurrent queries per tenant | 16 |
| Log search max hits | 10,000 |
| Log tail WebSocket max rate | 1,000 lines/sec per connection |
| Dashboard panels max | 50 per dashboard |
| Alert rules per tenant | 10,000 |
| Silences per tenant | 1,000 |
| Notification channels per tenant | 50 |
| Tenants per cluster | 1,000 |

## 4. Data Format Specifications

### 4.1 Metric Format

Metrics follow the Prometheus exposition format:
```
# TYPE erp_http_requests_total counter
# HELP erp_http_requests_total Total HTTP requests
erp_http_requests_total{module="erp-crm",method="GET",status="200",endpoint="/api/v1/contacts"} 42567

# TYPE erp_http_request_duration_seconds histogram
erp_http_request_duration_seconds_bucket{module="erp-crm",le="0.005"} 24054
erp_http_request_duration_seconds_bucket{module="erp-crm",le="0.01"} 33444
erp_http_request_duration_seconds_bucket{module="erp-crm",le="0.025"} 100392
erp_http_request_duration_seconds_bucket{module="erp-crm",le="+Inf"} 144320
erp_http_request_duration_seconds_sum{module="erp-crm"} 53423.21
erp_http_request_duration_seconds_count{module="erp-crm"} 144320
```

### 4.2 Log Format (OTLP)

```json
{
  "resourceLogs": [{
    "resource": {
      "attributes": [
        {"key": "service.name", "value": {"stringValue": "erp-crm"}},
        {"key": "service.version", "value": {"stringValue": "1.0.0"}},
        {"key": "deployment.environment", "value": {"stringValue": "production"}},
        {"key": "tenant_id", "value": {"stringValue": "acme-corp"}}
      ]
    },
    "scopeLogs": [{
      "logRecords": [{
        "timeUnixNano": "1708786800000000000",
        "severityNumber": 17,
        "severityText": "ERROR",
        "body": {"stringValue": "database connection pool exhausted"},
        "attributes": [
          {"key": "db.system", "value": {"stringValue": "yugabytedb"}},
          {"key": "db.statement", "value": {"stringValue": "SELECT * FROM contacts WHERE..."}}
        ],
        "traceId": "abc123def456",
        "spanId": "789ghi"
      }]
    }]
  }]
}
```

### 4.3 Trace Format (OTLP)

```json
{
  "resourceSpans": [{
    "resource": {
      "attributes": [
        {"key": "service.name", "value": {"stringValue": "erp-crm"}},
        {"key": "tenant_id", "value": {"stringValue": "acme-corp"}}
      ]
    },
    "scopeSpans": [{
      "spans": [{
        "traceId": "abc123def456",
        "spanId": "span-001",
        "parentSpanId": "",
        "name": "HTTP GET /api/v1/contacts",
        "kind": 2,
        "startTimeUnixNano": "1708786800000000000",
        "endTimeUnixNano": "1708786801200000000",
        "status": {"code": 1, "message": "OK"},
        "attributes": [
          {"key": "http.method", "value": {"stringValue": "GET"}},
          {"key": "http.status_code", "value": {"intValue": 200}},
          {"key": "http.url", "value": {"stringValue": "/api/v1/contacts"}}
        ]
      }]
    }]
  }]
}
```

## 5. Protocol Specifications

| Protocol | Usage | Port | TLS |
|----------|-------|------|-----|
| OTLP gRPC | Telemetry ingestion | 4317 | Optional (internal) |
| OTLP HTTP | Telemetry ingestion | 4318 | Optional (internal) |
| Prometheus Remote Write | Metric export to VM | 8480 | No (internal) |
| PromQL HTTP | Metric queries | 8481 | No (internal) |
| Quickwit REST | Log/trace queries | 7280 | No (internal) |
| Quickwit OTLP | Log/trace ingestion | 7281 | No (internal) |
| Alertmanager HTTP | Alert management | 9093 | No (internal) |
| Grafana HTTP | Dashboard API | 3000 | No (internal) |
| Zabbix JSON-RPC | Infrastructure API | 80/443 | Yes (HTTPS) |
| OpenNMS REST | Event API | 8980 | Yes (HTTPS) |
| Redis Protocol | DragonflyDB cache | 6379 | No (internal) |
| PostgreSQL Wire | YugabyteDB queries | 5433 | Optional |
| S3 HTTP | RustFS object storage | 9000 | Optional |
| WebSocket | Log tailing | 8090 | Yes (WSS) |
| SMTP | Email notifications | 587 | STARTTLS |

## 6. Security Specifications

### 6.1 Encryption

| Layer | Standard |
|-------|----------|
| In transit (external) | TLS 1.3 |
| In transit (internal) | mTLS (optional) or plaintext |
| At rest (YugabyteDB) | AES-256 |
| At rest (RustFS) | AES-256 server-side encryption |
| JWT signing | RS256 or ES256 (via ERP-IAM) |
| DragonflyDB | Unencrypted (internal network only) |

### 6.2 Rate Limiting

| Endpoint Group | Rate Limit |
|---------------|-----------|
| PromQL queries | 100 req/min per tenant |
| Log searches | 60 req/min per tenant |
| Trace searches | 60 req/min per tenant |
| Alert rule CRUD | 30 req/min per tenant |
| Tenant CRUD | 10 req/min per user |
| Health/Ready | Unlimited |
| WebSocket (log tail) | 5 concurrent connections per tenant |

### 6.3 Input Validation

| Field | Validation |
|-------|-----------|
| PromQL expression | Syntax validation, max length 10KB |
| Quickwit query | Syntax validation, max length 10KB |
| Tenant ID | Alphanumeric + hyphens, 3-63 characters |
| Alert rule name | 1-255 characters |
| Silence duration | 1 minute to 7 days |
| Retention period | 1 day to 5 years |
| Time range | Max 1 year for range queries |
| Query step | Min 1s, max query points: 11,000 |

## 7. Observability Specifications (Self-Monitoring)

### 7.1 Internal Metrics

```
# Gateway metrics
observability_gateway_requests_total{method, path, status}
observability_gateway_request_duration_seconds{method, path}
observability_gateway_active_connections

# VictoriaMetrics metrics
vm_rows_inserted_total
vm_active_timeseries
vm_slow_queries_total
vm_storage_used_bytes

# Quickwit metrics
quickwit_indexing_docs_total
quickwit_search_queries_total
quickwit_storage_bytes

# OTel Collector metrics
otelcol_receiver_accepted_spans
otelcol_receiver_refused_spans
otelcol_exporter_sent_metric_points
otelcol_processor_batch_batch_send_size
```

### 7.2 Health Check Responses

Gateway health:
```json
{
  "status": "healthy",
  "service": "opensase-observability-gateway",
  "version": "1.0.0"
}
```

Readiness check:
```json
{
  "status": "ready",
  "backends": {
    "victoriametrics": {"status": "healthy", "latency_ms": 2},
    "quickwit": {"status": "healthy", "latency_ms": 5},
    "yugabytedb": {"status": "healthy", "latency_ms": 3},
    "dragonflydb": {"status": "healthy", "latency_ms": 1},
    "alertmanager": {"status": "healthy", "latency_ms": 4},
    "grafana": {"status": "healthy", "latency_ms": 8}
  }
}
```
