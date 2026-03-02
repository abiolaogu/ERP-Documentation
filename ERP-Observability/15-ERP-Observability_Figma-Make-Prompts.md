# Figma & Make Prompts -- ERP-Observability (Unified Observability Platform)
> Version: 1.0 | Last Updated: 2026-02-24 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make (Integromat) automation prompts for the ERP-Observability module. ERP-Observability is the unified observability vertical of the consolidated ERP platform, covering metrics collection and visualization (VictoriaMetrics), log aggregation and search (Quickwit), distributed tracing (OpenTelemetry + Quickwit), alert management (Alertmanager), custom dashboards (Grafana-style builder), multi-tenant management, infrastructure monitoring (Zabbix proxy, OpenNMS event correlation), and cross-module unified search. All prompts reference real services (observability-api, otel-collector, victoria-metrics, quickwit, grafana, alertmanager, zabbix-proxy, opennms) and align with the AIDD guardrails defined in `erp/aidd.guardrails.yaml`.

The target frontend stack is **Refine.dev + Ant Design** (React) with Hasura GraphQL federation (port 19120), `obs_` table prefix, multi-tenant isolation via `X-Scope-OrgID` and `X-Tenant-ID` headers, and DragonflyDB caching layer.

Key backend components:
- **VictoriaMetrics** -- high-performance metrics TSDB (PromQL-compatible)
- **Quickwit** -- log and trace storage with full-text search
- **OTel Collector** -- OpenTelemetry pipeline for metrics, logs, and traces
- **Grafana** -- embedded visualization and dashboard engine
- **Alertmanager** -- alert routing, grouping, silencing, and notification
- **Zabbix Proxy** -- infrastructure host/network monitoring
- **OpenNMS** -- event correlation and topology mapping
- **DragonflyDB** -- in-memory cache for hot queries and metadata
- **YugabyteDB** -- distributed SQL for configuration and tenant metadata
- **RustFS** -- S3-compatible object storage for long-term retention

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance on every screen; color contrast ratio >= 4.5:1 for text, >= 3:1 for large text and UI components
- Minimum 44x44px touch targets on all interactive elements (buttons, links, checkboxes, table row actions)
- Clear focus-visible outlines (2px solid, offset 2px) for keyboard navigation
- All form fields include visible labels (never placeholder-only), required field indicators, and inline validation messages
- Plain-language labels (e.g., "Active Alerts" not "Alert Rule Instance Management")
- Semantic HTML landmarks: `<nav>`, `<main>`, `<aside>`, `<footer>` reflected in Figma layer naming
- Screen reader annotations for charts (alt-text, data tables), icon-only buttons (aria-label), and live regions
- Log and trace views must support high-density monospace rendering while maintaining readability at 13px minimum

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading; initial route bundle < 220 KB gzipped
- Skeleton UIs shown within 200ms; full content within 1s on 4G
- Virtualized tables for datasets > 50 rows (log entries, metric series, trace spans, alert lists)
- Streaming log tail with backpressure; WebSocket via `graphql-ws` subscription
- Sparkline and time-series charts rendered with Canvas (not SVG) for > 1000 data points
- Prefetch next-likely routes (e.g., metric detail from metrics list, trace detail from trace list)
- DragonflyDB-backed query cache: hot metric queries served in < 50ms

### 2.3 Reliability, Trust, And Safety
- Every destructive action (delete alert rule, remove tenant, purge logs) requires a confirmation dialog with explicit typing or checkbox
- Inline recovery paths on all error states ("Retry", "Go back", "Contact support")
- Optimistic UI for low-risk mutations (silence alert, toggle alert rule) with undo toast (5s window)
- Trust signals: last-synced timestamp on dashboards, data freshness indicator, ingestion lag display
- Tenant isolation enforced at every API layer via `X-Scope-OrgID` header; cross-tenant data leakage is a P0 security defect
- Audit trail accessible for all configuration changes (alert rules, dashboard edits, tenant quota modifications)

### 2.4 Observability And Testability
- Every CTA emits a structured analytics event: `{event, module, entity, action, user_role, timestamp}`
- Error boundaries at page and widget level with correlation_id for support tickets
- Performance instrumentation: LCP, FID, CLS metrics on all page-level components
- Feature flag integration points annotated in Figma (e.g., "Visible when `opennms_correlation_enabled` = true")
- All dashboard panels include a "Query Inspector" debug drawer showing the underlying PromQL/LogQL/TraceQL query

### 2.5 Security And Privacy
- All API calls include `X-Tenant-ID` header; backend rejects requests without valid tenant context
- Log entries may contain PII; redaction toggle available in log viewer (masks emails, IPs, tokens)
- Alert notification channels (Slack, email, webhook) require validation before activation
- Dashboard sharing generates scoped embed tokens with configurable TTL and IP restrictions
- Role-based access: Viewer (read-only), Operator (silence/acknowledge), Editor (create/edit), Admin (tenant management)

---

## 3. Module-Specific Design Context

### 3.1 Backend Services

| Service | Port | Description |
|---------|------|-------------|
| observability-api | 19100 | Core REST/gRPC API gateway |
| hasura-graphql | 19120 | Hasura GraphQL federation endpoint |
| otel-collector | 4317/4318 | OpenTelemetry gRPC/HTTP receivers |
| victoria-metrics | 8428 | Metrics TSDB (PromQL read/write) |
| quickwit | 7280 | Log and trace search engine |
| grafana | 3000 | Visualization and dashboard engine |
| alertmanager | 9093 | Alert routing and notification |
| zabbix-proxy | 10051 | Infrastructure monitoring proxy |
| opennms | 8980 | Event correlation and topology |
| dragonflydb | 6379 | In-memory cache |
| yugabytedb | 5433 | Distributed SQL (config/metadata) |
| rustfs | 9000 | S3-compatible object storage |

### 3.2 User Personas

| Persona | Role | Primary Tasks |
|---------|------|---------------|
| Platform Engineer | SRE/DevOps | Monitor system health, investigate incidents, configure alerts |
| Application Developer | Dev | Search logs for debugging, trace request flows, view app metrics |
| Security Analyst | SecOps | Audit log analysis, anomaly detection, compliance reporting |
| Tenant Administrator | Ops Manager | Manage tenant quotas, provision new tenants, review usage |
| Infrastructure Engineer | NetOps | Monitor Zabbix hosts, OpenNMS events, network topology |
| Engineering Manager | Team Lead | Dashboard overview, SLO tracking, incident retrospectives |

### 3.3 Key Entities (obs_ prefix)

| Entity | Table | Key Fields |
|--------|-------|------------|
| MetricSeries | obs_metric_series | id, name, labels, module, type (counter/gauge/histogram/summary), value, timestamp, tenant_id |
| LogEntry | obs_log_entries | id, timestamp, level (debug/info/warn/error/fatal), message, module, service, trace_id, span_id, tenant_id, metadata |
| TraceSpan | obs_trace_spans | id, trace_id, span_id, parent_span_id, operation_name, service_name, start_time, duration, status (ok/error/unset), tags, events, tenant_id |
| AlertRule | obs_alert_rules | id, tenant_id, name, description, severity (info/warning/critical), module, promql_expression, duration, labels, annotations, enabled |
| AlertInstance | obs_alert_instances | id, alert_rule_id, status (firing/pending/resolved), starts_at, ends_at, value, labels, annotations |
| Dashboard | obs_dashboards | id, tenant_id, name, description, panels[], tags[], created_by |
| DashboardPanel | obs_dashboard_panels | id, title, type (graph/stat/table/heatmap/logs), query, position (x/y/w/h), options |
| ObsTenant | obs_tenants | id, name, display_name, status (active/suspended/provisioning), tier, retention_days, quotas (metrics_per_second, logs_per_second, traces_per_second, storage_gb) |

### 3.4 Status Color Mappings

| Context | Value | Color |
|---------|-------|-------|
| Severity | info | #3b82f6 (Blue) |
| Severity | warning | #f59e0b (Amber) |
| Severity | critical | #ef4444 (Red) |
| Log Level | debug | #94a3b8 (Slate) |
| Log Level | info | #3b82f6 (Blue) |
| Log Level | warn | #f59e0b (Amber) |
| Log Level | error | #ef4444 (Red) |
| Log Level | fatal | #7c2d12 (Dark Red) |
| Alert Status | firing | #ef4444 (Red) |
| Alert Status | pending | #f59e0b (Amber) |
| Alert Status | resolved | #10b981 (Emerald) |
| Tenant Status | active | #10b981 (Emerald) |
| Tenant Status | suspended | #ef4444 (Red) |
| Tenant Status | provisioning | #3b82f6 (Blue) |
| Span Status | ok | #10b981 (Emerald) |
| Span Status | error | #ef4444 (Red) |
| Span Status | unset | #94a3b8 (Slate) |
| Module Health | healthy | #10b981 (Emerald) |
| Module Health | degraded | #f59e0b (Amber) |

---

## 4. Figma Make Master Prompt (Copy-Paste Ready)

