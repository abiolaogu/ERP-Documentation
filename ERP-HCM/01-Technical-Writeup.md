# ERP-HCM Technical Writeup

## 1. Executive Summary

ERP-HCM (Human Capital Management) is a comprehensive, enterprise-grade workforce management platform that consolidates six previously independent repositories -- ERP-HRMS, HRMS/HRMS, ERP-opensase-hr, ERP-Workforce-Management, Workforce Management Platform/Time-Attendant-Software, and ERP-opensase-scheduling -- into a unified, multi-tenant microservices suite. The module delivers hire-to-retire employee lifecycle management, multi-country payroll processing (with first-class Nigerian PAYE/NHF/Pension support), recruitment via an AI-enabled applicant tracking system, performance management through OKR/KPI and 360-degree review cycles, learning management with SCORM/xAPI support, time and attendance with geofence and biometric verification, workforce planning with scenario modeling, benefits enrollment, document management with digital signatures, and compliance automation covering GDPR, SOC 2, and NDPR.

The system is designed as a standalone product that also integrates into the broader ERP suite through ERP-Platform entitlements, ERP-Directory/ERP-IAM identity federation, and NATS JetStream event backbone. It operates under a strict AIDD (AI-Driven Development) guardrail framework that governs autonomous, supervised, and prohibited actions across all services.

## 2. Architecture Overview

### 2.1 Consolidation Model

ERP-HCM was born from a strategic consolidation effort to unify six disparate HR-related codebases. The merge manifest (`merge/MERGE_MANIFEST.yaml`) documents the lineage:

- **ERP-HRMS** and **HRMS/HRMS**: Core HR information system with employee records, departments, and organizational hierarchy.
- **ERP-opensase-hr**: Open-source HR module contributing benefits administration and leave management.
- **ERP-Workforce-Management**: Workforce planning, compensation cycles, headcount planning.
- **Workforce Management Platform/Time-Attendant-Software**: Biometric attendance, geofencing, shift scheduling.
- **ERP-opensase-scheduling**: Shift and roster management.

Source snapshots from each predecessor are preserved under `merge/source-snapshots/` for audit and compliance traceability.

### 2.2 Service Decomposition

The consolidated module exposes 14 distinct microservices, each deployed independently:

| Service | Port | Responsibility |
|---------|------|----------------|
| `employee-service` | 8080 | Employee lifecycle, onboarding, profiles, org chart |
| `payroll-service` | 8080 | Multi-country payroll engine, salary structures, payslips |
| `recruitment-service` | 8080 | ATS, job requisitions, candidate pipeline, AI resume parsing |
| `performance-service` | 8080 | OKR cycles, KPIs, 360-degree reviews, calibration, 9-box grid |
| `leave-service` | 8080 | Leave policies, requests, approvals, holiday calendars |
| `time-attendance-service` | 8080 | Clock-in/out, geofencing, biometric, shift management |
| `benefits-service` | 8080 | Benefits plans, enrollment, EWA (Earned Wage Access) |
| `learning-service` | 8080 | LMS, SCORM packages, course catalog, certifications |
| `compensation-service` | 8080 | Salary bands, pay grades, compensation cycles |
| `workforce-planning-service` | 8080 | Headcount planning, scenario modeling, budget allocation |
| `compliance-service` | 8080 | Labor law compliance, GDPR/NDPR, audit trails |
| `document-service` | 8080 | Document management, digital signatures, templates |
| `facilities-service` | 8080 | Room/desk booking, facilities management |
| `facilities-management` | 8090 | Standalone facilities module (separate Go module) |

### 2.3 Imported Core (`imports/hrms_core`)

The `imports/hrms_core/` directory contains the deep-merged source from the HRMS predecessor repositories. This is a standalone Go module (`github.com/peopleforce/hrms`) built with Go 1.24+ and contains the production-grade implementations:

