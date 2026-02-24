# ERP-HCM Workflow Diagrams

---

## 1. Hire-to-Retire Lifecycle

```mermaid
flowchart TB
    WP[Workforce Planning] --> REQ[Requisition Created]
    REQ --> APP[Job Posted & Applications Received]
    APP --> SCR[Screening & Shortlisting]
    SCR --> INT[Interviews & Assessments]
    INT --> OFF[Offer Generation]
    OFF --> ACC{Offer Accepted?}
    ACC -->|No| DECLINE[Candidate Declines]
    DECLINE --> APP
    ACC -->|Yes| ONB[Onboarding]
    ONB --> ACTIVE[Active Employment]

    ACTIVE --> LV[Leave Management]
    ACTIVE --> ATT[Attendance Tracking]
    ACTIVE --> PAY[Payroll Processing]
    ACTIVE --> PERF[Performance Reviews]
    ACTIVE --> LRN[Learning & Development]
    ACTIVE --> BEN[Benefits Administration]
    ACTIVE --> COMP[Compensation Reviews]

    PERF --> CD{Career Decision}
    CD -->|Promote| PROMO[Promotion]
    CD -->|Transfer| TRANS[Transfer]
    CD -->|Retain| ACTIVE
    PROMO --> ACTIVE
    TRANS --> ACTIVE

    ACTIVE --> EXIT{Exit Type}
    EXIT -->|Resignation| RES[Resignation Processing]
    EXIT -->|Termination| TERM[Termination Processing]
    EXIT -->|Retirement| RET[Retirement Processing]

    RES --> OFFB[Offboarding]
    TERM --> OFFB
    RET --> OFFB

    OFFB --> FINAL[Final Settlement]
    FINAL --> ARCHIVE[Record Archived]
```

---

## 2. Payroll Run Workflow

```mermaid
flowchart TB
    START([Start Monthly Payroll]) --> CREATE_PERIOD[Create Payroll Period<br/>Year, Month, Start/End/Pay Dates]
    CREATE_PERIOD --> INIT_RUN[Initiate Payroll Run<br/>Select Type: Regular/Bonus/etc]
    INIT_RUN --> SELECT_EMP[Select Employees<br/>All or Subset]

    SELECT_EMP --> PROCESS[Process Payroll]

    subgraph "Calculation Engine"
        PROCESS --> FETCH[Fetch Employee Data<br/>Salary Structure, Allowances]
        FETCH --> GROSS[Calculate Gross Salary<br/>Basic + Allowances]
        GROSS --> CRA[Calculate CRA<br/>Higher of 200K or 1% + 20%]
        CRA --> PENSION[Calculate Pension<br/>Employee 8% + Employer 10%]
        PENSION --> NHF[Calculate NHF<br/>2.5% of Basic]
        NHF --> TAXABLE[Determine Taxable Income<br/>Gross - CRA - Pension - NHF]
        TAXABLE --> PAYE[Apply PAYE Tax Bands<br/>7%, 11%, 15%, 19%, 21%, 24%]
        PAYE --> DEDUCTIONS[Apply Other Deductions<br/>Loans, Advances, Cooperative]
        DEDUCTIONS --> NET[Calculate Net Pay]
        NET --> YTD[Update YTD Values]
    end

    YTD --> VALIDATE{Validation<br/>Passed?}
    VALIDATE -->|No| FIX[Fix Errors<br/>Missing Data, Anomalies]
    FIX --> PROCESS
    VALIDATE -->|Yes| REVIEW[Review Payroll Summary<br/>Totals, Variance, By Department]

    REVIEW --> SUBMIT[Submit for Approval]
    SUBMIT --> APPROVE{Approval<br/>Level N}
    APPROVE -->|Rejected| FIX2[Return for Correction]
    FIX2 --> PROCESS
    APPROVE -->|Approved| CHECK_LEVEL{More Levels?}
    CHECK_LEVEL -->|Yes| APPROVE
    CHECK_LEVEL -->|No| GEN[Generate Outputs]

    subgraph "Outputs"
        GEN --> PAYSLIP[Generate Payslips PDF]
        GEN --> BANK[Generate Bank File NIBSS]
        GEN --> STAT[Generate Statutory Reports]
    end

    PAYSLIP --> DISBURSE[Disburse Payments<br/>Flutterwave/Remita/Bank]
    BANK --> DISBURSE
    DISBURSE --> CONFIRM{Payment<br/>Confirmed?}
    CONFIRM -->|Yes| CLOSE[Close Payroll Period]
    CONFIRM -->|No| RETRY[Retry Payment]
    RETRY --> DISBURSE
    CLOSE --> PUBLISH[Publish Event<br/>erp.hcm.payroll.paid]
    PUBLISH --> END([End])
```

