# Use Cases -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Overview

This document defines 28 use cases covering the complete telecom subscriber lifecycle, utility metering, and operational scenarios. Each use case includes actors, preconditions, flow, postconditions, and a Mermaid diagram.

---

## 2. Use Case Index

| # | Use Case | Domain | Priority |
|---|----------|--------|----------|
| UC-01 | New Prepaid Subscriber Onboarding | Customer | Critical |
| UC-02 | Postpaid Subscriber Registration | Customer | Critical |
| UC-03 | KYC Document Verification | Customer | Critical |
| UC-04 | Customer 360 View | Customer | High |
| UC-05 | Product Catalog Browsing | Product | High |
| UC-06 | Create Product Bundle | Product | Medium |
| UC-07 | Place New Subscription Order | Order | Critical |
| UC-08 | Order Cancellation | Order | High |
| UC-09 | Plan Upgrade / Downgrade | Order | High |
| UC-10 | Real-Time Voice Call Rating | Billing | Critical |
| UC-11 | Data Session Charging | Billing | Critical |
| UC-12 | Prepaid Balance Top-Up | Billing | Critical |
| UC-13 | Auto-Recharge Trigger | Billing | High |
| UC-14 | Postpaid Invoice Generation | Billing | Critical |
| UC-15 | Bill Shock Prevention Alert | Billing | High |
| UC-16 | Dunning Escalation | Billing | High |
| UC-17 | Billing Dispute Resolution | Billing | Medium |
| UC-18 | CDR Collection and Normalization | Mediation | Critical |
| UC-19 | SIM Swap (Physical to eSIM) | Provisioning | High |
| UC-20 | Number Porting (Port-In) | Provisioning | High |
| UC-21 | Network Fault Detection and Ticketing | Operations | High |
| UC-22 | Field Workforce Dispatch | Operations | Medium |
| UC-23 | MVNO Partner Onboarding | Partner | High |
| UC-24 | Monthly Revenue Settlement | Partner | High |
| UC-25 | USSD Balance Check | Channel | High |
| UC-26 | SIM Box Fraud Detection | Revenue Assurance | High |
| UC-27 | Smart Meter Reading Collection | Utilities | Medium |
| UC-28 | Prepaid Electricity STS Token Purchase | Utilities | Medium |

---

## 3. Detailed Use Cases

### UC-01: New Prepaid Subscriber Onboarding

**Actors:** Subscriber, Customer Service Agent, System
**Preconditions:** SIM inventory has available SIMs; number pool has available MSISDNs
**Trigger:** Customer visits retail point or self-service portal

```mermaid
sequenceDiagram
    participant SUB as Subscriber
    participant AGENT as Agent/Portal
    participant CM as Customer Mgmt
    participant KYC as KYC Service
    participant RI as Resource Inventory
    participant PROV as Provisioning
    participant BIL as Billing

    SUB->>AGENT: Request new prepaid SIM
    AGENT->>CM: Create customer record
    CM-->>AGENT: Customer ID
    AGENT->>KYC: Upload KYC documents
    KYC->>KYC: OCR + Verification
    KYC-->>AGENT: KYC Approved
    AGENT->>RI: Allocate SIM (ICCID) + Number (MSISDN)
    RI-->>AGENT: SIM + Number assigned
    AGENT->>PROV: Activate SIM on HLR/HSS
    PROV-->>AGENT: Activation confirmed
    AGENT->>BIL: Create prepaid balance (initial top-up)
    BIL-->>AGENT: Balance created
    AGENT-->>SUB: Welcome pack (SIM + number + PIN)
```

**Postconditions:** Customer record created, KYC verified, SIM active on network, prepaid balance initialized.

---

### UC-05: Product Catalog Browsing

**Actors:** Subscriber, System
**Preconditions:** Product catalog has active products

