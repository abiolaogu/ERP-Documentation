# Figma & Make Prompts -- ERP-Platform
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma Make prompts for the **ERP-Platform** module -- the Unified Control Plane for the ERP suite. ERP-Platform manages the product catalog, standalone and suite subscriptions, entitlement enforcement, tenant provisioning, marketplace, module registry, AIDD guardrails, notifications, and the activation wizard. These prompts generate high-fidelity screens for platform administrators, tenant owners, and operations staff who manage subscriptions, monitor entitlements, browse the marketplace, and configure tenants.

**Services covered:** activation-wizard, audit-service, entitlement-engine, marketplace, module-registry, notification-hub, subscription-hub, tenant-provisioner, web-hosting

**API surface:** `/v1/products`, `/v1/subscriptions`, `/v1/entitlements`, `/v1/activation-wizard`, `/v1/audit`, `/v1/marketplace`, `/v1/module-registry`, `/v1/notification-hub`, `/v1/tenant-provisioner`, `/v1/web-hosting`

---

## 2. AIDD Guardrails (Apply To All Prompts)

Every prompt in this document must comply with the following cross-cutting guardrails. Append the relevant subset as context when pasting into Figma Make.

### 2.1 User Experience And Accessibility
- WCAG 2.2 AA minimum; target AAA for text contrast on primary surfaces.
- Keyboard navigable: every interactive element reachable via Tab/Shift-Tab; Enter/Space to activate; Escape to dismiss overlays.
- Command palette accessible via `Cmd+K` / `Ctrl+K` on every page.
- Role-first layout: adapt sidebar, dashboard widgets, and CTAs based on user role (Platform Admin, Tenant Owner, Developer, Billing Manager).
- Progressive disclosure: surface summary first, expand detail on demand.
- Skeleton loading states for every data-driven region; optimistic UI for safe mutations.
- Form quality: inline validation, autosave with "Draft saved" indicator, resumable state.
- Data density toggle: compact / comfortable / expanded modes on tables.
- Multi-language readiness: all strings externalizable; RTL-safe layout with `dir="auto"`.
- Touch targets minimum 44x44px on mobile; 32x32px on desktop.

### 2.2 Performance And Frontend Efficiency
- Lighthouse Performance score target: >= 95.
- First Contentful Paint < 1.2s; Largest Contentful Paint < 2.0s on 4G.
- Lazy-load below-the-fold panels, modals, and charts.
- Image assets: WebP/AVIF with `srcset` for 1x/2x; max 100KB hero images.
- Bundle budget: < 200KB initial JS (gzipped).
- Skeleton shimmer animation uses CSS-only (no JS animation library).

### 2.3 Reliability, Trust, And Safety
- Confirmation dialogs for destructive actions (delete tenant, revoke entitlement, cancel subscription) with explicit re-type or checkbox gate.
- Blast-radius indicators: show affected record count before bulk operations.
- AIDD confidence badges on AI-generated recommendations (Low / Medium / High) with tooltip rationale.
- Immutable audit trail link visible on every entity detail page.
- Tenant isolation: always display current `Tenant: {name}` in header; cross-tenant data never co-mingled visually.

### 2.4 Observability And Testability
- Every interactive component includes `data-testid` attribute naming convention: `{page}-{component}-{action}`.
- Analytics event hooks: page_view, cta_click, form_submit, error_displayed.
- Error states: distinct empty state, error state (with retry CTA), and permission-denied state for every data region.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a Figma design-token page titled "ERP-Platform / Tokens" at 1440x3000px.

COLOR PALETTE (light mode):
- Primary: #1A56DB (Platform Blue)
- Primary Hover: #1244B0
- Secondary: #6C63FF (Entitlement Violet)
- Success: #16A34A
- Warning: #F59E0B
- Danger: #DC2626
- Neutral-50: #F9FAFB through Neutral-900: #111827 (9-step gray scale)
- Surface: #FFFFFF
- Background: #F3F4F6

COLOR PALETTE (dark mode):
- Primary: #60A5FA
- Primary Hover: #93C5FD
- Secondary: #A78BFA
- Success: #4ADE80
- Warning: #FBBF24
- Danger: #F87171
- Surface: #1F2937
- Background: #111827
- Text Primary: #F9FAFB
- Text Secondary: #9CA3AF

TYPOGRAPHY (Inter font family):
- Display: 36px / 700 / 1.1 line-height
- H1: 30px / 700 / 1.2
- H2: 24px / 600 / 1.3
- H3: 20px / 600 / 1.3
- H4: 16px / 600 / 1.4
- Body-lg: 16px / 400 / 1.5
- Body: 14px / 400 / 1.5
- Caption: 12px / 400 / 1.4
- Overline: 11px / 600 / 1.4 / uppercase / 0.05em tracking

SPACING SCALE: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96px

ELEVATION:
- Shadow-sm: 0 1px 2px rgba(0,0,0,0.05)
- Shadow-md: 0 4px 6px -1px rgba(0,0,0,0.1)
- Shadow-lg: 0 10px 15px -3px rgba(0,0,0,0.1)
- Shadow-xl: 0 20px 25px -5px rgba(0,0,0,0.1)

BORDER RADIUS: 4px (sm), 8px (md), 12px (lg), 16px (xl), 9999px (full)

Show each token as a labeled swatch with hex value, usage note, and a light/dark side-by-side preview.
```

#### Prompt F-002: Component Library

```
Create a Figma component library page titled "ERP-Platform / Components" at 1440x5000px.

Include these components with light/dark variants and all interactive states (default, hover, active, focus, disabled):

