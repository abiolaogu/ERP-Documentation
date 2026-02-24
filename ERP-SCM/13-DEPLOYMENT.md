# ERP-SCM Deployment Guide

## 1. Overview

ERP-SCM supports multiple deployment models: local development with Docker Compose, single-node deployment, and production Kubernetes clusters. This guide covers all deployment scenarios with step-by-step instructions.

---

## 2. Deployment Architecture

```mermaid
flowchart TB
    subgraph "Production - Kubernetes"
        subgraph "Ingress Layer"
            IG["NGINX Ingress Controller"]
            CERT["cert-manager<br/>Let's Encrypt"]
        end

        subgraph "Application Layer"
            GW["API Gateway<br/>3 replicas"]
            SVC1["procurement-service<br/>2 replicas"]
            SVC2["inventory-service<br/>2 replicas"]
            SVC3["warehouse-service<br/>3 replicas"]
            SVC4["manufacturing-service<br/>2 replicas"]
            SVC5["demand-planning-service<br/>2 replicas"]
            SVC6["logistics-service<br/>2 replicas"]
            SVC7["quality-service<br/>2 replicas"]
            SVC8["fleet-service<br/>2 replicas"]
            SVC9["supplier-portal-service<br/>2 replicas"]
        end

        subgraph "Data Layer"
            PG["PostgreSQL 16<br/>Primary + 2 Replicas"]
            RD["Redis Sentinel<br/>3 nodes"]
            RP["Redpanda<br/>3 brokers"]
            S3["MinIO / S3"]
        end

        subgraph "Observability"
            OTEL["OpenTelemetry Collector"]
            PROM["Prometheus"]
            GRAF["Grafana"]
            LOKI["Loki"]
        end
    end

    IG --> GW
    GW --> SVC1 & SVC2 & SVC3 & SVC4 & SVC5 & SVC6 & SVC7 & SVC8 & SVC9
    SVC1 & SVC2 & SVC3 & SVC4 & SVC5 & SVC6 & SVC7 & SVC8 & SVC9 --> PG & RD & RP & S3
    SVC1 & SVC2 & SVC3 & SVC4 & SVC5 & SVC6 & SVC7 & SVC8 & SVC9 --> OTEL
    OTEL --> PROM & LOKI
    PROM --> GRAF
    LOKI --> GRAF
```

---

## 3. Prerequisites

| Component | Minimum Version | Purpose |
|---|---|---|
| Docker | 24.0+ | Container runtime |
| Docker Compose | 2.20+ | Local development |
| Kubernetes | 1.28+ | Production orchestration |
| Helm | 3.12+ | K8s package management |
| Python | 3.11+ | Backend runtime |
| Node.js | 20 LTS | Frontend build |
| PostgreSQL | 16+ | Primary database |
| Redis | 7+ | Cache and sessions |

---

## 4. Local Development (Docker Compose)

### 4.1 Quick Start

```bash
# Clone and navigate
cd /Users/AbiolaOgunsakin1/ERP/ERP-SCM

# Start all services
docker-compose up --build

# Access points:
# Frontend: http://localhost:3000
# Backend API: http://localhost:8000/docs
# API Health: http://localhost:8000/api/health
```

### 4.2 Development Without Docker

```bash
# Backend
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m seed_data          # Seed demo data
uvicorn app.main:app --reload --port 8000

# Frontend (separate terminal)
cd frontend
npm install
npm run dev                   # http://localhost:5173
```

### 4.3 Environment Variables

| Variable | Default | Description |
|---|---|---|
| `DATABASE_URL` | `sqlite:///./scm.db` | Database connection string |
| `SECRET_KEY` | `scm-secret-key-change-in-production` | JWT signing key |
| `ALGORITHM` | `HS256` | JWT algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `1440` | Token lifetime |
| `DEBUG` | `true` | Debug mode |
| `CORS_ORIGINS` | `["http://localhost:5173","http://localhost:3000"]` | Allowed origins |
| `AI_MODEL_RETRAIN_HOURS` | `24` | Model retraining interval |
| `DEMAND_FORECAST_HORIZON_DAYS` | `30` | Default forecast horizon |
| `ANOMALY_DETECTION_THRESHOLD` | `0.05` | Anomaly sensitivity |

---

## 5. Production Deployment (Kubernetes)

### 5.1 Namespace Setup

```bash
kubectl create namespace erp-scm
kubectl label namespace erp-scm app=erp-scm environment=production
```

### 5.2 Secrets

```bash
kubectl create secret generic scm-db-credentials \
  --namespace erp-scm \
  --from-literal=POSTGRES_USER=scm_app \
  --from-literal=POSTGRES_PASSWORD=$(openssl rand -base64 32) \
  --from-literal=DATABASE_URL="postgresql://scm_app:PASSWORD@pg-primary:5432/scm"

kubectl create secret generic scm-jwt-secret \
  --namespace erp-scm \
  --from-literal=SECRET_KEY=$(openssl rand -base64 64)
```

