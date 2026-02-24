# ERP-Observability API Documentation

> **Document ID:** ERP-OBS-API-021
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [22-Postman-Collection.md](./22-Postman-Collection.md)

---

## Base URLs

| Environment | Gateway URL | Direct API URL |
|---|---|---|
| Local Development | `http://localhost:8090` | `http://localhost:3000` |
| Staging | `https://observability-staging.erp.io` | Internal only |
| Production | `https://observability.erp.io` | Internal only |

All requests should go through the **Gateway** (port 8090), which proxies to the backend services with CORS handling and security headers.

---

## Authentication

### Development Mode

In development (`NODE_ENV=development`), authentication is optional. Unauthenticated requests receive a default identity:

```
user_id: dev-user
tenant_id: dev-tenant (or X-Tenant-ID header value)
roles: [admin]
```

### Production Mode

All requests must include a valid JWT token issued by Authentik:

```
Authorization: Bearer <jwt-token>
```

JWT claims must include:

```json
{
  "sub": "user-uuid",
  "https://erp.io/jwt/claims": {
    "tenant_id": "tenant-uuid",
    "roles": ["admin"],
    "user_id": "user-uuid"
  }
}
```

### Tenant Context

All data operations are scoped to the tenant. The tenant is resolved from:
1. JWT claim `https://erp.io/jwt/claims.tenant_id` (highest priority)
2. `X-Tenant-ID` request header (fallback)

System-level roles (`platform_admin`, `super_admin`, `system`) can query across tenants by omitting the tenant context.

---

## Common Headers

| Header | Required | Description |
|---|---|---|
| `Authorization` | Yes (prod) | `Bearer <jwt-token>` |
| `X-Tenant-ID` | Conditional | Tenant identifier (if not in JWT) |
| `X-Request-ID` | No | Request correlation ID (auto-generated if absent) |
| `Content-Type` | Yes (POST/PUT) | `application/json` |

---

## Common Response Format

### Success Response

```json
{
  "data": [...],
  "total": 42
}
```

### Error Response

```json
{
  "error": "error_code",
  "detail": "Human-readable description of the error",
  "request_id": "req-1234567890-1"
}
```

### HTTP Status Codes

