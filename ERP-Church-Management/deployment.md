# Deployment Guide -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Deployment Options

ERP-Church-Management supports three deployment strategies:

```mermaid
flowchart LR
    subgraph Options
        DC["Docker Compose\n(Development/Small Church)"]
        K8S["Kubernetes\n(Medium/Large Church)"]
        MNG["Managed Cloud\n(Enterprise/Multi-Campus)"]
    end
```

---

## 2. Docker Compose Deployment (Development / Small Church)

### 2.1 Prerequisites

- Docker 24+ with Docker Compose v2
- 4+ vCPU, 16 GB RAM
- 100 GB SSD storage
- Domain name with SSL certificate (production)

### 2.2 Steps

```bash
# 1. Clone repository
git clone <repo-url> ERP-Church-Management
cd ERP-Church-Management

# 2. Configure environment
cp source-monolith/.env.example .env
# Edit .env with production values:
# - Strong JWT_SECRET
# - Real database credentials
# - Twilio/WhatsApp/Telegram credentials
# - Church-specific configuration

# 3. Start all services
docker compose up -d

# 4. Verify health
curl http://localhost:8093/healthz

# 5. Run database migrations
docker compose exec postgres psql -U erp -d erp_church_management \
  -f /database/migrations/0001_initial_core.sql

# 6. Create initial admin user
docker compose exec member-service /app/cmd/seed --admin-email admin@church.org

# 7. Configure reverse proxy (nginx/traefik) for SSL
```

### 2.3 Docker Compose Service Map

```yaml
# Key ports:
# Gateway:    8093 -> 8090
# PostgreSQL: 5435 -> 5432
# Redis:      6381 -> 6379
# Redpanda:   19094 -> 9092
```

---

## 3. Kubernetes Deployment (Medium / Large Church)

### 3.1 Prerequisites

- Kubernetes 1.28+ cluster (EKS, GKE, AKS, or self-managed)
- kubectl configured
- Helm 3+
- Managed PostgreSQL, Redis, and Kafka/Redpanda (recommended)

### 3.2 Namespace Setup

```bash
kubectl create namespace erp-church-management
kubectl create namespace erp-data
```

### 3.3 Secrets Configuration

```bash
kubectl create secret generic db-credentials \
  --namespace erp-church-management \
  --from-literal=DATABASE_URL='postgres://erp:password@db-host:5432/erp_church_management'

kubectl create secret generic redis-credentials \
  --namespace erp-church-management \
  --from-literal=REDIS_ADDR='redis-host:6379' \
  --from-literal=REDIS_PASSWORD='redis-password'

kubectl create secret generic communication-credentials \
  --namespace erp-church-management \
  --from-literal=TWILIO_ACCOUNT_SID='...' \
  --from-literal=TWILIO_AUTH_TOKEN='...' \
  --from-literal=WHATSAPP_API_KEY='...' \
  --from-literal=TELEGRAM_BOT_TOKEN='...'
```

### 3.4 Deployment Architecture

```mermaid
flowchart TB
    ING["Ingress Controller\n(nginx-ingress)"] --> GW["Gateway Deployment\n(2 replicas)"]

    subgraph Core["Core Services"]
        MS["member-service\n(2 replicas)"]
        VS["visitor-service\n(1 replica)"]
        FS["followup-service\n(1 replica)"]
        GS["giving-service\n(1 replica)"]
    end

    subgraph Engagement["Engagement Services"]
        ES["event-service\n(2 replicas)"]
        GRS["group-service\n(1 replica)"]
        DS["discipleship-service\n(1 replica)"]
    end

    subgraph Operations["Operations Services"]
        WS["welfare-service\n(1 replica)"]
        CS["communication-service\n(2 replicas)"]
        KS["kpi-service\n(1 replica)"]
        VOS["volunteer-service\n(1 replica)"]
        FAS["facility-service\n(1 replica)"]
    end

    GW --> Core & Engagement & Operations
```

### 3.5 Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: event-service-hpa
  namespace: erp-church-management
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: event-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### 3.6 Ingress Configuration

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: church-gateway
  namespace: erp-church-management
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - api.church.example.com
      secretName: church-tls
  rules:
    - host: api.church.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: gateway
                port:
                  number: 8090
