# System Architecture -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Architecture Overview

ERP-BSS-OSS is a cloud-native, microservices-based BSS/OSS platform built on Rust (Axum) with polyglot persistence and event-driven communication. The architecture follows Domain-Driven Design (DDD) principles with bounded contexts aligned to TM Forum Frameworx (eTOM process areas).

---

## 2. C4 Model

### 2.1 Context Diagram (Level 1)

```mermaid
graph TB
    subgraph "External Users"
        SUBS["Subscribers<br/>(Prepaid + Postpaid)"]
        CSR["Customer Service Reps"]
        ADMIN["BSS Administrators"]
        PARTNER["MVNO Partners"]
        NOC_OP["NOC Operators"]
        UTIL["Utility Customers"]
    end

    subgraph "ERP-BSS-OSS Platform"
        BSS["BSS/OSS<br/>Microservices Platform"]
    end

    subgraph "External Systems"
        HLR["HLR/HSS<br/>(Network)"]
        SMPP_GW["SMS Gateway<br/>(SMPP)"]
        PAY_GW["Payment Gateways<br/>(Paystack, Flutterwave, Stripe)"]
        NPC["Number Portability<br/>Clearinghouse"]
        SMDP["SM-DP+<br/>(eSIM)"]
        AMI_HE["AMI Head-End<br/>(Smart Meters)"]
        ERP_SUITE["ERP Suite<br/>(IAM, Finance, CRM, BI, AI)"]
        TAX_ENG["Tax Engine<br/>(Avalara/Vertex)"]
    end

    SUBS -->|Self-Care, USSD| BSS
    CSR -->|Admin Console| BSS
    ADMIN -->|Configuration| BSS
    PARTNER -->|Partner Portal| BSS
    NOC_OP -->|NOC Dashboard| BSS
    UTIL -->|Meter Portal| BSS

    BSS -->|SS7/DIAMETER| HLR
    BSS -->|SMPP| SMPP_GW
    BSS -->|REST| PAY_GW
    BSS -->|SOAP/REST| NPC
    BSS -->|REST/ES2+| SMDP
    BSS -->|DLMS/COSEM| AMI_HE
    BSS -->|REST/Events| ERP_SUITE
    BSS -->|REST| TAX_ENG
```

### 2.2 Container Diagram (Level 2)

```mermaid
graph TB
    subgraph "API Layer"
        GW["API Gateway<br/>(Kong 3.5)"]
        MESH["Service Mesh<br/>(Istio)"]
    end

    subgraph "BSS Services"
        PC["Product Catalog<br/>TMF620<br/>(Rust/Go)"]
        CM["Customer Management<br/>TMF629<br/>(Rust/Go)"]
        OM["Order Management<br/>TMF622<br/>(Rust/Go)"]
        BR["Billing & Rating<br/>TMF678<br/>(Rust/Go)"]
        CE["Charging Engine<br/>OCS<br/>(Rust)"]
        MED["Mediation<br/>CDR Pipeline<br/>(Rust/Go)"]
        PS["Partner Service<br/>TMF668<br/>(Go)"]
        SC["Self-Care<br/>Portal<br/>(Go)"]
    end

    subgraph "OSS Services"
        PROV["Provisioning<br/>TMF641<br/>(Go)"]
        RI["Resource Inventory<br/>TMF639<br/>(Go)"]
        SI["Service Inventory<br/>TMF638<br/>(Go)"]
        NO["Network Operations<br/>Fault/Perf<br/>(Go)"]
        WF["Workforce Mgmt<br/>Field Dispatch<br/>(Rust)"]
    end

    subgraph "Specialized Services"
        RA["Revenue Assurance<br/>AI/ML<br/>(Go)"]
        USSD["USSD/IVR<br/>Gateway<br/>(Go)"]
        MM["Meter Management<br/>AMI<br/>(Go)"]
        TS["Tariff Service<br/>STS Tokens<br/>(Go)"]
    end

    subgraph "Data Layer"
        PG["PostgreSQL 16"]
        RD["Redis 7"]
        MG["MongoDB 7"]
        CH["ClickHouse"]
        KF["Kafka / Redpanda"]
        RMQ["RabbitMQ"]
    end

    GW --> MESH
    MESH --> PC & CM & OM & BR & CE & MED & PS & SC
    MESH --> PROV & RI & SI & NO & WF
    MESH --> RA & USSD & MM & TS

    PC & CM & OM & BR --> PG
    CE --> RD
    MED --> CH
    NO --> MG
    BR & OM & CM --> KF
    CE & PROV --> RMQ
```

