# ERP-Finance Setup and Installation Guide

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | Setup & Installation Guide |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

## Prerequisites

### Required Software

| Software | Minimum Version | Installation |
|----------|----------------|-------------|
| Go | 1.22+ | `brew install go` or https://go.dev/dl |
| Rust | 1.75+ | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Python | 3.12+ | `brew install python@3.12` |
| PostgreSQL | 16+ | `brew install postgresql@16` |
| Redis | 7+ | `brew install redis` |
| NATS Server | 2.10+ | `brew install nats-server` |
| Docker | 24+ | https://docs.docker.com/get-docker |
| Docker Compose | 2.20+ | Bundled with Docker Desktop |

### Environment Variables

Create a `.env` file in each service directory:

```bash
# Common
DATABASE_URL=postgres://erp_finance:secret@localhost:5432/erp_finance
REDIS_URL=redis://localhost:6379
NATS_URL=nats://localhost:4222

# Billing Service
PORT=8089

# Payments Service
PORT=8084
PAYSTACK_SECRET_KEY=sk_test_xxx
FLUTTERWAVE_SECRET_KEY=FLWSECK_TEST-xxx

# Asset Management
ANTHROPIC_API_KEY=sk-ant-xxx
AI_MODEL=claude-sonnet-4-20250514

# Gateway
PORT=8090
```

## Quick Start with Docker Compose

```yaml
# docker-compose.yml
version: '3.9'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: erp_finance
      POSTGRES_USER: erp_finance
      POSTGRES_PASSWORD: secret
    ports: ["5432:5432"]
    volumes: ["pgdata:/var/lib/postgresql/data"]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  nats:
    image: nats:2.10-alpine
    command: ["-js"]
    ports: ["4222:4222", "8222:8222"]

  clickhouse:
    image: clickhouse/clickhouse-server:24
    ports: ["8123:8123", "9000:9000"]

  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports: ["9000:9000", "9001:9001"]

volumes:
  pgdata:
```

### Starting the Infrastructure

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-Finance
docker-compose up -d
```

## Service-by-Service Installation

### 1. Gateway Service (Go)

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-Finance
go mod download
go build -o server ./cmd/server
./server
# Listening on :8090
```

### 2. Billing Service (Rust)

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-Finance/imports/billing_core
cargo build --release
# Run database migrations
DATABASE_URL=postgres://erp_finance:secret@localhost:5432/erp_finance \
  cargo run --release
# Listening on :8089
```

### 3. Payments Service (Rust)

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-Finance/imports/payments_core
cargo build --release
DATABASE_URL=postgres://erp_finance:secret@localhost:5432/erp_finance \
  cargo run --release
# Listening on :8084
```

### 4. Asset Management Service (Python)

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-Finance/imports/asset_core
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn src.main:app --host 0.0.0.0 --port 8096
# Listening on :8096 with docs at /docs
```

### 5. Sub-Module Services (Go)

Each Go sub-service follows the same pattern:

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-Finance/services/<service-name>
go build -o server ./cmd/server
./server
```

## Database Setup

### Schema Initialization

```mermaid
flowchart TD
    Start["Start Setup"] --> CreateDB["Create Database"]
    CreateDB --> RunMigrations["Run Migrations"]
    RunMigrations --> BillingMig["Billing Migrations<br/>(SQLx)"]
    RunMigrations --> PaymentMig["Payment Migrations<br/>(SQLx)"]
    RunMigrations --> AssetMig["Asset Migrations<br/>(SQLAlchemy)"]
    RunMigrations --> GLMig["GL Migrations<br/>(Go-Migrate)"]
    BillingMig --> SeedData["Seed Reference Data"]
    PaymentMig --> SeedData
    AssetMig --> SeedData
    GLMig --> SeedData
    SeedData --> Verify["Verify Setup"]
    Verify --> Done["Setup Complete"]
```

### PostgreSQL Database Creation

```sql
CREATE DATABASE erp_finance;
CREATE USER erp_finance WITH PASSWORD 'secret';
GRANT ALL PRIVILEGES ON DATABASE erp_finance TO erp_finance;

-- Create schemas for domain isolation
CREATE SCHEMA IF NOT EXISTS gl;
CREATE SCHEMA IF NOT EXISTS ap;
CREATE SCHEMA IF NOT EXISTS ar;
CREATE SCHEMA IF NOT EXISTS billing;
CREATE SCHEMA IF NOT EXISTS payments;
CREATE SCHEMA IF NOT EXISTS assets;
CREATE SCHEMA IF NOT EXISTS tax;
CREATE SCHEMA IF NOT EXISTS expense;
CREATE SCHEMA IF NOT EXISTS treasury;
CREATE SCHEMA IF NOT EXISTS budget;
```

### Billing Service Migrations

The billing service uses SQLx embedded migrations that run automatically on startup:

```bash
# Tables created automatically:
# - plans
# - subscriptions
# - usage_records
# - invoices
# - invoice_items
```

### Payments Service Migrations

```bash
# Tables created automatically:
# - transactions
# - wallets
# - payment_methods
# - refunds
```

### Asset Management Migrations

SQLAlchemy models auto-create tables via `init_db()`:

```bash
# Tables created automatically:
# - assets
# - maintenance_records
# - depreciation_records
# - lifecycle_events
```

## Verification

### Health Check All Services

```bash
# Gateway
curl http://localhost:8090/healthz
# Expected: {"status":"ok","module":"ERP-Finance"}

# Billing
curl http://localhost:8089/health
# Expected: {"status":"healthy","service":"opensase-billing"}

# Payments
curl http://localhost:8084/health
# Expected: {"status":"healthy","service":"opensase-payments","version":"0.1.0"}

# Asset Management
curl http://localhost:8096/health
# Expected: {"status":"healthy"}

# Capabilities
curl http://localhost:8090/v1/capabilities
# Expected: {"module":"ERP-Finance","capabilities":["billing","payments",...]}
```

## Configuration Reference

### Gateway Configuration

File: `configs/capabilities.json`

```json
{
  "module": "ERP-Finance",
  "capabilities": [
    "billing",
    "payments",
    "fixed_assets",
    "general_ledger",
    "ap",
    "ar",
    "tax",
    "expense",
    "treasury",
    "budgeting_forecasting"
  ]
}
```

### Module Dependencies

File: `configs/module_dependencies.yaml`

```yaml
module: ERP-Finance
dependencies:
  - ERP-Billing
  - ERP-Payments
  - ERP-Asset-Management
  - ERP-General-Ledger
  - ERP-AP-AR
  - ERP-Tax-Management
  - ERP-Expense-Management
runtime_contracts:
  entitlements: ERP-Platform
  identity: ERP-IAM
  events: NATS
```

### Module Manifest

File: `erp/module.manifest.yaml`

```yaml
api_version: v1
module_id: erp_finance
repository: ERP-Finance
sku: erp.finance
subscription:
  standalone: true
  suite: true
integration:
  control_plane: ERP-Platform
  identity_provider: ERP-Directory
  event_backbone: NATS
```

## Troubleshooting

| Issue | Solution |
|-------|---------|
| Database connection refused | Ensure PostgreSQL is running: `pg_isready` |
| Billing migrations fail | Check `DATABASE_URL` has correct credentials |
| Asset Management AI errors | Verify `ANTHROPIC_API_KEY` is set and valid |
| NATS connection timeout | Ensure NATS is started with JetStream: `nats-server -js` |
| Port conflicts | Check no other services on ports 8084-8102 |
