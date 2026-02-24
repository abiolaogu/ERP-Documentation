# Figma & Make Prompts — ERP-BI
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This prompt pack provides production-ready Figma Make design prompts and Make automation scenario prompts for **ERP-BI**, the Business Intelligence module of the ERP platform. ERP-BI comprises seven microservices — alert-service, dashboard-service, data-modeling-service, data-warehouse-service, nlq-service, query-engine, and report-service — connected through a Gateway/API layer authenticated via ERP-IAM (OIDC/JWT) with tenant-scoped entitlements managed by ERP-Platform. The module streams events via Redpanda/Kafka and persists data in PostgreSQL.

Designers use the Figma prompts to generate screens, components, and flows for the BI experience. Automation engineers use the Make prompts to build operational workflows supporting analytics operations and data quality monitoring.

---

## 2. AIDD Guardrails (Apply To All Prompts)

These are non-negotiable constraints that must be enforced in every generated output.

### 2.1 User Experience And Accessibility
- Meet WCAG 2.1 AA for color contrast, keyboard access, focus visibility, and semantic structure.
- Keep tap targets at least 44x44px on touch devices.
- Provide clear empty, loading, error, and success states for every asynchronous surface.
- Chart and visualization colors must be distinguishable by color-blind users (use patterns and shapes as secondary indicators).
- Data tables must support keyboard navigation (arrow keys for cell movement, Enter for selection).
- Natural language query results must clearly indicate data freshness, query confidence, and source tables.
- Large numbers must use locale-aware formatting (thousands separators, decimal notation, currency symbols).

### 2.2 Performance And Frontend Efficiency
- Design for route-level lazy loading and code splitting by default.
- Keep initial route JS budget under 220KB gzip for user-facing routes.
- Chart and visualization components are heavy — defer loading until visible in viewport.
- Use progressive disclosure for complex dashboards: Load above-the-fold widgets first, defer below-fold.
- Keep animation subtle (150-200ms) and non-blocking; chart animations should be fast (300ms max).
- Enforce skeleton UIs for dashboard widgets, report previews, and query results.
- Data tables must virtualize rows for datasets exceeding 100 rows.

### 2.3 Reliability, Trust, And Safety
- Build explicit recovery paths for query failures, warehouse connection errors, and report generation timeouts.
- All data visualizations must display data freshness timestamps and source information.
- Natural language query results must show the generated SQL and confidence level for transparency.
- Alert thresholds must have clear confirmation dialogs to prevent accidental over-alerting.
- Report exports must include metadata: generation timestamp, filters applied, data range, author.

### 2.4 Observability And Testability
- Every critical flow must define analytics events, error events, and performance checkpoints.
- Produce handoff notes that map UI states to test cases (unit, integration, E2E).
- Include performance instrumentation surfaces (LCP, INP, CLS, API latency markers).
- Query engine operations must expose execution plans and timing breakdowns.
- Dashboard load times must be instrumented per widget for performance optimization.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a comprehensive design token system for an enterprise Business Intelligence platform (ERP-BI).

Token categories:

