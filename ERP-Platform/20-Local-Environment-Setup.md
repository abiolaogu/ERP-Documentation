# ERP-Platform Local Environment Setup

> **Document ID:** ERP-PLAT-LES-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Audience:** Developers
> **Related Documents:** [19-CONTRIBUTING.md](./19-CONTRIBUTING.md), [17-README.md](./17-README.md)

---

## Prerequisites

| Tool | Minimum Version | Recommended Version | Installation |
|------|----------------|-------------------|-------------|
| Go | 1.22 | 1.22.x (latest patch) | [go.dev/dl](https://go.dev/dl/) |
| Docker | 20.10 | 25.0+ | [docker.com](https://www.docker.com/) |
| Docker Compose | 2.0 | 2.24+ | Included with Docker Desktop |
| Node.js | 18 | 20+ | [nodejs.org](https://nodejs.org/) |
| PostgreSQL Client | 15 | 16 | `brew install postgresql` (macOS) |
| Redis CLI | 6 | 7 | `brew install redis` (macOS) |
| NATS CLI | -- | Latest | `brew install nats-io/nats-tools/nats` (macOS) |
| jq | 1.6 | 1.7+ | `brew install jq` (macOS) |
| curl | 7.x | Latest | Pre-installed on macOS/Linux |
| Git | 2.39 | 2.43+ | Pre-installed or `brew install git` |

### Verify Prerequisites

```bash
go version          # Should show go1.22.x
docker --version    # Should show Docker 25.x
docker compose version  # Should show v2.24.x
node --version      # Should show v20.x
psql --version      # Should show psql 16.x
redis-cli --version # Should show redis-cli 7.x
jq --version        # Should show jq-1.7
```

---

## Step 1: Clone the Repository

```bash
git clone <repository-url> ERP-Platform
cd ERP-Platform
```

### Repository Structure

```
ERP-Platform/
  catalog/                    # Product catalog JSON files
    products.json             # 20 modules + 28 bundles
    entitlement-templates.json # Bundle-to-entitlement mappings
  docs/                       # Source documentation
  imports/                    # Imported legacy code (reference)
  infra/                      # Infrastructure configuration
    docker-compose.platform.yml
  merge/                      # Merge manifest and maps
  scripts/                    # Utility scripts
  services/                   # 9 Go microservices
    activation-wizard/
    audit-service/
    entitlement-engine/
    marketplace/
    module-registry/
    notification-hub/
    subscription-hub/         # Primary service
    tenant-provisioner/
    web-hosting/
  tools/                      # Development tools
  Makefile                    # Build targets
```

---

## Step 2: Install Dependencies

### Go Dependencies

The subscription hub has zero external dependencies (stdlib only):

```bash
cd services/subscription-hub
cat go.mod
# module erp-platform/subscription-hub
# go 1.22
```

No `go mod download` needed for the subscription hub. Other services follow the same pattern.

### Node.js Dependencies (Admin Console)

If working on the activation console:

```bash
cd imports/bac_activation/frontend
npm install

cd activation-console
npm install
```

---

## Step 3: Environment Variables

### Create .env File

Create a `.env` file in the project root (or export variables):

```bash
# .env.example -- copy to .env and customize

# Subscription Hub
ERP_CATALOG_PATH=../../catalog/products.json
PORT=8091

# Generic Services
MODULE_NAME=ERP-Platform

# Database
DATABASE_URL=postgres://erp:erp@localhost:5432/erp_platform?sslmode=disable
POSTGRES_USER=erp
POSTGRES_PASSWORD=erp
POSTGRES_DB=erp_platform

# Redis
REDIS_URL=redis://localhost:6379

# NATS
NATS_URL=nats://localhost:4222

# Logging
LOG_LEVEL=debug
LOG_FORMAT=text

# Health Checks
HEALTH_CHECK_INTERVAL=30s
HEALTH_CHECK_TIMEOUT=5s
HEALTH_CHECK_RETRIES=5

# AIDD Guardrails
AIDD_MIN_CONFIDENCE=0.70
AIDD_MED_CONFIDENCE=0.82
AIDD_MAX_BLAST_RADIUS=5000
AIDD_HIGH_VALUE_USD=100000
```

---

## Step 4: Database Setup

### Option A: Docker Compose (Recommended)

Start all infrastructure services:

```bash
docker compose -f infra/docker-compose.platform.yml up -d postgres redis nats
```

Verify:
```bash
# PostgreSQL
psql postgres://erp:erp@localhost:5432/erp_platform -c "SELECT 1"

# Redis
redis-cli ping  # Should return PONG

# NATS
nats server check connection  # Should show connected
```

### Option B: Local Installation

If running PostgreSQL locally:

```bash
# Create database
createdb -U postgres erp_platform

# Create user
psql -U postgres -c "CREATE USER erp WITH PASSWORD 'erp'"
psql -U postgres -c "GRANT ALL PRIVILEGES ON DATABASE erp_platform TO erp"
```

### Run Database Migrations

Note: As of v1.0.0, no SQL migrations are included. The database schema is defined in the [13-Low-Level-Design.md](./13-Low-Level-Design.md) document. To apply manually:

```bash
psql postgres://erp:erp@localhost:5432/erp_platform < docs/schema.sql
```

---

## Step 5: Run Services

### Run Subscription Hub (Primary)

```bash
cd services/subscription-hub
go run .
# Output: subscription-hub listening on :8091
```

### Run Other Services

Each service runs independently. Open separate terminal windows:

```bash
# Terminal 2
cd services/tenant-provisioner && PORT=8082 go run .

# Terminal 3
cd services/entitlement-engine && PORT=8083 go run .

# Terminal 4
cd services/module-registry && PORT=8084 go run .

# Terminal 5
cd services/marketplace && PORT=8085 go run .

# Terminal 6
cd services/audit-service && PORT=8086 go run .

# Terminal 7
cd services/notification-hub && PORT=8087 go run .

# Terminal 8
cd services/web-hosting && PORT=8088 go run .

# Terminal 9
cd services/activation-wizard && PORT=8089 go run .
```

### Run All Services with Docker Compose

```bash
docker compose -f infra/docker-compose.platform.yml up
```

---

## Step 6: Verify Setup

### Health Checks

```bash
# Subscription Hub
curl -s http://localhost:8091/healthz | jq
# {"status":"ok","catalog_version":"2026-02-23"}

# Tenant Provisioner (if running on port 8082)
curl -s http://localhost:8082/healthz | jq
# {"status":"healthy","module":"ERP-Platform","service":"tenant-provisioner"}
```

### Product Catalog

```bash
curl -s http://localhost:8091/v1/products | jq '.products | length'
# 48 (20 modules + 28 bundles/standalone wrappers)
```

### Create Test Subscription

```bash
curl -s -X POST http://localhost:8091/v1/subscriptions \
  -H "Content-Type: application/json" \
  -d '{"tenant_id":"test-dev","plan_type":"bundle","skus":["starter"]}' | jq
# {"tenant_id":"test-dev","plan_type":"bundle","skus":["erp-crm","erp-workspace","erp-bi"]}
```

### Query Entitlements

```bash
curl -s http://localhost:8091/v1/entitlements/test-dev | jq
# {"tenant_id":"test-dev","entitlements":["erp-crm","erp-workspace","erp-bi"]}
```

---

## Step 7: Run Tests

```bash
# Unit tests
make test

# Integration tests (requires running infrastructure)
make test-integration

# End-to-end tests
make test-e2e
```

---

## Step 8: Run Frontend (Optional)

### Activation Console

```bash
cd imports/bac_activation/frontend/activation-console
npm install
npm run dev
# Open http://localhost:5173
```

---

## Common Issues

| Issue | Cause | Resolution |
|-------|-------|------------|
| `catalog load failed: open ../../catalog/products.json: no such file or directory` | Running from wrong directory | Run from `services/subscription-hub/` or set `ERP_CATALOG_PATH` environment variable |
| `connection refused` on PostgreSQL | Database not running | Start with `docker compose up -d postgres` |
| `port already in use: 8091` | Another process using the port | Kill the process or use `PORT=8092 go run .` |
| `go: go.mod file not found` | Running from repository root | `cd services/subscription-hub` first |
| Health check returns connection refused | Service not started | Start the service; check port configuration |
| NATS connection failed | NATS not running | Start with `docker compose up -d nats` |
| Permission denied on Docker | Docker daemon not running or user not in docker group | Start Docker Desktop; `sudo usermod -aG docker $USER` on Linux |

---

## IDE Setup

### VS Code

Recommended extensions:
- `golang.go` -- Go language support
- `ms-azuretools.vscode-docker` -- Docker support
- `redhat.vscode-yaml` -- YAML support
- `bierner.markdown-mermaid` -- Mermaid diagram preview

### GoLand

- Set GOROOT to Go 1.22 installation.
- Mark `services/subscription-hub` as a Go module root.
- Configure run configurations for each service.

---

*For contribution guidelines, see [19-CONTRIBUTING.md](./19-CONTRIBUTING.md). For the project README, see [17-README.md](./17-README.md).*
