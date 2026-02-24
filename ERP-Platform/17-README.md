# ERP-Platform

> Unified control plane for the 20-module ERP suite. Business at the Speed of Prompt.

**Document ID:** ERP-PLAT-README-001 | **Version:** 1.0.0 | **Last Updated:** 2026-02-23

---

## Overview

ERP-Platform is the central nervous system of a comprehensive 20-module Enterprise Resource Planning suite. It provides:

- **Product Catalog**: Manages all ERP module SKUs, bundles, and categories
- **Subscription Service**: Handles single, bundle, and suite subscriptions with automatic bundle resolution
- **Tenant Provisioning**: Single API call provisions an entire tenant ("Business at the Speed of Prompt")
- **Entitlement Engine**: Runtime access control based on subscription
- **Module Registry**: Auto-discovery of all ERP modules via health checks
- **Marketplace**: Third-party module installation and lifecycle management
- **Audit Service**: Platform-wide immutable audit logging (45+ event topics)
- **Notification Hub**: Multi-channel notification dispatch (email, SMS, push, in-app)
- **Web Hosting**: Domain, SSL, and CDN management
- **Activation Wizard**: Guided tenant onboarding

---

## Quick Start

### Prerequisites

- Go 1.22+
- Docker and Docker Compose 2.0+
- (Optional) Node.js 20+ for admin console

### Run with Docker Compose

```bash
cd infra/
docker-compose -f docker-compose.platform.yml up
```

### Run Subscription Hub Locally

```bash
cd services/subscription-hub
go run .
```

### Verify

```bash
# Health check
curl http://localhost:8091/healthz | jq

# Product catalog
curl http://localhost:8091/v1/products | jq

# Create a subscription
curl -X POST http://localhost:8091/v1/subscriptions \
  -H "Content-Type: application/json" \
  -d '{"tenant_id":"demo","plan_type":"bundle","skus":["starter"]}'

# Query entitlements
curl http://localhost:8091/v1/entitlements/demo | jq
```

---

## Architecture Overview

```
ERP-Platform/
  catalog/           # Product catalog (products.json, entitlement-templates.json)
  docs/              # Architecture docs, ADRs, API specs, event topics
  imports/           # Imported source from legacy repos (BAC activation)
  infra/             # Docker Compose, Kubernetes manifests
  merge/             # Merge manifest and consolidation maps
  scripts/           # Utility scripts
  services/          # 9 Go microservices
    activation-wizard/
    audit-service/
    entitlement-engine/
    marketplace/
    module-registry/
    notification-hub/
    subscription-hub/    # Primary service (port 8091)
    tenant-provisioner/
    web-hosting/
  tools/             # Documentation generator
```

### Services

| Service | Port | Description |
|---------|------|-------------|
| subscription-hub | 8091 | Product catalog, subscriptions, entitlements |
| tenant-provisioner | 8080 | Tenant lifecycle management |
| entitlement-engine | 8080 | Runtime entitlement evaluation |
| module-registry | 8080 | Module discovery and health monitoring |
| marketplace | 8080 | Third-party module management |
| audit-service | 8080 | Immutable audit logging |
| notification-hub | 8080 | Multi-channel notifications |
| web-hosting | 8080 | Domain, SSL, CDN management |
| activation-wizard | 8080 | Guided onboarding |

---

## API Reference

### Core Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/healthz` | Health check |
| GET | `/v1/products` | List all products and bundles |
| POST | `/v1/subscriptions` | Create subscription |
| GET | `/v1/subscriptions/{tenant_id}` | Get tenant subscription |
| GET | `/v1/entitlements/{tenant_id}` | Get tenant entitlements |

### Service Endpoints

Each service exposes CRUD endpoints under `/v1/{service-name}` with `X-Tenant-ID` header required.

For full API documentation, see [21-API-Documentation.md](./21-API-Documentation.md).

---

## Product Catalog

20 standalone modules across 8 categories:

| Category | Modules |
|----------|---------|
| Core | ERP Platform |
| Business Ops | HCM, CRM, Marketing, Finance, Commerce, eCommerce, SCM, Projects |
| Productivity | Workspace |
| Intelligence | BI, AI |
| Security | IAM |
| Integration | iPaaS |
| Industry Verticals | Healthcare, School Management, Church Management, BSS-OSS |
| Platform Tools | Assistant, Autonomous Coding |

