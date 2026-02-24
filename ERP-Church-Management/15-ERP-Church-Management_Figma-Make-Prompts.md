# Figma & Make Prompts — ERP-Church-Management
> Version: 2.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System
> Note: Enhanced version of source-monolith/docs/design/Figma_Make_Prompts.md (v1.0, 2026-02-18)

---

## 1. Purpose

This document provides structured, copy-paste-ready prompts for generating production-grade UI designs in Figma Make and automation workflows in Make (formerly Integromat) for the ERP-Church-Management module. The platform is a standalone-plus-suite faith/nonprofit vertical supporting the full spectrum of church operations: member care, visitor management, giving/tithes/offerings, small groups, events, volunteers, discipleship pathways, welfare/benevolence, facility management, follow-up automation, KPI tracking, and multi-channel communication. All prompts reference the 12 backend microservices (`communication-service`, `discipleship-service`, `event-service`, `facility-service`, `followup-service`, `giving-service`, `group-service`, `kpi-service`, `member-service`, `visitor-service`, `volunteer-service`, `welfare-service`) and are designed for multi-campus tenancy with ERP-IAM authentication (OIDC/JWT).

### Enhancements Over v1.0
- Full AIDD guardrails section with accessibility, performance, reliability, and observability requirements
- Expanded design token system with dark mode, elevation, and motion tokens
- Added comprehensive component library prompt with church-domain-specific components
- Expanded from 4 desktop pages to 6, with deeper feature coverage per page
- Added 4 dedicated mobile screens (390px) beyond the original 6-screen mobile prompt
- Added tablet/responsive adaptations (1024px)
- Expanded Make automations from 4 to 6 with deeper integration to all 12 services
- Added performance targets, handoff gates, and output packaging conventions
- All prompts now reference actual API endpoints from the service architecture

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance across all surfaces
- Minimum 44x44px touch targets on all interactive elements
- Clear visual states: default, hover, focus (2px ring), active, disabled, loading, error
- Plain-language labels; use warm, pastoral tone appropriate for church context
- Color-blind-safe palette; never rely on color alone to convey meaning
- Keyboard navigation and command palette (Cmd+K) parity for power users
- Screen reader announcements for dynamic content (toast, drawer, modal)
- Sensitive data handling: prayer requests, welfare cases, and financial data display with appropriate privacy controls

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading for all page modules
- Initial route gzip bundle < 220KB
- Skeleton UIs for every data-dependent component (tables, cards, charts, gauges)
- Optimistic UI for safe mutations (attendance check-in, activity logging)
- Autosave for long forms (member registration, welfare case notes)
- Image lazy loading with blur-up placeholders for member photos

### 2.3 Reliability, Trust, And Safety
- Recovery paths for all error states (retry button, support link, fallback view)
- Trust signals: "Last synced at HH:MM", "Saved", "Auto-saved draft"
- Audit trail badges: "Recorded by [user] on [date]" on giving records, welfare cases
- Two-step confirmation for destructive actions (delete member, void donation, close welfare case)
- Multi-campus tenant isolation: campus name always visible in header
- Human-in-the-loop for AI recommendations: Approve / Edit / Reject actions
- Financial integrity: giving records are append-only with correction workflow, never direct edit/delete

### 2.4 Observability And Testability
- Analytics events on: page view, CTA click, form submit, check-in action, giving record created
- Error events on: API failure, form validation failure, payment processing error
- Performance instrumentation: FCP, LCP, CLS, TTI per route
- Feature flag awareness: components degrade gracefully when features are disabled

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```text
Create a comprehensive design system for a church management platform called "ERP-Church-Management" (ChMS) that manages members, visitors, giving, groups, events, volunteers, discipleship, welfare, facilities, and communications for multi-campus churches.

DESIGN TOKENS:

Color System:
- Primary: Deep Royal Blue (#1E3A5F) representing trust, depth, and spirituality
- Secondary: Warm Gold (#D4A843) representing faith, warmth, and generosity
- Accent: Living Green (#2E7D32) for growth, discipleship progress, and success states
- Success: Emerald (#059669) for completed milestones, fulfilled welfare, resolved follow-ups
- Warning: Rich Amber (#E65100) for overdue follow-ups, SLA alerts, and attention items
- Danger: Muted Rose (#C62828) for breached timelines, critical alerts, and destructive actions
- Info: Calm Sky (#0277BD) for informational badges, tooltips, and help content
- Neutral: Slate scale (50-950) for text hierarchy, borders, backgrounds, and surfaces
- Surface hierarchy: Base (#F5F7FA), Raised (#FFFFFF with shadow-sm), Overlay (#FFFFFF with shadow-lg)
- KPI gauge colors: On-target (#4CAF50), Warning (#FF9800), Below-target (#F44336)
- Member type colors: New Believer (#42A5F5), Worker (#66BB6A), Minister (#AB47BC), Pastor (#EF5350)
- Dark mode: Slate-900 (#0F172A) base, Slate-800 (#1E293B) raised, all colors adjusted for dark background contrast

Typography:
- Font family: Inter (UI and body text) / JetBrains Mono (data tables, IDs, metrics)
- Headings: Inter Bold — H1: 32px, H2: 24px, H3: 20px, H4: 16px
- Body: Inter Regular — 16px body, 14px body-sm, 13px small, 12px caption, 11px overline
- Weight tokens: regular (400), medium (500), semibold (600), bold (700)
- Line-height tokens: tight (1.2), normal (1.5), relaxed (1.75)
- Numeric tabular figures for all tables, financial data, attendance counts, and KPI displays

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
- Only purposeful motion: state transitions, drawer reveals, chart entry animations, KPI gauge fills

Dark Mode:
- Background: #020617 base, #0F172A raised, #1E293B overlay
- Text: #F8FAFC primary, #CBD5E1 secondary, #64748B tertiary
- Borders: #334155
- Church brand colors retain warmth with adjusted saturation/lightness for dark surfaces
- KPI gauges and charts use lighter tints

Deliver as a structured token library with light/dark theme variants, auto-layout primitives, and church-specific color semantics.
```

#### Prompt F-002: Component Library

