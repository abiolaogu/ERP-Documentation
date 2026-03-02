# Shared Infrastructure Architecture

All ERP modules must consume a centralized infrastructure contract.

| Component | Role | Connection Type |
|---|---|---|
| Hasura | Unified API Gateway | GraphQL |
| ERP-DBaaS | Shared Persistence Layer | PostgreSQL / SQL |
| ERP-IAM | Identity & Access | OAuth2 / JWT |
| ERP-Observability | Monitoring | gRPC / Prometheus |
| Workik | Local Agent | CLI / Webhook |

## Standard Endpoint Contract

- `ERP_SHARED_HASURA_URL`: `http://localhost:8090/v1/graphql`
- `ERP_SHARED_HASURA_WS_URL`: `ws://localhost:8090/v1/graphql`
- `ERP_SHARED_DB_URL`: `postgres://erp_admin:changeme_in_production@localhost:5432/erp_shared?sslmode=disable`
- `ERP_SHARED_IAM_URL`: `http://localhost:8081`
- `ERP_SHARED_OTLP_ENDPOINT`: `http://localhost:4317`

## Migration Rules

1. All module web/data providers use `ERP_SHARED_HASURA_URL` (or Vite equivalent) for GraphQL data access.
2. Local project database containers/configurations are removed from module-level compose files.
3. Local IAM containers/configurations are removed from module-level compose files; auth points to `ERP_SHARED_IAM_URL`.
4. Logging and telemetry exporters point to `ERP_SHARED_OTLP_ENDPOINT`.
5. Workik automation lives in `.workik/config.yml` in every module root.
