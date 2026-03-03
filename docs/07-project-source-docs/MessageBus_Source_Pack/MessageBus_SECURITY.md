# Security Model — ERP-MessageBus

## Authentication

### Broker Authentication (SASL/SCRAM)
- All clients authenticate via SASL/SCRAM-SHA-256
- Service accounts follow pattern: `svc-erp-<module>`
- Credentials managed via Kubernetes secrets (never hardcoded)

### Console Authentication (OIDC)
- Redpanda Console authenticates via Authentik OIDC
- Issuer: `http://localhost:9000/application/o/erp/` (dev)
- Roles mapped from Authentik groups to Console RBAC

## Authorization

### Topic-Level RBAC

| Role | Access Pattern | Permissions |
|------|---------------|-------------|
| `infra-admin` | `*` | Full (create/delete/read/write/alter) |
| `team-<module>` | `*.*.erp.<module>.*` | Read + Write |
| `team-<module>-readonly` | `*.*.erp.<module>.*` | Read only |
| `aiops` | `*.*.erp.*` | Read (all modules for observability) |
| `observability` | `*.*.erp.*` | Read (all modules for metrics) |

### Consumer Group RBAC

Consumer groups follow `cg.[env].[org].[module].[service]` convention.
Each team can only create/manage groups under their module prefix.

## Encryption

### In Transit
- **Dev**: Plaintext (localhost only)
- **Staging/Prod**: mTLS between all broker-client connections
- Certificates managed via cert-manager in Kubernetes

### At Rest
- Tiered storage to RustFS uses server-side encryption
- Broker disks use volume encryption in Kubernetes

## Secrets Management

- SASL credentials: Kubernetes Secrets (never in source)
- OIDC client secrets: Kubernetes Secrets
- mTLS certificates: cert-manager + Kubernetes Secrets
- Placeholders provided in `.env.example` — replace before deployment

## Audit

- All topic creation/deletion logged via Redpanda audit log
- Console access logged via Authentik audit trail
- AIOps consumes audit events from `*.*.erp.aiops.audit.actions`
