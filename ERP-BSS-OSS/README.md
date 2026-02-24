# Documentation Index -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## ERP-BSS-OSS: Telecom & Utilities Business/Operations Support Systems

A carrier-grade, TM Forum-compliant BSS/OSS platform built with Rust, supporting convergent billing, real-time charging, customer lifecycle management, network operations, and utilities metering for telecommunications operators, MVNOs, ISPs, and utility providers.

---

## Document Catalog

### Analysis & Requirements (3 documents)

| Document | Description | Audience |
|----------|-------------|----------|
| [Gap Analysis](gap-analysis.md) | Current vs target state assessment | Engineering, Product |
| [Product Requirements (PRD)](prd.md) | Full product specification with competitive analysis vs Amdocs/Netcracker/Oracle/Ericsson/CSG | Product, Engineering, Executive |
| [Business Requirements (BRD)](brd.md) | Business process requirements, financial rules, regulatory needs | Business, Finance, Compliance |

### Architecture & Design (8 documents)

| Document | Description | Audience |
|----------|-------------|----------|
| [System Architecture](architecture.md) | C4 model, service inventory, data architecture, security | Engineering, Architecture |
| [Software Architecture](software-architecture.md) | DDD, hexagonal architecture, CQRS, module structure | Engineering |
| [Enterprise Architecture](enterprise-architecture.md) | ERP suite integration, eTOM alignment, capacity planning | Architecture, Executive |
| [Database Schema & ERD](database-schema.md) | Complete ERD, all PostgreSQL/ClickHouse/Redis schemas | Engineering, DBA |
| [Workflows](workflows.md) | Subscriber lifecycle, O2A, billing cycle, settlement flows | All |
| [High-Level Design](hld.md) | Component architecture, design decisions, deployment model | Engineering, Architecture |
| [Low-Level Design](lld.md) | OCS algorithm, rating pipeline, CDR processing, STS tokens | Engineering |
| [Use Cases](use-cases.md) | 28 use cases covering telecom + utilities lifecycle | Product, QA, Engineering |

### Technical Documentation (4 documents)

| Document | Description | Audience |
|----------|-------------|----------|
| [Technical Write-Up](technical-writeup.md) | Engineering decisions, performance optimization, observability | Engineering |
| [Technical Specifications](technical-specifications.md) | API specs, event specs, protocol specs, configuration | Engineering, Integration |
| [Hardware Requirements](hardware-requirements.md) | Compute, storage, network sizing for 4 subscriber tiers | Infrastructure, Operations |
| [Software Requirements](software-requirements.md) | Dependencies, compatibility matrix, security audit status | Engineering, Operations |

### User & Training Manuals (7 documents)

| Document | Description | Audience |
|----------|-------------|----------|
| [Admin User Manual](user-manual-admin.md) | Admin console guide for BSS administrators | Admins |
| [End User Manual](user-manual-enduser.md) | Self-care portal guide for subscribers | End Users |
| [Developer User Manual](user-manual-developer.md) | Development environment setup, API guide, contributing | Developers |
| [Admin Training Manual](training-manual-admin.md) | 40-hour training program for administrators | Admins |
| [End User Training Manual](training-manual-enduser.md) | 2-hour self-paced training for subscribers | End Users |
| [Developer Training Manual](training-manual-developer.md) | 80-hour training program for developers | Developers |
| [Training Video Scripts](training-video-scripts.md) | 12 video scripts covering all aspects of the platform | All |

### Operations & Release (5 documents)

| Document | Description | Audience |
|----------|-------------|----------|
| [Release Notes](release-notes.md) | Release 1.0.0 features, benchmarks, known issues | All |
| [Acceptance Criteria](acceptance-criteria.md) | Acceptance criteria for all service domains | QA, Product |
| [Testing Requirements](testing-requirements-aidd.md) | Test strategy, unit/integration/load/security test plans | QA, Engineering |
| [Deployment Guide](deployment.md) | Local, cloud K8s, and on-premises deployment procedures | DevOps, Infrastructure |
| [Changelog](CHANGELOG.md) | Version history and planned changes | All |

### Compliance & Governance (1 document)

| Document | Description | Audience |
|----------|-------------|----------|
| [Compliance & Regulatory](compliance.md) | TM Forum, telecom regulations, GDPR, SOC 2, PCI DSS | Compliance, Legal, Executive |

### Design Assets (1 document)

| Document | Description | Audience |
|----------|-------------|----------|
| [Figma & Make.com Prompts](design/Figma_Make_Prompts.md) | 8 Figma design prompts + 4 automation workflows | Design, Product |

### Architecture Decision Records (2 documents)

| Document | Description | Audience |
|----------|-------------|----------|
| [ADR-001: Rust Selection](ADR/001-rust-language-selection.md) | Why Rust for telecom BSS/OSS | Engineering, Architecture |
| [ADR-002: Polyglot Persistence](ADR/002-polyglot-persistence.md) | Why 5 database technologies | Engineering, Architecture |

---

## Quick Reference

### Services (30 microservices)

**BSS Core:** product-catalog-service, customer-management-service, order-management-service, billing-rating-service, charging-engine, mediation-service, partner-service, self-care-service

**OSS Core:** provisioning-service, resource-inventory-service, service-inventory-service, network-operations-service, fault-management, performance-mgmt, workforce-mgmt, network-orchestration, network-inventory

**Specialized:** revenue-assurance-service, ussd-ivr-gateway, meter-management-service, tariff-service

**Foundation:** api-gateway, crm-service, finance-service, support-service, customer-management, order-management, product-catalog, partner-settlement, service-provisioning

### Technology Stack

| Component | Technology |
|-----------|-----------|
| Language | Rust 1.83, Go 1.22 |
| Web Framework | Axum 0.7, Tower |
| Database (OLTP) | PostgreSQL 16 |
| Database (Analytics) | ClickHouse |
| Cache | Redis 7 |
| Document Store | MongoDB 7 |
| Event Streaming | Apache Kafka 3.6 |
| Orchestration | Kubernetes 1.29 |
| Observability | OpenTelemetry, Prometheus, Grafana, Jaeger |

### TM Forum APIs

TMF620 (Product Catalog), TMF622 (Order Management), TMF629 (Customer Management), TMF638 (Service Inventory), TMF639 (Resource Inventory), TMF641 (Service Provisioning), TMF656 (Fault Management), TMF668 (Partnership Management), TMF678 (Customer Bill Management)

### Event Topics (84 topics)

Convention: `erp.bss_oss.<entity>.<action>`

---

## Source Repository

Source code: `/Users/AbiolaOgunsakin1/ERP/ERP-BSS-OSS/`
Documentation: `/Users/AbiolaOgunsakin1/ERP/Documentation/ERP-BSS-OSS/`
