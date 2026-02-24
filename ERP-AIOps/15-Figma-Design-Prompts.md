# ERP-AIOps Figma Design Prompts

> **Document ID:** ERP-AIOPS-FIGMA-015
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [07-User-Manual.md](./07-User-Manual.md), [08-Training-Manual.md](./08-Training-Manual.md)

---

## Design System Foundation

**Primary Color:** Purple `#7c3aed`
**Component Library:** Ant Design (antd)
**Framework:** React + Refine.dev
**Font:** Inter (body), JetBrains Mono (code/metrics)
**Layout:** Left sidebar navigation (240px collapsed to 64px), top header bar (64px)

### Color Palette

| Token | Hex | Usage |
|-------|-----|-------|
| Primary | `#7c3aed` | Buttons, links, active states |
| Primary Hover | `#6d28d9` | Button hover, link hover |
| Primary Light | `#ede9fe` | Selected row background, badges |
| Success | `#10b981` | Healthy status, resolved incidents |
| Warning | `#f59e0b` | Warning status, medium anomalies |
| Error | `#ef4444` | Critical status, P1 incidents |
| Info | `#3b82f6` | Informational badges, links |
| Neutral BG | `#f9fafb` | Page background |
| Card BG | `#ffffff` | Card/panel background |
| Text Primary | `#111827` | Headings, body text |
| Text Secondary | `#6b7280` | Descriptions, metadata |

---

## Prompt 1: AIOps Command Center Dashboard

**Figma Make Prompt:**

"Design a full-width AIOps command center dashboard using Ant Design components with a purple (#7c3aed) primary theme. The page has a left sidebar (collapsed) and top header with tenant selector dropdown and user avatar.

The dashboard contains:

1. **Top KPI Row** (4 cards in a row):
   - Health Score: Large circular gauge showing 87/100 in green
   - Active Incidents: Count badge showing 12, with breakdown by severity (P1: 1, P2: 3, P3: 8) using colored dots
   - Anomalies Today: Count showing 47 with a mini sparkline trend (last 24h)
   - Active Rules: Count showing 156 with enabled/disabled split

2. **Incident Feed** (left 60% of second row):
   - Ant Design List component with incident cards
   - Each card shows: severity badge (P1=red, P2=orange, P3=yellow), title, affected service tags, time since detection, assigned avatar
   - Scrollable, max 10 items visible

3. **Anomaly Trend Chart** (right 40% of second row):
   - Line chart showing anomaly count over last 7 days
   - Stacked by severity (critical=red, high=orange, medium=yellow, low=gray)
   - Purple-tinted chart area

4. **Service Health Grid** (full width third row):
   - Grid of service cards (4 per row) for each ERP module
   - Each card: module name, health indicator dot, request rate, error rate, latency p99
   - Cards with active incidents have a colored left border matching severity

Background: #f9fafb. Cards: white with subtle shadow. All text in #111827/#6b7280."

---

## Prompt 2: Incident List with Severity Badges and Timeline

**Figma Make Prompt:**

"Design an incident list page using Ant Design Table component with purple (#7c3aed) theme.

**Header Section:**
- Page title 'Incidents' with breadcrumb
- Filter bar: Ant Design Select for severity (P1-P5), state (Detected, Investigating, Resolved, etc.), service, date range picker
- 'Create Incident' primary purple button on right

**Table Columns:**
- Severity: Colored badge (P1=red filled, P2=orange filled, P3=yellow filled, P4=blue outline, P5=gray outline)
- Title: Bold text, clickable link in purple
- State: Tag component (Detected=blue, Investigating=orange, Mitigating=yellow, Resolved=green, Closed=gray)
- Affected Services: Up to 3 Ant Design Tags, +N more indicator
- Created: Relative time (e.g., '2h ago')
- Assignee: Avatar with name
- SLA: Timer showing remaining SLA time, red if breached

**Selected Row Expansion:**
- Mini timeline showing last 5 state transitions with timestamps
- Quick action buttons: Acknowledge, Investigate, Resolve

Show 8 sample rows with varied severities and states. Include pagination at bottom. Background #f9fafb, table background white."

---

## Prompt 3: Incident Detail with RCA Visualization

