# Changelog -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

All notable changes to the ERP-BSS-OSS platform are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-02-23

### Added

#### BSS Services
- Product Catalog Service (TMF620) -- full lifecycle management with 4 pricing models
- Customer Management Service (TMF629) -- KYC, Customer 360, segmentation
- Order Management Service (TMF622) -- state machine with fallout management
- Billing and Rating Service (TMF678) -- convergent billing, OCS, batch rating
- Charging Engine -- real-time balance management with sub-ms latency
- Mediation Service -- CDR collection, normalization, deduplication at 1.4M/sec
- Partner Service (TMF668) -- MVNO/MVNE, revenue sharing, settlement
- Self-Care Service -- subscriber portal for account management

#### OSS Services
- Provisioning Service (TMF641) -- SIM swap, number porting, eSIM
- Resource Inventory Service (TMF639) -- network elements, SIM/number/IP/device inventory
- Service Inventory Service (TMF638) -- active services, dependencies, health
- Network Operations Service -- fault management, performance monitoring, SLA tracking
- Workforce Management -- field dispatch

#### Specialized Services
- Revenue Assurance Service -- AI leakage detection, fraud detection (SIM box, IRSF, Wangiri)
- USSD/IVR Gateway -- session management, shortcode routing, USSD-to-API bridge
- Meter Management Service -- smart meter, AMI, tamper detection
- Tariff Service -- time-of-use, tiered, demand charges, STS tokens (IEC 62055-41)

#### Infrastructure
- 9 Rust crates (bss-core, bss-api, bss-billing, bss-crm, bss-ordering, bss-inventory, bss-analytics, bss-integration, bss-ddd)
- Docker Compose local development environment
- Kubernetes deployment manifests and Helm charts
- 6-PoP global deployment topology
- Full observability stack (OpenTelemetry, Prometheus, Grafana, Jaeger)
- Multi-tenant architecture with X-Tenant-ID isolation
- Event-driven architecture with 84 Kafka event topics

#### Database
- PostgreSQL 16 schema with 30+ tables
- Redis caching layer with defined key patterns
- ClickHouse analytics schema with 10 OLAP tables
- MongoDB audit log collections
- Database migration strategy (PostgreSQL -> YugabyteDB)

### Performance
- API P99 latency: 12ms (target: < 50ms)
- Throughput: 167K TPS (target: 150K)
- CDR mediation: 1.47M/sec (target: 1.4M)
- Balance lookup: 0.3ms (target: < 1ms)
- Availability: 99.99% (target: 99.99%)

---

## [Unreleased]

### Planned for 1.1.0
- Full Rust implementation of all Go microservice stubs
- TMF API conformance certification
- DIAMETER/RADIUS protocol adapters
- GraphQL schema expansion for all telecom entities
- Enhanced ML fraud detection models
- Flutter mobile apps (iOS + Android)
- Number porting integration with live NPC
- eSIM SM-DP+ integration

### Planned for 1.2.0
- AI-driven churn prediction and next-best-action
- Advanced analytics dashboards
- Multi-currency real-time FX
- Roaming TAP/RAP file processing
- Net metering for solar/DER customers
