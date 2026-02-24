# API Reference -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Overview

This document provides the complete API reference for ERP-iPaaS, covering the Open Integration Layer API and all six core service APIs.

**OpenAPI Specification**: `api/openapi/integration-layer.yaml`

## 2. Authentication

All API requests require authentication via one of:

| Method | Header | Format |
|--------|--------|--------|
| OAuth2 Bearer | `Authorization` | `Bearer <jwt>` |
| mTLS | Client certificate | X.509 |
| API Key | `X-API-Key` | String |

All requests require tenant context:

| Header | Required | Source |
|--------|----------|-------|
| `X-Tenant-ID` | Yes | Extracted from JWT `tenant_id` claim by gateway |

## 3. Open Integration Layer API

### 3.1 Connectors

#### List Connectors

```
GET /tenants/{tenantId}/connectors?page=1&pageSize=20
```

**Response** (200):
```json
{
  "items": [
    {
      "id": "conn-uuid",
      "name": "salesforce-crm",
      "version": "1.2.0",
      "status": "published",
      "owner": "integrations-team",
      "categories": ["CRM", "Sales"],
      "rating": 4.5,
      "qualityScore": 87,
      "badges": ["verified", "enterprise"],
      "createdAt": "2026-01-15T10:00:00Z",
      "updatedAt": "2026-02-20T14:30:00Z"
    }
  ],
  "page": 1,
  "pageSize": 20,
  "total": 105
}
```

#### Create Connector

```
POST /tenants/{tenantId}/connectors
Content-Type: application/json

{
  "name": "my-custom-connector",
  "description": "Connects to internal billing system",
  "semanticVersion": "1.0.0",
  "owner": "billing-team",
  "categories": ["Finance", "Billing"],
  "schemas": [
    { "name": "invoice-schema", "version": "1.0" }
  ],
  "rateLimits": [
    { "unit": "minute", "limit": 100, "burst": 20, "tenantScoped": true }
  ],
  "badges": []
}
```

**Response** (201): Connector object.

#### Update Connector

```
PATCH /tenants/{tenantId}/connectors/{connectorId}
Content-Type: application/json

{
  "description": "Updated description",
  "semanticVersion": "1.1.0",
  "marketplace": {
    "visibility": "public",
    "listingUrl": "https://marketplace.example.com/my-connector"
  }
}
```

#### Validate Connector

```
POST /tenants/{tenantId}/connectors/{connectorId}/versions/{version}/validate
Content-Type: application/json

{
  "sandboxBaseUrl": "https://sandbox.example.com",
  "concurrentUsers": 10,
  "expectedRateLimitPerMinute": 100
}
```

**Response** (202):
```json
{
  "jobId": "val-uuid",
  "status": "queued",
  "reportUrl": ""
}
```

### 3.2 Triggers

#### Create Trigger

```
POST /tenants/{tenantId}/triggers
Content-Type: application/json

{
  "connectorId": "conn-uuid",
  "type": "webhook",
  "schemaRef": "lead-event-schema:v1"
}
```

**Response** (201):
```json
{
  "id": "trg-uuid",
  "connectorId": "conn-uuid",
  "type": "webhook",
  "schemaRef": "lead-event-schema:v1",
  "webhookUrl": "https://api.example.com/webhooks/trg-uuid",
  "createdAt": "2026-02-23T10:00:00Z"
}
```

### 3.3 Actions

#### Create Action

```
POST /tenants/{tenantId}/actions
Content-Type: application/json

{
  "connectorId": "conn-uuid",
  "schemaRef": "create-invoice-schema:v1",
  "rateLimit": {
    "unit": "minute",
    "limit": 60,
    "burst": 10,
    "tenantScoped": true
  },
  "retryPolicy": {
    "maximumAttempts": 5,
    "initialIntervalSeconds": 5,
    "maximumIntervalSeconds": 300,
    "backoffCoefficient": 2.0,
    "jitterPercent": 10
  }
}
```

### 3.4 Secrets

#### Store Secret

```
POST /tenants/{tenantId}/secrets
Content-Type: application/json

{
  "name": "salesforce-api-key",
  "description": "Salesforce production API key",
  "payload": {
    "api_key": "sk_live_...",
    "api_secret": "..."
  },
  "rotationPeriodDays": 90,
  "complianceTags": ["PCI", "SOC2"]
}
```

**Response** (204): No content.

#### List Secret Metadata

```
GET /tenants/{tenantId}/secrets
```

**Response** (200):
```json
{
  "items": [
    {
      "name": "salesforce-api-key",
      "createdAt": "2026-01-15T10:00:00Z",
      "lastRotatedAt": "2026-02-15T10:00:00Z",
      "complianceTags": ["PCI", "SOC2"]
    }
  ]
}
```

#### Secret Audit Trail

