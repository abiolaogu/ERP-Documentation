# Sovereign SaaS 2026 Standardization Tranche Report

Date: 2026-03-01

## Scope Executed

This tranche applied the new Sovereign SaaS architecture baseline across the full ERP portfolio:

- Data layer standard: YugabyteDB
- Cache standard: DragonflyDB
- Event standard: Redpanda + Redpanda Connect
- Realtime standard: Centrifugo
- API federation target: WunderGraph Cosmo + Hasura DDN + Directus
- Frontend standard target: Next.js 16 + Shadcn + Workik service-layer pattern

## New Shared Infrastructure Bundle

Path: `/Users/AbiolaOgunsakin1/ERP/infra/sovereign-saas-2026`

Delivered artifacts:

- `docker-compose.yml`
- `.env.example`
- `redpanda-connect/connect-config.yaml`
- `centrifugo/config.json`
- `hasura-ddn/inventory-table-metadata.yaml`
- `k8s/namespace.yaml`
- `k8s/yugabytedb.yaml`
- `k8s/dragonflydb.yaml`
- `k8s/redpanda.yaml`
- `k8s/redpanda-console.yaml`
- `k8s/redpanda-connect.yaml`
- `k8s/centrifugo.yaml`
- `k8s/control-plane.yaml`
- `k8s/secrets.example.yaml`

## Frontend Standard Template

Path: `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/templates/next16-workik`

Template includes:

- Next.js 16 app-router scaffold with PPR enabled (`next.config.ts`)
- Workik-style feature slicing (`features/*/services`, `features/*/hooks`)
- Zod schema validation for inventory domain
- TanStack Query hooks with optimistic mutation behavior
- Centrifugo realtime provider that invalidates entity caches
- Shadcn-style UI primitives and responsive sidebar/sheet navigation

## Portfolio-Wide Application

Automation script:

- `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/apply-next16-workik-standard.sh`

Applied results:

- Repos with new `apps/sovereign-next16` scaffold: **24/24**
- Repos with Workik config upgraded to `version: 2`: **24/24**
- Repos with `framework: nextjs16` and `ui: shadcn` in `.workik/config.yml`: **24/24**
- Repos with migration runbook file: **24/24** (`docs/sovereign/SOVEREIGN_NEXT16_WORKIK_MIGRATION_2026-03-01.md`)

Operational standardization add-ons:

- Root orchestration now prioritizes `apps/sovereign-next16` for `make frontend`, `make frontend-install`, `make frontend-build`, and `make dev-module`.
- New cutover validation gate:
  - `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/ga-next16-cutover-check.sh`
  - Make target: `make ga-next16`
  - Integrated into `make ga-all`
- Portfolio cutover status report generated:
  - `/Users/AbiolaOgunsakin1/ERP/Documentation/SOVEREIGN_NEXT16_CUTOVER_STATUS.md`
- Legacy frontend manifest retirement script added and executed:
  - `/Users/AbiolaOgunsakin1/ERP/tools/sovereign/retire-legacy-frontend-manifests.sh`
  - Result: legacy Refine/AntD manifest debt reduced to `0/24`
- README standard marker applied across repositories:
  - `<!-- SOVEREIGN_NEXT16_STANDARD_2026_03 -->` in `24/24` repo `README.md` files

## Guardrail Alignment (AIDD)

- Tenant-aware data contract is preserved in service layer and realtime invalidation keying.
- Mutation-level validation is codified in service methods and schema parsing.
- Migration is non-destructive: legacy UIs remain intact until parity cutover is complete.

## Remaining Cutover Work (Execution Queue)

1. Port each domain module from legacy frontend stacks into `apps/sovereign-next16/features`.
2. Complete parity migration and retire transitional legacy UI stacks.
3. Remove residual legacy source folders after parity and regression tests pass.
4. Enforce `GA_NEXT16_BUILD_ENFORCE=1` in CI for all repositories once dependency install windows are allocated.
