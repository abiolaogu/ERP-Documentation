# ERP-Observability Figma Design Prompts

> **Document ID:** ERP-OBS-FIG-015
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [01-Technical-Writeup.md](./01-Technical-Writeup.md), [07-User-Manual.md](./07-User-Manual.md), [12-High-Level-Design.md](./12-High-Level-Design.md)

---

## 1. Overview

This document provides detailed Figma Make design prompts for every major screen and component in the ERP-Observability frontend. Each prompt is crafted to generate production-ready UI designs following the Refine.dev + Ant Design component library conventions with the OpenSASE ERP emerald green theme.

### Design System Foundation

| Property | Value |
|----------|-------|
| **Primary Color** | Emerald Green `#059669` |
| **Primary Hover** | `#047857` |
| **Primary Light** | `#D1FAE5` |
| **Secondary Color** | `#6366F1` (Indigo for accents) |
| **Error Color** | `#EF4444` |
| **Warning Color** | `#F59E0B` |
| **Success Color** | `#10B981` |
| **Info Color** | `#3B82F6` |
| **Background (Light)** | `#F9FAFB` |
| **Background (Dark)** | `#111827` |
| **Surface (Light)** | `#FFFFFF` |
| **Surface (Dark)** | `#1F2937` |
| **Text Primary (Light)** | `#111827` |
| **Text Primary (Dark)** | `#F9FAFB` |
| **Font Family** | Inter, system-ui, sans-serif |
| **Border Radius** | 8px (cards), 6px (inputs), 4px (buttons) |
| **Component Library** | Ant Design 5.x via Refine.dev |

### Layout Constants

| Element | Specification |
|---------|---------------|
| **Sidebar Width (Collapsed)** | 80px |
| **Sidebar Width (Expanded)** | 256px |
| **Top Header Height** | 64px |
| **Content Padding** | 24px |
| **Card Gap** | 16px |
| **KPI Card Height** | 120px |
| **Chart Min Height** | 320px |
| **Table Row Height** | 48px |
| **Breakpoints** | sm: 576px, md: 768px, lg: 992px, xl: 1200px, xxl: 1600px |

---

## 2. Global Shell & Navigation

### 2.1 Application Shell (Light Mode)

**Prompt:** Design the main application shell for an enterprise observability platform using Refine.dev layout conventions with Ant Design components. The left sidebar has a dark emerald green (`#059669`) background with white navigation icons and labels, collapsible to icon-only mode. Navigation items include: Dashboard, Metrics, Logs, Traces, Alerts, Infrastructure, Custom Dashboards, and Tenant Management, each with a corresponding Ant Design icon. The top header bar is white with a subtle bottom border, containing a global search input (Ant Design Search), a time range picker dropdown showing "Last 1h / Last 6h / Last 24h / Last 7d / Custom", a dark mode toggle switch, a notification bell with a red badge count, and a user avatar dropdown on the far right. The main content area has a light gray (`#F9FAFB`) background.

### 2.2 Application Shell (Dark Mode)

**Prompt:** Redesign the application shell for dark mode. The sidebar background becomes `#064E3B` (dark emerald), the top header becomes `#1F2937` with a subtle border of `#374151`, and the main content area background shifts to `#111827`. All text inverts to `#F9FAFB`, card surfaces become `#1F2937`, and borders become `#374151`. Maintain the same layout structure, navigation icons, and component placements as the light mode shell but ensure all Ant Design components use their dark theme token overrides. Charts and graphs should use bright, high-contrast colors against the dark background.

### 2.3 Breadcrumb & Page Header

**Prompt:** Design a reusable page header component that sits at the top of the content area, below the top header bar. It includes an Ant Design Breadcrumb showing the navigation path (e.g., "Dashboard > Metrics > CPU Usage"), a page title in 24px semibold, an optional subtitle/description in 14px gray text, and a right-aligned action area for page-level buttons (e.g., "Add Widget", "Export", "Refresh"). Include a horizontal divider below the header. The breadcrumb links use the emerald green primary color on hover.

---

## 3. Dashboard Overview Page

### 3.1 Main Dashboard

