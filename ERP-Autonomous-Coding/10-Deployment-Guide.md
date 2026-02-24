# ERP-Autonomous-Coding -- Deployment Guide

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Autonomous-Coding |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Deployment Topology

```mermaid
flowchart TB
    subgraph "Production Deployment"
        subgraph "Edge Layer"
            CDN["CDN<br/>(CloudFront/Cloudflare)"]
            LB["Load Balancer<br/>(ALB/NLB)"]
        end

        subgraph "Kubernetes Cluster"
            subgraph "Ingress"
                NGINX["NGINX Ingress<br/>TLS Termination"]
            end

            subgraph "Application Pods"
                GW["API Gateway x3"]
                AC["Agent Core x5-20<br/>(HPA)"]
                GB["Git Bridge x3"]
                IS["IDE Server x3-10<br/>(HPA)"]
                RE["Review Engine x3-10<br/>(HPA)"]
                TP["Task Planner x3"]
            end

            subgraph "Infrastructure Pods"
                SR["Sandbox Runtime<br/>(DaemonSet)"]
                DASH["Dashboard x2"]
            end

            subgraph "Data StatefulSets"
                PG["PostgreSQL HA<br/>(Patroni)"]
                RD["Redis Sentinel<br/>(3 nodes)"]
                RP["Redpanda<br/>(3 brokers)"]
            end

            subgraph "Observability"
                PROM["Prometheus"]
                GRAF["Grafana"]
                TEMPO2["Tempo"]
                LOKI2["Loki"]
            end
        end
    end

    CDN --> LB
    LB --> NGINX
    NGINX --> GW
    GW --> AC
    GW --> GB
    GW --> IS
    GW --> RE
    GW --> TP
    AC --> SR
    AC --> PG
    AC --> RD
    AC --> RP
```

---

## 2. Docker Compose (Development)

The development environment is defined in the project's `docker-compose.yml`:

```yaml
version: "3.9"

services:
  autonomous-coding-api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8095:8090"
    environment:
      - DATABASE_URL=postgres://user:pass@postgres:5432/ac_dev
      - REDIS_URL=redis://redis:6379
      - KAFKA_BROKERS=redpanda:9092

  agent-core:
    build: ./services/agent-core
    ports:
      - "8205:8080"
    environment:
      - CLAUDE_API_KEY=${CLAUDE_API_KEY}
      - SANDBOX_RUNTIME_URL=http://sandbox-runtime:8080

  review-engine:
    build: ./services/review-engine
    ports:
      - "8206:8080"

  sandbox-runtime:
    build: ./services/sandbox-runtime
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    privileged: true

  git-bridge:
    build: ./services/git-bridge

  ide-server:
    build: ./services/ide-server
    ports:
      - "8207:8080"

  task-planner:
    build: ./services/task-planner
    ports:
      - "8209:8080"

  redis:
    image: redis:7

  postgres:
    image: postgres:16
    environment:
      - POSTGRES_DB=ac_dev
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - pg_data:/var/lib/postgresql/data

  redpanda:
    image: redpandadata/redpanda:latest
    command: redpanda start --smp 1 --memory 512M

volumes:
  pg_data:
```

**Starting development environment**:
```bash
# Clone and navigate
cd /path/to/ERP/ERP-Autonomous-Coding

# Set required environment variables
export CLAUDE_API_KEY="sk-ant-..."

# Build and start all services
docker compose up --build -d

# Verify health
curl http://localhost:8095/healthz
curl http://localhost:8205/healthz
curl http://localhost:8206/healthz
curl http://localhost:8207/healthz
curl http://localhost:8209/healthz
```

---

## 3. Kubernetes Deployment

### 3.1 Namespace Setup

```bash
kubectl create namespace erp-autonomous-coding
kubectl label namespace erp-autonomous-coding istio-injection=enabled
```

### 3.2 Helm Chart Structure

```
helm/erp-autonomous-coding/
  Chart.yaml
  values.yaml
  values-staging.yaml
  values-production.yaml
  templates/
    _helpers.tpl
    api-gateway-deployment.yaml
    agent-core-deployment.yaml
    agent-core-hpa.yaml
    git-bridge-deployment.yaml
    ide-server-deployment.yaml
    ide-server-hpa.yaml
    review-engine-deployment.yaml
    review-engine-hpa.yaml
    task-planner-deployment.yaml
    sandbox-runtime-daemonset.yaml
    dashboard-deployment.yaml
    postgresql-statefulset.yaml
    redis-statefulset.yaml
    redpanda-statefulset.yaml
    configmap.yaml
    secrets.yaml
    ingress.yaml
    service-accounts.yaml
    network-policies.yaml
    pod-disruption-budgets.yaml
```

### 3.3 HPA Configuration (Agent Core)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: agent-core-hpa
  namespace: erp-autonomous-coding
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: agent-core
  minReplicas: 5
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: active_sessions
        target:
          type: AverageValue
          averageValue: "5"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 3
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
```

### 3.4 Sandbox Runtime DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: sandbox-runtime
  namespace: erp-autonomous-coding
spec:
  selector:
    matchLabels:
      app: sandbox-runtime
  template:
    metadata:
      labels:
        app: sandbox-runtime
    spec:
      containers:
        - name: sandbox-runtime
          image: erp/sandbox-runtime:1.0.0
          securityContext:
            privileged: true
          volumeMounts:
            - name: docker-sock
              mountPath: /var/run/docker.sock
            - name: sandbox-images
              mountPath: /var/lib/sandbox-images
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "2"
              memory: "4Gi"
      volumes:
        - name: docker-sock
          hostPath:
            path: /var/run/docker.sock
        - name: sandbox-images
          hostPath:
            path: /var/lib/sandbox-images
```

