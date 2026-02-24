# Figma & Make.com Prompts -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Overview

This document contains 8 detailed Figma design prompts for the ERP-BSS-OSS platform portals and dashboards, plus Make.com automation workflow prompts.

---

## 2. Figma Design Prompts

### Prompt 1: BSS Admin Console

```
Design a comprehensive BSS/OSS Admin Console for a telecom platform.

LAYOUT:
- Left sidebar navigation with icons: Dashboard, Customers, Products, Orders, Billing,
  Provisioning, Partners, Network Ops, Revenue Assurance, Meters, Configuration
- Top bar: tenant picker (dropdown), user avatar, notifications bell with badge count,
  search bar (global search across customers, orders, products)
- Main content area with responsive grid

DASHBOARD PAGE:
- KPI cards row: Active Subscribers (count + trend arrow), Monthly Revenue (currency +
  sparkline), CDR Processed Today (count + bar chart), Open Tickets (count + severity badges)
- Revenue chart: Line chart showing daily revenue for last 30 days, with toggle for
  voice/data/SMS/VAS breakdown
- Order funnel: Horizontal funnel showing acknowledged -> in_progress -> completed with
  counts and conversion rates
- Alarm severity panel: 4 colored boxes (Critical=red, Major=orange, Minor=yellow,
  Warning=blue) with count badges
- Recent activity feed: Scrollable list of recent events (order created, invoice generated,
  alarm raised) with timestamps and icons

COLOR PALETTE:
- Primary: #1E40AF (telecom blue)
- Secondary: #059669 (success green)
- Warning: #D97706 (amber)
- Error: #DC2626 (red)
- Background: #F9FAFB (light gray)
- Card: #FFFFFF with subtle shadow

TYPOGRAPHY:
- Headers: Inter Bold
- Body: Inter Regular
- Monospace (for IDs, codes): JetBrains Mono

Make it enterprise-grade but modern. Dark mode variant included.
No gradients. Clean borders. Data-dense without feeling cluttered.
```

---

### Prompt 2: Customer Self-Care Portal

```
Design a subscriber self-care portal for a mobile telecom operator.

TARGET USER: Non-technical subscriber (both smartphone and tablet)

HOME SCREEN:
- Greeting: "Hello, John!" with account number
- Balance card: Large circular progress ring showing data usage (35.2 / 50 GB), with
  airtime balance ($50.00) and SMS count (320/500) below
- Quick action buttons: Top Up, Buy Data, Pay Bill, Get Help (4 equal cards with icons)
- Usage summary: Horizontal bar charts for Voice (120/1000 min), Data (35.2/50 GB),
  SMS (320/500)
- Recent transactions: Last 5 transactions with type icon, description, amount, date

TOP-UP PAGE:
- Amount selector: Predefined amounts ($5, $10, $20, $50) as tappable cards, plus
  "Custom Amount" input
- Payment method selector: Saved cards (showing last 4 digits), Bank Transfer, Mobile Money,
  Voucher Code
- Confirm button: Large green button with amount shown
- Success animation: Checkmark with confetti, new balance displayed

PLAN MANAGEMENT PAGE:
- Current plan card with name, price, allowances
- "Change Plan" button showing available plans as comparison cards
- Each plan card: Name, price, data/voice/SMS included, "Select" button
- Comparison table toggle

MOBILE-FIRST DESIGN:
- Bottom tab navigation: Home, Usage, Plans, Bills, More
- Rounded corners (16px radius)
- Large touch targets (minimum 48px)
- Accessible contrast ratios (WCAG AA)

COLOR PALETTE:
- Primary: #7C3AED (vibrant purple - operator brand)
- Secondary: #10B981 (success)
- Background: #F5F3FF (lavender tint)
- Cards: White with shadow-sm

Include onboarding flow (3 screens) for first-time users.
```

---

### Prompt 3: Partner Portal

```
Design a partner portal for MVNO partners of a host MNO.

DASHBOARD:
- Welcome banner with partner name and logo
- KPI cards: Active Subscribers (count), Monthly Revenue (chart), Settlement Balance
  (pending amount), Support Tickets (open count)
- Revenue trend chart: Monthly revenue bar chart with partner_share and operator_share
  stacked bars
- Subscriber growth chart: Line chart showing subscriber count over 12 months

SETTLEMENT PAGE:
- Monthly settlement table: Period, Gross Revenue, Partner Share %, Partner Amount,
  Status (Calculated/Approved/Paid/Disputed)
- Settlement detail: Expandable rows showing revenue breakdown by product
- Dispute button: Opens dispute form with line-item selection
- Download: CSV and PDF export buttons

REPORTS PAGE:
- Pre-built reports: Revenue by Product, Subscriber Demographics, Usage Patterns,
  Churn Analysis
- Date range picker
- Chart + table views
- Export options (CSV, PDF, Excel)

RESOURCE MANAGEMENT:
- MSISDN allocation: Pool status, available numbers, assigned numbers
- SIM inventory: Available, assigned, activated counts with status filters

NAVIGATION:
- Left sidebar: Dashboard, Subscribers, Revenue, Settlement, Reports, Resources,
  API Keys, Support
- Breadcrumb trail
- Partner logo in top-left

STYLE: Clean enterprise SaaS. Blue-gray palette. Data tables with sorting and filtering.
```

