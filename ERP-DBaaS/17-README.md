# ERP-DBaaS Quick Start Guide

ERP-DBaaS is a Database-as-a-Service platform providing self-service provisioning, scaling, backup/restore, and lifecycle management for 8 AIDD-approved database engines, governed by policy profiles and deployed on Kubernetes via operator patterns.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start with Docker Compose](#quick-start-with-docker-compose)
3. [Verify Services](#verify-services)
4. [Provision Your First Instance](#provision-your-first-instance)
5. [API Reference Summary](#api-reference-summary)
6. [Development Workflow](#development-workflow)
7. [Project Structure](#project-structure)
8. [Configuration Reference](#configuration-reference)

---

## Prerequisites

### Required Software

| Tool | Minimum Version | Purpose |
|---|---|---|
| Go | 1.22+ | Go API gateway |
| Node.js | 20 LTS | Express.js API server |
| npm | 10+ | Node.js package manager |
| Docker | 24+ | Container runtime |
| Docker Compose | 2.20+ | Local orchestration |
| kubectl | 1.28+ | Kubernetes CLI (for K8s deployment) |
| psql | 15+ | PostgreSQL client (for migrations) |

### Optional Software

| Tool | Purpose |
|---|---|
| kind or minikube | Local Kubernetes cluster |
| Helm 3 | Kubernetes package management |
| Helmfile | Declarative Helm chart management |
| grpcurl | gRPC endpoint testing |

### Verify Prerequisites

```bash
# Verify all required tools
go version        # go1.22 or later
node --version    # v20.x or later
npm --version     # 10.x or later
docker --version  # 24.x or later
docker compose version  # v2.20 or later
kubectl version --client  # v1.28 or later
```

---

## Quick Start with Docker Compose

### Step 1: Clone and Install Dependencies

```bash
cd ERP-DBaaS

# Install Node.js dependencies for the API service
make install
# Or manually:
# cd services/dbaas-api && npm ci
```

### Step 2: Start All Services

```bash
# Start all services (YugabyteDB, DragonflyDB, Hasura, API, Gateway)
make docker-up
```

This command:
1. Builds Docker images for dbaas-gateway and dbaas-api.
2. Starts YugabyteDB (port 5433) and DragonflyDB (port 6379).
3. Waits for YugabyteDB health check to pass.
4. Runs database migrations (`001_initial_schema.sql`).
5. Starts the Express.js API (port 3000) and Go gateway (port 8090).
6. Starts Hasura GraphQL Engine (port 19108).

### Step 3: Verify Services Are Running

```bash
# Check Docker container status
cd infra && docker compose ps

# Expected output shows all services as "Up (healthy)"
```

### Alternative: Development Mode (without Docker)

```bash
# Start only infrastructure (YugabyteDB + DragonflyDB)
cd infra && docker compose up -d yugabytedb dragonfly

# Wait for YugabyteDB readiness, then run migrations
make migrate

# Start the API in watch mode (auto-reloads on file changes)
cd services/dbaas-api && npm run dev

# In a separate terminal, start the Go gateway
cd cmd/server && go run .
```

---

## Verify Services

### Health Checks

```bash
# Gateway health check
curl http://localhost:8090/healthz
# Response: {"status":"healthy","module":"ERP-DBaaS"}

# API health check
curl http://localhost:3000/healthz
# Response: {"status":"healthy","service":"dbaas-api","timestamp":"2026-02-24T10:00:00.000Z"}

# API readiness check
curl http://localhost:3000/readyz
# Response: {"status":"ready","service":"dbaas-api","timestamp":"2026-02-24T10:00:00.000Z"}

# Gateway capabilities
curl http://localhost:8090/v1/capabilities
# Response:
# {
#   "module": "ERP-DBaaS",
#   "version": "1.0.0",
#   "capabilities": [
#     "database_provisioning", "database_scaling", "backup_restore",
#     "credential_rotation", "policy_profiles", "plugin_management",
#     "metering", "multi_engine", "multi_tenant", "multi_region"
#   ],
#   "integration_mode": "platform_service",
#   "aidd_governance": "ERP_AIDD_STRICT"
# }
```

### Database Connectivity

```bash
# Connect to YugabyteDB
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry -c '\dt'

# Expected: 6 tables listed (service_instances, backup_records, plugin_registrations,
#           tenant_quotas, credential_rotations, metering_events)
```

---

## Provision Your First Instance

All API requests through the gateway use the `/v1/dbaas/` prefix. In development mode, authentication is bypassed using the `X-Dev-Tenant-ID` header.

### Step 1: List Available Engines

```bash
curl -s http://localhost:8090/v1/dbaas/engines \
  -H "X-Dev-Tenant-ID: tenant-001" | jq '.data[].engine'
```

Expected output (8 AIDD-approved engines under the default ERP_AIDD_STRICT profile):
```
"yugabytedb"
"scylladb"
"dragonfly"
"mongodb"
"couchdb"
"clickhouse"
"timescaledb"
"questdb"
```

### Step 2: List Available Plans

```bash
curl -s http://localhost:8090/v1/dbaas/plans \
  -H "X-Dev-Tenant-ID: tenant-001" | jq '.data'
```

Expected output:
```json
[
  { "size": "S", "displayName": "Small",       "resources": {"cpu":"500m","memory":"1Gi","storage":"10Gi"},  "monthlyPriceUsd": 29  },
  { "size": "M", "displayName": "Medium",      "resources": {"cpu":"2","memory":"8Gi","storage":"50Gi"},    "monthlyPriceUsd": 99  },
  { "size": "L", "displayName": "Large",       "resources": {"cpu":"4","memory":"16Gi","storage":"200Gi"},  "monthlyPriceUsd": 299 },
  { "size": "XL","displayName": "Extra Large",  "resources": {"cpu":"8","memory":"32Gi","storage":"500Gi"}, "monthlyPriceUsd": 599 }
]
```

### Step 3: Provision a YugabyteDB Instance

```bash
curl -s -X POST http://localhost:8090/v1/dbaas/instances \
  -H "Content-Type: application/json" \
  -H "X-Dev-Tenant-ID: tenant-001" \
  -d '{
    "engine": "yugabytedb",
    "version": "2.21",
    "plan": "M",
    "haMode": "standalone",
    "region": "africa-west"
  }' | jq .
```

Expected response:
```json
{
  "success": true,
  "data": {
    "id": "a1b2c3d4-...",
    "tenant_id": "tenant-001",
    "engine": "yugabytedb",
    "version": "2.21",
    "plan": "M",
    "ha_mode": "standalone",
    "status": "provisioning",
    "namespace": "dbaas-tenant-001-a1b2c3d4",
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "config": {},
    "created_at": "2026-02-24T10:05:00.000Z",
    "updated_at": "2026-02-24T10:05:00.000Z"
  },
  "message": "Instance provisioning initiated.",
  "requestId": "req-..."
}
```

### Step 4: Check Instance Status

```bash
# Replace {instance-id} with the actual ID from the provision response
curl -s http://localhost:8090/v1/dbaas/instances/{instance-id} \
  -H "X-Dev-Tenant-ID: tenant-001" | jq '.data.status'
```

### Step 5: List All Instances

```bash
curl -s http://localhost:8090/v1/dbaas/instances \
  -H "X-Dev-Tenant-ID: tenant-001" | jq .
```

### Step 6: Trigger a Backup

```bash
curl -s -X POST http://localhost:8090/v1/dbaas/instances/{instance-id}/backup \
  -H "Content-Type: application/json" \
  -H "X-Dev-Tenant-ID: tenant-001" \
  -d '{
    "type": "full",
    "retentionDays": 30
  }' | jq .
```

---

## API Reference Summary

All endpoints are prefixed with `/v1/dbaas/` through the gateway (port 8090).

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/healthz` | Gateway health check |
| `GET` | `/v1/capabilities` | Module capabilities |
| `POST` | `/v1/dbaas/instances` | Provision new instance |
| `GET` | `/v1/dbaas/instances` | List instances (paginated) |
| `GET` | `/v1/dbaas/instances/:id` | Get instance details |
| `GET` | `/v1/dbaas/instances/:id/metrics` | Get instance metrics |
| `PUT` | `/v1/dbaas/instances/:id/scale` | Scale instance |
| `PUT` | `/v1/dbaas/instances/:id/config` | Update configuration |
| `POST` | `/v1/dbaas/instances/:id/restart` | Rolling restart |
| `DELETE` | `/v1/dbaas/instances/:id` | Decommission instance |
| `POST` | `/v1/dbaas/instances/:id/backup` | Trigger backup |
| `GET` | `/v1/dbaas/instances/:id/backups` | List backups |
| `POST` | `/v1/dbaas/instances/:id/restore` | Restore from backup |
| `GET` | `/v1/dbaas/instances/:id/credentials` | Get credentials |
| `POST` | `/v1/dbaas/instances/:id/credentials/rotate` | Rotate credentials |
| `GET` | `/v1/dbaas/engines` | List available engines |
| `GET` | `/v1/dbaas/plans` | List available plans |
| `GET` | `/v1/dbaas/profiles` | List policy profiles |
| `GET` | `/v1/dbaas/plugins` | List registered plugins |
| `POST` | `/v1/dbaas/plugins` | Register new plugin |

For complete API documentation with request/response schemas, see [21-API-Documentation.md](./21-API-Documentation.md).

---

## Development Workflow

### Build

```bash
make build            # Build both gateway and API
make build-gateway    # Build Go gateway only
make build-api        # Build TypeScript API only
```

### Test

```bash
make test             # Run API unit tests (Vitest)
make test-integration # Run integration tests
make lint             # Run ESLint + Go vet
```

### Docker

```bash
make docker-build     # Build Docker images
make docker-up        # Start all services
make docker-down      # Stop all services
make docker-logs      # Tail all logs
```

### Database

```bash
make migrate          # Run YugabyteDB migrations
```

### Kubernetes (requires kubectl configured)

```bash
make crds-apply       # Apply CRDs to cluster
make crds-delete      # Remove CRDs from cluster
make k8s-deploy       # Deploy to Kubernetes
make k8s-delete       # Remove from Kubernetes
```

---

## Project Structure

```
ERP-DBaaS/
  cmd/server/
    main.go                      # Go API gateway (port 8090)
  services/dbaas-api/
    src/
      index.ts                   # Express.js entry point (port 3000)
      types/index.ts             # Core type definitions and enums
      routes/
        instances.ts             # Instance CRUD and lifecycle routes
        backups.ts               # Backup and restore routes
        credentials.ts           # Credential management routes
        engines.ts               # Engine, plan, and profile routes
        plugins.ts               # Plugin registration routes
      controllers/
        provisioner.ts           # K8s provisioning orchestrator
        scaler.ts                # Instance scaling controller
        backup-manager.ts        # Backup/restore orchestrator
        credential-rotator.ts    # Vault-based credential rotation
      policies/
        profile-engine.ts        # Central policy validation engine
        erp-aidd-strict.ts       # AIDD strict profile (immutable)
        commercial-flexible.ts   # Commercial flexible profile
      operators/
        base-operator.ts         # Base operator adapter interface
        kubedb-adapter.ts        # KubeDB (YugabyteDB, DragonflyDB, MongoDB)
        scylla-adapter.ts        # Scylla Operator
        clickhouse-adapter.ts    # Altinity Operator
        timescaledb-adapter.ts   # Zalando Postgres Operator
        questdb-adapter.ts       # QuestDB controller
        couchdb-adapter.ts       # CouchDB controller
        pulsar-adapter.ts        # Pulsar Operator (internal)
      plugins/
        registry.ts              # Plugin registry (CRUD)
        grpc-interface.ts        # gRPC lifecycle client
        validators/
          plugin-validator.ts    # Plugin validation pipeline
      metering/
        pulsar-emitter.ts        # Metering event emitter
      middleware/
        auth.ts                  # Authentik JWT validation
        rate-limiter.ts          # DragonflyDB rate limiter
        tenant-context.ts        # Tenant context enrichment
  migrations/
    001_initial_schema.sql       # YugabyteDB schema (6 tables)
  crds/
    service-instance.yaml        # ServiceInstance CRD
    backup-policy.yaml           # BackupPolicy CRD
    restore-job.yaml             # RestoreJob CRD
    tenant-data-plane.yaml       # TenantDataPlane CRD
    plugin-registration.yaml     # PluginRegistration CRD
    policy-profile.yaml          # PolicyProfile CRD
  operators/
    yugabytedb-cluster.yaml      # YugabyteDB operator manifest
    scylladb-cluster.yaml        # ScyllaDB operator manifest
    dragonfly-instance.yaml      # DragonflyDB operator manifest
    mongodb-cluster.yaml         # MongoDB operator manifest
    couchdb-statefulset.yaml     # CouchDB StatefulSet manifest
    clickhouse-cluster.yaml      # ClickHouse operator manifest
    timescaledb-cluster.yaml     # TimescaleDB operator manifest
    questdb-statefulset.yaml     # QuestDB StatefulSet manifest
    pulsar-cluster.yaml          # Pulsar operator manifest
  infra/
    docker-compose.yaml          # Local development environment
    Dockerfile.api               # API Docker image
    Dockerfile.gateway           # Gateway Docker image
    k8s/                         # Kubernetes deployment manifests
  configs/
    capabilities.json            # Module capability descriptor
  hasura/
    metadata/                    # Hasura metadata and actions
  scripts/
    provision-module.sh          # Module provisioning helper
    rotate-credentials.sh        # Credential rotation helper
  Makefile                       # Build and deployment targets
```

---

## Configuration Reference

### Environment Variables

#### Go Gateway (port 8090)

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8090` | Gateway listen port |
| `DBAAS_API_URL` | `http://dbaas-api:3000` | Backend API URL |
| `CORS_ORIGINS` | `*` | Allowed CORS origins (comma-separated) |

#### Node.js API (port 3000)

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3000` | API listen port |
| `NODE_ENV` | `development` | Environment mode |
| `LOG_LEVEL` | `info` | Pino log level |
| `YUGABYTE_URL` | `postgresql://yugabyte:yugabyte@yugabytedb:5433/dbaas_registry` | YugabyteDB connection string |
| `DRAGONFLY_URL` | `redis://dragonfly:6379` | DragonflyDB connection string |
| `PULSAR_URL` | `pulsar://pulsar:6650` | Apache Pulsar URL |
| `VAULT_ADDR` | `http://vault:8200` | HashiCorp Vault address |
| `RUSTFS_ENDPOINT` | `http://rustfs:9000` | RustFS object storage endpoint |
| `JWT_SECRET` | `dev-secret-change-in-production` | JWT signing key (development only) |
| `AUTHENTIK_JWKS_URL` | `http://authentik:9000/application/o/erp-dbaas/.well-known/jwks.json` | Authentik JWKS endpoint |
| `AUTHENTIK_ISSUER` | `http://authentik:9000/application/o/erp-dbaas/` | Authentik token issuer URL |

---

## Stopping Services

```bash
# Stop all Docker Compose services
make docker-down

# Or manually:
cd infra && docker compose down

# To also remove volumes (WARNING: destroys all data):
cd infra && docker compose down -v
```

---

## Next Steps

- **Full API Documentation**: See [21-API-Documentation.md](./21-API-Documentation.md)
- **Architecture Decisions**: See [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md)
- **Contributing**: See [19-CONTRIBUTING.md](./19-CONTRIBUTING.md)
- **Local Environment Setup**: See [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md)
- **Security**: See [31-SECURITY.md](./31-SECURITY.md)