1) Color Roles:
   - Primary: Deep navy (#1E3A5F) for navigation, headers, and primary surfaces
   - Accent: Bright blue (#2563EB) for primary actions, links, active states
   - Surface: White (#FFFFFF) cards on cool gray (#F8FAFC) background
   - Success: Emerald (#059669) for positive metrics, upward trends, targets met
   - Warning: Amber (#D97706) for approaching thresholds, data quality warnings
   - Error: Red (#DC2626) for failed queries, broken dashboards, critical alerts
   - Info: Sky blue (#0EA5E9) for informational indicators, neutral metrics
   - Neutral: Slate (#64748B) for secondary text, borders, dividers

   Chart palette (10 colors, color-blind safe):
   - Series 1: #2563EB (Blue)
   - Series 2: #16A34A (Green)
   - Series 3: #EA580C (Orange)
   - Series 4: #9333EA (Purple)
   - Series 5: #DC2626 (Red)
   - Series 6: #0891B2 (Cyan)
   - Series 7: #CA8A04 (Gold)
   - Series 8: #DB2777 (Pink)
   - Series 9: #4F46E5 (Indigo)
   - Series 10: #059669 (Teal)

   Trend colors:
   - Positive trend: #059669 (green) with up arrow
   - Negative trend: #DC2626 (red) with down arrow
   - Neutral trend: #64748B (gray) with horizontal arrow

   NLQ (Natural Language Query) Accent: Violet (#7C3AED) for NLQ interface elements and AI indicators

   Dark mode variants for every color role (including chart palette adjustments for dark backgrounds)

2) Typography Scale:
   - Font family (UI): Inter, system-ui, sans-serif
   - Font family (Data): Tabular numbers — Inter with font-variant-numeric: tabular-nums
   - Font family (Code): JetBrains Mono (for SQL queries, code blocks)
   - Display: 36px/2.25rem, weight 700 (page titles)
   - H1: 28px/1.75rem, weight 600 (section headers, dashboard titles)
   - H2: 22px/1.375rem, weight 600 (widget titles, report names)
   - H3: 18px/1.125rem, weight 500 (subsection headers)
   - Body: 15px/0.9375rem, weight 400, line-height 1.5 (default text)
   - Body Small: 13px/0.8125rem, weight 400 (table cells, metadata)
   - Caption: 11px/0.6875rem, weight 500 (axis labels, chart legends, timestamps)
   - Metric Large: 32px/2rem, weight 700, tabular-nums (KPI values)
   - Metric Medium: 24px/1.5rem, weight 600, tabular-nums (secondary metrics)
   - SQL: JetBrains Mono, 13px/0.8125rem (query editor, generated SQL)

3) Spacing Scale:
   - 4px (xs) | 8px (sm) | 12px (md) | 16px (lg) | 24px (xl) | 32px (2xl) | 48px (3xl) | 64px (4xl)

4) Border Radius:
   - None: 0px (data tables, chart areas)
   - Subtle: 4px (inputs, inline badges)
   - Default: 8px (cards, widgets, modals)
   - Rounded: 12px (floating panels, NLQ interface)
   - Pill: 9999px (status badges, filter chips)

5) Elevation:
   - Level 1: 0 1px 3px rgba(0,0,0,0.06) (dashboard widgets at rest)
   - Level 2: 0 4px 8px rgba(0,0,0,0.08) (hover widgets, dropdowns)
   - Level 3: 0 8px 16px rgba(0,0,0,0.10) (modals, NLQ overlay)
   - Level 4: 0 16px 32px rgba(0,0,0,0.14) (full-screen report viewer)

6) Motion:
   - Fast: 100ms ease-out (button press, toggle, filter chip add/remove)
   - Default: 150ms ease-in-out (card hover, tooltip appear)
   - Smooth: 250ms ease-in-out (modal open, panel slide, chart data transition)
   - Chart: 300ms ease-out (chart animation on data load, bar growth, line draw)
   - Progress: 2s linear infinite (query execution progress bar)

Deliver both light and dark theme token sets. Include contrast verification annotations for all foreground/background pairs. Include chart color accessibility verification (color-blind simulation for all chart palette combinations).
```

#### Prompt F-002: Component Library

```
Create a Business Intelligence component library for ERP-BI with these component families:

Navigation Components:
- Global top bar: Logo "ERP-BI", breadcrumb trail, global date range picker, tenant context indicator, notification bell with alert count, user avatar menu
- Left sidebar: Collapsible, icon + label items for Dashboards, Reports, Data Explorer, NLQ (Natural Language Query), Data Models, Data Warehouse, Alerts, Settings; active state with accent bar; collapse to icon-only mode; favorites section at top
- Command palette: Centered overlay (Cmd+K), search across dashboards, reports, metrics, saved queries

Chart/Visualization Components:
- Line chart: Multi-series with legends, tooltips, axis labels, zoom controls, data point hover detail, time range brush selector
- Bar chart: Vertical and horizontal, grouped and stacked variants, value labels, click-through to drill-down
- Pie/Donut chart: With legend, percentage labels, center metric (for donut), click segment to filter
- Area chart: Stacked area with transparency, baseline comparison overlay
- Heatmap: Matrix with color intensity scale, hover detail, axis labels
- Scatter plot: With trend line, cluster coloring, size dimension, click-through
- Gauge/Speedometer: Current value, target value, min/max, color zones (green/amber/red)
- Sparkline: Compact inline chart for table cells and KPI tiles
- Map visualization: Choropleth or bubble map with geographic data, legend, zoom
- Funnel chart: Stage labels, values, conversion percentages between stages
- Table chart: Data grid with conditional formatting (heatmap cells, bar-in-cell, icon indicators)

Dashboard Components:
- KPI tile: Metric name, large value (tabular-nums), trend arrow with percentage, comparison period label, sparkline, click to drill-down
- Dashboard widget container: Title bar with widget name, time range, refresh button, maximize button, options menu (edit, clone, remove, export), resize handle (drag corner)
- Dashboard grid: Responsive grid layout with drag-and-drop widget placement, resize handles, snap-to-grid, add widget button
- Filter bar: Global filters applied across all widgets — date range, dimension filters, saved filter sets, clear all
- Dashboard toolbar: Dashboard name (inline editable), "Edit Mode" toggle, "Share" button, "Export" button (PDF, PNG, CSV), "Schedule" button, auto-refresh indicator

Report Components:
- Report card: Report name, description, type badge (Ad-hoc, Scheduled, Template), last generated, format (PDF, Excel, CSV), preview thumbnail, "Open" and "Download" buttons
- Report builder: Drag-and-drop section composer with text blocks, charts, tables, page breaks, header/footer templates
- Report viewer: Paginated view with zoom, page navigation, print, download, share
- Report schedule form: Frequency selector (daily, weekly, monthly, custom cron), recipients list, format selector, delivery method (email, Slack, S3)

NLQ Components:
- NLQ input bar: Large search-style input with placeholder "Ask a question about your data...", voice input button, recent queries dropdown, AI sparkle icon
- NLQ result card: Natural language question displayed, AI-generated answer with formatted data, chart visualization of results, generated SQL (collapsible), confidence badge, "Save as Widget" button, "Refine Query" button
- NLQ suggestion chips: Pre-built questions — "What was revenue last month?", "Top 10 customers by spend", "Monthly growth rate", "Compare Q3 vs Q4"
- SQL editor: Syntax-highlighted SQL editor with auto-complete, table/column suggestions, run button, execution plan viewer, result grid

Data Modeling Components:
- Model diagram: Entity-relationship diagram with tables, columns, relationships (lines with cardinality), drag-to-connect, zoom and pan
- Table schema card: Table name, column list with types, primary key indicator, foreign key indicators, row count, last updated
- Relationship line: Connecting two tables with cardinality labels (1:1, 1:N, N:N), join type indicator

Alert Components:
- Alert rule card: Alert name, metric, condition (e.g., "Revenue < $50K"), threshold visualization, notification channels, status (Active/Paused/Triggered), last triggered
- Alert timeline: Chronological feed of triggered alerts with severity coloring, metric value at trigger, acknowledged/resolved status
- Alert configuration form: Metric selector, condition builder (>, <, =, % change), threshold input, evaluation frequency, notification channels (email, Slack, webhook, SMS), escalation rules

Data Table Components:
- Data table: Sortable columns with type indicators (text, number, date, boolean), row selection, pagination, column resize, column visibility toggle, bulk actions toolbar, export button, inline cell formatting (conditional colors, bar-in-cell, sparklines)
- Pivot table: Row and column dimension selectors, value aggregation (sum, avg, count, min, max), expandable row groups, subtotals, grand total
- Query result grid: Column headers from query result, type-inferred formatting, null value indicator, row count, execution time badge

State variants for every component:
- Default, hover, active, focus-visible, loading (skeleton), disabled, error, success, empty state (no data), refreshing, stale data warning
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Analytics Dashboard

```
Design the main Analytics Dashboard page for ERP-BI at 1440px width.

Layout:
- Global top bar with "ERP-BI" logo, breadcrumb "Dashboards > Revenue Overview", global date range picker (showing "Last 30 Days"), notifications (3 alerts), avatar
- Left sidebar with "Dashboards" active, showing favorites: "Revenue Overview" (starred), "Sales Pipeline", "Customer Metrics", "Operational KPIs"
- Main content area with 16px padding

Content — "Revenue Overview" dashboard:

1) Dashboard header:
   - Title "Revenue Overview" (inline editable in edit mode)
   - Last updated: "2 minutes ago" with auto-refresh indicator (every 5 min)
   - Actions: "Edit Dashboard" toggle, "Share" button, "Export" (PDF/PNG), "Schedule Report", "Full Screen"
   - Filter bar: Date range "Feb 1 - Feb 23, 2026", Region filter "All Regions", Product line filter "All Products", Customer segment "Enterprise"
   - Applied filters shown as removable chips

2) KPI row (4 tiles):
   - Total Revenue: $4,247,830 | +12.3% vs last period | Sparkline trending up | Target: $5,000,000
   - New Customers: 847 | +8.7% | Sparkline with slight dip mid-month
   - Average Order Value: $2,340 | -2.1% (red) | Sparkline relatively flat
   - Customer Retention: 94.2% | +1.1% | Sparkline steady
   Each tile: Click to drill-down to detailed view

3) Chart widgets grid (2x2):

   Top-left: "Revenue Trend" (Line chart, 50% width)
   - Multi-series line chart: Current period (solid blue), Previous period (dashed gray), Target (dotted green)
   - X-axis: Daily dates, Y-axis: Revenue ($)
   - Hover tooltip: Date, Revenue, vs Target delta, vs Previous delta
   - Time range brush at bottom for zoom

   Top-right: "Revenue by Product Line" (Bar chart, 50% width)
   - Horizontal bar chart, descending by revenue
   - Bars: Enterprise Software $1,847K, Cloud Services $1,203K, Professional Services $687K, Support & Maintenance $511K
   - Each bar: Value label, percentage of total
   - Click bar to drill into product detail

   Bottom-left: "Revenue by Region" (Map + Donut, 50% width)
   - World map choropleth with revenue intensity coloring
   - Or donut chart: North America 45%, Europe 28%, APAC 18%, LATAM 6%, MEA 3%
   - Legend with values, click segment to filter dashboard

   Bottom-right: "Top 10 Customers by Revenue" (Table chart, 50% width)
   - Data table with columns: Rank, Customer Name, Revenue, % of Total, Trend (sparkline), YoY Change
   - Top row: 1. Acme Corp $423,000 (9.9%) +15%
   - Conditional formatting: Green for positive YoY, red for negative
   - Click row to navigate to customer detail

4) Secondary widgets row (below grid):

   Left (60%): "Monthly Revenue vs Target" (Grouped bar chart)
   - Monthly bars: Actual (blue) vs Target (green outline)
   - Months: Oct, Nov, Dec, Jan, Feb (partial)
   - Cumulative line overlay showing trajectory

   Right (40%): "Revenue Alerts" (Alert feed widget)
   - Recent alerts:
     - "Revenue dropped 5% day-over-day — Feb 21" (amber, 2 days ago)
     - "Enterprise Software exceeded monthly target — Feb 18" (green, 5 days ago)
     - "APAC revenue below forecast threshold — Feb 15" (red, 8 days ago)
   - "View All Alerts" link

5) Widget options menu (on any widget hover):
   - Edit, Clone, Remove, Export (PNG/CSV), Maximize, Move, Resize

Edit mode (when "Edit Dashboard" toggled):
   - Grid shows snap guides
   - Widgets have visible drag handles and resize corners
   - "Add Widget" floating button opens widget gallery (chart types, KPI tiles, tables, text blocks, filters)
   - Widget configuration panel (right slide-over): Data source selector, dimensions, measures, chart type, filters, formatting options

States: Loading dashboard (widget skeleton placeholders), partial load (some widgets loaded, others still loading), widget error (individual widget error with retry), no data for filters (empty state per widget), stale data warning (amber banner with last successful refresh timestamp), edit mode active.

Include dark mode variant.
```

#### Prompt F-004: Natural Language Query Interface

```
Design the Natural Language Query (NLQ) interface for ERP-BI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-BI > Ask Your Data"
- Left sidebar with "NLQ" active
- Main content area: Two-section layout — Query area (top) + Results area (bottom, expanding)

Content sections:

1) NLQ Hero section (top, centered):
   - Large heading: "Ask Your Data" with AI sparkle icon
   - Subtitle: "Ask questions in plain English and get instant insights from your data warehouse"
   - Large input bar (centered, 720px wide):
     - Placeholder: "e.g., What was total revenue by region last quarter?"
     - AI sparkle icon (left)
     - Voice input button (right)
     - Send button (right, blue)
   - Suggestion chips below input (2 rows):
     Row 1: "What was revenue last month?", "Top 10 customers by spend", "Compare Q3 vs Q4 revenue"
     Row 2: "Show monthly churn rate trend", "Average deal size by product", "Revenue forecast next quarter"

2) Recent queries sidebar (left, 280px, collapsible):
   - "Recent Queries" header with clear all link
   - List items: Query text (truncated), timestamp, result type icon (chart/table/number)
   - "Saved Queries" section below with starred queries
   - Sample:
     - "What was total revenue by region in Q4?" — 2h ago — Chart
     - "Top 5 products by growth rate" — Yesterday — Table
     - "Monthly active users trend" — 2 days ago — Chart (starred)

3) Results area (main, expanding to fill):
   After user submits "What was total revenue by region last quarter?":

   a) Confidence banner:
      - "Confidence: High (94%)" in green badge
      - "Queried 3 tables: orders, customers, regions | 142,847 rows scanned | 0.8s execution"
      - "Based on Q4 2025 data (Oct 1 - Dec 31)"

   b) Natural language answer:
      - "Total revenue in Q4 2025 was **$12,847,320** across all regions. **North America** led with **$5,783,294** (45.0%), followed by **Europe** at **$3,597,250** (28.0%), **APAC** at **$2,312,517** (18.0%), **LATAM** at **$770,839** (6.0%), and **MEA** at **$383,420** (3.0%)."

   c) Auto-generated visualization:
      - Primary: Horizontal bar chart showing revenue by region (descending)
      - Bars with value labels and percentage
      - Chart type switcher: Bar | Pie | Table | Map
      - User can switch chart types and the visualization updates

   d) Data table below chart:
      - Region | Revenue | % of Total | vs Q3 | Growth Rate
      - North America | $5,783,294 | 45.0% | +$412,000 | +7.7%
      - Europe | $3,597,250 | 28.0% | +$198,000 | +5.8%
      - APAC | $2,312,517 | 18.0% | +$347,000 | +17.6%
      - LATAM | $770,839 | 6.0% | -$23,000 | -2.9%
      - MEA | $383,420 | 3.0% | +$15,000 | +4.1%
      - Totals row at bottom

   e) Generated SQL (collapsible section):
      ```sql
      SELECT r.region_name,
             SUM(o.total_amount) AS revenue,
             ROUND(SUM(o.total_amount) / SUM(SUM(o.total_amount)) OVER () * 100, 1) AS pct_total
      FROM orders o
      JOIN customers c ON o.customer_id = c.id
      JOIN regions r ON c.region_id = r.id
      WHERE o.order_date BETWEEN '2025-10-01' AND '2025-12-31'
      GROUP BY r.region_name
      ORDER BY revenue DESC;
      ```
      - "Copy SQL" button, "Edit SQL" button (opens SQL editor), "Explain Query" button

   f) Action buttons:
      - "Save as Dashboard Widget" (add to existing or new dashboard)
      - "Export" (CSV, Excel, PDF)
      - "Schedule as Report" (recurring)
      - "Refine Query" (opens follow-up prompt)
      - "Share" (link or embed)

4) Follow-up suggestions:
   - "Follow up: Show this as a monthly trend", "Break down North America by state", "Compare to Q3 2025", "Forecast Q1 2026"

5) NLQ conversation mode (optional, toggled):
   - Chat-style interface where each question builds on context
   - "What was revenue by region last quarter?" -> "Now show monthly trend" -> "Filter to just Enterprise customers"
   - Each follow-up inherits context from previous queries

States: Empty state (hero with suggestions), query processing (progress bar with "Analyzing your question...", then "Querying data warehouse...", then "Generating visualization..."), results loaded, low confidence result (amber banner: "I'm 62% confident — the query may not fully match your intent. Please review the generated SQL."), query error (red banner with "Could not understand query. Try rephrasing or use the SQL editor."), no results (empty but query was valid — "Your query returned 0 rows for the selected filters.").

Include dark mode variant.
```

#### Prompt F-005: Report Builder

```
Design the Report Builder page for ERP-BI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-BI > Reports > New Report"
- Left sidebar with "Reports" active
- Main area: Three-panel layout — Component palette (240px) | Report canvas (flexible) | Properties panel (320px)

Content sections:

1) Report toolbar (top, full width):
   - Report name: "Q4 2025 Executive Summary" (inline editable)
   - Status: Draft | Published | Scheduled
   - Actions: "Save" (Cmd+S), "Preview" (opens full preview), "Publish", "Schedule", "Export" (PDF/Excel/CSV), "Share"
   - Page navigation: "Page 1 of 3" with add page button
   - Zoom: 50% | 75% | 100% | Fit Width
   - Undo/Redo buttons

2) Component palette (left, 240px):
   - Draggable components grouped by category:
     Text: Heading, Paragraph, Metric Highlight, Data Label
     Charts: Line, Bar, Pie, Donut, Area, Heatmap, Scatter, Funnel, Gauge
     Tables: Data Table, Pivot Table, Summary Table
     Layout: Section Divider, Spacer, Page Break, Columns (2/3/4)
     Data: KPI Card, Metric Row, Comparison Card, Trend Indicator
     Branding: Header Template, Footer Template, Logo Block, Disclaimer Text
   - Drag components onto canvas to add

3) Report canvas (center, flexible, WYSIWYG):
   - A4/Letter page layout with margins
   - Page 1 content:
     - Company logo and report title header: "Quarterly Business Review — Q4 2025"
     - Date range: "October 1 - December 31, 2025"
     - Prepared by: "Business Intelligence Team"
     - Executive summary text block:
       "Q4 2025 saw total revenue of $12,847,320, representing 8.3% year-over-year growth. Key highlights include record APAC growth (+17.6%), Enterprise Software exceeding targets, and improved customer retention at 94.2%."
     - KPI row: Total Revenue $12.8M (+8.3%), New Customers 2,341 (+12%), Retention 94.2% (+1.1%), NPS 72 (+4)
     - Revenue trend chart: Monthly bars Oct/Nov/Dec with YoY comparison
     - Revenue by region table with conditional formatting

   - Page 2 content:
     - "Product Line Analysis" section header
     - Product performance table: Product, Revenue, Units, Growth, Margin
     - Top 10 customers chart
     - Customer acquisition funnel

   - Page 3 content:
     - "Outlook & Recommendations" section header
     - AI-generated insights section: Key trends, risks, recommendations
     - Appendix: Data sources, methodology notes, glossary

   - Selected component shows blue selection border with resize handles
   - Drag guidelines for alignment (snap to grid, snap to other components)

4) Properties panel (right, 320px):
   - When text selected: Font, size, color, alignment, spacing
   - When chart selected:
     - Data source: Dropdown of available datasets/queries
     - Dimensions: Drag fields (Region, Product, Date)
     - Measures: Drag fields (Revenue, Count, Average)
     - Chart type: Visual selector with thumbnails
     - Formatting: Colors, labels, legend position, axis settings
     - Filters: Add component-level filters
   - When table selected:
     - Data source, columns to include, sort, conditional formatting rules
   - When nothing selected: Report-level settings (page size, orientation, margins, theme)

5) Data connection panel (bottom, collapsible):
   - Connected data sources: "Sales Database", "Customer Warehouse", "Product Metrics"
   - Available datasets with column preview
   - "Connect New Source" button
   - Query builder shortcut

States: Empty report (template gallery with sample templates: "Executive Summary", "Sales Report", "Financial Report", "Operational Dashboard Report"), editing with selected component, preview mode (full-screen paginated view), publishing confirmation, schedule configuration modal.

Include dark mode variant.
```

#### Prompt F-006: Data Warehouse Explorer

```
Design the Data Warehouse Explorer page for ERP-BI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-BI > Data Warehouse"
- Left sidebar with "Data Warehouse" active
- Main content area

Content sections:

1) Header:
   - Title "Data Warehouse" with subtitle "Explore, query, and manage your data warehouse"
   - Actions: "New Query" primary button, "Import Data" secondary, "Manage Connections" tertiary
   - Warehouse status: "Connected to: production-warehouse" with health green dot, "Last sync: 2 min ago"

2) Schema browser (left panel, 300px):
   - Warehouse connection selector dropdown
   - Search schemas, tables, columns
   - Schema tree:
     public/
       orders (142,847 rows, 2.3GB)
         order_id (uuid, PK)
         customer_id (uuid, FK -> customers.id)
         order_date (timestamp)
         total_amount (decimal)
         status (varchar)
         region_id (integer, FK -> regions.id)
         product_line (varchar)
         created_at (timestamp)
       customers (23,456 rows, 890MB)
         id (uuid, PK)
         name (varchar)
         email (varchar)
         segment (varchar)
         region_id (integer, FK -> regions.id)
         created_at (timestamp)
         lifetime_value (decimal)
       products (1,247 rows, 45MB)
         id (uuid, PK)
         name (varchar)
         category (varchar)
         price (decimal)
         cost (decimal)
       regions (12 rows, 1KB)
         id (integer, PK)
         region_name (varchar)
         country_count (integer)
     analytics/
       daily_revenue_agg (365 rows)
       monthly_retention (24 rows)
       customer_segments (5 rows)

   - Each table: Click to preview first 100 rows in main area
   - Each column: Click to see distribution stats (min, max, avg, null count, distinct count)
   - Drag table/column to query editor to auto-insert

3) Query editor (center, main area):
   - Tabbed interface: Query 1 (active), Query 2, + New tab
   - SQL editor with syntax highlighting (JetBrains Mono):
     ```sql
     SELECT
       r.region_name,
       DATE_TRUNC('month', o.order_date) AS month,
       COUNT(*) AS order_count,
       SUM(o.total_amount) AS revenue
     FROM orders o
     JOIN regions r ON o.region_id = r.id
     WHERE o.order_date >= '2025-01-01'
     GROUP BY r.region_name, month
     ORDER BY month, revenue DESC;
     ```
   - Auto-complete: Table names, column names, SQL keywords, functions
   - Inline error highlighting (red squiggles for syntax errors)
   - "Run" button (green, Cmd+Enter), "Format SQL" button, "Explain Plan" button, "Save Query" button
   - Query history dropdown

4) Results area (below editor, split pane with drag divider):
   - Tab bar: Results | Execution Plan | Chart
   - Results tab:
     - Data grid: region_name | month | order_count | revenue
     - North America | 2025-01 | 4,231 | $1,847,320
     - Europe | 2025-01 | 2,847 | $1,203,450
     - ... (scrollable)
     - Footer: "12 rows returned in 0.8s | 142,847 rows scanned"
     - Export: CSV, Excel, JSON, Copy to clipboard
     - "Visualize" button to create chart from results
   - Execution Plan tab:
     - Visual query plan: Seq Scan, Index Scan, Hash Join nodes with cost estimates and row counts
   - Chart tab:
     - Auto-suggested chart from query results
     - Chart type selector to change visualization

5) Saved queries panel (right, 280px, collapsible):
   - "My Queries" folder
   - "Team Queries" folder
   - "Templates" folder
   - Each query: Name, description, last run, "Open" and "Run" buttons
   - Share query with team members

States: No warehouse connected (setup wizard), empty query editor (with template suggestions), query running (progress bar with cancel button and row count updating), query complete, query error (with error message and line number highlight), large result set (>10K rows warning with pagination), connection lost (reconnect button).

Include dark mode variant.
```

#### Prompt F-007: Alert Management Console

```
Design the Alert Management console for ERP-BI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-BI > Alerts"
- Left sidebar with "Alerts" active
- Main content area

Content sections:

1) Header:
   - Title "Alerts" with subtitle "Monitor metrics and get notified when thresholds are crossed"
   - Actions: "Create Alert" primary button
   - Filter: Status (All, Active, Triggered, Paused), Severity (All, Critical, Warning, Info), Metric category

2) Alert status dashboard (top):
   - 4 metric tiles:
     - Total Active Alerts: 23
     - Currently Triggered: 3 (red background)
     - Alerts Fired Today: 7
     - Avg Resolution Time: 34 min
   - Triggered alerts timeline: Mini timeline showing when alerts fired in last 24h (dots on timeline, color by severity)

3) Triggered alerts section (prominent, bordered):
   - Section header: "Currently Triggered" with count badge (3)
   - Alert cards (full width, red left border for critical, amber for warning):

   a) "Daily Revenue Below Threshold" — Critical
      - Metric: Daily Revenue
      - Condition: Revenue < $150,000
      - Current Value: $127,340 (red, below threshold)
      - Triggered: 2 hours ago (Feb 23, 08:15 UTC)
      - Notification sent to: #revenue-alerts (Slack), finance-team@company.com
      - Actions: "Acknowledge", "Investigate" (opens NLQ with context), "Snooze 1h", "Resolve"
      - Metric chart: Sparkline showing revenue over last 7 days with threshold line

   b) "Customer Churn Rate Spike" — Warning
      - Metric: Weekly Churn Rate
      - Condition: Churn rate > 5% (week-over-week)
      - Current Value: 6.2% (+1.8% vs last week)
      - Triggered: 45 min ago
      - Actions: Acknowledge, Investigate, Snooze, Resolve

   c) "Data Warehouse Query Latency" — Critical
      - Metric: Query P95 Latency
      - Condition: P95 > 5000ms
      - Current Value: 7,240ms
      - Triggered: 15 min ago
      - Actions: Acknowledge, Investigate, Snooze, Resolve

4) All alert rules table (below triggered section):
   Columns: Alert Name | Metric | Condition | Status | Severity | Last Triggered | Notification Channels | Actions
   Rows:
   - "Daily Revenue Below Threshold" | Daily Revenue | < $150K | Triggered (red) | Critical | 2h ago | Slack, Email | Edit, Pause, Delete
   - "Customer Churn Rate Spike" | Weekly Churn | > 5% WoW | Triggered (amber) | Warning | 45m ago | Slack | Edit, Pause, Delete
   - "Monthly Target Achievement" | Monthly Revenue | < 80% of target | Active (green) | Warning | Feb 18 | Email | Edit, Pause, Delete
   - "New Customer Acquisition Drop" | Weekly New Customers | < 50 per week | Active (green) | Info | Never | Slack | Edit, Pause, Delete
   - "APAC Revenue Growth" | APAC Revenue | > 20% MoM | Active (green) | Info | Feb 15 (positive) | Slack | Edit, Pause, Delete
   - ... more rows with pagination

5) Create/Edit alert modal (when "Create Alert" clicked):
   - Step 1: Select metric
     - Search metrics or browse categories (Revenue, Customers, Operations, Data Quality)
     - Selected: "Daily Revenue"
     - Preview chart of metric history
   - Step 2: Define condition
     - Operator: >, <, =, >=, <=, % change >, % change <
     - Threshold value: $150,000
     - Evaluation window: Last 1 day
     - Evaluation frequency: Every 15 minutes
     - Minimum consecutive violations before firing: 2
   - Step 3: Configure notifications
     - Channels: Email (add recipients), Slack (select channel), Webhook (URL), SMS (phone numbers)
     - Message template (customizable): "Alert: {alert_name} triggered. {metric}: {current_value} (threshold: {threshold})"
     - Escalation: If not acknowledged in {time}, escalate to {recipients}
   - Step 4: Review and activate
     - Summary of alert configuration
     - "Activate Alert" primary button, "Save as Draft" secondary

6) Alert history (bottom section, collapsible):
   - Table: Alert Name, Fired At, Resolved At, Duration, Resolution (Auto-resolved, Acknowledged, Snoozed), Value at Trigger
   - Filterable by date range, alert, severity

States: No alerts configured (setup first alert CTA with template suggestions), all alerts healthy (green status page), multiple triggered alerts (critical alert banner at top of page), alert creation wizard, alert acknowledgment confirmation, alert auto-resolved notification.

Include dark mode variant.
```

#### Prompt F-008: Data Modeling Studio

```
Design the Data Modeling studio page for ERP-BI at 1440px width.

Layout:
- Global top bar with breadcrumb "ERP-BI > Data Models > Revenue Model"
- Left sidebar with "Data Models" active
- Main area: Canvas (70%) + Properties panel (30%)

Content sections:

1) Model toolbar (top):
   - Model name: "Revenue Analytics Model" (inline editable)
   - Status: Draft | Published | Version badge "v2.3"
   - Actions: Save, Publish, Validate, Export (YAML/JSON), Version History
   - View controls: Zoom (50-200%), Fit All, Grid toggle, Minimap toggle

2) Model canvas (center, 70%):
   - Entity-relationship diagram on light grid background
   - Tables as rectangular cards with columns:

   "orders" table card:
   - Header: "orders" with table icon, "142K rows" badge
   - Columns listed:
     - order_id (uuid, PK) [key icon]
     - customer_id (uuid, FK) [link icon]
     - order_date (timestamp)
     - total_amount (decimal) [measure icon]
     - status (varchar) [dimension icon]
     - region_id (integer, FK) [link icon]
     - product_line (varchar) [dimension icon]

   "customers" table card:
   - customer_id (uuid, PK), name, email, segment (dimension), region_id (FK), lifetime_value (measure)

   "products" table card:
   - product_id (uuid, PK), name, category (dimension), price (measure), cost (measure)

   "regions" table card:
   - region_id (integer, PK), region_name (dimension), country_count

   "daily_revenue_agg" table card (derived/materialized):
   - date, region_name, product_line, revenue (measure), order_count (measure)
   - Badge: "Materialized View — Refreshes daily at 02:00 UTC"

   Relationship lines:
   - orders.customer_id -> customers.id (N:1, solid line)
   - orders.region_id -> regions.id (N:1, solid line)
   - orders.product_line -> products.category (N:1, dashed line — soft reference)
   - Cardinality labels at each end
   - Click line to see join details

   - Drag tables to rearrange
   - Draw new relationships by dragging from column to column
   - "Add Table" button on canvas (opens schema browser)

3) Properties panel (right, 30%):
   When table selected:
   - Table name, schema, row count, size on disk
   - Column list with editable metadata:
     - Column name, type, description, semantic role (Dimension/Measure/Key/Time), format pattern
     - Mark as "Hidden from NLQ" toggle
   - Relationships: List of joins involving this table
   - Calculated columns: Add derived columns with formulas
   - "Sync from Warehouse" button (refresh schema from data-warehouse-service)

   When relationship selected:
   - Source table/column -> Target table/column
   - Join type: Inner | Left | Right | Full
   - Cardinality: 1:1 | 1:N | N:1 | N:N
   - Relationship name (for NLQ: "customer's orders", "order's region")

   When nothing selected:
   - Model summary: Table count, relationship count, measure count, dimension count
   - Model validation results: "3 warnings, 0 errors"
     - Warning: "Table 'products' has no relationship to 'orders' via direct join — consider adding product_id to orders"
     - Warning: "Column 'email' in customers is PII — consider marking as hidden for NLQ"
     - Warning: "daily_revenue_agg has no refresh schedule configured"
   - NLQ readiness score: "87% ready" — what to fix to improve NLQ query accuracy

4) Lineage view (toggle in toolbar):
   - Shows data flow from source tables -> transformations -> materialized views -> dashboards/reports
   - Column-level lineage on hover
   - Impact analysis: "Changing orders.total_amount affects 3 dashboards and 7 reports"

States: Empty model (import from warehouse CTA), building model with tables being added, validation errors (highlighted relationships/columns), published model with locked editing (unlock to edit), version comparison (side-by-side diff of model versions).

Include dark mode variant.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Dashboard Viewer

```
Design the Dashboard viewer for ERP-BI at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu (left), "Revenue Overview" title (center, truncated), share icon (right)
- Bottom navigation bar: Dashboards (active), Reports, NLQ, Alerts, More

Content:

1) Filter strip (below top bar):
   - Date range chip: "Last 30 Days" (tap to change via bottom sheet date picker)
   - "+ Filters" chip (tap to open filter bottom sheet)
   - Applied filter count badge

2) KPI tiles (horizontal scrollable, 2 visible at a time):
   - Revenue: $4,247,830 | +12.3%
   - New Customers: 847 | +8.7%
   - AOV: $2,340 | -2.1%
   - Retention: 94.2% | +1.1%
   - Each tile: Tap to see drill-down detail in bottom sheet

3) Charts (stacked, full width, one chart per section):
   - Each chart in a card with title header:
     a) "Revenue Trend" — Line chart (full width, 200px height)
        - Simplified: Current period line only (no comparison)
        - Tap to expand to full-screen chart view with all series and zoom
     b) "Revenue by Region" — Horizontal bar chart (full width)
        - Top 5 regions only, "See All" link
     c) "Top Customers" — Compact table
        - 5 rows: Rank, Name (truncated), Revenue, Trend arrow
        - "See Full Table" link

