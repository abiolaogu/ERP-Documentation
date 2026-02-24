# ERP-CRM Deployment Pipeline

## Pipeline Overview

```mermaid
flowchart LR
    DEV2["Developer<br/>Push"] --> CI2["CI Pipeline<br/>(GitHub Actions)"]
    CI2 --> TEST2["Test Stage<br/>fmt + clippy + test"]
    TEST2 --> BUILD2["Build Stage<br/>cargo build --release"]
    BUILD2 --> DOCKER2["Docker Stage<br/>Build + Push to GHCR"]
    DOCKER2 --> STAGING["Staging<br/>Deploy + Smoke Test"]
    STAGING --> PROD["Production<br/>Rolling Update"]
```

## GitHub Actions CI/CD

### Trigger Configuration

```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
```

### Pipeline Stages

```mermaid
graph TB
    subgraph "Test Job"
        CHECKOUT1["Checkout Code"]
        RUST1["Install Rust<br/>(stable + clippy + rustfmt)"]
        CACHE1["Restore Cargo Cache"]
        FMT["cargo fmt -- --check"]
        CLIPPY["cargo clippy -- -D warnings"]
        TEST3["cargo test --all-features"]
    end

    subgraph "Build Job"
        CHECKOUT2["Checkout Code"]
        RUST2["Install Rust"]
        CACHE2["Restore Cargo Cache"]
        BUILD3["cargo build --release"]
    end

    subgraph "Docker Job (main only)"
        CHECKOUT3["Checkout Code"]
        BUILDX["Setup Docker Buildx"]
        LOGIN["Login to GHCR"]
        PUSH["Build + Push Image<br/>:latest + :sha"]
    end

    CHECKOUT1 --> RUST1 --> CACHE1 --> FMT --> CLIPPY --> TEST3
    TEST3 -->|passed| CHECKOUT2
    CHECKOUT2 --> RUST2 --> CACHE2 --> BUILD3
    TEST3 -->|passed, main branch| CHECKOUT3
    CHECKOUT3 --> BUILDX --> LOGIN --> PUSH
```

### CI Service Dependencies

The test job spins up a PostgreSQL service container:

```yaml
services:
  postgres:
    image: postgres:16
    env:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: crm_test
    ports:
      - 5432:5432
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
```

### Docker Image Build

Multi-stage build produces minimal production images:

```mermaid
graph LR
    BUILDER["Stage 1: Builder<br/>rust:1.75-bookworm<br/>cargo build --release"] --> RUNTIME["Stage 2: Runtime<br/>debian:bookworm-slim<br/>~50MB final image"]
```

Key features:
- Dependency caching via dummy `src/main.rs` build
- Non-root user (`appuser`) in production
- Migrations copied into image
- Minimal runtime dependencies (ca-certificates, libssl3)

### Image Tagging Strategy

| Tag | When | Purpose |
|-----|------|---------|
| `ghcr.io/{repo}:latest` | Main branch push | Latest stable |
| `ghcr.io/{repo}:{sha}` | Every main push | Immutable, audit-friendly |
| `ghcr.io/{repo}:v{version}` | Release tag | Semantic version |

## Go Microservice Deployment

Each microservice has its own Dockerfile:

```dockerfile
# Standard Go microservice Dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY main.go .
RUN go build -o service main.go

FROM alpine:3.19
COPY --from=builder /app/service /app/service
ENTRYPOINT ["/app/service"]
```

## Deployment Strategies

### Rolling Update (Default)

```mermaid
sequenceDiagram
    participant LB as Load Balancer
    participant V1a as Pod v1 (a)
    participant V1b as Pod v1 (b)
    participant V2a as Pod v2 (a)
    participant V2b as Pod v2 (b)

    Note over LB,V1b: Initial State: 2 replicas v1
    LB->>V1a: Traffic
    LB->>V1b: Traffic

    Note over V2a: Deploy v2 pod (a)
    V2a->>V2a: Health check passes
    LB->>V2a: Traffic started
    V1a->>V1a: Drain and terminate

    Note over V2b: Deploy v2 pod (b)
    V2b->>V2b: Health check passes
    LB->>V2b: Traffic started
    V1b->>V1b: Drain and terminate

    Note over LB,V2b: Final State: 2 replicas v2
```