BUTTONS: Primary, Secondary, Ghost, Danger, Icon-only (24px, 32px, 40px sizes).
INPUTS: Text field, Search bar with Cmd+K hint, Select dropdown, Multi-select with tags, Date picker, Number input with stepper, Textarea with character count.
DATA DISPLAY: Data table with sortable headers, row selection checkboxes, pagination bar (page size: 10/25/50/100), density toggle (compact/comfortable/expanded), and inline row actions. Stat card with label, value, trend arrow, and sparkline. Badge (status variants: active/inactive/pending/error/trial). Tag (removable, category-colored). Progress bar (determinate + indeterminate). Confidence badge for AIDD (Low amber, Medium blue, High green) with "i" tooltip icon.
NAVIGATION: Sidebar with collapsible groups, active state, icon+label, and notification dot. Top bar with tenant selector, search, notification bell with count badge, avatar dropdown. Breadcrumbs with ">" separator. Tabs (underlined, pill variants).
FEEDBACK: Toast notification (success, error, warning, info) with dismiss and action link. Modal dialog (sm 400px, md 560px, lg 720px) with title, body, footer actions. Confirmation dialog with destructive-action re-type gate. Empty state illustration with title, subtitle, and primary CTA. Error state with retry button. Skeleton loader strips and cards.
SPECIALIZED: Entitlement matrix chip grid (module x feature), Subscription plan card (Free/Starter/Pro/Enterprise tiers), Audit log timeline entry, AIDD guardrail threshold meter.

Every component must include auto-layout, consistent padding (16px default), and named layers following: `{category}/{component}/{variant}/{state}`.
```

---

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Platform Dashboard (1440px)

```
Design a desktop dashboard page titled "ERP-Platform / Dashboard" at 1440x1024px.

LAYOUT:
- Left sidebar (240px collapsed to 64px): Logo at top, nav groups: "Overview" (Dashboard, Analytics), "Catalog" (Products, Marketplace), "Business" (Subscriptions, Entitlements), "Operations" (Tenants, Modules, Notifications), "System" (Audit Log, AIDD Guardrails). Active item: Dashboard. Bottom: user avatar, role "Platform Admin", settings gear.
- Top bar (64px): Breadcrumb "Platform > Dashboard", search bar with "Cmd+K" hint, notification bell with "12" badge, tenant selector dropdown showing "Acme Corp", avatar menu.
- Content area (1136px): 24px padding.

CONTENT:
Row 1 -- KPI Cards (4 across):
- "Active Subscriptions": 2,847 | +12.3% | green sparkline
- "Monthly Revenue": $487,200 | +8.7% | blue sparkline
- "Active Tenants": 156 | +3 this week | violet sparkline
- "Entitlement Checks (24h)": 1.2M | 99.97% pass rate | green sparkline

Row 2 -- Two panels side-by-side:
Left (60%): "Subscription Trends" area chart, last 12 months, lines for Standalone vs Suite subscriptions, y-axis: count, tooltips on hover. Filter pills: "All Products", "CRM", "SCM", "Projects", "HR".
Right (40%): "Module Adoption" horizontal bar chart: ERP-CRM 89%, ERP-Finance 82%, ERP-SCM 71%, ERP-Projects 68%, ERP-HR 54%, ERP-School 23%.

Row 3 -- Two panels:
Left (50%): "Recent Activations" table: columns [Tenant, Plan, Modules, Activated, Status]. 5 rows with realistic company names. Status badges: Active (green), Provisioning (amber), Trial (blue).
Right (50%): "Notifications" feed: 6 items with icon, message, timestamp. Types: subscription renewed, entitlement warning, new marketplace listing, AIDD threshold alert, tenant provisioned, audit flag.

Use light theme. Include dark-mode variant as a second frame.
```

#### Prompt F-004: Subscription Management Page (1440px)

```
Design a desktop page titled "ERP-Platform / Subscriptions" at 1440x1200px.

LAYOUT: Same sidebar and top bar as F-003. Active sidebar item: Subscriptions. Breadcrumb: "Platform > Business > Subscriptions".

CONTENT:
Header row: Page title "Subscriptions" (H1), subtitle "Manage standalone and suite subscription plans". Right side: "Create Subscription" primary button, "Export CSV" ghost button.

Filter bar: Search field "Search by tenant or plan...", dropdowns for Status (All/Active/Trial/Expired/Cancelled), Plan Type (All/Standalone/Suite), Module filter multi-select, Date range picker.

Data table (full width):
Columns: Checkbox | Tenant Name | Plan | Type (chip: Standalone/Suite) | Modules (tag list, +N overflow) | MRR | Status (badge) | Renewal Date | Actions (kebab menu).
15 rows with realistic data:
- "Globex Industries" | Enterprise Suite | Suite | CRM, SCM, Finance, HR, Projects | $12,400 | Active (green) | Mar 15, 2026
- "Initech Solutions" | Pro Standalone | Standalone | CRM | $299 | Trial (blue) | Feb 28, 2026
- (13 more varied rows)

Pagination: "Showing 1-15 of 2,847 subscriptions" | Page size dropdown | Prev/Next with page numbers.

Bulk action bar (appears when rows selected): "3 selected" | "Bulk Renew" | "Export Selected" | "Cancel Selected" (danger).

Include a kebab-expanded state showing: View Details, Edit Plan, Renew, View Entitlements, View Audit Log, Cancel Subscription (red text).

Light and dark mode variants.
```

#### Prompt F-005: Marketplace Page (1440px)

```
Design a desktop page titled "ERP-Platform / Marketplace" at 1440x1400px.

LAYOUT: Same sidebar/top bar pattern. Active item: Marketplace. Breadcrumb: "Platform > Catalog > Marketplace".

CONTENT:
Hero section (200px height, gradient blue-to-violet background):
- "ERP Marketplace" (Display), "Discover modules, integrations, and vertical accelerators for your business" (Body-lg, white).
- Search bar (480px wide, centered): "Search marketplace..." with filter icon.

