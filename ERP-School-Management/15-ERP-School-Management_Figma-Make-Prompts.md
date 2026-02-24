# Figma & Make Prompts -- ERP-School-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma Make prompts for the **ERP-School-Management** module -- a comprehensive education vertical for schools and universities. ERP-School-Management handles student records, academics (multi-curriculum), learning management (LMS), finance (fees, scholarships), placements, research, gamification, communications, events, analytics, IoT campus management, and multi-app experiences (web, mobile, parent portal, teacher portal, bus tracking). These prompts generate high-fidelity screens for Administrators, Teachers, Students, Parents, Placement Officers, Finance Staff, and Campus Operations.

**Services covered:** academic-service, admin-service, ai-service, aiops-service, analytics-service, auth-service, blockchain-service, communication-service, event-service, file-service, finance-service, gamification-service, gateway-service, integration-service, iot-service, lms-service, migration-service, notification-service, placement-service, research-service, scholarship-service, search-service, student-service, subscription-service

**API surface:** `/v1/:service/*` via NestJS gateway, `/v1/capabilities`, `/v1/events`

**Multi-app targets:** Web admin, Student portal, Parent app (mobile), Teacher app (mobile/tablet), Bus tracking app

---

## 2. AIDD Guardrails (Apply To All Prompts)

Every prompt must comply with these cross-cutting guardrails. Append the relevant subset when pasting into Figma Make.

### 2.1 User Experience And Accessibility
- WCAG 2.2 AA minimum; AAA for student-facing content (reading text, assessment questions).
- Keyboard navigable: Tab/Shift-Tab, Enter/Space, Escape for overlays, arrow keys for timetable and calendar navigation.
- Command palette via `Cmd+K` / `Ctrl+K` on admin/teacher portals: "Find Student", "Create Assignment", "View Grades", "Schedule Event".
- Role-first layout with completely distinct interfaces:
  - **Administrator**: full system access, analytics, configuration, compliance.
  - **Teacher**: class management, grade entry, attendance, LMS content authoring, communication.
  - **Student**: dashboard, courses, assignments, grades, LMS, gamification, events, placement.
  - **Parent**: child progress, attendance, fees, communication with teachers, bus tracking.
- Progressive disclosure: overview dashboards first, drill into individual student records, course details, financial breakdowns.
- Skeleton loading for grade books, analytics charts, and LMS content; optimistic UI for attendance marking and assignment submission.
- Academic calendar awareness: UI adapts to current semester/term, shows countdown to exams, registration deadlines.
- Multi-language critical: support for English, Spanish, French, Arabic (RTL), Hindi, Mandarin at minimum.
- Age-appropriate design: Student-facing UI uses larger text (16px body minimum), clear hierarchy, encouraging tone.
- Touch targets: 48x48px on student/parent mobile apps for younger users.

### 2.2 Performance And Frontend Efficiency
- Lighthouse Performance: >= 95.
- LMS video player: adaptive streaming, < 2s to first frame.
- Grade book: virtual scroll for 500+ students at 60fps.
- Analytics dashboards: lazy-load charts, < 1.5s interactive.
- First Contentful Paint < 1.2s; Largest Contentful Paint < 2.0s on 4G.
- Mobile app offline capability annotations: attendance marking, assignment viewing work offline.
- Bundle budget: < 200KB initial JS (gzipped).

### 2.3 Reliability, Trust, And Safety
- Student data privacy: FERPA/GDPR compliance indicators. Never display full student ID or SSN; mask to last 4 digits.
- Grade modifications require audit trail: confirmation dialog with reason field for any grade change.
- Financial transactions (fee payments, refunds): confirmation dialog with amount, breakdown, and receipt preview.
- AI recommendations (course suggestions, placement matching, early warning): always show confidence level and "This is a suggestion, not a decision" disclaimer.
- Parental consent gates: parent must authorize certain student actions (field trip enrollment, data sharing with placement partners).
- Gamification safety: XP and leaderboards never penalize; only reward. Option to opt-out of public leaderboards.
- Communication safety: messages between students and teachers are logged; parents can view.

### 2.4 Observability And Testability
- `data-testid` on every interactive element: `{portal}-{page}-{component}-{action}`.
- Analytics events: page_view, assignment_submitted, grade_entered, attendance_marked, fee_paid, course_enrolled, event_registered.
- Error states: no courses found, enrollment failed, payment declined, LMS content unavailable, bus location unknown -- each with appropriate illustration and action.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a Figma design-token page titled "ERP-School / Tokens" at 1440x3500px.

COLOR PALETTE (light mode):
- Primary: #4F46E5 (Education Indigo)
- Primary Hover: #4338CA
- Secondary: #0891B2 (Academic Teal)
- Accent: #D946EF (Gamification Fuchsia)
- Success: #16A34A
- Warning: #F59E0B
- Danger: #DC2626
- Neutral-50 through Neutral-900 (9-step gray scale)
- Surface: #FFFFFF
- Background: #F3F4F6

ACADEMIC COLORS:
- Science: #2563EB (Blue)
- Mathematics: #DC2626 (Red)
- Languages: #16A34A (Green)
- Arts: #D946EF (Fuchsia)
- Social Studies: #F59E0B (Amber)
- Technology: #0891B2 (Teal)
- Physical Education: #F97316 (Orange)
- Electives: #6B7280 (Gray)

GRADE COLORS:
- A / Excellent: #16A34A
- B / Good: #2563EB
- C / Satisfactory: #F59E0B
- D / Needs Improvement: #F97316
- F / Failing: #DC2626

GAMIFICATION COLORS:
- XP: #D946EF (Fuchsia)
- Level Up: #F59E0B (Gold)
- Badge: #4F46E5 (Indigo)
- Streak: #F97316 (Orange)
- Leaderboard: gradient Indigo-to-Fuchsia

PORTAL-SPECIFIC ACCENTS:
- Admin Portal: #4F46E5 (Indigo)
- Teacher Portal: #0891B2 (Teal)
- Student Portal: #7C3AED (Violet)
- Parent Portal: #059669 (Emerald)

COLOR PALETTE (dark mode):
- Primary: #818CF8 (Indigo-400)
- Secondary: #22D3EE (Teal-400)
- Accent: #E879F9 (Fuchsia-400)
- Surface: #1F2937
- Background: #111827

TYPOGRAPHY:
- Display: 36px / 700 / 1.1 (Inter)
- H1: 30px / 700 / 1.2
- H2: 24px / 600 / 1.3
- H3: 20px / 600 / 1.3
- H4: 16px / 600 / 1.4
- Body-lg: 16px / 400 / 1.5 (student-facing minimum body size)
- Body: 14px / 400 / 1.5
- Caption: 12px / 400 / 1.4
- Overline: 11px / 600 / 1.4 / uppercase
- Student ID: 13px / JetBrains Mono / 400 (ID codes, roll numbers)

SPACING: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96px
ELEVATION: shadow-sm, shadow-md, shadow-lg, shadow-xl
BORDER RADIUS: 4px (sm), 8px (md), 12px (lg), 16px (xl), 9999px (full for avatars)
MOTION: 150ms (micro), 250ms (standard), 350ms (page transition), 500ms (celebration animation for gamification)

Show each token as labeled swatch with light/dark side-by-side. Include all portal accent variants.
```

#### Prompt F-002: Component Library

```
Create a Figma component library page titled "ERP-School / Components" at 1440x7000px.

Include components with light/dark variants and all states:

