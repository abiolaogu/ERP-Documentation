# ERP-CRM Local Environment Setup

## Prerequisites

### Required Software

| Software | Version | Installation |
|----------|---------|-------------|
| Rust | 1.75+ (2021 edition) | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Docker | 24+ | https://docs.docker.com/get-docker/ |
| Docker Compose | v2+ | Included with Docker Desktop |
| Go | 1.21+ | https://go.dev/dl/ |
| Node.js | 18+ | https://nodejs.org/ |

### Optional Software

| Software | Purpose | Installation |
|----------|---------|-------------|
| sqlx-cli | Database migrations | `cargo install sqlx-cli --no-default-features --features postgres` |
| Flutter | Mobile development | https://flutter.dev/docs/get-started/install |
| Android Studio | Android development | https://developer.android.com/studio |
| Xcode | iOS development | Mac App Store |
| psql | Database CLI | `brew install postgresql` (macOS) |

## Setup Steps

### Step 1: Clone the Repository

```bash
git clone https://github.com/opensase/ERP-CRM.git
cd ERP-CRM
```

### Step 2: Configure Environment

```bash
# Copy the example environment file
cp .env.example .env
```

Edit `.env` with your local settings:

```env
# Required
DATABASE_URL=postgres://postgres:postgres@localhost:5432/crm

# Optional
PORT=8081
HOST=0.0.0.0
NATS_URL=nats://localhost:4222
RUST_LOG=debug,opensase_crm=debug
DATABASE_MAX_CONNECTIONS=10

# Platform integration (optional)
# SASE_API_URL=https://platform.example.com
# SASE_API_KEY=your-api-key
# SASE_TENANT_ID=your-tenant-id

# Pulsar (optional, for event streaming)
# PULSAR_BROKER_URL=pulsar://localhost:6650
# PULSAR_TENANT=billyronks
# PULSAR_NAMESPACE=extract-crm

# Quickwit (optional, for log search)
# QUICKWIT_ENDPOINT=http://localhost:7280
# QUICKWIT_INDEX=logs-extract-crm
```

### Step 3: Start Infrastructure with Docker Compose

```bash
# Start PostgreSQL and NATS
docker compose up -d

# Verify services are running
docker compose ps
```

```mermaid
graph TB
    subgraph "Docker Compose Services"
        CRM_SVC["crm<br/>Port: 8081<br/>(Rust application)"]
        DB_SVC["db<br/>Port: 5432<br/>(PostgreSQL 16)"]
        NATS_SVC["nats<br/>Port: 4222, 8222<br/>(NATS JetStream)"]
    end

    CRM_SVC --> DB_SVC
    CRM_SVC --> NATS_SVC
```

Expected output:
```
NAME    STATUS          PORTS
crm     Up (healthy)    0.0.0.0:8081->8081/tcp
db      Up (healthy)    0.0.0.0:5432->5432/tcp
nats    Up              0.0.0.0:4222->4222/tcp, 0.0.0.0:8222->8222/tcp
```

### Step 4: Run Without Docker (Manual)

If you prefer to run the Rust application directly:

```bash
# Ensure PostgreSQL is running (via Docker or local install)
docker compose up -d db nats

# Set environment variables
export DATABASE_URL=postgres://postgres:postgres@localhost:5432/crm
export NATS_URL=nats://localhost:4222
export RUST_LOG=debug,opensase_crm=debug
export PORT=8081

# Build the project
cargo build

# Run database migrations (automatic on startup, or manual)
# sqlx migrate run --source ./migrations

# Run the application
cargo run
```

Expected startup output:
```
2026-02-23T10:00:00.000Z  INFO opensase_crm: Starting OpenSASE CRM service...
2026-02-23T10:00:00.001Z  INFO opensase_crm: Connecting to database...
2026-02-23T10:00:00.050Z  INFO opensase_crm: Running database migrations...
2026-02-23T10:00:00.100Z  INFO opensase_crm: Connecting to NATS at nats://localhost:4222...
2026-02-23T10:00:00.110Z  INFO opensase_crm: Connected to NATS
2026-02-23T10:00:00.111Z  INFO opensase_crm: OpenSASE CRM listening on 0.0.0.0:8081
```

### Step 5: Verify Installation

