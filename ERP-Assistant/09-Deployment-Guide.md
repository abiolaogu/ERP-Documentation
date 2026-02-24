# ERP-Assistant Deployment Guide

## 1. Overview

ERP-Assistant is deployed as a multi-service application using Docker Compose for development and Kubernetes for production. The system comprises six application services, two infrastructure services (PostgreSQL, Redis), and optional external dependencies (Qdrant, Redpanda).

### Service Inventory

| Service | Language | Port (Container) | Port (Host) | Image |
|---------|----------|------------------|-------------|-------|
| assistant-api (gateway) | Go 1.22 | 8090 | 8094 | Custom (Dockerfile) |
| assistant-core | Go 1.22 | 8080 | - | Custom |
| connector-hub | Go 1.22 | 8080 | - | Custom |
| action-engine | Go 1.22 | 8080 | - | Custom |
| memory-service | Python 3.12 | 8080 | 8204 | Custom |
| briefing-service | Go 1.22 | 8080 | - | Custom |
| voice-service | Python 3.12 | 8080 | 8208 | Custom |
| PostgreSQL | - | 5432 | 5432 | postgres:16 |
| Redis | - | 6379 | 6379 | redis:7 |

## 2. Prerequisites

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| Docker | 24+ | Container runtime |
| Docker Compose | 3.9+ | Multi-service orchestration |
| Go | 1.22+ | Building Go services |
| Python | 3.12+ | Building Python services |
| Node.js | 20+ | Frontend development |
| Flutter | 3.19+ | Mobile development |

### Required Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `ERP_PLATFORM_BASE_URL` | Yes | `http://host.docker.internal:8091` | ERP-Platform API URL |
| `ERP_IAM_BASE_URL` | Yes | - | ERP-IAM API URL for JWT validation |
| `CLAUDE_API_KEY` | Yes | - | Anthropic Claude API key |
| `DATABASE_URL` | No | `postgres://erp:erp@postgres:5432/erp_assistant` | PostgreSQL connection string |
| `REDIS_URL` | No | `redis://redis:6379` | Redis connection string |
| `QDRANT_URL` | No | `http://qdrant:6333` | Qdrant vector DB URL |
| `ELEVENLABS_API_KEY` | No | - | ElevenLabs TTS API key |
| `ENCRYPTION_KEY` | Yes | - | AES-256 master encryption key |

## 3. Local Development Setup

### Quick Start

```bash
# Clone the repository
cd /Users/AbiolaOgunsakin1/ERP/ERP-Assistant

# Start all services
docker compose up --build

# Verify services are running
curl http://localhost:8094/healthz
# {"status":"healthy","module":"ERP-Assistant"}

curl http://localhost:8094/v1/capabilities
# Returns capabilities JSON
```

### Docker Compose Architecture

```mermaid
graph TB
    subgraph "Docker Network: erp-assistant"
        API["assistant-api<br/>:8094 -> :8090"]
        CORE["assistant-core"]
        HUB["connector-hub"]
        ENGINE["action-engine"]
        MEM["memory-service<br/>:8204 -> :8080"]
        BRIEF["briefing-service"]
        VOICE["voice-service<br/>:8208 -> :8080"]
        PG["postgres:16<br/>:5432"]
        RD["redis:7<br/>:6379"]
    end

    HOST["Host Machine"] -->|":8094"| API
    HOST -->|":8204"| MEM
    HOST -->|":8208"| VOICE
    API --> CORE & HUB & ENGINE & BRIEF
    CORE --> MEM
    API --> PG
    API --> RD
```

### Building Individual Services

```bash
# Build and test the gateway
cd /Users/AbiolaOgunsakin1/ERP/ERP-Assistant
go test ./cmd/server ./tests

# Build the gateway binary
go build -o assistant ./cmd/server

# Build frontend
cd frontend/web
npm install
npm run dev     # Development server
npm run build   # Production build
```

## 4. Production Deployment

### Kubernetes Architecture

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "Ingress"
            ING["Nginx Ingress<br/>TLS Termination"]
        end

        subgraph "Application Namespace: erp-assistant"
            DEP_API["Deployment: assistant-api<br/>replicas: 3"]
            DEP_CORE["Deployment: assistant-core<br/>replicas: 3"]
            DEP_HUB["Deployment: connector-hub<br/>replicas: 2"]
            DEP_ENGINE["Deployment: action-engine<br/>replicas: 2"]
            DEP_MEM["Deployment: memory-service<br/>replicas: 2"]
            DEP_BRIEF["Deployment: briefing-service<br/>replicas: 2"]
            DEP_VOICE["Deployment: voice-service<br/>replicas: 2"]
        end

        subgraph "Data Namespace: erp-data"
            PG["StatefulSet: postgres<br/>replicas: 3 (1 primary + 2 read)"]
            RD["StatefulSet: redis<br/>Sentinel: 3 nodes"]
            QD["StatefulSet: qdrant<br/>replicas: 3"]
            RP["StatefulSet: redpanda<br/>brokers: 3"]
        end

        subgraph "Monitoring"
            PROM["Prometheus"]
            GRAF["Grafana"]
            JAEG["Jaeger"]
        end
    end

    ING --> DEP_API
    DEP_API --> DEP_CORE & DEP_HUB & DEP_ENGINE & DEP_BRIEF
    DEP_CORE --> DEP_MEM
    DEP_API --> PG & RD
    DEP_MEM --> QD
    DEP_ENGINE --> RP