BUTTONS: Primary, Secondary, Ghost, Danger, Icon-only. Sizes: sm, md, lg.
INPUTS: Text, Search (Cmd+K), Select, Multi-select, Date picker, Time picker, File upload (assignment submission), Rich text editor toolbar, Grade input (letter/number/percentage toggle).

STUDENT COMPONENTS:
- Student Card: avatar (64px circle, fallback initials), name, roll number (mono), grade/class, section, status badge (Active/Alumni/Suspended/Transferred).
- Student Avatar: circle image with online status dot (green/gray). Sizes: 24, 32, 40, 64px.
- Grade Badge: letter grade in colored circle (A=green, B=blue, C=amber, D=orange, F=red). Variants: with percentage, with GPA.
- Attendance Marker: date cell with status icon: Present (green check), Absent (red X), Late (amber clock), Excused (blue E), Holiday (gray dash).

ACADEMIC COMPONENTS:
- Course Card: subject color stripe (left), course name, teacher name + avatar, schedule (days + time), room number, enrolled count / capacity, progress bar (for semester completion).
- Timetable Cell: colored by subject, compact: time + subject + room. Hover/tap for detail.
- Assignment Card: title, course name, due date (red if overdue, amber if < 24h), submission status (Not Started/In Progress/Submitted/Graded), grade if available, file attachment count.
- Exam Score Bar: horizontal bar showing student score vs class average vs maximum, with percentile annotation.

LMS COMPONENTS:
- Lesson Card: thumbnail, lesson title, type icon (video/document/quiz/interactive), duration, completion status (not started/in progress/completed), progress bar.
- Quiz Question: question text, answer options (radio for single, checkbox for multiple), "Submit" button, progress indicator (Q3 of 10).
- Content Viewer: embedded video player with progress bar, playback controls, transcript toggle, speed control, bookmark button.

GAMIFICATION COMPONENTS:
- XP Badge: circular, showing current XP / next level requirement, animated fill.
- Achievement Badge: icon + name + description + earned date. Locked variant (grayed out, "?" icon).
- Streak Counter: flame icon + day count + "Keep it up!" text. Broken streak variant.
- Leaderboard Row: rank (1/2/3 with medal), avatar, name, XP, level, trend arrow.
- Level Progress Bar: current level, XP bar to next level, estimated actions to level up.

FINANCE COMPONENTS:
- Fee Card: fee type, amount (currency formatted), due date, status badge (Paid/Due/Overdue/Partial), payment method icon.
- Payment Receipt: mini-receipt format with institution name, student name, amount, date, transaction ID (mono), QR code placeholder.
- Scholarship Badge: scholarship name, amount, status (Active/Expired/Applied/Awarded).

COMMUNICATION COMPONENTS:
- Message Bubble: sender avatar + name, message text, timestamp, read receipt indicator. Variants: sent (right-aligned, primary bg), received (left-aligned, surface bg).
- Announcement Card: title, body preview (2 lines), author + avatar, date, audience badge (All/Grade 10/Section A), pinned indicator.

NAVIGATION:
- Admin sidebar: collapsible, with nav groups (Academics, Students, Finance, LMS, Analytics, Campus, Settings).
- Student/Parent bottom tab bar: Dashboard, Courses, Grades, Calendar, More.
- Teacher top bar: class selector dropdown, date, search, notifications.

FEEDBACK: Toast, Modal, Confirmation dialog, Empty state ("No assignments yet"), Celebration animation (confetti for achievements), Skeleton loaders.

Named layers: `{portal}/{category}/{component}/{variant}/{state}`.
```

---

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Admin Dashboard (1440px)

```
Design a desktop admin dashboard titled "ERP-School / Admin Dashboard" at 1440x1200px.

LAYOUT:
- Left sidebar (240px, collapsible): School logo + name "Greenwood International Academy", nav groups: "Overview" (Dashboard, Analytics, Calendar), "Academic" (Courses, Curriculum, Timetable, Exams), "Students" (Directory, Admissions, Attendance, Grades), "Staff" (Teachers, Non-Teaching, Payroll), "Finance" (Fees, Scholarships, Budgets, Payments), "LMS" (Content, Assessments, Reports), "Campus" (Events, Facilities, IoT, Transport), "Communication" (Announcements, Messages, Parent Portal), "Placement" (Companies, Drives, Reports), "Research" (Projects, Publications, Grants), "System" (Settings, Integrations, Audit, AIDD). Active: Dashboard.
- Top bar (64px): Academic year selector "2025-2026 | Semester 2", search, notification bell "18", admin avatar "Dr. Priya Sharma, Principal".
- Content: 24px padding.

CONTENT:
Row 1 -- KPI Cards (5 across):
- "Total Students": 3,247 | Active: 3,180, On Leave: 67 | mini trend
- "Attendance Today": 94.2% | 3,063 present | 117 absent | green gauge
- "Fee Collection (Semester)": $2.8M / $3.4M | 82% collected | amber
- "Active Courses": 186 | 24 departments | blue
- "Campus IoT Alerts": 3 | HVAC Zone B, Water Leak Lab 4, Parking Full | amber

Row 2 -- Two panels:
Left (55%): "Academic Performance Overview" grouped bar chart:
X-axis: departments (Science, Mathematics, Languages, Arts, Technology, Social Studies).
Bars per department: Average Grade (colored by grade: A=green, B=blue, etc.), Pass Rate %. Reference line at 80% (minimum target).

Right (45%): "Attendance Trend" area chart:
Last 30 school days. Lines: Overall (blue), Elementary (green), Middle School (amber), High School (indigo). Y-axis: percentage (80-100%). Today's value highlighted.

Row 3 -- Three panels:
Left (33%): "Upcoming Events" list:
6 events: icon (colored by type), event name, date, location.
- "Science Fair 2026" | Mar 5 | Auditorium
- "Parent-Teacher Conference" | Mar 12 | All Classrooms
- "Basketball Championship" | Mar 15 | Gymnasium
- "SAT Prep Workshop" | Mar 18 | Room 201
- "Spring Break Begins" | Mar 22
- "Admission Deadline (Fall)" | Apr 1

Center (34%): "Fee Collection Status" stacked bar:
By grade level: Elementary, Middle, High School, Graduate.
Each bar: Paid (green), Partial (amber), Overdue (red), Not Due Yet (gray).

Right (33%): "AI Early Warning" feed:
4 AI-generated alerts with confidence badges:
- "12 students at risk of falling below minimum GPA (< 2.0). Intervention recommended." Confidence: High (0.88).
- "Attendance drop in Grade 10 Section B: 15% below average this week." Confidence: Medium (0.76).
- "Fee collection pace 8% behind last semester at this point." Confidence: High (0.91).
- "Placement season: 34 students have not uploaded resumes. Deadline in 14 days." Confidence: High (0.95).

Row 4 -- "Quick Actions" row:
6 icon-button cards: "Mark Attendance", "Enter Grades", "Send Announcement", "View Calendar", "Generate Report", "Manage Admissions".

Light and dark mode.
```

#### Prompt F-004: Student Directory & Profile (1440px)

```
Design a desktop page titled "ERP-School / Student Directory" at 1440x1300px.

LAYOUT: Standard admin sidebar/top bar. Active: Students > Directory. Breadcrumb: "Students > Directory".

CONTENT:
Header: "Student Directory" (H1), "3,247 students enrolled" subtitle. Right: "Add Student" primary, "Import CSV" ghost, "Export" ghost.

