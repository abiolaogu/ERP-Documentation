# ERP-Platform Architecture Decision Records (ADRs)

> **Document ID:** ERP-PLAT-ADR-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Active
> **Related Documents:** [04-Software-Architecture.md](./04-Software-Architecture.md), [03-Enterprise-Architecture.md](./03-Enterprise-Architecture.md)

---

## ADR-001: Go as Primary Service Language

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, CTO

### Context
ERP-Platform needs a language for building nine microservices that will serve as the control plane for a 20-module ERP suite. Requirements include high performance, low memory footprint, fast compilation, excellent concurrency, and minimal dependency overhead.

### Decision
Use Go 1.22 as the primary language for all platform microservices, leveraging the standard library's `net/http` package for HTTP servers.

### Consequences
- **Positive**: Zero external dependencies in core services (only `go.mod` with module declaration). Fast compilation (< 5s per service). Small binaries (8-15MB). Excellent goroutine-based concurrency. Strong typing catches errors at compile time.
- **Positive**: Go's standard library HTTP server is production-grade and battle-tested.
- **Negative**: Go's type system lacks generics ergonomics for complex data transformations (though Go 1.22 generics help).
- **Negative**: No framework-level middleware (authentication, logging) -- must implement or adopt thin wrappers.
- **Risks**: Team must be proficient in Go; limited talent pool compared to Java/JavaScript.

---

## ADR-002: PostgreSQL for Relational Data

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, Data Engineering Lead

### Context
The platform requires a relational database for tenants, subscriptions, entitlements, audit logs, and module registry. Multi-tenant isolation is critical. The database must support ACID transactions, complex queries, and row-level security.

### Decision
Use PostgreSQL 16 as the primary relational database, with Redis 7 for caching and NATS 2.10 JetStream for event streaming.

### Consequences
- **Positive**: Row-Level Security (RLS) enables tenant isolation at the database layer. JSONB columns allow flexible metadata storage. Mature ecosystem with excellent tooling (pgAdmin, pg_dump, WAL archiving). Strong consistency guarantees.
- **Positive**: Streaming replication for read replicas and DR.
- **Negative**: Vertical scaling has limits; horizontal sharding requires Citus or application-level routing.
- **Negative**: Connection management requires PgBouncer at scale.

---

## ADR-003: Multi-Tenant Isolation via Row-Level Security

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, CISO

### Context
As a multi-tenant control plane, ERP-Platform must prevent cross-tenant data access. A single leaked query could expose one tenant's data to another.

### Decision
Implement multi-tenant isolation via three layers: (1) `X-Tenant-ID` header validation at the API layer, (2) PostgreSQL Row-Level Security policies on all tenant-scoped tables, and (3) `SET app.current_tenant = '{tenant_id}'` on every database connection.

### Consequences
- **Positive**: Defense in depth -- even if application-level checks fail, RLS prevents cross-tenant reads.
- **Positive**: Developers cannot accidentally write queries that leak tenant data.
- **Negative**: Slight performance overhead from RLS policy evaluation on every query.
- **Negative**: Database migrations must include RLS policies for every new table.
- **Negative**: Global admin queries require bypassing RLS (superuser connection).

---

## ADR-004: Event-Driven Architecture with NATS JetStream

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council

### Context
Platform services need to communicate asynchronously for audit logging, notification delivery, and entitlement seeding. The event system must support at-least-once delivery, be operationally simple, and scale to 50K+ events per second.

### Decision
Use NATS 2.10 with JetStream for all inter-service event communication, following the CloudEvents specification for event envelopes. Topic naming convention: `erp.platform.<service>.<action>`.

### Consequences
- **Positive**: NATS is lightweight (single binary), sub-millisecond latency, and operationally simple.
- **Positive**: JetStream provides persistence, replay, and consumer groups.
- **Positive**: CloudEvents envelope ensures interoperability with external systems.
- **Negative**: NATS has a smaller ecosystem than Kafka; fewer third-party integrations.
- **Negative**: At-least-once delivery means consumers must be idempotent.

---

## ADR-005: AIDD Guardrails Framework

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, AI Ethics Board, CTO

### Context
The ERP suite integrates AI agents (ERP-AI, ERP-Assistant) that can perform automated operations. Without guardrails, AI-driven actions could cause data loss, financial errors, or compliance violations.

### Decision
Implement AIDD (AI-Integrated Development Discipline) guardrails as a mandatory enforcement layer for all AI-driven operations. Key thresholds: minimum confidence 0.70, medium confidence 0.82, maximum blast radius 5,000 records, high-value threshold $100,000 USD, high-risk auto-execute disabled by default.

### Consequences
- **Positive**: Prevents runaway AI operations from causing large-scale damage.
- **Positive**: All AI decisions logged to immutable audit trail for accountability.
- **Positive**: Configurable thresholds allow adjustment as AI models improve.
- **Negative**: Human approval requirements slow down some automated workflows.
- **Negative**: Threshold tuning requires expertise and monitoring.

---

## ADR-006: Catalog-Driven Product Architecture

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, Product Management

### Context
Adding new modules to the ERP suite should not require code changes to the platform. The product catalog, bundles, and pricing must be managed as configuration, not code.

