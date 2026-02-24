# ERP-HCM Video Training Scripts

## 8 Video Training Modules

---

## Video 1: Employee Onboarding (12 minutes)

### Title: "Your First Day with ERP-HCM: Employee Onboarding Guide"

**[INTRO - 0:00-0:30]**

NARRATOR: "Welcome to ERP-HCM, your organization's Human Capital Management platform. This video will walk you through the complete onboarding experience, from your first login to completing your profile setup."

**[SCENE 1 - Login and Dashboard - 0:30-2:00]**

NARRATOR: "When you receive your welcome email, click the link to set your password. You can enable multi-factor authentication for additional security by scanning the QR code with your authenticator app."

ON SCREEN: Show login page, password setup, MFA enrollment.

NARRATOR: "After logging in, you arrive at your dashboard. This is your central hub showing your quick stats, pending tasks, company announcements, and upcoming events like team birthdays and holidays."

ON SCREEN: Dashboard walkthrough with callouts on each widget.

**[SCENE 2 - Profile Setup - 2:00-5:00]**

NARRATOR: "Your first onboarding task is completing your profile. Click on Profile in the sidebar. Let us walk through each section."

ON SCREEN: Navigate to Profile page.

NARRATOR: "Start with personal information: your preferred name, phone number, date of birth, and address. Next, review your employment details, which HR has pre-filled: your employee number, department, position, and hire date."

ON SCREEN: Fill in personal details with sample data.

NARRATOR: "The bank details section is where you enter your account information for salary payments. This data is encrypted using AES-256 encryption and only visible to authorized payroll administrators."

ON SCREEN: Bank details form with encryption indicator.

NARRATOR: "Add your emergency contacts -- this is required for workplace safety compliance. Finally, upload your profile picture."

**[SCENE 3 - Onboarding Tasks - 5:00-8:00]**

NARRATOR: "Back on your dashboard, you will see your onboarding checklist. These are tasks assigned by HR that you need to complete within your first week."

ON SCREEN: Onboarding task list with checkboxes.

NARRATOR: "Common tasks include: submitting identification documents, signing the employee handbook, completing mandatory compliance training, and setting up your attendance preferences."

ON SCREEN: Document upload workflow, policy signing page.

**[SCENE 4 - Key Features Tour - 8:00-11:00]**

NARRATOR: "Let us quickly tour the key features you will use daily. The Attendance page lets you clock in and out. The Leave page shows your balances and lets you request time off. The Payslips page gives you access to your monthly pay breakdown."

ON SCREEN: Quick navigation through Attendance, Leave, Payslips pages.

NARRATOR: "The Performance section shows your goals and any active review cycles. The Learning page gives you access to training courses. And the Communication hub keeps you connected with your team."

**[SCENE 5 - Wrap Up - 11:00-12:00]**

NARRATOR: "Congratulations on completing your onboarding setup. If you have questions, use the help icon in the top navigation or reach out to your HR administrator. Welcome to the team."

---

## Video 2: Payroll Processing (15 minutes)

### Title: "Mastering Payroll: End-to-End Monthly Processing"

**[INTRO - 0:00-0:30]**

NARRATOR: "This video covers the complete monthly payroll process in ERP-HCM, from period creation through disbursement. This is designed for payroll administrators."

**[SCENE 1 - Payroll Overview - 0:30-2:00]**

NARRATOR: "ERP-HCM's payroll engine supports multi-country processing with first-class Nigerian statutory compliance. The system calculates PAYE tax using PITA graduated bands, pension contributions per the Pension Reform Act, NHF, NHIS, and other statutory deductions."

ON SCREEN: Payroll architecture diagram showing calculation flow.

**[SCENE 2 - Period Creation - 2:00-4:00]**

NARRATOR: "Step one: Create a payroll period. Navigate to Admin, Payroll, and click Create Period. Set the year, month, start date, end date, and pay date. The system auto-calculates working days."

ON SCREEN: Period creation form with date pickers.

**[SCENE 3 - Initiating a Run - 4:00-6:00]**

NARRATOR: "Click Initiate Run. Select the run type -- regular for your monthly payroll. You can process all employees or select a specific subset. The system supports seven run types: regular, off-cycle, bonus, correction, reversal, thirteenth month, and back pay."

ON SCREEN: Run initiation dialog with type selection.

**[SCENE 4 - Processing - 6:00-9:00]**

