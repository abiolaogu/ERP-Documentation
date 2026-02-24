# ERP-HCM Figma Design Prompts

## 25+ Figma/UI Design Prompts for ERP-HCM

---

## Prompt 1: Employee Dashboard

Design a modern enterprise HR dashboard with the following elements: a top bar showing the employee name, profile avatar, notification bell with badge count, and a search bar. The main content area should have four stat cards (leave balance, attendance streak, pending approvals, upcoming reviews) in a row. Below that, a two-column layout with "Recent Activity" feed on the left (showing events like "Leave approved", "Payslip available", "Review cycle started") and "Upcoming Events" on the right (birthdays, anniversaries, holidays). Use a professional color palette with teal/blue as the primary brand color. Include a left sidebar with navigation icons for Dashboard, Profile, Attendance, Leave, Payslips, Performance, Learning, Recruitment. The design should follow the Radix UI component library aesthetic with Tailwind CSS spacing conventions.

---

## Prompt 2: Org Chart Visualization

Design an interactive organizational chart that shows a hierarchical tree structure of the company. Each node should display the employee's profile picture (circular), full name, job title, department, and a subtle status indicator (green dot for active, yellow for on leave). The root node should be the CEO/MD at the top. Include expand/collapse controls on nodes with reports. Add a search bar at the top to find and highlight specific employees. Include a minimap in the bottom-right corner for navigation in large org charts. Support zoom in/out controls and drag-to-pan. Show a side panel that opens when clicking a node, displaying full employee details.

---

## Prompt 3: Payslip Detail View

Design a payslip view resembling a professional pay statement document. Header should show company logo, payslip title, period (e.g., "February 2026"), and employee details (name, employee number, department, position, bank account). The body should be divided into three sections: Earnings (basic salary, housing allowance, transport allowance, meal allowance, overtime -- each with amount), Deductions (PAYE tax, pension employee, NHF, loan repayment -- each with amount), and Employer Contributions (pension employer, NSITF, ITF). Show running totals for each section. At the bottom, display "Net Pay" prominently in a highlighted box. Include a YTD summary section. Add a "Download PDF" button in the top-right corner. Use a clean, print-friendly layout with alternating row shading.

---

## Prompt 4: Leave Calendar View

Design a full-page calendar view for leave management. Show a monthly calendar grid with color-coded blocks for different leave types (annual leave = blue, sick leave = red, maternity = purple, public holiday = green). Include a sidebar on the right listing upcoming leave requests with status badges (Pending, Approved, Rejected). Add a floating action button "Request Leave" that opens a modal. The modal should have a leave type dropdown, date range picker, day type selector (full day / half day), reason textarea, delegatee selector, and file upload for attachments. Below the calendar, show a horizontal bar chart of leave balance utilization by type.

---

## Prompt 5: Recruitment Kanban Board

Design a kanban-style recruitment pipeline board with columns: Applied, Screening, Interview, Assessment, Offer, Hired. Each column should have a header with the count of candidates. Candidate cards should show: name, current position/company, applied date, and a circular progress indicator for match score. Cards should be draggable between columns. Include a top bar with filters (department, job title, date range, source). Add a "New Requisition" button. When clicking a card, show a detailed side panel with candidate resume, interview notes, assessment scores, and action buttons (Advance, Reject, Schedule Interview).

---

## Prompt 6: Performance Review Form

Design a multi-section performance review form. Section 1: "Self-Assessment" with a text area for achievements and challenges. Section 2: "Goals Review" showing a table of OKR objectives with progress bars, target vs actual values, and a rating dropdown (1-5 stars). Section 3: "Competencies" with a radar/spider chart showing competency scores and individual rating sliders for each competency (Communication, Leadership, Technical, Teamwork). Section 4: "Manager Feedback" with a text area and overall rating. Section 5: "Development Plan" with add/remove action items. Include a progress indicator at the top showing completion percentage. Add "Save Draft" and "Submit" buttons.

---

## Prompt 7: Attendance Tracker

Design a time and attendance page with a prominent clock-in/out button at the top center, showing current status (Clocked In / Not Clocked In) with duration timer. Below that, show today's shift details (scheduled start/end, actual clock-in time). Include a weekly view showing each day with clock-in/out times, total hours, and status (On Time, Late, Absent, Leave). Add a monthly summary card showing total days present, total hours, overtime hours, and late count. Include a small map widget showing the geofence boundary circle and the employee's clock-in location pin.

