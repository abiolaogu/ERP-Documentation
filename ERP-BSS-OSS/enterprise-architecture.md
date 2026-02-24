# Enterprise Architecture -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Enterprise Context

ERP-BSS-OSS operates as a module within the BillyRonks Global Limited ERP suite. It integrates with ERP-IAM for identity management, ERP-Finance for general ledger entries, ERP-CRM for sales pipeline enrichment, ERP-BI for analytics dashboards, and ERP-AI for machine learning model serving.

---

## 2. Enterprise Integration Map

```mermaid
graph TB
    subgraph "ERP Suite"
        IAM["ERP-IAM<br/>OIDC / JWT / RBAC"]
        FIN["ERP-Finance<br/>GL / AP / AR"]
        CRM_ERP["ERP-CRM<br/>Sales / Marketing"]
        BI["ERP-BI<br/>Dashboards / Reports"]
        AI["ERP-AI<br/>ML Models"]
        PLATFORM["ERP-Platform<br/>Entitlements / Config"]
        IPAAS["ERP-iPaaS<br/>Integration Hub"]
    end

    subgraph "ERP-BSS-OSS"
        BSS_GW["API Gateway"]
        BSS_BILL["Billing Service"]
        BSS_CUST["Customer Service"]
        BSS_ORD["Order Service"]
        BSS_RA["Revenue Assurance"]
        BSS_NOC["Network Ops"]
    end

    BSS_GW -->|"OIDC token validation"| IAM
    BSS_BILL -->|"Journal entries"| FIN
    BSS_CUST -->|"Lead/opportunity sync"| CRM_ERP
    BSS_RA -->|"Dashboard feeds"| BI
    BSS_RA -->|"Fraud model inference"| AI
    BSS_GW -->|"Feature flags"| PLATFORM
    BSS_ORD -->|"Webhook relay"| IPAAS
```

---

## 3. eTOM Process Alignment

The platform maps to TM Forum eTOM Level 2 processes:

```mermaid
graph TB
    subgraph "Strategy & Commit"
        SC1["Product Lifecycle Mgmt<br/>(Product Catalog)"]
    end

    subgraph "Operations"
        subgraph "Fulfillment"
            F1["Order Handling<br/>(Order Management)"]
            F2["Service Configuration<br/>(Provisioning)"]
            F3["Resource Provisioning<br/>(Resource Inventory)"]
        end

        subgraph "Assurance"
            A1["Problem Handling<br/>(Fault Management)"]
            A2["Service Quality Mgmt<br/>(Network Operations)"]
            A3["Resource Trouble Mgmt<br/>(Workforce Mgmt)"]
        end

        subgraph "Billing"
            B1["Billing & Collections<br/>(Billing/Rating)"]
            B2["Charging<br/>(Charging Engine)"]
            B3["Bill Inquiry<br/>(Self-Care Portal)"]
        end
    end

    subgraph "Enterprise Management"
        E1["Revenue Assurance"]
        E2["Fraud Management"]
        E3["Partner Management"]
    end
```

---

## 4. Integration Patterns

### 4.1 Synchronous (Request/Response)

| Integration | Protocol | Pattern | SLA |
|------------|----------|---------|-----|
| ERP-IAM token validation | REST/OIDC | Gateway filter | < 5 ms (cached) |
| Payment gateway (Paystack) | REST + webhook | Request/callback | < 3 sec |
| Tax engine (Avalara) | REST | Request/response | < 500 ms |
| Number portability (NPC) | SOAP/REST | Request/response | < 10 sec |

### 4.2 Asynchronous (Event-Driven)

| Integration | Protocol | Pattern | Topics |
|------------|----------|---------|--------|
| BSS -> ERP-Finance | Kafka | Publish/subscribe | `erp.bss_oss.billing-rating.created` |
| BSS -> ERP-CRM | Kafka | Publish/subscribe | `erp.bss_oss.customer-management.created` |
| BSS -> ERP-BI | Kafka | Publish/subscribe | All `erp.bss_oss.*` topics |
| BSS -> ERP-AI | Kafka + gRPC | Publish + inference | `erp.bss_oss.revenue-assurance.*` |
| Network -> BSS | DIAMETER/RADIUS | Stream | CDR events |

### 4.3 File-Based

