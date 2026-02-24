# ERP-Commerce -- Low-Level Design (LLD)

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

## 1. Service Internal Architecture

Each Go microservice follows a consistent hexagonal (ports and adapters) architecture.

### 1.1 Service Package Structure

```
services/<name>-service/
  main.go                  # Entry point, HTTP server bootstrap
  internal/
    domain/
      models.go            # Domain entities and value objects
      events.go            # Domain event definitions
      errors.go            # Domain-specific errors
    ports/
      inbound.go           # Service interface (use cases)
      outbound.go          # Repository and external service interfaces
    adapters/
      http/
        handlers.go        # HTTP route handlers
        middleware.go       # Tenant extraction, auth, logging
        dto.go             # Request/response DTOs
      grpc/
        server.go          # gRPC service implementation
      repository/
        postgres.go        # PostgreSQL repository implementation
        redis.go           # Redis cache adapter
      events/
        publisher.go       # NATS/Redpanda event publisher
        subscriber.go      # Event subscription handlers
    application/
      service.go           # Application service (orchestrates ports)
      commands.go          # Command handlers
      queries.go           # Query handlers
  migrations/
    *.sql                  # Database migration scripts
  Dockerfile
  README.md
```

### 1.2 Request Processing Pipeline

```mermaid
sequenceDiagram
    participant Client
    participant Middleware as HTTP Middleware
    participant Handler as Route Handler
    participant AppSvc as Application Service
    participant Domain as Domain Logic
    participant Repo as Repository
    participant Events as Event Publisher

    Client->>Middleware: HTTP Request
    Middleware->>Middleware: Extract JWT Claims
    Middleware->>Middleware: Extract X-Tenant-ID
    Middleware->>Middleware: Rate Limit Check
    Middleware->>Handler: Enriched Context

    Handler->>Handler: Validate DTO
    Handler->>AppSvc: Execute Command/Query

    AppSvc->>Domain: Apply Business Rules
    Domain-->>AppSvc: Domain Result

    AppSvc->>Repo: Persist Changes
    Repo-->>AppSvc: Confirmation

    AppSvc->>Events: Publish Domain Event
    Events-->>AppSvc: Published

    AppSvc-->>Handler: Result
    Handler-->>Client: HTTP Response
```

---

## 2. Catalog Service Detailed Design

### 2.1 Domain Model

```mermaid
classDiagram
    class Product {
        +UUID id
        +UUID tenantId
        +string sku
        +string name
        +string description
        +UUID categoryId
        +UUID brandId
        +ProductStatus status
        +map~string,string~ attributes
        +[]MediaAsset media
        +time.Time createdAt
        +time.Time updatedAt
    }

    class ProductVariant {
        +UUID id
        +UUID productId
        +string variantSku
        +string name
        +map~string,string~ options
        +decimal weight
        +string weightUnit
        +string barcode
    }

    class Category {
        +UUID id
        +UUID tenantId
        +UUID parentId
        +string name
        +string slug
        +int level
        +int sortOrder
    }

    class Brand {
        +UUID id
        +UUID tenantId
        +string name
        +string logo
    }

    class BrandPolicy {
        +UUID id
        +UUID tenantId
        +UUID brandId
        +UUID categoryId
        +int distributorMOQ
        +int wholesalerMOQ
        +int retailerMOQ
    }

    class MediaAsset {
        +UUID id
        +UUID productId
        +string url
        +string mimeType
        +int sortOrder
    }

    Product "1" --> "*" ProductVariant
    Product "1" --> "*" MediaAsset
    Product --> Category
    Product --> Brand
    Brand "1" --> "*" BrandPolicy
    Category "1" --> "*" Category : parent
```

### 2.2 Key Algorithms

**Category Tree Retrieval** -- Uses materialized path pattern for efficient tree queries:

```sql
-- Materialized path: /root/electronics/phones/smartphones
SELECT * FROM categories
WHERE tenant_id = $1
  AND path LIKE $2 || '%'
ORDER BY path;
```

**Product Search** -- Elasticsearch with faceted filtering:

```json
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "search term" } },
        { "term": { "tenant_id": "uuid" } }
      ],
      "filter": [
        { "term": { "category_id": "uuid" } },
        { "range": { "price": { "gte": 100, "lte": 500 } } }
      ]
    }
  },
  "aggs": {
    "brands": { "terms": { "field": "brand_id" } },
    "price_ranges": { "range": { "field": "price", "ranges": [...] } }
  }
}
```

---

## 3. Order Service Detailed Design

