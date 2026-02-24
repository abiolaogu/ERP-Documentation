# ERP-SCM Workflow Diagrams

## 1. Overview

This document captures the key business workflows in ERP-SCM as visual diagrams, showing the sequence of actions, decision points, and system interactions for each major process.

---

## 2. Procure-to-Pay (P2P) Workflow

```mermaid
flowchart TD
    START(("Start")) --> REQ["Create Purchase<br/>Requisition"]
    REQ --> APPROVE_REQ{"Approved?"}
    APPROVE_REQ -->|No| REVISE["Revise & Resubmit"]
    REVISE --> APPROVE_REQ
    APPROVE_REQ -->|Yes| RFQ_NEEDED{"RFQ Needed?"}
    RFQ_NEEDED -->|Yes| RFQ["Create RFQ"]
    RFQ --> COLLECT["Collect<br/>Supplier Bids"]
    COLLECT --> AI_EVAL["AI Vendor<br/>Selection"]
    AI_EVAL --> SELECT["Select Winner"]
    RFQ_NEEDED -->|No| SELECT_SUPPLIER["Select Supplier"]
    SELECT --> PO["Create<br/>Purchase Order"]
    SELECT_SUPPLIER --> PO
    PO --> APPROVE_PO{"PO Approved?"}
    APPROVE_PO -->|No| REVISE_PO["Revise PO"]
    REVISE_PO --> APPROVE_PO
    APPROVE_PO -->|Yes| SEND["Send to Supplier"]
    SEND --> ACK{"Supplier<br/>Acknowledges?"}
    ACK -->|No| FOLLOWUP["Follow Up"]
    FOLLOWUP --> ACK
    ACK -->|Yes| RECEIVE["Receive Goods"]
    RECEIVE --> INSPECT{"Inspection<br/>Required?"}
    INSPECT -->|Yes| QC["Quality<br/>Inspection"]
    QC --> QC_PASS{"Passed?"}
    QC_PASS -->|No| NCR["Create NCR"]
    NCR --> DISPOSITION["Determine<br/>Disposition"]
    QC_PASS -->|Yes| STOCK["Put to Stock"]
    INSPECT -->|No| STOCK
    STOCK --> INVOICE["Receive Invoice"]
    INVOICE --> MATCH["3-Way Match"]
    MATCH --> MATCH_OK{"Match OK?"}
    MATCH_OK -->|No| EXCEPTION["Exception<br/>Processing"]
    EXCEPTION --> MATCH
    MATCH_OK -->|Yes| PAY["Authorize<br/>Payment"]
    PAY --> END(("End"))
```

---

## 3. Order-to-Cash (O2C) Workflow

```mermaid
flowchart TD
    START(("Start")) --> ORDER["Sales Order<br/>Received"]
    ORDER --> ATP{"Stock<br/>Available?"}
    ATP -->|No| BACKORDER["Create<br/>Backorder"]
    BACKORDER --> MRP_TRIGGER["Trigger MRP"]
    ATP -->|Yes| ALLOCATE["Allocate<br/>Inventory"]
    ALLOCATE --> WAVE["Create<br/>Pick Wave"]
    WAVE --> PICK["Execute Picks"]
    PICK --> VERIFY{"All Items<br/>Picked?"}
    VERIFY -->|No| SHORT["Short Pick<br/>Resolution"]
    SHORT --> PICK
    VERIFY -->|Yes| PACK["Pack Order"]
    PACK --> LABEL["Generate<br/>Shipping Label"]
    LABEL --> CARRIER["Select Carrier<br/>(Rate Shopping)"]
    CARRIER --> SHIP["Ship Order"]
    SHIP --> TRACK["Track Shipment"]
    TRACK --> DELIVER["Delivery<br/>Confirmed"]
    DELIVER --> POD["Proof of<br/>Delivery"]
    POD --> INVOICE_OUT["Generate<br/>Invoice"]
    INVOICE_OUT --> COLLECT["Collect<br/>Payment"]
    COLLECT --> END(("End"))
```

---

## 4. Plan-to-Produce Workflow

