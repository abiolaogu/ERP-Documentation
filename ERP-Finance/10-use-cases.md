# ERP-Finance Use Cases

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | Use Cases |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

## UC-001: Month-End Close

### Overview
The month-end close process ensures all financial transactions for a period are complete, accurate, and properly reported.

### Actors
- Controller (primary)
- AP Clerk, AR Manager, Asset Manager (supporting)

### Flow

```mermaid
flowchart TD
    Start["Start Month-End Close"] --> PreCheck["Pre-Close Checklist"]
    PreCheck --> AP["Verify all AP invoices<br/>are entered and matched"]
    PreCheck --> AR["Verify all AR invoices<br/>issued and cash applied"]
    PreCheck --> ASSET["Run depreciation<br/>for the period"]
    PreCheck --> EXP["Process all pending<br/>expense claims"]
    PreCheck --> BANK["Complete bank<br/>reconciliation"]

    AP --> Accrue["Post accruals and<br/>prepayment adjustments"]
    AR --> Accrue
    ASSET --> Accrue
    EXP --> Accrue
    BANK --> Accrue

    Accrue --> FX["Revalue foreign<br/>currency balances"]
    FX --> INTERCO["Process intercompany<br/>eliminations"]
    INTERCO --> TB["Generate trial balance"]
    TB --> Review{"Review<br/>balanced?"}
    Review -->|No| Adjust["Post adjusting<br/>journal entries"]
    Adjust --> TB
    Review -->|Yes| Statements["Generate financial<br/>statements"]
    Statements --> Close["Close accounting period"]
    Close --> Lock["Lock period against<br/>further postings"]
    Lock --> Report["Distribute reports<br/>to stakeholders"]
    Report --> Done["Month-End Complete"]
```

### Preconditions
- All sub-ledgers (AP, AR, Asset) have completed their period processing
- Bank statements received for all accounts
- FX rates updated to period-end rates

### Success Criteria
- Trial balance balances (total debits = total credits)
- All reconciling items documented
- Financial statements generated (Income Statement, Balance Sheet, Cash Flow)
- Period locked with audit trail

---

## UC-002: Invoice Processing (AP)

### Overview
End-to-end processing of a vendor invoice from receipt through payment.

### Actors
- AP Clerk (primary), Approver (secondary)

### Flow

```mermaid
sequenceDiagram
    participant V as Vendor
    participant AP as AP Clerk
    participant OCR as OCR AI Engine
    participant SYS as ERP-Finance
    participant APPR as Approver
    participant PAY as Payment Service
    participant GL as General Ledger

    V->>AP: Send invoice (email/upload)
    AP->>OCR: Upload invoice PDF
    OCR->>AP: Return extracted fields<br/>(vendor, amount, items, tax)
    AP->>SYS: Review & confirm extraction
    SYS->>SYS: 3-way match<br/>(PO + Receipt + Invoice)

    alt Match successful
        SYS->>APPR: Route for approval
        APPR->>SYS: Approve invoice
        SYS->>GL: Post AP journal entry<br/>(DR Expense, CR AP)
        SYS->>PAY: Add to payment run queue
        PAY->>V: Execute payment
        PAY->>GL: Post payment entry<br/>(DR AP, CR Cash)
    else Match exception
        SYS->>AP: Flag discrepancy
        AP->>SYS: Resolve or escalate
    end
```

### Business Rules
- Invoices over $10,000 require manager approval
- Invoices over $50,000 require VP Finance approval
- 3-way match tolerance: 2% on quantity, 5% on unit price
- Duplicate invoice detection based on vendor + invoice number + date

---

## UC-003: Payment Collection (AR)

### Overview
Collecting payment on outstanding customer invoices through multiple channels.

### Actors
- AR Manager (primary), Customer (external)

### Flow

