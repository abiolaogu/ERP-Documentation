# ERP-DBaaS API Documentation

Complete API reference for the ERP-DBaaS platform. All endpoints are accessible through the Go gateway on port 8090 with the `/v1/dbaas/` prefix.

---

## Table of Contents

1. [Authentication](#authentication)
2. [Common Response Format](#common-response-format)
3. [Error Codes](#error-codes)
4. [Instances API](#instances-api)
5. [Backups API](#backups-api)
6. [Credentials API](#credentials-api)
7. [Engines API](#engines-api)
8. [Plans API](#plans-api)
9. [Profiles API](#profiles-api)
10. [Plugins API](#plugins-api)
11. [Tenants API](#tenants-api)
12. [Gateway Endpoints](#gateway-endpoints)

---

## Authentication

All `/v1/dbaas/` endpoints require authentication via Bearer JWT token issued by Authentik.

### Production Authentication

```
Authorization: Bearer <jwt-token>
```

The JWT token must contain:
- `sub` (string): User ID.
- `tenantId` (string): Tenant ID for multi-tenant isolation.
- `roles` (string[]): Must include at least one of: `dbaas_admin`, `dbaas_operator`, `dbaas_viewer`, `admin`, `platform_admin`.
- `modules` (string[]): Must include `ERP-DBaaS`.

### Development Authentication

In development mode (`NODE_ENV=development`), authentication can be bypassed using headers:

```
X-Dev-Tenant-ID: tenant-001
X-Dev-User-ID: dev-user
X-Dev-Profile: erp-aidd-strict
```

### Role Permissions

| Role | Instances (Read) | Instances (Write) | Backups | Credentials | Plugins |
|---|---|---|---|---|---|
| `dbaas_viewer` | Yes | No | Read | No | Read |
| `dbaas_operator` | Yes | Yes | Yes | Read | Read |
| `dbaas_admin` | Yes | Yes | Yes | Yes | Yes |

---

## Common Response Format

### Success Response

```json
{
  "success": true,
  "data": { ... },
  "message": "Human-readable message.",
  "requestId": "req-1708765432-1"
}
```

### Paginated Response

```json
{
  "success": true,
  "data": [ ... ],
  "total": 42,
  "page": 1,
  "pageSize": 20,
  "requestId": "req-1708765432-2"
}
```

### Error Response

```json
{
  "success": false,
  "error": "error_code",
  "message": "Human-readable error description.",
  "requestId": "req-1708765432-3"
}
```

---

## Error Codes

| HTTP Status | Error Code | Description |
|---|---|---|
| 400 | `validation_error` | Request body failed Zod schema validation. |
| 401 | `unauthorized` | Missing or invalid Authorization header. |
| 401 | `invalid_token` | JWT verification failed (expired, malformed, invalid signature). |
| 403 | `forbidden` | Token lacks required role or module access. |
| 403 | `policy_violation` | Request violates AIDD policy profile rules. |
| 404 | `not_found` | Requested resource does not exist or does not belong to the tenant. |
| 409 | `invalid_state` | Instance is in an incompatible state for the requested operation. |
| 409 | `conflict` | Resource already exists (e.g., duplicate plugin name). |
| 409 | `rotation_in_progress` | A credential rotation is already in progress. |
| 422 | `connectivity_error` | Cannot reach plugin gRPC endpoint. |
| 429 | `quota_exceeded` | Tenant has reached the maximum instance limit. |
| 429 | `rate_limit_exceeded` | API rate limit exceeded for the tenant's tier. |
| 500 | `internal_error` | Unexpected server error. |
| 502 | `bad_gateway` | Upstream service unreachable (gateway proxy error). |

### Rate Limit Headers

All rate-limited responses include:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1708765500
Retry-After: 45
```

---

## Instances API

### POST /v1/dbaas/instances

Provision a new database instance.

**Request Body**:

```json
{
  "engine": "yugabytedb",
  "version": "2.21",
  "plan": "M",
  "haMode": "standalone",
  "region": "africa-west",
  "profile": "erp-aidd-strict",
  "config": {
    "ysql_max_connections": 200,
    "tls_enabled": true
  }
}
```

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `engine` | string | Yes | - | Database engine. Validated against active policy profile. |
| `version` | string | Yes | - | Engine version (1-20 chars). |
| `plan` | enum | No | `M` | Size preset: `S`, `M`, `L`, `XL`. |
| `haMode` | enum | No | `standalone` | HA mode: `standalone`, `ha`, `multi_region`. |
| `region` | string | No | `africa-west` | Deployment region. |
| `profile` | enum | No | tenant's profile | Policy profile override: `erp-aidd-strict`, `commercial-flexible`. |
| `config` | object | No | `{}` | Engine-specific configuration key-value pairs. |

**Valid Engines**:

| Profile | Engines |
|---|---|
| `erp-aidd-strict` | `yugabytedb`, `scylladb`, `dragonfly`, `mongodb`, `couchdb`, `clickhouse`, `timescaledb`, `questdb` |
| `commercial-flexible` | All of the above PLUS `postgresql`, `mysql`, `mariadb`, `redis_valkey` |

**Valid Regions**: `africa-west`, `africa-east`, `africa-south`, `europe-west`, `europe-central`, `us-east`, `us-west`, `asia-southeast`.

**Response (201 Created)**:

```json
{
  "success": true,
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "tenant_id": "tenant-001",
    "engine": "yugabytedb",
    "version": "2.21",
    "plan": "M",
    "ha_mode": "standalone",
    "status": "provisioning",
    "namespace": "dbaas-tenant-001-a1b2c3d4",
    "endpoint": null,
    "port": null,
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "config": {},
    "resource_usage": {},
    "created_at": "2026-02-24T10:00:00.000Z",
    "updated_at": "2026-02-24T10:00:00.000Z"
  },
  "message": "Instance provisioning initiated.",
  "requestId": "req-1708765432-1"
}
```

**Error Examples**:

Policy violation (trying PostgreSQL under strict profile):
```json
{
  "success": false,
  "error": "policy_violation",
  "message": "Engine 'postgresql' is DENIED under ERP_AIDD_STRICT profile. Only AIDD-approved engines are allowed: yugabytedb, scylladb, dragonfly, mongodb, couchdb, clickhouse, timescaledb, questdb. If you need PostgreSQL, MySQL, MariaDB, or Redis, the tenant must use the COMMERCIAL_FLEXIBLE profile."
}
```

Quota exceeded:
```json
{
  "success": false,
  "error": "quota_exceeded",
  "message": "Tenant has reached the maximum instance limit (5)."
}
```

---

### GET /v1/dbaas/instances

List all instances for the authenticated tenant with pagination and filters.

**Query Parameters**:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `engine` | string | - | Filter by engine type. |
| `status` | string | - | Filter by status. |
| `page` | integer | `1` | Page number. |
| `pageSize` | integer | `20` | Items per page. |

**Example Request**:

```bash
curl "http://localhost:8090/v1/dbaas/instances?engine=yugabytedb&status=running&page=1&pageSize=10" \
  -H "Authorization: Bearer <token>"
```

**Response (200 OK)**:

```json
{
  "success": true,
  "data": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "tenant_id": "tenant-001",
      "engine": "yugabytedb",
      "version": "2.21",
      "plan": "M",
      "ha_mode": "standalone",
      "status": "running",
      "namespace": "dbaas-tenant-001-a1b2c3d4",
      "endpoint": "yugabytedb-a1b2c3d4.dbaas-system.svc.cluster.local",
      "port": 5433,
      "region": "africa-west",
      "profile": "erp-aidd-strict",
      "config": {},
      "resource_usage": {},
      "created_at": "2026-02-24T10:00:00.000Z",
      "updated_at": "2026-02-24T10:05:00.000Z"
    }
  ],
  "total": 1,
  "page": 1,
  "pageSize": 10,
  "requestId": "req-1708765432-2"
}
```

---

### GET /v1/dbaas/instances/:id

Get detailed information about a specific instance.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "tenant_id": "tenant-001",
    "engine": "yugabytedb",
    "version": "2.21",
    "plan": "M",
    "ha_mode": "standalone",
    "status": "running",
    "namespace": "dbaas-tenant-001-a1b2c3d4",
    "endpoint": "yugabytedb-a1b2c3d4.dbaas-system.svc.cluster.local",
    "port": 5433,
    "region": "africa-west",
    "profile": "erp-aidd-strict",
    "config": {"ysql_max_connections": 200},
    "resource_usage": {"cpu": "1.2", "memory": "4.5Gi", "storage": "25Gi"},
    "created_at": "2026-02-24T10:00:00.000Z",
    "updated_at": "2026-02-24T10:05:00.000Z"
  },
  "requestId": "req-1708765432-3"
}
```

---

### GET /v1/dbaas/instances/:id/metrics

Get metrics and recent metering events for an instance.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {
    "instanceId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "engine": "yugabytedb",
    "status": "running",
    "resourceUsage": {"cpu": "1.2", "memory": "4.5Gi", "storage": "25Gi"},
    "recentMetering": [
      {
        "id": "m1m2m3m4-...",
        "instance_id": "a1b2c3d4-...",
        "tenant_id": "tenant-001",
        "event_type": "usage",
        "cpu_seconds": 3600.5,
        "memory_gb_seconds": 16200.0,
        "storage_gb_hours": 600.0,
        "io_operations": 125000,
        "recorded_at": "2026-02-24T10:00:00.000Z"
      }
    ]
  },
  "requestId": "req-1708765432-4"
}
```

---

### PUT /v1/dbaas/instances/:id/scale

Scale an instance to a different plan or resource configuration.

**Prerequisites**: Instance must be in `running` state.

**Request Body**:

```json
{
  "plan": "L",
  "replicas": 3,
  "resources": {
    "cpu": "4",
    "memory": "16Gi",
    "storage": "200Gi"
  }
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `plan` | enum | No | New plan: `S`, `M`, `L`, `XL`. |
| `replicas` | integer | No | Number of replicas (1-15). |
| `resources.cpu` | string | No | Custom CPU allocation. |
| `resources.memory` | string | No | Custom memory allocation. |
| `resources.storage` | string | No | Custom storage allocation. |

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {"instanceId": "a1b2c3d4-...", "status": "scaling"},
  "message": "Scaling initiated.",
  "requestId": "req-1708765432-5"
}
```

---

### PUT /v1/dbaas/instances/:id/config

Update engine-specific configuration parameters.

**Request Body**:

```json
{
  "config": {
    "ysql_max_connections": 300,
    "log_min_duration_statement": 1000
  },
  "restartRequired": true
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `config` | object | Yes | Key-value pairs to merge into existing config. |
| `restartRequired` | boolean | No | Hint that a restart is needed for changes to take effect. |

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {
    "instanceId": "a1b2c3d4-...",
    "config": {"ysql_max_connections": 300, "log_min_duration_statement": 1000, "tls_enabled": true},
    "restartRequired": true
  },
  "message": "Configuration updated. A restart is required for changes to take effect.",
  "requestId": "req-1708765432-6"
}
```

**Forbidden Config Keys (ERP_AIDD_STRICT)**:
`ssl_disabled`, `auth_disabled`, `allow_external_access`, `disable_encryption`, `disable_audit_log`.

---

### POST /v1/dbaas/instances/:id/restart

Trigger a rolling restart of the instance.

**Prerequisites**: Instance must be in `running` state.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {"instanceId": "a1b2c3d4-...", "action": "restart"},
  "message": "Rolling restart initiated.",
  "requestId": "req-1708765432-7"
}
```

---

### DELETE /v1/dbaas/instances/:id

Decommission an instance. This triggers async cleanup of all K8s resources, PVCs, and secrets.

**Prerequisites**: Instance must not already be in `decommissioning` or `decommissioned` state.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {"instanceId": "a1b2c3d4-...", "status": "decommissioning"},
  "message": "Instance decommissioning initiated.",
  "requestId": "req-1708765432-8"
}
```

---

## Backups API

### POST /v1/dbaas/instances/:instanceId/backup

Trigger an on-demand backup.

**Prerequisites**: Instance must be in `running` state.

**Request Body**:

```json
{
  "type": "full",
  "retentionDays": 30
}
```

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `type` | enum | No | `full` | Backup type: `full`, `incremental`, `snapshot`. |
| `retentionDays` | integer | No | `30` | Retention period (1-365 days). |

**Response (201 Created)**:

```json
{
  "success": true,
  "data": {
    "id": "b1b2b3b4-...",
    "instance_id": "a1b2c3d4-...",
    "type": "full",
    "status": "pending",
    "storage_path": null,
    "size_bytes": null,
    "started_at": "2026-02-24T10:30:00.000Z",
    "completed_at": null,
    "retention_days": 30,
    "encrypted": true
  },
  "message": "Backup initiated.",
  "requestId": "req-1708765432-9"
}
```

---

### GET /v1/dbaas/instances/:instanceId/backups

List backups for an instance.

**Query Parameters**:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `page` | integer | `1` | Page number. |
| `pageSize` | integer | `20` | Items per page. |

**Response (200 OK)**:

```json
{
  "success": true,
  "data": [
    {
      "id": "b1b2b3b4-...",
      "instance_id": "a1b2c3d4-...",
      "type": "full",
      "status": "completed",
      "storage_path": "rustfs://backups/tenant-001/a1b2c3d4/2026-02-24T10-30-00.full.zstd",
      "size_bytes": 1073741824,
      "started_at": "2026-02-24T10:30:00.000Z",
      "completed_at": "2026-02-24T10:35:00.000Z",
      "retention_days": 30,
      "encrypted": true
    }
  ],
  "total": 1,
  "page": 1,
  "pageSize": 20,
  "requestId": "req-1708765432-10"
}
```

---

### POST /v1/dbaas/instances/:instanceId/restore

Restore an instance from a backup.

**Request Body**:

```json
{
  "backupId": "b1b2b3b4-e5f6-7890-abcd-ef1234567890",
  "targetTimestamp": "2026-02-24T10:32:00.000Z",
  "targetInstance": "c1c2c3c4-e5f6-7890-abcd-ef1234567890"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `backupId` | UUID | Yes | ID of a completed backup. |
| `targetTimestamp` | ISO 8601 | No | Point-in-time restore target (if engine supports PITR). |
| `targetInstance` | UUID | No | Restore to a different instance (cross-instance restore). |

**Response (202 Accepted)**:

```json
{
  "success": true,
  "data": {
    "instanceId": "a1b2c3d4-...",
    "backupId": "b1b2b3b4-...",
    "status": "in_progress"
  },
  "message": "Restore initiated.",
  "requestId": "req-1708765432-11"
}
```

---

## Credentials API

### GET /v1/dbaas/instances/:instanceId/credentials

Retrieve connection credentials for an instance from HashiCorp Vault.

**Prerequisites**: Instance must be in `running` state.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {
    "instanceId": "a1b2c3d4-...",
    "engine": "yugabytedb",
    "host": "yugabytedb-a1b2c3d4.dbaas-system.svc.cluster.local",
    "port": 5433,
    "username": "dbaas_a1b2c3d4",
    "database": "dbaas_registry",
    "connectionString": "postgresql://dbaas_a1b2c3d4:***@yugabytedb-a1b2c3d4.dbaas-system.svc.cluster.local:5433/dbaas_registry?sslmode=require",
    "expiresAt": "2026-03-24T10:00:00.000Z"
  },
  "requestId": "req-1708765432-12"
}
```

---

### POST /v1/dbaas/instances/:instanceId/credentials/rotate

Initiate credential rotation for an instance.

**Prerequisites**:
- Instance must be in `running` state.
- No rotation already in progress for this instance.

**Response (202 Accepted)**:

```json
{
  "success": true,
  "data": {
    "rotationId": "r1r2r3r4-...",
    "instanceId": "a1b2c3d4-...",
    "status": "pending"
  },
  "message": "Credential rotation initiated.",
  "requestId": "req-1708765432-13"
}
```

---

## Engines API

### GET /v1/dbaas/engines

List all available database engines filtered by the tenant's active policy profile.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": [
    {
      "engine": "yugabytedb",
      "displayName": "YugabyteDB",
      "description": "Distributed SQL database for transactional workloads. YSQL (PostgreSQL-compatible) + YCQL.",
      "versions": ["2.20", "2.21", "2.23"],
      "operator": "kubedb",
      "supportedHAModes": ["standalone", "ha", "multi_region"],
      "profile": "both"
    },
    {
      "engine": "scylladb",
      "displayName": "ScyllaDB",
      "description": "High-performance wide-column NoSQL database, Cassandra-compatible.",
      "versions": ["5.4", "6.0", "6.1"],
      "operator": "scylla-operator",
      "supportedHAModes": ["standalone", "ha", "multi_region"],
      "profile": "both"
    },
    {
      "engine": "dragonfly",
      "displayName": "DragonflyDB",
      "description": "Ultra-fast in-memory datastore, Redis/Memcached compatible.",
      "versions": ["1.21", "1.22", "1.23"],
      "operator": "kubedb",
      "supportedHAModes": ["standalone", "ha"],
      "profile": "both"
    },
    {
      "engine": "mongodb",
      "displayName": "MongoDB",
      "description": "General-purpose document database with flexible schema.",
      "versions": ["7.0", "8.0"],
      "operator": "kubedb",
      "supportedHAModes": ["standalone", "ha", "multi_region"],
      "profile": "both"
    },
    {
      "engine": "couchdb",
      "displayName": "Apache CouchDB",
      "description": "Distributed document database with multi-master replication.",
      "versions": ["3.3", "3.4"],
      "operator": "couchdb-controller",
      "supportedHAModes": ["standalone", "ha"],
      "profile": "both"
    },
    {
      "engine": "clickhouse",
      "displayName": "ClickHouse",
      "description": "Column-oriented OLAP database for real-time analytics.",
      "versions": ["24.3", "24.8", "24.12"],
      "operator": "altinity-operator",
      "supportedHAModes": ["standalone", "ha", "multi_region"],
      "profile": "both"
    },
    {
      "engine": "timescaledb",
      "displayName": "TimescaleDB",
      "description": "Time-series database built on PostgreSQL engine.",
      "versions": ["2.14", "2.15", "2.16"],
      "operator": "zalando-operator",
      "supportedHAModes": ["standalone", "ha"],
      "profile": "both"
    },
    {
      "engine": "questdb",
      "displayName": "QuestDB",
      "description": "High-performance time-series database with SQL support.",
      "versions": ["8.0", "8.1"],
      "operator": "questdb-controller",
      "supportedHAModes": ["standalone"],
      "profile": "both"
    }
  ],
  "total": 8,
  "requestId": "req-1708765432-14"
}
```

Under `COMMERCIAL_FLEXIBLE` profile, the response includes 4 additional engines: PostgreSQL, MySQL, MariaDB, and Redis/Valkey.

---

## Plans API

### GET /v1/dbaas/plans

List available sizing plans.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": [
    {"size": "S",  "displayName": "Small",       "resources": {"cpu": "500m", "memory": "1Gi",  "storage": "10Gi"},  "monthlyPriceUsd": 29},
    {"size": "M",  "displayName": "Medium",      "resources": {"cpu": "2",    "memory": "8Gi",  "storage": "50Gi"},  "monthlyPriceUsd": 99},
    {"size": "L",  "displayName": "Large",       "resources": {"cpu": "4",    "memory": "16Gi", "storage": "200Gi"}, "monthlyPriceUsd": 299},
    {"size": "XL", "displayName": "Extra Large",  "resources": {"cpu": "8",    "memory": "32Gi", "storage": "500Gi"}, "monthlyPriceUsd": 599}
  ],
  "total": 4,
  "requestId": "req-1708765432-15"
}
```

---

## Profiles API

### GET /v1/dbaas/profiles

List available policy profiles.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": [
    {
      "name": "erp-aidd-strict",
      "mode": "strict",
      "allowedEngines": ["yugabytedb", "scylladb", "dragonfly", "mongodb", "couchdb", "clickhouse", "timescaledb", "questdb"],
      "deniedEngines": ["postgresql", "mysql", "mariadb", "redis_valkey"],
      "requiredCompliance": ["soc2", "gdpr", "iso27001"],
      "backupRequirements": {
        "minRetentionDays": 30,
        "encryptionRequired": true,
        "crossRegionRequired": true
      }
    },
    {
      "name": "commercial-flexible",
      "mode": "flexible",
      "allowedEngines": ["yugabytedb", "scylladb", "dragonfly", "mongodb", "couchdb", "clickhouse", "timescaledb", "questdb", "postgresql", "mysql", "mariadb", "redis_valkey"],
      "deniedEngines": [],
      "requiredCompliance": ["soc2"],
      "backupRequirements": {
        "minRetentionDays": 7,
        "encryptionRequired": true,
        "crossRegionRequired": false
      }
    }
  ],
  "total": 2,
  "requestId": "req-1708765432-16"
}
```

---

## Plugins API

### GET /v1/dbaas/plugins

List registered plugins with optional filters.

**Query Parameters**:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `status` | string | - | Filter by status: `pending_validation`, `validated`, `active`, `disabled`, `rejected`. |
| `engine` | string | - | Filter by engine type. |
| `page` | integer | `1` | Page number. |
| `pageSize` | integer | `20` | Items per page. |

**Response (200 OK)**:

```json
{
  "success": true,
  "data": [
    {
      "id": "p1p2p3p4-...",
      "name": "neo4j-plugin",
      "engine": "neo4j",
      "version": "1.0.0",
      "grpcEndpoint": "http://neo4j-plugin.dbaas-plugins.svc.cluster.local:50051",
      "status": "active",
      "capabilities": ["provision", "scale", "backup", "restore", "health"],
      "registeredAt": "2026-02-20T10:00:00.000Z",
      "validatedAt": "2026-02-20T10:00:05.000Z"
    }
  ],
  "total": 1,
  "page": 1,
  "pageSize": 20,
  "requestId": "req-1708765432-17"
}
```

---

### POST /v1/dbaas/plugins

Register a new plugin.

**Request Body**:

```json
{
  "name": "neo4j-plugin",
  "engine": "neo4j",
  "version": "1.0.0",
  "grpcEndpoint": "http://neo4j-plugin.dbaas-plugins.svc.cluster.local:50051",
  "capabilities": ["provision", "scale", "backup", "restore", "rotate", "health"],
  "securityContext": {
    "serviceAccount": "neo4j-plugin-sa",
    "namespace": "dbaas-plugins"
  }
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | Plugin name (3-64 chars, DNS-compatible: `^[a-z0-9][a-z0-9-]*[a-z0-9]$`). |
| `engine` | string | Yes | Engine type this plugin manages. |
| `version` | string | Yes | Plugin version. |
| `grpcEndpoint` | URL | Yes | gRPC endpoint URL. |
| `capabilities` | string[] | Yes | At least one capability. |
| `securityContext` | object | No | K8s security context. |

**Response (201 Created)**:

```json
{
  "success": true,
  "data": {
    "id": "p1p2p3p4-...",
    "name": "neo4j-plugin",
    "engine": "neo4j",
    "version": "1.0.0",
    "grpcEndpoint": "http://neo4j-plugin.dbaas-plugins.svc.cluster.local:50051",
    "status": "pending_validation",
    "capabilities": ["provision", "scale", "backup", "restore", "rotate", "health"],
    "registeredAt": "2026-02-24T10:00:00.000Z",
    "validatedAt": null
  },
  "message": "Plugin registered and pending validation.",
  "requestId": "req-1708765432-18"
}
```

---

## Tenants API

### GET /v1/dbaas/tenants/:tenantId/quota

Retrieve quota information for a tenant.

**Response (200 OK)**:

```json
{
  "success": true,
  "data": {
    "tenantId": "tenant-001",
    "tier": "C",
    "maxInstances": 5,
    "maxCpu": "16",
    "maxMemory": "64Gi",
    "maxStorage": "500Gi",
    "currentInstances": 2,
    "currentCpu": "4",
    "currentMemory": "16Gi",
    "currentStorage": "100Gi",
    "updatedAt": "2026-02-24T10:00:00.000Z"
  },
  "requestId": "req-1708765432-19"
}
```

---

## Gateway Endpoints

These endpoints are served directly by the Go gateway and do not proxy to the Node.js API.

### GET /healthz

Gateway health check.

```json
{"status": "healthy", "module": "ERP-DBaaS"}
```

### GET /v1/capabilities

Module capabilities.

```json
{
  "module": "ERP-DBaaS",
  "version": "1.0.0",
  "capabilities": [
    "database_provisioning", "database_scaling", "backup_restore",
    "credential_rotation", "policy_profiles", "plugin_management",
    "metering", "multi_engine", "multi_tenant", "multi_region"
  ],
  "integration_mode": "platform_service",
  "aidd_governance": "ERP_AIDD_STRICT"
}
```