### Rollback Procedure

```bash
# Kubernetes rollback
kubectl rollout undo deployment/crm-core -n crm

# Docker Compose rollback
docker compose pull  # pulls previous :latest
docker compose up -d

# Specific version rollback
docker compose up -d --build  # with pinned version in docker-compose.yml
```

## Database Migration Strategy

```mermaid
flowchart TB
    DEPLOY_START([Deployment Start]) --> MIGRATE["Run Migrations<br/>(automatic on startup)"]
    MIGRATE --> MIGRATE_OK{Migrations<br/>Succeeded?}
    MIGRATE_OK -->|Yes| START_APP["Start Application"]
    MIGRATE_OK -->|No| ROLLBACK_DEPLOY["Rollback Deployment"]
    START_APP --> HEALTH_OK{Health Check<br/>Passes?}
    HEALTH_OK -->|Yes| TRAFFIC["Accept Traffic"]
    HEALTH_OK -->|No| ROLLBACK_DEPLOY
```

Rules:
1. Migrations are forward-only (no down migrations)
2. All migrations are idempotent (`CREATE TABLE IF NOT EXISTS`, `ON CONFLICT DO NOTHING`)
3. Migrations run before the application accepts traffic
4. Schema changes must be backward-compatible (add columns, never remove)

## Environment Configuration

### Environment Hierarchy

```mermaid
graph TB
    DEV3["Development<br/>.env file<br/>docker-compose.yml"]
    STAGING2["Staging<br/>K8s ConfigMap/Secret<br/>staging namespace"]
    PROD2["Production<br/>K8s ConfigMap/Secret<br/>production namespace"]

    DEV3 --> STAGING2 --> PROD2
```

### Configuration Per Environment

| Variable | Development | Staging | Production |
|----------|------------|---------|-----------|
| DATABASE_URL | localhost:5432/crm | staging-pg:5432/crm | prod-pg:5432/crm |
| RUST_LOG | debug | info | info,opensase_crm=info |
| PORT | 8081 | 8081 | 8081 |
| DATABASE_MAX_CONNECTIONS | 5 | 10 | 25 |
| NATS_URL | localhost:4222 | staging-nats:4222 | prod-nats:4222 |

## Monitoring Post-Deploy

After each deployment, verify:

```bash
# 1. Health check
curl -f http://crm-service:8081/health

# 2. Readiness check
curl -f http://crm-service:8081/ready

# 3. Smoke test: create and retrieve a contact
CONTACT=$(curl -s -X POST http://crm-service:8081/api/v1/contacts \
  -H "Content-Type: application/json" \
  -d '{"email": "smoke-test@example.com"}')
ID=$(echo $CONTACT | jq -r '.id')
curl -f http://crm-service:8081/api/v1/contacts/$ID
# Clean up
curl -X DELETE http://crm-service:8081/api/v1/contacts/$ID

# 4. Check error rates in Quickwit
# 5. Check event publishing in NATS/Pulsar
```

## Quality Gates

| Gate | Criteria | Enforcement |
|------|----------|------------|
| Code Quality | `cargo fmt -- --check` passes | CI blocks merge |
| Linting | `cargo clippy -- -D warnings` passes | CI blocks merge |
| Unit Tests | `cargo test --all-features` passes | CI blocks merge |
| Build | `cargo build --release` succeeds | CI blocks merge |
| Docker Build | Image builds successfully | CI blocks deploy |
| Health Check | `/health` returns 200 | K8s readiness gate |
| DB Ready | `/ready` returns 200 | K8s readiness gate |
