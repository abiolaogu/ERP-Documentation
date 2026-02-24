# ERP-Platform Figma Design Prompts

> **Document ID:** ERP-PLAT-FDP-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Audience:** UI/UX Designers
> **Related Documents:** [07-User-Manual.md](./07-User-Manual.md), [05-Product-Requirements-Document.md](./05-Product-Requirements-Document.md)

---

## Design System Foundation

**Brand Colors:** Primary: #1A56DB (Indigo), Secondary: #059669 (Emerald), Accent: #D97706 (Amber), Error: #DC2626 (Red), Background: #F9FAFB (Light Gray), Surface: #FFFFFF
**Typography:** Inter for UI text, JetBrains Mono for code/data. Heading sizes: H1 32px, H2 24px, H3 20px, Body 14px, Caption 12px.
**Border Radius:** 8px for cards, 6px for buttons, 4px for inputs.
**Spacing Scale:** 4px base unit (4, 8, 12, 16, 24, 32, 48, 64).

---

## Prompt 01: Platform Dashboard

Design a dashboard page for the ERP-Platform admin console. The layout should have a left sidebar navigation (240px wide, dark indigo #1E293B background) with the ERP-Platform logo at the top, followed by navigation items: Dashboard (active), Subscriptions, Tenants, Modules, Marketplace, Audit Logs, Notifications, Web Hosting, Settings. The main content area has a top header bar (64px height) with global search (Cmd+K shortcut), notification bell icon with badge count, and user avatar dropdown. The dashboard body contains a 4-column stat card row showing: Total Tenants (number with trend arrow), Active Subscriptions, Healthy Modules (20/20 green), and Pending Alerts. Below the stats, a 2-column layout: left column (60% width) has a "Recent Activity" feed showing audit events with timestamps, service icons, and event descriptions; right column (40% width) has a "Module Health" card showing a vertical list of all 9 platform services with green/yellow/red status dots and last-check timestamps. At the bottom, a "Subscription Distribution" chart showing plan type breakdown (Single/Bundle/Suite) as a horizontal bar chart. Use the indigo/emerald/amber color palette. Responsive: on mobile, sidebar collapses to icons only, stat cards stack to 2x2 grid.

---

## Prompt 02: Subscription Management Page

Design a subscription management list view. Left sidebar navigation as in Prompt 01 with "Subscriptions" highlighted. The page has a header row with "Subscriptions" title (H1), a "+ New Subscription" primary button (indigo), and filter controls: plan type dropdown (All/Single/Bundle/Suite), status dropdown (All/Active/Suspended/Cancelled), and a search input for tenant ID. Below, a data table with columns: Tenant ID (link, truncated with tooltip), Plan Type (badge: blue for Single, emerald for Bundle, amber for Suite), Status (badge: green Active, yellow Suspended, red Cancelled, gray Grace Period), Module Count (number), Created Date, and Actions (kebab menu with Edit, Cancel, View Details). Table supports sorting on all columns. Pagination at bottom: "Showing 1-25 of 142" with page controls. Empty state: illustration with "No subscriptions found" message. On row click, navigate to subscription detail page. Responsive: on tablet, hide Module Count and Created Date columns; on mobile, switch to card layout with stacked fields.

---

## Prompt 03: New Subscription Form

Design a full-page form for creating a new subscription. Centered content area (max 720px wide). Step indicator at top showing 3 steps: 1. Tenant & Plan (active), 2. Module Selection, 3. Review & Confirm. Step 1 shows: Tenant ID text input with autocomplete dropdown of existing tenants plus "Create new tenant" option; Plan Type radio group with cards (Single -- "Select individual modules", Bundle -- "Choose a curated package", Suite -- "Build your own combination") each with description text and an icon. When Bundle is selected, show a bundle selector grid: 8 cards in 2x4 grid, each showing bundle name, module count, key modules listed, and a price indicator. Cards have a selected state with indigo border and check icon. The Starter card shows 3 modules, Professional shows 11, Enterprise shows 20. Industry bundles (Healthcare, Education, Faith, Telecom) have domain-specific icons. Developer Suite has a code icon. "Next" button at bottom right, "Cancel" link at bottom left.

---

## Prompt 04: Tenant Provisioning Wizard

Design a multi-step tenant provisioning wizard. Full-screen overlay with a progress sidebar on the left (280px, light gray background) showing 5 steps vertically: 1. Organization Profile (check icon, completed), 2. Plan Selection (active, indigo highlight), 3. Module Configuration (grayed), 4. Admin Setup (grayed), 5. Review & Launch (grayed). Each step shows title and brief description. Main content area for the active step. Step 2 shows: "Select Your Plan" heading, current plan pre-selected if coming from subscription flow. Below the plan cards, a "Selected Modules" expandable section showing a chip/tag list of all resolved modules with capability counts. Each chip is removable for Suite plan types. At the bottom: progress bar showing "Step 2 of 5", "Back" secondary button, "Continue" primary button. On the final step, show a summary card with all configuration, a "Provision Tenant" call-to-action button, and an estimated provisioning time indicator. During provisioning, show an animated progress indicator with real-time status updates for each step (Creating record, Seeding entitlements, Configuring hosting, Sending notification).

---

## Prompt 05: Module Marketplace

Design a marketplace browsing page with a top banner area (200px height, gradient indigo to emerald) with "ERP Marketplace" heading, search bar (full width, 48px height), and category filter chips (All, Business Ops, Productivity, Intelligence, Security, Integration, Industry, Platform Tools). Below the banner, a grid layout (3 columns on desktop, 2 on tablet, 1 on mobile) of module cards. Each card (aspect ratio 4:3) has: module icon/logo placeholder (64px), module name (H3), publisher name (caption, gray), brief description (2 lines max with ellipsis), category badge (colored pill), rating stars (5-star with average), install count (e.g., "2.4k installs"), and "Install" button (primary) or "Installed" chip (emerald, with checkmark). Cards have hover state with subtle shadow elevation. Sidebar filter panel (hidden by default, toggled by "Filters" button) with checkboxes for category, rating minimum, and compatibility version. Empty search state with illustration.

---

## Prompt 06: Audit Log Viewer

Design an audit log viewer with a powerful filtering interface. Top section has a horizontal filter bar with: date range picker (preset options: Last 24h, Last 7d, Last 30d, Custom), service dropdown (subscription-hub, tenant-provisioner, etc.), event type multi-select (Created, Updated, Deleted, Listed, Read), tenant ID search input, and actor search input. "Apply Filters" button and "Reset" link. An "Export" button with dropdown (CSV, JSON, PDF) on the right. Below filters, a timeline-style log view: each entry shows a timestamp (monospace font, gray), colored dot matching event type (green for created, blue for updated, red for deleted), event topic in monospace (e.g., "erp.platform.tenant-provisioner.created"), tenant ID, actor, and a collapsible payload section. Payload section shows formatted JSON with syntax highlighting when expanded. Infinite scroll with "Loading more..." indicator. Summary bar at top showing "Showing 1,247 events matching filters". On mobile, simplify to compact card view with expandable details.

---

## Prompt 07: User Management Page

Design a user management page with a data table. Header with "Users" title, "+ Invite User" button, and role filter dropdown (All Roles, Platform Admin, Tenant Admin, Module Admin, Viewer). Table columns: Avatar circle with initials, Full Name (with email below in gray caption), Role (colored badge), Tenant Scope (chip list of tenant IDs or "All Tenants"), Last Active (relative timestamp), Status (Active/Inactive), Actions. "+ Invite User" opens a slide-over panel from the right with: email input, role selector (radio buttons with role descriptions), tenant scope multi-select, and optional message textarea. Send Invitation button with loading state.

---

## Prompt 08: System Health Dashboard

Design a health monitoring dashboard. Top row: 4 large metric cards showing: Platform Uptime (99.95% with gauge chart), Avg Response Time (12ms with sparkline), Active Services (9/9 with green indicator), Event Throughput (24.5K/min with trend). Middle section: a grid of 9 service health cards (3x3) each showing: service name, status indicator (green dot, pulse animation for healthy), response time (P50/P99), uptime percentage, and a mini time-series graph (24h). Cards for unhealthy services have red border and alert icon. Bottom section: "Health Check History" table showing recent health checks with module, status, latency, and timestamp. Auto-refresh toggle in top-right corner with "Refreshing every 30s" indicator.

---

## Prompt 09: Notification Center

Design a notification center as a slide-over panel (400px wide, from right edge). Header with "Notifications" title, "Mark All Read" link, and close (X) button. Tab bar: All, Subscriptions, Tenants, Modules, System. Notification list with each item showing: icon (type-specific: bell for system, user for tenant, credit card for subscription), title text (bold if unread), description text (2 lines), relative timestamp, and unread indicator (indigo dot). Swipe-to-dismiss on mobile. Group notifications by date (Today, Yesterday, This Week, Earlier). At the bottom, "Notification Preferences" link. Empty state with bell icon illustration and "You're all caught up!" message.

---

## Prompt 10: Billing & Usage Page

Design a billing page. Top section with current plan card: plan name (e.g., "Professional"), monthly cost, renewal date, payment method on file (masked card), and "Change Plan" button. Below, a "Module Usage" section with a horizontal bar chart showing each subscribed module with usage metrics (API calls, storage, users). Bars colored by usage level (green < 60%, amber 60-80%, red > 80%). Right column: "Invoices" list with date, amount, status (Paid/Pending/Overdue with colored badges), and PDF download link per invoice. At bottom, "Payment Methods" section with add/remove payment method functionality.

---

## Prompt 11: Settings Page

Design a settings page with a left tab navigation (vertical, 200px wide): General, Security, AIDD Guardrails, Notifications, API Keys, Integrations. General tab shows: Organization name input, default timezone dropdown, contact email input, logo upload area. AIDD Guardrails tab shows: a form with threshold sliders for min_confidence (0.0-1.0), medium_confidence (0.0-1.0), max_blast_radius (numeric input), high_value_amount_usd (currency input), and a toggle for high_risk_auto_execute (with warning when enabling). Each threshold shows current value and a description of its effect. "Save Changes" button requires confirmation dialog for AIDD changes.

---

## Prompt 12: Tenant Detail Page

Design a tenant detail page with a header section showing: tenant name (H1), tenant ID (monospace, copyable), status badge, and action buttons (Edit, Suspend, Decommission). Below header, a tab bar: Overview, Subscription, Modules, Users, Domains, Audit Log. Overview tab has two-column layout: left column with tenant info card (created date, domain, metadata), and right column with quick stats (active modules count, users count, API calls this month). Subscription tab shows current plan details and entitled modules as a chip grid. Modules tab shows a list of activated modules with health status. Audit Log tab shows tenant-scoped audit entries.

---

## Prompt 13: Module Configuration Panel

Design a module configuration slide-over panel (500px wide) that appears when clicking "Configure" on a module in the registry. Header with module name and icon. Sections: Connection Settings (endpoint URL, health check URL, timeout settings), Feature Flags (toggle list of module-specific features), Resource Limits (sliders for CPU, memory, max connections), and Notification Settings (checkboxes for which events trigger notifications). Save and Cancel buttons at bottom. Changes preview area showing diff of modified settings.

---

## Prompt 14: Web Hosting Management Page

Design a web hosting page with a table of configured domains. Columns: Domain name, Tenant, SSL Status (valid/expiring/expired with color-coded badges), CDN Status (enabled/disabled toggle), DNS Verification (verified/pending with check/clock icon), Created Date. "+ Add Domain" button opens a multi-step form: 1. Enter domain, 2. Verify DNS (shows TXT record to add), 3. Configure SSL (auto Let's Encrypt toggle), 4. CDN Settings (enable/disable, cache TTL).

---

## Prompt 15: API Keys Management Page

Design an API keys management page. Table with columns: Key Name, Key Preview (first 8 chars + masked), Created Date, Last Used, Scopes (chip list), Status, Actions (Revoke). "+ Generate Key" form: key name input, scope checkboxes (read:products, write:subscriptions, read:entitlements, admin:tenants, etc.), expiration dropdown (30d, 90d, 1y, Never). On generation, show full key once in a copyable field with warning "This key will only be shown once."

---

## Prompt 16: Onboarding Welcome Screen

Design a welcome screen for first-time platform administrators. Centered layout (max 800px). ERP-Platform logo at top. "Welcome to ERP-Platform" heading with "Business at the Speed of Prompt" tagline. Three feature highlight cards in a row: "Unified Admin" with console icon, "Instant Provisioning" with rocket icon, "AI Guardrails" with shield icon. Below, a "Get Started" call-to-action button (large, indigo) that launches the activation wizard. Secondary links: "Explore the API", "Read the Documentation", "Watch Training Videos". Subtle animated background with module icons floating.

---

## Prompt 17: Bundle Comparison View

Design a pricing/comparison table page for subscription bundles. Horizontal layout with columns for each bundle tier: Starter, Professional, Enterprise. Each column header shows bundle name, price placeholder, and "Select" button. Below, a feature matrix with rows for each of the 20 modules. Checkmarks in cells where the module is included in the bundle. The Enterprise column has all checkmarks and a "Most Popular" badge. Module names are grouped by category with section headers. Sticky header on scroll. "Compare Industry Bundles" tab switches to Healthcare/Education/Faith/Telecom comparison.

---

## Prompt 18: Incident Alert Banner

Design a system-wide alert banner that appears at the top of every page during incidents. The banner is full-width, 48px height. P1 Critical: red background (#DC2626), white text, pulsing dot animation, message text, and "View Status" link. P2 High: amber background, dark text. P3 Medium: blue background, white text. Dismiss (X) button on right. Banner pushes page content down. Multiple incidents stack vertically.

---

## Prompt 19: Entitlement Checker Tool

Design a developer tool page for testing entitlements. Split panel: left side has input fields (Tenant ID text input, Module SKU dropdown with all 20 modules). "Check Entitlement" button. Right side shows result: green card with "Entitled" and checkmark if the tenant has the module, or red card with "Not Entitled" and X icon. Below the result, show the full entitlement list for the tenant in a scrollable chip list. At the bottom, a "Bulk Check" section where developers can paste a JSON array of {tenant_id, sku} pairs and get a table of results.

---

## Prompt 20: AIDD Guardrail Decision Log

Design a decision log page specific to AIDD guardrail evaluations. Filter bar with: date range, decision outcome (Auto-Approved, Human Review, Blocked), confidence range slider (0.0-1.0), blast radius range. Table columns: Timestamp, AI Agent ID, Action Description, Confidence Score (with color-coded bar: green >= 0.82, amber 0.70-0.82, red < 0.70), Blast Radius (record count), Financial Value, Decision (colored badge), Reviewer (if human review). Click any row to expand details showing full action payload, guardrail evaluation trace, and outcome. Summary stats at top: total evaluations, auto-approved %, human reviewed %, blocked %.

---

## Prompt 21: Product Catalog Browser

Design a visual catalog browser showing all 20 modules organized by category. Use a masonry or card grid layout. Each module card shows: module icon (unique per module), module name, category badge, capability count in a small counter badge, repository link, and a "View Details" button. Category sections are collapsible with headers: Core (1 module), Business Operations (8 modules), Productivity (1), Intelligence (2), Security (1), Integration (1), Industry Verticals (4), Platform Tools (2). Each section header shows the module count. A "View as List" toggle switches to a compact table view. Search bar at top filters modules in real-time.

---

*For user manual reference, see [07-User-Manual.md](./07-User-Manual.md). For product requirements, see [05-Product-Requirements-Document.md](./05-Product-Requirements-Document.md).*