4) Chart full-screen view (when chart tapped):
   - Full screen with close button
   - Pinch-to-zoom on chart
   - Landscape orientation support for charts
   - Full legend, all data series
   - Export button (share sheet: Copy Image, Save, Email)

5) Dashboard selector (hamburger menu or bottom sheet):
   - List of available dashboards with star/favorite toggle
   - Search dashboards

Pull-to-refresh for live data update.

States: Loading (skeleton cards for KPIs and charts), data loaded, widget error (individual card error with retry), offline mode (last cached data with timestamp), no dashboards available (create or request access CTA).

Touch targets minimum 44x44px. Numbers use tabular-nums for alignment.
```

#### Prompt F-010: Mobile NLQ Interface

```
Design the Natural Language Query interface for ERP-BI at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu, "Ask Your Data" title, history icon
- No bottom navigation (immersive NLQ experience)

Content:

1) Query input (top, prominent):
   - Large input bar (full width, 48px height): "Ask a question about your data..."
   - Voice input button (microphone icon, right)
   - Send button (blue, appears when text entered)

2) Suggestion chips (below input, horizontal scrollable):
   - "Revenue last month", "Top customers", "Growth rate", "Churn trend"

3) Results area (below, scrollable):
   After querying "What was revenue by region last quarter?":

   a) Confidence badge: "94% confident" (green, inline)

   b) Answer text (full width card):
      "Total revenue in Q4 was **$12.8M**. North America led at **$5.8M** (45%)."
      - Expandable for full answer text

   c) Chart (full width, 220px height):
      - Horizontal bar chart: Revenue by region
      - Tap chart to expand full screen
      - Chart type switcher strip: Bar | Pie | Table

   d) Data table (collapsible):
      - Compact: Region, Revenue, % Total
      - Horizontal scroll if needed

   e) SQL toggle (collapsible):
      - "View SQL" expandable section
      - Monospace SQL with copy button

   f) Action buttons (stacked, full width):
      - "Save as Widget" (blue outline)
      - "Export" (gray outline)
      - "Refine" (opens follow-up input)