Filter bar: Search "Search by name, ID, or email...", Grade Level (All/Elementary/Middle/High/Graduate), Section, Status (Active/On Leave/Alumni/Suspended), Scholarship (Yes/No), Gender, Year of Admission.

View toggle: Grid | List (active).

Data table (full width):
Columns: Photo (32px circle) | Student ID (mono, masked last 4 visible: ***-4521) | Full Name | Grade | Section | GPA | Attendance % | Fee Status badge | Scholarship badge | Actions kebab.

15 rows with diverse student data:
- [avatar] | ***-4521 | "Amara Okonkwo" | Grade 11 | A | 3.74 | 96% | Paid (green) | Merit Scholar (indigo badge) | View/Edit/...
- [avatar] | ***-1893 | "Liam Chen" | Grade 9 | B | 3.21 | 88% | Partial (amber) | -- | View/Edit/...
- [avatar] | ***-7734 | "Sofia Rodriguez" | Grade 12 | A | 3.92 | 99% | Paid | STEM Scholar | View/Edit/...
- [avatar] | ***-2210 | "Raj Patel" | Grade 10 | C | 2.15 | 72% (red) | Overdue (red) | -- | View/Edit/...
- (11 more diverse names, grades, statuses)

Pagination: "Showing 1-15 of 3,247" | page size | prev/next.

STUDENT PROFILE (slide-in panel, 560px, shown for "Amara Okonkwo"):
Header: large avatar (80px), name, ID (masked), grade/section, status "Active" green badge, "Email Student" / "Email Parent" action buttons.

Tabs: Overview | Academics | Attendance | Finance | LMS | Activities | Placement.

Overview tab:
- Personal Info card: DOB, gender, blood group, address (masked), emergency contact (masked).
- Guardian Info: Parent name, relationship, phone (masked), email.
- Enrollment: admission date, curriculum (IB/Cambridge/National), house/team.
- AI Summary: "Amara is a high-performing student in the STEM track with consistent 96% attendance. Recommend for advanced placement in Mathematics. Confidence: Medium (0.79)." Disclaimer: "This is an AI-generated summary for reference only."

Academics tab content:
- Current semester GPA: 3.74 / 4.0
- Course list with grades: 6 courses, each with current grade, trend arrow, class rank percentile.
- GPA trend chart: 4 semesters, line chart.

Finance tab content:
- Outstanding balance: $0 (all paid)
- Payment history: 4 entries with date, amount, method, receipt #.
- Scholarship: "Merit Scholar -- $5,000/year -- Active through 2027."

Light and dark mode.
```

#### Prompt F-005: Timetable & Academic Calendar (1440px)

```
Design a desktop page titled "ERP-School / Timetable" at 1440x1100px.

LAYOUT: Standard sidebar/top bar. Active: Academic > Timetable. Breadcrumb: "Academic > Timetable".

CONTENT:
Header: "Master Timetable" (H1). Right: "Edit Mode" toggle switch, "Export PDF" ghost, filters: Grade dropdown "Grade 11", Section dropdown "Section A".

View tabs: Weekly (active) | Daily | Term Overview.

WEEKLY TIMETABLE GRID (full width):
Row headers (left, 80px): Time slots in 45-min periods:
- 08:00 - 08:45 (Period 1)
- 08:45 - 09:30 (Period 2)
- 09:30 - 09:45 (Break)
- 09:45 - 10:30 (Period 3)
- 10:30 - 11:15 (Period 4)
- 11:15 - 12:00 (Period 5)
- 12:00 - 12:45 (Lunch)
- 12:45 - 13:30 (Period 6)
- 13:30 - 14:15 (Period 7)
- 14:15 - 15:00 (Period 8)

Column headers: Monday | Tuesday | Wednesday | Thursday | Friday.

Cells (colored by subject):
Monday:
- P1: Math (red) "Calculus | Mr. Johnson | Room 301"
- P2: Math (red) "Calculus (cont.)"
- Break
- P3: Physics (blue) "Mechanics | Dr. Lee | Lab 2"
- P4: Physics (blue) "Lab Session"
- P5: English (green) "Literature | Ms. Adams | Room 205"
- Lunch
- P6: Computer Science (teal) "Data Structures | Mr. Park | Lab 1"
- P7: Computer Science (teal) (cont.)
- P8: Free Period (gray, dashed border)

(Fill all 5 days with realistic schedule including: Art, PE, History, Chemistry, study halls, and assemblies.)

Break/Lunch rows: gray background, full width, no subject cells.

Cell hover state: expanded tooltip with full course name, teacher, room, enrolled count.

ACADEMIC CALENDAR (show as separate frame for "Term Overview" tab):
Month-view calendar for March 2026:
- Regular school days: white background.
- Holidays: green background with label ("Spring Break").
- Exam days: red border with label ("Mid-Term Exams").
- Events: small colored dot below date number.
- Today: bold border.
- Legend: Holiday, Exam, Event, Normal Day.

Light and dark mode.
```

#### Prompt F-006: LMS Course View (1440px)

```
Design a desktop page titled "ERP-School / LMS Course View" at 1440x1400px.

NOTE: This is the STUDENT-FACING LMS view. Use the Student Portal accent color (violet).

LAYOUT:
- Top bar (64px): School logo, "Student Portal" label, course breadcrumb "My Courses > AP Physics - Mechanics", search, notifications, student avatar "Amara Okonkwo".
- Left sidebar (240px): Course navigation: "Overview", "Modules" (expanded: Module 1-5), "Assignments", "Grades", "Discussions", "Resources", "Announcements". Active: Module 3.

CONTENT:
Course header (full width, 120px, gradient violet):
- Course name: "AP Physics - Mechanics" (H2, white)
- Teacher: "Dr. Lee" avatar + name
- Progress: "Module 3 of 5 | 58% complete" with progress bar (white on violet)
- Next deadline: "Lab Report due Feb 26" (amber badge)

Module 3 content area:
Module title: "Module 3: Newton's Laws of Motion" (H2).
Module progress: "4/7 lessons completed" with progress bar.

Lesson list (vertical, full width):
Each lesson row:
- Completion icon: green checkmark (done), blue play (current), gray lock (locked), gray circle (not started).
- Lesson number + title
- Type icon: video camera, document, quiz brain, interactive flask
- Duration / questions count
- "Continue" / "Start" / "Review" button (right)

7 lessons:
1. Checkmark | "Introduction to Newton's Laws" | Video | 15 min | Review
2. Checkmark | "First Law: Inertia" | Video + Interactive | 22 min | Review
3. Checkmark | "Second Law: F = ma" | Video | 18 min | Review
4. Checkmark | "Practice Problems Set 1" | Quiz (12 Qs) | Scored: 10/12 | Review
5. Blue play (current) | "Third Law: Action and Reaction" | Video | 20 min | Continue (primary)
6. Gray circle | "Lab Simulation: Force and Acceleration" | Interactive | 30 min | Locked (unlock after lesson 5)
7. Gray circle | "Module 3 Assessment" | Quiz (20 Qs) | 45 min | Locked

Current lesson expanded preview (for lesson 5):
Video player placeholder (full width, 400px):
- Thumbnail image showing physics diagram.
- Play button overlay.
- Duration bar.
- Controls: play/pause, volume, speed (1x/1.5x/2x), captions toggle, fullscreen, bookmark.
- Transcript panel toggle (right sidebar, 300px).

Below video: "Related Resources" section: 3 downloadable PDFs + 2 external links.