NARRATOR: "Click Process. The engine now calculates each employee's payroll entry. For each employee, it computes gross salary from the salary structure, applies all allowances, calculates statutory deductions using decimal-precision arithmetic, and determines the net pay."

ON SCREEN: Processing progress bar, then calculation breakdown for one employee.

NARRATOR: "The Nigerian PAYE calculation works as follows: First, the Consolidated Relief Allowance or CRA is computed as the higher of two hundred thousand Naira or one percent of annual gross, plus twenty percent of annual gross. Then pension and NHF are subtracted to get the taxable income. Finally, PAYE is applied using the graduated tax bands."

ON SCREEN: Animated PAYE calculation with actual numbers.

**[SCENE 5 - Review and Approval - 9:00-12:00]**

NARRATOR: "After processing, review the payroll summary. Check the totals for gross, net, and deductions. The variance report highlights differences from last month, including new hires, exits, and salary changes."

ON SCREEN: Payroll summary dashboard with department breakdown.

NARRATOR: "Submit for approval. Your organization may have multiple approval levels. Each approver reviews and approves or rejects with comments."

**[SCENE 6 - Disbursement - 12:00-14:00]**

NARRATOR: "Once all approvals are received, generate payslips for all employees. Then generate the bank file in NIBSS format for bulk transfer, or initiate payment directly through Flutterwave or Remita integration."

ON SCREEN: Payslip generation, bank file download, payment initiation.

**[SCENE 7 - Reports - 14:00-15:00]**

NARRATOR: "Finally, generate your statutory reports: PAYE remittance to FIRS, pension schedule to PFAs, and NHF remittance. These reports are available in the Payroll Reports section."

---

## Video 3: Leave Management (10 minutes)

### Title: "Leave Management: From Request to Return"

**[INTRO - 0:00-0:30]**

NARRATOR: "This video covers how to submit leave requests, how managers approve them, and how administrators configure leave policies."

**[SCENE 1 - Employee View - 0:30-3:30]**

ON SCREEN: Employee navigates to Leave page, views balances, submits a leave request with dates, reason, and delegatee. Shows half-day option.

**[SCENE 2 - Manager Approval - 3:30-6:00]**

ON SCREEN: Manager views pending approval, reviews request details and employee's leave balance, approves with comment. Shows rejection workflow.

**[SCENE 3 - Admin Configuration - 6:00-9:00]**

ON SCREEN: HR admin configures leave types, sets accrual rules, manages holiday calendar, generates leave utilization report.

**[SCENE 4 - Integration - 9:00-10:00]**

NARRATOR: "When leave is approved, the attendance system is automatically updated, and the payroll engine accounts for leave days during processing."

---

## Video 4: Recruitment Pipeline (12 minutes)

### Title: "Hiring Top Talent: Using the Recruitment Module"

**[INTRO - 0:00-0:30]**

NARRATOR: "This video covers the full recruitment workflow from requisition to hire."

**[SCENE 1 - Creating a Requisition - 0:30-3:00]**

ON SCREEN: HR admin creates a job requisition with title, department, grade, salary range, employment type, work arrangement, requirements, and budget code. Submits for approval.

**[SCENE 2 - Managing Applications - 3:00-6:00]**

ON SCREEN: Viewing applications, candidate profiles, resume details, moving candidates through pipeline stages (Applied, Screening, Interview, Assessment, Offer).

**[SCENE 3 - Interview and Assessment - 6:00-9:00]**

ON SCREEN: Scheduling interviews, recording feedback, viewing assessment scores, comparing candidates.

**[SCENE 4 - Offer and Onboarding - 9:00-12:00]**

ON SCREEN: Creating an offer with compensation details, approval workflow, sending offer to candidate, tracking acceptance, triggering onboarding workflow that creates an employee record.

---

## Video 5: Performance Reviews (12 minutes)

### Title: "Driving Performance: OKRs, Reviews, and Feedback"

**[INTRO - 0:00-0:30]**

NARRATOR: "This video covers the performance management lifecycle including OKR setting, reviews, and 360-degree feedback."

**[SCENE 1 - OKR Cycles - 0:30-4:00]**

ON SCREEN: Creating an OKR cycle, setting company objectives, cascading to departments and individuals, tracking key result progress with percentage updates.

**[SCENE 2 - Review Cycles - 4:00-8:00]**

ON SCREEN: HR creates a review cycle selecting annual type with self-assessment, manager assessment, and peer feedback. Employees complete self-assessment. Managers complete ratings and written feedback.

