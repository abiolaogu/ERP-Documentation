# ERP-DBaaS Release Notes

## Document Control

| Field             | Value                          |
|-------------------|--------------------------------|
| Document Title    | ERP-DBaaS Release Notes        |
| Product           | ERP-DBaaS                      |
| Current Version   | 1.0.0                         |
| Release Date      | 2026-02-24                     |
| Classification    | Internal                       |

---

## Version 1.0.0 - General Availability

**Release Date**: February 24, 2026
**Release Type**: Major Release (GA)
**Upgrade Path**: Fresh installation (no prior version)

---

### Release Highlights

ERP-DBaaS v1.0.0 marks the General Availability release of the Database-as-a-Service platform for the ERP ecosystem. This release delivers a complete, production-ready database provisioning and lifecycle management platform supporting eight database engines with full AIDD governance, multi-tenant isolation, and enterprise-grade backup and restore capabilities.

---

### New Features

#### 1. Multi-Engine Database Provisioning

**Feature ID**: DBAAS-001
**Priority**: P0

The core provisioning engine supports eight database technologies, each managed through a dedicated Kubernetes operator:

| Engine       | Supported Versions | Deployment Modes         | HA Support |
|--------------|-------------------|--------------------------|------------|
| YugabyteDB   | 2.18, 2.20       | Single-node, RF3, RF5    | Yes        |
| ScyllaDB     | 5.2, 5.4         | Single-node, 3-node, 5-node | Yes     |
| DragonflyDB  | 1.13, 1.15       | Single-node, HA pair     | Yes        |
| MongoDB      | 6.0, 7.0         | Standalone, ReplicaSet   | Yes        |
| CouchDB      | 3.3               | Single-node, Clustered   | Yes        |
| ClickHouse   | 23.8, 24.1       | Single-node, Clustered   | Yes        |
| TimescaleDB  | 2.13, 2.14       | Single-node, HA pair     | Yes        |
| QuestDB      | 7.3, 7.4         | Single-node              | Roadmap    |

Each engine includes pre-configured size presets:

- **Small**: 1 vCPU, 2 GB RAM, 20 GB storage
- **Medium**: 2 vCPU, 4 GB RAM, 50 GB storage
- **Large**: 4 vCPU, 8 GB RAM, 100 GB storage
- **X-Large**: 8 vCPU, 16 GB RAM, 250 GB storage
- **Custom**: User-defined resource allocations (subject to quota)

**Usage Notes**:
- Instance provisioning initiates through the UI wizard, API, or GraphQL mutation
- Each instance receives a unique identifier in the format `{tenant}-{engine}-{random}`
- Connection strings are automatically generated and stored in Kubernetes Secrets
- Provisioning time varies by engine (15 seconds for DragonflyDB to 5 minutes for large YugabyteDB clusters)

---

#### 2. AIDD Governance (Strict Profile)

**Feature ID**: DBAAS-002
**Priority**: P0

AI-Driven Database Design (AIDD) governance enforces organizational policies on all database instances. The v1.0.0 release includes the Strict profile for production environments.

**Strict Profile Rules**:

| Rule ID   | Rule                                    | Enforcement |
|-----------|-----------------------------------------|-------------|
| AIDD-S01  | Minimum 3-node deployment for HA engines | Hard block  |
| AIDD-S02  | Backup schedule required (minimum daily)  | Hard block  |
| AIDD-S03  | Backup retention >= 30 days              | Hard block  |
| AIDD-S04  | Encryption at rest enabled               | Hard block  |
| AIDD-S05  | Encryption in transit (TLS) enabled      | Hard block  |
| AIDD-S06  | Resource requests within quota           | Hard block  |
| AIDD-S07  | Storage class must be SSD-backed         | Hard block  |
| AIDD-S08  | Network policy isolation enabled         | Hard block  |
| AIDD-S09  | Credential rotation <= 90 days           | Warning     |
| AIDD-S10  | Monitoring endpoints configured          | Warning     |

**Flexible Profile Rules** (for dev/staging):

| Rule ID   | Rule                                    | Enforcement |
|-----------|-----------------------------------------|-------------|
| AIDD-F01  | Single-node deployments permitted        | Allowed     |
| AIDD-F02  | Backup schedule recommended              | Soft warning|
| AIDD-F03  | Encryption at rest recommended           | Soft warning|
| AIDD-F04  | Resource requests within soft quota      | Soft warning|
| AIDD-F05  | Monitoring endpoints recommended         | Soft warning|