| Code | Meaning |
|---|---|
| 200 | Success |
| 201 | Created |
| 202 | Accepted (async operation) |
| 204 | No Content (successful delete) |
| 400 | Validation error |
| 401 | Unauthorized (missing/invalid token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not found |
| 500 | Internal server error |
| 502 | Bad gateway (upstream unreachable) |
| 503 | Service unavailable |

---

## 1. Gateway Endpoints

### GET /healthz

Gateway health check.

**Request:**
```
GET /healthz
```

**Response (200):**
```json
{
  "status": "healthy",
  "module": "ERP-Observability"
}
```

### GET /v1/capabilities

Module capability declaration.

**Request:**
```
GET /v1/capabilities
```

**Response (200):**
```json
{
  "module": "ERP-Observability",
  "version": "1.0.0",
  "capabilities": [
    "log_aggregation",
    "metric_collection",
    "distributed_tracing",
    "full_text_search",
    "cross_module_search",
    "alerting",
    "dashboards",
    "audit_trail",
    "slo_monitoring",
    "anomaly_detection"
  ],
  "integration_mode": "platform_service",
  "aidd_governance": "ERP_AIDD_STRICT"
}
```

---

## 2. Health Endpoint

### GET /v1/observability/health

Returns the health status of all observability components.

**Request:**
```
GET /v1/observability/health
```

**Response (200 - healthy):**
```json
{
  "status": "healthy",
  "components": {
    "quickwit": { "status": "up", "latency_ms": 5 },
    "prometheus": { "status": "up", "latency_ms": 3 },
    "grafana": { "status": "up", "latency_ms": 8 },
    "dragonfly": { "status": "up", "latency_ms": 1 },
    "yugabytedb": { "status": "up", "latency_ms": 12 },
    "otel_collector": { "status": "up", "latency_ms": 2 },
    "alertmanager": { "status": "up", "latency_ms": 4 }
  },
  "timestamp": "2026-02-24T10:00:00.000Z"
}
```

**Response (503 - degraded):**
```json
{
  "status": "degraded",
  "components": {
    "quickwit": { "status": "up", "latency_ms": 5 },
    "prometheus": { "status": "down", "latency_ms": 3000, "details": "Connection refused" },
    "grafana": { "status": "up", "latency_ms": 8 },
    "dragonfly": { "status": "up", "latency_ms": 1 },
    "yugabytedb": { "status": "up", "latency_ms": 12 },
    "otel_collector": { "status": "up", "latency_ms": 2 },
    "alertmanager": { "status": "up", "latency_ms": 4 }
  },
  "timestamp": "2026-02-24T10:00:00.000Z"
}
```

---

## 3. Logs API

### GET /v1/observability/logs

Query logs from the Quickwit `erp-logs` index.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `module` | string | No | -- | Filter by ERP module (e.g., `finance`, `iam`, `platform`) |
| `level` | string | No | -- | Filter by log level: `trace`, `debug`, `info`, `warn`, `error`, `fatal` |
| `tenant_id` | string | No | From JWT | Override tenant (system users only) |
| `start` | string | No | -- | Start timestamp (RFC 3339) |
| `end` | string | No | -- | End timestamp (RFC 3339) |
| `q` | string | No | -- | Free-text search query (max 2000 chars) |
| `limit` | number | No | 100 | Results per page (1-1000) |
| `offset` | number | No | 0 | Pagination offset |
| `sort_order` | string | No | `desc` | Sort by timestamp: `asc` or `desc` |

**Request:**
```
GET /v1/observability/logs?module=finance&level=error&limit=10&start=2026-02-24T00:00:00Z
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "data": [
    {
      "timestamp": "2026-02-24T10:15:30.123Z",
      "level": "error",
      "message": "Failed to process payment: insufficient funds",
      "module": "finance",
      "service": "payment-service",
      "tenant_id": "tenant-123",
      "request_id": "req-abc-123",
      "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
      "span_id": "00f067aa0ba902b7",
      "host": "finance-api-7b8c4d5f6-x2k3m",
      "container": "finance-api",
      "namespace": "erp-finance",
      "metadata": {
        "payment_id": "pay-456",
        "amount": 1500.00,
        "currency": "USD"
      }
    }
  ],
  "total": 1,
  "limit": 10,
  "offset": 0
}
```

### GET /v1/observability/logs/stream

Server-Sent Events (SSE) stream for real-time log tailing. Polls Quickwit every 2 seconds.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `module` | string | No | -- | Filter by module |
| `level` | string | No | -- | Filter by log level |

**Request:**
```
GET /v1/observability/logs/stream?module=finance&level=error
X-Tenant-ID: tenant-123
Accept: text/event-stream
```

**Response (200 - SSE stream):**
```
data: {"type":"connected","timestamp":"2026-02-24T10:00:00.000Z"}

data: {"timestamp":"2026-02-24T10:15:30.123Z","level":"error","message":"...","module":"finance",...}

data: {"timestamp":"2026-02-24T10:15:32.456Z","level":"error","message":"...","module":"finance",...}

: heartbeat

```

### GET /v1/observability/logs/stats

Log volume statistics aggregated by level, module, and time.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `start` | string | No | 24 hours ago | Start timestamp (RFC 3339) |
| `end` | string | No | Now | End timestamp (RFC 3339) |

**Request:**
```
GET /v1/observability/logs/stats?start=2026-02-23T00:00:00Z
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "total_count": 125000,
  "by_level": {
    "trace": 5000,
    "debug": 45000,
    "info": 60000,
    "warn": 10000,
    "error": 4500,
    "fatal": 500
  },
  "by_module": {
    "finance": 25000,
    "iam": 15000,
    "platform": 30000
  },
  "time_range": {
    "start": "2026-02-23T00:00:00Z",
    "end": "2026-02-24T10:00:00Z"
  }
}
```

---

## 4. Metrics API

### GET /v1/observability/metrics/query

Instant PromQL query proxied to Prometheus/VictoriaMetrics.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `query` | string | Yes | -- | PromQL expression (max 5000 chars) |
| `time` | string | No | Now | Evaluation timestamp (RFC 3339 or Unix) |
| `timeout` | string | No | -- | Query timeout (e.g., `30s`) |

**Request:**
```
GET /v1/observability/metrics/query?query=up{module="finance"}
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "__name__": "up",
          "module": "finance",
          "instance": "erp-finance-gateway.erp-finance.svc.cluster.local:8090",
          "job": "erp-modules"
        },
        "value": [1708768800, "1"]
      }
    ]
  }
}
```

### GET /v1/observability/metrics/range

Range PromQL query for time-series data.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `query` | string | Yes | -- | PromQL expression |
| `start` | string | Yes | -- | Range start (RFC 3339 or Unix) |
| `end` | string | Yes | -- | Range end (RFC 3339 or Unix) |
| `step` | string | No | `15s` | Query resolution step |
| `timeout` | string | No | -- | Query timeout |

**Request:**
```
GET /v1/observability/metrics/range?query=rate(http_requests_total{module="finance"}[5m])&start=2026-02-24T09:00:00Z&end=2026-02-24T10:00:00Z&step=60s
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "status": "success",
  "data": {
    "resultType": "matrix",
    "result": [
      {
        "metric": {
          "module": "finance",
          "method": "GET",
          "path": "/v1/invoices"
        },
        "values": [
          [1708765200, "12.5"],
          [1708765260, "13.2"],
          [1708765320, "11.8"]
        ]
      }
    ]
  }
}
```

### GET /v1/observability/metrics/modules

Per-module metric summary across all ERP modules.

**Request:**
```
GET /v1/observability/metrics/modules
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "modules": [
    {
      "module": "finance",
      "status": "up",
      "request_rate": 125.5,
      "error_rate": 0.02,
      "p99_latency_ms": 245,
      "uptime_percent": 99.98
    },
    {
      "module": "iam",
      "status": "up",
      "request_rate": 340.2,
      "error_rate": 0.01,
      "p99_latency_ms": 120,
      "uptime_percent": 99.99
    }
  ]
}
```

---

## 5. Traces API

### GET /v1/observability/traces

Query distributed traces from the Quickwit `erp-traces` index.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `service` | string | No | -- | Filter by service name |
| `module` | string | No | -- | Filter by ERP module |
| `tenant_id` | string | No | From JWT | Override tenant (system users only) |
| `min_duration_ms` | number | No | -- | Minimum trace duration |
| `max_duration_ms` | number | No | -- | Maximum trace duration |
| `start` | string | No | -- | Start timestamp (RFC 3339) |
| `end` | string | No | -- | End timestamp (RFC 3339) |
| `limit` | number | No | 20 | Results per page (1-200) |
| `offset` | number | No | 0 | Pagination offset |
| `has_errors` | string | No | -- | Filter: `true` for error traces, `false` for success |

**Request:**
```
GET /v1/observability/traces?module=finance&has_errors=true&limit=5
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "data": [
    {
      "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
      "spans": [],
      "duration_ms": 1245,
      "service_count": 3,
      "span_count": 12,
      "has_errors": true,
      "root_service": "finance-gateway",
      "root_operation": "POST /v1/invoices"
    }
  ],
  "total": 1,
  "limit": 5,
  "offset": 0
}
```

### GET /v1/observability/traces/:traceId

Get complete trace with all spans.

**Path Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `traceId` | string | Trace ID (min 8 chars) |

**Request:**
```
GET /v1/observability/traces/4bf92f3577b34da6a3ce929d0e0e4736
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "spans": [
    {
      "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
      "span_id": "00f067aa0ba902b7",
      "parent_span_id": null,
      "operation_name": "POST /v1/invoices",
      "service_name": "finance-gateway",
      "module": "finance",
      "tenant_id": "tenant-123",
      "start_timestamp": "2026-02-24T10:15:30.000Z",
      "duration_ms": 1245,
      "status_code": "ERROR",
      "attributes": {
        "http.method": "POST",
        "http.url": "/v1/invoices",
        "http.status_code": 500
      },
      "events": [
        {
          "name": "exception",
          "timestamp": "2026-02-24T10:15:31.200Z",
          "attributes": {
            "exception.type": "InsufficientFundsError",
            "exception.message": "Account balance insufficient"
          }
        }
      ]
    }
  ],
  "duration_ms": 1245,
  "service_count": 3,
  "span_count": 12,
  "has_errors": true,
  "root_service": "finance-gateway",
  "root_operation": "POST /v1/invoices"
}
```

### GET /v1/observability/traces/:traceId/timeline

Flattened timeline view for waterfall visualization.

**Request:**
```
GET /v1/observability/traces/4bf92f3577b34da6a3ce929d0e0e4736/timeline
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "total_duration_ms": 1245,
  "spans": [
    {
      "span_id": "00f067aa0ba902b7",
      "parent_span_id": null,
      "operation_name": "POST /v1/invoices",
      "service_name": "finance-gateway",
      "start_offset_ms": 0,
      "duration_ms": 1245,
      "status_code": "ERROR",
      "depth": 0
    },
    {
      "span_id": "11a067bb1cb903c8",
      "parent_span_id": "00f067aa0ba902b7",
      "operation_name": "validateInvoice",
      "service_name": "finance-api",
      "start_offset_ms": 5,
      "duration_ms": 120,
      "status_code": "OK",
      "depth": 1
    }
  ]
}
```

---

## 6. Alerts API

### GET /v1/observability/alerts

List active alerts from Alertmanager.

**Request:**
```
GET /v1/observability/alerts
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "data": [
    {
      "fingerprint": "abc123def456",
      "status": "firing",
      "labels": {
        "alertname": "ERPHighErrorRate",
        "module": "finance",
        "severity": "warning"
      },
      "annotations": {
        "summary": "High error rate detected for finance module",
        "description": "Error rate is 5.2% (threshold: 1%)",
        "runbook_url": "https://runbooks.erp.io/finance-high-error-rate"
      },
      "starts_at": "2026-02-24T09:45:00.000Z",
      "generator_url": "http://prometheus:9090/graph?g0.expr=..."
    }
  ],
  "total": 1
}
```

### GET /v1/observability/alerts/rules

List configured alert rules.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `module` | string | No | -- | Filter by module |
| `enabled` | string | No | -- | Filter: `true` or `false` |

**Request:**
```
GET /v1/observability/alerts/rules?module=finance&enabled=true
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "tenant_id": "tenant-123",
      "name": "Finance High Error Rate",
      "description": "Alert when finance module error rate exceeds 1%",
      "severity": "warning",
      "module": "finance",
      "promql_expression": "rate(http_requests_total{module=\"finance\",status=~\"5..\"}[5m]) / rate(http_requests_total{module=\"finance\"}[5m]) > 0.01",
      "duration": "5m",
      "labels": { "team": "finance-eng" },
      "annotations": { "runbook_url": "https://runbooks.erp.io/finance-error-rate" },
      "enabled": true,
      "created_at": "2026-02-20T10:00:00.000Z",
      "updated_at": "2026-02-20T10:00:00.000Z"
    }
  ],
  "total": 1
}
```

### POST /v1/observability/alerts/rules

Create a new alert rule. **Requires `admin` role.**

**Request Body:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | -- | Alert name (1-200 chars) |
| `description` | string | No | -- | Description (max 2000 chars) |
| `severity` | string | No | `warning` | `info`, `warning`, or `critical` |
| `module` | string | No | -- | Target ERP module |
| `promql_expression` | string | Yes | -- | PromQL expression (1-5000 chars) |
| `duration` | string | No | `5m` | How long condition must be true |
| `labels` | object | No | `{}` | Additional labels |
| `annotations` | object | No | `{}` | Annotations (summary, runbook_url) |
| `enabled` | boolean | No | `true` | Whether the rule is active |

**Request:**
```
POST /v1/observability/alerts/rules
Content-Type: application/json
X-Tenant-ID: tenant-123

{
  "name": "Finance High Latency",
  "description": "p99 latency exceeds 500ms for finance module",
  "severity": "warning",
  "module": "finance",
  "promql_expression": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{module=\"finance\"}[5m])) > 0.5",
  "duration": "10m",
  "labels": { "team": "finance-eng", "category": "slo" },
  "annotations": {
    "summary": "Finance module p99 latency high",
    "runbook_url": "https://runbooks.erp.io/finance-latency"
  }
}
```

**Response (201):**
```json
{
  "id": "660e8400-e29b-41d4-a716-446655440001",
  "tenant_id": "tenant-123",
  "name": "Finance High Latency",
  "description": "p99 latency exceeds 500ms for finance module",
  "severity": "warning",
  "module": "finance",
  "promql_expression": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{module=\"finance\"}[5m])) > 0.5",
  "duration": "10m",
  "labels": { "team": "finance-eng", "category": "slo" },
  "annotations": {
    "summary": "Finance module p99 latency high",
    "runbook_url": "https://runbooks.erp.io/finance-latency"
  },
  "enabled": true,
  "created_at": "2026-02-24T10:30:00.000Z",
  "updated_at": "2026-02-24T10:30:00.000Z"
}
```

### PUT /v1/observability/alerts/rules/:id

Update an existing alert rule (partial update). **Requires `admin` role.**

**Request:**
```
PUT /v1/observability/alerts/rules/660e8400-e29b-41d4-a716-446655440001
Content-Type: application/json
X-Tenant-ID: tenant-123

{
  "severity": "critical",
  "duration": "5m"
}
```

**Response (200):** Updated alert rule object.

### DELETE /v1/observability/alerts/rules/:id

Delete an alert rule. **Requires `admin` role.**

**Request:**
```
DELETE /v1/observability/alerts/rules/660e8400-e29b-41d4-a716-446655440001
X-Tenant-ID: tenant-123
```

**Response (204):** No content.

### POST /v1/observability/alerts/:id/silence

Silence an alert for a specified duration. **Requires `admin` role.**

**Request Body:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `reason` | string | Yes | -- | Reason for silencing (1-1000 chars) |
| `duration_hours` | number | No | 4 | Silence duration in hours (0.5-720) |

**Request:**
```
POST /v1/observability/alerts/660e8400-e29b-41d4-a716-446655440001/silence
Content-Type: application/json
X-Tenant-ID: tenant-123

{
  "reason": "Known issue, fix deploying in next release",
  "duration_hours": 8
}
```

**Response (201):**
```json
{
  "id": "770e8400-e29b-41d4-a716-446655440002",
  "alert_rule_id": "660e8400-e29b-41d4-a716-446655440001",
  "tenant_id": "tenant-123",
  "reason": "Known issue, fix deploying in next release",
  "created_by": "user-123",
  "starts_at": "2026-02-24T10:30:00.000Z",
  "ends_at": "2026-02-24T18:30:00.000Z",
  "created_at": "2026-02-24T10:30:00.000Z"
}
```

---

## 7. Dashboards API

### GET /v1/observability/dashboards

List dashboards for the current tenant.

**Request:**
```
GET /v1/observability/dashboards
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "data": [
    {
      "id": "880e8400-e29b-41d4-a716-446655440003",
      "tenant_id": "tenant-123",
      "name": "Finance Overview",
      "description": "Key finance module metrics",
      "tags": ["finance", "overview"],
      "created_by": "user-123",
      "created_at": "2026-02-20T10:00:00.000Z",
      "updated_at": "2026-02-24T09:00:00.000Z",
      "panel_count": 6
    }
  ],
  "total": 1
}
```

### GET /v1/observability/dashboards/:id

Get full dashboard with panels.

**Response (200):**
```json
{
  "id": "880e8400-e29b-41d4-a716-446655440003",
  "tenant_id": "tenant-123",
  "name": "Finance Overview",
  "description": "Key finance module metrics",
  "panels": [
    {
      "id": "panel-1",
      "title": "Request Rate",
      "type": "timeseries",
      "query": "rate(http_requests_total{module=\"finance\"}[5m])",
      "datasource": "prometheus",
      "position": { "x": 0, "y": 0, "w": 12, "h": 8 },
      "options": { "legend": { "show": true } }
    },
    {
      "id": "panel-2",
      "title": "Error Count",
      "type": "stat",
      "query": "sum(increase(http_requests_total{module=\"finance\",status=~\"5..\"}[1h]))",
      "datasource": "prometheus",
      "position": { "x": 12, "y": 0, "w": 6, "h": 4 },
      "options": { "colorMode": "value" }
    }
  ],
  "tags": ["finance", "overview"],
  "created_by": "user-123",
  "created_at": "2026-02-20T10:00:00.000Z",
  "updated_at": "2026-02-24T09:00:00.000Z"
}
```

### POST /v1/observability/dashboards

Create a new dashboard.

**Request Body:**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `name` | string | Yes | -- | Dashboard name (1-200 chars) |
| `description` | string | No | -- | Description (max 2000 chars) |
| `panels` | array | No | `[]` | Panel definitions |
| `tags` | array | No | `[]` | String tags |

**Panel Schema:**

| Field | Type | Required | Description |
|---|---|---|---|
| `id` | string | Yes | Unique panel ID within dashboard |
| `title` | string | Yes | Panel title (1-200 chars) |
| `type` | string | Yes | `timeseries`, `stat`, `gauge`, `table`, `bar`, `pie` |
| `query` | string | Yes | PromQL or Quickwit query (1-5000 chars) |
| `datasource` | string | Yes | `prometheus` or `quickwit` |
| `position` | object | Yes | `{ x, y, w, h }` grid position |
| `options` | object | No | Panel-specific options |

### PUT /v1/observability/dashboards/:id

Update a dashboard (partial update).

---

## 8. Search API

### GET /v1/observability/search

Cross-module full-text search across all ERP modules.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `q` | string | Yes | -- | Search query (1-1000 chars) |
| `modules` | string | No | All | Comma-separated module filter |
| `types` | string | No | All | Comma-separated entity type filter |
| `tenant_id` | string | No | From JWT | Override tenant (system only) |
| `limit` | number | No | 20 | Results per page (1-100) |
| `offset` | number | No | 0 | Pagination offset |
| `sort_by` | string | No | `relevance` | `relevance` or `date` |

**Request:**
```
GET /v1/observability/search?q=invoice+payment+failed&modules=finance,commerce&limit=10
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "results": [
    {
      "id": "search-result-1",
      "module": "finance",
      "entity_type": "invoice",
      "entity_id": "inv-789",
      "title": "Invoice #INV-2026-0789",
      "content": "Payment failed for invoice INV-2026-0789. Retry scheduled.",
      "score": 0.95,
      "highlights": {
        "title": ["<em>Invoice</em> #INV-2026-0789"],
        "content": ["<em>Payment</em> <em>failed</em> for <em>invoice</em> INV-2026-0789"]
      },
      "metadata": { "amount": 2500.00, "currency": "USD" },
      "indexed_at": "2026-02-24T09:00:00.000Z"
    }
  ],
  "total": 1,
  "took_ms": 45,
  "query": "invoice payment failed",
  "modules": ["finance", "commerce"]
}
```

### GET /v1/observability/search/suggest

Autocomplete suggestions.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `q` | string | Yes | -- | Partial query (1-200 chars) |
| `limit` | number | No | 5 | Max suggestions (1-10) |

**Request:**
```
GET /v1/observability/search/suggest?q=inv&limit=5
X-Tenant-ID: tenant-123
```

**Response (200):**
```json
{
  "suggestions": [
    "invoice",
    "inventory",
    "investment",
    "invitation",
    "invalid"
  ]
}
```

### POST /v1/observability/search/reindex

Trigger reindex for a module/entity type. **Requires `admin` role.**

**Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `module` | string | Yes | ERP module name |
| `entity_type` | string | Yes | Entity type to reindex |
| `entity_id` | string | No | Specific entity ID (omit for full reindex) |

**Request:**
```
POST /v1/observability/search/reindex
Content-Type: application/json
X-Tenant-ID: tenant-123

{
  "module": "finance",
  "entity_type": "invoice",
  "entity_id": "inv-789"
}
```

**Response (202):**
```json
{
  "status": "accepted",
  "message": "Reindex triggered for finance/invoice"
}
```

---

## 9. Tenant API

### POST /api/tenants

Register a new tenant for observability.

### GET /api/tenants/:id

Retrieve tenant configuration.

### PUT /api/tenants/:id/config

Update tenant observability configuration.

### PUT /api/tenants/:id/retention

Configure per-signal retention policies per tenant.

### GET /api/tenants/:id/usage

Retrieve tenant usage metrics.

### DELETE /api/tenants/:id

Soft-delete tenant and schedule data purge.

---

## Rate Limits

| Endpoint Category | Rate Limit | Window |
|---|---|---|
| Read endpoints (GET) | 1000 req/min | Per tenant |
| Write endpoints (POST/PUT/DELETE) | 100 req/min | Per tenant |
| Search endpoints | 200 req/min | Per tenant |
| Stream endpoints (SSE) | 10 concurrent | Per tenant |
| Reindex trigger | 5 req/hour | Per tenant |