---

## 3. Leave Approval Workflow

```mermaid
flowchart TB
    START([Employee Initiates]) --> CHECK_BALANCE{Balance<br/>Sufficient?}
    CHECK_BALANCE -->|No| REJECT_BALANCE[Inform: Insufficient Balance]
    CHECK_BALANCE -->|Yes| SUBMIT[Submit Leave Request]

    SUBMIT --> STATUS_PENDING[Status: PENDING]
    STATUS_PENDING --> NOTIFY_MGR[Notify Manager]
    NOTIFY_MGR --> MGR_REVIEW[Manager Reviews Request]

    MGR_REVIEW --> MGR_CHECK{Check:<br/>Team Coverage?}
    MGR_CHECK --> MGR_DECISION{Decision}
    MGR_DECISION -->|Approve| APPROVED[Status: APPROVED]
    MGR_DECISION -->|Reject| REJECTED[Status: REJECTED]

    APPROVED --> NOTIFY_EMP_OK[Notify Employee: Approved]
    APPROVED --> UPDATE_ATT[Update Attendance Calendar]
    APPROVED --> UPDATE_BAL[Deduct from Leave Balance]
    APPROVED --> NOTIFY_DELEGATE[Notify Delegatee]

    REJECTED --> NOTIFY_EMP_REJ[Notify Employee: Rejected<br/>with Reason]

    APPROVED --> LEAVE_START{Leave<br/>Starts?}
    LEAVE_START -->|Yes| ON_LEAVE[Employee on Leave]
    ON_LEAVE --> LEAVE_END{Leave<br/>Ends?}
    LEAVE_END -->|Yes| COMPLETED[Status: COMPLETED]
    COMPLETED --> RESTORE_STATUS[Restore Employee Status: Active]

    subgraph "Cancellation"
        APPROVED --> CANCEL{Employee<br/>Cancels?}
        CANCEL -->|Yes| CANCELLED[Status: CANCELLED]
        CANCELLED --> RESTORE_BAL[Restore Leave Balance]
    end
```

---

## 4. Recruitment Pipeline

```mermaid
flowchart TB
    START([Hiring Need]) --> REQ[Create Job Requisition]
    REQ --> REQ_APPROVAL{HR Approval}
    REQ_APPROVAL -->|Rejected| REQ_REVISE[Revise Requisition]
    REQ_REVISE --> REQ
    REQ_APPROVAL -->|Approved| PUBLISH[Publish Job Posting]

    PUBLISH --> RECEIVE[Receive Applications]
    RECEIVE --> PARSE[AI Resume Parsing<br/>Skill Extraction]

    PARSE --> SCREEN[Screening Stage]
    SCREEN --> SCREEN_FILTER{Meets<br/>Requirements?}
    SCREEN_FILTER -->|No| SCREEN_REJECT[Reject - Not Qualified]
    SCREEN_FILTER -->|Yes| SHORTLIST[Shortlist]

    SHORTLIST --> INTERVIEW[Interview Stage]
    INTERVIEW --> SCHEDULE[Schedule Interview<br/>Panel Assignment]
    SCHEDULE --> CONDUCT[Conduct Interview<br/>Record Feedback]
    CONDUCT --> INTERVIEW_PASS{Passed?}
    INTERVIEW_PASS -->|No| INT_REJECT[Reject - Interview]
    INTERVIEW_PASS -->|Yes| ASSESS[Assessment Stage]

    ASSESS --> TEST[Technical/Aptitude Test]
    TEST --> SCORE[Score Assessment]
    SCORE --> ASSESS_PASS{Passed?}
    ASSESS_PASS -->|No| ASSESS_REJECT[Reject - Assessment]
    ASSESS_PASS -->|Yes| BGC[Background Check]

    BGC --> BGC_RESULT{Clear?}
    BGC_RESULT -->|No| BGC_REJECT[Reject - Background]
    BGC_RESULT -->|Yes| OFFER_STAGE[Offer Stage]

    OFFER_STAGE --> PREPARE_OFFER[Prepare Offer<br/>Salary, Benefits, Start Date]
    PREPARE_OFFER --> OFFER_APPROVAL{Approval Chain}
    OFFER_APPROVAL -->|Approved| SEND_OFFER[Send Offer to Candidate]
    SEND_OFFER --> CANDIDATE_DECISION{Candidate Decision}
    CANDIDATE_DECISION -->|Decline| DECLINE_OFFER[Offer Declined]
    DECLINE_OFFER --> OFFER_STAGE
    CANDIDATE_DECISION -->|Accept| ACCEPT_OFFER[Offer Accepted]

    ACCEPT_OFFER --> CREATE_EMP[Create Employee Record]
    CREATE_EMP --> ONBOARD[Trigger Onboarding Workflow]
    ONBOARD --> HIRED[Requisition: FILLED]

    subgraph "Talent Pool"
        SCREEN_REJECT --> POOL{Add to<br/>Talent Pool?}
        INT_REJECT --> POOL
        DECLINE_OFFER --> POOL
        POOL -->|Yes| TALENT_DB[(Talent Pool DB)]
    end
```

