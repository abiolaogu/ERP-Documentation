# Figma & Make Prompts -- ERP-Projects
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma Make prompts for the **ERP-Projects** module -- a comprehensive Project Management platform. ERP-Projects manages projects, tasks, Kanban and Scrum boards, portfolios, Gantt timelines, budgets, resource allocation, time tracking, and agile workflows. These prompts generate high-fidelity screens for Project Managers, Team Leads, Scrum Masters, Resource Managers, Portfolio Directors, and individual contributors.

**Services covered:** agile-service, board-service, budget-service, portfolio-service, project-service, resource-service, task-service, time-tracking-service, timeline-service

**API surface:** `/v1/project`, `/v1/task`, `/v1/board`, `/v1/portfolio`, `/v1/timeline`, `/v1/budget`, `/v1/resource`, `/v1/time-tracking`, `/v1/agile`

---

## 2. AIDD Guardrails (Apply To All Prompts)

Every prompt in this document must comply with the following cross-cutting guardrails. Append the relevant subset as context when pasting into Figma Make.

### 2.1 User Experience And Accessibility
- WCAG 2.2 AA minimum; target AAA for text contrast on primary content.
- Keyboard navigable: Tab/Shift-Tab for focus traversal; Enter/Space to activate; Escape to dismiss; arrow keys for board card movement and timeline navigation.
- Drag-and-drop interactions must have keyboard-accessible alternatives (move-to menu, arrow keys with modifier).
- Command palette accessible via `Cmd+K` / `Ctrl+K` on every page with quick actions: "Create Task", "Go to Board", "Find Project", "Log Time".
- Role-first layout: adapt views for Project Manager (overview, budgets, resources), Scrum Master (sprint boards, velocity), Team Member (my tasks, time log), Portfolio Director (cross-project rollups).
- Progressive disclosure: project summary first, drill into tasks, subtasks, and time entries on demand.
- Skeleton loading for boards, timelines, and data tables; optimistic UI for task status changes and card moves.
- Form quality: inline validation, autosave for task descriptions, resumable draft tasks.
- Data density toggle: compact / comfortable / expanded on task lists and tables.
- Multi-language readiness: all strings externalizable; RTL-safe layout.
- Touch targets: minimum 44x44px on mobile for drag handles, buttons, and card actions.

### 2.2 Performance And Frontend Efficiency
- Lighthouse Performance score: >= 95.
- Board rendering: < 200 cards visible in viewport with virtual scrolling; 60fps drag-and-drop.
- Gantt timeline: virtual horizontal scroll, lazy-load task bars outside visible date range.
- First Contentful Paint < 1.2s; Largest Contentful Paint < 2.0s on 4G.
- Image assets: WebP/AVIF with `srcset`; avatar images < 10KB each.
- Bundle budget: < 200KB initial JS (gzipped).

### 2.3 Reliability, Trust, And Safety
- Undo support for task moves, status changes, and deletions (5-second toast with "Undo" action).
- Confirmation dialogs for destructive actions (delete project, archive sprint, remove team member from project).
- AIDD confidence badges on AI-generated task estimates, resource recommendations, and risk predictions.
- Real-time collaboration indicators: show avatars of users currently viewing the same board/project.
- Conflict resolution: if two users edit the same task, show diff and merge prompt.

### 2.4 Observability And Testability
- Every interactive component includes `data-testid`: `{page}-{component}-{action}`.
- Analytics event hooks: page_view, card_moved, task_created, sprint_started, time_logged, filter_applied.
- Error states: empty board, no tasks found, timeline load failure, permission denied -- each with distinct illustration and CTA.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a Figma design-token page titled "ERP-Projects / Tokens" at 1440x3000px.

COLOR PALETTE (light mode):
- Primary: #2563EB (Projects Blue)
- Primary Hover: #1D4ED8
- Secondary: #7C3AED (Agile Violet)
- Accent: #0891B2 (Timeline Teal)
- Success: #16A34A
- Warning: #F59E0B
- Danger: #DC2626
- Neutral-50: #F9FAFB through Neutral-900: #111827 (9-step gray scale)
- Surface: #FFFFFF
- Background: #F3F4F6

BOARD-SPECIFIC COLORS (for Kanban columns):
- To Do: #E0E7FF (Indigo-100)
- In Progress: #DBEAFE (Blue-100)
- In Review: #FEF3C7 (Amber-100)
- Done: #D1FAE5 (Green-100)
- Blocked: #FEE2E2 (Red-100)

PRIORITY COLORS:
- Critical: #DC2626 (Red)
- High: #F97316 (Orange)
- Medium: #F59E0B (Amber)
- Low: #6B7280 (Gray)

COLOR PALETTE (dark mode):
- Primary: #60A5FA
- Secondary: #A78BFA
- Accent: #22D3EE
- Surface: #1F2937
- Background: #111827
- Board column backgrounds: 10% opacity versions of light colors

TYPOGRAPHY (Inter font family):
- Display: 36px / 700 / 1.1
- H1: 30px / 700 / 1.2
- H2: 24px / 600 / 1.3
- H3: 20px / 600 / 1.3
- H4: 16px / 600 / 1.4
- Body-lg: 16px / 400 / 1.5
- Body: 14px / 400 / 1.5
- Caption: 12px / 400 / 1.4
- Overline: 11px / 600 / 1.4 / uppercase / 0.05em
- Code: 13px / JetBrains Mono / 400 / 1.5 (for task IDs)

SPACING: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96px
ELEVATION: shadow-sm, shadow-md (card default), shadow-lg (dragging card), shadow-xl (modal)
BORDER RADIUS: 4px (sm), 8px (md, card default), 12px (lg), 16px (xl)
MOTION: 150ms ease-out (micro), 250ms ease-in-out (card move), 350ms ease-in-out (panel slide)

Show each token as a labeled swatch with light/dark side-by-side.
```

#### Prompt F-002: Component Library

```
Create a Figma component library page titled "ERP-Projects / Components" at 1440x6000px.

Include components with light/dark variants and all states (default, hover, active, focus, disabled, dragging where applicable):

BUTTONS: Primary, Secondary, Ghost, Danger, Icon-only. Sizes: sm (28px), md (36px), lg (44px).
INPUTS: Text field, Search bar (Cmd+K hint), Select, Multi-select with tags, Date picker, Date range picker, Time duration input (HH:MM format), Textarea (markdown-enabled), Number with stepper.

