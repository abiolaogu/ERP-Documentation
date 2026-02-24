# ERP-Platform Product Requirements Document (PRD)

> **Document ID:** ERP-PLAT-PRD-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Product Manager:** ERP Product Team
> **Status:** Approved
> **Related Documents:** [06-Business-Requirements-Document.md](./06-Business-Requirements-Document.md), [10-Use-Cases.md](./10-Use-Cases.md)

---

## 1. Problem Statement

Enterprise organizations deploying multi-module ERP suites face three critical challenges:

1. **Fragmented Administration**: Each ERP module (CRM, HCM, Finance, etc.) typically has its own subscription model, tenant management, and administrative console. Administrators must log into multiple systems to manage their technology stack.

2. **Slow Time-to-Value**: Provisioning a new tenant across multiple modules requires coordinated setup of databases, configurations, entitlements, domains, and user accounts. This process typically takes 2-6 weeks and involves multiple teams.

3. **Compliance Complexity**: Audit trails are scattered across modules, making regulatory compliance (SOC 2, GDPR, HIPAA) difficult and expensive. AI-driven automation lacks centralized guardrails.

ERP-Platform solves these problems by providing a unified control plane that consolidates product catalog management, subscription handling, tenant provisioning, entitlement enforcement, module discovery, and platform governance into a single system.

---

## 2. Target Users

### 2.1 User Personas

#### Persona 1: Sarah -- Platform Administrator
- **Role**: IT Operations Lead
- **Organization Size**: 5,000+ employees
- **Goals**: Manage the entire ERP suite from a single console; provision new tenants quickly; monitor platform health
- **Pain Points**: Currently uses 5+ admin panels; provisioning takes 3 weeks; no centralized audit view
- **Technical Skill**: High (CLI comfortable, API-literate)

#### Persona 2: Marcus -- Tenant Administrator
- **Role**: Department IT Manager
- **Organization Size**: Business unit of 200 people within large enterprise
- **Goals**: Self-service subscription management; activate new modules; manage users within tenant
- **Pain Points**: Must submit tickets for module activation; no visibility into entitlements
- **Technical Skill**: Medium (web UI preferred)

#### Persona 3: Priya -- Developer/Integrator
- **Role**: Software Engineer
- **Organization Size**: ISV building on ERP platform
- **Goals**: Publish modules to marketplace; integrate via APIs; test entitlements
- **Pain Points**: Unclear API documentation; no sandbox environment; manual marketplace listing
- **Technical Skill**: Very High (API-first, CLI-native)

#### Persona 4: James -- Compliance Officer
- **Role**: Chief Compliance Officer
- **Organization Size**: Regulated enterprise (healthcare/education)
- **Goals**: Centralized audit trail; data privacy compliance; AIDD guardrail oversight
- **Pain Points**: Audit data scattered across systems; manual compliance reporting
- **Technical Skill**: Low (needs dashboard views)

#### Persona 5: AI Agent
- **Role**: Automated system actor (ERP-AI, ERP-Assistant)
- **Goals**: Execute automated workflows; provision resources; manage configurations
- **Constraints**: Must pass AIDD guardrail checks; all actions must be auditable
- **Technical Skill**: API-only interaction

---

## 3. Feature Requirements

### P0 -- Must Have (v1.0.0)

#### F-001: Product Catalog API
- **Priority**: P0
- **Description**: RESTful API serving the complete ERP module catalog including standalone modules, bundles, and industry vertical suites.
- **Acceptance Criteria**:
  - `GET /v1/products` returns all products with SKU, name, repo, type, category, capabilities, and standalone flag
  - Catalog is versioned (current: `2026-02-23`)
  - Response time < 20ms P99
  - Bundle products include `includes` array listing constituent module SKUs
  - Catalog supports 6 categories: core, business-ops, productivity, intelligence, security, integration, industry-vertical, platform-tools

#### F-002: Subscription Management
- **Priority**: P0
- **Description**: Create and query subscriptions with automatic bundle resolution.
- **Acceptance Criteria**:
  - `POST /v1/subscriptions` creates subscription with tenant_id, plan_type, and SKUs
  - Supports three plan types: `single`, `bundle`, `suite`
  - Bundle SKUs are automatically resolved to constituent module SKUs
  - Duplicate SKUs are deduplicated in resolved list
  - `single` plan rejects bundle SKUs with descriptive error
  - `GET /v1/subscriptions/{tenant_id}` returns subscription with resolved SKUs
  - Returns 404 for unknown tenant_id

