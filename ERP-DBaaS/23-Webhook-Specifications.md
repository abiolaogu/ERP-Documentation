# ERP-DBaaS Webhook Specifications

This document defines the webhook events emitted by ERP-DBaaS for integration with external systems, monitoring, alerting, and audit logging.

---

## Table of Contents

1. [Overview](#overview)
2. [Webhook Delivery](#webhook-delivery)
3. [Common Payload Format](#common-payload-format)
4. [Instance Events](#instance-events)
5. [Backup Events](#backup-events)
6. [Credential Events](#credential-events)
7. [Plugin Events](#plugin-events)
8. [Webhook Registration](#webhook-registration)
9. [Retry Policy](#retry-policy)
10. [Security](#security)

---

## Overview

ERP-DBaaS emits webhook events for all significant lifecycle operations. Events are delivered via HTTP POST to registered webhook endpoints and are also published to Apache Pulsar for internal consumption.

### Event Categories

| Category | Events | Transport |
|---|---|---|
| Instance | provisioned, scaled, failed, decommissioned | HTTP + Pulsar |
| Backup | completed, failed | HTTP + Pulsar |
| Credential | rotated | HTTP + Pulsar |
| Plugin | validated, rejected | HTTP + Pulsar |

### Pulsar Topics

| Topic | Events |
|---|---|
| `persistent://dbaas/events/instance-lifecycle` | All instance events |
| `persistent://dbaas/events/backup-lifecycle` | All backup events |
| `persistent://dbaas/events/credential-lifecycle` | Credential events |
| `persistent://dbaas/events/plugin-lifecycle` | Plugin events |
| `persistent://dbaas/events/metering` | Metering events |

---

## Webhook Delivery

### HTTP Delivery

- **Method**: POST
- **Content-Type**: `application/json`
- **Timeout**: 10 seconds per delivery attempt
- **Retry policy**: Exponential backoff (3 attempts: 1s, 5s, 30s)
- **Idempotency**: Every event has a unique `eventId`; receivers should deduplicate

### Headers

Every webhook request includes these headers:

```
Content-Type: application/json
X-DBaaS-Event: instance.provisioned
X-DBaaS-Event-ID: evt-a1b2c3d4-e5f6-7890-abcd-ef1234567890
X-DBaaS-Timestamp: 2026-02-24T10:05:00.000Z
X-DBaaS-Signature: sha256=a1b2c3d4e5f6...
X-DBaaS-Delivery: dlv-1708765500-001
User-Agent: ERP-DBaaS-Webhook/1.0
```

---

## Common Payload Format

All webhook events share this envelope structure:

```json
{
  "eventId": "evt-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "eventType": "instance.provisioned",
  "version": "1.0",
  "timestamp": "2026-02-24T10:05:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708765432-1"
  },
  "data": { ... }
}
```

| Field | Type | Description |
|---|---|---|
| `eventId` | UUID | Globally unique event identifier for deduplication. |
| `eventType` | string | Dot-notation event type (e.g., `instance.provisioned`). |
| `version` | string | Event schema version. |
| `timestamp` | ISO 8601 | UTC timestamp when the event was generated. |
| `source` | string | Always `erp-dbaas`. |
| `tenantId` | string | Tenant ID that owns the resource. |
| `metadata` | object | Additional context (region, profile, correlation ID). |
| `data` | object | Event-specific payload. |

---

## Instance Events

### instance.provisioned

Emitted when an instance has been fully provisioned and is ready for use.

```json
{
  "eventId": "evt-11111111-1111-1111-1111-111111111111",
  "eventType": "instance.provisioned",
  "version": "1.0",
  "timestamp": "2026-02-24T10:05:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708765432-1"
  },
  "data": {
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "version": "2.21",
    "plan": "M",
    "haMode": "standalone",
    "namespace": "dbaas-tenant-001-a1b2c3d4",
    "endpoint": "yugabytedb-a1b2c3d4.dbaas-system.svc.cluster.local",
    "port": 5433,
    "resources": {
      "cpu": "2",
      "memory": "8Gi",
      "storage": "50Gi"
    },
    "provisionDurationSeconds": 120,
    "previousStatus": "provisioning",
    "currentStatus": "running"
  }
}
```

### instance.scaled

Emitted when an instance has been successfully scaled.

```json
{
  "eventId": "evt-22222222-2222-2222-2222-222222222222",
  "eventType": "instance.scaled",
  "version": "1.0",
  "timestamp": "2026-02-24T11:00:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708765500-5"
  },
  "data": {
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "previousPlan": "M",
    "newPlan": "L",
    "previousResources": {
      "cpu": "2",
      "memory": "8Gi",
      "storage": "50Gi",
      "replicas": 1
    },
    "newResources": {
      "cpu": "4",
      "memory": "16Gi",
      "storage": "200Gi",
      "replicas": 3
    },
    "scaleDurationSeconds": 180,
    "previousStatus": "scaling",
    "currentStatus": "running"
  }
}
```

### instance.failed

Emitted when an instance operation fails (provisioning, scaling, backup, restore).

```json
{
  "eventId": "evt-33333333-3333-3333-3333-333333333333",
  "eventType": "instance.failed",
  "version": "1.0",
  "timestamp": "2026-02-24T10:10:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708765432-1",
    "severity": "critical"
  },
  "data": {
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "failedOperation": "provisioning",
    "errorCode": "operator_timeout",
    "errorMessage": "Operator failed to provision instance within 600 seconds. The StatefulSet did not reach ready state.",
    "previousStatus": "provisioning",
    "currentStatus": "failed",
    "operatorDetails": {
      "operator": "kubedb",
      "crdStatus": "ProvisionFailed",
      "conditions": [
        {
          "type": "Ready",
          "status": "False",
          "reason": "PodSchedulingFailed",
          "message": "0/3 nodes are available: 3 Insufficient memory."
        }
      ]
    },
    "suggestedActions": [
      "Check cluster resource availability.",
      "Try a smaller plan (S or M).",
      "Contact platform support if the issue persists."
    ]
  }
}
```

### instance.decommissioned

Emitted when an instance has been fully decommissioned and all resources cleaned up.

```json
{
  "eventId": "evt-44444444-4444-4444-4444-444444444444",
  "eventType": "instance.decommissioned",
  "version": "1.0",
  "timestamp": "2026-02-24T12:00:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708765800-8"
  },
  "data": {
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "namespace": "dbaas-tenant-001-a1b2c3d4",
    "cleanedResources": ["StatefulSet", "PVC", "Service", "Secret", "Namespace"],
    "backupsRetained": 5,
    "finalMeteringEvent": {
      "totalCpuHours": 48.5,
      "totalMemoryGbHours": 194.0,
      "totalStorageGbDays": 4.2,
      "totalIoOperations": 5250000
    },
    "previousStatus": "decommissioning",
    "currentStatus": "decommissioned"
  }
}
```

---

## Backup Events

### backup.completed

Emitted when a backup operation completes successfully.

```json
{
  "eventId": "evt-55555555-5555-5555-5555-555555555555",
  "eventType": "backup.completed",
  "version": "1.0",
  "timestamp": "2026-02-24T10:35:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708765600-9"
  },
  "data": {
    "backupId": "b1b2b3b4-e5f6-7890-abcd-ef1234567890",
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "type": "full",
    "sizeBytes": 1073741824,
    "storagePath": "rustfs://backups/tenant-001/a1b2c3d4/2026-02-24T10-30-00.full.zstd",
    "storageTarget": "rustfs",
    "encrypted": true,
    "compressed": "zstd",
    "crossRegionReplicated": true,
    "retentionDays": 30,
    "expiresAt": "2026-03-26T10:35:00.000Z",
    "durationSeconds": 300,
    "previousStatus": "in_progress",
    "currentStatus": "completed"
  }
}
```

### backup.failed

Emitted when a backup operation fails.

```json
{
  "eventId": "evt-66666666-6666-6666-6666-666666666666",
  "eventType": "backup.failed",
  "version": "1.0",
  "timestamp": "2026-02-24T10:40:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708765600-10",
    "severity": "high"
  },
  "data": {
    "backupId": "b2b3b4b5-e5f6-7890-abcd-ef1234567890",
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "type": "full",
    "errorCode": "storage_unavailable",
    "errorMessage": "Failed to upload backup to RustFS: connection refused at rustfs:9000.",
    "partialSizeBytes": 536870912,
    "durationSeconds": 600,
    "previousStatus": "in_progress",
    "currentStatus": "failed",
    "suggestedActions": [
      "Check RustFS service availability.",
      "Verify network connectivity to RustFS endpoint.",
      "Retry the backup operation."
    ]
  }
}
```

---

## Credential Events

### credentials.rotated

Emitted when credentials have been successfully rotated for an instance.

```json
{
  "eventId": "evt-77777777-7777-7777-7777-777777777777",
  "eventType": "credentials.rotated",
  "version": "1.0",
  "timestamp": "2026-02-24T14:00:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708776000-13"
  },
  "data": {
    "rotationId": "r1r2r3r4-e5f6-7890-abcd-ef1234567890",
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "initiatedBy": "user-001",
    "rotationMethod": "vault-dynamic",
    "newCredentialExpiresAt": "2026-03-24T14:00:00.000Z",
    "previousUsername": "dbaas_a1b2c3d4_v1",
    "newUsername": "dbaas_a1b2c3d4_v2",
    "durationSeconds": 15,
    "previousStatus": "in_progress",
    "currentStatus": "completed",
    "impactedConnections": 0
  }
}
```

### credentials.rotation_failed

Emitted when a credential rotation fails.

```json
{
  "eventId": "evt-78787878-7878-7878-7878-787878787878",
  "eventType": "credentials.rotation_failed",
  "version": "1.0",
  "timestamp": "2026-02-24T14:05:00.000Z",
  "source": "erp-dbaas",
  "tenantId": "tenant-001",
  "metadata": {
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "correlationId": "req-1708776300-14",
    "severity": "high"
  },
  "data": {
    "rotationId": "r2r3r4r5-e5f6-7890-abcd-ef1234567890",
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "initiatedBy": "user-001",
    "errorCode": "vault_unreachable",
    "errorMessage": "Failed to connect to Vault at http://vault:8200. Connection timed out.",
    "rollbackPerformed": true,
    "previousStatus": "in_progress",
    "currentStatus": "failed",
    "suggestedActions": [
      "Check Vault service availability.",
      "Verify Vault token is not expired.",
      "Retry the rotation after resolving Vault connectivity."
    ]
  }
}
```

---

## Plugin Events

### plugin.validated

Emitted when a plugin has passed validation and is ready for activation.

```json
{
  "eventId": "evt-88888888-8888-8888-8888-888888888888",
  "eventType": "plugin.validated",
  "version": "1.0",
  "timestamp": "2026-02-24T10:00:05.000Z",
  "source": "erp-dbaas",
  "tenantId": "system",
  "metadata": {
    "correlationId": "plg-registration-001"
  },
  "data": {
    "pluginId": "p1p2p3p4-e5f6-7890-abcd-ef1234567890",
    "name": "neo4j-plugin",
    "engine": "neo4j",
    "version": "1.0.0",
    "grpcEndpoint": "http://neo4j-plugin.dbaas-plugins.svc.cluster.local:50051",
    "capabilities": ["provision", "scale", "backup", "restore", "health"],
    "validationChecks": {
      "grpcConnectivity": "passed",
      "capabilityDiscovery": "passed",
      "securityScan": "passed",
      "healthCheck": "passed"
    },
    "validationDurationSeconds": 5,
    "previousStatus": "pending_validation",
    "currentStatus": "validated"
  }
}
```

### plugin.rejected

Emitted when a plugin fails validation.

```json
{
  "eventId": "evt-99999999-9999-9999-9999-999999999999",
  "eventType": "plugin.rejected",
  "version": "1.0",
  "timestamp": "2026-02-24T10:00:10.000Z",
  "source": "erp-dbaas",
  "tenantId": "system",
  "metadata": {
    "correlationId": "plg-registration-002",
    "severity": "medium"
  },
  "data": {
    "pluginId": "p2p3p4p5-e5f6-7890-abcd-ef1234567890",
    "name": "untrusted-plugin",
    "engine": "custom-db",
    "version": "0.1.0",
    "grpcEndpoint": "http://untrusted.external.com:50051",
    "rejectionReason": "gRPC connectivity check failed: connection refused.",
    "validationChecks": {
      "grpcConnectivity": "failed",
      "capabilityDiscovery": "skipped",
      "securityScan": "skipped",
      "healthCheck": "skipped"
    },
    "previousStatus": "pending_validation",
    "currentStatus": "rejected"
  }
}
```

---

## Webhook Registration

Webhook endpoints are registered per-tenant via the tenants API:

```bash
POST /v1/dbaas/tenants/{{tenantId}}/webhooks
```

**Request Body**:

```json
{
  "url": "https://monitoring.example.com/webhooks/dbaas",
  "events": [
    "instance.provisioned",
    "instance.failed",
    "backup.completed",
    "backup.failed",
    "credentials.rotated"
  ],
  "secret": "whsec_a1b2c3d4e5f6g7h8i9j0",
  "active": true
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `url` | URL | Yes | HTTPS endpoint to deliver webhooks to. |
| `events` | string[] | Yes | List of event types to subscribe to. Use `*` for all events. |
| `secret` | string | Yes | Secret for HMAC-SHA256 signature verification. |
| `active` | boolean | No | Enable/disable the webhook (default: true). |

---

## Retry Policy

Failed webhook deliveries are retried with exponential backoff:

| Attempt | Delay | Total Elapsed |
|---|---|---|
| 1 (initial) | 0s | 0s |
| 2 (retry 1) | 1s | 1s |
| 3 (retry 2) | 5s | 6s |
| 4 (retry 3) | 30s | 36s |

A delivery is considered failed if:
- The endpoint returns a non-2xx status code.
- The connection times out (10 seconds).
- The endpoint is unreachable.

After 4 attempts, the delivery is marked as failed. Failed deliveries are stored for 72 hours and can be replayed manually.

---

## Security

### Signature Verification

Every webhook includes an `X-DBaaS-Signature` header containing an HMAC-SHA256 signature of the raw request body, computed using the webhook secret.

**Verification example (Node.js)**:

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(rawBody, signature, secret) {
  const expected = 'sha256=' + crypto
    .createHmac('sha256', secret)
    .update(rawBody, 'utf8')
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expected)
  );
}

// In your Express handler:
app.post('/webhooks/dbaas', express.raw({ type: 'application/json' }), (req, res) => {
  const signature = req.headers['x-dbaas-signature'];
  const isValid = verifyWebhookSignature(req.body, signature, process.env.WEBHOOK_SECRET);

  if (!isValid) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const event = JSON.parse(req.body);
  // Process event...

  res.status(200).json({ received: true });
});
```

### HTTPS Requirement

In production, webhook endpoints must use HTTPS. HTTP endpoints are only allowed in development mode.

### IP Allowlisting

Webhook requests originate from the DBaaS API pods in the `dbaas-system` namespace. If your firewall requires IP allowlisting, contact the platform team for the current egress IP range.
