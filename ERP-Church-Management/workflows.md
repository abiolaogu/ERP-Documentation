# Workflows -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Core Workflows Overview

ERP-Church-Management implements six primary workflows that map to the RCCG Follow-up & Visitation Ministry framework. Each workflow is event-driven, with Kafka/Redpanda messages triggering transitions between services.

---

## 2. Workflow 1: 4Cs Assimilation Pipeline

The 4Cs (Connect, Capture, Communicate, Convert) represent the end-to-end visitor-to-member journey.

```mermaid
flowchart TD
    subgraph Connect["Phase 1: CONNECT"]
        C1["Visitor arrives at church"]
        C2["Greeter welcomes visitor"]
        C3["Visitor seated in designated area"]
        C4["Post-service personal greeting"]
    end

    subgraph Capture["Phase 2: CAPTURE"]
        CA1["Visitor fills digital card (kiosk/tablet)"]
        CA2["System creates visitor record"]
        CA3["Account Officer auto-assigned"]
        CA4["Welcome gift logged"]
    end

    subgraph Communicate["Phase 3: COMMUNICATE"]
        CO1["72-hour automated follow-up triggered"]
        CO2["Multi-channel outreach (SMS + WhatsApp + Email)"]
        CO3["Account Officer personal call/visit"]
        CO4["Follow-up activities logged"]
        CO5{"Visitor responds?"}
    end

    subgraph Convert["Phase 4: CONVERT"]
        CV1["Invite to next service / special event"]
        CV2["Enroll in New Believer Class (NBC)"]
        CV3["Assign Mentor (90-120 days)"]
        CV4["Place in Small Group / Home Fellowship"]
        CV5["Integrate into Workforce / Ministry"]
        CV6["Full membership conversion"]
    end

    C1 --> C2 --> C3 --> C4
    C4 --> CA1 --> CA2 --> CA3 --> CA4
    CA4 --> CO1 --> CO2 --> CO3 --> CO4 --> CO5
    CO5 -->|Yes| CV1
    CO5 -->|No| FU["Further Follow-up Directorate"]
    FU --> CO2
    CV1 --> CV2 --> CV3 --> CV4 --> CV5 --> CV6
```

---

## 3. Workflow 2: 72-Hour Follow-up Automation

```mermaid
sequenceDiagram
    participant VIS as Visitor Service
    participant RP as Redpanda
    participant FU as Follow-up Service
    participant COM as Communication Service
    participant KPI as KPI Service

    Note over VIS: Visitor registered (digitally or manually)
    VIS->>RP: Publish visitor.created
    RP->>FU: Consume visitor.created
    FU->>FU: Auto-assign Account Officer
    FU->>RP: Publish followup.assigned

    loop Every hour (cron)
        FU->>FU: Check visitors within 72-hour window
        FU->>COM: Request multi-channel outreach
        COM->>COM: Send SMS via Twilio
        COM->>COM: Send WhatsApp via Business API
        COM->>COM: Send Email via SMTP
        COM->>RP: Publish communication.sent
    end

    Note over FU: Account Officer records contact
    FU->>VIS: Update contactedWithin72Hours = true
    FU->>RP: Publish visitor.72hr.contact.recorded
    RP->>KPI: Consume and update 72-hour KPI
```

---

## 4. Workflow 3: 6-Directorate Routing

Each directorate handles a specific phase of the soul-care pipeline:

```mermaid
flowchart TD
    NEW["New Visitor/Convert"] --> D1["1st Timer Directorate"]
    D1 -->|Initial contact & welcome| D2["Further Follow-up Directorate"]
    D2 -->|Needs counseling| D3["Counselling Directorate"]
    D2 -->|Ready for group placement| D4["Natural Group Directorate"]
    D3 -->|Counseling complete| D4
    D4 -->|Placed in Youth/Men/Women/etc.| D5["Development & Structuring Directorate"]
    D5 -->|Trained and deployed| D6["Welfare & Finance Directorate"]

    subgraph D1_Tasks["1st Timer Directorate Tasks"]
        D1T1["Welcome gift distribution"]
        D1T2["First contact within 72 hours"]
        D1T3["Home visitation scheduling"]
        D1T4["Church information sharing"]
    end

    subgraph D2_Tasks["Further Follow-up Tasks"]
        D2T1["Repeat contact attempts"]
        D2T2["Barrier identification"]
        D2T3["Needs assessment"]
        D2T4["Invitation to events"]
    end

    subgraph D3_Tasks["Counselling Tasks"]
        D3T1["Salvation counseling"]
        D3T2["Marriage counseling"]
        D3T3["Crisis intervention"]
        D3T4["Spiritual guidance"]
    end

    subgraph D4_Tasks["Natural Group Tasks"]
        D4T1["Age/gender group assignment"]
        D4T2["Home fellowship placement"]
        D4T3["Sunday school enrollment"]
        D4T4["Group leader introduction"]
    end

    subgraph D5_Tasks["Development & Structuring Tasks"]
        D5T1["Leadership training enrollment"]
        D5T2["Ministry skill discovery"]
        D5T3["Workforce integration"]
        D5T4["Spiritual growth tracking"]
    end

    subgraph D6_Tasks["Welfare & Finance Tasks"]
        D6T1["Needs assessment"]
        D6T2["Benevolence fund application"]
        D6T3["Financial literacy training"]
        D6T4["Ongoing welfare monitoring"]
    end

    D1 --> D1_Tasks
    D2 --> D2_Tasks
    D3 --> D3_Tasks
    D4 --> D4_Tasks
    D5 --> D5_Tasks
    D6 --> D6_Tasks
```

---

## 5. Workflow 4: Giving Lifecycle

```mermaid
flowchart TD
    START["Member initiates giving"] --> TYPE{"Giving Type?"}
    TYPE -->|Tithe| T["Record as Tithe"]
    TYPE -->|Offering| O["Record as Offering"]
    TYPE -->|Pledge| P["Check Pledge Campaign"]
    TYPE -->|Donation| D["Record as Donation"]

    T --> REC["Create donation record"]
    O --> REC
    D --> REC

    P --> PCHECK{"Existing pledge?"}
    PCHECK -->|Yes| PFULFILL["Update pledge fulfillment"]
    PCHECK -->|No| PNEW["Create new pledge + first payment"]
    PFULFILL --> REC
    PNEW --> REC

    REC --> TAX{"Tax deductible?"}
    TAX -->|Yes| RECEIPT["Generate tax receipt"]
    TAX -->|No| SKIP["Skip receipt"]

    RECEIPT --> EVT["Publish giving.recorded event"]
    SKIP --> EVT

    EVT --> FIN["ERP-Finance: Journal entry"]
    EVT --> KPI["KPI Service: Update giving metrics"]

    subgraph Annual["Annual Process"]
        YEAR_END["Year-end trigger"] --> STMT["Generate giving statements"]
        STMT --> NOTIFY["Notify members via email"]
    end
```

---

## 6. Workflow 5: Event & Attendance

```mermaid
flowchart TD
    CREATE["Admin creates event"] --> PUB["Publish event details"]
    PUB --> NOTIFY["Communication Service: Notify members"]

    subgraph Check_In["Check-in Process"]
        ARRIVE["Member/Visitor arrives"]
        ARRIVE --> METHOD{"Check-in method?"}
        METHOD -->|QR Code| QR["Scan QR code at kiosk"]
        METHOD -->|NFC| NFC["Tap NFC badge"]
        METHOD -->|Manual| MAN["Worker marks attendance"]

        QR --> VALIDATE["Validate member/visitor ID"]
        NFC --> VALIDATE
        MAN --> VALIDATE

        VALIDATE --> RECORD["Create attendance record"]
    end

    RECORD --> CHECK{"First-time visitor?"}
    CHECK -->|Yes| VIS_FLOW["Trigger visitor.created workflow"]
    CHECK -->|No| UPDATE["Update member last_attendance"]

    UPDATE --> EVT["Publish event.checkin"]
    VIS_FLOW --> EVT
    EVT --> KPI["KPI Service: Update attendance metrics"]
```

---

## 7. Workflow 6: Welfare Case Management

```mermaid
flowchart TD
    REQ["Need identified"] --> CREATE["Create welfare case"]
    CREATE --> ASSESS["Case worker assessment"]
    ASSESS --> TRIAGE{"Urgency level?"}

    TRIAGE -->|Critical| FAST["Fast-track approval"]
    TRIAGE -->|High| REVIEW["Directorate head review"]
    TRIAGE -->|Medium/Low| QUEUE["Standard queue"]

    FAST --> APPROVE["Auto-approve (within limit)"]
    REVIEW --> DECISION{"Approved?"}
    QUEUE --> REVIEW

    DECISION -->|Yes| APPROVE
    DECISION -->|No| REJECT["Case rejected with reason"]

    APPROVE --> DISBURSE["Funds disbursed"]
    DISBURSE --> DELIVER["Assistance delivered"]
    DELIVER --> FOLLOWUP["Follow-up on effectiveness"]
    FOLLOWUP --> CLOSE["Case closed"]
    REJECT --> CLOSE

    CLOSE --> EVT["Publish welfare.case.closed"]
    EVT --> KPI["KPI Service: Update welfare metrics"]
```

