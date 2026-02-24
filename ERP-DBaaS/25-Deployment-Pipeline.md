# ERP-DBaaS Deployment Pipeline

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Deployment Pipeline          |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Engineering                 |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [Pipeline Overview](#1-pipeline-overview)
2. [GitLab CI Stages](#2-gitlab-ci-stages)
3. [Operator Image Builds](#3-operator-image-builds)
4. [Helm Chart Packaging](#4-helm-chart-packaging)
5. [Integration Test Pipeline](#5-integration-test-pipeline)
6. [Canary Deployment Strategy](#6-canary-deployment-strategy)
7. [Rollback Procedures](#7-rollback-procedures)

---

## 1. Pipeline Overview

The ERP-DBaaS deployment pipeline follows a multi-stage CI/CD process built on GitLab CI. The pipeline enforces quality gates at each stage to ensure only validated, tested, and approved artifacts reach production.

### Pipeline Architecture

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│  Build   │──>│   Test   │──>│ Package  │──>│  Deploy  │──>│ Verify   │
│          │   │          │   │          │   │          │   │          │
│ - Lint   │   │ - Unit   │   │ - Docker │   │ - Canary │   │ - Smoke  │
│ - Compile│   │ - Integ. │   │ - Helm   │   │ - Staged │   │ - Health │
│ - SAST   │   │ - E2E    │   │ - Push   │   │ - Full   │   │ - Perf   │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
```

### Branch Strategy

| Branch          | Pipeline Stages             | Target Environment |
|-----------------|-----------------------------|--------------------|
| `feature/*`     | Build, Test (unit)          | None               |
| `develop`       | Build, Test, Package        | Development        |
| `release/*`     | Build, Test, Package, Deploy| Staging            |
| `main`          | Build, Test, Package, Deploy| Production         |
| `hotfix/*`      | Build, Test, Package, Deploy| Production (fast)  |

---

## 2. GitLab CI Stages

### 2.1 Build Stage

```yaml
build:gateway:
  stage: build
  image: golang:1.22-alpine
  script:
    - cd cmd/gateway
    - go vet ./...
    - golangci-lint run --timeout 5m
    - go build -ldflags "-X main.version=${CI_COMMIT_TAG:-dev}" -o /build/gateway
  artifacts:
    paths:
      - /build/gateway
    expire_in: 1 hour

build:api:
  stage: build
  image: node:20-alpine
  script:
    - cd dbaas-api
    - npm ci --prefer-offline
    - npm run lint
    - npm run build
  artifacts:
    paths:
      - dbaas-api/dist/
    expire_in: 1 hour

build:operators:
  stage: build
  image: golang:1.22-alpine
  parallel:
    matrix:
      - ENGINE: [yugabytedb, dragonflydb, clickhouse, tembo, surrealdb, questdb, doris, influxdb]
  script:
    - cd operators/${ENGINE}
    - go vet ./...
    - go build -o /build/operator-${ENGINE}
  artifacts:
    paths:
      - /build/operator-*
    expire_in: 1 hour

build:frontend:
  stage: build
  image: node:20-alpine
  script:
    - cd frontend
    - npm ci --prefer-offline
    - npm run lint
    - npm run type-check
    - npm run build
  artifacts:
    paths:
      - frontend/dist/
    expire_in: 1 hour
```

### 2.2 Test Stage

```yaml
test:unit:gateway:
  stage: test
  image: golang:1.22-alpine
  script:
    - cd cmd/gateway
    - go test -race -coverprofile=coverage.out ./...
    - go tool cover -func=coverage.out | tail -1
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

test:unit:api:
  stage: test
  image: node:20-alpine
  services:
    - name: yugabytedb/yugabyte:2.20.1.0-b97
      alias: yugabytedb
  script:
    - cd dbaas-api
    - npm ci
    - npm run test -- --coverage --forceExit
  coverage: '/All files\s+\|\s+(\d+\.\d+)/'

test:unit:operators:
  stage: test
  image: golang:1.22-alpine
  parallel:
    matrix:
      - ENGINE: [yugabytedb, dragonflydb, clickhouse, tembo, surrealdb, questdb, doris, influxdb]
  script:
    - cd operators/${ENGINE}
    - go test -race -coverprofile=coverage.out ./...
  coverage: '/total:\s+\(statements\)\s+(\d+\.\d+)%/'

test:sast:
  stage: test
  image: securego/gosec:latest
  script:
    - gosec -fmt json -out results.json ./...
  artifacts:
    reports:
      sast: results.json
```

### 2.3 Package Stage

```yaml
package:docker:
  stage: package
  image: docker:24-dind
  services:
    - docker:24-dind
  parallel:
    matrix:
      - COMPONENT: [gateway, dbaas-api, frontend, operator-yugabytedb, operator-dragonflydb, operator-clickhouse, operator-tembo, operator-surrealdb, operator-questdb, operator-doris, operator-influxdb, backup-controller, credential-manager, metering-collector]
  script:
    - docker build -t ${CI_REGISTRY_IMAGE}/${COMPONENT}:${CI_COMMIT_SHA} -f docker/${COMPONENT}.Dockerfile .
    - docker push ${CI_REGISTRY_IMAGE}/${COMPONENT}:${CI_COMMIT_SHA}
    - |
      if [ -n "$CI_COMMIT_TAG" ]; then
        docker tag ${CI_REGISTRY_IMAGE}/${COMPONENT}:${CI_COMMIT_SHA} ${CI_REGISTRY_IMAGE}/${COMPONENT}:${CI_COMMIT_TAG}
        docker push ${CI_REGISTRY_IMAGE}/${COMPONENT}:${CI_COMMIT_TAG}
      fi
```

### 2.4 Deploy Stage

```yaml
deploy:staging:
  stage: deploy
  image: alpine/helm:3.14
  environment:
    name: staging
    url: https://dbaas-staging.erp.io
  script:
    - helm upgrade --install dbaas ./helm/dbaas
        --namespace dbaas-system
        --values helm/dbaas/values-staging.yaml
        --set global.image.tag=${CI_COMMIT_SHA}
        --wait --timeout 10m
  rules:
    - if: '$CI_COMMIT_BRANCH =~ /^release\//'

deploy:production:
  stage: deploy
  image: alpine/helm:3.14
  environment:
    name: production
    url: https://dbaas.erp.io
  script:
    - helm upgrade --install dbaas ./helm/dbaas
        --namespace dbaas-system
        --values helm/dbaas/values-production.yaml
        --set global.image.tag=${CI_COMMIT_TAG}
        --wait --timeout 15m
  rules:
    - if: '$CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/'
  when: manual
```

---

## 3. Operator Image Builds

### 3.1 Operator Dockerfile Pattern

Each operator follows a consistent multi-stage Dockerfile pattern for minimal image size and security.

```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
RUN apk add --no-cache git ca-certificates
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
ARG ENGINE
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags="-w -s" \
    -o /operator ./operators/${ENGINE}/cmd/

# Stage 2: Runtime
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /operator /operator
USER nonroot:nonroot
ENTRYPOINT ["/operator"]
```

### 3.2 Image Specifications

| Property              | Value                                      |
|-----------------------|--------------------------------------------|
| Base image            | `gcr.io/distroless/static:nonroot`         |
| User                  | `nonroot` (UID 65532)                      |
| Image size target     | < 50 MiB per operator                      |
| Scan policy           | Trivy scan on every push, block on CRITICAL|
| Registry              | GitLab Container Registry                  |
| Tag strategy          | `${SHA}` for commits, `v${SEMVER}` for releases |
| Platform              | `linux/amd64`, `linux/arm64`               |

---

## 4. Helm Chart Packaging

### 4.1 Chart Structure

```
helm/dbaas/
├── Chart.yaml
├── values.yaml
├── values-staging.yaml
├── values-production.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── gateway-deployment.yaml
│   ├── gateway-service.yaml
│   ├── api-deployment.yaml
│   ├── api-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── frontend-ingress.yaml
│   ├── hasura-deployment.yaml
│   ├── hasura-service.yaml
│   ├── operators/
│   │   ├── yugabytedb-operator.yaml
│   │   ├── dragonflydb-operator.yaml
│   │   ├── clickhouse-operator.yaml
│   │   ├── tembo-operator.yaml
│   │   ├── surrealdb-operator.yaml
│   │   ├── questdb-operator.yaml
│   │   ├── doris-operator.yaml
│   │   └── influxdb-operator.yaml
│   ├── backup-controller.yaml
│   ├── credential-manager.yaml
│   ├── metering-collector.yaml
│   ├── crds/
│   │   └── *.yaml
│   ├── rbac/
│   │   ├── cluster-role.yaml
│   │   └── service-accounts.yaml
│   └── network-policies/
│       └── *.yaml
└── charts/           # Sub-chart dependencies
```

### 4.2 Chart Versioning

- Chart version follows SemVer aligned with the application version.
- CRD versions are managed separately to support backward compatibility.
- Charts are published to the GitLab Helm registry on every tagged release.

```yaml
# Chart.yaml
apiVersion: v2
name: erp-dbaas
description: ERP Database-as-a-Service Platform
version: 1.0.0
appVersion: "1.0.0"
dependencies:
  - name: yugabytedb-operator
    version: ">=1.0.0"
    repository: "file://charts/yugabytedb-operator"
    condition: operators.yugabytedb.enabled
```

---

## 5. Integration Test Pipeline

### 5.1 Test Environment

Integration tests run against a dedicated Kind (Kubernetes-in-Docker) cluster provisioned for each pipeline run.

```yaml
test:integration:
  stage: test
  image: docker:24-dind
  services:
    - docker:24-dind
  variables:
    KIND_CLUSTER_NAME: "dbaas-integration-${CI_PIPELINE_ID}"
  script:
    - kind create cluster --name ${KIND_CLUSTER_NAME} --config test/kind-config.yaml
    - kubectl apply -f helm/dbaas/templates/crds/
    - helm install dbaas ./helm/dbaas --namespace dbaas-system --create-namespace
        --values test/values-integration.yaml
        --set global.image.tag=${CI_COMMIT_SHA}
        --wait --timeout 10m
    - go test -tags=integration -timeout 30m ./test/integration/...
  after_script:
    - kind delete cluster --name ${KIND_CLUSTER_NAME}
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop" || $CI_COMMIT_BRANCH =~ /^release\//'
```

### 5.2 Integration Test Suites

| Suite                   | Tests | Timeout | Description                                    |
|-------------------------|-------|---------|------------------------------------------------|
| Provisioning            | 16    | 10m     | Provision each engine (S and L sizes)           |
| Scaling                 | 8     | 10m     | Vertical and horizontal scaling per engine      |
| Backup & Restore        | 8     | 15m     | Full backup, incremental, and restore per engine|
| Credential Rotation     | 4     | 5m      | Rotate and verify connectivity                  |
| Quota Enforcement       | 6     | 3m      | Quota approval and rejection scenarios          |
| Plugin Management       | 4     | 5m      | Install, validate, and uninstall plugins        |
| HA Failover             | 4     | 15m     | Failover simulation for HA-capable engines      |

---

## 6. Canary Deployment Strategy

### 6.1 Canary Process

Production deployments use a canary strategy to minimize risk. The process is:

```
Full Fleet ──> Deploy Canary (5%) ──> Monitor (15 min) ──> Expand (25%)
                                                              │
                                                              ▼
                                                         Monitor (15 min)
                                                              │
                                                              ▼
                                                         Expand (50%)
                                                              │
                                                              ▼
                                                         Monitor (15 min)
                                                              │
                                                              ▼
                                                         Full Rollout (100%)
```

### 6.2 Canary Health Criteria

The canary is promoted to the next stage only if all of the following conditions are met:

| Metric                  | Threshold           | Action on Violation        |
|-------------------------|---------------------|----------------------------|
| HTTP error rate (5xx)   | < 0.1%              | Automatic rollback         |
| P99 latency             | < 500ms             | Automatic rollback         |
| Pod restart count       | 0                   | Automatic rollback         |
| Operator reconcile errors| 0                  | Pause and alert            |
| Backup success rate     | 100%                | Pause and alert            |

### 6.3 Canary Implementation

```yaml
# Argo Rollouts canary spec
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: dbaas-gateway
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 15m }
        - analysis:
            templates:
              - templateName: dbaas-canary-analysis
        - setWeight: 25
        - pause: { duration: 15m }
        - setWeight: 50
        - pause: { duration: 15m }
        - setWeight: 100
      canaryService: dbaas-gateway-canary
      stableService: dbaas-gateway-stable
```

---

## 7. Rollback Procedures

### 7.1 Automated Rollback

Automated rollback is triggered when canary health criteria are violated. The process is:

1. Canary analysis detects a threshold violation.
2. The Argo Rollout controller scales down the canary replicas to zero.
3. Traffic is shifted back to 100% stable version.
4. An alert is sent to the engineering Slack channel and PagerDuty.
5. The failed deployment artifact is tagged as `failed-{SHA}` in the registry.

### 7.2 Manual Rollback

For post-deployment issues discovered after full rollout, use the following procedure.

```bash
# Step 1: Identify the last known good version
helm history dbaas -n dbaas-system

# Step 2: Rollback to the previous revision
helm rollback dbaas <REVISION_NUMBER> -n dbaas-system --wait --timeout 10m

# Step 3: Verify rollback
kubectl get pods -n dbaas-system -w
kubectl logs -n dbaas-system deployment/dbaas-gateway --tail=100

# Step 4: Run smoke tests
./scripts/smoke-test.sh --env production
```

### 7.3 CRD Rollback Considerations

CRD changes require special handling because Helm does not roll back CRD modifications.

| Scenario              | Rollback Action                                          |
|-----------------------|----------------------------------------------------------|
| New CRD fields added  | Safe to leave in place; backward compatible              |
| CRD fields removed    | Restore the CRD manually: `kubectl apply -f crds/`      |
| CRD version change    | Conversion webhook must handle both old and new versions |
| CRD deleted           | Re-apply CRD from the previous release tag               |

### 7.4 Rollback SLA

| Rollback Type    | Target Time | Automation Level |
|------------------|-------------|------------------|
| Canary rollback  | < 2 minutes | Fully automated  |
| Helm rollback    | < 5 minutes | Semi-automated   |
| CRD rollback     | < 15 minutes| Manual           |
| Database restore | < 30 minutes| Semi-automated   |

---

## Appendix: Pipeline Environment Variables

| Variable                    | Source          | Description                          |
|-----------------------------|-----------------|--------------------------------------|
| `CI_REGISTRY_IMAGE`         | GitLab CI       | Container registry base path         |
| `CI_COMMIT_SHA`             | GitLab CI       | Full commit SHA for image tagging    |
| `CI_COMMIT_TAG`             | GitLab CI       | Git tag for release versioning       |
| `KUBECONFIG`                | CI Secret       | Kubernetes config for deployment     |
| `HELM_REGISTRY_TOKEN`       | CI Secret       | Helm chart registry authentication   |
| `SONAR_TOKEN`               | CI Secret       | SonarQube analysis token             |
| `TRIVY_SEVERITY`            | CI Variable     | `CRITICAL,HIGH` for image scanning   |
