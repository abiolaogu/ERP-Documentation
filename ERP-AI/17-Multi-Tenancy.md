# ERP-AI Multi-Tenancy Architecture

| Field | Value |
|---|---|
| Module | ERP-AI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Tenant Isolation Model

```mermaid
graph TB
    subgraph "Per-Tenant Isolation"
        subgraph "Tenant A"
            A_AGENTS[Agent Pods]
            A_MEMORY[Qdrant Partition]
            A_MODELS[Model Artifacts]
        end
        subgraph "Tenant B"
            B_AGENTS[Agent Pods]
            B_MEMORY[Qdrant Partition]
            B_MODELS[Model Artifacts]
        end
    end

    subgraph "Shared Infrastructure"
        SVC[ERP-AI Services]
        QD[(Qdrant Cluster)]
        PG[(PostgreSQL)]
    end

    A_AGENTS --> SVC
    B_AGENTS --> SVC
    A_MEMORY --> QD
    B_MEMORY --> QD
```

---

## 2. Isolation by Layer

| Layer | Mechanism |
|---|---|
| API | X-Tenant-ID header validated against JWT |
| Agent Pods | Kubernetes labels + network policies per tenant |
| Qdrant | Collection partitioning by tenant_id payload field |
| PostgreSQL | tenant_id column on all tables |
| Redis | Key prefix `{tenant_id}:` |
| Model Artifacts | Tenant-prefixed storage paths |
| Claude API | Separate API keys per enterprise tenant |
| Audit | Tenant-scoped audit trail |

---

## 3. Resource Limits

| Resource | Free | Professional | Enterprise |
|---|---|---|---|
| Concurrent agents | 5 | 50 | 500 |
| Agent executions/day | 100 | 5,000 | Unlimited |
| Copilot requests/min | 30 | 300 | 1,000 |
| NLP requests/min | 20 | 200 | 1,000 |
| Embeddings stored | 10,000 | 1,000,000 | 100,000,000 |
| ML models | 2 | 20 | Unlimited |
| Claude tokens/month | 100K | 5M | Unlimited |

---

## 4. Tenant Onboarding

```mermaid
graph LR
    A[Tenant Created] --> B[Create DB records]
    B --> C[Create Qdrant collections]
    C --> D[Initialize guardrail policies]
    D --> E[Register default agents]
    E --> F[Seed feature store]
    F --> G[Ready]
```

---

## 5. Tenant Data Deletion

1. Terminate all running agent pods
2. Delete Qdrant vectors (filter by tenant_id)
3. Delete PostgreSQL records (CASCADE)
4. Purge Redis keys
5. Delete model artifacts from storage
6. Retain audit logs per compliance policy
7. Publish `erp.ai.tenant.decommissioned`