```bash
# Health check
curl -s http://localhost:8081/health | jq
# {"status":"healthy","service":"opensase-crm","version":"0.1.0"}

# Readiness check
curl -s http://localhost:8081/ready
# ready

# Create a test contact
curl -s -X POST http://localhost:8081/api/v1/contacts \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","first_name":"Test","last_name":"User"}' | jq

# List contacts
curl -s http://localhost:8081/api/v1/contacts | jq

# Dashboard stats
curl -s http://localhost:8081/api/v1/dashboard/stats | jq
```

### Step 6: Start Go Microservices (Optional)

Each microservice can be started independently:

```bash
# Start contact service
cd services/contact-service
PORT=8082 MODULE_NAME=ERP-CRM go run main.go

# Start helpdesk service
cd services/helpdesk-service
PORT=8083 MODULE_NAME=ERP-CRM go run main.go

# Start any other service
cd services/{name}-service
PORT={port} MODULE_NAME=ERP-CRM go run main.go
```

### Step 7: Web Frontend Setup (Optional)

```bash
cd web

# Install dependencies
npm install

# Generate GraphQL types
npx graphql-codegen

# Start development server
npm run dev
# Available at http://localhost:5173
```

### Step 8: Flutter Mobile Setup (Optional)

```bash
cd flutter

# Get dependencies
flutter pub get

# Run on connected device or emulator
flutter run
```

## Database Management

### Connect to PostgreSQL

```bash
# Via psql
psql postgres://postgres:postgres@localhost:5432/crm

# Via Docker
docker compose exec db psql -U postgres -d crm
```

### Useful Queries

```sql
-- List all tables
\dt

-- Count records
SELECT 'contacts' as table_name, COUNT(*) FROM contacts
UNION ALL
SELECT 'companies', COUNT(*) FROM companies
UNION ALL
SELECT 'deals', COUNT(*) FROM deals
UNION ALL
SELECT 'activities', COUNT(*) FROM activities
UNION ALL
SELECT 'pipelines', COUNT(*) FROM pipelines
UNION ALL
SELECT 'pipeline_stages', COUNT(*) FROM pipeline_stages;

-- View default pipeline stages
SELECT ps.name, ps.position, ps.probability
FROM pipeline_stages ps
JOIN pipelines p ON ps.pipeline_id = p.id
WHERE p.is_default = true
ORDER BY ps.position;
```

### Reset Database

```bash
# Drop and recreate
docker compose down -v
docker compose up -d
# Migrations run automatically on CRM startup
```

## Running Tests

```bash
# All tests
cargo test --all-features

# With detailed output
cargo test -- --nocapture

# Specific test
cargo test domain::aggregates::contact::tests::test_qualify_contact

# Check formatting
cargo fmt -- --check

# Lint
cargo clippy -- -D warnings
```

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|---------|
| `DATABASE_URL must be set` | Missing env var | Set `DATABASE_URL` or create `.env` file |
| `Connection refused (port 5432)` | PostgreSQL not running | Run `docker compose up -d db` |
| `Failed to connect to NATS` | NATS not running | Run `docker compose up -d nats` (optional) |
| `cargo build` fails | Missing Rust toolchain | Install Rust via rustup |
| `sqlx compile-time error` | DB schema mismatch | Run migrations or set `SQLX_OFFLINE=true` |
| Port 8081 in use | Another process | Change `PORT` env var or kill conflicting process |

### Log Levels

Control logging via `RUST_LOG`:

```bash
# Debug logging for CRM, info for everything else
RUST_LOG=info,opensase_crm=debug cargo run

# Trace logging (very verbose)
RUST_LOG=trace cargo run

# Only errors
RUST_LOG=error cargo run
```

### Performance Profiling

```bash
# Build with release optimizations
cargo build --release

# Run performance profile script
bash scripts/perf/profile.sh

# Run benchmarks
cargo bench
```

## IDE Setup

### VS Code

Recommended extensions:
- `rust-analyzer` -- Rust language server
- `crates` -- Cargo.toml dependency management
- `Even Better TOML` -- TOML syntax highlighting
- `REST Client` -- API testing
- `Docker` -- Docker file support
- `PostgreSQL` -- Database browsing

### IntelliJ IDEA

- Install the Rust plugin
- Set up database connection to `localhost:5432/crm`
- Configure run configuration with environment variables
