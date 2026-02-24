# ERP-DBaaS Local Environment Setup

This guide walks through setting up a complete local development environment for ERP-DBaaS, including all required tooling, infrastructure services, CRD installation, and operator deployment.

---

## Table of Contents

1. [Install Go 1.22](#install-go-122)
2. [Install Node.js 20 LTS](#install-nodejs-20-lts)
3. [Install Docker and Docker Compose](#install-docker-and-docker-compose)
4. [Install kubectl](#install-kubectl)
5. [Install a Local Kubernetes Cluster](#install-a-local-kubernetes-cluster)
6. [Clone and Configure the Repository](#clone-and-configure-the-repository)
7. [Start Infrastructure Services](#start-infrastructure-services)
8. [Run Database Migrations](#run-database-migrations)
9. [Install CRDs](#install-crds)
10. [Deploy Operators](#deploy-operators)
11. [Start the API and Gateway](#start-the-api-and-gateway)
12. [Verify the Setup](#verify-the-setup)
13. [Troubleshooting](#troubleshooting)

---

## Install Go 1.22

The Go gateway requires Go 1.22 or later.

### macOS (Homebrew)

```bash
brew install go

# Verify
go version
# Expected: go version go1.22.x darwin/arm64 (or darwin/amd64)
```

### Linux (Ubuntu/Debian)

```bash
# Download Go 1.22
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz

# Remove previous installation and extract
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

# Verify
go version
```

### Windows

Download the MSI installer from https://go.dev/dl/ and run it. Verify in PowerShell:

```powershell
go version
```

---

## Install Node.js 20 LTS

The Express.js API requires Node.js 20 LTS.

### macOS (Homebrew)

```bash
brew install node@20

# Verify
node --version  # v20.x.x
npm --version   # 10.x.x
```

### Using nvm (Recommended for version management)

```bash
# Install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Reload shell
source ~/.zshrc  # or ~/.bashrc

# Install and use Node.js 20
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node --version  # v20.x.x
npm --version   # 10.x.x
```

### Linux (Ubuntu/Debian via NodeSource)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify
node --version
npm --version
```

---

## Install Docker and Docker Compose

Docker is required for the local development infrastructure (YugabyteDB, DragonflyDB, Hasura).

### macOS

Install Docker Desktop from https://www.docker.com/products/docker-desktop/

Docker Compose v2 is included with Docker Desktop.

```bash
# Verify
docker --version          # Docker version 24.x or later
docker compose version    # Docker Compose version v2.20 or later
```

### Linux (Ubuntu/Debian)

```bash
# Install Docker
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your user to the docker group
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker --version
docker compose version
```

### Resource Recommendations

For running ERP-DBaaS locally with all infrastructure services:

| Resource | Minimum | Recommended |
|---|---|---|
| CPU cores | 4 | 8 |
| RAM | 8 GB | 16 GB |
| Disk space | 20 GB | 50 GB |

Configure Docker Desktop resource limits in Settings > Resources.

---

## Install kubectl

kubectl is required for Kubernetes CRD management and cluster operations.

### macOS (Homebrew)

```bash
brew install kubectl

# Verify
kubectl version --client
# Expected: Client Version: v1.28.x or later
```

### Linux

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Verify
kubectl version --client
```

---

## Install a Local Kubernetes Cluster

A local K8s cluster is needed for CRD testing and operator development. Choose one of the following:

### Option A: kind (Kubernetes in Docker) - Recommended

```bash
# Install kind
# macOS:
brew install kind
# Linux:
go install sigs.k8s.io/kind@v0.22.0

# Create a cluster with extra port mappings
cat <<EOF | kind create cluster --name dbaas-dev --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30090
        hostPort: 30090
        protocol: TCP
  - role: worker
  - role: worker
EOF

# Verify
kubectl cluster-info --context kind-dbaas-dev
kubectl get nodes
```

### Option B: minikube

```bash
# Install minikube
# macOS:
brew install minikube
# Linux:
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start with adequate resources
minikube start \
  --cpus=4 \
  --memory=8192 \
  --disk-size=50g \
  --kubernetes-version=v1.29.0

# Verify
kubectl cluster-info
minikube status
```

### Cluster Requirements for Full Testing

For testing all engine operators, the cluster needs:

| Requirement | kind | minikube |
|---|---|---|
| Nodes | 3 (1 control + 2 worker) | 1 (multi-node optional) |
| CPU | 4+ cores total | 4+ cores |
| Memory | 8+ GB total | 8+ GB |
| Storage class | standard (default) | standard (default) |

---

## Clone and Configure the Repository

```bash
# Navigate to the ERP directory
cd /path/to/ERP

# The ERP-DBaaS module is at:
cd ERP-DBaaS

# Install Node.js dependencies
make install
# Or manually:
cd services/dbaas-api && npm ci && cd ../..

# Verify Go module
cd cmd/server && go mod tidy && cd ../..
```

### Environment Configuration

Create a `.env` file in the project root for local overrides (optional):

```bash
# .env (for local development overrides)
PORT=8090
DBAAS_API_URL=http://localhost:3000
CORS_ORIGINS=http://localhost:3000,http://localhost:5173
NODE_ENV=development
LOG_LEVEL=debug
YUGABYTE_URL=postgresql://yugabyte:yugabyte@localhost:5433/dbaas_registry
DRAGONFLY_URL=redis://localhost:6379
JWT_SECRET=dev-secret-change-in-production
```

---

## Start Infrastructure Services

### Using Docker Compose (Recommended)

```bash
# Start YugabyteDB and DragonflyDB only (minimal for development)
cd infra && docker compose up -d yugabytedb dragonfly

# Or start all services including Hasura
cd infra && docker compose up -d

# Check service status
docker compose ps
```

### Service Ports

| Service | Port | Protocol | Purpose |
|---|---|---|---|
| YugabyteDB YSQL | 5433 | TCP | PostgreSQL-compatible SQL |
| YugabyteDB YCQL | 9042 | TCP | Cassandra-compatible CQL |
| YugabyteDB Master Web | 7000 | HTTP | Admin UI |
| YugabyteDB TServer Web | 9000 | HTTP | TServer metrics |
| DragonflyDB | 6379 | TCP | Redis-compatible protocol |
| Hasura Console | 19108 | HTTP | GraphQL Engine admin |

### Wait for YugabyteDB Readiness

YugabyteDB takes approximately 15-30 seconds to initialize:

```bash
# Wait for health check
until docker compose exec yugabytedb bin/ysqlsh -U yugabyte -c '\l' 2>/dev/null; do
  echo "Waiting for YugabyteDB..."
  sleep 5
done
echo "YugabyteDB is ready."
```

---

## Run Database Migrations

```bash
# Run the initial schema migration
make migrate

# Or manually with psql:
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -f migrations/001_initial_schema.sql
```

### Verify Migration

```bash
# List tables
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry -c '\dt'

# Expected output:
#                   List of relations
#  Schema |         Name          | Type  |  Owner
# --------+-----------------------+-------+----------
#  public | backup_records        | table | yugabyte
#  public | credential_rotations  | table | yugabyte
#  public | metering_events       | table | yugabyte
#  public | plugin_registrations  | table | yugabyte
#  public | service_instances     | table | yugabyte
#  public | tenant_quotas         | table | yugabyte
```

---

## Install CRDs

If you have a local Kubernetes cluster configured, install the DBaaS CRDs:

```bash
# Apply all CRDs
make crds-apply

# Or manually:
kubectl apply -f crds/

# Verify CRDs are installed
kubectl get crds | grep dbaas.businessactivation.cloud
```

Expected CRDs:
```
backuppolicies.dbaas.businessactivation.cloud
pluginregistrations.dbaas.businessactivation.cloud
policyprofiles.dbaas.businessactivation.cloud
restorejobs.dbaas.businessactivation.cloud
serviceinstances.dbaas.businessactivation.cloud
tenantdataplanes.dbaas.businessactivation.cloud
```

### Verify CRD Schema

```bash
# Check ServiceInstance CRD details
kubectl explain serviceinstance.spec
kubectl explain serviceinstance.status

# List short names
kubectl api-resources | grep dbaas
# Expected:
# serviceinstances   si     dbaas.businessactivation.cloud/v1   true    ServiceInstance
# backuppolicies     bp     dbaas.businessactivation.cloud/v1   true    BackupPolicy
# restorejobs        rj     dbaas.businessactivation.cloud/v1   true    RestoreJob
# tenantdataplanes   tdp    dbaas.businessactivation.cloud/v1   false   TenantDataPlane
# pluginregistrations plg   dbaas.businessactivation.cloud/v1   false   PluginRegistration
# policyprofiles     pp     dbaas.businessactivation.cloud/v1   false   PolicyProfile
```

---

## Deploy Operators

### Create the dbaas-system Namespace

```bash
kubectl apply -f infra/k8s/namespace.yaml
```

### Install KubeDB Operator (for YugabyteDB, DragonflyDB, MongoDB)

```bash
# Add KubeDB Helm repository
helm repo add appscode https://charts.appscode.com/stable/
helm repo update

# Install KubeDB operator
helm install kubedb appscode/kubedb \
  --namespace kubedb-system \
  --create-namespace \
  --set kubedb-provisioner.enabled=true \
  --set kubedb-ops-manager.enabled=true \
  --set kubedb-autoscaler.enabled=false

# Verify operator is running
kubectl get pods -n kubedb-system
```

### Install Scylla Operator

```bash
# Install cert-manager (prerequisite)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml

# Install Scylla Operator
helm repo add scylla https://scylla-operator-charts.storage.googleapis.com/stable
helm install scylla-operator scylla/scylla-operator \
  --namespace scylla-operator \
  --create-namespace
```

### Install Altinity ClickHouse Operator

```bash
kubectl apply -f https://raw.githubusercontent.com/Altinity/clickhouse-operator/master/deploy/operator/clickhouse-operator-install-bundle.yaml
```

### Install Zalando Postgres Operator (for TimescaleDB)

```bash
helm repo add postgres-operator-charts https://opensource.zalando.com/postgres-operator/charts/postgres-operator
helm install postgres-operator postgres-operator-charts/postgres-operator \
  --namespace postgres-operator \
  --create-namespace
```

### Verify All Operators

```bash
# Check all operator pods are running
kubectl get pods --all-namespaces | grep -E "(kubedb|scylla|clickhouse|postgres-operator)"
```

---

## Start the API and Gateway

### Option 1: Development Mode (hot-reloading)

```bash
# Terminal 1: Start the API with hot-reloading
cd /path/to/ERP/ERP-DBaaS
make dev

# Terminal 2: Start the Go gateway
cd /path/to/ERP/ERP-DBaaS
make dev-gateway
```

### Option 2: Docker Compose (all services)

```bash
cd /path/to/ERP/ERP-DBaaS
make docker-up
```

### Option 3: Build and Run Manually

```bash
# Build Go gateway
cd cmd/server && go build -o ../../bin/gateway . && cd ../..

# Build TypeScript API
cd services/dbaas-api && npm run build && cd ../..

# Run gateway
DBAAS_API_URL=http://localhost:3000 ./bin/gateway &

# Run API
cd services/dbaas-api && \
  YUGABYTE_URL="postgresql://yugabyte:yugabyte@localhost:5433/dbaas_registry" \
  DRAGONFLY_URL="redis://localhost:6379" \
  node dist/index.js
```

---

## Verify the Setup

### Service Health

```bash
# Gateway
curl -s http://localhost:8090/healthz | jq .
# {"status":"healthy","module":"ERP-DBaaS"}

# API
curl -s http://localhost:3000/healthz | jq .
# {"status":"healthy","service":"dbaas-api","timestamp":"..."}

# Capabilities
curl -s http://localhost:8090/v1/capabilities | jq .
```

### API Functionality

```bash
# List engines (using dev auth bypass)
curl -s http://localhost:8090/v1/dbaas/engines \
  -H "X-Dev-Tenant-ID: dev-tenant" | jq '.data | length'
# Expected: 8

# List plans
curl -s http://localhost:8090/v1/dbaas/plans \
  -H "X-Dev-Tenant-ID: dev-tenant" | jq '.data | length'
# Expected: 4

# Provision a test instance
curl -s -X POST http://localhost:8090/v1/dbaas/instances \
  -H "Content-Type: application/json" \
  -H "X-Dev-Tenant-ID: dev-tenant" \
  -d '{"engine":"yugabytedb","version":"2.21","plan":"S","haMode":"standalone"}' | jq .

# List instances
curl -s http://localhost:8090/v1/dbaas/instances \
  -H "X-Dev-Tenant-ID: dev-tenant" | jq '.data | length'
# Expected: 1
```

### Database Verification

```bash
# Check data in YugabyteDB
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT id, engine, status FROM service_instances;"

# Check tenant quota auto-created
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT * FROM tenant_quotas;"
```

### Kubernetes (if cluster configured)

```bash
# Check CRDs
kubectl get crds | grep dbaas

# Check for ServiceInstance resources
kubectl get si --all-namespaces
```

---

## Troubleshooting

### YugabyteDB Fails to Start

```bash
# Check YugabyteDB logs
cd infra && docker compose logs yugabytedb

# Common issue: Port 5433 already in use
lsof -i :5433
# Kill conflicting process or change the port in docker-compose.yaml
```

### DragonflyDB Connection Error

```bash
# Check DragonflyDB logs
cd infra && docker compose logs dragonfly

# Test connectivity
redis-cli -h localhost -p 6379 ping
# Expected: PONG

# If unavailable, the API will continue working (rate limiting disabled)
```

### Migration Fails

```bash
# Check if the database exists
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -c '\l'

# Create the database if it does not exist
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte \
  -c "CREATE DATABASE dbaas_registry;"

# Re-run migration
make migrate
```

### Go Gateway Build Errors

```bash
# Ensure Go 1.22+
go version

# Clean module cache and re-download
cd cmd/server && go clean -modcache && go mod tidy
```

### API Startup Errors

```bash
# Ensure dependencies are installed
cd services/dbaas-api && npm ci

# Check TypeScript compilation
cd services/dbaas-api && npx tsc --noEmit

# Run in verbose mode
LOG_LEVEL=debug npm run dev
```

### CRD Application Fails

```bash
# Ensure kubectl is connected to a cluster
kubectl cluster-info

# Check for CRD conflicts
kubectl get crds | grep dbaas

# Delete and re-apply if needed
make crds-delete
make crds-apply
```

---

## IDE Configuration

### VS Code Recommended Extensions

- **ESLint** (`dbaeumer.vscode-eslint`): TypeScript linting.
- **Go** (`golang.go`): Go language support.
- **YAML** (`redhat.vscode-yaml`): YAML/CRD validation.
- **Kubernetes** (`ms-kubernetes-tools.vscode-kubernetes-tools`): K8s resource management.
- **REST Client** (`humao.rest-client`): HTTP request testing.
- **PostgreSQL** (`ckolkman.vscode-postgres`): Database browser.

### VS Code Settings

```json
{
  "typescript.tsdk": "services/dbaas-api/node_modules/typescript/lib",
  "eslint.workingDirectories": ["services/dbaas-api"],
  "go.toolsEnvVars": {
    "GOFLAGS": "-mod=mod"
  },
  "yaml.schemas": {
    "kubernetes": ["crds/*.yaml", "infra/k8s/*.yaml", "operators/*.yaml"]
  }
}
```