```text
Create a production-ready component library for ERP-Church-Management with these domain-specific components:

NAVIGATION:
- Sidebar navigation with sections: Dashboard, Members, Visitors, Events, Attendance, Groups, Discipleship, Giving, Welfare, Volunteers, Facilities, Communications, Reports, Settings
- Each item: icon (Lucide icon set) + label + optional count badge
- Collapsed (64px, icon-only) and expanded (256px) states
- Active state: left 3px accent bar + background highlight
- Top bar: ChMS logo, campus name ("Grace Community Church — Main Campus"), global search trigger, notification bell with unread count, user avatar with role badge (Admin, Pastor, Worker, Volunteer)
- Breadcrumb trail with clickable segments
- Command palette (Cmd+K) for searching members, visitors, events, groups, and actions

DATA DISPLAY:
- Data table with sortable headers, sticky header, row selection, bulk actions, pagination, CSV export
- Member card: avatar (photo or initials), full name, member type badge (New Believer/Worker/Minister/Pastor), natural group badge (Youth/Men/Women/Elders/Teens/Children), phone, email, last attendance date
- Visitor card: avatar, name, visit date, visit type badge (First Time/Returning/Referral), assigned officer avatar, follow-up status badge, "how heard" tag
- Giving record card: donor name, amount (currency formatted), giving type badge (Tithe/Offering/Special/Missions/Welfare), payment method icon, date, receipt status
- Event card: event name, date/time, location, registered count, capacity bar, status badge (Upcoming/Live/Completed/Cancelled)
- KPI gauge widget: circular gauge with percentage, target line indicator, label, trend arrow, color-coded (green/amber/red vs. target)
- Activity timeline: actor avatar, action description, entity link, timestamp, optional note
- Attendance trend sparkline: mini line chart for member attendance over last 12 weeks
- Welfare case card: case ID, beneficiary name, need category badge, urgency level, assigned team, status, created date

FORMS & INPUT:
- Text input with label, placeholder, helper text, validation error
- Select, multi-select, combobox with search
- Date picker, date range picker, time picker
- Rich text editor (for announcements, welfare case notes)
- File upload with drag-and-drop (for member documents, event flyers)
- Toggle switch, checkbox, radio group
- Currency input with currency symbol
- Phone input with country code selector
- Multi-step form wizard (for member registration, welfare case intake)
- Prayer request text area with privacy toggle (public/private)

FEEDBACK & OVERLAY:
- Toast notifications: success (green), error (red), warning (amber), info (blue) with auto-dismiss
- Modal dialog (sm, md, lg) with header, body, footer actions
- Drawer (right-slide) for member detail, visitor detail, giving detail
- Confirmation dialog with warmth: "Are you sure you want to archive this member?" with reason field
- Progress stepper for multi-step workflows (discipleship pathway, volunteer onboarding)
- Skeleton loaders matching each component shape
- Empty state with church-themed illustration, encouraging message, and CTA
- Error state with retry action and support link

CHURCH-SPECIFIC:
- Follow-up Kanban board: columns for New, In Progress, Completed, Overdue with visitor cards, drag-and-drop, 72-hour countdown timers
- Discipleship pathway tracker: horizontal stepper showing stages (New Believer Class, Water Baptism, Holy Ghost Baptism, Workers Training, Ministry Assignment) with completion checkmarks and dates
- Giving summary card: total giving by type (donut chart), year-to-date total, comparison to prior year
- Attendance trend chart: line chart with Sunday markers, average line, event annotation pins
- Volunteer schedule grid: calendar view with shift slots, volunteer name chips, conflict indicators
- Facility booking calendar: week/month view with room names, time slots, booking status colors (available/booked/maintenance)
- Campus selector: dropdown with campus name, location, pastor name, member count
- Welfare urgency indicator: traffic light (green/amber/red) with urgency label
- Prayer wall component: card layout of prayer requests with "Praying" reaction counter
- QR code check-in component: camera viewfinder overlay with alignment frame

ACCESSIBILITY:
- All interactive elements: visible focus rings (2px offset, primary color)
- Minimum contrast ratio 4.5:1 for text, 3:1 for UI elements
- All icons paired with labels or aria-labels
- Form inputs with associated labels, helper text, and error messages
- Screen reader announcements for dynamic content
- Keyboard navigation for all patterns

Deliver as a component library with variant properties, auto-layout, and design token references. Include both light and dark mode variants.
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Church Admin Dashboard (1440px)

```text
Design an admin dashboard page for ERP-Church-Management at 1440px desktop width with the following layout:

TOP BAR:
- Left: ChMS logo and campus name "Grace Community Church — Main Campus"
- Center: Global search bar with placeholder "Search members, visitors, events, groups..."
- Right: Notification bell with "12" count badge, user avatar "PJ" with name "Pastor James" and role badge "Admin"

LEFT SIDEBAR (expanded, 256px):
- Dashboard (active, highlighted with left accent bar)
- Members (badge: "1,247 active")
- Visitors (badge: "38 this month")
- Events (badge: "2 today")
- Attendance
- Groups (badge: "24 groups")
- Discipleship
- Giving
- Welfare (badge: "3 pending")
- Volunteers
- Facilities
- Communications
- KPI Reports
- Settings
- Collapse toggle at bottom

MAIN CONTENT AREA:

Row 1 — Welcome & Quick Actions:
- Greeting: "Good morning, Pastor James. Sunday service is in 3 days."
- Quick action buttons: "Register Visitor" (gold), "Record Giving" (green), "Check In" (blue), "Send Message" (indigo)

Row 2 — Summary Cards (4 columns):
- Active Members: 1,247 (green up arrow +12 this month)
- Recent Visitors: 38 (last 30 days, gold icon)
- Pending Follow-ups: 5 (amber warning icon, tappable)
- Today's Events: 2 ("Youth Bible Study" and "Choir Rehearsal", with total registrations)

Row 3 — KPI Gauges (5 columns):
- 72-Hour Contact Rate: 92% (green ring, target line at 90%)
- NBC Enrollment: 87% (amber ring, target line at 100%)
- Mentorship Completion: 88% (green ring, target line at 85%)
- Visitor Conversion Rate: 64% (green ring, target line at 60%)
- Welfare Cases Fulfilled: 18/20 (amber ring, target 20)
- Each gauge: circular with percentage, target line, trend arrow, and "View Details" link

Row 4 — Two Column Layout:
- Left (60%): "Attendance Trend" — line chart showing last 12 Sundays
  - X-axis: Sunday dates (Jan 5 through Mar 22, 2026)
  - Y-axis: attendance count (range 800-1400)
  - Data points with hover tooltips showing exact count
  - Average line (dashed) at 1,147
  - Annotation pin on Feb 16: "Special Service — 1,389 attendees"
  - Compare toggle: "vs. Same Period Last Year" (lighter line)

- Right (40%): "Giving Summary" — donut chart by type
  - Tithe: 45% ($234,500) — blue
  - Offering: 25% ($130,200) — gold
  - Special Offering: 15% ($78,100) — green
  - Missions: 10% ($52,100) — purple
  - Welfare: 5% ($26,000) — amber
  - Center: Total YTD $520,900
  - Legend with toggleable segments

Row 5 — Two Column Layout:
- Left (50%): "Recent Visitors" — table
  - Columns: Avatar, Name, Visit Date, Visit Type (badge), Assigned Officer (avatar+name), Follow-up Status (badge: New/In Progress/Completed/Overdue)
  - 5 sample visitors with diverse names
  - Overdue rows highlighted with amber background
  - "View All Visitors" link

- Right (50%): "Upcoming Events" — list cards
  - Event 1: "Sunday Worship Service" — Mar 1, 2026, 09:00 AM — Main Auditorium — 1,200 expected — "View" link
  - Event 2: "Youth Bible Study" — Feb 25, 2026, 06:00 PM — Fellowship Hall — 85 registered / 100 capacity — "View" link
  - Event 3: "Women's Conference" — Mar 8, 2026, 10:00 AM — Main Auditorium — 450 registered / 500 capacity — "Register" link
  - Event 4: "Marriage Enrichment Seminar" — Mar 15, 2026, 09:00 AM — Room 201 — 32 registered — "View" link
  - "View All Events" link

Row 6 — Bottom Panel:
- "Welfare Cases" — table
  - Columns: Case ID, Beneficiary, Need Category (badge: Financial/Medical/Housing/Food/Counseling), Urgency (traffic light), Assigned Team, Status (Open/In Progress/Fulfilled/Closed), Created Date
  - 4 sample cases
  - "View All Cases" link

Include hover states on all interactive elements. Show both light and dark mode variants. Use the ChMS design system colors: Deep Royal Blue primary, Warm Gold secondary, Living Green for success.
```

#### Prompt F-004: Member Directory & Profile (1440px)

```text
Design a member directory page for ERP-Church-Management at 1440px desktop width.

PAGE HEADER:
- Title: "Members" with subtitle "1,247 active members across 3 campuses"
- Actions: "Add Member" (primary blue button), "Import Members" (secondary), "Export" (icon), "Print Directory" (icon)
- View toggle: List (active) | Grid | Map

FILTER BAR:
- Search input with magnifying glass icon and placeholder "Search by name, phone, or email..."
- Campus: All | Main Campus | Satellite Campus A | Satellite Campus B
- Member Type: All | New Believer | Worker | Minister | Pastor
- Natural Group: All | Youth | Men | Women | Elders | Teens | Children
- Status: Active | Inactive | Transferred | Deceased
- Joined Date: date range picker
- Group: dropdown of all small groups
- Clear Filters button (text/link style)

