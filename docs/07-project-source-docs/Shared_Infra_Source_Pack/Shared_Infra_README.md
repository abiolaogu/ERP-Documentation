# shared-infra

Shared Sovereign ERP 2026 infrastructure. Module repos are frontend-only and connect to this shared runtime.

## Included Services

- yugabytedb
- redpanda
- redpanda-console
- redpanda-connect
- dragonflydb
- rustfs
- authentik
- centrifugo
- cosmo-router

## Startup Order

1. `docker compose up -d yugabytedb redpanda dragonflydb rustfs`
2. `docker compose up -d redpanda-console redpanda-connect`
3. `docker compose up -d authentik centrifugo cosmo-router`

## Local Endpoints

- YugabyteDB UI: `http://localhost:7000`
- Redpanda Console: `http://localhost:8080`
- Redpanda Connect: `http://localhost:4195`
- DragonflyDB: `localhost:6379`
- RustFS: `http://localhost:9001`
- Authentik: `http://localhost:9000`
- Centrifugo: `http://localhost:8000`
- Cosmo Router: `http://localhost:3002`

## Module Connectivity Contract

Module repos consume:

- `NEXT_PUBLIC_COSMO_URL=http://localhost:3002/graphql`
- `NEXT_PUBLIC_CENTRIFUGO_URL=http://localhost:8000/connection/websocket`

## Topic Naming Guardrail

`[env].[org].[module].[tenant].[entity].[event]`

Example:

`prod.abiola.healthcare.tnt_001.lab_result.updated`

## Documentation Index
- Start here: [docs/README.md](./docs/README.md)
- Architecture: [docs/01-architecture/SAD.md](./docs/01-architecture/SAD.md)
- DevOps and quality: [docs/03-quality-devops/CI-CD.md](./docs/03-quality-devops/CI-CD.md)
