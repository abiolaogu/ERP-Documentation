# ERP-Platform Data Dictionary

> **Document ID:** ERP-PLAT-DD-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Related Documents:** [27-Entity-Relationship-Diagram.md](./27-Entity-Relationship-Diagram.md), [13-Low-Level-Design.md](./13-Low-Level-Design.md)

---

## 1. Tenant Domain

### Table: `tenants`

Central tenant registry. Every business entity using the platform has one row.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Internal primary key | `550e8400-e29b-41d4-a716-446655440000` |
| tenant_id | VARCHAR(255) | No | -- | Unique business identifier for the tenant | `acme-corp` |
| name | VARCHAR(500) | No | -- | Organization display name | `Acme Corporation` |
| domain | VARCHAR(255) | Yes | NULL | Custom domain for tenant portal | `acme.example.com` |
| status | VARCHAR(50) | No | `'active'` | Tenant lifecycle status | `active` |
| metadata | JSONB | No | `'{}'` | Flexible key-value metadata | `{"industry":"technology","size":"enterprise"}` |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp | `2026-02-23T10:00:00Z` |
| updated_at | TIMESTAMPTZ | No | NOW() | Last modification timestamp | `2026-02-23T14:30:00Z` |

**Constraints:** status IN ('active', 'suspended', 'decommissioned'). tenant_id is UNIQUE. RLS enabled.

---

### Table: `activation_wizard_states`

Tracks onboarding wizard progress per tenant.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| tenant_id | VARCHAR(255) | No | -- | FK to tenants.tenant_id | `acme-corp` |
| current_step | INTEGER | No | 1 | Current wizard step number | `3` |
| total_steps | INTEGER | No | 5 | Total steps in wizard | `5` |
| step_data | JSONB | No | `'{}'` | Collected data per step | `{"1":{"org_name":"Acme"},"2":{"plan":"professional"}}` |
| status | VARCHAR(50) | No | `'in_progress'` | Wizard completion status | `in_progress` |
| started_at | TIMESTAMPTZ | No | NOW() | Wizard start time | `2026-02-23T10:05:00Z` |
| completed_at | TIMESTAMPTZ | Yes | NULL | Wizard completion time | `2026-02-23T10:15:00Z` |

---

## 2. Subscription Domain

### Table: `subscriptions`

Records the subscription linking a tenant to a set of product modules.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| tenant_id | VARCHAR(255) | No | -- | FK to tenants.tenant_id | `acme-corp` |
| plan_type | VARCHAR(50) | No | -- | Subscription plan type | `bundle` |
| status | VARCHAR(50) | No | `'active'` | Subscription lifecycle status | `active` |
| resolved_skus | TEXT[] | No | -- | Module SKUs after bundle resolution | `{erp-crm,erp-workspace,erp-bi}` |
| original_skus | TEXT[] | No | -- | SKUs as submitted by the user | `{starter}` |
| created_at | TIMESTAMPTZ | No | NOW() | Subscription creation time | `2026-02-23T10:00:00Z` |
| updated_at | TIMESTAMPTZ | No | NOW() | Last modification time | `2026-02-23T10:00:00Z` |

**Constraints:** plan_type IN ('single', 'bundle', 'suite'). status IN ('active', 'suspended', 'cancelled', 'grace_period').

---

### Table: `entitlements`

Runtime access grants linking tenants to product modules.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| tenant_id | VARCHAR(255) | No | -- | FK to tenants.tenant_id | `acme-corp` |
| sku | VARCHAR(255) | No | -- | FK to products.sku | `erp-crm` |
| active | BOOLEAN | No | true | Whether entitlement is currently active | `true` |
| granted_at | TIMESTAMPTZ | No | NOW() | When entitlement was granted | `2026-02-23T10:00:00Z` |
| expires_at | TIMESTAMPTZ | Yes | NULL | Expiration time (null = permanent) | `NULL` |

**Constraints:** UNIQUE(tenant_id, sku).

---

## 3. Product Domain

### Table: `products`

