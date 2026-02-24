# ERP-Observability Deployment Pipeline

> **Document ID:** ERP-OBS-DP-025
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [26-Disaster-Recovery-Plan.md](./26-Disaster-Recovery-Plan.md)

---

## Overview

ERP-Observability uses a GitLab CI/CD pipeline for continuous integration, container image building, and multi-environment deployment. The pipeline enforces quality gates at each stage and supports environment promotion from development through staging to production with automated and manual approval gates.

---

## Pipeline Architecture

```
+----------+     +----------+     +----------+     +----------+     +----------+     +----------+
|  Lint &  | --> |  Build   | --> |  Test    | --> |  Package | --> |  Deploy  | --> |  Deploy  |
|  Validate|     |  Compile |     |  Suite   |     |  Images  |     |  Staging |     |  Prod    |
+----------+     +----------+     +----------+     +----------+     +----------+     +----------+
     |                |                |                |                |                |
  Go vet           Go build         Unit tests      Docker build     Helmfile         Canary
  ESLint           npm build        Integration     Push to          sync staging     then full
  YAML lint        TypeScript       Coverage        Registry         Smoke tests      rollout
  Schema val.      compile          E2E (staging)                    Integration      Approval
```

---

## GitLab CI Configuration

```yaml
# .gitlab-ci.yml

stages:
  - validate
  - build
  - test
  - package
  - deploy-dev
  - deploy-staging
  - deploy-production

variables:
  GO_VERSION: "1.22"
  NODE_VERSION: "20"
  DOCKER_REGISTRY: "registry.erp.io"
  IMAGE_GATEWAY: "${DOCKER_REGISTRY}/erp-observability/gateway"
  IMAGE_API: "${DOCKER_REGISTRY}/erp-observability/api"
  IMAGE_TENANT_API: "${DOCKER_REGISTRY}/erp-observability/tenant-api"
  HELM_RELEASE: "erp-observability"
  K8S_NAMESPACE: "erp-observability"

# ---------------------
# Stage: Validate
# ---------------------

lint-go:
  stage: validate
  image: golang:${GO_VERSION}
  script:
    - go vet ./...
    - go install honnef.co/go/tools/cmd/staticcheck@latest
    - staticcheck ./...
  rules:
    - changes:
        - "cmd/**"
        - "go.mod"
        - "go.sum"

lint-typescript:
  stage: validate
  image: node:${NODE_VERSION}
  script:
    - cd services/observability-api
    - npm ci
    - npm run lint
  rules:
    - changes:
        - "services/observability-api/**"

lint-yaml:
  stage: validate
  image: cytopia/yamllint:latest
  script:
    - yamllint -d relaxed configs/
    - yamllint -d relaxed infra/k8s/
  rules:
    - changes:
        - "configs/**"
        - "infra/k8s/**"

validate-quickwit-schemas:
  stage: validate
  image: curlimages/curl:latest
  script:
    - |
      for f in configs/quickwit/index-*.yaml; do
        echo "Validating $f..."
        # Schema validation using yq
        yq e '.version' "$f" > /dev/null || exit 1
        yq e '.index_id' "$f" > /dev/null || exit 1
        yq e '.doc_mapping' "$f" > /dev/null || exit 1
      done
  rules:
    - changes:
        - "configs/quickwit/**"

validate-sql-migration:
  stage: validate
  image: postgres:16
  script:
    - |
      for f in migrations/*.sql; do
        echo "Validating SQL syntax: $f"
        psql -h localhost -U postgres -c "\i $f" --set ON_ERROR_STOP=1 2>&1 || true
      done
  rules:
    - changes:
        - "migrations/**"

# ---------------------
# Stage: Build
# ---------------------

build-gateway:
  stage: build
  image: golang:${GO_VERSION}
  script:
    - CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o bin/gateway ./cmd/server/
  artifacts:
    paths:
      - bin/gateway
    expire_in: 1 hour
  rules:
    - changes:
        - "cmd/**"
        - "go.mod"

build-api:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - cd services/observability-api
    - npm ci
    - npm run build
  artifacts:
    paths:
      - services/observability-api/dist/
    expire_in: 1 hour
  rules:
    - changes:
        - "services/observability-api/**"

# ---------------------
# Stage: Test
# ---------------------

test-go:
  stage: test
  image: golang:${GO_VERSION}
  script:
    - go test ./... -v -race -coverprofile=coverage-go.out -count=1
    - go tool cover -func=coverage-go.out
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage-go.xml
  rules:
    - changes:
        - "cmd/**"
        - "go.mod"

test-typescript:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - cd services/observability-api
    - npm ci
    - npm test -- --coverage --reporter=junit --outputFile=../../test-results.xml
  coverage: '/All files\s+\|\s+(\d+\.\d+)/'
  artifacts:
    reports:
      junit: test-results.xml
    paths:
      - services/observability-api/coverage/
  rules:
    - changes:
        - "services/observability-api/**"

test-integration:
  stage: test
  image: node:${NODE_VERSION}
  services:
    - name: yugabyte/yugabyte:2.20.0.0-b78
      alias: yugabytedb
      command: ["bin/yugabyted", "start", "--daemon=false"]
    - name: docker.dragonflydb.io/dragonflydb/dragonfly:latest
      alias: dragonfly
  variables:
    YUGABYTE_HOST: yugabytedb
    YUGABYTE_PORT: "5433"
    DRAGONFLY_HOST: dragonfly
    DRAGONFLY_PORT: "6379"
    NODE_ENV: test
  script:
    - cd services/observability-api
    - npm ci
    - sleep 15  # Wait for YugabyteDB to initialize
    - npm run test:integration
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == "main"'

# ---------------------
# Stage: Package
# ---------------------

package-gateway:
  stage: package
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -f infra/Dockerfile.gateway -t ${IMAGE_GATEWAY}:${CI_COMMIT_SHA} -t ${IMAGE_GATEWAY}:latest .
    - docker push ${IMAGE_GATEWAY}:${CI_COMMIT_SHA}
    - docker push ${IMAGE_GATEWAY}:latest
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

package-api:
  stage: package
  image: docker:24
  services:
    - docker:24-dind
  script:
    - docker build -f infra/Dockerfile.api -t ${IMAGE_API}:${CI_COMMIT_SHA} -t ${IMAGE_API}:latest .
    - docker push ${IMAGE_API}:${CI_COMMIT_SHA}
    - docker push ${IMAGE_API}:latest
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

# ---------------------
# Stage: Deploy Dev
# ---------------------

deploy-dev:
  stage: deploy-dev
  image: alpine/helm:3.13
  environment:
    name: development
    url: https://observability-dev.erp.io
  script:
    - helmfile -e dev -f helmfile.yaml sync
    - kubectl rollout status deployment/observability-gateway -n ${K8S_NAMESPACE}-dev --timeout=300s
    - kubectl rollout status deployment/observability-api -n ${K8S_NAMESPACE}-dev --timeout=300s
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

smoke-test-dev:
  stage: deploy-dev
  image: curlimages/curl:latest
  needs: ["deploy-dev"]
  script:
    - |
      echo "Running smoke tests against dev..."
      curl -sf "https://observability-dev.erp.io/healthz" || exit 1
      curl -sf "https://observability-dev.erp.io/v1/capabilities" || exit 1
      curl -sf "https://observability-dev.erp.io/v1/observability/health" || exit 1
      echo "Smoke tests passed."
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

# ---------------------
# Stage: Deploy Staging
# ---------------------

deploy-staging:
  stage: deploy-staging
  image: alpine/helm:3.13
  environment:
    name: staging
    url: https://observability-staging.erp.io
  script:
    - helmfile -e staging -f helmfile.yaml sync
    - kubectl rollout status deployment/observability-gateway -n ${K8S_NAMESPACE}-staging --timeout=300s
    - kubectl rollout status deployment/observability-api -n ${K8S_NAMESPACE}-staging --timeout=300s
  needs: ["smoke-test-dev"]
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

integration-test-staging:
  stage: deploy-staging
  image: node:${NODE_VERSION}
  needs: ["deploy-staging"]
  script:
    - cd services/observability-api
    - npm ci
    - API_URL=https://observability-staging.erp.io npm run test:e2e
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'

# ---------------------
# Stage: Deploy Production
# ---------------------

deploy-production-canary:
  stage: deploy-production
  image: alpine/helm:3.13
  environment:
    name: production
    url: https://observability.erp.io
  script:
    - |
      # Deploy canary (10% traffic)
      helmfile -e production -f helmfile.yaml \
        --set gateway.canary.enabled=true \
        --set gateway.canary.weight=10 \
        sync
    - echo "Canary deployed with 10% traffic. Monitoring for 15 minutes..."
    - sleep 900
    - |
      # Check canary health
      ERROR_RATE=$(curl -s "http://vmselect:8481/select/0/prometheus/api/v1/query?query=rate(http_requests_total{deployment='canary',status=~'5..'}[15m])/rate(http_requests_total{deployment='canary'}[15m])" | jq -r '.data.result[0].value[1] // "0"')
      if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
        echo "Canary error rate too high: $ERROR_RATE. Rolling back."
        helmfile -e production -f helmfile.yaml \
          --set gateway.canary.enabled=false \
          sync
        exit 1
      fi
      echo "Canary healthy. Error rate: $ERROR_RATE"
  needs: ["integration-test-staging"]
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
      allow_failure: false

deploy-production-full:
  stage: deploy-production
  image: alpine/helm:3.13
  environment:
    name: production
    url: https://observability.erp.io
  script:
    - |
      # Full rollout (100% traffic)
      helmfile -e production -f helmfile.yaml \
        --set gateway.canary.enabled=false \
        sync
    - kubectl rollout status deployment/observability-gateway -n ${K8S_NAMESPACE} --timeout=600s
    - kubectl rollout status deployment/observability-api -n ${K8S_NAMESPACE} --timeout=600s
    - echo "Production deployment complete."
  needs: ["deploy-production-canary"]
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
      allow_failure: false

post-deploy-verification:
  stage: deploy-production
  image: curlimages/curl:latest
  needs: ["deploy-production-full"]
  script:
    - |
      echo "Post-deployment verification..."
      curl -sf "https://observability.erp.io/healthz" || exit 1
      curl -sf "https://observability.erp.io/v1/observability/health" || exit 1

      # Verify all components healthy
      HEALTH=$(curl -s "https://observability.erp.io/v1/observability/health")
      STATUS=$(echo "$HEALTH" | jq -r '.status')
      if [ "$STATUS" != "healthy" ]; then
        echo "WARNING: System status is $STATUS (not healthy)"
        echo "$HEALTH" | jq '.components'
        exit 1
      fi
      echo "All components healthy. Deployment verified."
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

---

## Environment Promotion Flow

```
Feature Branch
     |
     v
  [MR Created] --> Validate + Build + Test (automatic)
     |
     v
  [MR Merged to main] --> Deploy Dev (automatic)
     |                         |
     |                    Smoke Tests (automatic)
     |                         |
     v                         v
  Deploy Staging (automatic after dev smoke)
     |
     v
  Integration Tests + E2E (automatic)
     |
     v
  Deploy Production Canary (MANUAL APPROVAL)
     |
     | 15-minute canary monitoring period
     |
     v
  Deploy Production Full (MANUAL APPROVAL)
     |
     v
  Post-Deploy Verification (automatic)
