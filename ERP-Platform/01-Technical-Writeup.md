# ERP-Platform Technical Writeup

> **Document ID:** ERP-PLAT-TW-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [04-Software-Architecture.md](./04-Software-Architecture.md), [12-High-Level-Design.md](./12-High-Level-Design.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## 1. Executive Summary

ERP-Platform is the unified control plane for a comprehensive 20-module Enterprise Resource Planning suite. It serves as the central nervous system that orchestrates product catalog management, subscription services, tenant provisioning, entitlement enforcement, module discovery, web hosting, marketplace operations, audit logging, and notification delivery across the entire ERP ecosystem.

The platform embodies the philosophy of "Business at the Speed of Prompt" -- a single API call can provision an entire tenant with all subscribed modules, configure entitlements, set up web hosting, and activate the necessary services. This dramatically reduces the time-to-value for enterprise customers from weeks to minutes.

The system is built on a microservices architecture using Go as the primary language, deployed on Kubernetes, and backed by PostgreSQL for relational data, Redis for caching, NATS/Redpanda for event streaming, and Docker for containerization.

---

## 2. System Purpose and Vision

### 2.1 Problem Statement

Enterprise ERP deployments traditionally suffer from fragmented administration. Each module (CRM, HCM, Finance, SCM, etc.) often has its own subscription model, its own tenant management, and its own administrative console. This creates operational overhead, billing complexity, and a disjointed user experience.

### 2.2 Solution

ERP-Platform consolidates all cross-cutting platform concerns into a single control plane:

- **Unified Product Catalog**: A single `products.json` catalog defines all 20 modules, their SKUs, categories, capabilities, and bundling rules. The catalog currently tracks 20 standalone modules and 28+ bundle/standalone SKUs across categories including core, business-ops, productivity, intelligence, security, integration, industry-vertical, and platform-tools.

- **Subscription Service**: Supports three plan types -- `single` (individual module), `bundle` (curated groupings like Starter, Professional, Enterprise), and `suite` (custom combinations). Bundle SKU resolution automatically expands bundle identifiers into their constituent module SKUs.

- **Tenant Provisioning**: A single POST to `/v1/tenant-provisioner` triggers the full provisioning pipeline -- database schema creation, entitlement seeding, module activation, and web hosting configuration.

- **Entitlement Engine**: Runtime entitlement checks ensure tenants can only access modules included in their active subscription. The entitlement query path (`GET /v1/entitlements/{tenant_id}`) returns the resolved list of entitled module SKUs.

- **Module Registry**: Auto-discovery of all ERP modules via periodic health checks. Each module exposes `/healthz` and the registry polls at configurable intervals to maintain a real-time topology map.

- **Web Hosting**: Domain management, SSL certificate provisioning, and CDN configuration for tenant-facing applications.

- **Marketplace**: Third-party module installation, listing management, and activation lifecycle.

- **Audit Service**: Platform-wide immutable audit logging for all CRUD operations, with CloudEvents-compliant event publishing.

- **Notification Hub**: Centralized notification routing across email, SMS, push, and in-app channels.

- **Activation Wizard**: Guided tenant onboarding flow that walks administrators through initial configuration.

### 2.3 AIDD Guardrails

All AI-integrated operations within the platform enforce AIDD (AI-Integrated Development Discipline) guardrails:

| Guardrail | Threshold |
|-----------|-----------|
| Minimum confidence for auto-execution | 0.70 |
| Medium confidence (human review suggested) | 0.82 |
| High-risk auto-execute | Disabled |
| Maximum blast radius (records) | 5,000 |
| High-value threshold (USD) | $100,000 |

The enforcement path ensures every AI-driven action passes through the policy engine, which evaluates guardrails and either auto-approves, queues for human approval, or blocks the action. All decisions are written to the immutable audit stream.

---

## 3. Architecture Summary

### 3.1 Microservices Topology

ERP-Platform consists of nine core microservices, each deployed as an independent Go binary in a Docker container:

| Service | Port | Base Path | Responsibility |
|---------|------|-----------|----------------|
| subscription-hub | 8091 | `/v1/subscriptions`, `/v1/products`, `/v1/entitlements` | Product catalog, subscription CRUD, entitlement resolution |
| tenant-provisioner | 8080 | `/v1/tenant-provisioner` | Tenant lifecycle management |
| entitlement-engine | 8080 | `/v1/entitlement-engine` | Runtime entitlement evaluation |
| module-registry | 8080 | `/v1/module-registry` | Module discovery and health tracking |
| marketplace | 8080 | `/v1/marketplace` | Third-party module management |
| audit-service | 8080 | `/v1/audit` | Immutable audit log storage |
| notification-hub | 8080 | `/v1/notification-hub` | Multi-channel notification dispatch |
| web-hosting | 8080 | `/v1/web-hosting` | Domain, SSL, CDN management |
| activation-wizard | 8080 | `/v1/activation-wizard` | Guided onboarding flows |

### 3.2 Integration with ERP Suite

The platform integrates with all 19 other ERP modules across six categories:

- **Business Operations**: ERP-HCM, ERP-CRM, ERP-Marketing, ERP-Finance, ERP-Commerce, ERP-eCommerce, ERP-SCM, ERP-Projects
- **Industry Verticals**: ERP-Healthcare, ERP-School-Management, ERP-Church-Management, ERP-BSS-OSS
- **Productivity**: ERP-Workspace
- **Intelligence**: ERP-BI, ERP-AI
- **Security**: ERP-IAM
- **Integration**: ERP-iPaaS
- **Platform Tools**: ERP-Assistant, ERP-Autonomous-Coding

Each module is registered in the product catalog with a unique SKU, repository reference, category classification, and capability count. The docker-compose topology (`infra/docker-compose.platform.yml`) defines health checks for module discovery using the standard `/healthz` endpoint pattern.

### 3.3 Consolidation Heritage

ERP-Platform was formed through the consolidation of multiple previously independent repositories:

- **ERP-Platform** (original control plane)
- **ERP-BAC-Business-Activation** (business activation center)
- **ERP-BAC-BOS-AI** (BOS AI activation layer)
- **BOS/BAC-BOS-AI** (legacy BOS activation)
- **ERP-Web-Hosting** (web hosting service)

The merge manifest (`merge/MERGE_MANIFEST.yaml`) tracks the consolidation lineage, and imported source artifacts are preserved in `imports/bac_activation/` for reference.

---

## 4. Technology Stack

### 4.1 Backend

| Technology | Version | Purpose |
|-----------|---------|---------|
| Go | 1.22+ | Primary service language |
| Go net/http | stdlib | HTTP server and routing |
| encoding/json | stdlib | JSON serialization |

### 4.2 Data Layer

| Technology | Version | Purpose |
|-----------|---------|---------|
| PostgreSQL | 16 | Primary relational database |
| Redis | 7 | Caching, session store, rate limiting |
| NATS | 2.10 | Event streaming with JetStream |
| Redpanda/Kafka | -- | Alternative event backbone |

### 4.3 Infrastructure

| Technology | Version | Purpose |
|-----------|---------|---------|
| Docker | -- | Container runtime |
| Kubernetes | -- | Container orchestration |
| Docker Compose | 3.9 | Local development orchestration |
| Alpine Linux | 3.20 | Base container image |

### 4.4 Frontend (Admin Console)

| Technology | Version | Purpose |
|-----------|---------|---------|
| Next.js | 14 | Admin console framework |
| React | 18 | UI component library |
| TypeScript | 5+ | Type-safe frontend development |
| Vite | -- | Build tooling for activation console |
| Tailwind CSS | -- | Utility-first CSS framework |

### 4.5 CI/CD

| Technology | Purpose |
|-----------|---------|
| GitHub Actions | CI/CD pipeline |
| Python 3.11 | Documentation generation tooling |

---

## 5. Key Design Decisions

### 5.1 Go Standard Library HTTP Server

The subscription-hub and all platform services use Go's standard library `net/http` package rather than third-party frameworks like Gin, Echo, or Chi. This decision provides:

- **Zero external dependencies**: The subscription-hub's `go.mod` contains only the module declaration and Go version constraint.
- **Long-term stability**: No framework upgrade churn.
- **Minimal attack surface**: No transitive dependency vulnerabilities.
- **Performance**: The stdlib HTTP server is production-grade and well-optimized.

### 5.2 In-Memory Store with Mutex-Based Concurrency

The subscription-hub uses an in-memory `Store` struct with `sync.RWMutex` for concurrent access control. This design choice reflects the current stage of development:

- Read operations use `RLock()` for maximum concurrency.
- Write operations use exclusive `Lock()` for consistency.
- The store maps tenant IDs to subscription records and SKUs to products.

This pattern allows the service to function without a database dependency during development and testing, with a clear migration path to PostgreSQL for production persistence.

### 5.3 Bundle Resolution at Subscription Time

When a subscription request includes bundle SKUs (e.g., `enterprise`), the system resolves bundles to their constituent module SKUs at subscription creation time rather than at entitlement query time. This means:

- Entitlement lookups are fast (direct SKU list retrieval).
- Bundle changes do not retroactively affect existing subscriptions.
- The `dedupe()` function ensures no duplicate SKUs in the resolved list.

### 5.4 CloudEvents-Compatible Event Topology

All services publish events following the naming convention `erp.<module>.<entity>.<action>`. The platform defines 45+ event topics covering CRUD operations across all nine services. This aligns with the CloudEvents specification and enables:

- Cross-service event-driven workflows.
- Audit trail completeness.
- Integration with external event consumers via NATS or Kafka.

### 5.5 Catalog-Driven Architecture

The product catalog (`catalog/products.json`) serves as the single source of truth for the entire ERP suite topology. It defines:

- Module SKUs, names, and repository references.
- Module categories and capability counts.
- Bundle definitions with include lists.
- Standalone availability flags.

This catalog-driven approach means adding a new module to the suite requires only a catalog entry update -- no code changes to the platform services.

### 5.6 Multi-Stage Docker Builds

All Dockerfiles use multi-stage builds with `golang:1.22-alpine` as the build stage and `alpine:3.20` as the runtime stage. This produces minimal container images (typically under 20MB) with no build toolchain in production.

---

## 6. Performance Characteristics

### 6.1 Latency Profile

| Operation | Target P50 | Target P99 |
|-----------|-----------|-----------|
| Health check (`/healthz`) | < 1ms | < 5ms |
| Product catalog retrieval | < 5ms | < 20ms |
| Subscription creation | < 10ms | < 50ms |
| Subscription lookup | < 2ms | < 10ms |
| Entitlement query | < 2ms | < 10ms |
| Tenant provisioning | < 500ms | < 2s |
| Module registry health poll | < 100ms | < 500ms |

### 6.2 Throughput Targets

- **Subscription Hub**: 10,000+ requests/second per instance (catalog reads and entitlement queries).
- **Audit Service**: 50,000+ events/second ingestion rate.
- **Notification Hub**: 1,000+ notifications/second dispatch rate.
- **Module Registry**: Supports health polling of 100+ modules with 30-second intervals.

### 6.3 Resource Footprint

Each Go microservice binary compiles to approximately 8-15MB. The Alpine-based Docker images are under 25MB each. Memory usage per service instance is typically 20-50MB under normal load, scaling to 200MB under heavy write workloads.

---

## 7. Scalability Approach

### 7.1 Horizontal Scaling

Every platform service is stateless (or will be once persistence moves to PostgreSQL) and can be horizontally scaled via Kubernetes HPA (Horizontal Pod Autoscaler). The recommended scaling strategy:

- **Subscription Hub**: Scale based on CPU utilization (target 60%) or request rate.
- **Audit Service**: Scale based on event queue depth.
- **Notification Hub**: Scale based on notification queue backlog.
- **Tenant Provisioner**: Scale based on provisioning job queue depth.

### 7.2 Database Scaling

- **PostgreSQL**: Read replicas for entitlement queries, connection pooling via PgBouncer.
- **Redis**: Redis Cluster for horizontal cache scaling.
- **NATS**: JetStream clustering for event stream durability and scalability.

### 7.3 Multi-Tenant Isolation

The platform enforces tenant isolation through:

- **Row-Level Security (RLS)** in PostgreSQL: Every table includes a `tenant_id` column with RLS policies.
- **X-Tenant-ID Header**: Required on all business endpoints; validated at the API gateway.
- **Connection pooling per tenant**: Prevents noisy-neighbor effects.

### 7.4 Caching Strategy

- **Product Catalog**: Cached in Redis with 5-minute TTL; invalidated on catalog updates.
- **Entitlements**: Cached per tenant with 60-second TTL; invalidated on subscription changes.
- **Module Registry**: Health status cached with 30-second TTL matching the poll interval.

---

## 8. Security Architecture

### 8.1 Authentication

- JWT tokens issued by ERP-IAM via OIDC flow.
- All business endpoints require valid JWT in the `Authorization` header.
- The `/healthz` endpoint is unauthenticated for Kubernetes probe compatibility.

### 8.2 Authorization

- Role-Based Access Control (RBAC) with platform-defined roles: `platform-admin`, `tenant-admin`, `module-admin`, `viewer`.
- Entitlement-based access: API calls are checked against the tenant's active subscription SKUs.
- AIDD guardrails add an additional authorization layer for AI-driven operations.

### 8.3 Data Protection

- TLS 1.3 for all inter-service communication.
- Encryption at rest for PostgreSQL (AES-256).
- Secret management via Kubernetes Secrets or HashiCorp Vault.
- Audit logs are append-only and tamper-evident.

---

## 9. Operational Excellence

### 9.1 Observability

- **Structured Logging**: All services use Go's `log` package with structured JSON output.
- **Health Checks**: Every service exposes `/healthz` for Kubernetes liveness and readiness probes.
- **Event Tracing**: CloudEvents envelope includes correlation IDs for distributed tracing.

### 9.2 Deployment

- **CI/CD**: GitHub Actions pipeline with build, test, security scan, and deployment stages.
- **Container Registry**: Docker images tagged with Git SHA and semantic version.
- **Kubernetes**: Helm charts for production deployment with configurable replicas, resource limits, and pod disruption budgets.

### 9.3 Disaster Recovery

- **RPO**: 1 hour (PostgreSQL continuous archiving).
- **RTO**: 15 minutes (Kubernetes pod restart + database failover).
- **Backup Strategy**: Automated daily full backups, continuous WAL archiving.
- **Multi-Region**: Active-passive with DNS failover capability.

---

## 10. Conclusion

ERP-Platform represents a modern, cloud-native approach to ERP suite management. By consolidating cross-cutting concerns into a single control plane with a catalog-driven architecture, it enables rapid tenant provisioning, flexible subscription management, and comprehensive platform governance. The microservices architecture ensures each capability can evolve independently while maintaining coherent integration through event-driven patterns and shared contracts.

The platform's commitment to AIDD guardrails, immutable audit logging, and tenant isolation positions it as an enterprise-grade control plane suitable for regulated industries including healthcare, education, telecommunications, and financial services.

---

*For detailed architecture diagrams, see [04-Software-Architecture.md](./04-Software-Architecture.md). For API contracts, see [21-API-Documentation.md](./21-API-Documentation.md). For deployment procedures, see [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md).*
