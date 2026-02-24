# Architecture Decision Records (ADRs)

---

## ADR-001: Go as Primary Backend Language

**Status**: Accepted
**Date**: 2026-02-23
**Context**: The HCM platform requires high concurrency for payroll processing, low memory footprint for cost-effective deployment, and strong typing for financial calculations.
**Decision**: Use Go 1.24+ as the primary backend language for all microservices.
**Rationale**: Go provides excellent concurrency via goroutines, compiles to single static binaries for minimal container images (< 20MB), has a strong standard library for HTTP servers, and offers the performance needed for batch payroll processing (10K+ employees in < 5 minutes). The type system prevents common bugs in financial calculations.
**Consequences**: Team must be proficient in Go. No dynamic language flexibility. Generics (available since Go 1.18) mitigate some boilerplate.

---

## ADR-002: PostgreSQL 16 as Primary Database

**Status**: Accepted
**Date**: 2026-02-23
**Context**: HCM data is highly relational (employees belong to departments, departments have hierarchies, payroll references employees). The system needs ACID compliance for financial data, JSONB for flexible custom fields, and strong ecosystem support.
**Decision**: Use PostgreSQL 16 as the primary relational database with schema-per-domain isolation.
**Rationale**: PostgreSQL provides ACID compliance critical for payroll, JSONB columns for custom employee fields, recursive CTEs for org chart queries, window functions for analytics, row-level security for multi-tenant data isolation, and excellent performance with pgx driver. TimescaleDB extension enables time-series data for attendance.
**Consequences**: Single database technology for most services. Need connection pooling (pgx with 100 max connections). Must manage 30+ schemas and 47+ migrations.

---

## ADR-003: NATS JetStream for Event-Driven Architecture

**Status**: Accepted
**Date**: 2026-02-23
**Context**: Services need to communicate asynchronously for loose coupling. Employee creation must trigger payroll setup, benefits enrollment, and notification without synchronous API calls.
**Decision**: Use NATS JetStream as the event backbone with CloudEvents envelope format.
**Rationale**: NATS JetStream provides at-least-once delivery guarantees, consumer group support for horizontal scaling, built-in persistence (stream storage), low latency (< 1ms), and simple operational overhead compared to Kafka. CloudEvents provides a vendor-neutral event format.
**Consequences**: All services must implement NATS publishing/subscribing. Need dead letter queue handling for failed events. Event schema evolution must be managed carefully.

---

## ADR-004: Multi-Tenant Architecture via Tenant ID Middleware

**Status**: Accepted
**Date**: 2026-02-23
**Context**: The platform must serve multiple organizations from a single deployment. Options considered: separate databases per tenant, separate schemas per tenant, shared schema with tenant_id column.
**Decision**: Use shared database with `tenant_id` column on all tables, enforced via middleware and repository layer.
**Rationale**: Shared database minimizes operational complexity (one migration path, one backup strategy). The `tenant_id` column with composite indexes (tenant_id, entity_id) provides adequate isolation at the application layer. The TenantContext middleware extracts tenant from JWT claims and injects into Go context. Super-admin cross-tenant access is governed by AIDD guardrails.
**Consequences**: Every query must include tenant_id WHERE clause. Risk of data leakage if middleware is bypassed (mitigated by repository-level enforcement). No per-tenant database tuning.

---

## ADR-005: Decimal Arithmetic for Financial Calculations

**Status**: Accepted
**Date**: 2026-02-23
**Context**: Payroll calculations involve currency amounts where floating-point rounding errors are unacceptable. A 0.01 Naira rounding error multiplied by 10,000 employees over 12 months could cause significant discrepancies.
**Decision**: Use `shopspring/decimal` library for all financial calculations and store monetary amounts as `int64` (kobo/cents) in the database.
**Rationale**: `shopspring/decimal` provides arbitrary-precision decimal arithmetic, matching how humans think about money. Storing as int64 (smallest currency unit) eliminates database-level floating-point issues. The PAYE tax band calculations require precise band boundary comparisons that floating-point cannot guarantee.
**Consequences**: All monetary values must be converted to/from int64 at the API boundary. Developers must consistently use decimal operations. Slightly more verbose code than using float64.

---

## ADR-006: RS256 JWT with MFA for Authentication

**Status**: Accepted
**Date**: 2026-02-23
**Context**: The system handles sensitive PII and financial data. Authentication must be robust, support SSO integration, and enable stateless API authorization.
**Decision**: Use RS256 (RSA-SHA256) JWT tokens with TOTP-based MFA.
**Rationale**: RS256 allows the public key to be distributed to all services for token verification without sharing the private signing key. 15-minute access token expiry limits the blast radius of token theft. TOTP MFA (RFC 6238) provides a second factor without depending on SMS. Account lockout (5 attempts, 30min) prevents brute force. Password policy (8 chars, special required, 5-password history) meets SOC 2 requirements.
**Consequences**: RSA key pair must be securely managed (Vault). Token refresh flow adds complexity. MFA enrollment adds onboarding friction.