TASK CARD (Kanban):
- Default: task ID (monospaced, e.g., "PRJ-142"), title (1-2 lines), priority icon (colored dot), assignee avatar (24px), due date, story points badge, tag chips (max 2 visible + overflow).
- States: default, hover (shadow-lg), dragging (rotated 3deg, shadow-xl, 90% opacity), selected (blue border), blocked (red left border + chain icon).
- Sizes: compact (for list view), comfortable (board default), expanded (with description preview).

TASK ROW (List view): Checkbox | Task ID | Title | Status pill | Priority icon | Assignee avatar | Due date | Story points | Tags | Actions kebab.

BOARD COLUMN: Header with column name, card count, "+" add card button, color-coded top border. Scrollable card area. Column actions menu (rename, set WIP limit, archive).

GANTT BAR: Horizontal bar with task title inside, drag handles on left/right edges for date adjustment, dependency arrows (finish-to-start), milestone diamond marker, progress fill percentage, critical path highlight (red).

SPRINT CARD: Sprint name, date range, story points (committed vs completed pie), status (Planning/Active/Completed), "Start Sprint" / "Complete Sprint" button.

DATA DISPLAY: Data table with sortable headers and pagination. Stat card with sparkline. Badge (status: Open/In Progress/In Review/Done/Blocked/Cancelled). Tag (category-colored, removable). Progress bar. Avatar stack (overlapping, +N indicator). Story point badge (circle). Burndown mini-chart. Velocity bar.

NAVIGATION: Sidebar with collapsible project tree (folders > projects > boards). Top bar with project switcher, search, notification bell, avatar. Breadcrumbs. Tabs (Views: Board | List | Timeline | Calendar).

FEEDBACK: Toast with undo ("Task moved to Done. Undo"), Modal, Confirmation dialog, Empty state ("No tasks yet. Create your first task"), Skeleton loaders for board columns and Gantt rows.

SPECIALIZED: Time entry row (date, project, task, duration, billable toggle, notes), Resource capacity bar (allocated vs available hours), Budget gauge (spent vs approved vs forecast), AI estimate badge with confidence.

Named layers: `{category}/{component}/{variant}/{state}`.
```

---

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Project Dashboard (1440px)

```
Design a desktop dashboard titled "ERP-Projects / Dashboard" at 1440x1024px.

LAYOUT:
- Left sidebar (240px, collapsible to 64px): Logo, nav groups: "My Work" (My Tasks, My Time, My Calendar), "Projects" (tree: "Website Redesign", "Mobile App v2", "Data Migration", "Marketing Q1"), "Portfolios" (Engineering, Marketing, Operations), "Reports" (Velocity, Burndown, Budget, Resources). Active: Dashboard. Bottom: user avatar, "Project Manager" role.
- Top bar (64px): Breadcrumb "Projects > Dashboard", command palette "Cmd+K", notification bell with "8" badge, avatar with dropdown.
- Content area: 24px padding.

CONTENT:
Row 1 -- KPI Cards (4 across):
- "Active Projects": 12 | On Track: 9, At Risk: 2, Behind: 1 | mini donut chart
- "Open Tasks": 347 | Due This Week: 42 | amber highlight
- "Team Utilization": 78% | Target: 80% | horizontal bar
- "Budget Health": $1.24M / $1.5M spent | 82.7% | green gauge

Row 2 -- Two panels:
Left (55%): "My Tasks" list table:
Columns: Priority icon | Task ID | Title | Project | Status pill | Due Date | Story Points.
8 rows of realistic tasks:
- Red dot | PRJ-342 | "Implement OAuth2 PKCE flow" | Website Redesign | In Progress (blue) | Feb 25 | 5
- Orange dot | MOB-118 | "Fix crash on image upload" | Mobile App v2 | In Review (amber) | Feb 24 | 3
- (6 more varied tasks)
Sort by: due date ascending. Quick filter chips: All | To Do | In Progress | In Review.

Right (45%): "Upcoming Deadlines" timeline (vertical):
6 items chronologically: task/milestone name, project name (muted), due date, days remaining badge (red if < 3 days), assignee avatar.

Row 3 -- Two panels:
Left (50%): "Sprint Progress" card for active sprint "Sprint 23 (Feb 10-24)":
- Burndown chart: ideal line (dashed gray), actual line (blue), x-axis: days, y-axis: story points.
- Stats: Committed: 45 pts, Completed: 32 pts, Remaining: 13 pts.
- Progress bar: 71% complete.

Right (50%): "Team Workload" horizontal stacked bar chart:
5 team members: name, avatar, allocated hours (blue) vs available hours (gray) out of 40h/week.
- "Sarah Chen": 36/40h (90%)
- "James Okonkwo": 32/40h (80%)
- "Maria Rodriguez": 42/40h (105% -- red overflow indicator)
- "Alex Kim": 28/40h (70%)
- "Pat Johnson": 38/40h (95%)

Light and dark mode variants.
```

#### Prompt F-004: Kanban Board (1440px)

```
Design a desktop Kanban board titled "ERP-Projects / Board: Website Redesign" at 1440x1024px.

LAYOUT: Standard sidebar/top bar. Active sidebar: "Website Redesign" under Projects. Breadcrumb: "Projects > Website Redesign > Board".

View tabs (below breadcrumb): Board (active, underlined) | List | Timeline | Calendar | Budget.

Board header:
- Project name "Website Redesign" (H2), status badge "On Track" (green).
- Right: Sprint selector dropdown "Sprint 23 (Feb 10-24)", "Filter" button with active filter count, "+ Add Task" primary button, view options (group by, swimlanes toggle).

Filter bar (collapsible): Assignee multi-select (avatar chips), Priority, Labels, Due date range.

BOARD COLUMNS (5, horizontal scroll if needed):
Column 1 -- "To Do" (indigo top border): 8 task cards, WIP limit: none, count badge "8".
Column 2 -- "In Progress" (blue top border): 5 task cards, WIP limit: 6, count "5/6".
Column 3 -- "In Review" (amber top border): 3 task cards, WIP limit: 4, count "3/4".
Column 4 -- "Done" (green top border): 12 task cards (collapsed, show count + "Show completed"), count "12".
Column 5 -- "Blocked" (red top border): 1 task card with blocked indicator, count "1".

