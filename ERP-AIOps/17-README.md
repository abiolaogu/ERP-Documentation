# ERP-AIOps -- Quick Start Guide

> **Document ID:** ERP-AIOPS-QS-017
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md), [21-API-Documentation.md](./21-API-Documentation.md)

---

## Overview

ERP-AIOps is an AI-powered operations platform providing intelligent incident management, anomaly detection, root cause analysis, event correlation, auto-remediation, cost optimization, security scanning, topology mapping, and forecasting for the ERP ecosystem.

**Architecture**: Rust core (Axum API with 40+ crates) + Python AI Brain (FastAPI) + Go Gateway + Refine.dev Frontend

**Data Stores**: YugabyteDB (primary), DragonflyDB (cache), RustFS (object storage)

---

## Prerequisites

Ensure the following tools are installed on your development machine before proceeding:

| Tool | Minimum Version | Recommended Version | Purpose |
|------|----------------|---------------------|---------|
| Docker | 24.0+ | 25.x | Container runtime |
| Docker Compose | 2.20+ | 2.24+ | Multi-container orchestration |
| Go | 1.22+ | 1.22.x | Gateway service |
| Rust | 1.76+ | 1.78+ | Core platform |
| Python | 3.12+ | 3.12.x | AI Brain service |
| Node.js | 20+ | 20 LTS | Frontend application |
| npm | 10+ | 10.x | Package management |
| Git | 2.40+ | 2.44+ | Version control |

### Verify Prerequisites

```bash
# Verify all prerequisites
docker --version          # Docker version 25.x.x
docker compose version    # Docker Compose version v2.24.x
go version                # go1.22.x
rustc --version           # rustc 1.78.x
cargo --version           # cargo 1.78.x
python3 --version         # Python 3.12.x
node --version            # v20.x.x
npm --version             # 10.x.x
git --version             # git version 2.44.x
```

---

## Quick Start (5 Minutes)

### Step 1: Clone the Repository

```bash
git clone https://github.com/your-org/ERP-AIOps.git
cd ERP-AIOps
```

### Step 2: Configure Environment

```bash
# Copy the example environment file
cp .env.example .env

# Review and customize environment variables (optional for development)
# Default values are pre-configured for local development
```

**Default `.env` configuration:**

```env
# Rust API
RUST_API_HOST=0.0.0.0
RUST_API_PORT=8080
RUST_LOG=info,aiops=debug

# AI Brain (Python FastAPI)
AI_BRAIN_HOST=0.0.0.0
AI_BRAIN_PORT=8081
AI_BRAIN_WORKERS=4

# Go Gateway
GATEWAY_HOST=0.0.0.0
GATEWAY_PORT=3000
GATEWAY_RUST_API_URL=http://rust-api:8080
GATEWAY_AI_BRAIN_URL=http://ai-brain:8081

# YugabyteDB
YUGABYTE_HOST=yugabytedb
YUGABYTE_PORT=5433
YUGABYTE_DB=aiops
YUGABYTE_USER=aiops
YUGABYTE_PASSWORD=aiops_dev_password

# DragonflyDB
DRAGONFLY_HOST=dragonflydb
DRAGONFLY_PORT=6379

# RustFS (S3-compatible)
RUSTFS_ENDPOINT=http://rustfs:9000
RUSTFS_ACCESS_KEY=minioadmin
RUSTFS_SECRET_KEY=minioadmin
RUSTFS_BUCKET=aiops-models

# Frontend
VITE_API_URL=http://localhost:3000
VITE_WS_URL=ws://localhost:3000
```

### Step 3: Start All Services

```bash
# Start the entire stack with Docker Compose
docker compose -f infra/docker-compose.yml up -d
```

This starts the following services:

| Service | Container Name | Port | Description |
|---------|---------------|------|-------------|
| YugabyteDB | `aiops-yugabytedb` | 5433 | Primary database |
| DragonflyDB | `aiops-dragonflydb` | 6379 | Cache layer |
| RustFS | `aiops-rustfs` | 9000/9001 | Object storage |
| Rust API | `aiops-rust-api` | 8080 | Core API server |
| AI Brain | `aiops-ai-brain` | 8081 | ML inference service |
| Go Gateway | `aiops-gateway` | 3000 | API gateway |
| Frontend | `aiops-frontend` | 5173 | Web application |

### Step 4: Run Database Migrations

```bash
# Migrations run automatically on first start, but can be triggered manually:
docker compose -f infra/docker-compose.yml exec rust-api ./aiops-migrate up
```

### Step 5: Verify Services

```bash
# Verify Rust API is healthy
curl -s http://localhost:8080/healthz | jq .
# Expected: { "status": "healthy", "version": "1.0.0", "uptime_seconds": ... }

# Verify AI Brain is healthy
curl -s http://localhost:8081/health | jq .
# Expected: { "status": "healthy", "models_loaded": [...], "gpu_available": false }

# Verify Go Gateway is healthy
curl -s http://localhost:3000/health | jq .
# Expected: { "status": "healthy", "backends": { "rust_api": "up", "ai_brain": "up" } }

# Verify all services via Gateway health aggregation
curl -s http://localhost:3000/health/all | jq .
```

### Step 6: Access the Frontend

Open your browser and navigate to:

```
http://localhost:5173
```

**Default development credentials:**

| Field | Value |
|-------|-------|
| Email | `admin@aiops.dev` |
| Password | `admin123` |
| Tenant | `dev-tenant-001` |

---

## First API Calls

### Create an Incident

```bash
curl -X POST http://localhost:3000/api/v1/incidents \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-jwt-token>" \
  -H "X-Tenant-ID: dev-tenant-001" \
  -d '{
    "title": "High CPU utilization on production API servers",
    "description": "CPU usage exceeded 90% on api-server-01, api-server-02, and api-server-03 for the past 15 minutes.",
    "severity": "high",
    "source": "prometheus",
    "tags": ["cpu", "production", "api-servers"]
  }'
```

### List Incidents

```bash
curl -s http://localhost:3000/api/v1/incidents \
  -H "Authorization: Bearer <your-jwt-token>" \
  -H "X-Tenant-ID: dev-tenant-001" | jq .
```

### Trigger Anomaly Detection

```bash
curl -X POST http://localhost:3000/api/v1/anomalies/detect \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-jwt-token>" \
  -H "X-Tenant-ID: dev-tenant-001" \
  -d '{
    "metric_name": "api_response_time_ms",
    "values": [120, 125, 118, 122, 450, 480, 520, 495, 510],
    "timestamps": [
      "2026-02-24T10:00:00Z",
      "2026-02-24T10:01:00Z",
      "2026-02-24T10:02:00Z",
      "2026-02-24T10:03:00Z",
      "2026-02-24T10:04:00Z",
      "2026-02-24T10:05:00Z",
      "2026-02-24T10:06:00Z",
      "2026-02-24T10:07:00Z",
      "2026-02-24T10:08:00Z"
    ],
    "detection_method": "adaptive"
  }'
```

### Request AI Root Cause Analysis

```bash
curl -X POST http://localhost:3000/api/v1/incidents/<incident-id>/rca \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-jwt-token>" \
  -H "X-Tenant-ID: dev-tenant-001" \
  -d '{
    "include_topology": true,
    "include_recent_changes": true,
    "max_hypotheses": 5
  }'
```

---

## Project Structure

