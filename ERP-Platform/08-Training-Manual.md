# ERP-Platform Training Manual

> **Document ID:** ERP-PLAT-TM-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Audience:** Platform Administrators, Tenant Administrators, Support Engineers
> **Related Documents:** [07-User-Manual.md](./07-User-Manual.md), [09-Video-Training-Script.md](./09-Video-Training-Script.md)

---

## Training Curriculum Overview

| Module | Title | Duration | Audience | Prerequisites |
|--------|-------|----------|----------|---------------|
| TM-01 | Platform Fundamentals | 2 hours | All | None |
| TM-02 | Admin Console Orientation | 1.5 hours | All | TM-01 |
| TM-03 | Subscription Management | 2 hours | Platform Admin | TM-02 |
| TM-04 | Tenant Provisioning | 2 hours | Platform Admin | TM-03 |
| TM-05 | Module Registry & Health | 1.5 hours | Platform Admin | TM-02 |
| TM-06 | Marketplace Operations | 1.5 hours | Platform Admin, Tenant Admin | TM-05 |
| TM-07 | Audit & Compliance | 2 hours | Platform Admin, Compliance | TM-02 |
| TM-08 | Security Configuration | 2 hours | Platform Admin, Security | TM-04 |
| TM-09 | Troubleshooting & Support | 2 hours | Support Engineers | TM-01 through TM-08 |
| TM-10 | API Integration & Development | 2 hours | Developers | TM-01, TM-03 |

---

## Module TM-01: Platform Fundamentals

### Learning Objectives
- Describe the purpose and scope of ERP-Platform within the 20-module ERP suite
- Identify the nine core platform services and their responsibilities
- Explain the product catalog structure including modules, bundles, and categories
- Articulate the "Business at the Speed of Prompt" value proposition

### Content Outline

**Section 1.1: What is ERP-Platform?** (30 min)
- Unified control plane concept
- 20-module ERP suite overview
- Six module categories: core, business-ops, productivity, intelligence, security, integration, industry-vertical, platform-tools
- AIDD guardrails overview

**Section 1.2: Platform Services Architecture** (30 min)
- Nine services: subscription-hub, tenant-provisioner, entitlement-engine, module-registry, marketplace, audit-service, notification-hub, web-hosting, activation-wizard
- Service interaction patterns
- Event-driven communication via NATS

**Section 1.3: Product Catalog Deep Dive** (30 min)
- 20 standalone modules and their capabilities
- 8 curated bundles (Starter, Professional, Enterprise, Healthcare, Education, Faith, Telecom, Developer)
- Bundle resolution logic
- Catalog versioning

**Section 1.4: Technology Stack Overview** (30 min)
- Go microservices, PostgreSQL, Redis, NATS, Docker, Kubernetes
- Multi-stage Docker builds
- Health check convention (/healthz)

### Hands-On Exercise
1. **Exercise 1.1**: Use `curl` to call the health check endpoint and examine the response structure.
   ```bash
   curl http://localhost:8091/healthz | jq
   ```
2. **Exercise 1.2**: Retrieve the product catalog and identify all modules in the "business-ops" category.
   ```bash
   curl http://localhost:8091/v1/products | jq '.products[] | select(.category == "business-ops")'
   ```
3. **Exercise 1.3**: List all bundles and their included modules.
   ```bash
   curl http://localhost:8091/v1/products | jq '.products[] | select(.type == "bundle") | {sku, includes}'
   ```

### Assessment Questions
1. Name all nine platform services and describe each in one sentence.
2. What is the difference between a "module" and a "bundle" in the product catalog?
3. How many modules does the Enterprise bundle include? Name five.
4. What is the purpose of AIDD guardrails?
5. Which service handles tenant lifecycle management?

---

## Module TM-02: Admin Console Orientation

### Learning Objectives
- Navigate all sections of the ERP-Platform admin console
- Use global search to find tenants, modules, and audit entries
- Customize dashboard widgets and notification preferences
- Access context-sensitive help

### Content Outline