4) Conversation history (accessible via history icon):
   - Full-screen list of past queries
   - Tap to re-run or view cached results
   - Swipe to delete

5) Follow-up suggestions (below results):
   - "Show monthly trend", "Break down by product", "Compare to Q3"
   - Tap to auto-fill and submit

States: Empty (hero with suggestions), querying (animated progress "Analyzing your question..."), results loaded, low confidence (amber banner with "Try rephrasing"), error (red banner with text input encouragement), voice listening mode.
```

#### Prompt F-011: Mobile Report Viewer

```
Design the Report viewer for ERP-BI at 390px mobile width.

Layout:
- Sticky top bar: Back arrow, "Q4 Executive Summary" title (truncated), three-dot menu (download, share, schedule)
- No bottom navigation (full-screen report)

Content:

1) Report library (main Reports page):
   - Search input at top
   - Filter chips: All, My Reports, Scheduled, Shared, Templates
   - Report cards (single column):
     Each card: Report name, type badge, last generated date, format icons (PDF, Excel), preview thumbnail
     Tap to open viewer
   - Sample:
     - "Q4 Executive Summary" — Scheduled — Feb 23 — PDF, Excel
     - "Monthly Sales Report" — Ad-hoc — Feb 20 — PDF
     - "Customer Retention Analysis" — Template — Feb 15 — PDF

