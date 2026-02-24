# Workflows and User Journeys -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Overview

This document maps the end-to-end workflows for key business processes in the ERP-BSS-OSS platform, covering the complete telecom subscriber lifecycle from acquisition through retention and winback, plus utility metering workflows.

---

## 2. Subscriber Lifecycle Workflow

```mermaid
graph TB
    subgraph "Acquisition"
        A1[Lead Generation] --> A2[Customer Application]
        A2 --> A3[KYC Verification]
        A3 --> A4[Product Selection]
        A4 --> A5[Order Creation]
    end

    subgraph "Activation"
        A5 --> B1[Resource Allocation]
        B1 --> B2[SIM / Number Assignment]
        B2 --> B3[Network Provisioning]
        B3 --> B4[Billing Account Setup]
        B4 --> B5[Welcome Communication]
    end

    subgraph "In-Life Management"
        B5 --> C1[Usage & Charging]
        C1 --> C2[Top-Up / Bill Payment]
        C2 --> C3[Plan Changes]
        C3 --> C4[VAS Subscriptions]
        C4 --> C5[Support Requests]
        C5 --> C1
    end

    subgraph "Retention"
        C1 --> D1{Churn Risk?}
        D1 -->|High| D2[Retention Offer]
        D2 --> D3{Accepted?}
        D3 -->|Yes| C1
        D3 -->|No| E1
        D1 -->|Low| C1
    end

    subgraph "Termination"
        E1[Voluntary Churn / Involuntary] --> E2[Service Deactivation]
        E2 --> E3[Final Bill]
        E3 --> E4[Number Quarantine]
        E4 --> E5[Data Retention]
    end
```

---

## 3. Order-to-Activate (O2A) Workflow

The O2A workflow is the most critical cross-service orchestration in the platform.

```mermaid
sequenceDiagram
    participant PORTAL as Portal / USSD
    participant OM as Order Management
    participant PC as Product Catalog
    participant RI as Resource Inventory
    participant BIL as Billing
    participant PROV as Provisioning
    participant SI as Service Inventory
    participant NOTIFY as Notification

    PORTAL->>OM: Submit order (customer_id, products[])
    OM->>PC: Validate product availability + eligibility
    PC-->>OM: Products valid

    OM->>OM: Create order (status=acknowledged)
    OM->>OM: Generate order number (ORD-YYYYMMDD-XXXXXX)

    par Resource Reservation
        OM->>RI: Reserve SIM
        OM->>RI: Reserve MSISDN
        OM->>RI: Reserve IP address (if broadband)
    end

    RI-->>OM: Resources reserved

    OM->>OM: Update status = in_progress

    OM->>BIL: Create billing items (subscription + activation fee)
    BIL-->>OM: Billing configured

    OM->>PROV: Provision services
    PROV->>PROV: Activate on HLR/HSS
    PROV->>PROV: Configure PCRF (QoS profile)
    PROV->>SI: Register active service

    alt Provisioning Success
        PROV-->>OM: All services activated
        OM->>OM: Update status = completed
        OM->>NOTIFY: Send activation confirmation
    else Partial Failure
        PROV-->>OM: Some services failed
        OM->>OM: Update status = partial
        OM->>OM: Create fallout record
        OM->>NOTIFY: Send partial activation notice
    else Total Failure
        PROV-->>OM: All services failed
        OM->>RI: Release reserved resources
        OM->>BIL: Reverse billing items
        OM->>OM: Update status = failed
        OM->>NOTIFY: Send failure notice
    end
```

---

## 4. Billing Cycle Workflow

```mermaid
graph TB
    A[Scheduler triggers billing cycle] --> B[Select customers in cycle]
    B --> C[For each customer]

    C --> D[Collect recurring charges]
    D --> E[Collect rated CDRs from ClickHouse]
    E --> F[Collect one-time charges]
    F --> G[Collect adjustments / credits]
    G --> H[Apply discount hierarchy]

    H --> I[Calculate subtotal]
    I --> J[Calculate taxes per jurisdiction]
    J --> K[Generate invoice]
    K --> L{Bill amount > 0?}

    L -->|Yes| M[Store invoice in PostgreSQL]
    L -->|No| N[Store zero-value invoice]

    M --> O[Deliver invoice]
    O --> P{Delivery method}
    P --> Q[Email PDF]
    P --> R[SMS with portal link]
    P --> S[Portal notification]

    Q & R & S --> T[Start dunning timer]
    T --> U[Monitor payment]
```

---

## 5. Revenue Assurance Workflow

```mermaid
graph TB
    subgraph "Data Collection"
        SRC1[CDR Stream] --> COLLECT[Collect]
        SRC2[Billing Records] --> COLLECT
        SRC3[Network Events] --> COLLECT
        SRC4[Payment Records] --> COLLECT
    end

    subgraph "Reconciliation"
        COLLECT --> REC1[CDR-to-Bill Reconciliation]
        REC1 --> REC2{Variance > threshold?}
        REC2 -->|Yes| ALERT[Create RA Finding]
        REC2 -->|No| PASS[Mark Reconciled]
    end

    subgraph "Fraud Detection"
        COLLECT --> ML[ML Feature Extraction]
        ML --> MODEL[Fraud Model Inference]
        MODEL --> SCORE{Score > 0.85?}
        SCORE -->|Yes| FRAUD[Create Fraud Alert]
        SCORE -->|No| LOG[Log for batch review]
    end

    subgraph "Action"
        ALERT --> ANALYST[RA Analyst Review]
        FRAUD --> ANALYST
        ANALYST --> FIX[Corrective Action]
        FIX --> REPORT[Management Report]
    end
```

