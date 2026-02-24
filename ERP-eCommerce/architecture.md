# System Architecture -- FusionCommerce (ERP-eCommerce)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

FusionCommerce is an API-first, event-driven, composable commerce platform organized as a TypeScript monorepo containing 15 microservices, 5 shared packages, and multi-platform client applications. The architecture follows a six-layer model: Headless Presentation, Commerce Core Services, Eventing and Workflow Orchestration, Intelligence and Data, State and Storage, and Infrastructure and Operations.

## 2. C4 Model Architecture

### 2.1 System Context Diagram

```mermaid
C4Context
    title FusionCommerce - System Context (Level 1)

    Person(consumer, "Consumer", "Browses products, makes purchases, manages subscriptions")
    Person(merchant, "Merchant Admin", "Manages catalog, orders, fulfillment, analytics")
    Person(creator, "Content Creator", "Hosts livestreams, creates UGC, manages referrals")

    System(fc, "FusionCommerce Platform", "Composable commerce with 15 microservices")

    System_Ext(iam, "ERP-IAM", "OIDC/JWT authentication and authorization")
    System_Ext(platform, "ERP-Platform", "Entitlements, licensing, multi-tenancy")
    System_Ext(payments, "Payment Processors", "Stripe, PayPal, Apple Pay, Google Pay")
    System_Ext(shipping, "Shipping Carriers", "EasyPost, Shippo, FedEx, UPS, DHL")
    System_Ext(social, "Social Platforms", "Instagram, Facebook, TikTok")
    System_Ext(email, "Email/SMS", "SendGrid, Twilio")
    System_Ext(cdn, "CDN/Edge", "Cloudflare, Voxility")

    Rel(consumer, fc, "REST/GraphQL API")
    Rel(merchant, fc, "Admin Dashboard")
    Rel(creator, fc, "Livestream Console, UGC Hub")
    Rel(fc, iam, "OIDC/JWT")
    Rel(fc, platform, "Entitlements")
    Rel(fc, payments, "Payment APIs")
    Rel(fc, shipping, "Shipping APIs")
    Rel(fc, social, "Commerce APIs")
    Rel(fc, email, "Notification APIs")
    Rel(fc, cdn, "Asset delivery")
```

### 2.2 Container Diagram

```mermaid
flowchart TB
    subgraph "Client Layer"
        WEB["Web Storefront<br/>(Next.js)"]
        MOBILE["Mobile Apps<br/>(Flutter / Native)"]
        ADMIN["Merchant Admin<br/>(React)"]
    end

    subgraph "API Gateway"
        GW["API Gateway<br/>(Kong / Traefik)"]
    end

    subgraph "Commerce Core Services"
        CAT["Catalog Service<br/>:3000"]
        ORD["Orders Service<br/>:3001"]
        INV["Inventory Service<br/>:3002"]
        GRP["Group Commerce<br/>:3003"]
        PAY["Payments Service<br/>:3004"]
        SHIP["Shipping Service<br/>:3005"]
        CHK["Checkout Service<br/>:3006"]
        STF["Storefront Service<br/>:3007"]
        THM["Theme Service<br/>:3008"]
        SRC["Search Service<br/>:3009"]
        SOC["Social Commerce<br/>:3010"]
        SUB["Subscription Commerce<br/>:3011"]
        LOY["Loyalty Service<br/>:3012"]
        FUL["Fulfillment Service<br/>:3013"]
        ANL["Analytics Service<br/>:3014"]
    end

    subgraph "Eventing & Workflow"
        KFK["Apache Kafka<br/>(Redpanda)"]
        N8N["n8n Workflow Engine"]
    end

    subgraph "Data Layer"
        YDB[(YugabyteDB<br/>Transactional)]
        SDB[(ScyllaDB<br/>Low-Latency)]
        MIO[(MinIO<br/>Object Storage)]
        DRD[(Apache Druid<br/>Analytics)]
        OS[(OpenSearch<br/>Product Search)]
    end

    WEB --> GW
    MOBILE --> GW
    ADMIN --> GW
    GW --> CAT & ORD & INV & GRP & PAY & SHIP
    GW --> CHK & STF & THM & SRC & SOC & SUB & LOY & FUL & ANL

    CAT --> KFK
    ORD --> KFK
    INV --> KFK
    GRP --> KFK
    PAY --> KFK
    CHK --> KFK
    SOC --> KFK
    SUB --> KFK
    LOY --> KFK
    FUL --> KFK

    KFK --> N8N
    KFK --> DRD
    KFK --> INV
    KFK --> FUL
    KFK --> ANL

    CAT --> YDB
    ORD --> YDB
    INV --> YDB
    GRP --> YDB
    PAY --> YDB
    CHK --> YDB
    SUB --> YDB
    LOY --> YDB
    FUL --> YDB
    SHIP --> YDB

    STF --> SDB
    LOY --> SDB
    SRC --> OS
    THM --> MIO
    CAT --> MIO
    SOC --> MIO
    ANL --> DRD
```

