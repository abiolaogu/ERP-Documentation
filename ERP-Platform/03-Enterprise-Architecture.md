# ERP-Platform Enterprise Architecture

> **Document ID:** ERP-PLAT-EA-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Framework:** TOGAF 10
> **Status:** Approved
> **Related Documents:** [04-Software-Architecture.md](./04-Software-Architecture.md), [06-Business-Requirements-Document.md](./06-Business-Requirements-Document.md)

---

## 1. Architecture Vision

ERP-Platform serves as the unified control plane for a 20-module ERP suite, enabling enterprises to manage their entire business technology stack from a single administrative surface. The architecture vision is to deliver "Business at the Speed of Prompt" -- reducing tenant provisioning from weeks to a single API call while maintaining enterprise-grade security, compliance, and multi-tenant isolation.

---

## 2. Business Architecture

### 2.1 Business Capability Map

```mermaid
mindmap
  root((ERP-Platform<br/>Business Capabilities))
    Product Management
      Catalog Management
      SKU Lifecycle
      Bundle Definition
      Pricing Strategy
      Category Taxonomy
    Subscription Management
      Plan Creation
      Plan Modification
      Plan Cancellation
      Bundle Resolution
      Usage Metering
    Tenant Management
      Tenant Provisioning
      Tenant Configuration
      Tenant Decommissioning
      Tenant Isolation
      Resource Allocation
    Entitlement Management
      Entitlement Seeding
      Runtime Evaluation
      Feature Gating
      Usage Limits
    Module Management
      Module Discovery
      Health Monitoring
      Version Tracking
      Dependency Resolution
    Marketplace
      Listing Management
      Installation Lifecycle
      Review and Rating
      Publisher Management
    Platform Operations
      Audit Logging
      Notification Delivery
      Web Hosting
      Activation Guidance
    Compliance
      AIDD Guardrails
      Data Privacy
      Regulatory Reporting
      Policy Enforcement
```

### 2.2 Value Stream Map

```mermaid
graph LR
    subgraph "Customer Acquisition"
        A1[Lead Generation] --> A2[Product Demo]
        A2 --> A3[Plan Selection]
    end
    subgraph "Onboarding"
        A3 --> B1[Subscription Creation]
        B1 --> B2[Tenant Provisioning]
        B2 --> B3[Module Activation]
        B3 --> B4[Activation Wizard]
    end
    subgraph "Steady State"
        B4 --> C1[Daily Operations]
        C1 --> C2[Marketplace Expansion]
        C2 --> C3[Plan Upgrades]
        C3 --> C1
    end
    subgraph "Platform Governance"
        D1[Audit Logging] -.-> C1
        D2[Entitlement Checks] -.-> C1
        D3[Health Monitoring] -.-> C1
        D4[AIDD Guardrails] -.-> C1
    end
```

### 2.3 Business Process Architecture

```mermaid
graph TB
    subgraph "Core Business Processes"
        direction TB
        P1["Tenant Onboarding<br/>(Minutes)"]
        P2["Subscription Management<br/>(Self-Service)"]
        P3["Module Marketplace<br/>(Instant Install)"]
        P4["Platform Administration<br/>(Centralized)"]
    end
    subgraph "Supporting Processes"
        S1[Audit & Compliance]
        S2[Notification Management]
        S3[Web Hosting Operations]
        S4[Health Monitoring]
    end
    subgraph "Management Processes"
        M1[Catalog Governance]
        M2[Entitlement Policy]
        M3[AIDD Guardrail Policy]
        M4[Capacity Planning]
    end
    P1 --> S1
    P2 --> S1
    P3 --> S1
    P4 --> S1
    P1 --> S2
    P2 --> S2
    P3 --> S4
    M1 --> P1
    M2 --> P2
    M3 --> P4
    M4 --> S4
```

### 2.4 Organizational Impact

| Stakeholder | Impact | Benefit |
|-------------|--------|---------|
| IT Operations | Reduced provisioning effort (weeks to minutes) | 95% reduction in manual setup tasks |
| Sales | Faster customer onboarding | Reduced time-to-revenue |
| Finance | Unified billing through subscriptions | Simplified revenue recognition |
| Compliance | Centralized audit trail | Automated regulatory reporting |
| Customer Success | Self-service tenant management | Reduced support ticket volume |
| Development | Module registry auto-discovery | Faster integration testing |