---

### Prompt 4: Billing Dashboard

```
Design a billing operations dashboard for telecom billing managers.

BILLING CYCLE VIEW:
- Cycle selector: Dropdown for billing period (January 2026, February 2026, etc.)
- Progress bar: Customers Processed: 45,000 / 50,000 (90%) with ETA
- Stats cards: Invoices Generated, Total Billed Amount, Average Invoice, Errors
- Invoice distribution chart: Histogram of invoice amounts
- Error queue: Table showing failed invoices with customer_id, error_reason, retry_count

REVENUE DASHBOARD:
- Revenue by service type: Donut chart (Voice 35%, Data 45%, SMS 10%, VAS 10%)
- Revenue trend: Area chart showing daily revenue for last 90 days
- ARPU chart: Average Revenue Per User monthly trend
- Top 10 revenue customers: Table sorted by revenue

DUNNING OVERVIEW:
- Dunning funnel: Visual funnel showing customer counts at each dunning level
  (Current -> 7d Overdue -> 14d -> 30d Barred -> 45d Suspended -> 90d Terminated)
- Aging report: Table showing overdue amounts by aging bucket (0-30, 31-60, 61-90, 90+)
- Collection rate: Percentage of overdue amounts collected per month

PAYMENT TRACKING:
- Daily payment receipts: Bar chart
- Payment method distribution: Pie chart (Card, Bank, Mobile Money, Voucher)
- Failed payments: Table with reason codes and retry status

LAYOUT: Dense data display with collapsible sections. Dark theme option for NOC-style
viewing. Auto-refresh every 60 seconds.
```

---

### Prompt 5: Network Operations Center (NOC) Dashboard

```
Design a Network Operations Center dashboard for telecom NOC engineers.

PRIMARY VIEW (wall-mounted large screen):
- Network health indicator: Large circle with overall score (98.5%) and color
  (green/yellow/red)
- Active alarms: Scrolling ticker at top showing critical and major alarms
- Network topology map: Simplified diagram showing major network elements (BTS, NodeB,
  eNodeB, core switches) with color-coded status
- Alarm severity breakdown: 4 large counters (Critical: 2, Major: 5, Minor: 12, Warning: 25)

ALARM MANAGEMENT VIEW:
- Alarm table: Severity icon, Element Name, Alarm Text, Time Raised, Duration,
  Acknowledge button
- Filter bar: By severity, element type, time range, acknowledged/unacknowledged
- Alarm correlation panel: Shows related alarms grouped by root cause
- Trouble ticket auto-creation button

PERFORMANCE VIEW:
- KPI gauges: CPU utilization (per element), bandwidth utilization, call drop rate,
  handover success rate
- Traffic charts: Erlang (voice), Throughput Mbps (data), SMS/hour
- SLA compliance: Table showing SLA metrics vs targets with RAG status

WORKFORCE VIEW:
- Map showing field engineer locations (GPS dots)
- Dispatched tickets list with status (en route, on site, completed)
- Available engineers count

STYLE:
- Dark background (#0F172A) for NOC wall visibility
- High-contrast colors for status (green #22C55E, yellow #EAB308, red #EF4444)
- Large font sizes for wall viewing distance
- Auto-refresh every 10 seconds
- Audible alarm tone option for critical alerts
```

---

### Prompt 6: USSD Simulator

```
Design a USSD simulator tool for testing USSD menu flows.

LAYOUT:
- Left panel: Phone mockup (feature phone style with small screen, 12-key keypad)
- Right panel: Session inspector showing request/response JSON

PHONE MOCKUP:
- Small monochrome screen (Nokia 3310 style)
- Display shows USSD menu text with numbered options
- Keypad: 0-9, *, #, Send, End buttons
- MSISDN display at top of phone

INTERACTION:
- User types *123# on keypad, presses Send
- Screen shows: "Welcome to BSS-OSS\n1.Balance\n2.TopUp\n3.Plans\n4.Data\n0.Exit"
- User presses 1
- Screen shows: "Your balance:\nAirtime: $50.00\nData: 2.5GB\nSMS: 320"

SESSION INSPECTOR:
- Request tab: JSON showing {session_id, msisdn, shortcode, input, type}
- Response tab: JSON showing {message, type: "continue"|"end"}
- Timeline: Vertical timeline of all session steps with timestamps
- Session state: Current menu_state shown in badge

CONFIGURATION PANEL:
- MSISDN to simulate (dropdown of test numbers)
- Shortcode to dial
- Subscriber profile (balance, plan, etc.) for simulation
- Reset session button

STYLE: Split view with thin border. Inspector uses monospace font.
Phone mockup has retro aesthetic. Clean white background for contrast.
```

---

### Prompt 7: Meter Management Dashboard

