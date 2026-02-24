# ERP-DBaaS Architecture Decision Records (ADRs)

This document captures the key architectural decisions made during the design and implementation of ERP-DBaaS. Each ADR follows a structured format: context, decision, rationale, consequences, and status.

---

## Table of Contents

1. [ADR-001: Kubernetes Operator Pattern for Database Lifecycle Management](#adr-001-kubernetes-operator-pattern-for-database-lifecycle-management)
2. [ADR-002: CRD-Driven Provisioning Model](#adr-002-crd-driven-provisioning-model)
3. [ADR-003: AIDD Strict Policy as Immutable Compiled Profile](#adr-003-aidd-strict-policy-as-immutable-compiled-profile)
4. [ADR-004: gRPC Plugin Interface for Engine Extensibility](#adr-004-grpc-plugin-interface-for-engine-extensibility)
5. [ADR-005: DragonflyDB for Rate Limiting and Caching](#adr-005-dragonflydb-for-rate-limiting-and-caching)
6. [ADR-006: Tier-Based Quota System](#adr-006-tier-based-quota-system)
7. [ADR-007: Refine.dev for Frontend Framework](#adr-007-refinedev-for-frontend-framework)
8. [ADR-008: Express.js for API Layer Flexibility](#adr-008-expressjs-for-api-layer-flexibility)

---

## ADR-001: Kubernetes Operator Pattern for Database Lifecycle Management

**Status**: Accepted
**Date**: 2025-10-15
**Deciders**: Platform Engineering Team

### Context

ERP-DBaaS must manage the full lifecycle of 8+ database engines across multiple Kubernetes clusters. Each engine has unique operational requirements for provisioning, scaling, backup, failover, and version upgrades. We needed a pattern that could handle:

- Engine-specific lifecycle logic (e.g., YugabyteDB tablet rebalancing during scaling, ScyllaDB shard migration).
- Declarative desired-state management rather than imperative scripting.
- Self-healing behavior (automatic recovery from pod failures, node drains).
- Integration with existing K8s ecosystem tooling.

### Decision

We adopt the Kubernetes Operator pattern for all database lifecycle management. Each supported engine is managed by either an upstream open-source operator or a custom operator:

| Engine | Operator | Type |
|---|---|---|
| YugabyteDB | KubeDB | Upstream |
| ScyllaDB | Scylla Operator | Upstream |
| DragonflyDB | KubeDB | Upstream |
| MongoDB | KubeDB | Upstream |
| CouchDB | Custom StatefulSet Controller | Custom |
| ClickHouse | Altinity Operator | Upstream |
| TimescaleDB | Zalando Postgres Operator | Upstream |
| QuestDB | Custom StatefulSet Controller | Custom |

The DBaaS API communicates with operators exclusively through Kubernetes CRD spec changes. The API never directly manages pods, StatefulSets, or PVCs.

### Rationale

1. **Declarative convergence**: Operators continuously reconcile desired state with actual state, providing self-healing without DBaaS API intervention.
2. **Separation of concerns**: Database operational knowledge lives in the operator (domain expert), while tenant/business logic lives in the DBaaS API.
3. **Ecosystem leverage**: Using upstream operators (KubeDB, Scylla, Altinity, Zalando) gives us production-hardened lifecycle management without building it from scratch.
4. **Extensibility**: New engines can be added by deploying a new operator and writing an adapter in the DBaaS API.
5. **Auditability**: All state changes flow through the Kubernetes API server and are captured in etcd, providing a complete audit trail.

### Consequences

**Positive**:
- Self-healing and automatic failover for all managed databases.
- Consistent operational model across all engines.
- Kubernetes-native observability (events, conditions, metrics).
- Operator upgrades are independent of DBaaS API releases.

**Negative**:
- Operator version compatibility must be tracked and tested against K8s versions.
- Custom operators for CouchDB and QuestDB require in-house maintenance.
- Debugging requires understanding both the DBaaS API layer and the operator reconciliation loop.
- Increased cluster-level resource consumption from operator controller pods.

### Alternatives Considered

1. **Direct K8s API management (StatefulSets)**: Rejected because it would require reimplementing all engine-specific lifecycle logic in the DBaaS API.
2. **Helm-based provisioning**: Rejected because Helm lacks a reconciliation loop; it is fire-and-forget, providing no self-healing.
3. **VM-based provisioning**: Rejected because it conflicts with the container-native architecture of the ERP platform and increases operational overhead.

---

## ADR-002: CRD-Driven Provisioning Model

**Status**: Accepted
**Date**: 2025-10-22
**Deciders**: Platform Engineering Team

### Context

The DBaaS API needs a mechanism to request database provisioning that integrates naturally with the operator pattern (ADR-001). We needed to define how provisioning requests flow from the API to the cluster.

### Decision

We use Custom Resource Definitions (CRDs) as the primary interface between the DBaaS API and Kubernetes operators. The DBaaS platform defines 6 CRDs under the `dbaas.businessactivation.cloud` API group:

1. **ServiceInstance** (`serviceinstances.dbaas.businessactivation.cloud`): Represents a managed database instance with engine, version, plan, HA mode, region, config, and backup policy.
2. **BackupPolicy** (`backuppolicies.dbaas.businessactivation.cloud`): Defines scheduled and on-demand backup configuration with retention, encryption, cross-region replication, and compression.
3. **RestoreJob** (`restorejobs.dbaas.businessactivation.cloud`): Represents a one-shot restore operation from a backup to a target instance.
4. **TenantDataPlane** (`tenantdataplanes.dbaas.businessactivation.cloud`): Cluster-scoped resource defining a tenant's data plane with isolation level, resource quotas, compliance requirements, and region allocation.
5. **PluginRegistration** (`pluginregistrations.dbaas.businessactivation.cloud`): Cluster-scoped resource registering a plugin with its gRPC endpoint, capabilities, and security context.
6. **PolicyProfile** (`policyprofiles.dbaas.businessactivation.cloud`): Cluster-scoped resource defining allowed/denied engines, compliance requirements, and backup requirements per profile.

### Provisioning Flow

```
Client Request
    |
    v
Go Gateway (port 8090) --> Reverse proxy
    |
    v
Express.js API (port 3000)
    |
    |--> Validate request (Zod schema)
    |--> Check AIDD policy profile
    |--> Check tenant quota
    |--> Insert record into YugabyteDB
    |--> Create/patch ServiceInstance CRD
    |
    v
K8s Operator watches CRD
    |
    |--> Creates StatefulSet/ReplicaSet
    |--> Creates PVCs
    |--> Creates Services
    |--> Creates Secrets
    |--> Updates CRD status
    |
    v
DBaaS API polls CRD status
    |
    |--> Updates service_instances table
    |--> Emits metering event
    |--> Fires webhook
```

### Rationale

1. **Kubernetes-native**: CRDs are first-class Kubernetes objects with RBAC, admission webhooks, validation, versioning, and conversion.
2. **Schema validation**: OpenAPI v3 schemas in CRDs enforce structural validation at the API server level before operators even see the request.
3. **Declarative lifecycle**: The ServiceInstance CRD status subresource tracks the full lifecycle (Provisioning, Running, Scaling, BackingUp, Restoring, Failed, Decommissioning, Decommissioned).
4. **Printer columns**: Custom printer columns make `kubectl get si` immediately useful for operators.
5. **Separation of API and cluster concerns**: The DBaaS API only needs Kubernetes API access to create/patch/watch CRDs; it never needs to manage low-level cluster resources.

### Consequences

**Positive**:
- All provisioning state is persisted in etcd with automatic replication.
- CRDs integrate with K8s RBAC, allowing fine-grained access control.
- `kubectl` becomes a natural debugging and operational tool.
- CRD status conditions provide structured error reporting.

**Negative**:
- CRD schema changes require careful versioning and migration.
- The API must handle eventual consistency between CRD state and YugabyteDB state.
- CRD object count at scale (thousands of instances) requires etcd sizing considerations.

---

## ADR-003: AIDD Strict Policy as Immutable Compiled Profile

**Status**: Accepted
**Date**: 2025-11-10
**Deciders**: AIDD Governance Board, Platform Engineering Team

### Context

The AIDD (AI-Driven Development) governance framework requires that all ERP platform services use only AIDD-approved technology choices. For DBaaS, this means:

- Only approved database engines can be provisioned for ERP internal workloads.
- PostgreSQL, MySQL, MariaDB, and Redis/Valkey are explicitly denied for ERP internal use (they are replaced by YugabyteDB, ScyllaDB, DragonflyDB respectively).
- Backup encryption and cross-region replication are mandatory.
- Certain configuration parameters (e.g., disabling TLS, disabling auth) are forbidden.

However, ERP-DBaaS also serves third-party customers who may need commercial engines. We needed a mechanism to enforce strict governance for ERP workloads while allowing flexibility for customers.

### Decision

The `ERP_AIDD_STRICT` profile is compiled directly into the application as an immutable TypeScript class (`ErpAiddStrictProfile`). It cannot be modified at runtime through configuration, environment variables, APIs, or CRDs.

Key properties of the immutable profile:

```typescript
// ALLOWED engines (ReadonlySet - cannot be modified at runtime)
ALLOWED_ENGINES = new Set([
  'yugabytedb', 'scylladb', 'dragonfly', 'mongodb',
  'couchdb', 'clickhouse', 'timescaledb', 'questdb'
]);

// DENIED engines (ReadonlySet - cannot be modified at runtime)
DENIED_ENGINES = new Set([
  'postgresql', 'mysql', 'mariadb', 'redis_valkey',
  'postgres', 'pg', 'redis', 'valkey'  // aliases
]);

// FORBIDDEN config keys (ReadonlySet)
FORBIDDEN_CONFIG_KEYS = new Set([
  'ssl_disabled', 'auth_disabled', 'allow_external_access',
  'disable_encryption', 'disable_audit_log'
]);

// Non-negotiable requirements
MIN_RETENTION_DAYS = 30;
BACKUP_ENCRYPTION = mandatory;
CROSS_REGION_BACKUP = mandatory;
TLS_ENABLED = mandatory;
```

The `COMMERCIAL_FLEXIBLE` profile is provided as a separate class for third-party customer workloads. It extends the allowed engine list and relaxes some constraints while maintaining baseline security (encryption, audit logging).

### Rationale

1. **Immutability prevents bypass**: By compiling the strict profile into the application binary, it cannot be modified by misconfigured environment variables, CRD changes, or API calls. The only way to change it is to modify the source code, which requires code review and CI/CD pipeline approval.
2. **Explicit deny list with aliases**: The DENIED_ENGINES set includes common aliases (`postgres`, `pg`, `redis`, `valkey`) to prevent bypass through alternative naming.
3. **Defense in depth**: Even if authentication or authorization is compromised, the compiled profile still prevents provisioning of non-approved engines.
4. **Dual-profile architecture**: Separating ERP internal (strict) from customer (flexible) workloads allows the platform to serve both use cases without compromising governance.
5. **Audit trail**: All policy violations are logged with detailed reasons, creating an audit trail for governance compliance.

### Consequences

**Positive**:
- AIDD governance is enforced at the application level, not just at the policy level.
- No risk of accidental engine approval through configuration drift.
- Clear separation between ERP internal and customer workloads.
- Policy violation messages include full context (allowed engines, required profile).

**Negative**:
- Adding a new engine to the AIDD-approved list requires a code change and redeployment.
- The strict profile cannot be temporarily relaxed for emergency situations without a code change.
- Two profile implementations must be maintained and tested independently.

### Alternatives Considered

1. **Configuration-based profiles (YAML/JSON)**: Rejected because configuration files can be modified at runtime, creating a governance bypass risk.
2. **OPA/Gatekeeper policies**: Considered but rejected as the sole mechanism because Kubernetes admission webhooks can be bypassed if the API creates resources directly. OPA may be added as an additional defense layer in the future.
3. **Single flexible profile with feature flags**: Rejected because it creates a single point of failure for governance enforcement.

---

## ADR-004: gRPC Plugin Interface for Engine Extensibility

**Status**: Accepted
**Date**: 2025-12-20
**Deciders**: Platform Engineering Team

### Context

While the 8 AIDD-approved engines cover most use cases, third-party customers (under COMMERCIAL_FLEXIBLE profile) may need database engines not built into the platform. We needed an extensibility mechanism that:

- Allows third parties to add new engine support without modifying the core DBaaS codebase.
- Provides structured lifecycle hooks (provision, scale, backup, restore, rotate, deprovision).
- Enforces security boundaries between the platform and plugin code.
- Supports health checking and capability discovery.

### Decision

We adopt a gRPC-based plugin interface. Every plugin must implement the `dbaas.plugin.v1.LifecycleService` gRPC service:

```protobuf
service LifecycleService {
  rpc GetCapabilities(GetCapabilitiesRequest) returns (GetCapabilitiesResponse);
  rpc OnProvision(ProvisionEvent) returns (ProvisionResponse);
  rpc OnScale(ScaleEvent) returns (ScaleResponse);
  rpc OnBackup(BackupEvent) returns (BackupResponse);
  rpc OnRestore(RestoreEvent) returns (RestoreResponse);
  rpc OnRotateCredentials(RotateEvent) returns (RotateResponse);
  rpc OnDeprovision(DeprovisionEvent) returns (DeprovisionResponse);
  rpc HealthCheck(HealthCheckRequest) returns (HealthCheckResponse);
}
```

Plugin registration flow:
1. Plugin developer deploys their gRPC service to the cluster.
2. Developer calls `POST /v1/dbaas/plugins` with plugin metadata and gRPC endpoint.
3. DBaaS validates the plugin against the tenant's policy profile.
4. DBaaS performs a connectivity check to the gRPC endpoint.
5. Async validation: gRPC health check, capability verification, security scan.
6. Plugin transitions from `pending_validation` to `validated` to `active`.

### Rationale

1. **Language-agnostic**: gRPC supports Go, Rust, Java, Python, and more. Plugin developers are not constrained to Node.js/TypeScript.
2. **Strongly typed contracts**: Protobuf schemas enforce a strict API contract between the platform and plugins.
3. **Performance**: gRPC uses HTTP/2 with binary serialization, providing lower latency than REST for the frequent lifecycle callbacks.
4. **Streaming support**: gRPC supports server streaming, which is useful for long-running operations like backups.
5. **Deadlines and cancellation**: gRPC deadlines (30-second timeout configured) prevent hung plugin calls from blocking the platform.
6. **Security context**: Plugin registrations include a `securityContext` field specifying the Kubernetes service account and namespace, enabling RBAC enforcement.

### Consequences

**Positive**:
- New engines can be added without modifying or redeploying the core DBaaS API.
- Plugins run in separate pods/namespaces with their own security boundaries.
- gRPC health checks enable automatic detection of unhealthy plugins.
- Capability discovery allows the platform to advertise plugin-provided engines.

**Negative**:
- Plugin developers must learn gRPC and implement the protobuf interface.
- gRPC debugging is less intuitive than REST (requires grpcurl or similar tools).
- Network policies must be configured to allow dbaas-api to reach plugin gRPC endpoints.
- Plugin validation adds latency to the registration flow (mitigated by async validation).

### Alternatives Considered

1. **REST webhook interface**: Rejected because REST lacks strong typing, streaming, and built-in deadline support.
2. **Shared library/SDK approach**: Rejected because it would force plugins to use the same language and runtime as the platform.
3. **Container sidecar pattern**: Rejected because it would couple plugin lifecycle to instance pod lifecycle.

---

## ADR-005: DragonflyDB for Rate Limiting and Caching

**Status**: Accepted
**Date**: 2025-10-25
**Deciders**: AIDD Governance Board, Platform Engineering Team

### Context

The DBaaS API requires in-memory caching and rate limiting capabilities. Redis is the industry-standard choice for these use cases, but Redis is NOT on the AIDD-approved technology list for ERP internal use (it is available only under the COMMERCIAL_FLEXIBLE profile for customer workloads).

We needed an AIDD-approved in-memory datastore that:
- Is Redis-protocol compatible (to use existing client libraries like ioredis).
- Provides sub-millisecond performance for rate limiting counters.
- Is memory-efficient and multi-threaded.
- Is open-source with an acceptable license.

### Decision

We use DragonflyDB as the in-memory datastore for rate limiting and caching in the DBaaS platform. DragonflyDB speaks the Redis wire protocol, so we use the `ioredis` Node.js client library to communicate with it.

Rate limiting implementation:
- Sliding window counter using `INCR` + `EXPIRE` commands.
- Key format: `ratelimit:{tenantId}:{windowKey}`.
- Window size: 60 seconds.
- Tier-based limits: A=1000/min, B=500/min, C=100/min.
- Rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`.

Graceful degradation:
- If DragonflyDB is unavailable, the rate limiter is bypassed and requests are allowed through.
- Connection errors are logged but do not block API operations.
- Lazy connection with configurable retry strategy (max 5 retries, exponential backoff up to 2 seconds).

### Rationale

1. **AIDD compliance**: DragonflyDB is on the AIDD-approved technology list; Redis is not.
2. **Drop-in compatibility**: DragonflyDB supports the Redis command set, so the ioredis client works without modification.
3. **Performance**: DragonflyDB is multi-threaded (unlike Redis's single-threaded model), providing better utilization of modern multi-core hardware.
4. **Memory efficiency**: DragonflyDB's memory management uses significantly less memory than Redis for equivalent datasets.
5. **Operational simplicity**: DragonflyDB is deployed as a single container with no cluster mode complexity for the rate limiting use case.

### Consequences

**Positive**:
- Full AIDD compliance without compromising functionality.
- Existing Redis tooling (redis-cli, ioredis) works seamlessly.
- Better multi-core performance for rate limiting at scale.
- Simpler deployment than Redis Cluster for the caching use case.

**Negative**:
- DragonflyDB has a smaller community and ecosystem than Redis.
- Some advanced Redis features (e.g., Redis Streams with consumer groups) may have subtle behavioral differences.
- Operations teams need to learn DragonflyDB-specific configuration and monitoring.

---

## ADR-006: Tier-Based Quota System

**Status**: Accepted
**Date**: 2025-11-25
**Deciders**: Product Management, Platform Engineering Team

### Context

As a multi-tenant platform, ERP-DBaaS must prevent any single tenant from consuming disproportionate resources. We needed a quota system that:

- Limits the number of instances per tenant.
- Limits aggregate CPU, memory, and storage per tenant.
- Provides different limits for different tenant classes.
- Integrates with the rate limiting system.
- Can be adjusted without code changes.

### Decision

We implement a three-tier quota system stored in the `tenant_quotas` table in YugabyteDB:

| Tier | Max Instances | Max CPU | Max Memory | Max Storage | Rate Limit |
|---|---|---|---|---|---|
| A (Enterprise) | 50 | 128 cores | 512Gi | 5Ti | 1000 req/min |
| B (Professional) | 20 | 64 cores | 256Gi | 2Ti | 500 req/min |
| C (Starter) | 5 | 16 cores | 64Gi | 500Gi | 100 req/min |

Quota enforcement points:
1. **Provisioning**: Check `current_instances < max_instances` before allowing provisioning (HTTP 429 if exceeded).
2. **Scaling**: Check aggregate resource consumption against tier limits before allowing scale-up.
3. **Rate limiting**: DragonflyDB rate limiter applies tier-specific request rate limits.

Quota tracking:
- `current_instances` is incremented on provisioning and decremented on decommission.
- `current_cpu`, `current_memory`, `current_storage` are updated on provision and scale operations.
- Auto-creation: New tenants are automatically assigned Tier C with default quotas on first API request.

### Rationale

1. **Simplicity**: Three tiers are easy to understand, communicate, and enforce. More granular quotas can be added later.
2. **Database-backed**: Quotas are stored in YugabyteDB, making them durable, queryable, and modifiable without code changes.
3. **Atomic enforcement**: Quota checks and updates use SQL transactions to prevent race conditions.
4. **Auto-provisioning**: New tenants get a default quota automatically, eliminating the need for manual onboarding.
5. **Integrated rate limiting**: Tying API rate limits to the tier system provides consistent resource governance across both compute and API dimensions.

### Consequences

**Positive**:
- Clear resource boundaries prevent noisy-neighbor problems.
- Tier upgrades/downgrades are a simple database update.
- Rate limiting and resource quotas are unified under the same tier model.
- Quota data is available for billing and metering.

**Negative**:
- Three tiers may not provide enough granularity for all customer needs.
- Aggregate resource tracking (CPU, memory, storage) depends on accurate accounting during all lifecycle operations.
- Quota enforcement adds latency to provisioning requests (mitigated by in-memory caching).

---

## ADR-007: Refine.dev for Frontend Framework

**Status**: Accepted
**Date**: 2025-10-18
**Deciders**: Frontend Engineering Team

### Context

ERP-DBaaS requires a web-based management console for tenant self-service. The frontend must:

- Integrate with the existing ERP platform UI patterns (all ERP modules use Refine.dev + Ant Design).
- Provide CRUD interfaces for instances, backups, plugins, and quotas.
- Support real-time status updates for long-running operations (provisioning, scaling, backup).
- Authenticate via Authentik JWT tokens.
- Be extensible for future features (cost dashboards, capacity planning).

### Decision

We use Refine.dev with Ant Design as the UI component library, graphql-request for GraphQL data fetching via Hasura, and graphql-ws for real-time subscriptions. This matches the established pattern across all other ERP modules.

Frontend stack:
- **Framework**: React + Vite + Refine.dev
- **UI Components**: Ant Design
- **Data Provider**: graphql-request (via Hasura GraphQL)
- **Realtime**: graphql-ws (Hasura subscriptions)
- **Authentication**: Authentik OIDC/JWT provider

### Rationale

1. **Consistency**: All 14+ ERP modules use the same frontend stack. Developers can move between modules without learning new frameworks.
2. **Refine.dev features**: Built-in CRUD scaffolding, data provider abstraction, access control, real-time support, and routing reduce boilerplate.
3. **Hasura integration**: Refine.dev's GraphQL data provider works natively with Hasura, providing automatic pagination, filtering, sorting, and subscription support.
4. **Ant Design**: Enterprise-grade UI components with comprehensive TypeScript types, accessibility support, and theming.
5. **Developer velocity**: Refine.dev's resource-based architecture means a new CRUD interface for a new entity (e.g., plugins) can be scaffolded in minutes.

### Consequences

**Positive**:
- Rapid frontend development with minimal boilerplate.
- Consistent look and feel across all ERP modules.
- Real-time updates for instance status changes via Hasura subscriptions.
- Shared component library and design patterns reduce frontend maintenance.

**Negative**:
- Refine.dev opinionated architecture may limit flexibility for highly custom UI requirements.
- Dependency on Hasura for GraphQL layer adds an infrastructure dependency.
- Ant Design's bundle size is significant (mitigated by tree-shaking with Vite).

---

## ADR-008: Express.js for API Layer Flexibility

**Status**: Accepted
**Date**: 2025-10-20
**Deciders**: Platform Engineering Team

### Context

The DBaaS API layer needs to:

- Handle complex orchestration logic (provisioning, scaling, backup) that involves multiple async operations.
- Integrate with YugabyteDB (via pg driver), DragonflyDB (via ioredis), Kubernetes API (via @kubernetes/client-node), gRPC plugins (via @grpc/grpc-js), Apache Pulsar (via pulsar-client), and HashiCorp Vault.
- Implement middleware-based request processing (auth, rate limiting, tenant context).
- Support Zod-based request validation.
- Be familiar to the development team.

The Go gateway handles reverse proxying, CORS, and basic health checks. The Node.js API handles all business logic.

### Decision

We use Express.js (v4) with TypeScript for the API layer, running on Node.js 20 LTS.

Architecture:
- **Go gateway** (port 8090): Lightweight reverse proxy with CORS, security headers, health check, and capabilities endpoint.
- **Express.js API** (port 3000): Full business logic with middleware stack (rate limiter -> auth -> tenant context), route handlers, controllers, and policy engine.

Key dependencies:
- `express` v4.18: HTTP framework
- `pg` v8.11: YugabyteDB driver (YSQL is PostgreSQL wire-compatible)
- `ioredis` v5.3: DragonflyDB driver (Redis wire-compatible)
- `@kubernetes/client-node` v0.20: Kubernetes API client
- `@grpc/grpc-js` v1.9: gRPC client for plugins
- `pulsar-client` v1.10: Apache Pulsar client for metering
- `jsonwebtoken` v9.0: JWT validation
- `zod` v3.22: Request validation
- `pino` v8.16: Structured logging

### Rationale

1. **TypeScript type safety**: The entire DBaaS type system (engines, plans, profiles, statuses) is defined in TypeScript with enums, interfaces, and Zod schemas. This catches errors at compile time rather than runtime.
2. **Async/await for orchestration**: Node.js's event-driven model is well-suited for the DBaaS API's workload: many concurrent requests, each involving multiple async I/O operations (database queries, K8s API calls, gRPC calls).
3. **Middleware composability**: Express's middleware pattern cleanly separates cross-cutting concerns (rate limiting, auth, tenant context) from business logic.
4. **Ecosystem breadth**: Node.js has mature client libraries for all required integrations (pg, ioredis, k8s client, gRPC, Pulsar).
5. **Team familiarity**: The development team has deep experience with Express.js and TypeScript.
6. **Go gateway for performance**: The Go gateway handles the high-throughput, low-latency reverse proxying and security header injection, while Express.js handles the complex business logic. This hybrid approach plays to each language's strengths.

### Consequences

**Positive**:
- Rapid development with TypeScript type safety and Express middleware composability.
- Zod validation provides runtime type checking with excellent error messages.
- Pino structured logging integrates with centralized log aggregation.
- The hybrid Go+Node.js architecture optimizes for both performance and developer productivity.

**Negative**:
- Two languages (Go and TypeScript) increase the skill requirements for the team.
- Node.js single-threaded model requires care with CPU-intensive operations (mitigated by offloading to async operations and K8s operators).
- Express.js v4 callback-style error handling requires wrapping all route handlers in try/catch.
- The gateway-to-API reverse proxy adds one network hop of latency.

### Alternatives Considered

1. **Go for the entire API**: Rejected because the team's primary expertise is in TypeScript, and the complex orchestration logic with multiple async integrations is more ergonomic in Node.js.
2. **Fastify instead of Express**: Considered but rejected because Express has a larger ecosystem and the team has more experience with it. Fastify could be adopted in a future iteration if performance becomes a bottleneck.
3. **NestJS instead of Express**: Rejected because the additional abstraction layers and decorators were deemed unnecessary for the DBaaS API's scope. Express provides sufficient structure with the existing controller/service pattern.

---

## ADR Index

| ADR | Title | Status | Date |
|---|---|---|---|
| ADR-001 | K8s Operator Pattern for Lifecycle | Accepted | 2025-10-15 |
| ADR-002 | CRD-Driven Provisioning | Accepted | 2025-10-22 |
| ADR-003 | AIDD Strict Policy as Immutable | Accepted | 2025-11-10 |
| ADR-004 | gRPC Plugin Interface | Accepted | 2025-12-20 |
| ADR-005 | DragonflyDB for Rate Limiting | Accepted | 2025-10-25 |
| ADR-006 | Tier-Based Quota System | Accepted | 2025-11-25 |
| ADR-007 | Refine.dev Frontend | Accepted | 2025-10-18 |
| ADR-008 | Express.js for API Flexibility | Accepted | 2025-10-20 |
