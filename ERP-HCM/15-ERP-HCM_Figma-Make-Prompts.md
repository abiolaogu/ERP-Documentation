# Figma & Make Prompts -- ERP-HCM (Human Capital Management)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make (Integromat) automation prompts for the ERP-HCM module. ERP-HCM is the Human Capital Management vertical of the consolidated ERP platform, covering employee lifecycle management, payroll processing, leave and attendance tracking, benefits administration, compensation planning, recruitment, learning management, performance reviews, workforce planning, compliance, document management, and facilities management. All prompts reference real services (employee-service, payroll-service, leave-service, time-attendance-service, benefits-service, compensation-service, recruitment-service, learning-service, performance-service, workforce-planning-service, compliance-service, document-service, facilities-service, facilities-management) and align with the AIDD guardrails defined in `erp/aidd.guardrails.yaml`.

The target frontend stack is **Refine + Ant Design** (React) with NATS event backbone and ERP-Platform entitlement integration.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance on every screen; color contrast ratio >= 4.5:1 for text, >= 3:1 for large text and UI components
- Minimum 44x44px touch targets on all interactive elements (buttons, links, checkboxes, table row actions)
- Clear focus-visible outlines (2px solid, offset 2px) for keyboard navigation
- All form fields include visible labels (never placeholder-only), required field indicators, and inline validation messages
- Plain-language labels (e.g., "Time Off Requests" not "Leave Requisition Management")
- Semantic HTML landmarks: `<nav>`, `<main>`, `<aside>`, `<footer>` reflected in Figma layer naming
- Screen reader annotations for charts (alt-text, data tables), icon-only buttons (aria-label), and live regions

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading; initial route bundle < 220 KB gzipped
- Skeleton UIs shown within 200ms; full content within 1s on 4G
- Virtualized tables for datasets > 50 rows (employee lists, payroll runs, attendance logs)
- Image assets: WebP format, max 80 KB hero, max 20 KB thumbnails
- Prefetch next-likely routes (e.g., employee detail from employee list)

### 2.3 Reliability, Trust, And Safety
- Every destructive action (terminate employee, delete payroll run, bulk leave rejection) requires a confirmation dialog with explicit typing or checkbox
- Inline recovery paths on all error states ("Retry", "Go back", "Contact support")
- Optimistic UI for low-risk mutations (leave approval) with undo toast (5s window)
- Trust signals: last-synced timestamp on dashboards, "Saved" indicator on forms, payroll calculation audit trail
- Audit trail accessible for compliance-service integration

### 2.4 Observability And Testability
- Every CTA emits a structured analytics event: `{event, module, entity, action, user_role, timestamp}`
- Error boundaries at page and widget level with correlation_id for support tickets
- Performance instrumentation: LCP, FID, CLS metrics on all page-level components
- Feature flag integration points annotated in Figma (e.g., "Visible when `workforce_planning_enabled` = true")

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens -- HCM Theme

```
Create a Figma page titled "HCM Design Tokens" containing the complete token
specification for the Human Capital Management module.

COLOR PALETTE (Light Theme):
- Primary: #1B5E7B (Deep Teal -- represents trust and professionalism)
- Primary Hover: #154A62
- Primary Light: #E3F2F9
- Secondary: #6C63FF (Accent Purple -- used for CTAs, badges)
- Success: #16A34A (Approved, Active, Completed)
- Warning: #F59E0B (Pending, Expiring Soon)
- Danger: #DC2626 (Rejected, Terminated, Overdue)
- Info: #2563EB (Informational banners, links)
- Neutral-50: #F9FAFB (Page background)
- Neutral-100: #F3F4F6 (Card background)
- Neutral-200: #E5E7EB (Borders, dividers)
- Neutral-500: #6B7280 (Secondary text)
- Neutral-900: #111827 (Primary text)
- Surface: #FFFFFF (Cards, modals)

COLOR PALETTE (Dark Theme):
- Primary: #4FC3F7 (Light Teal)
- Primary Hover: #81D4FA
- Background: #0F172A
- Surface: #1E293B
- Surface Elevated: #334155
- Border: #475569
- Text Primary: #F1F5F9
- Text Secondary: #94A3B8
- Success/Warning/Danger: same hues, lightened 20%

TYPOGRAPHY (Inter font family):
- Display: 30px/36px, semibold 600 -- page titles ("Employee Directory")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Benefits Enrollment")
- Heading 2: 20px/28px, medium 500 -- card titles ("Leave Balance")
- Heading 3: 16px/24px, medium 500 -- subsection headers
- Body: 14px/20px, regular 400 -- paragraph text, table cells
- Body Small: 12px/16px, regular 400 -- captions, timestamps, helper text
- Label: 14px/20px, medium 500 -- form labels, column headers
- Mono: 13px/20px, JetBrains Mono -- employee IDs, amounts, codes

SPACING SCALE (4px base):
- 4px (xs), 8px (sm), 12px (md), 16px (lg), 24px (xl), 32px (2xl), 48px (3xl)
- Card padding: 24px
- Section gap: 32px
- Page margin: 32px (desktop), 16px (mobile)

ELEVATION / SHADOWS:
- Level 0: none (flat elements)
- Level 1: 0 1px 3px rgba(0,0,0,0.1) -- cards, dropdowns
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, popovers
- Level 3: 0 8px 24px rgba(0,0,0,0.12) -- dialogs, slide-overs

BORDER RADIUS:
- Small: 4px (badges, chips)
- Medium: 8px (cards, inputs, buttons)
- Large: 12px (modals, panels)
- Full: 9999px (avatars, status dots)

GRID:
- Desktop (1440px): 12-column, 72px columns, 24px gutter, 80px sidebar
- Tablet (1024px): 12-column, 56px columns, 16px gutter, collapsed sidebar
- Mobile (390px): 4-column, fluid, 16px gutter, bottom nav

Include token swatches, type scale samples, spacing ruler, and shadow comparison.
Both light and dark themes must be fully specified side-by-side.
```

#### Prompt F-002: Component Library -- HCM Components

