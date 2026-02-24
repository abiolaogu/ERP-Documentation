# High-Level Design -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This High-Level Design (HLD) document describes the overall system design of ERP-iPaaS, covering system boundaries, major components, interaction patterns, data flow, and deployment topology at an abstraction level suitable for architecture review boards and technical leadership.

## 2. System Boundary

```mermaid
graph TB
    subgraph "External Systems"
        EXT_SAAS[External SaaS<br/>Salesforce, Slack, Stripe]
        EXT_API[Third-Party APIs]
        EXT_WH[External Webhooks]
    end

    subgraph "BillyRonks ERP Ecosystem"
        IDAAS[IDaaS / Keycloak]
        ERP_MOD[ERP Modules<br/>Finance, HCM, CRM, etc.]
        DBAAS[DBaaS Platform]
        OBS[Observability Platform]
    end

    subgraph "ERP-iPaaS Boundary"
        GW[API Gateway]
        CORE[Core Services]
        RUNTIME[Workflow Runtimes]
        DATA[Data Stores]
        EVENTS[Event Backbone]
    end

    EXT_SAAS --> GW
    EXT_API --> GW
    EXT_WH --> GW
    IDAAS --> GW
    ERP_MOD <--> EVENTS
    DBAAS --> DATA
    OBS --> CORE

    GW --> CORE
    CORE --> RUNTIME
    CORE --> DATA
    CORE --> EVENTS
    RUNTIME --> DATA
    RUNTIME --> EVENTS
```

## 3. Major Components

### 3.1 Component Interaction Diagram

```mermaid
graph TB
    subgraph "Ingress"
        TRF[Traefik API Gateway<br/>Auth + Rate Limiting]
    end

    subgraph "Service Mesh"
        WE[Workflow Engine]
        CF[Connector Framework]
        EB[Event Backbone Manager]
        AM[API Management]
        ETL[ETL Pipeline Manager]
        WH[Webhook Manager]
    end

    subgraph "Runtimes"
        AP[Activepieces<br/>Controller + Workers]
        TMP[Temporal<br/>Frontend + History + Matching + Workers]
        NF[Nexum Flow<br/>DAG Engine]
    end

    subgraph "Data Tier"
        PG[(PostgreSQL)]
        CH[(ClickHouse)]
        RP[Redpanda Cluster]
        DF[Dragonfly Cache]
        MIO[(MinIO)]
    end

    TRF --> WE
    TRF --> CF
    TRF --> EB
    TRF --> AM
    TRF --> ETL
    TRF --> WH

    WE --> AP
    WE --> TMP
    WE --> NF

    WE --> PG
    WE --> CH
    WE --> RP
    CF --> PG
    CF --> RP
    EB --> RP
    EB --> CH
    AM --> PG
    ETL --> PG
    ETL --> CH
    ETL --> MIO
    WH --> PG
    WH --> RP

    AP --> DF
    AP --> PG
    TMP --> PG
    NF --> DF
```

### 3.2 Component Responsibilities

| Component | Responsibility | Technology | Scaling Strategy |
|-----------|---------------|-----------|-----------------|
| API Gateway | Authentication, rate limiting, routing | Traefik v2.10 | HPA on CPU |
| Workflow Engine | Workflow CRUD, execution orchestration | Go service | KEDA on queue depth |
| Connector Framework | Connector lifecycle, marketplace | Go service | HPA on requests |
| Event Backbone Manager | Topic management, schema registry | Go service | HPA on requests |
| API Management | Key management, analytics | Go service | HPA on requests |
| ETL Pipeline Manager | Pipeline CRUD, execution | Go service | KEDA on pipeline queue |
| Webhook Manager | Registration, delivery, retry | Go service | KEDA on delivery queue |
| Activepieces | Visual builder, piece execution | Node.js | KEDA on task queue |
| Temporal | Durable workflow execution | Go + TypeScript | KEDA on task backlog |
| Nexum Flow | AI-augmented DAG execution | TypeScript | HPA on CPU |

## 4. High-Level Data Flow

### 4.1 Workflow Execution Flow

```mermaid
sequenceDiagram
    participant Client
    participant GW as API Gateway
    participant WE as Workflow Engine
    participant AP as Activepieces
    participant TMP as Temporal
    participant RP as Redpanda
    participant CH as ClickHouse
    participant PG as PostgreSQL

    Client->>GW: POST /v1/workflow-engine (create workflow)
    GW->>GW: Validate JWT, extract tenant_id
    GW->>WE: Forward with X-Tenant-ID
    WE->>PG: INSERT workflow
    WE->>RP: Publish workflow.created event
    WE-->>Client: 201 Created

    Note over Client: Trigger fires (webhook/schedule/event)

    GW->>WE: Trigger payload
    WE->>WE: Determine runtime (Activepieces/Temporal)

    alt Activepieces workflow
        WE->>AP: Execute flow
        AP->>AP: Run pieces sequentially
        AP->>RP: Publish step events
    else Temporal workflow
        WE->>TMP: Start workflow execution
        TMP->>TMP: Execute activities with retry
        TMP->>RP: Publish completion event
    end

    RP->>CH: Sink run data
```

### 4.2 Event Propagation Flow

```mermaid
graph LR
    subgraph "Producers"
        P1[ERP Module]
        P2[Workflow]
        P3[Webhook]
    end

    subgraph "Redpanda"
        T[Topic<br/>tenant.{id}.{type}]
        SR[Schema Registry]
        DLQ[Dead Letter Queue]
    end

    subgraph "Consumers"
        C1[Workflow Trigger]
        C2[ClickHouse Sink]
        C3[Alert Evaluator]
        C4[Audit Logger]
    end

    P1 -->|"Produce"| SR
    SR -->|"Validate"| T
    P2 --> T
    P3 --> T

    T --> C1
    T --> C2
    T --> C3
    T --> C4

    T -.->|"Failed"| DLQ
```

