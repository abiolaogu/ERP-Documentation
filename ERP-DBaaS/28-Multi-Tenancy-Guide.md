# ERP-DBaaS Multi-Tenancy Guide

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Multi-Tenancy Guide          |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Engineering                 |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [Tenant Isolation via tenant_id](#1-tenant-isolation-via-tenant_id)
2. [Namespace-Per-Tenant K8s Isolation](#2-namespace-per-tenant-k8s-isolation)
3. [Resource Quota Enforcement](#3-resource-quota-enforcement)
4. [Network Policy Isolation](#4-network-policy-isolation)
5. [Credential Separation](#5-credential-separation)
6. [Audit Trail per Tenant](#6-audit-trail-per-tenant)

---

## 1. Tenant Isolation via tenant_id

### 1.1 Data Model

All DBaaS platform tables enforce tenant isolation through the `tenant_id` column. This column is a `TEXT` type present on every table and is included in all queries as a mandatory filter.

```sql
-- Core tenant isolation pattern (consistent with all ERP modules)
CREATE TABLE instances (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id   TEXT NOT NULL,
    engine      TEXT NOT NULL,
    name        TEXT NOT NULL,
    size        TEXT NOT NULL,
    status      TEXT NOT NULL DEFAULT 'pending',
    config      JSONB DEFAULT '{}',
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Every table has a tenant_id index for query performance
CREATE INDEX idx_instances_tenant ON instances(tenant_id);

-- Composite unique constraints include tenant_id
CREATE UNIQUE INDEX idx_instances_tenant_name ON instances(tenant_id, name);
```

### 1.2 Hasura Row-Level Security

Hasura enforces tenant isolation at the GraphQL layer through row-level permissions. The `tenant_id` is extracted from the JWT token and applied automatically to all queries.

```json
{
  "role": "tenant_user",
  "permission": {
    "columns": ["id", "name", "engine", "size", "status", "config", "created_at"],
    "filter": {
      "tenant_id": {
        "_eq": "X-Hasura-Tenant-Id"
      }
    }
  }
}
```

### 1.3 API-Level Enforcement

The Node.js dbaas-api enforces tenant isolation at the middleware level. Every request must include a valid JWT with a `tenant_id` claim.

```typescript
// Middleware: extract and enforce tenant_id on every request
export function tenantMiddleware(req: Request, res: Response, next: NextFunction) {
  const tenantId = req.jwt?.claims?.tenant_id;
  if (!tenantId) {
    return res.status(403).json({ error: 'Missing tenant_id claim in JWT' });
  }

  // Attach tenant_id to request context for use in all downstream queries
  req.tenantId = tenantId;

  // Override any tenant_id in the request body to prevent spoofing
  if (req.body?.tenant_id && req.body.tenant_id !== tenantId) {
    return res.status(403).json({ error: 'tenant_id mismatch' });
  }
  req.body.tenant_id = tenantId;

  next();
}
```

### 1.4 Cross-Tenant Access Prevention

| Layer         | Enforcement Mechanism                              |
|---------------|-----------------------------------------------------|
| Gateway       | JWT validation, tenant_id extraction                |
| API           | Middleware injects tenant_id into all queries        |
| GraphQL       | Hasura row-level permissions filter by tenant_id     |
| Database      | All queries include `WHERE tenant_id = $1`          |
| Kubernetes    | Namespace-per-tenant with RBAC                       |
| Backups       | Storage paths partitioned by tenant_id               |

---

## 2. Namespace-Per-Tenant K8s Isolation

### 2.1 Namespace Strategy

Each tenant receives a dedicated Kubernetes namespace for their database instances. The namespace naming convention is:

```
dbaas-{tenant_id}
```

For example, tenant `acme-corp` would have namespace `dbaas-acme-corp`.

### 2.2 Namespace Provisioning

When a new tenant is onboarded, the following Kubernetes resources are created:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dbaas-acme-corp
  labels:
    dbaas.erp.io/tenant-id: acme-corp
    dbaas.erp.io/tier: tier-b
    dbaas.erp.io/managed-by: dbaas-control-plane
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dbaas-tenant-sa
  namespace: dbaas-acme-corp
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dbaas-tenant-binding
  namespace: dbaas-acme-corp
subjects:
  - kind: ServiceAccount
    name: dbaas-tenant-sa
    namespace: dbaas-acme-corp
roleRef:
  kind: ClusterRole
  name: dbaas-tenant-role
  apiGroup: rbac.authorization.k8s.io
```

### 2.3 Namespace Isolation Benefits

| Isolation Aspect      | Mechanism                                          |
|-----------------------|----------------------------------------------------|
| Resource visibility   | Tenants can only see resources in their namespace   |
| Resource limits       | ResourceQuota per namespace enforces tier limits     |
| Network              | NetworkPolicy restricts cross-namespace traffic      |
| RBAC                 | RoleBinding scoped to tenant namespace               |
| Secrets              | Credentials stored in tenant namespace only          |
| DNS                  | Services scoped to namespace DNS                     |

---

## 3. Resource Quota Enforcement

### 3.1 Kubernetes ResourceQuota

Each tenant namespace has a ResourceQuota that reflects their tier limits.

```yaml
# Tier B example
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dbaas-quota
  namespace: dbaas-acme-corp
spec:
  hard:
    requests.cpu: "80"
    requests.memory: "256Gi"
    limits.cpu: "80"
    limits.memory: "256Gi"
    persistentvolumeclaims: "60"
    requests.storage: "5Ti"
    pods: "100"
    services: "40"
    secrets: "60"
```

### 3.2 Application-Level Quota

In addition to K8s ResourceQuota, the dbaas-api maintains an application-level quota ledger for finer-grained control.

```sql
CREATE TABLE tenant_quotas (
    tenant_id           TEXT PRIMARY KEY,
    tier                TEXT NOT NULL DEFAULT 'tier-c',
    max_instances       INTEGER NOT NULL DEFAULT 5,
    max_cpu_cores       INTEGER NOT NULL DEFAULT 16,
    max_memory_gib      INTEGER NOT NULL DEFAULT 32,
    max_storage_gib     INTEGER NOT NULL DEFAULT 500,
    max_backup_gib      INTEGER NOT NULL DEFAULT 200,
    used_instances      INTEGER NOT NULL DEFAULT 0,
    used_cpu_cores      DECIMAL(10,2) NOT NULL DEFAULT 0,
    used_memory_gib     DECIMAL(10,2) NOT NULL DEFAULT 0,
    used_storage_gib    DECIMAL(10,2) NOT NULL DEFAULT 0,
    used_backup_gib     DECIMAL(10,2) NOT NULL DEFAULT 0,
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 3.3 Quota Check Flow

```
API Request ──> Extract tenant_id from JWT
      │
      ▼
Fetch tenant_quotas ──> Calculate requested resources
      │
      ▼
Compare (used + requested) vs max
      │
      ├── Within limits ──> APPROVE, update used_* counters
      │
      └── Exceeds limits ──> REJECT with quota violation response:
                              {
                                "error": "quota_exceeded",
                                "resource": "memory_gib",
                                "used": 240,
                                "requested": 32,
                                "limit": 256,
                                "suggestion": "Upgrade to Tier A or free resources"
                              }
```

### 3.4 Tier Limits Summary

| Resource         | Tier C (Starter) | Tier B (Professional) | Tier A (Enterprise) |
|------------------|------------------|-----------------------|---------------------|
| Max Instances    | 5                | 20                    | 50                  |
| Max CPU (cores)  | 16               | 80                    | 200                 |
| Max Memory (GiB) | 32               | 256                   | 512                 |
| Max Storage (GiB)| 500              | 5,120                 | 10,240              |
| Max Backup (GiB) | 200              | 2,048                 | 10,240              |
| Max Engines      | 3                | 6                     | 8                   |

---

## 4. Network Policy Isolation

### 4.1 Default Deny Policy

Each tenant namespace starts with a default deny-all policy, then allows only specific traffic patterns.

```yaml
# Default deny all ingress and egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: dbaas-acme-corp
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### 4.2 Allowed Traffic Patterns

```yaml
# Allow intra-namespace communication (database replicas)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-intra-namespace
  namespace: dbaas-acme-corp
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector: {}
  egress:
    - to:
        - podSelector: {}
---
# Allow traffic from dbaas-system (operators, backup controller)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-control-plane
  namespace: dbaas-acme-corp
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: dbaas-system
---
# Allow egress to RustFS for backups
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-rustfs
  namespace: dbaas-acme-corp
spec:
  podSelector:
    matchLabels:
      dbaas.erp.io/component: backup-agent
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.100.0/24  # RustFS subnet
      ports:
        - protocol: TCP
          port: 443
---
# Allow DNS resolution
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: dbaas-acme-corp
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

### 4.3 Network Isolation Matrix

| Source Namespace     | Destination Namespace     | Allowed? | Justification               |
|----------------------|---------------------------|----------|-----------------------------|
| dbaas-acme-corp      | dbaas-acme-corp           | Yes      | Intra-tenant replication     |
| dbaas-acme-corp      | dbaas-other-tenant        | No       | Cross-tenant isolation       |
| dbaas-system         | dbaas-acme-corp           | Yes      | Control plane operations     |
| dbaas-acme-corp      | dbaas-system              | No       | Tenants cannot reach control plane |
| dbaas-acme-corp      | kube-system (DNS only)    | Yes      | DNS resolution               |
| dbaas-acme-corp      | RustFS subnet             | Yes      | Backup operations only       |

---

## 5. Credential Separation

### 5.1 Credential Storage Strategy

Each tenant's database credentials are stored in Kubernetes Secrets within the tenant's own namespace. No cross-namespace Secret references are permitted.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: yugabyte-finance-prod-credentials
  namespace: dbaas-acme-corp
  labels:
    dbaas.erp.io/tenant-id: acme-corp
    dbaas.erp.io/instance-id: yugabyte-finance-prod
    dbaas.erp.io/engine: yugabytedb
type: Opaque
data:
  username: <base64>
  password: <base64>
  host: <base64>
  port: <base64>
  database: <base64>
  connection-string: <base64>
```

### 5.2 Credential Generation Rules

| Parameter              | Specification                                     |
|------------------------|---------------------------------------------------|
| Password length        | 32 characters minimum                             |
| Character set          | Uppercase + lowercase + digits + special (!@#$%^) |
| Entropy                | >= 192 bits                                       |
| Generation method      | `crypto/rand` (Go) or `crypto.randomBytes` (Node) |
| Storage                | K8s Secret (etcd at-rest encryption enabled)       |
| Rotation period        | 90 days default (configurable per tenant)          |
| Access                 | RBAC-restricted to tenant ServiceAccount           |

### 5.3 Cross-Tenant Credential Isolation

```
Tenant A Namespace                    Tenant B Namespace
┌─────────────────────┐              ┌─────────────────────┐
│ Secret: db-creds-a  │              │ Secret: db-creds-b  │
│ Password: Ax9#...   │              │ Password: Bz7$...   │
│                     │   BLOCKED    │                     │
│ ServiceAccount A    │──────X──────>│ ServiceAccount B    │
│ (can only read      │              │ (can only read      │
│  secrets in ns-a)   │              │  secrets in ns-b)   │
└─────────────────────┘              └─────────────────────┘
```

---

## 6. Audit Trail per Tenant

### 6.1 Audit Log Schema

```sql
CREATE TABLE audit_logs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       TEXT NOT NULL,
    actor_id        TEXT NOT NULL,
    actor_type      TEXT NOT NULL CHECK (actor_type IN ('user', 'service', 'system')),
    action          TEXT NOT NULL,
    resource_type   TEXT NOT NULL,
    resource_id     TEXT NOT NULL,
    details         JSONB DEFAULT '{}',
    ip_address      INET,
    user_agent      TEXT,
    status          TEXT NOT NULL CHECK (status IN ('success', 'failure', 'denied')),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_audit_tenant_time ON audit_logs(tenant_id, created_at DESC);
CREATE INDEX idx_audit_resource ON audit_logs(tenant_id, resource_type, resource_id);
CREATE INDEX idx_audit_actor ON audit_logs(tenant_id, actor_id);
```

### 6.2 Audited Actions

| Resource Type  | Actions Audited                                              |
|----------------|--------------------------------------------------------------|
| Instance       | create, update, scale, delete, start, stop, restart          |
| Backup         | create, restore, delete, schedule_update                     |
| Credential     | rotate, view_connection_string, download                     |
| Plugin         | install, uninstall, update                                   |
| Quota          | check, update_tier, override_request                         |
| User           | login, logout, permission_change                             |
| Configuration  | update_engine_config, update_backup_policy                   |

### 6.3 Audit Log Example Entry

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "tenant_id": "acme-corp",
  "actor_id": "user-john-doe",
  "actor_type": "user",
  "action": "scale",
  "resource_type": "instance",
  "resource_id": "yugabyte-finance-prod",
  "details": {
    "previous_size": "M",
    "new_size": "L",
    "previous_cpu": 2,
    "new_cpu": 4,
    "previous_memory_gib": 4,
    "new_memory_gib": 16
  },
  "ip_address": "10.2.5.100",
  "user_agent": "Mozilla/5.0 Chrome/122",
  "status": "success",
  "created_at": "2026-02-24T14:30:00Z"
}
```

### 6.4 Audit Retention Policy

| Tier   | Retention Period | Export Format   | Export Frequency |
|--------|------------------|-----------------|------------------|
| Tier A | 7 years          | JSON + Parquet  | Daily to RustFS  |
| Tier B | 3 years          | JSON            | Weekly to RustFS |
| Tier C | 1 year           | JSON            | Monthly to RustFS|

### 6.5 Compliance Reporting

Tenants can generate compliance reports from the audit trail through the API or dashboard:

- **Access Report**: All users who accessed a specific database instance within a time range.
- **Change Report**: All configuration changes made to instances, including who, when, and what changed.
- **Credential Report**: All credential rotation events with timestamps and actors.
- **Denied Access Report**: All failed authentication or authorization attempts.

These reports support SOC2 Type II, PCI-DSS, and internal audit requirements.

---

## Appendix: Multi-Tenancy Checklist

| Requirement                              | Status      | Enforcement Layer         |
|------------------------------------------|-------------|---------------------------|
| tenant_id on all database tables          | Implemented | Database schema            |
| Hasura row-level permissions              | Implemented | GraphQL layer              |
| API middleware tenant_id injection        | Implemented | Application layer          |
| K8s namespace-per-tenant                  | Implemented | Infrastructure layer       |
| ResourceQuota per namespace               | Implemented | Kubernetes                 |
| NetworkPolicy default-deny                | Implemented | Kubernetes                 |
| Credential isolation in namespace Secrets | Implemented | Kubernetes                 |
| Audit logging for all operations          | Implemented | Application layer          |
| Backup path segregation by tenant_id      | Implemented | Storage layer              |
| Cross-tenant access prevention tests      | Implemented | Integration test suite     |
