# Figma & Make.com Automation Prompts -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Figma Design Prompts

### 1.1 Screen 1: Senior Pastor Dashboard

**Prompt for Figma AI / UI Generation:**

"Design a senior pastor dashboard for a church management system. Dark mode with warm accent colors (gold, deep purple). The dashboard should include:

**Header**: Church name, pastor's avatar, notification bell with badge count, current date/time.

**Row 1 - Quick Stats (4 cards)**: Total Active Members (2,450 with +5.2% trend arrow), This Week's Attendance (1,890 with comparison to last week), New Visitors This Month (47 with 72-hour contact rate percentage), Monthly Giving Total ($125,000 with comparison to budget).

**Row 2 - Charts (2 columns)**: Left: Attendance trend line chart (12 weeks, Sunday vs Midweek lines). Right: Visitor conversion funnel (Registered -> Contacted -> Engaged -> NBC Enrolled -> Converted) with counts at each stage.

**Row 3**: Left: KPI Scorecard table showing 5 KPIs (72-Hour Contact, NBC Enrollment, Mentorship Completion, Visitor Conversion, Welfare Cases) each with target, actual, and traffic light status (green/yellow/red). Right: Recent Activity feed showing latest follow-ups, conversions, welfare cases.

**Row 4**: Directorate Overview - 6 small cards showing each directorate name, active cases count, and completion percentage ring chart.

Use Inter font family. Cards with subtle shadow and rounded corners. Responsive layout 1440px wide."

---

### 1.2 Screen 2: Campus Pastor View

**Prompt:**

"Design a campus pastor view for a multi-campus church management system. This is a filtered version of the senior pastor dashboard showing only one campus's data. Include:

**Top bar**: Campus selector dropdown (e.g., 'RCCG City Campus'), campus address below.

**Stats row**: Campus-specific membership count, this Sunday attendance, new visitors this week, campus giving total.

