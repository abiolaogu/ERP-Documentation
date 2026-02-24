# ERP-Platform Video Training Scripts

> **Document ID:** ERP-PLAT-VTS-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Audience:** Platform Administrators, Tenant Administrators
> **Related Documents:** [08-Training-Manual.md](./08-Training-Manual.md), [07-User-Manual.md](./07-User-Manual.md)

---

## Video Series Overview

| # | Title | Duration | Key Topics |
|---|-------|----------|------------|
| V1 | Introduction to ERP-Platform | 8 min | Platform overview, 20-module suite, value proposition |
| V2 | Tenant Setup & Provisioning | 10 min | Creating tenants, activation wizard, first login |
| V3 | Subscription Management | 8 min | Plans, bundles, creating/modifying subscriptions |
| V4 | Module Marketplace | 7 min | Browsing, installing, managing modules |
| V5 | Audit & Compliance | 9 min | Audit logs, CloudEvents, compliance reports |
| V6 | User Management | 7 min | Roles, permissions, tenant isolation |
| V7 | Integration Configuration | 8 min | API keys, webhooks, entitlement checks |
| V8 | Advanced Administration | 10 min | AIDD guardrails, health monitoring, troubleshooting |

---

## Video V1: Introduction to ERP-Platform

**Duration:** 8 minutes
**Audience:** All users

### Script

**[0:00 - 0:30] Opening**

*Screen: ERP-Platform logo animation, tagline "Business at the Speed of Prompt"*

**Narrator:** "Welcome to ERP-Platform, the unified control plane for your entire ERP suite. In this video, we will explore what ERP-Platform does, why it exists, and how it transforms the way enterprises manage their business technology stack."

**[0:30 - 2:00] The Problem**

*Screen: Split view showing 20 different admin consoles with arrows showing confusion*

**Narrator:** "Traditional ERP suites force administrators to juggle multiple admin consoles -- one for CRM, another for Finance, yet another for HCM. Provisioning a new tenant means coordinating across these systems, often taking weeks. Audit trails are scattered, making compliance a manual, expensive process."

*Screen: Animated timeline showing "2-6 weeks" for traditional provisioning*

**Narrator:** "ERP-Platform solves these challenges by providing a single control plane that manages your entire 20-module ERP suite."

**[2:00 - 4:00] The Solution**

*Screen: ERP-Platform dashboard overview with module health indicators*

**Narrator:** "ERP-Platform consists of nine specialized services working together."

*Screen: Animated diagram showing the nine services appearing one by one*

**Narrator:** "The Subscription Hub manages your product catalog and subscriptions. The Tenant Provisioner handles single-call provisioning -- from subscription to active tenant in minutes, not weeks. The Entitlement Engine ensures each tenant can only access modules they are subscribed to."

*Screen: Module Registry showing green health indicators*

**Narrator:** "The Module Registry automatically discovers and monitors all twenty modules through health checks. The Marketplace enables third-party module installations. The Audit Service provides an immutable, platform-wide audit trail."

**[4:00 - 5:30] The Product Catalog**

*Screen: Product catalog visualization showing categories and bundles*

**Narrator:** "The product catalog is the heart of ERP-Platform. It defines twenty standalone modules across eight categories, and eight curated bundles including Starter, Professional, and Enterprise, plus industry-specific bundles for Healthcare, Education, Faith, and Telecom."

*Screen: Bundle expansion animation showing Enterprise expanding to 20 modules*

**Narrator:** "When you select a bundle, the system automatically resolves it to the individual modules, deduplicates, and seeds entitlements -- all in a single API call."

**[5:30 - 7:00] AIDD Guardrails**

*Screen: AIDD guardrails configuration panel*

**Narrator:** "ERP-Platform enforces AIDD guardrails -- AI-Integrated Development Discipline -- on all automated operations. Every AI action passes through a policy engine that checks confidence thresholds, blast radius limits, and high-value gates. Actions below seventy percent confidence are blocked. Actions affecting more than five thousand records require human approval."

*Screen: Audit log showing an AIDD decision record*

**Narrator:** "Every guardrail decision is logged to the immutable audit trail for compliance."

**[7:00 - 8:00] Closing**

*Screen: Quick demo montage of admin console actions*

**Narrator:** "In the following videos, we will walk through each capability in detail -- from tenant provisioning to subscription management, marketplace operations, and advanced administration. Let us get started."

*Screen: End card with links to next videos*

---

