# ERP-CRM Enterprise Architecture

## 1. Business Architecture

### 1.1 Business Capability Model

```mermaid
graph TB
    subgraph "ERP-CRM Business Capabilities"
        subgraph "Sales Management"
            SM1["Contact Management"]
            SM2["Company Management"]
            SM3["Pipeline Management"]
            SM4["Opportunity Tracking"]
            SM5["Deal Forecasting"]
            SM6["Territory Management"]
        end
        subgraph "Marketing Operations"
            MO1["Lead Capture"]
            MO2["Lead Scoring"]
            MO3["Lead Nurturing"]
            MO4["Form Builder"]
            MO5["Web-to-Lead"]
            MO6["Campaign Tracking"]
        end
        subgraph "Customer Service"
            CS1["Ticket Management"]
            CS2["SLA Management"]
            CS3["Knowledge Base"]
            CS4["Live Chat"]
            CS5["Escalation Management"]
            CS6["Customer Self-Service"]
        end
        subgraph "Automation & Intelligence"
            AI1["Workflow Automation"]
            AI2["Assignment Rules"]
            AI3["AI Lead Scoring"]
            AI4["AI Article Suggestions"]
            AI5["Chatbot Automation"]
            AI6["Predictive Analytics"]
        end
        subgraph "Analytics & Reporting"
            AR1["Dashboard Builder"]
            AR2["Funnel Analytics"]
            AR3["Win/Loss Analysis"]
            AR4["Activity Reports"]
            AR5["SLA Compliance"]
            AR6["Revenue Forecasting"]
        end
    end
```

### 1.2 Value Stream Mapping

```mermaid
graph LR
    L["Lead Capture<br/>(Forms, Chat, Web)"] --> Q["Lead Qualification<br/>(AI Scoring)"]
    Q --> N["Nurturing<br/>(Email, Activities)"]
    N --> O["Opportunity<br/>(Pipeline Stages)"]
    O --> C["Close<br/>(Won/Lost)"]
    C --> S["Support<br/>(Helpdesk, KB)"]
    S --> R["Retention<br/>(Renewals, Upsell)"]
    R --> A["Advocacy<br/>(Evangelist Stage)"]
```

### 1.3 Stakeholder Map

| Stakeholder | Role | Primary Concerns |
|------------|------|-----------------|
| Sales Representatives | End User | Pipeline visibility, contact 360 view, mobile access |
| Sales Managers | End User | Forecasting, territory management, team performance |
| Support Agents | End User | Ticket queue, SLA tracking, knowledge base |
| Marketing Team | End User | Lead capture, scoring, form builder |
| CRM Administrator | Admin | Configuration, automation rules, reporting |
| IT Operations | Technical | Deployment, monitoring, security, compliance |
| Executive Leadership | Sponsor | Revenue insights, customer health, ROI |
| Customers | External | Self-service portal, chat, knowledge base |

## 2. Application Architecture

### 2.1 Application Portfolio

```mermaid
graph TB
    subgraph "ERP Suite"
        IAM["ERP-IAM<br/>(Identity & Access)"]
        PLAT["ERP-Platform<br/>(Control Plane)"]
        DIR["ERP-Directory<br/>(User Provisioning)"]
        CRM["ERP-CRM<br/>(This Module)"]
    end

    subgraph "ERP-CRM Internal"
        CORE["Rust Monolith Core"]
        MS1["12 Go Microservices"]
        WEB["React Web App"]
        MOB["Flutter Mobile"]
        NAT["Native Apps"]
    end

    IAM -->|OIDC/JWT| CRM
    PLAT -->|Entitlements| CRM
    DIR -->|Provisioning| CRM
    CRM --> CORE
    CRM --> MS1
    CRM --> WEB
    CRM --> MOB
    CRM --> NAT
```

### 2.2 Integration Architecture

