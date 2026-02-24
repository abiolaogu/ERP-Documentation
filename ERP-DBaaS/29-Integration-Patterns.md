# ERP-DBaaS Integration Patterns

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Integration Patterns         |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Engineering                 |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [How ERP Modules Request Database Instances](#1-how-erp-modules-request-database-instances)
2. [Hasura Remote Schema Integration](#2-hasura-remote-schema-integration)
3. [Service Discovery for Database Endpoints](#3-service-discovery-for-database-endpoints)
4. [Connection String Injection via K8s Secrets](#4-connection-string-injection-via-k8s-secrets)
5. [Observability Integration](#5-observability-integration)
6. [Event Notifications via Apache Pulsar](#6-event-notifications-via-apache-pulsar)

---

## 1. How ERP Modules Request Database Instances

### 1.1 Request Flow

ERP modules interact with the DBaaS platform through a standardized API to provision and manage their database instances. The flow ensures tenant isolation and quota enforcement at every step.

```
ERP Module (e.g., Finance)
    │
    │ POST /api/v1/instances
    │ Authorization: Bearer <JWT>
    ▼
┌──────────────────┐
│ Go Gateway        │ ── JWT validation
│ (Port 8090)       │ ── Rate limit check
└────────┬─────────┘ ── Route to dbaas-api
         │
         ▼
┌──────────────────┐
│ Node.js dbaas-api │ ── Extract tenant_id from JWT
│ (Port 3000)       │ ── Validate request schema
└────────┬─────────┘ ── Check quota
         │
         ▼
┌──────────────────┐
│ K8s API Server    │ ── Create CRD in tenant namespace
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Engine Operator   │ ── Reconcile CRD to running instance
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Instance Ready    │ ── Credentials in K8s Secret
│ Event Published   │ ── Pulsar event emitted
└──────────────────┘
```

### 1.2 Provisioning API Request

```bash
curl -X POST https://dbaas.erp.io/api/v1/instances \
  -H "Authorization: Bearer ${JWT_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "finance-ledger-db",
    "engine": "yugabytedb",
    "version": "2.20",
    "size": "M",
    "highAvailability": true,
    "backup": {
      "enabled": true,
      "schedule": "0 2 * * *",
      "retentionDays": 30,
      "encryption": true
    },
    "tags": {
      "module": "erp-finance",
      "environment": "production"
    }
  }'
```

### 1.3 Provisioning API Response

```json
{
  "id": "inst-a1b2c3d4",
  "name": "finance-ledger-db",
  "engine": "yugabytedb",
  "version": "2.20.1",
  "size": "M",
  "status": "provisioning",
  "namespace": "dbaas-acme-corp",
  "connectionInfo": {
    "host": "finance-ledger-db.dbaas-acme-corp.svc.cluster.local",
    "port": 5433,
    "secretRef": "finance-ledger-db-credentials"
  },
  "createdAt": "2026-02-24T10:30:00Z",
  "estimatedReadyAt": "2026-02-24T10:32:00Z"
}
```

### 1.4 ERP Module SDK (Optional)

For ERP modules that prefer a programmatic interface, a lightweight SDK is available.

```typescript
import { DBaaSClient } from '@erp/dbaas-sdk';

const client = new DBaaSClient({
  endpoint: 'https://dbaas.erp.io',
  token: process.env.DBAAS_TOKEN,
});

// Provision a new instance
const instance = await client.instances.create({
  name: 'finance-ledger-db',
  engine: 'yugabytedb',
  size: 'M',
  highAvailability: true,
});

// Wait for provisioning to complete
await client.instances.waitForReady(instance.id, { timeout: 300_000 });

// Get connection info
const connInfo = await client.instances.getConnectionInfo(instance.id);
console.log(`Connection string: ${connInfo.connectionString}`);
```

---

## 2. Hasura Remote Schema Integration

### 2.1 Federation Architecture

ERP-DBaaS exposes its GraphQL API as a Hasura Remote Schema, allowing other ERP modules to query database instance metadata through the federated Hasura Gateway.

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│ ERP-Finance      │     │ ERP-HRM          │     │ ERP-Inventory    │
│ Hasura Instance  │     │ Hasura Instance   │     │ Hasura Instance  │
└────────┬─────────┘     └────────┬─────────┘     └────────┬─────────┘
         │                        │                         │
         └────────────────────────┴─────────────────────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ Hasura Federation │
                         │ Gateway           │
                         └────────┬─────────┘
                                  │ Remote Schema
                                  ▼
                         ┌──────────────────┐
                         │ DBaaS Hasura     │
                         │ Instance         │
                         └──────────────────┘
```

### 2.2 Remote Schema Configuration

```json
{
  "type": "add_remote_schema",
  "args": {
    "name": "dbaas",
    "definition": {
      "url": "http://dbaas-hasura.dbaas-system.svc.cluster.local:8080/v1/graphql",
      "headers": [
        {
          "name": "x-hasura-admin-secret",
          "value_from_env": "DBAAS_HASURA_ADMIN_SECRET"
        }
      ],
      "forward_client_headers": true,
      "timeout_seconds": 30
    }
  }
}
```

### 2.3 Available Federated Queries

```graphql
# Query available from any ERP module through the federation gateway
query GetMyDatabaseInstances {
  dbaas_instances(
    where: { status: { _eq: "running" } }
    order_by: { created_at: desc }
  ) {
    id
    name
    engine
    version
    size
    status
    connection_info {
      host
      port
    }
    backups_aggregate {
      aggregate {
        count
      }
    }
    created_at
  }
}

# Subscribe to instance status changes (real-time)
subscription InstanceStatusUpdates {
  dbaas_instances(
    where: { status: { _nin: ["running", "deleted"] } }
  ) {
    id
    name
    status
    updated_at
  }
}
```

### 2.4 Permission Model

| Role              | Access Level                                          |
|-------------------|-------------------------------------------------------|
| `tenant_admin`    | Full CRUD on own tenant instances                     |
| `tenant_user`     | Read-only on own tenant instances                     |
| `platform_admin`  | Full CRUD on all instances across all tenants         |
| `service_account` | Read-only on own tenant, used by ERP module backends  |

---

## 3. Service Discovery for Database Endpoints

### 3.1 Discovery Mechanisms

ERP modules can discover database endpoints through multiple mechanisms.

| Mechanism              | Use Case                               | Latency   | Freshness |
|------------------------|----------------------------------------|-----------|-----------|
| K8s Service DNS        | In-cluster service-to-service calls     | 0ms       | Real-time |
| K8s Secret reference   | Connection string from mounted secret   | 0ms       | Real-time |
| GraphQL query          | Dynamic discovery from any context      | 10-50ms   | Real-time |
| Pulsar event           | Event-driven discovery on provisioning  | < 1s      | Event-time|
| REST API               | Programmatic lookup                     | 10-50ms   | Real-time |

### 3.2 Kubernetes DNS-Based Discovery

Every database instance provisioned by DBaaS creates a Kubernetes Service with a predictable DNS name.

```
Format: {instance-name}.{tenant-namespace}.svc.cluster.local

Examples:
  finance-ledger-db.dbaas-acme-corp.svc.cluster.local:5433     (YugabyteDB)
  cache-session.dbaas-acme-corp.svc.cluster.local:6379          (DragonflyDB)
  analytics-warehouse.dbaas-acme-corp.svc.cluster.local:9000    (ClickHouse)
  timeseries-iot.dbaas-acme-corp.svc.cluster.local:9009         (QuestDB)
```

### 3.3 Service Discovery API

```bash
# Look up all database endpoints for the current tenant
curl -X GET https://dbaas.erp.io/api/v1/instances?fields=name,engine,connectionInfo \
  -H "Authorization: Bearer ${JWT_TOKEN}"
```

```json
{
  "instances": [
    {
      "name": "finance-ledger-db",
      "engine": "yugabytedb",
      "connectionInfo": {
        "host": "finance-ledger-db.dbaas-acme-corp.svc.cluster.local",
        "port": 5433,
        "database": "finance_db",
        "secretRef": "finance-ledger-db-credentials"
      }
    },
    {
      "name": "cache-session",
      "engine": "dragonflydb",
      "connectionInfo": {
        "host": "cache-session.dbaas-acme-corp.svc.cluster.local",
        "port": 6379,
        "secretRef": "cache-session-credentials"
      }
    }
  ]
}
```

---

## 4. Connection String Injection via K8s Secrets

### 4.1 Secret Structure

When DBaaS provisions a database instance, it creates a Kubernetes Secret in the tenant namespace containing all connection details.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: finance-ledger-db-credentials
  namespace: dbaas-acme-corp
  labels:
    dbaas.erp.io/instance: finance-ledger-db
    dbaas.erp.io/engine: yugabytedb
type: Opaque
stringData:
  POSTGRES_HOST: "finance-ledger-db.dbaas-acme-corp.svc.cluster.local"
  POSTGRES_PORT: "5433"
  POSTGRES_USER: "finance_app"
  POSTGRES_PASSWORD: "Ax9#kL2m...32chars..."
  POSTGRES_DB: "finance_db"
  DATABASE_URL: "postgresql://finance_app:Ax9%23kL2m...@finance-ledger-db.dbaas-acme-corp.svc.cluster.local:5433/finance_db?sslmode=require"
```

### 4.2 Consuming Secrets in ERP Module Pods

**Environment Variable Injection:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: erp-finance-api
  namespace: erp-finance
spec:
  template:
    spec:
      containers:
        - name: finance-api
          image: erp-finance-api:latest
          envFrom:
            - secretRef:
                name: finance-ledger-db-credentials
          # Or individual env vars:
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: finance-ledger-db-credentials
                  key: DATABASE_URL
```

**Volume Mount Injection:**

```yaml
spec:
  containers:
    - name: finance-api
      volumeMounts:
        - name: db-credentials
          mountPath: /etc/dbaas/credentials
          readOnly: true
  volumes:
    - name: db-credentials
      secret:
        secretName: finance-ledger-db-credentials
```

### 4.3 Cross-Namespace Secret Sharing

When an ERP module runs in a different namespace from its database, the DBaaS platform uses a Secret Mirror Controller to replicate the Secret.

```
dbaas-acme-corp namespace          erp-finance namespace
┌──────────────────────┐          ┌──────────────────────┐
│ Secret: finance-     │ ──mirror──> │ Secret: finance-    │
│ ledger-db-creds      │          │ ledger-db-creds      │
│ (source of truth)    │          │ (read-only copy)     │
└──────────────────────┘          └──────────────────────┘
```

The mirror controller watches for annotation `dbaas.erp.io/mirror-to: erp-finance` on the source Secret and maintains a synchronized copy in the target namespace.

---

## 5. Observability Integration

### 5.1 Metrics Pipeline

```
Database Pods                  VictoriaMetrics           Grafana
┌──────────┐                  ┌──────────────┐          ┌──────────┐
│ Engine    │──[exporter]────>│ vmagent      │────────>│ Dashboard│
│ Metrics   │                  │ (scrape)     │          │          │
│ Exporter  │                  │              │          │          │
│ (sidecar) │                  │ vmselect     │<────────│          │
└──────────┘                  │ (query)      │          └──────────┘
                              └──────────────┘
```

### 5.2 Exported Metrics per Engine

Each database instance exports a standard set of metrics via a Prometheus-compatible exporter sidecar.

| Metric Name                          | Type      | Labels                          | Description                |
|--------------------------------------|-----------|---------------------------------|----------------------------|
| `dbaas_instance_up`                  | Gauge     | engine, instance, tenant        | Instance health (0/1)      |
| `dbaas_instance_cpu_usage_percent`   | Gauge     | engine, instance, tenant        | CPU utilization percentage |
| `dbaas_instance_memory_usage_bytes`  | Gauge     | engine, instance, tenant        | Memory usage in bytes      |
| `dbaas_instance_storage_usage_bytes` | Gauge     | engine, instance, tenant        | Storage usage in bytes     |
| `dbaas_instance_connections_active`  | Gauge     | engine, instance, tenant        | Active connection count    |
| `dbaas_instance_queries_total`       | Counter   | engine, instance, tenant        | Total queries executed     |
| `dbaas_instance_query_latency_seconds`| Histogram| engine, instance, tenant        | Query latency distribution |
| `dbaas_backup_last_success_timestamp`| Gauge     | engine, instance, tenant        | Last successful backup time|
| `dbaas_backup_size_bytes`            | Gauge     | engine, instance, tenant        | Last backup size           |
| `dbaas_replication_lag_seconds`      | Gauge     | engine, instance, tenant, role  | Replication lag in seconds |

### 5.3 Alerting Rules

```yaml
groups:
  - name: dbaas-alerts
    rules:
      - alert: DBaaSInstanceDown
        expr: dbaas_instance_up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database instance {{ $labels.instance }} is down"

      - alert: DBaaSHighCPU
        expr: dbaas_instance_cpu_usage_percent > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High CPU on {{ $labels.instance }}: {{ $value }}%"

      - alert: DBaaSStorageNearFull
        expr: dbaas_instance_storage_usage_bytes / dbaas_instance_storage_limit_bytes > 0.85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Storage at {{ $value | humanizePercentage }} on {{ $labels.instance }}"

      - alert: DBaaSReplicationLagHigh
        expr: dbaas_replication_lag_seconds > 30
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Replication lag {{ $value }}s on {{ $labels.instance }}"

      - alert: DBaaSBackupStale
        expr: time() - dbaas_backup_last_success_timestamp > 86400 * 2
        for: 1h
        labels:
          severity: critical
        annotations:
          summary: "No successful backup in 48h for {{ $labels.instance }}"
```

---

## 6. Event Notifications via Apache Pulsar

### 6.1 Event Architecture

DBaaS publishes lifecycle events to Apache Pulsar topics. ERP modules can subscribe to these events for automated workflows.

```
DBaaS Platform ──[produce]──> Apache Pulsar ──[consume]──> ERP Modules
                               │
                          Topics:
                          persistent://erp/dbaas/instance-events
                          persistent://erp/dbaas/backup-events
                          persistent://erp/dbaas/credential-events
                          persistent://erp/dbaas/alert-events
```

### 6.2 Event Schema

All events follow a common envelope format.

```json
{
  "eventId": "evt-a1b2c3d4",
  "eventType": "instance.provisioned",
  "timestamp": "2026-02-24T10:32:00Z",
  "tenantId": "acme-corp",
  "source": "dbaas-api",
  "data": {
    "instanceId": "inst-a1b2c3d4",
    "instanceName": "finance-ledger-db",
    "engine": "yugabytedb",
    "version": "2.20.1",
    "size": "M",
    "status": "running",
    "connectionInfo": {
      "host": "finance-ledger-db.dbaas-acme-corp.svc.cluster.local",
      "port": 5433,
      "secretRef": "finance-ledger-db-credentials"
    }
  }
}
```

### 6.3 Event Types

| Topic                  | Event Type                | Description                          |
|------------------------|---------------------------|--------------------------------------|
| instance-events        | `instance.provisioning`   | Instance provisioning started        |
| instance-events        | `instance.provisioned`    | Instance is running and ready        |
| instance-events        | `instance.scaling`        | Instance scaling in progress         |
| instance-events        | `instance.scaled`         | Instance scaling completed           |
| instance-events        | `instance.failed`         | Instance entered failed state        |
| instance-events        | `instance.deleted`        | Instance has been deleted            |
| backup-events          | `backup.started`          | Backup operation initiated           |
| backup-events          | `backup.completed`        | Backup operation succeeded           |
| backup-events          | `backup.failed`           | Backup operation failed              |
| backup-events          | `backup.restored`         | Backup restoration completed         |
| credential-events      | `credential.rotated`      | Credentials have been rotated        |
| credential-events      | `credential.expiring`     | Credentials will expire soon         |
| alert-events           | `alert.high_cpu`          | High CPU usage detected              |
| alert-events           | `alert.storage_full`      | Storage approaching capacity         |
| alert-events           | `alert.replication_lag`   | Replication lag exceeded threshold   |

### 6.4 Consumer Example (ERP Module)

```typescript
import Pulsar from 'pulsar-client';

const client = new Pulsar.Client({
  serviceUrl: 'pulsar://pulsar.erp.internal:6650',
});

const consumer = await client.subscribe({
  topic: 'persistent://erp/dbaas/instance-events',
  subscription: 'erp-finance-consumer',
  subscriptionType: 'Shared',
});

while (true) {
  const msg = await consumer.receive();
  const event = JSON.parse(msg.getData().toString());

  switch (event.eventType) {
    case 'instance.provisioned':
      console.log(`Instance ${event.data.instanceName} is ready!`);
      // Run database migrations
      await runMigrations(event.data.connectionInfo);
      break;
    case 'credential.rotated':
      console.log(`Credentials rotated for ${event.data.instanceName}`);
      // Restart connection pool
      await restartConnectionPool(event.data.instanceId);
      break;
  }

  await consumer.acknowledge(msg);
}
```

### 6.5 Event Delivery Guarantees

| Property               | Specification                          |
|------------------------|----------------------------------------|
| Delivery guarantee     | At-least-once                          |
| Message ordering       | Per-key ordering (key = instance_id)   |
| Retention              | 7 days (configurable per topic)        |
| Max message size       | 5 MiB                                  |
| Consumer ack timeout   | 30 seconds                             |
| Dead letter topic      | Enabled after 3 delivery attempts      |
| Schema enforcement     | Avro schema registry for event schemas |

---

## Appendix: Integration Checklist for New ERP Modules

| Step | Action                                                     | Owner         |
|------|------------------------------------------------------------|---------------|
| 1    | Request DBaaS tenant onboarding with tier assignment        | Module Lead   |
| 2    | Provision required database instances via API or dashboard  | Module Dev    |
| 3    | Configure Secret references in module K8s deployment specs  | Module DevOps |
| 4    | Set up Pulsar consumer for instance lifecycle events        | Module Dev    |
| 5    | Add Hasura Remote Schema for DBaaS (if using federation)    | Platform Team |
| 6    | Configure monitoring dashboards for database metrics        | Module SRE    |
| 7    | Run integration tests against provisioned instances         | Module QA     |
| 8    | Document database architecture in module ADR                | Module Lead   |
