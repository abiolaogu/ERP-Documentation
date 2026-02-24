# ERP-AIOps Webhook Specifications

> **Document ID:** ERP-AIOPS-WH-023
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [21-API-Documentation.md](./21-API-Documentation.md), [29-Integration-Patterns.md](./29-Integration-Patterns.md)

---

## 1. Webhook Overview

ERP-AIOps supports outbound webhooks to notify external systems of important events. Webhooks are configured per tenant in **Settings > Integrations > Webhooks**.

### General Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| URL | The HTTPS endpoint to receive webhook payloads | Required |
| Events | List of event types to subscribe to | All |
| Secret | HMAC secret for payload signing | Auto-generated |
| Active | Enable/disable the webhook | true |
| Timeout | Request timeout in seconds | 10s |
| Retry Policy | Number of retries on failure | 3 |

---

## 2. Incident Webhook Payload

Triggered when an incident is created, updated, resolved, or closed.

### Event Types

- `incident.created`
- `incident.updated`
- `incident.state_changed`
- `incident.resolved`
- `incident.closed`

### Payload Format

```json
{
  "event_type": "incident.created",
  "event_id": "evt-550e8400-e29b-41d4-a716-446655440001",
  "timestamp": "2026-02-24T10:30:00Z",
  "tenant_id": "tenant-001",
  "data": {
    "incident": {
      "id": "inc-550e8400-e29b-41d4-a716-446655440002",
      "title": "ERP-CRM Database Connection Pool Exhaustion",
      "description": "Connection pool for ERP-CRM PostgreSQL reached 100% utilization, causing request timeouts.",
      "severity": "P2",
      "state": "detected",
      "priority": 2,
      "affected_services": [
        {
          "id": "svc-crm-api",
          "name": "ERP-CRM API",
          "health": "critical"
        }
      ],
      "anomaly_ids": ["anom-001", "anom-002"],
      "assignee": {
        "id": "user-001",
        "name": "Jane Doe",
        "email": "jane.doe@example.com"
      },
      "created_at": "2026-02-24T10:30:00Z",
      "updated_at": "2026-02-24T10:30:00Z",
      "sla": {
        "acknowledge_by": "2026-02-24T10:45:00Z",
        "resolve_by": "2026-02-24T14:30:00Z"
      },
      "url": "https://aiops.example.com/incidents/inc-550e8400-e29b-41d4-a716-446655440002"
    }
  }
}
```

---

## 3. Anomaly Detection Webhook

Triggered when a new anomaly is detected or when an anomaly score significantly changes.

### Event Types

- `anomaly.detected`
- `anomaly.score_changed`
- `anomaly.resolved`

### Payload Format

```json
{
  "event_type": "anomaly.detected",
  "event_id": "evt-550e8400-e29b-41d4-a716-446655440003",
  "timestamp": "2026-02-24T10:28:00Z",
  "tenant_id": "tenant-001",
  "data": {
    "anomaly": {
      "id": "anom-550e8400-e29b-41d4-a716-446655440004",
      "metric_name": "cpu_usage_percent",
      "service": {
        "id": "svc-commerce-api",
        "name": "ERP-Commerce API"
      },
      "score": 0.87,
      "severity": "critical",
      "algorithm_scores": {
        "zscore": 0.82,
        "iqr": 0.79,
        "isolation_forest": 0.91,
        "lstm": 0.85,
        "moving_average": 0.78
      },
      "current_value": 94.2,
      "expected_value": 45.8,
      "threshold": 0.7,
      "detected_at": "2026-02-24T10:28:00Z",
      "contributing_factors": [
        {"factor": "rate_of_change", "contribution": 0.42},
        {"factor": "absolute_deviation", "contribution": 0.31},
        {"factor": "seasonal_mismatch", "contribution": 0.18},
        {"factor": "cross_metric_correlation", "contribution": 0.09}
      ],
      "url": "https://aiops.example.com/anomalies/anom-550e8400-e29b-41d4-a716-446655440004"
    }
  }
}
```

---

## 4. Remediation Execution Webhook

Triggered at key stages of remediation execution.

### Event Types

