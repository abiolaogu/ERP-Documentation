# ERP-Platform Use Cases

> **Document ID:** ERP-PLAT-UC-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [05-Product-Requirements-Document.md](./05-Product-Requirements-Document.md), [11-Workflow-Diagrams.md](./11-Workflow-Diagrams.md)

---

## UC-001: View Product Catalog

| Field | Value |
|-------|-------|
| **UC-ID** | UC-001 |
| **Title** | View Product Catalog |
| **Actor** | Platform Admin, Tenant Admin, Developer |
| **Preconditions** | Service is running; catalog loaded successfully |
| **Postconditions** | Actor receives complete product listing |
| **Business Rules** | Catalog is read-only via API; modifications require code change |

**Main Flow:**
1. Actor sends GET /v1/products to the Subscription Hub.
2. System loads the versioned catalog from memory.
3. System returns JSON response with all products including SKU, name, repo, type, category, capabilities, standalone flag, and includes array (for bundles).
4. Actor receives 200 OK with the catalog payload.

**Alternative Flows:**
- **AF-1**: Service not running -- Actor receives connection refused. Resolution: check service health.
- **AF-2**: GET with wrong method -- Actor sends POST; system returns 405 Method Not Allowed.

---

## UC-002: Create Single Module Subscription

| Field | Value |
|-------|-------|
| **UC-ID** | UC-002 |
| **Title** | Create Single Module Subscription |
| **Actor** | Platform Admin |
| **Preconditions** | Valid tenant_id; target SKU exists in catalog as module type |
| **Postconditions** | Subscription record created; entitlements seeded |
| **Business Rules** | BR-001 (tenant_id required), BR-002 (valid plan_type), BR-003 (no bundles in single), BR-006 (min one SKU) |

**Main Flow:**
1. Actor sends POST /v1/subscriptions with `{"tenant_id":"t-001","plan_type":"single","skus":["erp-crm"]}`.
2. System validates tenant_id is non-empty.
3. System validates plan_type is "single".
4. System validates each SKU exists in catalog.
5. System confirms no bundle SKUs are present (single plan rule).
6. System creates SubscriptionRecord with resolved SKUs.
7. System stores record under tenant_id.
8. System returns 201 Created with the subscription record.

**Alternative Flows:**
- **AF-1**: Empty tenant_id -- System returns 400 `{"error":"tenant_id is required"}`.
- **AF-2**: Invalid plan_type -- System returns 400 `{"error":"plan_type must be single, bundle, or suite"}`.
- **AF-3**: Unknown SKU -- System returns 400 `{"error":"unknown sku: erp-nonexistent"}`.
- **AF-4**: Bundle SKU in single plan -- System returns 400 `{"error":"bundle sku is not allowed in single plan: enterprise"}`.
- **AF-5**: Empty SKU list -- System returns 400 `{"error":"at least one sku is required"}`.

---

## UC-003: Create Bundle Subscription

| Field | Value |
|-------|-------|
| **UC-ID** | UC-003 |
| **Title** | Create Bundle Subscription |
| **Actor** | Platform Admin |
| **Preconditions** | Valid tenant_id; bundle SKU exists in catalog |
| **Postconditions** | Subscription created with resolved module SKUs |
| **Business Rules** | BR-004 (bundle resolution at creation), BR-005 (deduplication) |

**Main Flow:**
1. Actor sends POST /v1/subscriptions with `{"tenant_id":"t-002","plan_type":"bundle","skus":["professional"]}`.
2. System validates input fields.
3. System identifies "professional" as a bundle type product.
4. System expands bundle to its includes list: erp-hcm, erp-crm, erp-marketing, erp-finance, erp-commerce, erp-ecommerce, erp-scm, erp-projects, erp-workspace, erp-bi, erp-ai.
5. System deduplicates the resolved SKU list.
6. System creates SubscriptionRecord with 11 resolved module SKUs.
7. System returns 201 Created.

**Alternative Flows:**
- **AF-1**: Multiple bundles with overlap -- System resolves both, deduplicates resulting list.

---

## UC-004: Query Tenant Subscription

| Field | Value |
|-------|-------|
| **UC-ID** | UC-004 |
| **Title** | Query Tenant Subscription |
| **Actor** | Platform Admin, Tenant Admin, System |
| **Preconditions** | Subscription exists for tenant_id |
| **Postconditions** | Subscription record returned |

**Main Flow:**
1. Actor sends GET /v1/subscriptions/{tenant_id}.
2. System looks up tenant_id in the store (RLock for concurrent reads).
3. System finds the subscription record.
4. System returns 200 OK with `{"tenant_id":"...","plan_type":"...","skus":[...]}`.

**Alternative Flows:**
- **AF-1**: Unknown tenant_id -- System returns 404 `{"error":"tenant subscription not found"}`.
- **AF-2**: Wrong HTTP method -- System returns 405 Method Not Allowed.

