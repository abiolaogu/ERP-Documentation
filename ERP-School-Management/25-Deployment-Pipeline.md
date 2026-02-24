# ERP-School-Management -- Deployment Pipeline

**Product:** EduCore Pro
**Version:** 1.0.0
**Date:** 2026-02-23

---

## 1. Pipeline Overview

```mermaid
flowchart LR
    A["Developer<br/>Push"] --> B["CI: Lint<br/>& Build"]
    B --> C["CI: Unit<br/>Tests"]
    C --> D["CI: Integration<br/>Tests"]
    D --> E["Build Docker<br/>Images"]
    E --> F["Push to<br/>Registry"]
    F --> G["Deploy to<br/>Staging"]
    G --> H["E2E Tests<br/>on Staging"]
    H --> I["Manual<br/>Approval"]
    I --> J["Deploy to<br/>Production"]
    J --> K["Smoke Tests"]
    K --> L["Monitor"]
```

---

## 2. CI/CD Stages

### Stage 1: Code Quality

```yaml
lint:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    - run: npm ci
    - run: npm run lint
    - run: npm run format -- --check
```

### Stage 2: Build

```yaml
build:
  runs-on: ubuntu-latest
  needs: lint
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    - run: npm ci
    - run: npm run build
    - uses: actions/cache@v4
      with:
        path: |
          services/*/dist
          apps/*/dist
          apps/web/.next
        key: build-${{ github.sha }}
```

### Stage 3: Unit Tests

```yaml
test:
  runs-on: ubuntu-latest
  needs: build
  services:
    postgres:
      image: postgres:16
      env:
        POSTGRES_USER: erp
        POSTGRES_PASSWORD: erp
        POSTGRES_DB: erp_school_management_test
      ports:
        - 5432:5432
    redpanda:
      image: redpandadata/redpanda:v24.2.8
      ports:
        - 9092:9092
  steps:
    - uses: actions/checkout@v4
    - run: npm ci
    - run: npm run db:migrate
      env:
        DATABASE_URL: postgres://erp:erp@localhost:5432/erp_school_management_test
    - run: npm run test -- --coverage
      env:
        DATABASE_URL: postgres://erp:erp@localhost:5432/erp_school_management_test
        REDPANDA_BROKERS: localhost:9092
```

### Stage 4: Docker Build & Push

```yaml
docker:
  runs-on: ubuntu-latest
  needs: test
  strategy:
    matrix:
      service:
        - gateway
        - academic-service
        - auth-service
        - student-service
        - finance-service
        - lms-service
        - admin-service
        - analytics-service
        - communication-service
        - notification-service
        - file-service
        - search-service
        - blockchain-service
        - gamification-service
        - iot-service
        - integration-service
        - migration-service
        - subscription-service
        - ai-service
        - research-service
  steps:
    - uses: actions/checkout@v4
    - uses: docker/setup-buildx-action@v3
    - uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    - uses: docker/build-push-action@v5
      with:
        context: ./services/${{ matrix.service }}
        push: true
        tags: ghcr.io/educore-pro/${{ matrix.service }}:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

### Stage 5: Deploy to Staging

```yaml
deploy-staging:
  runs-on: ubuntu-latest
  needs: docker
  environment: staging
  steps:
    - uses: actions/checkout@v4
    - uses: azure/setup-kubectl@v3
    - run: |
        kubectl set image deployment/gateway \
          gateway=ghcr.io/educore-pro/gateway:${{ github.sha }} \
          -n educore-staging
        kubectl set image deployment/academic-service \
          academic=ghcr.io/educore-pro/academic-service:${{ github.sha }} \
          -n educore-staging
        # ... repeat for all services
    - run: kubectl rollout status deployment --timeout=300s -n educore-staging
```

### Stage 6: E2E Tests on Staging

```yaml
e2e-staging:
  runs-on: ubuntu-latest
  needs: deploy-staging
  steps:
    - uses: actions/checkout@v4
    - run: |
        npm run test:e2e -- --baseUrl=https://staging-api.educorepro.com
