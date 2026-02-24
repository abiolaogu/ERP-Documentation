# ERP-CRM Changelog

All notable changes to the ERP-CRM module are documented in this file. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned
- Contact import/export (CSV)
- Data enrichment integration (Clearbit/ZoomInfo)
- Email integration (IMAP/SMTP)
- Calendar integration (CalDAV)
- Advanced reporting with drag-and-drop builder
- Customer self-service portal
- CSAT surveys
- Chatbot AI responses

---

## [0.1.0] - 2026-02-23

### Added

#### Core CRM (Rust)
- Contact CRUD with email validation, lead scoring, lifecycle stages, and custom fields
- Company CRUD with domain, industry, size, website, address, and custom fields
- Deal CRUD with multi-pipeline support, stage transitions, probability tracking, and close dates
- Activity CRUD with types (call, email, meeting, task) linked to contacts, companies, and deals
- Dashboard statistics endpoint (total contacts, companies, deals, pipeline value)
- Health check, readiness probe, and Prometheus metrics endpoints
- Pagination support (page, per_page, search, sort) for all list endpoints
- NATS JetStream integration for domain event publishing (optional)
- CloudEvents-compatible event format for all state changes
- Input validation via `validator` crate with email and length rules

#### Domain-Driven Design
- Contact aggregate with rich business logic (qualify, disqualify, convert, score, merge)
- Deal aggregate with stage management, CPQ products, competitor tracking, weighted value
- Value objects: Email, Money, Currency, Phone, Address, EntityId
- Domain events: ContactEvent (6 types), DealEvent (5 types), AccountEvent (3 types)
- Domain services: LeadScoringService, ForecastService, ContactMergeService
- Hexagonal architecture with inbound/outbound port interfaces
- Application layer with ContactService and DealService orchestration

#### Microservices (Go)
- 12 Go microservices: contact, lead, pipeline, opportunity, activity, helpdesk, knowledge-base, form-builder, chat, automation, reporting, territory
- Standard CRUD pattern with tenant isolation via X-Tenant-ID header
- CloudEvents topic emission for all operations (60+ event topics)
- Health check endpoint (/healthz) per service
- Individual Dockerfiles for each microservice

#### Helpdesk (from opensase-support)
- Ticket aggregate with lifecycle (New, Open, Pending, OnHold, Solved, Closed)
- Ticket assignment with auto-status transition
- Public and internal comments with first-response tracking
- SLA policy attachment with breach time tracking
- Priority levels (Low, Normal, High, Urgent) with escalation
- Knowledge base categories and articles with view tracking
- Ticket comments schema with author types and visibility

#### Form Builder (from opensase-forms)
- Form creation with JSONB field definitions
- Form submission tracking with metadata
- Active/inactive form status management
- Slug-based URLs for form embedding

#### Frontend Scaffolding
- React web app (Refine + Ant Design + React Query) with GraphQL codegen
- Flutter mobile app (Ferry + Riverpod + GoRouter) with GraphQL operations
- Android native app (Jetpack Compose + Apollo + Hilt + Orbit) with tests
- iOS native app (SwiftUI + Apollo + TCA) with tests
- GraphQL schema with users, organizations, and projects types
- Code generation scripts for all platforms

#### Infrastructure
- Docker Compose for local development (PostgreSQL 16 + NATS 2.10)
- Multi-stage Dockerfile producing minimal Debian slim production images
- GitHub Actions CI with test, build, and Docker publish jobs
- Cargo caching for faster CI builds
- Kubernetes storage baseline for Harvester HCI (Mayastor/Vitastor)
- Apache Pulsar topic configuration (command, event, audit, observability)
- Quickwit index configuration for log search
- Observability log schema definition

#### Documentation
- ARCHITECTURE.md with data flow diagram
- CHANGELOG.md with version history
- COMPLIANCE.md with SOC2/HIPAA/PCI-DSS control mapping
- CONTRIBUTING.md with contribution guidelines
- PERFORMANCE.md with deep audit findings
- PLAN.md with engineering roadmap
- RUNBOOK.md placeholder
- SECURITY.md placeholder
- API_SPEC.yaml with OpenAPI 3.0 health and events endpoints
- ADR-0001: Sovereign baseline architecture decision

#### Tests
- 7 contact aggregate unit tests (creation, events, qualify, convert, score, tags)
- 6 deal aggregate unit tests (creation, stage move, close won/lost, modification guard, weighted value)
- 6 email value object unit tests (valid, lowercase, trim, empty, no-at, no-domain)
- 5 money value object unit tests (creation, from-cents, add, currency-mismatch, multiply)
- 2 domain service unit tests (lead scoring, weighted pipeline)
- 1 ticket aggregate integration test (ticket workflow)
- Frontend test scaffolding (React App.test.tsx, Android ProjectsViewModelTest.kt, iOS ProjectsFeatureTests.swift)

### Changed
- Nothing (initial release)

### Deprecated
- Nothing (initial release)

### Removed
- Nothing (initial release)

### Fixed
- Nothing (initial release)

### Security
- Non-root container execution in production Docker image
- Compile-time SQL injection prevention via sqlx
- JWT-based authentication via ERP-IAM
- Tenant isolation via X-Tenant-ID header validation
- CORS middleware (currently permissive, to be tightened)

---

## [Pre-release] - 2026-02-18 to 2026-02-22

### 2026-02-19 - Frontend Stack Standardization
- Added/aligned web (Refine + AntD + React Query), Flutter (Ferry + Riverpod + GoRouter), Android (Compose + Apollo + Hilt + Orbit), and iOS (SwiftUI + Apollo + TCA) scaffolds and workflows

### 2026-02-18 - Frontend Stack Scaffolding
- Added unified frontend stack scaffolding for web, flutter, android, and ios
- Added GraphQL schema/codegen scripts and CI workflows for metadata/schema triggers
- Added docs for architecture and code generation workflow

### 2026-02-20 - Sovereign Standardization
- Pulsar event boundary standardization
- Quickwit observability integration
- Harvester HCI storage class alignment
- Performance deep audit (risk score: 25/low)
- ADR-0001: Sovereign baseline
