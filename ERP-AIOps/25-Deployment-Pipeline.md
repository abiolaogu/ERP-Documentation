# ERP-AIOps Deployment Pipeline

> **Document ID:** ERP-AIOPS-DEPLOY-025
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [24-Runbooks.md](./24-Runbooks.md), [26-Disaster-Recovery-Plan.md](./26-Disaster-Recovery-Plan.md)

---

## 1. Pipeline Overview

ERP-AIOps uses GitLab CI/CD for continuous integration and deployment. The pipeline handles three distinct build targets (Go, Rust, Python) and deploys via Helm charts to Kubernetes.

### Pipeline Stages

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Lint &  │───>│  Build   │───>│  Test    │───>│  Docker  │───>│  Deploy  │───>│  Verify  │
│  Check   │    │  Compile │    │  Suite   │    │  Images  │    │  (Helm)  │    │  (E2E)   │
└──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### Branch Strategy

| Branch | Trigger | Deploy Target | Approval |
|--------|---------|---------------|----------|
| `feature/*` | Push | None (CI only) | None |
| `develop` | Merge | Staging | Auto |
| `release/*` | Merge | Pre-production | Manual |
| `main` | Merge | Production | Manual (2 approvers) |
| `hotfix/*` | Push | Staging + Production (fast-track) | Manual (1 approver) |

---

## 2. Stage 1: Lint and Check

### Go Gateway Linting

```yaml
gateway-lint:
  stage: lint
  image: golangci/golangci-lint:v1.56
  script:
    - cd gateway/
    - golangci-lint run --timeout 5m
    - go vet ./...
  rules:
    - changes:
        - gateway/**/*
```

### Rust API Linting

```yaml
rust-lint:
  stage: lint
  image: rust:1.76
  script:
    - cd rust-api/
    - cargo fmt --check
    - cargo clippy --all-targets --all-features -- -D warnings
  rules:
    - changes:
        - rust-api/**/*
```

### Python AI Brain Linting

```yaml
python-lint:
  stage: lint
  image: python:3.12
  script:
    - cd ai-brain/
    - pip install ruff mypy
    - ruff check .
    - mypy --strict ai_brain/
  rules:
    - changes:
        - ai-brain/**/*
```

### Frontend Linting

```yaml
frontend-lint:
  stage: lint
  image: node:20
  script:
    - cd frontend/
    - npm ci
    - npm run lint
    - npm run type-check
  rules:
    - changes:
        - frontend/**/*
```

---

## 3. Stage 2: Rust Compilation and Testing

### Rust Build

```yaml
rust-build:
  stage: build
  image: rust:1.76
  cache:
    key: rust-cargo-${CI_COMMIT_REF_SLUG}
    paths:
      - rust-api/target/
      - /usr/local/cargo/registry/
  script:
    - cd rust-api/
    - cargo build --release --workspace
  artifacts:
    paths:
      - rust-api/target/release/aiops-api
    expire_in: 1 hour
  rules:
    - changes:
        - rust-api/**/*
```

### Rust Unit Tests

```yaml
rust-test:
  stage: test
  image: rust:1.76
  services:
    - name: yugabytedb/yugabyte:2.20
      alias: yugabytedb
    - name: docker.dragonflydb.io/dragonflydb/dragonfly:latest
      alias: dragonfly
  variables:
    DATABASE_URL: "postgresql://yugabyte@yugabytedb:5433/aiops_test"
    REDIS_URL: "redis://dragonfly:6379"
  cache:
    key: rust-cargo-${CI_COMMIT_REF_SLUG}
    paths:
      - rust-api/target/
  script:
    - cd rust-api/
    - cargo test --workspace --release -- --test-threads=4
  artifacts:
    reports:
      junit: rust-api/target/test-results.xml
```

---

## 4. Stage 3: Python Environment Setup and Testing

### Python Build and Test