```
Create a Figma page titled "HCM Component Library" with the following reusable
components, each showing all states (default, hover, active, disabled, error,
loading) and both light/dark theme variants.

EMPLOYEE AVATAR COMPONENT:
- Circular avatar (40px, 32px, 24px sizes)
- Fallback: initials on Primary Light background
- Online/offline status dot (bottom-right, 10px)
- Variants: with name + role subtitle, standalone, grouped (stacked for team views)
- Example: Avatar for "Adaeze Okonkwo", Senior HR Analyst, online

EMPLOYEE CARD:
- 320px wide card with avatar, name, employee ID (EMP-2024-0847), department
  ("Human Resources"), job title, employment status badge (Active/green,
  On Leave/amber, Probation/blue, Terminated/red)
- Quick actions: View Profile, Send Message, Request Leave
- Example data: "Chidi Nnamdi | EMP-2024-1203 | Engineering | Staff Software Engineer | Active"

LEAVE BALANCE WIDGET:
- Compact card showing leave type (Annual, Sick, Parental, Compassionate),
  used/total days as progress bar, days remaining as large number
- Color-coded by leave type: Annual=#2563EB, Sick=#DC2626, Parental=#8B5CF6, Compassionate=#F59E0B
- Example: "Annual Leave: 8 of 21 days used | 13 remaining"

PAYROLL SUMMARY CARD:
- Shows gross pay, deductions breakdown (tax, pension, NHF, insurance),
  net pay, payment date, payslip download icon
- Currency formatting: NGN 1,245,000.00
- Status: Processed/green, Pending/amber, Failed/red
- Example: "Feb 2026 | Gross: NGN 1,450,000 | Deductions: NGN 205,000 | Net: NGN 1,245,000 | Processed"

APPROVAL ACTION BAR:
- Horizontal bar with approve (green) / reject (red) / request-info (amber) buttons
- Requestor mini-card (avatar + name + request summary)
- Supports bulk selection mode for managers
- Example: "Fatima Bello requests 3 days Annual Leave (Mar 10-12, 2026)"

DATA TABLE (HCM variant):
- Ant Design-based table with sortable columns, row selection checkboxes,
  inline status badges, row hover actions (eye icon = view, pencil = edit)
- Pagination: "Showing 1-25 of 342 employees"
- Filters bar above table with department, status, location, date-range chips
- Empty state: illustration + "No employees match your filters" + clear filters button

STAT/KPI CARD:
- Icon (left), metric value (large), label (small), trend indicator (up/down arrow + %)
- Example: Headcount icon, "1,247", "Total Employees", "+3.2% this quarter"

TIMELINE / ACTIVITY FEED:
- Vertical timeline with dot markers, timestamp, actor avatar, action description
- Example entries:
  -- "Adaeze Okonkwo approved Chidi Nnamdi's leave request" -- 2 hours ago
  -- "Payroll run for February 2026 completed" -- Yesterday
  -- "Recruitment: 3 new applications for Senior DevOps role" -- 2 days ago

NOTIFICATION BELL + DROPDOWN:
- Bell icon with unread count badge
- Dropdown: grouped by Today / Earlier, each item has icon, title, description, timestamp
- Mark all as read, View All links

SIDEBAR NAVIGATION:
- Collapsible sidebar (expanded 260px, collapsed 72px)
- Sections: Dashboard, Employees, Payroll, Leave & Attendance, Benefits,
  Compensation, Recruitment, Learning, Performance, Workforce Planning,
  Compliance, Documents, Facilities, Settings
- Active state: Primary background + white text + left border accent
- Badge counts on Recruitment (12 new), Leave (5 pending)
- User profile mini-card at bottom: avatar, name, role, logout

FORM COMPONENTS (HCM-specific):
- Employee search/select dropdown with avatar + name + department
- Date range picker for leave requests
- Currency input with NGN/USD/GBP prefix
- File upload zone for documents (drag-and-drop, supported formats list)
- Multi-step wizard progress bar (e.g., Onboarding: Personal > Employment > Documents > Review)
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: HR Dashboard -- Command Center (1440px)

```
Design a desktop HR Dashboard page at 1440x900px (scrollable) for the ERP-HCM
module. This is the primary landing page for HR Managers and HR Directors.

LAYOUT:
- Left: Collapsible sidebar navigation (260px, expanded by default)
- Top: Horizontal header bar with breadcrumb ("Dashboard"), global search (Cmd+K),
  notification bell (3 unread), user avatar "Ngozi Adeyemi" (HR Director)
- Main content area: 1156px wide, 32px padding

CONTENT -- ROW 1 (KPI Cards, 4-column grid):
- Card 1: "Total Headcount" -- 1,247 employees -- +12 this month (green arrow up)
- Card 2: "Open Positions" -- 23 roles -- 8 in final interview stage
- Card 3: "Attendance Today" -- 94.2% -- 1,175 of 1,247 present
- Card 4: "Pending Approvals" -- 17 -- 5 leave, 8 expenses, 4 documents

CONTENT -- ROW 2 (Two-column, equal width):
- Left: "Headcount Trend" -- Line chart showing last 12 months, x-axis months,
  y-axis headcount (range 1,100-1,300), with hire/termination annotations.
  Data: Jan=1,180, Feb=1,192, Mar=1,201, Apr=1,210, May=1,215, Jun=1,220,
  Jul=1,228, Aug=1,230, Sep=1,235, Oct=1,240, Nov=1,243, Dec=1,247
- Right: "Department Distribution" -- Donut chart with legend.
  Engineering: 342 (27%), Operations: 256 (21%), Sales: 198 (16%),
  HR: 89 (7%), Finance: 112 (9%), Product: 134 (11%), Support: 116 (9%)

CONTENT -- ROW 3 (Two-column, 60/40 split):
- Left (60%): "Recent Activity" timeline (last 10 items from employee-service,
  leave-service, recruitment-service). Example entries:
  -- "Chinedu Obi completed probation review" -- 35 min ago
  -- "Annual Leave approved for Amara Johnson (Mar 3-7)" -- 1 hour ago
  -- "New hire onboarding started: David Adeleke, Engineering" -- 2 hours ago
  -- "Payroll Feb 2026 -- 1,247 payslips generated" -- Yesterday
  -- "Compliance alert: 14 employees missing updated ID documents" -- Yesterday
- Right (40%): "Upcoming Events" list card:
  -- "Performance Review Cycle Q1 opens" -- Mar 1, 2026
  -- "Benefits Open Enrollment deadline" -- Mar 15, 2026
  -- "Company Town Hall" -- Mar 20, 2026
  -- "Payroll Processing Window" -- Mar 25-27, 2026

CONTENT -- ROW 4 (Full width):
- "Leave Calendar Heatmap" -- Month view (Feb 2026) showing team availability.
  Each cell = one day, colored by number of people on leave:
  0-2 green, 3-5 amber, 6+ red. Hover shows names.

STATES:
- Loading: Skeleton shimmer on all cards and charts
- Empty: "Welcome to HCM! Start by adding your first employee." with CTA button
- Error: Inline error card with retry for failed data fetches

Include both light theme (default) and dark theme toggle in the header.
Annotate all interactive elements with click targets >= 44px.
```

#### Prompt F-004: Employee Directory and Profile (1440px)

```
Design two connected desktop screens at 1440x900px for the ERP-HCM employee-service.

SCREEN A -- EMPLOYEE DIRECTORY:
Layout: Sidebar (collapsed 72px) + main area.
- Top bar: "Employees" title, "Add Employee" primary button, search bar, filter chips
- Filter row: Department dropdown, Employment Status (Active/On Leave/Probation/Terminated),
  Location dropdown, Date Hired range picker
