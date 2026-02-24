# Figma & Make Prompts -- ERP-AIOps (AI for IT Operations)
> Version: 1.0 | Last Updated: 2026-02-24 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make (Integromat) automation prompts for the ERP-AIOps module. ERP-AIOps is the AI-powered IT Operations vertical of the consolidated ERP platform, powered by OpsTrac. It covers incident management (AI-powered detection, classification, root cause analysis, resolution), anomaly detection (ML-based anomaly detection across metrics, logs, events), AIOps rules (correlation rules, threshold rules, pattern matching), service topology (dependency mapping, impact analysis, change propagation), auto-remediation (runbooks, automated actions, approval workflows), cost optimization (resource waste detection, rightsizing, reserved instance planning), and security scanning (vulnerability detection, compliance checks, threat analysis). All prompts reference real services (aiops-gateway, opstrac-api, ai-brain) and align with the AIDD guardrails defined in `erp/aidd.guardrails.yaml`.

The backend is a polyglot architecture: **Rust core** (40+ crates for anomaly detection, causal analysis, event correlation, ML inference via Axum on port 8080), **Python AI brain** (FastAPI on port 8001 for LLM routing, forecasting, adaptive thresholds, incident summarization), and **Go gateway** (aiops-gateway on port 8090 for API orchestration). The frontend stack is **Refine.dev + Ant Design** (React) with Hasura GraphQL (port 19109, `aiops_` table prefix), multi-tenant via `tenant_id` / `X-Tenant-ID` headers, and integration with ALL 20+ ERP modules via OpenTelemetry Collector telemetry.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance on every screen; color contrast ratio >= 4.5:1 for text, >= 3:1 for large text and UI components
- Minimum 44x44px touch targets on all interactive elements (buttons, links, checkboxes, table row actions)
- Clear focus-visible outlines (2px solid, offset 2px) for keyboard navigation
- All form fields include visible labels (never placeholder-only), required field indicators, and inline validation messages
- Plain-language labels (e.g., "Open Incidents" not "Incident Queue Management Interface")
- Semantic HTML landmarks: `<nav>`, `<main>`, `<aside>`, `<footer>` reflected in Figma layer naming
- Screen reader annotations for charts (alt-text, data tables), icon-only buttons (aria-label), and live regions
- Real-time alert banners must include ARIA live region annotations for screen reader announcement

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading; initial route bundle < 220 KB gzipped
- Skeleton UIs shown within 200ms; full content within 1s on 4G
- Virtualized tables for datasets > 50 rows (incident lists, anomaly feeds, security findings)
- Image assets: WebP format, max 80 KB hero, max 20 KB thumbnails
- Prefetch next-likely routes (e.g., incident detail from incident list)
- Topology graph rendering: WebGL-accelerated, < 500ms for 200 nodes
- Real-time WebSocket feeds (anomalies, incidents) debounced at 1s intervals to prevent UI thrashing
- Time-series charts: downsample to max 500 data points for smooth rendering

### 2.3 Reliability, Trust, And Safety
- Every destructive action (delete rule, cancel remediation, dismiss anomaly, close incident) requires a confirmation dialog with explicit typing or checkbox
- Inline recovery paths on all error states ("Retry", "Go back", "Contact support")
- Optimistic UI for low-risk mutations (acknowledge incident) with undo toast (5s window)
- Trust signals: last-synced timestamp on dashboards, "Saved" indicator on rule forms, remediation execution audit trail
- Audit trail accessible for compliance integration; all remediation actions logged with initiator, timestamp, and approval chain
- AI-generated content (root cause analysis, incident summaries, remediation suggestions) must be clearly labeled as "AI-generated" with confidence scores where applicable
- Human-in-the-loop gates on all auto-remediation actions affecting production systems

### 2.4 Observability And Testability
- Every CTA emits a structured analytics event: `{event, module, entity, action, user_role, timestamp}`
- Error boundaries at page and widget level with correlation_id for support tickets
- Performance instrumentation: LCP, FID, CLS metrics on all page-level components
- Feature flag integration points annotated in Figma (e.g., "Visible when `cost_optimization_enabled` = true")
- All API calls include `X-Tenant-ID` and `X-Correlation-ID` headers for distributed tracing

### 2.5 Security And Privacy
- All tenant data scoped via `tenant_id`; no cross-tenant data leakage in any UI component
- Sensitive fields (API keys, secrets in security findings, remediation parameters) masked by default with "Reveal" toggle
- Role-based visibility: SRE sees all modules, Developer sees assigned services only, Manager sees cost and summary views
- Session timeout warning at 25 minutes with "Extend Session" option; forced logout at 30 minutes
- CSP-compliant inline styles; no `eval()` or dynamic script injection in any component

---

## 3. Module-Specific Design Context

### 3.1 Services Architecture

| Service | Technology | Port | Role |
|---------|-----------|------|------|
| aiops-gateway | Go (Gin) | 8090 | API gateway, routing, auth, rate limiting |
| opstrac-api | Rust (Axum) | 8080 | Core engine: anomaly detection, event correlation, causal analysis, ML inference |
| ai-brain | Python (FastAPI) | 8001 | LLM routing, forecasting, adaptive thresholds, incident summarization, NLP |
| Hasura GraphQL | Haskell | 19109 | GraphQL API layer over PostgreSQL (`aiops_` table prefix) |
| PostgreSQL | - | 5432 | Primary data store for incidents, anomalies, rules, topology, cost, security |
| Redis | - | 6379 | Caching, real-time event streams, session store |
| OTel Collector | Go | 4317/4318 | Telemetry ingestion from all 20+ ERP modules |

### 3.2 Target Personas

| Persona | Role | Primary Tasks |
|---------|------|---------------|
| SRE / Platform Engineer | Site Reliability Engineer | Monitor incidents, investigate anomalies, manage topology, execute remediations |
| DevOps Engineer | Infrastructure Automation | Configure rules, manage runbooks, review auto-remediation results |
| Security Analyst | Security Operations | Review security findings, track vulnerabilities, compliance monitoring |
| Engineering Manager | Team Lead | Cost optimization oversight, incident trends, service health summary |
| FinOps Analyst | Cost Management | Resource waste detection, rightsizing recommendations, reserved instance planning |
| IT Director | Executive Oversight | Dashboard KPIs, cross-module health, cost savings tracking |

### 3.3 Key Entities (from `aiops.types.ts`)

| Entity | Hasura Resource | Key Fields |
|--------|----------------|------------|
| Incident | `incidents` | id, tenant_id, title, description, severity (critical/high/medium/low/info), status (open/acknowledged/investigating/resolved/closed), source, affected_services[], root_cause, correlation_id |
| Anomaly | `anomalies` | id, tenant_id, metric_name, service, module, anomaly_type, severity, expected_value, actual_value, deviation_percent, status (active/investigating/resolved/dismissed) |
| Rule | `rules` | id, tenant_id, name, description, type, condition (JSON), action (JSON), enabled, priority |
| TopologyNode | `topology_nodes` | id, tenant_id, name, type, module, status (healthy/degraded/down/unknown), dependencies[] |
| RemediationAction | `remediation_actions` | id, tenant_id, incident_id, action_type, target_service, parameters (JSON), status (pending/running/completed/failed/cancelled) |
| CostReport | `cost_reports` | id, tenant_id, period_start, period_end, total_cost, breakdown (JSON), recommendations[] |
| SecurityFinding | `security_findings` | id, tenant_id, title, description, severity, category (vulnerability/misconfiguration/exposed_secret/weak_authentication/network_exposure/compliance_violation/data_leakage/privilege_escalation), status |

### 3.4 Status Color Mappings

```
Severity: critical=#ef4444, high=#f97316, medium=#f59e0b, low=#3b82f6, info=#94a3b8
Incident Status: open=#ef4444, acknowledged=#f59e0b, investigating=#3b82f6, resolved=#10b981, closed=#64748b
Anomaly Status: active=#ef4444, investigating=#f59e0b, resolved=#10b981, dismissed=#94a3b8
Remediation Status: pending=#f59e0b, running=#3b82f6, completed=#10b981, failed=#ef4444, cancelled=#94a3b8
Topology Status: healthy=#10b981, degraded=#f59e0b, down=#ef4444, unknown=#94a3b8
Security Category: vulnerability=#ef4444, misconfiguration=#f97316, exposed_secret=#dc2626,
    weak_authentication=#f59e0b, network_exposure=#3b82f6, compliance_violation=#8b5cf6,
    data_leakage=#ec4899, privilege_escalation=#ef4444
```

---

## 4. Figma Make Master Prompt (Copy-Paste Ready)

```
Create a complete Figma design for the ERP-AIOps module -- an AI-powered IT Operations
platform (powered by OpsTrac) within a multi-module enterprise ERP system.

DESIGN SYSTEM:
- Primary: #7c3aed (Purple -- represents intelligence and AI-driven operations)
- Primary Hover: #6d28d9
- Primary Light: rgba(124, 58, 237, 0.08)
- Success: #10b981 (Healthy, Resolved, Completed)
- Warning: #f59e0b (Acknowledged, Investigating, Pending, Degraded)
- Danger: #ef4444 (Critical, Open, Failed, Down)
- Info: #3b82f6 (Investigating, Running, Low severity)
- Orange: #f97316 (High severity, Security findings)
- Neutral-50: #f5f7fa (Page background / colorBgLayout)
- Neutral-100: #fafbfc (Table header background)
- Neutral-200: #e5e7eb (Borders, dividers)
- Neutral-500: #64748b (Secondary text, table headers)
- Neutral-900: #111827 (Primary text)
- Surface: #ffffff (Cards, modals, container background)
- Sidebar Dark: #001529 (Sidebar background)
- Sidebar Active: #7c3aed (Selected menu item background)
- Sidebar Hover: rgba(124, 58, 237, 0.6) (Menu item hover)

TYPOGRAPHY (Plus Jakarta Sans):
- Display: 28px/36px, semibold 600 -- page titles ("Dashboard", "Incidents")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Recent Incidents")
- Heading 2: 20px/28px, medium 500 -- card titles ("Open Incidents")
- Heading 3: 16px/24px, medium 500 -- subsection headers
- Body: 14px/20px, regular 400 -- paragraph text, table cells
- Body Small: 13px/18px, regular 400 -- captions, timestamps, helper text
- Label: 14px/20px, medium 500 -- form labels, column headers
- Mono: 13px/20px, JetBrains Mono -- incident IDs, metric values, JSON, correlation IDs

SPACING SCALE (4px base):
- 4px (xs), 8px (sm), 12px (md), 16px (lg), 24px (xl), 32px (2xl), 48px (3xl)
- Card padding: 24px (paddingLG)
- Section gap: 24px
- Page margin: 32px (desktop), 16px (mobile)

BORDER RADIUS:
- Small: 6px (tags, badges, chips)
- Medium: 8px (buttons, inputs, selects, menu items)
- Large: 10px (cards, tables, modals)
- Full: 9999px (status dots, avatars)

ELEVATION / SHADOWS:
- Level 0: none (flat elements)
- Level 1: 0 1px 3px rgba(0,0,0,0.1) -- cards, dropdowns
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, popovers
- Level 3: 0 8px 24px rgba(0,0,0,0.12) -- dialogs, command palette

GRID:
- Desktop (1440px): 12-column, 72px columns, 24px gutter, 260px sidebar
- Tablet (1024px): 12-column, 56px columns, 16px gutter, collapsed sidebar (72px)
- Mobile (390px): 4-column, fluid, 16px gutter, bottom nav

PAGES TO DESIGN (8 routes from App.tsx):
1. Dashboard (/) -- AIOps command center with 6 KPI cards and dual data tables
2. Incidents List (/incidents) -- filterable, sortable incident table with severity/status badges
3. Incident Detail (/incidents/:id) -- full incident view with timeline, RCA, affected services
4. Incident Create (/incidents/create) -- multi-step incident creation form
5. Anomalies List (/anomalies) -- anomaly feed with metric charts and severity indicators
6. Anomaly Detail (/anomalies/:id) -- anomaly deep-dive with expected vs actual visualization
7. Rules List (/rules) -- rule management table with enable/disable toggles
8. Rule Create (/rules/create) -- rule builder with condition/action JSON editors
9. Topology View (/topology) -- interactive service dependency graph
10. Remediation List (/remediation) -- remediation action log with execution status
11. Cost Dashboard (/cost) -- cost optimization overview with savings recommendations
12. Security Dashboard (/security) -- security findings by category with severity breakdown

NAVIGATION (from Sidebar.tsx):
- Collapsible sidebar (260px expanded, 72px collapsed) with dark theme (#001529)
- Logo area: 32x32px purple gradient badge "AI" + "ERP AIOps" text
- Menu items: Dashboard, Incidents, Anomalies, Rules, Topology, Remediation, Cost, Security
- Icons: DashboardOutlined, AlertOutlined, ExperimentOutlined, SettingOutlined,
         ApartmentOutlined, ThunderboltOutlined, DollarOutlined, SafetyOutlined
- Active state: #7c3aed background + white text
- Hover state: rgba(124, 58, 237, 0.6) background

Each page must include: loading skeleton state, empty state, error state with retry,
and both light theme (default) and dark theme variant. Annotate all interactive
elements with click targets >= 44px. Use realistic AIOps data throughout.
```