**Prompt:** Design a monitoring dashboard overview page with a 4-column responsive grid of KPI summary cards at the top. The first card shows "Active Metrics" with a large number (e.g., "2.4M"), a sparkline trend, and a green up-arrow percentage change. The second card shows "Log Ingestion Rate" with "142K/sec" and a mini bar chart. The third card shows "Active Traces" with "8,921" and a mini line chart. The fourth card shows "Firing Alerts" with "7" in red text with a pulsing dot indicator. Below the KPI row, render a 2-column grid: the left column contains a large time-series area chart titled "Metrics Ingestion Rate (24h)" using Ant Design Card with emerald green fill, and the right column has a stacked bar chart titled "Log Volume by Severity" with color-coded bars (red for error, amber for warn, blue for info, gray for debug). Below those, render a full-width "Recent Alerts" Ant Design Table with columns for Severity (color-coded badge), Alert Name, Module, Tenant, Duration, and Status, showing 5 rows of sample data with alternating row backgrounds.

### 3.2 System Health Summary

**Prompt:** Design a system health overview panel as a horizontal strip of service status indicators below the KPI cards. Each indicator is a compact card showing a service name (VictoriaMetrics, Quickwit, Grafana, OTel Collector, Alertmanager, Zabbix, OpenNMS), a colored status dot (green for healthy, yellow for degraded, red for down), uptime percentage, and current latency in milliseconds. The strip scrolls horizontally on smaller screens. Use Ant Design Tag components for the status with matching colors, and render each service icon as a small 24px monochrome logo. Include a "View All Services" link at the end of the strip in emerald green.

### 3.3 Tenant Overview Widget

**Prompt:** Design a tenant activity widget for the dashboard showing a donut chart of telemetry volume distribution across the top 5 tenants, with the center showing the total volume. To the right of the donut chart, display a ranked list of tenants with their name, color swatch matching the chart segment, total data volume, and a mini trend arrow. Below the chart, include a horizontal bar showing quota usage per tenant with percentage labels. Use Ant Design Card as the container with a "Manage Tenants" button in the card header using the emerald green primary color.

---

## 4. Metrics Explorer

### 4.1 Metrics Explorer Main View

**Prompt:** Design a metrics exploration interface with a PromQL query editor at the top inside an Ant Design Card. The query editor has a full-width code input with syntax highlighting, autocomplete dropdown visible showing metric names (e.g., `erp_http_requests_total`, `erp_db_query_duration_seconds`), and action buttons for "Execute" (emerald green primary), "Add to Dashboard", and "Share Query". Below the editor, render a time range selector bar with preset buttons (15m, 1h, 6h, 24h, 7d, 30d) and a custom date-time range picker using Ant Design DatePicker.RangePicker. The main chart area shows a large responsive line chart with multiple series, a legend at the bottom with toggleable series, crosshair tooltip showing all values at the hovered timestamp, and Y-axis auto-scaling. Include a "Table" tab that switches the chart to a tabular view of the raw data points.

### 4.2 Metric Browser Sidebar

**Prompt:** Design a collapsible left sidebar panel for the Metrics Explorer that serves as a metric catalog browser. At the top, place a search input with a magnifying glass icon for filtering metrics by name. Below, render an Ant Design Tree component grouping metrics by ERP module (CRM, IAM, Accounting, Inventory, HRM, etc.), where each module node expands to show its metrics with type icons (counter, gauge, histogram, summary). Clicking a metric inserts it into the PromQL editor. Below the tree, show a "Labels" section that lists available label keys with expandable values, each clickable to add as a filter. The sidebar has a fixed width of 280px with a drag handle for resizing.

### 4.3 Multi-Panel Metrics View

**Prompt:** Design a multi-panel view where users can compare multiple metrics side-by-side. The layout has an "Add Panel" button at the top right that creates a new chart panel in a 2-column grid. Each panel is an Ant Design Card with its own PromQL query input, individual time range override toggle, and chart visualization. Panels can be dragged to reorder (show drag handle in the card header), resized between half-width and full-width using a toggle button, and deleted with a red trash icon. Below all panels, include a synchronized time range selector that controls all panels without individual overrides. Use emerald green for the "Add Panel" button and active panel border highlight.

---

## 5. Log Search & Filter Interface

### 5.1 Log Search Main View

