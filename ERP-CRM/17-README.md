# ERP-CRM

**Enterprise Customer Relationship Management Module for the OpenSASE ERP Suite**

ERP-CRM is a self-hosted, open-source CRM platform that unifies sales automation, customer support, and lead capture into a single module. Built with Rust for performance and Go for service extensibility, it replaces Salesforce, HubSpot, Zoho CRM, and Freshdesk.

```mermaid
graph LR
    subgraph "ERP-CRM Capabilities"
        CRM["Sales CRM<br/>Contacts, Deals, Pipeline"]
        HELP["Helpdesk<br/>Tickets, SLA, KB"]
        FORMS["Lead Capture<br/>Forms, Chat, Web-to-Lead"]
    end
    CRM --> HELP --> FORMS
```

## Features

- **Contact & Company Management** -- Full CRUD, custom fields, tags, lifecycle stages, deduplication
- **Sales Pipeline** -- Visual Kanban board, multi-pipeline, stage transitions, probability tracking
- **Deal Management** -- CPQ products, competitor tracking, win/loss analysis, weighted forecasting
- **Lead Scoring** -- AI-powered scoring (demographics + behavior), hot/warm/cold classification
- **Activity Tracking** -- Calls, emails, meetings, tasks linked to contacts and deals
- **Helpdesk** -- Multi-channel ticketing, SLA management, escalation, agent assignment
- **Knowledge Base** -- Categories, articles, view tracking, customer self-service
- **Form Builder** -- JSONB field definitions, web-to-lead, submission tracking
- **Live Chat** -- Real-time chat sessions, chatbot integration (planned)
- **Automation** -- Workflow rules, assignment rules, escalation rules
- **Reporting** -- Dashboard analytics, funnel metrics, forecast views
- **Territory Management** -- Geographic and segment-based territory assignment

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend Core | Rust (axum, sqlx, tokio, serde) |
| Microservices | Go (12 domain services) |
| Database | PostgreSQL 16 |
| Message Broker | NATS JetStream |
| Event Streaming | Apache Pulsar |
| Search/Logs | Quickwit |
| Web Frontend | React + Refine + Ant Design |
| Mobile | Flutter (Ferry + Riverpod) |
| Android Native | Jetpack Compose + Apollo + Hilt |
| iOS Native | SwiftUI + Apollo + TCA |
| GraphQL | Hasura (auto-generated) |
| Infrastructure | Kubernetes (Harvester HCI) |

## Quick Start

### Prerequisites

- Rust 1.75+ (`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`)
- Docker and Docker Compose
- PostgreSQL 16 (or use Docker Compose)

### Using Docker Compose

```bash
# Clone the repository
git clone https://github.com/opensase/ERP-CRM.git
cd ERP-CRM

# Copy environment file
cp .env.example .env

# Start services
docker compose up -d

# The CRM API is available at http://localhost:8081
# PostgreSQL is available at localhost:5432
# NATS is available at localhost:4222
```

### Manual Setup

```bash
# Set required environment variables
export DATABASE_URL=postgres://postgres:postgres@localhost:5432/crm
export PORT=8081

# Optional: NATS for events
export NATS_URL=nats://localhost:4222

# Run database migrations
sqlx migrate run --source ./migrations

# Build and run
cargo build --release
./target/release/opensase-crm
```

### Verify Installation

```bash
# Health check
curl http://localhost:8081/health

# Readiness check
curl http://localhost:8081/ready

# Create a contact
curl -X POST http://localhost:8081/api/v1/contacts \
  -H "Content-Type: application/json" \
  -d '{"email": "jane@example.com", "first_name": "Jane", "last_name": "Smith"}'

# List contacts
curl http://localhost:8081/api/v1/contacts

# Dashboard stats
curl http://localhost:8081/api/v1/dashboard/stats
```

## Project Structure

```
ERP-CRM/
  src/                          # Rust core (main application)
    main.rs                     # Entry point, router, HTTP handlers
    lib.rs                      # Module registry, re-exports
    config.rs                   # Environment configuration
    domain/                     # DDD domain layer
      aggregates/               # Contact, Deal aggregates
      value_objects/            # Email, Money, Phone, Address
      events/                   # Domain events
      services/                 # Lead scoring, forecasting, merge
    application/                # Use case orchestration
    ports/                      # Hexagonal architecture interfaces
    infrastructure/             # PostgreSQL, NATS implementations
  services/                     # Go microservices (12 services)
  web/                          # React frontend (Refine + AntD)
  flutter/                      # Flutter mobile app
  android/                      # Android native app
  ios/                          # iOS native app
  migrations/                   # PostgreSQL migrations
  imports/                      # Merged source modules
    crm_core/                   # Original CRM module
    crm_legacy/                 # Legacy CRM module
    support_core/               # Helpdesk/KB module
    forms_core/                 # Form builder module
  docs/                         # Internal documentation
  infrastructure/               # Kubernetes manifests
  eventing/                     # Pulsar topic configuration
  observability/                # Quickwit/log schemas
  scripts/                      # Build and test scripts
```

## Configuration

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | Yes | - | PostgreSQL connection string |
| `PORT` | No | 8081 | HTTP server port |
| `HOST` | No | 0.0.0.0 | HTTP server host |
| `DATABASE_MAX_CONNECTIONS` | No | 10 | Connection pool size |
| `NATS_URL` | No | - | NATS server URL |
| `RUST_LOG` | No | info | Log level filter |
| `SASE_API_URL` | No | - | Platform API URL |
| `SASE_API_KEY` | No | - | Platform API key |
| `SASE_TENANT_ID` | No | - | Tenant identifier |

## API Documentation

See [21-API-Documentation.md](../Documentation/ERP-CRM/21-API-Documentation.md) for complete API reference.

### Key Endpoints

```
GET    /health                    Health check
GET    /ready                     Readiness probe
GET    /metrics                   Prometheus metrics
GET    /api/v1/contacts           List contacts
POST   /api/v1/contacts           Create contact
GET    /api/v1/contacts/:id       Get contact
PUT    /api/v1/contacts/:id       Update contact
DELETE /api/v1/contacts/:id       Delete contact
GET    /api/v1/companies          List companies
POST   /api/v1/companies          Create company
GET    /api/v1/deals              List deals
POST   /api/v1/deals              Create deal
GET    /api/v1/activities         List activities
POST   /api/v1/activities         Create activity
GET    /api/v1/dashboard/stats    Dashboard statistics
```

## Testing

```bash
# Run all tests
cargo test --all-features

# Run with output
cargo test -- --nocapture

# Run specific module tests
cargo test domain::aggregates::contact
cargo test domain::value_objects::email
cargo test domain::services

# Frontend tests
cd web && npm test
```

## Contributing

See [19-CONTRIBUTING.md](../Documentation/ERP-CRM/19-CONTRIBUTING.md) for contribution guidelines.

## License

Apache License 2.0. See [LICENSE](LICENSE) for details.

## Architecture

See [04-Software-Architecture.md](../Documentation/ERP-CRM/04-Software-Architecture.md) for detailed architecture documentation.
