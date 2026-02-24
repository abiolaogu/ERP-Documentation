# ERP-Platform Technical Specifications

> **Document ID:** ERP-PLAT-TS-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [13-Low-Level-Design.md](./13-Low-Level-Design.md), [21-API-Documentation.md](./21-API-Documentation.md)

---

## 1. API Contracts

### 1.1 GET /healthz

**Subscription Hub Response:**
```json
{
    "status": "ok",
    "catalog_version": "2026-02-23"
}
```

**Service Health Response (all other services):**
```json
{
    "status": "healthy",
    "module": "ERP-Platform",
    "service": "tenant-provisioner"
}
```

### 1.2 GET /v1/products

**Response (200 OK):**
```json
{
    "version": "2026-02-23",
    "products": [
        {
            "sku": "erp-platform",
            "id": "erp-platform",
            "name": "ERP Platform",
            "repo": "ERP-Platform",
            "type": "module",
            "standalone": true,
            "category": "core",
            "capabilities": 9
        },
        {
            "sku": "enterprise",
            "id": "enterprise",
            "name": "Enterprise",
            "repo": "ERP-Platform",
            "type": "bundle",
            "standalone": false,
            "includes": [
                "erp-platform", "erp-hcm", "erp-crm", "erp-marketing",
                "erp-finance", "erp-commerce", "erp-ecommerce", "erp-bss-oss",
                "erp-scm", "erp-workspace", "erp-iam", "erp-ipaas",
                "erp-bi", "erp-ai", "erp-projects", "erp-healthcare",
                "erp-school-management", "erp-church-management",
                "erp-assistant", "erp-autonomous-coding"
            ]
        }
    ]
}
```

**Error Response (405):**
```json
{
    "error": "method not allowed"
}
```

### 1.3 POST /v1/subscriptions

**Request Body:**
```json
{
    "tenant_id": "acme-corp",
    "plan_type": "bundle",
    "skus": ["professional"]
}
```

| Field | Type | Required | Constraints |
|-------|------|----------|------------|
| tenant_id | string | Yes | Non-empty, trimmed |
| plan_type | string | Yes | One of: "single", "bundle", "suite" |
| skus | string[] | Yes | Non-empty array; each SKU must exist in catalog |

**Response (201 Created):**
```json
{
    "tenant_id": "acme-corp",
    "plan_type": "bundle",
    "skus": [
        "erp-hcm", "erp-crm", "erp-marketing", "erp-finance",
        "erp-commerce", "erp-ecommerce", "erp-scm", "erp-projects",
        "erp-workspace", "erp-bi", "erp-ai"
    ]
}
```

**Error Responses:**

| Code | Condition | Body |
|------|-----------|------|
| 400 | Invalid JSON | `{"error":"invalid json"}` |
| 400 | Missing tenant_id | `{"error":"tenant_id is required"}` |
| 400 | Invalid plan_type | `{"error":"plan_type must be single, bundle, or suite"}` |
| 400 | No SKUs | `{"error":"at least one sku is required"}` |
| 400 | Unknown SKU | `{"error":"unknown sku: xyz"}` |
| 400 | Bundle in single plan | `{"error":"bundle sku is not allowed in single plan: enterprise"}` |

### 1.4 GET /v1/subscriptions/{tenant_id}

**Response (200 OK):**
```json
{
    "tenant_id": "acme-corp",
    "plan_type": "bundle",
    "skus": ["erp-hcm", "erp-crm", "erp-marketing"]
}
```

**Error Response (404):**
```json
{
    "error": "tenant subscription not found"
}
```

### 1.5 GET /v1/entitlements/{tenant_id}

**Response (200 OK):**
```json
{
    "tenant_id": "acme-corp",
    "entitlements": ["erp-hcm", "erp-crm", "erp-marketing"]
}
```

### 1.6 Generic Service Endpoints

**POST /v1/{service} -- Create**

Request: JSON body with `X-Tenant-ID` header.

Response (201 Created):
```json
{
    "item": { /* echoed request body */ },
    "event_topic": "erp.platform.{service}.created"
}
```

**GET /v1/{service} -- List**

Response (200 OK):
```json
{
    "items": [],
    "event_topic": "erp.platform.{service}.listed"
}
```

**GET /v1/{service}/{id} -- Read**

Response (200 OK):
```json
{
    "id": "resource-id",
    "event_topic": "erp.platform.{service}.read"
}
```

**PUT/PATCH /v1/{service}/{id} -- Update**

Response (200 OK):
```json
{
    "id": "resource-id",
    "item": { /* updated body */ },
    "event_topic": "erp.platform.{service}.updated"
}
```

**DELETE /v1/{service}/{id} -- Delete**

Response (200 OK):
```json
{
    "id": "resource-id",
    "event_topic": "erp.platform.{service}.deleted"
}
```

---

## 2. Database Table Definitions