```
ERP-AIOps/
├── Cargo.toml                    # Rust workspace root
├── Cargo.lock
├── crates/
│   ├── aiops-core/               # Core types, traits, and utilities
│   ├── aiops-api/                # Axum HTTP API server
│   ├── aiops-db/                 # Database layer (YugabyteDB)
│   ├── aiops-models/             # Data models and schemas
│   ├── aiops-config/             # Configuration management
│   ├── aiops-cache/              # DragonflyDB caching layer
│   ├── aiops-auth/               # Authentication & authorization
│   ├── aiops-incidents/          # Incident management
│   ├── aiops-anomaly/            # Anomaly detection engine
│   ├── aiops-rca/                # Root cause analysis
│   ├── aiops-correlation/        # Event correlation engine
│   ├── aiops-topology/           # Topology mapping
│   ├── aiops-remediation/        # Auto-remediation framework
│   ├── aiops-cost/               # Cost optimization
│   ├── aiops-security/           # Security scanning
│   ├── aiops-forecasting/        # Forecasting & capacity planning
│   ├── aiops-rules/              # Rules engine
│   ├── aiops-webhooks/           # Webhook delivery
│   ├── aiops-ai-client/          # AI Brain gRPC/REST client
│   ├── aiops-storage/            # RustFS/S3 integration
│   ├── aiops-metrics/            # Prometheus metrics
│   ├── aiops-tracing/            # OpenTelemetry tracing
│   ├── aiops-migrate/            # Database migrations
│   └── ...                       # Additional crates (40+ total)
├── ai-brain/
│   ├── app/
│   │   ├── main.py               # FastAPI application entry
│   │   ├── routers/              # API route handlers
│   │   ├── models/               # ML model definitions
│   │   ├── services/             # Business logic services
│   │   ├── schemas/              # Pydantic request/response schemas
│   │   └── utils/                # Utility functions
│   ├── models/                   # Trained model artifacts
│   ├── tests/                    # Python test suite
│   ├── requirements.txt
│   ├── pyproject.toml
│   └── Dockerfile
├── gateway/
│   ├── cmd/server/main.go        # Go gateway entry point
│   ├── internal/
│   │   ├── routes/               # Route definitions
│   │   ├── middleware/           # Middleware (auth, rate limit, etc.)
│   │   ├── proxy/                # Reverse proxy handlers
│   │   └── health/               # Health check aggregation
│   ├── go.mod
│   ├── go.sum
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── App.tsx               # Refine.dev application root
│   │   ├── pages/                # Page components
│   │   ├── components/           # Shared components
│   │   ├── providers/            # Data & auth providers
│   │   ├── hooks/                # Custom React hooks
│   │   └── theme/                # Purple (#7c3aed) theme config
│   ├── package.json
│   ├── vite.config.ts
│   └── Dockerfile
├── migrations/
│   ├── 001_initial_schema.sql
│   ├── 002_anomaly_tables.sql
│   ├── 003_topology_tables.sql
│   ├── 004_remediation_tables.sql
│   ├── 005_cost_tables.sql
│   ├── 006_security_tables.sql
│   └── ...
├── infra/
│   ├── docker-compose.yml        # Development stack
│   ├── docker-compose.prod.yml   # Production overrides
│   ├── helm/                     # Kubernetes Helm charts
│   └── terraform/                # Infrastructure as Code
├── proto/
│   └── aiops/                    # gRPC protocol definitions
├── scripts/
│   ├── setup.sh                  # One-click setup script
│   ├── seed-data.sh              # Seed development data
│   └── run-tests.sh              # Run all test suites
└── docs/                         # Additional documentation
```

---

## Common Commands

### Build Commands

```bash
# Build all Rust crates
cargo build --workspace

# Build in release mode
cargo build --workspace --release

# Build specific crate
cargo build -p aiops-api

# Build AI Brain (no build step, but install dependencies)
cd ai-brain && pip install -r requirements.txt

# Build Go Gateway
cd gateway && go build -o gateway ./cmd/server/

# Build Frontend
cd frontend && npm install && npm run build
```

### Test Commands