### 2.3 Component Diagram -- Billing Service (Level 3)

```mermaid
graph TB
    subgraph "Billing & Rating Service"
        API["REST API Layer<br/>(Axum handlers)"]
        DOMAIN["Domain Layer"]
        REPO["Repository Layer"]

        subgraph "Domain Components"
            RATER["Rating Engine"]
            INVOICER["Invoice Generator"]
            TAX["Tax Calculator"]
            DUNNING["Dunning Manager"]
            DISPUTE["Dispute Handler"]
            DISCOUNT["Discount Engine"]
        end

        subgraph "External Adapters"
            PG_REPO["PostgreSQL Repo"]
            CH_REPO["ClickHouse Repo"]
            KF_PUB["Kafka Publisher"]
            PAY_CL["Payment Gateway Client"]
            TAX_CL["Tax Engine Client"]
        end
    end

    API --> DOMAIN
    DOMAIN --> RATER & INVOICER & TAX & DUNNING & DISPUTE & DISCOUNT
    DOMAIN --> REPO
    REPO --> PG_REPO & CH_REPO
    DOMAIN --> KF_PUB & PAY_CL & TAX_CL
```

---

## 3. Service Inventory

The platform comprises **30 microservices** organized into four domains:

| Domain | Services | Count |
|--------|----------|-------|
| **BSS Core** | product-catalog-service, customer-management-service, order-management-service, billing-rating-service, charging-engine, mediation-service, partner-service, self-care-service | 8 |
| **OSS Core** | provisioning-service, resource-inventory-service, service-inventory-service, network-operations-service, fault-management, performance-mgmt, workforce-mgmt, network-orchestration, network-inventory | 9 |
| **Specialized** | revenue-assurance-service, ussd-ivr-gateway, meter-management-service, tariff-service | 4 |
| **Foundation** | api-gateway, crm-service, finance-service, support-service, customer-management, order-management, product-catalog, partner-settlement, service-provisioning | 9 |

---

## 4. Rust Crate Architecture

```mermaid
graph TB
    subgraph "Application Crates"
        API["bss-api<br/>REST + GraphQL"]
        BILLING["bss-billing<br/>Rating + Invoicing"]
        CRM["bss-crm<br/>Contacts + Leads"]
        ORDERING["bss-ordering<br/>Order State Machine"]
        INVENTORY["bss-inventory<br/>Resource Tracking"]
        ANALYTICS["bss-analytics<br/>ClickHouse Reports"]
        INTEGRATION["bss-integration<br/>External Connectors"]
    end

    subgraph "Foundation Crates"
        CORE["bss-core<br/>DB + Cache + MQ"]
        DDD["bss-ddd<br/>Aggregate + Entity + ValueObject"]
    end

    API --> CORE & DDD
    BILLING --> CORE & DDD
    CRM --> CORE & DDD
    ORDERING --> CORE & DDD
    INVENTORY --> CORE & DDD
    ANALYTICS --> CORE
    INTEGRATION --> CORE
```

---

## 5. Event-Driven Architecture

### 5.1 Event Bus Topology

All inter-service communication uses CloudEvents envelope format published to Kafka/Redpanda topics.

**Topic naming convention:** `erp.bss_oss.<entity>.<action>`