---

## 3. Application Architecture

### 3.1 Application Portfolio

```mermaid
graph TB
    subgraph "ERP-Platform Applications"
        direction TB
        APP1["Admin Console<br/>(Next.js 14)"]
        APP2["Activation Console<br/>(React + Vite)"]
    end
    subgraph "Platform Services"
        SVC1["Subscription Hub"]
        SVC2["Tenant Provisioner"]
        SVC3["Entitlement Engine"]
        SVC4["Module Registry"]
        SVC5["Marketplace"]
        SVC6["Audit Service"]
        SVC7["Notification Hub"]
        SVC8["Web Hosting"]
        SVC9["Activation Wizard"]
    end
    subgraph "ERP Module Suite"
        direction LR
        M1["ERP-CRM"]
        M2["ERP-HCM"]
        M3["ERP-Finance"]
        M4["ERP-Marketing"]
        M5["ERP-Commerce"]
        M6["ERP-eCommerce"]
        M7["ERP-SCM"]
        M8["ERP-Projects"]
        M9["ERP-Workspace"]
        M10["ERP-BI"]
        M11["ERP-AI"]
        M12["ERP-IAM"]
        M13["ERP-iPaaS"]
        M14["ERP-Healthcare"]
        M15["ERP-School-Mgmt"]
        M16["ERP-Church-Mgmt"]
        M17["ERP-BSS-OSS"]
        M18["ERP-Assistant"]
        M19["ERP-Autonomous-Coding"]
    end
    APP1 --> SVC1 & SVC2 & SVC3 & SVC4 & SVC5 & SVC6 & SVC7 & SVC8 & SVC9
    APP2 --> SVC9
    SVC4 -.->|health checks| M1 & M2 & M3 & M4 & M5 & M6 & M7 & M8 & M9 & M10 & M11 & M12 & M13 & M14 & M15 & M16 & M17 & M18 & M19
    SVC3 -->|entitlement gates| M1 & M2 & M3 & M4 & M5 & M6 & M7 & M8 & M9 & M10 & M11 & M12 & M13 & M14 & M15 & M16 & M17 & M18 & M19
```

### 3.2 Module Interaction Matrix

| Source Service | Target Service | Interaction Type | Protocol |
|---------------|---------------|-----------------|----------|
| Subscription Hub | Entitlement Engine | Sync (HTTP) | REST |
| Subscription Hub | Tenant Provisioner | Async (Event) | NATS |
| Tenant Provisioner | Module Registry | Sync (HTTP) | REST |
| Tenant Provisioner | Web Hosting | Async (Event) | NATS |
| Module Registry | All ERP Modules | Sync (HTTP) | Health Check |
| Marketplace | Module Registry | Sync (HTTP) | REST |
| Marketplace | Entitlement Engine | Sync (HTTP) | REST |
| Activation Wizard | Subscription Hub | Sync (HTTP) | REST |
| Activation Wizard | Tenant Provisioner | Sync (HTTP) | REST |
| All Services | Audit Service | Async (Event) | NATS |
| All Services | Notification Hub | Async (Event) | NATS |

### 3.3 Application Communication Patterns

```mermaid
sequenceDiagram
    participant AC as Admin Console
    participant GW as API Gateway
    participant SH as Subscription Hub
    participant TP as Tenant Provisioner
    participant EE as Entitlement Engine
    participant MR as Module Registry
    participant AS as Audit Service
    participant NH as Notification Hub

    AC->>GW: POST /v1/subscriptions
    GW->>SH: Forward (JWT validated)
    SH->>SH: Resolve bundle SKUs
    SH->>EE: Seed entitlements
    SH-->>AS: subscription.created event
    SH->>GW: 201 Created
    GW->>AC: Subscription record

    AC->>GW: POST /v1/tenant-provisioner
    GW->>TP: Forward (JWT validated)
    TP->>EE: Get entitlements
    TP->>MR: Get active modules
    TP->>TP: Provision resources
    TP-->>AS: tenant.provisioned event
    TP-->>NH: Welcome notification
    TP->>GW: 201 Created
    GW->>AC: Tenant record
```

---

## 4. Data Architecture

### 4.1 Data Flow Architecture

