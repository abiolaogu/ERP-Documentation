# ERP-Platform User Manual

> **Document ID:** ERP-PLAT-UM-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Audience:** Platform Administrators, Tenant Administrators
> **Related Documents:** [08-Training-Manual.md](./08-Training-Manual.md), [10-Use-Cases.md](./10-Use-Cases.md)

---

## 1. Getting Started

### 1.1 System Requirements

- Modern web browser (Chrome 120+, Firefox 120+, Safari 17+, Edge 120+)
- Network access to the ERP-Platform admin console
- Valid user credentials from ERP-IAM

### 1.2 Accessing the Admin Console

1. Navigate to `https://admin.erp-platform.example.com` in your browser.
2. You will be redirected to the ERP-IAM login page.
3. Enter your credentials (email + password) or use SSO.
4. Upon successful authentication, you will see the Platform Dashboard.

> **[Screenshot Placeholder: Login page with ERP-IAM integration]**

### 1.3 First-Time Setup

If this is your first login as a Platform Administrator:

1. The **Activation Wizard** will launch automatically.
2. Follow the guided steps to complete initial configuration:
   - Organization profile (name, domain, industry)
   - Module selection (choose from available bundles or individual modules)
   - Initial user setup (invite first set of administrators)
   - Notification preferences (email, SMS, in-app)
3. Click "Complete Setup" to finalize your platform configuration.

> **[Screenshot Placeholder: Activation Wizard -- Step 1: Organization Profile]**

---

## 2. Navigating the Admin Console

### 2.1 Dashboard Overview

The main dashboard provides at-a-glance visibility into your platform status:

- **Module Health**: Green/yellow/red status indicators for all registered modules
- **Active Subscriptions**: Count of active tenant subscriptions by plan type
- **Recent Activity**: Latest audit log entries
- **System Alerts**: Critical notifications requiring attention

> **[Screenshot Placeholder: Main dashboard with health indicators and activity feed]**

### 2.2 Navigation Structure

| Menu Item | Description | Access Level |
|-----------|-------------|-------------|
| Dashboard | Platform health overview | All admins |
| Subscriptions | Subscription management | Platform Admin |
| Tenants | Tenant provisioning and management | Platform Admin |
| Modules | Module registry and health status | Platform Admin |
| Marketplace | Third-party module browsing and installation | Platform Admin, Tenant Admin |
| Audit Logs | Platform-wide audit trail | Platform Admin, Compliance |
| Notifications | Notification center and preferences | All admins |
| Web Hosting | Domain, SSL, and CDN management | Platform Admin |
| Settings | System configuration and AIDD guardrails | Platform Admin |
| API Keys | API key management for integrations | Platform Admin |

### 2.3 Common Navigation Patterns

- **Breadcrumbs**: Displayed at the top of each page for navigation context.
- **Search**: Global search bar (Ctrl/Cmd + K) for finding tenants, modules, and audit entries.
- **Quick Actions**: Right-side panel with frequently used actions.
- **Help**: Context-sensitive help icon (?) on every page.

---

## 3. Managing Subscriptions

### 3.1 Viewing Current Subscriptions

1. Navigate to **Subscriptions** in the left sidebar.
2. The subscription list shows all active subscriptions with:
   - Tenant ID
   - Plan type (Single, Bundle, Suite)
   - Subscribed modules (resolved SKU list)
   - Creation date
   - Status (Active, Pending, Cancelled)

> **[Screenshot Placeholder: Subscription list view with filters]**

### 3.2 Creating a New Subscription

1. Click **"+ New Subscription"** in the top-right corner.
2. Fill in the subscription form:
   - **Tenant ID**: Select existing tenant or enter new tenant identifier.
   - **Plan Type**: Choose from:
     - **Single**: Select individual modules (e.g., erp-crm only)
     - **Bundle**: Choose a pre-defined bundle:
       - Starter (CRM + Workspace + BI)
       - Professional (11 modules)
       - Enterprise (all 20 modules)
       - Healthcare Suite (5 modules)
       - Education Suite (5 modules)
       - Faith Suite (4 modules)
       - Telecom Suite (5 modules)
       - Developer Suite (4 modules)
     - **Suite**: Custom combination of modules
   - **SKUs**: Select or auto-populate based on plan type.
