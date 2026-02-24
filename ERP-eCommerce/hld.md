# High-Level Design -- FusionCommerce (ERP-eCommerce)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This High-Level Design document describes the system-level architecture of FusionCommerce, covering the 15 microservices, their interactions, data flows, event topology, and deployment strategy at a level suitable for architecture review and capacity planning.

## 2. System Overview

FusionCommerce is a composable, event-driven commerce platform implemented as a TypeScript monorepo. It contains 15 microservices, 5 shared packages, multi-platform client applications, and infrastructure configuration for Apache Kafka (Redpanda), n8n workflow engine, and a polyglot persistence layer.

### 2.1 High-Level Component Diagram

```mermaid
flowchart TB
    subgraph "Client Layer"
        WEB["Web Storefront<br/>(Next.js)"]
        MOB["Mobile<br/>(Flutter/Native)"]
        ADM["Admin Dashboard<br/>(React)"]
        SOC_CH["Social Channels<br/>(IG/FB/TT)"]
    end

    subgraph "API Gateway Layer"
        GW["API Gateway<br/>JWT Auth | Rate Limit | Routing"]
    end

    subgraph "Commerce Services"
        direction TB
        subgraph "Product Domain"
            CAT["Catalog :3000"]
            INV["Inventory :3002"]
            SRC["Search :3009"]
        end
        subgraph "Order Domain"
            STF["Storefront :3007"]
            CHK["Checkout :3006"]
            ORD["Orders :3001"]
            PAY["Payments :3004"]
        end
        subgraph "Fulfillment Domain"
            FUL["Fulfillment :3013"]
            SHIP["Shipping :3005"]
        end
        subgraph "Engagement Domain"
            LOY["Loyalty :3012"]
            SOC["Social Commerce :3010"]
            GRP["Group Commerce :3003"]
            SUB["Subscriptions :3011"]
        end
        subgraph "Experience Domain"
            THM["Themes :3008"]
            ANL["Analytics :3014"]
        end
    end

    subgraph "Event & Workflow Layer"
        KFK["Apache Kafka / Redpanda"]
        N8N["n8n Workflow Engine :5678"]
    end

    subgraph "Data Layer"
        YDB[(YugabyteDB)]
        SDB[(ScyllaDB)]
        MIO[(MinIO)]
        DRD[(Apache Druid)]
        OSR[(OpenSearch)]
        RDS[(Redis)]
    end

    WEB & MOB & ADM & SOC_CH --> GW
    GW --> CAT & INV & SRC & STF & CHK & ORD & PAY
    GW --> FUL & SHIP & LOY & SOC & GRP & SUB & THM & ANL

    CAT & ORD & INV & CHK & PAY & FUL & LOY & SOC & GRP & SUB --> KFK
    KFK --> N8N
    KFK --> DRD
    KFK <--> INV & FUL & ANL

    CAT & ORD & INV & PAY & CHK & GRP & SUB & LOY & FUL & SHIP --> YDB
    STF & LOY --> SDB
    THM & CAT & SOC --> MIO
    SRC --> OSR
    ANL --> DRD
    STF & SRC --> RDS
```

## 3. Service Descriptions

### 3.1 Product Domain

| Service | Port | Responsibility | Key APIs |
|---------|------|---------------|----------|
| Catalog | 3000 | Product CRUD, variants, categories, images, reviews | POST/GET/PUT/DELETE /products, /categories, /collections |
| Inventory | 3002 | Stock levels, reservations, multi-warehouse allocation | PUT /inventory/:sku, GET /inventory/:sku, POST /inventory/reserve |
| Search | 3009 | NLQ, full-text, faceted, visual, voice search; merchandising rules | GET /search?q=, POST /search/visual, GET /search/suggest |

### 3.2 Order Domain

| Service | Port | Responsibility | Key APIs |
|---------|------|---------------|----------|
| Storefront | 3007 | Headless API aggregation, cart, wishlist, recently viewed | GET /storefront/products, POST /cart, GET /wishlist |
| Checkout | 3006 | Multi-step checkout, guest checkout, coupon/discount engine | POST /checkout/start, PUT /checkout/:id/shipping, POST /checkout/:id/complete |
| Orders | 3001 | Order lifecycle, status management, order history | POST /orders, GET /orders/:id, PUT /orders/:id/status |
| Payments | 3004 | Payment processing, Stripe/PayPal integration, refunds | POST /payments/intent, POST /payments/confirm, POST /payments/refund |

