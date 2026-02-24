# Deployment Guide -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Deployment Overview

ERP-BSS-OSS supports three deployment models: local development (Docker Compose), cloud Kubernetes (EKS/GKE/AKS), and on-premises bare metal or VM.

```mermaid
graph TB
    subgraph "Deployment Models"
        DEV["Local Dev<br/>(Docker Compose)"]
        CLOUD["Cloud K8s<br/>(EKS/GKE/AKS)"]
        ONPREM["On-Premises<br/>(kubeadm/K3s)"]
    end

    subgraph "Environments"
        DEV --> DEV_ENV["development"]
        CLOUD --> STG["staging"]
        CLOUD --> PROD["production"]
        ONPREM --> PROD_OP["production (on-prem)"]
    end
```

---

## 2. Local Development Deployment

### 2.1 Prerequisites

| Tool | Version |
|------|---------|
| Docker | 24+ |
| Docker Compose | 2.0+ |
| Rust | 1.83+ |
| Make | GNU Make |

### 2.2 Quick Start

```bash
# Clone
git clone https://github.com/abiolaogu/BSS-OSS.git
cd BSS-OSS

# Start all infrastructure
docker-compose up -d

# Verify services
docker-compose ps

# Build and run the API
cargo run --release --bin bss-api-server
```

### 2.3 Service Ports

| Service | Port | URL |
|---------|------|-----|
| BSS API | 8080 | http://localhost:8080 |
| PostgreSQL | 5432 | postgresql://postgres:postgres@localhost/bss |
| Redis | 6379 | redis://localhost:6379 |
| MongoDB | 27017 | mongodb://localhost:27017/bss |
| RabbitMQ | 5672/15672 | http://localhost:15672 (guest/guest) |
| Kafka | 9092 | localhost:9092 |
| Jaeger | 16686 | http://localhost:16686 |
| Prometheus | 9091 | http://localhost:9091 |
| Grafana | 3000 | http://localhost:3000 (admin/admin) |

---

## 3. Kubernetes Deployment

### 3.1 Cluster Requirements

| Component | Minimum |
|-----------|---------|
| K8s version | 1.29+ |
| Worker nodes | 3 |
| Total vCPU | 12 |
| Total RAM | 48 GB |
| Storage class | SSD-backed |

### 3.2 Helm Chart Deployment

```bash
# Add BSS-OSS Helm repository
helm repo add bss-oss https://charts.bss-oss.com
helm repo update

# Create namespace
kubectl create namespace bss-production

# Install with values
helm install bss-oss bss-oss/bss-oss \
  --namespace bss-production \
  --values values-production.yaml \
  --set global.environment=production \
  --set global.domain=bss.example.com
```

### 3.3 Production values.yaml

```yaml
global:
  environment: production
  domain: bss.example.com
  tenant_isolation: true

postgresql:
  primary:
    persistence:
      size: 500Gi
      storageClass: gp3
  readReplicas:
    replicaCount: 2

redis:
  sentinel:
    enabled: true
  replica:
    replicaCount: 3

kafka:
  replicaCount: 3
  defaultReplicationFactor: 3

services:
  billing-rating:
    replicas: 5
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 2000m
        memory: 2Gi

  charging-engine:
    replicas: 5
    resources:
      requests:
        cpu: 1000m
        memory: 1Gi

  mediation:
    replicas: 5
    resources:
      requests:
        cpu: 1000m
        memory: 1Gi
```

---

## 4. CI/CD Pipeline

```mermaid
graph LR
    PR["Pull Request"] --> LINT["Lint<br/>(clippy + fmt)"]
    LINT --> TEST["Test<br/>(cargo test)"]
    TEST --> BUILD["Build<br/>(cargo build --release)"]
    BUILD --> DOCKER["Docker Build"]
    DOCKER --> SCAN["Security Scan<br/>(Trivy)"]
    SCAN --> PUSH["Push to<br/>Container Registry"]
    PUSH --> DEPLOY_STG["Deploy Staging<br/>(helm upgrade)"]
    DEPLOY_STG --> SMOKE["Smoke Tests"]
    SMOKE --> APPROVE["Manual Approval"]
    APPROVE --> DEPLOY_PROD["Rolling Deploy<br/>Production"]
    DEPLOY_PROD --> VERIFY["Health Check<br/>Verification"]
```

---

## 5. Rolling Update Strategy

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  minReadySeconds: 30
  progressDeadlineSeconds: 600
```

**Zero-downtime deployment:**
1. New pods are created with new version
2. Readiness probe must pass (health check)
3. Old pods receive no new traffic
4. Old pods are terminated after drain period
5. Rollback on failure: `helm rollback bss-oss <revision>`

---

## 6. Database Migration Deployment

```bash
# Run migrations before application deployment
kubectl run --rm -i --tty migrate \
  --image=bss-oss/migration:latest \
  --namespace=bss-production \
  --env="DATABASE_URL=postgresql://..." \
  -- sqlx migrate run
```

**Migration safety rules:**
- Always add columns as nullable first
- Never drop columns in the same release that removes code references
- Use online schema change for large tables (> 1M rows)
- Test migrations against production-sized datasets

---

## 7. Monitoring Post-Deployment

### 7.1 Health Check Verification

```bash
# Verify all services are healthy
for svc in billing-rating customer-management order-management mediation provisioning; do
  echo "$svc: $(curl -s http://$svc.bss-production:8080/healthz | jq -r .status)"
done
```

### 7.2 Key Metrics to Watch

| Metric | Threshold | Action if breached |
|--------|-----------|-------------------|
| Error rate | > 1% for 2 min | Auto-rollback |
| P99 latency | > 200ms for 5 min | Alert + investigate |
| CPU utilization | > 80% for 5 min | HPA scales up |
| Memory usage | > 85% | Alert |
| Kafka consumer lag | > 100K messages | Alert |

---

## 8. Rollback Procedure

```bash
# List Helm releases
helm history bss-oss -n bss-production

# Rollback to previous version
helm rollback bss-oss <previous-revision> -n bss-production

# Verify rollback
kubectl get pods -n bss-production
```

---

## 9. Disaster Recovery

### 9.1 Backup Schedule

| Component | Method | Frequency | Retention |
|-----------|--------|-----------|-----------|
| PostgreSQL | WAL archiving + pg_dump | Continuous WAL, daily full | 30 days |
| Redis | RDB snapshots + AOF | Hourly RDB, continuous AOF | 7 days |
| ClickHouse | Partition backup to S3 | Daily | 90 days |
| MongoDB | Point-in-time (oplog) | Continuous | 30 days |
| Kafka | Topic mirroring (MM2) | Continuous | Per retention policy |
| K8s config | GitOps (Terraform + Helm) | Every commit | Unlimited (Git) |

### 9.2 Recovery Procedure

```mermaid
graph TB
    A[Disaster detected] --> B{Scope?}
    B -->|Single service| C[K8s auto-restart]
    B -->|Database failure| D[Failover to replica]
    B -->|Full PoP failure| E[DNS failover to nearest PoP]
    B -->|Data corruption| F[Restore from backup]

    C --> G[Verify health]
    D --> G
    E --> G
    F --> G

    G --> H[Post-incident review]
```