---

## 5. Figma Make Design-System Prompt (Component Library)

### Prompt F-001: Core Design Tokens -- AIOps Theme

```
Create a Figma page titled "AIOps Design Tokens" containing the complete token
specification for the AI for IT Operations module.

COLOR PALETTE (Light Theme):
- Primary: #7c3aed (Purple -- represents AI intelligence and operational insight)
- Primary Hover: #6d28d9
- Primary Light: rgba(124, 58, 237, 0.08)
- Secondary: #3b82f6 (Blue -- used for info states, investigating status)
- Success: #10b981 (Resolved, Healthy, Completed)
- Warning: #f59e0b (Acknowledged, Investigating, Pending, Degraded)
- Danger: #ef4444 (Critical, Open, Failed, Down)
- Orange: #f97316 (High severity, Security warnings)
- Pink: #ec4899 (Data leakage category)
- Info: #3b82f6 (Low severity, Running status)
- Neutral-50: #f5f7fa (Page background)
- Neutral-100: #fafbfc (Table header background)
- Neutral-200: #e5e7eb (Borders, dividers)
- Neutral-500: #64748b (Secondary text, column headers)
- Neutral-900: #111827 (Primary text)
- Surface: #ffffff (Cards, modals)

COLOR PALETTE (Dark Theme):
- Primary: #a78bfa (Light Purple)
- Primary Hover: #8b5cf6
- Background: #0f172a
- Surface: #1e293b
- Surface Elevated: #334155
- Border: #475569
- Text Primary: #f1f5f9
- Text Secondary: #94a3b8
- Success/Warning/Danger: same hues, lightened 20%

TYPOGRAPHY (Plus Jakarta Sans font family):
- Display: 28px/36px, semibold 600 -- page titles ("Dashboard", "Incidents")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Recent Incidents")
- Heading 2: 20px/28px, medium 500 -- card titles ("Open Incidents")
- Heading 3: 16px/24px, medium 500 -- subsection headers
- Body: 14px/20px, regular 400 -- paragraph text, table cells
- Body Small: 13px/18px, regular 400 -- captions, timestamps, helper text
- Label: 14px/20px, medium 500 -- form labels, column headers
- Mono: 13px/20px, JetBrains Mono -- incident IDs, metric values, JSON configs

SPACING SCALE (4px base):
- 4px (xs), 8px (sm), 12px (md), 16px (lg), 24px (xl), 32px (2xl), 48px (3xl)
- Card padding: 24px
- Section gap: 24px
- Page margin: 32px (desktop), 16px (mobile)

ELEVATION / SHADOWS:
- Level 0: none (flat elements)
- Level 1: 0 1px 3px rgba(0,0,0,0.1) -- cards, dropdowns
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, popovers
- Level 3: 0 8px 24px rgba(0,0,0,0.12) -- dialogs, command palette

BORDER RADIUS:
- Small: 6px (badges, tags, chips)
- Medium: 8px (buttons, inputs, selects, menu items)
- Large: 10px (cards, tables, modals)
- Full: 9999px (status dots, avatars, circular indicators)

GRID:
- Desktop (1440px): 12-column, 72px columns, 24px gutter, 260px sidebar
- Tablet (1024px): 12-column, 56px columns, 16px gutter, collapsed sidebar
- Mobile (390px): 4-column, fluid, 16px gutter, bottom nav

Include token swatches, type scale samples, spacing ruler, and shadow comparison.
Both light and dark themes must be fully specified side-by-side.
```

### Prompt F-002: Component Library -- AIOps Components

```
Create a Figma page titled "AIOps Component Library" with the following reusable
components, each showing all states (default, hover, active, disabled, error,
loading) and both light/dark theme variants.

KPI CARD (from KPICard component):
- Icon (left, colored circle background), metric value (large, Statistic 28px),
  title label (small, secondary text), trend indicator (arrow + percentage)
- Variants for each KPI:
  -- "Open Incidents" | AlertOutlined | value: 7 | color: #ef4444 | trend: -12.5% (red arrow down = bad)
  -- "Active Anomalies" | ExperimentOutlined | value: 14 | color: #f59e0b | trend: -8.3%
  -- "Rules Enabled" | SettingOutlined | value: 12 | color: #7c3aed | trend: +2
  -- "Auto-Remediations" | ThunderboltOutlined | value: 8 | color: #3b82f6 | trend: +15%
  -- "Cost Savings" | DollarOutlined | value: "$24.5K" | color: #10b981 | trend: +22.1%
  -- "Security Findings" | SafetyOutlined | value: 5 | color: #f97316 | trend: -5.2%
- Loading state: skeleton shimmer on value and trend
- Size: responsive, min 180px width

STATUS BADGE (from StatusBadge component):
- Rounded pill badge with colored dot + label text
- Severity variants: Critical (#ef4444), High (#f97316), Medium (#f59e0b),
  Low (#3b82f6), Info (#94a3b8)
- Incident status variants: Open (#ef4444), Acknowledged (#f59e0b),
  Investigating (#3b82f6), Resolved (#10b981), Closed (#64748b)
- Anomaly status variants: Active (#ef4444), Investigating (#f59e0b),
  Resolved (#10b981), Dismissed (#94a3b8)
- Remediation status variants: Pending (#f59e0b), Running (#3b82f6),
  Completed (#10b981), Failed (#ef4444), Cancelled (#94a3b8)
- Topology status variants: Healthy (#10b981), Degraded (#f59e0b),
  Down (#ef4444), Unknown (#94a3b8)
- Size variants: small (for table cells), default (for detail views)

PAGE HEADER (from PageHeader component):
- Title (Display typography) + subtitle (Body Small, secondary color)
- Optional action buttons area (right-aligned)
- Breadcrumb variant above title
- Examples: "Dashboard" / "Welcome back! Here's your AIOps operations overview."

INCIDENT CARD:
- Full-width card for incident list items
- Left: Severity indicator bar (4px wide, colored by severity)
- Content: Title (linked, font-weight 500), description excerpt (1 line),
  affected services as tags, source badge, correlation ID (mono)
- Right: Status badge, created timestamp (relative), assigned avatar
- Hover: subtle shadow elevation, row action icons appear (View, Acknowledge, Assign)
- Example: "High CPU Usage on ERP-Finance Gateway" | Critical | Open | 35 min ago

ANOMALY METRIC CARD:
- Compact card showing metric_name, service name, anomaly_type badge
- Mini sparkline chart showing expected vs actual value trend
- Deviation percentage (large, colored by severity): "+47.3%"
- Expected: 45.2 | Actual: 66.6 (mono typography)
- Status badge and detected_at timestamp
- Example: "api_latency_p99 | erp-finance-gateway | spike | +47.3% | Critical"

TOPOLOGY NODE:
- Circular node (48px) with service icon inside
- Status ring: 3px border colored by status (healthy=green, degraded=amber, down=red)
- Label below: service name (truncated at 16 chars)
- Connection lines between nodes (solid=healthy, dashed=degraded, dotted=unknown)
- Hover: enlarged to 64px with full service info tooltip
- Selected: purple ring highlight (#7c3aed)

REMEDIATION ACTION CARD:
- Horizontal card: action_type icon, target_service, status badge, timing info
- Parameters summary (collapsed JSON, expandable)
- Initiated by: user avatar + name | initiated_at timestamp
- Progress bar for running state
- Result summary for completed/failed state
- Example: "Restart Service | erp-hcm-gateway | Running | Initiated by Abiola 2m ago"

COST RECOMMENDATION CARD:
- Card with recommendation type icon (rightsizing, reserved instance, waste)
- Title: "Rightsize erp-finance-api from 4 vCPU to 2 vCPU"
- Estimated savings: "$450/month" (large, green)
- Confidence: "High (92%)" badge
- Impact level: Low/Medium/High indicator
- Action buttons: "Apply" primary, "Dismiss" secondary, "Schedule" tertiary

SECURITY FINDING CARD:
- Severity indicator (left border), category badge (colored by SECURITY_CATEGORY_COLORS)
- Title, description (1-2 lines), affected_resource (mono)
- Remediation suggestion (collapsible)
- Status: Open/In Progress/Resolved/Dismissed
- Detected timestamp
- Example: "SQL Injection Vulnerability | erp-commerce-api | Critical | vulnerability"

SIDEBAR NAVIGATION (from Sidebar.tsx):
- Collapsible sidebar (expanded 260px, collapsed 72px)
- Dark theme (#001529 background)
- Logo: 32x32px purple gradient badge with "AI" text + "ERP AIOps" title
- Menu items: Dashboard, Incidents, Anomalies, Rules, Topology, Remediation, Cost, Security
- Active state: #7c3aed background + white text, 8px border radius
- Hover state: rgba(124, 58, 237, 0.6) background
- Icons: Ant Design outlined icons in rgba(255,255,255,0.75), selected: #ffffff
- Bottom: user profile mini-card (avatar, name, role, logout)

DATA TABLE (AIOps variant):
- Ant Design-based table with sortable columns, row selection checkboxes,
  inline status badges, row hover actions (eye icon = view, pencil = edit)
- Header background: #fafbfc, header text: #64748b
- Pagination: "Showing 1-25 of 342 incidents" with page selector
- Filters bar above table with severity, status, service, date-range chips
- Empty state: illustration + "No incidents match your filters" + clear filters button
- Border radius: 10px

FORM COMPONENTS (AIOps-specific):
- Service search/select dropdown with status dot + service name + module
- Severity selector (radio group with colored indicators)
- JSON condition editor (code editor with syntax highlighting)
- Tag input for affected_services (autocomplete from topology_nodes)
- Date-time range picker for incident time windows
- Threshold input with unit selector (%, ms, req/s, MB)
- Rule priority slider (1-10 with labeled ticks)

NOTIFICATION/ALERT BANNER:
- Full-width banner at top of content area for critical alerts
- Variants: Critical (red bg), Warning (amber bg), Info (blue bg), Success (green bg)
- Content: Icon + message text + action link + dismiss button
- Auto-dismiss after 10s for non-critical, persistent for critical
- Example: "Critical: 3 incidents detected on ERP-Finance services in the last 5 minutes"
```