```mermaid
graph TB
    A[Subscriber opens portal/app] --> B[Browse product categories]
    B --> C{Filter by}
    C --> D[Category: Mobile/Broadband/VAS]
    C --> E[Price range]
    C --> F[Contract type: Prepaid/Postpaid]
    D & E & F --> G[View product list]
    G --> H[Select product]
    H --> I[View product details]
    I --> J{Action}
    J --> K[Add to cart]
    J --> L[Compare with current plan]
    J --> M[Save for later]
```

---

### UC-07: Place New Subscription Order

**Actors:** Customer Service Agent, Subscriber (self-service), System
**Preconditions:** Customer exists, product is active, resources available

```mermaid
stateDiagram-v2
    [*] --> Acknowledged: Order submitted
    Acknowledged --> Validating: Validate products + customer
    Validating --> ResourceReservation: Validation passed
    Validating --> Failed: Validation failed
    ResourceReservation --> BillingSetup: Resources reserved
    ResourceReservation --> Failed: No resources
    BillingSetup --> Provisioning: Billing configured
    Provisioning --> Completed: All services activated
    Provisioning --> PartiallyCompleted: Some services activated
    PartiallyCompleted --> FalloutQueue: Manual intervention
    FalloutQueue --> Provisioning: Retry
    Completed --> [*]
    Failed --> [*]
```

---

### UC-10: Real-Time Voice Call Rating

**Actors:** Subscriber (caller), Network, Charging Engine
**Preconditions:** Subscriber has active subscription; balance > 0 (prepaid) or credit available (postpaid)

```mermaid
sequenceDiagram
    participant NET as Network (MSC)
    participant CE as Charging Engine (OCS)
    participant RD as Redis (Balance Cache)
    participant PG as PostgreSQL

    NET->>CE: DIAMETER CCR-I (Initial)
    CE->>RD: Get balance for MSISDN
    RD-->>CE: Balance = 5000 cents
    CE->>CE: Lookup tariff: $0.05/min to destination
    CE->>CE: Reserve 300 seconds worth = 1500 cents
    CE->>RD: Deduct reservation
    CE-->>NET: DIAMETER CCA-I (Granted: 300 sec)

    Note over NET: Call in progress...

    NET->>CE: DIAMETER CCR-U (Update at 300 sec)
    CE->>CE: Reserve additional 300 sec
    CE->>RD: Deduct additional reservation
    CE-->>NET: DIAMETER CCA-U (Granted: 300 sec)

    Note over NET: Call ends at 420 sec

    NET->>CE: DIAMETER CCR-T (Terminate)
    CE->>CE: Calculate actual: 420 sec = 2100 cents
    CE->>RD: Commit 2100, refund unused reservation
    CE->>PG: Write CDR
    CE-->>NET: DIAMETER CCA-T (Success)
```

**Postconditions:** Balance decremented by actual usage, CDR recorded, unused reservation refunded.

---

### UC-12: Prepaid Balance Top-Up

**Actors:** Subscriber, System
**Preconditions:** Active subscriber, valid payment method

```mermaid
sequenceDiagram
    participant SUB as Subscriber
    participant CH as Channel (USSD/Web/App)
    participant BIL as Billing Service
    participant PAY as Payment Gateway
    participant RD as Redis
    participant PG as PostgreSQL

    SUB->>CH: Request top-up ($10)
    CH->>BIL: Initiate top-up
    BIL->>PAY: Charge payment method
    PAY-->>BIL: Payment confirmed
    BIL->>PG: Record balance_operation (credit, 1000 cents)
    BIL->>RD: Update balance cache
    BIL-->>CH: Top-up successful
    CH-->>SUB: SMS: "Your balance is now $15.00"
```

---

### UC-13: Auto-Recharge Trigger

**Actors:** System
**Preconditions:** Subscriber has auto-recharge enabled with threshold and amount configured

```mermaid
graph TB
    A[Balance drops below threshold] --> B{Auto-recharge enabled?}
    B -->|Yes| C[Charge saved payment method]
    B -->|No| D[Send low balance alert]
    C --> E{Payment successful?}
    E -->|Yes| F[Credit balance]
    E -->|No| G[Send payment failure notification]
    F --> H[Send recharge confirmation SMS]
    G --> I[Retry in 1 hour, max 3 attempts]
```