```

### Kubernetes Manifests

```yaml
# deployment-assistant-api.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: assistant-api
  namespace: erp-assistant
  labels:
    app: assistant-api
    module: erp-assistant
spec:
  replicas: 3
  selector:
    matchLabels:
      app: assistant-api
  template:
    metadata:
      labels:
        app: assistant-api
    spec:
      containers:
        - name: assistant-api
          image: registry.erp.internal/erp-assistant/api:1.0.0
          ports:
            - containerPort: 8090
          env:
            - name: ERP_PLATFORM_BASE_URL
              valueFrom:
                configMapKeyRef:
                  name: assistant-config
                  key: platform_url
            - name: ENCRYPTION_KEY
              valueFrom:
                secretKeyRef:
                  name: assistant-secrets
                  key: encryption_key
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8090
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /healthz
              port: 8090
            initialDelaySeconds: 3
            periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: assistant-api
  namespace: erp-assistant
spec:
  selector:
    app: assistant-api
  ports:
    - port: 8090
      targetPort: 8090
  type: ClusterIP
```

### Resource Sizing

| Service | CPU Request | CPU Limit | Memory Request | Memory Limit | Replicas |
|---------|-----------|-----------|---------------|-------------|----------|
| assistant-api | 250m | 1000m | 256Mi | 512Mi | 3 |
| assistant-core | 500m | 2000m | 512Mi | 1Gi | 3 |
| connector-hub | 250m | 1000m | 256Mi | 512Mi | 2 |
| action-engine | 250m | 1000m | 256Mi | 512Mi | 2 |
| memory-service | 500m | 2000m | 512Mi | 2Gi | 2 |
| briefing-service | 250m | 1000m | 256Mi | 512Mi | 2 |
| voice-service | 1000m | 4000m | 1Gi | 4Gi | 2 |

## 5. CI/CD Pipeline

### GitHub Actions Workflow

```mermaid
flowchart LR
    PUSH["git push /<br/>Pull Request"] --> LINT["Go Lint +<br/>Python Lint"]
    LINT --> TEST["Unit Tests<br/>(go test)"]
    TEST --> BUILD["Docker Build<br/>(all services)"]
    BUILD --> SCAN["Image Scan<br/>(Trivy)"]
    SCAN --> PUSH_REG["Push to Registry"]
    PUSH_REG --> DEPLOY_STG["Deploy to Staging"]
    DEPLOY_STG --> INT_TEST["Integration Tests"]
    INT_TEST --> DEPLOY_PROD["Deploy to Production<br/>(manual approval)"]
```

### Current CI Configuration

The existing GitHub Actions workflow runs Go tests on push and pull request:

```yaml
name: ci
on:
  push:
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: "1.22"
      - run: go test ./cmd/server ./tests
```

## 6. Database Migrations

```bash
# Run migrations (using golang-migrate)
migrate -path db/migrations -database $DATABASE_URL up

# Rollback last migration
migrate -path db/migrations -database $DATABASE_URL down 1

# Check migration status
migrate -path db/migrations -database $DATABASE_URL version
```

## 7. Health Checks

| Service | Endpoint | Expected Response | Interval |
|---------|----------|------------------|----------|
| assistant-api | `GET /healthz` | `{"status": "healthy"}` | 10s |
| memory-service | `GET /healthz` | `{"status": "healthy", "service": "memory-service"}` | 10s |
| briefing-service | `GET /healthz` | `{"status": "healthy", "service": "briefing-service"}` | 10s |
| voice-service | `GET /healthz` | `{"status": "healthy", "service": "voice-service"}` | 10s |
| PostgreSQL | TCP :5432 | Connection accepted | 10s |
| Redis | `PING` | `PONG` | 5s |

## 8. Rollback Procedures

### Service Rollback

```bash
# Kubernetes: Roll back to previous version
kubectl rollout undo deployment/assistant-api -n erp-assistant

# Check rollback status
kubectl rollout status deployment/assistant-api -n erp-assistant

# Roll back to specific revision
kubectl rollout undo deployment/assistant-api --to-revision=2 -n erp-assistant
```

### Database Rollback

```bash
# Roll back one migration
migrate -path db/migrations -database $DATABASE_URL down 1

# Roll back to specific version
migrate -path db/migrations -database $DATABASE_URL goto 5
```

## 9. SSL/TLS Configuration

| Connection | Protocol | Certificate |
|-----------|----------|-------------|
| Client -> Ingress | TLS 1.3 | Let's Encrypt / ACM |
| Ingress -> Services | mTLS | Internal CA |
| Services -> PostgreSQL | TLS 1.3 | Internal CA |
| Services -> Redis | TLS 1.3 | Internal CA |
| Services -> External APIs | TLS 1.3 | Provider certificates |