```mermaid
flowchart TD
    Start["AR Invoice Issued"] --> Send["Send invoice to customer<br/>via email/portal"]
    Send --> Wait["Wait for payment"]
    Wait --> Check{"Payment<br/>received?"}

    Check -->|Yes, online| Auto["Auto-apply via<br/>Cash Application AI"]
    Check -->|Yes, bank transfer| Bank["Bank reconciliation<br/>matches deposit"]
    Check -->|No, past due| Dun["Trigger dunning<br/>sequence"]

    Auto --> Match{"Amount<br/>matches?"}
    Bank --> Match

    Match -->|Exact match| Apply["Apply payment<br/>to invoice"]
    Match -->|Partial| Partial["Apply partial payment<br/>leave remainder open"]
    Match -->|Overpayment| Over["Apply to invoice<br/>credit remaining to account"]

    Apply --> GL["Post to GL<br/>DR Cash, CR AR"]
    Partial --> GL
    Over --> GL

    Dun --> D1["Day 7: Friendly reminder"]
    D1 --> D2["Day 14: Second reminder"]
    D2 --> D3["Day 30: Final warning"]
    D3 --> D4["Day 60: Escalate to collections"]

    GL --> Close["Invoice closed"]
```

---

## UC-004: Expense Claim

### Overview
Employee submits an expense claim with receipts through approval to reimbursement.

### Actors
- Employee (primary), Manager (approver), Finance Team (processor)

### Flow

```mermaid
sequenceDiagram
    participant E as Employee
    participant OCR as Receipt OCR
    participant SYS as ERP-Finance
    participant MGR as Manager
    participant FIN as Finance Team
    participant PAY as Payroll/Payment

    E->>OCR: Upload receipt photo
    OCR->>E: Extract: vendor, date, amount, category
    E->>SYS: Submit expense claim<br/>(trip purpose, project code)
    SYS->>SYS: Validate against policy<br/>(per-diem limits, category rules)

    alt Within policy
        SYS->>MGR: Route for approval
        MGR->>SYS: Approve claim
        SYS->>FIN: Queue for processing
        FIN->>SYS: Verify and process
        SYS->>PAY: Schedule reimbursement
        PAY->>E: Reimburse to bank/payroll
        SYS->>SYS: Post GL entry<br/>(DR Expense, CR AP/Cash)
    else Policy violation
        SYS->>E: Return with violation details
        E->>SYS: Revise and resubmit
    end
```

### Business Rules
- Per-diem meal limits by city/country
- Hotel rates capped per policy tier
- Mileage reimbursement at standard rate
- Receipts required for expenses over $25
- Corporate card transactions auto-imported

---

## UC-005: Asset Depreciation

### Overview
Calculate and record depreciation for fixed assets using the appropriate method.

### Actors
- Asset Manager (primary), Controller (reviewer)

### Flow

```mermaid
flowchart TD
    Start["Depreciation Run Initiated"] --> Select["Select assets for<br/>depreciation period"]
    Select --> Method{"Check depreciation<br/>method per asset"}

    Method -->|Straight-Line| SL["Calculate: (Cost - Salvage) / Life"]
    Method -->|Declining Balance| DB["Calculate: Book Value x Rate"]
    Method -->|Double Declining| DD["Calculate: Book Value x 2/Life"]
    Method -->|Sum of Years| SY["Calculate: Depreciable x Remaining/Sum"]
    Method -->|Units of Production| UP["Calculate: Rate x Units Used"]

    SL --> Record["Create depreciation record"]
    DB --> Record
    DD --> Record
    SY --> Record
    UP --> Record

    Record --> UpdateBV["Update asset book value"]
    UpdateBV --> GL["Post GL journal entry<br/>DR Depreciation Expense<br/>CR Accumulated Depreciation"]
    GL --> AI{"AI analysis<br/>requested?"}
    AI -->|Yes| Optimize["AI recommends optimal<br/>depreciation strategy"]
    AI -->|No| Report["Generate depreciation<br/>report"]
    Optimize --> Report
    Report --> Done["Period Depreciation Complete"]
```

---

## UC-006: Tax Filing

### Overview
Calculate and file tax returns for multiple jurisdictions (VAT/GST/Sales Tax).

### Actors
- Tax Analyst (primary), External Tax Engine (Avalara/Vertex)

### Flow