**Prompt:** Design a log search interface inspired by Kibana Discover but using Ant Design components. At the top, render a full-width search bar with a large text input supporting Quickwit query syntax, a "Search" button in emerald green, and a dropdown for selecting the log index. Below the search bar, place a horizontal filter bar with clickable Ant Design Tag chips for severity levels: FATAL (dark red), ERROR (red), WARN (amber), INFO (blue), DEBUG (gray), TRACE (light gray) -- each toggleable with a checkmark. Next, show a time histogram bar chart showing log volume over the selected time range with color-coded severity stacking. The main content area is a virtualized log list where each log entry is a single expandable row showing timestamp, severity badge, service name tag, and the first line of the log message, with monospaced font for the message content.

### 5.2 Log Detail Expanded View

**Prompt:** Design the expanded state of a single log entry row. When a user clicks a log row, it expands inline to reveal the full log message in a monospaced code block with syntax highlighting for JSON content. Below the message, render a two-column key-value table showing all log attributes: service.name, tenant_id, trace_id (clickable link to trace view in emerald green), span_id, host, container_id, and any custom attributes. Include action buttons: "Copy JSON" (copies the raw log as JSON), "View in Context" (shows surrounding logs), "Add to Filter" (adds attribute as a filter chip), and "Find Trace" (navigates to the trace waterfall view). Use an Ant Design Descriptions component for the attribute table with a light green (`#D1FAE5`) background accent strip on the left border.

### 5.3 Saved Searches & Filters

**Prompt:** Design a slide-out drawer from the right side (Ant Design Drawer, 400px width) for managing saved log searches. The drawer has a title "Saved Searches" with a close X button, a search input to filter saved items, and a list of saved search cards. Each card shows the search name in bold, the query text in monospaced gray, applied filters as small tags, the creator username, and last run timestamp. Cards have hover actions: "Run" (executes the search), "Edit" (opens edit mode inline), "Duplicate", and "Delete" (with confirmation popover). At the bottom, include a "Create New Saved Search" button that expands an inline form with name input, description textarea, and "Save" / "Cancel" buttons.

---

## 6. Trace Waterfall View

### 6.1 Trace Search

**Prompt:** Design a trace search page with a form at the top containing: a Service dropdown (Ant Design Select with search, listing all ERP modules), an Operation dropdown (populated based on selected service), min/max Duration inputs with unit selector (ms/s), Tags key-value input pairs with an "Add Tag" button, and a Lookback time selector. Below the form, render search results as a list of trace summary cards. Each card shows the root span operation name, total trace duration with a proportional duration bar, number of spans, number of services involved as colored dots, and timestamp. Cards are sorted by time descending. The "Search" button uses emerald green, and clicking a trace card navigates to the waterfall detail view.

### 6.2 Trace Waterfall Detail

**Prompt:** Design a trace waterfall (Gantt chart) visualization for distributed tracing. At the top, show a trace summary header with: Trace ID (monospaced, with copy button), total duration, number of spans, number of services, and start timestamp. Below the summary, render the waterfall chart where each span is a horizontal bar positioned relative to the trace start time. Indent child spans under their parent with tree-like connector lines. Color-code span bars by service (each ERP module gets a distinct color from a categorical palette, e.g., CRM = blue, IAM = purple, Accounting = emerald). Each bar shows the operation name to the left and duration to the right. Spans with errors have a red bar with an exclamation icon. Include a minimap timeline at the top for navigating long traces, and a zoom slider. Clicking a span opens a detail panel on the right.

### 6.3 Span Detail Panel

**Prompt:** Design a right-side detail panel (Ant Design Drawer or fixed panel, 420px width) that appears when a span is clicked in the waterfall view. The panel shows: span operation name as the title, service name with a colored badge, span kind (CLIENT, SERVER, INTERNAL) as a tag, start time and duration prominently displayed, and a status indicator (OK in green, ERROR in red). Below, render an Ant Design Tabs component with three tabs: "Attributes" (a key-value table of all span attributes including http.method, http.url, db.statement, etc.), "Events" (a timeline of span events like exceptions with expandable stack traces in monospaced red text), and "Links" (related spans/traces as clickable links). Include a "View Logs" button that navigates to the log search pre-filtered by this span's trace_id and span_id.

---

## 7. Alert Management

### 7.1 Alert List View