```mermaid
flowchart TD
    START(("Start")) --> FORECAST["AI Demand<br/>Forecast"]
    FORECAST --> CONSENSUS["Consensus<br/>Planning"]
    CONSENSUS --> MRP["Run MRP"]
    MRP --> PLAN_PO["Planned Purchase<br/>Orders"]
    MRP --> PLAN_PROD["Planned Production<br/>Orders"]
    PLAN_PO --> FIRM_PO["Firm PO &<br/>Send to Supplier"]
    PLAN_PROD --> BOM_CHECK["BOM & Material<br/>Availability Check"]
    BOM_CHECK --> MATERIAL_OK{"Materials<br/>Available?"}
    MATERIAL_OK -->|No| PROCURE["Procure<br/>Materials"]
    PROCURE --> MATERIAL_OK
    MATERIAL_OK -->|Yes| SCHEDULE["Schedule<br/>Production"]
    SCHEDULE --> RELEASE["Release to<br/>Shop Floor"]
    RELEASE --> SETUP["Machine Setup"]
    SETUP --> PRODUCE["Execute<br/>Production"]
    PRODUCE --> REPORT["Report Output<br/>(qty, scrap, time)"]
    REPORT --> QC{"In-Process<br/>Inspection?"}
    QC -->|Yes| INSPECT["Inspect"]
    INSPECT --> PASS{"Passed?"}
    PASS -->|No| REWORK["Rework or Scrap"]
    REWORK --> PRODUCE
    PASS -->|Yes| NEXT_OP{"More<br/>Operations?"}
    QC -->|No| NEXT_OP
    NEXT_OP -->|Yes| SETUP
    NEXT_OP -->|No| FINAL_QC["Final<br/>Inspection"]
    FINAL_QC --> FG["Transfer to<br/>Finished Goods"]
    FG --> END(("End"))
```

---

## 5. Warehouse Receiving & Putaway Workflow

```mermaid
flowchart TD
    START(("Start")) --> ARRIVAL["Truck Arrives<br/>at Dock"]
    ARRIVAL --> SCAN_PO["Scan PO /<br/>Enter PO Number"]
    SCAN_PO --> UNLOAD["Unload &<br/>Count Items"]
    UNLOAD --> DAMAGE{"Damage<br/>Detected?"}
    DAMAGE -->|Yes| LOG_DAMAGE["Log Damage<br/>& Photos"]
    LOG_DAMAGE --> PARTIAL["Partial Receipt"]
    DAMAGE -->|No| FULL["Full Receipt"]
    PARTIAL --> RECEIPT["Create Goods<br/>Receipt"]
    FULL --> RECEIPT
    RECEIPT --> QP_EXISTS{"Quality Plan<br/>Exists?"}
    QP_EXISTS -->|Yes| HOLD["Hold in QC<br/>Staging"]
    HOLD --> INSPECT["Incoming<br/>Inspection"]
    INSPECT --> ACCEPT{"Accepted?"}
    ACCEPT -->|No| REJECT["Reject &<br/>Return to Supplier"]
    ACCEPT -->|Yes| SUGGEST["AI Putaway<br/>Suggestion"]
    QP_EXISTS -->|No| SUGGEST
    SUGGEST --> OPTIMIZE["Optimize by:<br/>Velocity, Zone, Weight"]
    OPTIMIZE --> MOVE["Move to<br/>Designated Bin"]
    MOVE --> SCAN_BIN["Scan Bin<br/>Barcode"]
    SCAN_BIN --> CONFIRM["Confirm<br/>Putaway"]
    CONFIRM --> UPDATE["Update Inventory<br/>& Bin Records"]
    UPDATE --> END(("End"))
```

---

## 6. AI Anomaly Detection Workflow

```mermaid
flowchart TD
    START(("Scheduled<br/>Every 15 min")) --> LOAD["Load Current<br/>Data Snapshot"]

    LOAD --> INV_SCAN["Inventory<br/>Scan"]
    INV_SCAN --> INV_CHECK{"Below<br/>Reorder?"}
    INV_CHECK -->|Yes| INV_ALERT["Low Stock Alert"]
    INV_CHECK -->|No| INV_OVER{"Above<br/>5x Reorder Qty?"}
    INV_OVER -->|Yes| OVER_ALERT["Overstock Alert"]

    LOAD --> DEM_SCAN["Demand<br/>Scan"]
    DEM_SCAN --> ZSCORE["Calculate<br/>Z-Scores"]
    ZSCORE --> Z_CHECK{"|Z| > 2.5?"}
    Z_CHECK -->|Yes, Positive| SPIKE_ALERT["Demand Spike Alert"]
    Z_CHECK -->|Yes, Negative| DROP_ALERT["Demand Drop Alert"]

    LOAD --> DEL_SCAN["Delivery<br/>Scan"]
    DEL_SCAN --> DEL_CHECK{"Past Expected<br/>Delivery?"}
    DEL_CHECK -->|Yes| LATE_ALERT["Late Delivery Alert"]

    LOAD --> SUP_SCAN["Supplier<br/>Scan"]
    SUP_SCAN --> IF["Isolation Forest<br/>Analysis"]
    IF --> IF_CHECK{"Anomaly<br/>Detected?"}
    IF_CHECK -->|Yes| SUP_ALERT["Supplier Anomaly<br/>Alert"]

    INV_ALERT & OVER_ALERT & SPIKE_ALERT & DROP_ALERT & LATE_ALERT & SUP_ALERT --> DEDUP["De-duplicate<br/>vs Existing Alerts"]
    DEDUP --> PERSIST["Persist New<br/>Alerts"]
    PERSIST --> CLASSIFY["Classify Severity<br/>Critical/High/Medium/Low"]
    CLASSIFY --> NOTIFY["Publish to<br/>WebSocket & Email"]
    NOTIFY --> DASHBOARD["Update<br/>Dashboard"]
    DASHBOARD --> END(("End"))
```