```mermaid
graph LR
    subgraph "Producers"
        P_OM["Order Management"]
        P_BR["Billing/Rating"]
        P_CM["Customer Management"]
        P_PROV["Provisioning"]
        P_MED["Mediation"]
    end

    subgraph "Event Bus (Kafka/Redpanda)"
        T1["erp.bss_oss.order-management.*"]
        T2["erp.bss_oss.billing-rating.*"]
        T3["erp.bss_oss.customer-management.*"]
        T4["erp.bss_oss.provisioning.*"]
        T5["erp.bss_oss.mediation.*"]
    end

    subgraph "Consumers"
        C_BR["Billing/Rating"]
        C_PROV["Provisioning"]
        C_RA["Revenue Assurance"]
        C_NOTIFY["Notification Service"]
        C_ANALYTICS["Analytics"]
    end

    P_OM --> T1
    P_BR --> T2
    P_CM --> T3
    P_PROV --> T4
    P_MED --> T5

    T1 --> C_BR & C_PROV & C_ANALYTICS
    T2 --> C_RA & C_NOTIFY & C_ANALYTICS
    T3 --> C_BR & C_ANALYTICS
    T4 --> C_NOTIFY & C_ANALYTICS
    T5 --> C_BR & C_RA & C_ANALYTICS
```

### 5.2 Event Catalog

| Topic | Payload | Producer | Consumers |
|-------|---------|----------|-----------|
| `erp.bss_oss.order-management.created` | Order + Items | Order Service | Billing, Provisioning, Analytics |
| `erp.bss_oss.order-management.updated` | Order delta | Order Service | Billing, Notification |
| `erp.bss_oss.billing-rating.created` | Invoice / Charge | Billing Service | Revenue Assurance, Analytics |
| `erp.bss_oss.customer-management.created` | Customer record | Customer Service | Billing, CRM, Analytics |
| `erp.bss_oss.provisioning.created` | Provisioning task | Provisioning | Network Ops, Analytics |
| `erp.bss_oss.mediation.created` | Normalized CDR | Mediation | Billing, Revenue Assurance |
| `erp.bss_oss.revenue-assurance.created` | Alert / Finding | Revenue Assurance | Notification, Analytics |

---

## 6. Data Architecture

### 6.1 Polyglot Persistence Strategy

```mermaid
graph TB
    subgraph "OLTP (PostgreSQL / YugabyteDB)"
        PG_CUST["Customers"]
        PG_ORD["Orders"]
        PG_PROD["Products"]
        PG_SUB["Subscriptions"]
        PG_INV["Invoices"]
        PG_BAL["Balances"]
        PG_PART["Partners"]
    end

    subgraph "Cache (Redis / DragonflyDB)"
        RD_BAL["Balance Cache (60s TTL)"]
        RD_RATE["Rate Cache (300s TTL)"]
        RD_SESS["Session Cache (3600s TTL)"]
        RD_RL["Rate Limiter (60s TTL)"]
    end

    subgraph "Analytics (ClickHouse)"
        CH_CDR["CDR Analytics"]
        CH_REV["Revenue Rollup"]
        CH_PERF["Performance Metrics"]
        CH_CHURN["Churn Indicators"]
    end

    subgraph "Documents (MongoDB)"
        MG_AUDIT["Audit Logs"]
        MG_CONFIG["Dynamic Config"]
        MG_ALARM["Network Alarms"]
    end

    subgraph "Messaging (Kafka + RabbitMQ)"
        KF_EVT["Domain Events"]
        RMQ_CMD["Command Queue"]
    end
```

### 6.2 Database Sizing (per 1M subscribers)

| Database | Storage | IOPS | Connections |
|----------|---------|------|------------|
| PostgreSQL | 500 GB | 10K | 200 |
| Redis | 32 GB RAM | N/A | 500 |
| ClickHouse | 2 TB | 5K | 50 |
| MongoDB | 100 GB | 2K | 100 |
| Kafka | 1 TB (7-day retention) | 20K | 300 |

---

## 7. Network Architecture

### 7.1 Global 6-PoP Deployment