```bash
# Run all Rust tests
cargo test --workspace

# Run tests for a specific crate
cargo test -p aiops-anomaly

# Run Python AI Brain tests
cd ai-brain && pytest tests/ -v

# Run Go Gateway tests
cd gateway && go test ./...

# Run frontend tests
cd frontend && npm test

# Run integration tests (requires running stack)
./scripts/run-tests.sh integration
```

### Development Commands

```bash
# Start stack in development mode with hot-reload
docker compose -f infra/docker-compose.yml up -d

# View logs for all services
docker compose -f infra/docker-compose.yml logs -f

# View logs for a specific service
docker compose -f infra/docker-compose.yml logs -f rust-api

# Restart a specific service
docker compose -f infra/docker-compose.yml restart rust-api

# Stop all services
docker compose -f infra/docker-compose.yml down

# Stop and remove volumes (clean start)
docker compose -f infra/docker-compose.yml down -v

# Seed development data
./scripts/seed-data.sh

# Run Rust with hot-reload (using cargo-watch)
cargo watch -x 'run --bin aiops-api'

# Run AI Brain with hot-reload
cd ai-brain && uvicorn app.main:app --reload --port 8081

# Run frontend dev server
cd frontend && npm run dev
```

### Database Commands

```bash
# Connect to YugabyteDB
docker compose -f infra/docker-compose.yml exec yugabytedb ysqlsh -U aiops -d aiops

# Run migrations
cargo run --bin aiops-migrate -- up

# Rollback last migration
cargo run --bin aiops-migrate -- down

# Check migration status
cargo run --bin aiops-migrate -- status
```

---

## Troubleshooting

### Services fail to start

```bash
# Check Docker resource limits (minimum 8GB RAM recommended)
docker system info | grep -i memory

# Check if ports are already in use
lsof -i :8080 -i :8081 -i :3000 -i :5173 -i :5433 -i :6379 -i :9000

# View detailed container logs
docker compose -f infra/docker-compose.yml logs --tail=100
```

### Rust API fails to connect to YugabyteDB

```bash
# Verify YugabyteDB is accepting connections
docker compose -f infra/docker-compose.yml exec yugabytedb ysqlsh -U aiops -d aiops -c "SELECT 1;"

# Check YugabyteDB logs
docker compose -f infra/docker-compose.yml logs yugabytedb

# Ensure DATABASE_URL is correctly set
echo $DATABASE_URL
# Expected: postgres://aiops:aiops_dev_password@localhost:5433/aiops
```

### AI Brain models fail to load

```bash
# Check AI Brain logs for model loading errors
docker compose -f infra/docker-compose.yml logs ai-brain

# Verify RustFS is accessible and models bucket exists
curl -s http://localhost:9000/minio/health/live

# Re-upload model artifacts
./scripts/upload-models.sh
```

### Frontend shows blank page

```bash
# Verify the Gateway is running and proxying correctly
curl -s http://localhost:3000/health

# Check frontend build logs
docker compose -f infra/docker-compose.yml logs frontend

# Rebuild frontend
docker compose -f infra/docker-compose.yml build frontend
docker compose -f infra/docker-compose.yml up -d frontend
```

---

## Next Steps

1. **Explore the API**: See [21-API-Documentation.md](./21-API-Documentation.md) for complete API reference.
2. **Set up local development**: See [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md) for detailed environment setup.
3. **Understand the architecture**: See [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md) for key design decisions.
4. **Contribute**: See [19-CONTRIBUTING.md](./19-CONTRIBUTING.md) for contribution guidelines.
5. **Import Postman Collection**: See [22-Postman-Collection.md](./22-Postman-Collection.md) for ready-to-use API examples.

---

## Support

- **Internal Wiki**: https://wiki.internal/erp-aiops
- **Slack Channel**: `#erp-aiops-dev`
- **Issue Tracker**: https://github.com/your-org/ERP-AIOps/issues
- **On-Call Rotation**: See [24-Runbooks.md](./24-Runbooks.md) for operational runbooks

---

*For detailed setup instructions, see [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md).*