**[SCENE 3 - 360 Feedback and Calibration - 8:00-11:00]**

ON SCREEN: Peer nomination, anonymous feedback submission, calibration session with 9-box grid placing employees by performance vs potential.

**[SCENE 4 - Communication - 11:00-12:00]**

ON SCREEN: Final review shared with employee, acknowledgment workflow, appeal process overview.

---

## Video 6: Time and Attendance (10 minutes)

### Title: "Tracking Time: Geofenced Attendance Made Simple"

**[INTRO - 0:00-0:30]**

NARRATOR: "This video covers the time and attendance system including geofenced clock-in, shift management, and attendance reporting."

**[SCENE 1 - Clock In/Out - 0:30-3:30]**

ON SCREEN: Employee opens the attendance page, clicks Clock In, browser requests location permission, system validates against geofence showing distance from office and whether within radius. Shows successful and failed clock-in scenarios.

**[SCENE 2 - Anti-Spoofing - 3:30-5:30]**

ON SCREEN: Explanation of GPS accuracy check (50m threshold), teleport detection (100km/hr), multi-device detection. Shows admin view of flagged attendance records.

**[SCENE 3 - Shift Management - 5:30-8:00]**

ON SCREEN: Admin creates shift templates (morning, afternoon, night), assigns shifts to employees, views shift calendar.

**[SCENE 4 - Reports - 8:00-10:00]**

ON SCREEN: Attendance summary report by department, late arrivals, absences, overtime hours, export to CSV.

---

## Video 7: Reports and Analytics (10 minutes)

### Title: "Data-Driven HR: Reports and Analytics Dashboard"

**[INTRO - 0:00-0:30]**

NARRATOR: "This video covers the reporting and analytics capabilities across all HR modules."

**[SCENE 1 - Dashboard Analytics - 0:30-3:00]**

ON SCREEN: Executive dashboard with headcount trends, attrition rate, diversity breakdown, department cost analysis using Recharts visualizations.

**[SCENE 2 - Payroll Reports - 3:00-5:00]**

ON SCREEN: Monthly payroll summary, statutory remittance, variance analysis, YTD reports. Drill-down from summary to individual employee level.

**[SCENE 3 - Workforce Analytics - 5:00-7:00]**

ON SCREEN: Headcount planning vs actual, turnover analysis by department, salary benchmarking, compensation distribution.

**[SCENE 4 - Custom Reports - 7:00-9:00]**

ON SCREEN: Building a custom report by selecting data fields, applying filters, choosing visualization, scheduling automated delivery.

**[SCENE 5 - Export and Integration - 9:00-10:00]**

ON SCREEN: Exporting reports to Excel/PDF, sharing via email, API access for BI tool integration.

---

## Video 8: System Administration (12 minutes)

### Title: "Admin Mastery: Configuring and Securing ERP-HCM"

**[INTRO - 0:00-0:30]**

NARRATOR: "This video is for IT administrators and covers system configuration, security settings, and operational management."

**[SCENE 1 - Organization Setup - 0:30-3:00]**

ON SCREEN: Configuring organization name, logo, fiscal year, default currency, timezone, work week. Setting up departments, locations, and cost centers.

**[SCENE 2 - User and Access Management - 3:00-6:00]**

ON SCREEN: Creating user accounts, assigning roles (super admin, tenant admin, HR admin, payroll admin, manager, employee), configuring MFA requirements, setting session limits and password policies.

**[SCENE 3 - Security Configuration - 6:00-9:00]**

ON SCREEN: Configuring CORS allowed origins, setting rate limits, viewing security dashboard, reviewing audit logs, configuring AIDD guardrails.

**[SCENE 4 - Integration Setup - 9:00-11:00]**

ON SCREEN: Configuring payment gateway credentials (Flutterwave, Remita), setting up webhook endpoints, testing NATS event publishing, API key management.

**[SCENE 5 - Operational Tasks - 11:00-12:00]**

ON SCREEN: Health check monitoring, reviewing system logs, backup status, performance metrics overview.

---

## Production Notes

### Video Specifications
- Resolution: 1920x1080 (Full HD)
- Format: MP4 (H.264)
- Audio: Stereo, professional narration
- Captions: English, French, Arabic (SRT files)
- Hosting: Internal LMS or YouTube (unlisted)

### Screen Recording Tools
- OBS Studio for screen capture
- Figma for animated diagrams
- After Effects for transitions and callouts