| Integration | Format | Protocol | Frequency |
|------------|--------|----------|-----------|
| Roaming (TAP/RAP) | ASN.1 | SFTP | Daily |
| Bank reconciliation | CSV/ISO 20022 | SFTP | Daily |
| Regulatory reporting | XML | HTTPS upload | Monthly |
| Meter readings (AMI) | DLMS/COSEM | MQTT/HTTPS | Every 15 min |

---

## 5. Data Flow Architecture

```mermaid
graph LR
    subgraph "Operational Data"
        SRC_NET["Network<br/>(CDRs, Alarms)"]
        SRC_PAY["Payments"]
        SRC_SELF["Self-Care<br/>Actions"]
    end

    subgraph "BSS-OSS Processing"
        MED["Mediation<br/>(Normalize)"]
        RATE["Rating<br/>(Price)"]
        BILL["Billing<br/>(Invoice)"]
        RA_PROC["Revenue Assurance<br/>(Reconcile)"]
    end

    subgraph "Data Stores"
        PG["PostgreSQL<br/>(OLTP)"]
        CH["ClickHouse<br/>(OLAP)"]
        RD["Redis<br/>(Cache)"]
    end

    subgraph "Consumers"
        BI_DASH["BI Dashboards"]
        FIN_SYS["Finance System"]
        REG_RPT["Regulatory Reports"]
    end

    SRC_NET --> MED --> RATE --> BILL --> PG
    SRC_PAY --> BILL
    SRC_SELF --> BILL
    BILL --> CH
    BILL --> RA_PROC
    CH --> BI_DASH
    BILL --> FIN_SYS
    RA_PROC --> REG_RPT
    RATE --> RD
```

---

## 6. Governance Model

### 6.1 API Governance

- All APIs must conform to TM Forum Open API specifications
- API versioning: URI path (`/v1/`, `/v2/`)
- Breaking changes require major version bump
- Deprecation period: minimum 6 months

### 6.2 Data Governance

- PII fields encrypted at column level
- Data retention: CDRs (7 years), invoices (10 years), audit logs (5 years)
- Right-to-erasure: soft delete with 30-day purge cycle
- Data classification: Public, Internal, Confidential, Restricted

### 6.3 Service Governance

- Each service owns its database schema (database-per-service)
- No direct database access across service boundaries
- Inter-service communication via API or events only
- Service SLOs: P99 < 100ms, error rate < 0.1%, availability > 99.9%

---

## 7. Capacity Planning

### 7.1 Subscriber Tiers

| Tier | Subscribers | K8s Nodes | PostgreSQL | Redis | Kafka |
|------|------------|-----------|-----------|-------|-------|
| Starter | 1K - 50K | 3 | 1 primary + 1 replica | 1 node | 3 brokers |
| Growth | 50K - 500K | 6 | 1 primary + 2 replicas | 3 nodes | 3 brokers |
| Scale | 500K - 5M | 15 | 3 primary (sharded) + 6 replicas | 6 nodes | 6 brokers |
| Enterprise | 5M - 50M | 50+ | YugabyteDB 9+ nodes | 12 nodes | 12 brokers |

### 7.2 Throughput Projections

```mermaid
graph LR
    subgraph "1M Subscribers"
        T1["API: 10K TPS"]
        T2["CDR: 500K/sec"]
        T3["Billing: 4hr cycle"]
    end

    subgraph "10M Subscribers"
        T4["API: 100K TPS"]
        T5["CDR: 1.4M/sec"]
        T6["Billing: 8hr cycle"]
    end

    subgraph "50M Subscribers"
        T7["API: 500K TPS"]
        T8["CDR: 5M/sec"]
        T9["Billing: 12hr cycle"]
    end
```

---

## 8. Disaster Recovery

| Component | Strategy | RTO | RPO |
|-----------|----------|-----|-----|
| Application services | Multi-PoP active-active | 0 | 0 |
| PostgreSQL | Streaming replication + WAL archiving | 5 min | 30 sec |
| Redis | AOF + RDB snapshots | 1 min | 10 sec |
| ClickHouse | Partition backups to S3 | 30 min | 1 hour |
| Kafka | Multi-broker replication (RF=3) | 0 | 0 |
| Configuration | GitOps (Terraform + Helm) | 15 min | 0 |
