# ERP-DBaaS Changelog

All notable changes to the ERP-DBaaS module are documented in this file. This project adheres to [Semantic Versioning](https://semver.org/).

---

## [v1.0.0] - 2026-02-24

### Summary

General Availability release of ERP-DBaaS. This release marks production readiness for all 8 AIDD-approved database engines, full AIDD governance enforcement, and enterprise-grade backup/restore with RustFS.

### Added

- **Production hardening**: Circuit breakers on all operator adapter calls with configurable retry policies.
- **Audit logging**: Comprehensive audit trail for all provisioning, scaling, backup, credential rotation, and decommission operations; events emitted to Apache Pulsar for downstream consumption.
- **HPA autoscaling**: Horizontal Pod Autoscaler for both dbaas-gateway and dbaas-api deployments with CPU/memory-based scaling policies.
- **Helmfile deployment**: Helmfile-based deployment pipeline for staging and production environments.
- **RBAC enforcement**: Role-based access control validation on all mutating endpoints (dbaas_admin, dbaas_operator, dbaas_viewer).
- **Metering completeness**: Metering events emitted for all lifecycle operations including usage sampling, provision, scale, backup, restore, and deprovision.

### Changed

- Upgraded YugabyteDB base image to 2.21.1.0-b271 for service registry.
- Increased default connection pool max from 10 to 20 for YugabyteDB connections.
- Rate limiter now uses sliding window counter instead of fixed window for more accurate rate limiting.
- Improved error messages on policy violations to include the full list of allowed engines and a pointer to the COMMERCIAL_FLEXIBLE profile.

### Fixed

- Fixed race condition in credential rotation where concurrent rotation requests for the same instance could cause deadlock.
- Fixed tenant quota decrement on decommission to use `GREATEST(current_instances - 1, 0)` to prevent negative counts.
- Fixed backup status not reverting to `running` when backup job fails.

### Security

- Enabled `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and `Referrer-Policy: no-referrer` on all gateway responses.
- Added `ReadHeaderTimeout` (2s), `ReadTimeout` (15s), `WriteTimeout` (30s), and `IdleTimeout` (120s) to the Go gateway HTTP server to mitigate slowloris attacks.
- MaxHeaderBytes limited to 1 MiB.

---

## [v0.9.0] - 2026-02-10

### Summary

Release Candidate 1. All core features complete; focus on security hardening, observability, and documentation.

### Added

- **NetworkPolicy enforcement**: Ingress and egress NetworkPolicies for dbaas-api and dbaas-gateway pods restricting traffic to only approved namespaces and ports.
- **CORS configuration**: CORS origins configurable via `CORS_ORIGINS` environment variable with support for comma-separated origin lists; defaults to `*` in development.
- **Request ID propagation**: Automatic `X-Request-ID` injection on all requests; forwarded through gateway to API; included in all structured log entries.
- **Plugin gRPC connectivity check**: Pre-registration connectivity validation to the plugin's gRPC endpoint before allowing registration to proceed.
- **Policy profile CRD**: New `PolicyProfile` CRD (`policyprofiles.dbaas.businessactivation.cloud`) for declarative profile management.

### Changed

- Tenant context middleware now caches profile/tier lookups for 5 minutes to reduce YugabyteDB query load.
- Plugin validation is now scheduled asynchronously with a 5-second delay to avoid blocking the registration response.

### Fixed

- Fixed JWKS cache not refreshing after TTL expiry (1 hour) due to incorrect timestamp comparison.
- Fixed plugin deregistration not cleaning up associated CRD resources.

---

## [v0.8.0] - 2026-01-27

### Summary

Plugin system and commercial profile implementation.

### Added

- **Plugin system**: Full plugin registration, validation, and lifecycle management via gRPC interface.
  - `POST /v1/dbaas/plugins` for plugin registration.
  - `GET /v1/dbaas/plugins` with status and engine filtering.
  - gRPC `LifecycleService` with 8 lifecycle hooks: `GetCapabilities`, `OnProvision`, `OnScale`, `OnBackup`, `OnRestore`, `OnRotateCredentials`, `OnDeprovision`, `HealthCheck`.
  - Plugin validation pipeline: gRPC connectivity check, capability verification, security context validation.
- **PluginRegistration CRD**: `pluginregistrations.dbaas.businessactivation.cloud` for K8s-native plugin management.
- **COMMERCIAL_FLEXIBLE profile**: Full implementation allowing PostgreSQL, MySQL, MariaDB, and Redis/Valkey engines for third-party customer workloads.
  - Minimum 7-day backup retention (vs. 30 days for ERP_AIDD_STRICT).
  - Cross-region backup replication optional.
  - Plugin-registered engines supported.
- **Profile validation**: `ProfileEngine` class with full validation chain for engine, HA mode, region, config, plugin engine, plugin capabilities, and backup configuration.

### Changed

- Refactored policy validation into dedicated `ProfileImplementation` interface with `ErpAiddStrictProfile` and `CommercialFlexibleProfile` implementations.
- Plugin name validation now enforces DNS-compatible naming: `^[a-z0-9][a-z0-9-]*[a-z0-9]$`, 3-64 characters.

---

## [v0.7.0] - 2026-01-13

### Summary

Credential management and rotation with HashiCorp Vault integration.

### Added

- **Credential rotation**: API endpoints for credential retrieval and rotation.
  - `GET /v1/dbaas/instances/:instanceId/credentials` to retrieve current credentials from Vault.
  - `POST /v1/dbaas/instances/:instanceId/credentials/rotate` to initiate zero-downtime credential rotation.
- **credential_rotations table**: Tracks rotation history including initiator, status, and timestamps.
- **Vault integration**: `CredentialRotator` controller for reading and rotating credentials via HashiCorp Vault's KV v2 and database secret engines.
- **Concurrent rotation guard**: Prevents multiple simultaneous credential rotations for the same instance.
- **RestoreJob CRD**: `restorejobs.dbaas.businessactivation.cloud` for declarative restore job management.

### Changed

- Instance status check now required for credential retrieval (instance must be in `running` state).
- Added `RotationStatus` enum: `pending`, `in_progress`, `completed`, `failed`.

---

## [v0.6.0] - 2025-12-30

### Summary

Backup and restore with RustFS storage backend.

### Added

- **Backup system**: Full backup lifecycle with RustFS as the AIDD-approved object storage backend.
  - `POST /v1/dbaas/instances/:instanceId/backup` to trigger on-demand backups.
  - `GET /v1/dbaas/instances/:instanceId/backups` with pagination.
  - `POST /v1/dbaas/instances/:instanceId/restore` for point-in-time restore.
- **BackupPolicy CRD**: `backuppolicies.dbaas.businessactivation.cloud` with cron scheduling, retention, encryption, cross-region replication, and compression options (none, gzip, zstd).
- **Backup types**: Full, incremental, and snapshot backups.
- **backup_records table**: Tracks backup ID, instance ID, type, status, storage path, size, retention, and encryption status.
- **BackupManager controller**: Async backup/restore orchestration with status tracking and error recovery.

### Changed

- Instance status transitions now include `backing_up` and `restoring` states.
- Backup encryption is mandatory across all profiles.

---

## [v0.5.0] - 2025-12-16

### Summary

Multi-engine operator adapters and CRD-driven provisioning.

### Added

- **Operator adapters**: Engine-specific operator adapters for all 8 AIDD-approved engines:
  - `kubedb-adapter.ts` for YugabyteDB, DragonflyDB, and MongoDB (KubeDB operator).
  - `scylla-adapter.ts` for ScyllaDB (Scylla Operator).
  - `clickhouse-adapter.ts` for ClickHouse (Altinity Operator).
  - `timescaledb-adapter.ts` for TimescaleDB (Zalando Postgres Operator).
  - `questdb-adapter.ts` for QuestDB (custom StatefulSet controller).
  - `couchdb-adapter.ts` for CouchDB (custom StatefulSet controller).
  - `pulsar-adapter.ts` for Apache Pulsar (StreamNative Operator, internal messaging).
- **ServiceInstance CRD**: `serviceinstances.dbaas.businessactivation.cloud` with spec fields for tenantId, engine, version, plan (S/M/L/XL), haMode (standalone/ha/multi_region), region, config, backupPolicy, and credentials.
- **TenantDataPlane CRD**: `tenantdataplanes.dbaas.businessactivation.cloud` for cluster-scoped tenant data plane management with isolation level, tier, region, namespace allocation, resource quotas, compliance requirements, and backup tier.
- **Operator manifest templates**: YAML manifests in `/operators/` for all engine cluster definitions.

### Changed

- Provisioning now creates a ServiceInstance CRD which triggers the appropriate K8s operator.
- Scaling operations applied through CRD spec patches rather than direct API calls.

---

## [v0.4.0] - 2025-12-02

### Summary

Tenant quota management and metering.

### Added

- **Tenant quotas**: Tier-based resource quota enforcement (A, B, C tiers).
  - Tier A: 50 instances, 128 CPU, 512Gi memory, 5Ti storage.
  - Tier B: 20 instances, 64 CPU, 256Gi memory, 2Ti storage.
  - Tier C: 5 instances, 16 CPU, 64Gi memory, 500Gi storage.
- **tenant_quotas table**: Tracks per-tenant limits and current usage.
- **Metering pipeline**: `PulsarEmitter` for emitting metering events to Apache Pulsar.
- **metering_events table**: Records CPU seconds, memory GB-seconds, storage GB-hours, and I/O operations per instance.
- **Quota enforcement**: Provisioning blocked when tenant reaches max instance limit (HTTP 429).
- **Tenant context middleware**: Enriches requests with tenant profile, tier, and quota information; auto-creates default quota for new tenants.
- **Profile cache**: In-memory cache for tenant profiles with 5-minute TTL.

### Changed

- Decommission now decrements `current_instances` in tenant quota.
- Added `X-Tenant-ID`, `X-Tenant-Profile`, and `X-Tenant-Tier` response headers.

---

## [v0.3.0] - 2025-11-18

### Summary

AIDD governance enforcement and authentication.

### Added

- **ERP_AIDD_STRICT profile**: Immutable policy profile compiled into the application.
  - Allowed engines: YugabyteDB, ScyllaDB, DragonflyDB, MongoDB, CouchDB, ClickHouse, TimescaleDB, QuestDB.
  - Denied engines: PostgreSQL, MySQL, MariaDB, Redis/Valkey (plus aliases: postgres, pg, redis, valkey).
  - Mandatory backup encryption and cross-region replication.
  - Minimum 30-day backup retention.
  - Forbidden config keys: `ssl_disabled`, `auth_disabled`, `allow_external_access`, `disable_encryption`, `disable_audit_log`.
  - TLS/SSL cannot be disabled.
- **Authentik JWT authentication**: Middleware for validating JWT tokens issued by Authentik.
  - JWKS endpoint support with 1-hour cache TTL.
  - Symmetric key fallback for development.
  - Module-level access control (`ERP-DBaaS` module claim required).
  - Role validation: `dbaas_admin`, `dbaas_operator`, `dbaas_viewer`, `admin`, `platform_admin`.
- **Development auth bypass**: `X-Dev-Tenant-ID` header for local development without Authentik.

### Changed

- All provisioning requests now validated against the tenant's active policy profile before proceeding.
- Improved error responses to include `requestId` for traceability.

---

## [v0.2.0] - 2025-11-04

### Summary

Core API implementation with Express.js and DragonflyDB rate limiting.

### Added

- **Express.js API server**: Node.js API on port 3000 with structured Pino logging.
  - `POST /v1/dbaas/instances` for provisioning.
  - `GET /v1/dbaas/instances` with pagination, engine, and status filters.
  - `GET /v1/dbaas/instances/:id` for instance details.
  - `GET /v1/dbaas/instances/:id/metrics` for instance metrics and metering data.
  - `PUT /v1/dbaas/instances/:id/scale` for scaling operations.
  - `PUT /v1/dbaas/instances/:id/config` for configuration updates.
  - `POST /v1/dbaas/instances/:id/restart` for rolling restarts.
  - `DELETE /v1/dbaas/instances/:id` for decommissioning.
- **Request validation**: Zod-based schema validation on all mutating endpoints.
- **DragonflyDB rate limiter**: Token bucket rate limiter using DragonflyDB (Redis-protocol compatible) with tier-based limits (A: 1000/min, B: 500/min, C: 100/min).
- **Engine and plan endpoints**:
  - `GET /v1/dbaas/engines` listing all available engines filtered by tenant profile.
  - `GET /v1/dbaas/plans` listing S/M/L/XL plans with resource specs and pricing.
  - `GET /v1/dbaas/profiles` listing policy profile definitions.
- **Service instance lifecycle**: Full status machine: provisioning, running, scaling, backing_up, restoring, failed, decommissioning, decommissioned.

### Changed

- Instance namespace format standardized to `dbaas-{tenantId}-{instanceIdPrefix}`.

---

## [v0.1.0] - 2025-10-20

### Summary

Initial project scaffolding and infrastructure setup.

### Added

- **Go API gateway**: Reverse proxy gateway on port 8090 with health check (`/healthz`) and capabilities (`/v1/capabilities`) endpoints.
  - Routes: `/v1/dbaas/instances`, `/v1/dbaas/backups`, `/v1/dbaas/credentials`, `/v1/dbaas/engines`, `/v1/dbaas/plans`, `/v1/dbaas/profiles`, `/v1/dbaas/plugins`, `/v1/dbaas/tenants`, `/v1/dbaas/health`.
  - Configurable backend URL via `DBAAS_API_URL` environment variable.
  - Security headers on all responses.
- **Docker Compose infrastructure**: Development environment with YugabyteDB 2.21, DragonflyDB, Hasura GraphQL Engine v2.40, and dbaas-api/gateway services.
- **YugabyteDB schema**: Initial migration (`001_initial_schema.sql`) with 6 tables:
  - `service_instances`: Instance registry with engine, version, plan, HA mode, status, namespace, endpoint, region, profile, config, resource usage.
  - `backup_records`: Backup history with type, status, storage path, size, retention, encryption.
  - `plugin_registrations`: Plugin registry with gRPC endpoint, status, capabilities.
  - `tenant_quotas`: Per-tenant resource quotas and current usage.
  - `credential_rotations`: Credential rotation audit trail.
  - `metering_events`: Usage metering with CPU, memory, storage, and I/O metrics.
- **CRDs**: 6 Custom Resource Definitions under `dbaas.businessactivation.cloud` API group:
  - `ServiceInstance`, `BackupPolicy`, `RestoreJob`, `TenantDataPlane`, `PluginRegistration`, `PolicyProfile`.
- **Kubernetes manifests**: Namespace, deployments, services, HPA, and NetworkPolicies.
- **Hasura federation**: Apollo Federation v2 enabled with JWT claims mapping from Authentik.
- **Makefile**: Build, dev, test, lint, Docker, migration, CRD, and K8s deployment targets.
- **capabilities.json**: Module capability descriptor for federation registry.

---

## Version Comparison Matrix

| Feature | v0.1.0 | v0.2.0 | v0.3.0 | v0.4.0 | v0.5.0 | v0.6.0 | v0.7.0 | v0.8.0 | v0.9.0 | v1.0.0 |
|---|---|---|---|---|---|---|---|---|---|---|
| Go Gateway | X | X | X | X | X | X | X | X | X | X |
| Express.js API | - | X | X | X | X | X | X | X | X | X |
| Instance Provisioning | - | X | X | X | X | X | X | X | X | X |
| AIDD Policy Enforcement | - | - | X | X | X | X | X | X | X | X |
| Tenant Quotas | - | - | - | X | X | X | X | X | X | X |
| K8s Operator Adapters | - | - | - | - | X | X | X | X | X | X |
| Backup/Restore | - | - | - | - | - | X | X | X | X | X |
| Credential Rotation | - | - | - | - | - | - | X | X | X | X |
| Plugin System | - | - | - | - | - | - | - | X | X | X |
| NetworkPolicies | - | - | - | - | - | - | - | - | X | X |
| HPA Autoscaling | - | - | - | - | - | - | - | - | - | X |
| Audit Logging | - | - | - | - | - | - | - | - | - | X |

---

## Upgrading

### From v0.9.x to v1.0.0

1. Apply updated CRDs: `kubectl apply -f crds/`
2. Run database migration: `make migrate`
3. Update Helm values for HPA configuration.
4. Rolling update of dbaas-api and dbaas-gateway deployments.
5. Verify NetworkPolicies are applied: `kubectl get networkpolicies -n dbaas-system`

### From v0.8.x to v0.9.0

1. Apply new PolicyProfile CRD: `kubectl apply -f crds/policy-profile.yaml`
2. Update NetworkPolicy manifests: `kubectl apply -f infra/k8s/network-policies.yaml`
3. Configure `CORS_ORIGINS` environment variable if restricting origins.

### From v0.7.x to v0.8.0

1. Apply PluginRegistration CRD: `kubectl apply -f crds/plugin-registration.yaml`
2. Ensure gRPC port 50051 is accessible from dbaas-api pods for plugin communication.
3. Run database migration for `plugin_registrations` table.