---

### UC-14: Postpaid Invoice Generation

**Actors:** System (scheduler), Billing Service
**Preconditions:** Billing cycle date reached

```mermaid
sequenceDiagram
    participant SCHED as Scheduler
    participant BIL as Billing Service
    participant PG as PostgreSQL
    participant CH as ClickHouse
    participant NOTIFY as Notification

    SCHED->>BIL: Trigger billing cycle for January
    BIL->>PG: Get all postpaid customers in cycle
    loop For each customer
        BIL->>PG: Get subscriptions + recurring charges
        BIL->>CH: Get rated CDRs for billing period
        BIL->>BIL: Calculate: subscriptions + usage + one-time + adjustments
        BIL->>BIL: Apply discounts (loyalty, promotional)
        BIL->>BIL: Calculate taxes (VAT/GST per jurisdiction)
        BIL->>PG: Create invoice + line items
        BIL->>NOTIFY: Send invoice (email + SMS link)
    end
    BIL-->>SCHED: Billing cycle complete: 50,000 invoices
```

---

### UC-15: Bill Shock Prevention Alert

**Actors:** System, Subscriber
**Preconditions:** Postpaid subscriber with spend threshold configured

```mermaid
graph TB
    A[Usage event processed] --> B[Check accumulated spend this cycle]
    B --> C{Spend > 80% of limit?}
    C -->|No| D[Continue normal processing]
    C -->|Yes| E{Spend > 100% of limit?}
    E -->|No| F[Send 80% warning SMS]
    E -->|Yes| G{Hard limit enabled?}
    G -->|Yes| H[Bar outgoing calls except emergency]
    G -->|No| I[Send 100% exceeded notification]
    H --> J[Send service restriction SMS]
```

---

### UC-16: Dunning Escalation

**Actors:** System, Collections Team
**Preconditions:** Invoice past due date

```mermaid
graph TB
    A[Invoice due date passed] --> B[Day 7: Send reminder SMS/email]
    B --> C{Payment received?}
    C -->|Yes| D[Clear dunning status]
    C -->|No| E[Day 14: Send warning letter]
    E --> F{Payment received?}
    F -->|Yes| D
    F -->|No| G[Day 30: Bar outgoing calls]
    G --> H{Payment received?}
    H -->|Yes| I[Unbar + clear dunning]
    H -->|No| J[Day 45: Full suspension]
    J --> K{Payment received?}
    K -->|Yes| L[Reactivate + reconnection fee]
    K -->|No| M[Day 90: Terminate + write off]
    M --> N[Send to collections agency]
```

---

### UC-18: CDR Collection and Normalization

**Actors:** Network Elements, Mediation Service
**Preconditions:** Network generating CDRs

```mermaid
graph LR
    subgraph "Network Sources"
        MSC["MSC (Voice CDR)"]
        SGSN["SGSN/PGW (Data CDR)"]
        SMSC["SMSC (SMS CDR)"]
        MMSC["MMSC (MMS CDR)"]
        VAS_P["VAS Platform (Content CDR)"]
    end

    subgraph "Mediation Pipeline"
        COLLECT["1. Collect<br/>(SFTP/Stream)"]
        PARSE["2. Parse<br/>(ASN.1/CSV/XML)"]
        NORM["3. Normalize<br/>(Unified Schema)"]
        DEDUP["4. Deduplicate"]
        CORR["5. Correlate<br/>(Partial CDRs)"]
        ENRICH["6. Enrich<br/>(Subscriber lookup)"]
        VALIDATE["7. Validate"]
    end

    subgraph "Outputs"
        RATING["Rating Engine"]
        ANALYTICS["ClickHouse Analytics"]
        RA["Revenue Assurance"]
    end

    MSC & SGSN & SMSC & MMSC & VAS_P --> COLLECT
    COLLECT --> PARSE --> NORM --> DEDUP --> CORR --> ENRICH --> VALIDATE
    VALIDATE --> RATING & ANALYTICS & RA
```