## Video V2: Tenant Setup & Provisioning

**Duration:** 10 minutes

### Script

**[0:00 - 0:30] Opening**

*Screen: Tenant management page in admin console*

**Narrator:** "In this video, we will walk through the complete tenant provisioning process -- from creating a subscription to having a fully active tenant with all modules configured."

**[0:30 - 2:30] Creating a Subscription**

*Screen: Navigate to Subscriptions > New Subscription*

**Narrator:** "First, we create a subscription. Navigate to the Subscriptions page and click 'New Subscription'."

*Screen action: Fill in tenant_id as "acme-corp", select plan_type "bundle", choose "professional"*

**Narrator:** "Enter the tenant identifier -- 'acme-corp'. Select the plan type 'bundle' and choose the 'Professional' bundle. Notice how the system immediately shows the eleven modules that will be included."

*Screen action: Click "Create Subscription", show the 201 response with resolved SKUs*

**Narrator:** "Click 'Create Subscription'. The system resolves the Professional bundle to its constituent modules: HCM, CRM, Marketing, Finance, Commerce, eCommerce, SCM, Projects, Workspace, BI, and AI."

**[2:30 - 5:00] Provisioning the Tenant**

*Screen action: Navigate to Tenants > New Tenant*

**Narrator:** "Now navigate to Tenants and click 'New Tenant'. Enter the tenant name 'Acme Corporation', select the subscription we just created, and specify the initial administrator email."

*Screen action: Fill in form fields, click "Provision Tenant"*

**Narrator:** "Click 'Provision Tenant'. Watch the progress indicator as the system performs each step."

*Screen: Animated progress bar showing steps: Create record, Seed entitlements, Configure hosting, Send notification*

**Narrator:** "The provisioner creates the tenant record, seeds entitlements for all eleven Professional modules, configures web hosting, and sends a welcome notification to the initial administrator. Total time: under two minutes."

**[5:00 - 7:00] Activation Wizard**

*Screen: Switch to new tenant admin perspective, show activation wizard*

**Narrator:** "When the initial administrator logs in for the first time, the Activation Wizard launches automatically."

*Screen action: Walk through wizard steps*

**Narrator:** "The wizard guides through company profile setup, module configuration preferences, and initial user invitations. Each step is optional and can be completed later."

**[7:00 - 8:30] Verifying the Tenant**

*Screen action: Return to platform admin console, show tenant detail page*

**Narrator:** "Back in the platform admin console, we can verify the tenant is fully provisioned. The tenant detail page shows the active subscription, entitled modules, and provisioning status."

*Screen action: Click on each module to verify health status*

**Narrator:** "Each module shows a green health indicator, confirming the tenant is operational."

**[8:30 - 9:30] Using the API**

*Screen: Terminal with curl commands*

**Narrator:** "For automation, the same process works via API."

*Screen action: Show curl commands for subscription creation and tenant provisioning*

```
curl -X POST http://localhost:8091/v1/subscriptions \
  -H "Content-Type: application/json" \
  -d '{"tenant_id":"acme-corp","plan_type":"bundle","skus":["professional"]}'
```

**Narrator:** "A single curl command creates the subscription. A second call provisions the tenant. This is 'Business at the Speed of Prompt.'"

**[9:30 - 10:00] Closing**

**Narrator:** "You have now seen the complete tenant provisioning flow. In the next video, we will explore subscription management in more detail."

---

## Video V3: Subscription Management

**Duration:** 8 minutes

### Script

**[0:00 - 0:20] Opening**

**Narrator:** "In this video, we will cover subscription management -- creating, querying, modifying, and understanding the three plan types."

**[0:20 - 2:30] Plan Types**

*Screen: Diagram showing Single, Bundle, and Suite plan types*

**Narrator:** "ERP-Platform supports three subscription plan types. 'Single' plans grant access to individual modules. 'Bundle' plans use curated groupings like Starter with three modules or Enterprise with all twenty. 'Suite' plans allow custom module combinations."

*Screen action: Demo creating each plan type*

**[2:30 - 4:30] Bundle Resolution**

*Screen: Animation showing bundle SKU expanding to module list*

**Narrator:** "When you subscribe to a bundle, the system resolves it automatically. The Starter bundle with SKU 'starter' expands to erp-crm, erp-workspace, and erp-bi. The Enterprise bundle expands to all twenty modules."

*Screen: Code showing the dedupe function*