```
Design a complete Figma project for the ERP-Observability module -- a unified
observability platform for an enterprise ERP system. The platform consolidates
metrics, logs, traces, alerts, custom dashboards, multi-tenant management, and
infrastructure monitoring into a single SPA.

TECH STACK CONTEXT:
- Frontend: React + Refine.dev + Ant Design + graphql-request + graphql-ws
- Backend: Hasura GraphQL (port 19120), observability-api (Go microservice)
- Data stores: VictoriaMetrics (metrics), Quickwit (logs/traces), YugabyteDB (metadata)
- Cache: DragonflyDB | Object storage: RustFS (S3-compatible)
- Infrastructure: Zabbix Proxy, OpenNMS, OTel Collector
- Multi-tenant: X-Scope-OrgID and X-Tenant-ID headers

BRAND / THEME:
- Primary: #059669 (Emerald Green -- represents system health and uptime)
- Primary Hover: #047857
- Primary Light: #d1fae5
- Success: #10b981 (Healthy, Resolved, Active)
- Warning: #f59e0b (Pending, Degraded, Expiring)
- Danger: #ef4444 (Critical, Error, Firing, Suspended)
- Info: #3b82f6 (Informational, Debug links)
- Accent Purple: #8b5cf6 (Traces, Distributed systems)
- Neutral-50: #f5f7fa (Page background / layout body)
- Neutral-100: #fafbfc (Table header background)
- Neutral-200: #e5e7eb (Borders, dividers)
- Neutral-500: #64748b (Secondary text, table headers)
- Neutral-900: #111827 (Primary text)
- Surface: #ffffff (Cards, modals)
- Sidebar Background: #001529 (Dark navy)
- Sidebar Active: #059669 (Emerald, matches primary)
- Font Family: "Plus Jakarta Sans", system stack fallback
- Monospace: "JetBrains Mono" for metric names, PromQL, log lines, trace IDs, JSON
- Border Radius: 10px (cards), 8px (buttons, inputs, tags), 6px (small tags)
- Control Height: 40px (buttons, inputs, selects, date pickers)
- Card Padding: 24px
- Statistic Font Size: 28px

NAVIGATION (Sidebar, 260px expanded / 72px collapsed):
- Logo: 32x32px rounded square with gradient (#059669 to #0d9488), "OBS" text
- Title: "ERP Observability" (hidden when collapsed)
- Menu Items:
  1. Dashboard (DashboardOutlined icon) -- /
  2. Metrics (LineChartOutlined) -- /metrics, /metrics/:id
  3. Logs (FileTextOutlined) -- /logs, /logs/:id
  4. Traces (ApartmentOutlined) -- /traces, /traces/:id
  5. Alerts (AlertOutlined) -- /alerts, /alerts/new, /alerts/:id
  6. Dashboards (AppstoreOutlined) -- /dashboards, /dashboards/new, /dashboards/:id
  7. Tenants (TeamOutlined) -- /tenants, /tenants/new, /tenants/:id
  8. Search (SearchOutlined) -- /search
  9. Infrastructure (MonitorOutlined, parent group):
     a. Zabbix (MonitorOutlined) -- /infrastructure/zabbix
     b. OpenNMS (ClusterOutlined) -- /infrastructure/opennms
- Active state: #059669 background, white text, 8px border radius
- Hover state: rgba(5, 150, 105, 0.6)
- Sidebar theme: dark (#001529)

PAGES TO DESIGN (18 total):
1.  Login -- /login
2.  Dashboard -- / (KPI command center)
3.  Metrics List -- /metrics
4.  Metric Show -- /metrics/:id
5.  Logs List -- /logs
6.  Log Show -- /logs/:id
7.  Traces List -- /traces
8.  Trace Show -- /traces/:id
9.  Alert List -- /alerts
10. Alert Create -- /alerts/new
11. Alert Show -- /alerts/:id
12. Dashboard List -- /dashboards
13. Dashboard Create -- /dashboards/new
14. Dashboard Show -- /dashboards/:id
15. Tenant List -- /tenants
16. Tenant Create -- /tenants/new
17. Tenant Show -- /tenants/:id
18. Unified Search -- /search
19. Zabbix Overview -- /infrastructure/zabbix
20. OpenNMS Overview -- /infrastructure/opennms

GLOBAL ELEMENTS:
- Header bar: breadcrumbs, global search (Cmd+K), time range picker (Last 15m / 1h / 6h /
  24h / 7d / 30d / custom), tenant selector dropdown, notification bell, user avatar
- Time range picker persists across all telemetry pages (metrics, logs, traces)
- Auto-refresh toggle with interval selector (Off / 10s / 30s / 1m / 5m)
- Data freshness indicator: green dot + "Live" or amber dot + "2m ago"

BREAKPOINTS: 1440px (desktop), 1024px (tablet), 390px (mobile)
THEMES: Light (default) and Dark theme support on all pages.
All designs must comply with WCAG 2.1 AA accessibility standards.
```

---

## 5. Figma Make Design-System Prompt (Component Library)

### Prompt F-001: Core Design Tokens -- Observability Theme

```
Create a Figma page titled "Observability Design Tokens" containing the complete
token specification for the Observability module.

COLOR PALETTE (Light Theme):
- Primary: #059669 (Emerald Green -- represents health and uptime)
- Primary Hover: #047857
- Primary Light: #d1fae5
- Secondary: #8b5cf6 (Accent Purple -- used for traces, distributed systems)
- Success: #10b981 (Healthy, Resolved, Active, OK spans)
- Warning: #f59e0b (Pending, Degraded, Expiring, Warning severity)
- Danger: #ef4444 (Critical, Error, Firing, Fatal, Suspended)
- Info: #3b82f6 (Informational, Debug, Provisioning)
- Fatal: #7c2d12 (Fatal log level, darkest severity)
- Neutral-50: #f5f7fa (Page background)
- Neutral-100: #fafbfc (Table header background)
- Neutral-200: #e5e7eb (Borders, dividers)
- Neutral-500: #64748b (Secondary text, column headers)
- Neutral-900: #111827 (Primary text)
- Surface: #ffffff (Cards, modals, panels)
- Sidebar-Bg: #001529 (Dark navy)
- Sidebar-Active: #059669 (Primary)
- Sidebar-Hover: rgba(5, 150, 105, 0.6)

COLOR PALETTE (Dark Theme):
- Primary: #34d399 (Light Emerald)
- Primary Hover: #6ee7b7
- Background: #0f172a (Slate-900)
- Surface: #1e293b (Slate-800)
- Surface Elevated: #334155 (Slate-700)
- Border: #475569 (Slate-600)
- Text Primary: #f1f5f9 (Slate-100)
- Text Secondary: #94a3b8 (Slate-400)
- Success/Warning/Danger: same hues, lightened 20%

TYPOGRAPHY (Plus Jakarta Sans, JetBrains Mono):
- Display: 30px/36px, semibold 600 -- page titles ("Dashboard", "Metrics Explorer")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Module Health Overview")
- Heading 2: 20px/28px, medium 500 -- card titles ("Recent Alerts")
- Heading 3: 16px/24px, medium 500 -- subsection headers
- Body: 14px/20px, regular 400 -- paragraph text, table cells
- Body Small: 12px/16px, regular 400 -- captions, timestamps, helper text
- Label: 14px/20px, medium 500 -- form labels, column headers
- Mono: 13px/20px, JetBrains Mono -- metric names, PromQL expressions, log messages,
  trace IDs, span IDs, JSON payloads, durations (e.g., "245ms")
- KPI Value: 28px/32px, bold 700 -- statistic card values

SPACING SCALE (4px base):
- 4px (xs), 8px (sm), 12px (md), 16px (lg), 24px (xl), 32px (2xl), 48px (3xl)
- Card padding: 24px
- Section gap: 24px (tighter than HCM for information density)
- Page margin: 32px (desktop), 16px (mobile)

ELEVATION / SHADOWS:
- Level 0: none (flat elements)
- Level 1: 0 1px 3px rgba(0,0,0,0.1) -- cards, dropdowns
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, popovers
- Level 3: 0 8px 24px rgba(0,0,0,0.12) -- dialogs, slide-overs

BORDER RADIUS:
- Small: 6px (tags, status badges)
- Medium: 8px (buttons, inputs, menu items)
- Large: 10px (cards, tables)
- Full: 9999px (status dots, avatars)

GRID:
- Desktop (1440px): 12-column, 72px columns, 24px gutter, 260px sidebar
- Tablet (1024px): 12-column, 56px columns, 16px gutter, collapsed 72px sidebar
- Mobile (390px): 4-column, fluid, 16px gutter, bottom nav

Include token swatches, type scale samples, spacing ruler, and shadow comparison.
Both light and dark themes must be fully specified side-by-side.
```

### Prompt F-002: Component Library -- Observability Components

```
Create a Figma page titled "Observability Component Library" with the following
reusable components, each showing all states (default, hover, active, disabled,
error, loading) and both light/dark theme variants.

KPI CARD COMPONENT:
- Left icon (24px, colored circle background matching category)
- Metric value (28px, bold) with optional unit suffix ("/s", "%", "ms")
- Label (14px, secondary text)
- Trend indicator: up/down arrow + percentage (green for positive, red for negative)
- Loading state: shimmer skeleton
- Variants: 4 across in a row at lg breakpoint, 2x3 at sm, 1-column at xs
- Examples:
  -- LineChartOutlined, "24.8K", "Metric Series", +4.2% (green #059669)
  -- FileTextOutlined, "15.4K/s", "Log Rate", +12.8% (blue #3b82f6)
  -- AlertOutlined, "7", "Active Alerts", -2.1% (red #ef4444)
  -- ApartmentOutlined, "8.3K/s", "Trace Throughput", +6.5% (purple #8b5cf6)
  -- WarningOutlined, "2.4%", "Error Rate", -0.3% (amber #f59e0b)
  -- TeamOutlined, "12", "Active Tenants", +1.0% (teal #0d9488)

STATUS BADGE COMPONENT:
- Small pill badge with dot + text
- Severity variants: info (#3b82f6), warning (#f59e0b), critical (#ef4444)
- Alert status: firing (red pulse), pending (amber), resolved (green)
- Tenant status: active (green), suspended (red), provisioning (blue)
- Span status: ok (green), error (red), unset (gray)
- Log level: debug (slate), info (blue), warn (amber), error (red), fatal (dark red)
- Sizes: small (20px height), medium (24px), large (28px)

METRIC SPARKLINE COMPONENT:
- Inline SVG/Canvas sparkline (100px wide x 24px tall)
- Renders 20-point time series as area chart
- Color follows metric health (green = normal, amber = elevated, red = critical)
- Hover tooltip shows exact value + timestamp
- Used in Module Health table and Metric List rows

LOG LINE COMPONENT:
- Single log entry row with:
  -- Timestamp (12px, mono, secondary)
  -- Level badge (debug/info/warn/error/fatal with colors)
  -- Service name tag (rounded, 6px radius)
  -- Message text (13px, mono, truncated with ellipsis)
  -- Expand chevron to show full metadata JSON
- Expanded state: full message + JSON metadata viewer + trace link
- Highlighted state: search term matches shown with amber background

TRACE WATERFALL BAR:
- Horizontal bar representing span duration relative to trace total
- Nested indentation for parent-child span hierarchy
- Color-coded by service name (auto-assigned from palette)
- Error spans: red bar with error icon
- Hover: tooltip with operation name, service, duration, status
- Click: expand to show span tags and events

ALERT RULE CARD:
- Name (16px, semibold), description (14px, secondary)
- Severity badge (info/warning/critical)
- Module tag (rounded)
- Enabled/disabled toggle switch
- PromQL expression preview (mono, truncated)
- Last fired timestamp
- Quick actions: Edit, Silence, Delete

DASHBOARD PANEL COMPONENT:
- Panel types: graph (line/area/bar chart), stat (single big number), table,
  heatmap (color grid), logs (scrolling log lines)
- Panel header: title, time range override, refresh button, three-dot menu
  (Edit, Duplicate, Delete, View Query)
- Drag handle for repositioning in grid layout
- Resize handles at corners and edges
- Loading state: centered spinner inside panel bounds

TENANT CARD:
- Display name (16px, semibold), slug name (12px, mono, secondary)
- Status badge (active/suspended/provisioning)
- Tier tag (free/standard/enterprise)
- Quota meters: 4 mini progress bars
  -- Metrics: 5,000/10,000 per second
  -- Logs: 2,000/5,000 per second
  -- Traces: 1,000/3,000 per second
  -- Storage: 450GB/1TB
- Created date (12px, secondary)

SEARCH RESULT CARD:
- Result type icon (metric/log/trace) with colored background
- Title/identifier (linked)
- Preview snippet with search term highlighted
- Source metadata: module, service, timestamp
- Type filter tabs above results: All | Metrics | Logs | Traces

TIME RANGE PICKER:
- Dropdown with preset options: Last 15m, 1h, 6h, 24h, 7d, 30d
- Custom range: two date-time pickers (start, end)
- Auto-refresh toggle: Off, 10s, 30s, 1m, 5m
- Compact display in header: "Last 24h" with clock icon

SIDEBAR NAVIGATION:
- Collapsible sidebar (expanded 260px, collapsed 72px)
- Logo area: 32px gradient square + "ERP Observability" title
- Sections: Dashboard, Metrics, Logs, Traces, Alerts, Dashboards, Tenants,
  Search, Infrastructure (Zabbix, OpenNMS submenu)
- Active state: #059669 background + white text + 8px border radius
- Hover state: rgba(5, 150, 105, 0.6)
- Collapsed: icon-only with tooltip
- Dark theme (#001529 background)

DATA TABLE (Observability variant):
- Ant Design-based table with sortable columns, row selection checkboxes,
  inline status badges, row hover actions (eye icon = view)
- Pagination: "Showing 1-25 of 1,247 entries"
- Filter bar above table with type, severity, module, time-range chips
- Empty state: illustration + "No data matches your filters" + clear filters button
- Loading state: skeleton rows
- High-density mode toggle for log/trace tables (reduced row padding)
```