```yaml
python-test:
  stage: test
  image: python:3.12
  services:
    - name: yugabytedb/yugabyte:2.20
      alias: yugabytedb
  variables:
    DATABASE_URL: "postgresql://yugabyte@yugabytedb:5433/aiops_test"
  cache:
    key: python-pip-${CI_COMMIT_REF_SLUG}
    paths:
      - ai-brain/.venv/
  script:
    - cd ai-brain/
    - python -m venv .venv
    - source .venv/bin/activate
    - pip install -r requirements.txt -r requirements-test.txt
    - pytest tests/ -v --junitxml=test-results.xml --cov=ai_brain --cov-report=xml
  artifacts:
    reports:
      junit: ai-brain/test-results.xml
      coverage_report:
        coverage_format: cobertura
        path: ai-brain/coverage.xml
  coverage: '/Total.*?(\d+\.\d+)%/'
```

### ML Model Validation

```yaml
model-validation:
  stage: test
  image: python:3.12
  script:
    - cd ai-brain/
    - python -m venv .venv
    - source .venv/bin/activate
    - pip install -r requirements.txt
    - python -m ai_brain.validate --model isolation_forest --min-precision 0.90 --min-recall 0.85
    - python -m ai_brain.validate --model lstm --min-precision 0.92 --min-recall 0.88
  rules:
    - changes:
        - ai-brain/ai_brain/models/**/*
```

---

## 5. Stage 4: Docker Image Builds

### Go Gateway Image

```yaml
gateway-docker:
  stage: docker
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t ${CI_REGISTRY}/aiops/gateway:${CI_COMMIT_SHA} -f gateway/Dockerfile gateway/
    - docker push ${CI_REGISTRY}/aiops/gateway:${CI_COMMIT_SHA}
    - docker tag ${CI_REGISTRY}/aiops/gateway:${CI_COMMIT_SHA} ${CI_REGISTRY}/aiops/gateway:latest
    - docker push ${CI_REGISTRY}/aiops/gateway:latest
```

### Rust API Image

```yaml
rust-api-docker:
  stage: docker
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t ${CI_REGISTRY}/aiops/api:${CI_COMMIT_SHA} -f rust-api/Dockerfile rust-api/
    - docker push ${CI_REGISTRY}/aiops/api:${CI_COMMIT_SHA}
```

**Rust Dockerfile (multi-stage):**

```dockerfile
# Build stage
FROM rust:1.76 AS builder
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY crates/ crates/
RUN cargo build --release --bin aiops-api

# Runtime stage
FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/target/release/aiops-api /usr/local/bin/
EXPOSE 8080
CMD ["aiops-api"]
```

### Python AI Brain Image

```yaml
ai-brain-docker:
  stage: docker
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -t ${CI_REGISTRY}/aiops/ai-brain:${CI_COMMIT_SHA} -f ai-brain/Dockerfile ai-brain/
    - docker push ${CI_REGISTRY}/aiops/ai-brain:${CI_COMMIT_SHA}
```

**Python Dockerfile:**

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY ai_brain/ ai_brain/
COPY models/ /models/
EXPOSE 8001
CMD ["uvicorn", "ai_brain.main:app", "--host", "0.0.0.0", "--port", "8001", "--workers", "4"]
```

---

## 6. Stage 5: Helm Chart Deployment

### Helm Chart Structure

```
helm/aiops/
├── Chart.yaml
├── values.yaml
├── values-staging.yaml
├── values-production.yaml
├── templates/
│   ├── gateway-deployment.yaml
│   ├── gateway-service.yaml
│   ├── api-deployment.yaml
│   ├── api-service.yaml
│   ├── ai-brain-deployment.yaml
│   ├── ai-brain-service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secrets.yaml
│   ├── hpa.yaml
│   └── pdb.yaml
```

### Staging Deployment

```yaml
deploy-staging:
  stage: deploy
  image: alpine/helm:3.14
  environment:
    name: staging
    url: https://aiops-staging.erp.internal
  script:
    - helm upgrade --install aiops ./helm/aiops
      --namespace aiops-staging
      --create-namespace
      -f helm/aiops/values-staging.yaml
      --set gateway.image.tag=${CI_COMMIT_SHA}
      --set api.image.tag=${CI_COMMIT_SHA}
      --set aiBrain.image.tag=${CI_COMMIT_SHA}
      --wait --timeout 300s
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"
```

### Production Deployment

```yaml
deploy-production:
  stage: deploy
  image: alpine/helm:3.14
  environment:
    name: production
    url: https://aiops.erp.internal
  script:
    - helm upgrade --install aiops ./helm/aiops
      --namespace aiops
      -f helm/aiops/values-production.yaml
      --set gateway.image.tag=${CI_COMMIT_SHA}
      --set api.image.tag=${CI_COMMIT_SHA}
      --set aiBrain.image.tag=${CI_COMMIT_SHA}
      --wait --timeout 600s
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
  allow_failure: false