Product catalog defining all modules and bundles.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| sku | VARCHAR(255) | No | -- | Unique product identifier (PK) | `erp-crm` |
| name | VARCHAR(500) | No | -- | Display name | `ERP CRM` |
| repo | VARCHAR(255) | No | -- | Source repository name | `ERP-CRM` |
| type | VARCHAR(50) | No | -- | Product type | `module` |
| standalone | BOOLEAN | No | true | Available as standalone subscription | `true` |
| category | VARCHAR(100) | No | -- | Product category | `business-ops` |
| capabilities | INTEGER | No | 0 | Number of capabilities provided | `12` |
| includes | TEXT[] | Yes | NULL | Constituent module SKUs (bundles only) | `{erp-crm,erp-workspace,erp-bi}` |
| catalog_version | VARCHAR(50) | No | -- | Catalog version | `2026-02-23` |

**Category values:** core, business-ops, productivity, intelligence, security, integration, industry-vertical, platform-tools.

---

## 4. Module Domain

### Table: `modules`

Registry of all deployed ERP modules and their health status.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | VARCHAR(255) | No | -- | Module identifier (PK) | `erp-crm` |
| name | VARCHAR(500) | No | -- | Module display name | `ERP CRM` |
| category | VARCHAR(100) | No | -- | Module category | `business-ops` |
| health_url | VARCHAR(1000) | No | -- | Health check endpoint | `http://erp-crm:8090/healthz` |
| status | VARCHAR(50) | No | `'unknown'` | Current health status | `healthy` |
| last_check_at | TIMESTAMPTZ | Yes | NULL | Last health check time | `2026-02-23T14:30:00Z` |
| registered_at | TIMESTAMPTZ | No | NOW() | Module registration time | `2026-02-23T10:00:00Z` |

---

### Table: `health_checks`

Historical health check results for each module.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| module_id | VARCHAR(255) | No | -- | FK to modules.id | `erp-crm` |
| status | VARCHAR(50) | No | -- | Check result | `healthy` |
| latency_ms | INTEGER | No | -- | Response time in ms | `12` |
| response_body | JSONB | Yes | NULL | Raw health response | `{"status":"healthy"}` |
| checked_at | TIMESTAMPTZ | No | NOW() | Check timestamp | `2026-02-23T14:30:00Z` |

---

## 5. Marketplace Domain

### Table: `marketplace_listings`

Published module listings in the marketplace.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| module_id | VARCHAR(255) | No | -- | FK to modules.id | `analytics-pro` |
| publisher | VARCHAR(500) | No | -- | Publisher organization name | `Acme Analytics Inc.` |
| version | VARCHAR(50) | No | -- | Semantic version | `2.1.0` |
| description | TEXT | Yes | NULL | Module description | `Advanced analytics...` |
| status | VARCHAR(50) | No | `'draft'` | Listing status | `published` |
| install_count | INTEGER | No | 0 | Number of installations | `1450` |
| rating | DECIMAL(3,2) | Yes | NULL | Average rating (0.00-5.00) | `4.35` |
| created_at | TIMESTAMPTZ | No | NOW() | Listing creation time | `2026-02-23T10:00:00Z` |
| updated_at | TIMESTAMPTZ | No | NOW() | Last update time | `2026-02-23T14:30:00Z` |

---

### Table: `marketplace_installations`

Tracks which tenants have installed which marketplace modules.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| listing_id | UUID | No | -- | FK to marketplace_listings.id | UUID |
| tenant_id | VARCHAR(255) | No | -- | FK to tenants.tenant_id | `acme-corp` |
| version | VARCHAR(50) | No | -- | Installed version | `2.1.0` |
| status | VARCHAR(50) | No | `'installed'` | Installation status | `installed` |
| installed_at | TIMESTAMPTZ | No | NOW() | Installation time | `2026-02-23T14:00:00Z` |
| updated_at | TIMESTAMPTZ | No | NOW() | Last update time | `2026-02-23T14:00:00Z` |

---

## 6. Audit Domain

### Table: `audit_logs`

Immutable, append-only audit trail for all platform operations.

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| tenant_id | VARCHAR(255) | No | -- | Tenant identifier | `acme-corp` |
| event_topic | VARCHAR(500) | No | -- | CloudEvents event type | `erp.platform.tenant-provisioner.created` |
| actor_id | VARCHAR(255) | Yes | NULL | User or system ID that performed the action | `user:admin@acme.com` |
| payload | JSONB | No | -- | Full event data | `{"name":"Acme Corp","domain":"acme.com"}` |
| correlation_id | UUID | Yes | NULL | Request correlation for tracing | UUID |
| created_at | TIMESTAMPTZ | No | NOW() | Event timestamp | `2026-02-23T14:30:00Z` |

