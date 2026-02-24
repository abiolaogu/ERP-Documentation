# ERP-Commerce -- Workflows Document

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

## 1. Workflow Engine

ERP-Commerce uses Temporal for long-running workflow orchestration. Each workflow is durable, retryable, and auditable.

```mermaid
flowchart TB
    subgraph Temporal["Temporal Workflow Engine"]
        OFW["Order Fulfillment<br/>Workflow"]
        CRW["Credit Decision<br/>Workflow"]
        COL["Collections<br/>Workflow"]
        EDI["EDI Processing<br/>Workflow"]
        SYNC["POS Sync<br/>Workflow"]
        RPL["Replenishment<br/>Workflow"]
        VND["Vendor Onboarding<br/>Workflow"]
    end

    subgraph Triggers
        T1["Order Created Event"]
        T2["Credit Application"]
        T3["Invoice Overdue"]
        T4["EDI Document Received"]
        T5["POS Online Event"]
        T6["Stock Below Threshold"]
        T7["Vendor Registration"]
    end

    T1 --> OFW
    T2 --> CRW
    T3 --> COL
    T4 --> EDI
    T5 --> SYNC
    T6 --> RPL
    T7 --> VND
```

---

## 2. Order Fulfillment Workflow

### 2.1 Complete Flow

```mermaid
flowchart TD
    START(["Order Created"]) --> VALIDATE["Activity: Validate Order<br/>(items, addresses, format)"]
    VALIDATE -->|Valid| CREDIT["Activity: Check Credit<br/>(trade-credit-service)"]
    VALIDATE -->|Invalid| REJECT["Activity: Reject Order<br/>(notify buyer)"]

    CREDIT -->|Approved| POLICY["Activity: Evaluate<br/>Brand Policy"]
    CREDIT -->|Denied| REJECT

    POLICY -->|MOQ Met| SOURCE["Activity: Source<br/>Selection"]
    POLICY -->|Under MOQ| GROUP["Activity: MOQ<br/>Grouping"]
    GROUP -->|Grouped| SOURCE
    GROUP -->|Timeout 24h| REJECT

    SOURCE --> SPLIT{"Multi-Source?"}
    SPLIT -->|Yes| SPLIT_ACT["Activity: Create<br/>Sub-Orders"]
    SPLIT -->|No| ALLOCATE["Activity: Reserve<br/>Inventory"]

    SPLIT_ACT --> ALLOCATE

    ALLOCATE -->|Success| NOTIFY_WH["Activity: Notify<br/>Warehouse"]
    ALLOCATE -->|Failed| BACKORDER["Activity: Create<br/>Backorder"]

    NOTIFY_WH --> PICK["Activity: Wait for<br/>Pick Complete Signal"]
    PICK --> PACK["Activity: Wait for<br/>Pack Complete Signal"]
    PACK --> CARRIER["Activity: Assign<br/>Carrier"]
    CARRIER --> SHIP["Activity: Wait for<br/>Ship Confirm Signal"]
    SHIP --> TRACK["Activity: Monitor<br/>Delivery (GPS)"]
    TRACK --> POD["Activity: Wait for<br/>POD Signal"]
    POD --> INVOICE["Activity: Generate<br/>Invoice"]
    INVOICE --> SETTLE["Activity: Process<br/>Settlement"]
    SETTLE --> COMPLETE(["Order Complete"])

    REJECT --> END(["Order Ended"])
```

### 2.2 Retry and Compensation

| Activity          | Retry Policy           | Compensation Action         |
|-------------------|------------------------|-----------------------------|
| Validate Order    | 3 retries, 5s backoff  | Mark as validation failed   |
| Check Credit      | 3 retries, 10s backoff | Queue for manual review     |
| Reserve Inventory | 5 retries, 5s backoff  | Release any partial reserves|
| Assign Carrier    | 5 retries, 30s backoff | Reassign to backup carrier  |
| Generate Invoice  | 3 retries, 10s backoff | Queue for manual invoicing  |

---

## 3. Credit Decision Workflow