```mermaid
flowchart TB
    subgraph "External Systems"
        EMAIL["Email Providers<br/>(SMTP/IMAP)"]
        CAL["Calendar Systems<br/>(CalDAV/Google/O365)"]
        ENRICH["Data Enrichment<br/>(Clearbit/ZoomInfo)"]
        PHONE["Telephony<br/>(VoIP/SIP)"]
        PAY["Payment Gateways"]
    end

    subgraph "ERP-CRM"
        GW["API Gateway"]
        EVT["Event Bus<br/>(NATS + Pulsar)"]
        HOOK["Webhook Engine"]
    end

    subgraph "ERP Platform"
        IAM2["ERP-IAM"]
        PLAT2["ERP-Platform"]
        ACCT["ERP-Accounting"]
        INV["ERP-Inventory"]
    end

    EMAIL <-->|IMAP/SMTP| GW
    CAL <-->|CalDAV| GW
    ENRICH -->|REST API| GW
    PHONE <-->|SIP/WebRTC| GW
    PAY <-->|REST| GW

    GW <--> EVT
    EVT <--> HOOK

    IAM2 -->|JWT Validation| GW
    PLAT2 <-->|License/Feature Flags| GW
    EVT -->|deal.won| ACCT
    EVT -->|order.created| INV
```

### 2.3 Application Decomposition

| Application Component | Technology | Deployment | Scaling Strategy |
|-----------------------|-----------|------------|-----------------|
| CRM Core Monolith | Rust (axum) | Docker/K8s | Horizontal (stateless) |
| Contact Service | Go | Docker/K8s | Horizontal |
| Lead Service | Go | Docker/K8s | Horizontal |
| Pipeline Service | Go | Docker/K8s | Horizontal |
| Opportunity Service | Go | Docker/K8s | Horizontal |
| Activity Service | Go | Docker/K8s | Horizontal |
| Helpdesk Service | Go | Docker/K8s | Horizontal |
| Knowledge Base Service | Go | Docker/K8s | Horizontal |
| Form Builder Service | Go | Docker/K8s | Horizontal |
| Chat Service | Go | Docker/K8s | Horizontal (WebSocket affinity) |
| Automation Service | Go | Docker/K8s | Horizontal |
| Reporting Service | Go | Docker/K8s | Vertical (memory-bound) |
| Territory Service | Go | Docker/K8s | Horizontal |
| Web Frontend | React (static) | CDN/Nginx | CDN edge caching |
| Mobile Frontend | Flutter | App Stores | N/A (client-side) |
| GraphQL Engine | Hasura | Docker/K8s | Horizontal |

## 3. Data Architecture

### 3.1 Data Flow Architecture

```mermaid
flowchart LR
    subgraph "Sources"
        UI["User Interface"]
        API["External APIs"]
        FORM["Web Forms"]
        CHAT["Live Chat"]
        EMAIL2["Email Integration"]
    end

    subgraph "Ingestion"
        REST2["REST Endpoints"]
        GQL2["GraphQL Mutations"]
        WH["Webhook Receivers"]
    end

    subgraph "Processing"
        CMD["Command Handlers"]
        EVT2["Event Processors"]
        AUTO["Automation Engine"]
    end

    subgraph "Storage"
        PG2["PostgreSQL 16<br/>(Primary Store)"]
        QW2["Quickwit<br/>(Search/Logs)"]
        CACHE["In-Memory Cache<br/>(DashMap)"]
    end

    subgraph "Delivery"
        NATS2["NATS JetStream"]
        PULSAR2["Apache Pulsar"]
        WHOUT["Outbound Webhooks"]
    end

    UI --> REST2 & GQL2
    API --> REST2
    FORM --> WH
    CHAT --> REST2
    EMAIL2 --> WH

    REST2 & GQL2 & WH --> CMD
    CMD --> PG2
    CMD --> EVT2
    EVT2 --> AUTO
    EVT2 --> NATS2 & PULSAR2
    NATS2 & PULSAR2 --> WHOUT
    PG2 --> QW2
```

### 3.2 Data Classification