- Active filters shown as dismissible chips: e.g., "Department: Engineering" x
- View toggle: List view (default) | Grid view (card-based)

TABLE (List View):
| Checkbox | Avatar+Name           | Employee ID    | Department  | Job Title                | Status    | Location   | Actions |
|----------|-----------------------|----------------|-------------|--------------------------|-----------|------------|---------|
| [ ]      | Adaeze Okonkwo        | EMP-2024-0847  | Engineering | Staff Software Engineer  | Active    | Lagos, NG  | ... |
| [ ]      | Chidi Nnamdi          | EMP-2024-1203  | Engineering | Senior DevOps Engineer   | Active    | Abuja, NG  | ... |
| [ ]      | Fatima Bello          | EMP-2023-0412  | HR          | HR Business Partner      | Active    | Lagos, NG  | ... |
| [ ]      | Olumide Adesanya      | EMP-2024-0099  | Finance     | Financial Analyst        | Probation | Lagos, NG  | ... |
| [ ]      | Ngozi Eze             | EMP-2022-0651  | Product     | Senior Product Manager   | On Leave  | Remote     | ... |

- Pagination: "Showing 1-25 of 1,247 employees" with page selector
- Bulk action bar (appears when rows selected): "3 selected -- Bulk Update | Export | Message"

SCREEN B -- EMPLOYEE PROFILE:
Layout: Back breadcrumb "Employees > Adaeze Okonkwo", three-column profile layout.

LEFT COLUMN (280px):
- Large avatar (96px) with camera icon overlay for upload
- Name: Adaeze Okonkwo
- Title: Staff Software Engineer
- Department: Engineering
- Employee ID: EMP-2024-0847
- Status badge: Active (green)
- Quick actions: Edit Profile, Send Message, View Org Chart position

CENTER COLUMN (540px) -- Tabbed content:
Tab 1 "Personal": Full name, date of birth (1992-06-14), gender, marital status,
  phone (+234 802 345 6789), email (a.okonkwo@company.com), address, emergency contacts
Tab 2 "Employment": Hire date (2024-02-15), employment type (Full-Time), reporting
  manager (Emeka Okafor), work location, probation end date, confirmation date
Tab 3 "Compensation": Current salary (NGN 12,500,000/yr), pay grade (L5),
  last review date, next review date, salary history table
Tab 4 "Leave": Leave balance cards (Annual: 13/21, Sick: 8/10, Parental: 0/90),
  leave history table with status badges
Tab 5 "Documents": Uploaded documents list (Offer Letter, ID Card, Tax Certificate,
  NDA) with upload dates, file sizes, download buttons
Tab 6 "Performance": Current cycle rating, goal progress bars, 360 feedback summary

RIGHT COLUMN (280px):
- "Team" section: Direct reports list (3 people with avatars)
- "Org Chart" mini-view: Manager above, peers beside, reports below
- "Recent Activity" mini-timeline (last 5 events for this employee)

Both screens support light and dark themes. All form fields are read-only by
default with an "Edit" mode toggle. Annotate keyboard navigation order.
```

#### Prompt F-005: Payroll Processing Dashboard (1440px)

```
Design a desktop Payroll Processing page at 1440x900px for the payroll-service.
Target user: Payroll Administrator.

HEADER:
- Breadcrumb: "Payroll" > "February 2026 Run"
- Pay period selector dropdown: "February 2026" with left/right arrows
- Status chip: "Processing" (amber pulse animation) or "Completed" (green)
- Actions: "Run Payroll" primary button (disabled if already running), "Export" dropdown
  (Excel, PDF, CSV), "Settings" gear icon

SUMMARY SECTION (4 KPI cards):
- Total Gross: NGN 1,812,450,000.00
- Total Deductions: NGN 324,680,000.00
- Total Net Pay: NGN 1,487,770,000.00
- Employees Processed: 1,247 of 1,247 (100% progress bar)

PAYROLL TABLE:
| # | Employee         | ID            | Department   | Gross Pay       | Tax       | Pension   | NHF     | Insurance | Other Ded. | Net Pay         | Status    |
|---|------------------|---------------|--------------|-----------------|-----------|-----------|---------|-----------|------------|-----------------|-----------|
| 1 | Adaeze Okonkwo   | EMP-2024-0847 | Engineering  | 1,041,667.00    | 187,500.00| 83,333.33 | 2,500.00| 15,000.00 | 0.00       | 753,333.67      | Processed |
| 2 | Chidi Nnamdi     | EMP-2024-1203 | Engineering  | 958,333.00      | 172,500.00| 76,666.64 | 2,500.00| 15,000.00 | 0.00       | 691,666.36      | Processed |
| 3 | Fatima Bello     | EMP-2023-0412 | HR           | 833,333.00      | 150,000.00| 66,666.64 | 2,500.00| 12,000.00 | 5,000.00   | 597,166.36      | Exception |
| 4 | Olumide Adesanya | EMP-2024-0099 | Finance      | 625,000.00      | 112,500.00| 50,000.00 | 2,500.00| 10,000.00 | 0.00       | 450,000.00      | Pending   |

- Sortable columns, filterable by department, status
- Row click opens payslip detail side-panel
- Exception rows highlighted with amber background and warning icon
- Status badges: Processed (green), Pending (gray), Exception (amber), Failed (red)

SIDE PANEL (Payslip Detail -- 480px slide-in from right):
- Employee header with avatar and details
- Earnings breakdown: Basic Salary, Housing Allowance, Transport Allowance, Other
- Deductions breakdown: PAYE Tax, Pension, NHF, Health Insurance, Loan Repayment
- Net Pay (large, bold)
- "Download Payslip PDF" button
- "Flag for Review" button with comment field

BOTTOM SECTION:
- "Payroll Exceptions" card: List of issues needing resolution
  -- "Fatima Bello: Missing tax ID -- payroll held" with "Resolve" link
  -- "New hire David Adeleke: Bank details not verified" with "Verify" link
- "Payroll History" table: Last 6 months with totals and status

Include audit trail link ("View payroll audit log") referencing compliance-service.
```

#### Prompt F-006: Leave Management (1440px)

```
Design a desktop Leave Management page at 1440x900px for the leave-service.
This page serves both Employees (self-service) and Managers (team approvals).

EMPLOYEE VIEW (Toggle at top: "My Leave" / "Team Leave" -- Team visible for managers only):

MY LEAVE VIEW:
- Top section: 4 leave balance cards in a row
  -- Annual Leave: 13 of 21 remaining (circular progress, blue)
  -- Sick Leave: 8 of 10 remaining (circular progress, red)
  -- Parental Leave: 90 of 90 remaining (circular progress, purple)
  -- Compassionate Leave: 5 of 5 remaining (circular progress, amber)
