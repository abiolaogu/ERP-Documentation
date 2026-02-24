# ERP-HCM v1.0.0 Release Notes

**Release Date**: 2026-02-23
**Module**: ERP-HCM (Human Capital Management)
**SKU**: `erp.hcm`
**Go Version**: 1.24+
**Stability**: General Availability (GA)

---

## Overview

ERP-HCM v1.0.0 marks the first general availability release of the consolidated Human Capital Management module. This release unifies six previously independent repositories (ERP-HRMS, HRMS/HRMS, ERP-opensase-hr, ERP-Workforce-Management, Workforce Management Platform/Time-Attendant-Software, and ERP-opensase-scheduling) into a single, cohesive microservices platform delivering end-to-end hire-to-retire employee management.

---

## Highlights

### Unified HCM Platform
- Consolidated 6 source repositories into one coherent module
- 14 microservices with consistent API contracts
- Shared authentication, multi-tenancy, and event infrastructure
- Preserved source snapshots for full audit traceability

### Multi-Country Payroll Engine
- Nigerian PAYE tax calculation with graduated bands per PITA
- Consolidated Relief Allowance (CRA) computation
- Pension (employee 8% + employer 10%) per Pension Reform Act
- NHF (2.5%), NHIS, NSITF, ITF statutory deductions
- Support for US FICA, UK HMRC (framework established)
- Earned Wage Access (EWA) built-in
- Multi-currency support with exchange rate management
- Bank file generation (NIBSS format)
- Payment integrations: Flutterwave, Remita

### Performance Management Suite
- OKR (Objectives and Key Results) cycle management
- KPI tracking with decimal-precision scoring
- 360-degree review cycles (self, manager, peer, upward)
- Calibration workflow with 9-box grid talent matrix
- Competency frameworks with proficiency levels
- Badge and recognition system

### Recruitment & ATS
- Job requisition workflow (draft to filled)
- Multi-stage candidate pipeline
- AI resume parsing (framework)
- Assessment and interview scheduling
- Offer management with approval chains
- Talent pool management
- Onboarding task automation

### Time & Attendance
- Geofenced clock-in/out with Haversine distance calculation
- Anti-spoofing: GPS accuracy validation, teleport detection, multi-device checks
- Biometric verification integration
- Shift management and scheduling
- Automatic clock-out policies
- QR code attendance support

### Learning Management System
- SCORM package support
- Course catalog with categories and prerequisites
- Multi-delivery modes (online, classroom, blended)
- Certificate generation on completion
- Enrollment management with approval workflows
- Progress tracking and completion settings

---

## New Features (v1.0.0)

### Employee Service
- [x] Complete employee lifecycle management (hire, transfer, promote, terminate, retire)
- [x] Employee number auto-generation with configurable formats
- [x] Onboarding engine with task checklists
- [x] Bulk operations engine for mass updates
- [x] Document management per employee
- [x] Disciplinary record tracking
- [x] Probation management with confirmation workflow
- [x] Status history and audit trail
- [x] Employee analytics (headcount, attrition, demographics)
- [x] CSV/Excel bulk import with validation
- [x] Field-level access control

### Payroll Service
- [x] Payroll period management (open, process, approve, pay, close)
- [x] Multiple run types: regular, off-cycle, bonus, correction, reversal, 13th month, back pay
- [x] Multi-level approval workflow
- [x] Salary structure and component configuration
- [x] Allowance management (22 types including housing, transport, meal, utility)
- [x] Overtime calculation engine
- [x] Expense reimbursement processing
- [x] Salary advance management
- [x] Payslip generation (PDF)
- [x] Year-to-date (YTD) tracking
- [x] Variance analysis (period-over-period)
- [x] Proration engine for mid-month hires/exits
- [x] Disbursement service with multi-bank support
- [x] Webhook notifications for payment events

### Leave Service
- [x] Configurable leave types with policies
- [x] Leave request workflow (draft, pending, approved, rejected, cancelled, completed)
- [x] Half-day and hourly leave support
- [x] Delegation management during leave
- [x] Holiday calendar per location/country
- [x] Leave balance tracking and accrual
- [x] Leave reports and analytics

### Recruitment Service
- [x] Job requisition management with budget tracking
- [x] Candidate pipeline with stage management
- [x] Assessment framework
- [x] Interview scheduling
- [x] Onboarding automation
- [x] Talent pool management