**Main content**: Left panel (60%): Attendance comparison chart (this campus vs. all campuses percentage), visitor assimilation pipeline for this campus. Right panel (40%): My follow-up tasks (pastor's personal assignments), prayer requests requiring attention, upcoming campus events list.

**Bottom row**: Staff directory for this campus (account officers, department heads) with contact shortcuts. Welfare cases for this campus with urgency color coding.

Maintain the same design system as the senior pastor dashboard. Include a 'Switch to All Campuses' toggle."

---

### 1.3 Screen 3: Directorate Dashboard

**Prompt:**

"Design a directorate head dashboard for the '1st Timer Directorate' in a church follow-up system. Include:

**Header**: Directorate name with icon, directorate head name, time period filter (This Week / This Month / This Quarter).

**KPI Row**: 3 large metric cards: New Cases Received (count with week-over-week change), Cases Completed (count with completion rate percentage), Average Response Time (hours/days).

**Main Area - Kanban Board**: 4 columns: New (unassigned), Assigned (to account officers), In Progress (contact made), Completed (moved to next directorate). Each card shows visitor/member name, days since assigned, assigned officer name, contact attempt count. Color-code cards by urgency (>48h yellow, >72h red).

**Right Sidebar**: Account Officer Performance table: Officer name, assigned count, completed count, average response time, 72-hour compliance rate. Sortable by any column.

**Bottom**: Activity log showing recent actions: assignments, contacts, transfers to other directorates. Each entry has timestamp, actor, action, and target person.

Use warm church-appropriate colors. Kanban cards should be draggable."

---

### 1.4 Screen 4: Account Officer Workspace

**Prompt:**

"Design a mobile-first Account Officer workspace for a church follow-up app. This is the primary daily tool for soul shepherding. Include:

**Top Section**: Welcome message with officer name, today's date. 3 summary badges: My Souls (total assigned), Pending Follow-ups (count with urgency), 72-Hour Alerts (visitors approaching deadline, red if overdue).

**Main Content - Task List**: Sorted by urgency. Each task card shows: Person name and photo placeholder, task type icon (phone call, home visit, WhatsApp, etc.), due date with countdown, brief context (e.g., 'First-timer from Sunday'), quick action buttons (Call, WhatsApp, Log Visit, Snooze).

**Quick Actions Bar**: Floating action bar at bottom: Record Follow-up, Log Visit, Create Welfare Case, Call Next Person.

**Person Detail View (slide up)**: When tapping a person: Full contact info, communication history timeline (all past follow-ups with channel used, date, notes), family information, group membership, attendance history mini-calendar, current discipleship stage badge.

Optimized for 375px width (iPhone). Use thumb-friendly tap targets (44px minimum). Support dark mode."

---

### 1.5 Screen 5: Member Portal

**Prompt:**

"Design a member self-service portal for a church management system. Clean, light theme with the church's brand color as accent. Include:

**Dashboard**: Welcome message with member name. 4 cards: My Attendance (visual streak calendar), My Giving (year-to-date summary), My Groups (list with next meeting), My Growth (discipleship progress bar).

**Navigation Tabs**: Profile, Giving, Events, Groups, Discipleship, Notifications.

**Profile Tab**: Personal information with edit button, membership ID, join date, natural group badge, communication preferences toggles.

**Giving Tab**: Summary chart (monthly giving trend), transaction list with filters, pledge progress bars, 'Download Statement' button for each fiscal year.

**Events Tab**: Calendar view with upcoming events highlighted, 'Check In' button (shows QR code in modal), attendance history with percentage.

**Groups Tab**: My groups with meeting details, 'Find Groups' section with search and filters (type, day, location), join request button.

1440px desktop with mobile-responsive breakpoints."

---

### 1.6 Screen 6: Visitor Check-In Kiosk

**Prompt:**

"Design a touch-screen kiosk interface for church visitor check-in. Optimized for 10-12 inch tablets in landscape orientation. High contrast, large touch targets.

**Welcome Screen**: Church logo centered, 'Welcome! Are you visiting us today?' Large buttons: 'Yes, First Time' (green, prominent), 'Returning Visitor' (blue), 'I'm a Member' (gray, for QR scan).

**Registration Form (First Time)**: Step-by-step wizard (3 steps). Step 1: Name (first, last), Phone, Email - large input fields with on-screen keyboard. Step 2: How did you hear about us (large icon buttons: Friend, Social Media, Website, Walk-in, Other). Step 3: Optional fields: Address, Date of Birth, Marital Status.

**Confirmation Screen**: 'Welcome, [Name]! We're glad you're here.' with church service time, Wi-Fi password, restroom directions. 'Your personal greeter [Account Officer Name] will connect with you.' Print badge option.

**QR Check-In Screen (for members)**: Camera viewfinder for QR scanning. 'Point your QR code at the camera.' Success animation with member name display. 'Welcome back, [Name]!'

No keyboard required. All inputs via touch. Auto-reset after 30 seconds of inactivity."

---

### 1.7 Screen 7: Giving Portal

**Prompt:**

"Design an online giving portal for a church management system. Trust-building design with security badges. Include:

**Give Now Section**: Giving type selector (Tithe, Offering, Building Fund, Missions, Other) as pill buttons. Amount input with pre-set buttons ($50, $100, $250, $500, Custom). Payment method selector (Card, Bank Transfer, Mobile Money). Recurring toggle (One-time, Weekly, Monthly). 'Give Now' button - prominent and reassuring.

**My Giving Dashboard**: Year-to-date summary with donut chart by giving type. Monthly trend bar chart. Pledge tracker with progress bars. Transaction history table with date, type, amount, method, receipt link.

**Tax Statement Section**: Year selector, 'Generate Statement' button, preview panel, download as PDF button. Statement shows church letterhead, member details, itemized giving, totals.

Include security badge, SSL indicator, and privacy assurance text. Responsive design prioritizing mobile giving experience."

---

### 1.8 Screen 8: Event Calendar

**Prompt:**

"Design an event calendar view for a church management system. Include:

**Calendar View**: Monthly calendar with event dots on dates. Color-coded by event type (Sunday Service=blue, Midweek=green, Special=gold, Outreach=orange). Click/tap date to see event list.

**Event Detail Modal**: Event name, date/time, location, description, expected attendance, event type badge. Check-in button (for day-of events), Register button (for future events), Add to Calendar button (iCal export).

**List View Toggle**: Alternative list view sorted by date with event cards. Each card: title, date, time, location, attendee count, event type.

**Attendance View**: For completed events - attendance count vs expected, attendance rate percentage, check-in timeline chart, attendee list with check-in times.

**Create Event Form (Admin)**: Title, date/time pickers, location (with facility booking integration), event type dropdown, expected attendance, recurring toggle, description rich text editor."

---

### 1.9 Screen 9: Small Group Finder

**Prompt:**

"Design a small group finder page for a church management app. Include:

**Search and Filters**: Search bar, filter chips: Group Type (Small Group, Home Fellowship, Cell Group, Ministry), Meeting Day (Mon-Sun), Time of Day (Morning, Afternoon, Evening), Location (map or area picker).

**Results Grid**: Cards showing: Group name, leader photo and name, meeting day/time, location, member count vs capacity (e.g., '8/12'), group type badge, brief description (2 lines). 'Join' button on each card.

**Map View Toggle**: Map with pins for group locations. Cluster markers for dense areas. Click pin to see group card popup.

**Group Detail Page**: Leader profile card, full description, meeting schedule, location with map, current members (avatars), recent activities, 'Request to Join' button with optional message field."

---

### 1.10 Screen 10: Volunteer Scheduler

**Prompt:**

"Design a volunteer scheduling interface for a church management system. Include:

**Calendar View**: Weekly calendar grid (7 days x 3 time slots: morning, afternoon, evening). Cells show scheduled volunteers with role icons. Empty slots highlighted for open positions.

**Shift Creation Panel**: Date/time picker, role selector (Usher, Greeter, Tech, Worship, Children), required skill tags, volunteer count needed, notes field.

**Auto-Assign Feature**: 'Smart Assign' button that matches volunteers by: skills, availability preferences, recent service history (avoid over-scheduling), team preferences. Shows suggested assignments for review before confirmation.

**Volunteer Directory**: Searchable list with: name, skills (tags), availability (day/time matrix), total hours served, last served date, status (Active, On Leave). Click to view full profile and service history."

---

### 1.11 Screen 11: KPI Dashboard

**Prompt:**

"Design a comprehensive KPI dashboard for church leadership. Include:

**Header**: Period selector (Weekly, Monthly, Quarterly), campus filter, export button.

**KPI Scorecard (top row)**: 5 large gauge/dial widgets, one for each Quarterly Shepherding KPI: 72-Hour Contact Rate (target 90%), NBC Enrollment Rate (target 100%), Mentorship Completion (target 85%), Visitor Conversion (target 60%), Welfare Cases Fulfilled (target 20/month). Each gauge shows actual vs target with color coding.

**Trend Charts (middle row)**: 2 line charts showing KPI trends over time (12 weeks for weekly, 12 months for monthly). One chart for engagement metrics, one for growth metrics.

**Directorate Comparison (bottom left)**: Horizontal bar chart comparing 6 directorates on key metrics: active cases, completion rate, average response time.

**Leaderboard (bottom right)**: Top 10 Account Officers by performance score. Show name, score, souls assigned, follow-ups completed, 72-hour compliance rate."

---

### 1.12 Screen 12: Discipleship Tracker

**Prompt:**

"Design a discipleship progress tracker for a church management system. Include:

**Pipeline View**: Horizontal pipeline showing stages: New Believer -> NBC -> Mentorship -> Sunday School -> Home Fellowship -> Workforce. Each stage shows count of people currently in it. Click stage to see list.

**Individual Progress Card**: Member name and photo, current stage badge, progress bar (0-100%), started date, expected completion, mentor name (if applicable), milestones checklist (completed items checked).

**NBC Management Panel**: Current NBC classes with: class name, start/end dates, facilitator, enrolled count, completion rate. Student list with individual progress.

**Mentorship Pairs View**: Two-column card showing mentor on left, mentee on right, connected by progress bar. Meeting log timeline between them. Days remaining counter."

---

### 1.13 Screen 13: Welfare Case Manager

**Prompt:**

"Design a welfare case management interface. Include:

**Case Board (Kanban)**: Columns: Open, In Progress, Approval Pending, Approved, Fulfilled, Closed. Cards show: case number, member name, category icon (Financial, Medical, Housing, etc.), urgency badge (Low/Medium/High/Critical), assigned case worker, days since opened.

**Case Detail Panel**: Full case information, member profile summary, need description, amount requested vs approved vs disbursed, approval chain (who approved, when), activity timeline, attached documents, follow-up notes.

**New Case Form**: Member search, category selector, urgency assessment, need description (rich text), amount requested, supporting documents upload.

**Dashboard Summary**: Total active cases by category (pie chart), average resolution time (trend), fund balance, cases by urgency (stacked bar), monthly case volume trend."

---

### 1.14 Screen 14: Communication Center

**Prompt:**

"Design a communication center for multi-channel church messaging. Include:

**Compose View**: Rich text editor for message body, audience selector (All Members, Natural Group filter, Custom list, Individual), channel selector (checkboxes: SMS, WhatsApp, Telegram, Facebook, Email, Push, In-App), schedule picker (Send Now or future date/time), preview button showing message across channels.

**Template Library**: Grid of saved templates with: template name, preview text, channels supported, last used date. Create/edit template button.

**Delivery Dashboard**: Sent messages list with: date, subject, audience size, delivery stats per channel (sent/delivered/failed/read). Click for detailed report.

**Channel Status**: Status indicators for each channel (Connected/Error/Rate Limited). Last successful delivery time. Monthly usage vs quota.

Include character count for SMS (160 limit), WhatsApp formatting preview, and email HTML preview panel."

---

## 2. Make.com Automation Prompts

### 2.1 Visitor Follow-up Automation

"Create a Make.com scenario that triggers when a new visitor is registered in the church management system (webhook). The scenario should: (1) Wait 2 hours, (2) Send a WhatsApp welcome message via WhatsApp Business API, (3) Wait 24 hours, (4) If no response recorded (check via API), send an SMS via Twilio, (5) Wait 48 hours, (6) If still no response, create a task for the Account Officer in the system, (7) Log all communication attempts back to the system via API."

### 2.2 Absentee Member Alert

"Create a Make.com scenario that runs every Monday at 9 AM. It should: (1) Call the church management API to get members absent for 3+ weeks, (2) For each absentee, send a caring SMS: 'Hi [Name], we missed you at [Church]. Hope you're well. Your shepherd [Officer Name] will be in touch.', (3) Notify the Account Officer via email with the list of their absent souls, (4) Log all notifications back to the system."

### 2.3 Weekly KPI Report

"Create a Make.com scenario that runs every Friday at 5 PM. It should: (1) Call the KPI API to get this week's metrics, (2) Format a summary report in HTML, (3) Send the report via email to all users with role 'pastor' or 'directorate_head', (4) If any KPI is 'Behind', add a red alert section with recommended actions, (5) Post a summary to the leadership Telegram group."

### 2.4 Giving Receipt Automation

"Create a Make.com scenario that triggers when a giving record is created (webhook). It should: (1) Generate a PDF receipt using a template, (2) Send the receipt via the giver's preferred channel (WhatsApp, Email, or SMS with link), (3) If the giving completes a pledge, send a congratulatory message, (4) Log the receipt delivery back to the system."

### 2.5 Birthday Greeting Automation

"Create a Make.com scenario that runs daily at 8 AM. It should: (1) Call the member API with a date_of_birth filter for today's date, (2) For each birthday member, send personalized greetings via their preferred channel, (3) Notify the Account Officer to make a personal call, (4) If the member is a leader (pastor, minister, HOD), also send a greeting from the senior pastor."