```mermaid
flowchart TD
    START(["Credit Application<br/>Received"]) --> COLLECT["Activity: Collect<br/>Business Data"]
    COLLECT --> INTERNAL["Activity: Pull<br/>Internal History"]
    INTERNAL --> EXTERNAL["Activity: Pull<br/>External Bureau Data"]
    EXTERNAL --> SCORE["Activity: Run<br/>AI Scoring Model"]
    SCORE --> THRESHOLD{"Score<br/>Threshold?"}

    THRESHOLD -->|750+| AUTO_APPROVE["Activity: Auto-Approve<br/>Full Limit, Net 60"]
    THRESHOLD -->|500-749| PARTIAL["Activity: Auto-Approve<br/>Reduced Limit, Net 30"]
    THRESHOLD -->|300-499| MANUAL["Activity: Route to<br/>Credit Officer Queue"]
    THRESHOLD -->|<300| AUTO_DENY["Activity: Auto-Deny<br/>with Reason"]

    MANUAL --> REVIEW["Human Activity:<br/>Credit Officer Review"]
    REVIEW -->|Approve| PARTIAL
    REVIEW -->|Deny| AUTO_DENY

    AUTO_APPROVE --> CREATE["Activity: Create<br/>Credit Account"]
    PARTIAL --> CREATE
    CREATE --> MONITOR["Activity: Start<br/>Monitoring Timer"]
    MONITOR --> COMPLETE(["Credit Active"])

    AUTO_DENY --> NOTIFY["Activity: Notify<br/>Applicant"]
    NOTIFY --> END(["Application Closed"])
```

---

## 4. Collections Workflow

```mermaid
flowchart TD
    START(["Invoice Overdue"]) --> GRACE["Timer: 3-day<br/>Grace Period"]
    GRACE --> R1["Activity: Send<br/>Reminder 1 (SMS+Email)"]
    R1 --> WAIT1["Timer: 7 days"]
    WAIT1 --> CHECK1{"Payment<br/>Received?"}
    CHECK1 -->|Yes| CLOSE(["Case Closed"])
    CHECK1 -->|No| R2["Activity: Send<br/>Reminder 2 (Phone)"]
    R2 --> WAIT2["Timer: 7 days"]
    WAIT2 --> CHECK2{"Payment<br/>Received?"}
    CHECK2 -->|Yes| CLOSE
    CHECK2 -->|No| R3["Activity: Send<br/>Reminder 3 (WhatsApp)"]
    R3 --> WAIT3["Timer: 14 days"]
    WAIT3 --> CHECK3{"Payment<br/>Received?"}
    CHECK3 -->|Yes| CLOSE
    CHECK3 -->|No| HOLD["Activity: Place<br/>Account on Hold"]
    HOLD --> WAIT4["Timer: 30 days"]
    WAIT4 --> CHECK4{"Payment<br/>Received?"}
    CHECK4 -->|Yes| UNHOLD["Activity: Remove<br/>Hold"] --> CLOSE
    CHECK4 -->|No| ESCALATE["Activity: Escalate<br/>to Collections Team"]
    ESCALATE --> WAIT5["Timer: 30 days"]
    WAIT5 --> LEGAL["Human Activity:<br/>Legal Review"]
    LEGAL -->|Recover| CLOSE
    LEGAL -->|Write-off| WRITEOFF["Activity:<br/>Write Off Debt"]
    WRITEOFF --> CLOSE
```

---

## 5. EDI Processing Workflow