---

## 6. Make Automation Prompt Pack

### Prompt M-001: Alert Escalation And Notification Pipeline

```
Create a Make (Integromat) scenario titled "OBS -- Alert Escalation Pipeline".

TRIGGER: Webhook from alertmanager when an alert transitions to "firing" state.
Payload includes: { alert_rule_id, alert_name, severity, module, tenant_id,
  starts_at, value, labels, annotations, promql_expression }

STEP 1 -- Classify Severity:
- Critical: immediate escalation (page on-call)
- Warning: standard notification (Slack + email)
- Info: log only (in-app notification)

STEP 2 -- Check Silence Rules:
- Call observability-api: GET /v1/silences?matchers={labels}
- If matched by active silence: log silenced event and stop
- If not silenced: continue pipeline

STEP 3 -- Enrich Context:
- Call observability-api: GET /v1/metrics/query?query={promql_expression}&time=now
  (get current metric value and recent trend)
- Call quickwit: search recent error logs from same module/service (last 15 minutes)
- Compile enrichment: metric value, trend direction, correlated log count

STEP 4 -- Notify (Parallel, based on severity):
- Critical:
  -- PagerDuty/Opsgenie: Create incident with full context
  -- Slack #incidents: Post alert card with metric graph thumbnail
  -- Email: on-call engineer + engineering manager
  -- SMS: on-call engineer phone
- Warning:
  -- Slack #alerts-{module}: Post alert summary
  -- Email: team lead for affected module
  -- In-app: notification bell for all tenant Operators
- Info:
  -- In-app notification only
  -- Dashboard KPI update via NATS event

STEP 5 -- Track Acknowledgement:
- Start 15-minute timer for Critical alerts
- If not acknowledged: escalate to skip-level manager
- If not acknowledged in 30 minutes: escalate to VP Engineering
- Log all escalation steps to audit trail

STEP 6 -- Resolution Tracking:
- Listen for "resolved" webhook from alertmanager
- Calculate MTTA (Mean Time to Acknowledge) and MTTR (Mean Time to Resolve)
- Update incident record with resolution metadata
- Post resolution summary to original Slack thread
- Publish NATS event: "alert.resolved" with MTTA/MTTR metrics

ERROR HANDLING:
- Retry up to 3 times with exponential backoff on all API calls
- On final failure: fallback to email-only notification
- All steps logged to compliance audit trail

ESTIMATED RUN TIME: < 10 seconds for notification delivery
TRIGGER VOLUME: ~200/day (varies by environment health)
```

### Prompt M-002: Tenant Provisioning Automation

```
Create a Make scenario titled "OBS -- Tenant Provisioning Pipeline".

TRIGGER: Webhook from observability-api when a new tenant record is created.
Payload: { tenant_id, name, display_name, tier, retention_days,
           quotas: { metrics_per_second, logs_per_second, traces_per_second, storage_gb },
           admin_email, admin_name }

STEP 1 -- Validate Tenant Configuration:
- Check name matches regex: ^[a-z][a-z0-9-]{2,63}$
- Verify quotas are within tier limits (free/standard/enterprise)
- Verify storage_gb does not exceed cluster capacity threshold
- Route: If validation fails -> Step ERROR with details

STEP 2 -- Provision Data Stores (Parallel):
- VictoriaMetrics: Create tenant namespace via vmauth configuration
  POST /v1/victoria-metrics/tenants { tenant_id, retention: retention_days }
- Quickwit: Create log and trace indexes for tenant
  POST /v1/quickwit/indexes { tenant_id, type: "logs" }
  POST /v1/quickwit/indexes { tenant_id, type: "traces" }
- YugabyteDB: Initialize tenant schema and default configurations
- DragonflyDB: Create tenant cache namespace with TTL policies
- RustFS: Create tenant storage bucket with lifecycle policies

STEP 3 -- Configure OTel Collector:
- Update collector pipeline configuration to route tenant data
- Apply rate limiting based on quotas (metrics_per_second, logs_per_second, traces_per_second)
- Deploy configuration via hot-reload endpoint

STEP 4 -- Create Default Resources:
- Create default dashboard: "System Overview" with standard panels
  (request rate, error rate, latency p50/p95/p99, log volume)
- Create default alert rules:
  -- "High Error Rate" (error_rate > 5% for 5m, severity: warning)
  -- "Service Down" (up == 0 for 2m, severity: critical)
  -- "High Latency P99" (latency_p99 > 1s for 10m, severity: warning)
  -- "Disk Usage Critical" (disk_used_pct > 90, severity: critical)

STEP 5 -- Notify:
- Email admin: Welcome email with getting-started guide and API keys
- In-app notification: "Tenant {display_name} is ready"
- Update tenant status: provisioning -> active
- Publish NATS event: "tenant.provisioned"

STEP 6 -- Verify:
- Run health check against all provisioned components
- Verify data ingestion path by sending test metric/log/trace
- If verification fails: mark tenant as "provisioning_failed" and alert ops team

ERROR HANDLING:
- Any step failure triggers rollback of completed steps
- Partial provisioning logged with step-level status
- Alert ops team via Slack #tenant-ops on failure

ESTIMATED RUN TIME: < 60 seconds
TRIGGER VOLUME: ~10/month
```

### Prompt M-003: Automated SLO Compliance Reporting

```
Create a Make scenario titled "OBS -- Weekly SLO Compliance Report".

TRIGGER: Scheduled every Monday at 07:00 UTC.

STEP 1 -- Gather SLO Data Per Module:
- For each ERP module (HCM, CRM, Finance, Procurement, etc.):
  -- Call victoria-metrics: query availability SLI
     avg_over_time(up{module="{mod}"}[7d]) * 100
  -- Call victoria-metrics: query latency SLI
     histogram_quantile(0.99, rate(http_request_duration_seconds_bucket{module="{mod}"}[7d]))
  -- Call victoria-metrics: query error rate SLI
     sum(rate(http_requests_total{module="{mod}",status=~"5.."}[7d])) / sum(rate(http_requests_total{module="{mod}"}[7d])) * 100
  -- Call quickwit: count error/fatal logs for module (last 7 days)

STEP 2 -- Calculate SLO Compliance:
- For each module, compare against targets:
  -- Availability: target >= 99.95%
  -- P99 Latency: target < 200ms (read), < 500ms (write)
  -- Error Rate: target < 0.1%
- Calculate error budget remaining:
  budget_remaining = (1 - (1 - actual_availability) / (1 - target_availability)) * 100
- Flag modules with < 20% error budget remaining as "at risk"

STEP 3 -- Generate Report:
- Create structured report with:
  -- Executive summary (overall platform health score)
  -- Per-module SLO table (module, availability, latency, error rate, budget %)
  -- Trend comparison vs previous week
  -- Top 5 incidents impacting SLOs
  -- Recommendations for at-risk modules
- Generate PDF via document template

STEP 4 -- Distribute:
- Email: VP Engineering, SRE Team Lead, Module Owners
- Slack #slo-reports: Post summary with link to full report
- Store report in RustFS: /reports/slo/{year}/{week}/slo-report.pdf
- Update observability dashboard "SLO Tracker" panel data

STEP 5 -- Alert On Breaches:
- If any module breached SLO target: create incident ticket
- If error budget < 10%: freeze deployments notification to module team
- Publish NATS event: "slo.weekly_report" with compliance summary

ESTIMATED RUN TIME: < 120 seconds
```

### Prompt M-004: Log Retention And Archival Pipeline

```
Create a Make scenario titled "OBS -- Log Retention And Archival".

TRIGGER: Scheduled daily at 02:00 UTC (low-traffic window).

STEP 1 -- Assess Storage Per Tenant:
- For each active tenant:
  -- Query quickwit: GET /v1/indexes/{tenant_id}-logs/stats (current index size)
  -- Query rustfs: GET /v1/buckets/{tenant_id}/usage (archived storage)
  -- Compare against tenant quota (storage_gb from obs_tenants)
  -- Calculate days until quota exhaustion at current ingestion rate

STEP 2 -- Apply Retention Policies:
- For each tenant:
  -- Delete log entries older than tenant.retention_days from quickwit
  -- Delete trace spans older than tenant.retention_days from quickwit
  -- Archive (not delete) data between retention_days and 2x retention_days to RustFS
     (compressed Parquet format for cost-efficient cold storage)

STEP 3 -- Quota Enforcement:
- If tenant at > 90% storage quota:
  -- In-app warning notification to tenant admins
  -- Rate-limit ingestion to 50% of quota (graceful degradation)
- If tenant at > 100% storage quota:
  -- Email alert to tenant admin with upgrade options
  -- Apply strict rate limiting (10% of quota, critical data only)
  -- Create support ticket for account team

STEP 4 -- Optimize:
- Run quickwit index compaction for tenants with high fragmentation
- Update DragonflyDB cache: evict stale query results
- Verify RustFS archive integrity (checksum verification on recent archives)

STEP 5 -- Report:
- Generate daily storage utilization report
- Update tenant dashboard "Storage Usage" panels
- Publish NATS event: "storage.daily_maintenance" with summary

ERROR HANDLING:
- Retention deletion failures: retry next day, alert if persistent
- Archive failures: retain data in hot storage until successful archive
- Never delete data without successful archive confirmation

ESTIMATED RUN TIME: < 300 seconds per tenant
TRIGGER VOLUME: 1/day
```

### Prompt M-005: Infrastructure Health Correlation

```
Create a Make scenario titled "OBS -- Infrastructure Health Correlation".

TRIGGER: Webhook from Zabbix proxy on host problem event OR
         Webhook from OpenNMS on alarm event.

STEP 1 -- Normalize Event:
- Map Zabbix problem or OpenNMS alarm to unified schema:
  { source, host, severity, description, timestamp, category }
- Categories: hardware, network, disk, cpu, memory, process, service

STEP 2 -- Correlate With Application Metrics:
- Identify ERP modules running on affected host
- Query victoria-metrics: recent metric anomalies for those modules
- Query quickwit: error log spike correlation (last 30 minutes)
- Query trace data: latency increase for services on affected host

STEP 3 -- Impact Assessment:
- Calculate blast radius: number of affected services, tenants, users
- Determine if any SLOs are at risk based on current error budget
- Check if redundancy is available (other healthy replicas)

STEP 4 -- Automated Response:
- If redundancy available: no immediate action, monitor
- If single point of failure:
  -- Critical: page on-call, create incident
  -- Warning: notify infrastructure team via Slack
- If self-healing action available (e.g., restart service):
  -- Execute with confirmation gate (AIDD supervised action)

STEP 5 -- Track And Report:
- Create correlated incident record linking infra event to app impact
- Update infrastructure dashboard with event timeline
- Publish NATS event: "infrastructure.correlated_event"
- Post-resolution: generate RCA (Root Cause Analysis) template

ERROR HANDLING:
- Deduplication: ignore duplicate events within 5-minute window
- Rate limiting: max 10 notifications per host per hour
- Fallback: if correlation API unavailable, forward raw event to Slack

ESTIMATED RUN TIME: < 15 seconds
TRIGGER VOLUME: ~50/day (varies by infrastructure size)
```

