# ERP-Commerce -- High-Level Design (HLD)

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

## 1. System Decomposition

### 1.1 Domain Boundaries

```mermaid
flowchart TB
    subgraph Commerce_Core["Commerce Core Domain"]
        CAT["Catalog &<br/>PIM"]
        ORD["Order<br/>Orchestration"]
        PRC["Pricing<br/>Engine"]
        INV["Inventory<br/>Management"]
    end

    subgraph Trade_Finance["Trade Finance Domain"]
        TCR["Trade<br/>Credit"]
        COL["Collections &<br/>Aging"]
        FIN["Trade<br/>Finance"]
    end

    subgraph Distribution_Domain["Distribution Domain"]
        DST["Distribution<br/>& RTM"]
        TER["Territory<br/>Management"]
        VAN["Van Sales"]
    end

    subgraph Channel_Domain["Channel Domain"]
        POS["Point of<br/>Sale"]
        PRT["Role<br/>Portals"]
        MKT["B2B<br/>Marketplace"]
    end

    subgraph Fulfillment_Domain["Fulfillment Domain"]
        LOG["Logistics &<br/>Last Mile"]
        WMS["Warehouse<br/>Management"]
        FLEET["Fleet<br/>Management"]
    end

    Commerce_Core --> Trade_Finance
    Commerce_Core --> Distribution_Domain
    Commerce_Core --> Channel_Domain
    Commerce_Core --> Fulfillment_Domain
    Trade_Finance --> Channel_Domain
    Distribution_Domain --> Fulfillment_Domain
```

### 1.2 Service Interaction Matrix

| Service       | Calls To                                        | Called By                              |
|---------------|------------------------------------------------|---------------------------------------|
| catalog       | pricing, inventory                              | order, portal, marketplace, pos       |
| order         | pricing, inventory, trade-credit, logistics     | portal, pos, marketplace              |
| pricing       | (AI sidecar)                                    | catalog, order, pos, marketplace      |
| inventory     | (demand AI sidecar)                             | order, pos, distribution, logistics   |
| trade-credit  | (credit AI sidecar)                             | order, portal, marketplace            |
| distribution  | inventory, logistics                            | portal, order                         |
| pos           | catalog, pricing, inventory, order              | (direct terminal)                     |
| portal        | all services                                    | (browser/app)                         |
| logistics     | inventory, (route AI sidecar)                   | order, distribution                   |
| marketplace   | catalog, pricing, order, trade-credit           | portal                                |

---

## 2. Order Orchestration Design

### 2.1 Multi-Party Order Flow

```mermaid
flowchart TD
    START["Retailer places order<br/>via portal/POS/USSD"] --> VALIDATE["Validate Order"]
    VALIDATE --> CREDIT{"Credit<br/>Check"}
    CREDIT -->|Approved| POLICY["Brand Policy<br/>Evaluation"]
    CREDIT -->|Denied| REJECT["Order Rejected<br/>+ Notification"]

    POLICY -->|MOQ Met| SOURCE["Source<br/>Selection"]
    POLICY -->|Under MOQ| GROUP["MOQ Grouping<br/>Engine"]
    GROUP --> SOURCE

    SOURCE --> SINGLE{"Single<br/>Source?"}
    SINGLE -->|Yes| ALLOCATE["Inventory<br/>Allocation"]
    SINGLE -->|No| SPLIT["Order<br/>Splitting"]
    SPLIT --> ALLOCATE

    ALLOCATE --> FULFILL["Fulfillment<br/>Triggered"]
    FULFILL --> WMS["Warehouse<br/>Pick/Pack"]
    WMS --> SHIP["Carrier<br/>Assignment"]
    SHIP --> DELIVER["Delivery<br/>& POD"]
    DELIVER --> INVOICE["Invoice<br/>Generation"]
    INVOICE --> SETTLE["Settlement<br/>& Commission"]
```

### 2.2 Order Splitting Logic

