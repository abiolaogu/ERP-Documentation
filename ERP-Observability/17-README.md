# ERP-Observability

> **Document ID:** ERP-OBS-RM-017
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md), [21-API-Documentation.md](./21-API-Documentation.md)

---

## Overview

ERP-Observability is the unified observability platform for the ERP Cloud suite, providing centralized log aggregation, metric collection, distributed tracing, full-text search, alerting, and visualization across all 20 ERP modules. It is designed for multi-tenant enterprise deployments with strict data isolation, AIDD governance compliance, and operational excellence.

### Core Components

| Component | Role | Port |
|---|---|---|
| **Go Gateway** | API entry point, reverse proxy, CORS, security headers | 8090 |
| **Node.js Observability API** | REST API for logs, metrics, traces, alerts, dashboards, search | 3000 |
| **Go Tenant API** | Tenant lifecycle, retention config, usage metering | 8080 |
| **VictoriaMetrics** | Long-term metrics storage (PromQL compatible) | 8428 |
| **Quickwit** | Log/trace/audit/search indexing and querying | 7280 |
| **Grafana** | Visualization and dashboarding | 3001 |
| **OTel Collector** | Unified telemetry pipeline (OTLP gRPC/HTTP) | 4317/4318 |
| **Alertmanager** | Alert routing, grouping, silencing | 9093 |
| **Zabbix** | Infrastructure monitoring (host, hardware, SNMP) | 10051 |
| **OpenNMS** | Event correlation and root cause analysis | 8980 |
| **DragonflyDB** | High-performance cache (Redis-compatible) | 6379 |
| **YugabyteDB** | Distributed SQL storage (PostgreSQL-compatible) | 5433 |
| **RustFS** | S3-compatible object storage | 9000 |
| **Fluent-Bit** | Log collection DaemonSet | -- |
| **Prometheus** | Metrics scraping agent (remote_write to VictoriaMetrics) | 9090 |

### Architecture Diagram

```
                                    +------------------+
                                    |   ERP Modules    |
                                    | (20 modules)     |
                                    +--------+---------+
                                             |
                            OTLP (4317/4318) | Metrics (/metrics)
                                             |
                    +------------------------+------------------------+
                    |                        |                        |
            +-------v--------+     +---------v--------+     +--------v--------+
            | OTel Collector |     |   Prometheus     |     |   Fluent-Bit    |
            | (traces/logs)  |     | (scrape agent)   |     | (container logs)|
            +-------+--------+     +---------+--------+     +--------+--------+
                    |                        |                        |
                    v                        v                        v
            +-------+--------+     +---------+--------+     +--------+--------+
            |    Quickwit    |     | VictoriaMetrics  |     |    Quickwit     |
            | (traces/audit) |     | (long-term)      |     | (erp-logs)      |
            +-------+--------+     +---------+--------+     +--------+--------+
                    |                        |                        |
                    +------------------------+------------------------+
                                             |
                                    +--------v---------+
                                    |     Grafana      |
                                    | (visualization)  |
                                    +------------------+
                                             |
         +-----------------------------------+-----------------------------------+
         |                                   |                                   |
 +-------v--------+                 +--------v---------+                +--------v--------+
 |   Go Gateway   |                 | Node.js Obs API  |                | Go Tenant API   |
 |   (port 8090)  +---------------->|   (port 3000)    |                |   (port 8080)   |
 +-------+--------+                 +--------+---------+                +--------+--------+
         |                                   |                                   |
         |                          +--------v---------+                         |
         |                          |   DragonflyDB    |                         |
         |                          |   (cache)        |                         |
         |                          +------------------+                         |
         |                          +--------+---------+                         |
         +------------------------->|   YugabyteDB     |<------------------------+
                                    |   (storage)      |
                                    +------------------+
```

---

## Prerequisites

| Requirement | Minimum Version | Notes |
|---|---|---|
| Docker Desktop | 4.25+ | Required for local development |
| Docker Compose | 2.23+ | Included with Docker Desktop |
| Go | 1.22+ | For gateway and tenant-api |
| Node.js | 20.x LTS | For observability-api |
| npm | 10.x | Included with Node.js 20 |
| Git | 2.40+ | For cloning the repository |
| Make | 3.81+ | For build automation |

**Optional (for Kubernetes deployment):**