- `remediation.started`
- `remediation.approval_required`
- `remediation.approved`
- `remediation.rejected`
- `remediation.completed`
- `remediation.failed`
- `remediation.rolled_back`

### Payload Format

```json
{
  "event_type": "remediation.completed",
  "event_id": "evt-550e8400-e29b-41d4-a716-446655440005",
  "timestamp": "2026-02-24T10:31:30Z",
  "tenant_id": "tenant-001",
  "data": {
    "remediation": {
      "id": "remed-550e8400-e29b-41d4-a716-446655440006",
      "playbook": {
        "id": "pb-001",
        "name": "Auto-Scale on CPU Spike"
      },
      "trigger": {
        "type": "incident",
        "id": "inc-550e8400-e29b-41d4-a716-446655440002"
      },
      "action": {
        "type": "scale_out",
        "target_service": "svc-commerce-api",
        "parameters": {
          "current_replicas": 3,
          "target_replicas": 6
        }
      },
      "approval": {
        "policy": "auto_approve",
        "approved_by": "system",
        "approved_at": "2026-02-24T10:30:05Z"
      },
      "execution": {
        "started_at": "2026-02-24T10:30:06Z",
        "completed_at": "2026-02-24T10:31:30Z",
        "duration_ms": 84000,
        "status": "success"
      },
      "verification": {
        "condition": "cpu_usage_percent < 80",
        "result": "passed",
        "verified_at": "2026-02-24T10:31:30Z"
      },
      "url": "https://aiops.example.com/remediation/activity/remed-550e8400-e29b-41d4-a716-446655440006"
    }
  }
}
```

---

## 5. GitHub Issue Creation Webhook

Triggered when AIOps creates or updates a GitHub issue for an incident.

### Event Types

- `github.issue_created`
- `github.issue_updated`
- `github.issue_closed`

### Payload Format

```json
{
  "event_type": "github.issue_created",
  "event_id": "evt-550e8400-e29b-41d4-a716-446655440007",
  "timestamp": "2026-02-24T10:30:15Z",
  "tenant_id": "tenant-001",
  "data": {
    "github": {
      "repository": "org/erp-operations",
      "issue_number": 847,
      "issue_url": "https://github.com/org/erp-operations/issues/847",
      "title": "[P2] ERP-CRM Database Connection Pool Exhaustion",
      "labels": ["critical", "aiops-incident", "erp-crm"],
      "assignees": ["janedoe"],
      "incident_id": "inc-550e8400-e29b-41d4-a716-446655440002",
      "action": "created"
    }
  }
}
```

---

## 6. Slack/Teams Notification Webhooks

### Slack Notification Payload

AIOps sends Slack Block Kit formatted messages to configured Slack webhook URLs.

```json
{
  "event_type": "notification.slack",
  "event_id": "evt-550e8400-e29b-41d4-a716-446655440008",
  "timestamp": "2026-02-24T10:30:02Z",
  "tenant_id": "tenant-001",
  "data": {
    "channel": "#aiops-incidents",
    "blocks": [
      {
        "type": "header",
        "text": {
          "type": "plain_text",
          "text": ":rotating_light: P2 Incident: ERP-CRM Database Connection Pool Exhaustion"
        }
      },
      {
        "type": "section",
        "fields": [
          {"type": "mrkdwn", "text": "*Severity:* P2 - High"},
          {"type": "mrkdwn", "text": "*State:* Detected"},
          {"type": "mrkdwn", "text": "*Services:* ERP-CRM API"},
          {"type": "mrkdwn", "text": "*Assigned:* Jane Doe"}
        ]
      },
      {
        "type": "actions",
        "elements": [
          {
            "type": "button",
            "text": {"type": "plain_text", "text": "View in AIOps"},
            "url": "https://aiops.example.com/incidents/inc-002",
            "style": "primary"
          },
          {
            "type": "button",
            "text": {"type": "plain_text", "text": "Acknowledge"},
            "action_id": "acknowledge_incident",
            "value": "inc-002"
          }
        ]
      }
    ]
  }
}
```

### Microsoft Teams Notification Payload

AIOps sends Adaptive Card formatted messages to configured Teams webhook URLs.