2) Report viewer (when report opened):
   - Paginated PDF-style view
   - Pinch-to-zoom
   - Page indicator: "Page 1 of 3"
   - Swipe left/right for page navigation
   - Tap to toggle controls visibility
   - Controls overlay (semi-transparent bottom bar):
     - Page navigation arrows
     - Page number input
     - Zoom in/out
     - Rotate (landscape for charts)

3) Report actions (three-dot menu):
   - Download PDF
   - Download Excel
   - Share via link
   - Share via email
   - Schedule recurring delivery
   - Print (if supported)

States: Report library loading, report loading (progress bar), report loaded, report generation in progress ("Generating report... 45%"), report error (retry CTA), no reports available.
```

#### Prompt F-012: Mobile Alert Feed

```
Design the Alerts feed for ERP-BI at 390px mobile width.

Layout:
- Sticky top bar: Hamburger menu, "Alerts" title, "+" button (create alert)
- Bottom navigation bar: Dashboards, Reports, NLQ, Alerts (active), More

Content:

1) Alert summary strip (below top bar):
   - "3 triggered" (red badge), "23 active" (green badge), "7 fired today" (gray badge)

2) Triggered alerts section (prominent):
   - Section header: "Triggered Now" with count
   - Alert cards (red/amber left border, full width):
     Each card:
     - Severity icon and color (Critical: red, Warning: amber)
     - Alert name (bold): "Daily Revenue Below Threshold"
     - Current value: "$127,340" (red) vs threshold "$150,000"
     - Triggered time: "2 hours ago"
     - Quick actions row: "Acknowledge" | "Snooze" | "Investigate"
     - Tap card to expand with metric chart and details