```

---

## 7. Stage 6: Integration Test Suite

```yaml
integration-tests:
  stage: verify
  image: python:3.12
  script:
    - cd tests/integration/
    - pip install -r requirements.txt
    - pytest test_api.py -v --junitxml=api-results.xml
    - pytest test_anomaly_detection.py -v --junitxml=anomaly-results.xml
    - pytest test_incident_lifecycle.py -v --junitxml=incident-results.xml
    - pytest test_remediation.py -v --junitxml=remediation-results.xml
    - pytest test_websocket.py -v --junitxml=ws-results.xml
  artifacts:
    reports:
      junit:
        - tests/integration/*-results.xml
```

### Test Coverage Requirements

| Component | Minimum Coverage | Current |
|-----------|-----------------|---------|
| Rust API | 80% | 84% |
| Python AI Brain | 75% | 79% |
| Go Gateway | 70% | 73% |
| Integration Tests | N/A (pass/fail) | 47 tests |

---

## 8. Canary Deployment for ML Model Updates

ML model updates are deployed using a canary strategy to catch accuracy regressions before full rollout.

### Canary Process

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Deploy at  │───>│  Monitor    │───>│  Promote    │───>│  Full       │
│  10% traffic│    │  24 hours   │    │  to 50%     │    │  Rollout    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                         │
                    Metrics fail
                         │
                         v
                   ┌─────────────┐
                   │  Auto       │
                   │  Rollback   │
                   └─────────────┘
```

### Canary Success Criteria

| Metric | Threshold | Action on Failure |
|--------|-----------|-------------------|
| Anomaly Detection Precision | > 85% | Auto-rollback |
| False Positive Rate | < 20% | Auto-rollback |
| Inference Latency p99 | < 150ms | Auto-rollback |
| Error Rate | < 1% | Auto-rollback |

---

## 9. Rollback Procedures

### Application Rollback (Helm)

```bash
# List release history
helm history aiops -n aiops

# Rollback to previous revision
helm rollback aiops [REVISION] -n aiops --wait --timeout 300s

# Verify
kubectl get pods -n aiops
curl -s http://localhost:8080/health | jq .
```

### ML Model Rollback

```bash
# List model versions
curl -s http://localhost:8001/admin/models | jq '.[] | {name, version, active}'

# Rollback to previous model version
curl -s -X POST http://localhost:8001/admin/rollback \
  -H "Content-Type: application/json" \
  -d '{"model":"isolation_forest","target_version":"v1.2.3"}' | jq .
```

### Database Migration Rollback

See [Runbook 5: Database Migration Procedure](./24-Runbooks.md#runbook-5-database-migration-procedure) for database rollback steps.

### Emergency Rollback Checklist

1. Identify the failing component (gateway, API, AI brain, database).
2. Execute the appropriate rollback command.
3. Verify health checks pass.
4. Notify the team in `#aiops-deploys`.
5. Create a post-mortem ticket to investigate the failure.
