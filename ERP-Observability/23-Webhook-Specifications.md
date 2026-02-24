# ERP-Observability Webhook Specifications

> **Document ID:** ERP-OBS-WH-023
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [21-API-Documentation.md](./21-API-Documentation.md), [24-Runbooks.md](./24-Runbooks.md)

---

## Overview

ERP-Observability emits webhook events for key observability lifecycle changes. External systems (notification services, incident management platforms, ITSM tools, and other ERP modules) can subscribe to these events to trigger automated workflows.

Webhooks are delivered as HTTP POST requests with JSON payloads to registered endpoint URLs. All webhook payloads follow the [CloudEvents v1.0](https://cloudevents.io/) specification.

---

## Webhook Registration

Webhooks are registered via the Tenant API:

```
POST /api/tenants/:tenant_id/webhooks
Content-Type: application/json

{
  "url": "https://your-service.example.com/webhooks/observability",
  "events": ["alert.firing", "alert.resolved", "dashboard.updated"],
  "secret": "whsec_your-shared-secret-for-hmac",
  "enabled": true,
  "description": "PagerDuty integration for critical alerts"
}
```

**Response (201):**
```json
{
  "id": "wh-550e8400-e29b-41d4-a716-446655440000",
  "tenant_id": "tenant-123",
  "url": "https://your-service.example.com/webhooks/observability",
  "events": ["alert.firing", "alert.resolved", "dashboard.updated"],
  "enabled": true,
  "created_at": "2026-02-24T10:00:00.000Z"
}
```

---

## Authentication

### HMAC-SHA256 Signature

Every webhook request includes an `X-ERP-Signature-256` header containing an HMAC-SHA256 signature of the request body using the shared secret:

```
X-ERP-Signature-256: sha256=a1b2c3d4e5f6...
```

**Verification (Node.js example):**

```javascript
const crypto = require('crypto');

function verifyWebhook(body, signature, secret) {
  const expected = 'sha256=' + crypto
    .createHmac('sha256', secret)
    .update(body, 'utf8')
    .digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expected)
  );
}
```

**Verification (Go example):**

```go
func verifyWebhook(body []byte, signature, secret string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(body)
    expected := "sha256=" + hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(signature), []byte(expected))
}
```

### Additional Headers

| Header | Description |
|---|---|
| `X-ERP-Signature-256` | HMAC-SHA256 signature |
| `X-ERP-Webhook-ID` | Unique delivery ID (for idempotency) |
| `X-ERP-Webhook-Timestamp` | Unix timestamp of delivery |
| `X-ERP-Event-Type` | Event type (e.g., `alert.firing`) |
| `Content-Type` | `application/json` |
| `User-Agent` | `ERP-Observability-Webhook/1.0` |

---

## Retry Policy

| Attempt | Delay | Total Elapsed |
|---|---|---|
| 1st delivery | Immediate | 0s |
| 1st retry | 30 seconds | 30s |
| 2nd retry | 2 minutes | 2m 30s |
| 3rd retry | 10 minutes | 12m 30s |
| 4th retry | 30 minutes | 42m 30s |
| 5th retry | 1 hour | 1h 42m 30s |
| 6th retry (final) | 4 hours | 5h 42m 30s |

**Success criteria:** HTTP status code 2xx (200-299).

**Failure criteria:** Any non-2xx response, connection timeout (10 seconds), or DNS resolution failure.

**Circuit breaker:** After 10 consecutive failures to a webhook endpoint, the webhook is automatically disabled. The tenant admin is notified via email and in-app notification. The webhook can be re-enabled via the API after fixing the endpoint.

**Idempotency:** Each delivery includes an `X-ERP-Webhook-ID` header. Receivers should use this ID to deduplicate retried deliveries.

---

## Event Types

### 1. alert.firing

Emitted when an alert transitions from inactive to firing state.

**Trigger:** Alertmanager fires an alert based on a PromQL rule evaluation.

**Payload:**

```json
{
  "specversion": "1.0",
  "id": "evt-550e8400-e29b-41d4-a716-446655440001",
  "source": "erp-observability/alertmanager",
  "type": "alert.firing",
  "time": "2026-02-24T10:15:00.000Z",
  "datacontenttype": "application/json",
  "subject": "alert-rule/550e8400-e29b-41d4-a716-446655440000",
  "data": {
    "alert": {
      "fingerprint": "abc123def456",
      "status": "firing",
      "starts_at": "2026-02-24T10:15:00.000Z",
      "labels": {
        "alertname": "ERPHighErrorRate",
        "module": "finance",
        "severity": "critical",
        "tenant_id": "tenant-123"
      },
      "annotations": {
        "summary": "Finance module error rate exceeds 5%",
        "description": "Current error rate: 7.2%. Threshold: 5%. Duration: 10m.",
        "runbook_url": "https://runbooks.erp.io/finance-high-error-rate",
        "dashboard_url": "https://grafana.erp.io/d/finance-overview"
      },
      "generator_url": "http://prometheus:9090/graph?g0.expr=rate(http_requests_total%7Bmodule%3D%22finance%22%2Cstatus%3D~%225..%22%7D%5B5m%5D)+%2F+rate(http_requests_total%7Bmodule%3D%22finance%22%7D%5B5m%5D)+%3E+0.05"
    },
    "rule": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Finance High Error Rate",
      "promql_expression": "rate(http_requests_total{module=\"finance\",status=~\"5..\"}[5m]) / rate(http_requests_total{module=\"finance\"}[5m]) > 0.05",
      "duration": "5m"
    },
    "tenant_id": "tenant-123",
    "module": "finance"
  }
}
```

### 2. alert.resolved

Emitted when a firing alert returns to normal state.

**Trigger:** The PromQL expression no longer evaluates to true for the specified duration.

**Payload:**

```json
{
  "specversion": "1.0",
  "id": "evt-550e8400-e29b-41d4-a716-446655440002",
  "source": "erp-observability/alertmanager",
  "type": "alert.resolved",
  "time": "2026-02-24T10:45:00.000Z",
  "datacontenttype": "application/json",
  "subject": "alert-rule/550e8400-e29b-41d4-a716-446655440000",
  "data": {
    "alert": {
      "fingerprint": "abc123def456",
      "status": "resolved",
      "starts_at": "2026-02-24T10:15:00.000Z",
      "ends_at": "2026-02-24T10:45:00.000Z",
      "labels": {
        "alertname": "ERPHighErrorRate",
        "module": "finance",
        "severity": "critical",
        "tenant_id": "tenant-123"
      },
      "annotations": {
        "summary": "Finance module error rate exceeds 5%",
        "description": "Alert resolved. Current error rate: 0.3%."
      }
    },
    "duration_seconds": 1800,
    "tenant_id": "tenant-123",
    "module": "finance"
  }
}
```

### 3. tenant.created

Emitted when a new tenant is registered in the observability system.

**Trigger:** `POST /api/tenants` successfully creates a new tenant.

**Payload:**

```json
{
  "specversion": "1.0",
  "id": "evt-550e8400-e29b-41d4-a716-446655440003",
  "source": "erp-observability/tenant-api",
  "type": "tenant.created",
  "time": "2026-02-24T11:00:00.000Z",
  "datacontenttype": "application/json",
  "subject": "tenant/tenant-456",
  "data": {
    "tenant": {
      "id": "tenant-456",
      "name": "Acme Corporation",
      "plan": "enterprise",
      "retention_config": {
        "logs_days": 90,
        "traces_days": 30,
        "audit_days": 730,
        "metrics_days": 395
      },
      "quotas": {
        "log_ingest_gb_per_day": 50,
        "metric_series_limit": 500000,
        "trace_spans_per_day": 10000000,
        "dashboard_limit": 100,
        "alert_rule_limit": 200
      }
    },
    "created_by": "platform-admin-user-001",
    "provisioned_indexes": [
      "erp-logs",
      "erp-traces",
      "erp-audit",
      "erp-search"
    ]
  }
}
```

### 4. tenant.updated

Emitted when a tenant's observability configuration is changed.

**Trigger:** `PUT /api/tenants/:id/config` or `PUT /api/tenants/:id/retention`.

**Payload:**

```json
{
  "specversion": "1.0",
  "id": "evt-550e8400-e29b-41d4-a716-446655440004",
  "source": "erp-observability/tenant-api",
  "type": "tenant.updated",
  "time": "2026-02-24T11:30:00.000Z",
  "datacontenttype": "application/json",
  "subject": "tenant/tenant-456",
  "data": {
    "tenant_id": "tenant-456",
    "changes": {
      "retention_config": {
        "old": { "logs_days": 90, "traces_days": 30 },
        "new": { "logs_days": 180, "traces_days": 60 }
      }
    },
    "updated_by": "tenant-admin-user-002",
    "reason": "Extended retention for compliance audit"
  }
}
```

### 5. dashboard.updated

Emitted when a dashboard is created, updated, or deleted.

**Trigger:** `POST /v1/observability/dashboards`, `PUT /v1/observability/dashboards/:id`.

**Payload:**

```json
{
  "specversion": "1.0",
  "id": "evt-550e8400-e29b-41d4-a716-446655440005",
  "source": "erp-observability/api",
  "type": "dashboard.updated",
  "time": "2026-02-24T12:00:00.000Z",
  "datacontenttype": "application/json",
  "subject": "dashboard/880e8400-e29b-41d4-a716-446655440003",
  "data": {
    "dashboard": {
      "id": "880e8400-e29b-41d4-a716-446655440003",
      "name": "Finance Overview",
      "panel_count": 8,
      "tags": ["finance", "overview"]
    },
    "action": "updated",
    "tenant_id": "tenant-123",
    "updated_by": "user-123",
    "changes": {
      "panels_added": 2,
      "panels_removed": 0,
      "panels_modified": 1
    }
  }
}
```

---

## Event Catalog

| Event Type | Source | Frequency | Criticality |
|---|---|---|---|
| `alert.firing` | Alertmanager | Variable (per alert rule evaluation) | High |
| `alert.resolved` | Alertmanager | Variable (per alert resolution) | Medium |
| `tenant.created` | Tenant API | Rare (new tenant onboarding) | Low |
| `tenant.updated` | Tenant API | Occasional (config changes) | Low |
| `dashboard.updated` | Observability API | Frequent (user activity) | Low |

---

## Webhook Delivery Logs

Webhook delivery history is available via the Tenant API:

```
GET /api/tenants/:tenant_id/webhooks/:webhook_id/deliveries?limit=50
```

**Response:**
```json
{
  "data": [
    {
      "id": "del-001",
      "webhook_id": "wh-550e8400-e29b-41d4-a716-446655440000",
      "event_type": "alert.firing",
      "event_id": "evt-550e8400-e29b-41d4-a716-446655440001",
      "status": "success",
      "http_status": 200,
      "attempt": 1,
      "duration_ms": 245,
      "delivered_at": "2026-02-24T10:15:01.000Z"
    },
    {
      "id": "del-002",
      "webhook_id": "wh-550e8400-e29b-41d4-a716-446655440000",
      "event_type": "alert.firing",
      "event_id": "evt-550e8400-e29b-41d4-a716-446655440006",
      "status": "failed",
      "http_status": 500,
      "attempt": 3,
      "duration_ms": 1024,
      "error": "Internal Server Error",
      "delivered_at": "2026-02-24T10:20:30.000Z",
      "next_retry_at": "2026-02-24T10:30:30.000Z"
    }
  ],
  "total": 2
}
```

---

## Testing Webhooks

### Using webhook.site

For development and testing, use [webhook.site](https://webhook.site/) to inspect webhook payloads:

```bash
# Register a test webhook
curl -X POST http://localhost:8080/api/tenants/dev-tenant/webhooks \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://webhook.site/your-unique-id",
    "events": ["alert.firing", "alert.resolved", "dashboard.updated"],
    "secret": "test-secret-123"
  }'
```

### Local Testing with ngrok

```bash
# Start ngrok to expose a local endpoint
ngrok http 4000

# Register the ngrok URL as a webhook
curl -X POST http://localhost:8080/api/tenants/dev-tenant/webhooks \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-id.ngrok.io/webhooks/observability",
    "events": ["alert.firing"],
    "secret": "test-secret-123"
  }'
```

---

## Best Practices for Webhook Consumers

1. **Respond quickly:** Return a 2xx response within 5 seconds. Process the webhook payload asynchronously.
2. **Implement idempotency:** Use the `X-ERP-Webhook-ID` header to deduplicate retried deliveries.
3. **Verify signatures:** Always validate the `X-ERP-Signature-256` header to ensure the webhook is authentic.
4. **Handle all event types:** If you subscribe to multiple event types, implement a switch/case on the `type` field. Ignore unknown event types gracefully.
5. **Monitor delivery health:** Periodically check the delivery logs endpoint for failed deliveries.
6. **Return meaningful errors:** If your endpoint rejects a webhook, return an appropriate HTTP status code and error body. This information appears in delivery logs.