---

## 6. Make Automation Prompt Pack

### Prompt M-001: AI-Powered Incident Detection and Triage Workflow

```
Create a Make (Integromat) scenario titled "AIOps -- Incident Detection & Triage".

TRIGGER: Webhook from opstrac-api (Rust core) when a new anomaly correlation
produces an incident candidate.
Payload includes: { correlation_id, anomaly_ids[], affected_services[],
  metric_signatures[], severity_score, confidence, detected_at, tenant_id }

STEP 1 -- Validate and Enrich:
- Verify tenant_id exists and is active
- Call aiops-gateway: GET /v1/topology/services?names={affected_services}
  to enrich with service metadata (owners, criticality, dependencies)
- Call opstrac-api: GET /v1/anomalies/batch?ids={anomaly_ids}
  to fetch full anomaly details
- Route: If confidence < 0.5 -> create low-priority investigation ticket only

STEP 2 -- AI Classification (ai-brain):
- Call ai-brain: POST /v1/classify
  Body: { anomaly_data, service_context, historical_patterns }
- Returns: { incident_type, severity_recommendation, suggested_title,
  root_cause_hypothesis, confidence_score }
- If AI unavailable: fallback to rule-based classification from opstrac-api

STEP 3 -- Create Incident:
- Call Hasura GraphQL (port 19109): mutation insert_aiops_incidents_one
  Body: { title: suggested_title, description: auto-generated summary,
  severity: severity_recommendation, status: "open", source: "ai-detection",
  affected_services, root_cause: root_cause_hypothesis, correlation_id }
- Store AI confidence and classification metadata

STEP 4 -- Generate Root Cause Analysis:
- Call ai-brain: POST /v1/rca
  Body: { incident_id, anomaly_data, topology_graph, recent_changes[] }
- Returns: { root_cause_summary, contributing_factors[], blast_radius,
  similar_past_incidents[], recommended_actions[] }
- Attach RCA report to incident record

STEP 5 -- Notify and Assign:
- Determine on-call engineer from service ownership mapping
- Push notification to on-call engineer (email + Slack + in-app)
- If severity = critical: page the on-call via PagerDuty/OpsGenie integration
- If severity = critical AND blast_radius > 5 services: escalate to Engineering Manager
- Post to #incidents Slack channel with incident summary card

STEP 6 -- Auto-Remediation Check:
- Query rules engine: GET /v1/rules?type=remediation&service={affected_services}
- If matching auto-remediation rule exists AND rule.enabled = true:
  -- If rule requires approval: create pending remediation action, notify approver
  -- If rule is fully automated: execute remediation, log action
- Publish NATS event: "incident.created" with full metadata

ERROR HANDLING:
- Retry up to 3 times with exponential backoff for API calls
- On AI service failure: proceed with rule-based fallback, flag for human review
- On final failure: create manual incident with all raw data attached
- Log all steps to audit trail via aiops-gateway

ESTIMATED RUN TIME: < 30 seconds
TRIGGER VOLUME: ~200/day (varies with system load)
```

### Prompt M-002: Anomaly-Driven Auto-Remediation Pipeline

```
Create a Make scenario titled "AIOps -- Auto-Remediation Pipeline".

TRIGGER: Webhook from aiops-gateway when a remediation action transitions to
"approved" status (either auto-approved or human-approved).
Payload: { remediation_id, incident_id, action_type, target_service,
  parameters, approved_by, tenant_id }

STEP 1 -- Pre-Execution Validation:
- Call aiops-gateway: GET /v1/remediation/{remediation_id} (full action details)
- Call topology API: GET /v1/topology/{target_service}/dependencies
- Validate: target service is not in maintenance window
- Validate: no other remediation running against same service
- Risk assessment: calculate blast radius from dependency graph
- If risk > threshold: require additional approval, halt pipeline

STEP 2 -- Execute Remediation:
- Route by action_type:
  -- "restart_service": Call infrastructure API: POST /v1/services/{target}/restart
  -- "scale_up": Call infrastructure API: POST /v1/services/{target}/scale
     Body: { replicas: parameters.target_replicas }
  -- "rollback_deploy": Call CI/CD API: POST /v1/deployments/{target}/rollback
     Body: { target_version: parameters.rollback_version }
  -- "clear_cache": Call service endpoint: POST /v1/admin/cache/clear
  -- "circuit_breaker": Call mesh API: POST /v1/circuit-breaker/{target}/open
- Update remediation status to "running"
- Start execution timer

STEP 3 -- Monitor Execution:
- Poll target service health every 10 seconds for up to 5 minutes
- Check: anomaly that triggered incident is resolving (deviation decreasing)
- Check: service health status transitioning to "healthy"
- If timeout reached without improvement: mark as "failed"

STEP 4 -- Post-Execution:
- If successful:
  -- Update remediation status to "completed" with result summary
  -- Update incident: add resolution note, transition to "resolved" if all anomalies clear
  -- Publish NATS event: "remediation.completed"
- If failed:
  -- Update remediation status to "failed" with error details
  -- Escalate to on-call engineer with execution logs
  -- Suggest alternative remediation actions via ai-brain
  -- Publish NATS event: "remediation.failed"

STEP 5 -- Learn and Improve:
- Call ai-brain: POST /v1/feedback
  Body: { remediation_id, success: true/false, execution_time, metrics_before, metrics_after }
- Update remediation rule effectiveness score
- If repeated failures for same rule: auto-disable rule and notify rule owner

ERROR HANDLING:
- Execution failures trigger immediate rollback where possible
- All actions logged with before/after state snapshots
- Circuit breaker: if 3 consecutive failures for a service, pause all auto-remediation
- Human escalation for any infrastructure-level action failure

ESTIMATED RUN TIME: < 5 minutes (including monitoring)
TRIGGER VOLUME: ~30/day
```

### Prompt M-003: Cost Optimization Report Generation

```
Create a Make scenario titled "AIOps -- Weekly Cost Optimization Report".

TRIGGER: Scheduled every Monday at 07:00 UTC.

STEP 1 -- Gather Resource Utilization:
- Call opstrac-api: GET /v1/metrics/resource-utilization?period=7d
  Returns: per-service CPU, memory, disk, network utilization averages and peaks
- Call topology API: GET /v1/topology/services (full service inventory)
- Call cloud provider API: GET /v1/billing/current-period

STEP 2 -- AI-Powered Analysis:
- Call ai-brain: POST /v1/cost/analyze
  Body: { utilization_data, billing_data, service_inventory, reserved_instances }
- Returns: {
    total_waste_identified: dollar amount,
    rightsizing_recommendations: [{ service, current_size, recommended_size, savings }],
    idle_resources: [{ resource, last_active, monthly_cost }],
    reserved_instance_opportunities: [{ service_pattern, on_demand_cost, ri_cost, savings }],
    anomalous_spending: [{ service, expected_cost, actual_cost, deviation }]
  }

STEP 3 -- Generate Cost Report:
- Create CostReport record via Hasura: mutation insert_aiops_cost_reports_one
  Body: { period_start, period_end, total_cost, breakdown, recommendations }
- Generate PDF report with charts and tables
- Store report in document service

STEP 4 -- Distribute Report:
- Email to FinOps team: full report with actionable recommendations
- Email to Engineering Managers: department-specific cost summaries
- Post to #finops Slack channel: top 5 savings opportunities
- Update Cost Dashboard with latest data

STEP 5 -- Track Recommendations:
- Compare current recommendations against previous week
- Identify: new recommendations, resolved recommendations, recurring waste
- Calculate cumulative savings from implemented recommendations
- Update recommendation status based on infrastructure changes

ERROR HANDLING:
- If cloud billing API unavailable: use cached data with "stale data" warning
- Retry failed API calls 3 times
- Send error summary to FinOps team if report generation fails

ESTIMATED RUN TIME: < 2 minutes
TRIGGER VOLUME: 1/week (scheduled)
```

### Prompt M-004: Security Vulnerability Scan and Alert Workflow

```
Create a Make scenario titled "AIOps -- Security Scan & Alert".

TRIGGER: Webhook from opstrac-api when security scan completes.
Payload: { scan_id, scan_type, findings[], tenant_id, scanned_at }

STEP 1 -- Classify Findings:
- For each finding in findings[]:
  -- Assign category (vulnerability, misconfiguration, exposed_secret,
     weak_authentication, network_exposure, compliance_violation,
     data_leakage, privilege_escalation)
  -- Assign severity based on CVSS score and context:
     Critical (CVSS >= 9.0), High (7.0-8.9), Medium (4.0-6.9), Low (< 4.0)
  -- Deduplicate against existing open findings

STEP 2 -- AI Enrichment:
- Call ai-brain: POST /v1/security/enrich
  Body: { findings, service_context, compliance_frameworks }
- Returns: { enriched_findings with remediation_suggestions, priority_ranking,
  compliance_impact_assessment, attack_surface_analysis }

STEP 3 -- Store Findings:
- For each new/updated finding:
  -- Upsert via Hasura: mutation insert_aiops_security_findings
  -- Link to affected topology nodes
  -- Create timeline entry

STEP 4 -- Alert Routing:
- Critical findings: immediate page to Security Analyst on-call
  -- Email + Slack DM + in-app notification with deep link
  -- Auto-create incident if finding indicates active exploitation
- High findings: email Security team + Slack #security-alerts
- Medium findings: daily digest email at 09:00 UTC
- Low findings: weekly summary only

STEP 5 -- Compliance Tracking:
- Map findings to compliance frameworks (SOC2, ISO27001, GDPR, HIPAA)
- Update compliance dashboard scores
- If compliance violation detected:
  -- Notify Compliance Officer immediately
  -- Create compliance ticket with remediation deadline
  -- Track against audit requirements

ERROR HANDLING:
- Failed enrichment: store raw findings, mark for manual review
- Retry scan if partial failure detected
- Alert Security team on scan infrastructure failure

ESTIMATED RUN TIME: < 45 seconds per scan
TRIGGER VOLUME: ~10 scans/day
```

### Prompt M-005: Cross-Module Health Telemetry Aggregation