- "Request Leave" primary button (opens modal)

REQUEST LEAVE MODAL (centered, 560px wide):
- Leave Type dropdown (Annual, Sick, Parental, Compassionate, Study, Unpaid)
- Start Date picker
- End Date picker
- Duration auto-calculated: "3 working days"
- Reason textarea (required for Sick > 2 days)
- Supporting Document upload zone (required for Sick > 2 days, Parental)
- Relief Officer search-select (employee-service autocomplete)
- "Submit Request" button + "Save as Draft" secondary
- Shows team calendar mini-view (conflicting absences highlighted)

LEAVE HISTORY TABLE:
| Type    | Start Date | End Date   | Duration | Status    | Approved By     | Actions |
|---------|------------|------------|----------|-----------|-----------------|---------|
| Annual  | 2026-01-15 | 2026-01-17 | 3 days   | Approved  | Emeka Okafor    | View    |
| Sick    | 2025-12-20 | 2025-12-20 | 1 day    | Approved  | Emeka Okafor    | View    |
| Annual  | 2025-11-25 | 2025-11-29 | 5 days   | Approved  | Emeka Okafor    | View    |
| Study   | 2025-10-01 | 2025-10-03 | 3 days   | Rejected  | Emeka Okafor    | View    |

TEAM LEAVE VIEW (Manager):
- Team calendar: Month view showing all direct reports' leave on a Gantt-like timeline
  -- Each row = team member name + avatar
  -- Colored bars for leave periods (blue=annual, red=sick, purple=parental)
- Pending Approvals section:
  -- Card per request with: requestor avatar+name, leave type, dates, duration, reason
  -- Approve (green) / Reject (red) / Request Info (amber) action buttons
  -- Bulk approve checkbox mode
  -- Example: "Chidi Nnamdi | Annual Leave | Mar 10-12, 2026 | 3 days | Family event"

LEAVE POLICY SIDEBAR (collapsible right panel, 320px):
- Current policy summary: carry-over rules, max consecutive days, blackout periods
- "Company leave calendar" showing public holidays highlighted
- Link to full policy document (document-service)
```

#### Prompt F-007: Recruitment Pipeline (1440px)

```
Design a desktop Recruitment Pipeline page at 1440x900px for the recruitment-service.
Target user: Talent Acquisition Manager.

HEADER:
- Title: "Recruitment Pipeline"
- "Create Job Posting" primary button
- Filter bar: Department, Location, Status, Date Range
- View toggle: Kanban (default) | Table | Calendar

KANBAN VIEW (default):
Five columns representing pipeline stages, each scrollable vertically:

COLUMN 1 -- "New Applications" (badge: 34):
  Card: Oluwaseun Adebayo | Senior Backend Engineer | Engineering
         Applied: Feb 20, 2026 | Source: LinkedIn
         Skills: Go, PostgreSQL, Kubernetes | Match Score: 92%
         [Star rating: unrated] [Quick actions: Move, Reject, Schedule]

  Card: Amina Yusuf | Product Designer | Product
         Applied: Feb 19, 2026 | Source: Company Website
         Skills: Figma, User Research, Design Systems | Match Score: 87%

COLUMN 2 -- "Screening" (badge: 12):
  Card: Tunde Bakare | DevOps Engineer | Engineering
         Screening call: Feb 22, 2026 | Recruiter: Fatima Bello
         Notes: "Strong background in cloud infra"

COLUMN 3 -- "Interview" (badge: 8):
  Card: Grace Nwosu | Financial Controller | Finance
         Round 2 Technical | Interviewer: Olumide Adesanya
         Scheduled: Feb 25, 2026 14:00 | Video link attached
         Feedback from Round 1: 4/5 stars

COLUMN 4 -- "Offer" (badge: 3):
  Card: Ibrahim Musa | Security Engineer | Engineering
         Offer sent: Feb 18, 2026 | Package: NGN 15,000,000/yr
         Status: "Awaiting response" | Deadline: Feb 28, 2026

COLUMN 5 -- "Hired" (badge: 5):
  Card: Blessing Eze | Junior Frontend Developer | Engineering
         Start date: Mar 3, 2026 | Offer accepted: Feb 15, 2026
         Onboarding checklist: 2 of 8 items complete

DRAG-AND-DROP: Cards can be dragged between columns. Dropping triggers a
confirmation if moving backward (e.g., Interview back to Screening).

SIDE PANEL (Candidate Detail -- opens on card click, 480px):
- Full candidate profile: photo, contact, resume download, cover letter
- Application timeline with all stage transitions and notes
- Interview schedule and feedback forms
- Scorecards from each interviewer (1-5 star per competency)
- "Advance" / "Reject" / "Schedule Interview" action buttons
- Email thread history (notification-service integration)

JOB POSTINGS LIST (accessible via tab):
| Job Title                 | Department  | Location | Applications | Stage Breakdown          | Days Open | Actions     |
|---------------------------|-------------|----------|-------------|---------------------------|-----------|-------------|
| Senior Backend Engineer   | Engineering | Lagos    | 47          | 34/12/8/3/0              | 14        | View, Edit  |
| Product Designer          | Product     | Remote   | 23          | 15/5/2/1/0              | 21        | View, Edit  |
| Financial Controller      | Finance     | Lagos    | 12          | 4/3/3/2/0              | 30        | View, Close |
```

#### Prompt F-008: Learning Management System (1440px)

```
Design a desktop LMS page at 1440x900px for the learning-service.
Two views: Employee (learner) and HR Admin (content manager).

EMPLOYEE VIEW -- "My Learning":

TOP SECTION -- "Continue Learning" (horizontal scroll of in-progress courses):
  Course Card (280px wide, 200px tall):
  - Course thumbnail image
  - Title: "Advanced Go Programming"
  - Progress bar: 65% complete
  - Duration: "12 hours total | 4h 12m remaining"
  - Next lesson: "Concurrency Patterns"
  - "Resume" button
  - Provider badge: "Internal" or "Coursera" or "Udemy"

  Course Card 2: "Leadership Essentials for New Managers"
  - Progress: 30% | 8 hours total
  - Assigned by: HR Department | Due: Mar 31, 2026

  Course Card 3: "Data Privacy & GDPR Compliance" (compliance-service flag)
  - Progress: 100% COMPLETED (green check overlay)
  - Certificate available: download icon
  - Completed: Feb 10, 2026

MIDDLE SECTION -- "Recommended For You" (AI-suggested based on role + career path):
  Grid of 4 course cards with match percentage:
  - "System Design for Senior Engineers" -- 95% match
  - "AWS Solutions Architect Prep" -- 88% match
  - "Technical Writing for Engineers" -- 82% match
  - "Agile Project Management" -- 78% match