### 3.3 Fulfillment Domain

| Service | Port | Responsibility | Key APIs |
|---------|------|---------------|----------|
| Fulfillment | 3013 | Pick/pack/ship workflows, multi-warehouse routing, 3PL | POST /fulfillments, PUT /fulfillments/:id/pack, PUT /fulfillments/:id/ship |
| Shipping | 3005 | Label generation, carrier integration, tracking | POST /shipments, GET /shipments/:id/tracking, POST /shipments/rates |

### 3.4 Engagement Domain

| Service | Port | Responsibility | Key APIs |
|---------|------|---------------|----------|
| Loyalty | 3012 | Points, tiers, cashback, digital wallet, gamification | POST /loyalty/earn, POST /loyalty/redeem, GET /loyalty/balance |
| Social Commerce | 3010 | Instagram/Facebook/TikTok sync, livestream, referrals | POST /social/sync, POST /social/livestream, GET /social/referrals |
| Group Commerce | 3003 | Group buying campaigns, participant management | POST /campaigns, POST /campaigns/:id/join, GET /campaigns/:id |
| Subscriptions | 3011 | Subscription plans, recurring billing, skip/pause/swap | POST /subscriptions, PUT /subscriptions/:id/skip, PUT /subscriptions/:id/swap |

### 3.5 Experience Domain

| Service | Port | Responsibility | Key APIs |
|---------|------|---------------|----------|
| Themes | 3008 | Visual builder, template rendering, theme management | GET /themes, POST /themes/:id/customize, GET /themes/:id/render |
| Analytics | 3014 | Druid queries, funnel analysis, cohort, attribution | GET /analytics/funnel, GET /analytics/cohort, GET /analytics/clv |

## 4. Event Flow Architecture

### 4.1 Order Processing Saga

```mermaid
sequenceDiagram
    participant C as Consumer
    participant CHK as Checkout
    participant ORD as Orders
    participant INV as Inventory
    participant PAY as Payments
    participant FUL as Fulfillment
    participant SHIP as Shipping
    participant LOY as Loyalty
    participant N8N as n8n
    participant KFK as Kafka

    C->>CHK: Complete Checkout
    CHK->>ORD: Create Order
    ORD->>KFK: order.created
    KFK->>INV: Consume order.created
    INV->>INV: Reserve Stock
    alt Stock Available
        INV->>KFK: inventory.reserved
        KFK->>N8N: Trigger Payment Workflow
        N8N->>PAY: Process Payment
        PAY->>KFK: payment.succeeded
        KFK->>FUL: Create Fulfillment
        FUL->>FUL: Pick & Pack
        FUL->>KFK: fulfillment.shipped
        KFK->>SHIP: Generate Label
        SHIP->>C: Tracking Number Email
        KFK->>LOY: Award Points
    else Stock Insufficient
        INV->>KFK: inventory.insufficient
        KFK->>ORD: Cancel Order
        KFK->>C: Out of Stock Notification
    end
```

### 4.2 Cart Abandonment Recovery

```mermaid
sequenceDiagram
    participant C as Consumer
    participant CHK as Checkout
    participant KFK as Kafka
    participant N8N as n8n
    participant ANL as Analytics
    participant EMAIL as SendGrid

    C->>CHK: Add Items to Cart
    Note over CHK: 30 min timer starts
    CHK-->>CHK: No activity detected
    CHK->>KFK: cart.abandoned
    KFK->>ANL: Record abandonment event
    KFK->>N8N: Trigger recovery workflow
    N8N->>N8N: Wait 1 hour
    N8N->>EMAIL: Send recovery email (10% off)
    alt Customer Returns
        C->>CHK: Complete checkout
        CHK->>KFK: cart.recovered
    else No Response
        N8N->>N8N: Wait 24 hours
        N8N->>EMAIL: Send 2nd email (15% off)
    end
```

### 4.3 Group Buying Flow