Filter sidebar (left, 280px):
- Category checkboxes: Core Modules, Vertical Accelerators, Integrations, AI Add-ons, Themes, Compliance Packs.
- Pricing: Free, Included in Suite, Paid Add-on.
- Rating: 4+ stars, 3+ stars.
- Publisher: Verified, Community.

Product grid (3 columns, right 1136px minus filter width):
12 marketplace cards with:
- Module icon (32x32)
- Title: "ERP-CRM", "SCM Logistics AI", "GDPR Compliance Pack", "Slack Integration", "Healthcare Vertical", "ERP-School-Management", "Advanced Analytics", "Custom Report Builder", "Shopify Connector", "Multi-Currency Engine", "AIDD Governance Pack", "Kanban Power-Up"
- Publisher name + verified badge where applicable
- Short description (2 lines max)
- Rating (stars + count): "4.8 (342 reviews)"
- Price badge: "Included" (green), "Free" (blue), "$49/mo" (neutral)
- "Install" or "Included in Plan" button
- Installed count: "2.1k installs"

Pagination: Load more button centered.

Include one expanded product detail modal (720px wide): product banner, full description, screenshots carousel (3 thumbnails), reviews section, version history, install button, "Requires: subscription-hub, entitlement-engine" dependency note.

Light and dark mode.
```

#### Prompt F-006: Tenant Provisioning Page (1440px)

```
Design a desktop page titled "ERP-Platform / Tenants" at 1440x1100px.

LAYOUT: Standard sidebar/top bar. Active item: Tenants. Breadcrumb: "Platform > Operations > Tenants".

CONTENT:
Header: "Tenant Management" (H1), "Provision, configure, and monitor all tenants" (subtitle). Right: "Provision New Tenant" primary button.

Summary cards (4 across):
- "Total Tenants": 156 | Active: 148, Suspended: 5, Provisioning: 3
- "Avg Provisioning Time": 47s | Target: < 60s | green checkmark
- "Storage Utilization": 2.4 TB / 5 TB | 48% | progress bar
- "AIDD Violations (30d)": 7 | -3 vs prior month | green trend

Tenant table:
Columns: Logo/Avatar | Tenant Name | Tenant ID (monospaced, truncated) | Plan | Region (flag icon + code: US-East, EU-West, AP-South) | Modules (count badge) | Users | Status (badge: Active/Suspended/Provisioning/Decommissioned) | Created | Actions.

10 rows with realistic data including diverse company names and regions.

Side panel (slide-in from right, 480px, shown for selected tenant "Acme Corp"):
- Header: Company logo, name, tenant ID, status badge.
- Tabs: Overview | Modules | Entitlements | Audit | Settings.
- Overview tab content: Plan details card, resource usage bars (CPU, Memory, Storage, API calls), recent activity timeline (5 items), quick actions: "Manage Modules", "View Billing", "Suspend Tenant" (danger ghost).

Include a "Provision New Tenant" modal (560px): Step wizard (3 steps): 1. Organization Details (name, domain, region dropdown, plan selector), 2. Module Selection (checkbox grid of available modules with entitlement check), 3. Review & Provision (summary with estimated provisioning time, "Provision" primary CTA).

Light and dark mode.
```

#### Prompt F-007: AIDD Guardrails & Audit Page (1440px)

```
Design a desktop page titled "ERP-Platform / AIDD Guardrails" at 1440x1300px.

LAYOUT: Standard sidebar/top bar. Active item: AIDD Guardrails. Breadcrumb: "Platform > System > AIDD Guardrails".

CONTENT:
Header: "AIDD Guardrails Dashboard" (H1), "Monitor AI confidence, blast-radius controls, and approval workflows" (subtitle).

Row 1 -- Guardrail Health (3 cards):
- "Confidence Threshold": Gauge chart at 82% (current mean), min_confidence: 70% (red line), medium_confidence: 82% (amber line). Status: "Healthy".
- "Blast Radius (24h)": Max records affected: 1,247 / 5,000 limit. 3 operations flagged for review. Bar chart of last 7 days.
- "Approval Queue": 4 pending approvals. Breakdown: 2 financial (>$100K), 1 security policy change, 1 bulk data operation.

Row 2 -- Guardrail Policy Table (full width):
Columns: Policy Name | Scope | Threshold | Current Value | Status (pass/warn/fail badge) | Last Evaluated | Action.
8 rows:
- "Min Confidence Auto-Execute" | All AI Actions | >= 0.70 | 0.84 | Pass
- "High-Risk Auto-Execute Block" | Financial, HR, Security | false | false | Pass
- "Max Blast Radius Records" | Bulk Operations | <= 5,000 | 1,247 | Pass
- "High-Value USD Gate" | Financial Mutations | >= $100,000 requires approval | $87,500 last | Pass
- "Tenant Isolation Validation" | Cross-Tenant Queries | 0 violations | 0 | Pass
- (3 more with varied statuses including one "Warn")

Row 3 -- Two panels:
Left (50%): "Audit Stream (Live)" -- real-time feed with entries: timestamp, actor (user or AI agent), action, entity, confidence score, decision (auto-approved/queued/blocked), rationale snippet. 8 entries with alternating backgrounds.
Right (50%): "Approval Queue" -- card list: each card shows request summary, requester, AI confidence badge, blast radius count, affected tenants, "Approve" (green) / "Reject" (red) / "Request Info" (ghost) buttons.

Include one expanded audit entry detail modal (640px): full JSON payload preview, confidence breakdown chart (radar: accuracy, relevance, safety, completeness), decision tree visualization, "View Full Audit Trail" link.

Light and dark mode.
```

#### Prompt F-008: Notification Hub Page (1440px)

```
Design a desktop page titled "ERP-Platform / Notifications" at 1440x1100px.

LAYOUT: Standard sidebar/top bar. Active item: Notifications. Breadcrumb: "Platform > Operations > Notifications".