### 2.1 tenants

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | PRIMARY KEY |
| tenant_id | VARCHAR(255) | No | -- | UNIQUE |
| name | VARCHAR(500) | No | -- | -- |
| domain | VARCHAR(255) | Yes | NULL | -- |
| status | VARCHAR(50) | No | 'active' | CHECK IN ('active','suspended','decommissioned') |
| metadata | JSONB | No | '{}' | -- |
| created_at | TIMESTAMPTZ | No | NOW() | -- |
| updated_at | TIMESTAMPTZ | No | NOW() | -- |

### 2.2 subscriptions

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | PRIMARY KEY |
| tenant_id | VARCHAR(255) | No | -- | REFERENCES tenants(tenant_id) |
| plan_type | VARCHAR(50) | No | -- | CHECK IN ('single','bundle','suite') |
| status | VARCHAR(50) | No | 'active' | CHECK IN ('active','suspended','cancelled','grace_period') |
| resolved_skus | TEXT[] | No | -- | -- |
| original_skus | TEXT[] | No | -- | SKUs as submitted (before resolution) |
| created_at | TIMESTAMPTZ | No | NOW() | -- |
| updated_at | TIMESTAMPTZ | No | NOW() | -- |

### 2.3 products

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| sku | VARCHAR(255) | No | -- | PRIMARY KEY |
| name | VARCHAR(500) | No | -- | -- |
| repo | VARCHAR(255) | No | -- | -- |
| type | VARCHAR(50) | No | -- | CHECK IN ('module','bundle') |
| standalone | BOOLEAN | No | true | -- |
| category | VARCHAR(100) | No | -- | -- |
| capabilities | INTEGER | No | 0 | -- |
| includes | TEXT[] | Yes | NULL | Only for bundles |
| catalog_version | VARCHAR(50) | No | -- | -- |

### 2.4 entitlements

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | PRIMARY KEY |
| tenant_id | VARCHAR(255) | No | -- | -- |
| sku | VARCHAR(255) | No | -- | -- |
| active | BOOLEAN | No | true | -- |
| granted_at | TIMESTAMPTZ | No | NOW() | -- |
| expires_at | TIMESTAMPTZ | Yes | NULL | -- |
| -- | -- | -- | -- | UNIQUE(tenant_id, sku) |

### 2.5 audit_logs

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | PRIMARY KEY |
| tenant_id | VARCHAR(255) | No | -- | -- |
| event_topic | VARCHAR(500) | No | -- | -- |
| actor_id | VARCHAR(255) | Yes | NULL | -- |
| payload | JSONB | No | -- | -- |
| correlation_id | UUID | Yes | NULL | -- |
| created_at | TIMESTAMPTZ | No | NOW() | -- |

### 2.6 modules

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| id | VARCHAR(255) | No | -- | PRIMARY KEY |
| name | VARCHAR(500) | No | -- | -- |
| category | VARCHAR(100) | No | -- | -- |
| health_url | VARCHAR(1000) | No | -- | -- |
| status | VARCHAR(50) | No | 'unknown' | CHECK IN ('healthy','degraded','unhealthy','unknown') |
| last_check_at | TIMESTAMPTZ | Yes | NULL | -- |
| registered_at | TIMESTAMPTZ | No | NOW() | -- |

### 2.7 marketplace_listings

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | PRIMARY KEY |
| module_id | VARCHAR(255) | No | -- | -- |
| publisher | VARCHAR(500) | No | -- | -- |
| version | VARCHAR(50) | No | -- | -- |
| description | TEXT | Yes | NULL | -- |
| status | VARCHAR(50) | No | 'draft' | CHECK IN ('draft','published','deprecated') |
| created_at | TIMESTAMPTZ | No | NOW() | -- |

### 2.8 notifications

| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | PRIMARY KEY |
| tenant_id | VARCHAR(255) | No | -- | -- |
| channel | VARCHAR(50) | No | -- | CHECK IN ('email','sms','push','in_app') |
| recipient | VARCHAR(500) | No | -- | -- |
| subject | VARCHAR(500) | No | -- | -- |
| body | TEXT | No | -- | -- |
| status | VARCHAR(50) | No | 'pending' | CHECK IN ('pending','sent','failed') |
| sent_at | TIMESTAMPTZ | Yes | NULL | -- |
| created_at | TIMESTAMPTZ | No | NOW() | -- |

---

## 3. Event Schemas

### 3.1 CloudEvents Envelope

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| specversion | string | Yes | Always "1.0" |
| id | string (UUID) | Yes | Unique event identifier |
| source | string | Yes | Service identifier (e.g., "erp-platform/subscription-hub") |
| type | string | Yes | Event topic (e.g., "erp.platform.subscription.created") |
| datacontenttype | string | Yes | Always "application/json" |
| time | string (ISO8601) | Yes | Event timestamp |
| tenantid | string | Yes | Tenant identifier (custom extension) |
| correlationid | string | No | Request correlation ID |
| data | object | Yes | Event-specific payload |

### 3.2 Event Topics