GAMIFICATION SIDEBAR (right panel, 280px, only on student portal):
- "Your Progress" card: XP: 2,450 / 3,000 to Level 8 (progress bar with fuchsia fill).
- Current streak: "12-day learning streak" (flame icon + number).
- Recent badges: 3 badge icons (e.g., "Physics Pro", "Quiz Master", "Perfect Attendance").
- Leaderboard peek: "You are #7 in AP Physics" with top 3 names.

Light and dark mode.
```

#### Prompt F-007: Grade Book (Teacher View) (1440px)

```
Design a desktop page titled "ERP-School / Teacher Grade Book" at 1440x1200px.

NOTE: This is the TEACHER-FACING portal. Use Teacher Portal accent (teal).

LAYOUT:
- Top bar (64px): School logo, "Teacher Portal" label, class selector dropdown "AP Physics - Grade 11A", search, bell, avatar "Dr. Lee".
- Left sidebar (200px): Dashboard, My Classes (expanded: AP Physics 11A, AP Physics 11B, General Physics 10), Grade Book (active), Attendance, Assignments, LMS Content, Messages, Calendar.

CONTENT:
Header: "Grade Book: AP Physics - Grade 11A" (H1), "28 students" subtitle. Right: "Add Assessment" primary, "Import Grades" ghost, "Export Report" ghost, "AI Analysis" secondary (AI icon).

Assessment columns headers (horizontal scroll):
- Student Name (sticky left, 200px)
- "Quiz 1 (10)" | "Lab Report 1 (20)" | "Midterm (100)" | "Quiz 2 (10)" | "Lab Report 2 (20)" | "Assignment 1 (15)" | "Quiz 3 (10)" | "Final Project (25)"
- "Total" (auto-calculated) | "Grade" | "Attendance %" | "Actions"

Grade grid (28 rows):
Each cell: score entry (editable), colored background based on performance (green >= 80%, amber 60-79%, red < 60%).

Sample rows:
- "Amara Okonkwo" | 9 | 18 | 92 | 10 | 19 | 14 | 9 | 24 | 195/210 (93%) | A | 96% | kebab
- "Liam Chen" | 7 | 15 | 78 | 8 | 16 | 12 | 7 | 20 | 163/210 (78%) | B+ | 88% | kebab
- "Raj Patel" | 4 (red bg) | 10 (red) | 55 (red) | 5 (red) | 11 | 8 (red) | 4 (red) | -- | 97/185 (52%) | F | 72% (red) | kebab
- (25 more rows with varied performance)

Class statistics bar (below grid):
- Class Average: 78.4% (B+) | Highest: 95% | Lowest: 42% | Median: 80% | Std Dev: 14.2
- Distribution histogram: A: 6, B: 10, C: 7, D: 3, F: 2 (mini bar chart).

AI Analysis panel (expandable, right sidebar 320px):
- "At-Risk Students" (3 flagged): name, current grade, attendance, AI recommendation.
  - "Raj Patel: Current F (52%). Attendance 72%. Recommend: schedule parent conference, offer tutoring." Confidence: High (0.89).
- "Grade Distribution Insight": "Class performance skews higher than department average. Quiz 1 had unusually high scores -- consider adjusting difficulty." Confidence: Medium (0.71).
- Disclaimer: "AI analysis is advisory. All decisions must be made by qualified educators."

Grade edit modal (triggered on cell click):
- Student name, assessment name, max score.
- Current score input.
- "Previous value: --" (for audit).
- Reason for change dropdown (Initial Entry / Correction / Late Submission / Regrade).
- Notes field.
- "Save" primary, "Cancel" ghost.

Light and dark mode.
```

#### Prompt F-008: Finance & Fee Management (1440px)

```
Design a desktop page titled "ERP-School / Finance Dashboard" at 1440x1200px.

LAYOUT: Standard admin sidebar/top bar. Active: Finance > Fees. Breadcrumb: "Finance > Fee Management".

CONTENT:
Header: "Fee Management" (H1), "Semester 2, 2025-2026" subtitle. Right: "Generate Invoice Batch" primary, "Payment Reminders" secondary, "Reports" ghost.

Row 1 -- KPI Cards (5 across):
- "Total Fees Due": $3.4M | 3,247 students
- "Collected": $2.8M (82%) | green with progress ring
- "Outstanding": $420K | 487 students | amber
- "Overdue (> 30d)": $180K | 134 students | red
- "Scholarships Applied": $340K | 156 students | indigo

Row 2 -- Two panels:
Left (55%): "Collection Trend" area chart:
Monthly for current academic year (Aug 2025 - Jul 2026). Stacked areas: Tuition (blue), Lab Fees (teal), Activity Fees (amber), Transport (gray). Cumulative line overlay. Target line (dashed).

Right (45%): "Fee Status by Grade" stacked horizontal bar:
Grade levels (K-12 + Graduate). Each bar: Paid (green), Partial (amber), Overdue (red), Not Yet Due (gray). Percentage labels.

Row 3 -- "Fee Ledger" table (full width):
Columns: Student ID (masked) | Student Name | Grade | Total Fee | Paid | Balance | Due Date | Status badge | Scholarship | Payment Method | Last Payment | Actions.

12 rows:
- ***-4521 | "Amara Okonkwo" | 11 | $8,500 | $8,500 | $0 | Jan 15 | Paid (green) | Merit: -$5,000 | Bank Transfer | Jan 12 | View
- ***-2210 | "Raj Patel" | 10 | $9,200 | $4,600 | $4,600 | Jan 15 | Overdue (red) | -- | -- | Dec 5 | View/Remind
- (10 more varied rows)

Pagination.

Row 4 -- Scholarship panel (full width):
"Active Scholarships" table: Scholarship Name | Type (Merit/Need/Athletic/STEM) | Count | Total Value | Fund Remaining | Expiry.
6 scholarships:
- "Merit Excellence Award" | Merit | 45 students | $225,000 | $180,000 remaining | Jul 2027
- "STEM Innovation Grant" | STEM | 20 students | $100,000 | $75,000 | Jul 2026
- (4 more)

Payment recording modal (560px):
- Student name + ID
- Fee breakdown table (line items)
- Amount to pay (pre-filled with balance)
- Payment method: Cash / Bank Transfer / Card / Check / Online Gateway
- Reference number
- Date
- Receipt preview (mini-receipt format)
- "Record Payment" primary

Light and dark mode.
```

---

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Student Mobile Dashboard (390px)

```
Design a mobile student dashboard titled "ERP-School / Student App" at 390x844px.

LAYOUT:
- Bottom tab bar (56px): Home (active), Courses, Grades, Calendar, Profile.
- Top bar (56px): School logo (24px), "Good morning, Amara" text, notification bell "5", avatar (32px).
- Content: scrollable, 16px padding.

CONTENT:
Gamification banner (full width, gradient violet-to-fuchsia, rounded-lg):
- "Level 7" badge, "2,450 XP" large text, progress bar to Level 8 (82%).
- "12-day streak" with flame icon.
- Recent badge: "Quiz Master" with small badge icon.

"Today's Schedule" section:
Compact timetable for today (Wednesday):
4 visible periods:
- 08:00 "Calculus" | Mr. Johnson | Room 301 | (completed, green check)
- 09:45 "Physics" | Dr. Lee | Lab 2 | (current, blue highlight, "NOW" badge)
- 11:15 "English Lit" | Ms. Adams | Room 205
- 12:45 "Comp Sci" | Mr. Park | Lab 1
"View Full Schedule" link.