| Data Category | Sensitivity | Retention | Encryption |
|--------------|------------|-----------|-----------|
| Contact PII (email, phone, name) | High | Per tenant policy | AES-256 at rest, TLS in transit |
| Company Information | Medium | Indefinite | TLS in transit |
| Deal Financial Data | High | 7 years (regulatory) | AES-256 at rest, TLS in transit |
| Activity Logs | Medium | 2 years | TLS in transit |
| Ticket Content | Medium | Per tenant policy | TLS in transit |
| Form Submissions | High (may contain PII) | Per form policy | AES-256 at rest, TLS in transit |
| Chat Transcripts | Medium | 1 year default | TLS in transit |
| Audit Events | High | 7 years | Immutable, signed |
| Analytics/Reports | Low | 90 days cache | TLS in transit |

## 4. Technology Architecture

### 4.1 Infrastructure Topology

```mermaid
graph TB
    subgraph "Kubernetes Cluster (Harvester HCI)"
        subgraph "Ingress"
            LB["Load Balancer"]
            INGRESS["Nginx Ingress Controller"]
        end

        subgraph "Application Pods"
            CRM_POD["CRM Core<br/>(2+ replicas)"]
            SVC_PODS["12 Service Pods<br/>(1+ replicas each)"]
            WEB_POD["Web Frontend<br/>(Nginx)"]
            HASURA_POD["Hasura GraphQL<br/>(2+ replicas)"]
        end

        subgraph "Data Pods"
            PG_POD["PostgreSQL 16<br/>(Primary + Replica)"]
            NATS_POD["NATS JetStream<br/>(3-node cluster)"]
        end

        subgraph "Platform Pods"
            PULSAR_POD["Apache Pulsar<br/>(3+ bookies)"]
            QW_POD["Quickwit<br/>(Searcher + Indexer)"]
        end

        subgraph "Storage"
            MAYASTOR["Mayastor/Vitastor<br/>Persistent Volumes"]
        end
    end

    LB --> INGRESS
    INGRESS --> CRM_POD & SVC_PODS & WEB_POD & HASURA_POD
    CRM_POD --> PG_POD & NATS_POD
    SVC_PODS --> PG_POD & PULSAR_POD
    PG_POD --> MAYASTOR
    NATS_POD --> MAYASTOR
    PULSAR_POD --> MAYASTOR
    QW_POD --> MAYASTOR
```

### 4.2 Technology Standards

| Standard | Implementation |
|----------|---------------|
| Authentication | OIDC/JWT via ERP-IAM |
| Authorization | RBAC with tenant isolation |
| API Style | REST (core) + GraphQL (frontend) |
| Messaging | Apache Pulsar (async inter-service) |
| Internal Events | NATS JetStream (real-time) |
| Observability | Quickwit (logs) + shared log schema |
| Storage | Mayastor/Vitastor on Harvester HCI |
| CI/CD | GitHub Actions |
| Containerization | Docker (multi-stage) |
| Orchestration | Kubernetes |
| Event Format | CloudEvents v1.0 |

## 5. Governance

### 5.1 Architecture Principles

1. **Sovereign-first**: Self-hosted, no external SaaS dependencies for core functionality
2. **Event-driven**: All state changes emit domain events for reactive architecture
3. **DDD alignment**: Business logic encapsulated in domain aggregates, not in handlers
4. **Hexagonal isolation**: Infrastructure concerns never leak into domain logic
5. **Multi-tenant by design**: All data scoped by `X-Tenant-ID`
6. **Performance-first**: Rust core for latency-critical paths, Go for CRUD services
7. **Compliance-as-code**: SOC2/HIPAA/PCI-DSS controls tracked as Git artifacts

### 5.2 Architecture Review Checklist

- [ ] All new services follow the standard Go microservice template
- [ ] Domain events published for all state-changing operations
- [ ] Tenant isolation verified at API gateway and database layers
- [ ] Performance benchmarks established for critical paths
- [ ] Compliance artifacts updated in `COMPLIANCE.md`
- [ ] Architecture Decision Record created for significant changes