### 3.1 Order State Machine

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> PendingValidation: submit()
    PendingValidation --> CreditCheck: validate()
    PendingValidation --> Rejected: validationFailed()
    CreditCheck --> PolicyCheck: creditApproved()
    CreditCheck --> Rejected: creditDenied()
    PolicyCheck --> Approved: policyPass()
    PolicyCheck --> MOQGrouping: underMOQ()
    MOQGrouping --> Approved: grouped()
    Approved --> Splitting: multiSource()
    Approved --> Allocated: singleSource()
    Splitting --> Allocated: split()
    Allocated --> Picking: warehouseTrigger()
    Picking --> Packed: pickComplete()
    Packed --> Shipped: carrierAssigned()
    Shipped --> InTransit: departed()
    InTransit --> Delivered: podCaptured()
    Delivered --> Invoiced: invoiceGenerated()
    Invoiced --> Settled: paymentReceived()
    Settled --> [*]

    Rejected --> [*]
    Draft --> Cancelled: cancel()
    Approved --> Cancelled: cancel()
    Cancelled --> [*]
```

### 3.2 EDI Message Processing

```mermaid
flowchart LR
    subgraph Inbound
        EDI_IN["EDI Document<br/>(AS2/SFTP)"]
        PARSER["Rust EDI Parser"]
        MAP["Message Mapper"]
    end

    subgraph Processing
        VALIDATE["Schema Validation"]
        TRANSFORM["Transform to<br/>Internal Order"]
        ENRICH["Enrich with<br/>Catalog Data"]
    end

    subgraph Outbound
        GEN["EDI Generator"]
        SIGN["Digital Signature"]
        SEND["AS2/SFTP Send"]
    end

    EDI_IN --> PARSER
    PARSER --> MAP
    MAP --> VALIDATE
    VALIDATE --> TRANSFORM
    TRANSFORM --> ENRICH
    ENRICH --> GEN
    GEN --> SIGN
    SIGN --> SEND
```

**Supported EDI Transactions**:

| Standard | Code     | Name           | Direction |
|----------|----------|----------------|-----------|
| X12      | 850      | Purchase Order | Inbound   |
| X12      | 855      | PO Acknowledgment | Outbound |
| X12      | 856      | Ship Notice    | Outbound  |
| X12      | 810      | Invoice        | Outbound  |
| EDIFACT  | ORDERS   | Purchase Order | Inbound   |
| EDIFACT  | ORDRSP   | Order Response | Outbound  |
| EDIFACT  | DESADV   | Dispatch Advice| Outbound  |
| EDIFACT  | INVOIC   | Invoice        | Outbound  |

---

## 4. Pricing Service Detailed Design

### 4.1 Price Calculation Pipeline

```mermaid
flowchart TD
    INPUT["Price Request<br/>(productId, quantity, customerId, location)"] --> CACHE{"Redis<br/>Cache Hit?"}
    CACHE -->|Hit| RETURN["Return Cached Price"]
    CACHE -->|Miss| BASE["Load Base Price<br/>from product"]
    BASE --> TIER["Apply Trade Level<br/>Pricing Rule"]
    TIER --> VOLUME["Apply Volume<br/>Discount Schedule"]
    VOLUME --> PROMO["Apply Active<br/>Promotions"]
    PROMO --> CONTRACT["Apply Contract<br/>Pricing Override"]
    CONTRACT --> GEO["Apply Geographic<br/>Adjustment"]
    GEO --> TAX["Calculate Tax<br/>(VAT/GST)"]
    TAX --> ROUND["Currency<br/>Rounding"]
    ROUND --> STORE["Cache Result<br/>(TTL: 5min)"]
    STORE --> RETURN
```

### 4.2 Rust Price Calculator

The high-performance price calculator is implemented in Rust and called via FFI from the Go pricing-service for bulk pricing operations (catalog repricing, promotional batch updates).

```rust
// Simplified interface
pub struct PriceRequest {
    pub product_id: Uuid,
    pub base_price: Decimal,
    pub trade_level: TradeLevel,
    pub quantity: u32,
    pub customer_id: Option<Uuid>,
    pub location: Option<GeoPoint>,
}

