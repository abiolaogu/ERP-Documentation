# Figma & Make Prompts — BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-18 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Purpose

This document provides structured prompts for generating UI/UX designs in Figma (using AI-assisted design tools) and automation workflows in Make (formerly Integromat) for the BSS-OSS platform. These prompts serve as inputs to AI design assistants and automation builders within the AIDD pipeline, ensuring consistent design language and workflow patterns across the telecom platform.

---

## 2. Figma Design Prompts

### 2.1 Design System Foundation

#### Prompt F-001: Core Design Tokens
```
Create a design token system for a telecom BSS/OSS platform called "BSS-OSS" with the following specifications:

Color Palette:
- Primary: Deep navy blue (#1A2B4A) for headers and primary actions
- Secondary: Teal (#0D9488) for success states and positive metrics
- Accent: Amber (#F59E0B) for warnings and attention items
- Error: Red (#DC2626) for alarms, faults, and critical states
- Neutral: Gray scale from #F9FAFB to #111827

Typography:
- Headings: Inter, semi-bold, sizes 32/24/20/16
- Body: Inter, regular, sizes 14/12
- Monospace: JetBrains Mono for IDs, codes, and technical values

Spacing: 4px base unit (4, 8, 12, 16, 24, 32, 48, 64)
Border radius: 4px (inputs), 8px (cards), 12px (modals), 9999px (pills)
Shadows: sm (subtle card), md (dropdown), lg (modal overlay)

The design must support both light and dark themes, as network operations
centers (NOC) typically use dark mode during night shifts.
```

#### Prompt F-002: Component Library
```
Design a reusable component library for BSS-OSS with these telecom-specific components:

1. SubscriberCard: Shows subscriber name, MSISDN, status badge (active/suspended/terminated), balance, and current plan. Include avatar placeholder and quick-action buttons (call, ticket, top-up).

2. AlarmBanner: Horizontal alert bar with severity color coding (Critical=red, Major=orange, Minor=yellow, Warning=blue, Cleared=green). Shows alarm count, most recent alarm text, and "View All" link.

3. KPIWidget: Square card with large numeric value, trend arrow (up/down), percentage change, sparkline chart, and label. Used for revenue, subscriber count, ARPU, churn rate.

4. OrderTimeline: Vertical timeline showing order state transitions (Acknowledged -> In Progress -> Completed/Failed) with timestamps, responsible service, and status icons.

5. NetworkTopologyNode: Circular node for network diagrams representing eNodeB, gNodeB, router, switch, or OLT. Shows status color ring, label, and connection lines.

6. InvoiceLineItem: Table row component with columns: description, quantity, unit price, discount, tax, total. Alternating row colors, expandable for CDR details.

7. TicketStatusPill: Rounded pill showing ticket status with color: Open (blue), In Progress (yellow), Resolved (green), Closed (gray), Escalated (red).

All components must be responsive and work at 1440px (desktop), 1024px (tablet), and 375px (mobile) breakpoints.
```

### 2.2 Page-Level Design Prompts

#### Prompt F-003: Subscriber 360 View
```
Design a "Customer 360" page for a telecom CRM system with these sections:

Header Area:
- Subscriber photo/avatar, full name, MSISDN, account ID
- Status badge (Active, Suspended, Terminated)
- Action buttons: Edit, Suspend, Top Up, Create Ticket
- Last interaction timestamp

Left Column (40%):
- Personal Details card: name, DOB, email, address, KYC status
- Active Plans card: list of subscribed products with expiry dates
- Balance card: main balance, data balance, bonus balance with progress bars

Right Column (60%):
- Interaction Timeline: chronological feed of calls, tickets, payments, plan changes
- Usage Summary: donut charts for voice minutes, data GB, SMS used vs. allocated
- Open Tickets: compact table with ticket ID, subject, priority, SLA countdown
- Recent Transactions: table of last 10 transactions with type, amount, date

Footer:
- Quick navigation tabs: Invoices, CDR History, Linked Accounts, Notes

Design for 1440px desktop. Use the navy/teal color scheme. Data should look realistic for an African telecom subscriber.
```

#### Prompt F-004: Network Operations Center (NOC) Dashboard
```
Design a dark-themed Network Operations Center dashboard for a telecom OSS platform:

Top Bar:
- Real-time clock, NOC shift indicator, operator logo
- Global alarm summary: Critical (count), Major (count), Minor (count)
- Quick filters: Region, Technology (4G/5G), Status

Main Area (70%):
- Geographic map view showing network element locations as colored dots
  (green=healthy, yellow=degraded, red=alarmed)
- Zoom and filter controls
- Clicking a dot shows a popup with element details and active alarms

Right Panel (30%):
- Live alarm feed: scrolling list of most recent alarms with severity icon,
  element name, probable cause, timestamp
- Top 5 affected sites table
- Network KPIs: availability %, active alarms, MTTR, alarm rate/hour

Bottom Bar:
- Performance charts: 4 small time-series graphs showing aggregate throughput,
  latency, packet loss, and CPU utilization across the network

Color scheme: Dark background (#0F172A), bright accent colors for alarm severity.
All text must be legible at 2-meter viewing distance (NOC wall displays).
```

