# ERP-CRM Technical Specifications

## 1. System Requirements

### 1.1 Hardware Requirements

| Component | Minimum | Recommended | Enterprise |
|-----------|---------|-------------|-----------|
| CPU | 2 cores | 4 cores | 8+ cores |
| RAM | 4 GB | 8 GB | 16+ GB |
| Storage | 20 GB SSD | 100 GB SSD | 500 GB+ NVMe |
| Network | 100 Mbps | 1 Gbps | 10 Gbps |

### 1.2 Software Requirements

| Software | Version | Required |
|----------|---------|----------|
| Rust | 2021 edition (1.75+) | Yes (build) |
| PostgreSQL | 16+ | Yes |
| NATS JetStream | 2.10+ | Optional |
| Docker | 24+ | Recommended |
| Kubernetes | 1.28+ | Production |
| Go | 1.21+ | Yes (microservices) |
| Node.js | 18+ | Yes (web frontend) |

## 2. API Specifications

### 2.1 REST API Base URL

```
Production:  https://crm.{domain}/api/v1
Development: http://localhost:8081/api/v1
```

### 2.2 Authentication

All business endpoints require JWT authentication via ERP-IAM:

```
Authorization: Bearer <jwt_token>
X-Tenant-ID: <tenant_uuid>
```

### 2.3 Common Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | Bearer JWT token |
| `X-Tenant-ID` | Yes | Tenant identifier for data scoping |
| `Content-Type` | Yes (POST/PUT) | `application/json` |
| `Accept` | Optional | `application/json` (default) |

### 2.4 Pagination

All list endpoints support pagination:

| Parameter | Type | Default | Max | Description |
|-----------|------|---------|-----|-------------|
| `page` | u32 | 1 | - | Page number (1-based) |
| `per_page` | u32 | 20 | 100 | Items per page |
| `search` | string | - | - | Full-text search |
| `sort_by` | string | `created_at` | - | Sort field |
| `sort_order` | string | `desc` | - | `asc` or `desc` |

Response format:
```json
{
  "data": [...],
  "total": 150,
  "page": 1,
  "per_page": 20,
  "total_pages": 8
}
```

### 2.5 Endpoint Specifications

#### Contacts

```
GET    /api/v1/contacts              List contacts (paginated)
POST   /api/v1/contacts              Create contact
GET    /api/v1/contacts/:id          Get contact by ID
PUT    /api/v1/contacts/:id          Update contact
DELETE /api/v1/contacts/:id          Delete contact
```

**Create Contact Request:**
```json
{
  "email": "jane@example.com",
  "first_name": "Jane",
  "last_name": "Smith",
  "phone": "+1-555-0100",
  "company_id": "uuid-optional",
  "source": "website",
  "tags": ["enterprise", "vip"],
  "custom_fields": {
    "industry": "Technology"
  }
}
```

**Contact Response:**
```json
{
  "id": "019503a2-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "email": "jane@example.com",
  "first_name": "Jane",
  "last_name": "Smith",
  "phone": "+1-555-0100",
  "company_id": null,
  "lead_score": 0,
  "lifecycle_stage": "subscriber",
  "source": "website",
  "owner_id": null,
  "tags": ["enterprise", "vip"],
  "custom_fields": {"industry": "Technology"},
  "created_at": "2026-02-23T10:00:00Z",
  "updated_at": "2026-02-23T10:00:00Z"
}
```

#### Companies

```
GET    /api/v1/companies              List companies (paginated)
POST   /api/v1/companies              Create company
GET    /api/v1/companies/:id          Get company by ID
PUT    /api/v1/companies/:id          Update company
DELETE /api/v1/companies/:id          Delete company
```

#### Deals

```
GET    /api/v1/deals                   List deals (paginated)
POST   /api/v1/deals                   Create deal
GET    /api/v1/deals/:id               Get deal by ID
PUT    /api/v1/deals/:id               Update deal
DELETE /api/v1/deals/:id               Delete deal
```

#### Activities

```
GET    /api/v1/activities              List activities (paginated)
POST   /api/v1/activities              Create activity
GET    /api/v1/activities/:id          Get activity by ID
```

#### Dashboard

```
GET    /api/v1/dashboard/stats         Get dashboard statistics
```

#### Health Endpoints

```
GET    /health                         Service health check
GET    /ready                          Database readiness probe
GET    /metrics                        Prometheus metrics
```

### 2.6 Microservice Endpoints

Each microservice exposes:

```
GET    /healthz                        Health check
GET    /v1/{entity}                    List (requires X-Tenant-ID)
POST   /v1/{entity}                    Create (requires X-Tenant-ID)
GET    /v1/{entity}/{id}               Read (requires X-Tenant-ID)
PUT    /v1/{entity}/{id}               Update (requires X-Tenant-ID)
DELETE /v1/{entity}/{id}               Delete (requires X-Tenant-ID)
```

Services: contact, lead, pipeline, opportunity, activity, helpdesk, knowledge-base, form-builder, chat, automation, reporting, territory.

## 3. Data Specifications

### 3.1 Entity ID Format

All entity IDs use UUID v7 (time-ordered):
- Format: `019503a2-xxxx-7xxx-xxxx-xxxxxxxxxxxx`
- Generated via `Uuid::now_v7()` in Rust
- Sortable by creation time
- B-tree friendly (sequential inserts)

### 3.2 Timestamp Format

