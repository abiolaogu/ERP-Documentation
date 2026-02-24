# Documentation Index -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## Module Overview

**ERP-Church-Management** is the Faith/Nonprofit vertical module within the BillyRonks ERP ecosystem. It provides comprehensive church management capabilities including member care, visitor assimilation (4Cs workflow), 6-directorate follow-up ministry, giving management, discipleship tracking, welfare case management, and 7-channel communication -- all unified within a multi-tenant microservices architecture.

**Key Differentiators**: RCCG 6-Directorate structure, Account Officers for Souls, 72-hour follow-up automation, Quarterly Shepherding KPIs, 7-channel unified communication.

**Technology Stack**: 12 Go microservices, Go API gateway, PostgreSQL 16, Redis 7, Redpanda/Kafka, React/Next.js web, Flutter mobile.

---

## Document Catalog

### Analysis & Requirements

| # | Document | Description | File |
|---|---|---|---|
| 1 | Gap Analysis | Architecture, feature, and documentation gaps | [gap-analysis.md](gap-analysis.md) |
| 2 | Product Requirements Document | Feature requirements, competitive analysis vs Planning Center/Tithe.ly/ChurchTrac/Breeze | [prd.md](prd.md) |
| 3 | Business Requirements Document | Business objectives, stakeholders, ROI, business rules | [brd.md](brd.md) |

### Architecture & Design

| # | Document | Description | File |
|---|---|---|---|
| 4 | System Architecture | C4 model, communication patterns, infrastructure | [architecture.md](architecture.md) |
| 5 | Software Architecture | Bounded contexts, service internals, event-driven design | [software-architecture.md](software-architecture.md) |
| 6 | Enterprise Architecture | ERP suite integration, TOGAF layers, governance | [enterprise-architecture.md](enterprise-architecture.md) |
| 7 | Database Schema | ER diagrams, table definitions, migration strategy | [database-schema.md](database-schema.md) |
| 8 | Workflows | 4Cs pipeline, 72-hour follow-up, directorate routing, giving, welfare | [workflows.md](workflows.md) |
| 9 | High-Level Design | System overview, component design, scalability, security | [hld.md](hld.md) |
| 10 | Low-Level Design | Algorithms, API contracts, state machines, query patterns | [lld.md](lld.md) |
| 11 | Use Cases | 24 use cases covering member lifecycle, visitor assimilation, giving, events, groups, discipleship, welfare | [use-cases.md](use-cases.md) |

### Technical Documentation

| # | Document | Description | File |
|---|---|---|---|
| 12 | Technical Write-up | Implementation details, migration strategy, performance | [technical-writeup.md](technical-writeup.md) |
| 13 | Hardware Requirements | Deployment tiers, client devices, network, storage planning | [hardware-requirements.md](hardware-requirements.md) |
| 14 | Software Requirements | Dependencies, environment configuration, compatibility | [software-requirements.md](software-requirements.md) |
| 15 | Technical Specifications | API reference, event catalog, rate limits, error codes | [technical-specifications.md](technical-specifications.md) |

### User Manuals

| # | Document | Description | File |
|---|---|---|---|
| 16 | Admin User Manual | System configuration, member/visitor management, giving, events | [user-manual-admin.md](user-manual-admin.md) |
| 17 | End User Manual | Member portal, giving history, groups, events, mobile app | [user-manual-enduser.md](user-manual-enduser.md) |
| 18 | Developer Manual | Local setup, gateway development, service development, testing | [user-manual-developer.md](user-manual-developer.md) |

### Training Manuals

| # | Document | Description | File |
|---|---|---|---|
| 19 | Admin Training Manual | 8-module curriculum with hands-on exercises | [training-manual-admin.md](training-manual-admin.md) |
| 20 | End User Training Manual | Role-based training for members, account officers, workers | [training-manual-enduser.md](training-manual-enduser.md) |
| 21 | Developer Training Manual | 6-module onboarding with coding exercises | [training-manual-developer.md](training-manual-developer.md) |
| 22 | Training Video Scripts | 12 video scripts across admin, user, and developer tracks | [training-video-scripts.md](training-video-scripts.md) |

### Operations & Release

| # | Document | Description | File |
|---|---|---|---|
| 23 | Release Notes | v1.0.0 release with all 12 services, known issues, migration notes | [release-notes.md](release-notes.md) |
| 24 | Acceptance Criteria | Testable Given-When-Then criteria per feature | [acceptance-criteria.md](acceptance-criteria.md) |
| 25 | Testing Requirements | Test pyramid, unit/integration/E2E tests, security tests | [testing-requirements-aidd.md](testing-requirements-aidd.md) |
| 26 | Deployment Guide | Docker Compose, Kubernetes, backup/recovery, monitoring | [deployment.md](deployment.md) |

### Design Assets

| # | Document | Description | File |
|---|---|---|---|
| 27 | Figma & Make.com Prompts | 14 Figma screen prompts + 5 Make.com automation prompts | [design/Figma_Make_Prompts.md](design/Figma_Make_Prompts.md) |

### Documentation Index

| # | Document | Description | File |
|---|---|---|---|
| 28 | README (this file) | Documentation navigation guide | [README.md](README.md) |

---

## Quick Reference

### 12 Microservices

| Service | Domain |
|---|---|
| member-service | Member CRUD, family units, natural groups |
| visitor-service | Registration, 72-hour follow-up, conversion |
| followup-service | Account officers, 6 directorates, activities |
| giving-service | Tithes, offerings, pledges, tax receipts |
| event-service | Events, attendance, QR/NFC check-in |
| group-service | Small groups, ministries, home fellowships |
| discipleship-service | NBC, mentorship, Sunday School |
| welfare-service | Welfare cases, benevolence fund |
| communication-service | SMS, WhatsApp, Telegram, Facebook, Email, Push, In-app |
| kpi-service | Dashboard metrics, directorate KPIs |
| volunteer-service | Scheduling, skill matching |
| facility-service | Room booking, equipment, campus management |

### 9 User Roles

`super_admin` > `admin` > `pastor` > `minister` > `HOD` > `directorate_head` > `account_officer` > `worker` > `member`

### 4Cs Assimilation Workflow

**Connect** -> **Capture** -> **Communicate** -> **Convert**

### 6 Directorates

1. 1st Timer
2. Further Follow-up
3. Counselling
4. Natural Group
5. Development & Structuring
6. Welfare & Finance