```
Create a Make scenario titled "AIOps -- Cross-Module Health Aggregation".

TRIGGER: Scheduled every 5 minutes.

STEP 1 -- Collect Telemetry:
- Call OTel Collector: GET /v1/metrics/summary (aggregated from all 20+ ERP modules)
- Modules monitored: IAM, HCM, Finance, CRM, Commerce, SCM, Projects, BI,
  Healthcare, Church-Management, School-Management, BSS-OSS, Marketing,
  Workspace, Platform, DBaaS, iPaaS, AI, Observability, eCommerce, Assistant
- Metrics: request_rate, error_rate, latency_p50/p95/p99, cpu, memory, connections

STEP 2 -- Anomaly Detection:
- Call opstrac-api: POST /v1/detect/batch
  Body: { metrics_batch: [...all module metrics] }
- Returns: { anomalies_detected[], normal_modules[], degraded_modules[] }

STEP 3 -- Update Topology:
- For each module with status change:
  -- Update topology_nodes status via Hasura mutation
  -- Propagate status to dependent services (downstream impact)
  -- Calculate overall platform health score (0-100)

STEP 4 -- Threshold Alerts:
- If any module error_rate > 5%: create warning anomaly
- If any module latency_p99 > 2000ms: create warning anomaly
- If platform health score < 80: notify Engineering Directors
- If platform health score < 60: trigger incident creation workflow (M-001)

STEP 5 -- Dashboard Update:
- Publish real-time metrics to WebSocket channel for Dashboard live update
- Update KPI card values (open incidents, active anomalies)
- Refresh topology graph node statuses

ERROR HANDLING:
- If OTel Collector unreachable: use last known good data, mark as stale
- If individual module unreachable: mark as "unknown" status
- Circuit breaker: if > 50% of health checks fail, alert on infrastructure issue

ESTIMATED RUN TIME: < 30 seconds
TRIGGER VOLUME: 288/day (every 5 minutes)
```

---

## 7. Page-by-Page Figma Prompts

### 7.1 Desktop Pages (1440px)

#### Prompt F-003: AIOps Dashboard -- Command Center (1440px)

```
Design a desktop AIOps Dashboard page at 1440x900px (scrollable) for the ERP-AIOps
module. This is the primary landing page for SREs and Platform Engineers.

LAYOUT:
- Left: Collapsible sidebar navigation (260px, expanded by default, dark theme #001529)
- Top: Horizontal header bar with breadcrumb ("Dashboard"), global search (Cmd+K),
  notification bell (5 unread), user avatar "Abiola Ogunsakin" (SRE Lead)
- Main content area: 1156px wide, 32px padding

CONTENT -- ROW 1 (KPI Cards, 6-column grid matching Dashboard.tsx):
- Card 1: "Open Incidents" -- 7 -- trend: -12.5% (red, fewer is better but still alarming)
  Icon: AlertOutlined | Color: #ef4444
- Card 2: "Active Anomalies" -- 14 -- trend: -8.3%
  Icon: ExperimentOutlined | Color: #f59e0b
- Card 3: "Rules Enabled" -- 12 -- trend: +2 (green, more is better)
  Icon: SettingOutlined | Color: #7c3aed
- Card 4: "Auto-Remediations" -- 8 -- trend: +15% (green)
  Icon: ThunderboltOutlined | Color: #3b82f6
- Card 5: "Cost Savings" -- "$24.5K" -- trend: +22.1% (green)
  Icon: DollarOutlined | Color: #10b981
- Card 6: "Security Findings" -- 5 -- trend: -5.2% (orange, fewer is better)
  Icon: SafetyOutlined | Color: #f97316

CONTENT -- ROW 2 (Two-column, equal width, matching Dashboard.tsx):
- Left: "Recent Incidents" card with AlertOutlined icon in title
  -- "View All" link with ArrowRightOutlined (top right)
  -- Table with columns: Title (linked, font-weight 500), Severity (StatusBadge),
     Status (StatusBadge), Created (relative time, 13px secondary)
  -- 5 rows of realistic data:
     | "Memory leak in ERP-Commerce checkout service" | Critical | Open | 12 min ago |
     | "High latency on ERP-Finance payment gateway" | High | Investigating | 35 min ago |
     | "Disk usage exceeding 90% on ERP-HCM DB replica" | Medium | Acknowledged | 1 hour ago |
     | "SSL certificate expiring on ERP-IAM auth endpoint" | Low | Open | 3 hours ago |
     | "Unusual login pattern detected on ERP-CRM admin" | High | Resolved | 5 hours ago |
  -- Empty state: "No incidents recorded"

- Right: "Recent Anomalies" card with ExperimentOutlined icon in title
  -- "View All" link with ArrowRightOutlined (top right)
  -- Table with columns: Metric (font-weight 500), Service (13px), Severity (StatusBadge),
     Detected (relative time, 13px secondary)
  -- 5 rows of realistic data:
     | "api_latency_p99" | erp-finance-gateway | Critical | 8 min ago |
     | "error_rate_5xx" | erp-commerce-api | High | 22 min ago |
     | "cpu_utilization" | erp-hcm-worker | Medium | 45 min ago |
     | "memory_heap_used" | erp-crm-api | Medium | 1 hour ago |
     | "db_connection_pool" | erp-scm-inventory | Low | 2 hours ago |
  -- Empty state: "No anomalies detected"

STATES:
- Loading: Skeleton shimmer on all KPI cards and tables (as per PageLoader component)
- Empty: "Welcome to AIOps! Connect your first service to start monitoring." with CTA button
- Error: Inline error card with retry for failed data fetches
- Real-time: Green pulse dot on header indicating live WebSocket connection

Include both light theme (default) and dark theme toggle in the header.
Annotate all interactive elements with click targets >= 44px.
```

#### Prompt F-004: Incidents List Page (1440px)

```
Design a desktop Incidents List page at 1440x900px for the incidents resource.
Target user: SRE / Platform Engineer triaging incidents.

HEADER:
- Breadcrumb: "Dashboard" > "Incidents"
- Title: "Incidents"
- "Create Incident" primary button (purple, #7c3aed) with PlusOutlined icon
- Global search bar

FILTER BAR:
- Severity: dropdown multi-select (Critical, High, Medium, Low, Info) with colored dots
- Status: dropdown multi-select (Open, Acknowledged, Investigating, Resolved, Closed)
- Source: dropdown (All, AI Detection, Manual, Alert Rule, External)
- Date Range: date picker (Last 24h, Last 7d, Last 30d, Custom)
- Affected Service: searchable dropdown populated from topology_nodes
- Active filters shown as dismissible chips with count

INCIDENTS TABLE:
| Severity | Title | Status | Source | Affected Services | Root Cause | Created | Actions |
|----------|-------|--------|--------|-------------------|------------|---------|---------|
| Critical (red dot) | Memory leak in ERP-Commerce checkout service | Open (red badge) | AI Detection | commerce-api, commerce-worker | AI: Memory allocation in cart module | 12 min ago | View, Ack |
| High (orange dot) | High latency on ERP-Finance payment gateway | Investigating (blue badge) | Alert Rule | finance-gateway, finance-api | Investigating... | 35 min ago | View |
| Medium (amber dot) | Disk usage exceeding 90% on ERP-HCM DB replica | Acknowledged (amber badge) | AI Detection | hcm-db-replica-01 | Disk: log rotation misconfigured | 1 hour ago | View |
| High (orange dot) | Unusual login pattern on ERP-CRM admin panel | Resolved (green badge) | AI Detection | crm-auth, iam-gateway | Resolved: legitimate migration script | 5 hours ago | View |
| Low (blue dot) | SSL certificate expiring on ERP-IAM auth endpoint | Open (red badge) | Cert Monitor | iam-auth-endpoint | Certificate renewal required | 3 hours ago | View, Ack |

- Sortable columns (click header to sort)
- Row hover: subtle background change, action icons appear
- Row click: navigates to /incidents/:id
- Pagination: "Showing 1-25 of 127 incidents" with page selector
- Bulk action bar (appears when rows selected): "3 selected -- Acknowledge All | Assign | Close"

SIDE PANEL (Quick View -- opens on eye icon click, 480px slide-in):
- Incident title and severity badge
- Status timeline: Open -> Acknowledged -> Investigating -> Resolved
- AI-generated summary (labeled "AI Summary" with confidence badge)
- Affected services list with topology status indicators
- Root cause (if available)
- "Open Full View" button to navigate to detail page
- Quick actions: Acknowledge, Assign, Add Note

EMPTY STATE:
- Illustration of calm monitoring dashboard
- "No incidents found" heading
- "All systems operational -- or try adjusting your filters" subtext
- "Clear Filters" button

Both light and dark themes. All interactive elements annotated >= 44px touch targets.
```

#### Prompt F-005: Incident Detail Page (1440px)

```
Design a desktop Incident Detail page at 1440x900px for /incidents/:id.
Target user: SRE investigating and resolving an incident.

HEADER:
- Breadcrumb: "Dashboard" > "Incidents" > "INC-2026-0847"
- Title: "Memory leak in ERP-Commerce checkout service"
- Severity badge: Critical (red)
- Status badge: Investigating (blue) with status transition dropdown
- Actions: "Acknowledge" (if open), "Assign" dropdown, "Escalate" button,
  "Close Incident" button (requires confirmation dialog)
- Time in current status: "Investigating for 23 min"
- Total duration: "Created 58 min ago"

LAYOUT: Two-column (65% / 35%)

LEFT COLUMN (65%):

SECTION 1 -- "Incident Summary":
- Description: Full AI-generated incident description (clearly labeled as AI-generated
  with confidence: 94% badge)
- Source: "AI Detection" badge
- Correlation ID: "corr-a7b3c9e2-4f1d" (mono font, click to copy)
- Affected Services: Tag pills for each service (commerce-api, commerce-worker)
  -- Each tag links to /topology with service highlighted

SECTION 2 -- "Root Cause Analysis" (AI-powered):
- AI confidence indicator: 87% confidence bar
- Root cause summary: "Memory allocation issue in the cart module's session handler.
  The checkout service is not releasing cart session objects after payment completion,
  leading to unbounded heap growth."
- Contributing factors (bulleted list):
  -- "Deploy of commerce-api v2.14.3 introduced new session pooling (2 hours ago)"
  -- "Cart traffic increased 40% due to flash sale event"
  -- "Garbage collector tuning params reset in last config push"
- Similar past incidents: links to 2 previous incidents
- "Regenerate RCA" button (calls ai-brain)

SECTION 3 -- "Metrics & Evidence":
- Time-series chart: Memory usage over last 2 hours (line chart with anomaly
  region highlighted in red)
  -- Expected baseline (dashed gray line)
  -- Actual values (solid purple line)
  -- Anomaly detection threshold (red dashed line)
- Error rate chart: 5xx errors over last 2 hours
- Request latency chart: P50, P95, P99 lines

SECTION 4 -- "Remediation Actions":
- List of associated remediation actions with status
  | Action | Target | Status | Initiated | Duration |
  | Restart service | commerce-api | Completed | 15 min ago | 45s |
  | Scale up replicas | commerce-worker | Running | 5 min ago | -- |
  | Rollback deploy | commerce-api v2.14.3 | Pending Approval | Now | -- |
- "Create Remediation" button

RIGHT COLUMN (35%):

SECTION A -- "Timeline":
- Vertical timeline with dot markers, timestamps, actor avatars:
  -- "Incident detected by AI anomaly correlation" -- 58 min ago (system)
  -- "Incident created with severity: Critical" -- 58 min ago (system)
  -- "Notification sent to on-call: Abiola Ogunsakin" -- 57 min ago (system)
  -- "Acknowledged by Abiola Ogunsakin" -- 45 min ago (user)
  -- "Status changed to Investigating" -- 45 min ago (user)
  -- "RCA generated by AI Brain (confidence: 87%)" -- 44 min ago (AI)
  -- "Remediation: Restart commerce-api initiated" -- 30 min ago (user)
  -- "Remediation: Restart completed successfully" -- 29 min ago (system)
  -- "Note: 'Restart didn't fully resolve. Scaling up.'" -- 20 min ago (user)

SECTION B -- "Blast Radius":
- Mini topology graph showing affected service and its immediate dependencies
  -- commerce-api (red/down), commerce-worker (amber/degraded),
     commerce-db (green/healthy), payment-gateway (amber/degraded)
- "View Full Topology" link

SECTION C -- "Assignment":
- Current assignee: avatar + "Abiola Ogunsakin" + "SRE Lead"
- "Reassign" dropdown
- On-call schedule: current and next on-call
- Escalation chain: L1 -> L2 -> Manager

SECTION D -- "Related":
- Linked anomalies (3): clickable links to /anomalies/:id
- Similar past incidents (2): clickable links
- Affected ERP modules: Commerce, Finance (impacted by downstream dependency)

STATES:
- Loading: skeleton for all sections
- Resolving: success animation with green check when status changes to "resolved"
- Closed: grayed out actions, "Reopen" button available

Both light and dark themes. Mobile-responsive annotations.
```

