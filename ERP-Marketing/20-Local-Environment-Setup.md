# ERP-Marketing -- Local Environment Setup

## Prerequisites

Before setting up the local development environment, ensure the following tools are installed:

| Tool | Version | Installation |
|---|---|---|
| Rust | 1.75+ | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Go | 1.22+ | `brew install go` or [golang.org/dl](https://golang.org/dl) |
| Node.js | 20 LTS | `brew install node@20` or [nvm](https://github.com/nvm-sh/nvm) |
| Docker | 24+ | [Docker Desktop](https://www.docker.com/products/docker-desktop) |
| Docker Compose | v2.x | Included with Docker Desktop |
| PostgreSQL client | 16 | `brew install postgresql@16` (optional, for `psql`) |
| Git | 2.40+ | `brew install git` |

## Step 1: Clone the Repository

```bash
git clone <repository-url> ERP-Marketing
cd ERP-Marketing
```

## Step 2: Start Infrastructure Services

The `docker-compose.yml` provides PostgreSQL and NATS (message broker) services:

```bash
docker compose up -d db nats
```

This starts:
- **PostgreSQL 16** on port `5432` with database `marketing`, user `postgres`, password `postgres`
- **NATS 2.10** with JetStream enabled on port `4222`

Verify services are healthy:

```bash
docker compose ps
```

```mermaid
flowchart LR
    subgraph "Docker Compose"
        PG[PostgreSQL 16<br/>Port 5432]
        NATS[NATS 2.10<br/>Port 4222]
    end
    subgraph "Local Development"
        RUST[Rust API Server<br/>Port 8086]
        WEB[React Dev Server<br/>Port 5173]
    end
    RUST --> PG
    RUST --> NATS
    WEB --> RUST
```

## Step 3: Configure Environment

Copy the example environment file and adjust if needed:

```bash
cp .env.example .env
```

The default `.env` configuration:

```bash
DATABASE_URL=postgres://postgres:postgres@localhost:5432/marketing
RUST_LOG=debug
PORT=8086
```

For Pulsar and Quickwit (optional for local development):

```bash
PULSAR_BROKER_URL=pulsar://localhost:6650
PULSAR_HTTP_URL=http://localhost:8080
PULSAR_TENANT=billyronks
PULSAR_NAMESPACE=extract-marketing
QUICKWIT_ENDPOINT=http://localhost:7280
QUICKWIT_INDEX=logs-extract-marketing
```

## Step 4: Run Database Migrations

Migrations run automatically when the Rust server starts, but you can also run them manually:

```bash
# The server runs migrations on startup
cargo run
# Migrations from ./migrations/ are applied automatically
```

## Step 5: Start the Rust API Server

```bash
cargo run
```

Expected output:
```
INFO  OpenSASE Marketing listening on 0.0.0.0:8086
```

Verify the server is running:
```bash
curl http://localhost:8086/health
# {"status":"healthy","service":"opensase-marketing"}
```

## Step 6: Start the Web Frontend

```bash
cd web
npm install
npm run dev
```

The frontend starts at `http://localhost:5173`.

## Step 7: Start Domain Microservices (Optional)

Each Go microservice can be started independently:

```bash
# Campaign service
cd services/campaign-service
PORT=8081 MODULE_NAME=ERP-Marketing go run main.go

# Journey service
cd services/journey-service
PORT=8082 MODULE_NAME=ERP-Marketing go run main.go

# ... repeat for other services
```

## Step 8: Run Tests

### Rust Tests

```bash
# Run all tests
cargo test --all-features

# Run with output
cargo test --all-features -- --nocapture

# Run specific test
cargo test test_campaign
```

### Frontend Tests

```bash
cd web
npm test           # Run vitest
npm run typecheck  # TypeScript checking
npm run lint       # ESLint
```

### All Tests

```bash
make test:all
```

## Step 9: Code Quality Checks

```bash
# Format code
cargo fmt

# Lint check
cargo clippy -- -D warnings

# Frontend lint
cd web && npm run lint
```

## Step 10: Build Docker Image

```bash
docker build -t erp-marketing:local .
```

Or use Docker Compose to build and run everything:

```bash
docker compose up --build
```

## Full Stack Local Setup (Docker Compose)

```bash
# Build and start all services
docker compose up -d

# View logs
docker compose logs -f marketing

# Stop all services
docker compose down

# Stop and remove data
docker compose down -v
```

## GraphQL Code Generation

To regenerate GraphQL types from the schema:

```bash
# Pull schema
make schema:pull

# Generate code for all platforms
make codegen:all
```

## Troubleshooting

### Database Connection Refused

```bash
# Check if PostgreSQL is running
docker compose ps db
# Verify connection
psql -h localhost -U postgres -d marketing -c "SELECT 1"
```

### Port Already in Use

```bash
# Check what is using port 8086
lsof -i :8086
# Kill the process or change PORT in .env
```

### Rust Compilation Errors

```bash
# Clean and rebuild
cargo clean
cargo build
```

### Frontend Build Errors

```bash
# Clear node_modules and reinstall
cd web
rm -rf node_modules
npm install
```

## IDE Setup

### VS Code Recommended Extensions

- `rust-analyzer` -- Rust language server
- `Even Better TOML` -- Cargo.toml support
- `crates` -- Dependency management
- `ESLint` -- JavaScript/TypeScript linting
- `Prettier` -- Code formatting

### IntelliJ / CLion

- Install Rust plugin
- Set project SDK to Rust 1.75+
- Configure cargo as build tool

## Environment Variable Reference

| Variable | Default | Description |
|---|---|---|
| `DATABASE_URL` | `postgres://postgres:postgres@localhost:5432/marketing` | PostgreSQL connection string |
| `RUST_LOG` | `info` | Log level (debug, info, warn, error) |
| `PORT` | `8086` | API server port |
| `NATS_URL` | `nats://localhost:4222` | NATS connection URL |
| `PULSAR_BROKER_URL` | `pulsar://localhost:6650` | Pulsar broker URL |
| `PULSAR_TENANT` | `billyronks` | Pulsar tenant name |
| `PULSAR_NAMESPACE` | `extract-marketing` | Pulsar namespace |
| `QUICKWIT_ENDPOINT` | `http://localhost:7280` | Quickwit search endpoint |
| `QUICKWIT_INDEX` | `logs-extract-marketing` | Quickwit index name |
| `STORAGE_CLASS` | `mayastor` | Kubernetes storage class |
