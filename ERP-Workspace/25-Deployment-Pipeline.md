# ERP-Workspace Deployment Pipeline

> **Document ID:** ERP-WS-DP-025
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## 1. Pipeline Overview

```mermaid
flowchart LR
    commit["Git Commit"]
    commit --> ci["CI Pipeline"]

    subgraph ci["CI Pipeline (GitHub Actions)"]
        lint["Lint<br/>golangci-lint, clippy, ruff"]
        unit_test["Unit Tests<br/>go test, cargo test, pytest"]
        integration_test["Integration Tests<br/>Testcontainers"]
        security_scan["Security Scan<br/>Semgrep, Trivy"]
        build_images["Build Images<br/>Docker multi-stage"]
        push_registry["Push to Registry<br/>tag: git-sha + semver"]
    end

    lint --> unit_test --> integration_test --> security_scan --> build_images --> push_registry

    push_registry --> deploy_staging["Deploy Staging<br/>Helm upgrade"]
    deploy_staging --> e2e["E2E Tests<br/>Playwright"]
    e2e --> approval["Manual Approval"]
    approval --> deploy_prod["Deploy Production<br/>Rolling Update"]
    deploy_prod --> smoke["Smoke Tests"]
    smoke --> monitor["Monitor<br/>30 min error rate"]
```

---

## 2. Build Stage

### 2.1 Go Services Build

```yaml
# Simplified GitHub Actions workflow
jobs:
  build-go:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [email, calendar, meet, chat, docs, drive, contacts]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.22'
      - run: golangci-lint run ./services/${{ matrix.service }}-service/...
      - run: go test ./services/${{ matrix.service }}-service/...
      - run: docker build -t erp-workspace-${{ matrix.service }}:${{ github.sha }}
             -f services/${{ matrix.service }}-service/Dockerfile .
      - run: docker push registry.erp.io/erp-workspace-${{ matrix.service }}:${{ github.sha }}
```

### 2.2 Rust Mail Server Build

```yaml
  build-rust:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      - run: cargo clippy --all-targets -- -D warnings
      - run: cargo test
      - run: cargo build --release
      - run: docker build -t erp-workspace-smtp:${{ github.sha }} .
```

### 2.3 Python AI Service Build

```yaml
  build-python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install ruff pytest
      - run: ruff check .
      - run: pytest tests/
      - run: docker build -t erp-workspace-ai:${{ github.sha }} .
```

---

## 3. Database Migration

```mermaid
flowchart TB
    deploy["Deployment Starts"]
    deploy --> check["Check pending migrations"]
    check -->|"None"| skip["Skip migration step"]
    check -->|"Pending"| backup["Backup current schema"]
    backup --> migrate["Run migrations<br/>(12 migration files)"]
    migrate --> verify["Verify schema integrity"]
    verify -->|"Pass"| continue["Continue deployment"]
    verify -->|"Fail"| rollback["Rollback to backup"]
    rollback --> alert["Alert operations team"]
```

Migration order:
1. `0001_init.sql` -- Tenants, mailboxes, provisioning
2. `0002_add_mailflex_schema.sql` -- Domains, audit logs
3. `0003_email_service_enhancements.sql` -- Templates, campaigns, events, webhooks
4. `0004_performance_indexes.sql` -- Covering and BRIN indexes
5. `0005_contacts_and_enhanced_email.sql` -- Contacts, rules, shared mailboxes
6. `0006_calendar_and_tasks.sql` -- Calendars, events, rooms, tasks
7. `0007_storage_and_chat.sql` -- Files, libraries, teams, conversations
8. `0008_knowledge_base.sql` -- Wikis, notes, custom lists
9. `0009_branding_and_i18n.sql` -- Branding, locale, user preferences
10. `0010_collaboration.sql` -- Distribution groups, public folders, co-editing
11. `0011_ai_search_analytics.sql` -- AI tables, search, analytics
12. `0012_innovation_features.sql` -- Knowledge graph, privacy, health scores

---

## 4. Deployment Strategy

### 4.1 Rolling Update Configuration

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  minReadySeconds: 30
```

### 4.2 Deployment Order

```mermaid
flowchart TB
    db["1. Database Migrations"]
    db --> infra["2. Infrastructure<br/>(Redis, Redpanda, MinIO)"]
    infra --> core["3. Core Services<br/>(email, calendar, chat, contacts)"]
    core --> collab["4. Collaboration Services<br/>(meet, docs, drive)"]
    collab --> gateway["5. API Gateway<br/>(update routing rules)"]
    gateway --> smoke["6. Smoke Tests"]
```

### 4.3 Canary Deployment (Optional)

For high-risk changes:
1. Deploy new version to 1 replica (canary)
2. Route 10% of traffic to canary
3. Monitor error rate and latency for 15 minutes
4. If healthy, roll out to all replicas
5. If unhealthy, roll back canary immediately

---

## 5. Rollback Procedure

```mermaid
flowchart TB
    issue["Issue Detected"]
    issue --> assess{"Data Migration<br/>Involved?"}
    assess -->|"No"| rollback_app["Rollback Application<br/>helm rollback"]
    assess -->|"Yes"| rollback_db["Rollback Database<br/>from backup"]
    rollback_db --> rollback_app
    rollback_app --> verify_rb["Verify Rollback"]
    verify_rb --> postmortem["Post-Mortem"]
```

- **Application rollback**: `helm rollback erp-workspace <revision>` (< 2 minutes)
- **Database rollback**: Restore from pre-migration snapshot (< 15 minutes)
- **Event bus rollback**: Consumer offsets are committed per-message; no rollback needed

---

## 6. Environment Configuration

| Environment | Purpose | Update Method |
|------------|---------|-------------|
| Development | Local development | `docker compose up` |
| Staging | Integration testing | Auto-deploy on merge to `develop` |
| Production | Live users | Manual approval after staging E2E pass |

### Environment Variables

| Variable | Description | Example |
|----------|------------|---------|
| `PORT` | Service listen port | `8080` |
| `MODULE_NAME` | Module identifier | `ERP-Workspace` |
| `DATABASE_URL` | PostgreSQL connection | `postgres://user:pass@host/db` |
| `REDIS_URL` | Redis connection | `redis://host:6379` |
| `MINIO_ENDPOINT` | MinIO S3 endpoint | `minio:9000` |
| `REDPANDA_BROKERS` | Kafka bootstrap servers | `redpanda:9092` |
| `IAM_JWKS_URL` | ERP-IAM JWKS endpoint | `https://iam.erp.io/.well-known/jwks.json` |
| `LIVEKIT_URL` | LiveKit SFU endpoint | `wss://livekit.erp.io` |
| `ONLYOFFICE_URL` | ONLYOFFICE Document Server | `https://docs.erp.io` |

---

*For monitoring during deployments, see [20-Monitoring-Observability.md](./20-Monitoring-Observability.md). For rollback procedures, see [27-Runbooks.md](./27-Runbooks.md).*