3) All alerts list (below triggered):
   - Section header: "All Alert Rules"
   - Compact list items:
     - Status indicator (green/red/amber dot)
     - Alert name (1 line truncated)
     - Condition summary: "Revenue < $150K"
     - Last triggered: "2h ago" or "Never"
     - Tap to open detail

4) Alert detail (separate screen):
   - Full alert name and description
   - Metric chart: Line chart showing metric over time with threshold line
   - Current value and threshold clearly displayed
   - Notification channels list
   - History: Recent triggers timeline
   - Actions: Edit, Pause, Delete, Snooze

5) Create alert (full-screen modal from "+" button):
   - Simplified mobile flow:
     Step 1: Pick metric (search + category browse)
     Step 2: Set condition and threshold
     Step 3: Choose notifications (pre-populated with email and Slack)
     Step 4: Activate
   - One step per screen, "Next" navigation

States: No alerts (create first alert CTA), all healthy (green checkmark illustration), critical alert with urgent styling, alert creation flow, offline (show last known alert states with timestamp).

Push notification design: Include a sample notification banner: "BI Alert: Daily Revenue dropped to $127K (threshold: $150K). Tap to investigate."
```

### 3.4 Tablet/Responsive (1024px)

```
Tablet adaptations for ERP-BI at 1024px:

General rules:
- Left sidebar collapses to icon-only mode (56px) by default; expand on hover or tap
- Content area uses full remaining width
- Bottom navigation bar: Hidden; all navigation via sidebar

Specific adaptations:
- Dashboard: KPI tiles in 2x2 grid; Chart widgets stack to single column (full width each); Filter bar wraps to two lines if needed; Edit mode grid maintains but with fewer snap columns
- NLQ: Query input remains full width; Results take full width; Suggestion chips wrap to 2 rows; Recent queries sidebar becomes a header dropdown
- Report Builder: Component palette collapses to icon-only toolbar (expandable); Canvas takes 65% width; Properties panel takes 35%; On very complex reports, properties panel becomes a bottom sheet
- Data Warehouse: Schema browser at 240px; Query editor and results take remaining width; Results panel below editor with adjustable split
- Alert Management: Triggered alerts remain full width cards; Alert rules table: Card view instead of table for readability; Alert creation modal max-width 600px
- Data Modeling: Canvas takes full width; Properties panel becomes a slide-over; Minimap repositions to bottom-right overlay
- Charts: Maintain full fidelity at 1024px; Reduce legend to inline format where possible; Tooltips remain hover-triggered (not long-press as on mobile)
- Tables: Horizontally scrollable with sticky first column and sticky header
- Modals: Max width 640px, centered
- Command palette: Full width minus 48px padding
```

---

## 4. Make Automation Prompts

### Prompt M-001: Dashboard Data Refresh Orchestrator

```
Build a Make scenario for orchestrating dashboard data refreshes in ERP-BI:

Trigger: Schedule — Every 15 minutes (configurable per dashboard)

Steps:
1. HTTP GET /v1/dashboard?auto_refresh=true
   Response: Array of dashboards with { "dashboard_id", "name", "refresh_interval_minutes", "last_refreshed", "widgets": [{ "widget_id", "data_source", "query_id" }], "subscribers_count" }

2. For each dashboard due for refresh:
   a. For each widget in dashboard:
      - POST /v1/query-engine/execute with { query_id, timeout: 30000, cache_ttl: refresh_interval }
      - Track execution time per query.
   b. If all widget queries succeed:
      - PATCH /v1/dashboard/{dashboard_id} with { last_refreshed: now() }.
      - If any query took > 5s: Log slow query to Google Sheets "Query Performance Log".
   c. If any widget query fails:
      - Mark widget as stale: PATCH /v1/dashboard/{dashboard_id}/widgets/{widget_id} with { stale: true, error: error_message }.
      - Continue refreshing other widgets (do not fail entire dashboard).
      - If > 50% of widgets fail: Mark dashboard as degraded.

3. For degraded dashboards:
   - Send Slack notification to #bi-ops: "Dashboard '{name}' is degraded. {failed_count}/{total_count} widgets failed to refresh."
   - If degraded for > 1 hour: Create PagerDuty incident.
   - Send email to dashboard owner.

4. Performance tracking:
   - Log per-dashboard: refresh time, widget count, slow queries, failed queries.
   - If average refresh time exceeds 60s: Flag for query optimization review.
   - Daily at 08:00 UTC: Send refresh health summary to #bi-metrics.

5. Smart caching:
   - If dashboard has 0 active viewers (no page views in 30 min): Skip refresh, mark as idle.
   - If dashboard has > 10 active viewers: Increase refresh priority.

Guardrails: Maximum 50 concurrent query executions. Timeout per query: 30s. Rate limit failed dashboard alerts to 1 per dashboard per hour. Preserve query execution metrics for 90 days. Never execute queries against warehouse during maintenance windows.
```

### Prompt M-002: Alert Evaluation and Notification Pipeline

```
Build a Make scenario for evaluating alert conditions and sending notifications in ERP-BI:

Trigger: Schedule — Every 5 minutes (evaluation frequency is per-alert, but the scenario polls all due alerts)

Steps:
1. HTTP GET /v1/alert?status=active&due=true
   Response: Array of alerts with { "alert_id", "name", "metric_query_id", "condition": { "operator": "<", "threshold": 150000 }, "severity", "min_consecutive_violations": 2, "current_violation_streak": 1, "notification_channels": ["slack:#revenue-alerts", "email:finance@company.com"], "escalation": { "after_minutes": 30, "to": "email:cfo@company.com" } }

2. For each due alert:
   a. Execute metric query: POST /v1/query-engine/execute with { query_id: metric_query_id }.
   b. Evaluate condition: Compare result against threshold using operator.
   c. If condition violated:
      - Increment violation streak: current_violation_streak + 1.
      - If streak >= min_consecutive_violations AND alert not already triggered:
        - Mark alert as triggered: PATCH /v1/alert/{alert_id} with { status: "triggered", triggered_at: now(), current_value: result }.
        - Send notifications to all channels:
          Slack: "Alert: {name} triggered. {metric}: {current_value} {operator} {threshold}. Severity: {severity}. [Investigate]({dashboard_link})"
          Email: HTML email with metric chart, current vs threshold, recommended actions, deep link to dashboard.
        - Log to event bus: Publish "erp.bi.alert.triggered" event to Redpanda.
      - If alert already triggered AND escalation.after_minutes elapsed AND not acknowledged:
        - Escalate: Send notification to escalation recipients.
        - Update alert: PATCH with { escalated: true, escalated_at: now() }.
   d. If condition NOT violated:
      - Reset violation streak to 0.
      - If alert was previously triggered:
        - Auto-resolve: PATCH /v1/alert/{alert_id} with { status: "active", resolved_at: now(), resolution: "auto_resolved" }.
        - Send resolution notification: "Alert '{name}' auto-resolved. Metric back to normal: {current_value}."
        - Publish "erp.bi.alert.resolved" event.