CONTENT:
Header: "Notification Hub" (H1), "Centralized notification management and delivery configuration" (subtitle). Right: "Create Template" primary button, "Delivery Settings" secondary button.

Row 1 -- Delivery Stats (4 cards):
- "Sent (24h)": 14,832 | Email: 8.2K, Push: 4.1K, In-App: 2.5K
- "Delivery Rate": 99.4% | SLO: > 99%
- "Open Rate (Email)": 34.2% | +2.1% vs last week
- "Failed Deliveries": 89 | Top reason: invalid email (43)

Tabs: All Notifications | Templates | Delivery Channels | Preferences

"All Notifications" tab active:
Filter bar: search, Type (System/Business/Alert/Marketing), Channel (All/Email/Push/In-App/SMS), Status (Delivered/Pending/Failed), Date range.

Notification table:
Columns: Type Icon | Subject | Recipients (count) | Channel Icons | Sent At | Status Badge | Open Rate | Actions.
10 rows:
- Bell icon | "Subscription Renewal Reminder" | 234 recipients | Email + Push | Feb 22, 2026 14:30 | Delivered | 41% | View/Resend
- Alert icon | "AIDD Threshold Warning" | 12 recipients | In-App + Email | Feb 22, 2026 11:15 | Delivered | 89% | View
- (8 more varied entries)

Include a notification detail panel (slide-out, 560px): preview of rendered notification (email mockup / push notification mockup / in-app toast mockup), delivery timeline, recipient breakdown pie chart, click-through analytics.

Light and dark mode.
```

---

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Dashboard (390px)

```
Design a mobile dashboard page titled "ERP-Platform / Mobile Dashboard" at 390x844px (iPhone 14 viewport).

LAYOUT:
- Bottom tab bar (56px): Dashboard (active), Subscriptions, Marketplace, Notifications, More.
- Top bar (56px): "Platform" title, hamburger menu (left), notification bell with "12" badge (right), avatar (right).
- Content: scrollable, 16px horizontal padding.

CONTENT:
Greeting: "Good morning, Sarah" (H3), "Platform Admin" (caption, muted).

KPI row: horizontal scroll of 4 stat cards (160px wide each):
- Active Subs: 2,847 (+12.3%)
- Revenue: $487K (+8.7%)
- Tenants: 156 (+3)
- Entitlements 24h: 1.2M (99.97%)

"Quick Actions" section: 2x2 grid of icon buttons (64x64):
- New Subscription | Provision Tenant | Browse Marketplace | View Audit

"Subscription Trends" compact chart (full width, 200px height): simplified area chart, last 6 months.

"Recent Activity" list: 6 items, each with icon (32px), title, subtitle, timestamp. Swipe left reveals "View" action.

"Pending Approvals" section: 2 approval cards with summary, confidence badge, "Approve" / "Deny" buttons.

Bottom safe area padding: 34px.

Light and dark mode variants.
```

#### Prompt F-010: Mobile Subscription List (390px)

```
Design a mobile page titled "ERP-Platform / Mobile Subscriptions" at 390x844px.

LAYOUT: Bottom tab bar with "Subscriptions" active. Top bar: back arrow, "Subscriptions" title, search icon, filter icon.

CONTENT:
Filter chips (horizontal scroll): All | Active | Trial | Expiring Soon | Suite | Standalone.

Search bar (full width, expandable from icon).

Subscription cards (vertical list, 16px gap):
Each card (full width, rounded-lg, shadow-sm):
- Row 1: Company avatar (40px) + Company name (H4) + Status badge (right-aligned)
- Row 2: Plan name + Type chip (Standalone/Suite)
- Row 3: "$X,XXX/mo" (bold) + "Renews: Mar 15" (caption, muted)
- Row 4: Module tags (horizontal scroll within card): CRM, SCM, Finance, +2 more

8 cards with varied data.

Pull-to-refresh indicator at top.

Floating action button (56px, bottom-right, 16px margin): "+" icon for new subscription.

When tapping a card, show subscription detail page (second frame):
- Header with company logo, name, full status
- Plan details card
- Module list (vertical, with entitlement status icons)
- Billing section: next invoice amount, payment method
- Actions: "Renew" primary, "Edit" secondary, "Cancel" danger ghost
- "View Audit Trail" link at bottom

Light and dark mode.
```

#### Prompt F-011: Mobile Marketplace (390px)

```
Design a mobile page titled "ERP-Platform / Mobile Marketplace" at 390x844px.

LAYOUT: Bottom tab bar with "Marketplace" active. Top bar: "Marketplace" title, search icon.

CONTENT:
Search bar (full width, visible): "Search modules and integrations..."

Category chips (horizontal scroll): All | Core | Integrations | AI | Vertical | Free.

"Featured" section: horizontal scroll of 2 large cards (300px wide, 180px tall):
- Card with gradient background, module icon, title "ERP-SCM Logistics AI", "Optimize routes with AI-powered planning", "Included in Suite" badge, 4.8 stars.
- Card for "GDPR Compliance Pack", "$29/mo", 4.6 stars.

"Popular Modules" section: vertical list of compact cards:
Each card (full width): Icon (40px) | Title + Publisher | Rating (stars) | Price badge | Install button (compact).
8 items.

"Recently Added" section: 4 items in same compact format.

Tapping a module shows detail page (second frame, 390x844px):
- Hero image (full width, 200px)
- Title, publisher, rating, install count
- Tab bar: Overview | Reviews | Changelog
- Overview: description text, screenshots horizontal scroll (3 thumbnails), requirements list
- Install button (full width, sticky bottom, 48px height, 16px padding from bottom)

Light and dark mode.
```

#### Prompt F-012: Mobile Notifications (390px)

```
Design a mobile page titled "ERP-Platform / Mobile Notifications" at 390x844px.