```

---

## Rollback Procedures

### Automatic Rollback (Canary Failure)

If the canary deployment shows an error rate > 1% during the 15-minute monitoring window, the pipeline automatically reverts to the previous stable version.

### Manual Rollback

```bash
# Option 1: Helm rollback
helm rollback erp-observability -n erp-observability

# Option 2: Helmfile with previous version
helmfile -e production -f helmfile.yaml \
  --set gateway.image.tag=<previous-sha> \
  --set api.image.tag=<previous-sha> \
  sync

# Option 3: Kubernetes rollback
kubectl rollout undo deployment/observability-gateway -n erp-observability
kubectl rollout undo deployment/observability-api -n erp-observability

# Verify rollback
kubectl rollout status deployment/observability-gateway -n erp-observability
kubectl rollout status deployment/observability-api -n erp-observability
```

### Rollback Checklist

1. Identify the issue (logs, metrics, alerts).
2. Determine if the issue is in the gateway, API, or infrastructure config.
3. Execute rollback.
4. Verify health endpoints return healthy.
5. Verify dashboards load correctly in Grafana.
6. Verify log ingestion is flowing (check Quickwit indexer status).
7. Create a post-incident report.

---

## Infrastructure Deployments

Infrastructure components (Quickwit, Prometheus, Grafana, etc.) are deployed separately from application code:

```bash
# Infrastructure changes require SRE approval
# Deploy via Helmfile with environment-specific values

