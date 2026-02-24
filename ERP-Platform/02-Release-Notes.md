# ERP-Platform Release Notes

> **Document ID:** ERP-PLAT-RN-001
> **Version:** 1.0.0
> **Release Date:** 2026-02-23
> **Status:** Released
> **Related Documents:** [16-Changelog.md](./16-Changelog.md), [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md)

---

## Release v1.0.0 -- "Business at the Speed of Prompt"

### Release Summary

ERP-Platform v1.0.0 is the inaugural general-availability release of the unified control plane for the 20-module ERP suite. This release delivers the foundational platform services that enable product catalog management, subscription lifecycle handling, tenant provisioning, entitlement enforcement, module discovery, marketplace operations, audit logging, notification delivery, web hosting, and guided activation.

---

## New Features

### Platform Core

- **Subscription Hub** (`subscription-hub`)
  - Product catalog API serving 20 standalone modules and 28+ bundle SKUs.
  - Three subscription plan types: `single`, `bundle`, `suite`.
  - Automatic bundle SKU resolution with deduplication.
  - Tenant-scoped subscription CRUD operations.
  - Entitlement query endpoint returning resolved module SKU lists.
  - Catalog versioning with `2026-02-23` schema.

- **Tenant Provisioner** (`tenant-provisioner`)
  - Single-call tenant provisioning via `POST /v1/tenant-provisioner`.
  - Tenant-scoped CRUD lifecycle (create, read, update, delete).
  - CloudEvents-compatible event emission on all operations.
  - X-Tenant-ID header enforcement for multi-tenant isolation.

- **Entitlement Engine** (`entitlement-engine`)
  - Runtime entitlement evaluation service.
  - Tenant-scoped entitlement CRUD operations.
  - Integration with subscription hub for entitlement seeding.

- **Module Registry** (`module-registry`)
  - Module auto-discovery via `/healthz` health checks.
  - Module registration and deregistration lifecycle.
  - Support for 20 modules across six categories.

- **Marketplace** (`marketplace`)
  - Third-party module listing management.
  - Module installation and activation lifecycle.
  - Tenant-scoped marketplace operations.

- **Audit Service** (`audit-service`)
  - Platform-wide immutable audit log storage.
  - CloudEvents-compliant event publishing.
  - 45+ event topics across all platform services.
  - CRUD operations for audit record management.

- **Notification Hub** (`notification-hub`)
  - Centralized notification dispatch service.
  - Multi-channel support architecture (email, SMS, push, in-app).
  - Tenant-scoped notification management.

- **Web Hosting** (`web-hosting`)
  - Domain management service.
  - SSL certificate provisioning integration.
  - CDN configuration management.
  - Tenant-scoped hosting operations.

- **Activation Wizard** (`activation-wizard`)
  - Guided tenant onboarding flow service.
  - Step-by-step activation configuration.
  - Tenant-scoped wizard state management.

### Product Catalog

- **20 Standalone Modules**: erp-platform, erp-hcm, erp-crm, erp-marketing, erp-finance, erp-commerce, erp-ecommerce, erp-scm, erp-projects, erp-workspace, erp-bi, erp-ai, erp-iam, erp-ipaas, erp-healthcare, erp-school-management, erp-church-management, erp-bss-oss, erp-assistant, erp-autonomous-coding.

- **8 Curated Bundles**: Starter, Professional, Enterprise, Healthcare Suite, Education Suite, Faith Suite, Telecom Suite, Developer Suite.

- **20 Standalone Bundles**: One wrapper bundle per module for uniform subscription handling.

### AIDD Guardrails

- Confidence threshold checks (min: 0.70, medium: 0.82).
- Blast-radius controls (max 5,000 records).
- Human approval gates for high-value operations (> $100,000 USD).
- Immutable audit logging for all AI recommendations and actions.
- Tenant isolation validation before data access.
- Policy-as-code enforcement in CI/CD pipeline.

### Infrastructure

- Docker Compose orchestration with PostgreSQL 16, Redis 7, NATS 2.10.
- Multi-stage Docker builds using Go 1.22 Alpine and Alpine 3.20 runtime.
- Health check configuration for all module containers (30s interval, 5s timeout, 5 retries).
- GitHub Actions CI/CD pipeline for documentation generation.

---

## Improvements

- Consolidated five legacy repositories (ERP-Platform, ERP-BAC-Business-Activation, ERP-BAC-BOS-AI, BOS/BAC-BOS-AI, ERP-Web-Hosting) into a single unified platform.
- Imported BAC activation frontend (React + Vite + Tailwind) and backend (Python/LangGraph) as reference implementations.
- Standardized all service health check responses with `module` and `service` identification.
- Unified event topic naming convention: `erp.platform.<service>.<action>`.