| Requirement | Minimum Version |
|---|---|
| kubectl | 1.28+ |
| Helm | 3.13+ |
| Helmfile | 0.158+ |

---

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/ERP-Observability.git
cd ERP-Observability
```

### 2. Environment Configuration

Copy the example environment file and adjust as needed:

```bash
cp .env.example .env
```

Key environment variables:

```env
# Gateway
PORT=8090
CORS_ORIGINS=http://localhost:5173,http://localhost:3001
OBSERVABILITY_API_URL=http://observability-api:3000

# Observability API
NODE_ENV=development
LOG_LEVEL=info
DRAGONFLY_HOST=dragonfly
DRAGONFLY_PORT=6379
YUGABYTE_HOST=yugabytedb
YUGABYTE_PORT=5433
YUGABYTE_DB=erp_observability
YUGABYTE_USER=erp
YUGABYTE_PASSWORD=erp
QUICKWIT_URL=http://quickwit:7280
PROMETHEUS_URL=http://prometheus:9090
ALERTMANAGER_URL=http://alertmanager:9093
GRAFANA_URL=http://grafana:3001
OTEL_COLLECTOR_URL=http://otel-collector:13133

# RustFS (S3)
RUSTFS_ACCESS_KEY=erp-observability
RUSTFS_SECRET_KEY=change-me-in-production
```

### 3. Start All Services

```bash
docker-compose up -d
```

This starts all infrastructure services: Quickwit, Prometheus, Grafana, Alertmanager, DragonflyDB, YugabyteDB, RustFS, OTel Collector, Fluent-Bit, the Go gateway, and the Node.js observability-api.

### 4. Initialize Quickwit Indexes

```bash
./scripts/create-indexes.sh
```

This creates the five Quickwit indexes: `erp-logs`, `erp-traces`, `erp-audit`, `erp-metrics`, `erp-search`.

### 5. Run Database Migration

```bash
# Using psql against YugabyteDB
psql -h localhost -p 5433 -U erp -d erp_observability -f migrations/001_initial_schema.sql
```

### 6. Verify Services

```bash
# Run the verification script
./scripts/verify-observability.sh

# Or check individually:
# Gateway health
curl http://localhost:8090/healthz

# Observability API health (shows component status)
curl http://localhost:8090/v1/observability/health

# Capabilities
curl http://localhost:8090/v1/capabilities

# Grafana
curl http://localhost:3001/api/health

# Quickwit
curl http://localhost:7280/health/readyz