---

## 6. USSD Session Workflow

```mermaid
stateDiagram-v2
    [*] --> MainMenu: Subscriber dials *123#

    state MainMenu {
        [*] --> Display: "1.Balance 2.TopUp 3.Plans 4.Data 5.Help"
    }

    MainMenu --> BalanceCheck: Input "1"
    MainMenu --> TopUpMenu: Input "2"
    MainMenu --> PlanMenu: Input "3"
    MainMenu --> DataMenu: Input "4"
    MainMenu --> HelpMenu: Input "5"

    state BalanceCheck {
        [*] --> FetchBalance
        FetchBalance --> ShowBalance: "Airtime: $50.00\nData: 2.5GB\nSMS: 100"
    }

    state TopUpMenu {
        [*] --> SelectAmount: "1.$5 2.$10 3.$20 4.Custom"
        SelectAmount --> ConfirmPayment
        ConfirmPayment --> ProcessPayment
        ProcessPayment --> TopUpSuccess: "Topped up $10. Balance: $60.00"
    }

    state PlanMenu {
        [*] --> ShowPlans: "1.Daily $1 2.Weekly $5 3.Monthly $20"
        ShowPlans --> SelectPlan
        SelectPlan --> ActivatePlan
        ActivatePlan --> PlanSuccess: "Plan activated. Valid for 30 days"
    }

    BalanceCheck --> [*]: Session end
    TopUpMenu --> [*]: Session end
    PlanMenu --> [*]: Session end
```

---

## 7. Partner Settlement Workflow

```mermaid
sequenceDiagram
    participant SCHED as Monthly Scheduler
    participant PS as Partner Service
    participant BIL as Billing
    participant CH as ClickHouse
    participant FIN as Finance (ERP-Finance)
    participant PORTAL as Partner Portal

    SCHED->>PS: Trigger settlement for January
    PS->>PS: Get all active partners + agreements
    loop For each partner
        PS->>CH: Query revenue by partner for January
        CH-->>PS: Gross revenue: $500,000
        PS->>PS: Apply revenue share model
        PS->>PS: Partner share: $325,000 (65%)
        PS->>PS: Operator share: $175,000 (35%)
        PS->>PS: Create settlement_run record
    end
    PS->>FIN: Post journal entries (AP to partner)
    PS->>PORTAL: Publish settlement report
    PS->>PS: Send settlement notification to partner
    Note over PS,PORTAL: Partner has 15 days to dispute
```

---

## 8. Network Fault Management Workflow

```mermaid
graph TB
    A[Network alarm received<br/>SNMP trap / syslog] --> B[Alarm correlation engine]
    B --> C{Known pattern?}
    C -->|Yes| D[Auto-create trouble ticket]
    C -->|No| E[Queue for NOC review]

    D --> F{Service impacting?}
    F -->|Yes| G[Priority: Critical]
    F -->|No| H[Priority: Medium]

    G --> I[Notify on-call engineer]
    I --> J[SLA timer starts (4 hour resolution)]
    J --> K{Remote fix possible?}
    K -->|Yes| L[Remote remediation]
    K -->|No| M[Dispatch field engineer]

    M --> N[Workforce management]
    N --> O[Assign nearest available tech]
    O --> P[Travel + repair]
    P --> Q[Verify service restored]

    L --> Q
    Q --> R[Close trouble ticket]
    R --> S[Update SLA metrics]
```

---

## 9. Provisioning Rollback Workflow

```mermaid
graph TB
    A[Provisioning task received] --> B[Step 1: Reserve resources]
    B --> C{Success?}
    C -->|Yes| D[Step 2: Configure HLR/HSS]
    C -->|No| Z1[Rollback: Release resources]

    D --> E{Success?}
    E -->|Yes| F[Step 3: Configure PCRF]
    E -->|No| Z2[Rollback: Remove HLR entry + Release resources]

    F --> G{Success?}
    G -->|Yes| H[Step 4: Register in Service Inventory]
    G -->|No| Z3[Rollback: Remove PCRF + HLR + Release resources]

    H --> I{Success?}
    I -->|Yes| J[Provisioning COMPLETE]
    I -->|No| Z4[Rollback: Full compensation chain]

    Z1 & Z2 & Z3 & Z4 --> FALLOUT[Add to Fallout Queue]
    FALLOUT --> RETRY{Retry count < 3?}
    RETRY -->|Yes| A
    RETRY -->|No| MANUAL[Manual intervention required]
```

---

## 10. Smart Meter Lifecycle Workflow

```mermaid
graph TB
    subgraph "Installation"
        M1[Meter procurement] --> M2[Meter registration in system]
        M2 --> M3[Assign to customer + location]
        M3 --> M4[Field installation]
        M4 --> M5[Communication test (DLMS/COSEM)]
        M5 --> M6[Initial reading recorded]
    end

    subgraph "Operations"
        M6 --> O1[Periodic readings every 15 min]
        O1 --> O2{Reading valid?}
        O2 -->|Yes| O3[Store + calculate charges]
        O2 -->|No| O4[Flag for investigation]
        O4 --> O5{Tamper detected?}
        O5 -->|Yes| O6[Create tamper alert + dispatch]
        O5 -->|No| O7[Estimate reading + flag]
        O3 --> O1
    end

    subgraph "Decommission"
        D1[Meter end of life / customer move] --> D2[Final reading]
        D2 --> D3[Final bill]
        D3 --> D4[Decommission meter]
        D4 --> D5[Return to inventory or dispose]
    end
```