- **40+ internal packages** organized in domain-driven design with `domain/`, `service/`, `repository/`, `handlers/`, and `engine/` layers.
- **47 SQL migration files** spanning 30+ domain schemas (employee, payroll, attendance, recruitment, performance, benefits, LMS, DMS, workforce, analytics, and more).
- **Next.js 14 frontend** with TypeScript, Tailwind CSS, Radix UI, React Query, Zustand state management, and internationalization (en, fr, es, ar).

## 3. Technology Stack

### 3.1 Backend

- **Language**: Go 1.24+ (module: `github.com/peopleforce/hrms`)
- **HTTP Router**: Chi v5.0.12 (with Gorilla Mux for legacy routes)
- **Database**: PostgreSQL 16 via pgx/v5 driver and sqlx ORM
- **Cache**: Redis 7 via go-redis/v9
- **Messaging**: NATS JetStream for event-driven architecture
- **Search**: Meilisearch / Elasticsearch v7 for full-text search
- **Authentication**: RS256 JWT tokens with MFA (TOTP), managed via ERP-IAM OIDC federation
- **Encryption**: AES-256-GCM field-level encryption with HashiCorp Vault key management
- **Monitoring**: Prometheus client + OpenTelemetry (OTLP export)
- **Logging**: zerolog (structured JSON logging)
- **File Storage**: AWS S3 via aws-sdk-go-v2
- **Scheduling**: robfig/cron for periodic jobs (payroll auto-run, leave accrual)
- **Excel Processing**: excelize for bulk import/export
- **Decimal Arithmetic**: shopspring/decimal for financial calculations (payroll, tax)

### 3.2 Frontend

- **Framework**: Next.js 14.1.0 with App Router
- **UI Components**: Radix UI primitives, Headless UI, Heroicons, Lucide icons
- **State Management**: Zustand v4.5, React Query v5
- **Forms**: React Hook Form + Zod validation
- **Charts**: Recharts v2
- **Data Tables**: TanStack Table v8
- **Styling**: Tailwind CSS 3.4 + tailwindcss-animate + class-variance-authority
- **Auth**: NextAuth v4
- **I18n**: next-intl with message catalogs in en, fr, es, ar
- **Testing**: Jest + React Testing Library + Playwright (E2E)

### 3.3 Mobile

- **Framework**: Flutter (planned, referenced in architecture)
- **Biometric Integration**: Device-native fingerprint/face scan for attendance clock-in

### 3.4 Infrastructure

- **Containerization**: Docker (Alpine-based multi-stage builds)
- **Orchestration**: Kubernetes (Helm charts)
- **CI/CD**: GitHub Actions
- **Secret Management**: HashiCorp Vault
- **API Gateway**: Custom Go gateway with rate limiting, CORS, CSRF, tenant context injection

## 4. Multi-Tenancy Architecture

ERP-HCM implements a **shared-database, schema-isolated** multi-tenancy model:

- Every request must include `X-Tenant-ID` header (UUID format).
- The `TenantContext` middleware extracts tenant ID from JWT claims and stores it in Go context.
- Super-admins may override tenant via `X-Tenant-ID` header for cross-tenant operations (the only scenario where the AIDD guardrail `cross_tenant_data_access` is permitted under supervision).
- All database queries are scoped by `tenant_id` column using repository-level filtering.
- Schema-level isolation uses PostgreSQL schemas (e.g., `employee.*`, `payroll.*`) for logical separation.

## 5. Nigerian Payroll Engine (Deep Dive)

The payroll engine (`internal/payroll/engine/nigeria/`) implements Nigerian tax law with precision:

### 5.1 PAYE Calculation

The Personal Income Tax Act (PITA) graduated tax bands are implemented:

| Band | Width (NGN) | Rate |
|------|-------------|------|
| 1 | First 300,000 | 7% |
| 2 | Next 300,000 | 11% |
| 3 | Next 500,000 | 15% |
| 4 | Next 500,000 | 19% |
| 5 | Next 1,600,000 | 21% |
| 6 | Above 3,200,000 | 24% |

### 5.2 Consolidated Relief Allowance (CRA)

```
CRA = Higher of (NGN 200,000 OR 1% of annual gross) + 20% of annual gross
```