## 3. Layered Architecture

### 3.1 Layer 1 -- Headless Presentation

The presentation layer consists of decoupled "heads" that consume FusionCommerce APIs:

| Head | Technology | Purpose |
|------|-----------|---------|
| Web Storefront | Next.js / Vue Storefront | Primary consumer shopping experience |
| Mobile Apps | Flutter + Native (Android/iOS) | Native mobile commerce |
| Merchant Admin | React + Ant Design | Back-office management dashboard |
| Social Embeds | Platform SDKs | Instagram Shopping, Facebook Shops, TikTok Shop |
| Conversational | WhatsApp/Messenger Bots | Chat-based commerce |
| In-Store | Kiosk / POS applications | Physical retail integration |

### 3.2 Layer 2 -- Commerce Core Services

Fifteen microservices implement the business logic, each owning its domain data and communicating through Kafka events:

```mermaid
flowchart LR
    subgraph "Product Domain"
        CAT[Catalog]
        INV[Inventory]
        SRC[Search]
    end
    subgraph "Order Domain"
        CHK[Checkout]
        ORD[Orders]
        PAY[Payments]
    end
    subgraph "Fulfillment Domain"
        FUL[Fulfillment]
        SHIP[Shipping]
    end
    subgraph "Engagement Domain"
        LOY[Loyalty]
        SOC[Social Commerce]
        GRP[Group Commerce]
        SUB[Subscriptions]
    end
    subgraph "Experience Domain"
        STF[Storefront]
        THM[Themes]
        ANL[Analytics]
    end

    CAT -->|product.created| SRC
    CAT -->|product.updated| INV
    CHK -->|checkout.completed| ORD
    ORD -->|order.created| INV
    ORD -->|order.created| FUL
    ORD -->|order.created| LOY
    INV -->|inventory.reserved| PAY
    PAY -->|payment.succeeded| FUL
    FUL -->|fulfillment.shipped| SHIP
    SOC -->|social.order| CHK
    GRP -->|campaign.success| ORD
    SUB -->|subscription.renewed| ORD
```

### 3.3 Layer 3 -- Eventing and Workflow ("Nervous System")

Apache Kafka (deployed as Redpanda for development simplicity) serves as the central event backbone. Every significant domain action produces a Kafka event:

| Topic | Producer | Consumers | Purpose |
|-------|----------|-----------|---------|
| product.created | Catalog | Search, Analytics, Social Commerce | Product published notification |
| product.updated | Catalog | Search, Inventory, Storefront | Product change propagation |
| order.created | Orders | Inventory, Fulfillment, Loyalty, Analytics | New order processing |
| inventory.reserved | Inventory | Payments, Orders | Stock reservation confirmation |
| inventory.insufficient | Inventory | Orders, Notifications | Out-of-stock alert |
| payment.succeeded | Payments | Fulfillment, Orders, Notifications | Payment confirmation |
| payment.failed | Payments | Orders, Notifications, Cart Recovery | Payment failure handling |
| fulfillment.shipped | Fulfillment | Shipping, Notifications, Analytics | Shipment created |
| campaign.created | Group Commerce | Social Commerce, Notifications | New group deal launched |
| campaign.success | Group Commerce | Orders, Notifications | Group deal threshold met |
| subscription.renewed | Subscription | Orders, Payments, Notifications | Recurring order triggered |
| loyalty.points_earned | Loyalty | Notifications, Analytics | Points credited |
| cart.abandoned | Checkout | Cart Recovery (n8n), Analytics | Abandoned cart detected |
| search.query | Search | Analytics, AI Recommendations | Search analytics event |

**n8n Workflow Orchestration** listens to Kafka topics and executes multi-step business workflows:

```mermaid
flowchart TD
    KFK[Kafka Event] --> N8N[n8n Trigger Node]
    N8N --> FRAUD{Fraud Check}
    FRAUD -->|Pass| RESERVE[Reserve Inventory]
    FRAUD -->|Fail| CANCEL[Cancel Order + Notify]
    RESERVE --> CHARGE[Charge Payment]
    CHARGE -->|Success| FULFILL[Create Fulfillment]
    CHARGE -->|Fail| RELEASE[Release Inventory + Notify]
    FULFILL --> LABEL[Generate Shipping Label]
    LABEL --> NOTIFY[Send Confirmation Email]
    NOTIFY --> POINTS[Award Loyalty Points]
    POINTS --> ANALYTICS[Record Analytics Event]
```

### 3.4 Layer 4 -- Intelligence and Data ("Brain")

- **Apache Druid** ingests Kafka streams for sub-second analytics queries across sales funnels, CLV, cart abandonment, cohort analysis, and channel attribution.
- **AI Decisioning Plane** provides intelligence-as-a-service for product recommendations, dynamic pricing, fraud detection, search relevance ranking, and marketing attribution.

### 3.5 Layer 5 -- State and Storage

```mermaid
flowchart TB
    subgraph "Polyglot Persistence"
        YDB[(YugabyteDB)]
        SDB[(ScyllaDB)]
        MIO[(MinIO)]
        OS[(OpenSearch)]
        DRD[(Apache Druid)]
        RDS[(Redis)]
    end

    YDB ---|"Orders, Products, Customers,<br/>Subscriptions, Payments"| TX[Transactional Data]
    SDB ---|"Sessions, Cart State,<br/>Wallet Balances, Counters"| RT[Real-Time Data]
    MIO ---|"Product Images, Theme Assets,<br/>UGC, Videos"| OBJ[Object Data]
    OS ---|"Product Index, Search Facets,<br/>Autocomplete"| SRCH[Search Data]
    DRD ---|"Sales Metrics, Funnels,<br/>Cohorts, Attribution"| ANLX[Analytics Data]
    RDS ---|"API Cache, Rate Limits,<br/>Session Tokens"| CACHE[Cache Data]
```

### 3.6 Layer 6 -- Infrastructure and Operations

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Container Orchestration | Kubernetes (Rancher) | Service deployment, autoscaling |
| CI/CD | GitLab CI + ArgoCD | Build, test, GitOps deployment |
| Service Mesh | Istio/Linkerd | mTLS, observability, traffic management |
| Edge/CDN | Cloudflare + Voxility | Global asset delivery, DDoS protection |
| Monitoring | OpenTelemetry + Grafana | Distributed tracing, metrics, logs |
| Secret Management | Vault | Credentials, API keys, certificates |

## 4. Service Communication Patterns

### 4.1 Synchronous (REST/HTTP)

- Client to API Gateway
- API Gateway to individual services for query operations
- Service-to-service for real-time queries (e.g., checkout querying inventory)

