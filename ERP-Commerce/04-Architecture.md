# ERP-Commerce -- System Architecture Document

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |
| Status   | Active                                  |

---

## 1. Architecture Overview

ERP-Commerce follows a microservices architecture with 10 domain services, an API gateway layer, event-driven communication, and integration with the broader ERP platform through ERP-IAM (identity), ERP-Platform (entitlements), and a NATS/Pulsar event backbone.

### 1.1 C4 Context Diagram

```mermaid
C4Context
    title ERP-Commerce System Context

    Person(manufacturer, "Manufacturer", "FMCG/CPG producer")
    Person(distributor, "Distributor", "Regional distributor")
    Person(retailer, "Retailer", "Shop/store owner")
    Person(driver, "Driver", "Delivery personnel")
    Person(fieldsales, "Field Sales", "Sales representative")

    System(commerce, "ERP-Commerce", "Multi-party trade platform")

    System_Ext(iam, "ERP-IAM", "Identity & Access")
    System_Ext(platform, "ERP-Platform", "Entitlements & Licensing")
    System_Ext(finance, "ERP-Finance", "GL & Accounting")
    System_Ext(scm, "ERP-SCM", "Supply Chain")
    System_Ext(crm, "ERP-CRM", "Customer Relationships")

    System_Ext(payments, "Payment Providers", "Stripe, Paystack, M-Pesa")
    System_Ext(edi, "EDI Networks", "X12/EDIFACT VAN")
    System_Ext(maps, "Maps/GPS", "Google Maps, HERE")

    Rel(manufacturer, commerce, "Manages catalog, pricing, distribution")
    Rel(distributor, commerce, "Orders, inventory, route management")
    Rel(retailer, commerce, "POS, ordering, credit")
    Rel(driver, commerce, "Deliveries, POD capture")
    Rel(fieldsales, commerce, "Territory visits, order capture")

    Rel(commerce, iam, "Authentication, RBAC")
    Rel(commerce, platform, "Entitlements, feature flags")
    Rel(commerce, finance, "Journal entries, invoicing")
    Rel(commerce, scm, "Procurement, suppliers")
    Rel(commerce, crm, "Customer data sync")
    Rel(commerce, payments, "Payment processing")
    Rel(commerce, edi, "EDI transactions")
    Rel(commerce, maps, "Geocoding, routing")
```

### 1.2 Container Diagram

```mermaid
flowchart TB
    subgraph Clients
        WEB["Next.js Web App"]
        MOB["Mobile Apps<br/>(React Native)"]
        USSD_GW["USSD Gateway"]
        WA["WhatsApp Bot"]
        POS_HW["POS Hardware<br/>(Sunmi/PAX/Square)"]
        EDI_VAN["EDI VAN"]
    end

    subgraph API_Layer["API Gateway Layer"]
        GW["API Gateway<br/>(Kong/Envoy)"]
        GQL["GraphQL Engine<br/>(Hasura)"]
        AUTH["Auth Middleware<br/>(JWT/OIDC)"]
    end

    subgraph Core_Services["Core Commerce Services"]
        CAT["catalog-service<br/>Go :8001"]
        ORD["order-service<br/>Go :8002"]
        PRC["pricing-service<br/>Go :8003"]
        INV["inventory-service<br/>Go :8004"]
        TCR["trade-credit-service<br/>Go :8005"]
    end

    subgraph Channel_Services["Channel & Operations Services"]
        DST["distribution-service<br/>Go :8006"]
        POS["pos-service<br/>Go :8007"]
        PRT["portal-service<br/>Go :8008"]
        LOG["logistics-service<br/>Go :8009"]
        MKT["marketplace-service<br/>Go :8010"]
    end

    subgraph AI_Services["AI/ML Services"]
        PRICE_AI["Dynamic Pricing<br/>Python :9001"]
        CREDIT_AI["Credit Scoring<br/>Python :9002"]
        ROUTE_AI["Route Optimizer<br/>Python (VRP) :9003"]
        DEMAND_AI["Demand Forecast<br/>Python :9004"]
    end

    subgraph Data_Layer["Data Layer"]
        PG["PostgreSQL<br/>(Primary DB)"]
        REDIS["Redis<br/>(Cache/Sessions)"]
        S3["Object Storage<br/>(Media/Assets)"]
        ES["Elasticsearch<br/>(Product Search)"]
    end

    subgraph Event_Layer["Event Backbone"]
        NATS["NATS/Redpanda"]
        TEMPO["Temporal<br/>(Workflow Engine)"]
    end

    Clients --> GW
    GW --> AUTH
    AUTH --> GQL
    AUTH --> Core_Services
    AUTH --> Channel_Services

    Core_Services --> Data_Layer
    Channel_Services --> Data_Layer
    Core_Services --> Event_Layer
    Channel_Services --> Event_Layer
    AI_Services --> Data_Layer

    PRC --> PRICE_AI
    TCR --> CREDIT_AI
    LOG --> ROUTE_AI
    INV --> DEMAND_AI
```