BOTTOM SECTION -- "Learning Path: Staff Engineer Track":
  Horizontal roadmap visualization with nodes:
  Node 1 (completed): "Core Engineering" -- 5 courses, all done
  Node 2 (in progress): "Advanced Engineering" -- 3 of 5 courses done
  Node 3 (locked): "Leadership & Mentoring" -- Prerequisites not met
  Node 4 (locked): "Staff Engineer Certification" -- Final assessment

ADMIN VIEW -- "Course Management" (HR Admin tab):
- "Create Course" button, "Import from Provider" button
- Courses table:
  | Title | Category | Enrolled | Completion Rate | Avg Rating | Status | Actions |
  | Advanced Go Programming | Technical | 89 | 72% | 4.6/5 | Active | Edit |
  | Leadership Essentials | Management | 45 | 58% | 4.2/5 | Active | Edit |
  | GDPR Compliance | Compliance | 1,247 | 94% | 3.8/5 | Mandatory | Edit |

- Analytics cards: Total Enrollments (3,456), Avg Completion Rate (71%),
  Training Hours This Month (2,890), Compliance Completion (94%)
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile HR Dashboard (390px)

```
Design a mobile HR Dashboard at 390x844px for the ERP-HCM module.
Target: HR Manager on-the-go.

NAVIGATION:
- Bottom tab bar (5 tabs): Home, Employees, Approvals (badge: 17), Notifications (badge: 3), Profile
- Top status bar with company logo (left), search icon (right)

HOME TAB:
- Greeting: "Good morning, Ngozi" + current date "Monday, Feb 23, 2026"
- Quick Actions row (horizontal scroll): Request Leave, View Payslip, My Team,
  Clock In, Directory

- KPI Cards (2x2 grid, each 170x100px):
  -- Headcount: 1,247
  -- Attendance: 94.2%
  -- Open Positions: 23
  -- Pending Approvals: 17

- "Pending Approvals" section (scrollable list):
  -- Each item: Avatar + Name + Request type + Date + "Approve" / "Reject" quick buttons
  -- "Chidi Nnamdi | Annual Leave | Mar 10-12 | [Approve] [Reject]"
  -- "Amara Johnson | Expense Claim | NGN 45,000 | [Approve] [Reject]"
  -- "David Adeleke | Document Upload | Tax Certificate | [Review]"
  -- "View All (17)" link

- "Recent Hires" horizontal scroll:
  -- Mini card per new hire: Avatar, Name, Department, Start Date
  -- "Blessing Eze | Engineering | Started Feb 17"

- "Announcements" card:
  -- "Performance Review Cycle Q1 opens March 1" with "Learn More" link
  -- "Benefits enrollment deadline: March 15" with "Enroll Now" link

All touch targets >= 44px. Pull-to-refresh on home. Skeleton loading states.
Bottom sheet for quick actions (not full page navigation).
```

#### Prompt F-010: Mobile Employee Self-Service (390px)

```
Design a mobile Employee Self-Service flow at 390x844px for self-service operations.

SCREEN 1 -- "My Profile" (from Profile tab):
- Profile header: Large avatar (80px), name "Adaeze Okonkwo", title + department
- Employee ID: EMP-2024-0847
- Quick stats row: "2 years tenure" | "Engineering" | "Lagos"
- Menu list with chevrons:
  -- Personal Information >
  -- Employment Details >
  -- My Payslips >
  -- Leave Balance >
  -- Benefits >
  -- Documents >
  -- Performance Goals >
  -- Learning >
  -- Settings >

SCREEN 2 -- "My Payslips":
- Month selector (horizontal scroll): Jan, [Feb highlighted], Mar...
- Payslip card for February 2026:
  -- "February 2026 Payslip" header
  -- Earnings section:
     Basic Salary:       NGN   625,000.00
     Housing Allowance:  NGN   208,333.33
     Transport Allowance:NGN   104,166.67
     Meal Allowance:     NGN   104,167.00
     ---
     Gross Pay:          NGN 1,041,667.00

  -- Deductions section:
     PAYE Tax:           NGN   187,500.00
     Pension (8%):       NGN    83,333.33
     NHF (2.5%):         NGN     2,500.00
     Health Insurance:   NGN    15,000.00
     ---
     Total Deductions:   NGN   288,333.33

  -- NET PAY:            NGN   753,333.67  (large, bold, Primary color)
  -- Payment Date: February 27, 2026
  -- [Download PDF] [Share] buttons

SCREEN 3 -- "Request Leave" (Bottom sheet, slides up to 85% height):
- Leave type chips: Annual | Sick | Parental | Compassionate | Study
- Calendar date picker (start and end)
- Duration auto-display: "3 working days"
- Reason input (multiline)
- Attach document (camera or file picker)
- Relief officer search field
- "Submit" full-width button at bottom (sticky)
- Team conflict warning: "2 team members already off on Mar 11"

SCREEN 4 -- "Clock In/Out" (time-attendance-service):
- Large clock display: 09:02 AM
- Location: "Lagos Office -- Verified" (green check with geofence icon)
- Status: "Not clocked in" / "Clocked in at 09:02 AM"
- Large "Clock In" button (circular, 120px, green) or "Clock Out" (red)
- Today's timeline: Clock In 09:02 -> [current] -> Expected Clock Out 17:00
- Weekly summary: Mon-Fri mini cards showing hours worked per day
- "Request Time Correction" link for mistakes

All screens use bottom navigation. Swipe gestures supported for month/date navigation.
```

#### Prompt F-011: Mobile Approvals Flow (390px)

```
Design a mobile Approvals flow at 390x844px for managers processing approvals
on mobile.

SCREEN 1 -- "Approvals" (from Approvals tab):
- Segment control: "Pending (17)" | "Approved" | "Rejected"
- Filter chips (horizontal scroll): All Types, Leave, Expense, Document, Overtime

PENDING LIST:
Each approval card (full width, ~120px tall):
- Left: Avatar (40px)
- Center: Name, request type, brief details, submitted date
- Right: Chevron to detail view
- Swipe left: Quick Reject (red)
- Swipe right: Quick Approve (green)

Example cards:
1. Chidi Nnamdi | Annual Leave
   "Mar 10-12, 2026 (3 days) -- Family event"
   Submitted: Feb 21, 2026

2. Amara Johnson | Expense Claim
   "Client dinner -- NGN 45,000"
   Submitted: Feb 20, 2026 | Receipt attached

3. David Adeleke | Overtime Request
   "Feb 18, 2026 -- 3 extra hours -- Deployment support"
   Submitted: Feb 19, 2026

4. Ngozi Eze | Document Upload
   "Updated Tax Clearance Certificate 2025"
   Submitted: Feb 22, 2026

SCREEN 2 -- "Approval Detail" (tapped from list):
- Full request details with all fields
- Requestor profile mini-card
- Attached documents with preview thumbnails
- Team calendar impact (for leave): visual bar showing who else is off
- Manager comment textarea
- Sticky bottom bar: [Reject] [Request Info] [Approve]
- Approve triggers success animation (green check) and auto-returns to list

SCREEN 3 -- "Bulk Approve" mode:
- Toggle "Select Multiple" at top
- Checkboxes appear on each card
- Bottom bar: "5 selected | [Reject All] [Approve All]"
- Confirmation bottom sheet: "Approve 5 requests?" with list summary

Haptic feedback on approve/reject. Toast notification on success.
```

