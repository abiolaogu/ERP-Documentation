# ERP-Platform Software Architecture

> **Document ID:** ERP-PLAT-SA-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Model:** C4 Architecture
> **Status:** Approved
> **Related Documents:** [03-Enterprise-Architecture.md](./03-Enterprise-Architecture.md), [12-High-Level-Design.md](./12-High-Level-Design.md), [13-Low-Level-Design.md](./13-Low-Level-Design.md)

---

## 1. Architectural Principles

| ID | Principle | Description |
|----|-----------|-------------|
| P1 | Microservices Independence | Each service is independently deployable, scalable, and replaceable |
| P2 | Zero External Dependencies | Services use Go standard library where possible to minimize supply chain risk |
| P3 | Catalog-Driven Behavior | Product catalog JSON drives subscription resolution, entitlement seeding, and module topology |
| P4 | Event Sourcing for Audit | All state mutations emit CloudEvents to NATS for audit trail reconstruction |
| P5 | Tenant-First Design | Every endpoint validates X-Tenant-ID; data stores enforce row-level security |
| P6 | Health-Check Discovery | Module registry discovers services via periodic `/healthz` polling, not static config |
| P7 | AIDD by Default | AI-driven operations pass through guardrail policy engine before execution |

---

## 2. C4 Level 1: System Context

```mermaid
graph TB
    subgraph "Actors"
        PA["Platform Admin"]
        TA["Tenant Admin"]
        DEV["Developer"]
        AI["AI Agent<br/>(ERP-AI / ERP-Assistant)"]
    end

    ERP["ERP-Platform<br/>(Control Plane)"]

    subgraph "External Systems"
        IAM["ERP-IAM<br/>(Identity Provider)"]
        MODULES["ERP Module Suite<br/>(19 modules)"]
        PAY["Payment Gateway<br/>(Stripe/Adyen)"]
        DNS["DNS Provider<br/>(Route53/Cloudflare)"]
        CERT["Certificate Authority<br/>(Let's Encrypt)"]
        SMTP["Email Service<br/>(SES/SendGrid)"]
    end

    PA -->|Manage platform| ERP
    TA -->|Manage tenant| ERP
    DEV -->|API integration| ERP
    AI -->|Automated actions| ERP

    ERP -->|Authenticate users| IAM
    ERP -->|Discover & manage| MODULES
    ERP -->|Process payments| PAY
    ERP -->|Manage domains| DNS
    ERP -->|Provision SSL| CERT
    ERP -->|Send notifications| SMTP
```

### Context Boundaries

| Boundary | Owned By | Integration Type |
|----------|----------|-----------------|
| Authentication/Authorization | ERP-IAM | OIDC/JWT tokens |
| Module Health Status | Each ERP Module | HTTP GET /healthz |
| Payment Processing | External Payment Gateway | REST API + Webhooks |
| Domain Management | External DNS Provider | REST API |
| Certificate Provisioning | External CA | ACME Protocol |
| Email/SMS Delivery | External Messaging Service | SMTP/REST API |

---

## 3. C4 Level 2: Container Diagram

```mermaid
graph TB
    subgraph "ERP-Platform System Boundary"
        subgraph "Frontend Containers"
            AC["Admin Console<br/>[Next.js 14]<br/>Platform administration UI"]
            ACV["Activation Console<br/>[React + Vite + Tailwind]<br/>Guided onboarding UI"]
        end

        subgraph "API Gateway"
            GW["API Gateway<br/>[Kong/Envoy]<br/>JWT validation, routing,<br/>rate limiting"]
        end

        subgraph "Core Services"
            SH["Subscription Hub<br/>[Go 1.22]<br/>Port 8091<br/>Catalog, subscriptions,<br/>entitlements"]
            TP["Tenant Provisioner<br/>[Go 1.22]<br/>Port 8080<br/>Tenant lifecycle"]
            EE["Entitlement Engine<br/>[Go 1.22]<br/>Port 8080<br/>Runtime entitlement checks"]
        end

        subgraph "Registry & Marketplace"
            MR["Module Registry<br/>[Go 1.22]<br/>Port 8080<br/>Module discovery"]
            MP["Marketplace<br/>[Go 1.22]<br/>Port 8080<br/>Third-party modules"]
        end

        subgraph "Operational Services"
            AS["Audit Service<br/>[Go 1.22]<br/>Port 8080<br/>Immutable audit logs"]
            NH["Notification Hub<br/>[Go 1.22]<br/>Port 8080<br/>Multi-channel notifications"]
            WH["Web Hosting<br/>[Go 1.22]<br/>Port 8080<br/>Domain, SSL, CDN"]
            AW["Activation Wizard<br/>[Go 1.22]<br/>Port 8080<br/>Onboarding flows"]
        end

        subgraph "Data Stores"
            PG["PostgreSQL 16<br/>[Database]<br/>Subscriptions, tenants,<br/>audit logs, entitlements"]
            RD["Redis 7<br/>[Cache]<br/>Catalog cache, session,<br/>rate limiting"]
            NATS["NATS 2.10<br/>[Event Stream]<br/>JetStream for events,<br/>inter-service messaging"]
        end
    end

    AC --> GW
    ACV --> GW
    GW --> SH & TP & EE & MR & MP & AS & NH & WH & AW
    SH & TP & EE & MR & MP & AS & NH & WH & AW --> PG
    SH & TP & EE & MR & MP --> RD
    SH & TP & EE & MR & MP & AS & NH & WH & AW --> NATS
```