3. Review the resolved module list (bundles are automatically expanded).
4. Click **"Create Subscription"**.

> **[Screenshot Placeholder: New subscription form with bundle selector]**

### 3.3 Modifying a Subscription

1. Navigate to the subscription you wish to modify.
2. Click **"Edit"** to enter modification mode.
3. Add or remove modules as needed.
4. Review changes and click **"Update Subscription"**.
5. New entitlements will be applied immediately; removed entitlements follow a 30-day grace period.

### 3.4 Cancelling a Subscription

1. Navigate to the subscription.
2. Click **"Cancel Subscription"**.
3. Confirm the cancellation. A 30-day grace period applies.
4. During the grace period, the subscription can be reactivated.

---

## 4. Managing Tenants

### 4.1 Viewing Tenants

Navigate to **Tenants** to see all provisioned tenants with:
- Tenant name and ID
- Active subscription plan
- Number of active modules
- Provisioning status
- Creation date

> **[Screenshot Placeholder: Tenant list with status indicators]**

### 4.2 Provisioning a New Tenant

1. Click **"+ New Tenant"**.
2. Complete the provisioning form:
   - **Tenant Name**: Display name for the organization
   - **Tenant ID**: Unique identifier (auto-generated or custom)
   - **Domain**: Custom domain for tenant portal (optional)
   - **Subscription**: Select or create a subscription
   - **Initial Admin**: Email address for the first tenant administrator
3. Click **"Provision Tenant"**.
4. The system will:
   - Create the tenant record
   - Seed entitlements based on subscription
   - Configure web hosting (if domain specified)
   - Send welcome notification to initial admin
   - Launch the activation wizard for the new admin

> **[Screenshot Placeholder: Tenant provisioning form]**

### 4.3 Managing Tenant Configuration

1. Click on a tenant name to open the detail view.
2. Available configuration tabs:
   - **Overview**: General tenant information and status
   - **Subscription**: Current plan and entitled modules
   - **Users**: User management within the tenant
   - **Modules**: Active module status and configuration
   - **Domains**: Custom domain and SSL settings
   - **Settings**: Tenant-specific configuration

### 4.4 Decommissioning a Tenant

1. Navigate to the tenant detail view.
2. Click **"Decommission Tenant"** (requires Platform Admin role).
3. Select decommission mode:
   - **Graceful**: 30-day data retention, export available
   - **Immediate**: Data purged after export confirmation
4. Confirm with your credentials.
5. An audit record is created for compliance.

---

## 5. Configuring Modules

### 5.1 Module Registry

Navigate to **Modules** to see all registered ERP modules:

| Column | Description |
|--------|-------------|
| Module Name | Display name (e.g., "ERP CRM") |
| SKU | Unique identifier (e.g., "erp-crm") |
| Category | Module category (e.g., "business-ops") |
| Status | Health status (Healthy / Degraded / Unhealthy) |
| Capabilities | Number of capabilities provided |
| Last Health Check | Timestamp of most recent health poll |

> **[Screenshot Placeholder: Module registry with health status]**

### 5.2 Module Health Monitoring

- The module registry automatically polls `/healthz` on each module every 30 seconds.
- Health status indicators:
  - **Green (Healthy)**: Module responding normally
  - **Yellow (Degraded)**: Module responding slowly or with warnings
  - **Red (Unhealthy)**: Module not responding after 5 retry attempts
- Click on any module to see detailed health check history.

### 5.3 Module Activation/Deactivation

1. Navigate to the module in the registry.
2. Click **"Activate"** or **"Deactivate"** as needed.
3. Note: Activation is subject to the tenant's subscription entitlements.
4. Deactivation does not remove data; it prevents access until reactivation.

---

## 6. Marketplace

### 6.1 Browsing the Marketplace

1. Navigate to **Marketplace** in the sidebar.
2. Browse modules by category, popularity, or rating.
3. Use search and filters to find specific modules.
4. Each listing shows:
   - Module name and description
   - Publisher information
   - Version and compatibility
   - Installation count and rating