**Prompt:** Design an alert management dashboard with an Ant Design Tabs component at the top switching between "Active Alerts", "Silenced", and "Alert Rules". The Active Alerts tab shows a full-width table with columns: Severity (Ant Design Badge with colors -- critical=red, warning=amber, info=blue), Alert Name (bold, clickable), Source (module name tag), Tenant, Started At (relative time like "2h ago"), Duration, and Actions (Acknowledge, Silence, View). Above the table, include filter controls: a severity multi-select, a source module dropdown, a tenant dropdown, and a search input. The table supports sorting by any column and has pagination. At the top-right, display a summary strip: "12 Critical, 23 Warning, 45 Info" with corresponding colored circles. Use red for critical rows with a subtle red left-border accent.

### 7.2 Alert Rule Creation Form

**Prompt:** Design a multi-step alert rule creation form using Ant Design Steps component at the top (steps: Define Rule, Set Conditions, Configure Notifications, Review & Save). Step 1 "Define Rule" shows: Rule Name input, Description textarea, Severity select (Critical/Warning/Info), and Module select. Step 2 "Set Conditions" shows: a PromQL/LogQL expression editor with syntax validation (green checkmark or red X with error message), evaluation interval input (e.g., "1m"), a "for" duration input (e.g., "5m"), and a preview chart showing what the alert would look like on recent data with a threshold line. Step 3 "Configure Notifications" shows: notification channel checkboxes (Email, Slack, PagerDuty, Webhook), recipient inputs per channel, and tenant routing rules. Step 4 shows a read-only summary of all settings. The "Create Rule" button is emerald green on the final step.

### 7.3 Alert Detail & Timeline

**Prompt:** Design an alert detail page that opens when clicking an alert name. At the top, show a large alert header with the alert name, severity badge, current state (FIRING in red or RESOLVED in green), and action buttons (Acknowledge, Silence, Create Incident). Below the header, show an Ant Design Descriptions panel with: Alert Rule, Expression, Evaluation Interval, For Duration, Source Module, Affected Tenant, Labels, and Annotations. The main body has two sections: a time-series chart showing the metric that triggered the alert with a horizontal threshold line and shaded regions where the alert was firing (red shading), and below it a "History" timeline (Ant Design Timeline component) showing state transitions with timestamps, user actions (who acknowledged, who silenced), and resolution events. Use a red top border for the page when the alert is firing.

---

## 8. Custom Dashboard Builder

### 8.1 Dashboard Grid Editor

**Prompt:** Design a custom dashboard builder with a drag-and-drop grid layout editor. The top bar has: dashboard title (editable inline), a "Save" button (emerald green), "Add Widget" dropdown button (options: Line Chart, Bar Chart, Gauge, Stat, Table, Log Panel, Alert List, Text/Markdown), time range picker, and auto-refresh interval selector. The grid area uses a 24-column layout where widgets can be dragged, resized (showing resize handles on hover), and rearranged. Each widget shows a drag handle in its header, an edit (pencil) icon, a clone icon, and a delete (trash) icon. When in edit mode, empty grid cells show a dashed border with a "+" icon. Include a right-side panel (collapsible, 320px) for editing the selected widget's data source, queries, and visualization options. Show a subtle grid overlay during drag operations.

### 8.2 Widget Configuration Panel

**Prompt:** Design a widget configuration panel as a right-side sliding panel (320px width) that appears when editing a widget. The panel has an Ant Design Tabs component with tabs: "Query" (PromQL or Quickwit query editor with data source selector), "Visualization" (chart type selector with thumbnail previews, color palette picker, axis configuration, legend position toggle), "Overrides" (field-specific overrides for colors, thresholds, units, decimal places), and "Thresholds" (add green/amber/red threshold values with color pickers and operators). Below the tabs, include a "Preview" section that live-updates as the user modifies settings. The Apply and Cancel buttons are fixed at the panel bottom. Use emerald green for the active tab indicator and apply button.

### 8.3 Dashboard Variable Controls

**Prompt:** Design a dashboard-level variable control bar that sits between the page header and the widget grid. The bar contains a row of Ant Design Select dropdowns for template variables (e.g., "Tenant: All / TenantA / TenantB", "Module: All / CRM / IAM / Accounting", "Environment: prod / staging / dev"). Each variable dropdown supports multi-select with "All" option and search filtering. Variables are defined by the dashboard creator and can source their values from a PromQL label_values() query or a static list. Include an "Add Variable" button (visible only in edit mode) that opens a popover form for defining new variables with name, label, query/static values, and multi/single select toggle. The bar collapses to a single "Filters" dropdown on mobile viewports.