## 5. Deployment Topology

### 5.1 Kubernetes Cluster Layout

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "ipaas-system namespace"
            GW_D[Traefik Deployment<br/>2+ replicas]
            WE_D[Workflow Engine Deployment<br/>3+ replicas]
            CF_D[Connector Framework<br/>2+ replicas]
            EB_D[Event Backbone<br/>2+ replicas]
            AM_D[API Management<br/>2+ replicas]
            ETL_D[ETL Service<br/>2+ replicas]
            WH_D[Webhook Service<br/>2+ replicas]
        end

        subgraph "ipaas-runtime namespace"
            AP_CTRL[Activepieces Controller<br/>1 replica]
            AP_WRK[Activepieces Workers<br/>KEDA-scaled]
            TMP_FE[Temporal Frontend<br/>2+ replicas]
            TMP_HIST[Temporal History<br/>2+ replicas]
            TMP_MATCH[Temporal Matching<br/>2+ replicas]
            TMP_WRK[Temporal Workers<br/>KEDA-scaled]
        end

        subgraph "ipaas-data namespace"
            PG_SS[PostgreSQL StatefulSet<br/>Primary + Replica]
            CH_SS[ClickHouse StatefulSet<br/>3 nodes]
            RP_SS[Redpanda StatefulSet<br/>3 nodes]
            DF_D[Dragonfly Deployment]
            MIO_SS[MinIO StatefulSet<br/>4 nodes]
        end

        subgraph "observability namespace"
            GF_D[Grafana]
            PM_D[Prometheus]
            LK_D[Loki]
            TP_D[Tempo]
            ST_D[Sentry]
        end
    end
```

### 5.2 Network Architecture

| Source | Destination | Protocol | Port | Purpose |
|--------|------------|----------|------|---------|
| External | Traefik | HTTPS | 443 | API ingress |
| Traefik | Services | HTTP | 8080 | Internal routing |
| Services | PostgreSQL | TCP | 5432 | Database |
| Services | Redpanda | TCP | 9092 | Event streaming |
| Services | ClickHouse | HTTP | 8123 | Analytics |
| Services | Dragonfly | TCP | 6379 | Cache |
| Services | MinIO | HTTP | 9000 | Object storage |
| Temporal | PostgreSQL | TCP | 5432 | Workflow state |
| Activepieces | Dragonfly | TCP | 6379 | Session cache |

## 6. Security Design

### 6.1 Authentication and Authorization Flow

```mermaid
sequenceDiagram
    participant Client
    participant KC as Keycloak
    participant TRF as Traefik
    participant SVC as Service
    participant PG as PostgreSQL

    Client->>KC: OAuth2 token request
    KC-->>Client: JWT (tenant_id, roles, scopes)
    Client->>TRF: API request + Bearer token
    TRF->>TRF: Validate JWT signature
    TRF->>TRF: Check rate limits
    TRF->>SVC: Forward + X-Tenant-ID header
    SVC->>PG: SET app.current_tenant = tenant_id
    PG->>PG: RLS filters all queries
    SVC-->>TRF: Response
    TRF-->>Client: Response
```

### 6.2 Encryption Strategy

| Layer | Method | Key Management |
|-------|--------|---------------|
| Transit | TLS 1.3 (mTLS between services) | Cert-manager + Let's Encrypt |
| At rest (PostgreSQL) | AES-256 | Kubernetes secrets |
| At rest (MinIO) | SSE-S3 | MinIO KMS |
| At rest (Redpanda) | Disk encryption | Node-level encryption |
| Secrets | Vault/SOPS | HashiCorp Vault |

## 7. High Availability Design

### 7.1 HA Configuration

| Component | HA Strategy | Min Replicas | RPO | RTO |
|-----------|------------|-------------|-----|-----|
| Traefik | Multi-replica | 2 | 0 | < 30s |
| Core Services | Multi-replica | 2 | 0 | < 30s |
| PostgreSQL | Primary-replica | 2 | < 1min | < 5min |
| ClickHouse | 3-node cluster | 3 | < 5min | < 10min |
| Redpanda | 3-node cluster | 3 | 0 | < 30s |
| Temporal | Multi-replica per role | 2 | 0 | < 60s |

### 7.2 Failure Modes

```mermaid
graph TD
    subgraph "Failure Recovery"
        F1[Service Pod Failure] --> R1[K8s restarts pod<br/>< 30s recovery]
        F2[Database Primary Failure] --> R2[Promote replica<br/>< 5min recovery]
        F3[Redpanda Node Failure] --> R3[Partition rebalance<br/>< 30s recovery]
        F4[Temporal Worker Failure] --> R4[Task re-assigned<br/>automatic retry]
        F5[Network Partition] --> R5[Circuit breaker<br/>graceful degradation]
    end
```

## 8. Integration with Platform Services

| Platform Service | Integration Point | Protocol |
|-----------------|-------------------|----------|
| IDaaS (Keycloak) | JWT issuance and validation | OIDC/OAuth2 |
| DBaaS | PostgreSQL, ClickHouse provisioning | Terraform |
| Observability | Metrics, logs, traces | Prometheus, Loki, Tempo |
| Orchestration (ArgoCD) | GitOps deployment | Kubernetes API |
| Secret Management (Vault) | Secret injection | Vault Agent |
