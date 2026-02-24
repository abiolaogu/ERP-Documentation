# High-Level Design (HLD) -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This High-Level Design document describes the system-level architecture, component interactions, deployment model, and key design decisions for the ERP-BSS-OSS telecom and utilities BSS/OSS platform.

---

## 2. System Context

```mermaid
graph TB
    subgraph "Users"
        U1["Subscribers"]
        U2["CSR Agents"]
        U3["NOC Engineers"]
        U4["Partner Managers"]
        U5["Finance Team"]
        U6["Utility Customers"]
    end

    subgraph "ERP-BSS-OSS"
        PLATFORM["BSS/OSS Platform<br/>30 Microservices<br/>9 Rust Crates"]
    end

    subgraph "External"
        NET["Telecom Network<br/>(HLR/HSS, MSC, PCRF)"]
        PAY["Payment Gateways"]
        ERP["ERP Suite"]
        SMPP["SMS Gateway"]
        NPC_SYS["Number Portability"]
        AMI["Smart Meters (AMI)"]
    end

    U1 & U2 & U3 & U4 & U5 & U6 --> PLATFORM
    PLATFORM --> NET & PAY & ERP & SMPP & NPC_SYS & AMI
```

---

## 3. Component Architecture

### 3.1 Layered Architecture

```mermaid
graph TB
    subgraph "Presentation Layer"
        WEB["Web Portal (React)"]
        MOB["Mobile App (Flutter)"]
        USSD_UI["USSD Menu"]
        API_EXT["External APIs"]
    end

    subgraph "API Gateway Layer"
        KONG["Kong API Gateway"]
        GQL["GraphQL Federation"]
    end

    subgraph "Service Layer"
        subgraph "BSS Domain"
            SVC_PC["Product Catalog"]
            SVC_CM["Customer Mgmt"]
            SVC_OM["Order Mgmt"]
            SVC_BR["Billing/Rating"]
            SVC_CE["Charging Engine"]
            SVC_MED["Mediation"]
            SVC_PS["Partner Service"]
        end

        subgraph "OSS Domain"
            SVC_PROV["Provisioning"]
            SVC_RI["Resource Inventory"]
            SVC_SI["Service Inventory"]
            SVC_NO["Network Ops"]
        end

        subgraph "Channel Domain"
            SVC_SC["Self-Care"]
            SVC_USSD["USSD/IVR GW"]
        end

        subgraph "Utilities Domain"
            SVC_MM["Meter Mgmt"]
            SVC_TS["Tariff Service"]
        end
    end

    subgraph "Integration Layer"
        KF["Kafka Event Bus"]
        RMQ["RabbitMQ Command Queue"]
    end

    subgraph "Data Layer"
        PG["PostgreSQL"]
        RD["Redis"]
        CH["ClickHouse"]
        MG["MongoDB"]
    end

    WEB & MOB & USSD_UI & API_EXT --> KONG
    KONG --> GQL
    GQL --> SVC_PC & SVC_CM & SVC_OM & SVC_BR
    KONG --> SVC_CE & SVC_MED & SVC_PS
    KONG --> SVC_PROV & SVC_RI & SVC_SI & SVC_NO
    KONG --> SVC_SC & SVC_USSD
    KONG --> SVC_MM & SVC_TS

    SVC_BR & SVC_OM & SVC_CM --> KF
    SVC_CE & SVC_PROV --> RMQ

    SVC_PC & SVC_CM & SVC_OM & SVC_BR --> PG
    SVC_CE --> RD
    SVC_MED & SVC_BR --> CH
    SVC_NO --> MG
```

### 3.2 Service Interaction Matrix

| Service | Depends On | Depended By |
|---------|-----------|-------------|
| Product Catalog | - | Order Mgmt, Billing, Self-Care, Partner |
| Customer Mgmt | - | Order Mgmt, Billing, Self-Care, USSD, Partner |
| Order Mgmt | Product Catalog, Customer Mgmt | Billing, Provisioning |
| Billing/Rating | Product Catalog, Customer Mgmt, Mediation | Self-Care, Partner, Revenue Assurance |
| Charging Engine | Billing/Rating | Mediation |
| Mediation | Charging Engine | Billing, Revenue Assurance |
| Provisioning | Order Mgmt, Resource Inventory | Service Inventory |
| Resource Inventory | - | Provisioning, Network Ops |
| Service Inventory | Provisioning | Network Ops, Self-Care |
| Partner Service | Billing, Product Catalog | - |
| Revenue Assurance | Billing, Mediation | - |
| Self-Care | Customer Mgmt, Billing, Service Inventory | - |
| USSD/IVR | Customer Mgmt, Billing | - |
| Meter Mgmt | Customer Mgmt | Tariff Service |
| Tariff Service | Meter Mgmt | Billing |

---

## 4. Key Design Decisions

### 4.1 Language Selection: Rust

**Decision:** Use Rust as the primary language for performance-critical services.

**Rationale:**
- Zero-cost abstractions for telecom-grade performance (sub-millisecond latency)
- Memory safety without garbage collector (critical for OCS real-time charging)
- Async runtime (Tokio) for high-concurrency CDR processing
- Strong type system prevents billing calculation errors at compile time