```
GET /tenants/{tenantId}/secrets/{secretId}/audit
```

**Response** (200):
```json
{
  "items": [
    {
      "eventId": "evt-uuid",
      "actor": "user@example.com",
      "action": "secret.read",
      "timestamp": "2026-02-23T10:30:00Z",
      "ipAddress": "192.168.1.100",
      "devicePosture": "compliant"
    }
  ]
}
```

### 3.5 Webhooks

#### Generate Signing Secret

```
POST /tenants/{tenantId}/webhooks/signatures
Content-Type: application/json

{
  "targetPlatform": "zapier",
  "callbackUrl": "https://hooks.zapier.com/...",
  "scopes": ["workflow.trigger"]
}
```

**Response** (201):
```json
{
  "signingSecret": "whsec_...",
  "expiresAt": "2027-02-23T10:00:00Z",
  "algorithm": "HMAC-SHA256",
  "verificationExample": "echo -n $BODY | openssl dgst -sha256 -hmac $SECRET"
}
```

### 3.6 Schemas

#### Upload Schema

```
POST /tenants/{tenantId}/schemas
Content-Type: application/json

{
  "name": "lead-event",
  "type": "avro",
  "definition": {
    "type": "record",
    "name": "LeadEvent",
    "fields": [
      { "name": "email", "type": "string" },
      { "name": "company", "type": "string" },
      { "name": "source", "type": "string" }
    ]
  }
}
```

### 3.7 Events

#### Publish Event

```
POST /tenants/{tenantId}/events
Content-Type: application/json

{
  "topic": "workflow.events",
  "payload": {
    "workflowId": "wf-uuid",
    "status": "completed"
  },
  "idempotencyKey": "unique-key-123",
  "attributes": {
    "source": "etl-service",
    "priority": "high"
  }
}
```

**Response** (202):
```json
{
  "enqueueTime": "2026-02-23T10:30:00.123Z",
  "offset": 42
}
```

## 4. Core Service APIs

### 4.1 Common Pattern

All six core services follow the same REST pattern:

```mermaid
graph LR
    H[GET /healthz] --> |"Health check"| S{Service}
    L[GET /v1/{svc}] --> |"List"| S
    C[POST /v1/{svc}] --> |"Create"| S
    R[GET /v1/{svc}/{id}] --> |"Read"| S
    U[PATCH /v1/{svc}/{id}] --> |"Update"| S
    D[DELETE /v1/{svc}/{id}] --> |"Delete"| S
```

### 4.2 Service Endpoints

| Service | Health | List | Create | Read | Update | Delete |
|---------|--------|------|--------|------|--------|--------|
| Workflow Engine | `/healthz` | `GET /v1/workflow-engine` | `POST /v1/workflow-engine` | `GET /v1/workflow-engine/{id}` | `PATCH /v1/workflow-engine/{id}` | `DELETE /v1/workflow-engine/{id}` |
| Connector Framework | `/healthz` | `GET /v1/connector-framework` | `POST /v1/connector-framework` | `GET /v1/connector-framework/{id}` | `PATCH /v1/connector-framework/{id}` | `DELETE /v1/connector-framework/{id}` |
| Event Backbone | `/healthz` | `GET /v1/event-backbone` | `POST /v1/event-backbone` | `GET /v1/event-backbone/{id}` | `PATCH /v1/event-backbone/{id}` | `DELETE /v1/event-backbone/{id}` |
| API Management | `/healthz` | `GET /v1/api-management` | `POST /v1/api-management` | `GET /v1/api-management/{id}` | `PATCH /v1/api-management/{id}` | `DELETE /v1/api-management/{id}` |
| ETL | `/healthz` | `GET /v1/etl` | `POST /v1/etl` | `GET /v1/etl/{id}` | `PATCH /v1/etl/{id}` | `DELETE /v1/etl/{id}` |
| Webhook | `/healthz` | `GET /v1/webhook` | `POST /v1/webhook` | `GET /v1/webhook/{id}` | `PATCH /v1/webhook/{id}` | `DELETE /v1/webhook/{id}` |

## 5. Error Codes

| HTTP Status | Meaning | Example |
|------------|---------|---------|
| 200 | Success | Resource retrieved |
| 201 | Created | Resource created |
| 202 | Accepted | Async job queued |
| 204 | No Content | Secret stored |
| 400 | Bad Request | Missing X-Tenant-ID |
| 401 | Unauthorized | Invalid JWT |
| 403 | Forbidden | Insufficient scopes |
| 404 | Not Found | Resource does not exist |
| 405 | Method Not Allowed | Wrong HTTP method |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected failure |

## 6. Pagination

```json
{
  "items": [...],
  "page": 1,
  "pageSize": 20,
  "total": 105
}
```

Parameters: `page` (min 1, default 1), `pageSize` (min 1, max 200, default 20).