---

## 7. Page-by-Page Figma Prompts

### 7.1 Desktop Pages (1440px)

#### Prompt F-003: Observability Dashboard -- Command Center (1440px)

```
Design a desktop Observability Dashboard page at 1440x900px (scrollable) for the
ERP-Observability module. This is the primary landing page for Platform Engineers
and Engineering Managers.

LAYOUT:
- Left: Collapsible sidebar navigation (260px, expanded by default)
- Top: Horizontal header bar with breadcrumb ("Dashboard"), global search (Cmd+K),
  time range picker (default "Last 24h"), auto-refresh toggle ("Live" with green dot),
  tenant selector dropdown, notification bell (3 unread), user avatar "Kemi Adeyemi"
  (Platform Engineer)
- Main content area: 1156px wide, 32px padding

CONTENT -- ROW 1 (KPI Cards, 6-column grid):
- Card 1: "Metric Series" -- 24.8K -- +4.2% this week (green arrow up) -- #059669 icon
- Card 2: "Log Rate" -- 15.4K/s -- +12.8% (blue #3b82f6 icon)
- Card 3: "Active Alerts" -- 7 -- -2.1% (red #ef4444 icon)
- Card 4: "Trace Throughput" -- 8.3K/s -- +6.5% (purple #8b5cf6 icon)
- Card 5: "Error Rate" -- 2.4% -- -0.3% (amber #f59e0b icon)
- Card 6: "Active Tenants" -- 12 -- +1.0% (teal #0d9488 icon)

CONTENT -- ROW 2 (Two-column, 66/34 split):
- Left (66%): "Module Health Overview" card with table:
  | Module | Status | Uptime | Error Rate | P99 Latency | Trend |
  |--------|--------|--------|------------|-------------|-------|
  | HCM | Healthy (green dot) | 99.72% (progress bar) | 1.24% (green) | 87ms | sparkline |
  | CRM | Healthy (green dot) | 99.95% | 0.52% (green) | 62ms | sparkline |
  | Finance | Degraded (amber dot) | 97.81% (amber bar) | 3.89% (amber) | 234ms | sparkline |
  | Procurement | Healthy (green dot) | 99.88% | 0.91% (green) | 95ms | sparkline |
  | Inventory | Healthy (green dot) | 99.94% | 0.33% (green) | 71ms | sparkline |
  | Manufacturing | Healthy (green dot) | 99.67% | 1.56% (green) | 112ms | sparkline |
  | Sales | Degraded (amber dot) | 98.12% (amber bar) | 4.21% (red) | 189ms | sparkline |
  | Compliance | Healthy (green dot) | 99.99% | 0.11% (green) | 45ms | sparkline |
  Each row: Module name (capitalized, 500 weight), status (icon + text), uptime (progress
  bar colored by threshold), error rate (colored by threshold), P99 latency (text),
  trend sparkline (100x24px)

- Right (34%): "Recent Alerts" card:
  -- Header: AlertOutlined icon + "Recent Alerts" + "View All" link with arrow
  -- List of 5 most recent alert rules:
     Each item: name (linked, 500 weight, 13px), severity badge + module tag,
     enabled/disabled status badge, border-bottom divider
  -- Examples:
     "High Error Rate - Finance" | critical | finance | enabled
     "Latency Spike - Sales API" | warning | sales | enabled
     "Memory Usage > 85%" | warning | infra | enabled
     "Service Restart Loop" | critical | hcm | enabled
     "Log Volume Anomaly" | info | platform | disabled
  -- Empty state: green check icon + "No active alerts"

STATES:
- Loading: Skeleton shimmer on all cards, sparkline placeholders, table row skeletons
- Empty: "Welcome to Observability! Configure your first data source." with CTA button
- Error: Inline error card with retry button for failed data fetches
- Live mode: green pulsing dot in header, auto-updating KPI values with fade transition

Include both light theme (default) and dark theme toggle in the header.
Annotate all interactive elements with click targets >= 44px.
```

#### Prompt F-004: Metrics List And Metric Show (1440px)

```
Design two connected desktop screens at 1440x900px for the observability metrics views.

SCREEN A -- METRICS LIST (/metrics):
Layout: Sidebar (260px) + main area.
- Top bar: "Metrics" title, time range picker, auto-refresh toggle
- Search bar: "Search metric names..." with real-time autocomplete
- Filter row: Module dropdown, Type chips (counter | gauge | histogram | summary),
  Tenant dropdown (admin only)
- Active filters shown as dismissible chips

METRICS TABLE:
| Name | Type | Module | Labels | Current Value | Trend | Actions |
|------|------|--------|--------|---------------|-------|---------|
| http_requests_total | counter | HCM | method=GET, path=/api/employees | 1,245,672 | sparkline | View |
| http_request_duration_seconds | histogram | CRM | method=POST, path=/api/contacts | p50=45ms p99=234ms | sparkline | View |
| go_goroutines | gauge | Finance | instance=finance-api-1 | 847 | sparkline | View |
| otel_collector_receiver_accepted_metric_points | counter | Platform | receiver=otlp | 15,420/s | sparkline | View |
| process_resident_memory_bytes | gauge | HCM | instance=hcm-api-1 | 512MB | sparkline | View |

- Type column uses colored chips: counter=#3b82f6, gauge=#059669, histogram=#8b5cf6, summary=#f59e0b
- Labels shown as truncated key=value pairs with "+3 more" expand
- Sparkline: 100x24px inline chart (last 1 hour)
- Pagination: "Showing 1-50 of 24,750 metric series"
- "Export CSV" button in toolbar

SCREEN B -- METRIC SHOW (/metrics/:id):
Layout: Back breadcrumb "Metrics > http_requests_total", full-width content.

HEADER SECTION:
- Metric name: "http_requests_total" (mono, 24px)
- Type badge: "counter" (blue chip)
- Module badge: "HCM"
- Description: "Total number of HTTP requests processed"
- Labels list: all key=value pairs as tags

CHART SECTION (full width, 400px tall):
- Time-series line chart with:
  -- X-axis: timestamps based on selected time range
  -- Y-axis: metric value (auto-scaled)
  -- Multiple series if label cardinality > 1 (e.g., by status_code)
  -- Legend below chart: clickable to toggle series visibility
  -- Crosshair on hover showing exact values for all series
- Chart controls: Zoom (scroll), Pan (drag), Reset zoom, Download PNG
- PromQL editor below chart: editable expression with syntax highlighting
  Default: rate(http_requests_total{module="hcm"}[5m])
- "Run Query" button, "Add to Dashboard" button

METADATA SECTION (two-column):
- Left: "Label Analysis" -- table of all label keys with cardinality counts
  | Label | Values | Example |
  | method | 4 | GET, POST, PUT, DELETE |
  | path | 28 | /api/employees, /api/payroll |
  | status_code | 5 | 200, 201, 400, 404, 500 |

- Right: "Related Alerts" -- list of alert rules referencing this metric
  -- "High Error Rate" (critical, firing)
  -- "Request Rate Anomaly" (warning, resolved)

Both screens support light and dark themes. PromQL editor uses dark background
even in light theme for readability.
```

#### Prompt F-005: Logs List And Log Show (1440px)

```
Design two connected desktop screens at 1440x900px for the observability log views.
High-density information layout optimized for debugging workflows.

SCREEN A -- LOG LIST (/logs):
Layout: Sidebar (260px) + main area.
- Top bar: "Logs" title, time range picker, auto-refresh toggle, "Tail" live button
  (green pulsing when active)
- Search bar: full-text search with Quickwit query syntax hints
  Placeholder: "Search log messages... (supports field:value syntax)"
- Filter row: Level chips (debug | info | warn | error | fatal), Module dropdown,
  Service dropdown, Trace ID input field
- Volume histogram: thin bar chart (60px tall, full width) showing log volume over
  time range, colored by level distribution (stacked bars)

LOG TABLE (high-density mode):
Each row is a log entry, monospace font, reduced padding (8px vertical):
| Timestamp | Level | Service | Message |
|-----------|-------|---------|---------|
| 2026-02-24 14:32:15.847 | ERROR | hcm-api | Failed to process payroll batch: connection refused to yugabytedb:5433 |
| 2026-02-24 14:32:15.234 | WARN | finance-api | Slow query detected: SELECT * FROM transactions WHERE... (2.3s) |
| 2026-02-24 14:32:14.991 | INFO | otel-collector | Batch export completed: 15,420 metric points, 3,210 log records |
| 2026-02-24 14:32:14.567 | DEBUG | crm-api | Cache hit for contact lookup: key=contact:uuid:abc123 |
| 2026-02-24 14:32:14.123 | FATAL | inventory-api | Out of memory: heap allocation failed, shutting down |

- Timestamp: mono 12px, gray
- Level: colored badge (debug=slate, info=blue, warn=amber, error=red, fatal=dark-red)
- Service: tag with 6px radius
- Message: mono 13px, truncated at single line, full text on expand
- Row click expands to show full entry
- Shift+click selects multiple rows for comparison
- Keyboard: J/K to navigate rows, Enter to expand, Escape to collapse
- Infinite scroll with virtualization (render only visible rows)
- "Export Selection" button for selected rows

LIVE TAIL MODE:
- When "Tail" is active: new logs stream in at top, auto-scroll
- Pause button to freeze stream
- Highlight: new entries flash briefly with green-tinted background
- Rate indicator: "~15,420 logs/sec" displayed in toolbar

SCREEN B -- LOG SHOW (/logs/:id):
Layout: Slide-in panel from right (600px) or full page via "Open in new tab".

HEADER:
- Timestamp: "2026-02-24 14:32:15.847 UTC" (mono, 16px)
- Level: "ERROR" (large badge)
- Service: "hcm-api" | Module: "HCM"

MESSAGE SECTION:
- Full log message displayed in mono font, word-wrapped
- Syntax highlighting for JSON content detected in message
- Copy button for full message

CONTEXT SECTION:
- "Show Context" button: loads 10 logs before and after this entry (timeline view)
- Surrounding logs displayed with current entry highlighted

METADATA (JSON viewer):
- Expandable JSON tree of all metadata fields
- Copy individual values on click
- Fields: trace_id (linked to trace view), span_id, host, container, pod, namespace

TRACE LINK:
- If trace_id exists: "View Trace" button linking to /traces/{trace_id}
- Mini trace waterfall preview (3-4 spans visible)

Dark theme strongly recommended for log views (easier on eyes during incident response).
```