```mermaid
flowchart TD
    START(["EDI Document<br/>Received"]) --> PARSE["Activity: Parse Document<br/>(Rust EDI Parser)"]
    PARSE -->|Valid| MAP["Activity: Map to<br/>Internal Format"]
    PARSE -->|Invalid| REJECT["Activity: Send<br/>997 Rejection"]

    MAP --> VALIDATE["Activity: Business<br/>Validation"]
    VALIDATE -->|Pass| CREATE["Activity: Create<br/>Internal Order"]
    VALIDATE -->|Fail| REJECT

    CREATE --> ACK["Activity: Generate<br/>855/ORDRSP Acknowledgment"]
    ACK --> SEND["Activity: Send via<br/>AS2/SFTP"]
    SEND --> MONITOR["Activity: Monitor<br/>Order Progress"]

    MONITOR -->|Shipped| ASN["Activity: Generate<br/>856/DESADV Ship Notice"]
    ASN --> SEND_ASN["Activity: Send ASN"]

    MONITOR -->|Invoiced| INV["Activity: Generate<br/>810/INVOIC Invoice"]
    INV --> SEND_INV["Activity: Send Invoice"]
    SEND_INV --> COMPLETE(["EDI Cycle Complete"])

    REJECT --> ARCHIVE["Activity: Archive<br/>Failed Document"]
    ARCHIVE --> END(["Processing Ended"])
```

---

## 6. Vendor Onboarding Workflow

```mermaid
flowchart TD
    START(["Vendor<br/>Registration"]) --> COLLECT["Activity: Collect<br/>Business Documents"]
    COLLECT --> KYC["Activity: KYC/KYB<br/>Auto-Verification"]
    KYC -->|Pass| REVIEW["Activity: Admin<br/>Review (Optional)"]
    KYC -->|Fail| MANUAL_KYC["Human Activity:<br/>Manual Document Review"]
    KYC -->|Rejected| DENY["Activity: Deny<br/>Application"]

    MANUAL_KYC -->|Approve| REVIEW
    MANUAL_KYC -->|Reject| DENY

    REVIEW --> ACTIVATE["Activity: Activate<br/>Vendor Account"]
    ACTIVATE --> COMMISSION["Activity: Assign<br/>Commission Structure"]
    COMMISSION --> WELCOME["Activity: Send<br/>Welcome Package"]
    WELCOME --> COMPLETE(["Vendor Active"])

    DENY --> NOTIFY["Activity: Notify<br/>with Reason"]
    NOTIFY --> END(["Application Closed"])
```

---

## 7. Demand-Driven Replenishment Workflow

```mermaid
flowchart TD
    START(["Stock Below<br/>Reorder Point"]) --> FORECAST["Activity: Run<br/>Demand Forecast"]
    FORECAST --> CALC["Activity: Calculate<br/>Reorder Quantity"]
    CALC --> SOURCE["Activity: Select<br/>Optimal Supplier"]
    SOURCE --> APPROVAL{"Auto-Approve?<br/>(within budget)"}
    APPROVAL -->|Yes| PO["Activity: Create<br/>Purchase Order"]
    APPROVAL -->|No| MANUAL["Human Activity:<br/>Manager Approval"]
    MANUAL -->|Approve| PO
    MANUAL -->|Reject| END(["Cancelled"])
    PO --> SEND_PO["Activity: Send PO<br/>to Supplier (EDI/API)"]
    SEND_PO --> WAIT["Timer: Wait for<br/>Delivery"]
    WAIT --> RECEIVE["Activity: Receive<br/>& Count Goods"]
    RECEIVE --> UPDATE["Activity: Update<br/>Inventory"]
    UPDATE --> COMPLETE(["Replenishment<br/>Complete"])
```

---

## 8. POS Sync Workflow

```mermaid
flowchart TD
    START(["POS Terminal<br/>Online"]) --> FETCH["Activity: Fetch<br/>Queued Transactions"]
    FETCH --> BATCH["Activity: Create<br/>Batch (max 50)"]
    BATCH --> UPLOAD["Activity: Upload<br/>to pos-service"]
    UPLOAD --> CONFLICT{"Conflicts<br/>Detected?"}
    CONFLICT -->|No| CONFIRM["Activity: Mark<br/>Synced"]
    CONFLICT -->|Yes| RESOLVE["Activity: Conflict<br/>Resolution (LWW)"]
    RESOLVE --> CONFIRM
    CONFIRM --> MORE{"More in<br/>Queue?"}
    MORE -->|Yes| FETCH
    MORE -->|No| DOWNLOAD["Activity: Download<br/>Updates (catalog, prices)"]
    DOWNLOAD --> COMPLETE(["Sync Complete"])
```