3. Track alert metrics:
   - Store evaluation results: POST /v1/alert/{alert_id}/history with { evaluated_at, value, triggered }.
   - Daily: Compute alert noise ratio (triggered vs total evaluations). If noise > 20%: Flag for threshold review.

4. Daily alert digest at 09:00 UTC:
   - Summary: Active alerts, triggered in last 24h, resolved in last 24h, most triggered alert, noisiest alert.
   - Send to #bi-alerts and alert owners.

Guardrails: Maximum 100 metric queries per evaluation cycle. Timeout per query: 10s. Deduplicate notifications per alert within evaluation window. Never suppress critical alerts. Rate limit escalation notifications to 1 per alert per hour. Preserve all evaluation history for audit (90-day retention).
```

### Prompt M-003: Report Generation and Distribution

```
Build a Make scenario for automated report generation and distribution in ERP-BI:

Trigger: Schedule — Every 15 minutes (checks for due scheduled reports) + Webhook on "erp.bi.report.requested" event

Steps:
1. HTTP GET /v1/report/scheduled?due=true
   Response: Array of due reports with { "report_id", "name", "template_id", "parameters": { "date_range": "last_month", "region": "all", "format": "pdf" }, "schedule": { "frequency": "monthly", "day": 1, "time": "08:00" }, "recipients": [{ "type": "email", "address": "exec-team@company.com" }, { "type": "slack", "channel": "#monthly-reports" }] }

2. For each due report:
   a. Generate report:
      - POST /v1/report/generate with { template_id, parameters, format }.
      - Response includes: { "generation_id", "status": "processing" }.
      - Poll for completion: GET /v1/report/generate/{generation_id} every 10s, max 5 min.

   b. If generation succeeds:
      - Download generated file: GET /v1/report/generate/{generation_id}/download.
      - For each recipient:
        - Email: Send email with report attached, subject "ERP-BI: {report_name} — {date_range}", body with summary stats preview.
        - Slack: Upload file to channel with message "Monthly report ready: {report_name}. Key highlights: Revenue {total}, Growth {growth}%."
        - S3: Upload to tenant's report archive: s3://erp-bi-reports/{tenant_id}/{year}/{month}/{report_name}.pdf.
      - Update report status: PATCH /v1/report/{report_id} with { last_generated: now(), last_status: "success" }.
      - Publish "erp.bi.report.delivered" event.

   c. If generation fails:
      - Retry once after 5 minutes.
      - If retry fails:
        - Notify report owner: "Report '{report_name}' failed to generate. Error: {error_message}. [View Details]"
        - Send Slack alert to #bi-ops: "Scheduled report '{report_name}' generation failed."
        - Update report status: PATCH with { last_status: "failed", error: error_message }.

3. For webhook-triggered (on-demand) reports:
   - Same generation flow but with immediate notification to requesting user.
   - Include "Your report is ready" push notification with download link.

4. Report quality checks (before delivery):
   - Verify report is not empty (has at least 1 data point).
   - Verify file size is reasonable (not 0 bytes, not > 50MB).
   - If report contains "No data available" warning: Notify owner, still deliver with warning banner.

5. Monthly report archive audit:
   - On 1st of each month: Count reports by type, failures, avg generation time.
   - Send to #bi-ops: "Report generation summary: {total} reports, {success_rate}% success rate, avg {gen_time}s."

Guardrails: Maximum 10 concurrent report generations. Timeout per report: 5 minutes. Never send reports containing errors without warning banner. Respect tenant data isolation in multi-tenant deployments. Rate limit report generation per user to 20 per hour. Preserve generation logs for 1 year.
```

### Prompt M-004: Data Quality Monitor

```
Build a Make scenario for continuous data quality monitoring in ERP-BI:

Trigger: Schedule — Every 1 hour + Webhook on "erp.bi.data-warehouse.sync.completed" event

Steps:
1. HTTP GET /v1/data-warehouse/tables
   Response: Array of tables with { "table_name", "schema", "row_count", "last_updated", "size_bytes" }

2. For each monitored table, run quality checks:
   a. Freshness check:
      - POST /v1/query-engine/execute with query: "SELECT MAX(updated_at) FROM {table}".
      - If data is older than expected freshness threshold (configurable per table):
        - Flag: "Table '{table_name}' data is stale. Last update: {last_updated}. Expected: within {threshold}."
   b. Volume check:
      - Compare current row_count to historical average (from previous checks).
      - If row_count dropped > 10% unexpectedly: Flag "Table '{table_name}' row count dropped by {delta}% ({previous} -> {current})."
      - If row_count increased > 200% unexpectedly: Flag "Table '{table_name}' row count surged by {delta}%."
   c. Null check on critical columns:
      - POST /v1/query-engine/execute with query: "SELECT COUNT(*) FILTER (WHERE {column} IS NULL) * 100.0 / COUNT(*) FROM {table}".
      - If null percentage exceeds threshold (e.g., > 5% for required fields): Flag.
   d. Duplicate check on primary keys:
      - POST /v1/query-engine/execute with query: "SELECT {pk_column}, COUNT(*) FROM {table} GROUP BY {pk_column} HAVING COUNT(*) > 1 LIMIT 10".
      - If duplicates found: Flag with sample duplicate keys.

3. For each flagged issue:
   a. Severity classification:
      - Critical: Duplicates on primary keys, data freshness > 4x threshold.
      - Warning: Freshness 1-4x threshold, volume anomalies, null rate increases.
      - Info: Minor null rate changes, expected volume fluctuations.
   b. Notifications:
      - Critical: Slack #data-quality-critical + PagerDuty + email to data-engineering@company.com.
      - Warning: Slack #data-quality + email to data-team@company.com.
      - Info: Log only, include in daily digest.
   c. Impact assessment:
      - Identify affected dashboards: GET /v1/dashboard?data_source_contains={table_name}.
      - Identify affected reports: GET /v1/report?data_source_contains={table_name}.
      - Include in notification: "This affects {dashboard_count} dashboards and {report_count} reports."

4. Store quality scores:
   - POST /v1/data-modeling/quality-scores with per-table quality metrics.
   - Compute overall data quality score: Weighted average of freshness, completeness, uniqueness, volume stability.

5. Daily data quality report at 07:00 UTC:
   - Overall quality score: 94.2% (trend +0.3%)
   - Tables with issues: List with severity
   - Resolved issues in last 24h
   - Recommendations: "Table 'orders' freshness degrading — check ETL pipeline."
   - Send to #data-quality-daily and data leadership.

Guardrails: Maximum 200 quality check queries per run. Timeout per query: 15s. Never modify source data during checks. Cache quality results for comparison. Rate limit critical alerts to 1 per table per 2 hours. Preserve quality history for 1 year for trend analysis.
```

### Prompt M-005: NLQ Usage Analytics and Improvement

```
Build a Make scenario for tracking NLQ usage patterns and improving query accuracy in ERP-BI:

Trigger: Schedule — Daily at 01:00 UTC (low-traffic analysis window)

Steps:
1. HTTP GET /v1/nlq/queries?date=yesterday
   Response: Array of NLQ queries with { "query_id", "user_id", "natural_language_input", "generated_sql", "confidence_score", "execution_status", "result_row_count", "execution_time_ms", "user_feedback": "positive|negative|none", "refinement_count", "saved_as_widget": boolean }

2. Analyze query patterns:
   a. Success metrics:
      - Total queries, success rate, average confidence, average execution time.
      - Positive feedback rate, negative feedback rate, no-feedback rate.
      - Query categories: Revenue (34%), Customers (22%), Operations (18%), Products (14%), Other (12%).

   b. Failed queries analysis:
      - Group by failure reason: SQL error, timeout, zero results, low confidence (<60%).
      - Extract common failure patterns:
        - "Queries about '{topic}' fail {count} times — missing table or column?"
        - "Low confidence on queries involving '{entity}' — data model may need synonym mapping."
      - Identify tables/columns most associated with failures.

   c. User behavior:
      - Average refinements per query (lower is better).
      - Most active NLQ users.
      - Queries saved as widgets (indicates high value).
      - Time of day distribution.

3. Generate improvement recommendations:
   a. For data modeling team:
      - "Add synonym '{term}' for column '{column}' — 15 failed queries used this term yesterday."
      - "Table '{table}' is frequently queried but not in the data model — consider adding."
      - "Column description for '{column}' is missing — NLQ confidence improves with descriptions."
   b. For NLQ service:
      - "These 5 query patterns had consistently low confidence — add training examples."
      - Store failed query patterns for model fine-tuning.