#### Prompt F-006: Incident Create Page (1440px)

```
Design a desktop Incident Create page at 1440x900px for /incidents/create.
Target user: SRE manually creating an incident (not AI-detected).

HEADER:
- Breadcrumb: "Dashboard" > "Incidents" > "Create Incident"
- Title: "Create Incident"

FORM LAYOUT (centered, max 720px wide):

SECTION 1 -- "Incident Details":
- Title*: Text input (required), placeholder "Brief description of the issue"
- Description: Rich text area (4 lines), placeholder "Detailed description..."
- Severity*: Radio group with colored indicators
  -- Critical (red dot) | High (orange dot) | Medium (amber dot) | Low (blue dot) | Info (gray dot)
  -- Helper text: "Critical: customer-facing service down; High: significant degradation..."

SECTION 2 -- "Classification":
- Source: Dropdown (Manual, Alert Rule, External Report, Customer Report)
- Affected Services*: Multi-select tag input with autocomplete from topology_nodes
  -- Shows service name + status dot, grouped by module
  -- Example: "erp-commerce-api (healthy)" | "erp-finance-gateway (healthy)"
- Correlation ID: Text input (optional, mono font), placeholder "Link to existing correlation"

SECTION 3 -- "Initial Assessment":
- Root Cause (optional): Textarea, placeholder "Initial hypothesis..."
- "Generate AI Assessment" button -- calls ai-brain for automated analysis
  -- When clicked: shows loading spinner, then populates root cause and suggests
     severity based on affected services
  -- AI output clearly labeled with confidence score

SECTION 4 -- "Assignment":
- Assign To: Searchable user dropdown with avatar + name + role
  -- Default: current on-call for primary affected service
- Notification: Checkbox group
  -- [ ] Notify assigned engineer
  -- [ ] Post to #incidents channel
  -- [ ] Page on-call (critical/high only)

FORM ACTIONS (sticky bottom bar):
- "Create Incident" primary button (purple)
- "Save as Draft" secondary button
- "Cancel" text link
- Unsaved changes warning on navigation

VALIDATION:
- Inline validation on required fields
- Severity auto-suggested when affected services selected (AI-powered)
- Duplicate detection: warning if similar title exists in last 24 hours

STATES:
- Default: empty form with placeholders
- Filling: fields populated with validation feedback
- Submitting: loading overlay with "Creating incident..."
- Success: redirect to incident detail with success toast
- Error: inline error messages on failed fields, banner error for API failures

Both light and dark themes.
```

#### Prompt F-007: Anomalies List Page (1440px)

```
Design a desktop Anomalies List page at 1440x900px for the anomalies resource.
Target user: SRE / DevOps Engineer monitoring system anomalies.

HEADER:
- Breadcrumb: "Dashboard" > "Anomalies"
- Title: "Anomalies"
- Subtitle: "ML-detected anomalies across all monitored services"
- View toggle: Table (default) | Timeline | Heatmap

FILTER BAR:
- Severity: multi-select (Critical, High, Medium, Low)
- Status: multi-select (Active, Investigating, Resolved, Dismissed)
- Service: searchable dropdown (from topology_nodes)
- Module: dropdown (all ERP modules: Finance, Commerce, HCM, IAM, etc.)
- Anomaly Type: dropdown (spike, drop, trend_change, pattern_break, seasonal_deviation)
- Time Range: Last 1h, Last 6h, Last 24h, Last 7d, Custom
- Active filters as dismissible chips

TABLE VIEW (default):
| Severity | Metric Name | Service | Module | Type | Expected | Actual | Deviation | Status | Detected | Actions |
|----------|-------------|---------|--------|------|----------|--------|-----------|--------|----------|---------|
| Critical | api_latency_p99 | erp-finance-gateway | Finance | spike | 45ms | 1,240ms | +2,655% | Active | 8 min ago | View, Link to Incident |
| High | error_rate_5xx | erp-commerce-api | Commerce | spike | 0.1% | 4.7% | +4,600% | Investigating | 22 min ago | View |
| Medium | cpu_utilization | erp-hcm-worker | HCM | trend_change | 35% | 78% | +122% | Active | 45 min ago | View, Dismiss |
| Medium | memory_heap_used | erp-crm-api | CRM | spike | 512MB | 890MB | +73% | Investigating | 1 hour ago | View |
| Low | db_connection_pool | erp-scm-inventory | SCM | trend_change | 40 | 62 | +55% | Active | 2 hours ago | View, Dismiss |

- Metric Name in mono font for values (expected, actual, deviation)
- Deviation colored by severity (red >500%, orange >100%, amber >50%, blue <50%)
- Row click navigates to /anomalies/:id
- Row hover: action icons appear

TIMELINE VIEW (alternative):
- Vertical timeline grouped by hour
- Each anomaly card on the timeline: severity indicator + metric + service + deviation
- Clustering: group related anomalies on same service
- Time ruler on left with current time at top

HEATMAP VIEW (alternative):
- X-axis: time (last 24 hours, 1-hour buckets)
- Y-axis: services (grouped by module)
- Cell color: intensity by anomaly count/severity in that time window
  -- Green: 0 anomalies, Amber: 1-2 low/medium, Red: any critical or 3+ anomalies
- Hover shows anomaly count and service details

PAGINATION: "Showing 1-25 of 89 anomalies" with page selector

EMPTY STATE:
- Illustration of green signal waves
- "No anomalies detected" heading
- "All metrics within expected ranges" subtext
- "Configure Detection Rules" link to /rules

Both light and dark themes. Real-time update indicator (green pulse dot).
```

#### Prompt F-008: Anomaly Detail Page (1440px)

```
Design a desktop Anomaly Detail page at 1440x900px for /anomalies/:id.
Target user: SRE investigating a specific anomaly.

HEADER:
- Breadcrumb: "Dashboard" > "Anomalies" > "ANO-2026-1492"
- Title: "api_latency_p99" (metric name in mono font)
- Subtitle: "erp-finance-gateway | Finance module"
- Severity badge: Critical (red) | Status badge: Active (red)
- Actions: "Investigate" (transitions to investigating), "Create Incident" button,
  "Dismiss" button (requires confirmation + reason), "Link to Incident" dropdown

LAYOUT: Two-column (70% / 30%)

LEFT COLUMN (70%):

SECTION 1 -- "Anomaly Overview":
- Anomaly type badge: "spike" (with icon)
- Detection time: "Detected 8 minutes ago (2026-02-24 14:52:00 UTC)"
- Duration: "Active for 8 minutes"
- Key metrics row:
  -- Expected Value: 45ms (mono, gray)
  -- Actual Value: 1,240ms (mono, red, large)
  -- Deviation: +2,655% (mono, red, bold)

SECTION 2 -- "Metric Visualization":
- Time-series chart (full width, 300px tall):
  -- X-axis: last 6 hours with 5-minute intervals
  -- Y-axis: latency in milliseconds
  -- Solid purple line: actual metric values
  -- Dashed gray line: expected baseline (from ML model)
  -- Light purple band: normal range (2-sigma)
  -- Red highlighted region: anomaly detection window
  -- Vertical marker: detection timestamp
  -- Tooltip on hover showing exact values
- Chart controls: zoom (1h, 6h, 24h, 7d), auto-refresh toggle

SECTION 3 -- "AI Analysis" (from ai-brain):
- AI confidence: 96% badge
- Summary: "Significant latency spike detected on the Finance payment gateway.
  The P99 latency increased from a baseline of 45ms to 1,240ms, a 27x increase.
  This correlates with a simultaneous memory pressure event on the same host."
- Probable causes (ranked by confidence):
  1. "Memory pressure on host fin-prod-03 (confidence: 89%)"
  2. "Increased database query latency from connection pool saturation (confidence: 72%)"
  3. "Network congestion between availability zones (confidence: 34%)"
- Related metrics that also show anomalous behavior:
  -- cpu_utilization (erp-finance-gateway): 92% (normally 35%)
  -- db_query_time_p99 (erp-finance-db): 450ms (normally 12ms)

SECTION 4 -- "Correlated Events":
- Timeline of events around detection time:
  -- "-15 min: Deploy erp-finance-api v3.2.1 (from CI/CD)"
  -- "-10 min: Config change on fin-prod-03 JVM heap settings"
  -- "-5 min: Memory utilization crossed 85% threshold"
  -- "0 min: Anomaly detected by ML model"
  -- "+2 min: First 5xx errors observed"

RIGHT COLUMN (30%):

SECTION A -- "Metadata":
- Module: Finance
- Service: erp-finance-gateway
- Anomaly Type: Spike
- ML Model: LSTM-based seasonal decomposition
- Confidence: 96%
- Metadata JSON viewer (expandable)

SECTION B -- "Linked Incidents":
- List of incidents that reference this anomaly
- "INC-2026-0847: High latency on ERP-Finance payment gateway" (linked)
- "Create New Incident" button

SECTION C -- "Service Context" (from topology):
- Mini card: erp-finance-gateway
  -- Status: Degraded (amber)
  -- Dependencies: erp-finance-db, erp-finance-cache, payment-provider-api
  -- Dependents: erp-commerce-checkout, erp-bi-pipeline
- "View in Topology" link

SECTION D -- "History":
- Previous anomalies on same metric/service (last 30 days)
  -- "Feb 18: +340% spike, resolved in 12 min (auto-remediation)"
  -- "Feb 5: +180% trend change, dismissed (expected load test)"
- Frequency trend: "2 anomalies in last 30 days (normal: <1/month)"

Both light and dark themes. Auto-refresh every 30s for active anomalies.
```

#### Prompt F-009: Rules List Page (1440px)

