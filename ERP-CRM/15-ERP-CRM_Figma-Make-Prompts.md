# Figma & Make Prompts — ERP-CRM
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides structured, copy-paste-ready prompts for generating production-grade UI designs in Figma Make and automation workflows in Make (formerly Integromat) for the ERP-CRM module. The CRM platform manages the full customer lifecycle: leads, contacts, opportunities, pipelines, helpdesk ticketing, knowledge base, live chat, form building, territory management, activity tracking, automation rules, and reporting. All prompts reference the 12 backend microservices (`activity-service`, `automation-service`, `chat-service`, `contact-service`, `form-builder-service`, `helpdesk-service`, `knowledge-base-service`, `lead-service`, `opportunity-service`, `pipeline-service`, `reporting-service`, `territory-service`) and are designed for multi-tenant, entitlement-aware deployment via ERP-Platform with ERP-IAM authentication (OIDC/JWT).

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance across all surfaces
- Minimum 44x44px touch targets on all interactive elements
- Clear visual states: default, hover, focus (2px ring), active, disabled, loading, error
- Plain-language labels; no jargon without tooltip explanations
- Color-blind-safe palette; never rely on color alone to convey meaning
- Keyboard navigation and command palette (Cmd+K) parity for power users
- Screen reader announcements for dynamic content (toast, drawer, modal)

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading for all page modules
- Initial route gzip bundle < 220KB
- Skeleton UIs for every data-dependent component (tables, cards, charts)
- Optimistic UI for safe mutations (activity log, notes, status changes)
- Autosave for long forms (contact details, opportunity editing)
- Image lazy loading with blur-up placeholders

### 2.3 Reliability, Trust, And Safety
- Recovery paths for all error states (retry button, support link, fallback view)
- Trust signals: "Last synced at HH:MM", "Saved", "Auto-saved draft"
- Audit trail badges: "Created by [user] on [date]", "Modified by [user]"
- Two-step confirmation for destructive actions (delete contact, remove deal, close ticket)
- Tenant isolation indicator: tenant/organization name always visible in header
- Human-in-the-loop for AI recommendations: Approve / Edit / Reject actions

### 2.4 Observability And Testability
- Analytics events on: page view, CTA click, form submit, search query, filter change
- Error events on: API failure, form validation failure, timeout
- Performance instrumentation: FCP, LCP, CLS, TTI per route
- Feature flag awareness: components degrade gracefully when features are disabled

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```text
Create a comprehensive design system for an enterprise CRM platform called "ERP-CRM" that manages leads, contacts, opportunities, pipelines, helpdesk, knowledge base, live chat, form building, territories, and reporting.

DESIGN TOKENS:

Color System:
- Primary: Deep Indigo (#3730A3) for navigation, headers, and primary actions — conveys trust and professionalism
- Secondary: Teal (#0D9488) for secondary actions, links, and data visualization accents
- Accent: Amber (#D97706) for warnings, attention items, and pipeline stage highlights
- Success: Emerald (#059669) for won deals, resolved tickets, qualified leads
- Danger: Rose (#E11D48) for lost deals, SLA breaches, destructive actions
- Info: Sky (#0284C7) for informational badges, tooltips, and help content
- Neutral: Slate scale (50-950) for text hierarchy, borders, backgrounds, and surfaces
- Surface hierarchy: 3 elevation levels — Base (#F8FAFC), Raised (#FFFFFF with shadow-sm), Overlay (#FFFFFF with shadow-lg)
- Pipeline stage colors: Prospect (#93C5FD), Qualification (#818CF8), Proposal (#A78BFA), Negotiation (#F59E0B), Closed Won (#10B981), Closed Lost (#EF4444)
- Lead score gradient: Cold (#94A3B8), Warm (#FBBF24), Hot (#EF4444)
- Dark mode: Slate-900 base, Slate-800 raised, Slate-700 borders, all text colors inverted appropriately

Typography:
- Font family: Inter (UI and body text) / JetBrains Mono (data tables, IDs, code)
- Scale: 11px (caption/overline), 12px (label/small), 13px (body-sm), 14px (body), 16px (subtitle), 20px (title), 24px (heading), 32px (display)
- Weight tokens: regular (400), medium (500), semibold (600), bold (700)
- Line-height tokens: tight (1.2), normal (1.5), relaxed (1.75)
- Numeric tabular figures for all tables, financial data, and metrics
- Letter spacing: -0.02em for headings, normal for body

Spacing & Layout:
- 4px base grid; spacing scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80
- Breakpoints: mobile (390px), tablet (1024px), desktop (1440px), wide (1920px)
- Max content width: 1440px with 24px gutters
- Sidebar widths: collapsed (64px), expanded (256px)
- Page padding: 24px (desktop), 16px (tablet), 12px (mobile)

Elevation & Motion:
- Shadow tokens: sm (cards), md (dropdowns/popovers), lg (modals/drawers), xl (command palette)
- Border-radius tokens: none, sm (4px), md (8px), lg (12px), full (9999px)
- Motion tokens: instant (0ms), fast (100ms), normal (200ms), slow (300ms)
- Purposeful motion only: state transitions, drawer/modal reveals, chart entry animations

Dark Mode:
- Background: Slate-950 (#020617) base, Slate-900 (#0F172A) raised, Slate-800 (#1E293B) overlay
- Text: Slate-50 (#F8FAFC) primary, Slate-300 (#CBD5E1) secondary, Slate-500 (#64748B) tertiary
- Borders: Slate-700 (#334155)
- Primary action buttons retain brand colors with adjusted contrast
- Charts and visualizations use lighter tints on dark backgrounds

Deliver as a structured token library with light/dark theme variants and auto-layout component primitives.
```

#### Prompt F-002: Component Library