---

## 2. Service Architecture

### 2.1 Service Inventory

| Service              | Language | Port | Domain                                    | Database Schema       |
|----------------------|----------|------|------------------------------------------|----------------------|
| catalog-service      | Go       | 8001 | Product catalog, PIM, categories          | `commerce_catalog`   |
| order-service        | Go       | 8002 | Order orchestration, splitting, EDI       | `commerce_orders`    |
| pricing-service      | Go       | 8003 | Pricing rules, promotions, contracts      | `commerce_pricing`   |
| inventory-service    | Go       | 8004 | Stock tracking, reservations, valuation   | `commerce_inventory` |
| trade-credit-service | Go       | 8005 | Credit scoring, terms, collections        | `commerce_credit`    |
| distribution-service | Go       | 8006 | RTM, territories, van sales, beat plans   | `commerce_dist`      |
| pos-service          | Go       | 8007 | Checkout, receipts, offline sync          | `commerce_pos`       |
| portal-service       | Go       | 8008 | Role-specific dashboards and workflows    | `commerce_portals`   |
| logistics-service    | Go       | 8009 | Delivery, routing, GPS, POD              | `commerce_logistics` |
| marketplace-service  | Go       | 8010 | Vendor mgmt, commissions, disputes       | `commerce_marketplace`|

### 2.2 AI/ML Sidecar Services

| Service              | Language | Port | Purpose                                  |
|----------------------|----------|------|------------------------------------------|
| dynamic-pricing      | Python   | 9001 | ML-based price optimization              |
| credit-scoring       | Python   | 9002 | AI credit score computation              |
| route-optimizer      | Python   | 9003 | VRP solving with OR-Tools                |
| demand-forecaster    | Python   | 9004 | Time-series demand prediction            |

### 2.3 Rust Components

| Component            | Purpose                                              |
|----------------------|-----------------------------------------------------|
| edi-parser           | High-performance EDI X12/EDIFACT parsing             |
| price-calculator     | Sub-millisecond pricing computation engine           |
| offline-sync-engine  | Conflict resolution for offline POS data sync        |

---

## 3. Communication Patterns

### 3.1 Synchronous Communication

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant OrderSvc as order-service
    participant PricingSvc as pricing-service
    participant InventorySvc as inventory-service
    participant CreditSvc as trade-credit-service

    Client->>Gateway: POST /v1/order
    Gateway->>OrderSvc: Create Order
    OrderSvc->>PricingSvc: Calculate Prices
    PricingSvc-->>OrderSvc: Price Response (<50ms)
    OrderSvc->>InventorySvc: Check & Reserve Stock
    InventorySvc-->>OrderSvc: Reservation Confirmed
    OrderSvc->>CreditSvc: Validate Credit
    CreditSvc-->>OrderSvc: Credit Approved
    OrderSvc-->>Gateway: Order Created (201)
    Gateway-->>Client: Order Response
```

### 3.2 Asynchronous Communication (Event-Driven)

```mermaid
flowchart LR
    subgraph Publishers
        ORD["order-service"]
        INV["inventory-service"]
        POS["pos-service"]
        TCR["trade-credit-service"]
    end

    subgraph Event_Bus["NATS / Redpanda"]
        T1["erp.commerce.order.*"]
        T2["erp.commerce.inventory.*"]
        T3["erp.commerce.pos.*"]
        T4["erp.commerce.trade-credit.*"]
    end

    subgraph Subscribers
        LOG["logistics-service"]
        DST["distribution-service"]
        PRT["portal-service"]
        MKT["marketplace-service"]
        FIN["ERP-Finance"]
    end

    ORD --> T1
    INV --> T2
    POS --> T3
    TCR --> T4

    T1 --> LOG
    T1 --> DST
    T1 --> PRT
    T1 --> FIN
    T2 --> PRT
    T2 --> MKT
    T3 --> INV
    T3 --> FIN
    T4 --> PRT
    T4 --> ORD
```

### 3.3 Event Naming Convention

All events follow the pattern: `erp.commerce.<entity>.<action>` with CloudEvents envelope.

| Topic Pattern                      | Publisher           | Description                    |
|------------------------------------|--------------------|---------------------------------|
| `erp.commerce.order.created`       | order-service      | New order submitted             |
| `erp.commerce.order.updated`       | order-service      | Order status changed            |
| `erp.commerce.inventory.updated`   | inventory-service  | Stock level changed             |
| `erp.commerce.pos.created`         | pos-service        | POS transaction completed       |
| `erp.commerce.pricing.updated`     | pricing-service    | Price rule changed              |
| `erp.commerce.trade-credit.created`| trade-credit-svc   | Credit decision made            |
| `erp.commerce.logistics.updated`   | logistics-service  | Delivery status changed         |
| `erp.commerce.marketplace.created` | marketplace-svc    | New vendor onboarded            |

---

## 4. Data Architecture

### 4.1 Database Strategy

- **PostgreSQL** as the primary OLTP database with per-service schema isolation
- **Redis** for caching (pricing rules, session data, rate limiting)
- **Elasticsearch** for full-text product search and analytics
- **Object Storage** (S3-compatible) for media assets and EDI documents
- **TimescaleDB** extension for time-series metrics (inventory levels, price history)

### 4.2 Multi-Tenancy

```mermaid
flowchart TB
    REQ["Incoming Request"] --> AUTH["Auth Middleware<br/>Extract X-Tenant-ID"]
    AUTH --> RLS["PostgreSQL Row-Level<br/>Security Policy"]
    RLS --> DATA["Tenant-Scoped Data"]

    AUTH --> CACHE["Redis Namespace<br/>tenant:{id}:*"]
    AUTH --> SEARCH["Elasticsearch Index<br/>tenant-{id}-products"]