| Topic | Trigger | Payload Fields |
|-------|---------|---------------|
| erp.platform.activation-wizard.created | Wizard step completed | item, step_number |
| erp.platform.activation-wizard.updated | Wizard state modified | id, item |
| erp.platform.activation-wizard.deleted | Wizard cancelled | id |
| erp.platform.audit.created | Audit entry written | item (audit record) |
| erp.platform.entitlement-engine.created | Entitlement granted | sku, tenant_id |
| erp.platform.entitlement-engine.deleted | Entitlement revoked | id, sku |
| erp.platform.marketplace.created | Module installed | module_id, publisher |
| erp.platform.marketplace.deleted | Module uninstalled | id, module_id |
| erp.platform.module-registry.created | Module registered | module_id, health_url |
| erp.platform.module-registry.deleted | Module deregistered | id |
| erp.platform.notification-hub.created | Notification sent | channel, recipient |
| erp.platform.tenant-provisioner.created | Tenant provisioned | tenant_id, name |
| erp.platform.tenant-provisioner.deleted | Tenant decommissioned | id, tenant_id |
| erp.platform.tenant-provisioner.updated | Tenant config changed | id, item |
| erp.platform.web-hosting.created | Hosting configured | domain, tenant_id |

---

## 4. Configuration Parameters

### 4.1 Environment Variables

| Variable | Service | Type | Default | Description |
|----------|---------|------|---------|-------------|
| ERP_CATALOG_PATH | subscription-hub | string | `../../catalog/products.json` | Path to product catalog JSON |
| PORT | all services | string | `8080` (8091 for sub-hub) | HTTP listen port |
| MODULE_NAME | all services | string | `ERP-Platform` | Module identifier for health checks |
| DATABASE_URL | all services | string | -- | PostgreSQL connection string |
| REDIS_URL | subscription-hub, entitlement-engine | string | `redis://localhost:6379` | Redis connection string |
| NATS_URL | all services | string | `nats://localhost:4222` | NATS connection string |
| LOG_LEVEL | all services | string | `info` | Log verbosity (debug, info, warn, error) |
| LOG_FORMAT | all services | string | `json` | Log format (json, text) |
| HEALTH_CHECK_INTERVAL | module-registry | duration | `30s` | Health poll interval |
| HEALTH_CHECK_TIMEOUT | module-registry | duration | `5s` | Health check timeout |
| HEALTH_CHECK_RETRIES | module-registry | int | `5` | Retries before marking unhealthy |
| CACHE_TTL_CATALOG | subscription-hub | duration | `300s` | Catalog cache TTL |
| CACHE_TTL_ENTITLEMENTS | entitlement-engine | duration | `60s` | Entitlement cache TTL |
| AIDD_MIN_CONFIDENCE | platform | float | `0.70` | Minimum AI confidence threshold |
| AIDD_MED_CONFIDENCE | platform | float | `0.82` | Medium AI confidence threshold |
| AIDD_MAX_BLAST_RADIUS | platform | int | `5000` | Max records for auto-execution |
| AIDD_HIGH_VALUE_USD | platform | float | `100000` | High-value financial threshold |

### 4.2 Docker Compose Configuration

```yaml
# From infra/docker-compose.platform.yml
services:
  subscription-hub:
    ports: ["8091:8091"]
    environment:
      ERP_CATALOG_PATH: /app/catalog/products.json
    volumes:
      - ../catalog/products.json:/app/catalog/products.json:ro

  postgres:
    image: postgres:16
    environment:
      POSTGRES_USER: erp
      POSTGRES_PASSWORD: erp
      POSTGRES_DB: erp_platform
    ports: ["5432:5432"]

  redis:
    image: redis:7
    ports: ["6379:6379"]

  nats:
    image: nats:2.10
    command: ["-js"]
    ports: ["4222:4222"]
```

---

## 5. Resource Requirements

### 5.1 Per Service Instance

| Service | CPU Request | CPU Limit | Memory Request | Memory Limit |
|---------|-----------|----------|---------------|-------------|
| subscription-hub | 250m | 500m | 128Mi | 256Mi |
| tenant-provisioner | 250m | 500m | 128Mi | 256Mi |
| entitlement-engine | 250m | 500m | 128Mi | 256Mi |
| module-registry | 100m | 250m | 64Mi | 128Mi |
| marketplace | 100m | 250m | 64Mi | 128Mi |
| audit-service | 250m | 1000m | 256Mi | 512Mi |
| notification-hub | 100m | 250m | 64Mi | 128Mi |
| web-hosting | 100m | 250m | 64Mi | 128Mi |
| activation-wizard | 100m | 250m | 64Mi | 128Mi |

### 5.2 Data Tier

| Service | Storage | IOPS | Connections |
|---------|---------|------|-------------|
| PostgreSQL Primary | 100GB SSD (expandable) | 3000 | 200 max |
| PostgreSQL Replica | 100GB SSD | 3000 | 200 max |
| Redis | 4GB memory | N/A | 10000 max |
| NATS JetStream | 50GB SSD | 1000 | 10000 max |

---

*For API documentation, see [21-API-Documentation.md](./21-API-Documentation.md). For low-level design, see [13-Low-Level-Design.md](./13-Low-Level-Design.md).*
