# System Architecture -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Architecture Overview

ERP-iPaaS follows a microservices architecture deployed on Kubernetes with six core services, an event-driven backbone, and a multi-layer security model providing complete tenant isolation.

```mermaid
graph TB
    subgraph "Client Layer"
        WEB[Web UI<br/>React + Next.js]
        CLI[CLI Tool<br/>connector-cli]
        SDK_TS[TypeScript SDK]
        SDK_GO[Go SDK]
        SDK_PY[Python SDK]
        MOB[Mobile<br/>Flutter/React Native]
    end

    subgraph "API Gateway Layer"
        TRF[Traefik v2.10<br/>Rate Limiting + Auth]
        KC[Keycloak 22.0<br/>OAuth2/OIDC]
    end

    subgraph "Core Services"
        WE[Workflow Engine<br/>Go :8080]
        CF[Connector Framework<br/>Go :8080]
        EB[Event Backbone<br/>Go :8080]
        AM[API Management<br/>Go :8080]
        ETL[ETL Service<br/>Go :8080]
        WH[Webhook Service<br/>Go :8080]
    end

    subgraph "Workflow Runtimes"
        AP[Activepieces 0.20.0<br/>Low-code Builder]
        TMP[Temporal 1.23.0<br/>Durable Execution]
        NF[Nexum Flow<br/>DAG Engine]
    end

    subgraph "Data Layer"
        PG[(PostgreSQL 16<br/>Primary + RLS)]
        CH[(ClickHouse 23.9<br/>Analytics)]
        RP[Redpanda<br/>Event Streaming]
        DF[Dragonfly<br/>Cache/Redis)]
        MIO[MinIO<br/>Object Storage]
    end

    subgraph "Observability"
        GF[Grafana 10.1]
        PM[Prometheus]
        LK[Loki]
        TP[Tempo]
        ST[Sentry]
    end

    WEB --> TRF
    CLI --> TRF
    SDK_TS --> TRF
    SDK_GO --> TRF
    MOB --> TRF

    TRF --> KC
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
    WE --> RP
    WE --> CH
    CF --> PG
    CF --> RP
    EB --> RP
    EB --> CH
    ETL --> PG
    ETL --> CH
    WH --> PG
    WH --> RP

    AP --> DF
    TMP --> PG
    NF --> DF

    WE --> MIO
    ETL --> MIO

    PM --> GF
    LK --> GF
    TP --> GF
```

## 2. Service Architecture

### 2.1 Service Decomposition

Each core service follows the same Go microservice pattern with tenant-scoped endpoints:

```mermaid
graph LR
    subgraph "Service Pattern"
        H[Health Check<br/>/healthz]
        L[List<br/>GET /v1/{service}]
        C[Create<br/>POST /v1/{service}]
        R[Read<br/>GET /v1/{service}/{id}]
        U[Update<br/>PATCH /v1/{service}/{id}]
        D[Delete<br/>DELETE /v1/{service}/{id}]
    end

    subgraph "Cross-Cutting"
        T[Tenant Isolation<br/>X-Tenant-ID header]
        E[Event Publishing<br/>erp.ipaas.{service}.*]
        A[Audit Logging<br/>ClickHouse audit table]
    end

    L --> T
    C --> T
    R --> T
    U --> T
    D --> T

    C --> E
    U --> E
    D --> E

    C --> A
    U --> A
    D --> A
```

### 2.2 Service Registry

| Service | Base Path | Port | Event Topic Prefix |
|---------|-----------|------|-------------------|
| workflow-engine | `/v1/workflow-engine` | 8080 | `erp.ipaas.workflow-engine.*` |
| connector-framework | `/v1/connector-framework` | 8080 | `erp.ipaas.connector-framework.*` |
| event-backbone | `/v1/event-backbone` | 8080 | `erp.ipaas.event-backbone.*` |
| api-management | `/v1/api-management` | 8080 | `erp.ipaas.api-management.*` |
| etl | `/v1/etl` | 8080 | `erp.ipaas.etl.*` |
| webhook | `/v1/webhook` | 8080 | `erp.ipaas.webhook.*` |

## 3. Tenant Isolation Architecture

