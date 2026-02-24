# ERP-Platform Changelog

> **Document ID:** ERP-PLAT-CL-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Format:** Conventional Commits + Semantic Versioning
> **Related Documents:** [02-Release-Notes.md](./02-Release-Notes.md)

---

All notable changes to ERP-Platform are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-02-23

### Added

- **feat**: Product catalog API with 20 standalone modules and 28+ bundle/standalone SKUs (`services/subscription-hub/main.go`)
- **feat**: Subscription service supporting single, bundle, and suite plan types with automatic bundle SKU resolution
- **feat**: Entitlement query endpoint (`GET /v1/entitlements/{tenant_id}`) returning resolved module SKU lists
- **feat**: Tenant provisioner service with full CRUD lifecycle and CloudEvents emission (`services/tenant-provisioner/main.go`)
- **feat**: Entitlement engine service for runtime entitlement evaluation (`services/entitlement-engine/main.go`)
- **feat**: Module registry service for module auto-discovery via health checks (`services/module-registry/main.go`)
- **feat**: Marketplace service for third-party module listing and installation lifecycle (`services/marketplace/main.go`)
- **feat**: Audit service for platform-wide immutable audit logging with 45+ event topics (`services/audit-service/main.go`)
- **feat**: Notification hub service for multi-channel notification dispatch (`services/notification-hub/main.go`)
- **feat**: Web hosting service for domain, SSL, and CDN management (`services/web-hosting/main.go`)
- **feat**: Activation wizard service for guided tenant onboarding (`services/activation-wizard/main.go`)
- **feat**: Product catalog JSON defining 8 curated bundles: Starter, Professional, Enterprise, Healthcare Suite, Education Suite, Faith Suite, Telecom Suite, Developer Suite (`catalog/products.json`)
- **feat**: 20 standalone bundle SKUs for individual module subscription wrapping
- **feat**: Entitlement templates for all bundle plans (`catalog/entitlement-templates.json`)
- **feat**: Docker Compose orchestration with PostgreSQL 16, Redis 7, NATS 2.10 (`infra/docker-compose.platform.yml`)
- **feat**: Multi-stage Dockerfiles for all services using Go 1.22 Alpine and Alpine 3.20 runtime
- **feat**: Health check endpoints (`/healthz`) on all nine platform services
- **feat**: X-Tenant-ID header enforcement on all business endpoints
- **feat**: CloudEvents-compatible event publishing with topic convention `erp.platform.<service>.<action>`
- **feat**: AIDD guardrails baseline with confidence thresholds, blast-radius controls, and human approval gates
- **feat**: Deep import of BAC activation frontend (React + Vite + Tailwind) and backend (Python/LangGraph)
- **feat**: Register healthcare, school, church, assistant, and autonomous-coding modules in catalog and platform topology

### Changed

- **refactor**: Consolidated five legacy repositories (ERP-Platform, ERP-BAC-Business-Activation, ERP-BAC-BOS-AI, BOS/BAC-BOS-AI, ERP-Web-Hosting) into unified ERP-Platform
- **refactor**: Standardized all service health check responses with module and service identification

### Documentation

- **docs**: Architecture documentation with C4 container view and service inventory (`docs/ARCHITECTURE.md`)
- **docs**: API documentation with discovered endpoints and permission model (`docs/API.md`)
- **docs**: Event topic documentation with publishing convention and 45+ topics (`docs/EVENTS.md`)
- **docs**: AIDD guardrails documentation with thresholds and enforcement path (`docs/AIDD_GUARDRAILS.md`)
- **docs**: ADR-001 Language Choice and ADR-002 Database Selection (`docs/ADR/`)
- **docs**: Integration map documenting merge sources and runtime contracts (`docs/INTEGRATION_MAP.md`)
- **docs**: Merge manifest YAML tracking consolidation lineage (`merge/MERGE_MANIFEST.yaml`)
- **docs**: PR-safe branch normalization record

### Infrastructure

- **chore**: GitHub Actions workflow for documentation generation across all 20 modules (`.github/workflows/doc-gen.yml`)
- **chore**: Makefile with test, test-integration, and test-e2e targets
- **chore**: Gitignore for local build artifacts
- **chore**: Docker Compose health checks for all module containers (30s interval, 5s timeout, 5 retries)

### Known Issues

- In-memory subscription store; data lost on restart (PostgreSQL persistence planned for v1.1.0)
- No SQL migrations detected; database schema managed externally
- No input validation middleware beyond tenant_id and plan_type
- No rate limiting on API endpoints
- No authentication middleware in service code (JWT validation at gateway layer)

---

## [Unreleased]

### Planned for v1.1.0

- PostgreSQL persistence for subscription hub
- Redis caching for entitlement queries
- Input validation middleware
- Rate limiting middleware
- OpenTelemetry tracing integration

### Planned for v1.2.0

- AIDD guardrail dashboard UI
- Subscription analytics
- Self-service catalog extension
- Webhook endpoint registration API

---

## Commit History (Git Log)

| Hash | Date | Message |
|------|------|---------|
| 6526737 | 2026-02-23 | feat: register healthcare school church assistant autonomous-coding in catalog and platform topology |
| 57c8ad5 | 2026-02-23 | docs: add PR-safe branch normalization record |
| f097e51 | 2026-02-23 | feat: deep import selected source service directories for consolidation |
| 5d5d57a | 2026-02-23 | feat: apply consolidation merger contracts and module scaffolds |
| e383b78 | 2026-02-23 | docs: add ERP repository map |
| 7e828bc | 2026-02-23 | chore: ignore local build artifacts |
| 28c3dd8 | 2026-02-23 | docs: map platform implementation to attached context artifacts |
| 668e694 | 2026-02-23 | feat: add ERP suite integration contract and AIDD baseline |
| 594f011 | 2026-02-23 | chore: initialize ERP-Platform |

---

*For full release notes, see [02-Release-Notes.md](./02-Release-Notes.md).*
