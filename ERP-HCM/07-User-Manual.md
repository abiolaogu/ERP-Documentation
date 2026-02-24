# ERP-HCM User Manual

## Version 1.0.0 | Last Updated: 2026-02-23

---

## Part 1: Getting Started

### 1.1 Logging In

1. Navigate to your organization's ERP-HCM URL (e.g., `https://hcm.yourcompany.com`)
2. Enter your work email address and password
3. If MFA is enabled, enter the 6-digit code from your authenticator app
4. You will be redirected to the Employee Dashboard

### 1.2 Dashboard Overview

The dashboard is your home screen and shows:
- **Quick stats**: Leave balance, attendance status, pending approvals
- **Announcements**: Company-wide announcements
- **Upcoming**: Birthdays, work anniversaries, holidays
- **Tasks**: Onboarding tasks, pending reviews, document submissions

### 1.3 Navigation

The left sidebar provides access to all modules:
- Dashboard
- Profile
- Attendance
- Leave
- Payslips
- Performance
- Learning
- Recruitment (if authorized)
- Admin (if authorized)

---

## Part 2: Employee Self-Service

### 2.1 My Profile

**Viewing your profile:**
1. Click "Profile" in the sidebar
2. View personal information, employment details, bank details, emergency contacts

**Updating your profile:**
1. Click the "Edit" button on any section
2. Modify the fields (some fields may require HR approval)
3. Click "Save Changes"
4. Changes to sensitive fields (bank details, tax info) require HR admin approval

### 2.2 Attendance

**Clocking In:**
1. Navigate to "Attendance"
2. Click the "Clock In" button
3. If geofencing is enabled, allow location access
4. The system validates your location against your assigned office geofence
5. If within range, your clock-in is recorded with a timestamp

**Clocking Out:**
1. Click the "Clock Out" button on the Attendance page
2. Your total hours for the day are calculated automatically

**Viewing Attendance History:**
1. Select a date range using the calendar picker
2. View daily records showing clock-in/out times, total hours, and status
3. Export attendance report as CSV/Excel

### 2.3 Leave Management

**Checking Leave Balance:**
1. Navigate to "Leave"
2. Your current balances are displayed for each leave type (annual, sick, maternity, etc.)

**Submitting a Leave Request:**
1. Click "Request Leave"
2. Select the leave type
3. Choose start and end dates
4. Select day type (full day, first half, second half, or hourly)
5. Enter the reason for leave
6. Optionally set a delegatee and delegation notes
7. Attach supporting documents if required
8. Click "Submit"
9. Your manager will receive an approval notification

**Tracking Request Status:**
- View all your requests with status: Draft, Pending, Approved, Rejected, Cancelled, Completed
- Received notifications when status changes

### 2.4 Payslips

**Viewing Payslips:**
1. Navigate to "Payslips"
2. Select the payroll period (month/year)
3. View detailed breakdown:
   - **Earnings**: Basic salary, allowances (housing, transport, meal, etc.)
   - **Deductions**: PAYE tax, pension (employee), NHF, loan repayments
   - **Employer contributions**: Pension (employer), NSITF, ITF
   - **Net pay**: Amount deposited to your bank
4. View YTD (Year-to-Date) totals

**Downloading Payslip:**
- Click "Download PDF" to save a copy for your records

### 2.5 Salary Advance

1. Navigate to "Salary Advance"
2. Click "Request Advance"
3. Enter the amount (subject to policy limits)
4. Provide the reason
5. Submit for approval
6. Approved advances are deducted from the next payroll run

### 2.6 Learning

**Browsing Courses:**
1. Navigate to "Learning"
2. Browse the course catalog by category
3. View course details: description, duration, difficulty, instructor, prerequisites

**Enrolling in a Course:**
1. Click on a course
2. Click "Enroll"
3. If approval is required, your manager will be notified
4. Once approved, access course content

**Completing a Course:**
1. Progress through modules/lessons
2. Complete assessments if required
3. On completion, a certificate is generated (if enabled)

### 2.7 Performance

**Viewing Goals (OKRs):**
1. Navigate to "Performance" then "Goals"
2. View your objectives and key results for the current cycle
3. Update progress on key results

**Participating in Reviews:**
1. Navigate to "Performance" then "Reviews"
2. Complete self-assessment when a review cycle is active
3. Provide peer feedback if requested
4. View your review results after manager submission and calibration

---

## Part 3: Manager Guide

### 3.1 Team Dashboard

As a manager, your dashboard includes additional widgets:
- **Team Overview**: Direct reports with their status
- **Pending Approvals**: Leave requests, expense claims, time-off
- **Team Attendance**: Today's attendance for your team
- **Performance**: Team OKR progress

### 3.2 Approving Leave Requests

1. Click on "Pending Approvals" or navigate to admin leave section
2. Review the leave request details (type, dates, reason, attachment)
3. Check the employee's leave balance
4. Click "Approve" or "Reject" with optional comments
5. The employee is notified of your decision

### 3.3 Managing Team Performance

**Setting Team Objectives:**
1. Navigate to "Performance" then "Goals"
2. Create objectives for your team aligned with company OKRs
3. Assign key results to team members
4. Set targets and deadlines

