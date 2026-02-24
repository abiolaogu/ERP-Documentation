# ERP-Commerce -- Changelog

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

All notable changes to ERP-Commerce are documented in this file. The format follows [Keep a Changelog](https://keepachangelog.com/).

---

## [2.0.0] -- 2026-02-23

### Added
- Consolidated ERP-Commerce module from three source repositories:
  - ERP-OmniRoute (commerce and logistics orchestration)
  - ERP-POS-Software-for-Physical-Storefront (point of sale)
  - ERP-opensase-ecommerce (B2B marketplace)
- 10 Go microservice scaffolds with HTTP handlers and health checks:
  - catalog-service (`/v1/catalog`)
  - order-service (`/v1/order`)
  - pricing-service (`/v1/pricing`)
  - inventory-service (`/v1/inventory`)
  - trade-credit-service (`/v1/trade-credit`)
  - distribution-service (`/v1/distribution`)
  - pos-service (`/v1/pos`)
  - portal-service (`/v1/portal`)
  - logistics-service (`/v1/logistics`)
  - marketplace-service (`/v1/marketplace`)
- API gateway entry point (`cmd/server/main.go`)
- Module manifest (`erp/module.manifest.yaml`) with standalone_plus_suite integration mode
- AIDD guardrails configuration (`erp/aidd.guardrails.yaml`)
- Capability registry (`configs/capabilities.json`)
- Merge manifest tracking consolidation sources (`merge/MERGE_MANIFEST.yaml`)
- 58 CloudEvents event topics defined in EVENTS.md
- Dockerfiles for all 10 services
- Service-level README files

### Changed
- Unified module identifier from three separate SKUs to single `erp.commerce`
- Standardized event naming convention to `erp.commerce.<entity>.<action>`
- Consolidated authentication to ERP-IAM (JWT/OIDC)
- Unified entitlement management through ERP-Platform

### Deprecated
- Previous module identifiers: `erp.omniroute`, `erp.pos`, `erp.opensase-ecommerce`
- Legacy API endpoints from source modules (maintained for backward compatibility)

### Architecture Decisions
- ADR-001: Chose Go as primary service language for performance and simplicity
- ADR-002: Chose NATS JetStream over Kafka for event streaming (lower operational overhead)
- ADR-003: Chose PostgreSQL with RLS for multi-tenant data isolation
- ADR-004: Chose Temporal for workflow orchestration over custom state machines
- ADR-005: Chose Rust for performance-critical components (EDI parsing, price calculation)

---

## [1.0.0] -- 2026-01-15 (Pre-Consolidation)

### Added
- Initial repository creation
- Placeholder README and documentation structure
- Integration with ERP-Platform control plane