---

## ADR-007: AES-256-GCM for Field-Level Encryption

**Status**: Accepted
**Date**: 2026-02-23
**Context**: Employee PII (bank account numbers, tax IDs, national IDs, RSA PINs) must be encrypted at rest per GDPR, NDPR, and SOC 2 requirements.
**Decision**: Use AES-256-GCM encryption for sensitive fields with keys managed by HashiCorp Vault.
**Rationale**: AES-256-GCM provides authenticated encryption (confidentiality + integrity). HashiCorp Vault Transit engine handles key generation, rotation, and access control. Field-level encryption means only specific columns are encrypted, not the entire database, allowing indexing on non-sensitive fields. Key rotation is automated with a configurable rotation period.
**Consequences**: Encrypted fields cannot be searched or indexed (must use separate search indexes). Key management adds infrastructure dependency. Encryption/decryption adds ~1ms per field operation.

---

## ADR-008: Chi Router for HTTP API Framework

**Status**: Accepted
**Date**: 2026-02-23
**Context**: Need an HTTP router that is idiomatic Go, supports middleware chaining, and has minimal dependencies.
**Decision**: Use `go-chi/chi/v5` as the primary HTTP router for all services.
**Rationale**: Chi is lightweight, idiomatic (uses standard `http.Handler` interface), supports middleware chaining, has built-in support for URL parameters, and is actively maintained. It does not impose an opinionated framework structure, allowing domain-driven design. Existing Gorilla Mux code (from legacy repos) coexists via adapter patterns.
**Consequences**: No built-in request binding or validation (handled by custom middleware). No automatic OpenAPI generation (documented manually).

---

## ADR-009: Next.js 14 with App Router for Frontend

**Status**: Accepted
**Date**: 2026-02-23
**Context**: The frontend needs server-side rendering for SEO (careers page), client-side interactivity for dashboards, and a modern component library.
**Decision**: Use Next.js 14 with App Router, Radix UI primitives, Tailwind CSS, React Query, and Zustand.
**Rationale**: Next.js 14 App Router provides server components for initial load performance, streaming SSR, and nested layouts matching the HR module hierarchy. Radix UI provides accessible, unstyled primitives. Tailwind CSS enables rapid UI development. React Query handles server state with caching and optimistic updates. Zustand provides minimal client state management. Zod schema validation is shared between forms and API contracts.
**Consequences**: Node.js 20 LTS required for development. React 18 concurrent features must be understood. Hydration mismatches can occur with server/client components.

---

## ADR-010: AIDD Guardrails for AI-Driven Operations

**Status**: Accepted
**Date**: 2026-02-23
**Context**: The platform incorporates AI-driven features (resume parsing, chatbot, automated workflows). These operations must be governed to prevent unintended data mutations, cross-tenant access, or irreversible actions.
**Decision**: Implement a three-tier AIDD guardrails framework (autonomous, supervised, prohibited) defined in `erp/aidd.guardrails.yaml`.
**Rationale**: Autonomous actions (read queries, notifications) can proceed without human approval. Supervised actions (data mutations, bulk operations) require human-in-the-loop confirmation. Prohibited actions (cross-tenant access, irreversible delete without backup, privilege escalation) are blocked entirely. All AI decisions are logged for audit. A 24-hour rollback window provides safety net.
**Consequences**: AI features have higher latency due to approval workflows. Developers must classify every AI-driven action into the correct tier. Audit logging increases storage requirements.

---

## ADR-011: Haversine Formula for Geofence Validation

**Status**: Accepted
**Date**: 2026-02-23
**Context**: Attendance clock-in requires validating that an employee is physically within a defined radius of their assigned office.
**Decision**: Use the Haversine formula for calculating great-circle distance between GPS coordinates, with configurable radius per office location.
**Rationale**: The Haversine formula is the standard approach for calculating distances between two points on a sphere given their latitudes and longitudes. It is computationally efficient (no external API calls), accurate to within ~0.3% for distances under 100km, and well-understood. Anti-spoofing measures (GPS accuracy threshold of 50m, teleport detection at 100km/hr, multi-device detection) provide additional validation.
**Consequences**: Does not account for altitude differences. GPS accuracy varies by device and environment. Indoor positioning requires alternative approaches (QR codes, Wi-Fi).

---

## ADR-012: Testcontainers for Integration Testing

**Status**: Accepted
**Date**: 2026-02-23
**Context**: Integration tests need real database and cache instances to validate SQL queries, migrations, and caching behavior.
**Decision**: Use `testcontainers-go` for PostgreSQL and Redis containers in integration tests.
**Rationale**: Testcontainers provides ephemeral, isolated database instances for each test suite. This eliminates shared test database state, ensures migration correctness, and runs in CI without external dependencies. The Go module provides first-class PostgreSQL and Redis module support.
**Consequences**: Docker must be available in CI environment. Integration tests take 10-30 seconds to start containers. Test isolation is guaranteed but requires careful teardown.