---

## 9. Tenant Management Interface

### 9.1 Tenant List View

**Prompt:** Design a tenant management page with an Ant Design Table listing all tenants. Columns include: Tenant ID (monospaced), Display Name, Status (Active/Suspended as colored tags), Data Tier (Free/Standard/Enterprise as outlined tags), Metrics Quota (progress bar showing usage vs. limit, emerald green below 80%, amber 80-95%, red above 95%), Log Quota (similar progress bar), Created Date, and Actions (Edit, Suspend, Delete with confirmation). Above the table, include a search bar, status filter dropdown, and tier filter dropdown. At the top-right, place a "Create Tenant" button in emerald green. Below the table, show an Ant Design Pagination component. Include a bulk action bar that appears when checkboxes are selected, with options for "Bulk Suspend" and "Bulk Update Tier".

### 9.2 Tenant Detail / Edit Form

**Prompt:** Design a tenant detail page with a two-column layout. The left column (60% width) contains an Ant Design Form with fields: Display Name, Description (textarea), Contact Email, Data Tier (radio group: Free, Standard, Enterprise with feature descriptions), Metrics Retention (select: 7d, 30d, 90d, 365d), Log Retention (similar select), and Custom Labels (dynamic key-value tag input). The right column (40% width) shows a live usage dashboard for this tenant: current metric series count gauge, log ingestion rate sparkline, storage used vs. quota donut chart, and a mini alert summary. Below the form, show "Save Changes" (emerald green) and "Cancel" buttons. Include an Ant Design Tabs section below the main form with tabs for "API Keys" (table of keys with create/revoke actions), "Audit Log" (timeline of tenant changes), and "Data Sources" (list of connected ERP modules for this tenant).

### 9.3 Tenant Onboarding Wizard

**Prompt:** Design a tenant onboarding wizard using Ant Design Steps with 4 steps: "Basic Info" (name, contact, description), "Data Tier Selection" (three-column pricing-style cards for Free/Standard/Enterprise with feature checkmarks and quota limits), "Module Connections" (checklist of ERP modules to enable observability for, with toggles and brief descriptions), and "Confirmation" (summary of all selections with estimated monthly data volume). Each step has "Previous" and "Next" buttons at the bottom, with the final step showing a "Create Tenant" emerald green button. Include a progress indicator showing completion percentage. The tier selection cards should highlight the recommended tier with an emerald green border and a "Recommended" badge.

---

## 10. Zabbix Infrastructure Overview

### 10.1 Infrastructure Host Map

**Prompt:** Design an infrastructure monitoring overview page powered by Zabbix data. At the top, show KPI cards: Total Hosts (number with icon), Hosts with Problems (red number), Availability Percentage (large gauge), and Unacknowledged Issues (amber number). The main content area shows a hexagonal host map (treemap alternative) where each hexagon represents a server/host, colored by health status (green=OK, amber=warning, red=critical, gray=maintenance). Hexagons are grouped by host group (labeled clusters). Hovering over a hexagon shows a tooltip with hostname, IP, CPU usage, memory usage, and current problems. Clicking opens a host detail flyout. Include a filter bar with host group dropdown, severity filter, and a toggle between "Map View" and "Table View". Use Ant Design components for all controls.

### 10.2 Infrastructure Table View

**Prompt:** Design the table view alternative for infrastructure monitoring. Render an Ant Design Table with columns: Host (name + IP in smaller gray text), Host Group (tag), Status (green/amber/red dot), CPU % (inline progress bar), Memory % (inline progress bar), Disk % (inline progress bar), Network I/O (formatted throughput), Active Problems (count with severity color), and Last Check (relative timestamp). The table supports column sorting, row selection with checkboxes, and expandable rows that show detailed problem list per host. Above the table, include quick filter buttons for "All", "Problems Only", "Maintenance", and a search input. Row backgrounds subtly tint red for hosts with critical problems. Pagination at the bottom with page size selector.

### 10.3 Network Topology View