"Due Soon" section:
3 assignment cards (compact):
- "Lab Report: Force & Acceleration" | Physics | Due: Tomorrow | amber badge | "Submit" button
- "Essay: Hamlet Act 3 Analysis" | English | Due: Feb 28 | 5 days left
- "Problem Set 7" | Calculus | Due: Mar 2 | 7 days left

"Recent Grades" section:
3 grade items:
- "Quiz 2: Newton's Laws" | Physics | 10/10 | A+ (green badge) | "New" dot
- "Midterm Exam" | Calculus | 87/100 | B+ (blue badge)
- "Assignment 1" | Comp Sci | 14/15 | A (green badge)

"Announcements" section:
2 items (compact): title, author, date.
- "Science Fair Registration Open" | Dr. Sharma | Today
- "Spring Break Schedule" | Admin | Yesterday

Light and dark mode.
```

#### Prompt F-010: Parent Mobile App (390px)

```
Design a mobile parent dashboard titled "ERP-School / Parent App" at 390x844px.

NOTE: This is the PARENT-FACING app. Use Parent Portal accent (emerald green).

LAYOUT:
- Bottom tab bar (56px): Home (active), Attendance, Fees, Messages, Bus.
- Top bar (56px): School logo, "Parent Portal", bell "3", avatar.
- Content: scrollable, 16px padding.

CONTENT:
Child selector (if multiple children): horizontal scroll of child cards (140px each):
- Avatar + name + grade: "Amara Okonkwo | Grade 11" (selected, emerald border), "Kemi Okonkwo | Grade 8".

"Today" card:
- "Amara is in school" green badge with checkmark.
- Current class: "Physics with Dr. Lee | Lab 2"
- Next: "English Literature | 11:15"
- "Marked present at 07:52" (caption).

"This Week's Attendance" row: Mon (green check) | Tue (green check) | Wed (green check, pulsing for today) | Thu (gray dash) | Fri (gray dash). "96% overall" caption.

"Academic Snapshot" card:
- Current GPA: 3.74 / 4.0 (large, emerald text)
- Trend: arrow up green, "+0.08 from last semester"
- Recent grades: 3 items (compact): subject, assessment, grade badge.

"Fee Status" card:
- "All fees paid for Semester 2" green banner with checkmark.
- "Next due: $8,500 | Aug 2026 (Semester 3)"
- "View Payment History" link.
OR (for a different child with outstanding fees):
- "Outstanding balance: $4,600" red text.
- "Due date: Jan 15, 2026 (38 days overdue)" red badge.
- "Pay Now" emerald button | "View Details" ghost.

"Messages" preview (2 items):
- From Dr. Lee: "Amara did excellent work on..." | 2h ago | unread dot
- From Admin: "Parent-Teacher Conference scheduled for..." | Yesterday

"Bus Tracking" card:
- Mini map (150px height) showing bus route with bus icon.
- "Bus #7 | ETA: 3:22 PM | 4 stops away"
- "Track Live" emerald button.

Light and dark mode.
```

#### Prompt F-011: Mobile LMS Lesson (390px)

```
Design a mobile page titled "ERP-School / Mobile LMS Lesson" at 390x844px.

LAYOUT: Top bar: back arrow, "Newton's Third Law" title, bookmark icon. No bottom tab bar.

CONTENT:
Video player (full width, 220px):
- Video thumbnail with play button overlay.
- Progress bar (30% complete).
- Controls: play/pause, 15s rewind, 15s forward, speed (current: 1x), captions (CC), fullscreen.
- Duration: "08:22 / 20:00".

Below video -- content section (scrollable):
Lesson info:
- "Lesson 5 of 7 | Module 3: Newton's Laws"
- "4/7 lessons completed" progress bar.

Transcript / Notes tabs:
Transcript tab: timestamped text, tappable to jump to video position.
- "00:00 - In this lesson, we'll explore Newton's third law..."
- "02:15 - For every action, there is an equal and opposite reaction..."
- (continues)

Notes tab: student's personal notes with rich text editor (bold, italic, bullet, highlight).

"Key Concepts" section (expandable):
- "Action-reaction pairs" with diagram thumbnail
- "Force vectors in opposite directions"
- "Applications: rocket propulsion, walking, swimming"

"Practice" section:
3 quick-check questions (inline, multiple choice):
- "Q1: If a book pushes down on a table with 10N, what force does the table exert on the book?"
  - A) 0N  B) 5N  C) 10N (correct, green highlight)  D) 20N

Bottom navigation bar (sticky):
"< Previous Lesson" (ghost) | Progress dots (7, current = 5) | "Next Lesson >" (primary, or "Complete & Continue" if video finished).

Gamification toast (overlay): "+50 XP earned for completing lesson" with progress bar animation.

Light and dark mode.
```

#### Prompt F-012: Mobile Attendance (Teacher) (390px)

```
Design a mobile page titled "ERP-School / Teacher Attendance" at 390x844px.

NOTE: Teacher-facing mobile app with teal accent.

LAYOUT: Top bar: back arrow, "Mark Attendance" title, class info. No bottom tab bar (action-focused flow).

CONTENT:
Class selector card:
- "AP Physics - Grade 11A" | "Period 3 | 09:45 - 10:30" | "Feb 23, 2026"
- "28 students" | "26 present, 2 absent so far"

Quick action bar:
- "Mark All Present" (teal button) | "Mark All Absent" (ghost) | "Undo" (ghost)

Student list (scrollable):
Each student row (full width, 64px height):
- Avatar (40px) + Student name + Roll number (caption)
- Right side: segmented toggle: P (green) | A (red) | L (amber) | E (blue)
  - P = Present, A = Absent, L = Late, E = Excused.
  - Tapping a segment marks attendance immediately (optimistic UI).

28 student rows with varied states:
- 22 marked "P" (green)
- 3 marked "A" (red)
- 2 marked "L" (amber)
- 1 not yet marked (all segments gray/neutral)

Alphabetical sort, with section headers if needed.

Search/filter bar (collapsible): search by name, filter: Not Yet Marked | Absent | Late.

Summary bar (sticky bottom, 72px):
- "26 Present | 3 Absent | 2 Late | 1 Pending"
- "Submit Attendance" teal primary button (disabled if any pending).

Confirmation dialog (overlay):
"Submit attendance for AP Physics 11A, Feb 23, 2026? 26 present, 3 absent, 2 late. This action will be recorded in the audit log."
"Submit" teal button | "Cancel" ghost.

Offline indicator (if applicable): yellow banner "You are offline. Attendance will sync when connected."

Light and dark mode.
```

#### Prompt F-013: Mobile Fee Payment (Parent) (390px)

```
Design a mobile page titled "ERP-School / Parent Fee Payment" at 390x844px.

NOTE: Parent-facing app with emerald accent.

LAYOUT: Top bar: back arrow, "Fee Payment" title. Accessed from parent dashboard.

CONTENT:
Student header: Avatar + "Amara Okonkwo" + "Grade 11 | ID: ***-4521".

Outstanding balance card (full width, red-tinged surface if overdue, green if none):
- "Outstanding Balance: $4,600" (H2, red) OR "$0 -- All Paid" (H2, green)
- "Due: Jan 15, 2026 (38 days overdue)" badge
- Late fee warning: "+$50 late fee applied" (caption, red)

Fee breakdown (expandable):
- Tuition Fee: $6,000 (Paid: $4,600, Remaining: $1,400)
- Lab Fee: $800 (Paid: $0, Remaining: $800)
- Activity Fee: $600 (Paid: $0, Remaining: $600)
- Transport Fee: $1,200 (Paid: $0, Remaining: $1,200)
- Scholarship Discount: -$5,000 (applied)
- Late Fee: $50
- Total Due: $4,650