---

## UC-005: Query Tenant Entitlements

| Field | Value |
|-------|-------|
| **UC-ID** | UC-005 |
| **Title** | Query Tenant Entitlements |
| **Actor** | ERP Module, API Gateway, System |
| **Preconditions** | Subscription exists for tenant_id |
| **Postconditions** | Entitlement list returned for runtime access check |

**Main Flow:**
1. Actor sends GET /v1/entitlements/{tenant_id}.
2. System looks up tenant subscription.
3. System returns 200 OK with `{"tenant_id":"...","entitlements":["erp-crm","erp-bi",...]}`.

**Alternative Flows:**
- **AF-1**: No subscription -- System returns 404.

---

## UC-006: Provision New Tenant

| Field | Value |
|-------|-------|
| **UC-ID** | UC-006 |
| **Title** | Provision New Tenant |
| **Actor** | Platform Admin |
| **Preconditions** | Valid X-Tenant-ID header; subscription exists |
| **Postconditions** | Tenant provisioned; entitlements seeded; events emitted |
| **Business Rules** | BR-008 (X-Tenant-ID required), BR-011 (provisioning triggers entitlement seeding) |

**Main Flow:**
1. Actor sends POST /v1/tenant-provisioner with X-Tenant-ID header and tenant configuration body.
2. System validates X-Tenant-ID header presence.
3. System creates tenant record.
4. System emits `erp.platform.tenant-provisioner.created` event.
5. System returns 201 Created with tenant record and event topic.

**Alternative Flows:**
- **AF-1**: Missing X-Tenant-ID -- System returns 400 `{"error":"missing X-Tenant-ID"}`.

---

## UC-007: Update Tenant Configuration

| Field | Value |
|-------|-------|
| **UC-ID** | UC-007 |
| **Title** | Update Tenant Configuration |
| **Actor** | Platform Admin, Tenant Admin |
| **Preconditions** | Tenant exists; valid X-Tenant-ID; valid tenant resource ID |
| **Postconditions** | Tenant configuration updated; event emitted |

**Main Flow:**
1. Actor sends PUT /v1/tenant-provisioner/{id} with X-Tenant-ID header and updated configuration.
2. System validates headers and path parameters.
3. System updates tenant record.
4. System emits `erp.platform.tenant-provisioner.updated` event.
5. System returns 200 OK.

**Alternative Flows:**
- **AF-1**: Missing X-Tenant-ID -- 400 error.
- **AF-2**: Missing resource ID -- 400 error.

---

## UC-008: Decommission Tenant

| Field | Value |
|-------|-------|
| **UC-ID** | UC-008 |
| **Title** | Decommission Tenant |
| **Actor** | Platform Admin |
| **Preconditions** | Tenant exists; actor has platform-admin role |
| **Postconditions** | Tenant marked for decommissioning; grace period started; event emitted |
| **Business Rules** | BR-010 (30-day grace period) |

**Main Flow:**
1. Actor sends DELETE /v1/tenant-provisioner/{id} with X-Tenant-ID header.
2. System validates headers and authorization.
3. System marks tenant for decommissioning (30-day grace period).
4. System emits `erp.platform.tenant-provisioner.deleted` event.
5. System returns 200 OK.

---

## UC-009: Install Marketplace Module

| Field | Value |
|-------|-------|
| **UC-ID** | UC-009 |
| **Title** | Install Marketplace Module |
| **Actor** | Platform Admin, Tenant Admin |
| **Preconditions** | Module listed in marketplace; tenant has valid subscription |
| **Postconditions** | Module installed; registered in module registry; health check initiated |

**Main Flow:**
1. Actor browses marketplace listings via GET /v1/marketplace.
2. Actor selects a module and sends POST /v1/marketplace with installation request.
3. System verifies tenant entitlements via entitlement engine.
4. System installs the module.
5. System registers module in module-registry.
6. System initiates health check.
7. System emits `erp.platform.marketplace.created` event.
8. System returns 201 Created.

**Alternative Flows:**
- **AF-1**: Tenant not entitled -- System returns 403 with entitlement error.
- **AF-2**: Health check fails -- System rolls back installation.

---

## UC-010: Create Audit Log Entry

| Field | Value |
|-------|-------|
| **UC-ID** | UC-010 |
| **Title** | Create Audit Log Entry |
| **Actor** | System (any platform service) |
| **Preconditions** | State-changing operation occurred |
| **Postconditions** | Immutable audit record created |
| **Business Rules** | BR-017 (all CRUD emits events), BR-018 (append-only) |

**Main Flow:**
1. A platform service performs a CRUD operation.
2. Service publishes CloudEvent to NATS on topic `erp.platform.<service>.<action>`.
3. Audit service consumes the event.
4. Audit service persists the record (append-only).
5. Record includes tenant_id, actor, action, payload, timestamp.