#### Prompt F-006: Traces List And Trace Show (1440px)

```
Design two connected desktop screens at 1440x900px for the distributed tracing views.

SCREEN A -- TRACE LIST (/traces):
Layout: Sidebar (260px) + main area.
- Top bar: "Traces" title, time range picker, auto-refresh toggle
- Search bar: "Search by trace ID, operation, or service..."
- Filter row: Service dropdown (multi-select), Operation dropdown,
  Status chips (ok | error | unset), Duration range (min/max in ms),
  Module dropdown

TRACE TABLE:
| Trace ID | Root Operation | Services | Spans | Duration | Status | Timestamp |
|----------|---------------|----------|-------|----------|--------|-----------|
| abc12345...ef | GET /api/employees | hcm-api, yugabytedb, dragonflydb | 12 | 245ms | OK (green) | 14:32:15 |
| def67890...ab | POST /api/contacts | crm-api, yugabytedb, otel-collector | 8 | 1,234ms | Error (red) | 14:32:10 |
| ghi24680...cd | ProcessPayroll | hcm-api, payroll-worker, yugabytedb, rustfs | 34 | 15,672ms | OK (green) | 14:31:55 |

- Trace ID: mono font, truncated to first 8 + last 2 chars, full on hover
- Services: colored chips for each service involved (auto-assigned colors)
- Duration: highlighted if > 1s (amber) or > 5s (red)
- Status: dot + text
- Mini waterfall preview on hover (showing top 3 spans as horizontal bars)
- Pagination: "Showing 1-25 of 8,340 traces"

SCATTER PLOT (above table, 200px tall):
- X-axis: time, Y-axis: duration (log scale)
- Each dot = one trace, colored by status (green=ok, red=error)
- Click dot to jump to trace detail
- Brush selection to filter table to time range

SCREEN B -- TRACE SHOW (/traces/:id):
Layout: Full-width page with breadcrumb "Traces > abc12345...ef".

HEADER:
- Trace ID: "abc12345...ef" (mono, full ID, copy button)
- Root operation: "GET /api/employees"
- Total duration: "245ms"
- Span count: "12 spans across 3 services"
- Status: "OK" (green badge)
- Start time: "2026-02-24 14:32:15.847 UTC"

WATERFALL DIAGRAM (full width, scrollable vertically):
- Horizontal timeline with microsecond precision
- Each row is a span:
  -- Left label: service name (colored) + operation name
  -- Right: horizontal bar showing relative start time and duration
  -- Nested spans indented under parent (tree structure)
  -- Error spans: red bar with exclamation icon
  -- Selected span: highlighted row with expanded detail below

  Example waterfall:
  hcm-api          GET /api/employees           |===========================| 245ms
    hcm-api        auth.validateToken           |==|                          12ms
    hcm-api        db.queryEmployees              |================|          156ms
      yugabytedb   SELECT employees                |============|            134ms
        dragonflydb CACHE_MISS employee:list        |=|                       3ms
        yugabytedb  pg.execute                       |==========|            128ms
      yugabytedb   SELECT departments                  |===|                 18ms
    hcm-api        json.marshal                                    |==|      8ms
    hcm-api        http.writeResponse                                |=|    4ms

SPAN DETAIL PANEL (bottom section, appears on span click):
- Operation: "db.queryEmployees"
- Service: "hcm-api"
- Duration: "156ms"
- Status: "OK"
- Tags table:
  | Key | Value |
  | db.system | yugabytedb |
  | db.statement | SELECT * FROM employees WHERE tenant_id = $1 LIMIT 25 |
  | db.row_count | 25 |
  | net.peer.name | yugabytedb |
  | net.peer.port | 5433 |
- Events timeline:
  -- "query.start" at +0ms
  -- "cache.miss" at +3ms { key: "employee:list" }
  -- "query.complete" at +156ms { rows: 25 }
- Linked logs: button to view logs with this span_id

SERVICE MAP (toggle view):
- Directed graph showing service-to-service call flow for this trace
- Nodes: service names, sized by span count
- Edges: arrows showing call direction, labeled with call count and avg duration
- Error paths highlighted in red

Both screens support light and dark themes. Waterfall uses dark theme by default.
```

#### Prompt F-007: Alert List, Create, And Show (1440px)

```
Design three connected desktop screens at 1440x900px for alert management.

SCREEN A -- ALERT LIST (/alerts):
Layout: Sidebar (260px) + main area.
- Top bar: "Alerts" title, "Create Alert Rule" primary button (#059669)
- Tab navigation: "Rules" (default) | "Firing" | "Silences" | "Notifications"

RULES TAB:
- Filter row: Severity chips (info | warning | critical), Module dropdown,
  Status toggle (enabled/disabled/all)
- Search bar: "Search alert rules..."

ALERT RULES TABLE:
| Name | Severity | Module | Expression | Duration | Enabled | Firing | Actions |
|------|----------|--------|------------|----------|---------|--------|---------|
| High Error Rate | critical (red badge) | Finance | rate(http_errors[5m]) > 0.05 | 5m | toggle ON | 1 instance (red dot) | Edit, Silence, Delete |
| Latency Spike | warning (amber badge) | Sales | histogram_quantile(0.99,...) > 1 | 10m | toggle ON | 0 | Edit, Silence, Delete |
| Service Down | critical (red badge) | All | up == 0 | 2m | toggle ON | 0 | Edit, Silence, Delete |
| Low Disk Space | warning (amber badge) | Infra | disk_used_pct > 85 | 15m | toggle OFF (gray) | -- | Edit, Enable, Delete |
| Memory Usage | info (blue badge) | Platform | process_memory_bytes > 1GB | 30m | toggle ON | 0 | Edit, Silence, Delete |

- "Firing" column shows active instances count with red dot if > 0
- Toggle switch inline for enable/disable (optimistic UI with undo toast)
- Delete requires confirmation dialog: "Type DELETE to confirm removal of this alert rule"
- Bulk actions: select multiple -> "Disable Selected", "Delete Selected"

FIRING TAB:
- Active alert instances grouped by alert rule
- Each instance card: labels, value, started_at, duration firing
- "Acknowledge" and "Silence" quick actions
- Timeline showing when alert started and severity transitions

SILENCES TAB:
- Active and expired silences table
- "Create Silence" button
- Each silence: matchers, creator, start time, end time, comment
- "Expire Now" button for active silences

SCREEN B -- ALERT CREATE (/alerts/new):
Layout: Centered form, max 800px wide.

FORM SECTIONS:
1. "Alert Rule Configuration" header
2. Name input: "High Error Rate - Finance Module"
3. Description textarea: "Fires when the HTTP 5xx error rate exceeds 5% for 5 minutes"
4. Severity radio group: info | warning | critical (with color indicators)
5. Module dropdown: select target module
6. PromQL Expression:
   - Syntax-highlighted code editor (dark background, mono font)
   - "Validate" button to test expression against VictoriaMetrics
   - Result preview: current value + mini chart
   - Example: rate(http_requests_total{module="finance",status=~"5.."}[5m]) / rate(http_requests_total{module="finance"}[5m]) > 0.05
7. Duration input: "5m" (for how long condition must be true)
8. Labels: key-value pair inputs (add/remove rows)
   - team: finance-sre
   - environment: production
9. Annotations: key-value pairs
   - summary: "Error rate is {{ $value | humanizePercentage }} for Finance module"
   - runbook_url: "https://wiki.internal/runbooks/finance-error-rate"
10. Notification channels: multi-select (Slack #finance-alerts, Email: finance-team@, PagerDuty)

ACTIONS:
- "Test Rule" secondary button (dry run against last 24h of data)
- "Save Rule" primary button (#059669)
- "Cancel" link

SCREEN C -- ALERT SHOW (/alerts/:id):
Layout: Breadcrumb "Alerts > High Error Rate - Finance Module", full width.

HEADER:
- Alert name (24px, semibold), severity badge, module tag, enabled toggle
- "Edit" and "Silence" buttons in header

SECTIONS:
1. "Current Status" card:
   - Status: "Firing" (red pulse badge) or "OK" (green)
   - Current value: "7.2%" (large)
   - Threshold: "> 5.0%"
   - Firing since: "2026-02-24 13:45:00 UTC (47 minutes ago)"
   - PromQL expression (mono, syntax highlighted)

2. "Alert History" timeline (last 24 hours):
   - Visual timeline bar showing firing (red), pending (amber), OK (green) periods
   - List of state transitions with timestamps

3. "Active Instances" table:
   | Labels | Value | Started | Duration | Actions |
   | method=POST, path=/api/transactions | 8.1% | 13:45 | 47m | Acknowledge, Silence |

4. "Notification History":
   - Sent notifications with channel, timestamp, delivery status
   - "Slack #finance-alerts -- Delivered -- 13:46 UTC"
   - "Email finance-team@ -- Delivered -- 13:46 UTC"

All screens support light and dark themes.
```

#### Prompt F-008: Dashboard List, Create, And Show (1440px)