### Container Inventory

| Container | Technology | Port | Responsibilities |
|-----------|-----------|------|-----------------|
| Admin Console | Next.js 14, React 18, TypeScript | 3000 | Platform administration, tenant management, subscription management |
| Activation Console | React, Vite, Tailwind CSS | 5173 | Guided onboarding, product selection |
| API Gateway | Kong/Envoy | 8080/443 | JWT validation, rate limiting, request routing |
| Subscription Hub | Go 1.22, stdlib net/http | 8091 | Product catalog, subscription CRUD, entitlement query |
| Tenant Provisioner | Go 1.22, stdlib net/http | 8080 | Tenant creation, configuration, decommissioning |
| Entitlement Engine | Go 1.22, stdlib net/http | 8080 | Runtime entitlement evaluation, feature gating |
| Module Registry | Go 1.22, stdlib net/http | 8080 | Module discovery, health monitoring |
| Marketplace | Go 1.22, stdlib net/http | 8080 | Third-party module listing, installation |
| Audit Service | Go 1.22, stdlib net/http | 8080 | Append-only audit log, event streaming |
| Notification Hub | Go 1.22, stdlib net/http | 8080 | Email, SMS, push, in-app notifications |
| Web Hosting | Go 1.22, stdlib net/http | 8080 | Domain, SSL, CDN management |
| Activation Wizard | Go 1.22, stdlib net/http | 8080 | Guided onboarding flow state |
| PostgreSQL | PostgreSQL 16 | 5432 | Primary relational data store |
| Redis | Redis 7 | 6379 | Cache, session, rate limiting |
| NATS | NATS 2.10 JetStream | 4222 | Event streaming, pub/sub |

---

## 4. C4 Level 3: Component Diagrams

### 4.1 Subscription Hub Components

```mermaid
graph TB
    subgraph "Subscription Hub"
        HC["Health Check Handler<br/>/healthz"]
        PC["Product Catalog Handler<br/>GET /v1/products"]
        SC["Subscription Handler<br/>POST /v1/subscriptions"]
        SG["Subscription Get Handler<br/>GET /v1/subscriptions/{id}"]
        EH["Entitlement Handler<br/>GET /v1/entitlements/{id}"]

        ST["Store<br/>(In-Memory)<br/>RWMutex-protected"]
        CAT["Catalog Loader<br/>JSON file reader"]
        BR["Bundle Resolver<br/>SKU expansion + dedupe"]
        JW["JSON Writer<br/>Response serializer"]
    end

    subgraph "External"
        FS["Filesystem<br/>products.json"]
    end

    HC --> JW
    PC --> CAT --> FS
    PC --> JW
    SC --> BR --> ST
    SC --> JW
    SG --> ST --> JW
    EH --> ST --> JW
```

### 4.2 Tenant Provisioner Components

```mermaid
graph TB
    subgraph "Tenant Provisioner"
        HC2["Health Check Handler<br/>/healthz"]
        TL["Tenant List Handler<br/>GET /v1/tenant-provisioner"]
        TC["Tenant Create Handler<br/>POST /v1/tenant-provisioner"]
        TR["Tenant Read Handler<br/>GET /v1/tenant-provisioner/{id}"]
        TU["Tenant Update Handler<br/>PUT /v1/tenant-provisioner/{id}"]
        TD["Tenant Delete Handler<br/>DELETE /v1/tenant-provisioner/{id}"]

        TV["Tenant Validator<br/>X-Tenant-ID check"]
        EP["Event Publisher<br/>CloudEvents emission"]
        JW2["JSON Writer"]
    end

    TL & TC & TR & TU & TD --> TV
    TC --> EP
    TU --> EP
    TD --> EP
    TL & TC & TR & TU & TD --> JW2
```