```mermaid
sequenceDiagram
    participant TAX as Tax Analyst
    participant SYS as ERP-Finance
    participant GL as General Ledger
    participant AVA as Avalara/Vertex
    participant GOV as Tax Authority

    TAX->>SYS: Initiate tax period close
    SYS->>GL: Pull all taxable transactions
    SYS->>AVA: Validate tax calculations
    AVA->>SYS: Return verified amounts
    SYS->>SYS: Reconcile GL tax accounts<br/>vs calculated obligations
    SYS->>TAX: Present tax return draft

    alt Balanced
        TAX->>SYS: Approve return
        SYS->>GOV: File electronically
        SYS->>GL: Post tax liability settlement
    else Discrepancy
        TAX->>SYS: Investigate and adjust
        SYS->>GL: Post adjusting entries
    end
```

---

## UC-007: Bank Reconciliation

### Overview
AI-powered matching of bank statement lines to internal GL transactions.

### Actors
- Treasurer (primary), AI Reconciliation Engine (automated)

### Flow

```mermaid
flowchart TD
    Start["Import Bank Statement"] --> Parse["Parse statement lines<br/>(date, description, amount, reference)"]
    Parse --> AI["AI Matching Engine"]

    AI --> Match["Auto-matched<br/>(high confidence > 95%)"]
    AI --> Suggest["Suggested matches<br/>(confidence 70-95%)"]
    AI --> Unmatched["Unmatched items"]

    Match --> Auto["Auto-apply matches"]
    Suggest --> Review["Treasurer reviews<br/>suggestions"]
    Unmatched --> Manual["Manual investigation"]

    Review -->|Accept| Auto
    Review -->|Reject| Manual

    Manual --> Identify{"Transaction<br/>identified?"}
    Identify -->|Yes| Link["Link to existing transaction"]
    Identify -->|No| Create["Create new GL entry<br/>(bank charge, interest, etc.)"]

    Auto --> Reconciled["Mark as reconciled"]
    Link --> Reconciled
    Create --> Reconciled

    Reconciled --> Report["Generate reconciliation report"]
    Report --> Done["Reconciliation Complete"]
```

---

## UC-008: Budget Approval

### Overview
Create, review, and approve organizational budgets using various methodologies.

### Actors
- Budget Analyst (creator), Department Heads (reviewers), CFO (approver)

### Flow

```mermaid
sequenceDiagram
    participant BA as Budget Analyst
    participant SYS as ERP-Finance
    participant DH as Department Heads
    participant CFO as CFO

    BA->>SYS: Create budget plan<br/>(top-down / bottom-up / zero-based)
    SYS->>SYS: Load prior period actuals<br/>as baseline reference

    loop For each department
        SYS->>DH: Request budget input
        DH->>SYS: Submit departmental budget
        SYS->>SYS: Validate against guidelines<br/>(growth caps, headcount limits)
    end

    BA->>SYS: Consolidate departmental budgets
    SYS->>SYS: Run scenario analysis<br/>(optimistic, realistic, pessimistic)
    BA->>SYS: Finalize budget proposal

    SYS->>CFO: Present consolidated budget<br/>with variance analysis
    CFO->>SYS: Request adjustments
    BA->>SYS: Revise budget
    CFO->>SYS: Approve final budget
    SYS->>SYS: Activate budget<br/>Set up variance monitoring
    SYS->>SYS: Emit budget.approved event
```

### Post-Approval
- Monthly variance analysis automatically generated
- Alert when actual spending exceeds budget by configurable threshold (e.g., 10%)
- Quarterly budget reforecast cycle supported
- Rolling 12-month forecast maintained

---

## Use Case Summary

```mermaid
flowchart TB
    subgraph UseCases["ERP-Finance Use Cases"]
        UC1["UC-001: Month-End Close"]
        UC2["UC-002: Invoice Processing"]
        UC3["UC-003: Payment Collection"]
        UC4["UC-004: Expense Claim"]
        UC5["UC-005: Asset Depreciation"]
        UC6["UC-006: Tax Filing"]
        UC7["UC-007: Bank Reconciliation"]
        UC8["UC-008: Budget Approval"]
    end

    UC2 --> UC1
    UC3 --> UC1
    UC5 --> UC1
    UC7 --> UC1
    UC6 --> UC1
    UC4 --> UC2
    UC3 --> UC7
```