```mermaid
graph LR
    subgraph "Data Sources"
        DS1["Product Catalog<br/>(JSON)"]
        DS2["Subscription Requests<br/>(API)"]
        DS3["Health Check Responses<br/>(HTTP)"]
        DS4["Audit Events<br/>(CloudEvents)"]
    end
    subgraph "Data Stores"
        DB1["PostgreSQL<br/>(Relational)"]
        DB2["Redis<br/>(Cache)"]
        DB3["NATS JetStream<br/>(Events)"]
    end
    subgraph "Data Consumers"
        DC1["Admin Console"]
        DC2["Reporting/BI"]
        DC3["Compliance Systems"]
        DC4["External Integrations"]
    end
    DS1 -->|Load at startup| DB1
    DS1 -->|Cache| DB2
    DS2 -->|Persist| DB1
    DS2 -->|Emit events| DB3
    DS3 -->|Store status| DB2
    DS4 -->|Append-only| DB1
    DS4 -->|Stream| DB3
    DB1 --> DC1
    DB1 --> DC2
    DB1 --> DC3
    DB3 --> DC4
```

### 4.2 Data Domain Model

```mermaid
erDiagram
    TENANT ||--o{ SUBSCRIPTION : has
    SUBSCRIPTION ||--o{ ENTITLEMENT : grants
    PRODUCT ||--o{ ENTITLEMENT : "entitled via"
    PRODUCT ||--o{ BUNDLE_ITEM : "included in"
    BUNDLE ||--o{ BUNDLE_ITEM : contains
    TENANT ||--o{ AUDIT_LOG : generates
    MODULE ||--o{ HEALTH_CHECK : reports
    MARKETPLACE_LISTING ||--o{ MODULE : references

    TENANT {
        uuid id PK
        string name
        string domain
        string status
        timestamp created_at
    }
    SUBSCRIPTION {
        uuid id PK
        uuid tenant_id FK
        string plan_type
        string status
        timestamp created_at
    }
    PRODUCT {
        string sku PK
        string name
        string category
        string type
        boolean standalone
        int capabilities
    }
    ENTITLEMENT {
        uuid id PK
        uuid tenant_id FK
        string sku FK
        boolean active
    }
    BUNDLE {
        string sku PK
        string name
        string type
    }
    BUNDLE_ITEM {
        string bundle_sku FK
        string module_sku FK
    }
    AUDIT_LOG {
        uuid id PK
        uuid tenant_id FK
        string event_topic
        json payload
        timestamp created_at
    }
    MODULE {
        string id PK
        string name
        string status
        string health_url
        timestamp last_check
    }
    HEALTH_CHECK {
        uuid id PK
        string module_id FK
        string status
        int latency_ms
        timestamp checked_at
    }
    MARKETPLACE_LISTING {
        uuid id PK
        string module_id FK
        string publisher
        string version
        string status
    }
```

### 4.3 Data Classification

| Data Domain | Classification | Retention | Encryption |
|------------|---------------|-----------|------------|
| Tenant configuration | Confidential | Lifetime of tenant | At rest + in transit |
| Subscription records | Internal | 7 years post-cancellation | At rest + in transit |
| Entitlement data | Internal | Lifetime of subscription | At rest + in transit |
| Audit logs | Restricted | 7 years minimum | At rest + in transit + integrity hash |
| Product catalog | Public | Indefinite | In transit |
| Health check data | Internal | 90 days rolling | In transit |
| Marketplace listings | Public | Lifetime of listing | In transit |
| Notification payloads | Confidential | 30 days | At rest + in transit |

---

## 5. Technology Architecture

### 5.1 Infrastructure Stack

```mermaid
graph TB
    subgraph "Presentation Tier"
        CDN["CDN (CloudFront/Cloudflare)"]
        LB["Load Balancer (Ingress)"]
    end
    subgraph "Application Tier"
        GW["API Gateway (Kong/Envoy)"]
        subgraph "Kubernetes Cluster"
            subgraph "Platform Namespace"
                S1["subscription-hub<br/>Pod (x3)"]
                S2["tenant-provisioner<br/>Pod (x2)"]
                S3["entitlement-engine<br/>Pod (x3)"]
                S4["module-registry<br/>Pod (x2)"]
                S5["marketplace<br/>Pod (x2)"]
                S6["audit-service<br/>Pod (x3)"]
                S7["notification-hub<br/>Pod (x2)"]
                S8["web-hosting<br/>Pod (x2)"]
                S9["activation-wizard<br/>Pod (x2)"]
            end
        end
    end
    subgraph "Data Tier"
        PG["PostgreSQL 16<br/>(Primary + Replica)"]
        RD["Redis 7<br/>(Cluster)"]
        NT["NATS 2.10<br/>(JetStream Cluster)"]
    end
    subgraph "Supporting Infrastructure"
        MON["Prometheus + Grafana"]
        LOG["ELK / Loki"]
        SEC["Vault (Secrets)"]
        REG["Container Registry"]
    end
    CDN --> LB
    LB --> GW
    GW --> S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 --> PG
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 --> RD
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 --> NT
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 -.-> MON
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 -.-> LOG
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 -.-> SEC
```