DATA TABLE:
- Columns: Checkbox, Photo (40px circle avatar), Full Name (sortable), Phone, Email, Member Type (colored badge), Natural Group (badge), Campus, Status (green/gray/blue dot), Last Attendance, Actions (view, edit, message, more)
- 10 sample members with diverse names and varied data:
  1. Photo | "Grace Adeyemi" | +234 803 123 4567 | grace@email.com | Worker (green badge) | Women | Main Campus | Active (green) | Feb 23, 2026
  2. Photo | "Emmanuel Okonkwo" | +234 805 234 5678 | e.okonkwo@email.com | Minister (purple badge) | Men | Main Campus | Active | Feb 16, 2026
  3-10: Additional diverse members including youth, elders, new believers, inactive members
- Hover state: light blue row highlight
- Sortable columns indicated by chevron icons
- Selected row shows bulk action bar: "Send Message", "Add to Group", "Export Selected", "Print Labels"

PAGINATION BAR:
- "Showing 1-20 of 1,247 members"
- Page size selector: 20 | 50 | 100
- Previous / page numbers / Next buttons

MEMBER PROFILE DRAWER (slides in from right, 480px, when row clicked):
- Large avatar (80px) with edit photo icon overlay
- Full name: "Grace Adeyemi"
- Member type: Worker (green badge)
- Natural group: Women (badge)
- Campus: Main Campus
- Joined: January 15, 2022
- Status: Active (green dot)

Contact Information Section:
- Phone: +234 803 123 4567 (call icon, WhatsApp icon)
- Email: grace@email.com (email icon)
- Address: 15 Victoria Island, Lagos
- Emergency Contact: "John Adeyemi — +234 801 111 2222"

Spiritual Milestones Section (stepper/checklist):
- Salvation Decision: Jan 15, 2022 (green check)
- New Believer's Class: Feb 28, 2022 (green check)
- Water Baptism: Apr 10, 2022 (green check)
- Holy Ghost Baptism: Jun 5, 2022 (green check)
- Workers Training: Sep 1, 2022 (green check)
- Ministry Assignment: "Choir" — Nov 15, 2022 (green check)
- Next: "Leadership Training" — Enrolled (blue in-progress)

