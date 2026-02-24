# ERP-Marketing

Self-hosted marketing automation platform -- a complete replacement for HubSpot Marketing Hub, Marketo Engage, Mailchimp, and Zoho Marketing Automation.

## What It Does

ERP-Marketing provides enterprise-grade marketing automation with full data sovereignty:

- **Campaign Management** -- Multi-channel campaigns (email, SMS, push, in-app, social) with A/B/n testing, budget tracking, and AIDD guardrails
- **Email Marketing** -- Template management with HTML/plain-text, deliverability monitoring, and send-time optimization
- **Journey Builder** -- Visual orchestration with if/then branching, behavioral triggers, and goal tracking
- **Social Media** -- Publish, schedule, and monitor across LinkedIn, X, Facebook, Instagram, TikTok
- **Ads Management** -- Google, Meta, LinkedIn, TikTok Ads with audience sync and ROI tracking
- **Content CMS** -- Blog posts, landing pages, SEO tools, form builder, and digital asset management
- **Attribution** -- Multi-touch models (first/last/linear/time-decay/position/AI-custom) with revenue attribution
- **Segmentation** -- Dynamic behavioral, demographic, and firmographic segments with lookalike audiences
- **Analytics** -- Email performance, social engagement, ad ROI, conversion funnels, MQL-to-SQL tracking

## Architecture

```mermaid
flowchart TB
    subgraph "Frontend"
        SPA[React + Ant Design + Refine]
        MOBILE[Flutter / Android / iOS]
    end
    subgraph "Backend"
        API[Rust API Gateway<br/>axum + sqlx + tokio]
        SERVICES[9 Go Microservices]
    end
    subgraph "Data"
        PG[(PostgreSQL 16)]
        PULSAR[Apache Pulsar]
        QW[Quickwit]
    end
    SPA & MOBILE --> API
    API --> SERVICES
    API & SERVICES --> PG
    SERVICES --> PULSAR --> QW
```

## Tech Stack

| Layer | Technology |
|---|---|
| API Gateway | Rust 1.75 (axum, sqlx, tokio, serde, chrono, uuid) |
| Domain Services | Go 1.22 (net/http) |
| Frontend | React 18 + Ant Design 5 + Refine 4 + TanStack Query 5 |
| Database | PostgreSQL 16 |
| Events | Apache Pulsar |
| Observability | Quickwit |
| Infrastructure | Kubernetes on Harvester HCI (Mayastor/Vitastor storage) |

## Quick Start

### Prerequisites

- Rust 1.75+
- Docker and Docker Compose
- Node.js 20+

### Run Locally

```bash
# Start PostgreSQL and services
docker compose up -d

# Run the Rust API server
cargo run

# Start the web frontend (in another terminal)
cd web && npm install && npm run dev
```

The API server starts at `http://localhost:8086` and the frontend at `http://localhost:5173`.

### Environment Variables

```bash
DATABASE_URL=postgres://postgres:postgres@localhost:5432/marketing
RUST_LOG=info
PORT=8086
```

See `.env.example` for the complete list including Pulsar and Quickwit configuration.

## API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/health` | Health check |
| GET/POST | `/api/v1/campaigns` | List / create campaigns |
| GET | `/api/v1/campaigns/:id` | Get campaign |
| POST | `/api/v1/campaigns/:id/send` | Send campaign |
| POST | `/api/v1/campaigns/:id/launch` | Launch with AIDD guardrail |
| GET/POST | `/api/v1/audiences` | List / create audiences |
| GET/POST | `/api/v1/templates` | List / create email templates |
| GET/POST | `/api/v1/contacts` | List / create contacts |
| GET | `/api/v1/journeys` | List journeys |
| GET | `/api/v1/segments` | List segments |
| GET | `/api/v1/dashboard/summary` | Dashboard KPIs |
| GET | `/api/v1/dashboard/attribution` | Attribution by channel |
| GET | `/api/v1/recommendations` | AI recommendations |
| GET | `/api/v1/audit/guardrails` | Guardrail audit log |

## Project Structure

```
ERP-Marketing/
  src/
    main.rs                    # axum HTTP server + API handlers
    lib.rs                     # Public module exports
    domain/
      aggregates/              # Campaign, Automation aggregates (DDD)
      value_objects/            # CampaignType, Segment, FilterCondition
      events/                  # DomainEvent, CampaignEvent, AutomationEvent
  services/
    campaign-service/          # Go campaign microservice
    email-marketing-service/   # Go email microservice
    journey-service/           # Go journey microservice
    social-service/            # Go social microservice
    ads-service/               # Go ads microservice
    content-service/           # Go content microservice
    attribution-service/       # Go attribution microservice
    segment-service/           # Go segment microservice
    analytics-service/         # Go analytics microservice
  web/                         # React + Ant Design + Refine SPA
  flutter/                     # Flutter mobile app
  android/                     # Android (Kotlin/Compose) app
  ios/                         # iOS (SwiftUI) app
  migrations/                  # PostgreSQL migrations
  eventing/pulsar/             # Pulsar topic configuration
  observability/               # Quickwit log schema + index config
  erp/                         # Module manifest + AIDD guardrails
  docs/                        # Architecture and API documentation
```

## Testing

```bash
# Run Rust tests
cargo test --all-features

# Run frontend tests
cd web && npm test

# Run all tests
make test:all
```

## Contributing

See [CONTRIBUTING.md](../CONTRIBUTING.md). All PRs require passing lint/format, unit/integration tests, docs validation, and dependency/security scan.

## License

Apache License 2.0