---

## 8. Workflow 7: Discipleship Pipeline

```mermaid
flowchart TD
    CONVERT["Visitor converts to member"] --> NBC["Enroll in New Believer Class"]
    NBC --> NBC_TRACK["Track NBC progress"]
    NBC_TRACK --> NBC_DONE{"NBC completed?"}

    NBC_DONE -->|Yes| MENTOR["Assign Mentor"]
    NBC_DONE -->|No| NBC_SUPPORT["Additional support / re-enroll"]
    NBC_SUPPORT --> NBC_TRACK

    MENTOR --> MENTOR_TRACK["Track mentorship (90-120 days)"]
    MENTOR_TRACK --> MENTOR_DONE{"Mentorship completed?"}

    MENTOR_DONE -->|Yes| SS["Enroll in Sunday School"]
    MENTOR_DONE -->|No| MENTOR_EXT["Extend mentorship"]
    MENTOR_EXT --> MENTOR_TRACK

    SS --> HF["Place in Home Fellowship"]
    HF --> WORK["Integrate into Workforce"]
    WORK --> LEAD["Leadership development path"]

    subgraph Progress_Tracking
        PT1["Engagement score calculated"]
        PT2["Milestones recorded"]
        PT3["KPIs updated"]
    end

    NBC_DONE --> Progress_Tracking
    MENTOR_DONE --> Progress_Tracking
    SS --> Progress_Tracking
```

---

## 9. Workflow 8: Absentee Member Detection

```mermaid
flowchart TD
    CRON["Daily cron job"] --> QUERY["Query members with no attendance > 3 weeks"]
    QUERY --> FOUND{"Absentees found?"}

    FOUND -->|Yes| LIST["Generate absentee list"]
    FOUND -->|No| DONE["Job complete"]

    LIST --> LOOP["For each absentee"]
    LOOP --> FU_CREATE["Create follow-up activity: Absentee Check"]
    FU_CREATE --> AO_NOTIFY["Notify Account Officer"]
    AO_NOTIFY --> OUTREACH["Multi-channel outreach"]
    OUTREACH --> LOG["Log contact attempt"]

    LOG --> RESP{"Member responsive?"}
    RESP -->|Yes| RETURN["Encourage return / address barriers"]
    RESP -->|No, 3+ attempts| WELFARE["Escalate to Welfare Directorate"]
    WELFARE --> VISIT["Home visitation"]
```

---

## 10. User Journey Maps

### 10.1 First-Time Visitor Journey

```mermaid
journey
    title First-Time Visitor Journey
    section Arrival
      Walk into church: 3: Visitor
      Greeted by usher: 4: Visitor
      Directed to info desk: 3: Visitor
    section Registration
      Fill digital visitor card: 4: Visitor
      Receive welcome gift: 5: Visitor
      Seated in service: 4: Visitor
    section Follow-up
      Receive SMS within 24 hours: 5: Account Officer
      Receive WhatsApp message: 5: Account Officer
      Personal phone call: 4: Account Officer
    section Return Visit
      Invited to next service: 4: Account Officer
      Attend 2nd service: 5: Visitor
      Meet small group leader: 4: Group Leader
    section Conversion
      Attend New Believer Class: 4: Visitor
      Accept mentorship: 5: Visitor
      Join home fellowship: 5: Member
      Begin serving as volunteer: 5: Member
```

### 10.2 Account Officer Daily Journey

```mermaid
journey
    title Account Officer Daily Workflow
    section Morning
      Check mobile app dashboard: 4: Account Officer
      Review pending follow-ups: 4: Account Officer
      See 72-hour countdown timers: 3: Account Officer
    section Outreach
      Call first-timer from yesterday: 5: Account Officer
      Send WhatsApp to absent member: 4: Account Officer
      Log follow-up in app: 4: Account Officer
    section Midday
      Visit welfare case at home: 4: Account Officer
      Log visit notes with photos: 4: Account Officer
    section Evening
      Check KPI progress: 3: Account Officer
      Prepare for tomorrow: 4: Account Officer
      Review mentor-mentee pair progress: 4: Account Officer
```