**Figma Make Prompt:**

"Design an incident detail page with tabs using Ant Design components and purple (#7c3aed) theme.

**Header:**
- Breadcrumb: Incidents > INC-2024-0847
- Title: 'ERP-CRM Database Connection Pool Exhaustion'
- Severity badge P2 (orange), State tag 'Investigating' (orange)
- Action buttons row: Acknowledge, Investigate, Mitigate, Resolve (state-appropriate enabled)
- Assignee avatar + name, Created timestamp, SLA timer

**Tabs:**
1. **Overview** (active):
   - Summary card with incident description
   - Affected Services: Tag list with health indicators
   - Related Anomalies: Mini cards (3 items) showing metric, score, time

2. **Root Cause Analysis**:
   - Causal Chain: Horizontal flowchart showing: 'Config Change (IAM rate limit)' → 'Auth Rejection Spike' → 'CRM Connection Timeout' → 'Pool Exhaustion'
   - Each node has a confidence percentage badge
   - Contributing Factors: Ranked list with progress bars showing contribution weight
   - Similar Past Incidents: 3 cards with title, date, resolution summary, similarity score

3. **Timeline**:
   - Vertical timeline (Ant Design Timeline component)
   - Events: anomaly detected, incident created, assigned, state transitions, comments, RCA completed
   - Each event has icon, timestamp, actor avatar, description

4. **Topology Impact**:
   - Mini service graph showing affected services highlighted in red/orange
   - Causal path overlay with animated arrows

5. **Activity**:
   - Comment thread with rich text input
   - Activity feed mixing comments and system events

Background #f9fafb, white cards with shadow."

---

## Prompt 4: Anomaly List with Score Sparklines

**Figma Make Prompt:**

"Design an anomaly list page using Ant Design Table with purple (#7c3aed) theme.

**Header:**
- Page title 'Anomalies'
- Filter bar: severity (Critical/High/Medium/Low), service selector, metric type, time range, status (Active/Reviewed/Dismissed)
- View toggles: List view (active) | Grid view

**Table Columns:**
- Score: Colored circular badge (0.8+ = red, 0.6-0.8 = orange, 0.3-0.6 = yellow) with numeric value
- Sparkline: Mini 50x20px line chart showing score evolution over last hour (inline in table cell)
- Metric: Metric name (e.g., 'cpu_usage_percent')
- Service: Service name with module icon
- Detected: Relative timestamp
- Algorithm: Tag showing primary detecting algorithm (Z-Score, IF, LSTM)
- Status: Tag (Active=purple, Reviewed=blue, Dismissed=gray)
- Actions: Eye icon (view), Check icon (review), X icon (dismiss)

Show 10 sample rows with varied scores and statuses. Include score distribution mini histogram in the header area showing distribution of today's anomaly scores. Background #f9fafb."

---

## Prompt 5: Anomaly Detail with Metric Charts

**Figma Make Prompt:**

"Design an anomaly detail page using Ant Design components with purple (#7c3aed) theme.

**Header:**
- Breadcrumb: Anomalies > ANOM-2024-1205
- Score badge: 0.87 in large red circle
- Metric: 'cpu_usage_percent', Service: 'ERP-Commerce', Detected: '15 minutes ago'
- Status tag: Active (purple)
- Action buttons: Review, Dismiss, Create Incident

**Main Content (2 columns):**

**Left Column (65%):**
- **Metric Chart**: Large time series chart (last 4 hours)
  - Actual value: solid purple line
  - Predicted baseline: dashed gray line
  - Confidence bands: light purple shaded area (upper/lower bounds)
  - Anomaly region: red-shaded background where score exceeded threshold
  - X-axis: time, Y-axis: percentage
- **Score Sparkline**: Smaller chart showing anomaly score over last 4 hours with threshold line

**Right Column (35%):**
- **Algorithm Breakdown**: Vertical bar chart showing per-algorithm scores
  - Z-Score: 0.82 (purple bar)
  - Isolation Forest: 0.91 (purple bar)
  - LSTM: 0.85 (purple bar)
  - Moving Avg: 0.78 (purple bar)