---

## 5. Performance Review Cycle

```mermaid
flowchart TB
    START([HR Creates Cycle]) --> CONFIG[Configure Review Cycle<br/>Type, Components, Dates]
    CONFIG --> SCHEDULE[Status: SCHEDULED]

    SCHEDULE --> ACTIVATE{Start Date<br/>Reached?}
    ACTIVATE -->|Yes| IN_PROGRESS[Status: IN PROGRESS]

    IN_PROGRESS --> PARALLEL_START{Parallel Activities}

    PARALLEL_START --> SELF[Self Assessment<br/>Employee rates self]
    PARALLEL_START --> PEER_NOM[Peer Nomination<br/>Select reviewers]
    PARALLEL_START --> GOALS[Goals Review<br/>OKR progress]

    PEER_NOM --> PEER_FB[Peer Feedback<br/>Anonymous reviews]

    SELF --> MGR_READY{All Self<br/>Assessments In?}
    PEER_FB --> MGR_READY
    GOALS --> MGR_READY

    MGR_READY -->|Yes| MGR_REVIEW[Manager Assessment<br/>Ratings + Feedback]
    MGR_REVIEW --> UPWARD[Upward Feedback<br/>Manager by reports]

    UPWARD --> CALIBRATION{Calibration<br/>Required?}
    CALIBRATION -->|Yes| CAL_SESSION[Calibration Session]

    subgraph "Calibration"
        CAL_SESSION --> NINE_BOX[9-Box Grid Placement]
        NINE_BOX --> ADJUST[Adjust Ratings<br/>Cross-Team Fairness]
        ADJUST --> CAL_APPROVE[Calibration Approved]
    end

    CALIBRATION -->|No| FINALIZE[Finalize Reviews]
    CAL_APPROVE --> FINALIZE

    FINALIZE --> COMMUNICATE[Communicate Results<br/>to Employees]
    COMMUNICATE --> ACK{Employee<br/>Acknowledges?}
    ACK -->|Yes| COMPLETE[Status: COMPLETED]
    ACK -->|Appeal| APPEAL[Appeal Process<br/>Within Deadline]
    APPEAL --> APPEAL_REVIEW[Appeal Reviewed]
    APPEAL_REVIEW --> COMPLETE

    COMPLETE --> ARCHIVE[Status: ARCHIVED]
```

---

## 6. Benefits Enrollment Workflow

```mermaid
flowchart TB
    START([Enrollment Trigger]) --> TYPE{Trigger Type}
    TYPE -->|Open Enrollment| OE[Open Enrollment Period<br/>Annual Window]
    TYPE -->|Life Event| LE[Life Event<br/>Marriage, Birth, etc]
    TYPE -->|New Hire| NH[New Hire Enrollment<br/>30-Day Window]

    OE --> NOTIFY[Notify Eligible Employees]
    LE --> NOTIFY
    NH --> NOTIFY

    NOTIFY --> BROWSE[Employee Browses Plans<br/>Health, Dental, Life, Pension]
    BROWSE --> COMPARE[Compare Plans<br/>Coverage, Cost, Network]
    COMPARE --> SELECT[Select Plans & Coverage Level]
    SELECT --> DEPENDENTS[Add/Update Dependents]
    DEPENDENTS --> REVIEW[Review Selections<br/>Premium Summary]

    REVIEW --> CONFIRM{Confirm?}
    CONFIRM -->|No| SELECT
    CONFIRM -->|Yes| SUBMIT[Submit Enrollment]

    SUBMIT --> APPROVAL{Requires<br/>Approval?}
    APPROVAL -->|Yes| HR_REVIEW[HR Reviews]
    HR_REVIEW --> HR_DECISION{Approved?}
    HR_DECISION -->|No| REJECT[Reject with Reason]
    HR_DECISION -->|Yes| ENROLL[Enrollment Confirmed]
    APPROVAL -->|No| ENROLL

    ENROLL --> UPDATE_PAYROLL[Update Payroll Deductions]
    ENROLL --> NOTIFY_PROVIDER[Notify Benefits Provider/HMO]
    ENROLL --> ISSUE_CARD[Issue Benefits Card/ID]
    ENROLL --> CONFIRM_EMP[Notify Employee: Enrolled]

    UPDATE_PAYROLL --> EFFECTIVE[Effective from Next Period]

    subgraph "Claims"
        EFFECTIVE --> CLAIM[Employee Submits Claim]
        CLAIM --> CLAIM_REVIEW[Claims Processing]
        CLAIM_REVIEW --> REIMBURSE[Reimbursement]
    end
```

