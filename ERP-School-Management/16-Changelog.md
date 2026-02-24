# ERP-School-Management -- Changelog

**Product:** EduCore Pro
**Module:** ERP-School-Management

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Planned
- Library management module
- Transport route optimization with live GPS
- Parent-teacher conference scheduling
- Hostel/dormitory management
- Full GraphQL API layer
- Advanced AI: dropout prediction, learning path recommendation
- WhatsApp Business API integration
- Multi-campus federation
- Offline-first mobile with conflict resolution

---

## [1.0.0] -- 2026-02-23

### Added

#### Core Platform
- 25 microservices architecture (NestJS, Go, Rust, Python)
- 5 frontend applications (Next.js 14, Flutter x4)
- ERP Gateway (NestJS) at port 8090
- LumaDB (PostgreSQL 16) unified data platform
- Redpanda event streaming backbone
- OpenTelemetry distributed tracing
- Turborepo monorepo build orchestration
- Docker Compose configuration for all services

#### Academic Management (academic-service)
- Multi-curriculum support: WAEC, NECO, KCPE, KCSE, ZIMSEC, Cambridge IGCSE/AS/A-Level, IB PYP/MYP/DP, Common Core, AP, GCSE, A-Level, custom
- Flexible grading scales: percentage, letter, GPA, points, descriptive
- Academic year management with term/semester/quarter/trimester support
- Subject management with credit hours and curriculum mapping
- Class management with capacity tracking and student assignment
- Timetable generation with day/period/room/teacher assignment
- 15+ assessment types with weighted scoring
- Student grade workflow: Draft > Submitted > Published > Locked
- Term summary with class rank and teacher comments

#### Student Information System (student-service)
- Student registration with comprehensive profiles
- Medical information tracking (blood type, conditions, medications, allergies)
- Dietary profile management with 14 restriction types
- Guardian management with relationship types and emergency contacts
- Enrollment lifecycle management
- Incident management with severity tracking and investigation workflow
- Academic progress tracking with GPA and cumulative calculations
- Transcript generation (official/unofficial)

#### Finance Management (finance-service)
- 13 fee types (tuition, registration, examination, laboratory, library, sports, transportation, uniform, books, feeding, excursion, PTA, development)
- Fee structure configuration by school, academic year, class, grade level
- Invoice generation with itemized breakdown
- 11 payment methods (cash, bank transfer, card, mobile money, check, wallet, scholarship, waiver, Stripe, Paystack, Flutterwave)
- Installment payment plans with automatic scheduling
- Automated payment reminders (upcoming, due, overdue, final notice)
- 8 discount types (sibling, scholarship, early payment, staff child, alumni child, promotional, hardship, other)
- Vendor management with contacts and receivables tracking
- Asset management with maintenance scheduling and task tracking
- School feeding subscription and wallet system
- Financial aid: scholarships, bursaries with merit/sibling/need criteria

#### Learning Management System (lms-service)
- Geo-partitioned course content (US, EU, APAC, LATAM, MEA)
- Course creation with modules and lessons
- 6 lesson types: video, text, quiz, interactive, assignment, live session
- Enrollment progress tracking with completion percentage
- Certificate generation with unique certificate numbers
- Organization-based B2B LMS support
- Course pricing with multi-currency support

#### Authentication & Security (auth-service)
- Email/password authentication with bcrypt hashing
- Multi-factor authentication (TOTP with backup codes)
- OAuth2 social login (Google, Microsoft, Facebook)
- Session management with device tracking and geolocation
- JWT access tokens (15 min expiry) + refresh tokens (7 day)
- Account lockout (5 failed attempts = 30 min lock)
- 12 user roles with RBAC
- Biometric attendance (fingerprint, facial recognition)
- Exeat request management
- Marketplace listing (student marketplace)
- Carpool profile management

#### Communication (communication-service, notification-service)
- Direct messaging with threading and attachments
- Announcement system with audience targeting and grade filtering
- Multi-channel notifications: SMS, email, push, in-app
- Communication preference management
- Priority-based message routing

#### Innovation Services
- **AI Service** (Python): Predictive analytics, performance modeling
- **Blockchain Service**: Certificate issuance with IPFS storage, on-chain verification
- **Gamification Service**: Badges, points, leaderboards, achievements
- **IoT Service**: Smart campus sensor integration, environmental monitoring
- **AIOps Service**: System monitoring, anomaly detection
- **Analytics Service**: Reporting, dashboards, data aggregation
- **Placement Service** (Rust): Career placement, job matching, internships
- **Scholarship Service** (Go): Scholarship management, financial aid
- **Research Service** (Rust): Research data management, paper tracking

#### Infrastructure
- Kubernetes manifests (infra/k8s/)
- Grafana dashboard configurations (infra/grafana/)
- OpenTelemetry collector config (infra/otel/)
- Apache Superset setup (infra/superset/)
- Terraform/IaC templates (infra/infrastructure/)

#### Database
- Initial schema migration (001_initial_schema.sql) with 27+ tables
- PostgreSQL extensions: uuid-ossp, pg_trgm, btree_gin, btree_gist
- Audit trigger function for automatic change tracking
- Updated_at trigger for all tables
- Current enrollments view
- Demo seed data (2 sample schools)

#### Shared Packages
- @educore/config: Shared configuration management
- @educore/database: Database utilities
- @educore/logger: Structured logging library
- @educore/superset-adapter: BI integration
- @educore/types: Shared TypeScript type definitions
- @educore/ui: Shared UI components
- @educore/utils: Common utility functions

#### Protobuf Contracts
- EventEnvelope: Base event wrapper with CloudEvents format
- ProgressEvent: Learning progress tracking
- SessionEvent: Session lifecycle management
- AssessmentEvent: Assessment results and grading
- EngagementEvent: User interaction tracking
- DeviceInfo: Device context for analytics

#### Documentation
- Architecture documentation (docs/ARCHITECTURE.md)
- Database schema documentation (docs/DATABASE.md)
- API documentation (docs/API.md)
- Event specification (docs/EVENTS.md)
- Deployment guide (docs/DEPLOYMENT.md)
- Integration map (docs/INTEGRATION_MAP.md)
- Architecture Decision Records (docs/ADR/)

### Changed
- N/A (initial release)

### Deprecated
- Legacy Payment model in finance-service (replaced by FeePayment)

### Removed
- N/A (initial release)

### Fixed
- N/A (initial release)

### Security
- JWT authentication with configurable expiry
- MFA enforcement per school
- CORS whitelist configuration
- Audit logging with IP tracking
- AIDD governance guardrails

---

## Commit Log

| Hash | Date | Author | Message |
|---|---|---|---|
| `dd07bd6` | 2026-02-23 | EduCore Team | feat: integrate school management sources and nest gateway scaffold |
| `e6e68c6` | 2026-02-23 | EduCore Team | chore: isolate imported source trees as nested modules |
| `67129e2` | 2026-02-23 | EduCore Team | feat: deep import selected source service directories for consolidation |
| `b6ce87b` | 2026-02-23 | EduCore Team | feat: apply consolidation merger contracts and module scaffolds |
| `c2638e3` | 2026-02-23 | EduCore Team | chore: initialize ERP-School-Management |