---

### UC-19: SIM Swap (Physical to eSIM)

**Actors:** Subscriber, Customer Service Agent, Provisioning Service
**Preconditions:** Subscriber has active SIM, device supports eSIM

```mermaid
sequenceDiagram
    participant SUB as Subscriber
    participant AGENT as Agent
    participant CM as Customer Mgmt
    participant PROV as Provisioning
    participant SMDP as SM-DP+ (eSIM)
    participant HLR as HLR/HSS

    SUB->>AGENT: Request SIM swap to eSIM
    AGENT->>CM: Verify customer identity (KYC re-check)
    CM-->>AGENT: Identity verified
    AGENT->>PROV: Initiate SIM swap
    PROV->>HLR: Deactivate old ICCID
    PROV->>SMDP: Generate eSIM profile
    SMDP-->>PROV: eSIM profile + activation code (QR)
    PROV->>HLR: Map MSISDN to new IMSI
    PROV->>PROV: Update SIM inventory (old=blocked, new=active)
    PROV-->>AGENT: SIM swap complete
    AGENT-->>SUB: Scan QR code to download eSIM profile
```

---

### UC-20: Number Porting (Port-In)

**Actors:** Subscriber, Receiving Operator (us), Donor Operator, NPC
**Preconditions:** Subscriber has valid account with donor; no outstanding debt

```mermaid
sequenceDiagram
    participant SUB as Subscriber
    participant REC as Receiving Operator (BSS)
    participant NPC as Number Portability Clearinghouse
    participant DON as Donor Operator

    SUB->>REC: Request port-in of +234-XXX-XXXX
    REC->>NPC: Submit porting request
    NPC->>DON: Validate eligibility
    DON-->>NPC: Eligible (no debt, no lock)
    NPC-->>REC: Porting approved, scheduled for D+2
    Note over REC,DON: Porting window (2:00 AM - 6:00 AM)
    NPC->>DON: Execute port-out
    DON->>DON: Release number from HLR
    NPC->>REC: Execute port-in
    REC->>REC: Provision number on HLR/HSS
    REC->>REC: Update number inventory (status=active)
    REC-->>SUB: Porting complete, welcome to new network
```

---

### UC-23: MVNO Partner Onboarding

**Actors:** MVNO Partner, Partner Manager, System

```mermaid
sequenceDiagram
    participant MVNO as MVNO Partner
    participant PM as Partner Manager
    participant PS as Partner Service
    participant RI as Resource Inventory
    participant PC as Product Catalog

    MVNO->>PM: Submit partnership application
    PM->>PS: Create partner record
    PS->>PS: KYC/due diligence check
    PS-->>PM: KYC approved
    PM->>PS: Define revenue share agreement
    PM->>RI: Allocate MSISDN block (10,000 numbers)
    PM->>RI: Allocate SIM batch (10,000 SIMs)
    PM->>PC: Configure MVNO product catalog
    PM->>PS: Activate partner portal access
    PS-->>MVNO: Welcome: portal URL + API keys
```

---

### UC-25: USSD Balance Check

**Actors:** Subscriber (feature phone), USSD Gateway

```mermaid
sequenceDiagram
    participant SUB as Subscriber
    participant NET as Network (USSD GW)
    participant USSD as USSD/IVR Service
    participant BIL as Billing Service
    participant RD as Redis

    SUB->>NET: Dial *123#
    NET->>USSD: USSD Begin (MSISDN, shortcode=*123#)
    USSD->>USSD: Load menu: "1.Balance 2.TopUp 3.Plans 4.Help"
    USSD-->>NET: Display menu
    NET-->>SUB: Show menu
    SUB->>NET: Select "1"
    NET->>USSD: USSD Continue (input=1)
    USSD->>BIL: Get balance for MSISDN
    BIL->>RD: Lookup balance
    RD-->>BIL: 5000 cents
    BIL-->>USSD: Balance: $50.00
    USSD-->>NET: "Your balance is $50.00\nData: 2.5GB remaining"
    NET-->>SUB: Display balance
```