```mermaid
sequenceDiagram
    participant CR as Campaign Creator
    participant GRP as Group Commerce
    participant KFK as Kafka
    participant P1 as Participant 1
    participant P2 as Participant 2
    participant ORD as Orders

    CR->>GRP: Create Campaign (min 5 participants)
    GRP->>KFK: campaign.created
    P1->>GRP: Join Campaign
    GRP->>KFK: campaign.joined (1/5)
    P2->>GRP: Join Campaign
    GRP->>KFK: campaign.joined (2/5)
    Note over GRP: ... more participants join ...
    GRP->>GRP: Threshold reached (5/5)
    GRP->>KFK: campaign.success
    KFK->>ORD: Create group orders for all participants
    ORD->>KFK: order.created (x5)
```

## 5. Data Flow Summary

```mermaid
flowchart LR
    subgraph "Write Path"
        API[API Request] --> SVC[Service]
        SVC --> DB[(YugabyteDB)]
        SVC --> KFK[Kafka Event]
    end

    subgraph "Read Path - Search"
        KFK -->|Index| OS[(OpenSearch)]
        SRCH[Search Query] --> OS
    end

    subgraph "Read Path - Analytics"
        KFK -->|Ingest| DRD[(Druid)]
        ANLQ[Analytics Query] --> DRD
    end

    subgraph "Real-Time Path"
        KFK -->|Session| SDB[(ScyllaDB)]
        RT[Storefront Request] --> SDB
    end
```

## 6. Scalability Design

### 6.1 Horizontal Scaling Strategy

| Component | Scaling Approach | Min Replicas | Max Replicas | Scale Trigger |
|-----------|-----------------|-------------|-------------|---------------|
| Catalog | HPA (CPU/Memory) | 2 | 10 | CPU > 70% |
| Orders | HPA (CPU/Memory) | 3 | 15 | CPU > 60% |
| Checkout | HPA (Custom - active carts) | 3 | 20 | Active carts > 5000 |
| Search | HPA (Latency) | 3 | 12 | p99 > 80ms |
| Storefront | HPA (RPS) | 3 | 25 | RPS > 1000/pod |
| Kafka | Partition-based | 3 brokers | 9 brokers | Partition lag > 10K |
| YugabyteDB | Tablet splitting | 3 nodes | 9 nodes | Tablet size > 10GB |

### 6.2 Performance Targets

```mermaid
flowchart LR
    subgraph "Latency Targets (p99)"
        A["Storefront API<br/>< 100ms"]
        B["Search Query<br/>< 50ms"]
        C["Cart Operations<br/>< 200ms"]
        D["Checkout Complete<br/>< 500ms"]
        E["Analytics Query<br/>< 2s"]
    end
```

## 7. Failure Handling

### 7.1 Circuit Breaker Pattern

```mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open: Failure threshold exceeded
    Open --> HalfOpen: Timeout elapsed
    HalfOpen --> Closed: Probe succeeds
    HalfOpen --> Open: Probe fails
```

Applied to all external service calls (Stripe, EasyPost, social platform APIs) with configurable thresholds:
- **Failure threshold**: 5 consecutive failures
- **Timeout**: 30 seconds
- **Probe interval**: 10 seconds

### 7.2 Dead Letter Queue Strategy

Failed Kafka event processing routes to DLQ topics for manual review and replay:

```
order.created          -> order.created.dlq
payment.succeeded      -> payment.succeeded.dlq
inventory.reserved     -> inventory.reserved.dlq
```

## 8. Security Architecture

```mermaid
flowchart TB
    EXT[External Traffic] --> WAF[Cloudflare WAF]
    WAF --> TLS[TLS Termination]
    TLS --> GW[API Gateway]
    GW --> JWT[JWT Validation]
    JWT --> RBAC[Role-Based Access]
    RBAC --> SVC[Service]
    SVC --> MTLS[mTLS<br/>Service Mesh]
    MTLS --> SVC2[Downstream Service]
    SVC --> ENC[Encrypted Storage]
```

| Layer | Control | Implementation |
|-------|---------|----------------|
| Edge | WAF, DDoS protection | Cloudflare |
| Transport | TLS 1.3 | Let's Encrypt certificates |
| Authentication | JWT validation | ERP-IAM OIDC |
| Authorization | RBAC + ABAC | Per-service policy enforcement |
| Service-to-Service | mTLS | Istio service mesh |
| Data at Rest | AES-256 encryption | YugabyteDB encryption |
| PCI Compliance | Tokenization | Stripe.js / PayPal.js (client-side) |