```

### Stage 7: Deploy to Production

```yaml
deploy-production:
  runs-on: ubuntu-latest
  needs: e2e-staging
  environment:
    name: production
    url: https://api.educorepro.com
  steps:
    - uses: actions/checkout@v4
    - uses: azure/setup-kubectl@v3
    - run: |
        # Blue-green deployment
        kubectl set image deployment/gateway \
          gateway=ghcr.io/educore-pro/gateway:${{ github.sha }} \
          -n educore-production
        kubectl rollout status deployment/gateway --timeout=300s \
          -n educore-production
```

---

## 3. Deployment Strategies

### 3.1 Rolling Update (Default)

```mermaid
flowchart TD
    subgraph "Rolling Update"
        A["Pod v1.0 (running)"] --> B["Pod v1.1 (starting)"]
        B --> C["Pod v1.1 (ready)"]
        C --> D["Pod v1.0 (terminating)"]
        D --> E["Pod v1.1 (serving)"]
    end
```

- **maxSurge:** 1
- **maxUnavailable:** 0
- **Rollback:** `kubectl rollout undo deployment/<service>`

### 3.2 Blue-Green (Gateway)

For the API gateway, use blue-green deployment to ensure zero downtime:
1. Deploy new version alongside current
2. Run health checks on new version
3. Switch traffic via service selector
4. Keep old version for 30 minutes as rollback target

### 3.3 Canary (High-Risk Changes)

For database schema changes or major feature releases:
1. Deploy to 10% of pods
2. Monitor error rates for 15 minutes
3. If healthy, deploy to 50%
4. If healthy, deploy to 100%
5. If errors, automatic rollback

---

## 4. Database Migration Strategy

```mermaid
flowchart TD
    A["Pre-deploy: Run<br/>backward-compatible migration"] --> B["Deploy new<br/>application version"]
    B --> C["Verify migration<br/>applied correctly"]
    C --> D{Issues?}
    D -->|No| E["Post-deploy: Clean up<br/>deprecated columns"]
    D -->|Yes| F["Rollback application"]
    F --> G["Rollback migration<br/>if possible"]
```

### Rules
1. All migrations must be backward-compatible
2. Column additions: add with default, never remove in same release
3. Column removals: deprecate in release N, remove in release N+1
4. Index additions: create concurrently to avoid table locks
5. Migration scripts must be idempotent

---

## 5. Environment Configuration

| Environment | Purpose | Deployment | URL |
|---|---|---|---|
| Development | Local development | Docker Compose | localhost:8092 |
| Staging | Integration testing | Kubernetes | staging-api.educorepro.com |
| Production | Live users | Kubernetes (multi-region) | api.educorepro.com |

### Environment Variables per Stage

| Variable | Dev | Staging | Production |
|---|---|---|---|
| NODE_ENV | development | staging | production |
| LOG_LEVEL | debug | info | warn |
| ALLOW_ON_ENTITLEMENT_FAILURE | true | true | false |
| RATE_LIMIT_RPM | unlimited | 300 | varies by tier |

---

## 6. Rollback Procedure

### Automatic Rollback Triggers
- Health check failure within 5 minutes of deploy
- Error rate > 5% within 10 minutes of deploy
- P95 latency > 2 seconds within 10 minutes of deploy

### Manual Rollback

```bash
# Kubernetes
kubectl rollout undo deployment/<service-name> -n educore-production

# Verify rollback
kubectl rollout status deployment/<service-name> -n educore-production

# Check which revision is active
kubectl rollout history deployment/<service-name> -n educore-production
```

---

## 7. Post-Deployment Verification

### Smoke Tests

```bash
# Health check
curl -f https://api.educorepro.com/healthz

# Capabilities
curl -f https://api.educorepro.com/v1/capabilities

# Auth flow
curl -X POST https://api.educorepro.com/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"smoke-test@educore.com","password":"..."}'
```

### Monitoring Checklist

- [ ] All service health checks passing
- [ ] Error rate < 0.1%
- [ ] P95 latency < 500ms
- [ ] Database connections healthy
- [ ] Redpanda consumers caught up
- [ ] No new error patterns in logs
