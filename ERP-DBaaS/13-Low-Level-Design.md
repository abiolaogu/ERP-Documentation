# ERP-DBaaS Low-Level Design

## Document Control

| Field             | Value                              |
|-------------------|------------------------------------|
| Document Title    | ERP-DBaaS Low-Level Design         |
| Version           | 1.0.0                             |
| Date              | 2026-02-24                         |
| Classification    | Internal - Engineering             |
| Author            | Platform Engineering Team          |

---

## Table of Contents

1. [CRD Schema Specifications](#1-crd-schema-specifications)
2. [Operator Reconciliation Loops](#2-operator-reconciliation-loops)
3. [Connection Pooling Strategy](#3-connection-pooling-strategy)
4. [Backup Encryption Implementation](#4-backup-encryption-implementation)
5. [Metering and Billing Data Collection](#5-metering-and-billing-data-collection)
6. [Health Check Implementation per Engine](#6-health-check-implementation-per-engine)

---

## 1. CRD Schema Specifications

### 1.1 Base CRD Structure (Common to All Engines)

All engine CRDs inherit from a common base specification. The following YAML defines the shared fields.

```yaml
apiVersion: dbaas.erp.io/v1alpha1
kind: <EngineInstance>  # e.g., YugabyteDBInstance, DragonflyDBInstance
metadata:
  name: <instance-name>
  namespace: <tenant-namespace>
  labels:
    dbaas.erp.io/tenant-id: <tenant-id>
    dbaas.erp.io/engine: <engine-name>
    dbaas.erp.io/tier: <tier-a|tier-b|tier-c>
spec:
  engine: <engine-name>
  version: <engine-version>
  size: <S|M|L|XL>
  replicas: <integer>
  storage:
    size: <storage-size>       # e.g., "50Gi"
    storageClass: <class-name> # e.g., "ssd-retain"
  backup:
    enabled: <boolean>
    schedule: <cron-expression>
    retentionDays: <integer>
    encryption:
      enabled: <boolean>
      algorithm: "AES-256-GCM"
      keyRef: <secret-name>
  highAvailability:
    enabled: <boolean>
    replicationMode: <sync|async>
  resources:
    requests:
      cpu: <cpu-request>
      memory: <memory-request>
    limits:
      cpu: <cpu-limit>
      memory: <memory-limit>
status:
  phase: <Pending|Provisioning|Running|Scaling|Failed|Deleting>
  conditions:
    - type: Ready
      status: "True|False"
      lastTransitionTime: <timestamp>
      reason: <reason-string>
      message: <human-readable-message>
  connectionInfo:
    host: <service-hostname>
    port: <port-number>
    secretRef: <credentials-secret-name>
  metrics:
    cpuUsage: <percentage>
    memoryUsage: <percentage>
    storageUsage: <percentage>
    connectionCount: <integer>
```

### 1.2 YugabyteDB CRD Extensions

```yaml
spec:
  yugabytedb:
    replicationFactor: 3
    masterCount: 3
    tserverCount: 3
    masterResources:
      requests:
        cpu: "2"
        memory: "4Gi"
    tserverResources:
      requests:
        cpu: "4"
        memory: "8Gi"
    ysqlConfig:
      maxConnections: 300
      sharedBuffers: "2GB"
      walLevel: "replica"
    tablespaces: []
    plugins:
      - name: "pg_trgm"
        version: "1.6"
```

### 1.3 DragonflyDB CRD Extensions

```yaml
spec:
  dragonflydb:
    maxMemoryPolicy: "allkeys-lru"
    snapshotInterval: "1h"
    aclRules:
      - user: "default"
        permissions: "+@all ~*"
    tlsEnabled: true
    threadCount: 4
```

### 1.4 ClickHouse CRD Extensions

```yaml
spec:
  clickhouse:
    shardCount: 2
    replicaCount: 2
    zookeeperRef: "zk-cluster"
    mergeTreeSettings:
      maxPartSize: "10Gi"
      mergeMaxBlockSize: 8192
    users:
      - name: "reader"
        profile: "readonly"
      - name: "writer"
        profile: "default"
    dictionaries: []
```

### 1.5 Additional Engine CRDs

Each of the remaining five engines (Tembo, SurrealDB, QuestDB, Apache Doris, InfluxDB) follows the base structure with engine-specific nested fields for configuration tuning, replication settings, and plugin management.

---

## 2. Operator Reconciliation Loops

### 2.1 Reconciliation State Machine

Each operator runs a reconciliation loop that evaluates the current state against the desired state defined in the CRD.

```
Reconcile() called by controller-runtime
    │
    ▼
┌────────────────────────┐
│ Fetch CRD from cache   │
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐     ┌─────────────────┐
│ CRD exists?            │─No─>│ Return (deleted) │
└──────────┬─────────────┘     └─────────────────┘
           │ Yes
           ▼
┌────────────────────────┐
│ Check finalizer        │
│ (add if missing)       │
└──────────┬─────────────┘
           │
           ▼
┌────────────────────────┐
│ Switch on status.phase │
└──────────┬─────────────┘
           │
     ┌─────┼──────┬──────────┬──────────┐
     ▼     ▼      ▼          ▼          ▼
  Pending Prov.  Running   Scaling   Deleting
     │     │      │          │          │
     ▼     ▼      ▼          ▼          ▼
  Create  Wait   Health   Resize    Cleanup
  Rsrcs   Ready  Check    Rsrcs     Resources
     │     │      │          │          │
     ▼     ▼      ▼          ▼          ▼
  Set     Set    Requeue  Verify    Remove
  Prov.   Run    (30s)    Health    Finalizer
```

### 2.2 Reconciliation Pseudocode

```go
func (r *Reconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    instance := &v1alpha1.EngineInstance{}
    if err := r.Get(ctx, req.NamespacedName, instance); err != nil {
        if apierrors.IsNotFound(err) {
            return ctrl.Result{}, nil // CRD deleted
        }
        return ctrl.Result{}, err
    }

    // Add finalizer if not present
    if !controllerutil.ContainsFinalizer(instance, finalizerName) {
        controllerutil.AddFinalizer(instance, finalizerName)
        return ctrl.Result{}, r.Update(ctx, instance)
    }

    // Handle deletion
    if !instance.DeletionTimestamp.IsZero() {
        return r.handleDeletion(ctx, instance)
    }

    switch instance.Status.Phase {
    case "":
        return r.handlePending(ctx, instance)
    case v1alpha1.PhasePending:
        return r.handlePending(ctx, instance)
    case v1alpha1.PhaseProvisioning:
        return r.handleProvisioning(ctx, instance)
    case v1alpha1.PhaseRunning:
        return r.handleRunning(ctx, instance)
    case v1alpha1.PhaseScaling:
        return r.handleScaling(ctx, instance)
    case v1alpha1.PhaseFailed:
        return r.handleFailed(ctx, instance)
    }

    return ctrl.Result{RequeueAfter: 30 * time.Second}, nil
}
```

### 2.3 Requeue Strategy

| Phase         | Requeue After | Reason                                    |
|---------------|---------------|-------------------------------------------|
| Pending       | 5s            | Wait for resource creation                |
| Provisioning  | 10s           | Wait for pods to become ready             |
| Running       | 30s           | Periodic health check                     |
| Scaling       | 10s           | Wait for scaling to complete              |
| Failed        | 60s           | Retry with backoff                        |
| Deleting      | 5s            | Wait for resource cleanup                 |

---

## 3. Connection Pooling Strategy

### 3.1 Architecture

Connection pooling is implemented at two levels to optimize resource utilization and prevent connection exhaustion.

```
Application Pods ──[connections]──> PgBouncer / ProxySQL (L1 Pool)
                                          │
                                          ▼
                                   Database Instance (L2 Engine Pool)
```

### 3.2 Pool Configuration per Engine

| Engine      | Proxy          | Pool Mode       | Default Max Connections | Per-Tenant Limit |
|-------------|----------------|-----------------|------------------------|------------------|
| YugabyteDB  | PgBouncer      | Transaction     | 300                    | 50               |
| Tembo (PG)  | PgBouncer      | Transaction     | 200                    | 40               |
| ClickHouse  | chproxy        | Session         | 100                    | 20               |
| DragonflyDB | Direct (built-in)| N/A           | 10000                  | 1000             |
| SurrealDB   | Direct         | N/A             | 500                    | 100              |
| QuestDB     | Direct         | N/A             | 200                    | 40               |
| Apache Doris| Direct         | N/A             | 200                    | 40               |
| InfluxDB    | Direct         | N/A             | 500                    | 100              |

### 3.3 PgBouncer Sidecar Configuration

```ini
[databases]
* = host=127.0.0.1 port=5433

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = md5
pool_mode = transaction
default_pool_size = 20
max_client_conn = 300
max_db_connections = 50
server_idle_timeout = 300
client_idle_timeout = 60
query_wait_timeout = 120
```

---

## 4. Backup Encryption Implementation

### 4.1 Encryption Pipeline

```
Raw Dump ──> zstd Compress ──> AES-256-GCM Encrypt ──> RustFS Upload
```

### 4.2 Key Management

```
┌──────────────────────────────────────────────────┐
│ Key Hierarchy                                     │
│                                                   │
│ Root Key (HSM-backed)                             │
│   └── Tenant Master Key (per tenant)              │
│         └── Data Encryption Key (per backup)      │
│               - Generated per backup operation    │
│               - Wrapped by Tenant Master Key      │
│               - Stored alongside backup metadata  │
└──────────────────────────────────────────────────┘
```

### 4.3 Encryption Implementation (Go)

```go
func encryptBackup(plaintext []byte, tenantKey []byte) ([]byte, []byte, error) {
    // Generate a per-backup DEK
    dek := make([]byte, 32) // 256-bit key
    if _, err := crypto_rand.Read(dek); err != nil {
        return nil, nil, fmt.Errorf("failed to generate DEK: %w", err)
    }

    // Create AES cipher
    block, err := aes.NewCipher(dek)
    if err != nil {
        return nil, nil, fmt.Errorf("failed to create cipher: %w", err)
    }

    // Create GCM
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, nil, fmt.Errorf("failed to create GCM: %w", err)
    }

    // Generate nonce
    nonce := make([]byte, gcm.NonceSize())
    if _, err := crypto_rand.Read(nonce); err != nil {
        return nil, nil, fmt.Errorf("failed to generate nonce: %w", err)
    }

    // Encrypt
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)

    // Wrap DEK with tenant master key
    wrappedDEK, err := wrapKey(dek, tenantKey)
    if err != nil {
        return nil, nil, fmt.Errorf("failed to wrap DEK: %w", err)
    }

    return ciphertext, wrappedDEK, nil
}
```

### 4.4 Backup Metadata Schema

```sql
CREATE TABLE backup_catalog (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       TEXT NOT NULL,
    instance_id     UUID NOT NULL REFERENCES instances(id),
    engine          TEXT NOT NULL,
    backup_type     TEXT NOT NULL CHECK (backup_type IN ('full', 'incremental', 'wal')),
    status          TEXT NOT NULL DEFAULT 'in_progress',
    storage_path    TEXT NOT NULL,
    size_bytes      BIGINT,
    checksum_sha256 TEXT,
    encryption_key_id TEXT NOT NULL,
    compression     TEXT NOT NULL DEFAULT 'zstd',
    started_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at    TIMESTAMPTZ,
    expires_at      TIMESTAMPTZ NOT NULL,
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 5. Metering and Billing Data Collection

### 5.1 Metering Architecture

```
Engine Pods ──[metrics exporter]──> VictoriaMetrics
                                          │
                                     ┌────┴────┐
                                     ▼         ▼
                              Metering       Alerting
                              Collector      Rules
                                  │
                                  ▼
                           YugabyteDB
                           (metering_records)
                                  │
                                  ▼
                           Billing Export
                           (hourly batch)
```

### 5.2 Metering Record Schema

```sql
CREATE TABLE metering_records (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       TEXT NOT NULL,
    instance_id     UUID NOT NULL,
    engine          TEXT NOT NULL,
    period_start    TIMESTAMPTZ NOT NULL,
    period_end      TIMESTAMPTZ NOT NULL,
    cpu_core_hours  DECIMAL(10,4) NOT NULL DEFAULT 0,
    memory_gb_hours DECIMAL(10,4) NOT NULL DEFAULT 0,
    storage_gb_hours DECIMAL(10,4) NOT NULL DEFAULT 0,
    network_gb      DECIMAL(10,4) NOT NULL DEFAULT 0,
    backup_gb       DECIMAL(10,4) NOT NULL DEFAULT 0,
    iops_thousands  DECIMAL(10,4) NOT NULL DEFAULT 0,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE(instance_id, period_start)
);

CREATE INDEX idx_metering_tenant_period ON metering_records(tenant_id, period_start);
```

### 5.3 Collection Intervals

| Metric             | Collection Interval | Aggregation Window | Retention   |
|--------------------|--------------------|--------------------|-------------|
| CPU usage          | 15s                | 1 hour             | 13 months   |
| Memory usage       | 15s                | 1 hour             | 13 months   |
| Storage usage      | 5m                 | 1 hour             | 13 months   |
| Network I/O        | 30s                | 1 hour             | 13 months   |
| Backup storage     | 1h                 | 1 day              | 13 months   |
| IOPS               | 15s                | 1 hour             | 13 months   |
| Connection count   | 30s                | 1 hour             | 90 days     |

---

## 6. Health Check Implementation per Engine

### 6.1 Health Check Framework

Each engine operator implements a `HealthChecker` interface that performs engine-specific health validation.

```go
type HealthChecker interface {
    CheckLiveness(ctx context.Context, instance *v1alpha1.EngineInstance) (HealthResult, error)
    CheckReadiness(ctx context.Context, instance *v1alpha1.EngineInstance) (HealthResult, error)
    CheckReplication(ctx context.Context, instance *v1alpha1.EngineInstance) (ReplicationStatus, error)
}

type HealthResult struct {
    Healthy     bool
    Latency     time.Duration
    Message     string
    CheckedAt   time.Time
}
```

### 6.2 Engine-Specific Health Checks

| Engine      | Liveness Check                 | Readiness Check                     | Replication Check                |
|-------------|--------------------------------|--------------------------------------|----------------------------------|
| YugabyteDB  | `SELECT 1` on YSQL port       | Tablet server registered + tablets balanced | `yb-admin list_all_masters` lag |
| DragonflyDB | `PING` command                 | `INFO replication` connected_slaves  | Replication offset delta         |
| ClickHouse  | `SELECT 1` on HTTP port       | `system.replicas` queue size = 0     | `system.replication_queue` check |
| Tembo (PG)  | `SELECT 1`                    | `pg_is_in_recovery()` check          | `pg_stat_replication` lag bytes  |
| SurrealDB   | HTTP `/health` endpoint       | HTTP `/status` ready flag            | N/A (TiKV handles replication)   |
| QuestDB     | HTTP `/exec?query=SELECT 1`   | `/status` endpoint                   | N/A (single-node default)        |
| Apache Doris| FE HTTP `/api/health`         | BE heartbeat via FE API              | FE follower sync status          |
| InfluxDB    | HTTP `/health`                | HTTP `/ready`                        | Anti-entropy repair status       |

### 6.3 Health Check Thresholds

```yaml
healthCheck:
  liveness:
    initialDelay: 30s
    interval: 10s
    timeout: 5s
    failureThreshold: 3      # Mark unhealthy after 3 consecutive failures
  readiness:
    initialDelay: 60s
    interval: 15s
    timeout: 10s
    failureThreshold: 3
  replication:
    maxLagSeconds: 30        # Alert threshold
    criticalLagSeconds: 120  # Failover threshold
```

---

## Appendix: Error Codes

| Code     | Description                          | Retry | Resolution                          |
|----------|--------------------------------------|-------|--------------------------------------|
| DBE-001  | CRD validation failed                | No    | Fix CRD spec fields                 |
| DBE-002  | Insufficient cluster resources       | Yes   | Wait for capacity or resize cluster  |
| DBE-003  | PVC creation failed                  | Yes   | Check StorageClass availability      |
| DBE-004  | Pod scheduling failed                | Yes   | Check node affinity and taints       |
| DBE-005  | Health check timeout                 | Yes   | Investigate engine logs              |
| DBE-006  | Credential generation failed         | Yes   | Check secrets manager connectivity   |
| DBE-007  | Backup encryption failed             | No    | Verify encryption key availability   |
| DBE-008  | Replication setup failed             | Yes   | Check network connectivity           |
| DBE-009  | Plugin installation failed           | No    | Check version compatibility          |
| DBE-010  | Quota enforcement error              | No    | Upgrade tenant tier or free resources|