### 5.2 Technology Standards

| Layer | Standard | Rationale |
|-------|----------|-----------|
| Language | Go 1.22+ | Performance, concurrency, small binaries |
| Container | Docker (Alpine 3.20) | Minimal attack surface, small images |
| Orchestration | Kubernetes 1.29+ | Industry standard, HPA, self-healing |
| Database | PostgreSQL 16 | ACID, RLS, JSON support, mature ecosystem |
| Cache | Redis 7 | Sub-millisecond reads, pub/sub, streams |
| Event Streaming | NATS 2.10 JetStream | Low latency, at-least-once delivery |
| API Gateway | Kong/Envoy | JWT validation, rate limiting, routing |
| Frontend | Next.js 14 / React 18 | SSR, TypeScript, large ecosystem |
| CI/CD | GitHub Actions | Native Git integration, marketplace actions |
| Monitoring | Prometheus + Grafana | Open-source, Kubernetes-native |
| Logging | Structured JSON | Machine-parseable, ELK/Loki compatible |

### 5.3 Network Architecture

```mermaid
graph TB
    subgraph "Internet"
        USER["End Users"]
        DEV["Developers"]
    end
    subgraph "DMZ"
        WAF["Web Application Firewall"]
        CDN2["CDN Edge"]
    end
    subgraph "Public Subnet"
        ING["Kubernetes Ingress"]
    end
    subgraph "Application Subnet"
        PODS["Platform Pods"]
    end
    subgraph "Data Subnet"
        DBSUB["PostgreSQL + Redis + NATS"]
    end
    USER --> WAF --> CDN2 --> ING
    DEV --> WAF
    ING --> PODS
    PODS --> DBSUB
```

---

## 6. Architecture Principles

| # | Principle | Rationale |
|---|-----------|-----------|
| AP-1 | Catalog-Driven Configuration | The product catalog is the single source of truth; adding modules requires no code changes |
| AP-2 | Event-First Integration | Services communicate via events for loose coupling; sync calls only when necessary |
| AP-3 | Tenant Isolation by Default | Every data access path enforces tenant boundaries via RLS and header validation |
| AP-4 | AIDD Guardrails on All AI Operations | No AI action executes without policy evaluation and audit logging |
| AP-5 | Zero-Dependency Services | Platform services minimize external library dependencies for security and stability |
| AP-6 | Health-Check Driven Discovery | Module availability determined by live health checks, not static configuration |
| AP-7 | Immutable Audit Trail | All state-changing operations produce append-only audit records |
| AP-8 | Infrastructure as Code | All infrastructure defined declaratively (Docker Compose, Helm, Terraform) |

---

## 7. Architecture Governance

### 7.1 Decision Authority

| Decision Type | Authority | Approval Required |
|--------------|-----------|------------------|
| New module addition | Architecture Council | Catalog review + health check certification |
| Bundle definition changes | Product Management + Architecture | Catalog versioning + migration plan |
| Technology stack changes | CTO + Architecture Council | ADR required (see [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md)) |
| Security policy changes | CISO + Architecture Council | Threat model review |
| AIDD guardrail threshold changes | AI Ethics Board + Architecture | Impact assessment required |

### 7.2 Compliance Framework

The architecture aligns with:

- **SOC 2 Type II**: Audit logging, access controls, encryption standards
- **GDPR**: Data classification, retention policies, right to erasure
- **HIPAA**: Tenant isolation, audit trails (for healthcare vertical)
- **FERPA**: Student data protection (for education vertical)

---

*For detailed software architecture with C4 diagrams, see [04-Software-Architecture.md](./04-Software-Architecture.md). For business requirements, see [06-Business-Requirements-Document.md](./06-Business-Requirements-Document.md).*