---

### UC-26: SIM Box Fraud Detection

**Actors:** System (ML model), Revenue Assurance Analyst

```mermaid
graph TB
    A[CDR Stream] --> B[Feature Extraction]
    B --> C[ML Model Inference]
    C --> D{Fraud Score > 0.85?}
    D -->|Yes| E[Create fraud_alert]
    D -->|No| F[Log for batch analysis]
    E --> G[Auto-bar suspected SIMs]
    E --> H[Alert RA Analyst]
    H --> I{Analyst Review}
    I -->|Confirmed| J[Block SIMs permanently]
    I -->|Dismissed| K[Unbar SIMs + whitelist]

    subgraph "SIM Box Indicators"
        IND1["High outgoing call ratio (>95%)"]
        IND2["Multiple SIMs same IMEI"]
        IND3["No SMS/data usage"]
        IND4["Fixed call durations"]
        IND5["Single cell_id concentration"]
    end
```

---

### UC-27: Smart Meter Reading Collection

**Actors:** Smart Meter, AMI Head-End, Meter Management Service

```mermaid
sequenceDiagram
    participant METER as Smart Meter
    participant AMI as AMI Head-End
    participant MM as Meter Management
    participant PG as PostgreSQL
    participant TS as Tariff Service

    Note over METER: Every 15 minutes
    METER->>AMI: DLMS/COSEM push (reading: 1234.56 kWh)
    AMI->>MM: Forward meter reading
    MM->>MM: Validate quality (within expected range?)
    MM->>PG: Insert meter_reading record
    MM->>TS: Calculate charge for delta consumption
    TS->>TS: Apply time-of-use tariff (peak rate: $0.25/kWh)
    TS-->>MM: Charge: $2.50
    MM->>MM: Update running bill or deduct prepaid balance
```

---

### UC-28: Prepaid Electricity STS Token Purchase

**Actors:** Customer, USSD/Web Portal, Tariff Service

```mermaid
sequenceDiagram
    participant CUST as Customer
    participant CH as USSD / Web
    participant TS as Tariff Service
    participant PAY as Payment Gateway
    participant PG as PostgreSQL

    CUST->>CH: Request prepaid token ($20 for meter 12345)
    CH->>TS: Calculate kWh for $20 at current tariff
    TS-->>CH: 80 kWh at $0.25/kWh
    CH->>PAY: Process payment ($20)
    PAY-->>CH: Payment confirmed
    CH->>TS: Generate STS token (IEC 62055-41)
    TS->>TS: Encode: meter_number + kWh + key
    TS->>PG: Store sts_token record
    TS-->>CH: Token: 1234 5678 9012 3456 7890
    CH-->>CUST: "Enter token on meter: 1234 5678 9012 3456 7890 (80 kWh)"
```

---

## 4. Use Case Traceability Matrix

| Use Case | TMF API | Service(s) | Database Table(s) |
|----------|---------|------------|-------------------|
| UC-01 | TMF629 | customer-management, provisioning, billing | customers, sim_inventory, number_inventory, balances |
| UC-07 | TMF622 | order-management, product-catalog | orders, order_items, products |
| UC-10 | TMF678 | charging-engine | balances, balance_operations, cdrs |
| UC-12 | TMF678 | billing-rating | balances, balance_operations, payments |
| UC-14 | TMF678 | billing-rating | invoices, invoice_line_items, cdrs |
| UC-18 | - | mediation | cdrs |
| UC-19 | TMF641 | provisioning | sim_inventory |
| UC-20 | TMF641 | provisioning | number_inventory |
| UC-23 | TMF668 | partner-service | partners, revenue_share_agreements |
| UC-25 | - | ussd-ivr-gateway, billing | ussd_sessions, balances |
| UC-26 | - | revenue-assurance | fraud_alerts, cdrs |
| UC-27 | - | meter-management | meters, meter_readings |
| UC-28 | - | tariff-service | sts_tokens, meters, utility_tariffs |