```

All data access is tenant-scoped using:
1. `X-Tenant-ID` header extraction in middleware
2. PostgreSQL Row-Level Security (RLS) policies
3. Redis key namespacing
4. Per-tenant Elasticsearch indices

---

## 5. Deployment Architecture

```mermaid
flowchart TB
    subgraph Edge
        CDN["CDN<br/>(CloudFlare)"]
        LB["Load Balancer<br/>(L7)"]
    end

    subgraph K8s["Kubernetes Cluster"]
        subgraph NS_Commerce["namespace: erp-commerce"]
            GW["API Gateway"]
            CAT["catalog-service<br/>replicas: 3"]
            ORD["order-service<br/>replicas: 3"]
            PRC["pricing-service<br/>replicas: 5"]
            INV["inventory-service<br/>replicas: 3"]
            TCR["trade-credit-svc<br/>replicas: 2"]
            DST["distribution-svc<br/>replicas: 2"]
            POS["pos-service<br/>replicas: 3"]
            PRT["portal-service<br/>replicas: 3"]
            LOG["logistics-svc<br/>replicas: 2"]
            MKT["marketplace-svc<br/>replicas: 2"]
        end

        subgraph NS_AI["namespace: erp-commerce-ai"]
            AI1["dynamic-pricing<br/>GPU-enabled"]
            AI2["credit-scoring"]
            AI3["route-optimizer"]
            AI4["demand-forecaster"]
        end
    end

    subgraph Data
        PG_PRIMARY["PostgreSQL Primary"]
        PG_REPLICA["PostgreSQL Read Replicas"]
        REDIS_CLUSTER["Redis Cluster"]
        ES_CLUSTER["Elasticsearch Cluster"]
        NATS_CLUSTER["NATS Cluster"]
    end

    CDN --> LB
    LB --> GW
    GW --> NS_Commerce
    NS_Commerce --> Data
    NS_AI --> Data
```

---

## 6. Security Architecture

### 6.1 Authentication and Authorization

```mermaid
sequenceDiagram
    participant User
    participant Gateway
    participant IAM as ERP-IAM
    participant Service
    participant Platform as ERP-Platform

    User->>Gateway: Request + JWT
    Gateway->>IAM: Validate JWT
    IAM-->>Gateway: Token Valid + Claims
    Gateway->>Platform: Check Entitlements
    Platform-->>Gateway: Entitlements (commerce.*)
    Gateway->>Service: Request + Tenant Context
    Service->>Service: RLS Enforcement
    Service-->>Gateway: Response
    Gateway-->>User: Response
```

### 6.2 Security Controls

| Layer            | Control                                              |
|------------------|-----------------------------------------------------|
| Network          | mTLS between all services, network policies          |
| Authentication   | JWT/OIDC via ERP-IAM                                 |
| Authorization    | RBAC with 13 role definitions                        |
| Data             | AES-256 encryption at rest, TLS 1.3 in transit       |
| API              | Rate limiting, request validation, CORS              |
| Payments         | PCI-DSS Level 1 compliance, tokenization             |
| Audit            | CloudEvents decision logging, 24h rollback window    |

---

## 7. Integration Points

| System          | Protocol     | Direction | Purpose                                |
|-----------------|-------------|-----------|----------------------------------------|
| ERP-IAM         | OIDC/JWT    | Bidirectional | Authentication, role management    |
| ERP-Platform    | REST        | Outbound  | Entitlement validation                  |
| ERP-Finance     | Events      | Outbound  | Journal entries, invoices               |
| ERP-SCM         | Events      | Bidirectional | Procurement, supplier management  |
| ERP-CRM         | Events      | Bidirectional | Customer data synchronization     |
| Payment Providers| REST/Webhooks| Bidirectional | Payment processing              |
| EDI Networks    | AS2/SFTP    | Bidirectional | X12/EDIFACT document exchange    |
| Maps APIs       | REST        | Outbound  | Geocoding, routing, distance matrix    |
| Credit Bureaus  | REST        | Outbound  | External credit data                    |