### 5.3 Statutory Deductions

- **Pension**: Employee (8%) + Employer (10%) per Pension Reform Act
- **NHF**: National Housing Fund (2.5% of basic salary)
- **NHIS**: National Health Insurance Scheme
- **NSITF**: Nigeria Social Insurance Trust Fund
- **ITF**: Industrial Training Fund

All monetary calculations use `shopspring/decimal` to avoid floating-point rounding errors, storing amounts in kobo (int64) internally.

### 5.4 Payment Integration

- **Flutterwave**: Primary payment processor for salary disbursement
- **Remita**: Alternative payment gateway (government-favored)
- **Bank file generation**: NIBSS-format bank instruction files
- **EWA (Earned Wage Access)**: Allows employees to access earned but unpaid wages

## 6. Security Architecture

### 6.1 Authentication

- RS256 JWT with configurable expiry (default: 15min access, 7d refresh)
- MFA via TOTP (Time-based One-Time Password)
- Account lockout after 5 failed attempts (30min lockout)
- Password policy: minimum 8 characters, special character required, 5-password history

### 6.2 Authorization

- Role-based access control (RBAC) with tenant-scoped roles
- Super-admin, tenant-admin, HR admin, payroll admin, manager, employee roles
- Field-level access control (e.g., salary data restricted to payroll admins)

### 6.3 Data Protection

- AES-256-GCM encryption for PII fields (SSN, bank details, tax IDs)
- HashiCorp Vault for key management with automated rotation
- Audit logging on all data mutations
- GDPR-compliant data subject request (DSR) handling

### 6.4 Middleware Security Stack

The gateway implements a comprehensive middleware chain:
1. CORS (configurable origins)
2. CSRF protection
3. Security headers (HSTS, CSP, X-Frame-Options)
4. Rate limiting (token bucket, per-tenant)
5. JWT authentication
6. Tenant context injection
7. Audit trail logging
8. Request/response encryption (optional)

## 7. Event-Driven Architecture

ERP-HCM publishes and subscribes to events using NATS JetStream with CloudEvents envelope format:

### 7.1 Topic Convention

```
erp.hcm.<entity>.<action>
```

### 7.2 Event Catalog

The system publishes 75+ event types across 15 domains (employee, payroll, leave, recruitment, performance, attendance, benefits, learning, compensation, compliance, document, facilities, workforce-planning, time-attendance). Each CRUD operation emits its corresponding event.

### 7.3 Cross-Module Integration

Events flow to other ERP modules:
- `erp.hcm.employee.created` triggers ERP-Finance GL journal entries
- `erp.hcm.payroll.paid` triggers ERP-Finance accounts payable
- `erp.hcm.employee.terminated` triggers ERP-Assets return workflow

## 8. AIDD Guardrails

The AI-Driven Development framework (`erp/aidd.guardrails.yaml`) enforces:

- **Autonomous actions**: Read-only queries, low-risk notifications
- **Supervised actions**: Data mutations, workflow automation, bulk operations (require human approval)
- **Prohibited actions**: Cross-tenant data access, irreversible delete without backup, privilege escalation
- **Controls**: Human-in-the-loop for high-risk operations, decision logging, 24-hour rollback window

## 9. Performance Characteristics

### 9.1 Database Optimization

- Connection pooling: 100 max open, 25 idle connections via pgx
- TimescaleDB for time-series attendance data (hypertables)
- Composite indexes on (tenant_id, entity_id) for all tenant-scoped queries
- Performance indexes added via dedicated migration (000002_add_performance_indexes)

### 9.2 Caching Strategy

- Redis 7 with 100-connection pool
- Session caching (5min TTL)
- Configuration caching (1hr TTL)
- Rate limit counters (sliding window)

### 9.3 API Performance Targets

- P99 latency < 200ms for read operations
- Payroll run processing: < 5 minutes for 10,000 employees
- Concurrent user support: 5,000+ per tenant

## 10. Frontend Architecture

