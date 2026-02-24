# ERP-HCM Use Cases

## 20+ Use Cases Across All HR Domains

---

## UC-01: New Employee Onboarding

**Actor**: HR Admin
**Precondition**: Offer accepted by candidate
**Trigger**: Offer acceptance event from recruitment service

**Main Flow**:
1. HR admin creates employee record with personal, employment, bank, and tax details
2. System auto-generates employee number (e.g., EMP-2026-0042)
3. Onboarding engine creates task checklist (document submission, orientation, IT setup)
4. System sends welcome email with login credentials
5. Employee completes profile setup and submits required documents
6. HR admin verifies documents and confirms onboarding completion
7. Employee status set to "Active"
8. Event `erp.hcm.employee.created` published to NATS

**Postcondition**: Employee record active, payroll record initialized

---

## UC-02: Monthly Payroll Processing (Nigeria)

**Actor**: Payroll Admin
**Precondition**: Payroll period created, employee salary structures configured
**Trigger**: Monthly payroll deadline (T-5 before pay date)

**Main Flow**:
1. Payroll admin creates payroll period (year, month, dates)
2. Initiates regular payroll run for all active employees
3. Engine processes each employee:
   a. Computes gross from salary structure components
   b. Calculates CRA: Higher of (NGN 200,000 or 1% gross) + 20% gross
   c. Computes pension: Employee 8% + Employer 10% of (basic + housing + transport)
   d. Computes NHF: 2.5% of basic salary
   e. Determines taxable income: Gross - CRA - Pension - NHF
   f. Applies PAYE graduated bands to taxable income
   g. Computes net pay: Gross - PAYE - Pension (employee) - NHF - other deductions
4. Reviews payroll summary with variance from previous month
5. Submits for multi-level approval
6. Generates payslips and bank file
7. Initiates disbursement via Flutterwave/Remita
8. Marks payroll as "Paid"

**Alternative Flow**:
- 3a. Validation fails for employee (missing bank details) -- flagged, excluded from run
- 5a. Approver rejects -- payroll returned for correction

---

## UC-03: Leave Request and Approval

**Actor**: Employee, Manager
**Precondition**: Leave types configured, employee has available balance

**Main Flow**:
1. Employee navigates to Leave page and clicks "Request Leave"
2. Selects leave type (annual), start/end dates, day type (full day)
3. Enters reason and optionally sets delegatee
4. Submits request (status: Pending)
5. Manager receives notification
6. Manager reviews request, checks team calendar and employee balance
7. Manager approves with optional comment
8. Employee notified of approval
9. Attendance system updated for leave days
10. Leave balance decremented

**Alternative Flow**:
- 7a. Manager rejects with reason -- employee notified
- 2a. Employee selects half-day option -- only 0.5 days deducted

---

## UC-04: Geofenced Clock-In

**Actor**: Employee
**Precondition**: Employee assigned to office location with geofence configured

**Main Flow**:
1. Employee opens Attendance page on mobile/web
2. Clicks "Clock In"
3. Browser/app requests GPS coordinates
4. System captures latitude, longitude, accuracy, device ID
5. Anti-spoofing checks:
   a. GPS accuracy < 50 meters? Pass
   b. Distance from last clock-in location plausible? (no teleport > 100km/hr) Pass
   c. Same device as registered? Pass
6. Geofence validation: Haversine distance from office < allowed radius
7. Clock-in recorded with timestamp and location
8. Employee sees confirmation with arrival time

**Alternative Flow**:
- 6a. Employee outside geofence -- clock-in rejected with distance shown
- 5a. GPS accuracy > 50m -- "Location accuracy too low, move to open area"
- 5b. Teleport detected -- flagged for review, clock-in blocked

---

## UC-05: Job Requisition and Hiring

**Actor**: Hiring Manager, HR Admin
**Precondition**: Department has approved headcount budget

