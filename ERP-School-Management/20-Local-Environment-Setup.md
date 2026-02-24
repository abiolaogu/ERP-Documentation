# ERP-School-Management -- Local Environment Setup

**Product:** EduCore Pro
**Version:** 1.0.0
**Date:** 2026-02-23

---

## 1. Prerequisites

### Required Software

| Software | Minimum Version | Purpose | Install Command |
|---|---|---|---|
| Node.js | 20.0.0 | NestJS/Next.js runtime | `brew install node` or [nvm](https://github.com/nvm-sh/nvm) |
| npm | 9.0.0 | Package manager | Bundled with Node.js |
| Docker | 24.0 | Container runtime | [Docker Desktop](https://docker.com) |
| Docker Compose | 2.20 | Multi-container orchestration | Bundled with Docker Desktop |
| Git | 2.40+ | Version control | `brew install git` |
| Go | 1.22+ | Scholarship service | `brew install go` |
| Rust | 1.75+ | Placement/research services | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Python | 3.11+ | AI service | `brew install python@3.11` |

### Optional Software

| Software | Purpose |
|---|---|
| VS Code | Recommended IDE |
| Postman | API testing |
| pgAdmin 4 | Database management |
| Flutter SDK | Mobile app development |

### System Requirements

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 16 GB |
| Disk | 20 GB free | 50 GB free |
| OS | macOS 13+ / Ubuntu 22.04+ / Windows 11 (WSL2) | macOS 14+ / Ubuntu 24.04 |

---

## 2. Repository Setup

```bash
# Clone the repository
git clone <repo-url>
cd ERP-School-Management

# Verify Node.js version
node --version  # Should be >= 20.0.0

# Install all workspace dependencies
npm install
```

### Monorepo Structure After Install

```mermaid
graph TD
    ROOT["ERP-School-Management/"] --> APPS["apps/ (5 frontend apps)"]
    ROOT --> SVC["services/ (25 microservices)"]
    ROOT --> PKG["packages/ (7 shared packages)"]
    ROOT --> GW["gateway/ (ERP gateway)"]
    ROOT --> INFRA["infra/ (infrastructure)"]
    ROOT --> DB["database/ (migrations)"]
    ROOT --> PROTO["proto/ (protobuf)"]
    ROOT --> NM["node_modules/ (workspace hoisted)"]
```

---

## 3. Infrastructure Setup

### 3.1 Start All Infrastructure

```bash
# Start database, event streaming, and observability
docker compose up -d lumadb redpanda otel-collector grafana superset
```

### 3.2 Verify Infrastructure Health

```bash
# PostgreSQL
docker compose exec lumadb pg_isready
# Expected: /var/run/postgresql:5432 - accepting connections

# Redpanda
docker compose exec redpanda rpk cluster health
# Expected: Healthy

# Check all containers
docker compose ps
```

### 3.3 Connection Details

| Service | Host | Port | Credentials |
|---|---|---|---|
| PostgreSQL | localhost | 5434 | User: `erp`, Password: `erp`, DB: `erp_school_management` |
| Redpanda | localhost | 19093 | No auth (development) |
| Grafana | localhost | 3000 | Default: admin/admin |
| Superset | localhost | 8088 | Default: admin/admin |
| OTel Collector (gRPC) | localhost | 4317 | No auth |
| OTel Collector (HTTP) | localhost | 4318 | No auth |

---

## 4. Database Setup

### 4.1 Run Migrations

```bash
# Run the initial schema migration
npm run db:migrate

# Seed demo data
npm run db:seed

# Or manually apply the SQL migration
docker compose exec -T lumadb psql -U erp -d erp_school_management < database/migrations/001_initial_schema.sql
```

### 4.2 Per-Service Prisma Migrations

```bash
# Auth service
cd services/auth-service
npx prisma migrate dev
npx prisma generate

# Student service
cd services/student-service
npx prisma migrate dev
npx prisma generate

# Academic service
cd services/academic-service
npx prisma generate

# Finance service
cd services/finance-service
npx prisma generate

# LMS service
cd services/lms-service
npx prisma migrate dev
npx prisma generate
```

### 4.3 Connect with pgAdmin

1. Open pgAdmin 4
2. Add server: Host=localhost, Port=5434, User=erp, Password=erp
3. Navigate to `erp_school_management` database
4. Browse tables under the `public` schema

---

## 5. Service Development

### 5.1 Start All Services (Development Mode)

```bash
# Start all services with hot reload via Turborepo
npm run dev
```

### 5.2 Start Individual Services

```bash
# Gateway
cd gateway && npm run start

# Specific service
cd services/auth-service && npm run start:dev

# Web app
cd apps/web && npm run dev
```

### 5.3 Start All Services via Docker

```bash
# Build and start everything
docker compose up --build

# Gateway will be available at http://localhost:8092
```

---

## 6. Environment Variables

### 6.1 Root Environment

Create `.env` in the project root:

```bash
# Database
DATABASE_URL=postgres://erp:erp@localhost:5434/erp_school_management

# Event Streaming
REDPANDA_BROKERS=localhost:19093

# Observability
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318

# ERP Suite Integration
ERP_PLATFORM_BASE_URL=http://localhost:8091
ALLOW_ON_ENTITLEMENT_FAILURE=true

# Gateway
PORT=8090
```

### 6.2 Service-Specific Variables

Each service reads from the shared `DATABASE_URL`, `REDPANDA_BROKERS`, and `OTEL_EXPORTER_OTLP_ENDPOINT` environment variables. Additional service-specific variables:

```bash
# Auth Service
JWT_SECRET=your-development-secret-key
JWT_EXPIRATION=15m
REFRESH_TOKEN_EXPIRATION=7d
MFA_ISSUER=EduCorePro

# Finance Service
STRIPE_SECRET_KEY=sk_test_...
PAYSTACK_SECRET_KEY=sk_test_...
FLUTTERWAVE_SECRET_KEY=FLWSECK_TEST-...

# Communication Service
TWILIO_SID=AC...
TWILIO_AUTH_TOKEN=...
SENDGRID_API_KEY=SG...

# AI Service
OPENAI_API_KEY=sk-...
MODEL_PATH=/models/
```

---

## 7. Testing Locally

```bash
# Run all tests
npm run test

# Run tests for a specific service
cd services/student-service && npm run test

# Run with coverage
npm run test -- --coverage

# Run gateway tests
cd gateway && npm run test

# Run Go tests
make test

# Integration tests
make test-integration

# E2E tests
make test-e2e
```

---

## 8. Common Development Tasks

### 8.1 Adding a Prisma Model

```bash
# 1. Edit the schema
vim services/<service>/prisma/schema.prisma

# 2. Create migration
cd services/<service>
npx prisma migrate dev --name add_new_model

# 3. Generate client
npx prisma generate

# 4. Restart the service
npm run start:dev
```

### 8.2 Building for Production

```bash
# Build all packages and services
npm run build

# Build Docker images
docker compose build
```

### 8.3 Code Quality

```bash
# Lint all code
npm run lint

# Format all code
npm run format

# Clean build artifacts
npm run clean
```

---

## 9. Troubleshooting

### Common Issues

| Issue | Solution |
|---|---|
| Port 5434 already in use | Stop local PostgreSQL or change port in docker-compose.yml |
| npm install fails | Delete `node_modules` and `package-lock.json`, then retry |
| Prisma generate fails | Ensure DATABASE_URL is set and database is running |
| Docker memory error | Increase Docker Desktop memory allocation to 8GB+ |
| Hot reload not working | Check Turborepo cache: `npm run clean` |
| Gateway connection refused | Ensure ERP_PLATFORM_BASE_URL points to running service |

### Health Checks

```bash
# Gateway health
curl http://localhost:8092/healthz

# Service capabilities
curl http://localhost:8092/v1/capabilities

# Database connectivity
docker compose exec lumadb psql -U erp -d erp_school_management -c "SELECT 1"
```

---

## 10. IDE Configuration

### VS Code Recommended Extensions

```json
{
  "recommendations": [
    "prisma.prisma",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "golang.go",
    "rust-lang.rust-analyzer",
    "ms-python.python"
  ]
}
```

### VS Code Settings

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "typescript.tsdk": "node_modules/typescript/lib",
  "eslint.workingDirectories": [{ "mode": "auto" }],
  "[prisma]": {
    "editor.defaultFormatter": "Prisma.prisma"
  }
}
```