LAYOUT: Bottom tab bar with "Notifications" active. Top bar: "Notifications" title, "Mark all read" text button (right).

CONTENT:
Segmented control (full width): All | Alerts | System | Business.

Notification list:
Each item (full width, no card border, divider line between):
- Left: Type icon in colored circle (32px): bell (blue), alert-triangle (amber), check-circle (green), info (gray).
- Center: Title (Body, semibold if unread), preview text (Caption, 1 line, muted), timestamp (Caption, muted): "2 min ago", "1h ago", "Yesterday".
- Right: Unread dot (8px, primary blue) for unread items.
- Swipe left: "Archive" (gray), "Delete" (red).

15 notification items mixing:
- "Subscription Renewed: Globex Industries Enterprise Suite" (unread)
- "AIDD Alert: Confidence below threshold on bulk import" (unread, amber icon)
- "New Marketplace Listing: Healthcare Compliance Pack" (read)
- "Tenant Provisioned: Wayne Enterprises (US-East)" (read)
- (11 more varied)

Group headers: "Today", "Yesterday", "This Week".

Pull-to-refresh indicator.

Tapping a notification shows inline expansion (not new page): full message body, action button ("View Subscription", "Review Alert", etc.), timestamp.

Light and dark mode.
```

#### Prompt F-013: Mobile Tenant Detail (390px)

```
Design a mobile page titled "ERP-Platform / Mobile Tenant Detail" at 390x844px.

LAYOUT: Top bar with back arrow, "Acme Corp" title, kebab menu (right). No bottom tab bar (pushed from Tenants list).

CONTENT:
Header card (full width, surface color, rounded-lg):
- Company logo (64px), "Acme Corp", Tenant ID (monospaced, truncated with copy icon), Status badge "Active", Region flag "US-East".

Tab bar (sticky below header): Overview | Modules | Usage | Audit.

Overview tab:
- "Plan" card: Enterprise Suite, $12,400/mo, Auto-renew: On, Next renewal: Mar 15, 2026.
- "Quick Stats" 2x2 grid: Users: 342, Modules: 8, API Calls (30d): 2.1M, Storage: 180 GB.
- "Resource Usage" section: 3 progress bars with labels: CPU 34%, Memory 52%, Storage 48%.
- "Quick Actions" list: Manage Users, Update Plan, View Billing, Suspend Tenant (red text).

Modules tab (second frame):
Vertical list of installed modules, each row: Module icon, name, status badge (Active/Degraded/Disabled), entitlement expiry, toggle switch.

Light and dark mode.
```

#### Prompt F-014: Mobile AIDD Alerts (390px)

```
Design a mobile page titled "ERP-Platform / Mobile AIDD Alerts" at 390x844px.

LAYOUT: Top bar with back arrow, "AIDD Alerts" title, filter icon (right). Accessed from "More" tab.

CONTENT:
Summary bar (full width, 64px): "4 pending approvals" amber background, "Tap to review" with right arrow.

Alert cards (vertical list):
Each card (full width, rounded-lg, shadow-sm, left border colored by severity):
- Severity stripe: red (critical), amber (warning), blue (info).
- Title: "Bulk Data Export Exceeds Blast Radius" (H4)
- Details: "Requested: 7,200 records | Limit: 5,000 | Requester: AI Agent v2.1"
- Confidence badge: "Medium (0.76)" amber pill
- Timestamp: "12 min ago"
- Actions row: "Approve" (green, compact), "Reject" (red, compact), "Details" (ghost).

6 alert cards with varied severities and types:
- Blast radius exceeded (red)
- High-value financial operation $125,000 (amber)
- Low confidence AI recommendation (amber)
- Tenant isolation check passed (blue, info)
- Security policy change request (red)
- Module deactivation for 3 tenants (amber)

Light and dark mode.
```

---

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-015: Tablet Dashboard (1024px)

```
Design a tablet dashboard page titled "ERP-Platform / Tablet Dashboard" at 1024x768px (landscape iPad).

LAYOUT:
- Collapsible sidebar (64px icon-only by default, expandable to 240px on tap): same nav as desktop F-003.
- Top bar (56px): same as desktop but with hamburger toggle for sidebar.
- Content area: 24px padding.

CONTENT:
KPI row: 4 cards in a row (narrower than desktop, numbers and sparklines still visible).

Row 2: Subscription Trends chart (full width, reduced to 240px height).

Row 3: Two panels stacked 50/50:
- "Recent Activations" (5-column table: Tenant, Plan, Modules count, Date, Status).
- "Notifications" feed (4 items, compact layout).

Bottom: "Module Adoption" as horizontal bar chart (full width, compact).

Ensure touch-friendly targets (44px min), comfortable spacing. Sidebar overlay on expand (does not push content on tablet).

Light and dark mode.
```

#### Prompt F-016: Tablet Subscription Management (1024px)

```
Design a tablet page titled "ERP-Platform / Tablet Subscriptions" at 1024x768px.

LAYOUT: Icon-only sidebar (64px), top bar (56px), content area.

CONTENT:
Header: "Subscriptions" (H2), "Create Subscription" button.

Filter bar: search field + status dropdown + plan type dropdown (3 across). Module filter moves to "More filters" expandable row.

Data table (full width):
Columns adjusted for tablet: Tenant Name | Plan | Type chip | MRR | Status | Renewal | Actions kebab.
Modules column hidden (available in detail view).

10 rows visible. Pagination below.

Row tap expands inline detail panel below the row (accordion style) showing: full module list, billing info, quick actions.

Light and dark mode.
```

---

## 4. Make Automation Prompts

#### Prompt M-001: Subscription Lifecycle Automation

```
Create a Make (Integromat) scenario titled "ERP-Platform: Subscription Lifecycle Manager".

TRIGGER: Webhook -- listens for CloudEvents on topic "erp.platform.subscription-hub.*" (created, updated, deleted).