#### Prompt F-005: Billing Invoice Review Page
```
Design an invoice review and approval page for a telecom billing manager:

Filter Bar:
- Date range picker (billing cycle), status dropdown (Draft/Issued/Paid/Overdue),
  amount range slider, subscriber search

Invoice List (left, 40%):
- Scrollable list of invoices as cards showing: invoice #, subscriber name,
  amount, status pill, issue date
- Bulk selection checkboxes for batch actions (approve, issue, export)
- Sorting by amount, date, status

Invoice Detail (right, 60%):
- Appears when an invoice is selected
- Invoice header: number, subscriber details, billing period, due date
- Line items table: recurring charges, usage charges (voice/data/SMS breakdown),
  one-time charges, discounts, taxes, total
- CDR summary expandable section
- Payment history for this invoice
- Action buttons: Approve, Adjust, Void, Send to Subscriber, Download PDF

Ensure the layout supports efficient review of 100+ invoices per session.
```

#### Prompt F-006: Product Catalog Manager
```
Design a product catalog management interface for telecom plan configuration:

Navigation:
- Tree view on the left showing product hierarchy:
  Product Categories > Product Offerings > Product Specifications > Pricing

Main Editor:
- When a product offering is selected, show an editable form with:
  - Basic info: name, description, validity period, status (draft/active/retired)
  - Included services: data allocation (GB), voice minutes, SMS count
  - Pricing section: recurring price, one-time activation fee, currency
  - Eligibility rules: subscriber type, region, minimum tenure
  - Bundled add-ons: list of optional value-added services

Preview Panel:
- Shows how the plan appears to subscribers in the self-service portal
- Mobile and desktop preview toggle

Version History:
- Timeline showing all changes to this product with author and timestamp
- Ability to compare two versions side by side

Use cards and clean form layouts. Support drag-and-drop for reordering bundled services.
```

---

## 3. Make (Integromat) Automation Prompts

### 3.1 Subscriber Lifecycle Automations

#### Prompt M-001: New Subscriber Welcome Flow
```
Create a Make automation scenario for BSS-OSS new subscriber onboarding:

Trigger: Webhook - receives POST from BSS-OSS when event "subscriber.created" fires

Step 1: Parse subscriber data (name, email, MSISDN, plan_name)

Step 2: Send welcome email via SendGrid
  - Template: "welcome_new_subscriber"
  - Variables: first_name, plan_name, support_phone
  - From: noreply@{operator_domain}

Step 3: Send welcome SMS via Africa's Talking API
  - Message: "Welcome to {operator_name}, {first_name}! Your {plan_name} is now active. Dial *100# to check your balance."

Step 4: Create onboarding task in project management tool
  - Title: "Follow up with new subscriber {MSISDN}"
  - Due: 3 days from now
  - Assigned to: CRM team

Step 5: Log event to Google Sheets (backup audit trail)
  - Columns: timestamp, subscriber_id, msisdn, plan, email_sent, sms_sent

Error handler: On any step failure, post alert to Slack #subscriber-alerts channel.
```

#### Prompt M-002: Churn Risk Alert
```
Create a Make automation for telecom churn prediction alerts:

Trigger: Scheduled - runs daily at 06:00 UTC

Step 1: HTTP GET to BSS-OSS API endpoint /analytics/churn-risk
  - Parameters: risk_score_threshold=0.7, limit=100
  - Authentication: Bearer token from Make vault

Step 2: Filter - only subscribers with risk_score >= 0.8 (high risk)

Step 3: For each high-risk subscriber:
  a. Create retention ticket in CRM via POST /api/tickets
     - Category: "Retention"
     - Priority: "High"
     - Notes: "AI churn score: {score}. Top factors: {factors}"
  b. Assign to retention team based on subscriber value tier

Step 4: Aggregate results and send daily summary email to retention manager
  - Total at-risk subscribers
  - Breakdown by risk tier (0.7-0.8, 0.8-0.9, 0.9+)
  - Top 10 highest-value at-risk subscribers

Step 5: Update Superset dashboard data source via ClickHouse INSERT
```

### 3.2 Billing Automations

#### Prompt M-003: Invoice Delivery Pipeline
```
Create a Make automation for invoice delivery after billing run:

Trigger: Webhook - receives POST from billing service when event "billing_run.completed"

Step 1: HTTP GET list of generated invoices from /api/invoices?billing_run_id={id}&status=approved

Step 2: Iterator - process each invoice

Step 3: For each invoice:
  a. HTTP GET invoice PDF from /api/invoices/{id}/pdf
  b. Send email via SendGrid with PDF attachment
     - Template: "monthly_invoice"
     - Variables: subscriber_name, amount_due, due_date, invoice_number
  c. If subscriber.preferred_channel == "sms":
     Send SMS notification: "Your invoice #{number} for {amount} is ready. Due: {date}. View at {portal_url}"
  d. Update invoice status to "delivered" via PATCH /api/invoices/{id}

Step 4: After all invoices processed, send summary to billing manager
  - Total invoices delivered: {count}
  - Failed deliveries: {failed_count}
  - Total amount billed: {total}

Error handler: Retry failed email/SMS delivery 3 times with 5-minute intervals.
Timeout: 60 minutes maximum for the entire scenario.
```