```
Design three connected desktop screens at 1440x900px for custom dashboards.

SCREEN A -- DASHBOARD LIST (/dashboards):
Layout: Sidebar (260px) + main area.
- Top bar: "Dashboards" title, "Create Dashboard" primary button
- Search bar: "Search dashboards..."
- Filter row: Tags (multi-select), Created By, Sort (name/created/modified)
- View toggle: Grid (default) | List

GRID VIEW (3 columns of cards):
Dashboard Card (350px wide):
- Thumbnail preview (16:9 aspect, placeholder with panel layout sketch)
- Title: "System Overview" (16px, semibold)
- Description: "Key performance metrics across all ERP modules" (14px, secondary, 2-line clamp)
- Tags: "production", "overview", "sre" (small chips)
- Metadata: "Created by Kemi Adeyemi -- Updated 2h ago"
- Actions on hover: View, Edit, Duplicate, Share, Delete
- Star/favorite toggle (top-right corner)

Example cards:
1. "System Overview" -- "Key performance metrics across all ERP modules" -- tags: production, overview
2. "HCM Module Deep Dive" -- "Detailed HCM service health and performance" -- tags: hcm, team
3. "Alert Status Board" -- "Real-time alert status across all modules" -- tags: alerts, oncall
4. "Capacity Planning" -- "Infrastructure capacity trends and forecasts" -- tags: infra, planning
5. "SLO Compliance" -- "Weekly SLO tracking for all modules" -- tags: slo, compliance
6. "Incident Response" -- "Live incident dashboard for war rooms" -- tags: incidents, oncall

LIST VIEW: Table with Name, Description, Tags, Panels, Created By, Updated, Actions

SCREEN B -- DASHBOARD CREATE (/dashboards/new):
Layout: Full-width builder interface.

TOOLBAR (top, 56px):
- Dashboard name (editable inline): "Untitled Dashboard"
- Time range picker | Auto-refresh toggle
- "Add Panel" dropdown: Graph, Stat, Table, Heatmap, Logs
- "Save" primary button | "Discard" | Settings gear

CANVAS (grid-based drag-and-drop):
- 24-column grid (snap-to-grid)
- Panels can be resized and repositioned
- Panel placeholder: dashed border + "Click to configure" or drag from toolbar
- Drag handle at panel header for repositioning

PANEL EDITOR (right sidebar, 400px, opens on panel click):
- Panel Title input
- Panel Type selector (graph/stat/table/heatmap/logs)
- Query Editor:
  -- Data source: VictoriaMetrics (default) | Quickwit
  -- PromQL/LogQL editor with syntax highlighting and autocomplete
  -- "Run Query" button with preview
  -- Multiple queries (A, B, C) with color assignment
- Visualization options (based on type):
  -- Graph: line/area/bar, stacking, fill opacity, legend position
  -- Stat: value reduction (last/avg/max/min), thresholds, color mode
  -- Table: column configuration, sort, pagination
  -- Heatmap: color scheme, bucket size, legend
  -- Logs: deduplication, sort order, wrap lines
- Thresholds: add color thresholds (green < 80%, amber < 95%, red >= 95%)

SETTINGS MODAL (gear icon):
- Dashboard description textarea
- Tags input (chips with autocomplete)
- Sharing: Private | Team | Public
  -- Public: generates embed URL with scoped token
  -- Embed code snippet (copy to clipboard)
- Variables: define template variables ($module, $service, $environment)
  -- Variable type: query (from label values), custom (CSV), interval

SCREEN C -- DASHBOARD SHOW (/dashboards/:id):
Layout: Full-width, minimal chrome (hide sidebar for maximum panel space).

HEADER (compact, 48px):
- Dashboard name, time range picker, auto-refresh, "Edit" button, share button,
  fullscreen toggle (F11), "Back to list" link

PANELS (example layout for "System Overview"):
Row 1 (stat panels, 4 across):
- Request Rate: 45.2K/s (green) | Error Rate: 2.4% (amber) | P99 Latency: 234ms | Active Users: 1,247

Row 2 (two graphs, 50/50):
- Left: "Request Rate by Module" -- multi-line time series chart
- Right: "Error Rate by Module" -- stacked area chart

Row 3 (full width):
- "Latency Distribution" -- heatmap showing latency buckets over time

Row 4 (two panels, 60/40):
- Left: "Top Endpoints by Latency" -- table with endpoint, p50, p95, p99, req/s
- Right: "Recent Error Logs" -- live-scrolling log panel with level badges

Auto-refresh: panels update independently, stagger refreshes to avoid thundering herd.
Panel loading: individual skeleton per panel, not full-page.
Error: per-panel error state with "Retry" button (does not affect other panels).
```

#### Prompt F-009: Tenant List, Create, And Show (1440px)

```
Design three connected desktop screens at 1440x900px for multi-tenant management.
Target user: Tenant Administrator / Platform Admin.

SCREEN A -- TENANT LIST (/tenants):
Layout: Sidebar (260px) + main area.
- Top bar: "Tenants" title, "Provision Tenant" primary button
- Filter row: Status chips (active | suspended | provisioning), Tier dropdown, Search

TENANT TABLE:
| Name | Display Name | Status | Tier | Metrics/s | Logs/s | Traces/s | Storage | Created | Actions |
|------|-------------|--------|------|-----------|--------|----------|---------|---------|---------|
| acme-corp | Acme Corporation | Active (green) | Enterprise | 8,420/10K (84%) | 3,210/5K (64%) | 1,890/3K (63%) | 672GB/1TB (67%) | 2025-06-15 | View, Edit, Suspend |
| startup-xyz | Startup XYZ | Active (green) | Standard | 2,100/5K (42%) | 890/2K (45%) | 450/1K (45%) | 89GB/500GB (18%) | 2025-09-20 | View, Edit, Suspend |
| test-tenant | Test Environment | Provisioning (blue) | Free | --/1K | --/500 | --/200 | 0/100GB | 2026-02-24 | View |
| legacy-co | Legacy Corp | Suspended (red) | Standard | 0/5K | 0/2K | 0/1K | 234GB/500GB | 2025-01-10 | View, Reactivate, Delete |

- Quota columns show usage/limit with inline progress bar
- Progress bar colors: green < 70%, amber < 90%, red >= 90%
- "Suspend" requires confirmation dialog
- "Delete" requires typing tenant name to confirm (destructive action)

SUMMARY CARDS (above table):
- Total Tenants: 15 | Active: 12 | Suspended: 2 | Provisioning: 1
- Total Metrics Ingestion: 45.2K/s | Total Logs: 18.7K/s | Total Storage: 3.2TB/8TB

SCREEN B -- TENANT CREATE (/tenants/new):
Layout: Centered form, max 720px wide.

FORM SECTIONS:
1. "Provision New Tenant" header
2. Tenant Name (slug): auto-generated from display name, editable
   Validation: ^[a-z][a-z0-9-]{2,63}$
   Helper text: "Used as the tenant identifier in API headers"
3. Display Name: "Acme Corporation"
4. Tier radio group: Free | Standard | Enterprise
   Each tier shows included quotas:
   - Free: 1K metrics/s, 500 logs/s, 200 traces/s, 100GB storage
   - Standard: 5K metrics/s, 2K logs/s, 1K traces/s, 500GB storage
   - Enterprise: 10K metrics/s, 5K logs/s, 3K traces/s, 1TB storage
5. Custom Quotas (expandable, overrides tier defaults):
   - Metrics per second: number input
   - Logs per second: number input
   - Traces per second: number input
   - Storage GB: number input
6. Retention Days: slider (7-365 days), default by tier (Free=7, Standard=30, Enterprise=90)
7. Admin Contact:
   - Name input
   - Email input (for provisioning notification)
8. Data Isolation Mode: Shared (default) | Dedicated (enterprise only)

ACTIONS:
- "Provision Tenant" primary button
- "Cancel" link
- Progress stepper after submit: Validating -> Creating Namespaces -> Configuring Collector -> Setting Up Defaults -> Verifying -> Complete

SCREEN C -- TENANT SHOW (/tenants/:id):
Layout: Breadcrumb "Tenants > Acme Corporation", full width.

HEADER:
- Display name (24px), tenant slug (mono, secondary), status badge, tier tag
- "Edit", "Suspend", "Delete" action buttons

SECTIONS:
1. "Quota Usage" (4 gauge cards):
   - Metrics: 8,420/10,000 per second (84%) -- circular gauge
   - Logs: 3,210/5,000 per second (64%)
   - Traces: 1,890/3,000 per second (63%)
   - Storage: 672GB/1TB (67%)
   Each gauge: colored by threshold, trend arrow, current vs limit text

2. "Ingestion Trends" (time series chart, full width):
   - Three lines: metrics/s, logs/s, traces/s over last 7 days
   - Quota limit shown as horizontal dashed line per series

3. "Configuration" card:
   - Retention: 90 days
   - Data isolation: Shared
   - OTel endpoint: otel-collector:4317 (with copy button)
   - Required headers: X-Scope-OrgID: acme-corp, X-Tenant-ID: acme-corp-uuid

4. "Default Alert Rules" table:
   - Pre-provisioned rules with enable/disable toggles
   - "Add Rule" button

5. "Audit Log" (recent config changes):
   - "Quota updated: storage 500GB -> 1TB" -- Admin -- 2 days ago
   - "Tenant provisioned" -- System -- 2025-06-15

All screens support light and dark themes.
```

#### Prompt F-010: Unified Search Page (1440px)

```
Design a desktop Unified Search page at 1440x900px for cross-module search.

LAYOUT:
- Sidebar (260px) + main area.
- Full-width search experience, modeled after a search engine interface.

SEARCH HEADER (prominent, centered):
- Large search input (640px wide, centered, 48px tall)
  Placeholder: "Search across metrics, logs, and traces..."
- Below input: type filter tabs -- "All" | "Metrics" | "Logs" | "Traces"
- Advanced filters toggle (expands additional filters):
  -- Time range picker
  -- Module dropdown
  -- Service dropdown
  -- Tenant dropdown (admin only)

SEARCH RESULTS (below, full width):
Result count: "2,847 results for 'connection timeout' across all signal types"

RESULT CARDS (mixed type, chronologically sorted):

Log Result:
- Type icon: FileTextOutlined in blue circle
- Headline: "hcm-api / ERROR / 2026-02-24 14:32:15"
- Snippet: "Failed to process payroll batch: **connection timeout** to yugabytedb:5433 after 30s"
  (search term bold/highlighted in amber)
- Tags: module=HCM, service=hcm-api, level=error
- Action: "View Log" link

Trace Result:
- Type icon: ApartmentOutlined in purple circle
- Headline: "POST /api/transactions / 1,234ms / ERROR"
- Snippet: "Span 'db.query' failed: **connection timeout** to database pool"
- Tags: module=Finance, services=finance-api,yugabytedb
- Action: "View Trace" link

Metric Result:
- Type icon: LineChartOutlined in green circle
- Headline: "db_connection_pool_timeouts_total"
- Snippet: "Counter tracking database **connection timeout** events"
- Tags: module=Platform, type=counter
- Action: "View Metric" link

SEARCH RESULTS TABLE VIEW (toggle from card view):
| Type | Source | Timestamp | Content Preview | Module | Actions |
|------|--------|-----------|-----------------|--------|---------|
| Log | hcm-api | 14:32:15 | Failed to process... connection timeout... | HCM | View |
| Trace | finance-api | 14:31:55 | db.query span error: connection timeout | Finance | View |

RESULT AGGREGATION SIDEBAR (right, 280px):
- "Results by Type": Logs: 2,134 | Traces: 587 | Metrics: 126
- "Results by Module": HCM: 890 | Finance: 723 | CRM: 412 | ...
- "Results by Severity/Level": error: 1,456 | warn: 892 | info: 499
- Click any aggregation to filter

EMPTY STATE: "No results found for '...' -- Try broadening your time range or adjusting filters"
LOADING STATE: Skeleton cards with shimmer animation

Keyboard shortcuts: / to focus search, Tab to switch type tabs, Enter to search.
```

#### Prompt F-011: Zabbix Overview (1440px)

