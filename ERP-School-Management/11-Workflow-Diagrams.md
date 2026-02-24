# ERP-School-Management -- Workflow Diagrams

**Product:** EduCore Pro
**Version:** 1.0.0
**Date:** 2026-02-23

---

## 1. Admission Flow

```mermaid
flowchart TD
    A["Parent visits<br/>school website"] --> B["Clicks 'Apply Now'"]
    B --> C["Fills online<br/>application form"]
    C --> D["Uploads required<br/>documents"]
    D --> E["Enters guardian<br/>information"]
    E --> F["Enters medical<br/>information"]
    F --> G["Submits<br/>application"]
    G --> H["System assigns<br/>reference number"]
    H --> I["Admin notified<br/>of new application"]
    I --> J["Admin reviews<br/>application"]
    J --> K{Decision}
    K -->|Request info| L["Parent notified<br/>for additional docs"]
    L --> J
    K -->|Schedule test| M["Admission test<br/>conducted"]
    M --> N["Results<br/>evaluated"]
    N --> K
    K -->|Approve| O["Generate admission<br/>letter"]
    K -->|Reject| P["Send rejection<br/>notification"]
    O --> Q["Generate registration<br/>fee invoice"]
    Q --> R["Send to parent<br/>via email/SMS"]
    R --> S["Parent pays<br/>registration fee"]
    S --> T["Payment<br/>confirmed"]
    T --> U["Complete enrollment"]
    U --> V["Assign class<br/>& student ID"]
    V --> W["Send welcome<br/>package"]
```

---

## 2. Enrollment Flow

```mermaid
flowchart TD
    A["Admission<br/>approved"] --> B["Admin opens<br/>enrollment module"]
    B --> C["Select student<br/>from approved list"]
    C --> D["Assign academic<br/>year"]
    D --> E["Assign grade<br/>level"]
    E --> F["Assign class<br/>/ section"]
    F --> G{Capacity check}
    G -->|Full| H["Select alternative<br/>class"]
    H --> F
    G -->|Available| I["Create enrollment<br/>record"]
    I --> J["Generate student<br/>number"]
    J --> K["Create user<br/>account"]
    K --> L["Link student<br/>to guardians"]
    L --> M["Create dietary<br/>profile"]
    M --> N["Generate fee<br/>invoices"]
    N --> O["Publish student.enrolled<br/>event"]
    O --> P["Notification sent<br/>to all stakeholders"]
```

---

## 3. Grading Workflow

```mermaid
flowchart TD
    A["Teacher creates<br/>assessment"] --> B["Set type, max score,<br/>weight, due date"]
    B --> C["Assessment<br/>published to students"]
    C --> D["Students complete<br/>assessment"]
    D --> E["Teacher enters<br/>scores"]
    E --> F["System calculates<br/>percentage"]
    F --> G["System maps to<br/>letter grade"]
    G --> H["Teacher adds<br/>feedback"]
    H --> I["Status: DRAFT"]
    I --> J["Teacher reviews<br/>& submits"]
    J --> K["Status: SUBMITTED"]
    K --> L{Approval<br/>required?}
    L -->|Yes| M["HOD reviews<br/>grades"]
    M --> N{Approved?}
    N -->|No| O["Return with<br/>comments"]
    O --> E
    N -->|Yes| P["Teacher publishes"]
    L -->|No| P
    P --> Q["Status: PUBLISHED"]
    Q --> R["Students & parents<br/>can view grades"]
    R --> S["grade.published<br/>event emitted"]
    S --> T["Analytics service<br/>processes"]
    S --> U["Gamification service<br/>awards points"]
    S --> V["Notification service<br/>alerts parents"]

    subgraph "Lock Phase"
        W["Admin locks grades<br/>after review period"]
        W --> X["Status: LOCKED"]
        X --> Y["Grades immutable"]
    end

    Q --> W
```

---

## 4. Fee Payment Workflow