#### Prompt F-012: Mobile Recruitment Quick View (390px)

```
Design a mobile Recruitment view at 390x844px for hiring managers reviewing
candidates on mobile.

SCREEN 1 -- "My Open Roles":
- List of job postings assigned to this manager
- Each card: Job title, department, applicant count, pipeline mini-bar
  -- "Senior Backend Engineer | Engineering | 47 applicants"
     Pipeline: [==34==|=12=|=8=|3|0] (colored segments)
  -- "Product Designer | Product | 23 applicants"

SCREEN 2 -- "Candidates" (after tapping a role):
- Pipeline stage tabs (horizontal scroll): New (34) | Screen (12) | Interview (8) | Offer (3) | Hired (0)
- Each candidate card:
  -- Avatar, Name, current role/company
  -- Applied date, source (LinkedIn badge, Referral badge)
  -- AI match score badge: "92% match"
  -- Star rating (if rated)
  -- Tap to expand: skills tags, summary note

SCREEN 3 -- "Candidate Profile":
- Full profile with resume viewer (scrollable PDF preview)
- Interview timeline with feedback from each round
- Scorecard summary: Communication 4/5, Technical 5/5, Culture 4/5
- Actions: "Schedule Interview", "Advance", "Reject", "Add Note"
- Notes section with team discussion thread

SCREEN 4 -- "Quick Interview Feedback" (bottom sheet):
- Star rating per competency (drag or tap)
- Verdict: Strong Yes | Yes | Maybe | No | Strong No
- Notes field
- "Submit Feedback" button
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Tablet HR Dashboard (1024px)

```
Design a tablet-adapted HR Dashboard at 1024x768px for the ERP-HCM module.

ADAPTATIONS FROM DESKTOP (1440px):
- Sidebar collapsed to icon-only (72px) by default, expandable via hamburger
- KPI cards: 2x2 grid instead of 4-column row
- Charts section: stacked vertically (Headcount Trend above Department Distribution)
  instead of side-by-side
- Activity feed and Upcoming Events: tabbed view in a single card instead of columns
- Leave heatmap calendar: scrollable horizontally, showing 2 weeks at a time
- Table rows: hide "Location" column, show on row expand
- Touch-optimized: all clickable areas >= 44px, increased spacing between action buttons

HEADER:
- Hamburger menu (left), "Dashboard" title (center), search icon + notification bell + avatar (right)
- Breadcrumbs hidden; replaced by title

SPECIFIC LAYOUT:
Row 1: 2x2 KPI grid (Headcount, Attendance, Open Positions, Pending Approvals)
Row 2: Headcount Trend chart (full width, 380px tall)
Row 3: Department Distribution donut (full width, 300px tall)
Row 4: Tab card ["Recent Activity" | "Upcoming Events"] -- full width
Row 5: Leave Calendar Heatmap -- horizontal scroll

Split-screen support: annotate that the dashboard can display in iOS/iPadOS split view
at 507px width (compact) falling back to mobile layout.
```

#### Prompt F-014: Tablet Employee Directory (1024px)

```
Design a tablet Employee Directory at 1024x768px.

ADAPTATIONS:
- Sidebar: icon-only (72px) with tooltip labels on hover
- Search bar: full width below header
- Filter chips: horizontal scrollable row
- Table: 5 visible columns (Avatar+Name, Employee ID, Department, Status, Actions)
  -- Job Title and Location accessible via row expansion
- Row actions: icon-only (eye, pencil, ellipsis) with tooltips
- "Add Employee" button in header, not floating

EMPLOYEE PROFILE (tablet):
- Two-column layout instead of three:
  -- Left (320px): Avatar, name, ID, department, status, quick actions
  -- Right (remaining): Tabbed content (same tabs as desktop)
  -- Team/Org Chart section moves into a dedicated tab

Table pagination: bottom-fixed bar with page controls.
Support landscape (1024x768) and portrait (768x1024) orientations.
```

---

## 4. Make Automation Prompts

### Prompt M-001: New Employee Onboarding Workflow

```
Create a Make (Integromat) scenario titled "HCM -- New Employee Onboarding".

TRIGGER: Webhook from employee-service when a new employee record is created.
Payload includes: employee_id, full_name, email, department, manager_id, start_date.

STEP 1 -- Validate Data:
- Check required fields are present
- Verify start_date is within 30 days
- Route: If validation fails -> Step ERROR

STEP 2 -- Create Accounts (Parallel):
- Call ERP-IAM identity-service: POST /v1/identities (create user account)
- Call ERP-IAM provisioning-service: POST /v1/provisions (provision app access based on department role template)
- Call document-service: POST /v1/document-requests (generate onboarding document checklist)

STEP 3 -- Assign Onboarding Learning Path:
- Call learning-service: POST /v1/enrollments
  Body: { employee_id, learning_path: "new_hire_onboarding", due_date: start_date + 30 days }

STEP 4 -- Notify Stakeholders:
- Email to new employee: Welcome email with first-day instructions
- Email to manager: New hire notification with onboarding checklist link
- Email to HR team: New hire confirmation
- Slack/Teams webhook: Post to #new-hires channel

STEP 5 -- Schedule Check-ins:
- Call calendar integration: Create 30/60/90 day check-in calendar events
  between employee and manager

STEP 6 -- Update Dashboard:
- Publish NATS event: "employee.onboarded" with metadata

ERROR HANDLING:
- Retry up to 3 times with exponential backoff
- On final failure: Send alert to hr-ops@company.com with error details
- Log all steps to compliance-service audit trail

ESTIMATED RUN TIME: < 15 seconds
TRIGGER VOLUME: ~50/month
```

### Prompt M-002: Automated Payroll Processing Pipeline

```
Create a Make scenario titled "HCM -- Monthly Payroll Processing".

TRIGGER: Scheduled -- 25th of every month at 00:01 UTC (or manual trigger).

STEP 1 -- Pre-flight Checks:
- Call employee-service: GET /v1/employees?status=active (get active employee list)
- Call benefits-service: GET /v1/deductions/current-period (get deduction schedules)
- Call time-attendance-service: GET /v1/attendance/summary?period=current_month
- Validate: all employees have bank details, tax IDs

STEP 2 -- Calculate Payroll:
- Call payroll-service: POST /v1/payroll/calculate
  Body: { period: "2026-02", employees: [...ids], include_overtime: true }