8 curated bundles: Starter (3), Professional (11), Enterprise (20), Healthcare Suite (5), Education Suite (5), Faith Suite (4), Telecom Suite (5), Developer Suite (4).

---

## Development Setup

See [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md) for detailed instructions.

```bash
# Clone
git clone <repo-url> ERP-Platform
cd ERP-Platform

# Start infrastructure
docker-compose -f infra/docker-compose.platform.yml up -d postgres redis nats

# Run a service
cd services/subscription-hub && go run .
```

---

## Testing

```bash
# Unit tests
make test

# Integration tests
make test-integration

# End-to-end tests
make test-e2e
```

---

## Deployment

- **Docker**: Multi-stage builds produce minimal Alpine images (< 25MB)
- **Kubernetes**: Helm charts for production deployment
- **CI/CD**: GitHub Actions pipeline

See [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md) for pipeline details.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Go 1.22, stdlib net/http |
| Database | PostgreSQL 16 |
| Cache | Redis 7 |
| Events | NATS 2.10 JetStream |
| Frontend | Next.js 14, React 18, TypeScript |
| Container | Docker (Alpine 3.20) |
| Orchestration | Kubernetes |
| CI/CD | GitHub Actions |

---

## Contributing

See [19-CONTRIBUTING.md](./19-CONTRIBUTING.md) for contribution guidelines.

---

## Documentation Suite

| # | Document | Description |
|---|----------|-------------|
| 01 | [Technical Writeup](./01-Technical-Writeup.md) | Comprehensive technical overview |
| 02 | [Release Notes](./02-Release-Notes.md) | v1.0.0 release notes |
| 03 | [Enterprise Architecture](./03-Enterprise-Architecture.md) | TOGAF-aligned EA document |
| 04 | [Software Architecture](./04-Software-Architecture.md) | C4 model architecture |
| 05 | [PRD](./05-Product-Requirements-Document.md) | Product requirements |
| 06 | [BRD](./06-Business-Requirements-Document.md) | Business requirements |
| 07 | [User Manual](./07-User-Manual.md) | End-user manual |
| 08 | [Training Manual](./08-Training-Manual.md) | 10-module training curriculum |
| 09 | [Video Training Scripts](./09-Video-Training-Script.md) | 8 video scripts |
| 10 | [Use Cases](./10-Use-Cases.md) | 16 detailed use cases |
| 11 | [Workflow Diagrams](./11-Workflow-Diagrams.md) | Mermaid workflow diagrams |
| 12 | [High-Level Design](./12-High-Level-Design.md) | HLD document |
| 13 | [Low-Level Design](./13-Low-Level-Design.md) | LLD document |
| 14 | [Technical Specs](./14-Technical-Specifications.md) | API contracts, schemas |
| 15 | [Figma Prompts](./15-Figma-Design-Prompts.md) | 21 UI design prompts |
| 16 | [Changelog](./16-Changelog.md) | Conventional commits log |
| 17 | README (this file) | Project overview |
| 18 | [ADRs](./18-Architecture-Decision-Records.md) | Architecture decisions |
| 19 | [Contributing](./19-CONTRIBUTING.md) | Contribution guide |
| 20 | [Local Setup](./20-Local-Environment-Setup.md) | Development environment |
| 21 | [API Docs](./21-API-Documentation.md) | OpenAPI 3.1 spec |
| 22 | [Postman Collection](./22-Postman-Collection.md) | Postman collection |
| 23 | [Webhooks](./23-Webhook-Specifications.md) | Webhook specs |
| 24 | [Runbooks](./24-Runbooks.md) | Operational runbooks |
| 25 | [Deployment Pipeline](./25-Deployment-Pipeline.md) | CI/CD documentation |
| 26 | [DR Plan](./26-Disaster-Recovery-Plan.md) | Disaster recovery |
| 27 | [ERD](./27-Entity-Relationship-Diagram.md) | Entity relationship diagrams |
| 28 | [Data Dictionary](./28-Data-Dictionary.md) | Complete data dictionary |
| 29 | [Privacy & Compliance](./29-Data-Privacy-Compliance.md) | GDPR, SOC 2 compliance |
| 30 | [Test Strategy](./30-Test-Strategy.md) | Testing strategy |
| 31 | [Security](./31-SECURITY.md) | Security policy |
| 32 | [Dependencies](./32-Dependency-Manifest.md) | Dependency inventory |

---

## License

Proprietary. Copyright 2026. All rights reserved.