pub struct PriceResult {
    pub unit_price: Decimal,
    pub total_price: Decimal,
    pub discount_amount: Decimal,
    pub tax_amount: Decimal,
    pub applied_rules: Vec<AppliedRule>,
}
```

---

## 5. Trade Credit Service Detailed Design

### 5.1 Credit Scoring Model Inputs

| Feature Category    | Inputs                                                      |
|--------------------|-------------------------------------------------------------|
| Transaction History | Order frequency, average order value, total GMV, tenure     |
| Payment Behavior   | On-time rate, average days to pay, default count            |
| Business Profile   | Business type, years in operation, number of outlets        |
| External Data      | Credit bureau score, bank statements, mobile money history  |
| Network Score      | Supplier references, peer network strength                  |

### 5.2 Collections Escalation Flow

```mermaid
flowchart TD
    DUE["Invoice Due Date"] --> GRACE["Grace Period<br/>(3 days)"]
    GRACE --> R1["Reminder 1<br/>SMS + Email"]
    R1 -->|+7 days| R2["Reminder 2<br/>Phone Call"]
    R2 -->|+7 days| R3["Reminder 3<br/>WhatsApp + Email"]
    R3 -->|+14 days| HOLD["Account Hold<br/>Block new orders"]
    HOLD -->|+30 days| ESCALATE["Escalation<br/>to Collections Team"]
    ESCALATE -->|+30 days| LEGAL["Legal Action<br/>or Write-Off"]
```

---

## 6. POS Service Detailed Design

### 6.1 Checkout Flow

```mermaid
sequenceDiagram
    participant Cashier
    participant POS_UI as POS UI
    participant LocalDB as Local SQLite
    participant Scanner as Barcode Scanner
    participant Printer as Receipt Printer
    participant Drawer as Cash Drawer
    participant Cloud as pos-service API

    Cashier->>Scanner: Scan Item Barcode
    Scanner->>POS_UI: Barcode Data
    POS_UI->>LocalDB: Lookup Product
    LocalDB-->>POS_UI: Product + Price
    POS_UI->>POS_UI: Add to Cart

    loop More Items
        Cashier->>Scanner: Scan Next Item
        Scanner->>POS_UI: Barcode Data
        POS_UI->>LocalDB: Lookup Product
        LocalDB-->>POS_UI: Product + Price
        POS_UI->>POS_UI: Add to Cart
    end

    Cashier->>POS_UI: Checkout
    POS_UI->>POS_UI: Calculate Total
    POS_UI->>Drawer: Open Cash Drawer
    Cashier->>POS_UI: Confirm Payment
    POS_UI->>LocalDB: Save Transaction
    POS_UI->>Printer: Print Receipt
    POS_UI->>Cloud: Sync Transaction (async)
```

### 6.2 Offline Sync Protocol

```mermaid
sequenceDiagram
    participant POS as POS Terminal
    participant Queue as Local Queue
    participant Sync as Sync Engine
    participant API as pos-service

    Note over POS,API: OFFLINE MODE
    POS->>Queue: Enqueue Transaction
    POS->>Queue: Enqueue Transaction
    POS->>Queue: Enqueue Transaction

    Note over POS,API: CONNECTIVITY RESTORED
    Sync->>Queue: Dequeue Batch (50 max)
    Sync->>API: POST /v1/pos/sync (batch)
    API->>API: Process + Conflict Check
    API-->>Sync: Sync Result (accepted/conflicts)
    Sync->>Sync: Resolve Conflicts (vector clock)
    Sync->>Queue: Remove Synced Items
    Sync->>POS: Update Local State
```

---

## 7. Logistics Service Detailed Design

### 7.1 VRP Solver Architecture

```mermaid
flowchart LR
    INPUT["Delivery Jobs<br/>(addresses, weights,<br/>time windows)"] --> PREP["Data<br/>Preparation"]
    PREP --> MATRIX["Distance Matrix<br/>(Maps API)"]
    MATRIX --> VRP["OR-Tools VRP<br/>Solver (Python)"]
    VRP --> ROUTES["Optimized<br/>Routes"]
    ROUTES --> ASSIGN["Driver<br/>Assignment"]
    ASSIGN --> NAV["Turn-by-Turn<br/>Navigation"]

    VRP --> CONSTRAINTS["Constraints"]
    CONSTRAINTS --> C1["Vehicle Capacity"]
    CONSTRAINTS --> C2["Time Windows"]
    CONSTRAINTS --> C3["Driver Hours"]
    CONSTRAINTS --> C4["Priority Stops"]
```

### 7.2 Proof of Delivery Capture

| POD Method    | Description                                | Use Case              |
|---------------|--------------------------------------------|-----------------------|
| Digital Signature | Recipient signs on driver's device     | Standard delivery     |
| Photo         | Photo of delivered goods at location        | Unattended delivery   |
| OTP           | One-time code sent to recipient via SMS     | High-value delivery   |
| Geofence      | GPS confirms driver at delivery location    | All deliveries        |
| Biometric     | Fingerprint verification                    | Cash-on-delivery      |
