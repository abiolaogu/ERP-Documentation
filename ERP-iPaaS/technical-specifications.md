# Technical Specifications -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. API Specification

### 1.1 Open Integration Layer API

The primary API is defined in OpenAPI 3.0.3 at `api/openapi/integration-layer.yaml`.

**Base URLs**:
- Production: `https://api.<DOMAIN>/integration`
- Sandbox: `https://sandbox.<DOMAIN>/integration`

**Authentication**: OAuth2 (authorization code or client credentials) and mTLS.

### 1.2 Endpoint Summary

```mermaid
graph LR
    subgraph "Integration Layer API"
        C[Connectors CRUD<br/>GET/POST/PATCH]
        V[Connector Validation<br/>POST]
        T[Triggers CRUD<br/>GET/POST]
        A[Actions CRUD<br/>GET/POST]
        S[Secrets Management<br/>GET/POST]
        SA[Secret Audit<br/>GET]
        W[Webhook Signatures<br/>POST]
        SC[Schema Registry<br/>GET/POST]
        E[Event Publishing<br/>POST]
    end
```

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/tenants/{tenantId}/connectors` | GET | List connectors for tenant |
| `/tenants/{tenantId}/connectors` | POST | Register new connector |
| `/tenants/{tenantId}/connectors/{id}` | GET | Fetch connector details |
| `/tenants/{tenantId}/connectors/{id}` | PATCH | Update connector metadata |
| `/tenants/{tenantId}/connectors/{id}/versions/{v}/validate` | POST | Validate connector |
| `/tenants/{tenantId}/triggers` | GET/POST | List/create triggers |
| `/tenants/{tenantId}/actions` | GET/POST | List/create actions |
| `/tenants/{tenantId}/secrets` | GET/POST | List/store secrets |
| `/tenants/{tenantId}/secrets/{id}/audit` | GET | Secret audit trail |
| `/tenants/{tenantId}/webhooks/signatures` | POST | Generate signing secret |
| `/tenants/{tenantId}/schemas` | GET/POST | List/upload schemas |
| `/tenants/{tenantId}/events` | POST | Publish event to topic |

### 1.3 Core Service APIs

| Service | Base Path | Methods |
|---------|-----------|---------|
| Workflow Engine | `/v1/workflow-engine` | GET, POST, GET/:id, PATCH/:id, DELETE/:id |
| Connector Framework | `/v1/connector-framework` | GET, POST, GET/:id, PATCH/:id, DELETE/:id |
| Event Backbone | `/v1/event-backbone` | GET, POST, GET/:id, PATCH/:id, DELETE/:id |
| API Management | `/v1/api-management` | GET, POST, GET/:id, PATCH/:id, DELETE/:id |
| ETL Service | `/v1/etl` | GET, POST, GET/:id, PATCH/:id, DELETE/:id |
| Webhook Service | `/v1/webhook` | GET, POST, GET/:id, PATCH/:id, DELETE/:id |

### 1.4 Common Request/Response Patterns

**Tenant Validation**:
All requests require `X-Tenant-ID` header (injected by gateway from JWT claim).

```
GET /v1/workflow-engine HTTP/1.1
Host: api.example.com
Authorization: Bearer <jwt>
X-Tenant-ID: a1b2c3d4-e5f6-...
```

**Success Response**:
```json
{
  "items": [...],
  "event_topic": "erp.ipaas.workflow-engine.listed"
}
```

**Error Response**:
```json
{
  "error": "missing X-Tenant-ID"
}
```

## 2. Data Specifications

### 2.1 Connector Schema

```json
{
  "id": "uuid",
  "name": "string",
  "description": "string",
  "version": "semver",
  "status": "draft | published | deprecated",
  "owner": "string",
  "categories": ["string"],
  "rating": 0.0-5.0,
  "badges": ["string"],
  "qualityScore": 0-100,
  "schemaRefs": ["string"],
  "marketplace": {
    "visibility": "private | tenant | public",
    "listingUrl": "string",
    "approvals": ["string"]
  },
  "createdAt": "ISO 8601",
  "updatedAt": "ISO 8601"
}
```

### 2.2 Rate Limit Policy Schema

```json
{
  "unit": "second | minute | hour | day",
  "limit": "integer",
  "burst": "integer (optional)",
  "tenantScoped": "boolean"
}
```

### 2.3 Retry Policy Schema

```json
{
  "maximumAttempts": 5,
  "initialIntervalSeconds": 5,
  "maximumIntervalSeconds": 300,
  "backoffCoefficient": 2.0,
  "jitterPercent": 10
}
```

## 3. Event Specifications

### 3.1 CloudEvents Envelope

```json
{
  "specversion": "1.0",
  "type": "erp.ipaas.<service>.<action>",
  "source": "/v1/<service>",
  "id": "uuid-v4",
  "time": "2026-02-23T10:30:00.000Z",
  "datacontenttype": "application/json",
  "tenantid": "uuid",
  "data": { ... }
}
```

### 3.2 Event Types

| Event Type | Description | Payload |
|-----------|-------------|---------|
| `erp.ipaas.workflow-engine.created` | New workflow registered | Workflow definition |
| `erp.ipaas.workflow-engine.updated` | Workflow modified | Updated fields |
| `erp.ipaas.workflow-engine.deleted` | Workflow removed | Workflow ID |
| `erp.ipaas.connector-framework.created` | New connector published | Connector metadata |
| `erp.ipaas.etl.created` | New ETL pipeline created | Pipeline definition |
| `erp.ipaas.webhook.created` | New webhook registered | Webhook configuration |

### 3.3 Avro Schema Specification

```json
{
  "type": "record",
  "name": "WorkflowCommand",
  "namespace": "com.billyronks.workflow",
  "fields": [
    { "name": "tenant_id", "type": "string" },
    { "name": "workflow_type", "type": "string" },
    { "name": "task_queue", "type": "string" },
    { "name": "payload", "type": { "type": "map", "values": "string" } },
    { "name": "idempotency_key", "type": "string" },
    { "name": "priority", "type": "int", "default": 0 },
    { "name": "timestamp", "type": "long", "logicalType": "timestamp-millis" }
  ]
}
```

## 4. GraphQL Schema

```graphql
schema {
  query: query_root
  mutation: mutation_root
}