**API Response Example (Policy Violation)**:
```json
{
  "status": 422,
  "error": "POLICY_VIOLATION",
  "violations": [
    {
      "rule": "AIDD-S01",
      "message": "Production instances require minimum 3-node deployment for HA",
      "field": "spec.replicas",
      "current": 1,
      "required": ">=3"
    },
    {
      "rule": "AIDD-S02",
      "message": "Backup schedule is required for production instances",
      "field": "spec.backup.schedule",
      "current": null,
      "required": "non-null cron expression"
    }
  ]
}
```

---

#### 3. Backup and Restore

**Feature ID**: DBAAS-003
**Priority**: P0

Full backup and restore lifecycle management with RustFS (S3-compatible) storage backend.

**Backup Features**:
- Scheduled backups via cron expressions (e.g., `0 2 * * *` for daily at 2 AM UTC)
- On-demand backup initiation through UI or API
- Incremental backups for supported engines (YugabyteDB, MongoDB, TimescaleDB)
- Full snapshot backups for all engines
- Backup encryption with tenant-specific AES-256 keys
- Configurable retention policies (7 to 365 days)
- Backup size estimation before execution
- Parallel backup streams for large databases

**Restore Features**:
- Full restore to new instance (non-destructive)
- Point-in-time recovery (PITR) for engines that support WAL archiving
- Cross-region restore capability (when multi-region is configured)
- Restore progress tracking with percentage completion
- Automatic credential regeneration on restore
- Restore validation checksums

**Backup CRD (BackupPolicy)**:
```yaml
apiVersion: dbaas.erp.io/v1alpha1
kind: BackupPolicy
metadata:
  name: daily-production-backup
  namespace: tenant-acme
spec:
  instanceRef:
    name: acme-yugabyte-prod-01
  schedule: "0 2 * * *"
  retentionDays: 90
  type: incremental
  encryption:
    enabled: true
    keyRef:
      name: backup-encryption-key
  storage:
    endpoint: rustfs.internal:9000
    bucket: dbaas-backups
    prefix: acme/yugabyte/prod-01
```

**Restore CRD (RestoreJob)**:
```yaml
apiVersion: dbaas.erp.io/v1alpha1
kind: RestoreJob
metadata:
  name: restore-acme-yugabyte-20260224
  namespace: tenant-acme
spec:
  backupRef:
    name: acme-yugabyte-prod-01-20260224-0200
  targetInstance:
    name: acme-yugabyte-prod-01-restored
    createNew: true
  validation:
    checksumVerification: true
    postRestoreCheck: true
```

---

#### 4. Plugin Registry and System

**Feature ID**: DBAAS-004
**Priority**: P1

Extensible plugin system enabling custom integrations through a gRPC-based plugin interface.

**Built-in Plugins (v1.0.0)**:

| Plugin Name          | Type          | Description                                    |
|----------------------|---------------|------------------------------------------------|
| prometheus-exporter  | Monitoring    | Exports database metrics to Prometheus          |
| slack-notifier       | Notification  | Sends instance lifecycle events to Slack        |
| pagerduty-alerter    | Alerting      | Triggers PagerDuty incidents on critical events |
| audit-logger         | Compliance    | Extended audit logging to external SIEM         |
| cost-tagger          | Metering      | Applies cost allocation tags to instances       |

**Plugin Registration CRD**:
```yaml
apiVersion: dbaas.erp.io/v1alpha1
kind: PluginRegistration
metadata:
  name: custom-backup-validator
  namespace: dbaas-system
spec:
  name: custom-backup-validator
  version: "1.0.0"
  type: validation
  grpc:
    image: registry.internal/plugins/backup-validator:1.0.0
    port: 50051
  hooks:
    - preBackup
    - postRestore
  healthCheck:
    endpoint: /healthz
    intervalSeconds: 30
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 500m
      memory: 256Mi
```

**Plugin Lifecycle**:
1. Registration via CRD or UI
2. Image pull and validation
3. Health check verification
4. Hook registration with operator
5. Active monitoring and auto-restart on failure