### 4.2 Polyglot Persistence

**Decision:** Use four database technologies, each optimized for its workload.

```mermaid
graph LR
    OLTP["OLTP Workloads<br/>Customers, Orders, Billing"] --> PG["PostgreSQL 16"]
    CACHE["Cache + Sessions<br/>Balance, Rate Limit"] --> RD["Redis 7"]
    OLAP["Analytics + CDR<br/>Revenue Reports"] --> CH["ClickHouse"]
    DOC["Documents + Audit<br/>Alarms, Config"] --> MG["MongoDB 7"]
```

### 4.3 Event-Driven Communication

**Decision:** All inter-service communication for state changes uses Kafka events; synchronous REST for queries.

**Rationale:**
- Decouples services temporally and spatially
- Enables event sourcing for audit trail
- Supports replay for data recovery
- Scales independently per consumer group

### 4.4 Multi-Tenant Architecture

**Decision:** Tenant isolation via `X-Tenant-ID` header with row-level security in PostgreSQL.

All service endpoints require the `X-Tenant-ID` header, enabling a single deployment to serve multiple operators/MVNOs.

---

## 5. Non-Functional Design

### 5.1 Scalability

```mermaid
graph TB
    subgraph "Horizontal Scaling"
        HPA["K8s HPA"] --> PODS["Scale pods 1-50"]
        PODS --> LB["Load Balancer (L7)"]
    end

    subgraph "Database Scaling"
        SHARD["Sharding by subscriber_id"]
        READ_REP["Read replicas for queries"]
        PART["Time-based partitioning for CDRs"]
    end

    subgraph "Event Scaling"
        KF_PART["Kafka partition scaling"]
        CG["Consumer groups per service"]
    end
```

### 5.2 High Availability

| Component | HA Strategy | Failover Time |
|-----------|------------|---------------|
| API Gateway | Active-active across PoPs | 0 ms (DNS routing) |
| Microservices | 3+ replicas per service | < 5 sec (K8s reschedule) |
| PostgreSQL | Streaming replication (primary + 2 replicas) | < 30 sec (automatic) |
| Redis | Sentinel (3 nodes) | < 15 sec |
| Kafka | 3+ brokers, RF=3 | 0 ms (leader election) |
| ClickHouse | 2 replicas | < 1 min |

### 5.3 Performance Targets

| Metric | Target |
|--------|--------|
| API P99 latency | < 50 ms |
| OCS charging latency | < 1 ms |
| CDR mediation throughput | 1.4 M/sec |
| Invoice generation (1M subs) | < 4 hours |
| USSD session response | < 200 ms |
| Kafka event delivery | < 10 ms |

---

## 6. Deployment Model

```mermaid
graph TB
    subgraph "Production Kubernetes Cluster"
        subgraph "Namespace: bss-prod"
            DEP_PC["product-catalog<br/>3 replicas"]
            DEP_CM["customer-management<br/>3 replicas"]
            DEP_OM["order-management<br/>3 replicas"]
            DEP_BR["billing-rating<br/>5 replicas"]
            DEP_CE["charging-engine<br/>5 replicas"]
            DEP_MED["mediation<br/>5 replicas"]
        end

        subgraph "Namespace: oss-prod"
            DEP_PROV["provisioning<br/>3 replicas"]
            DEP_RI["resource-inventory<br/>2 replicas"]
            DEP_NO["network-ops<br/>3 replicas"]
        end

        subgraph "Namespace: data-prod"
            DEP_PG["PostgreSQL<br/>StatefulSet"]
            DEP_RD["Redis<br/>StatefulSet"]
            DEP_KF["Kafka<br/>StatefulSet"]
            DEP_CH["ClickHouse<br/>StatefulSet"]
        end
    end
```

---

## 7. Security Design

### 7.1 Authentication Flow

```mermaid
sequenceDiagram
    participant CLIENT as Client
    participant KONG as Kong Gateway
    participant IAM as ERP-IAM (OIDC)
    participant SVC as BSS Service

    CLIENT->>IAM: Authenticate (username + password + MFA)
    IAM-->>CLIENT: JWT access token + refresh token
    CLIENT->>KONG: API request + Bearer JWT
    KONG->>KONG: Validate JWT signature (RS256)
    KONG->>KONG: Check token expiry
    KONG->>KONG: Extract tenant_id, roles, permissions
    KONG->>SVC: Forward request + X-Tenant-ID + X-User-Roles
    SVC->>SVC: RBAC check (role has permission?)
    SVC-->>KONG: Response
    KONG-->>CLIENT: Response
```

### 7.2 Data Classification

| Level | Examples | Controls |
|-------|---------|----------|
| **Restricted** | Passwords, encryption keys, PUK codes | HSM storage, no logging |
| **Confidential** | KYC documents, payment card data | Encrypted at rest + transit, PCI scope |
| **Internal** | Customer names, phone numbers, CDRs | Encrypted at rest, RBAC |
| **Public** | Product catalog, tariff rates | No special controls |