All timestamps use ISO 8601 with timezone:
- Format: `2026-02-23T10:00:00.000000Z`
- Stored as `TIMESTAMPTZ` in PostgreSQL
- Serialized via `chrono::DateTime<Utc>`

### 3.3 Currency Format

Amounts stored as `BIGINT` (cents/smallest unit) in the database and `Decimal` in domain objects:
- Database: `amount BIGINT` (e.g., 10000 = $100.00)
- Domain: `Money { amount: Decimal, currency: Currency }`
- Supported currencies: USD, EUR, GBP, CAD, AUD, NGN, JPY, CNY, INR, Other(String)
- Default currency: NGN

### 3.4 Custom Fields Format

Custom fields stored as PostgreSQL `JSONB`:
- No schema restrictions (key-value pairs)
- Supports nested objects and arrays
- Queryable via PostgreSQL JSONB operators
- Default: `'{}'::jsonb`

## 4. Event Specifications

### 4.1 CloudEvents Format

```json
{
  "specversion": "1.0",
  "type": "com.opensase.crm.{entity}.{action}",
  "source": "opensase-crm",
  "id": "unique-event-uuid",
  "time": "2026-02-23T10:00:00Z",
  "datacontenttype": "application/json",
  "data": { }
}
```

### 4.2 Event Topic Naming

Convention: `erp.crm.{entity}.{action}`

Actions: created, updated, deleted, listed, read

### 4.3 Pulsar Topic Configuration

```yaml
tenant: billyronks
namespace: extract-crm
topics:
  - persistent://billyronks/extract-crm/command   (6 partitions)
  - persistent://billyronks/extract-crm/event     (6 partitions)
  - persistent://billyronks/extract-crm/audit     (6 partitions)
  - persistent://billyronks/global/observability   (6 partitions)
```

## 5. Performance Specifications

### 5.1 Latency Targets

| Operation | p50 | p95 | p99 |
|-----------|-----|-----|-----|
| Contact GET | 1ms | 3ms | 5ms |
| Contact LIST (20 items) | 3ms | 10ms | 20ms |
| Contact CREATE | 2ms | 5ms | 10ms |
| Deal stage transition | 2ms | 5ms | 10ms |
| Lead score calculation | 0.1ms | 0.5ms | 1ms |
| Dashboard stats | 10ms | 50ms | 100ms |
| Health check | 0.1ms | 0.5ms | 1ms |

### 5.2 Throughput Targets

| Metric | Target |
|--------|--------|
| Requests per second | 10,000+ (single instance) |
| Concurrent connections | 10,000+ |
| Database connections | 10 (default pool) |
| Event publish rate | 5,000/sec |

### 5.3 Resource Limits

| Resource | Limit |
|----------|-------|
| Max contacts per tenant | 10,000,000 |
| Max deals per tenant | 1,000,000 |
| Max custom fields per entity | Unlimited (JSONB) |
| Max tags per contact | 100 |
| Max products per deal | 1,000 |
| Max stages per pipeline | 20 |
| Max pipelines per tenant | 50 |
| API response body max | 10 MB |
| File upload max | 50 MB |

## 6. Security Specifications

### 6.1 Encryption

| Layer | Standard |
|-------|----------|
| In transit | TLS 1.3 |
| At rest (database) | AES-256 (PostgreSQL TDE) |
| At rest (backups) | AES-256 |
| JWT signing | RS256 or ES256 |
| Password hashing | Argon2id (via ERP-IAM) |

### 6.2 Rate Limiting

| Endpoint Group | Rate Limit |
|---------------|-----------|
| Authentication | 10 req/min per IP |
| CRUD operations | 1000 req/min per tenant |
| Search | 100 req/min per tenant |
| Bulk operations | 10 req/min per tenant |
| Health/Metrics | Unlimited |

### 6.3 Input Validation

| Field | Validation |
|-------|-----------|
| Email | RFC 5322 format, max 255 chars |
| Phone | E.164 format recommendation |
| Name fields | Max 100 chars, trimmed |
| Company name | Min 1 char, max 255 chars |
| Deal name | Min 1 char, max 255 chars |
| Tags | Array of strings, max 50 chars each |
| Custom fields | Valid JSON, max 64 KB |
| Pagination per_page | Min 1, max 100 |

## 7. Observability Specifications

### 7.1 Log Format

```json
{
  "timestamp": "2026-02-23T10:00:00Z",
  "level": "INFO",
  "service": "opensase-crm",
  "trace_id": "abc123",
  "span_id": "def456",
  "tenant_id": "tenant-uuid",
  "message": "Contact created",
  "attrs": {
    "contact_id": "uuid",
    "email": "jane@example.com"
  }
}
```

### 7.2 Metrics

Prometheus-compatible metrics exposed at `/metrics`:

```
opensase_crm_up 1
opensase_crm_requests_total{method="GET",path="/api/v1/contacts",status="200"}
opensase_crm_request_duration_seconds{quantile="0.95"}
opensase_crm_db_pool_connections{state="active"}
opensase_crm_events_published_total{topic="contact.created"}
```

### 7.3 Health Check Response

```json
{
  "status": "healthy",
  "service": "opensase-crm",
  "version": "0.1.0"
}
```

Readiness probe returns:
- `200 OK` with body "ready" when database is accessible
- `503 Service Unavailable` with body "not ready" when database is down