#### F-003: Entitlement Query
- **Priority**: P0
- **Description**: Query tenant entitlements to determine which modules a tenant can access.
- **Acceptance Criteria**:
  - `GET /v1/entitlements/{tenant_id}` returns `{tenant_id, entitlements: [sku_list]}`
  - Response time < 10ms P99
  - Returns 404 for tenants without subscriptions

#### F-004: Health Check Endpoint
- **Priority**: P0
- **Description**: Standard health check endpoint for Kubernetes probes and module registry.
- **Acceptance Criteria**:
  - `GET /healthz` returns `{"status":"ok","catalog_version":"..."}` for subscription hub
  - All services return `{"status":"healthy","module":"ERP-Platform","service":"<name>"}`
  - No authentication required
  - Response time < 5ms P99

#### F-005: Tenant Provisioner
- **Priority**: P0
- **Description**: Single-call tenant provisioning service.
- **Acceptance Criteria**:
  - `POST /v1/tenant-provisioner` creates tenant with provided configuration
  - Requires `X-Tenant-ID` header for all operations
  - Full CRUD lifecycle (create, read, update, delete)
  - Emits CloudEvents for all state changes
  - Returns 400 if X-Tenant-ID missing

#### F-006: Audit Service
- **Priority**: P0
- **Description**: Immutable audit logging for all platform operations.
- **Acceptance Criteria**:
  - All service operations emit events to `erp.platform.<service>.<action>` topics
  - Audit records are append-only (no update/delete in production)
  - Supports query by tenant_id
  - 45+ event topics defined

#### F-007: Multi-Tenant Isolation
- **Priority**: P0
- **Description**: All endpoints enforce tenant boundary validation.
- **Acceptance Criteria**:
  - Business endpoints require `X-Tenant-ID` header
  - Missing header returns 400 with `{"error":"missing X-Tenant-ID"}`
  - Data access scoped to requesting tenant

### P1 -- Should Have (v1.1.0)

#### F-008: Module Registry with Auto-Discovery
- **Priority**: P1
- **Description**: Automatic discovery of ERP modules via health check polling.
- **Acceptance Criteria**:
  - Registry polls `/healthz` on all registered module endpoints
  - Health check interval: 30 seconds
  - Timeout: 5 seconds
  - Retries: 5 before marking unhealthy
  - Module status dashboard accessible via admin console

#### F-009: Marketplace
- **Priority**: P1
- **Description**: Third-party module listing and installation management.
- **Acceptance Criteria**:
  - Publishers can create marketplace listings
  - Tenants can browse, install, and uninstall modules
  - Installation updates entitlements
  - All operations tenant-scoped

#### F-010: Notification Hub
- **Priority**: P1
- **Description**: Centralized notification routing across channels.
- **Acceptance Criteria**:
  - Support email, SMS, push, and in-app channels
  - Notification preferences per tenant
  - Template-based notification content
  - Delivery status tracking

#### F-011: Web Hosting Management
- **Priority**: P1
- **Description**: Domain, SSL, and CDN management for tenant applications.
- **Acceptance Criteria**:
  - Custom domain registration and configuration
  - Automated SSL certificate provisioning via Let's Encrypt
  - CDN configuration per tenant
  - DNS verification flow

#### F-012: Activation Wizard
- **Priority**: P1
- **Description**: Guided onboarding experience for new tenants.
- **Acceptance Criteria**:
  - Step-by-step wizard with progress tracking
  - Covers: company profile, module selection, user setup, configuration
  - Persists wizard state across sessions
  - Skip/resume capability

### P2 -- Nice to Have (v1.2.0+)

#### F-013: AIDD Guardrail Dashboard
- **Priority**: P2
- **Description**: Visual dashboard for monitoring and configuring AIDD guardrails.
- **Acceptance Criteria**:
  - Display current guardrail thresholds
  - Show AI action approval/rejection history
  - Allow threshold adjustment (with approval workflow)
  - Integration with audit service for guardrail decision logs

#### F-014: Subscription Analytics
- **Priority**: P2
- **Description**: Analytics dashboard showing subscription metrics.
- **Acceptance Criteria**:
  - Active subscription count by plan type
  - Module adoption rates
  - Churn metrics
  - Revenue attribution by bundle