When an order cannot be fulfilled from a single source, the order-service implements a splitting algorithm:

1. **Product-level split**: Group line items by optimal fulfillment location
2. **Quantity-level split**: Distribute quantities across multiple locations when single location has insufficient stock
3. **Geographic split**: Route items from the nearest warehouse to minimize shipping cost and time

Each sub-order becomes an independent fulfillment unit with its own tracking, delivery, and invoicing.

---

## 3. Pricing Engine Design

### 3.1 Price Waterfall

```mermaid
flowchart LR
    BASE["Base Price<br/>(Manufacturer)"] --> TIER["Trade Level<br/>Adjustment"]
    TIER --> VOL["Volume<br/>Discount"]
    VOL --> PROMO["Promotional<br/>Pricing"]
    PROMO --> CONTRACT["Contract<br/>Override"]
    CONTRACT --> GEO["Geographic<br/>Adjustment"]
    GEO --> TAX["Tax<br/>Calculation"]
    TAX --> FINAL["Final<br/>Price"]

    AI["AI Dynamic<br/>Pricing"] -.->|Optional| PROMO
    COMP["Competitive<br/>Monitor"] -.->|Alerts| AI
```

The pricing engine processes prices through a waterfall pipeline:

1. **Base Price** -- manufacturer's list price per product
2. **Trade Level** -- markup/discount per trade level (distributor, wholesaler, retailer)
3. **Volume Discount** -- quantity-break adjustments based on order size
4. **Promotional** -- time-limited promotional pricing overrides
5. **Contract** -- customer-specific negotiated pricing
6. **Geographic** -- location-based adjustments for logistics costs
7. **Tax** -- applicable VAT/GST calculation

The Rust price-calculator component executes the waterfall computation in sub-millisecond time for high-throughput scenarios.

---

## 4. Inventory Management Design

### 4.1 Multi-Location Inventory Model

```mermaid
flowchart TB
    subgraph Locations
        WH1["Warehouse A<br/>(Central DC)"]
        WH2["Warehouse B<br/>(Regional)"]
        ST1["Store 1<br/>(Retail)"]
        ST2["Store 2<br/>(Retail)"]
        VAN1["Van 1<br/>(Mobile)"]
        CON1["Consignment<br/>(at Distributor)"]
    end

    subgraph Tracking
        SER["Serialized<br/>Tracking"]
        LOT["Lot/Batch<br/>Tracking"]
        EXP["Expiry<br/>Management"]
    end

    subgraph Operations
        RES["Stock<br/>Reservation"]
        TXF["Inter-Location<br/>Transfer"]
        RPL["Demand-Driven<br/>Replenishment"]
        VAL["Inventory<br/>Valuation"]
    end

    Locations --> Tracking
    Tracking --> Operations
    RPL -->|ML Forecast| DEMAND["Demand<br/>Forecaster AI"]
```

### 4.2 Stock Reservation Strategy

1. **Soft Reserve** -- temporary hold during checkout (TTL: 15 minutes)
2. **Hard Reserve** -- confirmed allocation after order approval
3. **Release** -- automatic release on order cancellation or timeout

---

## 5. Trade Credit Design

### 5.1 Credit Decision Flow

```mermaid
flowchart TD
    REQ["Credit Request"] --> SCORE["AI Credit<br/>Scoring"]
    SCORE --> DATA["Input Data"]
    DATA --> D1["Transaction History"]
    DATA --> D2["Payment Behavior"]
    DATA --> D3["External Bureau"]
    DATA --> D4["Business Profile"]

    SCORE --> DECISION{"Score<br/>Threshold"}
    DECISION -->|High Score| AUTO["Auto-Approve<br/>Full Limit"]
    DECISION -->|Medium Score| PARTIAL["Approve<br/>Reduced Limit"]
    DECISION -->|Low Score| REVIEW["Manual<br/>Review Queue"]
    DECISION -->|Very Low| DENY["Auto-Deny"]

    AUTO --> TERMS["Assign Payment<br/>Terms (Net 30/60/90)"]
    PARTIAL --> TERMS
    REVIEW --> MANUAL["Credit Officer<br/>Review"]
    MANUAL -->|Approve| TERMS
    MANUAL -->|Deny| DENY

    TERMS --> MONITOR["Real-Time<br/>Exposure Monitoring"]
    MONITOR --> ALERT["Threshold<br/>Alerts"]
```

