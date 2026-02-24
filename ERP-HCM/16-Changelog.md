# Changelog

All notable changes to ERP-HCM are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-02-23

### Added

#### Core Infrastructure
- feat: initialize ERP-HCM module with Go 1.22+ scaffold (aa94fe7)
- feat: apply consolidation merger contracts and module scaffolds (be45b8f)
- feat: deep import selected source service directories for consolidation (83e6e2c)
- chore: isolate imported source trees as nested modules (9d60f73)
- feat: multi-tenant architecture with X-Tenant-ID middleware
- feat: AIDD guardrails enforcement (autonomous, supervised, prohibited actions)
- feat: CloudEvents event publishing via NATS JetStream (75+ event types)
- feat: RS256 JWT authentication with MFA (TOTP) support
- feat: AES-256-GCM field-level encryption with HashiCorp Vault
- feat: rate limiting middleware (token bucket, per-tenant)
- feat: CORS, CSRF, security headers middleware chain
- feat: audit trail logging on all data mutations
- feat: health check endpoints (/healthz) for all services
- feat: Prometheus metrics and OpenTelemetry tracing integration
- feat: Docker multi-stage builds (Alpine, < 20MB images)
- feat: environment-based configuration (PF_* variables)

#### Employee Service
- feat(employee): complete employee lifecycle management (hire to retire)
- feat(employee): employee number auto-generation with configurable format
- feat(employee): onboarding engine with task checklists
- feat(employee): bulk operations engine for mass updates
- feat(employee): CSV/Excel bulk import with validation
- feat(employee): document management per employee
- feat(employee): disciplinary record tracking
- feat(employee): probation management with confirmation workflow
- feat(employee): status history and audit trail
- feat(employee): employee analytics (headcount, attrition, demographics)
- feat(employee): field-level access control
- feat(employee): org chart repository and API
- feat(employee): department and position management with hierarchy

#### Payroll Service
- feat(payroll): Nigerian PAYE tax calculation with graduated bands (PITA)
- feat(payroll): Consolidated Relief Allowance (CRA) computation
- feat(payroll): pension calculation (employee 8% + employer 10%)
- feat(payroll): NHF (2.5%), NHIS, NSITF statutory deductions
- feat(payroll): payroll period management (open, process, approve, pay, close)
- feat(payroll): 7 run types (regular, off-cycle, bonus, correction, reversal, 13th month, back pay)
- feat(payroll): 22 salary component types with calculation methods
- feat(payroll): multi-level approval workflow
- feat(payroll): payslip generation (PDF)
- feat(payroll): year-to-date (YTD) tracking
- feat(payroll): proration engine for mid-month hires/exits
- feat(payroll): bank file generation (NIBSS format)
- feat(payroll): Flutterwave payment integration
- feat(payroll): Remita payment integration
- feat(payroll): variance analysis (period-over-period)
- feat(payroll): EWA (Earned Wage Access) engine
- feat(payroll): salary advance management
- feat(payroll): overtime calculation engine
- feat(payroll): expense reimbursement processing
- feat(payroll): webhook notifications for payment events

#### Leave Service
- feat(leave): configurable leave types with policies
- feat(leave): leave request workflow (draft, pending, approved, rejected, cancelled)
- feat(leave): leave balance tracking and accrual
- feat(leave): half-day and hourly leave support
- feat(leave): holiday calendar management per location
- feat(leave): leave delegation management
- feat(leave): leave reports and analytics

#### Recruitment Service
- feat(recruitment): job requisition management with budget tracking
- feat(recruitment): multi-stage candidate pipeline
- feat(recruitment): AI resume parsing framework
- feat(recruitment): assessment framework
- feat(recruitment): interview scheduling
- feat(recruitment): offer management with approval chain
- feat(recruitment): talent pool management
- feat(recruitment): onboarding automation on offer acceptance

#### Performance Service
- feat(performance): OKR cycle management (quarterly, semi-annual, annual)
- feat(performance): key result progress tracking with decimal scoring
- feat(performance): review cycles with configurable components
- feat(performance): 360-degree feedback (self, manager, peer, upward)
- feat(performance): calibration workflow with 9-box grid
- feat(performance): competency framework management
- feat(performance): badge and recognition system

#### Time & Attendance Service
- feat(attendance): clock-in/out with GPS coordinates
- feat(attendance): geofence validation with Haversine distance
- feat(attendance): anti-spoofing (GPS accuracy, teleport detection, multi-device)
- feat(attendance): biometric integration framework
- feat(attendance): shift scheduling and management
- feat(attendance): auto clock-out policy enforcement
- feat(attendance): attendance reports

#### Benefits Service
- feat(benefits): benefits plan configuration
- feat(benefits): open enrollment management
- feat(benefits): claims processing
- feat(benefits): HMO adapter integrations
- feat(benefits): insurance management
- feat(benefits): pension administration
- feat(benefits): EWA engine

#### Learning Service
- feat(learning): course catalog with categories and prerequisites
- feat(learning): SCORM package support
- feat(learning): certificate templates and auto-generation
- feat(learning): enrollment and completion tracking
- feat(learning): external course integration adapter

#### Additional Services
- feat(compensation): salary bands and pay grade management
- feat(compensation): compensation cycle management
- feat(workforce): headcount planning with scenario modeling
- feat(compliance): labor law compliance engine
- feat(compliance): GDPR/NDPR data subject request handling
- feat(document): document management with digital signatures
- feat(facilities): room and desk booking
- feat(communication): internal communication hub
- feat(analytics): workforce analytics and dashboards
- feat(notification): multi-channel notification engine

#### Frontend
- feat(web): Next.js 14 application with App Router
- feat(web): Radix UI and Tailwind CSS component library
- feat(web): React Query v5 data fetching
- feat(web): Zustand state management
- feat(web): React Hook Form with Zod validation
- feat(web): Recharts data visualizations
- feat(web): TanStack Table with server-side pagination
- feat(web): NextAuth v4 authentication
- feat(web): internationalization (en, fr, es, ar)
- feat(web): Playwright E2E testing
- feat(web): 30+ page routes covering all HR functions

#### Testing
- test(payroll): Nigerian PAYE calculator unit tests
- test(payroll): NHF calculation tests
- test(payroll): pension contribution tests
- test(attendance): geofence Haversine distance tests
- test(attendance): anti-spoofing detection tests
- test(employee): field filter tests
- test(employee): number generator tests
- test(employee): onboarding engine tests
- test(payroll): integration tests with testcontainers
- test(security): encryption service tests

### Database Migrations
- migration: 47 SQL migration files across 30+ schemas
- migration(employee): legal_entities, locations, departments, cost_centers, employees
- migration(payroll): pay_grades, salary_components, payroll_runs, payroll_entries
- migration(leave): leave_types, leave_requests, leave_balances, holidays
- migration(attendance): clock_records, geofences, shifts, biometric_templates
- migration(recruitment): job_requisitions, candidates, applications, assessments
- migration(performance): okr_cycles, objectives, review_cycles, reviews
- migration(benefits): plans, enrollments, claims, ewa_transactions
- migration(lms): courses, modules, enrollments, certificates
- migration(auth): users, roles, permissions, sessions

---

## [0.1.0] - 2026-02-23 (Pre-release)

### Added
- chore: initialize ERP-HCM repository
- chore: configure module manifest and AIDD guardrails
- chore: set up capabilities.json with core feature flags