### 5.3 Helm Values (Production)

```yaml
# values-production.yaml
global:
  environment: production
  imageRegistry: registry.company.com/erp-scm
  imageTag: "1.0.0"

gateway:
  replicas: 3
  resources:
    requests:
      cpu: 500m
      memory: 512Mi
    limits:
      cpu: 1000m
      memory: 1Gi
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 10
    targetCPUUtilization: 70

services:
  procurement:
    replicas: 2
    resources:
      requests: { cpu: 250m, memory: 512Mi }
  inventory:
    replicas: 2
    resources:
      requests: { cpu: 250m, memory: 512Mi }
  warehouse:
    replicas: 3
    resources:
      requests: { cpu: 500m, memory: 512Mi }
  manufacturing:
    replicas: 2
    resources:
      requests: { cpu: 500m, memory: 1Gi }
  demandPlanning:
    replicas: 2
    resources:
      requests: { cpu: 1000m, memory: 2Gi }  # ML workloads
  logistics:
    replicas: 2
    resources:
      requests: { cpu: 250m, memory: 512Mi }
  quality:
    replicas: 2
    resources:
      requests: { cpu: 250m, memory: 256Mi }
  fleet:
    replicas: 2
    resources:
      requests: { cpu: 250m, memory: 256Mi }
  supplierPortal:
    replicas: 2
    resources:
      requests: { cpu: 250m, memory: 256Mi }

postgresql:
  enabled: true
  architecture: replication
  primary:
    resources:
      requests: { cpu: 2000m, memory: 4Gi }
    persistence:
      size: 100Gi
      storageClass: fast-ssd
  readReplicas:
    replicaCount: 2

redis:
  enabled: true
  architecture: sentinel
  sentinel:
    enabled: true

redpanda:
  enabled: true
  replicas: 3
  storage:
    persistentVolume:
      size: 50Gi

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: scm.company.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: scm-tls
      hosts:
        - scm.company.com
```

### 5.4 Deploy

```bash
helm install erp-scm ./helm/erp-scm \
  --namespace erp-scm \
  --values values-production.yaml \
  --wait --timeout 10m
```

---

## 6. Database Migrations

```bash
# Run migrations
cd backend
alembic upgrade head

# Create new migration
alembic revision --autogenerate -m "add_quality_tables"

# Rollback
alembic downgrade -1
```

---

## 7. Health Checks

| Endpoint | Type | Interval | Timeout |
|---|---|---|---|
| `/api/health` | HTTP GET | 30s | 10s |
| `/healthz` | HTTP GET (liveness) | 10s | 5s |
| `/readyz` | HTTP GET (readiness) | 10s | 5s |

### Health Response

```json
{
  "status": "healthy",
  "version": "1.0.0",
  "checks": {
    "database": "ok",
    "redis": "ok",
    "event_bus": "ok"
  }
}
```

---

## 8. CI/CD Pipeline

```mermaid
flowchart LR
    A["Git Push"] --> B["Lint & Format<br/>ruff + prettier"]
    B --> C["Unit Tests<br/>pytest"]
    C --> D["Integration Tests<br/>testcontainers"]
    D --> E["Build Docker Images"]
    E --> F["Security Scan<br/>Trivy + Bandit"]
    F --> G["Push to Registry"]
    G --> H["Deploy to Staging"]
    H --> I["E2E Tests<br/>Playwright"]
    I --> J["Manual Approval"]
    J --> K["Deploy to Production"]
    K --> L["Smoke Tests"]
    L --> M["Monitor 15min"]
```

---

## 9. Backup & Recovery

| Component | Backup Method | Frequency | Retention |
|---|---|---|---|
| PostgreSQL | pg_dump + WAL archiving | Continuous WAL, daily full | 30 days full, 7 days WAL |
| Redis | RDB snapshots | Every 15 minutes | 7 days |
| Object Storage | Cross-region replication | Real-time | Indefinite |
| Event Bus | Topic mirroring | Real-time | Per topic retention |

### Recovery Time Objectives

| Scenario | RTO | RPO |
|---|---|---|
| Single service failure | < 2 minutes (pod restart) | 0 (stateless) |
| Database failover | < 30 seconds (auto-failover) | 0 (synchronous replication) |
| Full cluster recovery | < 30 minutes | < 5 minutes |
| Disaster recovery (cross-region) | < 4 hours | < 15 minutes |

---

## 10. Scaling Guidelines

| Service | Scale Trigger | Max Pods |
|---|---|---|
| Gateway | CPU > 70% | 10 |
| Warehouse | Queue depth > 100 | 8 |
| Demand Planning | Forecast job queue > 5 | 6 |
| Fleet | GPS event rate > 1000/s | 6 |
| Others | CPU > 75% | 4 |