FLOW:
1. Router: Branch by event action:
   a. "created" path:
      - HTTP POST to notification-hub `/v1/notification-hub` with welcome email template, recipient = tenant admin email.
      - HTTP POST to entitlement-engine `/v1/entitlement-engine` to provision entitlements for subscribed modules.
      - HTTP POST to audit-service `/v1/audit` logging subscription creation with tenant_id, plan, modules.
   b. "updated" path (plan change):
      - HTTP GET current entitlements from `/v1/entitlements/{tenant_id}`.
      - Comparison module: diff old vs new modules.
      - HTTP PUT to entitlement-engine to add/remove entitlements.
      - HTTP POST to notification-hub: "Plan updated" notification.
   c. "deleted" path (cancellation):
      - HTTP POST to notification-hub: cancellation confirmation + exit survey link.
      - HTTP DELETE entitlements via entitlement-engine (grace period: 30 days).
      - HTTP POST to audit-service: cancellation audit entry.

2. Error handler on each branch: retry 3x with exponential backoff, then send alert to notification-hub with severity "critical".

OUTPUT: Log each step result to a Google Sheet or data store for reconciliation.

SCHEDULE: Always-on webhook listener.
HEADERS: Authorization: Bearer {{platform_service_token}}, X-Tenant-ID: {{event.tenant_id}}.
```

#### Prompt M-002: Tenant Provisioning Pipeline

```
Create a Make scenario titled "ERP-Platform: Tenant Provisioning Pipeline".

TRIGGER: Webhook -- CloudEvent "erp.platform.tenant-provisioner.created".

FLOW:
1. Parse CloudEvent payload: extract tenant_id, organization_name, region, selected_plan, selected_modules[].
2. HTTP POST to tenant-provisioner `/v1/tenant-provisioner` with full provisioning request body.
3. Wait/Poll loop (max 120s, poll every 5s): GET `/v1/tenant-provisioner/{tenant_id}/status` until status = "ready" or "failed".
4. If "ready":
   - HTTP POST to subscription-hub `/v1/subscriptions` to activate subscription.
   - HTTP POST to entitlement-engine `/v1/entitlement-engine` to set initial entitlements.
   - HTTP POST to module-registry `/v1/module-registry` to register enabled modules.
   - HTTP POST to notification-hub: send "Welcome to ERP" email with getting-started guide link.
   - HTTP POST to activation-wizard `/v1/activation-wizard` to initialize guided setup state.
   - HTTP POST to audit-service: log successful provisioning.
5. If "failed":
   - HTTP POST to notification-hub: alert platform admin with failure details.
   - HTTP POST to audit-service: log provisioning failure with error payload.

ERROR HANDLING: Circuit breaker -- if > 3 failures in 10 minutes, pause scenario and alert via Slack webhook.
```

#### Prompt M-003: AIDD Guardrail Enforcement Workflow

```
Create a Make scenario titled "ERP-Platform: AIDD Guardrail Enforcer".

TRIGGER: Webhook -- CloudEvent "erp.platform.entitlement-engine.*" and "erp.platform.audit.created".

FLOW:
1. Parse event: extract action_type, confidence_score, blast_radius_count, affected_amount_usd, requester_type (human/AI), tenant_id.
2. Decision tree:
   a. If confidence_score < 0.70 AND requester_type = "AI":
      - Block action. HTTP POST to audit-service: log blocked action with reason "Below minimum confidence threshold".
      - HTTP POST to notification-hub: alert tenant admin and platform admin.
   b. If blast_radius_count > 5000:
      - Queue for human approval. HTTP POST to a "pending-approvals" endpoint (internal queue or data store).
      - HTTP POST to notification-hub: send approval-request notification to designated approvers.
   c. If affected_amount_usd >= 100000:
      - Queue for financial approval. Same as (b) but with "financial-approvers" recipient group.
   d. Else: Auto-approve. HTTP POST to audit-service: log auto-approved action.
3. For queued items: Set a 4-hour SLA timer. If not resolved:
   - Escalate: HTTP POST to notification-hub with escalation template.
   - Update audit-service with escalation event.

ERROR HANDLING: All guardrail decisions are idempotent -- duplicate events produce same outcome.
```

#### Prompt M-004: Marketplace Listing Sync

```
Create a Make scenario titled "ERP-Platform: Marketplace Listing Sync".

TRIGGER: Scheduled -- every 6 hours.

FLOW:
1. HTTP GET `/v1/marketplace?updated_since={{last_run_timestamp}}` to fetch recently updated listings.
2. For each listing:
   a. HTTP GET `/v1/module-registry/{module_id}` to validate module still exists and is compatible.
   b. If module version changed:
      - HTTP PUT `/v1/marketplace/{listing_id}` to update compatibility metadata.
      - HTTP POST to notification-hub: notify publishers of compatibility change.
   c. If module deregistered:
      - HTTP PUT `/v1/marketplace/{listing_id}` to set status "deprecated".
      - HTTP POST to notification-hub: notify all subscribers of deprecated listing.
3. Aggregate stats: count updated, deprecated, new listings.
4. HTTP POST to audit-service: log marketplace sync results.
5. Update `last_run_timestamp` in data store.

ERROR HANDLING: If marketplace API returns 5xx, retry 3x then skip and alert.
```

#### Prompt M-005: Notification Digest Scheduler

```
Create a Make scenario titled "ERP-Platform: Daily Notification Digest".

TRIGGER: Scheduled -- daily at 08:00 UTC.

FLOW:
1. HTTP GET `/v1/notification-hub?status=unread&since={{yesterday_08:00_UTC}}` per tenant.
2. For each tenant with unread notifications > 0:
   a. Group notifications by category: system, business, alerts, marketplace.
   b. Generate digest HTML using template module:
      - Header: "Your ERP Platform Digest for {{date}}"
      - Section per category with notification count and top 3 items.
      - Footer: "View all notifications in ERP Platform" deep link.
   c. HTTP POST to notification-hub `/v1/notification-hub` with channel = "email", template = "daily-digest", recipient = tenant admin.