### Decision
Define the entire product catalog as a versioned JSON file (`catalog/products.json`). The subscription hub loads this catalog at startup and uses it for SKU validation, bundle resolution, and entitlement seeding. New modules are added by editing the JSON and redeploying.

### Consequences
- **Positive**: Adding a module requires only a catalog JSON update -- no code changes.
- **Positive**: Catalog versioning enables backward compatibility tracking.
- **Positive**: Bundle definitions are declarative and auditable.
- **Negative**: Catalog changes require service restart (until hot-reload is implemented).
- **Negative**: Schema evolution must maintain backward compatibility.

---

## ADR-007: Zero External Dependencies in Core Services

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, Security Lead

### Context
Supply chain attacks through transitive dependencies are a growing threat. The subscription hub is a security-critical service that handles entitlements and tenant access.

### Decision
Core platform services (especially subscription-hub) must use only Go standard library packages. The `go.mod` file contains only the module declaration and Go version. No third-party HTTP frameworks, ORMs, or utility libraries.

### Consequences
- **Positive**: Zero transitive dependency vulnerabilities.
- **Positive**: No dependency update churn or breaking changes from upstream libraries.
- **Positive**: Extremely small binary size and fast compilation.
- **Negative**: More boilerplate code for common patterns (middleware, validation).
- **Negative**: Cannot leverage battle-tested libraries for complex functionality.

---

## ADR-008: Health-Check Driven Module Discovery

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council

### Context
The platform needs to know which ERP modules are operational at any given time. Static configuration becomes stale; we need a dynamic discovery mechanism.

### Decision
All ERP modules expose a standard `/healthz` endpoint. The module registry polls these endpoints every 30 seconds (timeout: 5s, retries: 5). Module status is determined by live health checks, not static configuration.

### Consequences
- **Positive**: Real-time visibility into module availability.
- **Positive**: Self-healing -- modules automatically re-register when they recover.
- **Positive**: Consistent health check contract across all 20 modules.
- **Negative**: Polling creates network overhead (manageable at 20 modules, 30s intervals).
- **Negative**: False negatives possible during network partitions.

---

## ADR-009: Repository Consolidation Strategy

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, Engineering Management

### Context
The platform control plane was split across five separate repositories: ERP-Platform, ERP-BAC-Business-Activation, ERP-BAC-BOS-AI, BOS/BAC-BOS-AI, and ERP-Web-Hosting. This fragmentation made it difficult to maintain consistent APIs, deploy coordinated changes, and onboard new engineers.

### Decision
Consolidate all five repositories into a single ERP-Platform repository. Preserve source snapshots in `merge/source-snapshots/` for lineage tracking. Import relevant code into `imports/` directory for reference. Maintain a merge manifest (`merge/MERGE_MANIFEST.yaml`) documenting the consolidation.

### Consequences
- **Positive**: Single repository for all control plane code simplifies development and deployment.
- **Positive**: Consistent CI/CD pipeline across all platform services.
- **Positive**: Merge manifest provides clear audit trail of consolidation decisions.
- **Negative**: Larger repository size.
- **Negative**: Legacy code in imports directory requires eventual cleanup.

---

## ADR-010: CloudEvents for Audit Trail

**Status:** Accepted
**Date:** 2026-02-23
**Deciders:** Architecture Council, Compliance Officer

### Context
The audit service must capture all state-changing operations across all platform services. The event format must be standardized, machine-parseable, and compatible with external compliance tools.

### Decision
Adopt the CloudEvents v1.0 specification for all audit events. Every state-changing operation emits an event with the standard CloudEvents envelope (specversion, id, source, type, time) plus custom extensions (tenantid, correlationid). Events are published to NATS JetStream and consumed by the audit service for persistence.

### Consequences
- **Positive**: Industry-standard event format with broad tooling support.
- **Positive**: Correlation ID enables distributed trace reconstruction.
- **Positive**: 45+ event topics provide granular audit coverage.
- **Negative**: Envelope overhead adds ~200 bytes per event.
- **Negative**: All services must correctly implement event publishing.

---

## ADR-011: In-Memory Store as Development Bootstrap

**Status:** Accepted (Temporary)
**Date:** 2026-02-23
**Deciders:** Engineering Lead
**Superseded By:** ADR-012 (planned) -- PostgreSQL Persistence

### Context
For initial development and validation of the subscription hub API contract, we needed a fast way to store and retrieve subscription records without database infrastructure dependency.

### Decision
Implement an in-memory `Store` struct with `sync.RWMutex` for concurrent access. Map `tenant_id` to `SubscriptionRecord`, and index products by SKU and bundle SKU.

### Consequences
- **Positive**: Zero infrastructure dependency for development and testing.
- **Positive**: Extremely fast reads and writes (in-process, no network I/O).
- **Positive**: Simple concurrency model with RWMutex.
- **Negative**: Data lost on service restart.
- **Negative**: Not suitable for production (single-node, no persistence, no replication).
- **Migration Path**: Replace with PostgreSQL-backed repository implementing the same interface.

---

*For software architecture, see [04-Software-Architecture.md](./04-Software-Architecture.md). For enterprise architecture, see [03-Enterprise-Architecture.md](./03-Enterprise-Architecture.md).*
