# ERP-BI Multi-Tenancy Architecture

| Field | Value |
|---|---|
| Module | ERP-BI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Tenancy Model

ERP-BI supports two multi-tenancy models depending on the subscription tier:

```mermaid
graph TB
    subgraph "Shared Schema (Professional Tier)"
        DB1[(Single ClickHouse DB)]
        T1[Tenant A rows]
        T2[Tenant B rows]
        T3[Tenant C rows]
        DB1 --> T1 & T2 & T3
    end

    subgraph "Dedicated Schema (Enterprise Tier)"
        DB2[(Tenant A DB)]
        DB3[(Tenant B DB)]
        DB4[(Tenant C DB)]
    end
```

---

## 2. Tenant Isolation Mechanisms

| Layer | Shared Schema | Dedicated Schema |
|---|---|---|
| ClickHouse | `tenant_id` column filter | Separate database |
| PostgreSQL | `tenant_id` column filter | Separate schema |
| Redis | Key prefix `{tenant_id}:` | Separate Redis DB |
| NATS | Subject filter `erp.{tenant_id}.*` | Separate stream |
| Object Storage | Prefix `{tenant_id}/` | Separate bucket |

---

## 3. Request Flow with Tenant Context

```mermaid
sequenceDiagram
    participant Client
    participant GW as API Gateway
    participant IAM as ERP-IAM
    participant SVC as BI Service
    participant DB as Database

    Client->>GW: Request + JWT + X-Tenant-ID
    GW->>IAM: Validate JWT
    IAM-->>GW: Token claims (tenant_id, roles)
    GW->>GW: Verify X-Tenant-ID == JWT.tenant_id
    GW->>SVC: Forward with tenant context
    SVC->>DB: Query WHERE tenant_id = ?
    DB-->>SVC: Tenant-scoped results
    SVC-->>Client: Response
```

---

## 4. Tenant-Scoped Caching

Cache keys are always namespaced by tenant:

```
{tenant_id}:{service}:{resource}:{hash}

Example:
tenant_001:query-engine:model_sales:abc123def456
```

Cache eviction is per-tenant. One tenant's cache operations never affect another.

---

## 5. Resource Limits per Tenant

| Resource | Free | Professional | Enterprise |
|---|---|---|---|
| Dashboards | 5 | 50 | Unlimited |
| Reports | 10 | 100 | Unlimited |
| Scheduled reports | 2 | 20 | Unlimited |
| Alert rules | 5 | 50 | Unlimited |
| NLQ queries/day | 20 | 200 | Unlimited |
| Data storage | 1 GB | 100 GB | 10 TB |
| Users | 5 | 50 | Unlimited |
| API rate (req/min) | 60 | 300 | 1,000 |

---

## 6. Tenant Onboarding

```mermaid
graph LR
    A[Create Tenant in ERP-Platform] --> B[Provision BI resources]
    B --> C[Create ClickHouse schema/tables]
    C --> D[Configure NATS subscriptions]
    D --> E[Initialize CDC ingestion]
    E --> F[Create default dashboards]
    F --> G[Tenant ready]
```

---

## 7. Tenant Data Deletion

When a tenant is decommissioned:
1. Stop CDC ingestion for tenant
2. Delete all ClickHouse data (DROP TABLE or DELETE WHERE tenant_id)
3. Delete PostgreSQL metadata
4. Purge Redis cache keys
5. Delete Object Storage files
6. Publish `erp.bi.tenant.decommissioned` event
7. Retain audit logs per retention policy
