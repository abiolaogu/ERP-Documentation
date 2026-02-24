# ERP-Platform Webhook Specifications

> **Document ID:** ERP-PLAT-WH-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Related Documents:** [21-API-Documentation.md](./21-API-Documentation.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## 1. Overview

ERP-Platform emits webhook notifications for key platform events. External systems can register webhook endpoints to receive real-time notifications when subscriptions, tenants, modules, and other platform resources change state.

---

## 2. Events Emitted

### 2.1 Subscription Events

| Event | Topic | Trigger | Payload |
|-------|-------|---------|---------|
| subscription.created | `erp.platform.subscription.created` | New subscription created | tenant_id, plan_type, skus |
| subscription.updated | `erp.platform.subscription.updated` | Subscription modified | tenant_id, plan_type, old_skus, new_skus |
| subscription.cancelled | `erp.platform.subscription.cancelled` | Subscription cancelled | tenant_id, cancellation_date, grace_period_end |
| subscription.reactivated | `erp.platform.subscription.reactivated` | Subscription reactivated from grace period | tenant_id, plan_type, skus |

### 2.2 Tenant Events

| Event | Topic | Trigger | Payload |
|-------|-------|---------|---------|
| tenant.provisioned | `erp.platform.tenant-provisioner.created` | New tenant provisioned | tenant_id, name, domain, subscribed_modules |
| tenant.updated | `erp.platform.tenant-provisioner.updated` | Tenant configuration changed | tenant_id, changes |
| tenant.decommissioned | `erp.platform.tenant-provisioner.deleted` | Tenant decommissioned | tenant_id, decommission_date |
| tenant.suspended | `erp.platform.tenant.suspended` | Tenant suspended (e.g., payment failure) | tenant_id, reason |

### 2.3 Module Events

| Event | Topic | Trigger | Payload |
|-------|-------|---------|---------|
| module.activated | `erp.platform.module-registry.created` | Module activated for tenant | tenant_id, module_id, module_name |
| module.deactivated | `erp.platform.module-registry.deleted` | Module deactivated | tenant_id, module_id |
| module.health_changed | `erp.platform.module-registry.updated` | Module health status changed | module_id, old_status, new_status |

### 2.4 Marketplace Events

| Event | Topic | Trigger | Payload |
|-------|-------|---------|---------|
| marketplace.installed | `erp.platform.marketplace.created` | Module installed from marketplace | tenant_id, module_id, version |
| marketplace.uninstalled | `erp.platform.marketplace.deleted` | Marketplace module removed | tenant_id, module_id |
| marketplace.updated | `erp.platform.marketplace.updated` | Marketplace module upgraded | tenant_id, module_id, old_version, new_version |

### 2.5 Entitlement Events

| Event | Topic | Trigger | Payload |
|-------|-------|---------|---------|
| entitlement.granted | `erp.platform.entitlement-engine.created` | Entitlement added | tenant_id, sku, granted_at |
| entitlement.revoked | `erp.platform.entitlement-engine.deleted` | Entitlement removed | tenant_id, sku, revoked_at |

### 2.6 Audit Events

| Event | Topic | Trigger | Payload |
|-------|-------|---------|---------|
| audit.aidd_blocked | `erp.platform.audit.aidd.blocked` | AIDD guardrail blocked an action | tenant_id, action, confidence, reason |
| audit.aidd_approved | `erp.platform.audit.aidd.approved` | AIDD guardrail approved an action | tenant_id, action, confidence |

---

## 3. Payload Schema

### 3.1 Webhook Envelope

All webhook payloads follow the CloudEvents v1.0 specification:

```json
{
    "specversion": "1.0",
    "id": "evt-550e8400-e29b-41d4-a716-446655440000",
    "source": "erp-platform/subscription-hub",
    "type": "erp.platform.subscription.created",
    "datacontenttype": "application/json",
    "time": "2026-02-23T14:30:00.000Z",
    "tenantid": "acme-corp",
    "correlationid": "corr-123e4567-e89b-12d3-a456-426614174000",
    "data": {
        "tenant_id": "acme-corp",
        "plan_type": "bundle",
        "skus": ["erp-crm", "erp-workspace", "erp-bi"]
    }
}
```

### 3.2 Subscription Created Payload

```json
{
    "data": {
        "tenant_id": "acme-corp",
        "plan_type": "bundle",
        "skus": ["erp-crm", "erp-workspace", "erp-bi"],
        "bundle_sku": "starter",
        "created_at": "2026-02-23T14:30:00.000Z"
    }
}
```

### 3.3 Tenant Provisioned Payload

```json
{
    "data": {
        "tenant_id": "acme-corp",
        "name": "Acme Corporation",
        "domain": "acme.example.com",
        "subscribed_modules": ["erp-crm", "erp-workspace", "erp-bi"],
        "provisioned_at": "2026-02-23T14:31:00.000Z"
    }
}
```

---

## 4. Retry Policy

| Attempt | Delay | Timeout |
|---------|-------|---------|
| 1 (initial) | Immediate | 10 seconds |
| 2 (retry 1) | 30 seconds | 10 seconds |
| 3 (retry 2) | 2 minutes | 10 seconds |
| 4 (retry 3) | 10 minutes | 10 seconds |
| 5 (retry 4) | 1 hour | 10 seconds |
| 6 (retry 5) | 4 hours | 10 seconds |

**Success criteria:** HTTP 2xx response within timeout.
**Failure handling:** After 6 failed attempts, the event is moved to a dead-letter queue. The webhook endpoint is marked as "failing" in the admin console. After 24 hours of consecutive failures, the endpoint is deactivated and the tenant admin is notified.

---

## 5. Signature Verification

All webhook payloads are signed using HMAC-SHA256. The signature is included in the `X-ERP-Signature` header.

### Verification Process

```
signature = HMAC-SHA256(webhook_secret, raw_request_body)
```

### Verification Example (Go)

```go
func verifySignature(secret, body []byte, signature string) bool {
    mac := hmac.New(sha256.New, secret)
    mac.Write(body)
    expected := hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(expected), []byte(signature))
}
```

### Verification Example (Node.js)

```javascript
const crypto = require('crypto');

function verifySignature(secret, body, signature) {
    const expected = crypto
        .createHmac('sha256', secret)
        .update(body, 'utf8')
        .digest('hex');
    return crypto.timingSafeEqual(
        Buffer.from(expected),
        Buffer.from(signature)
    );
}
```

### Headers Sent

| Header | Description |
|--------|-------------|
| `Content-Type` | `application/json` |
| `X-ERP-Signature` | HMAC-SHA256 signature of the body |
| `X-ERP-Event-Type` | Event type (e.g., `subscription.created`) |
| `X-ERP-Event-ID` | Unique event identifier (UUID) |
| `X-ERP-Timestamp` | ISO 8601 event timestamp |
| `X-ERP-Delivery-Attempt` | Attempt number (1-6) |
| `User-Agent` | `ERP-Platform-Webhook/1.0` |

---

## 6. Endpoint Registration API

### Register Webhook Endpoint

```
POST /v1/webhooks
X-Tenant-ID: acme-corp
Content-Type: application/json

{
    "url": "https://acme.example.com/webhooks/erp-platform",
    "events": ["subscription.created", "tenant.provisioned", "module.activated"],
    "secret": "whsec_your_webhook_secret_here",
    "description": "Acme Corp integration endpoint",
    "active": true
}
```

**Response (201 Created):**
```json
{
    "id": "whk-550e8400-e29b-41d4-a716-446655440000",
    "url": "https://acme.example.com/webhooks/erp-platform",
    "events": ["subscription.created", "tenant.provisioned", "module.activated"],
    "active": true,
    "created_at": "2026-02-23T14:30:00.000Z"
}
```

### List Webhook Endpoints

```
GET /v1/webhooks
X-Tenant-ID: acme-corp
```

### Update Webhook Endpoint

```
PUT /v1/webhooks/{id}
X-Tenant-ID: acme-corp
```

### Delete Webhook Endpoint

```
DELETE /v1/webhooks/{id}
X-Tenant-ID: acme-corp
```

### Test Webhook Endpoint

```
POST /v1/webhooks/{id}/test
X-Tenant-ID: acme-corp
```

Sends a test event to the registered URL and returns the response status.

---

## 7. Best Practices

1. **Idempotency**: Use the `X-ERP-Event-ID` header to deduplicate events.
2. **Respond Quickly**: Return 200 immediately and process asynchronously.
3. **Verify Signatures**: Always verify the `X-ERP-Signature` header to prevent spoofing.
4. **Handle Retries**: Expect the same event to be delivered multiple times.
5. **Monitor Health**: Check the webhook status in the admin console for delivery failures.
6. **HTTPS Only**: Webhook URLs must use HTTPS in production.

---

*For API documentation, see [21-API-Documentation.md](./21-API-Documentation.md). For event schemas, see [14-Technical-Specifications.md](./14-Technical-Specifications.md).*