---

## 7. Supplier Onboarding Workflow

```mermaid
flowchart TD
    START(("Start")) --> REGISTER["Supplier<br/>Self-Registration"]
    REGISTER --> VERIFY["Procurement<br/>Reviews Application"]
    VERIFY --> DOCS{"Required Docs<br/>Submitted?"}
    DOCS -->|No| REQUEST["Request<br/>Documents"]
    REQUEST --> DOCS
    DOCS -->|Yes| ASSESS["Assess Supplier<br/>Capabilities"]
    ASSESS --> RISK["AI Risk<br/>Assessment"]
    RISK --> APPROVE{"Approved?"}
    APPROVE -->|No| REJECT["Reject with<br/>Feedback"]
    APPROVE -->|Yes| SETUP["Setup Supplier<br/>Profile"]
    SETUP --> PORTAL["Create Portal<br/>Access"]
    PORTAL --> CATALOG["Setup Product<br/>Catalog & Pricing"]
    CATALOG --> QUALIFY["Add to Qualified<br/>Supplier List"]
    QUALIFY --> WELCOME["Send Welcome<br/>& Training"]
    WELCOME --> END(("End"))
```

---

## 8. Fleet Trip Execution Workflow

```mermaid
flowchart TD
    START(("Start")) --> ASSIGN["Assign Vehicle<br/>& Driver"]
    ASSIGN --> PRECHECK["Pre-Trip<br/>Inspection"]
    PRECHECK --> PASS{"Passed?"}
    PASS -->|No| MAINT["Send to<br/>Maintenance"]
    MAINT --> REASSIGN["Reassign<br/>Vehicle"]
    REASSIGN --> PRECHECK
    PASS -->|Yes| DEPART["Depart"]
    DEPART --> GPS["GPS Tracking<br/>Active"]
    GPS --> MONITOR["Monitor Driver<br/>Behavior"]
    MONITOR --> ALERT{"Behavior<br/>Alert?"}
    ALERT -->|Yes| FLAG["Flag &<br/>Notify Manager"]
    ALERT -->|No| STOP{"Delivery<br/>Stop?"}
    STOP -->|Yes| DELIVER["Complete<br/>Delivery"]
    DELIVER --> POD["Capture<br/>POD"]
    POD --> MORE{"More<br/>Stops?"}
    MORE -->|Yes| GPS
    MORE -->|No| RETURN["Return to<br/>Base"]
    STOP -->|No| GPS
    RETURN --> LOG["Log Trip<br/>Summary"]
    LOG --> FUEL["Record Fuel<br/>Consumption"]
    FUEL --> SCORE["Calculate<br/>Driver Score"]
    SCORE --> END(("End"))
```

---

## 9. Quality NCR-to-CAPA Workflow

```mermaid
flowchart TD
    START(("Non-Conformance<br/>Identified")) --> CREATE["Create NCR"]
    CREATE --> CLASSIFY["Classify Severity<br/>Critical/Major/Minor"]
    CLASSIFY --> CONTAIN["Containment<br/>Action"]
    CONTAIN --> QUARANTINE["Quarantine<br/>Affected Product"]
    QUARANTINE --> INVESTIGATE["Root Cause<br/>Investigation"]
    INVESTIGATE --> METHOD["Investigation<br/>Method"]
    METHOD --> FIVEWHY["5-Why Analysis"]
    METHOD --> FISHBONE["Ishikawa Diagram"]
    FIVEWHY & FISHBONE --> ROOT_CAUSE["Document<br/>Root Cause"]
    ROOT_CAUSE --> DISPOSITION["Determine<br/>Disposition"]
    DISPOSITION --> D1["Rework"]
    DISPOSITION --> D2["Scrap"]
    DISPOSITION --> D3["Use As-Is"]
    DISPOSITION --> D4["Return to Vendor"]
    D1 & D2 & D3 & D4 --> CAPA["Create CAPA"]
    CAPA --> CORRECTIVE["Corrective<br/>Action"]
    CAPA --> PREVENTIVE["Preventive<br/>Action"]
    CORRECTIVE & PREVENTIVE --> IMPLEMENT["Implement<br/>Actions"]
    IMPLEMENT --> VERIFY["Verify<br/>Effectiveness"]
    VERIFY --> EFFECTIVE{"Effective?"}
    EFFECTIVE -->|No| REVISE["Revise<br/>Actions"]
    REVISE --> IMPLEMENT
    EFFECTIVE -->|Yes| CLOSE["Close<br/>NCR & CAPA"]
    CLOSE --> END(("End"))
```