Payment section:
Amount to pay: input pre-filled with "$4,650" (editable for partial payment).
Minimum: "$500 partial payment minimum" caption.

Payment method selector:
- Card (Visa ending 4521) -- selected, emerald border
- Bank Transfer
- Add New Payment Method (+)

"Pay $4,650" emerald primary button (full width, 48px).

Payment confirmation dialog:
"Confirm payment of $4,650 to Greenwood International Academy for Amara Okonkwo (Grade 11). Payment method: Visa ending 4521."
"Confirm & Pay" button | "Cancel"

Payment success page (third frame):
- Large green checkmark animation
- "Payment Successful" (H2)
- Amount: $4,650 | Date: Feb 23, 2026 | Ref: TXN-2026-8847
- Receipt card: institution, student, amount, date, transaction ID, QR code
- "Download Receipt" button | "Done" button
- "Receipt sent to okonkwo.parent@email.com" caption.

Light and dark mode.
```

#### Prompt F-014: Mobile Bus Tracking (Parent) (390px)

```
Design a mobile page titled "ERP-School / Bus Tracking" at 390x844px.

NOTE: Parent-facing with emerald accent. Real-time tracking.

LAYOUT: Bottom tab bar with "Bus" active. Top bar: "Bus Tracking" title, refresh icon.

CONTENT:
Map (full width, 350px height):
- School marker (building icon) at center-top.
- Home marker (house icon) at bottom.
- Bus route line (emerald dashed line) showing the route.
- Bus marker (bus icon, animated) at current position on route.
- Stop markers (small circles): 8 stops along route, 5 passed (green), current stop (pulsing emerald), 2 upcoming (gray).
- Traffic overlay option.

Bus info card (below map):
- "Bus #7" (H3) | "Route: Downtown - Greenwood" | Status: "In Transit" blue badge
- Driver: "Mr. David Williams" | phone call icon
- "ETA Home: 3:22 PM" (large, emerald) | "4 stops away"
- "Amara boarded at 07:48 AM" green text with checkmark
- Live speed: "35 km/h" | Capacity: "32/45 students"

Stop timeline (vertical):
8 stops with status:
- "Greenwood Academy (Departure)" | 15:00 | Done (green check)
- "Oak Street" | 15:05 | Done
- "Main Plaza" | 15:10 | Done
- "Park Avenue" | 15:14 | Done
- "River Bridge" | 15:18 | Done
- "Downtown Center" | 15:22 | Current (pulsing, "Bus is here")
- "Elm Street" | 15:28 (est.) | Upcoming (gray)
- "Cedar Lane (Your Stop)" | 15:32 (est.) | Upcoming, highlighted with emerald border and "Home" label.

Notification preferences (bottom section):
- "Notify me when bus is 2 stops away" toggle (on)
- "Notify me when child boards" toggle (on)
- "Notify me if bus is > 10 min late" toggle (on)

Push notification mockup (overlay): "Bus #7 is 2 stops away from Cedar Lane. ETA: 3:32 PM."

Light and dark mode.
```

---

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-015: Tablet Admin Dashboard (1024px)

```
Design a tablet admin dashboard titled "ERP-School / Tablet Admin" at 1024x768px.

LAYOUT: Icon-only sidebar (64px), top bar (56px) with academic year selector.

CONTENT:
KPI cards: 5 across (compact).
Academic performance chart: full width, 250px height.
Attendance trend: full width, 200px height.
Events + Fee Status + AI Warnings: 3-column grid below.
Quick actions: horizontal scroll of 6 icon-buttons.

Touch-optimized: 44px minimum targets, swipe for more items.

Light and dark mode.
```

#### Prompt F-016: Tablet Teacher Grade Book (1024px)

```
Design a tablet grade book titled "ERP-School / Tablet Grade Book" at 1024x768px.

PURPOSE: Teachers entering grades on a tablet during or after class.

LAYOUT: No sidebar. Full-screen grade entry. Top bar (48px): class selector, "Save" button, back arrow.

CONTENT:
Grade grid (full width):
- Student name column (180px, sticky left).
- Assessment columns (scrollable horizontally): show 5-6 visible, scroll for more.
- Cell size: 64x48px for easy tap-to-edit on tablet.
- Active cell: highlighted border with number pad popup (bottom sheet, large digits for touch).

Number pad bottom sheet (for grade entry):
- Current: "Student: Amara Okonkwo | Quiz 2 (max: 10)"
- Large number buttons (48px each): 0-9, backspace, decimal.
- Quick scores: "10/10", "0/10" shortcut buttons.
- "Save" (teal) and "Next Student" (ghost) buttons.

Class stats bar: fixed bottom, compact: avg, high, low, distribution mini-chart.

Light mode (classroom environment). High-contrast option.
```

#### Prompt F-017: Tablet Classroom Display (1024px)

```
Design a tablet page titled "ERP-School / Classroom Display" at 1024x768px (landscape).

PURPOSE: Wall-mounted or desk-mounted tablet showing class information. Read-only ambient display.

LAYOUT: Full screen, no sidebar or interactive elements. Auto-rotating content.

FRAME 1 -- "Today's Schedule" (auto-display for 15 seconds):
- School name + class "Grade 11 - Section A" (H2, centered).
- Current date + time (large).
- Timetable for today (vertical list): period, time, subject (color coded), teacher, room. Current period highlighted with pulsing border.

FRAME 2 -- "Announcements" (auto-display for 10 seconds):
- 3 current announcements, large text.
- School logo watermark.

FRAME 3 -- "Class Achievements" (auto-display for 10 seconds):
- Recent gamification achievements: 3 student names with badges earned.
- Class leaderboard top 5.
- Class average attendance this week.

FRAME 4 -- "Upcoming Events" (auto-display for 10 seconds):
- Next 3 events with date, name, location.

Transition: fade with 500ms duration.

Dark mode only (reduced glare for classroom). High-contrast text. Large fonts (minimum 20px body).
```

---

## 4. Make Automation Prompts

#### Prompt M-001: Attendance Workflow Automation

```
Create a Make (Integromat) scenario titled "ERP-School: Attendance Lifecycle".

TRIGGER: Webhook -- events from academic-service and notification-service.

FLOW:
1. Teacher submits attendance (webhook from academic-service):
   a. Parse: class_id, date, attendance_records[{student_id, status}].
   b. For each student marked "Absent":
      - HTTP GET `/v1/student/{id}` to fetch parent contact.
      - HTTP POST to notification-service: send push notification to parent app: "{{child_name}} was marked absent from {{class_name}} today at {{time}}."
      - HTTP POST to communication-service: send SMS to parent phone if configured.
   c. For each student marked "Late":
      - HTTP POST notification to parent: "{{child_name}} arrived late to {{class_name}} at {{time}}."
   d. Calculate daily attendance rate for class.
   e. If attendance < 85%: HTTP POST notification to admin: "Low attendance alert: {{class_name}} at {{rate}}% today."

2. Chronic absenteeism check (scheduled daily at 18:00):
   a. HTTP GET `/v1/analytics/attendance?threshold=80&period=30d` to find students below 80% attendance over 30 days.
   b. For each flagged student:
      - HTTP POST to notification-service: alert class teacher, grade coordinator, and parent.
      - HTTP POST to ai-service: generate intervention recommendation.
      - Create action item in admin-service for follow-up.

3. For all operations: HTTP POST to audit trail.