**Main Flow**:
1. Hiring manager creates job requisition with title, department, grade, salary range, requirements
2. Submits for HR approval
3. HR admin reviews and approves
4. Position published to careers page
5. Applications received and stored in ATS
6. HR screens resumes (AI parsing extracts skills)
7. Qualified candidates moved to "Interview" stage
8. Interview scheduled with hiring panel
9. Assessments conducted and scores recorded
10. Top candidate selected for offer
11. Offer created with compensation package, approval obtained
12. Offer sent to candidate
13. Candidate accepts -- triggers onboarding (UC-01)

---

## UC-06: Performance Review Cycle (360-Degree)

**Actor**: HR Admin, Manager, Employee, Peers
**Precondition**: Review cycle configured

**Main Flow**:
1. HR admin creates annual review cycle with components: self-assessment, manager review, peer feedback
2. Review cycle status set to "In Progress"
3. Employees complete self-assessment
4. Peers nominated and complete anonymous feedback
5. Managers complete assessment with ratings and written feedback
6. Review enters "Calibration" phase
7. HR leadership uses 9-box grid to calibrate ratings
8. Final ratings communicated to employees
9. Employees acknowledge or appeal within deadline
10. Review cycle archived

---

## UC-07: Salary Advance Request

**Actor**: Employee
**Precondition**: Employee is active, salary advance policy configured

**Main Flow**:
1. Employee navigates to Salary Advance page
2. Enters requested amount (within policy limits, e.g., max 50% of net salary)
3. Provides reason for advance
4. Submits for approval
5. Manager/HR approves
6. Amount disbursed or added to next pay cycle
7. Deduction automatically applied in next payroll run

---

## UC-08: Benefits Open Enrollment

**Actor**: Employee, HR Admin
**Precondition**: Benefits plans configured, enrollment period open

**Main Flow**:
1. HR admin opens enrollment period with start/end dates
2. Employees notified of open enrollment
3. Employee navigates to Benefits page
4. Reviews available plans (health, dental, life, pension options)
5. Selects desired plans with coverage levels
6. Reviews premium costs and employer contributions
7. Confirms enrollment
8. Enrollment recorded and effective from next period
9. Payroll deductions updated automatically

---

## UC-09: OKR Cycle Management

**Actor**: HR Admin, Manager, Employee
**Precondition**: OKR cycle created for the quarter

**Main Flow**:
1. HR creates quarterly OKR cycle with deadlines
2. Leadership sets company-level objectives
3. Managers cascade to department objectives
4. Employees create individual objectives aligned to department goals
5. Key results defined with measurable targets
6. Progress updated weekly/bi-weekly
7. Mid-cycle check-in meetings conducted
8. End of cycle: final scoring
9. Results aggregated for performance analytics

---

## UC-10: Employee Termination

**Actor**: HR Admin, Manager
**Precondition**: Termination decision made (voluntary or involuntary)

**Main Flow**:
1. Manager initiates termination request with reason and last working day
2. HR admin reviews and processes
3. System triggers offboarding checklist:
   a. IT asset return
   b. Access revocation
   c. Final payroll calculation (pro-rated salary, unused leave payout)
   d. Benefits termination
   e. Exit interview scheduling
4. Employee status changed to "Terminated" or "Resigned"
5. Event `erp.hcm.employee.terminated` published
6. Final payslip generated and disbursed
7. Employee record retained per data retention policy

---

## UC-11: Shift Schedule Management

**Actor**: Shift Supervisor, HR Admin
**Precondition**: Shift templates defined

**Main Flow**:
1. Supervisor creates weekly shift schedule
2. Assigns employees to shifts (morning, afternoon, night)
3. Employees notified of their schedule
4. Shift swaps can be requested between employees (requires approval)
5. Attendance tracked against scheduled shift times
6. Overtime calculated for hours beyond shift duration

---

## UC-12: SCORM Course Completion

**Actor**: Employee, LMS Admin
**Precondition**: SCORM course package uploaded and published

**Main Flow**:
1. Employee browses course catalog in Learning section
2. Enrolls in a SCORM course
3. SCORM player launches in browser
4. Employee progresses through content modules
5. Assessment scores captured via SCORM API
6. On completion, certificate auto-generated
7. Completion recorded for compliance tracking

---

## UC-13: Expense Reimbursement

**Actor**: Employee, Manager, Payroll Admin
**Precondition**: Expense policy configured