4. Store analytics:
   - POST to Google Sheets "NLQ Daily Metrics": Date, total queries, success rate, avg confidence, feedback distribution, top failure reasons.
   - POST failed patterns to /v1/nlq/training-data for model improvement.

5. Reports:
   a. Daily: Send NLQ usage summary to #nlq-metrics (Slack).
   b. Weekly: Send detailed analysis with improvement recommendations to #bi-product.
   c. Monthly: Executive summary of NLQ adoption: "NLQ handled {count} queries this month, saving an estimated {hours} hours of manual SQL writing."

Guardrails: Never log actual user query content in plain text to external systems (hash or anonymize). Aggregate user-level metrics (no individual user tracking in external reports). Rate limit training data submissions to prevent model poisoning. Preserve raw query logs for 30 days only (then anonymize and aggregate). Respect tenant isolation in multi-tenant analysis.
```

---

## 5. Prompt Usage Guidelines

### 5.1 Figma Prompt Best Practices
1. **Copy-paste directly** into Figma Make's prompt input. Each prompt is self-contained.
2. **Generate at the specified breakpoint** first (1440px for desktop, 390px for mobile), then use Figma's responsive features to verify at intermediate widths.
3. **Always generate both light and dark variants** unless the prompt specifies otherwise.
4. **Review AIDD guardrails** against each generated output before accepting — use the handoff gate template in Section 8.
5. **Iterate on component-level prompts** (F-001, F-002) before page-level prompts to establish a consistent design system.
6. **Use realistic financial and business data** from the sample values provided; avoid lorem ipsum. Numbers should use locale-aware formatting.
7. **Verify chart accessibility**: Run color-blind simulation on all chart color combinations. Ensure patterns or shapes supplement color coding.
8. **Verify data table readability**: Tables with many columns should demonstrate horizontal scroll behavior with sticky first column.
9. **Test NLQ prompts** with both simple and complex query examples to verify the interface handles varying result complexities.

### 5.2 Make Automation Best Practices
1. **Set up webhook endpoints** in your Make environment before deploying scenarios.
2. **Use environment variables** for API keys, Slack tokens, database connection strings, and endpoint URLs — never hardcode.
3. **Test with sample payloads** provided in each prompt before connecting to live Redpanda topics.
4. **Configure error handling** on every HTTP module: Retry 3x with exponential backoff (30s, 60s, 120s).
5. **Set up a dead-letter queue** scenario to capture events that fail all retries.
6. **Rate limit external notifications** as specified in each scenario's guardrails section.
7. **Respect data warehouse query limits**: Schedule heavy analytics during off-peak hours (01:00-06:00 UTC).
8. **Monitor Make scenario execution logs** weekly for silent failures or quota exhaustion.
9. **Data quality checks should not modify source data** — read-only operations only.
10. **NLQ training data submissions** must be reviewed before being applied to production models.

---

## 6. Output Packaging Convention

Request Figma Make outputs under these pages:

```
00_Context — Project brief, personas (analyst, executive, data engineer, business user), architecture diagram reference
01_IA_Flows — Information architecture, navigation map, user flow diagrams (dashboard creation, NLQ query flow, report generation, alert setup, data modeling)
02_Design_Tokens — Color, typography, spacing, elevation, motion tokens (light + dark), chart color palette with accessibility verification
03_Components — Full component library with all state variants (charts, dashboards, tables, NLQ, reports, alerts, data modeling)
04_Desktop_1440 — All desktop page designs (Dashboard, NLQ, Report Builder, Data Warehouse Explorer, Alert Console, Data Modeling Studio)
05_Tablet_1024 — Tablet-adapted layouts
06_Mobile_390 — All mobile page designs (Dashboard Viewer, NLQ, Report Viewer, Alert Feed)
07_Chart_Gallery — Dedicated page showing all chart types with sample data, color-blind safe variants, dark mode variants
08_States_And_Errors — Empty, loading, error, success, stale data, refreshing, offline states for every surface
09_Observability_And_Testability — Analytics event map, query performance instrumentation, test state matrix
10_Handoff_AIDD_Checklist — Completed AIDD gate template per page
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| LCP | <= 2.5s at p75 | Mid-tier mobile on 4G |
| INP | <= 200ms at p75 | All interactive surfaces |
| CLS | <= 0.10 | All pages |
| Time to first interaction | <= 1.8s | Key routes (Dashboard, NLQ, Reports) |
| JS route budget | <= 220KB gzip | Per initial route payload (excluding chart libraries) |
| Dashboard initial render | <= 3s | All above-fold widgets visible with data |
| Dashboard full render | <= 5s | All widgets on page loaded with data |
| NLQ query-to-result | <= 5s | From submit to first result visible (p75) |
| NLQ confidence display | <= 500ms | Confidence badge visible before full results |
| Chart render | <= 500ms | Per individual chart widget |
| Data table virtual scroll | 60fps | For tables with 1000+ rows |
| Report generation | <= 30s | For standard reports (< 20 pages) |
| Alert evaluation cycle | <= 2 min | Full cycle for all active alerts |
| SQL query editor autocomplete | <= 200ms | Suggestion dropdown appearance |
| Data model canvas render | <= 1s | For models with up to 50 tables |

---

## 8. AIDD Handoff Gate Template

Attach this completed checklist to every generated design before engineering handoff:

```
[ ] 1. Accessibility gate passed
      - Color contrast ratios verified (4.5:1 normal text, 3:1 large text)
      - Chart colors verified for color-blind accessibility (all 3 types: protanopia, deuteranopia, tritanopia)
      - Charts use patterns/shapes as secondary indicators alongside color
      - Focus order annotated for all interactive elements, including data table cell navigation
      - Keyboard navigation for charts: arrow keys for data point selection, Enter for drill-down
      - Screen-reader announcements for: data updates, alert triggers, query results, chart data summaries
      - Tap targets >= 44x44px on touch surfaces
      - Large numbers use tabular-nums and locale-aware formatting

[ ] 2. Performance gate passed
      - Route-level lazy loading boundaries defined
      - JS bundle budget per route documented (chart libraries deferred)
      - Dashboard widgets load progressively (above-fold first)
      - Skeleton/placeholder dimensions match final layout (no CLS)
      - Chart components deferred until visible in viewport
      - Data tables virtualize rows for large datasets (>100 rows)
      - Image strategy for report thumbnails: responsive srcset, WebP/AVIF

[ ] 3. Reliability gate passed
      - Error states defined for every query-dependent surface
      - Individual widget errors do not crash entire dashboard
      - Retry paths for failed queries, report generations, alert evaluations
      - Stale data warnings with last successful refresh timestamp
      - Offline mode: Show cached dashboard data with clear offline indicator
      - NLQ graceful degradation: Fall back to SQL editor on NLQ service failure

[ ] 4. Observability gate passed
      - Analytics events: dashboard views, widget interactions, NLQ queries, report downloads, alert acknowledgments
      - Query performance instrumented: execution time, rows scanned, cache hit rate
      - Error events with trace ID correlation across query-engine, data-warehouse, and nlq-service
      - Performance marks (LCP, INP, CLS) per page
      - Dashboard engagement: time on dashboard, widget click-through, filter usage

[ ] 5. Testability gate passed
      - State matrix mapped to test cases for all components
      - Chart component tests: empty data, single data point, large dataset, negative values, null values
      - NLQ tests: simple query, complex query, ambiguous query, no results, low confidence
      - API mock requirements specified per page
      - E2E critical paths: Dashboard view with filters, NLQ question-to-widget, Report generate-to-download, Alert create-to-trigger

[ ] 6. Security and privacy gate passed
      - Data access respects tenant isolation and user entitlements
      - NLQ queries restricted to user's authorized data sources
      - Report downloads audit-logged with user, timestamp, and report metadata
      - SQL editor: Prevent write operations (INSERT, UPDATE, DELETE, DROP) in ad-hoc queries
      - Alert notifications: No raw PII in external channels (Slack, email) — use aggregated values
      - Query results: Row-level security applied before display
      - Export data: Include watermark with user identity and timestamp
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial prompt pack covering all 7 ERP-BI services: alert-service, dashboard-service, data-modeling-service, data-warehouse-service, nlq-service, query-engine, report-service |