ERROR HANDLING: Per-student isolation. Continue processing even if individual notification fails.
```

#### Prompt M-002: Fee Collection & Reminder Pipeline

```
Create a Make scenario titled "ERP-School: Fee Collection Pipeline".

TRIGGER: Scheduled -- daily at 09:00 local timezone AND webhook on finance-service payment events.

FLOW:
1. Daily fee status check:
   a. HTTP GET `/v1/finance/fees?status=overdue` to fetch overdue accounts.
   b. For each overdue account:
      - Calculate days overdue.
      - If 7 days overdue: HTTP POST gentle reminder to parent (email + push).
      - If 14 days overdue: HTTP POST firm reminder with late fee notice.
      - If 30 days overdue: HTTP POST escalation to finance director + admin. Include student name, amount, contact info.
      - If 60 days overdue: HTTP POST final notice with consequences (service restriction warning).
   c. HTTP GET `/v1/finance/fees?due_within=7` for upcoming due dates.
   d. For each: HTTP POST pre-due reminder to parent: "Fee payment of ${{amount}} for {{child_name}} is due on {{date}}."

2. Payment received (webhook):
   a. HTTP PUT `/v1/finance/fees/{id}` to update payment status.
   b. HTTP POST receipt email to parent via notification-service.
   c. HTTP POST to blockchain-service: record immutable payment receipt.
   d. If payment completes all outstanding fees: HTTP POST confirmation to parent + admin.
   e. If partial payment: HTTP POST acknowledgment with remaining balance.
   f. HTTP POST to analytics-service: update collection metrics.

3. Monthly reconciliation (1st of each month):
   - Generate collection summary report.
   - HTTP POST to admin + finance director: monthly fee report.

ERROR HANDLING: Financial operations are idempotent. Retry payment recording 5x before manual intervention alert.
```

#### Prompt M-003: LMS Progress & Gamification Engine

```
Create a Make scenario titled "ERP-School: LMS Gamification Engine".

TRIGGER: Webhook -- events from lms-service (lesson_completed, quiz_submitted, assignment_submitted).

FLOW:
1. Router by event type:
   a. "lesson_completed":
      - HTTP POST to gamification-service `/v1/gamification/xp`: award XP based on lesson type:
        - Video watched: +30 XP
        - Interactive completed: +50 XP
        - Reading completed: +20 XP
      - HTTP GET `/v1/gamification/{student_id}/streak` to check learning streak.
      - If streak continues: increment streak counter. If streak milestone (7, 14, 30 days):
        - HTTP POST badge award to gamification-service.
        - HTTP POST notification to student: "Congratulations! {{streak_days}}-day learning streak! +100 bonus XP."

   b. "quiz_submitted":
      - HTTP GET quiz result from lms-service.
      - Award XP: score% * 100 (e.g., 90% = 90 XP).
      - If perfect score: HTTP POST "Perfect Score" badge.
      - If score < 60%: HTTP POST to ai-service for remediation recommendation.
        - HTTP POST notification to student: "Review recommended for {{topic}}. Here are some resources."
        - HTTP POST notification to teacher: "{{student_name}} scored {{score}}% on {{quiz_name}}. May need support."

   c. "assignment_submitted":
      - Award +40 XP for on-time submission. +20 XP for late submission.
      - If submitted > 24h early: award "Early Bird" bonus +25 XP.

2. Level-up check (after any XP award):
   - HTTP GET current XP and level thresholds.
   - If XP crosses level threshold:
     - HTTP POST level-up to gamification-service.
     - HTTP POST celebration notification: "Level Up! You reached Level {{level}}!"
     - HTTP POST to leaderboard update.

3. Weekly leaderboard compilation (Sunday 20:00):
   - HTTP GET top 50 students by XP.
   - HTTP POST leaderboard to gamification-service.
   - HTTP POST notification to top 3: "You're in the top 3 this week!"

ERROR HANDLING: XP operations are idempotent (event_id used as dedup key). Gamification must never block academic workflows.
```

#### Prompt M-004: Placement Drive Workflow

```
Create a Make scenario titled "ERP-School: Placement Drive Orchestrator".

TRIGGER: Webhook -- events from placement-service AND scheduled checks.

FLOW:
1. New placement drive created:
   a. HTTP GET eligible students from `/v1/student?grade=12&gpa_min={{company.min_gpa}}&status=active`.
   b. For each eligible student:
      - HTTP POST notification: "{{company_name}} is visiting for {{role_title}}. GPA requirement: {{min_gpa}}. Apply by {{deadline}}."
      - HTTP GET student profile completeness check.
      - If profile incomplete (missing resume/portfolio): HTTP POST reminder: "Complete your profile to apply. Missing: {{missing_items}}."
   c. HTTP POST to communication-service: send announcement to all eligible students.
   d. HTTP POST to event-service: create campus event for placement drive.

2. Application deadline reminder (3 days before):
   - HTTP GET students who are eligible but have not applied.
   - HTTP POST reminder notification to each.

3. After placement drive:
   a. HTTP POST results to placement-service.
   b. For selected students: HTTP POST congratulations notification + offer details.
   c. For not-selected students: HTTP POST encouragement notification + upcoming drive list.
   d. HTTP POST to analytics-service: placement statistics (applications, selections, offers).
   e. HTTP POST to admin: placement drive summary report.

4. Aggregate analytics (monthly):
   - Placement rate, average package, company satisfaction scores.
   - HTTP POST monthly report to admin + career counselor.

ERROR HANDLING: Student notifications are fire-and-forget. Placement data recording retries 3x.
```

#### Prompt M-005: IoT Campus Alert System

```
Create a Make scenario titled "ERP-School: IoT Campus Monitor".

TRIGGER: Webhook -- events from iot-service (sensor readings, threshold alerts).

FLOW:
1. Parse IoT event: sensor_id, location, metric_type, value, threshold, severity.

2. Router by severity:
   a. "critical" (fire alarm, water leak, security breach):
      - HTTP POST IMMEDIATE notification to campus operations, admin, security: "CRITICAL: {{alert_type}} at {{location}}. {{details}}."
      - HTTP POST to communication-service: campus-wide PA announcement (if integrated).
      - HTTP POST to event-service: create emergency event record.
      - HTTP POST to admin-service: create action item with highest priority.

   b. "warning" (HVAC temperature out of range, high occupancy, low air quality):
      - HTTP POST notification to facilities team: "Warning: {{metric}} at {{location}} is {{value}} (threshold: {{threshold}})."
      - If classroom affected during class hours: HTTP POST to teacher of that room.
      - Log warning in analytics-service for trend analysis.

   c. "info" (parking capacity, energy usage milestone):
      - Log to analytics-service.
      - If parking > 90%: HTTP POST to notification-service: campus-wide "Parking lot nearly full."

3. Energy efficiency check (hourly during school hours):
   - HTTP GET `/v1/iot/energy?zone=all` for current consumption.
   - Compare against baseline.
   - If > 120% of baseline: HTTP POST alert to facilities: "High energy consumption in {{zone}}. Investigate."

4. Daily IoT health report (06:00):
   - HTTP GET all sensor statuses.
   - Flag offline sensors.
   - HTTP POST report to facilities manager.

ERROR HANDLING: Critical alerts have 3x immediate retry with 1s delay. Never suppress critical alerts.
```

#### Prompt M-006: Academic Performance Early Warning

```
Create a Make scenario titled "ERP-School: AI Early Warning System".

TRIGGER: Scheduled -- weekly on Friday at 16:00 AND webhook on grade entry events.