```text
Create a production-ready component library for ERP-CRM with these domain-specific components:

NAVIGATION:
- Sidebar navigation with sections: Dashboard, Leads, Contacts, Opportunities, Pipelines, Helpdesk, Knowledge Base, Chat, Forms, Territories, Reports, Automation, Settings
- Each item: icon (Lucide icon set) + label + optional count badge
- Collapsed (64px, icon-only with tooltip) and expanded (256px) states
- Active state: left 3px accent bar + background highlight
- Top bar: org/tenant name, global search trigger ("Search contacts, leads, deals..."), notification bell with unread count, user avatar with role badge
- Breadcrumb trail with clickable segments
- Command palette (Cmd+K) with fuzzy search across contacts, leads, opportunities, tickets, articles, and actions

DATA DISPLAY:
- Data table with: sortable column headers, sticky header row, row selection checkboxes, bulk action bar, inline status editing, column resize handles, pagination with page size selector, CSV/Excel export button
- Contact card: avatar, name, company, role, email, phone, lead score badge, last activity timestamp
- Opportunity card: deal name, company, amount (currency formatted), pipeline stage badge, probability %, owner avatar, close date, days-in-stage counter
- Lead card: name, source badge (web form, referral, event, cold outreach), score indicator (cold/warm/hot), assigned rep avatar, age counter
- Ticket card: ID, subject, priority badge (low/medium/high/urgent), status badge (open/pending/resolved/closed), SLA countdown, assigned agent avatar
- KPI stat card: metric name, value, delta vs. prior period, trend sparkline, period selector
- Activity timeline: actor avatar, action verb, entity link, timestamp, optional note preview

FORMS & INPUT:
- Text input with label, placeholder, helper text, validation error
- Select, multi-select, combobox with search
- Date picker, date range picker
- Rich text editor (for knowledge base articles, ticket responses)
- File upload with drag-and-drop zone and progress indicator
- Toggle switch, checkbox, radio group
- Tag input (for contact tags, deal tags)
- Lead score input (slider or dropdown: 1-100)
- Currency input with currency symbol selector
- Phone input with country code selector

FEEDBACK & OVERLAY:
- Toast notifications: success (green), error (red), warning (amber), info (blue) with auto-dismiss and action link
- Modal dialog (sm, md, lg) with header, body, footer actions
- Drawer (right-slide) for record detail views (contact detail, ticket detail)
- Confirmation dialog with destructive variant (red button, type-to-confirm for deletes)
- Progress stepper for multi-step workflows (new lead qualification, opportunity stages)
- Skeleton loaders matching each component shape
- Empty state with illustration, message, and CTA button
- Error state with retry action and support link

CRM-SPECIFIC:
- Pipeline board: Kanban columns with stage headers (name, deal count, total value), draggable opportunity cards, column totals, "Add Deal" button per column
- Lead scoring badge: circular gauge with score number and cold/warm/hot color ring
- SLA timer: countdown display with green/amber/red color based on remaining time
- Chat widget: message bubbles (agent/visitor), typing indicator, file attachment, quick reply chips
- Form builder preview: live preview of form fields as they are configured
- Territory map overlay: colored regions on map with rep assignments and opportunity pins
- Automation rule card: trigger event, conditions summary, action summary, active/paused toggle, last fired timestamp

ACCESSIBILITY:
- All interactive elements: visible focus rings (2px offset, primary color)
- Minimum contrast ratio 4.5:1 for text, 3:1 for UI elements
- All icons paired with labels or aria-labels
- Form inputs with associated labels, helper text, and error messages
- Screen reader announcements for dynamic content updates
- Keyboard navigation for all patterns (Tab, Enter, Escape, Arrow keys)

Deliver as a component library with variant properties, auto-layout, and design token references. Include both light and dark mode variants.
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: CRM Command Dashboard (1440px)

```text
Design a CRM command dashboard for ERP-CRM at 1440px desktop width with the following layout:

TOP BAR:
- Left: ERP-CRM logo, tenant/organization name ("Apex Solutions Ltd")
- Center: global search bar with placeholder "Search contacts, leads, deals, tickets..."
- Right: notification bell with "7" badge, user avatar "JA" with name "James Adeyemi" and role "Sales Manager"

LEFT SIDEBAR (expanded, 256px):
- Dashboard (active, highlighted with left accent bar)
- Leads (badge: "23 new")
- Contacts
- Opportunities (badge: "$1.2M pipeline")
- Pipelines
- Helpdesk (badge: "5 open")
- Knowledge Base
- Chat (green dot indicating online)
- Forms
- Territories
- Reports
- Automation
- Settings
- Collapse toggle at bottom

MAIN CONTENT AREA:

Row 1 — Welcome & Quick Actions:
- Greeting: "Good morning, James. You have 5 follow-ups today."
- Quick action buttons: "New Lead", "New Contact", "Log Activity", "Create Ticket"

Row 2 — KPI Cards (4 columns):
- Pipeline Value: $1,247,500 (up 12% vs. last month, green sparkline)
- Qualified Leads: 47 (up 8, trend sparkline)
- Win Rate: 34% (down 2%, amber indicator)
- Avg Deal Cycle: 28 days (down 3 days, green indicator)

Row 3 — Two Panels:
- Left (60%): "Pipeline Overview" — horizontal stacked bar chart showing deal count and value by stage (Prospect: 12/$180K, Qualification: 8/$240K, Proposal: 6/$310K, Negotiation: 4/$290K, Closed Won: 3/$227K)
- Right (40%): "Lead Sources" — donut chart (Web Forms: 35%, Referral: 28%, Events: 20%, Cold Outreach: 12%, Other: 5%)

Row 4 — Two Panels:
- Left (50%): "Today's Activities" — timeline list showing:
  - 09:00 — Call with Sarah Chen (contact) re: Enterprise License renewal
  - 10:30 — Follow-up email to David Okafor (lead) — hot score
  - 14:00 — Demo presentation for TechVault Inc. (opportunity, $85K)
  - 15:30 — Team pipeline review meeting
- Right (50%): "Recent Leads" — table with columns: Name, Source, Score (color-coded badge), Assigned To, Age, Actions (view, convert)
  - Show 5 sample leads with diverse names and varied scores

Row 5 — Bottom Panel:
- "Open Helpdesk Tickets" — table: Ticket ID, Subject, Customer, Priority (badge), SLA Timer (countdown), Status, Assigned Agent
  - Show 5 sample tickets with varying priorities and SLA states

Include hover states on all interactive elements. Show both light and dark mode variants.
```

#### Prompt F-004: Pipeline Board View (1440px)

```text
Design a Kanban-style pipeline board for ERP-CRM at 1440px desktop width.

PAGE HEADER:
- Title: "Sales Pipeline" with pipeline selector dropdown (showing "Enterprise Sales Pipeline" selected)
- Subtitle: "42 deals | $1,247,500 total value"
- Actions: "Add Deal" (primary button), "Pipeline Settings" (icon button), "Export" (icon button)
- View toggle: Board (active) | List | Forecast
- Filter bar: Owner (multi-select), Close Date (date range), Amount (range slider), Tags (multi-select), "Clear Filters"

KANBAN BOARD (horizontal scroll):

Column 1 — "Prospect" (blue header, 12 deals, $180,000):
- Deal cards showing:
  - Company logo/avatar + Company name
  - Deal name (truncated if long)
  - Deal value: "$15,000"
  - Owner avatar (small circle)
  - Close date: "Mar 15, 2026"
  - Days in stage: "3 days"
  - Lead score indicator dot (cold/warm/hot)
- Show 4 sample deal cards
- "Add Deal" ghost button at bottom

Column 2 — "Qualification" (indigo header, 8 deals, $240,000):
- 3 sample deal cards with varied data
- One card showing an overdue indicator (red border)

Column 3 — "Proposal" (purple header, 6 deals, $310,000):
- 3 sample deal cards
- One card with "Proposal Sent" sub-badge

Column 4 — "Negotiation" (amber header, 4 deals, $290,000):
- 2 sample deal cards with higher values ($65K, $85K)
- One card showing "Contract Review" sub-badge

Column 5 — "Closed Won" (green header, 3 deals, $227,500):
- 2 sample cards with green checkmark and celebration confetti micro-icon

Column 6 — "Closed Lost" (red/muted header, 9 deals, $184,000):
- 1 sample card with "Lost Reason: Budget" tag