---

#### 5. Refine.dev Frontend Application

**Feature ID**: DBAAS-005
**Priority**: P0

Full-featured admin UI built with React, Refine.dev, and Ant Design with the blue theme (#2563eb).

**UI Features**:
- **Dashboard**: Real-time overview of all instances, health status, quota usage, and recent activity
- **Instance Management**: List, filter, sort, and search across all database instances with bulk operations
- **Provisioning Wizard**: 4-step guided wizard (Engine Selection -> Configuration -> Policy Review -> Confirmation)
- **Instance Detail**: Comprehensive view with metrics, connection info, backup history, and scaling controls
- **Backup Management**: Schedule configuration, backup history, restore initiation, and retention management
- **Engine Catalog**: Browse supported engines with version info, capabilities, and recommendations
- **Plugin Registry**: View, register, and manage plugins with status monitoring
- **Credential Management**: View connection strings, rotate credentials, and manage access
- **Quota Dashboard**: Tenant quota usage, limit management, and capacity planning
- **Administration**: Tenant management, policy profile editor, and system configuration

**Technical Details**:
- Responsive design supporting desktop (1920px), laptop (1440px), tablet (768px), and mobile (375px)
- Real-time updates via GraphQL subscriptions (graphql-ws)
- Optimistic UI updates for improved perceived performance
- Accessible design following WCAG 2.1 AA guidelines
- Dark mode support (toggle in user preferences)

---

#### 6. Multi-Tenant Quota Management

**Feature ID**: DBAAS-006
**Priority**: P0

Comprehensive quota system ensuring fair resource allocation across tenants.

**Quota Dimensions**:

| Dimension            | Free Tier  | Standard Tier | Enterprise Tier |
|----------------------|-----------|---------------|-----------------|
| Max instances        | 5         | 25            | Unlimited       |
| Total vCPU           | 8         | 64            | Custom          |
| Total RAM (GB)       | 16        | 128           | Custom          |
| Total storage (GB)   | 100       | 1,000         | Custom          |
| Backup storage (GB)  | 50        | 500           | Custom          |
| API rate (req/min)   | 100       | 500           | 2,000           |
| Max plugins          | 2         | 10            | Unlimited       |

**Quota Enforcement**:
- Real-time quota checking on all provisioning and scaling requests
- Soft limits generate warnings; hard limits block operations
- Quota usage visible in dashboard with trend graphs
- Automated alerts at 80% and 95% utilization thresholds
- Monthly quota usage reports per tenant

---

### API Changes

#### New REST Endpoints

| Method | Endpoint                              | Description                    |
|--------|---------------------------------------|--------------------------------|
| POST   | `/api/v1/instances`                   | Create new database instance   |
| GET    | `/api/v1/instances`                   | List instances (with filters)  |
| GET    | `/api/v1/instances/:id`               | Get instance details           |
| PUT    | `/api/v1/instances/:id`               | Update instance configuration  |
| DELETE | `/api/v1/instances/:id`               | Decommission instance          |
| POST   | `/api/v1/instances/:id/scale`         | Scale instance resources       |
| POST   | `/api/v1/instances/:id/backup`        | Trigger on-demand backup       |
| GET    | `/api/v1/instances/:id/backups`       | List instance backups          |
| POST   | `/api/v1/instances/:id/restore`       | Initiate restore operation     |
| POST   | `/api/v1/instances/:id/credentials/rotate` | Rotate credentials       |
| GET    | `/api/v1/engines`                     | List supported engines         |
| GET    | `/api/v1/engines/:engine`             | Get engine details             |
| GET    | `/api/v1/quotas`                      | Get tenant quota usage         |
| GET    | `/api/v1/plugins`                     | List registered plugins        |
| POST   | `/api/v1/plugins`                     | Register new plugin            |
| GET    | `/api/v1/policies`                    | List policy profiles           |
| POST   | `/api/v1/policies/evaluate`           | Evaluate configuration against policy |
| GET    | `/api/v1/metering/usage`              | Get metering usage data        |

#### New GraphQL Operations

```graphql
# Queries
query GetInstances($tenantId: String!, $filters: InstanceFilter) {
  instances(tenant_id: $tenantId, where: $filters) {
    id
    name
    engine
    version
    status
    size_preset
    created_at
    connection_endpoint
  }
}

# Mutations
mutation CreateInstance($input: CreateInstanceInput!) {
  createInstance(input: $input) {
    id
    name
    status
  }
}

# Subscriptions
subscription InstanceStatusChanged($instanceId: String!) {
  instance_by_pk(id: $instanceId) {
    status
    health_status
    last_health_check
  }
}
```

---

### Infrastructure Changes

#### New Custom Resource Definitions (CRDs)

| CRD Name          | API Group          | Description                        |
|-------------------|--------------------|------------------------------------|
| ServiceInstance    | dbaas.erp.io/v1alpha1 | Database instance lifecycle      |
| PolicyProfile     | dbaas.erp.io/v1alpha1 | AIDD governance policy definition|
| BackupPolicy      | dbaas.erp.io/v1alpha1 | Automated backup configuration   |
| RestoreJob        | dbaas.erp.io/v1alpha1 | Restore operation specification  |
| PluginRegistration| dbaas.erp.io/v1alpha1 | Plugin lifecycle management      |
| TenantDataPlane   | dbaas.erp.io/v1alpha1 | Tenant namespace and quota config|

#### New Docker Images

| Image                                  | Size    | Base Image       |
|----------------------------------------|---------|------------------|
| erp-dbaas/gateway:1.0.0               | 18 MB   | scratch          |
| erp-dbaas/api:1.0.0                   | 145 MB  | node:20-alpine   |
| erp-dbaas/frontend:1.0.0              | 25 MB   | nginx:alpine     |
| erp-dbaas/operator-controller:1.0.0   | 35 MB   | scratch          |
| erp-dbaas/yugabyte-operator:1.0.0     | 32 MB   | scratch          |
| erp-dbaas/scylla-operator:1.0.0       | 30 MB   | scratch          |
| erp-dbaas/dragonfly-operator:1.0.0    | 28 MB   | scratch          |
| erp-dbaas/mongodb-operator:1.0.0      | 31 MB   | scratch          |
| erp-dbaas/couchdb-operator:1.0.0      | 29 MB   | scratch          |
| erp-dbaas/clickhouse-operator:1.0.0   | 30 MB   | scratch          |
| erp-dbaas/timescale-operator:1.0.0    | 29 MB   | scratch          |
| erp-dbaas/questdb-operator:1.0.0      | 28 MB   | scratch          |

---

### Known Issues

| ID         | Severity | Description                                          | Workaround                        |
|------------|----------|------------------------------------------------------|-----------------------------------|
| DBAAS-KI01 | Medium   | QuestDB HA mode not yet available                    | Use single-node; manual replication |
| DBAAS-KI02 | Low      | Backup size estimation may be 10-15% off for MongoDB | Verify actual backup size post-completion |
| DBAAS-KI03 | Low      | Plugin logs not streamed in real-time in UI          | Check pod logs via kubectl         |
| DBAAS-KI04 | Medium   | Cross-region restore requires manual network config  | Contact platform team for cross-region restores |
| DBAAS-KI05 | Low      | Dark mode has minor styling issues on quota dashboard | Use light mode for quota views    |

---

### Upgrade Notes

This is the initial GA release. No upgrade path is required.

**Fresh Installation Requirements**:
- Kubernetes 1.28+
- Helm 3.13+
- cert-manager 1.13+
- CSI driver with dynamic provisioning
- Network policy support (Calico or Cilium recommended)
- Minimum 6 worker nodes (8 vCPU, 32 GB RAM each)

**Installation**:
```bash
# Add Helm repository
helm repo add erp-dbaas https://charts.erp.internal/dbaas
helm repo update

# Install CRDs
kubectl apply -f https://charts.erp.internal/dbaas/crds/v1.0.0.yaml

# Install platform
helm install erp-dbaas erp-dbaas/platform \
  --namespace dbaas-system \
  --create-namespace \
  --values values-production.yaml \
  --version 1.0.0
```

---

### Deprecations

None (initial release).

---

### Contributors

- Platform Engineering Team
- Database Reliability Team
- Frontend Engineering Team
- Security and Compliance Team

---

*For support, contact the Platform Engineering team via #dbaas-support on Slack or file a ticket in Jira under the DBAAS project.*