FLOW:
1. Weekly comprehensive analysis:
   a. HTTP GET `/v1/analytics/students?risk_indicators=true` to fetch at-risk student data.
   b. For each student with risk score > 0.7 (AI-generated):
      - HTTP POST to ai-service `/v1/ai/student-risk-assessment/{student_id}`:
        - Inputs: grades (trending), attendance (trending), LMS engagement (declining?), behavioral notes.
        - Output: risk_score, risk_factors[], recommended_interventions[], confidence.
      - If confidence >= 0.80:
        - HTTP POST notification to class teacher: "Early warning for {{student_name}}: {{risk_summary}}. Recommended: {{interventions}}. Confidence: {{confidence}}."
        - HTTP POST notification to grade coordinator.
        - If risk_score > 0.9: HTTP POST to admin and parents: "Your child {{name}} may benefit from additional support in {{subjects}}. Please schedule a meeting with {{teacher_name}}."
      - If confidence < 0.70: log for human review, do not auto-notify.
   c. Disclaimer in all notifications: "This assessment is AI-generated and intended as an early indicator. Please review the student's full profile before taking action."

2. Grade entry trigger (real-time):
   - When a grade is entered that drops a student below the minimum passing threshold:
     - HTTP POST immediate notification to teacher: "{{student_name}} is now below passing in {{course}}. Current average: {{average}}."
     - HTTP POST to analytics-service: update risk assessment.

3. Monthly trend report:
   - HTTP GET department-level academic performance from analytics-service.
   - Generate comparative analysis: this month vs last month vs same month last year.
   - HTTP POST to academic director and department heads.

ERROR HANDLING: AI assessments are advisory only. Never auto-enroll students in interventions. Human action required.
```

---

## 5. Prompt Usage Guidelines

### How to Use These Prompts

1. **Figma Make**: Paste prompt content (without backticks) into Figma's "Make a Design" feature.
2. **Portal-specific generation**: ERP-School has 4+ distinct portals (Admin, Teacher, Student, Parent). Generate components for each portal accent color.
3. **Component order**: F-001 (tokens) and F-002 (components) first. They define academic colors, gamification tokens, and portal-specific accents.
4. **Mobile apps**: Student (F-009), Parent (F-010, F-013, F-014), and Teacher (F-012) are distinct apps with different navigation, accent colors, and feature sets.
5. **Tablet use cases**: Grade book entry (F-016) and classroom display (F-017) are specialized tablet interfaces. The classroom display is read-only ambient mode.
6. **Privacy**: All student data in mockups uses masked IDs. Ensure real implementations follow the same pattern.
7. **Make scenarios**: Import as blueprints. Configure school-specific parameters (timezone, fee structure, grading scale).

### Prompt Customization Points

| Variable | Default | Description |
|----------|---------|-------------|
| `{{primary_color}}` | #4F46E5 | Brand primary (indigo) |
| `{{school_name}}` | Greenwood International Academy | Institution name |
| `{{academic_year}}` | 2025-2026 | Current academic year |
| `{{grading_scale}}` | A-F (4.0 GPA) | Grading system (customizable for GPA, percentage, letter) |
| `{{curriculum}}` | Multi-curriculum | IB / Cambridge / National / Custom |
| `{{currency}}` | USD | Fee currency |
| `{{timezone}}` | America/New_York | School timezone |
| `{{api_base_url}}` | `/v1` | API prefix via NestJS gateway |

---

## 6. Output Packaging Convention

```
ERP-School-Design/
  tokens/
    light-theme.json
    dark-theme.json
  components/
    buttons.fig
    inputs.fig
    student-components.fig
    academic-components.fig
    lms-components.fig
    gamification-components.fig
    finance-components.fig
    communication-components.fig
    navigation-admin.fig
    navigation-student.fig
    navigation-parent.fig
    navigation-teacher.fig
    feedback.fig
  pages/
    desktop/
      admin-dashboard.fig
      student-directory.fig
      timetable-calendar.fig
      lms-course-view.fig
      teacher-grade-book.fig
      finance-dashboard.fig
    tablet/
      admin-dashboard.fig
      teacher-grade-book.fig
      classroom-display.fig
    mobile/
      student-dashboard.fig
      parent-dashboard.fig
      lms-lesson.fig
      teacher-attendance.fig
      parent-fee-payment.fig
      parent-bus-tracking.fig
  make-scenarios/
    attendance-workflow.json
    fee-collection.json
    lms-gamification.json
    placement-drive.json
    iot-campus-monitor.json
    academic-early-warning.json
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Lighthouse Performance | >= 95 | Chrome DevTools |
| Lighthouse Accessibility | >= 95 (AAA for student text) | Chrome DevTools |
| First Contentful Paint | < 1.2s | WebPageTest, 4G |
| Largest Contentful Paint | < 2.0s | WebPageTest, 4G |
| Cumulative Layout Shift | < 0.1 | Chrome UX Report |
| Time to Interactive | < 3.0s | WebPageTest |
| LMS video first frame | < 2.0s | Custom benchmark |
| Grade book (500 students) | < 300ms render | Custom benchmark |
| Bus map real-time update | < 1s per position update | WebSocket latency |
| Attendance submission (offline) | < 500ms local save | Device test |
| Initial JS Bundle (gzip) | < 200 KB | Build output |
| API Read p99 | < 200ms | SLO dashboard |
| API Write p99 | < 500ms | SLO dashboard |
| Error rate | < 0.1% | Observability |
| Availability | >= 99.95% | Uptime monitor |

---

## 8. AIDD Handoff Gate Template

```
AIDD Design Handoff Checklist -- ERP-School-Management
=======================================================
[ ] WCAG 2.2 AA verified; AAA for student-facing reading text (16px min body)
[ ] Keyboard navigation documented for all portals (Admin, Teacher, Student, Parent)
[ ] All interactive states: default, hover, active, focus, disabled, loading, error, empty
[ ] Light and dark mode complete (except classroom display -- dark only)
[ ] Desktop (1440px), tablet (1024px), mobile (390px) breakpoints delivered
[ ] Four distinct portal themes verified (Admin=Indigo, Teacher=Teal, Student=Violet, Parent=Emerald)
[ ] Design tokens match exported JSON (including academic colors, grade colors, gamification palette)
[ ] Component naming: {portal}/{category}/{component}/{variant}/{state}
[ ] data-testid attributes documented with portal prefix
[ ] Student data privacy: all IDs masked, no SSN/full ID visible in designs
[ ] AI confidence badges and "advisory only" disclaimers on ALL AI-generated content
[ ] Grade modification requires audit reason field
[ ] Financial confirmation dialogs include full breakdown and receipt preview
[ ] Gamification never penalizes; opt-out of public leaderboard designed
[ ] Parent consent gates designed for sensitive student actions
[ ] Offline capability annotations for attendance and LMS (mobile)
[ ] Bus tracking real-time UX documented (update frequency, stale data handling)
[ ] Multi-language layout verified (RTL for Arabic at minimum)
[ ] Age-appropriate design: student UI uses encouraging language, larger touch targets
[ ] Performance budget annotations (LMS video lazy-load, virtual scroll for grade book)
[ ] Analytics annotations per portal (portal_name prefix on all events)
[ ] API endpoint mapping per service via NestJS gateway
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial creation. 17 Figma prompts (6 desktop, 6 mobile, 3 tablet, 2 foundation), 6 Make prompts covering all 24 ERP-School-Management services across Admin, Teacher, Student, and Parent portals. |