```
Design a utility meter management dashboard for electricity/water/gas operators.

OVERVIEW:
- KPI cards: Total Meters (count by type: smart/prepaid/conventional),
  Active Meters (% of total), Readings Today (count), Tamper Alerts (badge count)
- Meter status distribution: Donut chart (Active, Maintenance, Offline, Tamper Alert)
- Map view: Geographic map showing meter locations as colored dots (green=ok, red=alert)

METER DETAIL VIEW:
- Meter info card: Number, type, manufacturer, model, firmware, install date
- Customer info: Name, address, account number
- Consumption chart: Line chart showing daily kWh for last 30 days
- Readings table: Timestamp, value, quality flag (valid/estimated/suspect)
- Tamper history: Timeline of tamper events with severity and resolution

TOKEN MANAGEMENT:
- Recent tokens: Table showing meter_number, kWh, amount_paid, token_value, status
- Token generation: Form with meter_number, amount, shows calculated kWh, Generate button
- Token statistics: Bar chart showing daily token sales

AMI HEAD-END STATUS:
- Communication success rate: Gauge showing % of successful readings
- Failed communications: Table with meter_number, last_successful, failure_reason
- Firmware update status: Progress bars for batch firmware updates

STYLE: Industrial/utility aesthetic. Blue-gray palette. Dense data tables.
Map uses OpenStreetMap base layer.
```

---

### Prompt 8: Provisioning Workflow Designer

```
Design a visual provisioning workflow editor for defining service activation flows.

CANVAS:
- Drag-and-drop workflow canvas (similar to Node-RED or n8n)
- Left panel: Palette of provisioning steps (Reserve SIM, Activate HLR, Configure PCRF,
  Register Service, Send Notification, Check Balance, Conditional Branch, Parallel Split,
  Wait Timer, Rollback)
- Each step is a rectangular card with icon, name, and status indicator

WORKFLOW DISPLAY:
- Steps connected by arrows showing flow direction
- Parallel paths shown as branching arrows
- Conditional branches shown as diamond shapes
- Rollback path shown as red dashed arrows back to previous steps

STEP CONFIGURATION:
- Click a step to open configuration panel on the right
- Configuration fields depend on step type:
  - Reserve SIM: resource_type, pool_name, timeout
  - Activate HLR: msisdn, imsi, service_profile
  - Configure PCRF: qos_profile, data_rate, priority
  - Conditional: expression editor (e.g., "order.items.length > 1")
  - Rollback: compensation_action, notification_template

EXECUTION VIEW:
- Live execution overlay: Steps turn green (success), red (failed), yellow (in progress)
- Log panel at bottom: Timestamped execution log
- Retry button on failed steps
- Rollback button to trigger compensation chain

TEMPLATES:
- Pre-built templates: New Prepaid Activation, SIM Swap, Number Port-In,
  Postpaid to Prepaid Migration, eSIM Activation

STYLE: Clean whiteboard aesthetic with grid background. Colorful step icons.
Zoom and pan controls. Minimap in corner.
```

---

## 3. Make.com Automation Prompts

### Automation 1: New Customer Welcome Workflow

```
Create a Make.com scenario that triggers when a new customer is created in ERP-BSS-OSS
(webhook on erp.bss_oss.customer-management.created event).

Steps:
1. Webhook trigger: Receive CloudEvents payload
2. Parse customer data (name, email, phone, plan)
3. Send welcome email via SendGrid (template: welcome_new_subscriber)
4. Send welcome SMS via Twilio
5. Create welcome task in project management tool
6. Add customer to CRM segment in ERP-CRM
7. Log success/failure to Google Sheets for tracking
```

### Automation 2: Billing Alert Workflow

```
Create a Make.com scenario that triggers on invoice.overdue event.

Steps:
1. Webhook trigger: Receive overdue invoice event
2. Look up customer contact details
3. If dunning_level == 1: Send reminder SMS
4. If dunning_level == 2: Send warning email with invoice PDF attachment
5. If dunning_level >= 3: Create task for collections team in task management
6. Update CRM with dunning status
7. Log all actions to audit spreadsheet
```

### Automation 3: Fraud Alert Escalation

```
Create a Make.com scenario for fraud alert escalation.

Steps:
1. Webhook trigger: Receive fraud.detected event
2. If severity == "critical":
   a. Send Slack message to #fraud-alerts channel
   b. Create PagerDuty incident
   c. Auto-bar suspected SIMs via API call
3. If severity == "high":
   a. Send email to fraud team
   b. Create ticket in support system
4. Log all alerts to fraud database
5. Generate weekly fraud summary report (scheduled)
```

### Automation 4: Partner Settlement Notification

```
Create a Make.com scenario for monthly partner settlement.

Steps:
1. Scheduled trigger: 5th of each month
2. Call BSS API: GET /v1/partner-service/settlements?period=last_month
3. For each partner:
   a. Generate settlement report PDF
   b. Send email to partner with report attached
   c. Post to partner portal notification
4. Send summary to finance team
5. Log completion to operations dashboard
```
