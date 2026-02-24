# Enterprise Architecture -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Enterprise Context

ERP-iPaaS operates as the integration backbone within the BillyRonks enterprise ecosystem, connecting 15+ ERP modules and serving as the central nervous system for cross-module data flow, event propagation, and process automation.

## 2. Enterprise Integration Patterns

### 2.1 Pattern Catalog

```mermaid
graph TB
    subgraph "Integration Patterns Implemented"
        P1[Message Router<br/>Event Backbone]
        P2[Content-Based Router<br/>Workflow Conditions]
        P3[Publish-Subscribe<br/>Redpanda Topics]
        P4[Request-Reply<br/>REST API]
        P5[Dead Letter Channel<br/>DLQ]
        P6[Wire Tap<br/>Audit Logging]
        P7[Idempotent Receiver<br/>Idempotency Keys]
        P8[Circuit Breaker<br/>External Calls]
        P9[Saga Pattern<br/>Temporal Compensation]
        P10[Scatter-Gather<br/>Parallel Branches]
    end
```

### 2.2 Module Integration Map

```mermaid
graph TB
    subgraph "BillyRonks ERP Modules"
        FIN[ERP-Finance]
        HCM[ERP-HCM]
        CRM[ERP-CRM]
        COM[ERP-Commerce]
        SCM[ERP-SCM]
        HC[ERP-Healthcare]
        BI[ERP-BI]
        IAM[ERP-IAM]
        MKT[ERP-Marketing]
        PLAT[ERP-Platform]
    end

    subgraph "ERP-iPaaS"
        WE[Workflow Engine]
        EB[Event Backbone]
        CF[Connectors]
        AM[API Management]
        ETL[ETL Pipelines]
    end

    FIN -->|"Invoice events"| EB
    HCM -->|"Employee lifecycle"| EB
    CRM -->|"Lead/Contact sync"| EB
    COM -->|"Order events"| EB
    SCM -->|"Inventory events"| EB
    HC -->|"Patient records"| EB

    EB -->|"Cross-module events"| WE
    WE -->|"Orchestrated processes"| FIN
    WE -->|"Orchestrated processes"| CRM
    WE -->|"Orchestrated processes"| HCM

    CF -->|"External SaaS"| AM
    ETL -->|"Data pipelines"| BI

    IAM -->|"Identity/Auth"| AM
    PLAT -->|"Infra services"| WE
```

### 2.3 Integration Styles by Module

| Source Module | Target Module | Pattern | Protocol | Trigger |
|--------------|--------------|---------|----------|---------|
| CRM | Finance | Event-driven | Kafka | New deal closed |
| HCM | Finance | Workflow | Temporal | Payroll cycle |
| Commerce | SCM | Event-driven | Kafka | Order placed |
| Commerce | Finance | Workflow | Activepieces | Invoice generation |
| CRM | Marketing | Event-driven | Kafka | Lead stage change |
| Healthcare | BI | ETL | Batch + CDC | Daily/real-time |
| IAM | All modules | Request-reply | REST/OAuth2 | Authentication |
| Finance | BI | ETL | Batch | End of day |

## 3. TOGAF Alignment

### 3.1 Business Architecture

```mermaid
graph LR
    subgraph "Business Capabilities"
        BC1[Process Automation]
        BC2[Data Integration]
        BC3[API Ecosystem]
        BC4[Partner Connectivity]
        BC5[Compliance Monitoring]
    end

    subgraph "Business Services"
        BS1[Workflow Automation Service]
        BS2[Data Pipeline Service]
        BS3[API Gateway Service]
        BS4[Connector Marketplace]
        BS5[Audit and Compliance Service]
    end

    BC1 --> BS1
    BC2 --> BS2
    BC3 --> BS3
    BC4 --> BS4
    BC5 --> BS5
```

### 3.2 Application Architecture

| Application Component | Technology | Business Service |
|----------------------|-----------|-----------------|
| Activepieces | Low-code platform | Workflow Automation |
| Temporal | Durable workflow engine | Workflow Automation |
| Redpanda | Event streaming | Data Integration |
| Traefik | API gateway | API Gateway |
| ClickHouse | Analytics database | Audit and Compliance |
| Connector CLI | Developer tool | Connector Marketplace |

