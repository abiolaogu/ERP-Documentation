# ERP-DBaaS Technical Specifications

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Technical Specifications     |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Engineering                 |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [Supported Engine Versions Matrix](#1-supported-engine-versions-matrix)
2. [Size Presets and Resource Allocations](#2-size-presets-and-resource-allocations)
3. [Backup Retention Policies](#3-backup-retention-policies)
4. [SLA Targets per Tier](#4-sla-targets-per-tier)
5. [Rate Limits per Tier](#5-rate-limits-per-tier)
6. [Storage Backend (RustFS) Integration Specs](#6-storage-backend-rustfs-integration-specs)

---

## 1. Supported Engine Versions Matrix

### 1.1 Current Supported Versions

| Engine        | Current Version | Min Supported | Max Supported | EOL Date   | Auto-Upgrade |
|---------------|-----------------|---------------|---------------|------------|--------------|
| YugabyteDB    | 2.20.x          | 2.18.0        | 2.20.x        | 2027-06-01 | Minor only   |
| DragonflyDB   | 1.x             | 1.14.0        | 1.x           | N/A        | Minor only   |
| ClickHouse    | 24.x            | 23.8 LTS      | 24.x          | 2027-03-01 | Minor only   |
| Tembo (PG)    | 16.x            | 15.0          | 16.x          | 2028-11-01 | Minor only   |
| SurrealDB     | 2.x             | 1.5.0         | 2.x           | N/A        | Minor only   |
| QuestDB       | 8.x             | 7.4.0         | 8.x           | N/A        | Minor only   |
| Apache Doris  | 2.1.x           | 2.0.0         | 2.1.x         | N/A        | Minor only   |
| InfluxDB      | 2.7.x           | 2.6.0         | 2.7.x         | N/A        | Minor only   |

### 1.2 Version Upgrade Policy

- **Patch Versions**: Applied automatically during the next maintenance window (weekly, Saturday 02:00-06:00 UTC).
- **Minor Versions**: Available as opt-in upgrades. Tenant is notified and must approve. Automated rollback on failure.
- **Major Versions**: Require manual migration. A parallel instance is provisioned with the new version, data is migrated, and traffic is switched upon validation.

### 1.3 Deprecation Timeline

| Stage            | Notice Period | Action                                      |
|------------------|---------------|----------------------------------------------|
| Deprecation      | 6 months      | Version marked deprecated in UI, no new instances allowed |
| End of Support   | 3 months      | Security patches only, upgrade strongly recommended       |
| End of Life      | 0             | Instances forcibly upgraded to the next supported version  |

---

## 2. Size Presets and Resource Allocations

### 2.1 Standard Size Presets

| Preset | vCPU | Memory  | Storage  | IOPS (Baseline) | Network Bandwidth | Max Connections |
|--------|------|---------|----------|------------------|-------------------|-----------------|
| S      | 1    | 2 GiB   | 20 GiB   | 3,000            | 1 Gbps            | 50              |
| M      | 2    | 4 GiB   | 100 GiB  | 6,000            | 2.5 Gbps          | 150             |
| L      | 4    | 16 GiB  | 500 GiB  | 12,000           | 5 Gbps            | 300             |
| XL     | 8    | 32 GiB  | 1 TiB    | 24,000           | 10 Gbps           | 500             |

### 2.2 Engine-Specific Overrides

Some engines require resource adjustments beyond the standard presets.

**YugabyteDB (HA mode with 3 nodes)**

| Preset | Master CPU | Master Mem | TServer CPU | TServer Mem | Storage/Node |
|--------|------------|------------|-------------|-------------|--------------|
| S      | 0.5        | 1 GiB      | 1           | 2 GiB       | 20 GiB       |
| M      | 1          | 2 GiB      | 2           | 4 GiB       | 100 GiB      |
| L      | 2          | 4 GiB      | 4           | 16 GiB      | 500 GiB      |
| XL     | 4          | 8 GiB      | 8           | 32 GiB      | 1 TiB        |

**DragonflyDB (Memory-Optimized)**

| Preset | vCPU | Memory  | Threads | Snapshot Storage | Max Keys (est.)  |
|--------|------|---------|---------|------------------|------------------|
| S      | 1    | 1 GiB   | 2       | 2 GiB            | ~10M             |
| M      | 2    | 4 GiB   | 4       | 8 GiB            | ~40M             |
| L      | 4    | 8 GiB   | 8       | 16 GiB           | ~80M             |
| XL     | 8    | 32 GiB  | 16      | 64 GiB           | ~320M            |

**ClickHouse (Storage-Optimized)**

| Preset | vCPU | Memory  | Storage  | Merge Threads | Max Parts/Partition |
|--------|------|---------|----------|---------------|---------------------|
| S      | 2    | 4 GiB   | 100 GiB  | 2             | 300                 |
| M      | 4    | 8 GiB   | 500 GiB  | 4             | 300                 |
| L      | 8    | 32 GiB  | 2 TiB    | 8             | 500                 |
| XL     | 16   | 64 GiB  | 5 TiB    | 16            | 500                 |

### 2.3 Custom Size Requests

Tenants on Tier A (Enterprise) may request custom resource allocations outside the standard presets. Custom sizes require:

- Platform administrator approval
- Capacity verification on the target cluster
- Custom SLA negotiation if exceeding XL boundaries

---

## 3. Backup Retention Policies

### 3.1 Default Retention per Tier

| Tier   | Full Backup Frequency | Full Retention | Incremental Frequency | Incremental Retention | WAL Retention |
|--------|----------------------|----------------|----------------------|-----------------------|---------------|
| Tier A | Daily                | 90 days        | Hourly               | 14 days               | 7 days        |
| Tier B | Daily                | 30 days        | Every 6 hours        | 7 days                | 3 days        |
| Tier C | Weekly               | 14 days        | Daily                | 3 days                | None          |

### 3.2 Backup Storage Limits per Tier

| Tier   | Max Backup Storage | Overage Policy                                      |
|--------|--------------------|------------------------------------------------------|
| Tier A | 10 TiB             | Alert at 80%, soft limit with 10% overage grace      |
| Tier B | 2 TiB              | Alert at 80%, hard limit (oldest backups pruned)      |
| Tier C | 200 GiB            | Alert at 80%, hard limit (oldest backups pruned)      |

### 3.3 Retention Enforcement

- A daily retention job runs at 04:00 UTC to identify expired backups.
- Expired backups are soft-deleted (metadata retained for 7 days, storage object marked for deletion).
- After the soft-delete grace period, a weekly cleanup job permanently removes storage objects and metadata.
- Legal hold overrides: Tenants can place a legal hold on specific backups to prevent expiration.

---

## 4. SLA Targets per Tier

### 4.1 Availability SLA

| Tier   | Monthly Uptime Target | Max Planned Downtime/Month | Max Unplanned Downtime/Month |
|--------|-----------------------|----------------------------|-------------------------------|
| Tier A | 99.99%                | 30 minutes                 | 4.3 minutes                   |
| Tier B | 99.95%                | 1 hour                     | 21.9 minutes                  |
| Tier C | 99.9%                 | 4 hours                    | 43.8 minutes                  |

### 4.2 Recovery SLA

| Tier   | RPO (Recovery Point Objective) | RTO (Recovery Time Objective) |
|--------|-------------------------------|-------------------------------|
| Tier A | 1 hour                        | 15 minutes                    |
| Tier B | 6 hours                       | 1 hour                        |
| Tier C | 24 hours                      | 4 hours                       |

### 4.3 Provisioning SLA

| Operation              | Tier A Target | Tier B Target | Tier C Target |
|------------------------|---------------|---------------|---------------|
| New Instance (S)       | < 30s         | < 60s         | < 120s        |
| New Instance (M)       | < 60s         | < 120s        | < 300s        |
| New Instance (L)       | < 120s        | < 300s        | < 600s        |
| New Instance (XL)      | < 300s        | < 600s        | < 900s        |
| Scaling (vertical)     | < 60s         | < 120s        | < 300s        |
| Backup (full, 100GB)   | < 10 min      | < 15 min      | < 30 min      |
| Restore (100GB)        | < 15 min      | < 30 min      | < 60 min      |
| Credential rotation    | < 30s         | < 60s         | < 60s         |

---

## 5. Rate Limits per Tier

### 5.1 API Rate Limits

| Endpoint Category      | Tier A      | Tier B      | Tier C      |
|------------------------|-------------|-------------|-------------|
| Instance CRUD          | 100 req/min | 30 req/min  | 10 req/min  |
| Backup operations      | 50 req/min  | 20 req/min  | 5 req/min   |
| Credential rotation    | 20 req/min  | 10 req/min  | 3 req/min   |
| Plugin management      | 30 req/min  | 10 req/min  | 5 req/min   |
| Metrics queries        | 200 req/min | 100 req/min | 30 req/min  |
| List / search          | 300 req/min | 100 req/min | 30 req/min  |

### 5.2 Rate Limit Implementation

- Rate limiting is enforced at the Go Gateway layer using a sliding window algorithm.
- Rate limit headers are included in all responses:
  - `X-RateLimit-Limit`: Maximum requests per window
  - `X-RateLimit-Remaining`: Remaining requests in current window
  - `X-RateLimit-Reset`: Seconds until the window resets
- When rate limited, the API returns HTTP 429 with a `Retry-After` header.
- Tier A tenants may request rate limit increases through the support portal.

### 5.3 Burst Allowance

| Tier   | Burst Multiplier | Burst Window |
|--------|------------------|--------------|
| Tier A | 3x               | 10 seconds   |
| Tier B | 2x               | 10 seconds   |
| Tier C | 1.5x             | 10 seconds   |

---

## 6. Storage Backend (RustFS) Integration Specs

### 6.1 RustFS Configuration

RustFS serves as the S3-compatible object storage backend for all backup data and large artifacts.

| Parameter             | Value                                    |
|-----------------------|------------------------------------------|
| Protocol              | S3-compatible REST API (HTTPS)           |
| Endpoint              | `https://rustfs.internal.erp.io`         |
| Region                | `us-east-1` (default)                    |
| Authentication        | Access Key + Secret Key (per service)    |
| Max Object Size       | 5 TiB (multipart upload)                |
| Part Size             | 64 MiB (for multipart uploads)           |
| Bucket Naming         | `dbaas-backups-{region}`                 |
| Object Path Pattern   | `{tenant_id}/{instance_id}/{backup_id}/{filename}` |
| Encryption            | Server-side SSE-S3 + client-side AES-256-GCM        |
| Lifecycle Policy      | Objects deleted per retention policy via lifecycle rules |

### 6.2 RustFS Bucket Structure

```
dbaas-backups-us-east-1/
├── tenant-abc123/
│   ├── inst-yugabyte-001/
│   │   ├── bk-20260224-020000/
│   │   │   ├── full-dump.zst.enc
│   │   │   └── metadata.json
│   │   └── bk-20260224-080000/
│   │       ├── incremental-001.zst.enc
│   │       └── metadata.json
│   └── inst-dragonfly-002/
│       └── bk-20260224-020000/
│           ├── rdb-snapshot.zst.enc
│           └── metadata.json
└── tenant-def456/
    └── ...
```

### 6.3 Upload Performance Specifications

| Metric                          | Target                   |
|---------------------------------|--------------------------|
| Single-part upload throughput   | > 100 MiB/s             |
| Multipart upload throughput     | > 500 MiB/s (8 parts)   |
| Download throughput             | > 500 MiB/s             |
| List objects latency (1000 obj) | < 200ms                 |
| Put object latency (< 1MB)     | < 50ms                  |
| Durability                      | 99.999999999% (11 nines)|
| Availability                    | 99.99%                  |

### 6.4 RustFS Client Configuration (Go)

```go
type RustFSConfig struct {
    Endpoint        string        `env:"RUSTFS_ENDPOINT" default:"https://rustfs.internal.erp.io"`
    AccessKey       string        `env:"RUSTFS_ACCESS_KEY"`
    SecretKey       string        `env:"RUSTFS_SECRET_KEY"`
    Region          string        `env:"RUSTFS_REGION" default:"us-east-1"`
    Bucket          string        `env:"RUSTFS_BUCKET" default:"dbaas-backups-us-east-1"`
    PartSize        int64         `env:"RUSTFS_PART_SIZE" default:"67108864"` // 64 MiB
    MaxRetries      int           `env:"RUSTFS_MAX_RETRIES" default:"3"`
    Timeout         time.Duration `env:"RUSTFS_TIMEOUT" default:"30m"`
    UseSSL          bool          `env:"RUSTFS_USE_SSL" default:"true"`
    ConcurrentParts int           `env:"RUSTFS_CONCURRENT_PARTS" default:"8"`
}
```

---

## Appendix: Configuration Reference

### Environment Variables

| Variable                     | Required | Default               | Description                           |
|------------------------------|----------|-----------------------|----------------------------------------|
| `DBAAS_API_PORT`             | No       | `3000`                | Node.js API listening port             |
| `DBAAS_GATEWAY_PORT`         | No       | `8090`                | Go Gateway listening port              |
| `DBAAS_REGISTRY_DSN`         | Yes      | N/A                   | YugabyteDB connection string           |
| `DBAAS_HASURA_ENDPOINT`      | Yes      | N/A                   | Hasura GraphQL endpoint                |
| `DBAAS_PULSAR_URL`           | Yes      | N/A                   | Apache Pulsar service URL              |
| `DBAAS_VICTORIA_ENDPOINT`    | Yes      | N/A                   | VictoriaMetrics write endpoint         |
| `DBAAS_AUTHENTIK_ISSUER`     | Yes      | N/A                   | Authentik OIDC issuer URL              |
| `RUSTFS_ENDPOINT`            | Yes      | N/A                   | RustFS endpoint URL                    |
| `RUSTFS_ACCESS_KEY`          | Yes      | N/A                   | RustFS access key                      |
| `RUSTFS_SECRET_KEY`          | Yes      | N/A                   | RustFS secret key                      |