```
Design a desktop Rules List page at 1440x900px for the rules resource.
Target user: DevOps Engineer managing AIOps correlation and threshold rules.

HEADER:
- Breadcrumb: "Dashboard" > "Rules"
- Title: "Rules"
- Subtitle: "Correlation rules, threshold rules, and pattern matching"
- "Create Rule" primary button (purple) with PlusOutlined icon

FILTER BAR:
- Type: dropdown (All, Correlation, Threshold, Pattern, Remediation)
- Status: toggle (All | Enabled | Disabled)
- Priority: range slider (1-10)
- Search: text input for name/description search

RULES TABLE:
| Enabled | Name | Type | Priority | Condition Summary | Action Summary | Created | Updated | Actions |
|---------|------|------|----------|-------------------|----------------|---------|---------|---------|
| [Toggle ON] | High latency auto-scale | Threshold | 8 | latency_p99 > 500ms for 5min | Scale up target service | Jan 15 | Feb 20 | Edit, Clone, Delete |
| [Toggle ON] | Error spike incident creation | Threshold | 9 | error_rate > 5% for 3min | Create critical incident | Jan 10 | Feb 18 | Edit, Clone, Delete |
| [Toggle ON] | Memory + CPU correlation | Correlation | 7 | memory > 85% AND cpu > 80% on same host | Create high incident + alert | Feb 1 | Feb 22 | Edit, Clone, Delete |
| [Toggle OFF] | Weekend traffic pattern | Pattern | 4 | Seasonal drop > 50% on Sat-Sun | Suppress alerts | Dec 5 | Jan 30 | Edit, Clone, Delete |
| [Toggle ON] | Cert expiry pre-alert | Threshold | 6 | ssl_cert_days_remaining < 30 | Create low incident + notify | Nov 20 | Feb 15 | Edit, Clone, Delete |

- Enabled toggle: inline switch with confirmation dialog when disabling
  -- "Disabling this rule will stop automatic detection. Continue?"
- Type badges: colored by type (Correlation=#7c3aed, Threshold=#3b82f6,
  Pattern=#10b981, Remediation=#f97316)
- Condition/Action: truncated JSON preview with expand on hover
- Priority: numbered badge with color scale (9-10 red, 7-8 orange, 5-6 amber, 1-4 blue)
- Actions: icon buttons (edit pencil, clone copy, delete trash with confirmation)
- Row click: navigates to rule detail/edit page

STATISTICS BAR (above table):
- Total Rules: 24 | Enabled: 18 | Disabled: 6
- Rules triggered today: 47 | False positive rate: 3.2%

EMPTY STATE:
- "No rules configured" heading
- "Create your first AIOps rule to automate incident detection and response" subtext
- "Create Rule" CTA button
- "Import Rules" secondary link (bulk import from JSON/YAML)

Both light and dark themes.
```

#### Prompt F-010: Rule Create/Edit Page (1440px)

```
Design a desktop Rule Create page at 1440x900px for /rules/create.
Target user: DevOps Engineer building an AIOps rule.

HEADER:
- Breadcrumb: "Dashboard" > "Rules" > "Create Rule"
- Title: "Create Rule"

FORM LAYOUT (centered, max 800px wide):

SECTION 1 -- "Rule Basics":
- Name*: Text input, placeholder "e.g., High latency auto-scale"
- Description: Textarea (3 lines), placeholder "What this rule detects and how it responds"
- Type*: Segmented control (Correlation | Threshold | Pattern | Remediation)
  -- Selection changes the condition editor below
- Priority*: Slider (1-10) with labeled ticks
  -- 1-3: Low (blue) | 4-6: Medium (amber) | 7-8: High (orange) | 9-10: Critical (red)
- Enabled: Toggle switch (default ON)

SECTION 2 -- "Condition" (varies by type):

FOR THRESHOLD TYPE:
- Metric*: Searchable dropdown (api_latency_p99, error_rate_5xx, cpu_utilization, etc.)
- Operator*: Dropdown (>, <, >=, <=, ==, !=)
- Value*: Number input with unit selector (ms, %, req/s, MB, connections)
- Duration*: Number input + unit (seconds, minutes, hours)
  -- "Metric must breach threshold for this duration before triggering"
- Service scope: Multi-select (specific services or "All services")

FOR CORRELATION TYPE:
- Condition A: Metric + Operator + Value (same as threshold)
- Boolean: AND / OR dropdown
- Condition B: Metric + Operator + Value
- Scope: "Same host" / "Same service" / "Same module" / "Any"
- Time window: "Within X minutes"

FOR PATTERN TYPE:
- Pattern type: Dropdown (seasonal_drop, seasonal_spike, trend_reversal, flatline)
- Sensitivity: Slider (Low / Medium / High)
- Baseline period: Dropdown (1 week, 2 weeks, 1 month, 3 months)

ADVANCED CONDITION (collapsible):
- JSON editor with syntax highlighting for complex conditions
- Schema validation with inline error markers
- "Validate" button to test condition syntax
- Example template links: "Load threshold example" | "Load correlation example"

SECTION 3 -- "Action":
- Action type*: Dropdown:
  -- Create Incident (with severity selector)
  -- Send Alert (with channel selector: email, Slack, PagerDuty)
  -- Execute Remediation (with runbook selector)
  -- Suppress Alerts (with duration)
  -- Custom Webhook (with URL and payload template)
- Action JSON editor (for advanced configuration)
- "Test Action" button (dry run mode)

SECTION 4 -- "Preview & Test":
- Rule summary card showing human-readable interpretation:
  "When api_latency_p99 exceeds 500ms for more than 5 minutes on any Finance service,
   create a High severity incident and send an alert to #finance-ops Slack channel."
- "Test Against Last 24h" button -- runs rule retroactively and shows matches
- Estimated trigger frequency based on historical data

FORM ACTIONS (sticky bottom bar):
- "Create Rule" primary button (purple)
- "Save as Draft" secondary
- "Cancel" text link

Both light and dark themes. Inline validation on all required fields.
```

#### Prompt F-011: Topology View Page (1440px)

```
Design a desktop Topology View page at 1440x900px for the /topology route.
Target user: SRE / Platform Engineer understanding service dependencies.

HEADER:
- Breadcrumb: "Dashboard" > "Topology"
- Title: "Service Topology"
- Subtitle: "Interactive dependency map across all ERP modules"
- Controls: Zoom In (+), Zoom Out (-), Fit to Screen, Reset Layout
- Filter: Module dropdown (All, Finance, Commerce, HCM, IAM, etc.)
- Search: "Find service..." search bar (highlights matching node)
- Layout toggle: Force-directed (default) | Hierarchical | Grid

MAIN CANVAS (full width, 700px tall):
- Interactive graph rendered with WebGL (via d3-force or similar)
- Nodes: circular (48px) with service icon and status ring
  -- Each node colored by status: healthy=#10b981, degraded=#f59e0b,
     down=#ef4444, unknown=#94a3b8
  -- Node size proportional to dependency count
  -- Label below node: service name (truncated at 16 chars)
  -- Module grouping: nodes clustered by ERP module with subtle background regions

EXAMPLE NODES (showing realistic topology):
- erp-finance-gateway (healthy) -> erp-finance-api (degraded) -> erp-finance-db (healthy)
- erp-finance-api -> payment-provider-api (healthy)
- erp-commerce-api (down) -> erp-commerce-db (healthy)
- erp-commerce-api -> erp-finance-gateway (healthy) [cross-module dependency]
- erp-hcm-gateway (healthy) -> erp-hcm-api (healthy) -> erp-hcm-db (healthy)
- erp-iam-gateway (healthy) -> erp-iam-auth (healthy) [core service, many dependents]
- Show 15-20 total nodes across 5-6 modules

EDGE TYPES:
- Solid line: healthy dependency
- Dashed line: degraded dependency
- Dotted red line: broken dependency
- Arrow direction shows dependency flow (caller -> callee)
- Edge thickness proportional to request volume

INTERACTION:
- Hover on node: enlarge to 64px, show tooltip with:
  -- Service name, module, status, type (API, database, cache, queue, external)
  -- Dependencies count, dependents count
  -- Current metrics: latency, error rate, request rate
- Click on node: opens detail panel (right side, 360px)
  -- Full service card: name, type, module, status, metadata
  -- Dependencies list (outgoing edges)
  -- Dependents list (incoming edges)
  -- Recent anomalies for this service (last 3)
  -- Recent incidents for this service (last 3)
  -- "View Anomalies" and "View Incidents" links (filtered to this service)
- Click on edge: show dependency details (latency between services, request rate)
- Drag nodes to rearrange layout
- Double-click node: zoom to service and show 2-hop neighborhood

LEGEND (bottom-left overlay):
- Node status colors: Healthy, Degraded, Down, Unknown
- Edge types: Healthy, Degraded, Broken
- Module color coding

HEALTH SUMMARY BAR (below canvas):
- "34 services | 28 healthy (82%) | 4 degraded (12%) | 1 down (3%) | 1 unknown (3%)"
- Module breakdown: Finance: 2/6 degraded | Commerce: 1/4 down | All others healthy

STATES:
- Loading: animated graph skeleton with placeholder nodes
- Empty: "No topology data available. Configure services to see dependencies."
- Error: "Failed to load topology data" with retry button
- Real-time: nodes pulse briefly when status changes (WebSocket updates)

Both light and dark themes. Canvas supports pinch-to-zoom on trackpad.
```

#### Prompt F-012: Remediation List Page (1440px)

```
Design a desktop Remediation List page at 1440x900px for the remediation_actions resource.
Target user: SRE / DevOps Engineer managing auto-remediation actions.

HEADER:
- Breadcrumb: "Dashboard" > "Remediation"
- Title: "Remediation Actions"
- Subtitle: "Automated and manual remediation execution log"
- "Execute Runbook" primary button (triggers manual remediation selection)

FILTER BAR:
- Status: multi-select (Pending, Running, Completed, Failed, Cancelled)
- Action Type: dropdown (All, Restart, Scale, Rollback, Cache Clear, Circuit Breaker, Custom)
- Target Service: searchable dropdown
- Time Range: Last 24h, Last 7d, Last 30d, Custom
- Initiated By: User dropdown (or "Automated")

STATISTICS BAR:
- Total Executions (24h): 12 | Success Rate: 83% | Avg Duration: 2m 15s
- Pending Approvals: 2 | Currently Running: 1

REMEDIATION TABLE:
| Status | Action Type | Target Service | Incident | Initiated By | Started | Duration | Result | Actions |
|--------|-------------|----------------|----------|-------------|---------|----------|--------|---------|
| Completed (green) | Restart Service | erp-commerce-api | INC-0847 | Abiola (manual) | 30 min ago | 45s | Success: service healthy | View Log |
| Running (blue, pulse) | Scale Up (2->4) | erp-commerce-worker | INC-0847 | Auto-rule #12 | 5 min ago | Running... | -- | View, Cancel |
| Pending (amber) | Rollback Deploy | erp-commerce-api | INC-0847 | Auto-rule #8 | Now | Awaiting approval | -- | Approve, Reject |
| Failed (red) | Clear Cache | erp-crm-cache | INC-0831 | Chidi (manual) | 2 hours ago | 12s | Error: connection refused | View, Retry |
| Completed (green) | Circuit Breaker | erp-finance-gateway | INC-0839 | Auto-rule #5 | 6 hours ago | 3m 20s | Success: circuit opened | View Log |
| Cancelled (gray) | Scale Down | erp-hcm-worker | -- | Auto-rule #15 | 8 hours ago | -- | Cancelled by Ngozi | View |

- Running rows: animated progress indicator
- Pending approval rows: highlighted with amber border, prominent Approve/Reject buttons
- Failed rows: red left border, retry icon button
- Result column: expandable to show full JSON result
- Incident column: clickable link to incident detail

EXPANDABLE ROW DETAIL:
- Parameters JSON viewer (formatted, syntax highlighted)
- Execution timeline: Initiated -> Approved -> Started -> [Monitoring...] -> Completed
- Before/after metrics comparison (mini charts)
- Audit log: who approved, who initiated, all state transitions

PENDING APPROVAL PANEL (if pending items exist, shown above table):
- Yellow banner: "2 remediation actions awaiting your approval"
- Quick approval cards:
  -- "Rollback erp-commerce-api to v2.14.2" | Risk: Medium | "Approve" | "Reject"
  -- "Scale down erp-hcm-worker from 4 to 2 replicas" | Risk: Low | "Approve" | "Reject"

EMPTY STATE:
- "No remediation actions recorded"
- "Configure auto-remediation rules in the Rules section" with link to /rules

Both light and dark themes.
```

