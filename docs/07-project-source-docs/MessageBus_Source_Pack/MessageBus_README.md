# ERP-MessageBus

Shared multi-tenant Redpanda data plane for the Sovereign ERP 2026 platform. This repo is the single source of truth for topic governance, Connect pipeline submissions, Console RBAC, and schema registry management.

## Architecture

- **Redpanda Cluster**: Shared multi-tenant Kafka-compatible broker (deployed via shared-infra)
- **Redpanda Connect**: Stateless pipeline farm — teams submit YAML pipelines here
- **Redpanda Console**: Governance UI with OIDC via Authentik and prefix-based RBAC
- **Schema Registry**: Centralized Avro/Protobuf/JSON schema management

## Topic Naming Convention

All topics MUST follow:

```
[env].[org].erp.[module].[topic_name]
```

Examples:
- `prod.abiola.erp.healthcare.lab_result.updated`
- `dev.abiola.erp.finance.invoice.created`
- `prod.abiola.erp.aiops.health.heartbeat`

## Consumer Group Convention

```
cg.[env].[org].[module].[service]
```

Examples:
- `cg.prod.abiola.healthcare.portal`
- `cg.dev.abiola.finance.ledger-sync`

## Submitting a Connect Pipeline

1. Create your pipeline YAML under `configs/connect/pipelines/platforms/<module>/`
2. Follow the Bloblang envelope stamping template in `configs/connect/pipelines/templates/`
3. Submit a PR — the infra-admin team reviews for naming, quota, and security compliance

## Endpoints (from shared-infra)

| Service | Dev URL |
|---------|---------|
| Redpanda Broker | `localhost:9092` |
| Redpanda Console | `http://localhost:8080` |
| Redpanda Connect | `http://localhost:4195` |
| Schema Registry | `http://localhost:8081` |

## Security Model

- mTLS between brokers and clients (prod)
- SASL/SCRAM for client authentication
- OIDC via Authentik for Console access
- Prefix-based RBAC: teams can only access `[env].[org].erp.<their-module>.*`
- See [SECURITY.md](./SECURITY.md) for details

## Related

- [OPERATIONS.md](./OPERATIONS.md) — runbook and health checks
- [MIGRATION.md](./MIGRATION.md) — migrating from per-module brokers
- [SECURITY.md](./SECURITY.md) — mTLS, SASL, RBAC details
- [configs/topic-catalog/TOPICS.md](./configs/topic-catalog/TOPICS.md) — canonical topic registry

## Documentation Index
- Start here: [docs/README.md](./docs/README.md)
- Architecture: [docs/01-architecture/SAD.md](./docs/01-architecture/SAD.md)
- DevOps and quality: [docs/03-quality-devops/CI-CD.md](./docs/03-quality-devops/CI-CD.md)