**Note:** This table is append-only in production. UPDATE and DELETE are prohibited.

---

## 7. Notification Domain

### Table: `notifications`

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| tenant_id | VARCHAR(255) | No | -- | FK to tenants.tenant_id | `acme-corp` |
| channel | VARCHAR(50) | No | -- | Delivery channel | `email` |
| recipient | VARCHAR(500) | No | -- | Target address | `admin@acme.com` |
| subject | VARCHAR(500) | No | -- | Notification subject | `Welcome to ERP-Platform` |
| body | TEXT | No | -- | Notification content | `Your tenant has been provisioned...` |
| status | VARCHAR(50) | No | `'pending'` | Delivery status | `sent` |
| sent_at | TIMESTAMPTZ | Yes | NULL | Actual send time | `2026-02-23T14:30:05Z` |
| created_at | TIMESTAMPTZ | No | NOW() | Creation time | `2026-02-23T14:30:00Z` |

---

## 8. Web Hosting Domain

### Table: `web_hosting_configs`

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| tenant_id | VARCHAR(255) | No | -- | FK to tenants.tenant_id | `acme-corp` |
| domain | VARCHAR(255) | No | -- | Custom domain (UNIQUE) | `portal.acme.com` |
| ssl_status | VARCHAR(50) | No | `'pending'` | SSL certificate status | `valid` |
| cdn_enabled | BOOLEAN | No | false | CDN activation flag | `true` |
| dns_verification | VARCHAR(50) | No | `'pending'` | DNS verification status | `verified` |
| ssl_metadata | JSONB | Yes | NULL | Certificate details | `{"issuer":"Let's Encrypt","expiry":"2026-05-23"}` |
| created_at | TIMESTAMPTZ | No | NOW() | Configuration creation time | `2026-02-23T10:00:00Z` |
| updated_at | TIMESTAMPTZ | No | NOW() | Last update time | `2026-02-23T10:00:00Z` |

---

## 9. Webhook Domain

### Table: `webhook_endpoints`

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| tenant_id | VARCHAR(255) | No | -- | FK to tenants.tenant_id | `acme-corp` |
| url | VARCHAR(2000) | No | -- | Webhook delivery URL | `https://acme.com/webhooks` |
| events | TEXT[] | No | -- | Subscribed event types | `{subscription.created,tenant.provisioned}` |
| secret_hash | VARCHAR(255) | No | -- | Hashed HMAC secret | `$2a$12$...` |
| active | BOOLEAN | No | true | Endpoint active flag | `true` |
| last_delivery_at | TIMESTAMPTZ | Yes | NULL | Last successful delivery | `2026-02-23T14:30:00Z` |
| created_at | TIMESTAMPTZ | No | NOW() | Registration time | `2026-02-23T10:00:00Z` |

### Table: `webhook_deliveries`

| Column | Type | Nullable | Default | Description | Example |
|--------|------|----------|---------|-------------|---------|
| id | UUID | No | gen_random_uuid() | Primary key | UUID |
| endpoint_id | UUID | No | -- | FK to webhook_endpoints.id | UUID |
| event_type | VARCHAR(255) | No | -- | Event type delivered | `subscription.created` |
| event_id | UUID | No | -- | CloudEvent ID | UUID |
| attempt | INTEGER | No | 1 | Delivery attempt number | `1` |
| response_code | INTEGER | Yes | NULL | HTTP response code | `200` |
| status | VARCHAR(50) | No | `'pending'` | Delivery status | `success` |
| delivered_at | TIMESTAMPTZ | No | NOW() | Delivery timestamp | `2026-02-23T14:30:00Z` |

---

*For entity-relationship diagrams, see [27-Entity-Relationship-Diagram.md](./27-Entity-Relationship-Diagram.md). For low-level design, see [13-Low-Level-Design.md](./13-Low-Level-Design.md).*