---

## Prompt 8: Benefits Enrollment Wizard

Design a multi-step wizard for benefits enrollment. Step 1: "Plan Selection" showing benefit plan cards (Health Insurance, Dental, Life Insurance, Gym) with coverage details, monthly premium, and employer contribution. Step 2: "Coverage Level" with radio buttons for Individual, Employee+Spouse, Family. Step 3: "Dependents" with a form to add dependents (name, relationship, date of birth, ID). Step 4: "Review & Confirm" showing a summary table of all selections with total monthly premium. Each step should have a progress bar at the top. Include "Previous" and "Next" buttons at the bottom.

---

## Prompt 9: LMS Course Page

Design a learning course detail page. Header with course thumbnail (16:9 image), course title, instructor name with avatar, difficulty badge (Beginner/Intermediate/Advanced), estimated duration, and enrollment count. Below, a tabbed interface with: Overview (description, learning objectives, prerequisites), Curriculum (expandable module list with lesson names, duration, and completion checkmarks), Reviews (star ratings, review text), and Certificate (preview of completion certificate). Add a prominent "Enroll Now" button that becomes a progress bar after enrollment. Include a sidebar showing related courses.

---

## Prompt 10: Admin Console

Design a system administration dashboard. Top row: four metric cards (Total Employees, Active Users Today, Pending Approvals, System Health). Below, a two-column layout. Left column: "Recent Audit Log" showing a table with columns (Timestamp, User, Action, Entity, Details) with colored action badges. Right column: "System Status" showing service health indicators (green/yellow/red dots) for each microservice (Employee, Payroll, Leave, Recruitment, etc.). Below that, a "Quick Actions" grid with icon-labeled tiles: User Management, Role Configuration, Payroll Settings, Leave Policies, Integration Settings, Security Settings.

---

## Prompt 11: Payroll Run Dashboard

Design a payroll processing dashboard showing the current payroll run status. A horizontal stepper at the top showing: Create Period, Initiate Run, Process, Review, Approve, Disburse, Close -- with the current step highlighted. Below, a summary panel with total employees, total gross, total deductions, total net pay, and currency. Include a department-wise breakdown table and a donut chart of deduction composition (PAYE, Pension, NHF, Other). Show a comparison bar chart (this month vs last month). Add "Approve" and "Reject" buttons at the bottom for authorized users.

---

## Prompt 12: Employee Profile Page

Design a comprehensive employee profile page. A hero section with a large cover image, circular profile photo, employee name, title, department, and status badge. Below, a horizontal tab bar: Personal, Employment, Compensation, Documents, Timeline. Personal tab shows editable fields in a two-column form. Employment tab shows hire date, employment type, department, position, manager, and probation status. Timeline tab shows a vertical timeline of all employment events (hired, promoted, department change, leave, review completed).

---

## Prompt 13: Shift Calendar

Design a weekly shift calendar view showing employee schedules. Rows represent employees (with avatars and names), columns represent days of the week. Cells show shift assignments as colored blocks (Morning = light blue, Afternoon = orange, Night = dark purple). Include a toolbar with week navigation (previous/next), a "Create Shift" button, and filters (department, team). Add a drag-to-assign interaction pattern. Show total hours per employee in a right-side summary column.

---

## Prompt 14: Compensation Cycle Review

Design a compensation review page for managers. A budget summary card at the top showing total budget, allocated, remaining, and percentage used (with a progress bar). Below, a data table listing direct reports with columns: Employee Name, Current Salary, Proposed Increase (editable number input), New Salary, Compa-Ratio, Performance Rating, Last Increase Date. Include inline validation (exceeds budget = red highlight). Add a "Submit Proposals" button and a "Download Template" link for offline review.

---

## Prompt 15: Workforce Planning Scenario Modeler

Design a workforce planning tool with a split-view layout. Left panel: scenario parameters with sliders for growth rate, attrition rate, and budget multiplier. Input fields for planned hires by department. Right panel: real-time results showing projected headcount chart (line graph over 12 months), projected payroll cost (bar chart), and a table comparing Planned vs Current headcount by department with variance indicators (green/red arrows).

---

## Prompt 16: Engagement Survey Builder