TASK CARDS (varied data):
"To Do" column sample cards:
- PRJ-401 | "Design system color audit" | Low | Avatar: SC | Due: Mar 1 | 2 pts | Tags: "design"
- PRJ-399 | "Write E2E tests for checkout" | High | Avatar: JO | Due: Feb 28 | 5 pts | Tags: "testing", "automation"

"In Progress" sample:
- PRJ-387 | "Implement search autocomplete" | Critical | Avatar: AK | Due: Feb 25 | 8 pts | Tags: "frontend" | Subtask progress: 3/5

"Blocked" card:
- PRJ-356 | "Third-party API integration" | High | Avatar: MR | Due: Feb 22 (overdue, red) | 5 pts | Blocked reason tooltip: "Waiting for vendor API keys"

INTERACTION STATES (show as separate detail frames):
1. Card being dragged: rotated 3deg, elevated shadow, ghost placeholder in original position.
2. Drop target column: column background pulses to 5% primary blue.
3. Card detail modal (640px): Opens on card click. Shows: full title, description (markdown rendered), status dropdown, priority selector, assignee dropdown with search, due date picker, story points, labels, subtask checklist (add/check/delete), activity timeline (comments + status changes), time logged section, attachments section, "View in Timeline" link. Sidebar: watchers, related tasks, created/updated dates.

Real-time presence: 3 user avatars in top-right showing who's viewing this board.

Light and dark mode.
```

#### Prompt F-005: Gantt Timeline (1440px)

```
Design a desktop Gantt timeline page titled "ERP-Projects / Timeline: Website Redesign" at 1440x1024px.

LAYOUT: Standard sidebar/top bar. View tab: Timeline (active). Breadcrumb: "Projects > Website Redesign > Timeline".

Timeline header:
- Zoom controls: Day | Week (active) | Month | Quarter.
- Date range navigator: "<" prev | "Feb 10 - Mar 7, 2026" | ">" next | "Today" button.
- Right: "Add Task", "Add Milestone", "Show Critical Path" toggle, "Dependencies" toggle, "Export PDF" ghost button.

LEFT PANEL (320px, resizable):
Task hierarchy table:
- Expandable groups: "Phase 1: Discovery" > "Phase 2: Design" > "Phase 3: Development" > "Phase 4: Testing" > "Phase 5: Launch".
- Columns: Task Name (with indent for subtasks) | Assignee (avatar) | Start | End | Duration | Progress %.

RIGHT PANEL (1096px): Gantt chart area.
- Time axis: weeks across top (Feb 10, Feb 17, Feb 24, Mar 3...), days below.
- Today marker: red vertical dashed line at Feb 23.
- Weekend shading: subtle gray vertical bands on Sat-Sun.

GANTT BARS:
Phase 1 "Discovery" (summary bar, darker shade):
  - "Stakeholder interviews" | Feb 10-14 | 100% | green fill
  - "Competitive analysis" | Feb 12-18 | 80% | blue fill with remaining gray
Phase 2 "Design":
  - "Wireframes" | Feb 17-21 | 60% | blue fill
  - "Visual design" | Feb 19-28 | 20% | blue fill, extends past today line
  - "Design review" (milestone diamond) | Mar 1
Phase 3 "Development":
  - "Frontend scaffold" | Feb 24 - Mar 7 | 0% | gray bar (future)
  - "API integration" | Mar 3 - Mar 14 | 0%
  - (4 more tasks with realistic dates)

DEPENDENCY ARROWS: Finish-to-start arrows between:
- "Wireframes" -> "Visual design"
- "Visual design" -> "Design review" milestone
- "Design review" -> "Frontend scaffold"
Show as curved gray arrows with arrowheads.