3. HTTP POST to audit-service: log digest send stats (tenants notified, total notifications summarized).
4. Mark processed notifications with digest_sent_at timestamp.

ERROR HANDLING: Per-tenant isolation -- one tenant failure does not block others. Failed tenants queued for retry at 12:00 UTC.
```

---

## 5. Prompt Usage Guidelines

### How to Use These Prompts

1. **Figma Make**: Open Figma, use the "Make a Design" feature (or Figma AI), and paste the prompt text between the triple backticks (excluding the backticks themselves).
2. **Iteration**: After initial generation, refine by adding follow-up prompts like:
   - "Make the sidebar collapsible with a toggle button"
   - "Switch to dark mode"
   - "Add a loading skeleton state for the data table"
   - "Show the empty state when no subscriptions exist"
3. **Component reuse**: Generate the Design Tokens (F-001) and Component Library (F-002) first. Reference them in subsequent page prompts: "Use the ERP-Platform design tokens and component library."
4. **Breakpoint workflow**: Design desktop (1440px) first, then adapt to tablet (1024px) and mobile (390px). Each breakpoint prompt is self-contained but maintains visual consistency.
5. **Make scenarios**: Import each scenario as a blueprint into Make. Replace placeholder values (webhook URLs, API base URLs, tokens) with environment-specific configuration.

### Prompt Customization Points

| Variable | Default | Description |
|----------|---------|-------------|
| `{{primary_color}}` | #1A56DB | Brand primary; swap for white-labeling |
| `{{font_family}}` | Inter | System font; swap for brand font |
| `{{api_base_url}}` | `/v1` | Adjust for environment routing |
| `{{tenant_name}}` | Acme Corp | Sample tenant for mockup data |
| `{{platform_service_token}}` | -- | Service-to-service JWT for Make scenarios |

---

## 6. Output Packaging Convention

Each generated Figma output should follow this structure:

```
ERP-Platform-Design/
  tokens/
    light-theme.json
    dark-theme.json
  components/
    buttons.fig
    inputs.fig
    data-display.fig
    navigation.fig
    feedback.fig
    specialized.fig
  pages/
    desktop/
      dashboard.fig
      subscriptions.fig
      marketplace.fig
      tenants.fig
      aidd-guardrails.fig
      notifications.fig
    tablet/
      dashboard.fig
      subscriptions.fig
    mobile/
      dashboard.fig
      subscriptions.fig
      marketplace.fig
      notifications.fig
      tenant-detail.fig
      aidd-alerts.fig
  make-scenarios/
    subscription-lifecycle.json
    tenant-provisioning.json
    aidd-enforcer.json
    marketplace-sync.json
    notification-digest.json
```

**Naming convention**: `{project}-{breakpoint}-{page}-{variant}.fig`
**Layer naming**: `{category}/{component}/{variant}/{state}` (e.g., `navigation/sidebar/expanded/active`)

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Lighthouse Performance | >= 95 | Chrome DevTools on deployed build |
| Lighthouse Accessibility | >= 95 | Chrome DevTools audit |
| First Contentful Paint | < 1.2s | WebPageTest, 4G throttle |
| Largest Contentful Paint | < 2.0s | WebPageTest, 4G throttle |
| Cumulative Layout Shift | < 0.1 | Chrome UX Report |
| Time to Interactive | < 3.0s | WebPageTest |
| Initial JS Bundle (gzipped) | < 200 KB | Build output analysis |
| Image assets per page | < 500 KB total | Build output analysis |
| API Read p99 | < 200ms | Backend SLO dashboard |
| API Write p99 | < 500ms | Backend SLO dashboard |
| Error rate | < 0.1% | Observability platform |
| Availability | >= 99.95% | Uptime monitor |

---

## 8. AIDD Handoff Gate Template

Before any Figma design moves to development, verify:

```
AIDD Design Handoff Checklist -- ERP-Platform
==============================================
[ ] All WCAG 2.2 AA contrast ratios verified (use Figma A11y plugin)
[ ] Keyboard navigation flow documented for each page
[ ] All interactive states designed: default, hover, active, focus, disabled, loading, error, empty
[ ] Light and dark mode variants complete
[ ] Desktop (1440px), tablet (1024px), and mobile (390px) breakpoints delivered
[ ] Design tokens match `tokens/light-theme.json` and `tokens/dark-theme.json`
[ ] Component naming follows `{category}/{component}/{variant}/{state}` convention
[ ] data-testid attributes documented for QA
[ ] AIDD confidence badges included on all AI-generated content surfaces
[ ] Destructive action confirmation dialogs include blast-radius indicator
[ ] Tenant isolation (tenant name in header) verified on every authenticated page
[ ] Performance budget annotations added (lazy-load boundaries, image format notes)
[ ] Analytics event annotations added (page_view, cta_click, form_submit)
[ ] Responsive behavior documented (what collapses, what scrolls, what hides)
[ ] Handoff notes include API endpoint mapping for each data region
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial creation. 16 Figma prompts, 5 Make prompts covering all ERP-Platform services. |

---

## 8. Round 01 Implementation Handoff (Web, Mobile, Desktop)

### Prompt F-021: Token-to-Component Production Mapping

```text
Create a handoff board named "ERP-Platform Round 01 - Token Mapping".
Map each token family to a real implemented component from code:
- Primary/Secondary/Status color tokens -> button, badge, alert components.
- Spacing/radius tokens -> cards, table rows, shell paddings, tab bars.
- Typography tokens -> page title, subtitle, KPI value, data table text.
Include a 1:1 mapping table with columns:
Token, CSS variable, Component, File reference, Screenshot placeholder.
Add dark/light comparison frames and accessibility contrast annotations.
```