### 4.2 Asynchronous (Kafka Events)

- All state-changing operations produce events
- Downstream services consume events for eventual consistency
- n8n workflows orchestrate multi-service processes

### 4.3 CQRS Pattern

```mermaid
flowchart LR
    CMD[Command] --> WRITE[Write Model<br/>YugabyteDB]
    WRITE --> EVENT[Kafka Event]
    EVENT --> READ[Read Model<br/>OpenSearch / Druid]
    QUERY[Query] --> READ
```

Write operations go through the command path to YugabyteDB, producing Kafka events. Read operations query optimized read models in OpenSearch (for product search) or Druid (for analytics).

## 5. Cross-Cutting Concerns

### 5.1 Authentication and Authorization

All API requests pass through the API Gateway, which validates JWT tokens issued by ERP-IAM. Service-to-service calls use mTLS via the service mesh. Entitlements (which features a merchant can access) are managed by ERP-Platform.

### 5.2 Multi-Tenancy

Every data record includes a `tenant_id` field. Row-level security in YugabyteDB ensures data isolation. The API Gateway injects the tenant context from the JWT token into every downstream request.

### 5.3 Observability

```mermaid
flowchart LR
    SVC[Services] -->|Traces| OT[OpenTelemetry Collector]
    SVC -->|Metrics| OT
    SVC -->|Logs| OT
    OT --> TEMPO[Grafana Tempo<br/>Traces]
    OT --> PROM[Prometheus<br/>Metrics]
    OT --> LOKI[Grafana Loki<br/>Logs]
    TEMPO --> GRAF[Grafana Dashboard]
    PROM --> GRAF
    LOKI --> GRAF
```

## 6. Deployment Topology

```mermaid
flowchart TB
    subgraph "Edge Layer"
        CF[Cloudflare CDN]
    end
    subgraph "Kubernetes Cluster"
        subgraph "Ingress"
            IG[Ingress Controller]
        end
        subgraph "Service Pods"
            P1[Catalog x3]
            P2[Orders x3]
            P3[Inventory x3]
            P4[Checkout x3]
            P5[Other Services x2 each]
        end
        subgraph "Data Pods"
            K1[Kafka Brokers x3]
            Y1[YugabyteDB x3]
            S1[ScyllaDB x3]
            D1[Druid Cluster]
        end
    end
    subgraph "Object Storage"
        MIO[MinIO Cluster]
    end

    CF --> IG
    IG --> P1 & P2 & P3 & P4 & P5
    P1 & P2 & P3 & P4 & P5 --> K1
    P1 & P2 & P3 & P4 & P5 --> Y1
    P5 --> S1
    K1 --> D1
    P1 & P5 --> MIO
```

## 7. Technology Stack Summary

| Layer | Technology | Version | Rationale |
|-------|-----------|---------|-----------|
| Runtime | Node.js | 20 LTS | Async I/O, TypeScript native, large ecosystem |
| Language | TypeScript | 5.x | Type safety, developer productivity |
| HTTP Framework | Fastify | 4.x | Fastest Node.js HTTP framework |
| Event Bus | Apache Kafka (Redpanda) | Latest | Event-driven backbone, replay, ordering |
| Workflow Engine | n8n | 1.70+ | Low-code workflow automation |
| Primary DB | YugabyteDB | Latest | Distributed SQL, strong consistency |
| Low-Latency DB | ScyllaDB | Latest | Sub-millisecond reads for sessions/counters |
| Object Storage | MinIO | Latest | S3-compatible, self-hosted |
| Analytics DB | Apache Druid | Latest | Real-time OLAP on event streams |
| Search | OpenSearch | Latest | Full-text search, faceted filtering |
| Cache | Redis | 7.x | API caching, rate limiting, sessions |
| Container | Docker + Kubernetes | Latest | Containerized deployment, orchestration |