#### Prompt M-004: Payment Reconciliation
```
Create a Make automation for daily payment reconciliation:

Trigger: Scheduled - runs daily at 23:00 UTC

Step 1: HTTP GET payments from Paystack API for today
Step 2: HTTP GET payments from Flutterwave API for today
Step 3: HTTP GET recorded payments from BSS-OSS /api/payments?date=today

Step 4: Compare gateway records with BSS-OSS records
  - Match by transaction reference
  - Identify: matched, gateway-only (missing in BSS), BSS-only (missing in gateway)

Step 5: For each unmatched gateway payment:
  - POST to BSS-OSS /api/payments/reconcile with gateway data
  - Create reconciliation ticket in finance queue

Step 6: Generate reconciliation report
  - Total gateway transactions: {count}
  - Total BSS-OSS transactions: {count}
  - Matched: {count}
  - Discrepancies: {count}
  - Net difference: {amount}

Step 7: Send report to finance team via email and Slack #finance-ops

Step 8: Store report in Google Drive / SharePoint for audit trail
```

### 3.3 Network Operations Automations

#### Prompt M-005: Critical Alarm Escalation
```
Create a Make automation for critical network alarm escalation:

Trigger: Webhook - receives POST from fault management when event "alarm.raised" with severity="critical"

Step 1: Parse alarm data: element_id, element_name, alarm_text, probable_cause, region, timestamp

Step 2: Check if duplicate alarm exists (suppress flapping)
  - HTTP GET /api/alarms?element_id={id}&status=active&last_30_minutes=true
  - If duplicate found, skip escalation and increment alarm count

Step 3: Determine on-call engineer from PagerDuty schedule API
  - Route by region and technology type

Step 4: Create PagerDuty incident
  - Title: "CRITICAL: {element_name} - {alarm_text}"
  - Urgency: High
  - Service: Based on region
  - Assigned: On-call engineer

Step 5: Send Slack notification to #noc-critical channel
  - Formatted message with alarm details, affected subscribers estimate, and PagerDuty link

Step 6: If element supports auto-remediation:
  - POST to BSS-OSS /api/remediation/auto with element_id and alarm_type
  - Log remediation attempt

Step 7: Create trouble ticket in CRM linked to the alarm

Timeout: All steps must complete within 60 seconds of alarm receipt.
```

#### Prompt M-006: Scheduled Network Health Report
```
Create a Make automation for weekly network health reporting:

Trigger: Scheduled - runs every Monday at 07:00 UTC

Step 1: HTTP GET network KPIs from ClickHouse via BSS-OSS analytics API
  - Availability percentage by region and technology
  - Total alarms raised/cleared in the past 7 days
  - Mean Time to Repair (MTTR)
  - Top 10 worst-performing sites

Step 2: HTTP GET subscriber impact data
  - Number of subscribers affected by outages
  - Total downtime minutes

Step 3: Generate HTML report using template
  - Executive summary with key metrics
  - Regional breakdown table
  - Trend charts (compared to previous 4 weeks)
  - Action items from unresolved critical alarms

Step 4: Convert HTML to PDF using a document generation service

Step 5: Distribute report
  - Email to network operations leadership
  - Upload to SharePoint/Confluence
  - Post summary to Slack #network-weekly

Step 6: Update Superset dashboard data refresh timestamp
```

---

## 4. Prompt Usage Guidelines

### 4.1 Figma Prompt Best Practices
- Always specify the target viewport width (1440px desktop, 375px mobile)
- Reference the BSS-OSS design tokens from Prompt F-001 for color consistency
- Include realistic telecom data in mockups (Nigerian phone numbers, Naira currency, African names)
- Request both light and dark theme variants for NOC-facing screens
- Specify accessibility requirements: WCAG 2.1 AA minimum, 4.5:1 contrast ratio

### 4.2 Make Automation Best Practices
- Always include error handling and retry logic in prompts
- Specify timeout limits appropriate to the workflow criticality
- Include logging and audit trail steps for compliance
- Reference BSS-OSS API endpoints using the base URL variable `{{bss_oss_base_url}}`
- Store all API keys and tokens in Make's vault, never hardcoded in scenarios
- Test automations in staging environment before activating in production

### 4.3 Versioning
- Each prompt is versioned independently
- When a prompt is updated, increment the prompt ID suffix (e.g., F-003 becomes F-003-v2)
- Archive old prompts rather than deleting them

---

## 5. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-18 | AIDD System | Initial Figma and Make prompt library for BSS-OSS platform |