```json
{
  "event_type": "notification.teams",
  "event_id": "evt-550e8400-e29b-41d4-a716-446655440009",
  "timestamp": "2026-02-24T10:30:02Z",
  "tenant_id": "tenant-001",
  "data": {
    "type": "message",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.adaptive",
        "content": {
          "type": "AdaptiveCard",
          "version": "1.4",
          "body": [
            {
              "type": "TextBlock",
              "text": "P2 Incident: ERP-CRM Database Connection Pool Exhaustion",
              "weight": "bolder",
              "size": "medium",
              "color": "attention"
            },
            {
              "type": "FactSet",
              "facts": [
                {"title": "Severity", "value": "P2 - High"},
                {"title": "State", "value": "Detected"},
                {"title": "Services", "value": "ERP-CRM API"},
                {"title": "Assigned", "value": "Jane Doe"}
              ]
            }
          ],
          "actions": [
            {
              "type": "Action.OpenUrl",
              "title": "View in AIOps",
              "url": "https://aiops.example.com/incidents/inc-002"
            }
          ]
        }
      }
    ]
  }
}
```

---

## 7. Webhook Retry Policy and Error Handling

### Retry Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| Max Retries | 3 | Maximum number of retry attempts |
| Initial Delay | 5 seconds | Delay before first retry |
| Backoff Multiplier | 2x | Exponential backoff multiplier |
| Max Delay | 300 seconds | Maximum delay between retries |
| Retry On | 5xx, timeout, network error | HTTP status codes that trigger retry |
| No Retry On | 4xx (except 429) | Client errors do not trigger retry |

### Retry Schedule Example

| Attempt | Delay | Cumulative Time |
|---------|-------|-----------------|
| 1 (initial) | 0s | 0s |
| 2 (retry 1) | 5s | 5s |
| 3 (retry 2) | 10s | 15s |
| 4 (retry 3) | 20s | 35s |

### Rate Limiting (429)

When a 429 response is received, AIOps respects the `Retry-After` header. If no header is present, it falls back to the exponential backoff schedule.

### Failure Handling

After all retries are exhausted:

1. The event is moved to a **dead letter queue** (DLQ).
2. A `webhook.delivery_failed` event is logged internally.
3. An admin notification is sent if more than 10 deliveries fail within 1 hour.
4. DLQ events can be manually replayed from **Settings > Webhooks > Dead Letter Queue**.

### Webhook Status Dashboard

The webhook configuration page shows:

- **Last Delivery**: Timestamp of last successful delivery.
- **Success Rate (24h)**: Percentage of successful deliveries.
- **Average Latency**: Mean response time from the endpoint.
- **DLQ Count**: Number of events in the dead letter queue.

---

## 8. Webhook Authentication (HMAC Signatures)

### Signing Algorithm

All webhook payloads are signed using HMAC-SHA256. The signature is included in the `X-AIOps-Signature-256` header.

### Signature Computation

```
signature = HMAC-SHA256(webhook_secret, raw_request_body)
header_value = "sha256=" + hex(signature)
```

### Verification Example (Python)

```python
import hmac
import hashlib

def verify_webhook(payload_body: bytes, signature_header: str, secret: str) -> bool:
    expected = "sha256=" + hmac.new(
        secret.encode("utf-8"),
        payload_body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature_header)
```

### Verification Example (Go)

```go
func verifyWebhook(body []byte, signatureHeader, secret string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(body)
    expected := "sha256=" + hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(expected), []byte(signatureHeader))
}
```

### Additional Security Headers

| Header | Description |
|--------|-------------|
| `X-AIOps-Signature-256` | HMAC-SHA256 signature of the payload |
| `X-AIOps-Event-Type` | The event type (e.g., `incident.created`) |
| `X-AIOps-Event-ID` | Unique event ID for idempotency |
| `X-AIOps-Timestamp` | Unix timestamp of event generation |
| `X-AIOps-Tenant-ID` | Tenant ID for the event |
| `Content-Type` | `application/json` |
| `User-Agent` | `ERP-AIOps-Webhook/1.0` |

### Timestamp Validation

To prevent replay attacks, receivers should validate that `X-AIOps-Timestamp` is within an acceptable window (recommended: 5 minutes). Events older than the window should be rejected.