- Returns: array of { employee_id, gross, deductions_breakdown, net_pay }

STEP 3 -- Exception Handling:
- Filter employees with calculation errors
- Route exceptions to Slack channel #payroll-exceptions
- Email HR Payroll team with exception report
- Store exception list for manual resolution

STEP 4 -- Approval Gate (Supervised Action per AIDD guardrails):
- Send approval request to Payroll Manager via email
- Wait for webhook callback (approve/reject)
- If rejected: halt and notify

STEP 5 -- Process Payments:
- Call payment integration: POST /v1/bulk-transfer (bank integration)
- Generate individual payslips (PDF) via document-service
- Store payslips in employee records

STEP 6 -- Post-Processing:
- Send payslip notification to each employee (email + in-app notification)
- Publish NATS event: "payroll.completed" with summary
- Update compliance-service audit log
- Generate month-end payroll report

ERROR HANDLING:
- Any step failure halts the pipeline and alerts Payroll Manager
- Partial payment failures trigger per-employee retry queue
- Full audit trail maintained in compliance-service
```

### Prompt M-003: Leave Request Approval Automation

```
Create a Make scenario titled "HCM -- Leave Request Processing".

TRIGGER: Webhook from leave-service on new leave request submission.
Payload: { request_id, employee_id, leave_type, start_date, end_date,
           duration_days, manager_id, reason, documents[] }

STEP 1 -- Policy Validation:
- Check leave balance >= requested days (leave-service)
- Check blackout periods (no leave during payroll processing window)
- Check team coverage: GET /v1/attendance/team-availability?manager_id&dates
- If auto-reject conditions met: reject with reason and notify employee

STEP 2 -- Auto-Approve Rules:
- If leave_type = "Sick" AND duration <= 2 days: auto-approve
- If employee has no other leave in same month AND duration <= 1 day: auto-approve
- If auto-approved: skip to Step 4

STEP 3 -- Manager Notification:
- Push notification to manager mobile app
- Email to manager with approve/reject deep links
- If no response in 48 hours: escalate to skip-level manager
- If no response in 72 hours: auto-approve with escalation note

STEP 4 -- Post-Approval:
- Update leave-service: PUT /v1/leave-requests/{id}/status
- Notify employee (push + email)
- Update team calendar
- If leave > 5 days: notify HR Business Partner
- Publish NATS event: "leave.approved" or "leave.rejected"

STEP 5 -- Calendar Integration:
- Block dates in shared team calendar
- Update attendance forecasting in workforce-planning-service
```

### Prompt M-004: Performance Review Cycle Automation

```
Create a Make scenario titled "HCM -- Performance Review Cycle Orchestration".

TRIGGER: Scheduled or manual trigger to initiate quarterly performance review cycle.

STEP 1 -- Initialize Cycle:
- Call performance-service: POST /v1/review-cycles
  Body: { cycle: "Q1-2026", type: "quarterly", start_date: "2026-03-01",
          self_review_deadline: "2026-03-10", manager_review_deadline: "2026-03-20",
          calibration_deadline: "2026-03-25", finalize_deadline: "2026-03-31" }

STEP 2 -- Generate Review Forms:
- For each active employee (from employee-service):
  -- Create self-review form with auto-populated goals from last cycle
  -- Create manager-review form linked to employee
  -- Create peer-review requests (360 feedback) if enabled

STEP 3 -- Notification Cascade:
- Day 1: Email all employees "Q1 Performance Review cycle has started"
- Day 7: Reminder to employees who haven't started self-review
- Day 10: Self-review deadline reminder
- Day 15: Reminder to managers for pending reviews
- Day 20: Manager review deadline reminder
- Day 25: HR notification for calibration session

STEP 4 -- Completion Tracking:
- Poll performance-service daily for completion stats
- Update dashboard widget with progress
- Escalate overdue reviews to HR Business Partners

STEP 5 -- Calibration Support:
- Generate calibration report: department-level rating distributions
- Flag outliers (all 5s or all 1s) for HR review
- Schedule calibration meeting via calendar integration

STEP 6 -- Finalization:
- Lock review forms after finalize deadline
- Generate individual performance summary PDFs (document-service)
- Update compensation-service with review outcomes for merit planning
- Publish NATS event: "performance.cycle_completed"
```

### Prompt M-005: Compliance Document Expiry Monitor

```
Create a Make scenario titled "HCM -- Compliance Document Expiry Monitor".

TRIGGER: Scheduled daily at 08:00 UTC.

STEP 1 -- Scan Expiring Documents:
- Call document-service: GET /v1/documents?expiry_within=30_days
- Call compliance-service: GET /v1/compliance/requirements?status=expiring
- Categories: Tax certificates, professional licenses, work permits,
  health certificates, background checks, NDA renewals

STEP 2 -- Classify Urgency:
- Critical (expired or within 7 days): immediate action required
- Warning (8-14 days): schedule reminder
- Notice (15-30 days): informational

STEP 3 -- Notify:
- Critical: Email employee + HR + direct manager, push notification
- Warning: Email employee with renewal instructions
- Notice: In-app notification only
- Generate weekly compliance summary for HR Director

STEP 4 -- Track Resolution:
- Create compliance tasks in employee records
- Monitor for document upload events
- Auto-close task when renewed document uploaded and verified
- Escalate unresolved critical items daily

STEP 5 -- Report:
- Weekly compliance status report to HR Director
- Monthly regulatory compliance dashboard update
- Audit log entry for each notification sent (compliance-service)
```

### Prompt M-006: Workforce Planning Headcount Reconciliation

```
Create a Make scenario titled "HCM -- Weekly Workforce Reconciliation".

TRIGGER: Scheduled every Monday at 06:00 UTC.

STEP 1 -- Gather Current State:
- Call employee-service: GET /v1/employees/stats (active, on-leave, probation, terminated counts)
- Call recruitment-service: GET /v1/jobs/pipeline-summary (open positions, offers pending)
- Call workforce-planning-service: GET /v1/plans/current (approved headcount plan)

STEP 2 -- Compare Plan vs Actual:
- Calculate variance per department:
  { department, planned_headcount, actual_headcount, open_positions, variance, variance_pct }
- Flag departments with > 10% variance

STEP 3 -- Generate Report:
- Create weekly workforce report with:
  -- Headcount summary table
  -- New hires this week
  -- Terminations this week
  -- Upcoming starts (from recruitment-service accepted offers)
  -- Attrition rate calculation

STEP 4 -- Distribute:
- Email report to HR Director, CFO, Department Heads
- Update workforce-planning-service dashboard metrics
- Publish NATS event: "workforce.weekly_reconciliation"