---

## 4. Environment Configuration

### 4.1 Environment Variables

| Variable | Service | Required | Description |
|----------|---------|----------|-------------|
| `DATABASE_URL` | All Python services | Yes | PostgreSQL connection string |
| `REDIS_URL` | Agent Core, Gateway | Yes | Redis connection string |
| `KAFKA_BROKERS` | All | Yes | Comma-separated Kafka broker addresses |
| `CLAUDE_API_KEY` | Agent Core | Yes | Anthropic Claude API key |
| `CLAUDE_MODEL` | Agent Core | No | Default: `claude-sonnet-4-20250514` |
| `VAULT_ADDR` | All | Yes (prod) | HashiCorp Vault address |
| `VAULT_ROLE` | All | Yes (prod) | Vault AppRole role name |
| `IAM_JWKS_URL` | Gateway | Yes | ERP-IAM JWKS endpoint |
| `PLATFORM_GRPC_ADDR` | Gateway | Yes | ERP-Platform gRPC address |
| `SNYK_TOKEN` | Review Engine | Yes | Snyk API token |
| `SANDBOX_POOL_SIZE` | Sandbox Runtime | No | Default: 20 warm containers |
| `SANDBOX_MAX_CONTAINERS` | Sandbox Runtime | No | Default: 100 |
| `LOG_LEVEL` | All | No | Default: `info` |
| `OTEL_EXPORTER_ENDPOINT` | All | No | OpenTelemetry collector endpoint |

---

## 5. Deployment Pipeline

```mermaid
flowchart LR
    subgraph "CI (GitHub Actions)"
        LINT["Lint & Format"] --> TEST["Unit Tests"]
        TEST --> BUILD["Docker Build"]
        BUILD --> SCAN["Trivy Image Scan"]
        SCAN --> PUSH["Push to Registry"]
    end

    subgraph "CD (ArgoCD)"
        PUSH --> STAGE["Deploy to Staging"]
        STAGE --> SMOKE["Smoke Tests"]
        SMOKE --> GATE["Manual Approval Gate"]
        GATE --> CANARY["Canary Deploy (10%)"]
        CANARY --> MONITOR["Monitor 15min"]
        MONITOR --> FULL["Full Production Rollout"]
    end

    subgraph "Rollback"
        MONITOR -->|Errors > threshold| ROLLBACK["Auto Rollback"]
        FULL -->|Manual trigger| ROLLBACK
    end
```

---

## 6. Pre-built Sandbox Images

| Image | Base | Pre-installed | Size |
|-------|------|---------------|------|
| `erp/sandbox-go:1.22` | golang:1.22-alpine | go, git, make | ~400MB |
| `erp/sandbox-python:3.12` | python:3.12-slim | python, pip, pytest, mypy | ~350MB |
| `erp/sandbox-node:20` | node:20-slim | node, npm, yarn, pnpm | ~300MB |
| `erp/sandbox-rust:1.77` | rust:1.77-slim | rustc, cargo, clippy | ~600MB |
| `erp/sandbox-java:21` | eclipse-temurin:21 | java, maven, gradle | ~500MB |
| `erp/sandbox-dotnet:8` | mcr.microsoft.com/dotnet/sdk:8.0 | dotnet CLI, nuget | ~500MB |
| `erp/sandbox-multi` | ubuntu:22.04 | All of the above | ~2GB |

Images are rebuilt weekly and scanned by Trivy. Vulnerability-free images are promoted to the `latest` tag.

---

## 7. Health Checks and Readiness

| Service | Liveness | Readiness | Startup Probe |
|---------|----------|-----------|---------------|
| API Gateway | `GET /healthz` (5s interval) | `GET /readyz` (checks DB, Redis) | 30s timeout |
| Agent Core | `GET /healthz` (10s interval) | `GET /readyz` (checks Claude API, Sandbox) | 60s timeout |
| Git Bridge | `GET /healthz` (5s interval) | `GET /readyz` (checks Git providers) | 30s timeout |
| IDE Server | `GET /healthz` (5s interval) | `GET /readyz` (WebSocket ready) | 15s timeout |
| Review Engine | `GET /healthz` (10s interval) | `GET /readyz` (checks Snyk, Trivy) | 60s timeout |
| Task Planner | `GET /healthz` (5s interval) | `GET /readyz` (checks DB) | 30s timeout |
| Sandbox Runtime | `GET /healthz` (5s interval) | `GET /readyz` (checks Docker, pool size) | 30s timeout |

---

## 8. Disaster Recovery

| Scenario | RTO | RPO | Strategy |
|----------|-----|-----|----------|
| Single pod failure | < 30s | 0 | Kubernetes auto-restart |
| Node failure | < 2min | 0 | Pod rescheduling, PDB |
| AZ failure | < 5min | 0 | Multi-AZ deployment |
| Region failure | < 30min | < 5min | Active-passive failover |
| Database corruption | < 1hr | < 15min | WAL-based PITR |
| Complete cluster loss | < 2hr | < 1hr | Velero backup restore |