The Next.js 14 application uses the App Router pattern with nested layouts:

### 10.1 Page Structure

```
(app)/
  dashboard/           - Employee dashboard
  profile/             - Employee profile
  attendance/          - Clock-in/out, attendance history
  leaves/              - Leave requests
  payslips/            - Payslip viewer
  performance/         - Goals, reviews, feedback
    goals/             - OKR tracking
    reviews/           - Performance review cycles
    feedback/          - 360-degree feedback
    admin/             - Performance admin
  recruitment/         - Job postings, applications
    jobs/              - Job listing
    applications/      - Application tracker
    talent-pool/       - Talent database
  learning/            - LMS courses
  workforce/           - Workforce planning
    scenarios/         - Scenario modeling
  compensation/        - Compensation management
    cycles/            - Compensation cycle detail
  admin/               - Admin panels
    payroll/           - Payroll administration
    leave/             - Leave policy management
    benefits/          - Benefits administration
    documents/         - Document management
  org-chart/           - Interactive org chart
  tracking/            - Employee tracking
  communication/       - Internal communications
  engage/              - Engagement surveys
  assets/              - IT asset management
```

### 10.2 State Architecture

- **Server state**: React Query v5 with optimistic updates
- **Client state**: Zustand stores for UI state
- **Form state**: React Hook Form with Zod schemas
- **Data tables**: TanStack Table with server-side pagination

## 11. Testing Strategy

### 11.1 Unit Tests

- Go test files throughout (`*_test.go`) covering domain logic
- Nigerian payroll calculator tests (tax bands, CRA, NHF, pension)
- Geofence validation tests (Haversine distance, anti-spoofing)
- Employee field filter tests
- Leave policy tests

### 11.2 Integration Tests

- Testcontainers for PostgreSQL and Redis
- Payroll integration tests with real database
- API handler integration tests

### 11.3 E2E Tests

- Playwright for frontend E2E testing
- Makefile targets: `test`, `test-integration`, `test-e2e`

## 12. Deployment Model

### 12.1 Containerization

Each service uses a minimal two-stage Docker build:
1. `golang:1.22-alpine` build stage
2. `alpine:3.20` runtime (< 20MB image)

### 12.2 Configuration

Environment-based configuration via `PF_*` environment variables:
- `PF_SERVER_*`: HTTP server settings
- `PF_DATABASE_*`: PostgreSQL connection
- `PF_REDIS_*`: Redis connection
- `PF_NATS_*`: NATS JetStream
- `PF_AUTH_*`: JWT/MFA/password settings
- `PF_CORS_*`: CORS policy
- `PF_LOG_*`: Logging configuration

### 12.3 Health Checks

Every service exposes `GET /healthz` returning:
```json
{"status": "healthy", "module": "ERP-HCM", "service": "<service-name>"}
```

## 13. Competitive Positioning

ERP-HCM is benchmarked against industry leaders:

| Feature | ERP-HCM | Workday | SAP SuccessFactors | BambooHR |
|---------|---------|---------|---------------------|----------|
| Multi-country payroll | Yes (NG, US, UK, GH, KE) | Yes | Yes | Limited |
| Nigerian PAYE engine | Native | Plugin | Plugin | No |
| Open-source core | Yes | No | No | No |
| Self-hosted option | Yes | No | No | No |
| AIDD guardrails | Yes | No | No | No |
| GraphQL API | Yes (Hasura) | No | Yes | No |
| Real-time events | NATS JetStream | Proprietary | SAP Event Mesh | Webhooks |
| Geofenced attendance | Yes (Haversine) | Limited | Yes | No |
| EWA (Earned Wage Access) | Built-in | Third-party | Third-party | No |

## 14. Conclusion

ERP-HCM represents a mature, production-ready HCM platform that combines the depth of enterprise systems like Workday and SAP SuccessFactors with the flexibility of open-source software. Its Go microservices architecture, Nigerian-first payroll engine, AIDD guardrails, and comprehensive frontend deliver a complete hire-to-retire solution for organizations operating in Africa and globally.