CRITICAL PATH: When toggled on, highlight the longest dependency chain in red (#DC2626) with thicker bars.

INTERACTION STATES (detail frames):
1. Hover on bar: tooltip with task name, dates, duration, assignee, progress.
2. Dragging bar edge: date adjustment preview with snapping to days.
3. Right-click context menu: Edit Task, Add Dependency, Add Subtask, Set as Milestone, Delete.

Light and dark mode.
```

#### Prompt F-006: Portfolio Overview (1440px)

```
Design a desktop page titled "ERP-Projects / Portfolio: Engineering" at 1440x1200px.

LAYOUT: Standard sidebar/top bar. Active: "Engineering" under Portfolios. Breadcrumb: "Portfolios > Engineering".

Header: "Engineering Portfolio" (H1), "8 active projects | $4.2M total budget" (subtitle). Right: "Add Project" primary button, date range filter, "Export Report" ghost button.

Row 1 -- Portfolio Health Summary (4 cards):
- "On Track": 5 projects (62.5%) | green icon
- "At Risk": 2 projects (25%) | amber icon
- "Behind Schedule": 1 project (12.5%) | red icon
- "Budget Variance": -$42K under budget | green icon with chart

Row 2 -- Project Status Table (full width):
Columns: Status indicator (colored dot) | Project Name | Project Manager (avatar + name) | Timeline bar (mini Gantt showing start-end + progress) | Budget (spent/total + %) | Tasks (completed/total) | Team Size | Health Score (A-F grade badge) | Actions.

8 rows:
- Green dot | "Website Redesign" | Sarah Chen | [===========----] 72% | $280K/$350K | 89/124 | 6 | A
- Green dot | "Mobile App v2" | James Okonkwo | [========-------] 55% | $180K/$400K | 67/145 | 8 | A
- Amber dot | "Data Platform Migration" | Maria Rodriguez | [=====----------] 35% | $520K/$600K (87%) | 45/210 | 12 | C
- Amber dot | "API Gateway Rebuild" | Alex Kim | [==========-----] 65% | $95K/$100K (95%) | 34/52 | 4 | B
- Red dot | "Legacy System Decommission" | Pat Johnson | [====-----------] 28% | $320K/$250K (128% over!) | 23/98 | 5 | D
- (3 more green projects)

Row 3 -- Two panels:
Left (50%): "Budget Overview" stacked bar chart: 8 projects, each bar shows: spent (blue), committed (lighter blue), remaining (gray). Red line at approved amount. Y-axis: project names, X-axis: $0 - $600K.

Right (50%): "Resource Allocation" heatmap:
Rows: team member names (top 10). Columns: projects. Cell color intensity = hours allocated. Totals column shows over/under capacity.

Row 4 -- "Risk Register" (full width):
Table: Risk ID | Description | Project | Probability (H/M/L) | Impact (H/M/L) | Risk Score (colored) | Mitigation | Owner | Status.
5 rows with realistic project risks.

Light and dark mode.
```

#### Prompt F-007: Time Tracking Page (1440px)

```
Design a desktop page titled "ERP-Projects / Time Tracking" at 1440x1100px.

LAYOUT: Standard sidebar/top bar. Active: My Time. Breadcrumb: "My Work > Time Tracking".

Header: "Time Tracking" (H1), week navigator: "< Week of Feb 17, 2026 >", "This Week" button. Right: "Submit Timesheet" primary button (disabled until all days have entries), "Timer" floating button.

TIMESHEET GRID (full width):
Row headers (left, 280px): Project name > Task name hierarchy.
Column headers: Mon 17 | Tue 18 | Wed 19 | Thu 20 | Fri 21 | Sat 22 | Sun 23 | Total.
Cells: time duration inputs (HH:MM), editable inline. Billable indicator (dollar icon, toggleable).

Sample data (7 rows):
- Website Redesign > "Implement search autocomplete": 2:00 | 3:00 | 1:30 | 2:00 | -- | -- | -- | 8:30
- Website Redesign > "Code review": 0:30 | 0:45 | 1:00 | 0:30 | 0:30 | -- | -- | 3:15
- Mobile App v2 > "Fix crash on image upload": -- | 1:00 | 2:00 | 3:00 | 2:00 | -- | -- | 8:00
- Mobile App v2 > "Sprint planning": 1:00 | -- | -- | -- | -- | -- | -- | 1:00
- Meetings > "Standup": 0:15 | 0:15 | 0:15 | 0:15 | 0:15 | -- | -- | 1:15
- Admin > "Documentation": -- | -- | 0:30 | -- | 1:00 | -- | -- | 1:30
- "+ Add row" link

Totals row: 3:45 | 5:00 | 5:15 | 5:45 | 3:45 | 0:00 | 0:00 | 23:30 / 40:00 target
Daily capacity indicator: green if <= 8h, amber if 8-10h, red if > 10h.

RIGHT SIDEBAR (300px):
"Weekly Summary":
- Total hours: 23:30 / 40:00 (progress bar, 59%)
- Billable hours: 19:30 (83%)
- By project: pie chart (Website Redesign 50%, Mobile App v2 38%, Other 12%)
- Comparison vs last week: "Last week: 38:00 | -14:30" with trend arrow

"Active Timer" widget (if running):
- Task: "PRJ-387: Implement search autocomplete"
- Duration: 01:23:45 (counting up)
- Start time: 14:32
- "Pause" and "Stop & Log" buttons

Below timesheet: "Submit Timesheet" CTA with approval workflow status: Draft | Submitted | Approved | Rejected. Current: "Draft -- 3 days incomplete".

Light and dark mode.
```

#### Prompt F-008: Agile Sprint Board (1440px)

```
Design a desktop page titled "ERP-Projects / Sprint: Sprint 23" at 1440x1100px.

LAYOUT: Standard sidebar/top bar. Breadcrumb: "Projects > Website Redesign > Agile > Sprint 23".

Sprint header card:
- "Sprint 23" (H2), "Feb 10 - Feb 24, 2026" (subtitle), "Active" green badge.
- Sprint goal: "Complete search feature and begin design system audit."
- Stats row: Committed: 45 pts | Completed: 32 pts | Remaining: 13 pts | 2 days left.
- Progress bar: 71% (blue fill).
- Right: "Complete Sprint" primary button, "Sprint Settings" gear icon.

Tabs: Board (active) | Burndown | Velocity | Backlog.

BOARD VIEW (below tabs, same Kanban layout as F-004 but scoped to sprint tasks only):
4 columns: To Do (6 cards) | In Progress (4 cards) | In Review (3 cards) | Done (8 cards).
Each card shows sprint-specific story points prominently.
Swimlanes toggle: by Assignee (showing horizontal lanes per team member).

BURNDOWN TAB (show as alternate frame):
- Full-width burndown chart (600px height).
- X-axis: Sprint days (Feb 10 through Feb 24).
- Y-axis: Story points (0-50).
- Ideal burndown: straight dashed gray line from 45 to 0.
- Actual burndown: blue line -- starts tracking ideal, then plateaus around day 5, catches up around day 8, currently at 13 remaining on day 12.
- Scope changes annotated: vertical dashed line on day 7 "Scope +3 pts added".
- Legend: Ideal | Actual | Scope Changes.

VELOCITY TAB (show as alternate frame):
- Bar chart of last 6 sprints.
- Each sprint: two bars -- Committed (light blue) and Completed (dark blue).
- Sprint 18: 40/38 | Sprint 19: 42/40 | Sprint 20: 45/35 | Sprint 21: 38/37 | Sprint 22: 42/41 | Sprint 23: 45/32 (in progress).
- Average velocity line: 38.2 pts (horizontal dashed line).
- Stats: Avg Velocity: 38.2 | Completion Rate: 91% | Trend: Stable.

Light and dark mode.
```

---

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Project Dashboard (390px)

```
Design a mobile dashboard titled "ERP-Projects / Mobile Dashboard" at 390x844px.

LAYOUT:
- Bottom tab bar (56px): Dashboard (active), My Tasks, Board, Time, More.
- Top bar (56px): "Projects" title, search icon, notification bell "8", avatar.
- Content: scrollable, 16px padding.

CONTENT:
Greeting: "Good morning, Sarah" (H3), "Project Manager" (caption).

KPI scroll (horizontal, 4 cards at 140px wide):
- Active Projects: 12
- Open Tasks: 347
- Team Util: 78%
- Budget: 82.7%

"Sprint 23" progress card (full width):
- Sprint name, date range, "2 days left" badge.
- Progress bar: 71%.
- Stats: 32/45 pts completed.
- "View Board" link.

"My Tasks Due Soon" list (5 items):
Each: Priority dot | Task ID + Title (truncated) | Due date (red if overdue) | Status pill.
- Red | PRJ-342 "Implement OAuth2..." | Today | In Progress
- Orange | MOB-118 "Fix crash on image..." | Tomorrow | In Review
- (3 more)
"View All Tasks" link.

"Team Workload" compact horizontal bars (3 members shown):
- Name + avatar | bar showing allocated/40h.
"View All" link.

Light and dark mode.
```

#### Prompt F-010: Mobile Kanban Board (390px)

```
Design a mobile Kanban board titled "ERP-Projects / Mobile Board" at 390x844px.

LAYOUT: Bottom tab bar with "Board" active. Top bar: "Website Redesign" title, filter icon (right), board settings icon.

CONTENT:
Project selector (dropdown at top): "Website Redesign" with chevron.
Sprint indicator: "Sprint 23 | 2 days left" (compact bar).

Column tabs (horizontal scroll, replacing full columns):
- "To Do (8)" | "In Progress (5)" | "In Review (3)" | "Done (12)" | "Blocked (1)"
Active tab: "In Progress" with blue underline.

Card list (vertical, full width, showing cards from active column):
5 task cards, each:
- Priority dot + Task ID (Caption, monospaced) + kebab menu (right)
- Title (Body, semibold, 2 lines max)
- Row: Assignee avatar (24px) + name | Story points badge | Due date
- Tags (horizontal scroll, compact pills)
- Subtask progress: "3/5 subtasks" with mini progress bar

Long-press a card shows action sheet: View Details | Move to... (column selector) | Edit | Assign to... | Add to Sprint | Delete (red).

Floating action button: "+" for new task.

Task detail page (second frame, 390x844px):
- Top bar: back arrow, Task ID "PRJ-387", status dropdown, kebab.
- Title (H3, editable).
- Status/Priority/Assignee row (tappable chips).
- Description (expandable, markdown rendered).
- Subtasks checklist.
- Activity feed (comments + changes).
- Bottom sticky bar: "Log Time" + "Add Comment" buttons.

Light and dark mode.
```

#### Prompt F-011: Mobile Time Tracking (390px)

```
Design a mobile time tracking page titled "ERP-Projects / Mobile Time" at 390x844px.

LAYOUT: Bottom tab bar with "Time" active. Top bar: "Time Tracking" title, week navigator arrows.

CONTENT:
Week selector: "< Feb 17 - 23 >" centered, "This Week" pill.

Day selector (horizontal scroll): Mon | Tue | Wed | Thu (active, underlined) | Fri | Sat | Sun. Each shows total hours below: "3:45" | "5:00" | "5:15" | "5:45" | "3:45" | "--" | "--".

Weekly summary card:
- "23:30 / 40:00" large text with circular progress (59%).
- "Billable: 19:30 (83%)" caption.

Time entries for selected day (Thursday):
List of entries, each card:
- Project color dot + Project name (Caption)
- Task name (Body, semibold)
- Duration: "2:00" (large, right-aligned) | Billable icon
- Time range: "09:00 - 11:00"

Entries:
- Blue dot | Website Redesign | "Implement search autocomplete" | 2:00 | Billable
- Blue dot | Website Redesign | "Code review" | 0:30 | Billable
- Purple dot | Mobile App v2 | "Fix crash on image upload" | 3:00 | Billable
- Gray dot | Meetings | "Standup" | 0:15 | Non-billable

"+ Log Time" button (full width, primary).

Active timer widget (floating, above tab bar):
- "PRJ-387: Implement search..." truncated
- "01:23:45" counting
- Pause | Stop buttons (icon-only, 44px).

"Log Time" modal (bottom sheet, 390px):
- Project selector (required)
- Task selector (filtered by project)
- Date (pre-filled with selected day)
- Duration input (HH:MM) OR Start/End time toggle
- Billable toggle
- Notes textarea
- "Save" primary button

Light and dark mode.
```

#### Prompt F-012: Mobile Task Detail (390px)

```
Design a mobile page titled "ERP-Projects / Mobile Task Detail" at 390x844px.

LAYOUT: Top bar: back arrow, "PRJ-387" (task ID), share icon, kebab menu. No bottom tab bar.

CONTENT:
Status bar (full width, blue background): "In Progress" with left/right arrows to change status.

Title: "Implement search autocomplete" (H3).

Metadata grid (2 columns):
- Priority: "Critical" (red badge)
- Assignee: Avatar + "Alex Kim"
- Sprint: "Sprint 23"
- Story Points: "8"
- Due Date: "Feb 25, 2026"
- Labels: "frontend", "feature"

Description section (expandable):
"Implement full-text search with autocomplete dropdown for the global search bar. Should support fuzzy matching and show results grouped by type (tasks, projects, people)." (Body text, markdown rendered).

Subtasks section:
Checklist with 5 items:
- [x] "Research Elasticsearch vs Typesense" (strikethrough)
- [x] "Design autocomplete dropdown component" (strikethrough)
- [x] "Implement API endpoint" (strikethrough)
- [ ] "Build frontend autocomplete widget"
- [ ] "Write integration tests"
Progress: "3/5 complete" with bar.
"+ Add Subtask" link.

Time Logged section:
"4:30 total" | "Log Time" button.
3 entries: date, duration, person.

Activity / Comments section (tabs: Activity | Comments):
6 items chronologically:
- "Alex Kim changed status from To Do to In Progress" | Feb 20
- "Sarah Chen commented: Looks good so far, let's add debouncing." | Feb 21
- "Alex Kim added subtask: Write integration tests" | Feb 22
- (3 more)

Comment input (sticky bottom): Avatar + text input + send button.

Light and dark mode.
```

#### Prompt F-013: Mobile Portfolio Summary (390px)

```
Design a mobile page titled "ERP-Projects / Mobile Portfolio" at 390x844px.

LAYOUT: Top bar: back arrow, "Engineering Portfolio" title. Accessed from "More" tab.

CONTENT:
Summary card (full width):
- "8 Active Projects" (H3)
- Health pills: 5 On Track (green), 2 At Risk (amber), 1 Behind (red).
- Total Budget: $4.2M | Spent: $3.1M (74%).

Project cards (vertical list):
Each card:
- Row 1: Status dot + Project name (H4) + Health grade badge (A/B/C/D/F)
- Row 2: PM avatar + name (caption)
- Row 3: Mini progress bar + "72% complete"
- Row 4: Budget: "$280K / $350K" | Team: "6 members"

8 project cards with varied data.

Sort/filter bar: Sort by (Health/Budget/Timeline) | Filter (Status).

Tapping a card navigates to project detail (second frame):
- Project header with status, PM, dates.
- Tab bar: Overview | Tasks | Budget | Team.
- Overview: milestone timeline (vertical), key metrics, recent activity.

Light and dark mode.
```

#### Prompt F-014: Mobile Sprint View (390px)

```
Design a mobile page titled "ERP-Projects / Mobile Sprint" at 390x844px.

LAYOUT: Top bar: back arrow, "Sprint 23" title, sprint actions kebab. Accessed from Board or Dashboard.

CONTENT:
Sprint info card:
- "Sprint 23" (H3), "Feb 10 - 24, 2026", "Active" badge.
- Goal: "Complete search feature and begin design system audit."
- Large stat: "32 / 45 pts" (71%) with circular progress chart.
- "2 days remaining".

Tab bar: Tasks | Burndown | Velocity.

Tasks tab (active):
Group by status headers:
"In Progress (4)":
- Compact task cards (task ID, title, assignee avatar, points).
"To Do (6)":
- Same compact format.
"In Review (3)":
- Same format.
"Done (8)": collapsed, "Show 8 completed items" link.

Burndown tab (second frame):
Full-width burndown chart (300px height), simplified: ideal line, actual line, today marker.
Below chart: "On pace to finish 2 pts behind schedule."

Velocity tab (third frame):
Horizontal bar chart of last 4 sprints (screen-width friendly).
Avg velocity: 38.2 pts.

Bottom sticky bar: "Complete Sprint" primary button (if user is Scrum Master).

Light and dark mode.
```

---

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-015: Tablet Board View (1024px)

```
Design a tablet Kanban board titled "ERP-Projects / Tablet Board" at 1024x768px.

LAYOUT:
- Sidebar: icon-only (64px), expandable overlay to 240px.
- Top bar (56px): project name, search, bell, avatar.
- Content: full remaining width.

BOARD:
4 columns visible (To Do, In Progress, In Review, Done). Blocked column accessible via horizontal scroll.
Cards slightly narrower than desktop (240px vs 280px).
Touch-optimized: larger drag handles (16px grip area), tap-to-open card detail (no hover tooltip).

Sprint bar: compact, showing name + progress bar + points inline.

Card detail: slide-in panel from right (400px) instead of centered modal.

Light and dark mode.
```

#### Prompt F-016: Tablet Timeline (1024px)

```
Design a tablet Gantt timeline titled "ERP-Projects / Tablet Timeline" at 1024x768px.

LAYOUT: Icon-only sidebar (64px), top bar (56px).

LEFT PANEL: 240px (narrower than desktop), showing task names and assignee avatars only.
RIGHT PANEL: Gantt chart area, touch-scrollable horizontally and vertically.

Zoom: default to "Week" view. Pinch-to-zoom support annotation.
Task bars: 28px height (taller for touch). Tap to select, double-tap to edit.
Dependency arrows: simplified (straight lines instead of curves on tablet for clarity).
Today marker: prominent red line.

Bottom toolbar: "Add Task" | "Add Milestone" | "Zoom" segmented control | "Filters".

Light and dark mode.
```

#### Prompt F-017: Tablet Time Tracking (1024px)

```
Design a tablet time tracking page titled "ERP-Projects / Tablet Time" at 1024x768px.

LAYOUT: Icon-only sidebar (64px), top bar (56px).

CONTENT:
Timesheet grid similar to desktop but with 5 visible day columns (Mon-Fri). Weekend columns accessible via scroll.
Left column: 220px for project/task hierarchy.
Cell size: 64px wide, comfortable for touch input.
Weekly summary panel: below grid instead of right sidebar, full width, horizontal layout with stats.

Active timer: floating bar above grid.

Light and dark mode.
```

---

## 4. Make Automation Prompts

#### Prompt M-001: Sprint Lifecycle Automation

```
Create a Make (Integromat) scenario titled "ERP-Projects: Sprint Lifecycle Automation".

TRIGGER: Webhook -- CloudEvent "erp.projects.agile.*".

FLOW:
1. Router by event action:
   a. "sprint.started" (erp.projects.agile.created with type=sprint):
      - HTTP POST to board-service `/v1/board` to create/activate sprint board view.
      - HTTP GET `/v1/task?sprint_id={{sprint_id}}&status=backlog` to fetch committed tasks.
      - For each task: HTTP PUT `/v1/task/{id}` to set status = "To Do".
      - HTTP POST to notification (via ERP-Platform notification-hub): "Sprint {{sprint_name}} started" to all team members.
      - HTTP POST to timeline-service `/v1/timeline` to create sprint timeline markers.

   b. "sprint.completed":
      - HTTP GET `/v1/task?sprint_id={{sprint_id}}&status!=done` to find incomplete tasks.
      - HTTP PUT each incomplete task: set sprint_id = null (moved to backlog).
      - HTTP POST to agile-service `/v1/agile/velocity` to calculate and store sprint velocity.
      - HTTP POST to notification-hub: sprint summary with completed/remaining points.
      - HTTP POST to budget-service `/v1/budget` to log sprint cost (hours * team rates).

   c. "sprint.created" (planning):
      - HTTP POST to resource-service `/v1/resource/capacity` to check team availability for sprint dates.
      - Return capacity report via webhook response.

ERROR HANDLING: Retry 3x with 5s/15s/45s delays. On final failure, alert Scrum Master via notification-hub.
```

#### Prompt M-002: Task Status Change Workflow

```
Create a Make scenario titled "ERP-Projects: Task Status Sync".

TRIGGER: Webhook -- CloudEvent "erp.projects.task.updated" where changed_fields includes "status".

FLOW:
1. Parse event: extract task_id, project_id, old_status, new_status, assignee_id, story_points.

2. Router by new_status:
   a. "In Review":
      - HTTP GET `/v1/task/{task_id}` to fetch task with subtasks.
      - Validate: all subtasks must be complete. If not, HTTP PUT task back to "In Progress" with comment "Cannot move to review: {{incomplete_count}} subtasks remaining."
      - If valid: HTTP POST notification to project reviewers.

   b. "Done":
      - HTTP PUT `/v1/time-tracking` to stop any active timers for this task.
      - HTTP POST to budget-service: log task completion with actual hours vs estimated.
      - HTTP PUT to timeline-service: mark task as completed on Gantt.
      - HTTP POST notification to task creator: "Task {{task_id}} completed by {{assignee_name}}."
      - HTTP GET `/v1/agile/sprint/{sprint_id}/progress` and if all tasks done, notify Scrum Master.

   c. "Blocked":
      - HTTP POST notification to project manager with blocking reason.
      - HTTP POST to timeline-service: flag task as blocked (affects critical path).
      - Create follow-up task if configured: HTTP POST `/v1/task` with "Unblock: {{task_title}}" as title.

3. For all transitions: HTTP POST audit log entry with old_status, new_status, actor, timestamp.

ERROR HANDLING: Idempotent -- duplicate events produce same result. 3x retry with backoff.
```

#### Prompt M-003: Resource Allocation Alert

```
Create a Make scenario titled "ERP-Projects: Resource Overallocation Monitor".

TRIGGER: Scheduled -- every 4 hours during business days (Mon-Fri 08:00-20:00 UTC).

FLOW:
1. HTTP GET `/v1/resource?include_allocation=true` to fetch all team members with current allocations.
2. For each resource:
   a. Calculate total allocated hours across all projects for current week.
   b. Compare against capacity (default: 40h/week, or custom from resource profile).
   c. If allocated > capacity * 1.1 (10% buffer):
      - Flag as "Over-Allocated".
      - HTTP POST notification to resource's manager: "{{name}} is allocated {{allocated_hours}}h / {{capacity}}h this week."
      - HTTP POST notification to resource: "You are over-allocated. Review your task commitments."
3. Aggregate:
   - Build team utilization summary: names, allocated%, project breakdown.
   - HTTP POST to portfolio-service `/v1/portfolio/resource-report` to update portfolio resource health.
4. If any resource > 120% capacity:
   - HTTP POST escalation to portfolio director via notification-hub with severity "warning".

OUTPUT: Store weekly utilization snapshot in data store for trend analysis.
ERROR HANDLING: Skip individual resource on failure, continue processing others. Log errors.
```

#### Prompt M-004: Budget Threshold Alert

```
Create a Make scenario titled "ERP-Projects: Budget Threshold Monitor".

TRIGGER: Webhook -- CloudEvent "erp.projects.budget.updated" OR scheduled daily at 09:00 UTC.

FLOW:
1. HTTP GET `/v1/budget?active=true` to fetch all active project budgets.
2. For each budget:
   a. Calculate burn rate: (total_spent / elapsed_days) * total_project_days.
   b. Forecast completion cost: current_spent + (burn_rate * remaining_days).
   c. Determine thresholds:
      - 75% spent: HTTP POST notification to project manager -- "Budget 75% consumed with {{remaining_timeline}}% timeline remaining."
      - 90% spent: HTTP POST notification to PM + portfolio director -- "Budget 90% consumed. Review scope or request increase."
      - 100% exceeded: HTTP POST notification to PM + portfolio director + finance -- "Budget exceeded by ${{overage}}. Action required."
      - Forecast > approved: HTTP POST notification to PM -- "At current burn rate, projected overage of ${{projected_overage}} by project end."
3. Aggregate across portfolio:
   - HTTP POST to portfolio-service: portfolio-level budget health rollup.
4. Weekly summary (if triggered on Monday):
   - Build budget report across all projects.
   - HTTP POST to notification-hub: send digest to portfolio director.

ERROR HANDLING: Per-project isolation. Failed budget checks do not block others.
```

#### Prompt M-005: Timesheet Approval Workflow

```
Create a Make scenario titled "ERP-Projects: Timesheet Approval Flow".

TRIGGER: Webhook -- CloudEvent "erp.projects.time-tracking.updated" where status = "submitted".

FLOW:
1. Parse event: extract user_id, week_start_date, total_hours, entries[].
2. Validation checks:
   a. Total hours >= minimum (e.g., 32h for full-time): if not, HTTP PUT status = "returned" with comment "Minimum hours not met."
   b. All billable entries have project and task assigned: if not, HTTP PUT status = "returned" with comment "Unassigned billable entries."
   c. No single day > 12h without manager pre-approval.
3. If valid:
   - HTTP GET `/v1/resource/{user_id}/manager` to find approver.
   - HTTP POST notification to approver: "Timesheet for {{user_name}} ({{week}}) pending your approval. Total: {{total_hours}}h."
   - Set 48h SLA timer.
4. On approval (separate webhook for approval event):
   - HTTP PUT timesheet status = "approved".
   - HTTP POST to budget-service: allocate hours to project budgets.
   - HTTP POST notification to submitter: "Your timesheet has been approved."
5. On rejection:
   - HTTP PUT timesheet status = "returned" with rejection comments.
   - HTTP POST notification to submitter: "Timesheet returned. Reason: {{reason}}."
6. SLA expiry (48h no action):
   - HTTP POST escalation to approver's manager.
   - HTTP POST reminder to original approver.

ERROR HANDLING: Idempotent approval processing. Duplicate submissions ignored.
```

#### Prompt M-006: Cross-Project Dependency Sync

```
Create a Make scenario titled "ERP-Projects: Cross-Project Dependency Sync".

TRIGGER: Webhook -- CloudEvent "erp.projects.timeline.updated" where type = "dependency_change" OR "task_date_change".

FLOW:
1. Parse event: extract task_id, project_id, new_start_date, new_end_date, dependencies[].
2. For each dependency:
   a. HTTP GET `/v1/timeline/{dependency.target_task_id}` to fetch dependent task.
   b. If dependent task is in a DIFFERENT project:
      - Check: does the date change create a conflict? (predecessor end > successor start)
      - If conflict: HTTP POST notification to both project managers: "Cross-project dependency conflict: {{task_a}} in {{project_a}} now ends {{end_date}}, but {{task_b}} in {{project_b}} starts {{start_date}}."
      - HTTP PUT `/v1/timeline/{dependency.target_task_id}` to flag conflict status.
3. Cascade check: if date shift propagates (> 3 dependent tasks affected):
   - HTTP POST notification to portfolio director: "Cascade impact detected. {{count}} tasks across {{project_count}} projects affected by date change in {{source_project}}."
   - Generate impact summary with task list and date shifts.
4. HTTP POST audit entry with dependency change details.

ERROR HANDLING: Prevent infinite loops (max cascade depth: 10). Circuit breaker on > 50 notifications/minute.
```

---

## 5. Prompt Usage Guidelines

### How to Use These Prompts

1. **Figma Make**: Open Figma, use "Make a Design" or Figma AI, paste prompt content (without triple backticks).
2. **Iteration sequence**: Generate F-001 (tokens) and F-002 (components) first. Reference them in page prompts.
3. **Board interactions**: For the Kanban board (F-004), generate the base state first, then use follow-up prompts for drag, drop, and modal states.
4. **Gantt chart**: F-005 is complex; generate the left panel and right panel separately if needed, then compose.
5. **Breakpoint flow**: Desktop 1440px -> Tablet 1024px -> Mobile 390px. Each is self-contained.
6. **Make scenarios**: Import as blueprints. Replace webhook URLs, API base URLs, and tokens per environment.

### Prompt Customization Points

| Variable | Default | Description |
|----------|---------|-------------|
| `{{primary_color}}` | #2563EB | Brand primary |
| `{{font_family}}` | Inter | System font |
| `{{sprint_duration}}` | 2 weeks | Default sprint length |
| `{{work_hours_per_week}}` | 40 | Capacity baseline |
| `{{api_base_url}}` | `/v1` | API routing prefix |
| `{{project_name}}` | Website Redesign | Sample project for mockups |

---

## 6. Output Packaging Convention

```
ERP-Projects-Design/
  tokens/
    light-theme.json
    dark-theme.json
  components/
    buttons.fig
    inputs.fig
    task-card.fig
    board-column.fig
    gantt-bar.fig
    sprint-card.fig
    data-display.fig
    navigation.fig
    feedback.fig
    specialized.fig
  pages/
    desktop/
      dashboard.fig
      kanban-board.fig
      gantt-timeline.fig
      portfolio-overview.fig
      time-tracking.fig
      sprint-board.fig
    tablet/
      kanban-board.fig
      gantt-timeline.fig
      time-tracking.fig
    mobile/
      dashboard.fig
      kanban-board.fig
      time-tracking.fig
      task-detail.fig
      portfolio-summary.fig
      sprint-view.fig
  make-scenarios/
    sprint-lifecycle.json
    task-status-sync.json
    resource-overallocation.json
    budget-threshold.json
    timesheet-approval.json
    cross-project-dependency.json
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Lighthouse Performance | >= 95 | Chrome DevTools |
| Lighthouse Accessibility | >= 95 | Chrome DevTools |
| First Contentful Paint | < 1.2s | WebPageTest, 4G |
| Largest Contentful Paint | < 2.0s | WebPageTest, 4G |
| Cumulative Layout Shift | < 0.1 | Chrome UX Report |
| Time to Interactive | < 3.0s | WebPageTest |
| Board render (200 cards) | < 500ms | Custom perf test |
| Gantt render (100 tasks) | < 400ms | Custom perf test |
| Card drag-and-drop FPS | >= 60fps | Chrome DevTools Performance |
| Initial JS Bundle (gzip) | < 200 KB | Build output |
| API Read p99 | < 200ms | SLO dashboard |
| API Write p99 | < 500ms | SLO dashboard |
| Error rate | < 0.1% | Observability |
| Availability | >= 99.95% | Uptime monitor |

---

## 8. AIDD Handoff Gate Template

```
AIDD Design Handoff Checklist -- ERP-Projects
==============================================
[ ] WCAG 2.2 AA contrast verified (Figma A11y plugin)
[ ] Keyboard nav documented: Tab order, arrow-key board navigation, Escape to close
[ ] Drag-and-drop has keyboard alternative (move-to menu documented)
[ ] All interactive states: default, hover, active, focus, disabled, loading, error, empty, dragging
[ ] Light and dark mode complete
[ ] Desktop (1440px), tablet (1024px), mobile (390px) breakpoints delivered
[ ] Design tokens match exported JSON
[ ] Component naming: {category}/{component}/{variant}/{state}
[ ] data-testid attributes documented
[ ] Board column WIP limits visually enforced
[ ] Gantt critical path highlighting verified
[ ] Real-time collaboration indicators designed (avatar presence)
[ ] Undo toast designed for card moves and status changes
[ ] Performance budget annotations (virtual scroll boundaries, lazy-load zones)
[ ] Analytics annotations (card_moved, task_created, sprint_started, time_logged)
[ ] API endpoint mapping documented per data region
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial creation. 17 Figma prompts, 6 Make prompts covering all ERP-Projects services. |

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
Design a control plane view for ERP-Projects that shows:
1. Hasura GraphQL connectivity (latency, success %, schema drift status)
2. ERP-DBaaS health (connection pool, replica lag, failover posture)
3. ERP-IAM integration (OIDC issuer health, token validation errors, session invalidations)
4. ERP-Observability status (OTLP export rate, tracing availability, alert health)

Include: command surface, timeline, runbook links, and one-click diagnostics panel.
Add states for: fully healthy, degraded IAM, degraded DB, degraded GraphQL, global outage.
```

### Prompt F-901: Role-Aware Workspace + Guardrail Surface
```
Generate a role-aware workspace for ERP-Projects with:
- Role lenses: Operator, Manager, Auditor, Admin
- Guardrail panel with mode badge (Protected / Supervised / Autonomous)
- Inline policy rationale for blocked/supervised actions
- Approval request flow for supervised actions

Ensure each role sees distinct navigation and action priorities.
```

### Prompt F-902: Incident + Recovery UX
```
Design a full incident-response flow for ERP-Projects:
- Alert ingestion to triage board
- Impacted tenant view and blast-radius visualization
- Action timeline with runbook steps and rollback controls
- Post-incident report generator modal

Must include: trace-id copy, audit evidence export, and SLA breach indicators.
```

### Prompt F-903: Mobile-Responsive Executive Snapshot
```
Create mobile and tablet executive dashboards for ERP-Projects:
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
Create a Make scenario for ERP-Projects that:
1. Triggers on deployment/webhook events
2. Validates GraphQL, IAM, DB, and OTLP health in sequence
3. Creates incident ticket when thresholds are breached
4. Posts status updates to operations channels
5. Writes audit trail to persistent store

Add branching for: transient failures, policy violations, and hard-stop conditions.
```

### Prompt M-901: Tenant Onboarding Orchestration
```
Generate a Make scenario for tenant onboarding in ERP-Projects:
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