Design a survey creation interface with a drag-and-drop question builder. Left sidebar showing question types (Multiple Choice, Rating Scale, Open Text, NPS, Matrix). Main area showing the survey with ordered questions, each with edit/delete/reorder controls. Preview mode toggle in the top-right. Question settings panel on the right for each selected question (required toggle, anonymous toggle, branching logic).

---

## Prompt 17: AI Chat Assistant

Design an HR chatbot interface in a slide-out panel. Show a chat window with message bubbles (user on right, bot on left). Suggested question chips at the bottom: "What is my leave balance?", "When is next payday?", "Show my payslip", "Request leave". The bot should show rich cards for data responses (e.g., leave balance card with types and days remaining).

---

## Prompt 18: Document Signing Flow

Design a digital document signing page. Show the document content in a scrollable viewer with a signing area at the bottom. The signing area should have a "Click to Sign" button that opens a signature pad (draw or type name). Show a timeline of signing activity on the right (who signed, who is pending). Include "Download", "Print", and "Share" action buttons.

---

## Prompt 19: Notification Center

Design a notification center as a dropdown panel from the bell icon. Group notifications by category (HR, Payroll, Leave, Performance) with icons. Each notification shows: icon, title, description, time ago, and read/unread indicator. Include tabs for All, Unread, and a "Mark All as Read" link. Show a settings gear icon linking to notification preferences.

---

## Prompt 20: 9-Box Talent Grid

Design an interactive 9-box grid for talent assessment. The X-axis represents Performance (Low, Medium, High), the Y-axis represents Potential (Low, Medium, High). Each cell should show a count badge and list of employee names/avatars that can be clicked. Color-code cells from red (low/low) to green (high/high). Include a sidebar showing details of the selected cell's employees with options to view profile or add development plans.

---

## Prompt 21: Expense Claim Form

Design an expense reimbursement form with fields: expense date, category dropdown (Travel, Meals, Office Supplies, Training), amount with currency selector, description, and receipt upload (drag-and-drop zone with thumbnail preview). Show a running total at the bottom. Include an "Add Another Expense" button for multi-line claims. Add a submission summary with approval workflow preview.

---

## Prompt 22: Immigration Tracker Dashboard

Design a dashboard for tracking employee work visas and permits. Show a table with columns: Employee Name, Visa Type, Country, Issue Date, Expiry Date, Days Remaining, Status (Active/Expiring/Expired with color badges). Include filters by country and status. Add alert cards at the top showing "Expiring in 30 Days" and "Expired" counts.

---

## Prompt 23: Mobile Clock-In Screen

Design a mobile-first clock-in screen for Flutter. Large circular "Clock In" button in the center that pulses when active. Show current time prominently, today's shift schedule, and a small map showing the employee's current location relative to the geofence boundary. After clock-in, transform to show elapsed time counter with a "Clock Out" button. Include a tab for "My Attendance" showing this week's records.

---

## Prompt 24: Contractor Management

Design a contractor management page with a list/grid toggle view. Each contractor card shows: name, company, contract type, start/end dates, daily/hourly rate, and status (Active, Expiring, Expired). Include a search bar and filters. A detail view should show contract terms, deliverables checklist, timesheet history, and payment records.

---

## Prompt 25: Compliance Dashboard

Design a compliance management dashboard. Top row: compliance score (circular gauge, e.g., 94/100), pending actions count, upcoming deadlines count, policy acknowledgment rate. Below, a risk heatmap by department. Include a "Data Subject Requests" section showing GDPR/NDPR requests with status. Add a document compliance tracker showing policy documents with signed/pending/overdue counts.

---

## Prompt 26: Payroll Reports Page

Design a payroll reporting page with a report selector dropdown (Monthly Summary, Statutory Remittance, Variance Analysis, YTD Report, Department Cost). Show the selected report with interactive charts and data tables. Include date range pickers, export buttons (PDF, Excel, CSV), and a print button. Add comparison toggles (vs previous month, vs same month last year).

---

## Prompt 27: Room/Desk Booking

Design a facilities booking interface with a floor plan view showing room/desk layouts. Available spaces shown in green, booked in red, selected in blue. Include a time slot picker on the side. Show room details (capacity, amenities, AV equipment) in a popup when hovering. Add a calendar view alternative showing bookings by day/week.