**Section 2.1: Login and Authentication** (20 min)
- ERP-IAM SSO integration
- Multi-factor authentication
- Session management

**Section 2.2: Dashboard Navigation** (30 min)
- Module health indicators
- Subscription summary widgets
- Recent activity feed
- System alert panel

**Section 2.3: Core Navigation Areas** (30 min)
- Subscriptions, Tenants, Modules, Marketplace, Audit Logs, Notifications, Web Hosting, Settings

**Section 2.4: Customization** (10 min)
- Dashboard widget arrangement
- Notification preferences
- Theme and accessibility settings

### Hands-On Exercise
1. Log into the admin console and identify all navigation menu items.
2. Use global search (Ctrl+K) to find a specific tenant.
3. Customize the dashboard to show module health prominently.
4. Configure notification preferences for subscription events.

### Assessment Questions
1. How do you access the global search shortcut?
2. What do green, yellow, and red indicators mean on the module health widget?
3. Where do you configure AIDD guardrail thresholds?
4. How do you export audit logs?

---

## Module TM-03: Subscription Management

### Learning Objectives
- Create subscriptions for all three plan types (single, bundle, suite)
- Explain bundle SKU resolution and deduplication
- Modify existing subscriptions (upgrade/downgrade)
- Query tenant entitlements

### Content Outline

**Section 3.1: Subscription Concepts** (30 min)
- Plan types: single, bundle, suite
- SKU structure and naming conventions
- Bundle resolution flow
- Entitlement derivation from subscriptions

**Section 3.2: Creating Subscriptions** (30 min)
- Single module subscription walkthrough
- Bundle subscription with auto-resolution
- Suite (custom combination) subscription
- Validation rules and error handling

**Section 3.3: Querying and Managing** (30 min)
- Subscription lookup by tenant_id
- Entitlement query for runtime access checks
- Subscription modification workflow
- Cancellation and grace period

**Section 3.4: Advanced Scenarios** (30 min)
- Upgrading from Starter to Professional
- Adding industry vertical modules to existing subscription
- Handling duplicate SKUs across bundles
- API-based subscription management

### Hands-On Exercise
1. Create a "single" plan subscription for erp-crm:
   ```bash
   curl -X POST http://localhost:8091/v1/subscriptions \
     -H "Content-Type: application/json" \
     -d '{"tenant_id":"tenant-001","plan_type":"single","skus":["erp-crm"]}'
   ```
2. Create a "bundle" subscription for the Professional plan:
   ```bash
   curl -X POST http://localhost:8091/v1/subscriptions \
     -H "Content-Type: application/json" \
     -d '{"tenant_id":"tenant-002","plan_type":"bundle","skus":["professional"]}'
   ```
3. Query entitlements for both tenants and compare the module lists.
4. Attempt to create a single plan with a bundle SKU and observe the error.

### Assessment Questions
1. What happens when you include the "enterprise" bundle SKU in a subscription?
2. Why can't a "single" plan include bundle SKUs?
3. How many modules are included in the Starter bundle?
4. What endpoint do you use to check tenant entitlements?
5. How does the deduplication function work?

---

## Module TM-04: Tenant Provisioning

### Learning Objectives
- Provision a complete tenant using the single-call API
- Configure tenant settings during provisioning
- Manage the tenant lifecycle (create, update, decommission)
- Understand the "Business at the Speed of Prompt" provisioning flow

### Hands-On Exercise
1. Create a new tenant via the tenant provisioner API.
2. Verify the tenant was created by reading it back.
3. Update the tenant configuration.
4. Trace the event topics emitted during each operation.

### Assessment Questions
1. What header is required for all tenant provisioner operations?
2. What event topic is emitted when a tenant is provisioned?
3. Describe the full provisioning flow from subscription to active tenant.
4. What is the grace period for tenant decommissioning?

---

## Module TM-05: Module Registry & Health

### Learning Objectives
- Understand module auto-discovery via health checks
- Interpret module health status indicators
- Register and deregister modules in the registry
- Configure health check parameters