INTERACTIONS:
- Cards are draggable between columns (show subtle shadow elevation on drag)
- Drag from "Negotiation" to "Closed Won" triggers a confirmation modal: "Mark deal as won?" with fields for actual close date, final amount, notes
- Clicking a card opens a right-side drawer with full opportunity detail (company, contacts, activities, notes, documents, probability, forecast)
- Column headers are collapsible
- "+" button in each column header to add a deal directly to that stage

DEAL DETAIL DRAWER (400px right slide):
- Header: deal name, stage badge, owner avatar
- Tabs: Overview, Activities, Contacts, Documents, Notes
- Overview tab: deal value, close date, probability slider, next step field, tags
- Activities tab: timeline of calls, emails, meetings, notes with timestamps
- Contacts tab: associated contacts with role (Decision Maker, Champion, Influencer)
- Quick actions: "Log Call", "Send Email", "Schedule Meeting", "Add Note"

Include light and dark mode variants.
```

#### Prompt F-005: Contact 360 View (1440px)

```text
Design a comprehensive Contact 360 profile page for ERP-CRM at 1440px desktop width.

PAGE HEADER:
- Back arrow + Breadcrumb: "Contacts > Sarah Chen"
- Contact avatar (large, 80px, with online/offline indicator)
- Name: "Sarah Chen" | Title: "VP of Engineering" | Company: "TechVault Inc." (linked)
- Tags: "Enterprise", "Decision Maker", "Tech Vertical"
- Lead score badge: 87 (hot, red ring)
- Last activity: "Email sent 2 hours ago"
- Action buttons: "Edit Contact" (secondary), "Log Activity" (primary), "Send Email" (primary), three-dot menu (Convert, Merge, Archive, Delete)

MAIN CONTENT — TWO COLUMN LAYOUT:

LEFT COLUMN (65%):

Tab Bar: Overview | Activities | Emails | Deals | Tickets | Notes | Files

Overview Tab (default):
- Contact Information card:
  - Email: sarah.chen@techvault.io (with copy icon)
  - Phone: +1 (415) 555-0142 (with call icon)
  - Mobile: +1 (415) 555-0198 (with SMS icon, WhatsApp icon)
  - LinkedIn: linkedin.com/in/sarachen (link)
  - Address: 350 California St, Suite 400, San Francisco, CA 94104
  - Timezone: PST (UTC-8) — "Currently 10:23 AM for Sarah"
  - Preferred contact method: Email (badge)
  - Language: English

- Company Information card:
  - Company: TechVault Inc. (linked to company record)
  - Industry: Technology
  - Company Size: 500-1000 employees
  - Annual Revenue: $50M-100M
  - Website: techvault.io

- Related Opportunities card:
  - "Enterprise License Renewal" — $85,000 — Negotiation stage — 75% probability — Close: Mar 30, 2026
  - "Cloud Migration Add-on" — $42,000 — Proposal stage — 50% probability — Close: Apr 15, 2026

- Recent Activities card (last 5):
  - Feb 22 — Email sent: "Q1 License Renewal Proposal" by James Adeyemi
  - Feb 20 — Call (15 min): "Discussed timeline concerns" by James Adeyemi
  - Feb 18 — Meeting (1 hr): "Product demo with engineering team" — 4 attendees
  - Feb 15 — Note added: "Sarah prefers async communication, follow up via email"
  - Feb 12 — Form submitted: "Feature Request — SSO Integration"

RIGHT COLUMN (35%):

- Quick Stats card:
  - Total Interactions: 47
  - Emails Sent/Received: 23/18
  - Meetings Held: 6
  - Days Since First Contact: 142
  - Engagement Score: High (green badge)

- Associated Contacts card (at TechVault Inc.):
  - Michael Torres — CTO — Decision Maker
  - Lisa Park — Engineering Manager — Champion
  - David Kim — Procurement Lead — Influencer

- Upcoming Tasks card:
  - "Send revised proposal" — Due Feb 25 — Assigned: James Adeyemi
  - "Follow up on security questionnaire" — Due Feb 28 — Assigned: Maria Santos

- Custom Fields card:
  - NDA Signed: Yes (green check)
  - Contract Type: Annual
  - Support Tier: Premium
  - Referral Source: Industry Conference 2025

Include light and dark mode variants.
```

#### Prompt F-006: Helpdesk Queue & Ticket Detail (1440px)

```text
Design a helpdesk ticket management page for ERP-CRM at 1440px desktop width.

PAGE HEADER:
- Title: "Helpdesk" with subtitle "32 open tickets"
- Action buttons: "New Ticket" (primary), "Knowledge Base" (secondary), "Reports" (icon)
- View toggle: Queue (active) | Board | Calendar

FILTER BAR:
- Status: All | Open (18) | Pending (8) | Resolved (4) | Closed (2)
- Priority: All | Urgent (3) | High (7) | Medium (12) | Low (10)
- Assignee: dropdown with agent avatars and names
- Tags: multi-select (Bug, Feature Request, Billing, Integration, Account)
- SLA: All | Within SLA | Breached | At Risk
- Date range picker
- Sort by: Created Date | Priority | SLA | Last Updated
- "Clear Filters" link