### 4.3 Generic Service Component Pattern

All non-subscription services follow an identical component structure:

```mermaid
graph TB
    subgraph "Service Template Pattern"
        H["/healthz Handler"]
        L["List Handler<br/>GET /v1/{service}"]
        C["Create Handler<br/>POST /v1/{service}"]
        R["Read Handler<br/>GET /v1/{service}/{id}"]
        U["Update Handler<br/>PUT|PATCH /v1/{service}/{id}"]
        D["Delete Handler<br/>DELETE /v1/{service}/{id}"]

        VAL["Tenant Validator<br/>X-Tenant-ID required"]
        EVT["Event Topic Publisher<br/>erp.platform.{service}.{action}"]
        JSON["JSON Serializer<br/>writeJSON utility"]
    end

    L & C & R & U & D --> VAL
    C & U & D --> EVT
    H & L & C & R & U & D --> JSON
```

---

## 5. C4 Level 4: Code Patterns

### 5.1 Store Pattern (Subscription Hub)

```go
// Core data structures
type Product struct {
    SKU        string   `json:"sku"`
    Name       string   `json:"name"`
    Repo       string   `json:"repo"`
    Type       string   `json:"type"`       // "module" or "bundle"
    Standalone bool     `json:"standalone"`
    Includes   []string `json:"includes,omitempty"`
}

type Catalog struct {
    Version  string    `json:"version"`
    Products []Product `json:"products"`
}

type SubscriptionRecord struct {
    TenantID string   `json:"tenant_id"`
    PlanType string   `json:"plan_type"`  // "single", "bundle", "suite"
    SKUs     []string `json:"skus"`       // Resolved module SKUs
}

type Store struct {
    mu    sync.RWMutex
    recs  map[string]SubscriptionRecord  // tenant_id -> record
    skus  map[string]Product             // sku -> product
    bunds map[string]Product             // sku -> bundle product
}
```

### 5.2 Service Handler Pattern

```go
// All services follow this pattern
func main() {
    mux := http.NewServeMux()

    // Health check (unauthenticated)
    mux.HandleFunc("/healthz", healthHandler)

    // Collection endpoint (list + create)
    mux.HandleFunc(base, func(w http.ResponseWriter, r *http.Request) {
        // 1. Validate X-Tenant-ID header
        // 2. Switch on HTTP method
        // 3. Emit CloudEvents topic
        // 4. Return JSON response
    })

    // Resource endpoint (read + update + delete)
    mux.HandleFunc(base+"/", func(w http.ResponseWriter, r *http.Request) {
        // 1. Validate X-Tenant-ID header
        // 2. Extract resource ID from path
        // 3. Switch on HTTP method
        // 4. Emit CloudEvents topic
        // 5. Return JSON response
    })

    http.ListenAndServe(":"+port, mux)
}
```

### 5.3 Bundle Resolution Algorithm

```mermaid
flowchart TD
    A[Receive SubscriptionRequest] --> B{Validate tenant_id}
    B -->|Empty| ERR1[Error: tenant_id required]
    B -->|Present| C{Validate plan_type}
    C -->|Invalid| ERR2[Error: invalid plan_type]
    C -->|Valid| D[Iterate SKUs]
    D --> E{SKU exists in catalog?}
    E -->|No| ERR3[Error: unknown SKU]
    E -->|Yes| F{Is bundle SKU?}
    F -->|Yes + plan=single| ERR4[Error: bundle in single plan]
    F -->|Yes + plan!=single| G[Expand bundle includes]
    F -->|No| H[Add SKU directly]
    G --> I[Deduplicate resolved SKUs]
    H --> I
    I --> J[Create SubscriptionRecord]
    J --> K[Store with RWMutex lock]
    K --> L[Return 201 Created]
```

---

## 6. Quality Attributes

### 6.1 Performance

| Attribute | Target | Measurement |
|-----------|--------|-------------|
| Catalog read latency (P99) | < 20ms | Prometheus histogram |
| Entitlement query latency (P99) | < 10ms | Prometheus histogram |
| Subscription creation latency (P99) | < 50ms | Prometheus histogram |
| Tenant provisioning (P99) | < 2s | End-to-end trace |
| Throughput per instance | 10K req/s | Load test (k6) |