type query_root {
  users: [user!]!
  organizations: [organization!]!
  projects: [project!]!
}

type mutation_root {
  insert_user_one(object: user_insert_input!): user
  insert_organization_one(object: organization_insert_input!): organization
  insert_project_one(object: project_insert_input!): project
}
```

## 5. Performance Specifications

| Metric | Specification | Measurement Method |
|--------|-------------|-------------------|
| API latency (p50) | < 10ms | Traefik metrics |
| API latency (p99) | < 50ms | Traefik metrics |
| Workflow start latency | < 100ms | Temporal metrics |
| Event produce latency | < 5ms | Redpanda metrics |
| Event consume latency | < 10ms | Consumer lag metrics |
| ClickHouse query (simple) | < 100ms | ClickHouse query log |
| ClickHouse query (aggregation) | < 1s | ClickHouse query log |
| Throughput (events/sec) | 100,000+ | Load test |
| Throughput (workflows/sec) | 1,000+ | Load test |
| Concurrent connections | 10,000+ | Load test |

## 6. Security Specifications

### 6.1 Authentication

| Method | Standard | Implementation |
|--------|----------|---------------|
| OAuth2 Authorization Code | RFC 6749 | Keycloak |
| OAuth2 Client Credentials | RFC 6749 | Keycloak |
| JWT Validation | RFC 7519 | Traefik ForwardAuth |
| mTLS | TLS 1.3 | cert-manager |
| API Key | Custom | Gateway middleware |
| Webhook HMAC | HMAC-SHA256 | Webhook service |

### 6.2 Encryption

| Scope | Algorithm | Key Size |
|-------|-----------|----------|
| Transit | TLS 1.3 | 256-bit |
| At rest (PostgreSQL) | AES-256 | 256-bit |
| At rest (MinIO) | SSE-S3 AES-256 | 256-bit |
| Secrets | AES-256-GCM | 256-bit |
| Webhook signatures | HMAC-SHA256 | 256-bit |

### 6.3 Compliance Standards

| Standard | Scope | Status |
|----------|-------|--------|
| SOC2 Type II | Full platform | In progress |
| GDPR | EU tenant data | Compliant |
| NDPR | Nigeria tenant data | Compliant |
| ISO 27001 | Security controls | Aligned |

## 7. Availability Specifications

| Tier | SLA | RPO | RTO |
|------|-----|-----|-----|
| Platform services | 99.95% | 0 | < 30s |
| Data stores | 99.99% | < 5min | < 15min |
| Event backbone | 99.99% | 0 | < 30s |
| Overall platform | 99.95% | < 5min | < 15min |

## 8. Helm Chart Specifications

### 8.1 Chart Versions

| Chart | App Version | Chart Version | Location |
|-------|-----------|--------------|----------|
| activepieces | 0.20.0 | 1.0.0 | `infra/helm/activepieces/` |
| temporal | 1.23.0 | 1.0.0 | `infra/helm/temporal/` |
| redpanda | latest | 1.0.0 | `infra/helm/redpanda/` |
| traefik | v2.10 | 1.0.0 | `infra/helm/traefik/` |
| keycloak | 22.0 | 1.0.0 | `infra/helm/keycloak/` |
| clickhouse | 23.9 | 1.0.0 | `infra/helm/clickhouse/` |
| postgres | 16 | 1.0.0 | `infra/helm/postgres/` |
| minio | 2023-10-07 | 1.0.0 | `infra/helm/minio/` |
