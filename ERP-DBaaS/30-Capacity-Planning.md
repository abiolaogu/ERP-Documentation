# ERP-DBaaS Capacity Planning

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Capacity Planning            |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Engineering                 |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [Storage Sizing per Engine Type](#1-storage-sizing-per-engine-type)
2. [Memory Requirements per Size Preset](#2-memory-requirements-per-size-preset)
3. [CPU Allocation Guidelines](#3-cpu-allocation-guidelines)
4. [Network Bandwidth per Connection Count](#4-network-bandwidth-per-connection-count)
5. [Backup Storage Growth Projections](#5-backup-storage-growth-projections)
6. [Operator Resource Overhead](#6-operator-resource-overhead)

---

## 1. Storage Sizing per Engine Type

### 1.1 Storage Overhead Factors

Raw data size does not equal required storage. Each engine has overhead factors that must be accounted for during capacity planning.

| Engine       | Write Amplification | Index Overhead | Temp Space | Replication Factor | Total Multiplier |
|--------------|--------------------:|---------------:|-----------:|-------------------:|-----------------:|
| YugabyteDB   | 1.5x               | 30%            | 10%        | 3x (HA)           | ~6.0x            |
| DragonflyDB  | 1.0x               | 5%             | 5%         | 2x (HA)           | ~2.2x            |
| ClickHouse   | 1.2x               | 10%            | 20%        | 2x (HA)           | ~3.1x            |
| Tembo (PG)   | 1.4x               | 25%            | 15%        | 2x (HA)           | ~3.6x            |
| SurrealDB    | 1.3x               | 20%            | 10%        | 3x (TiKV)         | ~5.9x            |
| QuestDB      | 1.1x               | 5%             | 10%        | 1x (standalone)    | ~1.3x            |
| Apache Doris | 1.3x               | 15%            | 20%        | 3x (default)       | ~5.3x            |
| InfluxDB     | 1.2x               | 15%            | 10%        | 2x (HA)           | ~3.0x            |

### 1.2 Storage Sizing Formula

```
Required Storage = Raw Data Size x Total Multiplier + Headroom (20%)

Example (YugabyteDB, 100 GiB raw data, HA mode):
  = 100 GiB x 6.0 + (100 x 6.0 x 0.20)
  = 600 GiB + 120 GiB
  = 720 GiB total across 3 nodes
  = 240 GiB per node
```

### 1.3 Recommended Storage per Size Preset (with overhead)

| Size | YugabyteDB | DragonflyDB | ClickHouse | Tembo (PG) | SurrealDB | QuestDB | Doris  | InfluxDB |
|------|------------|-------------|------------|------------|-----------|---------|--------|----------|
| S    | 20 GiB     | 2 GiB       | 100 GiB    | 20 GiB     | 20 GiB    | 50 GiB  | 100 GiB| 50 GiB   |
| M    | 100 GiB    | 8 GiB       | 500 GiB    | 100 GiB    | 100 GiB   | 200 GiB | 500 GiB| 200 GiB  |
| L    | 500 GiB    | 16 GiB      | 2 TiB      | 500 GiB    | 500 GiB   | 1 TiB   | 2 TiB  | 1 TiB    |
| XL   | 1 TiB      | 64 GiB      | 5 TiB      | 1 TiB      | 1 TiB     | 2 TiB   | 5 TiB  | 2 TiB    |

### 1.4 Storage Growth Rate Estimation

| Workload Type              | Typical Growth Rate    | Reassessment Interval |
|----------------------------|------------------------|-----------------------|
| OLTP (YugabyteDB, Tembo)  | 5-15% per month        | Monthly               |
| Analytics (ClickHouse, Doris)| 20-40% per month     | Bi-weekly             |
| Caching (DragonflyDB)     | Stable (size-bound)    | Quarterly             |
| Time-Series (QuestDB, InfluxDB)| 10-30% per month  | Monthly               |
| Document (SurrealDB)      | 5-20% per month        | Monthly               |

---

## 2. Memory Requirements per Size Preset

### 2.1 Base Memory Allocations

| Size | YugabyteDB         | DragonflyDB | ClickHouse | Tembo (PG) | SurrealDB | QuestDB | Doris          | InfluxDB |
|------|---------------------|-------------|------------|------------|-----------|---------|----------------|----------|
| S    | 2 GiB (TS) + 1 GiB (M) | 1 GiB  | 4 GiB      | 2 GiB      | 2 GiB     | 2 GiB   | 4 GiB (FE+BE)  | 2 GiB    |
| M    | 4 GiB (TS) + 2 GiB (M) | 4 GiB  | 8 GiB      | 4 GiB      | 4 GiB     | 4 GiB   | 8 GiB (FE+BE)  | 4 GiB    |
| L    | 16 GiB (TS) + 4 GiB (M)| 8 GiB  | 32 GiB     | 16 GiB     | 16 GiB    | 16 GiB  | 32 GiB (FE+BE) | 16 GiB   |
| XL   | 32 GiB (TS) + 8 GiB (M)| 32 GiB | 64 GiB     | 32 GiB     | 32 GiB    | 32 GiB  | 64 GiB (FE+BE) | 32 GiB   |

TS = TServer, M = Master, FE = Frontend, BE = Backend

### 2.2 Memory Sizing Guidelines per Engine

**YugabyteDB:**
- Shared buffers: 25% of TServer memory
- OS page cache: ~50% of remaining memory used for tablet block cache
- Rule of thumb: 1 GiB memory per 10 GiB of frequently accessed data

**DragonflyDB:**
- Memory equals the maximum dataset size plus 10% overhead for fragmentation
- maxmemory-policy determines eviction behavior when limit is reached
- Rule of thumb: 1.1x the expected dataset size

**ClickHouse:**
- max_memory_usage per query: 50% of total memory
- max_bytes_before_external_sort: 25% of total memory
- Rule of thumb: 1 GiB memory per 100 GiB of data for efficient analytical queries

**Tembo (PostgreSQL):**
- shared_buffers: 25% of total memory
- effective_cache_size: 75% of total memory
- work_mem: total memory / (max_connections * 4)
- Rule of thumb: 1 GiB memory per 20 GiB of actively queried data

### 2.3 Memory Pressure Indicators

| Indicator                    | Warning Threshold | Critical Threshold | Action Required          |
|------------------------------|-------------------|--------------------|--------------------------|
| RSS / Limit ratio            | > 80%             | > 90%              | Scale up memory          |
| OOM kills (last 24h)        | >= 1              | >= 3               | Immediate scale up       |
| Swap usage                   | > 0               | > 100 MiB          | Investigate memory leak  |
| Page cache hit ratio         | < 90%             | < 80%              | Increase memory or cache |
| PgBouncer wait queue         | > 10              | > 50               | Increase pool or memory  |

---

## 3. CPU Allocation Guidelines

### 3.1 CPU Allocations per Size Preset

| Size | Base vCPU | Request (guaranteed) | Limit (burstable) | Burst Ratio |
|------|-----------|----------------------|--------------------|-------------|
| S    | 1         | 1.0                  | 2.0                | 2x          |
| M    | 2         | 2.0                  | 4.0                | 2x          |
| L    | 4         | 4.0                  | 6.0                | 1.5x        |
| XL   | 8         | 8.0                  | 10.0               | 1.25x       |

### 3.2 CPU Sizing Guidelines per Workload Type

| Workload Type  | CPU Utilization Target | Sizing Rule                              |
|----------------|------------------------|------------------------------------------|
| OLTP           | 50-70% sustained       | 1 vCPU per 1,000 TPS (simple queries)   |
| OLAP           | 80-95% during queries  | 1 vCPU per 10 concurrent analytical queries |
| Caching        | 30-50% sustained       | 1 vCPU per 200K ops/s                    |
| Time-Series    | 40-60% sustained       | 1 vCPU per 500K inserts/s                |
| Mixed          | 50-70% sustained       | Sum of OLTP + OLAP requirements           |

### 3.3 CPU Scaling Decision Matrix

| Current CPU Utilization | Duration           | Recommendation                      |
|-------------------------|--------------------|--------------------------------------|
| < 30%                   | > 7 days           | Consider scaling down                |
| 30-70%                  | Sustained          | Optimal range, no action             |
| 70-85%                  | > 1 hour           | Plan to scale up within 1 week       |
| 85-95%                  | > 15 minutes       | Scale up within 24 hours             |
| > 95%                   | > 5 minutes        | Scale up immediately                 |

### 3.4 Per-Engine CPU Specifics

| Engine       | CPU-Sensitive Operations                   | Recommended CPU:Memory Ratio |
|--------------|---------------------------------------------|------------------------------|
| YugabyteDB   | Compaction, tablet splitting, Raft consensus| 1:4 (1 vCPU per 4 GiB RAM)  |
| DragonflyDB  | Serialization, snapshot creation            | 1:8 (1 vCPU per 8 GiB RAM)  |
| ClickHouse   | Merge operations, query execution           | 1:4 (1 vCPU per 4 GiB RAM)  |
| Tembo (PG)   | Index scans, VACUUM, WAL generation         | 1:4 (1 vCPU per 4 GiB RAM)  |
| QuestDB      | Data ingestion, column compression          | 1:4 (1 vCPU per 4 GiB RAM)  |
| Apache Doris | Compaction, query execution                 | 1:4 (1 vCPU per 4 GiB RAM)  |

---

## 4. Network Bandwidth per Connection Count

### 4.1 Bandwidth Estimation per Engine

| Engine       | Bandwidth per Connection (Avg) | Bandwidth per Connection (Peak) | Notes                    |
|--------------|-------------------------------:|--------------------------------:|--------------------------|
| YugabyteDB   | 0.5 Mbps                      | 5 Mbps                          | OLTP queries             |
| DragonflyDB  | 0.1 Mbps                      | 2 Mbps                          | Small key-value pairs    |
| ClickHouse   | 2 Mbps                        | 50 Mbps                         | Large result sets        |
| Tembo (PG)   | 0.5 Mbps                      | 10 Mbps                         | Mixed workloads          |
| QuestDB      | 1 Mbps                        | 20 Mbps                         | Time-series bulk inserts |
| Apache Doris | 2 Mbps                        | 40 Mbps                         | Analytical queries       |

### 4.2 Total Bandwidth Requirements

| Size Preset | Max Connections | Avg Bandwidth | Peak Bandwidth | Recommended NIC  |
|-------------|-----------------|---------------|----------------|------------------|
| S           | 50              | 25 Mbps       | 250 Mbps       | 1 Gbps           |
| M           | 150             | 75 Mbps       | 750 Mbps       | 1 Gbps           |
| L           | 300             | 150 Mbps      | 1.5 Gbps       | 2.5 Gbps         |
| XL          | 500             | 250 Mbps      | 2.5 Gbps       | 5 Gbps           |

### 4.3 Replication Bandwidth

| Engine       | Replication Bandwidth (steady) | Replication Bandwidth (catch-up) |
|--------------|-------------------------------:|---------------------------------:|
| YugabyteDB   | 10-50 Mbps                     | 200-500 Mbps                    |
| DragonflyDB  | 5-20 Mbps                      | 100-300 Mbps                    |
| ClickHouse   | 20-100 Mbps                    | 500 Mbps - 1 Gbps              |
| Tembo (PG)   | 10-50 Mbps                     | 200-500 Mbps                    |

### 4.4 Cross-Region Bandwidth Planning

| Number of Instances | Replication Mode | Required Cross-Region Bandwidth |
|---------------------|------------------|---------------------------------|
| 10                  | Async            | 100 Mbps                        |
| 50                  | Async            | 500 Mbps                        |
| 100                 | Async            | 1 Gbps                          |
| 50                  | Sync             | 1 Gbps                          |
| 100                 | Sync             | 2 Gbps                          |

---

## 5. Backup Storage Growth Projections

### 5.1 Backup Storage Formula

```
Monthly Backup Storage = (Full Backup Size x Full Frequency x Retention)
                       + (Incremental Size x Incr. Frequency x Retention)
                       + (WAL Size x WAL Retention)

Example (Tier B, 100 GiB YugabyteDB instance):
  Full backup (compressed): 33 GiB (3:1 ratio)
  Full frequency: Daily, retention: 30 days
  Incremental (10% of full): 3.3 GiB every 6h, retention: 7 days
  WAL: 2 GiB/day, retention: 3 days

  Monthly = (33 x 30) + (3.3 x 4 x 7) + (2 x 3)
          = 990 + 92.4 + 6
          = 1,088.4 GiB per instance
```

### 5.2 Projected Backup Storage by Tenant Tier

| Tier   | Avg Instances | Avg Data/Instance | Monthly Backup/Instance | Total Monthly  |
|--------|---------------|-------------------|------------------------|----------------|
| Tier A | 35            | 200 GiB           | 2.5 TiB                | 87.5 TiB       |
| Tier B | 15            | 100 GiB           | 1.1 TiB                | 16.5 TiB       |
| Tier C | 3             | 30 GiB            | 150 GiB                | 450 GiB        |

### 5.3 Growth Projections (12-Month Forecast)

| Month | Total Instances | Avg Data Growth | New Backup Storage | Cumulative Backup Storage |
|-------|-----------------|-----------------|--------------------|--------------------------:|
| 1     | 50              | Baseline        | 15 TiB             | 15 TiB                    |
| 3     | 80              | +15%            | 25 TiB             | 55 TiB                    |
| 6     | 130             | +30%            | 45 TiB             | 140 TiB                   |
| 9     | 180             | +45%            | 68 TiB             | 280 TiB                   |
| 12    | 250             | +60%            | 95 TiB             | 480 TiB                   |

### 5.4 Backup Storage Optimization Strategies

| Strategy                        | Storage Savings | Implementation Complexity |
|---------------------------------|-----------------|---------------------------|
| Increase compression level      | 10-25%          | Low                       |
| Deduplication across backups    | 30-50%          | High                      |
| Shorter retention for Tier C    | 15-20%          | Low                       |
| Incremental-only after full     | 40-60%          | Medium                    |
| Cold storage tiering (>30 days) | 20-30% cost     | Medium                    |

---

## 6. Operator Resource Overhead

### 6.1 Control Plane Resource Requirements

| Component              | Replicas | CPU Request | CPU Limit | Memory Request | Memory Limit |
|------------------------|----------|-------------|-----------|----------------|--------------|
| Go Gateway             | 3        | 0.5         | 1.0       | 256 MiB        | 512 MiB      |
| Node.js dbaas-api      | 3        | 0.5         | 1.0       | 512 MiB        | 1 GiB        |
| Hasura GraphQL         | 2        | 0.5         | 1.0       | 512 MiB        | 1 GiB        |
| Frontend (Nginx)       | 2        | 0.1         | 0.25      | 64 MiB         | 128 MiB      |
| Backup Controller      | 2        | 0.25        | 0.5       | 256 MiB        | 512 MiB      |
| Credential Manager     | 2        | 0.1         | 0.25      | 128 MiB        | 256 MiB      |
| Metering Collector     | 2        | 0.5         | 1.0       | 512 MiB        | 1 GiB        |

### 6.2 Per-Engine Operator Resource Requirements

| Operator               | Replicas | CPU Request | CPU Limit | Memory Request | Memory Limit |
|------------------------|----------|-------------|-----------|----------------|--------------|
| YugabyteDB Operator    | 2        | 0.25        | 0.5       | 256 MiB        | 512 MiB      |
| DragonflyDB Operator   | 2        | 0.1         | 0.25      | 128 MiB        | 256 MiB      |
| ClickHouse Operator    | 2        | 0.25        | 0.5       | 256 MiB        | 512 MiB      |
| Tembo Operator         | 2        | 0.25        | 0.5       | 256 MiB        | 512 MiB      |
| SurrealDB Operator     | 2        | 0.1         | 0.25      | 128 MiB        | 256 MiB      |
| QuestDB Operator       | 2        | 0.1         | 0.25      | 128 MiB        | 256 MiB      |
| Apache Doris Operator  | 2        | 0.25        | 0.5       | 256 MiB        | 512 MiB      |
| InfluxDB Operator      | 2        | 0.1         | 0.25      | 128 MiB        | 256 MiB      |

### 6.3 Total Control Plane Overhead

```
Total CPU Requests:
  Gateway:   3 x 0.5  = 1.5 vCPU
  API:       3 x 0.5  = 1.5 vCPU
  Hasura:    2 x 0.5  = 1.0 vCPU
  Frontend:  2 x 0.1  = 0.2 vCPU
  Backup:    2 x 0.25 = 0.5 vCPU
  CredMgr:   2 x 0.1  = 0.2 vCPU
  Metering:  2 x 0.5  = 1.0 vCPU
  Operators: 16 x 0.18 (avg) = 2.88 vCPU
  ─────────────────────────────
  Total: ~8.78 vCPU

Total Memory Requests:
  Gateway:   3 x 256 MiB  = 768 MiB
  API:       3 x 512 MiB  = 1,536 MiB
  Hasura:    2 x 512 MiB  = 1,024 MiB
  Frontend:  2 x 64 MiB   = 128 MiB
  Backup:    2 x 256 MiB  = 512 MiB
  CredMgr:   2 x 128 MiB  = 256 MiB
  Metering:  2 x 512 MiB  = 1,024 MiB
  Operators: 16 x 192 MiB (avg) = 3,072 MiB
  ───────────────────────────────
  Total: ~8.32 GiB
```

### 6.4 Operator Scaling Considerations

| Instance Count | Operator Replicas | Operator CPU (total) | Operator Memory (total) | Notes                        |
|----------------|-------------------|----------------------|-------------------------|------------------------------|
| < 50           | 2 per operator    | 2.88 vCPU            | 3 GiB                  | Default sizing               |
| 50-200         | 2 per operator    | 2.88 vCPU            | 3 GiB                  | Increase memory to 512 MiB each |
| 200-500        | 3 per operator    | 6.5 vCPU             | 7.5 GiB                | Add leader election          |
| 500-1000       | 3 per operator    | 10 vCPU              | 12 GiB                 | Shard by namespace           |
| > 1000         | 5 per operator    | 18 vCPU              | 20 GiB                 | Dedicated operator nodes     |

---

## Appendix: Capacity Planning Worksheet

Use this worksheet to estimate total cluster requirements for a new deployment.

```
Step 1: Count instances by engine and size
  YugabyteDB:  S:___ M:___ L:___ XL:___
  DragonflyDB: S:___ M:___ L:___ XL:___
  ClickHouse:  S:___ M:___ L:___ XL:___
  (repeat for each engine)

Step 2: Sum CPU, Memory, Storage from size preset tables

Step 3: Add control plane overhead (~9 vCPU, ~9 GiB)

Step 4: Add 20% headroom for scheduling and burst

Step 5: Calculate backup storage (Section 5)

Step 6: Validate against cluster node capacity:
  Total nodes = ceil(Total CPU / Node CPU * 0.8)
  Verify: Total Memory <= Total Nodes x Node Memory x 0.8
  Verify: Total Storage <= Total Nodes x Node Storage x 0.8
```