### Performance Service
- [x] OKR cycles (quarterly, semi-annual, annual, custom)
- [x] Key results with progress tracking
- [x] Review cycles with configurable components
- [x] 360-degree feedback collection
- [x] Calibration sessions with 9-box grid
- [x] Competency framework management
- [x] Badge and recognition system

### Time & Attendance Service
- [x] Clock-in/out with GPS coordinates
- [x] Geofence validation with configurable radius
- [x] Anti-spoofing measures (GPS accuracy, teleport detection)
- [x] Biometric integration framework
- [x] Shift scheduling and management
- [x] Attendance reports
- [x] Auto clock-out policy enforcement

### Benefits Service
- [x] Benefits plan configuration
- [x] Enrollment management with open/special enrollment periods
- [x] Claims processing
- [x] HMO adapter integrations
- [x] Insurance management
- [x] Pension administration
- [x] EWA (Earned Wage Access) engine

### Learning Service
- [x] Course catalog management
- [x] SCORM package upload and tracking
- [x] Course category hierarchy
- [x] Certificate templates and issuance
- [x] Enrollment and completion tracking
- [x] External course integration adapter

### Infrastructure
- [x] Multi-tenant architecture (X-Tenant-ID based)
- [x] AIDD guardrails enforcement
- [x] NATS JetStream event publishing (75+ event types)
- [x] CloudEvents envelope format
- [x] RS256 JWT authentication with MFA
- [x] AES-256-GCM field-level encryption
- [x] HashiCorp Vault integration
- [x] Rate limiting (token bucket)
- [x] CORS, CSRF, security headers
- [x] Audit trail logging
- [x] Health check endpoints
- [x] Prometheus metrics
- [x] OpenTelemetry tracing
- [x] Docker multi-stage builds
- [x] Internationalization (en, fr, es, ar)

---

## Breaking Changes

As this is the initial GA release (v1.0.0), there are no breaking changes from prior versions. All APIs follow the `/v1/` prefix convention and will maintain backward compatibility through the v1 lifecycle.

---

## Known Limitations

1. **Mobile app**: Flutter mobile app is referenced in architecture but not yet shipped in this release.
2. **GraphQL**: Hasura GraphQL layer is planned but not yet configured.
3. **US/UK payroll**: Tax engines for US FICA and UK HMRC have framework stubs but are not production-complete. Nigerian payroll is fully implemented.
4. **Meilisearch**: Full-text search indexing is configured at the infrastructure level but not yet wired to all services.
5. **TimescaleDB**: Hypertable setup for attendance time-series is planned but requires manual migration activation.

---

## Upgrade Path

As this is the initial release, no upgrade path is required. For organizations migrating from legacy HRMS systems:

1. Export employee data as CSV/Excel
2. Use the bulk import engine (`employee-service`) with validation
3. Configure payroll structures and statutory rates
4. Run parallel payroll for 1-2 months before cutover

---

## Dependencies

### Runtime
- Go 1.24+
- PostgreSQL 16+
- Redis 7+
- NATS Server 2.10+ (JetStream enabled)
- HashiCorp Vault 1.15+ (optional, for encryption key management)

### Frontend
- Node.js 20 LTS
- Next.js 14.1.0

### Key Libraries
- `github.com/go-chi/chi/v5` v5.0.12
- `github.com/jackc/pgx/v5` v5.5.4
- `github.com/nats-io/nats.go` v1.33.1
- `github.com/shopspring/decimal` v1.3.1
- `github.com/golang-jwt/jwt/v5` v5.2.0
- `github.com/rs/zerolog` v1.32.0
- `github.com/redis/go-redis/v9` v9.7.3

---

## Commit History

```
9d60f73 2026-02-23 chore: isolate imported source trees as nested modules
83e6e2c 2026-02-23 feat: deep import selected source service directories for consolidation
be45b8f 2026-02-23 feat: apply consolidation merger contracts and module scaffolds
aa94fe7 2026-02-23 chore: initialize ERP-HCM
```

---

## Contributors

ERP-HCM is developed by the OpenSASE engineering team as part of the ERP product-line architecture initiative.

---

## License

Proprietary. See LICENSE file for terms.