### Prompt F-022: Web App Production Frames (Next.js)

```text
Create 8 responsive web frames (Desktop 1440, Tablet 1024, Mobile web 390) for:
Dashboard, Subscriptions, Marketplace, Tenants, Modules, Audit, Notifications, Activation.
Use role switch variants (Platform Admin, Tenant Owner, Billing Manager, Developer).
For each frame, include:
- Skeleton loading state
- Empty state
- Error state
- Success state
- Latency budget indicator chip
Add interaction notes for transitions <= 120ms and API state updates <= 300ms.
```

### Prompt F-023: Mobile App (Expo) Screen Set

```text
Design production mobile screens for ERP Platform Mobile:
- Dashboard tab
- Subscriptions tab
- Tenants tab
- Alerts tab
- Error/Offline state
Use native-safe touch targets (>=44px), concise KPI cards, and role-based tab visibility rules.
Include handoff specs for React Native style tokens and spacing scale.
```

### Prompt F-024: Desktop App (Electron) Operations Console

```text
Design a desktop operations console at 1366x900 for Electron:
- Left operations rail
- KPI strip (modules, bundles, total SKUs)
- Catalog matrix table
- Latency status badge
- Native window constraints and scalable layout breakpoints
Include density variants (compact and comfortable) for operations teams.
```

### Prompt F-025: Component Contract Export

```text
Generate final component contracts for implementation parity:
- Shell (sidebar + header)
- KPI card
- Badge system
- Data table
- Subscription creation form
- Role selector
Output acceptance criteria per component:
- Visual states
- Accessibility rules
- Interaction latency budget
- Data-testid naming convention
```

<!-- SOVEREIGN_FIGMA_MAKE_EXPANSION_2026_03 -->
## 2026-03 Shared Infra + AIDD + GA Expansion Pack

### Scope
This section upgrades the prompt pack to align with:
- Shared infrastructure (`Hasura + ERP-DBaaS + ERP-IAM + ERP-Observability`)
- AIDD guardrails (Protected, Supervised, Autonomous execution modes)
- GA deployment quality expectations (accessibility, performance, observability, rollback-safe UX)

### Global Prompt Rules (Apply To Every Screen)
- Use production-safe copy and deterministic states for loading/error/empty/success.
- Include tenant context, role context, and policy context in all critical admin flows.
- Require explicit confirmation UX for destructive actions; show blast radius and rollback hint.
- Include instrumentation notes: event name, trace ID propagation, KPI target, and SLO linkage.
- Ensure WCAG 2.1 AA contrast and keyboard focus maps in every frame set.

### Prompt F-900: Shared Infra Control Plane Screen
```
Design a control plane view for ERP-Platform that shows:
1. Hasura GraphQL connectivity (latency, success %, schema drift status)
2. ERP-DBaaS health (connection pool, replica lag, failover posture)
3. ERP-IAM integration (OIDC issuer health, token validation errors, session invalidations)
4. ERP-Observability status (OTLP export rate, tracing availability, alert health)

Include: command surface, timeline, runbook links, and one-click diagnostics panel.
Add states for: fully healthy, degraded IAM, degraded DB, degraded GraphQL, global outage.
```

### Prompt F-901: Role-Aware Workspace + Guardrail Surface
```
Generate a role-aware workspace for ERP-Platform with:
- Role lenses: Operator, Manager, Auditor, Admin
- Guardrail panel with mode badge (Protected / Supervised / Autonomous)
- Inline policy rationale for blocked/supervised actions
- Approval request flow for supervised actions

Ensure each role sees distinct navigation and action priorities.
```

### Prompt F-902: Incident + Recovery UX
```
Design a full incident-response flow for ERP-Platform:
- Alert ingestion to triage board
- Impacted tenant view and blast-radius visualization
- Action timeline with runbook steps and rollback controls
- Post-incident report generator modal

Must include: trace-id copy, audit evidence export, and SLA breach indicators.
```

### Prompt F-903: Mobile-Responsive Executive Snapshot
```
Create mobile and tablet executive dashboards for ERP-Platform:
- KPI cards (availability, p95 latency, error budget burn, throughput)
- AI-generated narrative summary with confidence score
- Risk highlights and next-best-action CTA

Use adaptive layout breakpoints and preserve semantic heading structure.
```

### Prompt F-904: Figma Make -> Engineering Handoff Packet
```
For each completed flow, generate a handoff packet containing:
- Component inventory and token usage map
- API contract touchpoints (GraphQL operations and IAM claims)
- Test matrix (unit/integration/e2e/accessibility/performance)
- Observability event catalog and dashboard mapping
- GA release checklist references
```

### Prompt M-900: Make Automation Blueprint (Shared Infra)
```
Create a Make scenario for ERP-Platform that:
1. Triggers on deployment/webhook events
2. Validates GraphQL, IAM, DB, and OTLP health in sequence
3. Creates incident ticket when thresholds are breached
4. Posts status updates to operations channels
5. Writes audit trail to persistent store

Add branching for: transient failures, policy violations, and hard-stop conditions.
```

### Prompt M-901: Tenant Onboarding Orchestration
```
Generate a Make scenario for tenant onboarding in ERP-Platform:
- Validate tenant metadata and compliance profile
- Provision IAM roles and claims mapping
- Apply Hasura metadata/migrations checks
- Seed tenant defaults and dashboards
- Emit onboarding completion event and SLA timer
```

### Completion Criteria
- Every screen has loading/error/empty/success/access-denied variants.
- Every critical action includes telemetry mapping and guardrail annotation.
- Every flow includes desktop + tablet + mobile-responsive frames.
- Every high-risk flow includes rollback and operator escalation UX.