```
Design a desktop Zabbix Infrastructure Overview page at 1440x900px.
Target user: Infrastructure Engineer.

LAYOUT:
- Sidebar (260px) + main area.
- Breadcrumb: "Infrastructure > Zabbix"

CONTENT -- ROW 1 (KPI Cards, 4-column):
- Card 1: "Monitored Hosts" -- 247 -- 3 new this week
- Card 2: "Active Problems" -- 12 -- 4 critical, 5 high, 3 average
- Card 3: "Avg Availability" -- 99.87% -- last 30 days
- Card 4: "Triggers Firing" -- 8 -- 2 more than yesterday

CONTENT -- ROW 2 (Two-column, 60/40):
- Left (60%): "Active Problems" table:
  | Severity | Host | Problem | Duration | Acknowledged | Actions |
  |----------|------|---------|----------|-------------|---------|
  | Critical (red) | db-primary-01 | Disk space < 5% on /data | 2h 15m | No | Ack, View |
  | Critical (red) | app-server-03 | Service 'hcm-api' not running | 45m | Yes (by Kemi) | View |
  | High (orange) | lb-prod-01 | High CPU utilization (>90%) | 1h 30m | No | Ack, View |
  | High (orange) | cache-01 | DragonflyDB memory usage > 85% | 3h | No | Ack, View |
  | Average (amber) | monitor-01 | System time drift > 1s | 12h | No | Ack, View |

  Severity colors: Critical=#ef4444, High=#f97316, Average=#f59e0b, Warning=#facc15, Info=#3b82f6
  "Acknowledged" column: red X or green check with acknowledger name

- Right (40%): "Host Group Health" summary:
  - Production Servers: 120 hosts, 3 problems (amber bar)
  - Database Servers: 15 hosts, 2 problems (red bar)
  - Network Devices: 45 hosts, 0 problems (green bar)
  - Load Balancers: 8 hosts, 1 problem (amber bar)
  - Monitoring: 4 hosts, 1 problem (amber bar)
  Each row: group name, host count, problem count, health bar

CONTENT -- ROW 3 (Full width):
- "Host Map" -- visual grid of all hosts as colored squares:
  -- Green = OK, Amber = warning, Red = problem
  -- Sized by importance (database servers larger)
  -- Hover: host name, IP, current problems
  -- Click: navigate to host detail
  -- Group by: Host Group dropdown selector

CONTENT -- ROW 4 (Full width):
- "Recent Events" timeline:
  -- Last 20 Zabbix events with timestamp, host, description, severity
  -- Auto-refreshing (10s interval)
  -- Link to correlated OpenNMS events and application metrics

Data sourced from Zabbix proxy via observability-api intermediary.
Include "Open in Zabbix" external link button for full Zabbix web interface access.
```

#### Prompt F-012: OpenNMS Overview (1440px)

```
Design a desktop OpenNMS Event Correlation Overview page at 1440x900px.
Target user: Infrastructure Engineer / Network Operations.

LAYOUT:
- Sidebar (260px) + main area.
- Breadcrumb: "Infrastructure > OpenNMS"

CONTENT -- ROW 1 (KPI Cards, 4-column):
- Card 1: "Managed Nodes" -- 189 -- across 12 locations
- Card 2: "Active Alarms" -- 23 -- 5 critical, 8 major, 10 minor
- Card 3: "Events (24h)" -- 4,567 -- +8% vs yesterday
- Card 4: "Outages" -- 2 active -- 99.94% availability

CONTENT -- ROW 2 (Full width):
- "Network Topology Map" (interactive, 600px tall):
  -- SVG/Canvas rendered network graph
  -- Nodes represent managed devices (routers, switches, servers, services)
  -- Edges represent network connections
  -- Node colors: green (up), red (down), amber (degraded), gray (unmanaged)
  -- Node size: by importance/traffic volume
  -- Hover: node name, IP, status, current alarms
  -- Click: expand to show connected nodes and alarm detail
  -- Zoom and pan controls
  -- Layout: force-directed or hierarchical (toggle)
  -- "Refresh topology" button

CONTENT -- ROW 3 (Two-column, 55/45):
- Left (55%): "Active Alarms" table:
  | Severity | Node | Alarm | Source | Count | First/Last Event | Actions |
  |----------|------|-------|--------|-------|-----------------|---------|
  | Critical (red) | core-router-01 | Interface Down: ge-0/0/1 | SNMP Trap | 3 | 14:20 / 14:32 | Ack, Escalate, Clear |
  | Major (orange) | dist-switch-05 | High Packet Loss (>5%) | Poller | 12 | 13:00 / 14:30 | Ack, Escalate |
  | Major (orange) | app-server-07 | ICMP Unreachable | Poller | 1 | 14:28 | Ack, Escalate |
  | Minor (amber) | printer-floor3 | SNMP Agent Not Responding | Poller | 5 | 10:00 / 14:15 | Ack, Clear |

  Severity: Critical=#ef4444, Major=#f97316, Minor=#f59e0b, Warning=#facc15, Normal=#10b981
  "Count" shows event deduplication count
  Alarm correlation: group related alarms with expandable parent row

- Right (45%): "Event Correlation Summary":
  - "Root Cause Analysis" card:
    -- "core-router-01 interface down appears to be causing:"
    -- "- dist-switch-05 high packet loss (downstream)"
    -- "- app-server-07 ICMP unreachable (via core-router-01)"
    -- Confidence score: 87%
    -- "View full correlation graph" link
  - "Correlated Application Impact" card:
    -- "Affected ERP modules: Finance (API latency +340%), CRM (intermittent errors)"
    -- Links to application metrics and traces for affected modules
    -- "This infrastructure event correlates with 23 application error logs in the last 15 minutes"

CONTENT -- ROW 4 (Full width):
- "Event Timeline" (last 100 events, paginated):
  -- Chronological event list with severity icon, timestamp, node, description
  -- Filter by severity, node, event type
  -- Auto-refresh (30s interval)
  -- "Export Events" button (CSV)

Data sourced from OpenNMS via observability-api. Event correlation engine runs server-side.
Include "Open in OpenNMS" external link button for full OpenNMS web console access.
```

### 7.2 Mobile Pages (390px)

#### Prompt F-013: Mobile Observability Dashboard (390px)

```
Design a mobile Observability Dashboard at 390x844px.
Target: Platform Engineer on-call, checking system health on mobile.

NAVIGATION:
- Bottom tab bar (5 tabs): Home, Alerts (badge: 7), Search, Infra, Profile
- Top status bar: "OBS" logo (left), time range selector (center), notification bell (right)

HOME TAB:
- Greeting: "Good afternoon, Kemi" + current date "Monday, Feb 24, 2026"
- System status banner: "All Systems Operational" (green) or "2 Modules Degraded" (amber)

- KPI Cards (2x3 grid, each 170x80px):
  -- Metric Series: 24.8K
  -- Log Rate: 15.4K/s
  -- Active Alerts: 7
  -- Trace Rate: 8.3K/s
  -- Error Rate: 2.4%
  -- Tenants: 12

- "Module Health" section (scrollable list):
  Each module row: colored dot + module name + uptime % + error rate
  -- HCM: green dot, 99.72%, 1.24%
  -- CRM: green dot, 99.95%, 0.52%
  -- Finance: amber dot, 97.81%, 3.89%
  -- (scrollable for remaining modules)

- "Active Alerts" section (scrollable cards):
  Each card: severity badge + alert name + module tag + firing duration
  -- "High Error Rate - Finance" | critical | 47m ago
  -- "Latency Spike - Sales" | warning | 23m ago
  -- "View All (7)" link

- Quick Actions row (horizontal scroll):
  -- "Silence Alert", "View Logs", "Search Traces", "Create Dashboard"

ALERTS TAB:
- Segment control: "Firing (7)" | "Rules" | "Silences"
- Firing list: cards with severity color bar (left edge), alert name, module, duration
- Swipe right: Acknowledge | Swipe left: Silence
- Tap: full alert detail bottom sheet

SEARCH TAB:
- Large search input at top
- Type filter chips: All | Metrics | Logs | Traces
- Recent searches list
- Results: compact cards with type icon, headline, snippet

All touch targets >= 44px. Pull-to-refresh on home. Skeleton loading states.
Haptic feedback on alert actions. Dark theme default for on-call use.
```

### 7.3 Tablet / Responsive (1024px)

#### Prompt F-014: Tablet Observability Dashboard (1024px)

```
Design a tablet-adapted Observability Dashboard at 1024x768px.

ADAPTATIONS FROM DESKTOP (1440px):
- Sidebar collapsed to icon-only (72px) by default, expandable via hamburger
- KPI cards: 3x2 grid instead of 6-column row
- Module Health table and Recent Alerts: stacked vertically instead of side-by-side
  (Module Health full-width above, Recent Alerts full-width below)
- Table columns: hide "Trend" sparkline column, show on row tap/expand
- Touch-optimized: all clickable areas >= 44px, increased spacing between action buttons
- Time range picker: compact dropdown instead of inline display

HEADER:
- Hamburger menu (left), "Dashboard" title (center), time range + notification bell + avatar (right)
- Breadcrumbs hidden; replaced by title

SPECIFIC LAYOUT:
Row 1: 3x2 KPI grid (Metric Series, Log Rate, Active Alerts / Trace Rate, Error Rate, Tenants)
Row 2: Module Health table (full width, hide sparkline column)
Row 3: Recent Alerts list (full width)

Split-screen support: annotate that the dashboard can display in iPadOS split view
at 507px width (compact) falling back to mobile layout.

All views support both landscape (1024x768) and portrait (768x1024) orientations.
Portrait: KPI cards become 2x3 grid; tables become horizontally scrollable.
```

---

## 8. Mobile And Responsive Prompt

```
Design responsive variants for all ERP-Observability pages at three breakpoints:
1440px (desktop), 1024px (tablet), 390px (mobile).

RESPONSIVE STRATEGY:

DESKTOP (1440px):
- Full sidebar (260px) + main content area
- Multi-column layouts (up to 6 columns for KPI cards)
- Side-by-side panels (tables + charts)
- Inline sparklines and trend indicators
- Full PromQL editor with autocomplete
- Waterfall diagram with full span tree
- Network topology map with pan/zoom

TABLET (1024px):
- Collapsed sidebar (72px, icon-only) with hamburger toggle
- KPI cards: 3 per row (2 rows for 6 cards)
- Tables: hide secondary columns (trend, labels), expand on tap
- Charts: full width, stacked vertically
- PromQL editor: full width, reduced height
- Waterfall: horizontal scroll for wide traces
- Topology map: simplified layout, zoom controls prominent

MOBILE (390px):
- Bottom tab navigation (5 tabs: Home, Alerts, Search, Infra, Profile)
- No sidebar; full-screen content
- KPI cards: 2 per row (3 rows for 6 cards)
- Tables replaced by card lists (one card per row)
- Charts: full width, swipe to navigate between charts
- Log viewer: single-column, expand on tap for details
- Trace waterfall: vertical layout (spans stacked, not horizontal timeline)
- Search: full-screen search overlay
- Forms: single-column, full-width inputs
- Modals replaced by bottom sheets (slide up to 85% height)
- Time range picker: bottom sheet with preset chips

NAVIGATION MAPPING:
| Desktop Sidebar | Tablet Sidebar | Mobile Bottom Tab |
|-----------------|---------------|-------------------|
| Dashboard | Dashboard | Home |
| Metrics | Metrics | (via Home or Search) |
| Logs | Logs | (via Search tab) |
| Traces | Traces | (via Search tab) |
| Alerts | Alerts | Alerts |
| Dashboards | Dashboards | (via Home) |
| Tenants | Tenants | (via Profile > Admin) |
| Search | Search | Search |
| Infrastructure | Infrastructure | Infra |

TOUCH INTERACTIONS:
- All interactive elements >= 44x44px touch target
- Pull-to-refresh on all list/dashboard pages
- Swipe left/right on alert cards for quick actions (Acknowledge/Silence)
- Swipe between dashboard panels on mobile
- Pinch-to-zoom on topology map and trace waterfall
- Long-press on log line to copy full message
- Haptic feedback on destructive actions and confirmations

PERFORMANCE:
- Mobile: lazy-load non-visible tabs, reduce chart data points to 50 (vs 200 desktop)
- Tablet: reduce sparkline resolution, defer topology map rendering
- All: skeleton loading within 200ms, content within 1s on 4G
- Images: WebP, max 80KB hero, 20KB thumbnails, responsive srcset

ACCESSIBILITY:
- VoiceOver/TalkBack support: all charts have screen reader descriptions
- Reduced motion: disable sparkline animations, chart transitions when prefers-reduced-motion
- High contrast mode: verified color contrast on all breakpoints
- Keyboard navigation: full tab order on desktop, focus management on modals/sheets
```