TICKET TABLE:
- Columns: Checkbox, Ticket ID (e.g., #CRM-1247), Subject, Customer (avatar + name), Priority (badge), Status (badge), SLA (countdown timer with color), Assigned To (avatar), Created, Updated, Actions
- Bulk action bar (appears when rows selected): "Assign To", "Change Priority", "Change Status", "Add Tags", "Merge"
- Sample tickets (8 rows):
  1. #CRM-1247 | "Cannot export contacts to CSV" | Sarah Chen | High | Open | 2h 15m remaining (amber) | Maria Santos | Feb 22 | Feb 23
  2. #CRM-1245 | "Pipeline board not loading" | David Okafor | Urgent | Open | 45m remaining (red) | Unassigned | Feb 22 | Feb 22
  3. #CRM-1243 | "Feature request: bulk email templates" | Lisa Park | Medium | Pending | Within SLA (green) | James Adeyemi | Feb 21 | Feb 22
  4. #CRM-1240 | "Integration webhook failing intermittently" | Michael Torres | High | Open | 1h 30m remaining (amber) | Alex Johnson | Feb 20 | Feb 23
  5-8: Additional varied tickets

TICKET DETAIL VIEW (click a ticket — full page or split view):

Left Panel (60%):
- Ticket header: ID, subject, status badge, priority badge, SLA timer, "Resolve" (primary), "Close" (secondary), three-dot menu
- Conversation thread (chronological):
  - Customer message: avatar + name + timestamp + message body (supports rich text, images, attachments)
  - Agent reply: avatar + name + timestamp + "via Email" badge + message body
  - System note: "Priority changed from Medium to High by Maria Santos" (gray background)
  - Internal note: (yellow background) "Checked with engineering — fix deployed in v2.4.1"
- Reply composer at bottom:
  - Rich text editor with formatting toolbar
  - Toggle: "Reply" | "Internal Note" (yellow indicator)
  - Attachment button, Knowledge Base article inserter, Canned response selector
  - "Send Reply" button, "Send & Resolve" button

Right Panel (40%):
- Ticket Properties card:
  - Status: Open (dropdown to change)
  - Priority: High (dropdown)
  - Assigned To: Maria Santos (dropdown with agent search)
  - Tags: Bug, Integration (editable tag input)
  - Created: Feb 22, 2026, 09:15 AM
  - Updated: Feb 23, 2026, 11:30 AM
  - SLA Policy: Premium Support — 4h response, 24h resolution

- Customer Information card:
  - Avatar + Name: Sarah Chen
  - Company: TechVault Inc.
  - Email: sarah.chen@techvault.io
  - Previous tickets: 3 (link to list)
  - Satisfaction rating: 4.5/5

- Related Articles card (from knowledge-base-service):
  - "How to export contacts" — 85% relevance match
  - "Troubleshooting CSV exports" — 72% relevance match
  - "Contact data formats" — 60% relevance match

- Ticket Timeline card:
  - Created by Sarah Chen via Web Form
  - Auto-assigned to Maria Santos (territory: West Coast)
  - First response sent (within SLA)
  - Priority escalated to High
  - Internal note added

Include light and dark mode variants.
```

#### Prompt F-007: Lead Management & Scoring (1440px)

```text
Design a lead management page with AI-powered lead scoring for ERP-CRM at 1440px desktop width.

PAGE HEADER:
- Title: "Leads" with subtitle "156 total leads | 47 qualified"
- Actions: "New Lead" (primary), "Import Leads" (secondary), "Lead Scoring Settings" (icon), "Export" (icon)
- View toggle: List (active) | Board | Map

FILTER BAR:
- Score: slider range (0-100) with cold/warm/hot labels
- Source: Web Form, Referral, Event, Cold Outreach, Social Media, Partner
- Status: New, Contacted, Qualified, Unqualified, Converted, Nurturing
- Owner: multi-select with avatars
- Date range: Created date
- Territory: dropdown (from territory-service)
- "Clear Filters"

LEAD TABLE:
- Columns: Checkbox, Lead Score (circular gauge badge), Name, Company, Source (badge), Status (badge), Owner (avatar), Territory, Created Date, Last Activity, Actions (view, convert, edit, more)
- Sort by any column
- Sample leads (8 rows) with diverse data:
  1. Score: 92 (hot/red) | "David Okafor" | NovaTech Solutions | Web Form | Qualified | James A. | Lagos | Feb 20 | "Opened pricing email 3 times"
  2. Score: 78 (warm/amber) | "Priya Sharma" | Meridian Corp | Event | Contacted | Maria S. | Mumbai | Feb 18 | "Downloaded whitepaper"
  3. Score: 45 (warm/amber) | "Thomas Müller" | EuroTech GmbH | Referral | New | Unassigned | Berlin | Feb 22 | N/A
  4. Score: 23 (cold/blue) | "Emma Williams" | StartupXYZ | Cold Outreach | Contacted | Alex J. | London | Feb 10 | "No response to 2 emails"
  5-8: Additional varied leads

LEAD DETAIL DRAWER (right slide, 450px):
- Large lead score gauge at top: circular ring with score (e.g., 92), label "Hot Lead"
- AI Score Breakdown (expandable panel):
  - "Score Components" heading
  - Engagement: 35/40 pts — "Opened 8 emails, clicked 5 links, visited pricing page 3x"
  - Fit: 30/30 pts — "Company size: 200+, Industry: Technology, Role: VP-level"
  - Behavior: 22/20 pts — "Downloaded 2 whitepapers, attended demo webinar"
  - Recency: 5/10 pts — "Last activity: 2 days ago"
  - "Confidence: 89%" badge
  - "Last scored: Feb 23, 2026, 08:00 AM"
  - Link: "View full scoring model"

- Contact Information: name, email, phone, company, title, LinkedIn
- Lead Source: badge + details (e.g., "Web Form — Pricing Page Form, Feb 20")
- Activity Timeline: last 5 interactions
- Conversion Readiness: AI recommendation card
  - "This lead is ready for conversion. Recommended next step: Schedule a discovery call."
  - Confidence: 89%
  - Reasoning: "High engagement, decision-maker role, company fits ICP"
  - Buttons: "Convert to Opportunity" (primary), "Convert to Contact" (secondary), "Dismiss"

- Quick Actions: "Send Email", "Log Call", "Schedule Meeting", "Assign Owner"

BULK ACTIONS BAR (when rows selected):
- "Assign To", "Change Status", "Add to Nurture Campaign", "Convert Selected", "Export", "Delete"

Include light and dark mode variants.
```

#### Prompt F-008: Reports & Analytics Dashboard (1440px)

```text
Design a reporting and analytics dashboard for ERP-CRM at 1440px desktop width, powered by reporting-service.

PAGE HEADER:
- Title: "Reports & Analytics"
- Date range selector: "Last 30 Days" dropdown with custom range picker
- Compare toggle: "vs. Previous Period" (checkbox)
- Actions: "Create Report" (primary), "Scheduled Reports" (secondary), "Export All" (icon)

TAB BAR:
- Overview (active) | Pipeline | Leads | Activities | Helpdesk | Territories | Custom

OVERVIEW TAB:

Row 1 — KPI Cards (6 columns):
- Revenue Won: $227,500 (up 15%, green)
- Pipeline Value: $1,247,500 (up 8%)
- New Leads: 67 (up 12%)
- Win Rate: 34% (down 2%, amber)
- Avg Deal Size: $75,833 (up 5%)
- Avg Cycle Time: 28 days (down 3 days, green)

Row 2 — Revenue & Pipeline Chart:
- Full-width area chart with dual Y-axis
- Left Y: Revenue ($) — filled area, green
- Right Y: Pipeline Value ($) — line overlay, indigo
- X-axis: last 12 months
- Hover tooltip showing exact values and period-over-period change

Row 3 — Three Panels:
- Left (33%): "Pipeline by Stage" — horizontal bar chart
  - Prospect: $180K (12 deals)
  - Qualification: $240K (8 deals)
  - Proposal: $310K (6 deals)
  - Negotiation: $290K (4 deals)

- Center (33%): "Lead Conversion Funnel" — funnel visualization
  - New Leads: 67
  - Contacted: 52 (78%)
  - Qualified: 31 (60%)
  - Opportunity Created: 18 (58%)
  - Won: 6 (33%)

- Right (33%): "Deals Won/Lost" — grouped bar chart by month
  - Won (green bars) vs Lost (red bars) for last 6 months
  - Running win rate line overlay

Row 4 — Two Panels:
- Left (50%): "Top Performers" — leaderboard table
  - Rank, Rep Name (avatar), Deals Won, Revenue Won, Win Rate, Pipeline Value
  - 5 sample reps with varied performance data
  - Highlight row for current user

- Right (50%): "Activity Summary" — stacked bar chart
  - Calls, Emails, Meetings, Notes per week for last 8 weeks
  - Legend with toggleable series

Row 5 — Bottom Panel:
- "Forecast vs. Actual" — grouped bar chart by quarter
  - Target bar (outline), Forecast bar (semi-transparent), Actual bar (solid)
  - Q1 2025, Q2 2025, Q3 2025, Q4 2025, Q1 2026 (current with projection)
  - Above/below target indicator per quarter

REPORT BUILDER (accessible via "Create Report"):
- Step 1: Choose report type (Pipeline, Lead, Activity, Contact, Territory, Custom)
- Step 2: Select metrics (checkboxes: revenue, deal count, win rate, avg deal size, cycle time, etc.)
- Step 3: Add dimensions (group by: owner, territory, source, stage, industry, time period)
- Step 4: Set filters (date range, owner, territory, pipeline)
- Step 5: Choose visualization (table, bar chart, line chart, pie/donut, funnel)
- Step 6: Save & Schedule (name, description, schedule frequency, email recipients)

Include light and dark mode variants.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: CRM Mobile Dashboard (390px)

```text
Design a mobile CRM dashboard screen for ERP-CRM at 390px width (iPhone 14 frame, 390x844px).

STATUS BAR: standard iOS status bar

TOP BAR:
- Left: hamburger menu icon
- Center: "ERP-CRM" logo
- Right: notification bell with "7" badge, user avatar "JA"

MAIN CONTENT (scrollable):

Greeting Section:
- "Good morning, James"
- "You have 5 follow-ups today" (tappable, links to task list)

KPI Row (horizontal scroll, 2 visible + peek):
- Pipeline: $1.2M (up 12%)
- Qualified Leads: 47 (up 8)
- Win Rate: 34% (down 2%)
- Avg Cycle: 28 days (down 3)

Quick Actions Grid (2x2):
- New Lead (blue icon)
- Log Activity (green icon)
- New Ticket (amber icon)
- Search (indigo icon)

Today's Schedule Card:
- "3 activities scheduled"
- 09:00 — Call: Sarah Chen
- 10:30 — Email: David Okafor
- 14:00 — Demo: TechVault Inc.
- "View All" link

Recent Leads Card:
- Scrollable list of 3 leads with score badge, name, source, age
- "View All Leads" link

Open Tickets Card:
- "5 open tickets"
- Top 3 with priority badge, subject (truncated), SLA timer
- "View All Tickets" link

AI Insight Card:
- "Lead David Okafor has high engagement — consider scheduling a call"
- "View Lead" (primary CTA), "Dismiss" (secondary)

BOTTOM NAVIGATION BAR (5 items):
- Dashboard (active, filled icon)
- Leads (with "23" badge)
- Deals
- Tickets (with "5" badge)
- More

NAVIGATION DRAWER (from hamburger):
- User profile section at top (avatar, name, role, tenant name)
- Full menu: Dashboard, Leads, Contacts, Opportunities, Pipelines, Helpdesk, Knowledge Base, Chat, Forms, Reports, Settings
- Logout at bottom

Include light and dark mode variants.
```

#### Prompt F-010: Mobile Pipeline Board (390px)

```text
Design a mobile pipeline board screen for ERP-CRM at 390px width (iPhone 14 frame).

TOP BAR:
- Back arrow
- Title: "Sales Pipeline"
- Pipeline selector: "Enterprise" dropdown
- Filter icon with active filter dot

VIEW TOGGLE (segmented control):
- Board (active) | List

BOARD VIEW:
- Horizontal scrollable columns (one column visible at a time, swipe to navigate)
- Column header: stage name, deal count, total value, color indicator bar
- Stage indicator dots at top showing current position (5 dots, current highlighted)

Currently visible column — "Qualification" (indigo):
- Column header: "Qualification | 8 deals | $240,000"
- Deal cards (stacked vertically, scrollable):
  - Card 1: "Enterprise License — TechVault" | $85,000 | James A. (avatar) | Close: Mar 30 | "42 days" badge
  - Card 2: "Platform Migration — NovaTech" | $62,000 | Maria S. (avatar) | Close: Apr 15 | "18 days" badge
  - Card 3: "Support Upgrade — Meridian" | $38,000 | Alex J. (avatar) | Close: Mar 20 | "7 days" badge
- "Add Deal" button at bottom of column

- Swipe left: shows "Proposal" column
- Swipe right: shows "Prospect" column

DEAL CARD TAP — Bottom Sheet (half-screen):
- Deal name and stage badge
- Company name (linked)
- Deal value: $85,000
- Probability: 65%
- Close date: Mar 30, 2026
- Owner: James Adeyemi (avatar + name)
- Next step: "Send revised proposal by Feb 25"
- Quick actions row: Call, Email, Note, Move Stage
- "View Full Details" link at bottom

FLOATING ACTION BUTTON:
- "+" icon — tap opens: "New Deal", "Log Activity", "Quick Search"

Include light and dark mode variants.
```

#### Prompt F-011: Mobile Contact Detail (390px)

```text
Design a mobile contact detail screen for ERP-CRM at 390px width (iPhone 14 frame).

TOP BAR:
- Back arrow
- Title: "Contact"
- Actions: Edit (pencil icon), More (three-dot menu)

PROFILE HEADER:
- Large avatar (64px) with online indicator
- Name: "Sarah Chen"
- Title: "VP of Engineering"
- Company: "TechVault Inc." (tappable, linked)
- Lead score badge: 87 (hot)
- Tags row: "Enterprise", "Decision Maker"

QUICK ACTIONS ROW (horizontal, icon + label):
- Call (phone icon)
- Email (mail icon)
- SMS (message icon)
- WhatsApp (WhatsApp icon)
- Log Activity (plus icon)

TABS (scrollable tab bar):
- Overview (active) | Activities | Deals | Tickets | Notes

OVERVIEW TAB (scrollable):

Contact Info Section:
- Email: sarah.chen@techvault.io (tap to email)
- Phone: +1 (415) 555-0142 (tap to call)
- Mobile: +1 (415) 555-0198 (tap to call)
- Location: San Francisco, CA
- Timezone: PST — "10:23 AM local time"

Company Section:
- Company: TechVault Inc.
- Industry: Technology
- Size: 500-1000 employees
- Revenue: $50M-100M

Open Deals Section:
- "Enterprise License Renewal" | $85K | Negotiation | 75%
- "Cloud Migration Add-on" | $42K | Proposal | 50%

Recent Activity Section (last 3):
- Feb 22 — Email sent: "Q1 Proposal"
- Feb 20 — Call: 15 min
- Feb 18 — Meeting: Product demo

Associated Contacts Section:
- Michael Torres — CTO
- Lisa Park — Eng Manager

FLOATING ACTION BUTTON:
- "Log Activity" — opens bottom sheet with: Call, Email, Meeting, Note, Task

Include light and dark mode variants.
```

#### Prompt F-012: Mobile Helpdesk Ticket (390px)

```text
Design a mobile helpdesk ticket view for ERP-CRM at 390px width (iPhone 14 frame).

TOP BAR:
- Back arrow
- Title: "Ticket #CRM-1247"
- Status badge: "Open" (blue)

TICKET HEADER CARD:
- Subject: "Cannot export contacts to CSV"
- Priority: High (amber badge)
- SLA: "2h 15m remaining" (amber countdown)
- Customer: Sarah Chen — TechVault Inc. (avatar + name)
- Assigned: Maria Santos (avatar + name, tappable to reassign)
- Tags: "Bug", "Export" (chips)
- Created: Feb 22, 2026

CONVERSATION THREAD (scrollable, takes majority of screen):
- Customer message (left-aligned, gray bubble):
  - "Sarah Chen — Feb 22, 09:15 AM"
  - "When I try to export my contacts to CSV, the download starts but the file is empty. I've tried with different filters but same result. This is blocking our quarterly review."

- Agent reply (right-aligned, indigo bubble):
  - "Maria Santos — Feb 22, 10:30 AM — via Email"
  - "Hi Sarah, thank you for reporting this. I can reproduce the issue on our end. Our engineering team is looking into it. As a workaround, could you try exporting from the Contacts list view instead of the search results?"

- Internal note (yellow background card, full width):
  - "Maria Santos — Feb 22, 11:00 AM — Internal Note"
  - "Checked with engineering — this is a known bug in v2.3.8. Fix deployed in v2.4.1 (releasing Feb 24)"

- Customer reply (left-aligned, gray bubble):
  - "Sarah Chen — Feb 23, 08:45 AM"
  - "The workaround works for now. When is the fix expected?"

REPLY COMPOSER (bottom, sticky):
- Toggle: "Reply" | "Note" (internal)
- Text input field with placeholder "Type your reply..."
- Action row: Attach file, Insert KB article, Canned response
- "Send" button (primary)

Include light and dark mode variants.
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: CRM Dashboard Tablet (1024px)

```text
Design the CRM command dashboard adapted for 1024px tablet width.

LAYOUT ADAPTATIONS FROM 1440px DESKTOP:
- Sidebar: collapsed to 64px (icon-only), expandable as overlay on tap
- Top bar: compressed, search becomes icon trigger opening full-width search overlay
- Page padding: reduced to 16px

Row 1 — Welcome & Quick Actions:
- Greeting text and quick action buttons stack into a full-width card
- Quick actions become 4-column icon grid

Row 2 — KPI Cards:
- Change from 4 columns to 2x2 grid (2 rows of 2 cards)

Row 3 — Charts:
- Change from 60/40 split to stacked vertically (pipeline chart full width, then lead sources full width below)

Row 4 — Tables:
- Change from 50/50 split to stacked vertically
- Tables show fewer columns (hide "Territory" and "Last Updated" columns, available via horizontal scroll or "View More")

Row 5 — Helpdesk table:
- Full width, show priority abbreviated (icon only instead of text badge)
- SLA timer condensed

INTERACTIONS:
- All touch targets remain minimum 44px
- Tables support horizontal swipe for hidden columns
- Charts are tap-interactive (tap bar/segment to see tooltip)
- Pull-to-refresh on main content area

Include both light and dark mode variants.
```

#### Prompt F-014: Pipeline Board Tablet (1024px)

```text
Design the pipeline board adapted for 1024px tablet width.

LAYOUT ADAPTATIONS:
- Sidebar collapsed to 64px
- Pipeline board shows 3 columns at a time (vs. 5-6 on desktop)
- Horizontal scroll with momentum scrolling to reach remaining columns
- Column width: approximately 280px each
- Stage indicator dots at top for orientation

- Deal cards: slightly more compact, reduce internal padding
- Card elements: company name, deal value, owner avatar, close date (days-in-stage hidden, shown on tap)

- Deal detail: opens as a bottom sheet (60% screen height) instead of right drawer
- Bottom sheet has drag handle, swipe down to dismiss

- Filter bar: collapses to a single "Filters" button that opens a full-width filter drawer
- Active filter count badge on the button

Include both light and dark mode variants.
```

---

## 4. Make Automation Prompts

### Prompt M-001: Lead Scoring & Routing Automation

```text
Create a Make (Integromat) scenario for ERP-CRM lead scoring and intelligent routing:

Trigger: Webhook — receives POST from ERP-CRM API when a new lead is created via form-builder-service
Payload: { leadId, firstName, lastName, email, phone, company, companySize, industry, source, formId, tenantId, utmSource, utmMedium, utmCampaign }

Step 1: HTTP Module — GET /v1/contact?email={email}
  - Check if contact already exists in contact-service
  - If exists: branch to "Existing Contact" path

Step 2: HTTP Module — POST /v1/lead/{leadId}/score
  - Calculate lead score based on:
    - Company size: Enterprise (30pts), Mid-market (20pts), SMB (10pts)
    - Industry match to ICP: Technology (30pts), Finance (25pts), Healthcare (20pts), Other (10pts)
    - Source quality: Referral (25pts), Event (20pts), Web Form (15pts), Cold Outreach (5pts)
    - UTM campaign bonus: specific high-intent campaigns (+10pts)
  - Update lead with calculated score

Step 3: Router — Branch by lead score
  - Branch A: Hot Lead (score >= 80)
  - Branch B: Warm Lead (score 40-79)
  - Branch C: Cold Lead (score < 40)

Branch A — Hot Lead:
  Step 4A: HTTP Module — GET /v1/territory?industry={industry}&region={region}
    - Find territory assignment from territory-service
  Step 5A: HTTP Module — PATCH /v1/lead/{leadId}
    - Assign to territory owner, set status to "Qualified"
  Step 6A: HTTP Module — POST /v1/activity
    - Create urgent follow-up task: "Hot lead — contact within 1 hour" via activity-service
  Step 7A: Slack/Teams — Send notification to sales channel
    - "Hot Lead Alert: {firstName} {lastName} from {company} (Score: {score}). Assigned to {ownerName}."
  Step 8A: Email — Send to assigned rep with lead details and suggested talking points

Branch B — Warm Lead:
  Step 4B: HTTP Module — PATCH /v1/lead/{leadId}
    - Auto-assign based on territory round-robin
  Step 5B: HTTP Module — POST /v1/activity
    - Create follow-up task: "Warm lead — contact within 24 hours"
  Step 6B: Email — Send welcome email to lead via automation-service template

Branch C — Cold Lead:
  Step 4C: HTTP Module — PATCH /v1/lead/{leadId}
    - Add to nurture campaign
  Step 5C: HTTP Module — POST /v1/automation/campaign/{nurtureCampaignId}/enroll
    - Enroll in automated nurture drip sequence

Error Handler: Log to error tracking sheet, send alert email to CRM admin
Schedule: Real-time (webhook trigger)
```

### Prompt M-002: Opportunity Stage Change Notifications

```text
Create a Make (Integromat) scenario for ERP-CRM opportunity stage change notifications:

Trigger: Webhook — receives POST from pipeline-service when an opportunity changes stage
Payload: { opportunityId, dealName, previousStage, newStage, amount, ownerId, ownerName, ownerEmail, companyName, probability, closeDate, tenantId }

Step 1: Router — Branch by stage transition type
  - Branch A: Advanced forward (e.g., Prospect -> Qualification, Qualification -> Proposal)
  - Branch B: Moved to "Negotiation" (high-value alert)
  - Branch C: Closed Won
  - Branch D: Closed Lost
  - Branch E: Moved backward (regression)

Branch A — Forward Progress:
  Step 2A: Slack/Teams — Post to sales channel
    - "Deal Update: {dealName} ({companyName}) moved from {previousStage} to {newStage}. Value: ${amount}. Owner: {ownerName}."

Branch B — Negotiation Stage:
  Step 2B: HTTP Module — GET /v1/opportunity/{opportunityId}/contacts
    - Fetch associated contacts and their roles
  Step 3B: Email — Notify sales manager
    - "High-value deal entering negotiation: {dealName} — ${amount}. Contacts: {contactList}. Close date: {closeDate}."
  Step 4B: HTTP Module — POST /v1/activity
    - Create task: "Review negotiation strategy for {dealName}" assigned to sales manager

Branch C — Closed Won:
  Step 2C: HTTP Module — GET /v1/opportunity/{opportunityId}
    - Fetch full deal details
  Step 3C: Slack/Teams — Post celebration to sales channel
    - "DEAL WON! {ownerName} closed {dealName} ({companyName}) for ${amount}! Pipeline contribution this month: ${monthTotal}."
  Step 4C: Email — Send congratulations to owner
  Step 5C: HTTP Module — POST /v1/activity
    - Create onboarding handoff task
  Step 6C: Google Sheets — Update sales tracker with won deal data

Branch D — Closed Lost:
  Step 2D: HTTP Module — GET /v1/opportunity/{opportunityId}
    - Fetch deal details including lost reason
  Step 3D: Email — Notify sales manager with loss analysis
  Step 4D: Google Sheets — Log lost deal for trend analysis
  Step 5D: HTTP Module — POST /v1/automation/campaign/{winbackCampaignId}/enroll
    - Enroll company contacts in win-back nurture campaign (90-day delay)

Branch E — Stage Regression:
  Step 2E: Email — Alert sales manager
    - "Deal Regression Alert: {dealName} moved BACK from {previousStage} to {newStage}. Owner: {ownerName}. Investigate."
  Step 3E: HTTP Module — POST /v1/activity
    - Create investigation task

Error Handler: Log failures and send admin alert
```

### Prompt M-003: Helpdesk SLA Breach Prevention

```text
Create a Make (Integromat) scenario for ERP-CRM helpdesk SLA breach prevention:

Trigger: Scheduled — runs every 15 minutes

Step 1: HTTP Module — GET /v1/helpdesk/tickets?status=open,pending&slaRisk=true
  - Fetch tickets from helpdesk-service where SLA is at risk (< 1 hour remaining)
  - Headers: Authorization Bearer token, X-Tenant-ID

Step 2: Iterator — Loop through at-risk tickets

Step 3: Router — Branch by SLA urgency
  - Branch A: SLA breach imminent (< 30 minutes remaining)
  - Branch B: SLA at risk (30-60 minutes remaining)
  - Branch C: Already breached

Branch A — Imminent Breach (< 30 min):
  Step 4A: HTTP Module — GET /v1/helpdesk/agents?status=available
    - Find available agents from helpdesk-service
  Step 5A: Router — Check if ticket is assigned
    - If unassigned: auto-assign to available agent with lowest queue
    - If assigned: escalate to team lead
  Step 6A: Slack/Teams — Send urgent alert
    - "@channel SLA ALERT: Ticket #{ticketId} ({subject}) — {timeRemaining} remaining. Customer: {customerName} ({company}). Priority: {priority}."
  Step 7A: SMS — Send text to assigned agent
    - "URGENT: Ticket #{ticketId} SLA expires in {timeRemaining}. Please respond immediately."
  Step 8A: HTTP Module — POST /v1/activity
    - Log escalation event via activity-service

Branch B — At Risk (30-60 min):
  Step 4B: Email — Notify assigned agent
    - "SLA Warning: Your ticket #{ticketId} ({subject}) has {timeRemaining} remaining. Please prioritize."
  Step 5B: Slack/Teams — Send warning to helpdesk channel

Branch C — Already Breached:
  Step 4C: HTTP Module — PATCH /v1/helpdesk/ticket/{ticketId}
    - Tag as "SLA Breached", escalate priority
  Step 5C: Email — Notify helpdesk manager
    - "SLA Breach Report: Ticket #{ticketId} from {customerName} ({company}). Breach time: {breachDuration}. Assigned: {agentName}."
  Step 6C: HTTP Module — POST /v1/reporting/sla-breach
    - Log breach event for SLA compliance reporting
  Step 7C: Google Sheets — Update SLA breach tracking spreadsheet

Error Handler: Continue on individual ticket failure, aggregate errors for daily report
```

### Prompt M-004: Weekly CRM Digest & Forecast Report

```text
Create a Make (Integromat) scenario for ERP-CRM weekly digest and sales forecast:

Trigger: Scheduled — runs every Monday at 7:00 AM

Step 1: HTTP Module — GET /v1/reporting/pipeline-summary?period=last_7_days
  - Fetch pipeline summary from reporting-service
  - Metrics: new deals, deals won, deals lost, pipeline value change, win rate

Step 2: HTTP Module — GET /v1/reporting/lead-summary?period=last_7_days
  - Fetch lead metrics: new leads, qualified leads, converted leads, top sources

Step 3: HTTP Module — GET /v1/reporting/activity-summary?period=last_7_days
  - Fetch activity metrics: calls made, emails sent, meetings held, tasks completed

Step 4: HTTP Module — GET /v1/reporting/helpdesk-summary?period=last_7_days
  - Fetch helpdesk metrics: new tickets, resolved tickets, avg resolution time, SLA compliance %

Step 5: HTTP Module — GET /v1/reporting/forecast?period=current_quarter
  - Fetch sales forecast: committed, best case, pipeline, target, gap analysis

Step 6: HTTP Module — GET /v1/reporting/top-performers?period=last_7_days&limit=5
  - Fetch top 5 reps by revenue won

Step 7: Text Aggregator — Compose digest report
  - Format all metrics into structured HTML email template with:
    - Week-over-week comparisons with up/down arrows
    - Traffic light indicators (green/amber/red) for KPIs vs. targets
    - Top performers leaderboard
    - Forecast gap analysis with AI recommendation
    - Key deals to watch this week

Step 8: Google Docs — Generate PDF report with branding

Step 9: Email — Send to sales leadership
  - Subject: "ERP-CRM Weekly Digest — Week of {date}"
  - Body: formatted digest with inline charts
  - Attachment: PDF report

Step 10: Slack/Teams — Post summary to #sales-leadership channel
  - Formatted message with key highlights and link to full report

Step 11: Google Sheets — Archive weekly metrics for trend analysis
  - Row: Week, New Leads, Qualified, Won Deals, Revenue, Win Rate, Pipeline Value, SLA %

Error Handler: On failure, send error notification to CRM admin with step details
```

### Prompt M-005: Chat-to-Ticket Escalation Automation

```text
Create a Make (Integromat) scenario for ERP-CRM chat-to-ticket escalation:

Trigger: Webhook — receives POST from chat-service when a chat session is flagged for escalation
Payload: { chatSessionId, visitorName, visitorEmail, visitorCompany, agentId, agentName, escalationReason, chatTranscript, sentiment, tenantId }

Step 1: HTTP Module — GET /v1/contact?email={visitorEmail}
  - Look up existing contact in contact-service
  - If found: enrich ticket with contact data (company, deals, previous tickets)

Step 2: HTTP Module — POST /v1/helpdesk/ticket
  - Create new ticket in helpdesk-service:
    - Subject: "Chat Escalation: {escalationReason}"
    - Description: Chat transcript formatted as conversation thread
    - Source: "Live Chat"
    - Priority: Based on sentiment (negative = High, neutral = Medium)
    - Customer: visitorEmail
    - Tags: ["chat-escalation", escalationReason]

Step 3: HTTP Module — GET /v1/knowledge-base/search?q={escalationReason}
  - Search knowledge-base-service for relevant articles
  - Attach top 3 article links to ticket as internal note

Step 4: Router — Branch by escalation reason
  - Branch A: "Technical Issue" — assign to technical support team
  - Branch B: "Billing Inquiry" — assign to billing team
  - Branch C: "Feature Request" — assign to product team
  - Branch D: "Complaint" — assign to senior support + notify manager

Step 5 (all branches): HTTP Module — POST /v1/activity
  - Log escalation activity via activity-service linking chat session, new ticket, and contact

Step 6: Email — Notify visitor
  - "Hi {visitorName}, we've created a support ticket (#{ticketId}) for your request. Our team will follow up within {slaTime}. In the meantime, these articles may help: {articleLinks}"

Step 7: Slack/Teams — Post to #support-escalations channel
  - "Chat Escalation: {visitorName} from {visitorCompany}. Reason: {escalationReason}. Sentiment: {sentiment}. Ticket: #{ticketId}. Assigned: {assignedTeam}."

Error Handler: If ticket creation fails, alert support manager and retain chat transcript
```

### Prompt M-006: Territory Performance Sync & Rebalancing Alert

```text
Create a Make (Integromat) scenario for ERP-CRM territory performance monitoring:

Trigger: Scheduled — runs daily at 6:00 AM

Step 1: HTTP Module — GET /v1/territory/all
  - Fetch all territories and their assignments from territory-service

Step 2: Iterator — For each territory:
  Step 3: HTTP Module — GET /v1/reporting/territory-performance?territoryId={id}&period=last_30_days
    - Fetch: lead count, opportunity count, pipeline value, revenue won, conversion rate, activity count
    - From reporting-service

Step 4: Aggregator — Collect all territory performance data

Step 5: Data Transform — Calculate territory health scores
  - Score based on: pipeline per rep, win rate vs. average, activity rate, lead coverage
  - Flag territories as: Healthy (green), Needs Attention (amber), Underperforming (red)

Step 6: Router — Branch by territory health
  - Branch A: All territories healthy — weekly summary only
  - Branch B: Territories needing attention — daily alert
  - Branch C: Underperforming territories — immediate alert with rebalancing recommendation

Branch B — Needs Attention:
  Step 7B: Email — Notify territory owners
    - "Territory Performance Alert: {territoryName} has {metric} below target. Current: {value}, Target: {target}. Trend: {trend}."

Branch C — Underperforming:
  Step 7C: Email — Notify sales VP
    - "Territory Rebalancing Recommended: {territoryName} is underperforming across {n} metrics. Suggested actions: {aiRecommendations}."
  Step 8C: Slack/Teams — Post to #sales-leadership
    - Formatted alert with territory comparison table
  Step 9C: HTTP Module — POST /v1/reporting/rebalancing-suggestion
    - Generate AI-powered rebalancing recommendation via reporting-service

Step 10: Google Sheets — Update territory performance tracking sheet
  - Row per territory: date, territory name, owner, leads, pipeline, revenue, win rate, health score

Error Handler: On failure, send admin notification with failed territory IDs
```

---

## 5. Prompt Usage Guidelines

### 5.1 Execution Order
1. Run **F-001** (Design Tokens) and **F-002** (Component Library) first to establish the design foundation
2. Run desktop page prompts (**F-003** through **F-008**) to create primary layouts
3. Run mobile prompts (**F-009** through **F-012**) for mobile-specific designs
4. Run tablet prompt (**F-013**, **F-014**) for responsive adaptations
5. Make automation prompts (**M-001** through **M-006**) can be executed independently

### 5.2 Customization
- Replace "Apex Solutions Ltd" and sample data with your organization's data
- Adjust currency symbols and formats for regional deployment
- Modify pipeline stage names and counts to match your sales process
- Add or remove sidebar navigation items based on licensed services

### 5.3 Design Quality Checks
- Every page must have: loading (skeleton), empty, populated, and error states
- Every interactive element must show: default, hover, focus, active, disabled states
- All forms must show: valid, invalid, and submitting states
- Dark mode must be tested for contrast compliance on every page

---

## 6. Output Packaging Convention

| Artifact | Format | Naming Convention |
|----------|--------|-------------------|
| Design tokens | Figma Variables / JSON | `erp-crm-tokens-v{version}` |
| Component library | Figma Library | `ERP-CRM Components v{version}` |
| Desktop pages | Figma Pages | `Desktop — {PageName}` |
| Mobile pages | Figma Pages | `Mobile — {PageName}` |
| Tablet pages | Figma Pages | `Tablet — {PageName}` |
| Prototype flows | Figma Prototype | `Flow — {JourneyName}` |
| Make blueprints | Make JSON export | `make-{scenario-name}-v{version}.json` |
| Design specs | Figma Dev Mode | Linked to each page |

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Lighthouse Performance | >= 95 | Per-route audit |
| Lighthouse Accessibility | >= 95 | Per-route audit |
| First Contentful Paint | < 1.2s | Lab + RUM |
| Largest Contentful Paint | < 2.5s | Lab + RUM |
| Cumulative Layout Shift | < 0.1 | Lab + RUM |
| Time to Interactive | < 3.0s | Lab measurement |
| Initial route bundle (gzip) | < 220KB | Build analysis |
| API response (p95) | < 500ms | Backend monitoring |
| Critical path completion | < 3 interactions | UX measurement |

---

## 8. AIDD Handoff Gate Template

Before any design is considered ready for development, verify:

| Gate | Check | Status |
|------|-------|--------|
| G-01 | All breakpoints designed (1440, 1024, 390) | Pending |
| G-02 | Light and dark mode variants complete | Pending |
| G-03 | All states covered (loading, empty, populated, error) | Pending |
| G-04 | WCAG 2.1 AA contrast ratios verified | Pending |
| G-05 | Touch targets >= 44px on all interactive elements | Pending |
| G-06 | Skeleton loaders designed for all data-dependent areas | Pending |
| G-07 | AI recommendations show Approve/Edit/Reject actions | Pending |
| G-08 | Destructive actions have two-step confirmation | Pending |
| G-09 | Audit trail badges present on key entities | Pending |
| G-10 | Tenant isolation indicator visible in header | Pending |
| G-11 | Analytics events documented per component | Pending |
| G-12 | Error recovery paths designed with retry and support link | Pending |
| G-13 | Design tokens reference Figma Variables (not hard-coded) | Pending |
| G-14 | Component auto-layout verified at all breakpoints | Pending |
| G-15 | Make automation scenarios tested with sample payloads | Pending |

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial Figma and Make design prompts for ERP-CRM covering 12 services: activity, automation, chat, contact, form-builder, helpdesk, knowledge-base, lead, opportunity, pipeline, reporting, territory |