**Conducting Reviews:**
1. Navigate to "Performance" then "Reviews"
2. Select the active review cycle
3. Complete manager assessment for each direct report
4. Provide ratings and written feedback
5. Submit for calibration (if required)

### 3.4 Recruitment

**Creating a Job Requisition:**
1. Navigate to "Recruitment"
2. Click "New Requisition"
3. Fill in position details, requirements, salary range, budget
4. Submit for approval
5. Once approved, the position is published

**Managing Candidates:**
1. View applications for your open positions
2. Review resumes and assessment scores
3. Schedule interviews
4. Provide feedback and recommendations
5. Move candidates through pipeline stages

---

## Part 4: HR Admin Guide

### 4.1 Employee Management

**Adding a New Employee:**
1. Navigate to admin employee section
2. Click "Add Employee"
3. Enter required fields: name, email, employee number, department, position, hire date, employment type
4. Enter optional fields: bank details, tax info, pension info
5. Click "Save"
6. The onboarding workflow is triggered automatically

**Bulk Import:**
1. Download the CSV/Excel template
2. Fill in employee data
3. Upload the file
4. Review validation results
5. Confirm import

**Managing Departments:**
1. Navigate to "Departments"
2. Create, edit, or deactivate departments
3. Set department hierarchy (parent-child relationships)
4. Assign department heads

### 4.2 Leave Administration

**Configuring Leave Policies:**
1. Navigate to "Admin" then "Leave" then "Policies"
2. Create leave types with rules:
   - Annual entitlement (days)
   - Carry-over policy
   - Accrual method
   - Eligibility criteria
   - Required documentation

**Managing Holidays:**
1. Navigate to "Admin" then "Leave" then "Holidays"
2. Add public holidays for the year
3. Assign holidays to locations/countries

### 4.3 Document Management

1. Navigate to "Admin" then "Documents"
2. Upload company policies and handbooks
3. Request employee signatures via digital signature workflow
4. Track document acknowledgments
5. Set document expiry dates for compliance tracking

---

## Part 5: Payroll Admin Guide

### 5.1 Salary Structures

**Setting Up Salary Structures:**
1. Navigate to "Admin" then "Payroll" then "Salary Structures"
2. Configure salary components:
   - Basic salary (percentage of gross)
   - Housing allowance
   - Transport allowance
   - Meal allowance
   - Other allowances
3. Set taxable/pensionable flags for each component

### 5.2 Running Payroll

**Step 1: Create Payroll Period**
1. Navigate to "Admin" then "Payroll"
2. Click "Create Period"
3. Set year, month, start date, end date, pay date

**Step 2: Initiate Payroll Run**
1. Select the period
2. Click "Initiate Run"
3. Choose run type (regular, off-cycle, bonus, etc.)
4. Select employees (all or specific subset)

**Step 3: Process Payroll**
1. Click "Process"
2. The engine calculates for each employee:
   - Gross salary based on salary structure
   - Statutory deductions (PAYE, pension, NHF)
   - Other deductions (loans, advances, cooperative)
   - Allowances and overtime
   - Net pay
3. Review validation results

**Step 4: Review and Approve**
1. Navigate to "Admin" then "Payroll" then "Review"
2. Review the payroll summary:
   - Total gross, net, deductions
   - Department-wise breakdown
   - Variance from previous period
   - Validation warnings/errors
3. Submit for approval
4. Each approval level reviews and approves

**Step 5: Generate Payslips and Disburse**
1. Click "Generate Payslips"
2. Click "Generate Bank File" for NIBSS-format file
3. Upload bank file to banking portal or initiate direct payment via Flutterwave/Remita
4. Mark as "Paid" once disbursement is confirmed

### 5.3 Reports

Navigate to "Admin" then "Payroll" then "Reports" for:
- Monthly payroll summary
- Statutory remittance report (PAYE, pension, NHF)
- Bank payment file
- Payroll variance report
- YTD report per employee
- Department cost analysis

---

## Part 6: System Administration

### 6.1 User Management

- Create user accounts with appropriate roles
- Assign tenant-scoped permissions
- Enable/disable MFA requirements
- Manage concurrent session limits

### 6.2 Configuration

Key configuration areas:
- Organization settings (name, logo, fiscal year)
- Payroll configuration (statutory rates, pay grades)
- Leave policies (types, accrual rules, holidays)
- Attendance settings (geofence locations, shift templates)
- Notification preferences (email, in-app, SMS)
- Integration settings (payment gateways, third-party systems)

### 6.3 Audit Trail

1. Navigate to admin audit section
2. Search audit logs by:
   - Date range
   - User
   - Action type (create, update, delete)
   - Entity type (employee, payroll, leave)
3. Export audit logs for compliance evidence

---

## Appendix A: Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl+K` | Global search |
| `Ctrl+N` | New item (context-dependent) |
| `Esc` | Close dialog/modal |

## Appendix B: Support

- **Help Center**: Available via the `?` icon in the top navigation
- **Support Email**: support@yourcompany.com
- **Knowledge Base**: Internal wiki documentation