#### F-015: Self-Service Catalog Extension
- **Priority**: P2
- **Description**: Allow platform admins to define custom bundles via UI.
- **Acceptance Criteria**:
  - Bundle creation wizard in admin console
  - SKU validation against product catalog
  - Preview mode before publishing
  - Version management for custom bundles

---

## 4. Success Metrics

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Tenant provisioning time | < 5 minutes (from single API call) | End-to-end timer in provisioner |
| Subscription API latency (P99) | < 50ms | Prometheus histogram |
| Entitlement query latency (P99) | < 10ms | Prometheus histogram |
| Platform uptime | 99.95% | Kubernetes health check monitoring |
| Module discovery accuracy | 100% registered modules detected | Module registry health poll audit |
| Audit log completeness | 100% state changes captured | Cross-reference event stream with API calls |
| Admin console user satisfaction | > 4.2/5.0 | In-app survey |
| Time-to-first-module-activation | < 10 minutes for new tenant | Activation wizard analytics |

---

## 5. Competitive Analysis

| Capability | ERP-Platform | Zoho One Admin | Microsoft 365 Admin | Oracle Cloud Console | SAP BTP Cockpit |
|-----------|-------------|---------------|-------------------|---------------------|-----------------|
| Unified multi-module catalog | Yes (20 modules) | Yes (~45 apps) | Yes (~30 apps) | Partial (siloed) | Partial (siloed) |
| Single-call tenant provisioning | Yes | No | No | No | No |
| Bundle resolution engine | Yes (8 bundles + custom) | Yes (tiers) | Yes (plans) | Yes (editions) | Yes (packages) |
| AIDD guardrails | Yes (native) | No | No | No | No |
| Real-time module health | Yes (30s polling) | Unknown | Yes | Limited | Limited |
| Marketplace for third-party | Yes | Yes (Zoho Marketplace) | Yes (AppSource) | Yes (Cloud Marketplace) | Yes (SAP Store) |
| Event-driven audit trail | Yes (CloudEvents) | Partial | Yes | Yes | Yes |
| Industry vertical bundles | Yes (Healthcare, Education, Faith, Telecom) | Limited | Limited | Yes | Yes |
| Open API standard | OpenAPI 3.1 | Proprietary | Microsoft Graph | REST/OData | REST |
| Self-hosted option | Yes (Kubernetes) | No (SaaS only) | No (SaaS only) | Limited | Limited |

### Differentiators

1. **"Business at the Speed of Prompt"**: Single API call provisions entire tenant -- no competitor offers this.
2. **AIDD Guardrails**: Native AI governance framework with configurable thresholds -- unique in ERP space.
3. **Catalog-Driven Architecture**: Adding modules requires only a JSON catalog update, not code changes.
4. **Zero External Dependencies**: Core services have no third-party Go library dependencies.
5. **Industry Vertical Bundles**: Pre-configured bundles for Healthcare, Education, Faith, and Telecom verticals.

---

## 6. Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| Performance | API response time (P50) | < 10ms |
| Performance | API response time (P99) | < 50ms |
| Scalability | Concurrent tenants | 10,000+ |
| Scalability | Requests per second per instance | 10,000+ |
| Availability | Uptime SLA | 99.95% |
| Security | Authentication | JWT (OIDC from ERP-IAM) |
| Security | Encryption in transit | TLS 1.3 |
| Security | Encryption at rest | AES-256 |
| Compliance | Audit log retention | 7 years |
| Compliance | Data residency | Configurable per tenant |
| Observability | Structured logging | JSON format |
| Observability | Health checks | /healthz on every service |

---

## 7. Out of Scope (v1.0.0)

- Multi-currency billing engine
- Custom workflow designer
- Mobile admin application
- GraphQL API (REST-only for v1)
- Real-time collaborative editing of platform configurations
- White-label / branding customization engine

---

## 8. Timeline and Milestones

| Milestone | Target Date | Features |
|-----------|-------------|----------|
| v1.0.0 GA | 2026-02-23 | F-001 through F-007 (P0 features) |
| v1.1.0 | 2026-Q2 | F-008 through F-012 (P1 features) + PostgreSQL persistence |
| v1.2.0 | 2026-Q3 | F-013 through F-015 (P2 features) + rate limiting |
| v2.0.0 | 2026-Q4 | GraphQL API, mobile admin, multi-currency billing |

---

*For business requirements, see [06-Business-Requirements-Document.md](./06-Business-Requirements-Document.md). For detailed use cases, see [10-Use-Cases.md](./10-Use-Cases.md).*