#### Prompt F-013: Cost Dashboard Page (1440px)

```
Design a desktop Cost Dashboard page at 1440x900px for the /cost route.
Target user: FinOps Analyst / Engineering Manager reviewing cost optimization.

HEADER:
- Breadcrumb: "Dashboard" > "Cost Optimization"
- Title: "Cost Optimization"
- Subtitle: "Resource waste detection, rightsizing, and savings tracking"
- Period selector: "Last 7 days" | "Last 30 days" | "Last 90 days" | Custom range
- "Generate Report" button | "Export CSV" button

CONTENT -- ROW 1 (4 KPI cards):
- Card 1: "Monthly Spend" -- $128,450 -- trend: +4.2% vs last month (amber)
- Card 2: "Identified Waste" -- $18,200 -- 14.2% of total spend (red)
- Card 3: "Savings Implemented" -- $24,500 -- this quarter (green)
- Card 4: "Optimization Score" -- 78/100 -- up from 71 last month (green arrow)

CONTENT -- ROW 2 (Two-column, 60/40):
- Left (60%): "Cost Trend" -- Area chart showing daily spend over selected period
  -- Stacked by module: Finance (blue), Commerce (purple), HCM (green),
     IAM (amber), Other (gray)
  -- Forecast dashed line for next 7 days
  -- Anomalous spending days highlighted with red marker
  -- Data: realistic daily costs ranging $3,800-$5,200

- Right (40%): "Cost By Module" -- Donut chart with legend
  -- Finance: $34,200 (26.6%)
  -- Commerce: $28,100 (21.9%)
  -- HCM: $19,800 (15.4%)
  -- IAM: $12,400 (9.7%)
  -- Observability: $10,300 (8.0%)
  -- Other: $23,650 (18.4%)

CONTENT -- ROW 3 (Full width): "Optimization Recommendations"
- Sortable by: Savings (default, highest first) | Confidence | Impact
- Recommendation cards (list layout):

  Card 1: RIGHTSIZING (purple badge)
  "Rightsize erp-finance-api from 4 vCPU / 16GB to 2 vCPU / 8GB"
  Savings: $450/month | Confidence: 92% (High) | Impact: Low
  Rationale: "Average CPU utilization is 18%, peak is 42%. Memory utilization averages 31%."
  [Apply] [Schedule] [Dismiss] buttons

  Card 2: IDLE RESOURCE (red badge)
  "Terminate idle staging environment erp-commerce-staging-02"
  Savings: $680/month | Confidence: 98% (High) | Impact: None
  Rationale: "No traffic in 45 days. Last deployment: Jan 10, 2026."
  [Apply] [Schedule] [Dismiss] buttons

  Card 3: RESERVED INSTANCE (blue badge)
  "Purchase 1-year reserved instance for erp-hcm-db (PostgreSQL r6g.xlarge)"
  Savings: $320/month (38% discount) | Confidence: 87% (High) | Impact: Low
  Rationale: "Consistent utilization >80% for 6+ months. Stable workload pattern."
  [Apply] [Schedule] [Dismiss] buttons

  Card 4: WASTE DETECTION (orange badge)
  "Remove unused load balancer for deprecated erp-legacy-api"
  Savings: $45/month | Confidence: 100% (High) | Impact: None
  Rationale: "Load balancer has 0 target instances and 0 requests for 90 days."
  [Apply] [Dismiss] buttons

CONTENT -- ROW 4 (Full width): "Cost Reports History"
- Table: | Period | Total Cost | Savings Identified | Savings Implemented | Optimization Score |
  -- Feb 2026: $128,450 | $18,200 | $8,400 | 78/100
  -- Jan 2026: $123,200 | $22,100 | $16,100 | 71/100
  -- Dec 2025: $118,900 | $19,800 | $12,300 | 65/100
- Each row expandable to show full report breakdown

STATES:
- Loading: skeleton on all cards and charts
- Empty: "Connect cloud billing data to start cost optimization"
- Error: retry banner

Both light and dark themes.
```

#### Prompt F-014: Security Dashboard Page (1440px)

```
Design a desktop Security Dashboard page at 1440x900px for the /security route.
Target user: Security Analyst reviewing vulnerability and compliance status.

HEADER:
- Breadcrumb: "Dashboard" > "Security"
- Title: "Security Dashboard"
- Subtitle: "Vulnerability detection, compliance checks, and threat analysis"
- "Run Scan" primary button | "Export Report" button
- Last scan: "2 hours ago" with refresh icon

CONTENT -- ROW 1 (4 KPI cards):
- Card 1: "Open Findings" -- 23 -- trend: -8% (improving, green arrow)
  Breakdown mini-bar: 3 critical, 7 high, 9 medium, 4 low
- Card 2: "Critical/High" -- 10 -- requires immediate attention (red)
- Card 3: "Compliance Score" -- 87% -- SOC2 + ISO27001 combined (amber)
- Card 4: "Avg Resolution Time" -- 4.2 days -- trend: -15% (improving, green)

CONTENT -- ROW 2 (Two-column, 50/50):
- Left: "Findings by Category" -- Horizontal bar chart
  -- Vulnerability: 8 (red bar)
  -- Misconfiguration: 5 (orange bar)
  -- Weak Authentication: 4 (amber bar)
  -- Network Exposure: 3 (blue bar)
  -- Compliance Violation: 2 (purple bar)
  -- Exposed Secret: 1 (dark red bar)
  -- Data Leakage: 0 (gray bar, empty)

- Right: "Findings by Severity Over Time" -- Stacked area chart (last 30 days)
  -- Critical (red), High (orange), Medium (amber), Low (blue)
  -- Shows trend of findings count over time (ideally decreasing)

CONTENT -- ROW 3 (Full width): "Security Findings"
- Filter chips: All | Critical | High | Medium | Low
- Category filter: All categories dropdown
- Status filter: Open | In Progress | Resolved | Dismissed

FINDINGS TABLE:
| Severity | Title | Category | Affected Resource | Status | Remediation | Detected | Actions |
|----------|-------|----------|-------------------|--------|-------------|----------|---------|
| Critical (red) | SQL Injection in query parameter handler | Vulnerability | erp-commerce-api /v1/products | Open | "Parameterize SQL queries in product search endpoint" | 1 day ago | View, Assign, Resolve |
| Critical (red) | Exposed AWS credentials in environment config | Exposed Secret | erp-finance-api /config/env | Open | "Rotate credentials immediately, move to Vault" | 2 hours ago | View, Assign |
| High (orange) | TLS 1.0 enabled on legacy endpoint | Misconfiguration | erp-hcm-legacy-api :443 | In Progress | "Disable TLS 1.0/1.1, enforce TLS 1.2+" | 3 days ago | View |
| High (orange) | Default admin password not changed | Weak Auth | erp-scm-admin-panel | Open | "Enforce password change on first login" | 5 days ago | View, Assign |
| Medium (amber) | Unrestricted CORS policy | Network Exposure | erp-crm-api /v1/* | Open | "Restrict CORS to known origins" | 1 week ago | View, Dismiss |
| Medium (amber) | Missing audit logging on delete operations | Compliance Violation | erp-workspace-api | Open | "Implement audit log for all DELETE endpoints" | 2 weeks ago | View, Assign |
| Low (blue) | HTTP security headers missing | Misconfiguration | erp-marketing-site | Open | "Add X-Frame-Options, CSP, HSTS headers" | 3 weeks ago | View |

- Severity indicator: colored dot + text
- Category badge: colored by SECURITY_CATEGORY_COLORS
- Remediation: truncated text with expand, or "View Details" link
- Row click: opens finding detail side panel
- Bulk actions: Assign, Mark Resolved, Dismiss with reason

SIDE PANEL (Finding Detail -- 480px slide-in):
- Full title and description
- Severity and category with color coding
- Affected resource with link to topology view
- Detailed remediation steps (numbered list)
- Compliance frameworks affected (SOC2 CC6.1, ISO27001 A.12)
- Timeline: detected, assigned, remediation started, resolved
- Related findings (same service or same category)
- "Mark Resolved" / "Dismiss" / "Create Incident" actions

CONTENT -- ROW 4: "Compliance Summary"
- Framework cards (side by side):
  -- SOC2: 91% compliance | 3 violations | "View Details"
  -- ISO27001: 83% compliance | 5 violations | "View Details"
  -- GDPR: 96% compliance | 1 violation | "View Details"
  -- Internal Policy: 88% compliance | 4 violations | "View Details"
- Each card shows a circular progress ring with percentage

Both light and dark themes.
```

### 7.2 Mobile Pages (390px)

#### Prompt F-015: Mobile AIOps Dashboard (390px)

```
Design a mobile AIOps Dashboard at 390x844px for the ERP-AIOps module.
Target: SRE on-call checking system health on mobile.

NAVIGATION:
- Bottom tab bar (5 tabs): Home, Incidents (badge: 7), Anomalies (badge: 14),
  Alerts (badge: 5), Profile
- Top status bar with "AI" logo badge (left), search icon (right)

HOME TAB:
- Greeting: "Good morning, Abiola" + current date "Monday, Feb 24, 2026"
- Platform health indicator: Large circular gauge (120px)
  -- 78/100 score, colored ring (green >80, amber 60-80, red <60)
  -- "Platform Health" label below

- KPI Cards (2x3 grid, each 170x80px):
  -- Open Incidents: 7 (red)
  -- Active Anomalies: 14 (amber)
  -- Rules Enabled: 12 (purple)
  -- Auto-Remediations: 8 (blue)
  -- Cost Savings: $24.5K (green)
  -- Security Findings: 5 (orange)

- "Critical Incidents" section (scrollable list):
  -- Each item: Severity dot + Title (truncated) + Status badge + Time
  -- "Memory leak in Commerce checkout" | Critical | Open | 12m
  -- "Finance gateway latency spike" | High | Investigating | 35m
  -- "View All (7)" link

- "Pending Actions" section:
  -- "2 remediations awaiting approval"
  -- Quick approve/reject buttons inline
  -- "Rollback commerce-api v2.14.3" | [Approve] [Reject]

- "Quick Links" horizontal scroll:
  -- Topology Map, Cost Report, Security Scan, Run Book

All touch targets >= 44px. Pull-to-refresh on home. Skeleton loading states.
Real-time WebSocket updates with subtle animation on count changes.
```

#### Prompt F-016: Mobile Incident Management (390px)