**Narrator:** "If multiple bundles contain overlapping modules, the deduplication logic ensures each module appears only once in the resolved list."

**[4:30 - 6:30] Querying and Entitlements**

*Screen action: Query subscription and entitlements endpoints*

**Narrator:** "Query any tenant's subscription at GET /v1/subscriptions/{tenant_id}. For runtime access checks, use GET /v1/entitlements/{tenant_id}, which returns just the entitled module SKUs."

**[6:30 - 8:00] Upgrades and Cancellations**

*Screen action: Demo upgrading from Starter to Professional*

**Narrator:** "To upgrade, modify the subscription to include the new bundle. The system calculates the delta, adds new entitlements, and activates the additional modules."

**Narrator:** "Cancellation triggers a thirty-day grace period during which the subscription can be reactivated."

---

## Video V4: Module Marketplace

**Duration:** 7 minutes

### Script

**[0:00 - 1:30] Overview**
**Narrator:** "The ERP-Platform Marketplace enables discovery and installation of third-party modules that extend your ERP capabilities."

**[1:30 - 3:30] Browsing and Searching**
*Screen action: Navigate marketplace, filter by category, search for specific module*

**[3:30 - 5:30] Installation Flow**
*Screen action: Select module, review details, click Install, watch progress, verify in module registry*

**[5:30 - 7:00] Management and Uninstallation**
*Screen action: View installed modules, check for updates, uninstall a module*

---

## Video V5: Audit & Compliance

**Duration:** 9 minutes

### Script

**[0:00 - 2:00] Why Audit Matters**
**Narrator:** "Every operation in ERP-Platform generates an immutable audit record. This video shows you how to leverage the audit system for compliance, troubleshooting, and governance."

**[2:00 - 4:30] Event Topics**
*Screen: List of 45+ event topics organized by service*
**Narrator:** "Events follow the convention erp.platform.{service}.{action}. For example, when a tenant is provisioned, the topic 'erp.platform.tenant-provisioner.created' is emitted."

**[4:30 - 6:30] Querying Audit Logs**
*Screen action: Filter by date, service, tenant, event type; expand payload details*

**[6:30 - 8:00] Compliance Reports**
*Screen action: Export filtered audit data for SOC 2 evidence*

**[8:00 - 9:00] AIDD Guardrail Audit**
*Screen action: Filter for AIDD-related audit events, review approval/rejection decisions*

---

## Video V6: User Management

**Duration:** 7 minutes

### Script

**[0:00 - 1:30] Role-Based Access**
**Narrator:** "ERP-Platform uses role-based access control with four roles: platform-admin, tenant-admin, module-admin, and viewer."

**[1:30 - 3:30] Managing Users**
*Screen action: Add user, assign role, assign tenant scope*

**[3:30 - 5:30] Tenant Isolation**
*Screen action: Demonstrate that tenant-admin can only see their tenant's data*

**[5:30 - 7:00] SSO Integration**
*Screen action: Show ERP-IAM SSO configuration*

---

## Video V7: Integration Configuration

**Duration:** 8 minutes

### Script

**[0:00 - 2:00] API Overview**
*Screen: API endpoint list with descriptions*

**[2:00 - 4:30] Authentication**
*Screen action: Obtain JWT token, use in API calls*

**[4:30 - 6:30] Webhook Setup**
*Screen action: Register webhook endpoint, test with subscription.created event*

**[6:30 - 8:00] Entitlement Checks in Client Code**
*Screen: Code example showing entitlement verification*

---

## Video V8: Advanced Administration

**Duration:** 10 minutes

### Script

**[0:00 - 2:30] AIDD Guardrails Configuration**
*Screen action: Review thresholds, understand enforcement flow*

**[2:30 - 5:00] Module Health Monitoring**
*Screen action: Module registry dashboard, health check history, alert configuration*

**[5:00 - 7:30] Troubleshooting Common Issues**
*Screen action: Diagnose missing X-Tenant-ID, unknown SKU errors, unhealthy modules*

**[7:30 - 9:30] Performance and Scaling**
*Screen action: Review Kubernetes HPA settings, database connection pools, Redis cache*

**[9:30 - 10:00] Closing**
**Narrator:** "You are now equipped to manage ERP-Platform at an advanced level. For more details, consult the comprehensive documentation suite and the operational runbooks."

---

*For the training manual, see [08-Training-Manual.md](./08-Training-Manual.md). For the user manual, see [07-User-Manual.md](./07-User-Manual.md).*