```

---

## 4. Database Deployment

### 4.1 Managed PostgreSQL (Recommended)

| Provider | Service | Recommended Instance |
|---|---|---|
| AWS | RDS for PostgreSQL | db.r6g.large (2 vCPU, 16 GB) |
| GCP | Cloud SQL | db-custom-2-16384 |
| Azure | Azure Database for PostgreSQL | GP_Gen5_2 |

### 4.2 Migration Execution

```bash
# Apply all migrations in order
for f in database/migrations/*.sql; do
  psql -h $DB_HOST -U $DB_USER -d $DB_NAME -f "$f"
  echo "Applied: $f"
done
```

---

## 5. SSL/TLS Configuration

### 5.1 Certificate Management

For Kubernetes: Use cert-manager with Let's Encrypt:
```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --set installCRDs=true
```

For Docker Compose: Use Caddy or nginx-proxy with ACME:
```bash
docker run -d \
  -p 80:80 -p 443:443 \
  -v caddy_data:/data \
  caddy reverse-proxy \
    --from api.church.example.com \
    --to localhost:8093
```

---

## 6. Monitoring Setup

### 6.1 Prometheus + Grafana Stack

```bash
# Kubernetes
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.adminPassword=admin

# Docker Compose: Add to docker-compose.yml
# prometheus:
#   image: prom/prometheus:v2.48.0
#   ports: ["9090:9090"]
# grafana:
#   image: grafana/grafana:10.2.0
#   ports: ["3001:3000"]
```

### 6.2 Key Dashboards to Configure

| Dashboard | Metrics |
|---|---|
| System Overview | Request rate, error rate, latency p50/p95/p99 |
| Service Health | Health status per service, restart count |
| Database | Connection pool usage, query latency, transaction rate |
| Business KPIs | 72-hour rate, conversion rate, giving totals |
| Sunday Peak | Real-time check-in rate, queue depth |

---

## 7. Backup and Recovery

### 7.1 Database Backup

```bash
# Daily backup script
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME \
  --format=custom \
  --file="/backups/erp_church_${DATE}.dump"

# Upload to S3
aws s3 cp "/backups/erp_church_${DATE}.dump" \
  "s3://church-backups/db/${DATE}.dump"

# Retain 30 days
find /backups -name "*.dump" -mtime +30 -delete
```

### 7.2 Recovery Procedure

```bash
# 1. Stop application traffic
kubectl scale deployment gateway --replicas=0 -n erp-church-management

# 2. Restore database
pg_restore -h $DB_HOST -U $DB_USER -d $DB_NAME \
  --clean --if-exists /backups/erp_church_YYYYMMDD.dump

# 3. Verify data integrity
psql -h $DB_HOST -U $DB_USER -d $DB_NAME \
  -c "SELECT count(*) FROM members;"

# 4. Resume traffic
kubectl scale deployment gateway --replicas=2 -n erp-church-management
```

---

## 8. Rolling Update Procedure

```bash
# 1. Build new image
docker build -t erp-church-member-service:v1.0.1 services/member-service/

# 2. Push to registry
docker push registry.example.com/erp-church-member-service:v1.0.1

# 3. Update deployment
kubectl set image deployment/member-service \
  member-service=registry.example.com/erp-church-member-service:v1.0.1 \
  -n erp-church-management

# 4. Monitor rollout
kubectl rollout status deployment/member-service -n erp-church-management

# 5. Rollback if needed
kubectl rollout undo deployment/member-service -n erp-church-management
```

---

## 9. Pre-Go-Live Checklist

| # | Item | Status |
|---|---|---|
| 1 | Database migrations applied | [ ] |
| 2 | Admin user created | [ ] |
| 3 | SSL certificate installed | [ ] |
| 4 | Communication channels tested (SMS, WhatsApp, Email) | [ ] |
| 5 | ERP-IAM integration verified | [ ] |
| 6 | ERP-Platform entitlements configured | [ ] |
| 7 | Backup schedule configured | [ ] |
| 8 | Monitoring dashboards operational | [ ] |
| 9 | Health check endpoints responding | [ ] |
| 10 | Load test completed (Sunday peak simulation) | [ ] |
| 11 | User accounts created for all roles | [ ] |
| 12 | Training completed for admin team | [ ] |
