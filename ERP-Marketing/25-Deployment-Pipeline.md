# ERP-Marketing -- Deployment Pipeline

## 1. Pipeline Overview

```mermaid
flowchart LR
    DEV[Developer Push] --> CI[CI Pipeline]
    CI --> TEST[Test Stage]
    TEST --> BUILD[Build Stage]
    BUILD --> DOCKER[Docker Stage]
    DOCKER --> STAGING[Staging Deploy]
    STAGING --> QA[QA Validation]
    QA --> PROD[Production Deploy]
```

## 2. CI Pipeline (GitHub Actions)

The CI pipeline runs on every push to `main` and `develop` branches, and on all pull requests targeting `main`.

### 2.1 Test Job

```yaml
Trigger: push (main, develop), pull_request (main)
Runner: ubuntu-latest
Services: PostgreSQL 16
Steps:
  1. Checkout code
  2. Install Rust (stable) with clippy and rustfmt
  3. Cache cargo dependencies
  4. Check formatting: cargo fmt -- --check
  5. Lint: cargo clippy -- -D warnings
  6. Run tests: cargo test --all-features
Environment:
  DATABASE_URL: postgres://postgres:postgres@localhost:5432/crm_test
  CARGO_TERM_COLOR: always
  RUST_BACKTRACE: 1
```

### 2.2 Build Job

```yaml
Trigger: After test job succeeds
Runner: ubuntu-latest
Steps:
  1. Checkout code
  2. Install Rust (stable)
  3. Cache cargo dependencies
  4. Build release binary: cargo build --release
```

### 2.3 Docker Job

```yaml
Trigger: After test job succeeds, only on main branch
Runner: ubuntu-latest
Steps:
  1. Checkout code
  2. Set up Docker Buildx
  3. Login to GHCR (ghcr.io)
  4. Build and push multi-layer image:
     - Tag: ghcr.io/<repo>:latest
     - Tag: ghcr.io/<repo>:<commit-sha>
     - Cache: GitHub Actions cache (gha)
```

### 2.4 Pipeline Diagram

```mermaid
flowchart TB
    subgraph "Test Job"
        FMT[cargo fmt --check] --> CLIPPY[cargo clippy -D warnings]
        CLIPPY --> UNIT[cargo test --all-features]
    end
    subgraph "Build Job"
        REL[cargo build --release]
    end
    subgraph "Docker Job (main only)"
        BUILDX[Docker Buildx Setup] --> LOGIN[GHCR Login]
        LOGIN --> PUSH[Build + Push Image]
    end
    UNIT --> REL
    UNIT --> BUILDX
```

## 3. Docker Build Process

### 3.1 Multi-Stage Dockerfile

```mermaid
flowchart TB
    subgraph "Stage 1: Builder"
        BASE1[rust:1.75-bookworm]
        DEPS[Copy Cargo.toml + build deps]
        SRC[Copy source code]
        BUILD[cargo build --release]
    end
    subgraph "Stage 2: Runtime"
        BASE2[debian:bookworm-slim]
        CERTS[Install ca-certificates + libssl3]
        BINARY[Copy release binary]
        USER[Create non-root user]
        RUN[ENTRYPOINT binary]
    end
    BASE1 --> DEPS --> SRC --> BUILD
    BUILD --> BINARY
    BASE2 --> CERTS --> BINARY --> USER --> RUN
```

**Key Features:**
- Dependency caching: Build dependencies first with dummy `main.rs`, then copy real source
- Minimal runtime image: `debian:bookworm-slim` (~80MB base)
- Non-root execution: `appuser` with `/bin/false` shell
- Port exposure: 8086
- Log level: INFO by default

## 4. Kubernetes Deployment

### 4.1 Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: marketing-api
  namespace: marketing
spec:
  replicas: 3
  selector:
    matchLabels:
      app: marketing-api
  template:
    spec:
      containers:
        - name: marketing-api
          image: ghcr.io/<repo>:latest
          ports:
            - containerPort: 8086
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: marketing-db-secret
                  key: url
            - name: RUST_LOG
              value: "info"
            - name: PORT
              value: "8086"
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /health
              port: 8086
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /health
              port: 8086
            initialDelaySeconds: 5
            periodSeconds: 10
```

### 4.2 Service Manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: marketing-api
  namespace: marketing
spec:
  selector:
    app: marketing-api
  ports:
    - port: 8086
      targetPort: 8086
  type: ClusterIP
```

### 4.3 Storage

```yaml
storageClassName: mayastor
# or vitastor-compatible class
```

## 5. Deployment Strategies

### 5.1 Rolling Update (Default)

```mermaid
flowchart LR
    V1_A[Pod v1 A] --> V2_A[Pod v2 A]
    V1_B[Pod v1 B] --> V2_B[Pod v2 B]
    V1_C[Pod v1 C] --> V2_C[Pod v2 C]
```

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

### 5.2 Blue-Green (Major Releases)

```mermaid
flowchart TB
    LB[Load Balancer] --> BLUE[Blue: v1.0<br/>3 pods]
    LB -.-> GREEN[Green: v2.0<br/>3 pods]
    VERIFY[Verify Green] --> SWITCH[Switch Traffic]
    SWITCH --> LB2[Load Balancer]
    LB2 --> GREEN2[Green: v2.0<br/>3 pods]
```

## 6. Database Migration Strategy

```mermaid
flowchart TB
    DEPLOY[Deploy New Version] --> MIGRATE[Run Migrations<br/>sqlx::migrate!]
    MIGRATE --> CHECK{Migration Success?}
    CHECK -->|Yes| READY[Service Ready]
    CHECK -->|No| ROLLBACK[Rollback Deployment]
    ROLLBACK --> INVESTIGATE[Investigate Migration Failure]
```

Migrations are embedded at compile time via `sqlx::migrate!("./migrations")` and run automatically on service startup. All migrations are forward-only.

## 7. Environment Promotion

```mermaid
flowchart LR
    DEV[Development<br/>docker compose] --> STAGING[Staging<br/>k8s namespace: marketing-staging]
    STAGING --> PROD[Production<br/>k8s namespace: marketing]
```

| Environment | Purpose | Database | Approval |
|---|---|---|---|
| Development | Local development | Local PostgreSQL | None |
| Staging | Pre-production validation | Staging DB | Automated tests |
| Production | Live service | Production DB | Manual approval |

## 8. Rollback Procedures

### 8.1 Quick Rollback

```bash
# Rollback to previous deployment
kubectl rollout undo deployment/marketing-api -n marketing

# Rollback to specific revision
kubectl rollout undo deployment/marketing-api --to-revision=3 -n marketing
```

### 8.2 Database Rollback

Since migrations are forward-only, database rollback requires:
1. Restore from the latest backup
2. Re-apply migrations up to the target version
3. This is a P1 procedure and requires DBa approval

## 9. Monitoring During Deployment

During deployment, monitor:
- Pod rollout status: `kubectl rollout status deployment/marketing-api`
- Health check responses: `/health` endpoint
- Error rate in Quickwit logs
- Pulsar topic lag (should not grow during deployment)
- API p95 latency (should not increase during deployment)

## 10. Deployment Checklist

- [ ] All CI checks pass (fmt, clippy, test, build, docker)
- [ ] Changelog updated
- [ ] Migration scripts tested in staging
- [ ] AIDD guardrail configuration verified
- [ ] Rollback plan documented
- [ ] Monitoring dashboards open
- [ ] On-call engineer available
- [ ] Deployment window communicated to team