```mermaid
flowchart TD
    A["Admin creates<br/>fee structure"] --> B["Define fee type,<br/>amount, due date"]
    B --> C["Generate invoices<br/>for students"]
    C --> D["Invoices visible<br/>on parent portal"]
    D --> E["System sends<br/>payment reminder"]
    E --> F["Parent reviews<br/>invoice"]
    F --> G{Payment<br/>method?}

    G -->|Credit Card| H["Redirect to<br/>Stripe"]
    G -->|Paystack| I["Redirect to<br/>Paystack"]
    G -->|Flutterwave| J["Redirect to<br/>Flutterwave"]
    G -->|Mobile Money| K["Enter mobile<br/>number"]
    G -->|Bank Transfer| L["Display bank<br/>details"]
    G -->|Cash| M["Admin records<br/>manual payment"]

    H & I & J & K --> N["Payment gateway<br/>processes"]
    N --> O{Success?}
    O -->|No| P["Display error<br/>retry"]
    P --> G
    O -->|Yes| Q["Webhook received"]

    L --> R["Parent transfers<br/>funds"]
    R --> S["Admin reconciles<br/>transfer"]

    Q & S & M --> T["Create FeePayment<br/>record"]
    T --> U["Update StudentFee<br/>balance"]
    U --> V{Fully paid?}
    V -->|Yes| W["Status: PAID"]
    V -->|No| X["Status: PARTIAL"]
    X --> Y["Schedule next<br/>reminder"]
    W --> Z["Generate receipt"]
    Z --> AA["Send confirmation<br/>notification"]
    AA --> AB["payment.completed<br/>event emitted"]
```

---

## 5. Exam Lifecycle

```mermaid
flowchart TD
    A["Admin sets<br/>exam period"] --> B["Teachers create<br/>exam assessments"]
    B --> C["Set exam type:<br/>MIDTERM or FINAL"]
    C --> D["Set max score,<br/>duration, date"]
    D --> E["Exam schedule<br/>published"]
    E --> F["Students & parents<br/>notified"]
    F --> G["Exam day:<br/>students attend"]
    G --> H["Teacher/invigilator<br/>supervises"]
    H --> I["Exam papers<br/>collected"]
    I --> J["Teacher grades<br/>exam papers"]
    J --> K["Enter scores<br/>in gradebook"]
    K --> L["Calculate term<br/>averages"]
    L --> M["Compute class<br/>rankings"]
    M --> N["Generate term<br/>summaries"]
    N --> O["Report cards<br/>prepared"]
    O --> P["Admin reviews<br/>& approves"]
    P --> Q["Report cards<br/>published"]
    Q --> R["Parents notified<br/>to view results"]
```

---

## 6. Timetable Generation

```mermaid
flowchart TD
    A["Admin opens<br/>timetable module"] --> B["Select class<br/>and term"]
    B --> C["Define daily<br/>period structure"]
    C --> D["Set period times<br/>& break slots"]
    D --> E["For each day<br/>of the week"]
    E --> F["Assign subject<br/>to period"]
    F --> G["Assign teacher<br/>to period"]
    G --> H["Assign room<br/>to period"]
    H --> I{Conflict<br/>check}
    I -->|Teacher conflict| J["Teacher already<br/>assigned elsewhere"]
    J --> K["Choose different<br/>teacher or period"]
    K --> G
    I -->|Room conflict| L["Room already<br/>booked"]
    L --> M["Choose different<br/>room or period"]
    M --> H
    I -->|No conflict| N["Save timetable<br/>slot"]
    N --> O{All periods<br/>assigned?}
    O -->|No| E
    O -->|Yes| P["Timetable<br/>complete"]
    P --> Q["Publish to<br/>students & teachers"]
    Q --> R["Visible on<br/>mobile app"]
```

---

## 7. Attendance Workflow