**Prompt:** Design a network topology visualization page showing infrastructure relationships. Render an interactive network graph where nodes represent hosts/services and edges represent connections (SNMP links, dependencies). Nodes are circular with icons representing device type (server, switch, router, firewall, database) and color-coded by health. Edge lines are green for healthy connections and red for degraded. Include zoom/pan controls, a minimap navigator in the bottom-right corner, and a legend panel. A left sidebar filter panel allows filtering by host group, device type, and status. Clicking a node selects it and highlights its connections while dimming others, showing a detail popup with key metrics. Use the emerald green accent for selected node outlines and the topology controls.

---

## 11. OpenNMS Event Correlation

### 11.1 Event Correlation Dashboard

**Prompt:** Design an OpenNMS event correlation dashboard showing how events from different ERP modules and infrastructure components are related. At the top, show a timeline visualization (horizontal swim lanes by source) where events are plotted as dots or bars on a time axis, with correlated events connected by curved lines. Below the timeline, render a correlation summary table with columns: Correlation ID, Root Cause Event, Related Events Count, Affected Modules (as colored tags), Impact Level (Low/Medium/High/Critical severity badges), Duration, and Status (Open/Investigating/Resolved). Include filter controls for time range, impact level, and source module. The emerald green primary color highlights the currently selected correlation group, and a right-side detail panel shows the full event chain for the selected correlation.

### 11.2 Event Feed

**Prompt:** Design a real-time event feed page showing OpenNMS events in a vertical scrolling list. Each event card shows: severity icon (colored circle), event UEI (Unique Event Identifier) as a label, source node name and IP, event description text, timestamp with relative time, and event parameters as expandable key-value chips. New events animate in from the top with a subtle slide-down and green highlight that fades. Include a pause/resume button for the live feed, severity filter checkboxes (Critical, Major, Minor, Warning, Normal, Cleared), and a node name search input. At the top, show a live event rate counter: "47 events/min" with a mini sparkline of the last 5 minutes. Use Ant Design List component for the event feed with infinite scroll.

---

## 12. Responsive & Mobile Considerations

### 12.1 Mobile Dashboard View

**Prompt:** Design the mobile-responsive version (375px width) of the dashboard overview page. The KPI cards stack in a single column, each taking full width. Charts render at full width with reduced height (200px). The sidebar collapses to a hamburger menu drawer. The time range picker becomes a compact dropdown instead of button group. Alert tables switch to a card-based list view where each alert is a stacked card showing severity, name, and time. The tenant overview donut chart resizes and moves the legend below the chart. Maintain the emerald green primary color and dark mode support. All tap targets are minimum 44px for touch accessibility.

### 12.2 Tablet Dashboard View

**Prompt:** Design the tablet-responsive version (768px width) of the metrics explorer page. The metric browser sidebar collapses by default and overlays when opened. The PromQL editor takes full width. Charts display at full width. The time range selector wraps to two rows if needed. The multi-panel view switches from 2-column to single-column layout. Table views reduce visible columns and add a horizontal scroll indicator. The filter bar wraps gracefully. Maintain all functionality but optimize for touch interaction with larger button hit areas and swipe gestures for panel navigation.

---

## 13. Accessibility & Theming Specifications

### 13.1 Accessibility Requirements

| Requirement | Implementation |
|-------------|----------------|
| **Color Contrast** | WCAG 2.1 AA minimum (4.5:1 for normal text, 3:1 for large text) |
| **Keyboard Navigation** | Full tab-order support, visible focus rings (2px emerald green outline) |
| **Screen Reader** | ARIA labels on all interactive elements, chart data available as tables |
| **Color-Blind Safe** | All severity indicators use shape + color (not color alone): circle=OK, triangle=warning, X=error |
| **Reduced Motion** | Respect `prefers-reduced-motion`, disable chart animations |
| **Font Scaling** | Support up to 200% text zoom without layout breaking |

### 13.2 Theme Token Override Prompt

**Prompt:** Design a theme configuration panel (accessible from user settings) that allows switching between Light Mode and Dark Mode using an Ant Design Switch, adjusting the primary color with a color picker (default emerald `#059669`), selecting font size (Small 12px, Medium 14px, Large 16px) with radio buttons, toggling high-contrast mode (increases all borders, adds underlines to links), and enabling compact mode (reduces padding and margins by 25%). Show a live preview section that updates as settings change. Include a "Reset to Defaults" link and a "Save Preferences" emerald green button. Store preferences in YugabyteDB per user.

---