### 3.3 Technology Architecture

| Layer | Technology | Standard |
|-------|-----------|----------|
| Compute | Kubernetes 1.28+ | CNCF |
| Networking | Traefik + Calico CNI | Service mesh ready |
| Identity | Keycloak (OIDC/OAuth2) | OpenID Connect |
| Messaging | Redpanda (Kafka protocol) | Apache Kafka 3.5 |
| Storage | PostgreSQL + ClickHouse + MinIO | SQL + Columnar + S3 |
| Observability | Grafana + Prometheus + Loki + Tempo | OpenTelemetry |
| GitOps | ArgoCD + Helm + Kustomize | CNCF GitOps |
| IaC | Terraform | HashiCorp |

### 3.4 Data Architecture

```mermaid
graph TB
    subgraph "Data Domains"
        OD[Operational Data<br/>PostgreSQL]
        AD[Analytical Data<br/>ClickHouse]
        ED[Event Data<br/>Redpanda]
        FD[File Data<br/>MinIO]
        CD[Cache Data<br/>Dragonfly]
    end

    subgraph "Data Governance"
        RLS[Row-Level Security]
        PII[PII Redaction]
        ENC[Encryption at Rest]
        TTL[Data Retention TTL]
        AUD[Audit Logging]
    end

    OD --> RLS
    OD --> ENC
    AD --> TTL
    AD --> AUD
    ED --> PII
    FD --> ENC
    CD --> TTL
```

## 4. Interoperability Strategy

### 4.1 Platform Interop Mappings

The platform provides bidirectional mapping templates for migrating workflows from competing platforms:

| Source Platform | Mapping Template | Location |
|----------------|-----------------|----------|
| Zapier | zapier-mapping.json | `templates/interop/` |
| Make (Integromat) | make-mapping.json | `templates/interop/` |
| Power Automate | power-automate-mapping.json | `templates/interop/` |
| Pabbly Connect | pabbly-mapping.json | `templates/interop/` |
| IFTTT | ifttt-mapping.json | `templates/interop/` |
| Integrately | integrately-mapping.json | `templates/interop/` |

### 4.2 Interop Recipes

```mermaid
graph LR
    ZAP[Zapier Zap] --> |"zapier-to-activepieces.json"| AP[Activepieces Flow]
    AP --> |"activepieces-to-power-automate.json"| PA[Power Automate Flow]
```

Migration recipes in `connectors/interop/recipes/` provide automated conversion between platforms.

## 5. Governance Framework

### 5.1 Integration Governance Model

| Governance Area | Policy | Enforcement |
|----------------|--------|-------------|
| API versioning | Semantic versioning required | Gateway validation |
| Schema evolution | Backward-compatible changes only | Schema registry |
| Rate limiting | Per-tenant rate limits | Traefik middleware |
| Data classification | PII tagging on all fields | PII guard config |
| Change management | GitOps + PR review | ArgoCD sync |
| Security scanning | Container image scanning | CI/CD pipeline |
| Compliance | SOC2/GDPR/NDPR controls | OPA constraints |

### 5.2 Architecture Decision Records

Architecture decisions are tracked in `docs/ADR/`:

| ADR | Decision | Status |
|-----|----------|--------|
| ADR-001 | Language choice: Go for services, TypeScript for SDKs | Accepted |

## 6. Capacity Model

### 6.1 Tenant Sizing Tiers

| Tier | Workflows/day | Events/sec | API calls/min | Storage |
|------|--------------|-----------|--------------|---------|
| Starter | 1,000 | 100 | 60 | 1 GB |
| Professional | 100,000 | 5,000 | 600 | 50 GB |
| Enterprise | 1,000,000+ | 50,000+ | 6,000+ | 500 GB+ |
| Dedicated | Unlimited | Unlimited | Unlimited | Custom |

### 6.2 Resource Allocation per Tier

```mermaid
graph LR
    subgraph "Resource Allocation"
        S[Starter<br/>0.5 CPU / 512Mi]
        P[Professional<br/>2 CPU / 2Gi]
        E[Enterprise<br/>8 CPU / 8Gi]
        D[Dedicated<br/>Isolated namespace]
    end
```