# Prometheus
curl http://localhost:9090/-/ready
```

Expected gateway health response:

```json
{
  "status": "healthy",
  "module": "ERP-Observability"
}
```

Expected observability API health response:

```json
{
  "status": "healthy",
  "components": {
    "quickwit": { "status": "up", "latency_ms": 5 },
    "prometheus": { "status": "up", "latency_ms": 3 },
    "grafana": { "status": "up", "latency_ms": 8 },
    "dragonfly": { "status": "up", "latency_ms": 1 },
    "yugabytedb": { "status": "up", "latency_ms": 12 },
    "otel_collector": { "status": "up", "latency_ms": 2 },
    "alertmanager": { "status": "up", "latency_ms": 4 }
  },
  "timestamp": "2026-02-24T10:00:00.000Z"
}
```

### 7. Access the Frontend

| Interface | URL | Credentials |
|---|---|---|
| Observability Frontend (Refine.dev) | http://localhost:5173 | Dev mode: auto-login |
| Grafana | http://localhost:3001 | admin / admin |
| Quickwit UI | http://localhost:7280/ui | No auth required |
| Prometheus UI | http://localhost:9090 | No auth required |
| Alertmanager UI | http://localhost:9093 | No auth required |

---

## API Endpoints

### Gateway (port 8090)

| Method | Path | Description |
|---|---|---|
| GET | `/healthz` | Gateway health check |
| GET | `/v1/capabilities` | Module capabilities |
| * | `/v1/observability/*` | Proxied to observability-api |

### Observability API (proxied via gateway)

| Method | Path | Description |
|---|---|---|
| GET | `/v1/observability/health` | Component health status |
| GET | `/v1/observability/logs` | Query logs |
| GET | `/v1/observability/logs/stream` | SSE log stream |
| GET | `/v1/observability/logs/stats` | Log volume statistics |
| GET | `/v1/observability/metrics/query` | Instant PromQL query |
| GET | `/v1/observability/metrics/range` | Range PromQL query |
| GET | `/v1/observability/metrics/modules` | Per-module metrics summary |
| GET | `/v1/observability/traces` | Query traces |
| GET | `/v1/observability/traces/:traceId` | Get full trace |
| GET | `/v1/observability/traces/:traceId/timeline` | Trace timeline |
| GET | `/v1/observability/alerts` | List active alerts |
| GET | `/v1/observability/alerts/rules` | List alert rules |
| POST | `/v1/observability/alerts/rules` | Create alert rule |
| PUT | `/v1/observability/alerts/rules/:id` | Update alert rule |
| DELETE | `/v1/observability/alerts/rules/:id` | Delete alert rule |
| POST | `/v1/observability/alerts/:id/silence` | Silence alert |
| GET | `/v1/observability/dashboards` | List dashboards |
| GET | `/v1/observability/dashboards/:id` | Get dashboard |
| POST | `/v1/observability/dashboards` | Create dashboard |
| PUT | `/v1/observability/dashboards/:id` | Update dashboard |
| GET | `/v1/observability/search` | Cross-module search |
| GET | `/v1/observability/search/suggest` | Search suggestions |
| POST | `/v1/observability/search/reindex` | Trigger reindex |

See [21-API-Documentation.md](./21-API-Documentation.md) for complete request/response schemas.

---

## Project Structure

```
ERP-Observability/
├── cmd/
│   └── server/
│       └── main.go                    # Go gateway (port 8090)
├── services/
│   └── observability-api/
│       ├── package.json               # Node.js dependencies
│       ├── tsconfig.json
│       └── src/
│           ├── index.ts               # Express app entry point
│           ├── types/
│           │   └── index.ts           # TypeScript type definitions
│           ├── middleware/
│           │   ├── auth.ts            # JWT/Authentik authentication
│           │   └── tenant-context.ts  # Multi-tenant context extraction
│           ├── routes/
│           │   ├── logs.ts            # Log querying & streaming
│           │   ├── metrics.ts         # PromQL query proxy
│           │   ├── traces.ts          # Trace querying
│           │   ├── alerts.ts          # Alert rule CRUD
│           │   ├── dashboards.ts      # Dashboard CRUD
│           │   └── search.ts          # Cross-module search
│           ├── controllers/
│           │   ├── log-aggregator.ts  # Quickwit log query engine
│           │   ├── metric-collector.ts # Prometheus query proxy
│           │   ├── trace-analyzer.ts  # Quickwit trace query engine
│           │   ├── alert-manager.ts   # Alertmanager integration
│           │   └── search-engine.ts   # Cross-module search engine
│           ├── indexing/
│           │   ├── quickwit-client.ts # Quickwit HTTP client
│           │   ├── index-manager.ts   # Index lifecycle management
│           │   └── ingest-pipeline.ts # Document ingestion pipeline
│           └── search/
│               ├── cross-module-search.ts
│               ├── query-builder.ts
│               └── result-aggregator.ts
├── configs/
│   ├── capabilities.json              # Module capability declaration
│   ├── module_dependencies.yaml       # Dependency graph
│   ├── quickwit/
│   │   ├── quickwit.yaml             # Quickwit node configuration
│   │   ├── index-erp-logs.yaml       # Log index schema
│   │   ├── index-erp-traces.yaml     # Trace index schema
│   │   ├── index-erp-audit.yaml      # Audit index schema
│   │   ├── index-erp-metrics.yaml    # Metrics index schema
│   │   └── index-erp-search.yaml     # Search index schema
│   ├── prometheus/
│   │   ├── prometheus.yaml            # Prometheus scrape config
│   │   └── rules/
│   │       ├── erp-slo-rules.yaml    # SLO alert rules
│   │       ├── database-rules.yaml   # Database alert rules
│   │       └── infrastructure-rules.yaml
│   ├── alertmanager/
│   │   └── alertmanager.yaml          # Alert routing config
│   ├── grafana/
│   │   └── dashboards/
│   │       ├── erp-overview.json
│   │       ├── module-health.json
│   │       └── database-health.json
│   └── fluent-bit/
│       └── fluent-bit.yaml
├── migrations/
│   └── 001_initial_schema.sql         # YugabyteDB schema
├── hasura/
│   └── metadata/
│       ├── databases.yaml
│       └── actions.yaml
├── infra/
│   ├── Dockerfile.api                 # Node.js API image
│   ├── Dockerfile.gateway             # Go gateway image
│   └── k8s/
│       ├── namespace.yaml
│       ├── network-policies.yaml
│       ├── quickwit/
│       ├── rustfs/
│       ├── prometheus/
│       ├── grafana/
│       ├── fluent-bit/
│       ├── otel/
│       ├── alertmanager/
│       └── api/
├── scripts/
│   ├── create-indexes.sh
│   ├── rotate-indexes.sh
│   └── verify-observability.sh
├── docs/
│   ├── search-guide.md
│   └── alerting-guide.md
├── go.mod
├── Makefile
└── README.md
```

---

## Configuration

### Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8090` | Gateway listen port |
| `CORS_ORIGINS` | `*` | Comma-separated allowed CORS origins |
| `OBSERVABILITY_API_URL` | `http://observability-api:3000` | Backend API URL |
| `NODE_ENV` | `development` | Node environment |
| `LOG_LEVEL` | `info` | Pino log level |
| `DRAGONFLY_HOST` | `localhost` | DragonflyDB host |
| `DRAGONFLY_PORT` | `6379` | DragonflyDB port |
| `YUGABYTE_HOST` | `localhost` | YugabyteDB host |
| `YUGABYTE_PORT` | `5433` | YugabyteDB YSQL port |
| `YUGABYTE_DB` | `erp_observability` | Database name |
| `YUGABYTE_USER` | `erp` | Database user |
| `YUGABYTE_PASSWORD` | `erp` | Database password |
| `QUICKWIT_URL` | `http://localhost:7280` | Quickwit REST URL |
| `PROMETHEUS_URL` | `http://localhost:9090` | Prometheus URL |
| `ALERTMANAGER_URL` | `http://localhost:9093` | Alertmanager URL |
| `GRAFANA_URL` | `http://localhost:3001` | Grafana URL |
| `OTEL_COLLECTOR_URL` | `http://localhost:13133` | OTel health check URL |
| `AUTHENTIK_JWKS_URL` | Authentik cluster URL | JWKS endpoint for JWT validation |
| `AUTHENTIK_ISSUER` | (none) | Expected JWT issuer |

---

## Testing

```bash
# Run Node.js unit tests
cd services/observability-api && npm test

# Run Go gateway tests
go test ./...

# Run linting
cd services/observability-api && npm run lint

# Run integration tests (requires running services)
npm run test:integration

# Build Go gateway
go build -o bin/gateway ./cmd/server/

# Build Node.js API
cd services/observability-api && npm run build
```

See [30-Test-Strategy.md](./30-Test-Strategy.md) for the complete testing approach.

---

## Deployment

### Docker

```bash
# Build images
docker build -f infra/Dockerfile.gateway -t erp-observability-gateway:latest .
docker build -f infra/Dockerfile.api -t erp-observability-api:latest .
```

### Kubernetes

```bash
# Apply namespace
kubectl apply -f infra/k8s/namespace.yaml

# Apply network policies
kubectl apply -f infra/k8s/network-policies.yaml

# Deploy infrastructure
kubectl apply -f infra/k8s/quickwit/
kubectl apply -f infra/k8s/rustfs/
kubectl apply -f infra/k8s/prometheus/
kubectl apply -f infra/k8s/grafana/
kubectl apply -f infra/k8s/fluent-bit/
kubectl apply -f infra/k8s/otel/
kubectl apply -f infra/k8s/alertmanager/

# Deploy API
kubectl apply -f infra/k8s/api/
```

See [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md) for the full CI/CD pipeline documentation.

---

## Related Documentation

| Document | Description |
|---|---|
| [16-Changelog.md](./16-Changelog.md) | Version history and migration guides |
| [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md) | Technology choices and rationale |
| [19-CONTRIBUTING.md](./19-CONTRIBUTING.md) | Contribution guidelines |
| [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md) | Detailed local setup guide |
| [21-API-Documentation.md](./21-API-Documentation.md) | Complete API reference |
| [24-Runbooks.md](./24-Runbooks.md) | Operational runbooks |
| [27-Entity-Relationship-Diagram.md](./27-Entity-Relationship-Diagram.md) | Database ER diagrams |
| [31-SECURITY.md](./31-SECURITY.md) | Security policy |

---

## License

Proprietary. All rights reserved.