---

## Known Issues

| ID | Severity | Description | Workaround |
|----|----------|-------------|------------|
| PLAT-001 | Medium | Subscription hub uses in-memory store; data lost on restart | Re-create subscriptions after restart; PostgreSQL persistence planned for v1.1.0 |
| PLAT-002 | Low | No SQL migrations detected; database schema managed externally | Use manual migration scripts or Flyway |
| PLAT-003 | Low | Service ports hardcoded to 8080 for non-hub services | Use PORT environment variable to override |
| PLAT-004 | Medium | No request validation beyond tenant_id and plan_type | Input validation middleware planned for v1.1.0 |
| PLAT-005 | Low | No rate limiting on API endpoints | Deploy behind API gateway with rate limiting |
| PLAT-006 | Medium | No authentication middleware in service code | JWT validation handled at API gateway layer |

---

## Migration Notes

### From Legacy Repositories

If migrating from the standalone ERP-BAC-Business-Activation or ERP-Web-Hosting repositories:

1. **API Endpoints**: All activation endpoints are now under `/v1/activation-wizard`. Update client configurations.
2. **Web Hosting**: All hosting endpoints are now under `/v1/web-hosting`. Update DNS management scripts.
3. **Event Topics**: Events now follow the `erp.platform.<service>.<action>` convention. Update event consumers.
4. **Docker Images**: New images are built from the `ERP-Platform/services/` directory. Update CI/CD references.

### Fresh Installation

1. Clone the ERP-Platform repository.
2. Run `docker-compose -f infra/docker-compose.platform.yml up` for the full stack.
3. Verify with `curl http://localhost:8091/healthz`.
4. Load the product catalog via `curl http://localhost:8091/v1/products`.

---

## Compatibility Matrix

| Component | Minimum Version | Recommended Version | Notes |
|-----------|----------------|-------------------|-------|
| Go | 1.22 | 1.22+ | Required for service compilation |
| Docker | 20.10 | 25.0+ | Multi-stage build support required |
| Docker Compose | 2.0 | 2.24+ | Compose spec 3.9 |
| PostgreSQL | 15 | 16 | Row-level security support |
| Redis | 6 | 7 | Streams and cluster support |
| NATS | 2.9 | 2.10 | JetStream required |
| Node.js | 18 | 20+ | Admin console build |
| Kubernetes | 1.27 | 1.29+ | HPA v2 required |
| ERP-IAM | 1.0.0 | 1.0.0+ | OIDC/JWT provider |
| ERP-Healthcare | 1.0.0 | 1.0.0+ | Health check compatible |
| ERP-School-Management | 1.0.0 | 1.0.0+ | Health check compatible |
| ERP-Church-Management | 1.0.0 | 1.0.0+ | Health check compatible |
| ERP-Assistant | 1.0.0 | 1.0.0+ | Health check compatible |
| ERP-Autonomous-Coding | 1.0.0 | 1.0.0+ | Health check compatible |

---

## Upgrade Instructions

### From Pre-Release

1. **Backup**: Export all subscription data from the in-memory store if any exists.
2. **Pull**: `git pull origin main`
3. **Rebuild**: `docker-compose -f infra/docker-compose.platform.yml build --no-cache`
4. **Deploy**: `docker-compose -f infra/docker-compose.platform.yml up -d`
5. **Verify**: Run health checks on all services.
6. **Re-seed**: Re-create subscriptions via the API if needed.

### Kubernetes Deployment

1. Update Helm chart values with new image tags.
2. Apply database migrations (when available).
3. Rolling update: `kubectl rollout restart deployment/erp-platform -n erp`.
4. Verify pod health: `kubectl get pods -n erp -l app=erp-platform`.

---

## Deprecations

| Feature | Deprecated In | Removal Target | Replacement |
|---------|--------------|----------------|-------------|
| In-memory subscription store | v1.0.0 | v1.1.0 | PostgreSQL persistence |
| Legacy BAC API paths | v1.0.0 | v2.0.0 | `/v1/activation-wizard` |
| Legacy web hosting paths | v1.0.0 | v2.0.0 | `/v1/web-hosting` |

---

## Contributors

- ERP Platform Engineering Team
- ERP Architecture Council

---

*For the complete commit-level changelog, see [16-Changelog.md](./16-Changelog.md). For deployment pipeline details, see [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md).*