- **Contributing Factors**: Ranked list
  - Rate of change: 42% contribution (progress bar)
  - Absolute deviation: 31%
  - Seasonal mismatch: 18%
  - Cross-metric correlation: 9%
- **Related Anomalies**: 2 mini cards linking to concurrent anomalies on same service
- **Recommended Actions**: AI-generated suggestion cards

Background #f9fafb, white cards."

---

## Prompt 6: Rule Management Table with Enable/Disable Toggles

**Figma Make Prompt:**

"Design a rule management page using Ant Design Table with purple (#7c3aed) theme.

**Header:**
- Page title 'Rules' with count badge (156 total)
- Tab bar: All Rules | Detection | Correlation | Suppression | Escalation
- Filter: search input, status (Enabled/Disabled), severity
- 'Create Rule' primary purple button

**Table Columns:**
- Enable/Disable: Ant Design Switch component (purple when enabled)
- Name: Bold clickable text
- Type: Tag (Detection=purple, Correlation=blue, Suppression=orange, Escalation=red)
- Condition: Truncated condition text (e.g., 'cpu > 90% for 5m')
- Severity: Colored dot + label
- Last Triggered: Relative time or 'Never'
- Trigger Count (7d): Number with mini bar chart
- Actions: Edit icon, Clone icon, Delete icon, History icon

Show 8 sample rules of different types. Some enabled, some disabled (grayed out row). Include a summary bar above table: 'Active: 142 | Disabled: 14 | Triggered today: 23'. Background #f9fafb."

---

## Prompt 7: Rule Creation Form with Condition Builder

**Figma Make Prompt:**

"Design a rule creation form using Ant Design Form components with purple (#7c3aed) theme.

**Layout:** Full-width modal or dedicated page with left sidebar showing form progress steps.

**Step 1 - Basic Info:**
- Rule Name: Text input
- Description: TextArea
- Type: Radio group (Detection, Correlation, Suppression, Escalation)
- Severity: Select dropdown (P1-P5) with colored indicators

**Step 2 - Condition Builder:**
- Visual condition builder with rows:
  - Row: [Metric Select] [Operator Select (>, <, >=, etc.)] [Value Input] [Duration Input]
  - 'AND' / 'OR' connector between rows
  - '+Add Condition' button
- For Correlation type: Two condition groups with 'correlates with' label between them
  - Source Event pattern
  - Correlated Event pattern
  - Time Window input
  - Topology Constraint toggle

**Step 3 - Actions:**
- Checkboxes: Create Incident, Send Notification, Trigger Remediation
- Notification channels: multi-select (Slack, Teams, Email, PagerDuty)
- Remediation playbook: select dropdown (only if Trigger Remediation checked)

**Step 4 - Review:**
- Summary card showing all configured values
- 'Test Rule' button (purple outline) and 'Save & Enable' button (purple filled)

Purple primary buttons, clean form layout with proper spacing. Background #f9fafb."

---

## Prompt 8: Service Topology Graph Visualization

**Figma Make Prompt:**

"Design a service topology page with an interactive graph visualization using purple (#7c3aed) theme.

**Layout:**
- Full-width graph canvas (takes 75% of the page)
- Right sidebar panel (25%) for selected node details (collapsible)

**Graph:**
- Nodes represent ERP modules as rounded rectangles with:
  - Module icon (top)
  - Module name (center, bold)
  - Health status dot (bottom-left: green/yellow/orange/red)
  - Mini metrics: request rate, error rate (bottom)
- Edges represent dependencies as curved lines with arrows
  - Line thickness proportional to traffic volume
  - Line color: green (healthy), orange (degraded), red (failing)
- Show 12 nodes arranged in a force-directed layout
- Highlight one node in red with its downstream blast radius nodes in orange

**Right Sidebar (when node selected):**
- Service name and health badge
- Key metrics: Request Rate, Error Rate, Latency P99, CPU, Memory
- Active Incidents: mini list (max 3)
- Recent Anomalies: mini list (max 3)
- Action buttons: View Details, Analyze Blast Radius

**Toolbar (top of graph):**
- Zoom controls (+/-)
- Filter dropdown (by module group, health state)
- Layout toggle (Force-directed, Hierarchical, Circular)
- Legend showing node colors and edge meanings