### 6.2 Scalability

| Dimension | Approach |
|-----------|----------|
| Horizontal | Kubernetes HPA per service (CPU/memory targets) |
| Vertical | Pod resource limits adjustable per environment |
| Data | PostgreSQL read replicas, Redis cluster sharding |
| Events | NATS JetStream partitioning by tenant |
| Catalog | Immutable versioned JSON; cached in Redis |

### 6.3 Security

| Control | Implementation |
|---------|---------------|
| Authentication | JWT from ERP-IAM (OIDC) |
| Authorization | RBAC roles + entitlement-based access |
| Tenant Isolation | X-Tenant-ID header + PostgreSQL RLS |
| Data Encryption | TLS 1.3 in transit, AES-256 at rest |
| Audit Trail | Immutable append-only logs via audit-service |
| AIDD Guardrails | Policy engine with confidence thresholds |
| Supply Chain | Zero external Go dependencies in core services |

### 6.4 Availability

| Target | Strategy |
|--------|----------|
| 99.95% uptime | Multi-replica deployment (min 2 pods per service) |
| Zero-downtime deploys | Kubernetes rolling updates with readiness probes |
| Self-healing | Liveness probes restart unhealthy pods |
| Circuit breaking | API gateway circuit breaker per upstream service |
| Graceful degradation | Cached entitlements serve during engine downtime |

### 6.5 Observability

| Pillar | Tool | Integration |
|--------|------|-------------|
| Metrics | Prometheus | Go metrics endpoint per service |
| Logging | Structured JSON | ELK/Loki aggregation |
| Tracing | OpenTelemetry | Correlation ID in CloudEvents |
| Alerting | Grafana | SLA-based alert rules |
| Health | /healthz | Kubernetes probes + module registry |

---

## 7. Technology Decisions

| Decision | Choice | Alternatives Considered | Rationale |
|----------|--------|------------------------|-----------|
| Primary Language | Go 1.22 | Java, Rust, Node.js | Fast compilation, small binaries, stdlib HTTP, goroutines |
| HTTP Framework | stdlib net/http | Gin, Echo, Chi | Zero dependencies, production-grade, long-term stability |
| Database | PostgreSQL 16 | MySQL, CockroachDB | RLS, JSON columns, mature tooling, ACID compliance |
| Cache | Redis 7 | Memcached, Hazelcast | Pub/sub, streams, cluster mode, broad ecosystem |
| Events | NATS JetStream | Kafka, Pulsar, RabbitMQ | Low latency, simple operations, JetStream persistence |
| Container | Docker Alpine | Distroless, Debian | Wide compatibility, apk for debugging, small size |
| Orchestration | Kubernetes | Docker Swarm, Nomad | Industry standard, HPA, service mesh ready |
| Frontend | Next.js 14 | Remix, SvelteKit, Angular | SSR, React ecosystem, TypeScript-first |
| CI/CD | GitHub Actions | GitLab CI, Jenkins, CircleCI | Native GitHub integration, marketplace actions |

---

## 8. Deployment Architecture

```mermaid
graph TB
    subgraph "Production Environment"
        subgraph "Region: US-East-1"
            subgraph "K8s Cluster (Primary)"
                NS1["Namespace: erp-platform"]
                NS1 --> D1["subscription-hub x3"]
                NS1 --> D2["tenant-provisioner x2"]
                NS1 --> D3["entitlement-engine x3"]
                NS1 --> D4["module-registry x2"]
                NS1 --> D5["marketplace x2"]
                NS1 --> D6["audit-service x3"]
                NS1 --> D7["notification-hub x2"]
                NS1 --> D8["web-hosting x2"]
                NS1 --> D9["activation-wizard x2"]
            end
            PG1["PostgreSQL Primary"]
            RD1["Redis Primary"]
            NT1["NATS Cluster (3 nodes)"]
        end
        subgraph "Region: EU-West-1 (DR)"
            PG2["PostgreSQL Replica"]
            RD2["Redis Replica"]
        end
    end

    PG1 -->|Streaming Replication| PG2
    RD1 -->|Cross-Region Replication| RD2
```

---

*For high-level design details, see [12-High-Level-Design.md](./12-High-Level-Design.md). For low-level implementation details, see [13-Low-Level-Design.md](./13-Low-Level-Design.md). For ADRs, see [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md).*
