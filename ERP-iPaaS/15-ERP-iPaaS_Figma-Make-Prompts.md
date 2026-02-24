# Figma & Make Prompts -- ERP-iPaaS (BillyRonks Workflow-as-a-Service)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make.com automation prompts for the ERP-iPaaS platform (BillyRonks Workflow-as-a-Service). The iPaaS is a multitenant Integration Platform-as-a-Service that combines low-code workflow building (Activepieces), durable developer-grade orchestration (Temporal), NexumFlow DAG engine, event-driven backbone (Redpanda/Kafka), and an open connector marketplace.

The platform serves four personas: Citizen Integrators (visual workflow builder), Platform Engineers (durable SDK workflows), Integration Architects (connector governance), and Operations Engineers (monitoring and alerting). Designs cover the Integration Studio canvas, Connector Marketplace, API Management Console, Event Stream Monitor, ETL Pipeline Builder, Webhook Debugger, Developer Portal, Monitoring Dashboard, NexumFlow Visual DAG Builder, and Tenant Administration.

Architecture references: Traefik/Kong gateway, Keycloak OIDC/JWT auth, Activepieces + Temporal orchestration, Kafka/Redpanda event backbone, PostgreSQL + ClickHouse data stores, Vault for secrets, OpenTelemetry observability.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance on all screens; contrast ratio >= 4.5:1 for body text, >= 3:1 for large text and UI elements
- Minimum 44x44px touch targets on all interactive elements including drag-and-drop handles, node connection points, canvas controls, table actions
- Visible focus indicators (2px Blue-600 outline offset 2px) for keyboard navigation; sequential tab order through all form fields, table rows, and canvas nodes
- Screen reader landmarks: `<nav>`, `<main>`, `<aside>`, `<header>` with `aria-label`; canvas nodes have `role="treeitem"` with `aria-describedby` for node configuration
- Plain-language labels; technical terms explained via tooltips (e.g., "DLQ" shows "Dead Letter Queue -- messages that failed processing"); action verbs for buttons ("Deploy", "Test Step", "Install Connector")
- Clear state communication: loading (skeleton), empty (illustration + CTA), error (inline message + retry action), success (toast + green check), in-progress (progress bar with step indicator)
- Reduced motion: honor `prefers-reduced-motion`; canvas flow animations and node transition effects disabled when set
- Color is never the sole indicator; all status indicators pair color with icon and text label

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading via React.lazy + Suspense for each major section (Studio, Marketplace, API Management, Events, ETL, Webhooks, Portal, Monitoring)
- Initial route bundle < 220KB gzip; canvas renderer (e.g., React Flow) loaded separately on demand (~80KB)
- Skeleton UIs for every data-dependent component; shimmer animation with 1.5s cycle
- Canvas rendering: hardware-accelerated transforms, virtual viewport for workflows with 100+ nodes; off-screen nodes not rendered
- Virtual scrolling for event streams, execution logs, and connector lists exceeding 50 items
- JSON viewers: syntax-highlighted with lazy parsing for payloads > 100KB; collapsible tree with expand-on-demand
- WebSocket connections for real-time event streams with automatic reconnection and exponential backoff
- Service worker for offline access to connector documentation and workflow definitions

### 2.3 Reliability, Trust, And Safety
- Optimistic UI for safe mutations (star connector, toggle notification) with instant visual feedback and server reconciliation
- Recovery paths: undo toast (5s) for destructive actions (delete workflow, remove connector, purge DLQ events); confirmation modals for irreversible actions (delete tenant, revoke API key)
- Autosave for workflow canvas and configuration forms (debounced 2s); "All changes saved" indicator with version number
- Trust signals: verified connector badge (shield icon), SLO compliance indicator (green/amber/red), tenant isolation badge ("Isolated" pill on namespace), encryption indicator on secret fields
- Audit trail: all platform actions emitted as CloudEvents to `tenant.<ID>.<domain>.<event>` topics; audit log viewer accessible per tenant
- PII redaction warnings when configuring LLM nodes or data transformation steps that may process sensitive data
- Tenant context always visible in header: tenant name, namespace, quota usage bar

### 2.4 Observability And Testability
- Analytics events: `ipaas.workflow.created`, `ipaas.workflow.executed`, `ipaas.connector.installed`, `ipaas.connector.tested`, `ipaas.api.key_generated`, `ipaas.event.published`, `ipaas.etl.pipeline_run`
- Error events: `ipaas.error.workflow_failed`, `ipaas.error.connector_timeout`, `ipaas.error.auth_expired`, `ipaas.error.quota_exceeded`
- Performance instrumentation: LCP, FID, CLS tracked per route; canvas interaction latency (node drag, connection create, zoom) tracked via Performance.mark/measure
- Feature flags: all new features behind LaunchDarkly flags; instrumented with exposure events

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

Design a comprehensive design token page for the ERP-iPaaS design system. Use an 8px spatial grid throughout. Display the following token categories in organized sections with visual swatches and code references:

**Color Palette:**
- Primary: Blue-600 (#2563EB) with shades Blue-50 (#EFF6FF) through Blue-900 (#1E3A5F)
- Secondary: Violet-600 (#7C3AED) for automation/AI accents, connectors
- Success: Emerald-600 (#059669) for active workflows, healthy connectors, passed tests, trigger nodes
- Warning: Amber-600 (#D97706) for condition nodes, approaching limits, degraded status
- Error: Red-600 (#DC2626) for failed executions, errors, delete actions, DLQ alerts
- Neutral: Slate-50 (#F8FAFC) background, White (#FFFFFF) surface, Slate-900 (#0F172A) text primary, Slate-500 (#64748B) text secondary, Slate-200 (#E2E8F0) borders
- Dark Mode: Slate-950 (#020617) background, Slate-900 (#0F172A) surface, Slate-100 (#F1F5F9) text primary
- Canvas: Slate-100 (#F1F5F9) grid background, Slate-200 (#E2E8F0) gridlines at 20px intervals
- Node Colors: Trigger Green-500 (#22C55E), Action Blue-500 (#3B82F6), Condition Amber-500 (#F59E0B), Loop Violet-500 (#8B5CF6), Error Red-500 (#EF4444), HTTP Cyan-500 (#06B6D4), Database Indigo-500 (#6366F1), Transform Orange-500 (#F97316)
- Status: Running (Blue-500 animated pulse), Completed (Emerald-500), Failed (Red-500), Queued (Slate-400), Paused (Amber-500), Draft (Slate-300)

**Typography:**
- Font family: Inter for UI text, JetBrains Mono for code, JSON, configuration values, and terminal output
- Scale: Display 36px/44px 700, H1 30px/36px 700, H2 24px/32px 600, H3 20px/28px 600, H4 16px/24px 600, Body-L 16px/24px 400, Body-M 14px/20px 400, Body-S 12px/16px 400, Caption 11px/14px 500, Code-L 14px/20px 400 (JetBrains Mono), Code-S 12px/16px 400 (JetBrains Mono), Overline 10px/14px 700 uppercase tracking 0.1em

**Spacing:** 4px, 8px, 12px, 16px, 20px, 24px, 32px, 40px, 48px, 64px, 80px

**Border Radius:** 4px inputs, 6px buttons, 8px cards/nodes, 12px modals/panels, 16px popovers/tooltips, 9999px pills/badges/avatars

**Elevation:** Level-0 none, Level-1 0 1px 3px rgba(0,0,0,0.1), Level-2 0 4px 6px rgba(0,0,0,0.1), Level-3 0 10px 15px rgba(0,0,0,0.1), Level-4 0 20px 25px rgba(0,0,0,0.15)

**Breakpoints:** Mobile 390px, Tablet 768px, Desktop 1024px, Wide 1440px

**Canvas-Specific Tokens:**
- Grid cell: 20x20px, Slate-200/30 stroke
- Node: 200x80px default, 8px radius, 2px border, Level-1 shadow on rest, Level-2 on hover, Level-3 on drag
- Connection line: 2px stroke, Slate-400, curved bezier with 50px control offset, animated dash (running state: 4px dash, 4px gap, 0.5s animation)
- Connection port: 12px circle, Slate-300 border, white fill; hover: Blue-500 fill with 4px glow ring
- Selection: 2px Blue-500 border + box-shadow 0 0 0 3px rgba(37,99,235,0.3)
- Minimap: 150x100px, Slate-900/80 background, viewport indicator rectangle with Blue-500 border

Show both light and dark mode variants side by side.

---

#### Prompt F-002: Component Library

Design a comprehensive component library page for ERP-iPaaS. Each component shows all states (default, hover, active, focus, disabled, loading, error) in light and dark mode:

**Navigation Components:**
- Platform Sidebar: 240px width, collapsible to 64px (icon-only). Logo at top with "BillyRonks" wordmark. Section headers (12px overline, Slate-500). Nav items (14px, 40px height): Workflows (zap icon), Connectors (puzzle-piece), API Management (globe), Events (radio), ETL Pipelines (git-merge), Webhooks (webhook), Monitoring (activity), Settings (settings). Active item: Blue-50 background, Blue-600 left border 3px, Blue-600 text. Badge counts (Blue-500 pill). Tenant selector dropdown at bottom with org name and namespace.
- Breadcrumb: Slash-separated, icons before segments (home > workflows > lead-intake > v1.2)
- Tab Bar: Underline style for content tabs; Pill style for filter tabs

**Canvas Components (iPaaS-Specific):**
- Workflow Node (200x80px):
  - Trigger variant: Green-500 left border (3px), icon (24px, Green-600), title (14px 600), subtitle (12px Slate-500, e.g., "Webhook"), status dot (8px, right side)
  - Action variant: Blue-500 left border, icon + title + subtitle (e.g., "POST to CRM"), connection indicator on left (input port) and right (output port)
  - Condition variant: Diamond shape (rotated 45deg square, 100x100px), Amber-500 border, "If/Else" label centered, two output ports (true/false)
  - Loop variant: Rounded rectangle with loop icon overlay, Violet-500 border
  - Error handler variant: Red-500 dashed border, error-triangle icon
  - States: idle (default), selected (blue glow), running (animated pulse border), completed (green check badge), failed (red X badge), disabled (50% opacity)
- Connection Line: Bezier curve from output port to input port. States: default (Slate-400), selected (Blue-500), running (animated dashes), error (Red-500 dashed)
- Connection Port: 12px circle on node edges. States: default (Slate-300 border, white fill), hover (Blue-500 fill, glow ring), connecting (Blue-500, drag line visible), incompatible (Red-300, shake animation)
- Node Palette Card (draggable from sidebar): 48px height, icon (20px) + label (14px) + grab handle dots on right. Hover: Blue-50 background.

**Data Display Components:**
- JSON Viewer: Collapsible tree with syntax highlighting (strings Green-600, numbers Blue-600, booleans Violet-600, keys Slate-700, null Red-400). Copy button top-right. Line numbers in Slate-300 gutter. Expand/collapse all toggle.
- Execution Log Row: 48px height. Step number pill (24px circle, Slate-100), node icon (16px), node name (14px), status badge (completed/failed/running/skipped), duration "2.3s" (12px Slate-500, monospace), timestamp (12px Slate-400). Expandable: shows input/output JSON viewers.
- Metric Card: 200px min width, top label (12px overline Slate-500), large value (28px 700), trend indicator (arrow up green or arrow down red + percentage), sparkline chart (48px height, Slate-200 with Blue-500 line). Optional: p50/p95/p99 sub-values.
- Status Badge: Pill shape. Variants: Active (Green-50 bg, Green-700 text, green dot), Inactive (Slate-100, Slate-600), Draft (Slate-50, Slate-500, dashed border), Failed (Red-50, Red-700), Running (Blue-50, Blue-700, animated pulse), Deprecated (Amber-50, Amber-700, strikethrough).
- Connector Card: 280x200px. Icon (48px top-left), name (16px 600), author (14px Slate-500), description (12px, 2 lines), category tags (pill badges), star rating (14px, amber stars), quality score pill, version, "Install"/"Installed" button.
- Schema Viewer: Two-column layout showing field name + type + required indicator on left, description on right. Nested objects indented with tree lines. Types color-coded: string (Green), number (Blue), boolean (Violet), object (Slate), array (Orange).

**Form Components:**
- Configuration Panel: 320px width right panel. Header with node name (editable, 16px 600) + type badge. Sections: Configuration fields, Input Mapping, Output Preview, Test. Each section collapsible with chevron.
- Field Mapper: Two columns (source fields left, target fields right) connected by draggable lines. Field items: icon (type) + field name + data type badge. "Auto-map" button. Unmapped fields highlighted in Amber-50. Conflict icon (warning triangle) for type mismatches.
- Cron Builder: Visual cron expression builder with dropdowns for minute, hour, day-of-month, month, day-of-week. Human-readable preview below: "Every weekday at 6:00 AM Africa/Lagos". Raw cron string shown in code font for power users with edit toggle.
- Secret Input: Masked field (dots), copy button, refresh/rotate button, "Stored in Vault" badge with lock icon. Show/hide toggle (eye icon). Last rotated timestamp.
- Code Editor: Monaco-style editor with syntax highlighting for TypeScript/JSON/YAML. Line numbers, minimap, bracket matching, autocomplete. Toolbar: Format, Copy, Expand (full screen), Language selector.

**Feedback Components:**
- Toast: 360px, Level-3 shadow. Icon + message + optional action + dismiss. Types: success (green), error (red), warning (amber), info (blue).
- Execution Progress: Multi-step progress bar showing workflow steps. Each step: circle node with icon, label below, connecting line between. States: completed (green filled circle + check), running (blue pulsing circle), pending (gray outline), failed (red filled circle + X), skipped (gray dashed outline).
- Quota Usage Bar: Horizontal bar with gradient fill (Green -> Amber -> Red), percentage label, "N / M executions today" text, "Upgrade" link when near limit.

---

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Integration Studio -- Visual Workflow Canvas (1440px)

Design the core visual workflow builder interface at 1440px viewport. This is the primary drag-and-drop integration authoring experience.

**Top Bar (56px height, white background, bottom border Slate-200):**
- Left: Back arrow (returns to workflows list), workflow icon (zap), editable workflow name "Lead Intake Pipeline" (16px 600, click to rename with inline input), version badge "v1.2.0" (12px Slate-500 pill)
- Center: Status badge -- "Draft" (Slate-200 pill, Slate-600 text) / "Active" (Green-50 pill, Green-700 text, green dot) / "Paused" (Amber-50 pill, Amber-700 text)
- Right: "Test Workflow" button (secondary, play-circle icon), "Save" button (secondary, save icon), "Activate" / "Deactivate" toggle button (primary Green-600 when activating, secondary Amber-600 when deactivating), user avatar (32px), tenant picker dropdown "Acme Corp / prod" (12px Slate-500)
- Breadcrumb below top bar (32px): Home > Workflows > Lead Intake Pipeline

**Left Panel (260px width, collapsible via arrow icon, white background, right border):**
- Tab bar (40px): "Triggers" | "Actions" | "Logic" | "AI" -- each tab with icon, underline active indicator Blue-600
- Search input (36px, full width minus padding, magnifying glass icon, placeholder "Search nodes...")
- Scrollable node palette organized by category:
  - **Triggers** tab:
    - Category "Webhook" (12px overline header):
      - Draggable card: Webhook icon (green, 20px) + "Webhook Trigger" (14px) + grab handle dots
      - "Schedule Trigger" (clock icon)
      - "Kafka Event" (radio icon)
      - "Polling Trigger" (refresh-cw icon)
    - Category "App Triggers":
      - "GitHub Push" (GitHub icon)
      - "Slack Message" (Slack icon)
      - "Email Received" (envelope icon)
  - **Actions** tab:
    - Category "HTTP": "HTTP Request" (globe icon, cyan), "GraphQL Request"
    - Category "Database": "PostgreSQL Query" (database icon, indigo), "ClickHouse Insert", "Redis Set"
    - Category "Communication": "Send Email" (envelope, blue), "Slack Message" (Slack icon), "PagerDuty Alert"
    - Category "CRM": "ERPNext Create" (ERPNext icon), "Salesforce Update", "HubSpot Contact"
    - Category "Cloud": "MinIO Upload" (cloud icon), "AWS S3 Get"
    - Category "Custom Connectors" (with "Browse Marketplace" link):
      - "ClickHouse Piece" (verified badge), "ERPNext Piece", "CRM Piece"
  - **Logic** tab:
    - "If/Else Condition" (diamond icon, amber)
    - "Switch (Multi-Branch)" (git-branch icon)
    - "Loop (For Each)" (repeat icon, violet)
    - "Delay/Wait" (clock icon)
    - "Error Handler" (alert-triangle icon, red)
    - "Code (JavaScript)" (code icon)
    - "Merge" (git-merge icon)
  - **AI** tab:
    - "LLM Prompt" (sparkle icon, violet)
    - "LangFlow Pipeline" (brain icon)
    - "Text Classifier" (tag icon)
    - "PII Detector" (shield icon)
    - "MCP Tool" (tool icon)

**Center Canvas (fluid width, ~760px):**
- Background: Slate-100 (#F1F5F9) with 20px grid dots (Slate-200/40)
- Canvas controls toolbar (floating, top-left of canvas, 40px height, white, Level-1 shadow, 8px radius):
  - Undo (rotate-ccw icon), Redo (rotate-cw icon), separator, Zoom out (-), zoom level "100%" badge, Zoom in (+), Zoom to fit (maximize icon), separator, Grid toggle, Auto-layout (layout icon), separator, Comment mode toggle (message-square)
- Sample 6-node workflow displayed on canvas:
  1. **Webhook Trigger** node (200x80px): Green-500 left border, webhook icon, "Webhook Trigger" title, "POST /hooks/lead-intake" subtitle, single output port (right edge). Position: left area.
  2. **Validate Lead** node (200x80px): Blue-500 left border, shield-check icon, "Validate Lead" title, "Check required fields" subtitle, input port (left), output port (right). Connected from node 1 via curved bezier line.
  3. **If/Else** node (diamond shape, 120x120px rotated): Amber-500 border, "Email Valid?" center text. Input port top. Two output ports: "True" (right, green label), "False" (bottom, red label). Connected from node 2.
  4. **Enrich Lead** node (200x80px): Cyan-500 left border, globe icon, "HTTP: Enrich API" title, "GET /api/clearbit" subtitle, running state (animated blue pulse border, spinner icon). Connected from If/Else "True" output.
  5. **Push to CRM** node (200x80px): Indigo-500 left border, database icon, "ERPNext: Create Lead" title, "POST /api/resource/Lead" subtitle, completed state (green check badge top-right). Connected from node 4.
  6. **Send Slack Alert** node (200x80px): Blue-500 left border, Slack icon, "Slack: Post to #leads" title, "#leads channel" subtitle, queued state (gray). Connected from node 5.
  - From If/Else "False" output: connection line to **Error Handler** node (200x80px): Red-500 dashed border, alert-triangle icon, "Log Invalid Lead" title, "Emit audit event" subtitle.
- Connection lines: 2px Slate-400 curved bezier with arrowhead at target. The line between nodes 3-4 shows animated running state (blue dashes moving along path).
- Minimap (bottom-right corner): 150x100px, Slate-900/80 background, showing simplified node rectangles and a blue viewport indicator rectangle. Draggable to navigate.
- When a connection line is being drawn (from output port to empty space): a dotted Blue-500 line follows the cursor with a ghost drop target indicator.

**Right Panel (320px width, collapsible, white background, left border):**
- Shown when node 5 "Push to CRM" is selected (blue glow selection state):
  - Header (48px): "Push to CRM" editable name, "ERPNext Action" badge (Indigo-100 background Indigo-700 text), close X button
  - **Configuration** section (collapsible, expanded by default):
    - "Connection" dropdown: "ERPNext Production" (with green dot = connected, lock icon = secrets in Vault)
    - "Resource Type" dropdown: "Lead"
    - "Method" read-only: "POST"
  - **Input Mapping** section (collapsible):
    - Field mapper showing:
      - "first_name" <- "$.enriched.firstName" (auto-mapped, green check)
      - "last_name" <- "$.enriched.lastName" (auto-mapped, green check)
      - "email" <- "$.trigger.body.email" (manually mapped, blue line)
      - "company" <- "$.enriched.company" (auto-mapped)
      - "lead_score" <- unmapped (amber warning indicator, "Map a source field" placeholder)
    - "Auto-Map Fields" button (wand icon), "Add Custom Field" button (+)
    - Source field picker: dropdown showing available fields from previous steps as a tree (Step 1 outputs > Step 4 outputs) with JSONPath references
  - **Output Preview** section (collapsible, collapsed):
    - JSON viewer showing the expected output structure with syntax highlighting
    - "Last output" toggle showing actual data from last test execution
  - **Test** section:
    - "Test this step" button (Blue-600, play icon) with dropdown: "Test with sample data" / "Test with last trigger data"
    - Test result: green "200 OK" badge + duration "342ms" + output JSON viewer
  - **Advanced** section (collapsible, collapsed):
    - Retry policy: Attempts input (number, default 3), Backoff dropdown (exponential/linear/fixed), Initial interval "5s"
    - Timeout: "30s" input
    - Error handling: dropdown "Stop workflow" / "Continue to error handler" / "Retry then continue"
  - **Delete Node** button (Red-600 text, trash icon, bottom) with confirmation dialog

**Bottom Panel (expandable from collapsed 40px to 240px, white background, top border):**
- Collapsed state: "Execution Log" label + last run status badge "Completed in 2.3s" + expand chevron
- Expanded state:
  - Header: "Execution Log" + Filter dropdown: "All" / "Running" / "Completed" / "Failed" + clear log button
  - Execution list (table format):
    - Columns: Step # | Node Name | Status | Duration | Timestamp | Actions
    - Row 1: "1" | "Webhook Trigger" | green check "Completed" | "12ms" | "10:42:01.234" | expand icon
    - Row 2: "2" | "Validate Lead" | green check "Completed" | "8ms" | "10:42:01.246" |
    - Row 3: "3" | "If/Else: Email Valid?" | green check "True" | "2ms" | "10:42:01.254" |
    - Row 4: "4" | "Enrich Lead" | blue spinner "Running" | "1.2s..." | "10:42:01.256" | cancel button
    - Row 5: "5" | "Push to CRM" | gray "Queued" | "--" | "--" |
    - Row 6: "6" | "Send Slack Alert" | gray "Queued" | "--" | "--" |
  - Expanded row (clicking expand on row 1): Shows tabbed view: "Input" JSON viewer / "Output" JSON viewer with full request/response payloads

**Context Menu (right-click on canvas or node):**
- On empty canvas: "Paste node", "Add comment", "Select all", "Zoom to fit"
- On node: "Copy", "Duplicate", "Delete", "Disable/Enable", "Add note", "View documentation"

**States to design:** Empty canvas ("Drag a trigger from the panel to start building"), 6-node workflow as described, single node selected, test execution running (animated nodes), test completed with error (node 4 shows red X, error details in bottom panel), workflow activated (green "Active" status, all nodes show last execution status).

---

#### Prompt F-004: Connector Marketplace (1440px)

Design the connector marketplace interface at 1440px viewport where users browse, install, and manage integration connectors.

**Top Navigation:** Standard iPaaS nav with Connectors tab active in sidebar.

**Page Header (120px):**
- Title: "Connector Marketplace" (H1, 30px 700)
- Subtitle: "Browse, install, and manage connectors for your workflows" (16px Slate-500)
- Stats bar (14px): "142 connectors" (with puzzle-piece icon) pipe separator "28 categories" pipe "18 verified" (with shield icon) pipe "3 installed in your workspace" (with check icon)
- Search bar (prominent, full width, 48px height, Level-1 shadow, magnifying glass icon left, filter funnel icon right): "Search connectors by name, category, or capability..."

**Left Sidebar (240px, Slate-50 background):**
- "Filter Connectors" header (14px 600)
- **Category** section (checkboxes with counts):
  - CRM (24), Finance (18), Communication (16), Data Warehouse (12), DevOps (15), Marketing (11), HR (8), Observability (9), AI/ML (7), Cloud Storage (6), Custom (16)
- **Status** section (radio buttons):
  - All, Published, Draft, Deprecated, Beta
- **Badge** section (checkboxes):
  - Verified (18), Enterprise (8), Community (96), Beta (20)
- **Rating** section:
  - Star selector: 4+ stars, 3+ stars, 2+ stars, Any
- **Compatibility** section (checkboxes):
  - Activepieces, Temporal, NexumFlow, All Engines
- "Clear All Filters" link (Blue-600, 12px)
- "Submit Your Connector" CTA button (secondary, full width, bottom of sidebar)

**Main Grid (fluid, ~1160px):**
- Sort bar (36px): "Sort by:" dropdown: Popularity (default), Rating, Newest, Name A-Z. View toggle: Grid (default) / List
- Grid: 4 columns, 16px gap
- Connector Cards (each ~270x220px, white background, 8px radius, Level-1 shadow, hover Level-2):
  - Top row: Connector icon (48x48px, colored background circle) top-left + "Verified" shield badge top-right (if applicable)
  - Connector name (16px 600): "Salesforce CRM"
  - Author (12px Slate-500): "by BillyRonks" or "by Community: David Kim"
  - Description (14px Slate-600, 2 lines max with ellipsis): "Full CRM integration with contacts, leads, opportunities, and custom objects"
  - Tags row: Category pills (12px, Slate-100 background): "CRM", "Sales"
  - Bottom row (separated by divider):
    - Left: Star rating "4.7" with amber star icon + "(23 reviews)" in Slate-400
    - Center: Quality score "92/100" in Blue-50 pill Blue-700 text
    - Right: Version "v2.1.0" in Slate-400
  - Footer: "Install" button (Blue-600, full width minus 16px padding, 36px height) or "Installed" badge (Green-50, Green-700, check icon, full width)
  - Card hover: subtle scale(1.02) transition, Level-2 shadow
- Sample connector cards (16 visible in 4x4 grid):
  - Row 1: "Salesforce CRM" (verified, 4.7 stars, installed), "ERPNext" (verified, 4.5), "HubSpot" (verified, 4.6), "Slack" (verified, 4.8, installed)
  - Row 2: "PostgreSQL" (verified, 4.9), "ClickHouse" (verified, 4.3, installed), "GitHub" (verified, 4.7), "Jira" (community, 4.1)
  - Row 3: "Stripe" (verified, 4.6), "Twilio" (verified, 4.4), "SendGrid" (verified, 4.5), "AWS S3" (enterprise, 4.8)
  - Row 4: "Google Sheets" (verified, 4.3), "PagerDuty" (verified, 4.2), "Datadog" (community, 3.9), "Custom: KYC Provider" (beta, 3.5)
- Pagination: "Showing 1-16 of 142" + page numbers + next/previous arrows

**Connector Detail Modal (720px wide, 85vh max, centered, Level-4 shadow, 12px radius):**
- Header (80px): Large icon (64px), connector name (24px 600) "Salesforce CRM", author "by BillyRonks", badges row: "Verified" (shield, Green-50), "Enterprise" (star, Violet-50), version "v2.1.0" (Slate-100 pill). Close X button top-right.
- Tab bar: "Overview" | "Actions & Triggers" | "Configuration" | "Documentation" | "Reviews" | "Versions" | "SLO"
- **Overview** tab:
  - Description paragraph (16px)
  - Features checklist with green checks
  - Compatibility badges: "Activepieces" (green), "Temporal" (blue), "NexumFlow" (violet)
  - Screenshot carousel (640px wide, 360px height, navigation dots)
  - Author info card: avatar + name + link to profile
  - "Install" CTA button (Blue-600, prominent, 48px height)
- **Actions & Triggers** tab:
  - **Triggers** section:
    - List: "New Contact Created" (webhook, Green-500 dot), "Contact Updated" (polling, Amber-500 dot), "Deal Stage Changed" (webhook)
    - Each shows: name, type badge (webhook/polling/schedule), description, schema preview expandable
  - **Actions** section:
    - List: "Create Contact" (POST), "Update Contact" (PATCH), "Search Contacts" (GET), "Delete Contact" (DELETE), "Create Opportunity" (POST), "Bulk Import" (POST)
    - Each shows: name, method badge (colored: GET green, POST blue, PATCH amber, DELETE red), description, input/output schema preview
- **Configuration** tab:
  - Auth setup wizard: Step 1 "Select auth type" (OAuth2/API Key), Step 2 "Enter credentials" (form fields), Step 3 "Test connection" (test button with result), Step 4 "Configure scopes" (checkboxes)
  - Rate limit display: "1000 req/hour, burst 50"
  - Retry policy defaults
- **Reviews** tab:
  - Average rating: large "4.7" with 5-star visual + "23 reviews"
  - Rating distribution bar chart (5 stars: 15, 4 stars: 5, 3 stars: 2, 2 stars: 1, 1 star: 0)
  - Review list: avatar + name + date + star rating + review text
- **Versions** tab:
  - Timeline: "v2.1.0 (current) - Feb 15, 2026" with changelog, "v2.0.0 - Jan 20, 2026", "v1.5.0 - Dec 10, 2025"
  - Each version: changelog bullet points, compatibility notes, "Install this version" button
- **SLO** tab:
  - Availability: 99.95% (green bar chart, monthly view)
  - P99 latency: 342ms (sparkline chart)
  - Error rate: 0.02% (area chart)
  - Uptime calendar (GitHub-style green squares grid, 90 days)

**States:** Marketplace loaded, search with filtered results, search with no results ("No connectors found. Try different filters or submit your own."), connector detail modal open, installation in progress (progress spinner on Install button), installed state.

---

#### Prompt F-005: Monitoring Dashboard (1440px)

Design an operations monitoring dashboard at 1440px viewport showing system health, workflow executions, and alerts.

**Top Navigation:** Standard iPaaS nav with Monitoring tab active.

**Dashboard Header (64px):**
- Title: "Operations Dashboard" (H2, 24px 600)
- Time range selector: "Last 24h" | "Last 7d" | "Last 30d" | Custom date picker. Auto-refresh toggle with interval dropdown (30s/1m/5m).
- Tenant scope: "All Tenants" dropdown or specific tenant filter
- "Export Report" button (secondary, download icon)

**Golden Signals Section (4 metric cards row, 16px gap, full width):**
- **Latency** card (white, Level-1 shadow, 8px radius):
  - Label: "Workflow Latency (P99)" (12px overline Slate-500)
  - Value: "342ms" (28px 700 Emerald-700, indicating healthy)
  - Sub-values: "p50: 85ms | p95: 210ms" (12px Slate-500)
  - Sparkline: 48px height area chart, last 24h, Blue-500 line
  - Trend: green down arrow "12% vs last week" (12px Green-600)
- **Traffic** card:
  - Label: "Executions / Hour"
  - Value: "2,847" (28px 700)
  - Sparkline: area chart showing traffic pattern
  - Trend: "8% increase" green arrow
- **Errors** card:
  - Label: "Error Rate"
  - Value: "0.23%" (28px 700, Green-700 if < 1%, Amber-700 if 1-5%, Red-700 if > 5%)
  - Sub-value: "47 failures / 20,431 total"
  - Sparkline: red-tinted if elevated
  - Trend: "0.05% improvement" green arrow
- **Saturation** card:
  - Label: "Worker Saturation"
  - Value: "67%" (28px 700 Amber-700)
  - Progress bar: 67% filled, gradient Green to Amber at 60%, Red at 90%
  - Sub-value: "24 of 36 workers active"
  - "Scale Workers" link (Blue-600)

**Workflow Execution Timeline (full width, 300px height, white card):**
- Header: "Workflow Executions" (16px 600), filter dropdown: All Types / Activepieces / Temporal / NexumFlow
- Stacked bar chart (x-axis: time in 1-hour buckets for 24h, y-axis: count):
  - Green bars: successful executions
  - Red bars: failed executions (stacked on top)
  - Amber bars: retried executions
  - Hover tooltip: "2:00 PM - 3:00 PM | 234 successful | 3 failed | 12 retried"
- Below chart: "Total today: 20,431 executions | 99.77% success rate" summary line

**Two-Column Layout Below (16px gap):**

**Left Column (60% width):**
- **Active Alerts** card (white, Level-1):
  - Header: "Active Alerts" (16px 600) + count badge "3" (Red-500 pill) + "View all" link
  - Alert rows (56px each):
    1. Red-500 left border. "CRITICAL" badge (Red-50 Red-700). "Consumer lag exceeding 10k on tenant.acme.orders" -- "12m ago" -- "Acknowledge" button
    2. Amber-500 left border. "WARNING" badge (Amber-50 Amber-700). "Worker saturation > 80% for tenant.beta" -- "45m ago" -- "Investigate" link
    3. Amber-500 left border. "WARNING" badge. "Connector 'Stripe' P99 latency > SLO (500ms)" -- "2h ago" -- "View details" link
  - Each alert: click to expand showing full context, related metrics, runbook link, and "Resolve" action

- **Top Failing Workflows** table card:
  - Header: "Top Failing Workflows (24h)" (16px 600)
  - Table columns: Workflow Name | Tenant | Failures | Error Rate | Last Error | Actions
  - Row 1: "Invoice Processing" | "tenant-acme" | "12" | "4.2%" (Red-600 text) | "Timeout on Stripe API" | "View" link
  - Row 2: "Data Sync - HubSpot" | "tenant-beta" | "8" | "2.1%" (Amber-600) | "401 Unauthorized" | "View"
  - Row 3: "Lead Enrichment" | "tenant-acme" | "5" | "1.3%" (Amber-600) | "Rate limit exceeded" | "View"
  - Row 4: "Social Monitoring" | "tenant-gamma" | "3" | "0.8%" (Green-600) | "API response malformed" | "View"
  - Sortable columns, hover row highlight

**Right Column (40% width):**
- **Tenant Usage** card:
  - Header: "Tenant Usage (24h)" (16px 600) + "Manage quotas" link
  - Horizontal bar chart showing top 5 tenants by execution count:
    - "tenant-acme": 8,234 executions, 82% of quota (progress bar, Amber tint)
    - "tenant-beta": 5,120 executions, 51% of quota (Green bar)
    - "tenant-gamma": 3,890 executions, 39% of quota (Green bar)
    - "tenant-delta": 2,100 executions, 21% of quota (Green bar)
    - "tenant-epsilon": 1,087 executions, 11% of quota (Green bar)
  - Each bar clickable to tenant detail view

- **Connector Health** card:
  - Header: "Connector Health" (16px 600) + "View all" link
  - Grid of connector health indicators (4 columns):
    - Each: icon (24px) + name (12px) + status dot:
      - "Salesforce" green dot
      - "ERPNext" green dot
      - "ClickHouse" green dot
      - "Stripe" amber dot (elevated latency)
      - "HubSpot" red dot (auth failure)
      - "Slack" green dot
      - "GitHub" green dot
      - "PagerDuty" green dot
  - Legend: Green = Healthy, Amber = Degraded, Red = Down

- **Event Backbone** card:
  - Header: "Kafka/Redpanda Health"
  - Metrics: "Topics: 47 active" | "Messages/sec: 1,247" | "Consumer Groups: 12"
  - Consumer lag chart: small bar chart per consumer group, green = caught up, red = lagging
  - DLQ count: "23 events in DLQ" with "Review" link (amber warning)

**States:** Dashboard loaded (healthy), dashboard with critical alerts (red header tint), dashboard loading (skeleton cards), dashboard with elevated errors (error rate card red, workflow chart showing red spike), empty state for new tenant ("No execution data yet. Deploy your first workflow to see metrics.").

---

#### Prompt F-006: Event Stream Monitor (1440px)

Design a real-time event stream monitoring interface at 1440px viewport for the Kafka/Redpanda event backbone.

**Top Navigation:** Standard iPaaS nav with Events tab active.

**Page Header (56px):**
- Title: "Event Stream Monitor" (H2, 24px 600)
- Live indicator: Green pulse dot + "Live" (Green-500 text, 14px)
- Pause/Resume toggle button (Blue-600, pause icon / play icon)
- Time window: "Showing last 5 minutes" dropdown (5m/15m/1h/6h/24h/custom)
- Tenant filter dropdown: "All Tenants" / specific tenant

**Topic Overview Section (200px height, horizontal scroll row of topic cards):**
- Each topic card (220x160px, white, Level-1 shadow, 8px radius):
  - Topic name: "tenant.acme.orders" (14px 600, truncated)
  - Metrics:
    - "Messages/sec: 47" with mini sparkline (Green-500 line)
    - "Consumer lag: 0" (Green-500 text) or "Consumer lag: 12,450" (Red-500 text with alert icon)
    - "Partitions: 6"
    - "Schema: OrderEvent v2.1" (Slate-500)
  - Health dot: large 12px circle, Green/Amber/Red
  - Click: filters the main event stream to this topic
- Sample topics: "tenant.acme.orders" (healthy), "tenant.acme.crm" (healthy), "tenant.acme.connector.dlq" (red, lag: 23), "tenant.beta.workflows" (healthy), "tenant.beta.etl" (amber, lag: 342), "tenant.gamma.notifications" (healthy)
- "View All Topics" link at end of row

**Main Event Stream (full width, flex height, white card):**
- Header bar (48px): "Live Event Stream" + filter inputs: Topic dropdown "All Topics", Event Type dropdown "All Types" (workflow.completed/workflow.failed/connector.invoked/audit...), Search input "Filter by event ID or payload..."
- Stream table (virtual scrolling, new events prepend at top with slide-down animation):
  - Columns: Timestamp | Topic | Event Type | Key | Size | Status | Actions
  - Each row (40px height):
    - Timestamp: "10:42:01.234" (12px JetBrains Mono Slate-600)
    - Topic: "tenant.acme.orders" (12px, truncated with tooltip)
    - Event Type: "order.created" (14px, colored badge: completed=Green, failed=Red, audit=Slate, created=Blue)
    - Key: "ord-2026-0847" (12px JetBrains Mono Blue-600, clickable)
    - Size: "1.2 KB" (12px Slate-400)
    - Status: "Delivered" green check or "Failed" red X or "DLQ" amber warning
    - Actions: Expand icon + Copy icon + Replay icon
  - Row expansion (click expand): Full CloudEvents envelope with syntax-highlighted JSON viewer
    ```json
    {
      "specversion": "1.0",
      "type": "erp.ipaas.order.created",
      "source": "/tenants/acme/workflows/order-intake",
      "id": "evt-2026-0847-abc123",
      "time": "2026-02-23T10:42:01.234Z",
      "datacontenttype": "application/json",
      "data": {
        "tenant_id": "tenant-acme",
        "order_id": "ord-2026-0847",
        "customer": "Acme Corp",
        "total": 2450.00,
        "currency": "NGN"
      }
    }
    ```
  - New events flash with Blue-50 background for 2s then fade to white
  - Failed events have Red-50 background
- Streaming stats bar (bottom of stream): "Events received: 12,847 in last 5m | Rate: 42.8 msg/s | DLQ: 23 pending"

**Dead Letter Queue Panel (bottom section, expandable, 280px when expanded):**
- Header: "Dead Letter Queue" + count badge "23" (Red-500) + "Replay All" button (secondary, refresh-cw icon, requires confirmation) + "Purge" button (danger, requires confirmation)
- DLQ Table:
  - Columns: Event ID | Original Topic | Failure Reason | Retry Count | First Failed | Last Retry | Actions
  - Row 1: "evt-abc123" | "tenant.acme.orders" | "Schema validation failed: missing 'currency'" (truncated) | "3/3" | "10:30 AM" | "10:42 AM" | "Replay" button + "Discard" button + "View" expand
  - Row 2: "evt-def456" | "tenant.beta.crm" | "Downstream timeout: ERPNext unavailable" | "5/5" | "9:15 AM" | "10:38 AM" | same actions
  - Row 3: "evt-ghi789" | "tenant.acme.connector.stripe" | "Rate limit exceeded (429)" | "3/3" | "10:35 AM" | "10:41 AM"
  - Expanded view: Full original event payload + error stack trace + retry timeline

**Schema Registry Tab (accessible via tab in page header):**
- Schema list table:
  - Columns: Schema Name | Type (Avro badge / JSON Schema badge / Protobuf badge) | Version | Compatibility | Last Updated | Actions
  - Rows: "OrderEvent" / JSON Schema / v2.1 / "Full" / "Feb 22, 2026" / View + Diff
  - "WorkflowCommand" / Avro / v1.0 / "Backward" / "Feb 15, 2026"
  - "AuditEvent" / JSON Schema / v3.0 / "Full" / "Feb 20, 2026"
- Schema detail (click row): Full schema definition with syntax-highlighted JSON/Avro viewer
- Version diff view: Side-by-side comparison of two schema versions with additions (green) and removals (red)
- "Upload Schema" button (Blue-600 primary)

**States:** Stream flowing with events, stream paused (amber banner "Stream paused -- click Resume"), DLQ panel expanded with events, schema registry view, empty state ("No events in the selected time window"), topic detail drilldown.

---

#### Prompt F-007: ETL Pipeline Builder (1440px)

Design a visual ETL (Extract-Transform-Load) pipeline builder at 1440px viewport with a three-panel horizontal layout.

**Top Bar (56px):**
- Breadcrumb: Home > ETL Pipelines > "Finance Data Sync" (editable name)
- Status: "Active" green badge + Schedule indicator "Daily at 6:00 AM WAT" with clock icon
- "Run Now" button (secondary, play icon), "Save" button (primary), "Deactivate" button (secondary amber)
- Last run: "Feb 22, 2026 06:00 AM -- 2m 34s -- 12,847 rows" with green check

**Source Panel (300px, left, white background, right border):**
- Header: "1. Source" (H3, 20px 600, numbered circle "1" Green-500)
- Source type selector (segmented control): "Database" (active) | "API" | "File" | "SaaS"
- **Database selected:**
  - Connection dropdown: "ERPNext Production (PostgreSQL)" with green status dot. "New connection" link.
  - Schema dropdown: "public"
  - Table selector (searchable dropdown): "tabSales Invoice" selected
  - "Query Mode" toggle: Visual / SQL
  - Visual mode: Column checkboxes with select all:
    - [x] invoice_id (varchar, PK icon)
    - [x] customer_name (varchar)
    - [x] posting_date (date)
    - [x] grand_total (decimal)
    - [x] currency (varchar)
    - [ ] docstatus (int)
    - [ ] modified (datetime)
  - "Where clause" section: "posting_date >= '2026-01-01'" with add condition "+" button
  - SQL mode: Code editor (Monaco) with SQL syntax highlighting
- "Test Connection" button (secondary, 36px) with result: green check "Connected -- 12,847 rows match"
- **Data Preview** section (200px height at bottom, expandable):
  - Table showing first 5 rows of source data with column headers, types, and sample values
  - Horizontal scroll for wide tables
  - Row count: "Previewing 5 of 12,847 rows"

**Transform Panel (fluid ~540px, center, white background, left+right borders):**
- Header: "2. Transform" (H3, 20px 600, numbered circle "2" Blue-500) + "Add Transform" button (Blue-600 secondary, plus icon)
- Vertical stack of transform cards (each 80px, full width minus 32px padding, white, Level-1 shadow, 8px radius):
  - Card 1: Drag handle dots (left) + "Map Fields" icon (arrows-right-left) + title "Rename Columns" (14px 600) + description "customer_name -> client_name, grand_total -> amount" (12px Slate-500) + "Configure" button (right) + delete X icon. Green check indicator (configured).
  - "+" Add Transform button between cards (Blue-600 dashed border, plus icon, on hover shows full button "Add Transform")
  - Card 2: "Filter Rows" icon (filter) + "Remove Zero-Amount Invoices" + "amount > 0" + Configure + delete. Green check.
  - "+" button
  - Card 3: "Aggregate" icon (calculator) + "Sum by Month" + "GROUP BY month(posting_date), SUM(amount)" + Configure + delete. Green check.
  - "+" button
  - Card 4: "Enrich" icon (plus-circle) + "Add Exchange Rate" + "Join with exchange_rates table on currency" + Configure + delete. Amber warning icon (needs configuration).
  - "+" button
  - Card 5: "Custom SQL" icon (code) + "Calculate NGN Equivalent" + "amount * exchange_rate AS amount_ngn" + Configure + delete. Green check.
- Cards are drag-and-drop reorderable (drag handle on left, drop indicator line appears between cards)
- **Transform Configuration** (right slide-over panel when "Configure" clicked, 400px, Level-3 shadow):
  - Title: "Configure: Filter Rows"
  - Form: Column dropdown "amount", Operator dropdown "> (greater than)", Value input "0"
  - "Add condition" button (AND/OR group builder)
  - Preview: table showing rows before/after filter with removed rows struck through in red
  - "Apply" button (primary) + "Cancel" button
- **Data Preview at Each Stage** (bottom of center panel, 200px, expandable):
  - Tab for each transform step: "Source" | "After Rename" | "After Filter" | "After Aggregate" | "After Enrich" | "After Custom SQL"
  - Selected tab shows data preview table at that point in the pipeline
  - Delta indicators: "12,847 rows -> 11,920 rows" (red arrow for reduction), "5 columns -> 6 columns" (green arrow for addition)

**Target Panel (300px, right, white background, left border):**
- Header: "3. Target" (H3, 20px 600, numbered circle "3" Violet-500)
- Target type selector: "Database" (active) | "API" | "File"
- **Database selected:**
  - Connection dropdown: "ClickHouse Analytics" with green dot
  - Database: "analytics"
  - Table: "finance_monthly_summary" (with "Create table" link if table doesn't exist)
  - Write mode: dropdown "Append" / "Upsert" / "Replace"
  - Key field (for upsert): "month" dropdown
- **Field Mapping Visualization** (240px height):
  - Two columns connected by lines:
    - Left (Transform output): "month" (date), "client_name" (string), "amount" (decimal), "currency" (string), "amount_ngn" (decimal)
    - Right (Target columns): "report_month" (Date), "customer" (String), "amount_usd" (Decimal64), "currency" (String), "amount_ngn" (Decimal64)
  - Connected by curved lines showing field mapping (matching colors per pair)
  - "Auto-Map" button (wand icon) + "Clear All" link
  - Unmapped fields: amber highlight + "Map this field" placeholder
  - Type mismatch warning: amber triangle icon between incompatible types with "Type coercion will be applied" tooltip
- **Write Settings:**
  - Batch size: "5000 rows" input
  - "Create table if not exists" checkbox (checked)
  - "Drop and recreate" checkbox (unchecked, red warning text if checked)

**Bottom Bar (48px, white, top border, sticky):**
- Left: Schedule configuration: "Schedule" toggle (on), Cron expression "0 6 * * 1-5" editable input, Human preview "Every weekday at 6:00 AM Africa/Lagos"
- Center: Pipeline stats: "Last run: Feb 22, 06:00 AM | 12,847 rows in 2m 34s"
- Right: "Test Run (100 rows)" button (secondary), "Save Pipeline" button (primary), "Delete Pipeline" button (danger ghost)

**States:** Pipeline configured and active, pipeline being configured (amber "Unsaved changes" indicator), test run in progress (animated progress through Source -> Transform -> Target with row counts), test completed (success summary with row counts and preview of target data), pipeline failed (red error state on failed step with error message and retry button), empty pipeline builder (step-by-step wizard: "Step 1: Choose a source").

---

#### Prompt F-008: Webhook Debugger (1440px)

Design a webhook debugging and testing interface at 1440px viewport.

**Top Navigation:** Standard iPaaS nav.

**Split Panel Layout:**

**Left Panel (360px, white, right border) -- Webhook Registry:**
- Header (48px): "Webhooks" (16px 600), "New Webhook" button (Blue-600 primary, plus icon)
- Filter row: Search input (full width, "Search webhooks..."), filter dropdown: All / Active / Inactive
- Webhook list (scrollable):
  - Each webhook item (72px, full width):
    - Status dot: Green (active) / Red (inactive) / Amber (error)
    - Name: "Lead Intake Hook" (14px 600)
    - URL: "https://api.billyr..." (12px JetBrains Mono Slate-500, truncated)
    - Last triggered: "2m ago" (12px Slate-400) + delivery count badge "247 deliveries"
    - Click to select (Blue-50 background when selected)
  - Sample webhooks:
    1. "Lead Intake Hook" (active, 2m ago, 247 deliveries)
    2. "Order Processing" (active, 15m ago, 1,847 deliveries)
    3. "GitHub PR Events" (active, 1h ago, 89 deliveries)
    4. "Stripe Payment Events" (error, 3h ago, 523 deliveries, red warning icon)
    5. "Custom: KYC Callback" (inactive, 2d ago, 12 deliveries)

**Right Panel (fluid ~1080px, white) -- Webhook Inspector:**
- Header: Selected webhook "Lead Intake Hook" (20px 600), URL display (14px JetBrains Mono, full URL with copy button), status badge "Active", "Edit" button, "Delete" button (red ghost)
- Tab bar: "Recent Deliveries" (active) | "Test" | "Configuration" | "Signing"

- **Recent Deliveries** tab:
  - Filter bar: Status dropdown (All/Succeeded/Failed/Retried), Date range picker
  - Delivery timeline (left side, 200px):
    - Vertical timeline with delivery items:
      - Each: Status icon (green check / red X / amber retry), delivery ID truncated, timestamp "10:42 AM", status code "200" (green) or "500" (red)
      - Failed deliveries highlighted with Red-50 background
    - Sample: 10 deliveries alternating success/failure
  - Selected delivery detail (right side, fluid):
    - Summary bar: "Delivery #247" | Status "200 OK" (Green-50 Green-700 badge) | Duration "342ms" | Attempts "1" | Signature "Valid" (green check)
    - **Request** section:
      - Method + URL: "POST https://api.acme.com/hooks/lead-intake" (14px JetBrains Mono)
      - Headers (collapsible): JSON viewer showing Content-Type, X-Hook-Secret, X-Signature, X-Timestamp, X-Tenant-ID
      - Body: JSON viewer with syntax highlighting showing the webhook payload:
        ```json
        {
          "event": "lead.created",
          "timestamp": "2026-02-23T10:42:01Z",
          "data": {
            "lead_id": "lead-2847",
            "email": "contact@example.com",
            "name": "Jane Doe",
            "source": "website_form",
            "score": 85
          }
        }
        ```
    - **Response** section:
      - Status: "200 OK" (green badge) + Duration "342ms"
      - Headers (collapsible)
      - Body: JSON viewer with response payload
    - **Signature Verification** section:
      - Algorithm: "HMAC-SHA256"
      - Expected signature (monospace, masked by default, show toggle)
      - Computed signature match: green check "Signatures match"
      - Timing: "Timestamp within 5-minute window: Valid"

- **Test** tab:
  - Payload editor (full width, 400px height): Monaco editor with JSON syntax highlighting, pre-populated with sample payload matching the webhook's schema
  - Headers editor: editable key-value pairs for custom headers
  - "Send Test" button (Blue-600 primary, 48px, send icon)
  - Response viewer: Status code badge, headers, body JSON viewer, duration, signature details
  - "Load from recent delivery" dropdown to pre-populate test data

- **Configuration** tab:
  - Webhook URL: editable text input with copy button
  - Events: Checkboxes for subscribed event types ("lead.created", "lead.updated", "lead.deleted", etc.)
  - Status: Active/Inactive toggle
  - Retry policy: Max retries input (default 3), Backoff strategy dropdown (exponential/linear), Timeout input "30s"
  - Rate limit: "100 requests per minute" with slider
  - "Save Configuration" button (primary)

- **Signing** tab:
  - Signing secret: masked field "*****...abc1" with "Copy" and "Rotate" buttons (rotate requires confirmation)
  - Algorithm: "HMAC-SHA256" (read-only badge)
  - Verification guide:
    - Code snippets with language tabs: TypeScript / Python / Go / cURL
    - TypeScript example:
      ```typescript
      const crypto = require('crypto');
      const signature = crypto
        .createHmac('sha256', signingSecret)
        .update(timestamp + '.' + payload)
        .digest('hex');
      ```
  - "Last rotated: Feb 10, 2026" timestamp
  - "Download verification library" link

**States:** Webhook selected with recent deliveries, webhook with all failures (red theme), test tab with response, configuration being edited, new webhook creation wizard (3-step: Name + URL, Events, Signing Secret), signing secret rotation confirmation modal.

---

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Workflow List (390px)

Design the workflow management list for 390px mobile viewport.

**Status Bar (44px, system).**

**App Header (56px):**
- Left: Hamburger menu (opens sidebar drawer)
- Center: "Workflows" (18px 600)
- Right: Search icon, "+" create workflow icon button

**Filter Bar (44px, horizontal scroll):**
- Filter pills: "All" (active), "Active", "Draft", "Paused", "Failed", "My Workflows"

**Workflow List (scrollable, remaining height minus bottom nav):**
- Each workflow card (100px height, 16px horizontal margin, white, Level-1 shadow, 8px radius, 8px vertical margin):
  - Top row: Workflow icon (zap, 20px, colored by status), name (14px 600 truncated) "Lead Intake Pipeline", status badge (right-aligned): "Active" (Green-50, 12px) or "Draft" (Slate-100)
  - Middle row: Description (12px Slate-500): "Webhook > Validate > Enrich > CRM"
  - Bottom row: "v1.2.0" (12px Slate-400) | "Last run: 5m ago" (12px Slate-400) | Success rate "99.2%" (12px Green-600)
  - Right edge: Chevron right (navigate to workflow detail)
- Sample workflows:
  1. "Lead Intake Pipeline" -- Active, v1.2.0, 5m ago, 99.2%
  2. "Order Processing" -- Active, v2.0.1, 15m ago, 97.8%
  3. "Finance ETL" -- Active, v1.0.0, 6h ago (scheduled), 100%
  4. "Social Monitoring" -- Paused (amber badge), v0.9.0, 2d ago, 94.5%
  5. "New Employee Onboarding" -- Draft (gray badge), v0.1.0, never run
  6. "Stripe Webhook Handler" -- Active, v1.1.0, 2m ago, 98.1%
- Pull-to-refresh indicator

**Workflow Detail (pushed view):**
- Header: Back arrow + workflow name + edit icon + more (kebab)
- Status card: Status badge + version + last run + success rate + active since
- "Run Now" button (Blue-600 full width) + "Pause" button (Amber-600 outline, below)
- "Execution History" section:
  - Last 10 executions as a list: status icon (green check/red X) + timestamp + duration + trigger type
  - Tap to expand: shows step-by-step execution with input/output
- "Configuration" section: read-only summary of trigger + steps
- "Edit in Studio" link (opens desktop view warning on mobile)

**FAB:** Blue-600 "+" (create new workflow), bottom-right. Options: "Start from template", "Start from scratch", "Import JSON".

**Bottom Navigation (80px with safe area):**
- 5 tabs: Workflows (zap, active), Connectors (puzzle), Events (radio), Monitoring (activity), More (grid)

**States:** Workflow list loaded, empty ("Create your first workflow" with illustration), workflow detail, execution history expanded, search overlay with results.

---

#### Prompt F-010: Mobile Connector Browser (390px)

Design the connector marketplace for 390px mobile.

**App Header (56px):**
- Left: Hamburger menu
- Center: "Connectors" (18px 600)
- Right: Search icon, filter funnel icon

**Search Bar (expanded when search icon tapped, 48px):**
- Full-width with back arrow and clear X, results update as typing

**Category Chips (horizontal scroll, 44px):**
- "All" (active), "CRM", "Finance", "Communication", "Database", "DevOps", "AI/ML", "Custom"

**Connector Cards (single column, scrollable):**
- Each card (full width minus 32px, 140px height, white, Level-1, 8px radius, 12px margin-bottom):
  - Left: Connector icon (40px circle with colored background)
  - Content (flex): Name (14px 600) "Salesforce CRM", author (12px Slate-500) "by BillyRonks", description (12px Slate-600 2 lines), bottom row: stars "4.7" + version "v2.1.0" + badge "Verified" (if applicable)
  - Right: "Install" button (Blue-600 pill, 36px height) or green check "Installed" badge
- Sample connectors: 8-10 cards visible with scroll

**Connector Detail (pushed view, full screen):**
- Large icon (64px centered) + name (20px 600) + author + badges row
- Tab bar (horizontal scroll): "Overview" | "Actions" | "Config" | "Reviews"
- "Install" CTA button (Blue-600, full width, sticky bottom)
- Content per tab scrollable above sticky button

**Filter Sheet (bottom sheet when funnel tapped):**
- Category checkboxes, Status radio buttons, Rating selector, "Apply Filters" button + "Clear" link

**Bottom Navigation:** Same as F-009 with Connectors tab active.

**States:** Connector list loaded, filtered results, search results, connector detail, installation in progress, installed connectors tab.

---

#### Prompt F-011: Mobile Monitoring (390px)

Design the monitoring dashboard for 390px mobile.

**App Header (56px):**
- Left: Hamburger menu
- Center: "Monitoring" (18px 600)
- Right: Refresh icon, time range "24h" dropdown

**Health Summary Cards (horizontal scroll, 120px height):**
- 4 cards (160px wide each, scrollable):
  - "Latency" -- "342ms" (Green) -- sparkline
  - "Traffic" -- "2,847/hr" -- sparkline
  - "Errors" -- "0.23%" (Green) -- sparkline
  - "Workers" -- "67%" (Amber) -- progress arc

**Active Alerts Section (expandable):**
- Header: "Alerts" + count badge "3" (Red-500) + expand/collapse chevron
- Alert cards (full width, stacked):
  - Each: Severity badge (CRITICAL/WARNING) + message (14px, 2 lines) + time + "View" link
  - Swipe right to acknowledge

**Execution Chart (full width, 200px):**
- Simplified stacked bar chart: green (success) + red (failures) over 24h
- "20,431 executions today -- 99.77% success" summary below

**Top Failures List:**
- Header: "Top Failures" + "View all" link
- List of 5 failing workflows: name + tenant + failure count + error rate bar

**Quick Actions (sticky bottom bar, above nav):**
- "Scale Workers" button + "Replay DLQ" button + "View Logs" button

**Bottom Navigation:** Same as F-009 with Monitoring tab active.

**States:** Healthy dashboard (green dominant), critical alerts (red dominant), loading skeleton, empty (new deployment).

---

#### Prompt F-012: Mobile Event Stream (390px)

Design the event stream monitor for 390px mobile.

**App Header (56px):**
- Left: Back arrow
- Center: "Events" (18px 600)
- Right: Pause/Play toggle, filter funnel icon

**Topic Selector (44px, horizontal scroll chips):**
- "All Topics", "tenant.acme.orders", "tenant.acme.crm", "DLQ" (red badge count)

**Event Stream (scrollable list):**
- Each event row (64px, full width):
  - Left: Status dot (12px) -- green (delivered), red (failed), amber (DLQ)
  - Content: Event type "order.created" (14px 600) + topic (12px Slate-500) + timestamp "10:42:01" (12px Slate-400)
  - Right: Size "1.2KB" (12px Slate-400), chevron right
  - New events: slide-in from top animation, Blue-50 flash
- Tap event: pushes to detail view
  - Full CloudEvents envelope JSON viewer (collapsible sections: metadata + data)
  - "Replay" button if DLQ event
  - "Copy" button for payload

**DLQ Tab (accessible via "DLQ" chip):**
- Filtered to DLQ events only
- Each row: event type + failure reason (truncated) + retry count + "Replay" button
- "Replay All" action in header

**Filter Sheet (bottom sheet):**
- Topic multi-select, Event type multi-select, Time range, Status filter
- "Apply" button

**States:** Stream flowing (events prepending), stream paused, DLQ view, event detail view, empty stream ("No events in this time window").

---

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Tablet Workflow Studio (1024px)

Design the workflow canvas for 1024px tablet viewport (iPad landscape).

**Layout:** Two-panel (sidebar hidden behind hamburger).

**Canvas Area (full 1024px):**
- Same canvas rendering as desktop but with larger touch-friendly node connection ports (16px circles vs 12px)
- Node palette: accessible via "+" FAB (bottom-left, 56px Blue-600 circle) that opens a bottom sheet with categorized node list
- Selected node: configuration opens as a bottom sheet (60vh) rather than right panel
- Pinch-to-zoom on canvas (multi-touch gesture support)
- Canvas toolbar: floating pill at top-center with Undo/Redo/Zoom/Fit/Grid controls

**Node Configuration Bottom Sheet (60vh):**
- Drag handle at top for dismiss
- Node name + type badge
- Configuration form fields (full width)
- Input mapping section
- Output preview
- "Test Step" button
- Scrollable content area

**Execution Log:** Bottom sheet (expandable from 48px collapsed bar to 40vh), same table format as desktop.

**Top Bar:** Workflow name + status + Save/Test/Activate buttons (icons only without text to save space).

**States:** Canvas with workflow, node selected (bottom sheet open), execution running, empty canvas with "+ Add a trigger" prompt.

---

#### Prompt F-014: Tablet Monitoring Dashboard (1024px)

Design the monitoring dashboard at 1024px tablet viewport.

**Layout:** Single column with responsive cards.

**Golden Signals:** 2x2 grid of metric cards (instead of 4-across on desktop). Each card: label + large value + sparkline + trend.

**Execution Chart:** Full width, 240px height, same stacked bar chart.

**Two-Column Below:**
- Left (50%): Active Alerts card (stacked list)
- Right (50%): Tenant Usage card (horizontal bar chart)

**Below Two-Column:**
- Full width: Top Failing Workflows table
- Full width: Connector Health grid (4 columns of health indicators)

**Event Backbone:** Full width card with key metrics and consumer lag mini chart.

**Auto-refresh indicator:** Subtle pulse animation on the refresh icon in header.

**States:** Dashboard loaded, alert drill-down (pushes to detail view), tenant detail.

---

### 3.5 NexumFlow DAG Builder (Desktop)

#### Prompt F-015: NexumFlow Visual DAG Builder (1440px)

Design the NexumFlow DAG pipeline builder at 1440px viewport -- a sequential node-based pipeline builder distinct from the Activepieces visual canvas.

**Top Bar (56px):**
- Title: "NexumFlow: Lead Enrichment Pipeline" (editable)
- Status: "Active" green badge
- Tenant: "tenant-acme" badge
- "Execute" button (Blue-600, play icon), "Save" button, "View Executions" button

**Canvas (full width below top bar):**
- Vertical DAG layout (top-to-bottom flow, centered):
  - Node 1 (top): "HTTP: Fetch Lead" (Cyan-500 left border, globe icon, 240x90px). Subtitle: "GET /api/leads/{id}". Output badge: "JSON object". Connected via vertical line with arrow to node 2.
  - Node 2: "LLM: Classify Lead" (Violet-500 left border, brain icon). Subtitle: "Prompt: Classify this lead as hot/warm/cold". Input badge: "Previous output". PII warning icon (amber shield). Connected to node 3.
  - Node 3: "Temporal: Enrich via CRM" (Blue-500 left border, clock icon). Subtitle: "Workflow: LeadIntakeWorkflow". Input badge: "LLM classification + lead data". Connected to node 4.
  - Node 4: "MCP Tool: Update IDE" (Indigo-500 left border, tool icon). Subtitle: "Tool: database-assistant". Input badge: "Enriched lead".
- Connection lines: 2px Slate-400 vertical lines with downward arrows, dashed animated when executing
- Between each node: "+" add node button (32px circle, Blue-600, appears on hover of the gap)
- Node selection: Blue glow border + right panel opens

**Right Configuration Panel (360px, when node selected):**
- Header: Node name + type icon + type label
- **HTTP Node config:** Method dropdown, URL input, Headers key-value editor, Body editor (JSON), Authentication dropdown (None/API Key/OAuth2/Bearer)
- **LLM Node config:** Model selector dropdown (GPT-4/Claude/Llama), Prompt template editor (with {input} variable highlighting), Temperature slider, Max tokens input, PII guard toggle (default on), "Preview prompt" button showing resolved template
- **Temporal Node config:** Workflow name dropdown (from registered workflows), Task queue input (auto-filled with tenant), Input mapping editor, Timeout input, Signal config
- **MCP Node config:** Tool selector (from registered MCP tools), Parameters form (dynamic based on tool schema), Backend indicator (Activepieces/Temporal/LangGraph)
- **Common sections:** Input (shows JSON from previous node), Output schema (expected structure), Test button, Execution history for this node

**Execution View (bottom panel or overlay):**
- Sequential execution progress: horizontal step indicators (circles with lines)
  - Node 1: green check (50ms), Node 2: green check (1.2s), Node 3: blue pulse (running, 3.4s...), Node 4: gray (pending)
- Expandable: shows input/output JSON for completed nodes
- Error state: red X on failed node with error message and "Retry from this step" button

**States:** Pipeline configured, pipeline executing (animated flow), execution completed (all green), execution failed at step 2 (node 2 red with error, remaining nodes gray), empty builder ("Add your first node to start building a pipeline").

---

### 3.6 Tenant Administration (Desktop)

#### Prompt F-016: Tenant Administration Panel (1440px)

Design the tenant management administration panel at 1440px viewport.

**Top Navigation:** Standard iPaaS nav with Settings/Admin tab active.

**Left Sidebar (240px, within admin context):**
- "Administration" header (16px 600)
- Nav items: Overview (active), Tenants, Users, Quotas, API Keys, Secrets, Audit Log, Billing

**Main Content: Tenants List (when Tenants nav selected):**
- Header: "Tenant Management" (H2) + "Onboard New Tenant" button (Blue-600 primary)
- Stats bar: "12 active tenants | 3 trial | 47 total workflows | 99.8% platform availability"
- Tenant table (full width, white card):
  - Columns: Tenant ID | Name | Status | Namespace | Workflows | Executions (24h) | Quota Usage | Actions
  - Row 1: "tenant-acme" | "Acme Corp" | "Active" (green) | "ns-acme-prod" | "12" | "8,234" | Progress bar 82% (amber) | "Manage" dropdown
  - Row 2: "tenant-beta" | "Beta Industries" | "Active" | "ns-beta-prod" | "8" | "5,120" | 51% (green) | "Manage"
  - Row 3: "tenant-gamma" | "Gamma Solutions" | "Trial" (blue badge) | "ns-gamma-trial" | "3" | "890" | 9% (green) | "Manage"
  - Row 4: "tenant-delta" | "Delta Corp" | "Suspended" (red badge) | "ns-delta-prod" | "5" | "0" | 0% | "Manage"
  - Sortable columns, search/filter bar above table

**Tenant Detail (when "Manage" clicked, pushed view or drawer):**
- Header: Tenant name + status badge + "Edit" button
- Tabs: Overview | Configuration | Quotas | API Keys | Secrets | Audit Log
- **Overview:** Card grid with: Active workflows count, Today's executions, Quota usage, Active users, Namespace info, Created date
- **Quotas:** Editable quota form: Daily executions (slider + input), Max workflows, Max connectors, Storage (MB), API rate limit (req/min), Worker allocation
- **API Keys:** Table of tenant's API keys with create/rotate/revoke actions
- **Secrets:** Vault-managed secrets list (masked) with rotate/delete actions
- **Audit Log:** Filterable table of all tenant actions with timestamp, user, action, resource, result

**Onboard New Tenant Wizard (modal, 640px, multi-step):**
- Step 1 "Basic Info": Tenant ID (slug), Display name, Admin email, Subscription tier dropdown (Trial/Standard/Enterprise)
- Step 2 "Configuration": Kubernetes namespace (auto-generated), Temporal namespace, Kafka topic prefix, default timezone, default quota template
- Step 3 "Seed": Checkbox list of workflow templates to seed, Default connectors to install
- Step 4 "Review & Provision": Summary of all settings + "Provision Tenant" button
- Progress indicator: 4-step numbered bar at top of modal
- Provisioning in progress: Animated checklist showing: "Creating namespace..." -> "Deploying workers..." -> "Configuring Keycloak realm..." -> "Seeding templates..." -> "Complete!"

**States:** Tenant list, tenant detail, onboarding wizard (each step), provisioning in progress, provisioning failed (error on specific step with retry).

---

## 4. Make Automation Prompts

### Prompt M-001: Connector Validation and Marketplace Publishing

**Trigger:** Webhook -- `POST /hooks/connector-submitted` fired by the Developer Portal when a connector author submits a new connector or version for marketplace review.

**Actions:**
1. Parse the submission payload: `connector_slug`, `version`, `openapi_spec_url`, `author_id`, `tenant_id`, `metadata` (categories, description, auth_type).
2. HTTP GET `{openapi_spec_url}` to download the OpenAPI specification. Validate it is parseable JSON/YAML.
3. HTTP POST to `{INTEGRATION_LAYER_API}/v1/tenants/{tenant_id}/connectors` with the connector metadata to register it in the Open Integration Layer.
4. HTTP POST to `{TEMPORAL_API}/v1/workflows/connector-validation` to start the Temporal validation workflow:
   - Activity 1: Schema validation (JSON Schema compliance, required fields, auth schemes present)
   - Activity 2: Rate limit configuration validation (backoff policy defined, retry limits set)
   - Activity 3: Security attestation (OAuth scopes minimal, secrets stored in Vault references, webhook signatures configured)
   - Activity 4: k6 load test (50 concurrent requests, measure p50/p95/p99 latency against SLO targets)
   - Activity 5: SBOM generation via Syft, vulnerability scan via Trivy
5. Poll `{TEMPORAL_API}/v1/workflows/{workflow_id}/status` every 30 seconds (max 10 minutes) until validation completes.
6. HTTP POST to `{CLICKHOUSE_API}/v1/insert` to record validation results in `connector_latency` and `connector_validation_results` tables with: connector_slug, version, test_results JSON, p99_latency, error_rate, security_score, sbom_hash.
7. If validation **passed** (all activities succeeded, p99 < SLO, error rate < 2%):
   a. HTTP PATCH `{INTEGRATION_LAYER_API}/v1/tenants/{tenant_id}/connectors/{slug}` to update status to "published" with quality badges.
   b. HTTP POST `{ARGOCD_API}/v1/sync` to trigger Argo CD sync that publishes the connector to Activepieces and Temporal catalogs.
   c. Send Slack notification to `#connectors`: "New connector published: {name} v{version} by {author} -- Quality: {score}/100 -- [View in Marketplace]({url})"
   d. Send email to author: "Your connector {name} v{version} has been published to the marketplace!"
8. If validation **failed**:
   a. HTTP PATCH status to "rejected" with failure reasons array.
   b. Create GitHub Issue in the connector's repository with title "Validation Failed: {connector} v{version}" and body containing each failed check with details and remediation guidance.
   c. Send email to author with failure summary and links to the GitHub issue and documentation.
   d. Send Slack notification to `#connectors`: "Connector validation failed: {name} v{version} -- {failure_count} issues found"
9. Emit `ipaas.connector.validation.completed` CloudEvent with connector_slug, version, result (pass/fail), duration, validation_details.

**Error Handling:** Temporal workflow timeout triggers an alert to the platform team. k6 load test infrastructure failures are retried once with a 2-minute delay. ClickHouse insert failures are queued for batch retry. All errors logged to the audit trail.

---

### Prompt M-002: Tenant Onboarding Automation

**Trigger:** Webhook -- `tenant.provisioning.requested` CloudEvent from the Admin API when a platform administrator initiates tenant onboarding.

**Actions:**
1. Parse payload: `tenant_id`, `display_name`, `admin_email`, `subscription_tier`, `timezone`, `quota_template`, `seed_templates[]`, `default_connectors[]`.
2. HTTP POST to `{KEYCLOAK_API}/admin/realms/billyronks/groups` to create a tenant group with `tenant_id` as the group name and set attributes: tier, timezone, created_date.
3. HTTP POST to `{KEYCLOAK_API}/admin/realms/billyronks/users` to create the tenant admin user with email, temporary password, and group membership. Set required action "UPDATE_PASSWORD".
4. HTTP POST to `{K8S_API}/api/v1/namespaces` to create a Kubernetes namespace `ns-{tenant_id}-prod` with labels: tenant_id, tier, managed_by=billyronks.
5. HTTP POST to `{K8S_API}/apis/apps/v1/namespaces/ns-{tenant_id}-prod/deployments` to deploy per-tenant Temporal worker pods using the Helm chart template for the subscription tier. Apply resource limits based on quota_template.
6. HTTP POST to `{TEMPORAL_API}/v1/namespaces` to create a Temporal namespace `{tenant_id}` with retention period (30 days standard, 90 days enterprise).
7. HTTP POST to `{KAFKA_API}/admin/topics` to create tenant-scoped Kafka topics: `tenant.{tenant_id}.workflows`, `tenant.{tenant_id}.connectors`, `tenant.{tenant_id}.audit`, `tenant.{tenant_id}.dlq` with configured partitions (3 standard, 6 enterprise) and retention (7d standard, 30d enterprise).
8. HTTP POST to `{VAULT_API}/v1/sys/mounts/secret/data/{tenant_id}` to create a Vault secret path for the tenant. Store initial secrets: Kafka credentials, Temporal TLS certs, default API key.
9. For each template in `seed_templates[]`: HTTP POST to `{ACTIVEPIECES_API}/v1/tenants/{tenant_id}/flows` to create the workflow from template JSON.
10. For each connector in `default_connectors[]`: HTTP POST to `{INTEGRATION_LAYER_API}/v1/tenants/{tenant_id}/connectors/{slug}/install` to install the connector in the tenant's workspace.
11. HTTP POST to `{CLICKHOUSE_API}/v1/insert` to initialize tenant analytics tables: `tenant_usage`, `connector_latency`, `audit_log` with the tenant_id partition.
12. Send welcome email to `admin_email`: Subject "Welcome to BillyRonks iPaaS -- {display_name}", body containing: login credentials (temporary), platform URL, quickstart guide link, support channel, mobile app links.
13. Send Slack notification to `#platform-ops`: "Tenant onboarded: {display_name} ({tenant_id}) -- Tier: {tier} -- Templates: {count} -- Connectors: {count}"
14. Emit `ipaas.tenant.onboarded` CloudEvent with tenant_id, provisioning_duration_ms, services_provisioned (list), template_count, connector_count.
15. HTTP POST to `{OBSERVABILITY}/v1/events` logging the full provisioning timeline for SLO tracking (target: < 15 minutes).

**Error Handling:** Each step is idempotent with a unique idempotency key. If step N fails, the workflow records the last successful step and can be resumed from step N+1. Kubernetes namespace creation failure triggers an immediate alert (cannot proceed). Kafka topic creation failure retries 3 times. Vault path creation failure is critical and escalates to security team. Partial onboarding (some templates failed to seed) completes with a warning email to the admin listing which templates need manual setup.

---

### Prompt M-003: Workflow Execution Failure Alert and Auto-Remediation

**Trigger:** Webhook -- `workflow.failed` CloudEvent from the Redpanda event backbone.

**Actions:**
1. Parse the failure event: `tenant_id`, `workflow_id`, `workflow_type`, `error_code`, `error_message`, `retryable`, `attempt`, `trace_id`, `occurred_at`.
2. HTTP GET `{CLICKHOUSE_API}/v1/query` to check recent failure pattern: "SELECT count(*) as failure_count FROM workflow_runs WHERE workflow_id = '{workflow_id}' AND status = 'failed' AND occurred_at > now() - INTERVAL 1 HOUR". This determines if this is an isolated failure or a pattern.
3. **If failure_count < 3 and retryable = true** (isolated, retryable):
   a. HTTP POST `{TEMPORAL_API}/v1/workflows/{workflow_id}/signal` with signal name "retry" to trigger automatic retry via the workflow's built-in retry mechanism.
   b. Log the auto-retry to ClickHouse audit trail.
   c. No alert sent (auto-remediation handled).
4. **If failure_count >= 3 and failure_count < 10** (pattern emerging):
   a. HTTP POST to Slack `#workflow-alerts`: "WARNING: Workflow '{workflow_type}' for tenant {tenant_id} has failed {failure_count} times in the last hour. Error: {error_message}. Trace: {trace_id}. [View in Temporal UI]({url})"
   b. HTTP POST to `{PAGERDUTY_API}/v2/enqueue` to create an incident with severity "warning".
   c. If error_code is "AUTH_EXPIRED" or "401": HTTP POST to `{VAULT_API}/v1/secret/data/{tenant_id}/oauth/{provider}/rotate` to trigger automatic token refresh. Then retry the workflow.
   d. If error_code is "RATE_LIMITED" or "429": HTTP POST to `{TEMPORAL_API}/v1/workflows/rate-limit-shield/start` with the pending batch to route through the RateLimitShield workflow with appropriate spacing.
5. **If failure_count >= 10** (critical pattern):
   a. HTTP POST to Slack `#platform-incidents`: "CRITICAL: Workflow '{workflow_type}' for tenant {tenant_id} has failed {failure_count} times in the last hour. Automatic remediation exhausted. Manual intervention required."
   b. HTTP POST to PagerDuty with severity "critical" and escalation policy.
   c. HTTP PATCH `{ACTIVEPIECES_API}/v1/tenants/{tenant_id}/flows/{workflow_id}` to pause the workflow automatically (prevent cascading failures).
   d. Send email to tenant admin: "Your workflow '{workflow_type}' has been automatically paused due to repeated failures. Our team is investigating. Reference: {trace_id}."
6. HTTP POST to `{CLICKHOUSE_API}/v1/insert` table `workflow_failure_events` with: tenant_id, workflow_id, error_code, error_message, failure_count, remediation_action, trace_id, occurred_at.
7. Emit `ipaas.workflow.failure.processed` CloudEvent with remediation_action taken and result.

**Error Handling:** Slack API failure falls back to email alerts. PagerDuty failure falls back to OpsGenie. ClickHouse insert failure queues to DLQ for batch retry. Token rotation failure escalates immediately to security team. Rate limit shield workflow failure creates a manual runbook task.

---

### Prompt M-004: Nightly Connector Health Check and SLO Reporting

**Trigger:** Schedule -- Daily at 00:30 AM Africa/Lagos.

**Actions:**
1. HTTP GET `{INTEGRATION_LAYER_API}/v1/connectors?status=published` to fetch all published connectors with their SLO targets.
2. For each connector (batched, 10 concurrent):
   a. HTTP POST `{CONNECTOR_CLI_API}/v1/validate` with the connector's OpenAPI spec URL to run the `billyronks-connector validate` command against the sandbox environment.
   b. HTTP POST `{K6_API}/v1/tests/run` with the connector's load profile to execute a k6 load test (10 virtual users, 2 minutes, measuring p50/p95/p99 latency and error rate).
   c. Collect results: latency_p99, error_rate, availability_percent, response_time_distribution.
3. HTTP POST `{CLICKHOUSE_API}/v1/insert` batch to update `connector_latency` table with nightly results for each connector.
4. Compare results against SLO targets for each connector:
   - If p99 > SLO latency target: flag as "degraded"
   - If error rate > 2%: flag as "unhealthy"
   - If validation failed: flag as "broken"
5. Generate SLO report:
   a. HTTP POST `{AI_SERVICE}/v1/summarize` with the nightly results to generate a human-readable summary: "3 connectors degraded, 1 broken, 138 healthy. Salesforce p99 increased 15% week-over-week. Stripe error rate spiked at 02:00 due to planned maintenance."
   b. Create a Markdown report document with: executive summary, per-connector SLO table (name, SLO target, actual, status, trend), action items for degraded connectors.
6. For **degraded** connectors:
   a. Send Slack notification to `#connectors`: "Nightly check: {name} v{version} is degraded -- p99 latency {actual}ms exceeds SLO {target}ms. [View metrics]({grafana_url})"
   b. Create Jira ticket if degraded for 3+ consecutive nights with escalation priority.
7. For **broken** connectors:
   a. Send email to connector author with validation failure details and remediation steps.
   b. If broken for 7+ consecutive nights: HTTP PATCH to set status "deprecated" with 90-day deprecation notice.
   c. Notify all tenants using the connector via email: "Connector {name} has been marked for deprecation. Please migrate to an alternative."
8. HTTP POST to Grafana annotations API to mark the nightly check timestamp on dashboard panels.
9. Send daily SLO report email to platform-ops distribution list with the Markdown report.
10. Emit `ipaas.connectors.nightly_check.completed` CloudEvent with: total_checked, healthy_count, degraded_count, broken_count, check_duration_ms.

**Error Handling:** Individual connector validation timeout (5 minutes) does not block other connectors. k6 infrastructure failure for a connector results in "check_skipped" status and retry in the next nightly run. ClickHouse batch insert failure retries 3 times. Report generation AI failure falls back to a template-based report without the summary narrative.

---

### Prompt M-005: Schema Registry Change Detection and Backward Compatibility

**Trigger:** Webhook -- `schema.version.uploaded` CloudEvent from the Schema Registry when a new schema version is submitted.

**Actions:**
1. Parse payload: `schema_name`, `new_version`, `previous_version`, `schema_type` (Avro/JSON Schema/Protobuf), `uploaded_by`, `tenant_id`.
2. HTTP GET `{SCHEMA_REGISTRY}/v1/schemas/{schema_name}/versions/{previous_version}` to fetch the previous schema definition.
3. HTTP GET `{SCHEMA_REGISTRY}/v1/schemas/{schema_name}/versions/{new_version}` to fetch the new schema definition.
4. HTTP POST `{SCHEMA_REGISTRY}/v1/compatibility/check` with both schema versions to run automated compatibility check:
   - Full compatibility: no breaking changes, new optional fields only
   - Backward compatible: readers using old schema can read new data
   - Forward compatible: readers using new schema can read old data
   - Breaking: removed required fields, type changes, renamed fields
5. If **fully compatible**:
   a. HTTP PATCH `{SCHEMA_REGISTRY}/v1/schemas/{schema_name}/versions/{new_version}` to set status "approved".
   b. Send Slack notification to `#data-engineering`: "Schema update approved: {schema_name} v{new_version} -- fully compatible with v{previous_version}"
   c. HTTP POST to update all consumers using this schema (Kafka consumer groups) to be aware of the new version.
6. If **backward compatible only**:
   a. Set status "approved_with_warning".
   b. Generate migration guide: list of new fields, deprecated fields, recommended consumer updates.
   c. Send email to all tenant admins whose workflows consume this schema with the migration guide.
7. If **breaking change detected**:
   a. Set status "rejected" with breaking change details.
   b. Send email to uploader: "Schema {schema_name} v{new_version} rejected -- breaking changes detected: {changes_list}. Submit a new major version instead."
   c. Create GitHub Issue with title "Breaking Schema Change: {schema_name}" and body containing the diff, affected consumers list, and migration path recommendations.
   d. Block the schema from being used in production topics.
8. Generate schema diff visualization data: additions (green), removals (red), modifications (amber) as a structured JSON for the Schema Registry UI to render.
9. HTTP POST to `{CLICKHOUSE_API}/v1/insert` table `schema_changes` with: schema_name, old_version, new_version, compatibility_result, changes_count, uploaded_by, tenant_id, checked_at.
10. Emit `ipaas.schema.compatibility.checked` CloudEvent with result and affected consumer count.

**Error Handling:** Schema Registry API timeout retries 3 times. Compatibility check failure defaults to "manual review required" status and alerts the data engineering team. Consumer notification failures are logged but do not block the schema approval.

---

### Prompt M-006: DLQ Auto-Replay and Escalation

**Trigger:** Schedule -- Every 15 minutes.

**Actions:**
1. HTTP GET `{KAFKA_API}/admin/topics?pattern=tenant.*.dlq` to list all DLQ topics across tenants.
2. For each DLQ topic, HTTP GET `{KAFKA_API}/v1/topics/{topic}/messages?limit=100&oldest=true` to fetch pending DLQ events.
3. For each DLQ event:
   a. Parse the original event metadata: original_topic, failure_reason, retry_count, first_failed_at, last_retried_at.
   b. Calculate age: `now() - first_failed_at`.
   c. **If retry_count < 5 and age < 24 hours and failure_reason is transient** (timeout, rate limit, temporary unavailability):
      - HTTP POST `{KAFKA_API}/v1/topics/{original_topic}/messages` to replay the event back to its original topic.
      - Increment retry_count metadata on the event.
      - Log replay attempt to ClickHouse audit trail.
   d. **If retry_count >= 5 or age >= 24 hours**:
      - Mark as "requires_manual_intervention" in DLQ metadata.
      - Send Slack alert to `#data-ops`: "DLQ event {event_id} in {topic} has exhausted automatic retries. Failure: {failure_reason}. Age: {age_hours}h. [View event]({url})"
      - Create Jira ticket with DLQ event details, original payload, failure trace, and suggested remediation.
   e. **If failure_reason is permanent** (schema validation error, missing required field, invalid data):
      - Mark as "dead" -- will not be auto-replayed.
      - Move to a separate archive topic `tenant.{id}.dlq.archive` for compliance retention.
      - Send email to tenant admin with the failed event details and data fix guidance.
4. Aggregate DLQ statistics per tenant: total pending, auto-replayed, escalated, dead.
5. HTTP POST `{CLICKHOUSE_API}/v1/insert` table `dlq_processing_runs` with per-tenant stats and processing duration.
6. If any tenant has > 50 pending DLQ events: send a summary alert to `#platform-ops` with tenant-level breakdown.
7. Emit `ipaas.dlq.processing.completed` CloudEvent with total_processed, replayed_count, escalated_count, dead_count.

**Error Handling:** Individual event replay failure increments retry_count and the event remains in DLQ for the next cycle. Kafka API failures trigger a full-cycle retry in 15 minutes. ClickHouse audit failures are logged to local file and batch-inserted on next cycle. Jira ticket creation failure falls back to Slack-only escalation.

---

## 5. Prompt Usage Guidelines

### 5.1 How To Use Figma Prompts
1. Open Figma and create a new design file named "ERP-iPaaS-{ScreenName}-{Breakpoint}".
2. Set the frame dimensions to the target breakpoint (1440px, 1024px, or 390px width).
3. Copy the relevant prompt text and paste it into Figma Make Mode or provide it to the design team as a specification.
4. Design all stated states (default, loading, empty, error, hover, active, disabled, running, completed, failed) as separate frames within the same page.
5. Apply the design tokens from F-001 consistently. Canvas-specific tokens (grid, nodes, connections) are critical for the Integration Studio and NexumFlow screens.
6. Use Auto Layout in Figma for all layouts to ensure responsive behavior.
7. For canvas-based screens (F-003, F-015): use Figma's vector tools to create node shapes, bezier connections, and grid backgrounds. Prototype with Smart Animate for node drag and connection creation.
8. Export as separate pages: "Desktop", "Tablet", "Mobile" with artboards per screen and state.

### 5.2 How To Use Make Prompts
1. Import each workflow into Make.com as a new scenario.
2. Configure the webhook trigger URL and register it in the iPaaS event system (Kafka topic subscriber or direct webhook).
3. Replace all `{API}` placeholders with actual service endpoints from the deployment configuration.
4. Set up error handling modules with the specified retry policies and escalation paths.
5. Test each workflow with sample CloudEvent payloads before activating.
6. Monitor execution logs in Make.com and cross-reference with ClickHouse analytics for the first 48 hours.
7. Set up Make.com monitoring: email alerts for scenario failures, Slack webhook for critical path failures.

### 5.3 Prompt Modification Rules
- All prompts are additive: extend, do not reduce scope.
- Color values and node shapes must match design tokens in F-001 exactly.
- Any new component should be added to the component library (F-002) before use in screens.
- Canvas interactions (drag, connect, zoom) must maintain 60fps performance target.
- Mobile prompts must maintain WCAG 2.1 AA touch target minimums (44x44px).
- Make workflow prompts must include error handling for every HTTP action.
- All Make workflows must emit a CloudEvent at completion for observability.

---

## 6. Output Packaging Convention

| Artifact | Format | Naming Convention |
|----------|--------|-------------------|
| Figma design file | .fig | `ERP-iPaaS-{Screen}-v{version}.fig` |
| Design tokens | JSON | `ipaas-design-tokens-v{version}.json` |
| Component library | .fig | `ERP-iPaaS-Components-v{version}.fig` |
| Icon set | SVG sprite | `ipaas-icons-v{version}.svg` |
| Canvas node library | .fig | `ERP-iPaaS-Canvas-Nodes-v{version}.fig` |
| Prototype | Figma link | `ERP-iPaaS-Prototype-{flow}-v{version}` |
| Make scenario export | JSON | `make-ipaas-{workflow}-v{version}.json` |
| Handoff spec | PDF + Zeplin/Figma dev mode | `ERP-iPaaS-Handoff-{Sprint}.pdf` |

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Lighthouse Performance | >= 95 | Chrome DevTools audit on each route |
| Lighthouse Accessibility | >= 95 | Chrome DevTools audit on each route |
| First Contentful Paint (FCP) | < 1.5s | WebPageTest, field data via CrUX |
| Largest Contentful Paint (LCP) | < 2.5s | WebPageTest, field data via CrUX |
| First Input Delay (FID) | < 100ms | Field data via CrUX |
| Cumulative Layout Shift (CLS) | < 0.1 | Field data via CrUX |
| Time to Interactive (TTI) | < 4.0s | Lighthouse audit |
| Initial route JS bundle (gzip) | < 220KB | Webpack bundle analyzer |
| Canvas renderer (lazy, gzip) | < 100KB | Separate bundle analysis |
| Canvas node drag latency | < 16ms (60fps) | Performance.mark/measure |
| Canvas zoom/pan latency | < 16ms | requestAnimationFrame timing |
| Node connection creation | < 100ms | User-perceived timing |
| Workflow save (autosave) | < 500ms | Network timing API |
| Event stream append | < 50ms per event | Performance.mark/measure |
| JSON viewer render (100KB payload) | < 200ms | Render timing |
| Connector marketplace search | < 300ms | Search response timing |
| API response (read p99) | < 200ms | Backend SLO monitoring |
| API response (write p99) | < 500ms | Backend SLO monitoring |
| Workflow execution trigger-to-start | < 500ms | End-to-end timing |
| Virtual scroll (10,000 events) | 60fps | Scroll performance profiler |

---

## 8. AIDD Handoff Gate Template

Before any screen is considered ready for development, it must pass the following gate:

| Gate Item | Requirement | Verified By |
|-----------|-------------|-------------|
| Visual fidelity | All states designed (default, loading, empty, error, hover, active, disabled, running, completed, failed) | Design Lead |
| Responsive coverage | Desktop (1440px), Tablet (1024px), Mobile (390px) artboards present | Design Lead |
| Dark mode | All screens have dark mode variants | Design Lead |
| Accessibility | WCAG 2.1 AA color contrast verified, focus states designed, screen reader annotations added, canvas keyboard navigation documented | Accessibility QA |
| Touch targets | All interactive elements >= 44x44px on mobile and tablet; canvas ports >= 16px on tablet | Design Lead |
| Design tokens | All colors, typography, spacing, elevation, and canvas tokens use tokens from F-001 | Design System Owner |
| Component reuse | All UI elements reference components from F-002 library | Design System Owner |
| Canvas interactions | Drag-drop, zoom, pan, connect, context menu behaviors documented with timing and easing | Design Lead + Frontend Lead |
| Interaction spec | Animations, transitions, and micro-interactions documented with timing (default 200ms ease-out) | Design Lead |
| Content | Real data (not lorem ipsum); realistic connector names, workflow names, event payloads, tenant IDs | Content Strategist |
| Performance annotations | Lazy loading boundaries, virtual scroll zones, canvas viewport culling zones annotated | Frontend Lead |
| Analytics annotations | All tracking events (click, view, error, canvas_interaction) annotated on the design | Product Manager |
| Handoff artifacts | Figma dev mode enabled, spacing/sizing tokens exported, canvas node SVGs exported, interaction prototypes linked | Design Lead |
| Prototype | Interactive prototype covering primary flow (create workflow, connect nodes, test, activate) linked from design file | Design Lead |
| Security annotations | Secret fields masked, PII warning locations, tenant isolation indicators marked | Security Lead |
| Approval | Sign-off from Product Manager, Engineering Lead, Design Lead, and Security Lead | All four |

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Complete rewrite -- comprehensive Figma & Make prompts covering Integration Studio, Connector Marketplace, Monitoring Dashboard, Event Stream Monitor, ETL Pipeline Builder, Webhook Debugger, NexumFlow DAG Builder, Tenant Administration across desktop, tablet, and mobile breakpoints. 6 Make automation workflows covering connector validation, tenant onboarding, failure remediation, nightly health checks, schema compatibility, and DLQ processing. |

---

*Generated by AIDD System on 2026-02-23*