### Hands-On Exercise
1. View the current module registry state.
2. Manually trigger a health check for a specific module.
3. Simulate a module failure and observe the registry update.
4. Review health check history for patterns.

### Assessment Questions
1. How frequently does the module registry poll health endpoints?
2. After how many failed retries is a module marked unhealthy?
3. What is the standard health check endpoint for ERP modules?
4. How does the docker-compose topology configure health checks?

---

## Module TM-06: Marketplace Operations

### Learning Objectives
- Browse and search the module marketplace
- Install and configure marketplace modules
- Manage installed module lifecycle
- Understand entitlement checks for marketplace installations

### Hands-On Exercise
1. Browse the marketplace and filter by category.
2. Install a module and verify it appears in the module registry.
3. Check that the health check passes for the newly installed module.
4. Uninstall the module and verify cleanup.

### Assessment Questions
1. What check is performed before a marketplace module can be installed?
2. How do marketplace modules integrate with the module registry?
3. What happens to module data when a marketplace module is uninstalled?

---

## Module TM-07: Audit & Compliance

### Learning Objectives
- Navigate and query the audit log system
- Understand CloudEvents event structure
- Generate compliance reports from audit data
- Configure audit retention policies

### Hands-On Exercise
1. Perform a subscription creation and trace the audit event.
2. Filter audit logs by service, date range, and tenant.
3. Export audit logs in CSV format for compliance review.
4. Map audit log events to SOC 2 control requirements.

### Assessment Questions
1. What is the naming convention for event topics?
2. Name five event topics for the tenant-provisioner service.
3. What is the minimum audit log retention period?
4. How are audit logs protected from tampering?

---

## Module TM-08: Security Configuration

### Learning Objectives
- Configure authentication integration with ERP-IAM
- Set up AIDD guardrail thresholds
- Implement tenant isolation policies
- Manage API keys and secrets

### Hands-On Exercise
1. Review current AIDD guardrail threshold configuration.
2. Simulate an AI action that exceeds the blast radius threshold.
3. Verify the action is blocked and an audit record is created.
4. Review tenant isolation enforcement on API endpoints.

### Assessment Questions
1. What are the five AIDD guardrail thresholds?
2. When is human approval required for AI actions?
3. How does the platform enforce tenant data isolation?
4. What authentication mechanism is used for business endpoints?

---

## Module TM-09: Troubleshooting & Support

### Learning Objectives
- Diagnose common platform issues using logs and health checks
- Resolve subscription and entitlement errors
- Handle tenant provisioning failures
- Escalate issues appropriately

### Hands-On Exercise
1. Diagnose a "missing X-Tenant-ID" error in API requests.
2. Troubleshoot a module that shows "Unhealthy" in the registry.
3. Resolve a subscription creation failure due to unknown SKU.
4. Analyze audit logs to reconstruct an incident timeline.

### Assessment Questions
1. What is the first thing to check when a service returns 400?
2. How do you determine if a module health check is failing?
3. What are the common causes of subscription creation failures?
4. Describe the incident escalation path (P1 through P4).

---

## Module TM-10: API Integration & Development

### Learning Objectives
- Use the ERP-Platform REST APIs for integration
- Authenticate API requests with JWT tokens
- Implement subscription and entitlement checks in client applications
- Handle API errors and implement retry logic

### Hands-On Exercise
1. Use the Postman collection to explore all API endpoints.
2. Create a subscription, query entitlements, and implement an entitlement check in sample code.
3. Subscribe to webhook events for subscription changes.
4. Implement error handling for common API error responses.

### Assessment Questions
1. What authentication is required for the `/v1/products` endpoint?
2. How does bundle resolution affect the response from `POST /v1/subscriptions`?
3. What HTTP status code indicates a successful subscription creation?
4. How would you implement an entitlement check in a client module?

---

*For video training scripts, see [09-Video-Training-Script.md](./09-Video-Training-Script.md). For the user manual, see [07-User-Manual.md](./07-User-Manual.md).*