```mermaid
graph TB
    DNS["AWS Route 53<br/>Geo-Proximity Routing"]

    subgraph "Lagos, Nigeria"
        L_K8S["K8s Cluster<br/>3 worker nodes"]
        L_DB["PostgreSQL Primary<br/>+ 2 Replicas"]
    end

    subgraph "London, UK"
        LN_K8S["K8s Cluster<br/>3 worker nodes"]
        LN_DB["PostgreSQL Primary<br/>+ 2 Replicas"]
    end

    subgraph "Ashburn, US"
        A_K8S["K8s Cluster<br/>3 worker nodes"]
        A_DB["PostgreSQL Primary<br/>+ 2 Replicas"]
    end

    subgraph "Mumbai, India"
        M_K8S["K8s Cluster<br/>3 worker nodes"]
        M_DB["PostgreSQL Primary<br/>+ 2 Replicas"]
    end

    subgraph "Singapore"
        S_K8S["K8s Cluster<br/>3 worker nodes"]
        S_DB["PostgreSQL Primary<br/>+ 2 Replicas"]
    end

    subgraph "Sao Paulo, Brazil"
        SP_K8S["K8s Cluster<br/>3 worker nodes"]
        SP_DB["PostgreSQL Primary<br/>+ 2 Replicas"]
    end

    DNS --> L_K8S & LN_K8S & A_K8S & M_K8S & S_K8S & SP_K8S

    L_DB <-.->|"Streaming Replication"| LN_DB
    LN_DB <-.->|"Streaming Replication"| A_DB
    A_DB <-.->|"Streaming Replication"| SP_DB
    M_DB <-.->|"Streaming Replication"| S_DB
```

---

## 8. Security Architecture

```mermaid
graph TB
    subgraph "Perimeter"
        WAF["WAF (AWS WAF)"]
        DDoS["DDoS Protection (AWS Shield)"]
    end

    subgraph "API Layer"
        KONG["Kong Gateway<br/>Rate Limiting, Auth"]
        JWT["JWT Validation<br/>(RS256)"]
    end

    subgraph "Service Mesh"
        ISTIO["Istio<br/>mTLS, RBAC"]
    end

    subgraph "Application"
        AUTHZ["RBAC + ABAC<br/>Authorization"]
        PII["PII Masking"]
        AUDIT["Audit Logging"]
    end

    subgraph "Data"
        ENC_REST["AES-256<br/>At Rest"]
        ENC_TRANSIT["TLS 1.3<br/>In Transit"]
    end

    WAF --> DDoS --> KONG --> JWT --> ISTIO --> AUTHZ --> PII --> ENC_REST
    AUTHZ --> AUDIT
    ISTIO --> ENC_TRANSIT
```

---

## 9. Deployment Architecture

```mermaid
graph LR
    DEV["Developer"] -->|"git push"| GH["GitHub"]
    GH -->|"webhook"| CI["GitHub Actions"]
    CI -->|"cargo build + test"| ARTIFACT["Container Registry"]
    ARTIFACT -->|"helm upgrade"| K8S["Kubernetes Cluster"]

    subgraph "Kubernetes"
        NS_STG["staging namespace"]
        NS_PROD["production namespace"]
        NS_STG -->|"promotion"| NS_PROD
    end

    K8S --> NS_STG
```

---

## 10. Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Language | Rust 1.83, Go 1.22 | Core logic, microservice stubs |
| Framework | Axum 0.7, Tower, Tokio | HTTP, middleware, async runtime |
| API | REST (JSON), GraphQL | External and internal APIs |
| Database | PostgreSQL 16, ClickHouse, MongoDB 7, Redis 7 | OLTP, analytics, documents, cache |
| Messaging | Apache Kafka, RabbitMQ | Events, commands |
| Observability | OpenTelemetry, Prometheus, Grafana, Jaeger | Traces, metrics, dashboards |
| Orchestration | Kubernetes 1.29, Istio 1.20 | Container orchestration, service mesh |
| Gateway | Kong 3.5 | API gateway, rate limiting |
| IaC | Terraform, Helm | Infrastructure as code |
| CI/CD | GitHub Actions | Build, test, deploy |