---

## UC-011: Check Service Health

| Field | Value |
|-------|-------|
| **UC-ID** | UC-011 |
| **Title** | Check Service Health |
| **Actor** | Module Registry, Kubernetes, Operations Team |
| **Preconditions** | Service is deployed |
| **Postconditions** | Health status returned |

**Main Flow:**
1. Actor sends GET /healthz to service.
2. Service responds with `{"status":"healthy","module":"ERP-Platform","service":"<name>"}`.
3. Actor records health status.

**Alternative Flows:**
- **AF-1**: Service unreachable -- Actor marks as unhealthy after 5 retries.

---

## UC-012: Send Platform Notification

| Field | Value |
|-------|-------|
| **UC-ID** | UC-012 |
| **Title** | Send Platform Notification |
| **Actor** | System, Platform Admin |
| **Preconditions** | Notification hub is running; recipient exists |
| **Postconditions** | Notification dispatched via configured channels |

**Main Flow:**
1. System or admin creates notification via POST /v1/notification-hub.
2. Notification hub validates X-Tenant-ID.
3. Hub routes to configured channels (email, SMS, push, in-app).
4. Hub emits `erp.platform.notification-hub.created` event.
5. Hub returns 201 Created.

---

## UC-013: Configure Web Hosting

| Field | Value |
|-------|-------|
| **UC-ID** | UC-013 |
| **Title** | Configure Web Hosting |
| **Actor** | Platform Admin |
| **Preconditions** | Tenant provisioned; domain available |
| **Postconditions** | Domain configured; SSL provisioned; CDN activated |

**Main Flow:**
1. Actor sends POST /v1/web-hosting with domain configuration.
2. System validates X-Tenant-ID and domain availability.
3. System provisions DNS records.
4. System requests SSL certificate from Let's Encrypt.
5. System configures CDN.
6. System emits `erp.platform.web-hosting.created` event.
7. System returns 201 Created.

---

## UC-014: Run Activation Wizard

| Field | Value |
|-------|-------|
| **UC-ID** | UC-014 |
| **Title** | Run Activation Wizard |
| **Actor** | Tenant Admin (first login) |
| **Preconditions** | Tenant provisioned; admin has first-login flag |
| **Postconditions** | Tenant configuration completed; wizard state persisted |

**Main Flow:**
1. Tenant admin logs in for the first time.
2. System redirects to activation wizard.
3. Admin completes organization profile step.
4. Admin reviews module selection step (pre-populated from subscription).
5. Admin configures initial users step.
6. Admin sets notification preferences.
7. System saves wizard state via POST /v1/activation-wizard.
8. System marks activation as complete.
9. System emits `erp.platform.activation-wizard.created` event.

---

## UC-015: AIDD Guardrail Evaluation

| Field | Value |
|-------|-------|
| **UC-ID** | UC-015 |
| **Title** | AIDD Guardrail Evaluation |
| **Actor** | AI Agent (ERP-AI, ERP-Assistant) |
| **Preconditions** | AI action request received; AIDD policy engine configured |
| **Postconditions** | Action approved, queued for human review, or blocked |
| **Business Rules** | BR-012 through BR-016 |

**Main Flow:**
1. AI agent submits action request to platform.
2. Policy engine evaluates confidence score against threshold (0.70 minimum).
3. Policy engine checks blast radius (max 5,000 records).
4. Policy engine checks financial value ($100,000 threshold).
5. Action is auto-approved (confidence >= 0.82, within blast radius).
6. Action is executed.
7. Decision and result are logged to audit service.

**Alternative Flows:**
- **AF-1**: Confidence < 0.70 -- Action is blocked; audit record created with "blocked" status.
- **AF-2**: Confidence 0.70-0.82 -- Action queued for human approval.
- **AF-3**: Blast radius > 5,000 -- Action queued for human approval regardless of confidence.
- **AF-4**: Financial value > $100,000 -- Action queued for human approval.

---

## UC-016: Register Module in Registry

| Field | Value |
|-------|-------|
| **UC-ID** | UC-016 |
| **Title** | Register Module in Registry |
| **Actor** | Platform Admin, System (auto-discovery) |
| **Preconditions** | Module deployed with /healthz endpoint |
| **Postconditions** | Module registered; health polling initiated |

**Main Flow:**
1. Actor sends POST /v1/module-registry with module configuration (name, health URL, category).
2. System validates X-Tenant-ID.
3. System registers the module.
4. System initiates first health check.
5. System emits `erp.platform.module-registry.created` event.
6. System begins periodic health polling (30-second interval).
7. System returns 201 Created.

---

*For workflow diagrams, see [11-Workflow-Diagrams.md](./11-Workflow-Diagrams.md). For product requirements, see [05-Product-Requirements-Document.md](./05-Product-Requirements-Document.md).*