---

## 6. POS Offline Architecture

### 6.1 Offline-First Design

```mermaid
flowchart TB
    subgraph POS_Terminal["POS Terminal"]
        UI["Checkout UI"]
        LOCAL_DB["Local SQLite<br/>Database"]
        QUEUE["Outbound<br/>Event Queue"]
        CACHE["Product/Price<br/>Cache"]
    end

    subgraph Cloud["Cloud Services"]
        API["pos-service API"]
        DB["PostgreSQL"]
        EVT["Event Bus"]
    end

    UI --> LOCAL_DB
    UI --> CACHE
    UI --> QUEUE

    QUEUE -->|"When Online"| API
    API --> DB
    API --> EVT

    API -->|"Sync Down"| CACHE
    API -->|"Sync Down"| LOCAL_DB

    style QUEUE fill:#ff9,stroke:#333
```

**Offline Operation**:
1. Product catalog and pricing cached locally
2. Transactions stored in local SQLite database
3. Events queued in outbound buffer
4. On connectivity restoration, queue drains in FIFO order
5. Conflict resolution via last-write-wins with vector clocks

**Supported Hardware**:
- Stripe Terminal (Verifone P400, BBPOS WisePOS E)
- Square Terminal, Square Reader
- Sunmi V2 Pro, T2 Mini, L2
- PAX A920 Pro, A77

---

## 7. Portal Architecture

### 7.1 13 Role-Specific Portals

```mermaid
flowchart TB
    subgraph Portal_Framework["Portal Service (Next.js)"]
        SHELL["App Shell<br/>(Shared Layout)"]
        AUTH_CTX["Auth Context<br/>(Role Detection)"]
    end

    subgraph Portals
        P1["Manufacturer Portal"]
        P2["Distributor Portal"]
        P3["Wholesaler Portal"]
        P4["Retailer Portal"]
        P5["Supermarket Portal"]
        P6["Warehouse Portal"]
        P7["Delivery Co Portal"]
        P8["Driver Portal"]
        P9["Agent Portal"]
        P10["Brand Manager Portal"]
        P11["Merchandiser Portal"]
        P12["Field Sales Portal"]
        P13["Trade Marketing Portal"]
    end

    SHELL --> AUTH_CTX
    AUTH_CTX --> Portals
```

Each portal is a micro-frontend loaded based on the authenticated user's role, sharing a common app shell with role-specific navigation, dashboards, and workflows.

---

## 8. Technology Stack Summary

| Layer         | Technology                                       |
|---------------|--------------------------------------------------|
| Frontend      | Next.js 14, React 18, TypeScript, Ant Design     |
| API Gateway   | Kong / Envoy                                     |
| GraphQL       | Hasura                                           |
| Core Services | Go 1.22                                          |
| AI/ML         | Python 3.12, scikit-learn, TensorFlow, OR-Tools  |
| High-Perf     | Rust (EDI parser, price calculator, sync engine)  |
| Database      | PostgreSQL 16, TimescaleDB                       |
| Cache         | Redis 7 Cluster                                  |
| Search        | Elasticsearch 8                                  |
| Events        | NATS JetStream / Redpanda                        |
| Workflows     | Temporal                                         |
| Container     | Kubernetes (EKS/GKE)                             |
| CI/CD         | GitHub Actions, ArgoCD                           |
| Observability | OpenTelemetry, Prometheus, Grafana, Jaeger        |