# Quickwit cluster upgrade
helmfile -e production -f helmfile-infra.yaml \
  -l component=quickwit \
  sync

# Prometheus configuration update
kubectl apply -f configs/prometheus/prometheus.yaml -n erp-observability
kubectl rollout restart deployment prometheus -n erp-observability

# Grafana dashboard update
kubectl create configmap grafana-dashboards \
  --from-file=configs/grafana/dashboards/ \
  -n erp-observability \
  --dry-run=client -o yaml | kubectl apply -f -
kubectl rollout restart deployment grafana -n erp-observability
```

---

## Database Migrations

Database migrations are executed manually during deployment:

```bash
# Step 1: Test migration on staging
psql -h yugabytedb-staging -p 5433 -U erp -d erp_observability \
  -f migrations/002_new_migration.sql

# Step 2: Verify staging
psql -h yugabytedb-staging -p 5433 -U erp -d erp_observability \
  -c "\dt" -c "\di"

# Step 3: Apply to production (with approval)
psql -h yugabytedb-prod -p 5433 -U erp -d erp_observability \
  -f migrations/002_new_migration.sql
```

**Migration rules:**
- All migrations must be backward-compatible (old code must work with new schema).
- Use `IF NOT EXISTS` for all CREATE statements.
- Never drop columns in a migration; deprecate first, remove in a later release.
- Test migrations on a YugabyteDB instance (not PostgreSQL) to catch YSQL-specific issues.

---

## Secrets Management

Secrets are managed via HashiCorp Vault and injected at deployment time:

```yaml
# Helm values for secret injection
envFrom:
  - secretRef:
      name: erp-observability-secrets

# Secrets managed in Vault
vault kv put secret/erp-observability/production \
  YUGABYTE_PASSWORD=<value> \
  RUSTFS_ACCESS_KEY=<value> \
  RUSTFS_SECRET_KEY=<value> \
  SLACK_WEBHOOK_URL=<value> \
  AUTHENTIK_JWKS_URL=<value>
```

---

## Quality Gates

| Gate | Stage | Criteria | Blocking |
|---|---|---|---|
| Lint (Go) | validate | `go vet` + `staticcheck` pass | Yes |
| Lint (TypeScript) | validate | ESLint pass | Yes |
| YAML Lint | validate | All YAML files valid | Yes |
| Unit Test Coverage (Go) | test | >= 70% | Yes |
| Unit Test Coverage (TS) | test | >= 80% | Yes |
| Integration Tests | test | All pass | Yes |
| Smoke Tests (Dev) | deploy-dev | Health endpoints 200 | Yes |
| E2E Tests (Staging) | deploy-staging | All pass | Yes |
| Canary Error Rate | deploy-production | < 1% | Yes |
| Post-Deploy Health | deploy-production | All components healthy | Yes |