```
Design mobile Incident screens at 390x844px for SRE incident management on mobile.

SCREEN 1 -- "Incidents" (from Incidents tab):
- Segment control: "Open (4)" | "Investigating (2)" | "Resolved" | "All"
- Filter chips (horizontal scroll): All Severities, Critical, High, Medium, Low
- Sort: "Newest first" dropdown

INCIDENT LIST:
Each incident card (full width, ~100px tall):
- Left: Severity indicator bar (4px, colored)
- Content: Title (1 line, truncated), affected services (tags, 1 line),
  source badge (AI / Manual), created time
- Right: Status badge, chevron
- Swipe left: Quick Acknowledge (purple)
- Long press: bulk selection mode

Example cards:
1. Memory leak in ERP-Commerce checkout
   Critical | commerce-api | AI Detection | 12 min ago | Open

2. Finance payment gateway high latency
   High | finance-gateway | Alert Rule | 35 min ago | Investigating

3. HCM DB replica disk usage 90%
   Medium | hcm-db-replica | AI Detection | 1 hour ago | Acknowledged

SCREEN 2 -- "Incident Detail" (tapped from list):
- Severity + Status badges (top)
- Title (full, multi-line)
- "Acknowledged by Abiola | Investigating for 23m"
- Tabs: Summary | Timeline | Metrics | Actions
- Summary tab: AI-generated description, root cause, affected services
- Timeline tab: vertical timeline (scrollable)
- Metrics tab: mini charts (stacked vertically)
- Actions tab: remediation list with approve/reject buttons
- Sticky bottom bar: [Acknowledge] [Escalate] [Resolve]

SCREEN 3 -- "Quick Incident Create" (bottom sheet, slides up 85%):
- Title input
- Severity selector (horizontal radio chips: C H M L I)
- Service selector (searchable)
- Description (multiline)
- "Create" full-width button (sticky bottom)

Haptic feedback on status transitions. Toast notifications on success.
```

---

## 8. Mobile & Responsive Prompt

### Prompt F-017: Tablet AIOps Dashboard (1024px)

```
Design a tablet-adapted AIOps Dashboard at 1024x768px for the ERP-AIOps module.

ADAPTATIONS FROM DESKTOP (1440px):
- Sidebar collapsed to icon-only (72px) by default, expandable via hamburger
- KPI cards: 3x2 grid instead of 6-column row
- Data tables: stacked vertically (Recent Incidents above Recent Anomalies)
  instead of side-by-side
- Table columns: hide less critical columns, show on row expand
- Touch-optimized: all clickable areas >= 44px, increased spacing

HEADER:
- Hamburger menu (left), "Dashboard" title (center),
  search icon + notification bell + avatar (right)
- Breadcrumbs hidden; replaced by title

SPECIFIC LAYOUT:
Row 1: 3x2 KPI grid (Open Incidents, Active Anomalies, Rules Enabled,
        Auto-Remediations, Cost Savings, Security Findings)
Row 2: "Recent Incidents" table (full width, 5 rows)
Row 3: "Recent Anomalies" table (full width, 5 rows)

TOPOLOGY VIEW ADAPTATION (1024px):
- Canvas height reduced to 500px
- Node size reduced to 40px
- Detail panel opens as bottom sheet instead of side panel
- Touch gestures: pinch to zoom, drag to pan, tap for selection

COST DASHBOARD ADAPTATION:
- KPI cards: 2x2 grid
- Charts stacked vertically
- Recommendations list: full width cards

Support landscape (1024x768) and portrait (768x1024) orientations.
Split-screen support annotated for iPadOS.
```

### Prompt F-018: Responsive Breakpoint Annotations

```
Create a Figma page titled "AIOps Responsive Breakpoints" documenting the
responsive behavior of all major components across breakpoints.

BREAKPOINTS:
- Desktop Large: >= 1440px (full sidebar, 6-column KPI, side-by-side tables)
- Desktop: 1024-1439px (collapsed sidebar, 3-column KPI, stacked tables)
- Tablet: 768-1023px (icon sidebar, 2-column KPI, stacked content)
- Mobile: < 768px (bottom nav, single column, bottom sheets)

FOR EACH COMPONENT, SHOW SIDE-BY-SIDE:
- KPI Cards: 6-col -> 3-col -> 2-col -> 2-col (scrollable)
- Data Tables: show/hide columns at each breakpoint
  -- 1440: all columns
  -- 1024: hide Root Cause, Correlation ID
  -- 768: hide Source, show on expand
  -- 390: Title + Severity + Status only, full detail on tap
- Topology Graph: full canvas -> reduced canvas -> simplified list view
- Charts: full size -> stacked -> horizontally scrollable
- Sidebar: expanded 260px -> collapsed 72px -> hidden (hamburger) -> bottom nav
- Forms: centered 720px -> full width -> stacked fields
- Detail panels: side panel 480px -> bottom sheet -> full page

Include specific pixel values and overflow behavior at each breakpoint.
Annotate touch target sizes for tablet and mobile.
```

---

## 9. Validation Checklist

```markdown
## Design Handoff Checklist -- ERP-AIOps

### Visual Completeness
- [ ] All 12 pages designed for 1440px, 1024px, and 390px breakpoints
- [ ] Light and dark theme variants complete for every page
- [ ] All component states documented (default, hover, active, disabled, error, loading, empty)
- [ ] Realistic AIOps data used (no "Lorem ipsum" or "John Doe")
- [ ] All 8 sidebar navigation items represented with corresponding pages
- [ ] Severity and status color mappings consistently applied across all views
- [ ] AI-generated content clearly labeled with confidence indicators

### Accessibility
- [ ] Color contrast ratios verified (>= 4.5:1 text, >= 3:1 UI components)
- [ ] Touch targets >= 44x44px verified on all interactive elements
- [ ] Focus order annotated for keyboard navigation
- [ ] Screen reader annotations added for all non-text elements (charts, topology graph, status indicators)
- [ ] Keyboard navigation paths documented for topology canvas
- [ ] ARIA live regions annotated for real-time alert banners and status changes
- [ ] Critical severity alerts have both color and icon/shape differentiation (not color-only)

### Developer Handoff
- [ ] Design tokens exported in Style Dictionary JSON format
- [ ] Component specs include spacing, sizing, and behavioral notes
- [ ] API data mapping documented (which Hasura resource populates which component)
- [ ] Interaction specifications (transitions, animations, gesture handling)
- [ ] Error states and empty states designed for every data-dependent component
- [ ] WebSocket real-time update behavior annotated (which components auto-refresh)
- [ ] Topology graph interaction model fully specified (zoom, pan, select, hover)
- [ ] JSON editor component behavior documented (syntax highlighting, validation, auto-complete)

### AIDD Guardrails
- [ ] No prohibited actions (cross-tenant data access, irreversible deletes) exposed without safeguards
- [ ] Supervised actions (rule disable, remediation approval, incident close) have confirmation dialogs
- [ ] Audit trail access points visible for compliance integration
- [ ] Feature flag annotation points marked for conditional rendering
  -- cost_optimization_enabled, security_scanning_enabled, auto_remediation_enabled
- [ ] Human-in-the-loop gates marked for high-risk operations
  -- Production service restart, deployment rollback, circuit breaker activation
- [ ] AI-generated content (RCA, incident summaries, cost recommendations) labeled with confidence scores
- [ ] Sensitive data masking applied to security findings, credentials, API keys

### Real-Time & Performance
- [ ] Skeleton loading states designed for all async data components
- [ ] WebSocket connection status indicator designed (connected/reconnecting/disconnected)
- [ ] Auto-refresh intervals documented per component (dashboard: 30s, topology: 10s, alerts: real-time)
- [ ] Chart downsampling behavior specified for large time ranges
- [ ] Virtualized table scrolling annotated for lists > 50 rows
- [ ] Topology graph performance target: < 500ms render for 200 nodes
```

---

## 10. Output Packaging Convention

```
Deliverable naming:
  AIOps-{PromptID}-{Breakpoint}-{Theme}-v{Version}.fig
  Example: AIOps-F003-1440-light-v1.fig

Layer naming:
  {Page}/{Section}/{Component}-{State}
  Example: Dashboard/KPICards/OpenIncidents-Default
  Example: Topology/Canvas/Node-erp-finance-api-Degraded
  Example: Incidents/Table/Row-Critical-Hover

Asset export:
  /exports/aiops/{breakpoint}/{page}/
  Example: /exports/aiops/1440/dashboard/kpi-open-incidents.png
  Example: /exports/aiops/1440/topology/node-healthy.svg

Handoff tokens:
  /tokens/aiops-tokens.json (Style Dictionary format)

Icon set:
  /exports/aiops/icons/
  -- severity-critical.svg, severity-high.svg, severity-medium.svg, severity-low.svg
  -- status-healthy.svg, status-degraded.svg, status-down.svg, status-unknown.svg
  -- action-restart.svg, action-scale.svg, action-rollback.svg, action-cache.svg
```

---

## 11. Performance Acceptance Targets

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
| Chart render (time-series) | < 500ms | Custom perf mark |
| Topology render (200 nodes) | < 500ms | Custom perf mark |
| WebSocket reconnect | < 3s | Custom perf mark |
| Anomaly detection to UI update | < 5s | End-to-end SLO |

---

## 12. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-24 | AIDD System | Initial comprehensive prompt set covering dashboard, incident management, anomaly detection, rules engine, topology view, auto-remediation, cost optimization, security scanning, mobile views, tablet adaptations, and 5 Make automation workflows |

---

## 13. Round 03 Implementation Handoff (Web + Mobile + Desktop)

### 13.1 Delivered UI Surfaces
- `web/`: role-based AIOps ops console for SRE, DevOps Engineer, Security Analyst, FinOps Analyst, and Engineering Manager.
- `mobile/`: on-call incident response and system health monitoring for SRE-first operations.
- `tablet/`: topology exploration and remediation approval workflow for field operations.

### 13.2 Design Token Mapping
- Tokens are implemented in `web/src/theme.ts` and consumed by all frontend components.
- Palette: primary `#7c3aed`, success `#10b981`, warning `#f59e0b`, danger `#ef4444`, info `#3b82f6`.
- Typography baseline: Plus Jakarta Sans (all weights), JetBrains Mono (metric values, IDs, JSON).
- Border radius: 6px (tags), 8px (buttons/inputs/menu), 10px (cards/tables).
- Control height: 40px for all input components.

### 13.3 Figma Make Build Prompt (Round 03)
```
Build ERP-AIOps GA-ready UI kit and application flows using the existing token system.

Deliverables:
1. Desktop web ops console with pages:
   - Dashboard, Incidents (List/Show/Create), Anomalies (List/Show),
     Rules (List/Create), Topology, Remediation, Cost, Security.
2. Mobile on-call app with:
   - KPI health gauge, incident triage queue, quick acknowledge/escalate actions.
3. Tablet topology explorer with:
   - Full canvas interaction, remediation approval workflow, cost overview.

Interaction constraints:
- p95 interaction latency <= 120ms for primary workflows.
- Incident acknowledge CTA visible above fold on 1366x768.
- Preserve AIDD confirmation patterns for all remediation actions.
- AI-generated content always labeled with confidence scores.
- Real-time WebSocket updates for incidents and anomalies.
```

### 13.4 Commercial UX Guardrails
- No dead-end states: every error/empty state includes clear recovery actions.
- Role-first navigation: SRE can acknowledge incident within 2 clicks from dashboard.
- KPI-first information architecture: health score and open incident count shown before raw tables.
- AI transparency: all ML-generated insights (RCA, anomaly classification, cost recommendations) clearly attributed with model confidence.
- Approval gates: production-impacting remediations require explicit human approval regardless of automation rules.