Background #f9fafb for sidebar, light gray for graph canvas."

---

## Prompt 9: Auto-Remediation Activity Log

**Figma Make Prompt:**

"Design a remediation activity log page using Ant Design Table and Timeline components with purple (#7c3aed) theme.

**Header:**
- Page title 'Remediation Activity'
- Tab bar: Activity Log | Playbooks | Action Catalog
- Filter: status (Success/Failed/Pending Approval/Rolled Back), playbook, service, date range
- Stats bar: 'Executed today: 8 | Success rate: 87.5% | Avg duration: 45s'

**Activity Table:**
- Status: Icon + colored badge (Success=green check, Failed=red X, Pending=yellow clock, Rolled Back=blue undo)
- Playbook Name: Bold text
- Trigger: Link to triggering incident or anomaly
- Action: Action type tag (scale_out, restart, rollback, etc.)
- Target Service: Service name with icon
- Duration: Time in seconds
- Executed At: Timestamp
- Executed By: 'System (auto)' or user avatar + name (manual approve)

**Expanded Row Detail:**
- Vertical timeline showing execution steps:
  - Trigger received
  - Approval check (auto-approved / manual-approved by [name])
  - Action execution started
  - Action execution completed / failed
  - Verification check (passed / failed)
- Full execution log in monospace font (collapsible)
- Outcome metrics before/after comparison

Show 6 sample rows. Background #f9fafb."

---

## Prompt 10: Cost Optimization Dashboard with Savings Charts

**Figma Make Prompt:**

"Design a cost optimization dashboard using Ant Design components and charts with purple (#7c3aed) theme.

**Top KPI Row (4 cards):**
- Total Monthly Cost: '$48,200' with trend arrow (up 3% from last month, red)
- Projected Savings: '$12,400/mo' with confidence badge
- Applied Savings: '$8,600/mo' (actual realized savings, green)
- Open Recommendations: '12' with breakdown by risk level

**Charts Row:**
- **Cost Trend** (left 50%): Area chart showing monthly cost over 6 months
  - Purple area fill, darker line
  - Budget line as dashed red horizontal line
  - Annotation markers for when recommendations were applied
- **Savings Tracker** (right 50%): Grouped bar chart
  - Projected vs. Actual savings per month
  - Purple (projected) vs. green (actual)

**Recommendations Table (full width bottom):**
- Priority: numbered badge (1, 2, 3...)
- Category: Tag (Right-sizing=purple, Idle Resource=blue, Reserved=green, Storage=orange)
- Service: Service name
- Current Cost: Dollar amount
- Projected Savings: Dollar amount in green
- Confidence: Progress bar with percentage
- Risk: Badge (Low=green, Medium=yellow, High=red)
- Actions: 'Apply' purple button, 'Reject' gray button, 'View Detail' link

Show 6 sample recommendations sorted by projected savings. Background #f9fafb."

---

## Prompt 11: Security Findings Table with Severity Distribution

**Figma Make Prompt:**

"Design a security findings page using Ant Design components with purple (#7c3aed) theme.

**Top Section:**
- Page title 'Security' with last scan timestamp
- Security Posture Score: Large gauge showing 78/100
- Severity Distribution: Horizontal stacked bar chart
  - Critical: red (3), High: orange (12), Medium: yellow (28), Low: gray (45)
- MTTR by Severity: Small bar chart showing average remediation time per severity

**Filter Bar:**
- Severity multi-select, Status (Open/Triaged/Remediated/Accepted Risk/False Positive), CVE ID search, Service filter, Date range

**Findings Table:**
- Severity: Colored badge (Critical=red, High=orange, Medium=yellow, Low=gray) with CVSS score
- CVE ID: Monospace text, clickable link
- Title: Vulnerability description (truncated)
- Affected Component: Service name + container image tag
- Status: Tag (Open=red, Triaged=blue, Remediated=green, Accepted Risk=yellow, False Positive=gray)
- Discovered: Relative timestamp
- SLA: Days remaining or 'Breached' in red
- Actions: Triage dropdown (Create Ticket, Accept Risk, Mark False Positive, Remediate)

Show 8 sample findings with varied severities. Include a 'Trigger Scan' purple button in header. Background #f9fafb."