```mermaid
graph TB
    subgraph "Request Flow"
        REQ[Incoming Request] --> TRF[Traefik Gateway]
        TRF --> |"JWT Validation"| KC[Keycloak]
        KC --> |"tenant_id claim"| TRF
        TRF --> |"X-Tenant-ID header"| SVC[Service]
        SVC --> |"SET app.current_tenant"| PG[PostgreSQL RLS]
        SVC --> |"tenant prefix"| RP[Redpanda Topic]
        SVC --> |"tenant_id column"| CH[ClickHouse]
    end

    subgraph "Namespace Isolation"
        K8S[Kubernetes] --> NS1[tenant-a namespace]
        K8S --> NS2[tenant-b namespace]
        K8S --> NS3[shared namespace]
        OPA[OPA Gatekeeper] --> |"Constraint"| K8S
    end
```

### 3.1 Isolation Layers

1. **Network**: Kubernetes namespace isolation with NetworkPolicies
2. **Identity**: Keycloak realm per tenant with JWT claims
3. **Database**: PostgreSQL Row-Level Security filtering on `tenant_id`
4. **Events**: Tenant-prefixed Kafka topics (`tenant.{id}.{topic}`)
5. **Storage**: MinIO bucket policies per tenant
6. **Compute**: Optional dedicated worker pools per tenant via KEDA

## 4. Event-Driven Architecture

```mermaid
graph LR
    subgraph "Producers"
        WE[Workflow Engine]
        CF[Connector Framework]
        ETL[ETL Service]
        WH[Webhook Service]
    end

    subgraph "Event Backbone (Redpanda)"
        T1[tenant.*.workflow.events]
        T2[tenant.*.connector.events]
        T3[tenant.*.etl.events]
        T4[tenant.*.webhook.events]
        DLQ[Dead Letter Queue]
        SR[Schema Registry<br/>Avro + Protobuf]
    end

    subgraph "Consumers"
        CH[ClickHouse Sink]
        ALERT[Alert Manager]
        AUDIT[Audit Logger]
        REACT[Reactive Workflows]
    end

    WE --> T1
    CF --> T2
    ETL --> T3
    WH --> T4

    T1 --> CH
    T2 --> CH
    T3 --> CH
    T4 --> CH

    T1 --> ALERT
    T1 --> REACT

    T1 -.->|"failures"| DLQ
    T2 -.->|"failures"| DLQ
    T3 -.->|"failures"| DLQ
    T4 -.->|"failures"| DLQ

    SR -.->|"validates"| T1
    SR -.->|"validates"| T2
```

### 4.1 Event Schema (Avro)

The `WorkflowCommand` Avro schema defines the canonical event format:

```json
{
  "type": "record",
  "name": "WorkflowCommand",
  "namespace": "com.billyronks.workflow",
  "fields": [
    { "name": "tenant_id", "type": "string" },
    { "name": "workflow_type", "type": "string" },
    { "name": "task_queue", "type": "string" },
    { "name": "payload", "type": { "type": "map", "values": "string" } },
    { "name": "idempotency_key", "type": "string" },
    { "name": "priority", "type": "int", "default": 0 },
    { "name": "timestamp", "type": "long", "logicalType": "timestamp-millis" }
  ]
}
```

## 5. Deployment Architecture

```mermaid
graph TB
    subgraph "GitOps Pipeline"
        GH[GitHub] --> |"push"| GA[GitHub Actions CI]
        GA --> |"build + test"| REG[Container Registry]
        GA --> |"update manifests"| ARGO[ArgoCD]
    end

    subgraph "Kubernetes Cluster"
        ARGO --> |"sync"| NS[Namespaces]

        subgraph "argocd namespace"
            ARGOCD[ArgoCD Server]
        end

        subgraph "ipaas namespace"
            WE_POD[Workflow Engine Pod]
            CF_POD[Connector Framework Pod]
            EB_POD[Event Backbone Pod]
            AM_POD[API Management Pod]
            ETL_POD[ETL Service Pod]
            WH_POD[Webhook Service Pod]
        end

        subgraph "data namespace"
            PG_POD[PostgreSQL StatefulSet]
            CH_POD[ClickHouse StatefulSet]
            RP_POD[Redpanda StatefulSet]
            DF_POD[Dragonfly Deployment]
            MIO_POD[MinIO StatefulSet]
        end

        subgraph "observability namespace"
            GF_POD[Grafana]
            PM_POD[Prometheus]
            LK_POD[Loki]
            TP_POD[Tempo]
        end
    end
```