---

## 9. Validation Checklist

```markdown
## Design Handoff Checklist -- ERP-Observability

### Visual Completeness
- [ ] All 20 pages designed for 1440px, 1024px, and 390px breakpoints
- [ ] Light and dark theme variants complete for all pages
- [ ] All component states documented (default, hover, active, disabled, error, loading, empty)
- [ ] Realistic data used (no "Lorem ipsum" or placeholder names)
- [ ] Time-series charts use realistic metric data with appropriate scales
- [ ] Log entries use realistic application log messages
- [ ] Trace waterfalls show realistic service call hierarchies

### Accessibility
- [ ] Color contrast ratios verified (>= 4.5:1 text, >= 3:1 UI)
- [ ] Touch targets >= 44x44px verified on all interactive elements
- [ ] Focus order annotated for keyboard navigation
- [ ] Screen reader annotations added for all non-text elements (charts, sparklines, badges)
- [ ] Keyboard navigation paths documented (J/K for log navigation, / for search)
- [ ] Log and trace monospace text meets minimum 13px font size
- [ ] Status indicators use both color and icon/text (never color-only)

### Developer Handoff
- [ ] Design tokens exported in Style Dictionary JSON format
- [ ] Component specs include spacing, sizing, and behavioral notes
- [ ] API data mapping documented (which service endpoint populates which component):
  -- KPI cards: observability-api /v1/stats
  -- Module health: victoria-metrics PromQL queries
  -- Alert list: Hasura GraphQL obs_alert_rules
  -- Log list: Quickwit search API
  -- Trace list: Quickwit trace search API
  -- Tenant list: Hasura GraphQL obs_tenants
  -- Zabbix data: observability-api /v1/zabbix/proxy
  -- OpenNMS data: observability-api /v1/opennms/proxy
- [ ] Interaction specifications (transitions, animations, gesture handling)
- [ ] Error states and empty states designed for every data-dependent component
- [ ] WebSocket subscription endpoints documented for live data (graphql-ws)
- [ ] PromQL editor component specs: syntax highlighting rules, autocomplete behavior

### Theme and Brand
- [ ] Primary color #059669 applied consistently across all pages
- [ ] Sidebar gradient logo (#059669 to #0d9488) matches implementation
- [ ] Status color mappings match types file (SEVERITY_COLORS, LOG_LEVEL_COLORS, etc.)
- [ ] JetBrains Mono used for all metric names, expressions, timestamps, IDs
- [ ] Plus Jakarta Sans used for all UI text
- [ ] Border radius: 10px cards, 8px buttons/inputs, 6px tags

### AIDD Guardrails
- [ ] No prohibited actions (cross-tenant data access, irreversible deletes) exposed without safeguards
- [ ] Destructive actions (delete alert rule, delete tenant, purge logs) have confirmation dialogs
- [ ] Supervised actions (tenant provisioning, bulk alert disable) have confirmation steps
- [ ] Tenant isolation visual indicators: tenant name always visible in header
- [ ] Audit trail access points visible for configuration changes
- [ ] Feature flag annotation points marked (opennms_correlation_enabled, zabbix_integration_enabled)
- [ ] PII redaction toggle present in log viewer
- [ ] Dashboard sharing generates scoped tokens with TTL display
- [ ] Role-based UI elements annotated (admin-only: tenant management, editor: dashboard create)

### Performance Acceptance Targets
| Metric | Target | Measurement |
|--------|--------|-------------|
| Initial route bundle size | < 220 KB gzipped | Webpack bundle analyzer |
| Largest Contentful Paint (LCP) | < 1.5s | Lighthouse CI |
| First Input Delay (FID) | < 100ms | Web Vitals |
| Cumulative Layout Shift (CLS) | < 0.1 | Web Vitals |
| Time to Interactive (TTI) | < 2.5s | Lighthouse CI |
| API read latency (p99) | < 200ms | perf/targets.yaml SLO |
| API write latency (p99) | < 500ms | perf/targets.yaml SLO |
| Availability | >= 99.95% | perf/targets.yaml SLO |
| Table render (1000 rows) | < 300ms | Custom perf mark |
| Chart render (time series) | < 500ms | Custom perf mark |
| Log tail latency (WebSocket) | < 200ms | Custom perf mark |
| Trace waterfall render (100 spans) | < 400ms | Custom perf mark |
| Search results (cross-signal) | < 1s | Custom perf mark |
```

---

## 10. Output Packaging Convention

```
Deliverable naming:
  OBS-{PromptID}-{Breakpoint}-{Theme}-v{Version}.fig
  Example: OBS-F003-1440-light-v1.fig
  Example: OBS-F005-1440-dark-v1.fig
  Example: OBS-F013-390-dark-v1.fig

Layer naming:
  {Page}/{Section}/{Component}-{State}
  Example: Dashboard/KPICards/MetricSeriesCard-Default
  Example: Logs/LogTable/LogRow-Expanded
  Example: Traces/Waterfall/SpanBar-Error
  Example: Alerts/AlertRuleForm/SeveritySelector-Critical
  Example: Tenants/TenantShow/QuotaGauge-Warning

Asset export:
  /exports/obs/{breakpoint}/{page}/
  Example: /exports/obs/1440/dashboard/kpi-metric-series.png
  Example: /exports/obs/1440/traces/waterfall-diagram.png
  Example: /exports/obs/390/alerts/alert-card-firing.png

Handoff tokens:
  /tokens/obs-tokens.json (Style Dictionary format)

Component library:
  /exports/obs/components/{component-name}-{state}-{theme}.png
  Example: /exports/obs/components/status-badge-firing-light.png
  Example: /exports/obs/components/log-line-error-dark.png
  Example: /exports/obs/components/trace-waterfall-bar-ok-light.png

Make automation diagrams:
  /exports/obs/automations/{scenario-id}.png
  Example: /exports/obs/automations/M001-alert-escalation.png
  Example: /exports/obs/automations/M002-tenant-provisioning.png
```

---

## 11. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-24 | AIDD System | Initial comprehensive prompt set covering dashboard, metrics, logs, traces, alerts, dashboards, tenants, search, Zabbix, OpenNMS, mobile views, tablet adaptations, design tokens, component library, and 5 Make automation workflows |

---

## 12. Round 03 Implementation Handoff (Web + Mobile + Desktop)

### 12.1 Delivered UI Surfaces
- `web/`: role-based observability console for Platform Engineers, Application Developers, Security Analysts, and Tenant Administrators.
- `apps/mobile/`: on-call incident response and system health monitoring for mobile-first operations.
- `apps/desktop/`: NOC command center for infrastructure monitoring and multi-module oversight.

### 12.2 Design Token Mapping
- Tokens are implemented in `web/src/theme.ts` and consumed by all frontend components.
- Palette: primary `#059669`, success `#10b981`, warning `#f59e0b`, danger `#ef4444`, info `#3b82f6`.
- Typography baseline: Plus Jakarta Sans (all UI text), JetBrains Mono (metrics, PromQL, logs, trace IDs).
- Sidebar: dark theme `#001529`, active item `#059669`, hover `rgba(5, 150, 105, 0.6)`.
- Border radius: 10px (cards, tables), 8px (buttons, inputs, menu items), 6px (tags, badges).
- Control height: 40px (buttons, inputs, selects, date pickers).

### 12.3 Figma Make Build Prompt (Round 03)
```
Build ERP-Observability GA-ready UI kit and application flows using the existing token system.

Deliverables:
1. Desktop web ops console with pages:
   - Dashboard, Metrics Explorer, Log Viewer, Trace Analyzer, Alert Manager,
     Dashboard Builder, Tenant Manager, Unified Search, Zabbix Overview, OpenNMS Overview.
2. Mobile on-call app with:
   - KPI cards, alert queue with swipe actions, quick search, infrastructure health.
3. Desktop NOC command center view with:
   - Multi-module health matrix, live alert feed, infrastructure topology, SLO compliance.

Interaction constraints:
- p95 interaction latency <= 120ms for primary workflows.
- Alert acknowledgement CTA visible above fold on 1366x768.
- Log tail streaming at >= 10,000 entries/second with virtualized rendering.
- Preserve AIDD confirmation patterns for destructive actions (delete tenant, purge data).
- Tenant isolation: X-Scope-OrgID header enforced on every API call.
```

### 12.4 Commercial UX Guardrails
- No dead-end states: every error/empty state includes clear recovery actions.
- Role-first navigation: all primary personas can complete top 3 tasks within two clicks.
- KPI-first information architecture: actionable metrics shown before raw tables.
- Incident-response optimized: dark theme default for on-call views, high-contrast status indicators.
- Data freshness always visible: "Live" indicator or "Last updated X ago" on every data-dependent view.

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
Design a control plane view for ERP-Observability that shows:
1. Hasura GraphQL connectivity (latency, success %, schema drift status)
2. ERP-DBaaS health (connection pool, replica lag, failover posture)
3. ERP-IAM integration (OIDC issuer health, token validation errors, session invalidations)
4. ERP-Observability status (OTLP export rate, tracing availability, alert health)

Include: command surface, timeline, runbook links, and one-click diagnostics panel.
Add states for: fully healthy, degraded IAM, degraded DB, degraded GraphQL, global outage.
```

### Prompt F-901: Role-Aware Workspace + Guardrail Surface
```
Generate a role-aware workspace for ERP-Observability with:
- Role lenses: Operator, Manager, Auditor, Admin
- Guardrail panel with mode badge (Protected / Supervised / Autonomous)
- Inline policy rationale for blocked/supervised actions
- Approval request flow for supervised actions

Ensure each role sees distinct navigation and action priorities.
```

### Prompt F-902: Incident + Recovery UX
```
Design a full incident-response flow for ERP-Observability:
- Alert ingestion to triage board
- Impacted tenant view and blast-radius visualization
- Action timeline with runbook steps and rollback controls
- Post-incident report generator modal

Must include: trace-id copy, audit evidence export, and SLA breach indicators.
```

### Prompt F-903: Mobile-Responsive Executive Snapshot
```
Create mobile and tablet executive dashboards for ERP-Observability:
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
Create a Make scenario for ERP-Observability that:
1. Triggers on deployment/webhook events
2. Validates GraphQL, IAM, DB, and OTLP health in sequence
3. Creates incident ticket when thresholds are breached
4. Posts status updates to operations channels
5. Writes audit trail to persistent store

Add branching for: transient failures, policy violations, and hard-stop conditions.
```

### Prompt M-901: Tenant Onboarding Orchestration
```
Generate a Make scenario for tenant onboarding in ERP-Observability:
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