```mermaid
flowchart TD
    A["School day<br/>begins"] --> B["Teacher opens<br/>attendance page"]
    B --> C["Student roster<br/>displayed"]
    C --> D["Mark each student"]
    D --> E{Status}
    E -->|Present| F["Record: PRESENT"]
    E -->|Absent| G["Record: ABSENT<br/>+ optional reason"]
    E -->|Tardy| H["Record: TARDY<br/>+ minutes late"]
    E -->|Excused| I["Record: EXCUSED_ABSENT<br/>+ reason"]
    E -->|Early Dismissal| J["Record: EARLY_DISMISSAL<br/>+ departure time"]
    F & G & H & I & J --> K["Save attendance<br/>records"]
    K --> L["Check for<br/>absences"]
    L --> M{Absent<br/>students?}
    M -->|Yes| N["Send notification<br/>to parents"]
    M -->|No| O["Attendance<br/>complete"]
    N --> O
    O --> P["Update attendance<br/>dashboard"]
    P --> Q["Analytics service<br/>processes trends"]
```

---

## 8. Blockchain Certificate Issuance

```mermaid
flowchart TD
    A["Student completes<br/>program/course"] --> B["Admin initiates<br/>certificate issuance"]
    B --> C["System prepares<br/>certificate data"]
    C --> D["Generate PDF<br/>certificate"]
    D --> E["Upload to<br/>IPFS"]
    E --> F["Get IPFS<br/>hash"]
    F --> G["Write to<br/>blockchain"]
    G --> H["Get transaction ID<br/>& block number"]
    H --> I["Store blockchain_credential<br/>record"]
    I --> J["Generate verification<br/>URL & QR code"]
    J --> K["Student notified<br/>of certificate"]
    K --> L["Student downloads<br/>certificate PDF"]
    L --> M["Student shares<br/>verification link"]
    M --> N["Verifier validates<br/>on blockchain"]
```

---

## 9. Communication Workflow

```mermaid
flowchart TD
    A["Sender composes<br/>message"] --> B{Message type}
    B -->|Individual| C["Select recipient"]
    B -->|Group| D["Select group<br/>(class, grade, all)"]
    B -->|Announcement| E["Set target audience<br/>& priority"]
    C & D & E --> F["Compose message<br/>body & attachments"]
    F --> G["Send message"]
    G --> H["Message stored<br/>in database"]
    H --> I["Determine delivery<br/>channels"]
    I --> J{Recipient<br/>preferences}
    J -->|Email| K["Queue email<br/>via SendGrid/SES"]
    J -->|SMS| L["Queue SMS<br/>via Twilio"]
    J -->|Push| M["Queue push<br/>notification"]
    J -->|In-App| N["Store in<br/>notification center"]
    K & L & M & N --> O["Track delivery<br/>status"]
    O --> P["Mark as read<br/>when viewed"]
```

---

## 10. Financial Aid Workflow

```mermaid
flowchart TD
    A["Admin creates<br/>financial aid program"] --> B["Define criteria:<br/>merit, sibling, need"]
    B --> C["Set discount value<br/>(fixed or %)"]
    C --> D["Identify eligible<br/>students"]
    D --> E["Review & approve<br/>awards"]
    E --> F["Create StudentFinancialAid<br/>records"]
    F --> G["Future fee calculations<br/>apply discount"]
    G --> H["Monitor student<br/>eligibility"]
    H --> I{Still<br/>eligible?}
    I -->|Yes| J["Continue aid"]
    I -->|No| K["Revoke aid"]
    K --> L["Notify student<br/>& parent"]
    J --> H
```

---

## 11. Data Migration Workflow

```mermaid
flowchart TD
    A["School provides<br/>legacy data"] --> B["Migration service<br/>receives files"]
    B --> C["Validate data<br/>format & schema"]
    C --> D{Valid?}
    D -->|No| E["Generate error<br/>report"]
    E --> F["School corrects<br/>data"]
    F --> B
    D -->|Yes| G["Transform to<br/>EduCore schema"]
    G --> H["Load into<br/>staging tables"]
    H --> I["Run validation<br/>checks"]
    I --> J["Admin reviews<br/>sample records"]
    J --> K{Approve?}
    K -->|No| L["Rollback staging"]
    K -->|Yes| M["Promote to<br/>production"]
    M --> N["Generate migration<br/>report"]
```