**Main Flow**:
1. Employee submits expense claim with receipts
2. Enters amount, category, date, description
3. Uploads receipt images
4. Manager reviews and approves
5. Approved expense added to next payroll run as a one-time allowance
6. Reimbursement included in next salary payment

---

## UC-14: Workforce Planning Scenario

**Actor**: HR Director, Finance Manager
**Precondition**: Current headcount data available

**Main Flow**:
1. HR Director creates a headcount plan for the fiscal year
2. Sets planned headcount by department and location
3. Creates budget scenarios (conservative, moderate, aggressive)
4. Models impact on payroll costs using current salary structures
5. Compares scenarios with visualizations
6. Selects preferred scenario and submits for executive approval
7. Approved plan guides recruitment and compensation decisions

---

## UC-15: Digital Document Signing

**Actor**: HR Admin, Employee
**Precondition**: Document template created

**Main Flow**:
1. HR admin creates a policy document (e.g., code of conduct)
2. Assigns document for signing to selected employees
3. Employees notified of pending signature
4. Employee opens document, reviews content
5. Applies digital signature
6. Signed document stored in DMS with audit trail
7. Compliance report shows signing completion rates

---

## UC-16: Background Check Processing

**Actor**: HR Admin
**Precondition**: Candidate selected for hire

**Main Flow**:
1. HR admin initiates background check for candidate
2. System submits request to background check provider
3. Provider conducts checks (criminal, education, employment history)
4. Results returned and stored in candidate record
5. HR reviews results and makes hiring decision
6. If passed, proceed to offer; if flagged, HR investigates

---

## UC-17: Immigration and Visa Tracking

**Actor**: HR Admin
**Precondition**: Employee is an expatriate

**Main Flow**:
1. HR admin records visa details for employee (type, number, issue/expiry dates)
2. System tracks visa expiry and sends reminders at 90, 60, 30 days
3. HR initiates renewal process
4. Updated visa details recorded
5. Compliance report shows visa status for all expatriate employees

---

## UC-18: Compensation Cycle Execution

**Actor**: HR Admin, Manager, Executive
**Precondition**: Compensation cycle created

**Main Flow**:
1. HR sets up compensation cycle with budget and guidelines
2. Budget allocated by department
3. Managers submit salary increase proposals for their direct reports
4. Proposals calibrated across departments
5. Executive approves final allocations
6. Salary changes communicated to employees
7. Payroll updated with new salary structures effective from next period

---

## UC-19: Employee Engagement Survey

**Actor**: HR Admin, Employee
**Precondition**: Survey builder configured

**Main Flow**:
1. HR admin creates engagement survey using the survey builder
2. Configures questions, rating scales, and anonymous responses
3. Distributes survey to target employees
4. Employees complete the survey
5. Results aggregated with demographic breakdowns
6. HR analyzes engagement scores and identifies action areas
7. Action plans created and tracked

---

## UC-20: Facilities Room Booking

**Actor**: Employee
**Precondition**: Rooms/desks configured in facilities management

**Main Flow**:
1. Employee navigates to Facilities section
2. Views available meeting rooms for desired date/time
3. Selects room and books time slot
4. Receives calendar invitation confirmation
5. Room displays as booked on the facilities dashboard
6. Check-in at room confirms booking usage

---

## UC-21: Contractor Management

**Actor**: HR Admin
**Precondition**: Contractor engagement approved

**Main Flow**:
1. HR creates contractor record with contract details (start/end, rate, deliverables)
2. Contractor assigned to project and cost center
3. Contractor submits timesheets/invoices
4. Manager approves timesheet
5. Payment processed through contractor payment service
6. Contract expiry tracked with renewal reminders

---

## UC-22: Multi-Tenant Data Isolation Verification

**Actor**: IT Admin
**Precondition**: Multiple tenants configured

**Main Flow**:
1. IT admin accesses system with super-admin credentials
2. Queries employee data for Tenant A
3. Switches to Tenant B via X-Tenant-ID header
4. Verifies no data leakage between tenants
5. Audit log confirms all cross-tenant access was supervised and logged
6. AIDD guardrails report shows no prohibited access attempts