STEP 5 -- Alerts:
- If attrition > threshold: alert HR Director
- If department understaffed > 15%: alert department head + recruitment team
- If planned hires not progressing: alert talent acquisition
```

---

## 5. Prompt Usage Guidelines

| Step | Action | Tool |
|------|--------|------|
| 1 | Copy the prompt text from Section 3 or 4 | Clipboard |
| 2 | Open Figma and activate the Make Design / AI plugin | Figma |
| 3 | Paste the prompt into the plugin's input field | Plugin UI |
| 4 | Review the generated output and adjust spacing, alignment, and token references | Manual |
| 5 | Connect components to the shared HCM Design System library | Figma Libraries |
| 6 | Export developer-ready assets using the naming convention below | Figma Export |

**Prompt modification guidelines:**
- Replace placeholder data with real tenant data when available
- Adjust currency (NGN) to match target market
- Adjust color palette if client branding overrides are provided
- Keep all AIDD guardrail annotations intact during modifications

---

## 6. Output Packaging Convention

```
Deliverable naming:
  HCM-{PromptID}-{Breakpoint}-{Theme}-v{Version}.fig
  Example: HCM-F003-1440-light-v1.fig

Layer naming:
  {Page}/{Section}/{Component}-{State}
  Example: Dashboard/KPICards/HeadcountCard-Default

Asset export:
  /exports/hcm/{breakpoint}/{page}/
  Example: /exports/hcm/1440/dashboard/kpi-headcount.png

Handoff tokens:
  /tokens/hcm-tokens.json (Style Dictionary format)
```

---

## 7. Performance Acceptance Targets

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
| Chart render | < 500ms | Custom perf mark |

---

## 8. AIDD Handoff Gate Template

```markdown
## Design Handoff Checklist -- ERP-HCM

### Visual Completeness
- [ ] All pages designed for 1440px, 1024px, and 390px breakpoints
- [ ] Light and dark theme variants complete
- [ ] All component states documented (default, hover, active, disabled, error, loading, empty)
- [ ] Realistic data used (no "Lorem ipsum" or "John Doe")

### Accessibility
- [ ] Color contrast ratios verified (>= 4.5:1 text, >= 3:1 UI)
- [ ] Touch targets >= 44x44px verified
- [ ] Focus order annotated
- [ ] Screen reader annotations added for all non-text elements
- [ ] Keyboard navigation paths documented

### Developer Handoff
- [ ] Design tokens exported in Style Dictionary JSON format
- [ ] Component specs include spacing, sizing, and behavioral notes
- [ ] API data mapping documented (which service endpoint populates which component)
- [ ] Interaction specifications (transitions, animations, gesture handling)
- [ ] Error states and empty states designed for every data-dependent component

### AIDD Guardrails
- [ ] No prohibited actions (cross-tenant data access, irreversible deletes) exposed without safeguards
- [ ] Supervised actions (data mutations, bulk operations) have confirmation dialogs
- [ ] Audit trail access points visible for compliance-service integration
- [ ] Feature flag annotation points marked for conditional rendering
- [ ] Human-in-the-loop gates marked for high-risk operations (payroll processing, termination)
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial comprehensive prompt set covering dashboard, employee management, payroll, leave, recruitment, LMS, mobile views, tablet adaptations, and 6 Make automation workflows |

---

## 9. Round 03 Implementation Handoff (Web + Mobile + Desktop)

### 9.1 Delivered UI Surfaces
- `web/`: role-based HCM ops console for HR Director, Payroll Lead, Recruiter, and Team Manager.
- `apps/mobile/`: workforce pulse and approval queue flow for manager-first operations.
- `apps/desktop/`: HR command center for operational oversight and multi-stream monitoring.

### 9.2 Design Token Mapping
- Tokens are implemented in `packages/design-tokens/tokens.css` and consumed by web/mobile/desktop apps.
- Palette: primary `#0f6fa8`, success `#16a34a`, warning `#f59e0b`, danger `#dc2626`.
- Typography baseline: Plus Jakarta Sans (heading), DM Sans (body), JetBrains Mono (codes/IDs).

### 9.3 Figma Make Build Prompt (Round 03)
```
Build ERP-HCM GA-ready UI kit and application flows using the existing token system.

Deliverables:
1. Desktop web ops console with pages:
   - Dashboard, Employee Directory, Payroll Operations, Leave & Attendance, Recruitment Pipeline, Performance.
2. Mobile manager app with:
   - KPI cards, approval queue, low-latency action sheet patterns.
3. Desktop command center view with:
   - workforce health, payroll SLA, compliance alerts, shift coverage, training completion.

Interaction constraints:
- p95 interaction latency <= 120ms for primary workflows.
- approval CTA controls visible above fold on 1366x768.
- preserve AIDD confirmation patterns for high-risk actions.
```

### 9.4 Commercial UX Guardrails
- No dead-end states: every error/empty state includes clear recovery actions.
- Role-first navigation: all primary roles can complete top 3 tasks within two clicks.
- KPI-first information architecture: actionable metrics shown before raw tables.

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
Design a control plane view for ERP-HCM that shows:
1. Hasura GraphQL connectivity (latency, success %, schema drift status)
2. ERP-DBaaS health (connection pool, replica lag, failover posture)
3. ERP-IAM integration (OIDC issuer health, token validation errors, session invalidations)
4. ERP-Observability status (OTLP export rate, tracing availability, alert health)

Include: command surface, timeline, runbook links, and one-click diagnostics panel.
Add states for: fully healthy, degraded IAM, degraded DB, degraded GraphQL, global outage.
```

### Prompt F-901: Role-Aware Workspace + Guardrail Surface
```
Generate a role-aware workspace for ERP-HCM with:
- Role lenses: Operator, Manager, Auditor, Admin
- Guardrail panel with mode badge (Protected / Supervised / Autonomous)
- Inline policy rationale for blocked/supervised actions
- Approval request flow for supervised actions

Ensure each role sees distinct navigation and action priorities.
```

### Prompt F-902: Incident + Recovery UX
```
Design a full incident-response flow for ERP-HCM:
- Alert ingestion to triage board
- Impacted tenant view and blast-radius visualization
- Action timeline with runbook steps and rollback controls
- Post-incident report generator modal

Must include: trace-id copy, audit evidence export, and SLA breach indicators.
```

### Prompt F-903: Mobile-Responsive Executive Snapshot
```
Create mobile and tablet executive dashboards for ERP-HCM:
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
Create a Make scenario for ERP-HCM that:
1. Triggers on deployment/webhook events
2. Validates GraphQL, IAM, DB, and OTLP health in sequence
3. Creates incident ticket when thresholds are breached
4. Posts status updates to operations channels
5. Writes audit trail to persistent store

Add branching for: transient failures, policy violations, and hard-stop conditions.
```

### Prompt M-901: Tenant Onboarding Orchestration
```
Generate a Make scenario for tenant onboarding in ERP-HCM:
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