## 14. Component Library Reference

### Frequently Used Ant Design Components

| Component | Usage in Observability | Custom Props |
|-----------|----------------------|--------------|
| `Table` | Log list, alert list, host table, tenant list | Virtual scroll, expandable rows, row selection |
| `Card` | KPI cards, chart containers, widget panels | Custom header with actions, loading skeleton |
| `Select` | Service picker, tenant picker, time presets | Search enabled, tag mode for multi-select |
| `DatePicker.RangePicker` | Time range selection, custom date queries | Preset ranges, timezone awareness |
| `Tabs` | Alert tabs, span detail tabs, dashboard settings | Animated ink bar in emerald green |
| `Form` | Alert rule creator, tenant form, widget config | Validation rules, async submit |
| `Badge` | Severity indicators, status dots, notification counts | Custom colors per severity level |
| `Tag` | Module labels, filter chips, severity labels | Closable for filters, color-coded |
| `Steps` | Alert rule wizard, tenant onboarding | Emerald green active step indicator |
| `Drawer` | Saved searches, span detail, widget editor | Right-side, custom widths |
| `Tree` | Metric browser, host group hierarchy | Lazy-loaded, searchable |
| `Descriptions` | Log attributes, alert detail, span attributes | Bordered style, responsive columns |
| `Timeline` | Alert history, audit log, event feed | Alternating sides, colored dots |
| `Progress` | Quota usage, CPU/memory bars, loading states | Custom colors for thresholds |
| `Statistic` | KPI numbers, counters, percentages | Prefix icons, value style override |

---

## 15. Icon & Illustration Prompts

### 15.1 Empty State Illustrations

**Prompt (No Metrics):** Design a friendly empty state illustration for when no metrics data is available. Show a minimalist line drawing of a flat-line chart with a magnifying glass and a subtle emerald green accent, with the text "No metrics found" in 18px semibold and "Try adjusting your time range or query filters" in 14px gray below. Center the illustration in the content area with 48px padding.

**Prompt (No Logs):** Design an empty state illustration for zero log search results. Show a minimalist illustration of a document page with a search icon and a subtle emerald green accent line, with "No matching logs" as the heading and "Modify your query or check the selected time range" as the description. Use the same centered layout and emerald accents.

**Prompt (No Alerts):** Design a positive empty state for the alert list when all is quiet. Show a minimalist shield icon with a checkmark in emerald green, with "All Clear" as the heading and "No active alerts. Your systems are running smoothly." as the description, conveying a reassuring tone.

### 15.2 Loading State Skeletons

**Prompt:** Design skeleton loading states for all major components. KPI cards show pulsing gray rectangles for the number and label. Chart areas show a pulsing rectangular placeholder with a faint sine wave outline. Table rows show alternating gray bars for each column. The skeleton pulse animation uses a soft gradient sweep from `#E5E7EB` to `#F3F4F6` at 1.5s duration. All skeletons match the exact dimensions of their loaded counterparts to prevent layout shift.

---

## 16. Export & Sharing Prompts

### 16.1 Dashboard Share Modal

**Prompt:** Design a sharing modal (Ant Design Modal, 520px width) for sharing dashboards or panels. The modal has tabs for "Link" (a read-only URL input with a copy button and options for "Current time range" or "Relative time range" checkboxes), "Embed" (an iframe code snippet in a code block with copy button and dimension inputs), "Snapshot" (creates a point-in-time snapshot with expiry options: 1h, 24h, 7d, never), and "PDF Export" (paper size selector, orientation toggle, and "Generate PDF" button with a progress indicator). The modal title is "Share Dashboard" and the close button is in the top-right corner. Primary action buttons use emerald green.

### 16.2 Report Builder

**Prompt:** Design a scheduled report builder form with fields for: Report Name, Description, Dashboard Selection (multi-select from saved dashboards), Time Range (select from Last 24h, Last 7d, Last 30d, Custom), Schedule (cron-style with a human-readable selector: Daily/Weekly/Monthly at specific time with timezone), Recipients (email tag input), Format (PDF, PNG, CSV radio buttons), and Tenant Scope (dropdown for which tenant data to include). Show a preview thumbnail of the selected dashboard and a "Send Test Report" secondary button alongside the "Save Schedule" emerald green primary button. Use Ant Design Form layout with labels on the left.