> **[Screenshot Placeholder: Marketplace browse view with category filters]**

### 6.2 Installing a Marketplace Module

1. Click on a marketplace listing.
2. Review the module details, permissions, and pricing.
3. Click **"Install"**.
4. The system will:
   - Verify entitlements
   - Install the module
   - Register in the module registry
   - Run health checks
   - Activate for the tenant
5. Verify the module appears in your module registry with "Healthy" status.

### 6.3 Managing Installed Modules

- View installed marketplace modules in **Marketplace > My Installations**.
- Update modules when new versions are available.
- Uninstall modules (data retention follows the module's policy).

---

## 7. Audit Logs

### 7.1 Viewing Audit Logs

1. Navigate to **Audit Logs** in the sidebar.
2. The log viewer displays all platform events in chronological order.
3. Each entry shows:
   - Timestamp
   - Event topic (e.g., `erp.platform.tenant-provisioner.created`)
   - Tenant ID
   - Actor (user or system)
   - Action details
   - Payload (expandable)

> **[Screenshot Placeholder: Audit log viewer with filters and search]**

### 7.2 Filtering and Searching

| Filter | Options |
|--------|---------|
| Date Range | Custom date picker |
| Event Type | Created, Updated, Deleted, Listed, Read |
| Service | subscription-hub, tenant-provisioner, audit-service, etc. |
| Tenant ID | Select or enter tenant identifier |
| Actor | User email or system identifier |

### 7.3 Exporting Audit Logs

1. Apply desired filters.
2. Click **"Export"** in the top-right.
3. Choose format: CSV, JSON, or PDF.
4. Select date range and fields to include.
5. Download the export file.

---

## 8. System Settings

### 8.1 Platform Configuration

Navigate to **Settings** for platform-wide configuration:

- **General**: Platform name, default timezone, contact information
- **Security**: Authentication settings, session timeout, password policies
- **AIDD Guardrails**: View and modify guardrail thresholds (requires elevated approval)
  - Minimum confidence: 0.70
  - Medium confidence: 0.82
  - High-risk auto-execute: Disabled
  - Maximum blast radius: 5,000 records
  - High-value threshold: $100,000
- **Notifications**: Default notification channels and templates
- **API**: API rate limits, webhook configuration

### 8.2 Catalog Management

The product catalog is the single source of truth for the ERP suite. To view:

1. Navigate to **Settings > Catalog**.
2. View current catalog version and product count.
3. Browse products by category.
4. View bundle definitions and their included modules.

> Note: Catalog modifications require a pull request to the `catalog/products.json` file and platform engineering approval.

### 8.3 Environment Variables

Key environment variables for service configuration:

| Variable | Description | Default |
|----------|-------------|---------|
| `ERP_CATALOG_PATH` | Path to products.json catalog | `../../catalog/products.json` |
| `PORT` | Service listening port | `8080` (8091 for subscription-hub) |
| `MODULE_NAME` | Module identifier for health checks | `ERP-Platform` |

---

## 9. Troubleshooting

### 9.1 Common Issues

| Issue | Possible Cause | Resolution |
|-------|---------------|------------|
| "Missing X-Tenant-ID" error | Tenant ID header not set | Ensure X-Tenant-ID header is included in API requests |
| Subscription creation fails | Invalid SKU | Verify SKU exists in product catalog via `GET /v1/products` |
| Module shows "Unhealthy" | Module service is down | Check module logs; restart module if needed |
| Entitlement query returns 404 | No subscription for tenant | Create subscription first via `POST /v1/subscriptions` |
| Bundle SKU rejected in single plan | Business rule violation | Use `bundle` or `suite` plan type for bundle SKUs |

### 9.2 Getting Help

- **Documentation**: Browse the full documentation suite at `/docs`
- **Support Portal**: Submit tickets at `support.erp-platform.example.com`
- **API Reference**: See [21-API-Documentation.md](./21-API-Documentation.md)
- **Training**: See [08-Training-Manual.md](./08-Training-Manual.md)

---

*For training curriculum, see [08-Training-Manual.md](./08-Training-Manual.md). For video training scripts, see [09-Video-Training-Script.md](./09-Video-Training-Script.md).*