---

## 7. Onboarding Checklist Workflow

```mermaid
flowchart TB
    TRIGGER([Employee Created]) --> CREATE_CHECKLIST[Generate Onboarding Checklist]
    CREATE_CHECKLIST --> TASKS{Task Categories}

    TASKS --> HR_TASKS[HR Tasks]
    TASKS --> EMP_TASKS[Employee Tasks]
    TASKS --> IT_TASKS[IT Tasks]
    TASKS --> MGR_TASKS[Manager Tasks]

    subgraph "HR Tasks"
        HR_TASKS --> OFFER_LETTER[Send Offer Letter]
        OFFER_LETTER --> CONTRACT[Employment Contract]
        CONTRACT --> HANDBOOK[Employee Handbook]
        HANDBOOK --> POLICY_SIGN[Policy Signing]
    end

    subgraph "Employee Tasks"
        EMP_TASKS --> PROFILE_COMPLETE[Complete Profile]
        PROFILE_COMPLETE --> DOCS_SUBMIT[Submit ID Documents]
        DOCS_SUBMIT --> BANK_SETUP[Bank Details Setup]
        BANK_SETUP --> TAX_FORM[Tax Information]
        TAX_FORM --> PENSION_FORM[Pension Registration]
    end

    subgraph "IT Tasks"
        IT_TASKS --> CREATE_ACCOUNTS[Create Email/System Accounts]
        CREATE_ACCOUNTS --> ASSIGN_EQUIP[Assign Equipment]
        ASSIGN_EQUIP --> ACCESS_CARD[Issue Access Card]
    end

    subgraph "Manager Tasks"
        MGR_TASKS --> INTRO_TEAM[Team Introduction]
        INTRO_TEAM --> SET_GOALS[Set Initial Goals/OKRs]
        SET_GOALS --> ASSIGN_BUDDY[Assign Onboarding Buddy]
    end

    POLICY_SIGN --> CHECK{All Tasks<br/>Complete?}
    PENSION_FORM --> CHECK
    ACCESS_CARD --> CHECK
    ASSIGN_BUDDY --> CHECK

    CHECK -->|No| REMINDER[Send Reminder]
    REMINDER --> CHECK
    CHECK -->|Yes| ONBOARD_COMPLETE[Onboarding Complete]
    ONBOARD_COMPLETE --> PROBATION[Start Probation Period]
```

---

## 8. Attendance and Shift Workflow

```mermaid
flowchart TB
    START([Shift Day Begins]) --> CHECK_SHIFT[Check Employee Shift Assignment]
    CHECK_SHIFT --> CLOCK_IN[Employee Attempts Clock-In]

    CLOCK_IN --> GPS[Capture GPS Coordinates]
    GPS --> ANTI_SPOOF{Anti-Spoofing Checks}

    ANTI_SPOOF --> GPS_ACC{GPS Accuracy<br/>< 50m?}
    GPS_ACC -->|No| REJECT_ACC[Reject: Low Accuracy]
    GPS_ACC -->|Yes| TELEPORT{Teleport<br/>Check OK?}
    TELEPORT -->|No| FLAG_TELEPORT[Flag: Teleport Detected]
    TELEPORT -->|Yes| DEVICE{Same Device<br/>as Registered?}
    DEVICE -->|No| FLAG_DEVICE[Flag: Multi-Device]
    DEVICE -->|Yes| GEOFENCE{Within<br/>Geofence?}

    GEOFENCE -->|No| REJECT_GEO[Reject: Outside Geofence<br/>Show Distance]
    GEOFENCE -->|Yes| RECORD_IN[Record Clock-In<br/>Time + Location]

    RECORD_IN --> WORK[Working Period]
    WORK --> CLOCK_OUT[Employee Clocks Out]
    CLOCK_OUT --> CALC_HOURS[Calculate Hours Worked]
    CALC_HOURS --> OVERTIME{Overtime<br/>Threshold?}
    OVERTIME -->|Yes| CALC_OT[Calculate Overtime Hours]
    OVERTIME -->|No| RECORD_OUT[Record Normal Hours]
    CALC_OT --> RECORD_OUT

    RECORD_OUT --> AUTO_CHECK{Auto Clock-Out<br/>Policy Active?}
    AUTO_CHECK -->|Missed Clock-Out| AUTO_CLOCK[Auto Clock-Out at Cutoff]

    RECORD_OUT --> SYNC[Sync to Payroll<br/>Attendance Summary]
```