### 5.1 Helm Chart Inventory

| Chart | Location | Purpose |
|-------|----------|---------|
| activepieces | `infra/helm/activepieces/` | Low-code workflow runtime |
| temporal | `infra/helm/temporal/` | Durable workflow runtime |
| redpanda | `infra/helm/redpanda/` | Event streaming |
| traefik | `infra/helm/traefik/` | API gateway |
| keycloak | `infra/helm/keycloak/` | Identity provider |
| clickhouse | `infra/helm/clickhouse/` | Analytics database |
| postgres | `infra/helm/postgres/` | Primary database |
| minio | `infra/helm/minio/` | Object storage |
| keda | `infra/helm/keda/` | Auto-scaling |
| prometheus-stack | `infrastructure/helm/prometheus-stack/` | Monitoring |
| loki-tempo | `infra/helm/loki-tempo/` | Log aggregation + tracing |
| sentry | `infra/helm/sentry/` | Error tracking |

## 6. Data Architecture

### 6.1 PostgreSQL (Operational Data)

- Tenant isolation via Row-Level Security
- Tables: `tenants`, `workflows`, `workflow_runs`, `connectors`, `webhooks`
- Session-scoped tenant context via `set_config('request.jwt.claim.tenant_id', ...)`

### 6.2 ClickHouse (Analytics and Metrics)

- Database: `billyronks`
- Tables: `runs`, `audit`, `connector_latency`, `tenant_usage`, `cost_approx`, `workflow_dlp`, `connector_marketplace`, `connector_validation_queue`, `connector_attestations`
- Materialized view: `connector_validation_daily`
- Timezone: `Africa/Lagos`
- TTL: 30-day default on `runs` table

### 6.3 Redpanda (Event Streaming)

- Kafka-compatible protocol
- Schema registry with Avro
- Tenant-scoped topics
- Dead-letter queues per topic

## 7. Security Architecture

```mermaid
graph TB
    subgraph "Zero Trust Layers"
        L1[Layer 1: Network<br/>mTLS + NetworkPolicies]
        L2[Layer 2: Identity<br/>Keycloak OAuth2/OIDC]
        L3[Layer 3: Authorization<br/>OPA + RBAC]
        L4[Layer 4: Data<br/>PostgreSQL RLS]
        L5[Layer 5: Encryption<br/>TLS 1.3 + AES-256]
        L6[Layer 6: Audit<br/>ClickHouse immutable logs]
    end

    L1 --> L2 --> L3 --> L4 --> L5 --> L6
```

### 7.1 Authentication Flow

1. Client authenticates with Keycloak via OAuth2 (authorization code or client credentials)
2. Keycloak issues JWT with `tenant_id` claim
3. Traefik validates JWT signature and expiry
4. Traefik injects `X-Tenant-ID` header from JWT claim
5. Service uses `X-Tenant-ID` for all data access

### 7.2 Secrets Management

- Vault/SOPS for infrastructure secrets
- Encrypted secret storage via integration layer API
- Rotation policies configurable per secret
- Audit trail for all secret access

## 8. Scalability Architecture

```mermaid
graph LR
    subgraph "Auto-Scaling"
        KEDA[KEDA ScaledObject] --> |"Kafka lag"| WRK[Worker Pods]
        KEDA --> |"HTTP queue depth"| SVC[Service Pods]
        HPA[HPA] --> |"CPU/Memory"| GW[Gateway Pods]
    end

    subgraph "Data Scaling"
        PG_R[PostgreSQL<br/>Read Replicas]
        CH_S[ClickHouse<br/>Sharding]
        RP_P[Redpanda<br/>Partition Scaling]
    end
```

- **KEDA**: Scales workflow workers based on Kafka consumer lag and Temporal task queue depth
- **HPA**: Scales API gateway and services based on CPU/memory
- **ClickHouse**: Partition-based scaling with MergeTree engine
- **Redpanda**: Horizontal partition scaling for throughput