Quick Stats Row:
- Attendance Rate: 85% (green gauge)
- Groups: 2 (Women's Fellowship, Choir)
- Giving YTD: $4,250

Tabs: Overview | Attendance (mini sparkline chart) | Giving (summary with privacy mask) | Groups (list) | Activities (timeline) | Notes (private pastoral notes)

Quick Actions: "Edit Profile", "Assign to Group", "Send Message", "Record Visit"

Include light and dark mode variants.
```

#### Prompt F-005: Visitor Follow-up Board (1440px)

```text
Design an enhanced Kanban-style visitor follow-up board for ERP-Church-Management at 1440px desktop width.

PAGE HEADER:
- Title: "Visitor Follow-up Board" with subtitle "February 2026"
- Summary bar: "38 visitors this month | 5 pending | 30 completed | 3 overdue"
- Actions: "Register Visitor" (primary gold button), "Follow-up Settings" (icon), "Export Report" (icon)
- View toggle: Board (active) | List | Calendar
- Filter bar: Visit Type (All/First Time/Returning/Referral), Date Range, Assigned Officer, Campus, "How Heard" source

KANBAN BOARD (4 columns, horizontal scroll if needed):

Column 1 — "New" (Blue header bar, 5 cards):
- Column header: "New — 5 visitors" with "Sort" dropdown (newest first, oldest first)
- Visitor cards showing:
  - Avatar (photo or initials) + visitor name
  - Phone number (with call icon)
  - Visit date: "Feb 23, 2026"
  - Visit type badge: "First Time" (blue) or "Returning" (gold)
  - "How heard": "Friend Invitation" tag
  - Salvation decision indicator: green heart icon if yes
  - Prayer request preview (truncated, expand on hover): "Please pray for my family..."
  - Bottom: Assigned officer avatar + name (or "Unassigned" in red)
  - "Assign" button if unassigned
- Sample cards with diverse visitor names

Column 2 — "In Progress" (Amber/Yellow header bar, 8 cards):
- Column header: "In Progress — 8 visitors"
- Cards showing:
  - Visitor name, phone
  - Countdown timer prominently displayed:
    - Green timer (>24h remaining): "47:15:00"
    - Amber timer (12-24h remaining): "18:30:00"
    - Red timer (<12h remaining): "06:45:00"
  - Assigned officer avatar + name
  - Contact attempt count: "1 attempt" / "2 attempts"
  - Last contact method icon (phone/WhatsApp/SMS)
  - Notes preview: "Called, no answer. Will try WhatsApp."

Column 3 — "Completed" (Green header bar, 22 cards showing top 5):
- Column header: "Completed — 22 visitors"
- Cards showing:
  - Visitor name
  - Contact completed date
  - Officer name
  - Outcome badge: "Interested" (green), "Not Interested" (gray), "Undecided" (amber)
  - "Convert to Member" button (primary) for "Interested" visitors
  - "Schedule NBC" link for visitors ready for New Believer's Class
- "Load More" link at bottom

Column 4 — "Overdue" (Red header bar, 3 cards):
- Column header: "Overdue — 3 visitors" with alert icon
- Cards showing:
  - Visitor name with red border
  - Hours overdue in large red text: "12h overdue"
  - Original assigned officer (grayed out)
  - "Reassign" button (primary, amber)
  - "Escalate to Pastor" button (secondary)
  - Alert badge: "72-hour SLA breached"

CARD INTERACTIONS:
- Cards are draggable between columns (show elevation on drag)
- Drag to "Completed" opens a completion dialog: "Contact outcome?" with radio buttons (Interested/Not Interested/Undecided) + notes field + "Mark Complete" button
- Click on card opens detailed drawer with full visitor info, contact attempt history, prayer requests, and action buttons

VISITOR DETAIL DRAWER (400px, right slide):
- Full visitor profile with photo upload
- All contact attempts as a timeline
- Prayer request (full text, private to leadership)
- Quick actions: "Call", "WhatsApp Message", "Send SMS", "Log Visit Note", "Convert to Member"
- NBC enrollment section: "Enroll in next New Believer's Class (Mar 8, 2026)"

Include light and dark mode variants.
```

#### Prompt F-006: Giving Management Dashboard (1440px)

```text
Design a giving management dashboard for ERP-Church-Management at 1440px desktop width, powered by giving-service.

PAGE HEADER:
- Title: "Giving" with subtitle "February 2026"
- Actions: "Record Giving" (primary green button), "Generate Receipts" (secondary), "Export Report" (icon)
- Date range selector: "February 2026" with month/quarter/year/custom options
- Campus filter: "All Campuses"

TAB BAR:
- Overview (active) | Transactions | Reports | Pledges | Recurring | Settings

OVERVIEW TAB:

Row 1 — KPI Cards (4 columns):
- Total Giving (Month): $87,450 (up 8% vs. last month, green arrow, sparkline)
- Tithes: $39,350 (45% of total, blue badge)
- Offerings: $21,860 (25% of total, gold badge)
- Special/Missions: $26,240 (30% of total, green badge)

Row 2 — Two Panels:
- Left (55%): "Giving Trend" — area chart showing last 12 months
  - X-axis: months (Mar 2025 — Feb 2026)
  - Y-axis: amount in dollars
  - Stacked areas: Tithes (blue), Offerings (gold), Special (green), Missions (purple), Welfare (amber)
  - Hover tooltips with breakdown
  - Compare toggle: "vs. Previous Year"

- Right (45%): "Giving by Type" — horizontal bar chart
  - Tithe: $39,350 (45%)
  - Offering: $21,860 (25%)
  - Special Offering: $13,120 (15%)
  - Missions: $8,745 (10%)
  - Welfare: $4,375 (5%)
  - Each bar color-coded with percentage label

Row 3 — Three Panels:
- Left (33%): "Payment Methods" — donut chart
  - Bank Transfer: 40%
  - Mobile Money: 25%
  - Cash: 20%
  - Card: 10%
  - USSD: 5%

- Center (33%): "Top Giving Categories" — vertical bar chart by campus
  - Main Campus: $52,470
  - Satellite A: $21,860
  - Satellite B: $13,120

- Right (33%): "Giving Health" card
  - Active Tithers: 487 of 1,247 members (39%)
  - New Tithers This Month: 12
  - Lapsed Tithers (90+ days): 34 (amber warning)
  - Average Gift: $179
  - Recurring Giving Setup: 156 members (12.5%)
  - "View Lapsed Tithers" link

Row 4 — Recent Transactions Table:
- Columns: Date, Donor Name (avatar), Amount ($), Type (badge), Payment Method (icon), Campus, Recorded By, Receipt Sent (check/pending), Actions (view, receipt, void)
- 8 sample transactions with diverse donors and types
- Privacy note: "Giving amounts visible to Finance and Leadership roles only"
- Bulk actions: "Generate Receipts", "Export Selected", "Send Thank You"

Row 5 — Pledge Tracking Card:
- Active Pledges: 45
- Total Pledged: $125,000
- Fulfilled: $87,500 (70%)
- Progress bar with fulfillment percentage
- "5 pledges behind schedule" (amber warning link)
- "View All Pledges" link

RECORD GIVING MODAL (triggered by "Record Giving" button):
- Step 1: Member lookup (search by name/phone/email, or "Guest Donor" toggle)
- Step 2: Giving details
  - Amount: currency input ($)
  - Type: Tithe / Offering / Special Offering / Missions / Welfare / Building Fund (radio or select)
  - Payment Method: Cash / Bank Transfer / Mobile Money / Card / USSD / Check
  - Date: date picker (default today)
  - Reference: optional text input for transaction reference
  - Campus: auto-filled based on user, changeable
  - Notes: optional text area
- Step 3: Review and confirm
  - Summary of all entered data
  - "Record Giving" button (primary green)
  - Confirmation toast: "Giving of $XXX recorded for [Member Name]"

Include light and dark mode variants.
```

#### Prompt F-007: Event & Volunteer Management (1440px)

```text
Design an event management page with integrated volunteer scheduling for ERP-Church-Management at 1440px desktop width, powered by event-service and volunteer-service.

PAGE HEADER:
- Title: "Events & Volunteers"
- Actions: "Create Event" (primary blue), "Manage Volunteers" (secondary), "Facility Calendar" (icon)
- View toggle: Calendar (active) | List | Board

TAB BAR:
- Events (active) | Volunteer Schedule | Facility Bookings

EVENTS TAB — CALENDAR VIEW:
- Month view calendar (March 2026)
- Navigation: < February | March 2026 | April >
- View options: Month | Week | Day
- Campus filter: "All Campuses"

Calendar Grid:
- Sunday cells highlighted (primary worship days)
- Event chips on each day:
  - "Sunday Service" (blue, recurring) — every Sunday
  - "Youth Bible Study" (green) — every Wednesday
  - "Women's Conference" (purple) — Mar 8, all-day
  - "Marriage Seminar" (gold) — Mar 15
  - "Easter Outreach Planning" (amber) — Mar 22
  - "Good Friday Service" (blue, special) — Mar 29
- Click event chip: opens event detail popover
- Click empty day: opens "Create Event" pre-filled with date

EVENT DETAIL POPOVER (on event click):
- Event name: "Women's Conference"
- Date: March 8, 2026, 10:00 AM - 4:00 PM
- Location: Main Auditorium (linked to facility-service)
- Capacity: 500 seats
- Registered: 450 (90% capacity progress bar, amber)
- Volunteers Assigned: 25 of 30 needed (progress bar, amber)
- Categories: "Conference", "Women's Ministry"
- Actions: "Edit Event", "Register Members", "Assign Volunteers", "Send Reminder", "View Details"

EVENT LIST VIEW (below calendar or as tab):
- Table: Event Name, Date/Time, Location, Registered/Capacity, Volunteer Status, Status (Upcoming/Live/Completed), Campus, Actions
- 6 sample events with varied types

EVENT CREATION MODAL:
- Event Name, Description (rich text)
- Date/Time: start, end, all-day toggle, recurring options (weekly, monthly, custom)
- Location: dropdown from facility-service available rooms
- Capacity: number input
- Registration: Open/Closed/Invite-only toggle
- Volunteer Requirements: role slots (Ushers: 10, Choir: 15, Tech: 5, Parking: 8)
- Campus: selector
- Categories/Tags
- Communication: auto-send invitation, reminder schedule (1 week, 1 day, 1 hour before)

VOLUNTEER SCHEDULE TAB:
- Week view calendar with volunteer names in time slot cells
- Columns: Sunday (AM Service, PM Service), Wednesday (Bible Study), Saturday (Prayer Meeting)
- Rows: Volunteer roles (Ushers, Greeters, Choir, Tech/AV, Children's Ministry, Parking, First Aid)
- Cell content: volunteer name chips (green = confirmed, amber = pending, gray = declined)
- Drag-and-drop to reassign volunteers
- Conflict indicators (red border) when a volunteer is double-booked
- "Fill Gaps" button: AI-suggested volunteers based on availability, skills, and rotation fairness
- Volunteer detail popover: name, phone, skills, availability, service history, reliability score

FACILITY BOOKINGS TAB:
- Week/month view of rooms from facility-service
- Rows: Room names ("Main Auditorium", "Fellowship Hall", "Room 201", "Room 202", "Youth Center", "Parking Lot")
- Cells: booking blocks with event name, time, booker
- Color coding: Available (green), Booked (blue), Maintenance (gray), Conflict (red)
- Click to book or view booking details

Include light and dark mode variants.
```

#### Prompt F-008: Discipleship & Group Management (1440px)

```text
Design a discipleship pathway and group management page for ERP-Church-Management at 1440px desktop width, powered by discipleship-service and group-service.

PAGE HEADER:
- Title: "Discipleship & Groups"
- Actions: "Enroll Member" (primary green), "Create Group" (secondary), "Pathway Settings" (icon)

TAB BAR:
- Discipleship Pathways (active) | Small Groups | Mentorship

DISCIPLESHIP PATHWAYS TAB:

Pathway Overview (full-width card):
- Visual pipeline/funnel showing the discipleship pathway stages:
  - Stage 1: "New Believer's Class (NBC)" — 45 enrolled, 92% completion — green
  - Stage 2: "Water Baptism" — 38 candidates, next date: Mar 15 — blue
  - Stage 3: "Holy Ghost Baptism" — 35 completed this quarter — purple
  - Stage 4: "Workers Training" — 28 in progress, 12-week program — amber
  - Stage 5: "Ministry Assignment" — 22 assigned, 6 pending placement — green
  - Stage 6: "Leadership Development" — 15 enrolled, advanced track — gold
- Each stage: card with count, completion rate, next milestone date, "View Enrolled" link
- Flow arrows connecting stages with conversion percentages between them

Enrollment Table (below pathway):
- Columns: Member (avatar + name), Current Stage (badge), Stage Progress (progress bar), Started Date, Expected Completion, Mentor (avatar + name), Campus, Actions (advance, hold, notes)
- Filter: Stage, Campus, Mentor, Status (On Track/Behind/Completed)
- 8 sample members at various stages
- "Behind Schedule" rows highlighted amber with "X days behind" indicator

AI INSIGHT CARD:
- "12 members have completed Workers Training but are not yet assigned to a ministry. Consider scheduling ministry placement meetings."
- Confidence: 95%
- Buttons: "View Members" (primary), "Schedule Meetings" (secondary), "Dismiss"

SMALL GROUPS TAB:
- Group list: grid of group cards (3 per row)
- Each group card:
  - Group name: "Faith Builders Men's Fellowship"
  - Category badge: "Men's Ministry" / "Youth" / "Couples" / "Mixed"
  - Leader: avatar + name
  - Members: "18 members" (avatar stack showing 5 + "+13")
  - Meeting schedule: "Wednesdays, 6:30 PM"
  - Location: "Room 201" or "Online (Zoom)"
  - Attendance trend: mini sparkline (last 8 weeks)
  - Status: Active (green dot) / Paused (amber) / Closed (gray)
  - Actions: "View", "Edit", "Message Group"

- Group creation modal:
  - Name, Category, Description
  - Leader assignment (search members)
  - Schedule: day, time, frequency
  - Location: physical (from facility-service) or virtual (link)
  - Max capacity
  - Open/Closed enrollment
  - Campus

- Group detail page (click "View"):
  - Group header with all info
  - Member list with attendance record
  - Attendance chart (last 12 weeks)
  - Meeting notes history
  - Communication log
  - Group health indicators: average attendance %, member engagement, leader feedback

MENTORSHIP TAB:
- Mentor-mentee pairing list
  - Columns: Mentor (avatar + name), Mentee (avatar + name), Program (badge), Started, Progress (bar), Last Meeting, Next Meeting, Status
  - 6 sample pairings
- "Create Pairing" button
- Mentorship program templates: "New Believer Mentorship", "Leadership Mentorship", "Marriage Mentorship"
- Meeting log: date, duration, topics discussed, action items
- Completion criteria checklist per program

Include light and dark mode variants.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Church Mobile Dashboard (390px)

```text
Design a mobile dashboard for ERP-Church-Management at 390px width (iPhone 14 frame, 390x844px).

STATUS BAR: standard iOS status bar

TOP BAR:
- Left: hamburger menu icon
- Center: "ChMS" logo
- Right: notification bell with "12" badge, user avatar "PJ"

MAIN CONTENT (scrollable):

Greeting Section:
- "Shalom, Pastor James"
- "Sunday service is in 3 days. 38 visitors this month."

KPI Gauge Row (horizontal scroll, 2 visible + peek):
- 72-Hour Contact: 92% (green)
- NBC Enrollment: 87% (amber)
- Mentorship: 88% (green)
- Visitor Conversion: 64% (green)
- Welfare: 18/20 (amber)

Quick Actions Grid (2x2):
- Register Visitor (gold icon)
- Record Giving (green icon)
- Check In (blue icon)
- Send Message (indigo icon)

Follow-up Alert Card (if follow-ups pending):
- "5 visitor follow-ups pending"
- "3 overdue — oldest: 18 hours" (red text)
- "View Follow-ups" button (primary)

Today's Events Card:
- "2 events today"
- "Youth Bible Study" — 6:00 PM — Fellowship Hall — 85 registered
- "Choir Rehearsal" — 7:30 PM — Main Auditorium — 40 expected
- "View All Events" link

Recent Visitors Card:
- 3 most recent visitors with avatar, name, visit type badge, follow-up status
- "View All Visitors" link

Giving Summary Card:
- "February 2026 — $87,450 total"
- Mini donut chart: Tithe/Offering/Special breakdown
- "Record Giving" link | "View Details" link

BOTTOM NAVIGATION BAR (5 items):
- Dashboard (active, filled icon)
- Members
- Visitors (with "5" badge for pending follow-ups)
- Events
- More

NAVIGATION DRAWER (from hamburger):
- Campus selector at top: "Main Campus" dropdown
- User profile section: avatar, name, role badge
- Full menu: Dashboard, Members, Visitors, Events, Attendance, Groups, Discipleship, Giving, Welfare, Volunteers, Facilities, Communications, KPI Reports, Settings
- Logout at bottom

Include light and dark mode variants.
```

#### Prompt F-010: Mobile Visitor Registration (390px)

```text
Design a mobile visitor registration flow for ERP-Church-Management at 390px width (iPhone 14 frame).

SCREEN 1 — VISITOR REGISTRATION FORM (scrollable):

TOP BAR:
- Back arrow
- Title: "Register Visitor"
- "Save Draft" link

Form Sections:

Personal Information:
- First Name (text input, required)
- Last Name (text input, required)
- Phone Number (phone input with country code +234, required)
- Email (email input, optional)
- Photo (camera button to take photo or select from gallery)

Visit Details:
- Visit Date: date picker (default: today)
- Visit Type: horizontal chip selector — "First Time" (selected by default, blue) | "Returning" (gold) | "Referral" (green)
- Campus: auto-filled based on user, changeable dropdown
- Service Attended: dropdown — "Sunday 1st Service" / "Sunday 2nd Service" / "Midweek" / "Special Event"
- How Did You Hear About Us?: dropdown — "Friend/Family" / "Social Media" / "Walk-in" / "Outreach Event" / "Online" / "Other"
- If "Referral": Referred By field (member search)

Spiritual:
- Salvation Decision Today? toggle (yes/no)
  - If yes: show "Decision Type" chips: "First Time" | "Rededication"
- Interested in New Believer's Class? toggle

Prayer Request:
- Text area with placeholder "Share your prayer request (confidential)..."
- Privacy note: "Prayer requests are visible only to the pastoral team"

SUBMIT SECTION (sticky bottom):
- "Register Visitor" primary button (full width, gold)
- "Register & Add Another" secondary button below

SCREEN 2 — REGISTRATION SUCCESS:
- Green checkmark animation
- "Visitor Registered Successfully!"
- Visitor name and photo preview
- "Auto-assigned to: Sister Grace (Account Officer)"
- "72-hour follow-up timer started"
- Action buttons:
  - "Register Another Visitor" (primary)
  - "View Visitor" (secondary)
  - "Back to Dashboard" (text link)

Include light and dark mode variants.
```

#### Prompt F-011: Mobile QR Check-In (390px)

```text
Design a mobile QR check-in screen for ERP-Church-Management at 390px width (iPhone 14 frame).

SCREEN 1 — QR SCANNER:
- Camera viewfinder taking 70% of screen
- Semi-transparent dark overlay around viewfinder
- White corner brackets framing the QR code target area
- Instruction text above viewfinder: "Point camera at event QR code"
- Event selector below viewfinder: "Sunday 1st Service — Feb 23, 2026" (tappable dropdown)
- Flash toggle button (top right corner)
- "Manual Check-In" text button below viewfinder
- "Cancel" button at bottom

SCREEN 2 — MEMBER IDENTIFIED (after QR scan):
- Green checkmark animation
- Member avatar (large, 80px)
- Name: "Grace Adeyemi"
- Member Type: "Worker" (green badge)
- Group: "Women's Fellowship"
- Check-in status: "Checked In at 09:03 AM" (green text)
- Attendance streak: "12 consecutive Sundays" (trophy icon)
- "Check In Another" button (primary)
- "View Profile" link

SCREEN 3 — MANUAL CHECK-IN:
- Search bar: "Search member by name or phone..."
- Recent check-ins list (last 5) for quick re-check-in
- Search results: member cards with avatar, name, member type badge
- Tap member card: shows confirmation "Check in Grace Adeyemi?" with "Confirm" button
- "Register as Visitor" button at bottom for unknown attendees
- Attendance counter at top: "347 checked in" (live updating)

SCREEN 4 — BULK CHECK-IN (for group leaders):
- Group selector: "Faith Builders Men's Fellowship"
- Member list with checkboxes (all pre-checked)
- Uncheck absent members
- "Check In 15 of 18 Members" button (primary)
- Absent members auto-flagged for follow-up

Include light and dark mode variants.
```

#### Prompt F-012: Mobile Giving & Wallet (390px)

```text
Design mobile giving screens for ERP-Church-Management at 390px width (iPhone 14 frame).

SCREEN 1 — GIVING HOME:

TOP BAR:
- Back arrow
- Title: "Giving"
- History icon

Summary Card (full width):
- "Your Giving — February 2026"
- Total: $1,250.00
- Breakdown row: Tithe: $800 | Offering: $300 | Missions: $150
- "View Statement" link

Quick Give Buttons (horizontal scroll):
- "Tithe" (blue circle icon)
- "Offering" (gold circle icon)
- "Special" (green circle icon)
- "Missions" (purple circle icon)
- "Welfare" (amber circle icon)
- "Building Fund" (gray circle icon)

Recent Transactions (list):
- "Tithe — $400.00 — Feb 16, 2026 — Bank Transfer" (receipt icon)
- "Offering — $150.00 — Feb 16, 2026 — Mobile Money"
- "Tithe — $400.00 — Feb 9, 2026 — Bank Transfer"
- "Missions — $150.00 — Feb 2, 2026 — Card"
- "View All Transactions" link

Recurring Giving Card:
- "Active Recurring: Monthly Tithe — $400.00 — 1st of each month"
- "Manage Recurring" link

Pledge Card:
- "Building Fund Pledge: $5,000 — $2,500 fulfilled (50%)"
- Progress bar
- "View Pledge" link

SCREEN 2 — RECORD GIVING (from Quick Give tap):
- Title: "Give — Tithe"
- Amount input (large, centered, currency formatted): "$0.00"
  - Preset amount chips: "$100", "$250", "$500", "$1,000", "Custom"
- Payment Method selector:
  - Bank Transfer (selected)
  - Mobile Money (MTN, Airtel, Glo)
  - Debit/Credit Card
  - Cash (for in-person recording by officers)
- Recurring toggle: "Make this recurring?"
  - If yes: frequency chips (Weekly/Monthly/Quarterly), start date
- Reference/Note: optional text input
- "Give $400.00" primary button (full width, green)
- Security badge: "Secure & encrypted" with lock icon

SCREEN 3 — GIVING CONFIRMATION:
- Green checkmark animation
- "Thank You!"
- "Your tithe of $400.00 has been recorded."
- Receipt number: "GV-2026-02-23-0047"
- "Download Receipt" button
- "Share Receipt" button (WhatsApp, Email)
- "Give Again" link | "Back to Giving" link

SCREEN 4 — GIVING STATEMENT:
- Period selector: "2026" dropdown
- Monthly summary list:
  - January 2026: $1,150.00 (Tithe: $800, Offering: $200, Missions: $150)
  - February 2026: $1,250.00 (Tithe: $800, Offering: $300, Missions: $150)
- Year-to-date total: $2,400.00
- "Download Annual Statement (PDF)" button
- "Email Statement" button

Include light and dark mode variants.
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Church Dashboard Tablet (1024px)

```text
Design the church admin dashboard adapted for 1024px tablet width.

LAYOUT ADAPTATIONS FROM 1440px DESKTOP:
- Sidebar: collapsed to 64px (icon-only with tooltips), expandable as overlay on tap
- Top bar: compressed, search becomes icon trigger opening full-width search overlay
- Page padding: reduced to 16px

Row 1 — Welcome & Quick Actions:
- Greeting text left-aligned, quick action buttons in a single row (4 icons with labels below)

Row 2 — Summary Cards:
- Change from 4 columns to 2x2 grid

Row 3 — KPI Gauges:
- Change from 5 columns to horizontal scroll row (3 visible + peek)
- Each gauge slightly smaller (120px diameter instead of 140px)

Row 4 — Charts:
- Change from 60/40 split to stacked vertically
- Attendance chart full width (reduce to last 8 Sundays for space)
- Giving donut full width below

Row 5 — Tables:
- Stacked vertically instead of side-by-side
- "Recent Visitors" table: hide "Campus" column, show fewer rows (4)
- "Upcoming Events" list: show 3 events instead of 4

Row 6 — Welfare Cases:
- Full width, condensed layout with fewer visible columns

INTERACTIONS:
- All touch targets remain minimum 44px
- Tables support horizontal swipe for hidden columns
- Charts are tap-interactive
- Pull-to-refresh on main content area
- KPI gauges tap to expand detail

Include both light and dark mode variants.
```

#### Prompt F-014: Visitor Follow-up Board Tablet (1024px)

```text
Design the visitor follow-up Kanban board adapted for 1024px tablet width.

LAYOUT ADAPTATIONS:
- Sidebar collapsed to 64px
- Kanban board shows 2 columns at a time (vs. 4 on desktop)
- Horizontal scroll with momentum to reach remaining columns
- Column width: approximately 440px each
- Column indicator dots at top for orientation (4 dots)

- Visitor cards: maintain full information density but reduce padding
- Countdown timers: keep prominent display
- Officer avatars: smaller (28px instead of 32px)

- Visitor detail: opens as bottom sheet (70% screen height) instead of right drawer
- Bottom sheet has drag handle, swipe down to dismiss

- Filter bar: collapses to "Filters" button with active filter count badge
- Tapping opens a full-width dropdown with filter options

- Summary bar: condensed to single line "38 visitors | 5 pending | 3 overdue"

Include both light and dark mode variants.
```

---

## 4. Make Automation Prompts

### Prompt M-001: Visitor Welcome & Assignment Automation

```text
Create a Make (Integromat) scenario for ChMS visitor welcome and officer assignment automation:

Trigger: Webhook — receives POST from visitor-service when a new visitor is registered
Payload: { visitorId, firstName, lastName, phone, email, visitType, campusId, howHeard, salvationDecision, prayerRequest, tenantId }

Step 1: Router — Branch based on visitType
  - Branch A: "First Time" visitors
  - Branch B: "Returning" visitors
  - Branch C: "Referral" visitors

Branch A — First Time Flow:
  Step 2A: HTTP Module — GET /v1/followup/officers?campusId={campusId}&available=true
    - Fetch available account officers from followup-service
    - Select officer with lowest active assignment count (round-robin fairness)
  Step 3A: HTTP Module — POST /v1/followup/assign
    - Assign visitor to selected officer via followup-service
    - Set 72-hour SLA timer
  Step 4A: HTTP Module — POST /v1/communication/whatsapp
    - Send personalized welcome message via communication-service:
      "Hello {firstName}, welcome to Grace Community Church! We are so glad you visited us today. A member of our care team will be in touch within 24 hours. God bless you!"
  Step 5A: HTTP Module — POST /v1/communication/notification
    - Notify assigned officer via push notification and email:
      "New visitor assigned: {firstName} {lastName} — Phone: {phone} — Visit: {visitType} — 72-hour timer started."
  Step 6A: Router — Check salvationDecision
    - If true: HTTP Module — POST /v1/discipleship/enroll
      - Auto-enroll in next New Believer's Class via discipleship-service
      - Send enrollment confirmation message
  Step 7A: Google Sheets — Log visitor in "Visitor Tracking" sheet
    - Row: Date, Name, Phone, Visit Type, How Heard, Assigned Officer, Follow-up Status, Campus

Branch B — Returning Visitor:
  Step 2B: HTTP Module — GET /v1/visitor/{email_or_phone}
    - Look up previous visit record
  Step 3B: HTTP Module — POST /v1/communication/whatsapp
    - "Welcome back, {firstName}! Great to see you again at Grace Community Church."
  Step 4B: HTTP Module — PATCH /v1/visitor/{visitorId}
    - Update visit count, flag for conversion consideration if 3+ visits

Branch C — Referral:
  Step 2C: HTTP Module — POST /v1/communication/whatsapp (to referrer)
    - "Thank you for inviting {firstName}! They visited us today."
  Step 3C: Continue with Branch A flow for the visitor

Error Handler: Send alert email to church admin, log to error sheet
Schedule: Real-time (webhook trigger)
```

### Prompt M-002: Absentee Follow-Up Automation

```text
Create a Make (Integromat) scenario for ChMS absentee member follow-up:

Trigger: Scheduled — runs daily at 8:00 AM (Monday through Saturday)

Step 1: HTTP Module — GET /v1/member?status=active&lastAttendance=before_{14_days_ago}
  - Fetch members absent for 14+ days from member-service
  - Headers: Authorization Bearer token, X-Tenant-ID

Step 2: Iterator — Loop through absentee list

Step 3: Router — Branch by absence duration
  - Branch A: 14-21 days absent (gentle check-in)
  - Branch B: 21-30 days absent (pastoral concern)
  - Branch C: 30-60 days absent (welfare check)
  - Branch D: 60+ days absent (retention outreach)

Branch A — 14-21 Days (Gentle Check-in):
  Step 4A: HTTP Module — POST /v1/communication/whatsapp via communication-service
    - "Hi {firstName}, we've missed you at church! Is everything okay? We'd love to see you this Sunday. God bless!"
  Step 5A: HTTP Module — POST /v1/followup/create
    - Create follow-up task for member's group leader via followup-service

Branch B — 21-30 Days (Pastoral Concern):
  Step 4B: HTTP Module — POST /v1/communication/whatsapp
    - "Dear {firstName}, we've been thinking about you and your family. Is there anything we can help with? Please don't hesitate to reach out. We care about you."
  Step 5B: HTTP Module — POST /v1/communication/email via communication-service
    - Notify group leader and assigned pastor
  Step 6B: HTTP Module — POST /v1/followup/create
    - Create follow-up task with "Pastoral Visit" type assigned to zone pastor

Branch C — 30-60 Days (Welfare Check):
  Step 4C: HTTP Module — POST /v1/welfare/case via welfare-service
    - Create welfare check case:
      - Beneficiary: {memberId}
      - Category: "Welfare Check"
      - Urgency: "medium"
      - Notes: "Member absent for {days} days. Automated welfare check initiated."
  Step 5C: HTTP Module — POST /v1/communication/notification
    - Notify welfare team and senior pastor via communication-service
  Step 6C: HTTP Module — POST /v1/communication/sms
    - Send caring SMS to member

Branch D — 60+ Days (Retention Outreach):
  Step 4D: HTTP Module — PATCH /v1/member/{memberId}
    - Flag as "At Risk of Inactivity" via member-service
  Step 5D: HTTP Module — POST /v1/communication/email
    - Send personalized letter from senior pastor
  Step 6D: HTTP Module — POST /v1/kpi/event via kpi-service
    - Log retention risk event for KPI tracking

Step 7: Google Sheets — Log all absentee contacts
  - Row: Date, Member Name, Days Absent, Action Taken, Branch, Assigned To, Campus

Error Handler: Continue on individual member failure, aggregate errors for daily admin report
```

### Prompt M-003: Giving Acknowledgment & Receipt Automation

```text
Create a Make (Integromat) scenario for ChMS giving acknowledgment and receipt generation:

Trigger: Webhook — receives POST from giving-service when a giving record is created
Payload: { givingId, memberId, memberName, amount, givingType, paymentMethod, date, campusId, tenantId, receiptRequired }

Step 1: HTTP Module — GET /v1/member/{memberId} via member-service
  - Fetch member contact details (phone, email, preferred contact method)

Step 2: Router — Branch by giving type
  - Branch A: Tithe — formal acknowledgment with tax receipt
  - Branch B: Offering/Special Offering — warm thank you
  - Branch C: Missions — impact update message
  - Branch D: Welfare/Building Fund — project update message

Step 3 (all branches): HTTP Module — POST /v1/communication/whatsapp via communication-service
  - Personalized thank you message:
    - Tithe: "Dear {firstName}, thank you for your faithful tithe of {amount}. 'Bring the whole tithe into the storehouse' — Malachi 3:10. Your receipt is attached."
    - Offering: "Thank you, {firstName}, for your generous offering of {amount}. Your generosity blesses our church family!"
    - Missions: "Thank you, {firstName}, for your missions gift of {amount}. Your giving reaches the nations!"
    - Welfare: "Thank you, {firstName}, for your welfare contribution of {amount}. You are helping families in need."

Step 4: Router — Check if receiptRequired or givingType === 'Tithe'
  - If yes: Generate receipt
    - HTTP Module — POST /v1/giving/{givingId}/receipt via giving-service
    - Google Docs — Generate branded receipt PDF
    - HTTP Module — POST /v1/communication/email via communication-service
      - Send receipt PDF as attachment

Step 5: HTTP Module — GET /v1/giving/summary?memberId={memberId}&year=2026
  - Fetch year-to-date giving summary

Step 6: Router — Milestone check
  - If YTD tithe reaches new threshold ($5K, $10K, $25K, $50K):
    - HTTP Module — POST /v1/communication/whatsapp
      - "Congratulations, {firstName}! Your faithful tithing has reached {milestone} this year. Thank you for your consistency."
    - Notify pastor of milestone

Step 7: Google Sheets — Update "Giving Log" spreadsheet
  - Row: Date, Member, Type, Amount, Method, Receipt Sent, Campus

Step 8: Aggregator — End of month summary
  - Google Sheets — Generate monthly giving summary
  - Email — Send to finance team and senior pastor

Error Handler: Log failures to error sheet, alert finance admin
```

### Prompt M-004: KPI Report Generation & Distribution

```text
Create a Make (Integromat) scenario for ChMS KPI report generation and distribution:

Trigger: Scheduled — runs weekly on Monday at 7:00 AM, and quarterly (1st of Jan, Apr, Jul, Oct at 9:00 AM)

Step 1: HTTP Module — GET /v1/kpi?period={currentPeriod} via kpi-service
  - Fetch all KPI metrics:
    - 72-Hour Contact Rate (target: 90%)
    - NBC Enrollment Rate (target: 100%)
    - Water Baptism Rate (target: 95%)
    - Mentorship Completion Rate (target: 85%)
    - Visitor Conversion Rate (target: 60%)
    - Welfare Cases Fulfilled (target: 100%)
    - Attendance Growth Rate (target: 5% QoQ)
    - Giving Growth Rate (target: 5% QoQ)
    - Volunteer Retention Rate (target: 85%)
    - Group Participation Rate (target: 40% of members)

Step 2: HTTP Module — GET /v1/member?status=active&count=true via member-service
  - Total active members, new members this period, inactive count

Step 3: HTTP Module — GET /v1/visitor?period={currentPeriod}&summary=true via visitor-service
  - Total visitors, conversion count, follow-up completion

Step 4: HTTP Module — GET /v1/giving/summary?period={currentPeriod} via giving-service
  - Total giving by type, comparison to prior period

Step 5: HTTP Module — GET /v1/volunteer/summary?period={currentPeriod} via volunteer-service
  - Active volunteers, new signups, retention rate

Step 6: Text Aggregator — Format KPI data into report template
  - For each KPI: current value, target, variance, trend arrow, status (green/amber/red)
  - Highlights: top achievements and areas needing attention
  - Campus-level breakdown

Step 7: Router — Branch by trigger type
  - Branch A: Weekly report (abbreviated, top-line KPIs only)
  - Branch B: Quarterly report (comprehensive with charts and commentary)

Branch A — Weekly:
  Step 8A: HTTP Module — POST /v1/communication/email via communication-service
    - Send to pastoral team with formatted KPI summary
  Step 9A: Slack/Teams — Post to #church-leadership channel

Branch B — Quarterly:
  Step 8B: Google Docs — Generate comprehensive PDF report
    - Church branding, color-coded KPI dashboard, charts, campus comparison
    - Pastor's commentary section (template)
    - Recommendations based on KPI trends
  Step 9B: HTTP Module — POST /v1/communication/email
    - Send to church board, senior pastoral team, department heads
    - Subject: "ChMS Quarterly KPI Report — {Quarter} {Year}"
    - Attach PDF report
  Step 10B: Slack/Teams — Post summary with PDF link
  Step 11B: Google Sheets — Archive KPI values in historical tracking sheet
    - Row per KPI: date, period, KPI name, value, target, status, campus

Error Handler: On failure, send error notification to IT admin and church administrator
```

### Prompt M-005: Event Registration & Volunteer Scheduling Automation

```text
Create a Make (Integromat) scenario for ChMS event registration and volunteer scheduling:

Trigger: Webhook — receives POST from event-service when a new event is created
Payload: { eventId, eventName, date, time, locationId, capacity, volunteerRequirements, campusId, tenantId, recurring }

Step 1: HTTP Module — GET /v1/facility/{locationId} via facility-service
  - Verify facility availability and capacity
  - If conflict: alert event creator and suggest alternatives

Step 2: HTTP Module — GET /v1/volunteer?campusId={campusId}&skills={requiredSkills} via volunteer-service
  - Fetch eligible volunteers based on event requirements
  - For each role (Ushers, Choir, Tech, Parking, Children's Ministry):
    - Filter by skill, availability, and rotation fairness

Step 3: Iterator — For each volunteer role requirement:
  Step 4: HTTP Module — POST /v1/volunteer/schedule via volunteer-service
    - Create volunteer schedule slots
    - Send invitation to top-ranked volunteers (based on availability + rotation + preference)
  Step 5: HTTP Module — POST /v1/communication/whatsapp via communication-service
    - "Hi {volunteerName}, you've been scheduled for {roleName} at {eventName} on {date} at {time}. Please confirm your availability."
    - Include "Confirm" and "Decline" quick reply options

Step 6: HTTP Module — POST /v1/communication/notification via communication-service
  - Send event announcement to relevant member segments:
    - If Women's event: send to Women's natural group
    - If Youth event: send to Youth natural group
    - If General event: send to all active members

Step 7: Router — Check if event is within 7 days
  - If yes: Schedule reminder sequence via communication-service
    - 7 days before: email announcement
    - 1 day before: WhatsApp reminder to registered attendees
    - 1 hour before: push notification to registered attendees
    - 1 day before: volunteer reminder with schedule details

Step 8: HTTP Module — POST /v1/facility/booking via facility-service
  - Confirm facility booking with event details

Step 9: Google Sheets — Update "Event Calendar" sheet
  - Row: Event Name, Date, Location, Capacity, Volunteers Needed/Assigned, Registration Count, Campus

Error Handler: Alert event coordinator on any failure
```

### Prompt M-006: Welfare Case Management & Escalation

```text
Create a Make (Integromat) scenario for ChMS welfare case management and escalation:

Trigger: Webhook — receives POST from welfare-service when a new welfare case is created
Payload: { caseId, beneficiaryId, beneficiaryName, needCategory, urgency, description, requestedBy, campusId, tenantId }

Step 1: Router — Branch by urgency level
  - Branch A: Critical (immediate response needed)
  - Branch B: High (24-hour response)
  - Branch C: Medium (72-hour response)
  - Branch D: Low (weekly review)

Branch A — Critical Urgency:
  Step 2A: HTTP Module — POST /v1/communication/sms via communication-service
    - Alert welfare team lead and senior pastor immediately:
      "URGENT WELFARE CASE: {beneficiaryName} — {needCategory}. Case #{caseId}. Immediate response required."
  Step 3A: HTTP Module — POST /v1/communication/notification
    - Push notification to welfare team members
  Step 4A: Slack/Teams — Post to #welfare-urgent channel
  Step 5A: HTTP Module — PATCH /v1/welfare/{caseId} via welfare-service
    - Auto-assign to welfare team lead
    - Set SLA: 4-hour initial response

Branch B — High Urgency:
  Step 2B: HTTP Module — POST /v1/communication/email via communication-service
    - Notify welfare team with case details
  Step 3B: HTTP Module — PATCH /v1/welfare/{caseId}
    - Assign based on need category and team member caseload
    - Set SLA: 24-hour initial response

Branch C — Medium Urgency:
  Step 2C: HTTP Module — POST /v1/communication/email
    - Notify assigned welfare officer
  Step 3C: HTTP Module — PATCH /v1/welfare/{caseId}
    - Round-robin assignment within welfare team
    - Set SLA: 72-hour initial response

Branch D — Low Urgency:
  Step 2D: HTTP Module — PATCH /v1/welfare/{caseId}
    - Add to weekly review queue

Step 6 (all branches): HTTP Module — POST /v1/communication/whatsapp via communication-service
  - Acknowledge to requestor:
    "Thank you for bringing {beneficiaryName}'s needs to our attention. Our welfare team has received the request (Case #{caseId}) and will respond within {slaTime}. God bless you for caring."

Step 7: HTTP Module — GET /v1/welfare/history?beneficiaryId={beneficiaryId}
  - Check for prior welfare cases to provide context to assigned team member
  - If repeat case within 90 days: flag for senior pastoral review

Step 8: HTTP Module — POST /v1/kpi/event via kpi-service
  - Log welfare case creation for KPI tracking (welfare cases metric)

Step 9: Google Sheets — Update "Welfare Case Log" spreadsheet
  - Row: Date, Case ID, Beneficiary, Category, Urgency, Assigned To, SLA, Status, Campus

FOLLOW-UP SCHEDULER (runs daily at 9:00 AM):
  Step 10: HTTP Module — GET /v1/welfare?status=open&slaRisk=true via welfare-service
    - Find cases approaching or past SLA
  Step 11: Iterator — For each at-risk case:
    - If past SLA: escalate to welfare team lead and pastor
    - If approaching SLA: remind assigned officer

Error Handler: On any failure, notify church administrator and log to error tracking
```

---

## 5. Prompt Usage Guidelines

### 5.1 Execution Order
1. Run **F-001** (Design Tokens) and **F-002** (Component Library) first to establish the foundation
2. Run desktop page prompts (**F-003** through **F-008**) for primary layouts
3. Run mobile prompts (**F-009** through **F-012**) for mobile-specific designs
4. Run tablet prompts (**F-013**, **F-014**) for responsive adaptations
5. Make automation prompts (**M-001** through **M-006**) can be executed independently

### 5.2 Customization
- Replace "Grace Community Church" with your church name
- Adjust currency symbols ($) for regional deployment (NGN, GBP, EUR, KES, etc.)
- Modify discipleship pathway stages to match your church's discipleship program
- Add or remove sidebar navigation items based on licensed services
- Adjust KPI targets to match your church's goals and benchmarks
- Update natural groups and member types to match your church structure

### 5.3 Pastoral Sensitivity Guidelines
- Prayer requests must always be displayed with privacy controls (visible to leadership only)
- Giving amounts should be masked by default, visible only to Finance and Senior Pastor roles
- Welfare case details should be restricted to assigned team and leadership
- Use warm, pastoral language in all user-facing copy (not corporate/technical)
- Avoid gamification language around spiritual milestones

### 5.4 Design Quality Checks
- Every page must have: loading (skeleton), empty, populated, and error states
- Every interactive element must show: default, hover, focus, active, disabled states
- All forms must show: valid, invalid, and submitting states
- Dark mode must be tested for contrast compliance on every page

---

## 6. Output Packaging Convention

| Artifact | Format | Naming Convention |
|----------|--------|-------------------|
| Design tokens | Figma Variables / JSON | `chms-tokens-v{version}` |
| Component library | Figma Library | `ChMS Components v{version}` |
| Desktop pages | Figma Pages | `Desktop — {PageName}` |
| Mobile pages | Figma Pages | `Mobile — {PageName}` |
| Tablet pages | Figma Pages | `Tablet — {PageName}` |
| Prototype flows | Figma Prototype | `Flow — {JourneyName}` |
| Make blueprints | Make JSON export | `make-chms-{scenario-name}-v{version}.json` |
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
| QR check-in scan-to-confirm | < 2.0s | End-to-end measurement |

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
| G-09 | Audit trail badges present on giving records, welfare cases | Pending |
| G-10 | Multi-campus tenant indicator visible in header | Pending |
| G-11 | Privacy controls on giving amounts, prayer requests, welfare details | Pending |
| G-12 | Pastoral tone verified in all user-facing copy | Pending |
| G-13 | Analytics events documented per component | Pending |
| G-14 | Error recovery paths designed with retry and support link | Pending |
| G-15 | Design tokens reference Figma Variables (not hard-coded) | Pending |
| G-16 | Component auto-layout verified at all breakpoints | Pending |
| G-17 | Make automation scenarios tested with sample payloads | Pending |

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-18 | AIDD System | Initial Figma and Make design prompts (source-monolith version) |
| 2.0 | 2026-02-23 | AIDD System | Enhanced version: full AIDD guardrails, expanded design tokens with dark mode, comprehensive component library, 6 desktop pages (dashboard, member directory, follow-up board, giving, events/volunteers, discipleship/groups), 4 mobile screens (dashboard, visitor registration, QR check-in, giving), 2 tablet adaptations, 6 Make automations (visitor welcome, absentee follow-up, giving acknowledgment, KPI reports, event/volunteer scheduling, welfare case management), performance targets, handoff gates. References all 12 services. |
