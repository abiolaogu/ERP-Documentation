# ERP-Observability Local Environment Setup

> **Document ID:** ERP-OBS-LES-020
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [19-CONTRIBUTING.md](./19-CONTRIBUTING.md)

---

## Table of Contents

1. [Prerequisites Installation](#1-prerequisites-installation)
2. [Repository Setup](#2-repository-setup)
3. [Environment Configuration](#3-environment-configuration)
4. [Starting Infrastructure Services](#4-starting-infrastructure-services)
5. [Initializing Quickwit Indexes](#5-initializing-quickwit-indexes)
6. [Running Database Migrations](#6-running-database-migrations)
7. [Starting Application Services](#7-starting-application-services)
8. [Verifying the Installation](#8-verifying-the-installation)
9. [Accessing the Frontend](#9-accessing-the-frontend)
10. [Running Tests](#10-running-tests)
11. [Common Issues and Troubleshooting](#11-common-issues-and-troubleshooting)
12. [Resetting the Environment](#12-resetting-the-environment)

---

## 1. Prerequisites Installation

### macOS

#### Go 1.22+

```bash
# Using Homebrew
brew install go@1.22

# Verify
go version
# Expected: go version go1.22.x darwin/arm64 (or amd64)

# Ensure GOPATH is set
echo 'export GOPATH=$HOME/go' >> ~/.zshrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.zshrc
source ~/.zshrc
```

#### Node.js 20.x LTS

```bash
# Using nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.zshrc
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version
# Expected: v20.x.x

npm --version
# Expected: 10.x.x
```

#### Docker Desktop

1. Download Docker Desktop from https://www.docker.com/products/docker-desktop/
2. Install and launch Docker Desktop.
3. In Docker Desktop Settings:
   - **Resources > Memory:** Allocate at least **8 GB** (16 GB recommended for full stack).
   - **Resources > CPUs:** Allocate at least **4 CPUs**.
   - **Resources > Disk:** Ensure at least **50 GB** available.
4. Verify:

```bash
docker --version
# Expected: Docker version 24.x or 25.x

docker compose version
# Expected: Docker Compose version v2.23.x+
```

#### Make

```bash
# macOS ships with Make via Xcode Command Line Tools
xcode-select --install

make --version
# Expected: GNU Make 3.81+
```

#### psql Client (for database migrations)

```bash
brew install libpq
echo 'export PATH="/opt/homebrew/opt/libpq/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

psql --version
# Expected: psql (PostgreSQL) 16.x
```

### Linux (Ubuntu/Debian)

#### Go 1.22+

```bash
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc
source ~/.bashrc

go version
```

#### Node.js 20.x LTS

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
nvm alias default 20
```

#### Docker Engine + Docker Compose

```bash
# Install Docker Engine
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker

# Install Docker Compose v2
sudo apt-get install docker-compose-plugin

docker --version
docker compose version
```

#### Make and psql

```bash
sudo apt-get update
sudo apt-get install -y build-essential postgresql-client
```

---

## 2. Repository Setup

```bash
# Clone the repository
git clone https://github.com/your-org/ERP-Observability.git
cd ERP-Observability

# Verify the project structure
ls -la
# Expected: cmd/ configs/ docs/ hasura/ infra/ migrations/ scripts/ services/ go.mod Makefile README.md

# Install Node.js dependencies for the observability API
cd services/observability-api
npm install
cd ../..

# Verify Go module
go mod download
go mod verify
```

---

## 3. Environment Configuration

Create a `.env` file in the project root:

```bash
cp .env.example .env
```

If `.env.example` does not exist, create `.env` with the following content:

```env
# ============================================================
# ERP-Observability Local Development Environment
# ============================================================

# --- Go Gateway ---
PORT=8090
CORS_ORIGINS=http://localhost:5173,http://localhost:3001,http://localhost:3000
OBSERVABILITY_API_URL=http://localhost:3000

# --- Node.js Observability API ---
NODE_ENV=development
LOG_LEVEL=debug
API_PORT=3000

# --- DragonflyDB ---
DRAGONFLY_HOST=localhost
DRAGONFLY_PORT=6379

# --- YugabyteDB ---
YUGABYTE_HOST=localhost
YUGABYTE_PORT=5433
YUGABYTE_DB=erp_observability
YUGABYTE_USER=erp
YUGABYTE_PASSWORD=erp_dev_password

# --- Quickwit ---
QUICKWIT_URL=http://localhost:7280

# --- Prometheus ---
PROMETHEUS_URL=http://localhost:9090

# --- Alertmanager ---
ALERTMANAGER_URL=http://localhost:9093

# --- Grafana ---
GRAFANA_URL=http://localhost:3001
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=admin

# --- OTel Collector ---
OTEL_COLLECTOR_URL=http://localhost:13133

# --- RustFS (S3-compatible storage for Quickwit) ---
RUSTFS_ACCESS_KEY=erp-observability-dev
RUSTFS_SECRET_KEY=erp-dev-secret-key-change-in-prod
RUSTFS_ENDPOINT=http://localhost:9000
RUSTFS_REGION=us-east-1

# --- Authentication (development mode) ---
# In development, auth middleware allows unauthenticated requests
# with a default dev-user identity and admin role.
# Set AUTHENTIK_JWKS_URL for production-like auth testing:
# AUTHENTIK_JWKS_URL=http://localhost:9000/application/o/erp-platform/jwks/
# AUTHENTIK_ISSUER=http://localhost:9000/application/o/erp-platform/

# --- Slack Webhooks (optional, for alert testing) ---
# SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

---

## 4. Starting Infrastructure Services

### Using Docker Compose

```bash
# Start all infrastructure services in the background
docker compose up -d

# This starts:
# - Quickwit (port 7280) - Log/trace/search engine
# - Prometheus (port 9090) - Metrics scraping
# - Grafana (port 3001) - Visualization
# - Alertmanager (port 9093) - Alert routing
# - DragonflyDB (port 6379) - Cache
# - YugabyteDB (port 5433 YSQL, 7000 master UI) - Database
# - RustFS (port 9000 API, 9001 console) - S3 storage
# - OTel Collector (port 4317 gRPC, 4318 HTTP, 13133 health)
```

### Verify Infrastructure Health

```bash
# Check all containers are running
docker compose ps

# Expected output (all should show "running" or "healthy"):
# NAME                STATUS
# quickwit            running
# prometheus          running
# grafana             running
# alertmanager        running
# dragonfly           running
# yugabytedb          running
# rustfs              running
# otel-collector      running
```

### Individual Service Checks

```bash
# Quickwit
curl -s http://localhost:7280/health/readyz
# Expected: {"status":"ready"}

# Prometheus
curl -s http://localhost:9090/-/ready
# Expected: "Prometheus Server is Ready."

# Grafana
curl -s http://localhost:3001/api/health
# Expected: {"commit":"...","database":"ok","version":"..."}

# Alertmanager
curl -s http://localhost:9093/-/ready
# Expected: "OK"

# DragonflyDB
docker compose exec dragonfly redis-cli ping
# Expected: PONG

# YugabyteDB
psql -h localhost -p 5433 -U erp -d erp_observability -c "SELECT 1;"
# Expected: ?column? | 1

# RustFS
curl -s http://localhost:9000/minio/health/ready
# Expected: (empty 200 response)

# OTel Collector
curl -s http://localhost:13133
# Expected: {"status":"Server available","..."}
```

---

## 5. Initializing Quickwit Indexes

Quickwit indexes must be created before data can be ingested:

```bash
# Make the script executable
chmod +x scripts/create-indexes.sh

# Run the index creation script
./scripts/create-indexes.sh
```

The script creates five indexes:

| Index | Configuration File | Purpose |
|---|---|---|
| `erp-logs` | `configs/quickwit/index-erp-logs.yaml` | Application logs (90-day retention) |
| `erp-traces` | `configs/quickwit/index-erp-traces.yaml` | Distributed traces (30-day retention) |
| `erp-audit` | `configs/quickwit/index-erp-audit.yaml` | Audit trail (730-day retention) |
| `erp-metrics` | `configs/quickwit/index-erp-metrics.yaml` | Prometheus remote_write metrics |
| `erp-search` | `configs/quickwit/index-erp-search.yaml` | Cross-module full-text search |

**Manual index creation (if the script fails):**

```bash
# Create each index individually
curl -X POST http://localhost:7280/api/v1/indexes \
  -H "Content-Type: application/yaml" \
  --data-binary @configs/quickwit/index-erp-logs.yaml

curl -X POST http://localhost:7280/api/v1/indexes \
  -H "Content-Type: application/yaml" \
  --data-binary @configs/quickwit/index-erp-traces.yaml

curl -X POST http://localhost:7280/api/v1/indexes \
  -H "Content-Type: application/yaml" \
  --data-binary @configs/quickwit/index-erp-audit.yaml

curl -X POST http://localhost:7280/api/v1/indexes \
  -H "Content-Type: application/yaml" \
  --data-binary @configs/quickwit/index-erp-metrics.yaml

curl -X POST http://localhost:7280/api/v1/indexes \
  -H "Content-Type: application/yaml" \
  --data-binary @configs/quickwit/index-erp-search.yaml

# Verify indexes exist
curl -s http://localhost:7280/api/v1/indexes | python3 -m json.tool
```

---

## 6. Running Database Migrations

```bash
# Create the database (if it does not exist)
psql -h localhost -p 5433 -U erp -c "CREATE DATABASE erp_observability;" 2>/dev/null || true

# Run the initial migration
psql -h localhost -p 5433 -U erp -d erp_observability -f migrations/001_initial_schema.sql

# Verify tables were created
psql -h localhost -p 5433 -U erp -d erp_observability -c "\dt"
# Expected tables:
#  Schema |       Name           | Type  | Owner
# --------+----------------------+-------+-------
#  public | alert_rules          | table | erp
#  public | alert_silences       | table | erp
#  public | dashboards           | table | erp
#  public | search_index_status  | table | erp

# Verify indexes
psql -h localhost -p 5433 -U erp -d erp_observability -c "\di"
```

---

## 7. Starting Application Services

### Start the Go Gateway

```bash
# From the project root
go run ./cmd/server/main.go

# Expected output:
# 2026/02/24 10:00:00 route /v1/observability/search -> http://localhost:3000
# 2026/02/24 10:00:00 route /v1/observability/logs -> http://localhost:3000
# 2026/02/24 10:00:00 route /v1/observability/metrics -> http://localhost:3000
# 2026/02/24 10:00:00 route /v1/observability/traces -> http://localhost:3000
# 2026/02/24 10:00:00 route /v1/observability/alerts -> http://localhost:3000
# 2026/02/24 10:00:00 route /v1/observability/dashboards -> http://localhost:3000
# 2026/02/24 10:00:00 route /v1/observability/health -> http://localhost:3000
# 2026/02/24 10:00:00 ERP-Observability gateway listening on :8090 (CORS origins: [http://localhost:5173 http://localhost:3001 http://localhost:3000])
```

### Start the Node.js Observability API

In a separate terminal:

```bash
cd services/observability-api
npm run dev

# Expected output:
# [10:00:01.123] INFO (observability-api): YugabyteDB connection verified
# [10:00:01.456] INFO (observability-api): Observability API listening {"port":3000}
```

### Start the Frontend (Optional)

If the frontend directory exists:

```bash
cd frontend
npm install
npm run dev

# Expected output:
# VITE v5.x.x ready in 500ms
# Local:   http://localhost:5173/
```

---

## 8. Verifying the Installation

### Run the Verification Script

```bash
chmod +x scripts/verify-observability.sh
./scripts/verify-observability.sh
```

### Manual Verification

```bash
# 1. Gateway health
curl -s http://localhost:8090/healthz | python3 -m json.tool
# Expected: {"status":"healthy","module":"ERP-Observability"}

# 2. Capabilities
curl -s http://localhost:8090/v1/capabilities | python3 -m json.tool
# Expected: {"module":"ERP-Observability","version":"1.0.0","capabilities":[...]}

# 3. Component health (all components)
curl -s http://localhost:8090/v1/observability/health | python3 -m json.tool
# Expected: {"status":"healthy","components":{...},"timestamp":"..."}

# 4. Ingest a test log entry
curl -X POST http://localhost:7280/api/v1/erp-logs/ingest \
  -H "Content-Type: application/json" \
  -d '[{
    "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%S.000Z)'",
    "level": "info",
    "message": "Test log entry from local setup verification",
    "module": "platform",
    "service": "test",
    "tenant_id": "dev-tenant"
  }]'

# 5. Wait for indexing (Quickwit commits every 30s)
sleep 35

# 6. Query logs
curl -s "http://localhost:8090/v1/observability/logs?limit=5" \
  -H "X-Tenant-ID: dev-tenant" | python3 -m json.tool

# 7. Test metrics query (Prometheus up metric)
curl -s "http://localhost:8090/v1/observability/metrics/query?query=up" \
  -H "X-Tenant-ID: dev-tenant" | python3 -m json.tool

# 8. Test search
curl -s "http://localhost:8090/v1/observability/search?q=test" \
  -H "X-Tenant-ID: dev-tenant" | python3 -m json.tool

# 9. Create a test dashboard
curl -X POST http://localhost:8090/v1/observability/dashboards \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: dev-tenant" \
  -d '{
    "name": "Local Dev Dashboard",
    "description": "Test dashboard for local development",
    "panels": [],
    "tags": ["test", "local"]
  }' | python3 -m json.tool

# 10. List dashboards
curl -s http://localhost:8090/v1/observability/dashboards \
  -H "X-Tenant-ID: dev-tenant" | python3 -m json.tool
```

---

## 9. Accessing the Frontend

| Service | URL | Notes |
|---|---|---|
| Observability Frontend | http://localhost:5173 | React + Refine.dev SPA |
| Grafana | http://localhost:3001 | Login: admin / admin |
| Quickwit UI | http://localhost:7280/ui | Browse indexes, run queries |
| Prometheus UI | http://localhost:9090 | Run PromQL queries, view targets |
| Alertmanager UI | http://localhost:9093 | View active alerts, silences |
| RustFS Console | http://localhost:9001 | S3 bucket browser, login with RUSTFS credentials |
| YugabyteDB Master UI | http://localhost:7000 | Cluster overview, tablet info |

---

## 10. Running Tests

```bash
# Go gateway unit tests
go test ./... -v -race -count=1

# TypeScript observability-api tests
cd services/observability-api
npm test

# TypeScript tests with coverage report
npm test -- --coverage

# TypeScript linting
npm run lint

# Build check (TypeScript compilation)
npm run build

# Frontend tests (if frontend exists)
cd frontend
npm test

# Run all checks (via Makefile)
make check
# Equivalent to: go vet && go test && cd services/observability-api && npm run lint && npm test
```

---

## 11. Common Issues and Troubleshooting

### Docker: Containers failing to start

**Symptom:** `docker compose up` fails with port conflicts.

**Fix:** Check for existing processes on required ports:
```bash
lsof -i :7280  # Quickwit
lsof -i :9090  # Prometheus
lsof -i :3001  # Grafana
lsof -i :9093  # Alertmanager
lsof -i :6379  # DragonflyDB
lsof -i :5433  # YugabyteDB
lsof -i :9000  # RustFS
```

Kill conflicting processes or change ports in `.env` and `docker-compose.yaml`.

### Docker: Insufficient memory

**Symptom:** Containers crash with OOM errors, especially Quickwit or YugabyteDB.

**Fix:** Increase Docker Desktop memory allocation to at least 8 GB (Settings > Resources > Memory).

### YugabyteDB: Connection refused

**Symptom:** `psql: error: connection refused` on port 5433.

**Fix:** YugabyteDB takes 15-30 seconds to initialize. Wait and retry:
```bash
# Check if YugabyteDB is ready
docker compose logs yugabytedb | tail -20
# Look for: "Tablet server successfully started"

# Retry connection
psql -h localhost -p 5433 -U erp -d erp_observability -c "SELECT 1;"
```

### Quickwit: Index creation fails

**Symptom:** `create-indexes.sh` returns errors.

**Fix:** Ensure Quickwit is fully started:
```bash
# Check Quickwit readiness
curl -s http://localhost:7280/health/readyz

# If not ready, check logs
docker compose logs quickwit

# RustFS must be running (Quickwit needs S3 for metastore)
curl -s http://localhost:9000/minio/health/ready

# If RustFS is down, restart it
docker compose restart rustfs
sleep 10
docker compose restart quickwit
```

### Node.js: DragonflyDB connection warning

**Symptom:** `DragonflyDB connection failed, caching disabled` warning on startup.

**Fix:** This is a non-blocking warning. DragonflyDB connection is lazy. Verify DragonflyDB is running:
```bash
docker compose exec dragonfly redis-cli ping
# Expected: PONG
```

If DragonflyDB is running but the API still shows the warning, check that `DRAGONFLY_HOST` and `DRAGONFLY_PORT` in `.env` match the Docker Compose service name and port.

### Go Gateway: "upstream unreachable"

**Symptom:** Gateway returns `502 Bad Gateway` with `"upstream /v1/observability/... unreachable"`.

**Fix:** The Node.js observability-api is not running or not reachable. Ensure:
1. The observability-api is running (`npm run dev` in `services/observability-api/`).
2. `OBSERVABILITY_API_URL` in the gateway environment points to `http://localhost:3000` (for local dev without Docker) or `http://observability-api:3000` (for Docker networking).

### Frontend: CORS errors

**Symptom:** Browser console shows CORS errors when the frontend calls the API.

**Fix:** Ensure the frontend origin is listed in `CORS_ORIGINS`:
```bash
export CORS_ORIGINS=http://localhost:5173,http://localhost:3001
```

Restart the Go gateway after changing environment variables.

---

## 12. Resetting the Environment

### Soft Reset (keep data, restart services)

```bash
docker compose restart
```

### Hard Reset (delete all data)

```bash
# Stop and remove all containers, volumes, and networks
docker compose down -v

# Remove Quickwit data directory
rm -rf quickwit-data/

# Restart fresh
docker compose up -d

# Re-create indexes
./scripts/create-indexes.sh

# Re-run migrations
psql -h localhost -p 5433 -U erp -d erp_observability -f migrations/001_initial_schema.sql
```

### Nuclear Reset (remove everything)

```bash
# Stop containers
docker compose down -v

# Remove Docker images
docker compose down --rmi all -v

# Remove node_modules
rm -rf services/observability-api/node_modules
rm -rf frontend/node_modules

# Remove Go build cache
go clean -cache

# Start from scratch
docker compose up -d
cd services/observability-api && npm install && cd ../..
./scripts/create-indexes.sh
psql -h localhost -p 5433 -U erp -d erp_observability -f migrations/001_initial_schema.sql
```

---

## Service Port Reference

| Service | Port | Protocol | Notes |
|---|---|---|---|
| Go Gateway | 8090 | HTTP | API entry point |
| Observability API | 3000 | HTTP | Node.js backend |
| Tenant API | 8080 | HTTP | Go tenant service |
| Frontend (Vite) | 5173 | HTTP | React SPA dev server |
| Quickwit REST | 7280 | HTTP | Index/search API |
| Quickwit gRPC | 7281 | gRPC | Inter-node communication |
| Prometheus | 9090 | HTTP | Metrics scraping |
| Grafana | 3001 | HTTP | Dashboards |
| Alertmanager | 9093 | HTTP | Alert routing |
| DragonflyDB | 6379 | Redis | Cache |
| YugabyteDB YSQL | 5433 | PostgreSQL | SQL database |
| YugabyteDB Master UI | 7000 | HTTP | Admin console |
| RustFS API | 9000 | HTTP(S) | S3-compatible |
| RustFS Console | 9001 | HTTP(S) | Web UI |
| OTel gRPC | 4317 | gRPC | Trace/log ingestion |
| OTel HTTP | 4318 | HTTP | Trace/log ingestion |
| OTel Health | 13133 | HTTP | Health check |
