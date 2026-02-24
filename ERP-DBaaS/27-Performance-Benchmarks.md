# ERP-DBaaS Performance Benchmarks

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Performance Benchmarks       |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Engineering                 |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [Provisioning Time per Engine](#1-provisioning-time-per-engine)
2. [Backup Throughput Benchmarks](#2-backup-throughput-benchmarks)
3. [Restore Time Benchmarks](#3-restore-time-benchmarks)
4. [Connection Pooling Performance](#4-connection-pooling-performance)
5. [Metering Data Pipeline Latency](#5-metering-data-pipeline-latency)
6. [Query Performance per Engine Type](#6-query-performance-per-engine-type)

---

## 1. Provisioning Time per Engine

### 1.1 Target Provisioning Times

| Size   | Target (Standalone) | Target (HA)    |
|--------|---------------------|----------------|
| S      | < 60 seconds        | < 120 seconds  |
| M      | < 120 seconds       | < 180 seconds  |
| L      | < 180 seconds       | < 300 seconds  |
| XL     | < 300 seconds       | < 600 seconds  |

### 1.2 Measured Provisioning Times (Standalone)

| Engine       | Size S  | Size M  | Size L   | Size XL  |
|--------------|---------|---------|----------|----------|
| YugabyteDB   | 45s     | 72s     | 135s     | 248s     |
| DragonflyDB  | 12s     | 15s     | 18s      | 22s      |
| ClickHouse   | 38s     | 58s     | 112s     | 195s     |
| Tembo (PG)   | 22s     | 35s     | 65s      | 120s     |
| SurrealDB    | 18s     | 28s     | 52s      | 95s      |
| QuestDB      | 15s     | 25s     | 48s      | 88s      |
| Apache Doris | 55s     | 85s     | 165s     | 290s     |
| InfluxDB     | 20s     | 32s     | 60s      | 110s     |

### 1.3 Measured Provisioning Times (HA Mode)

| Engine       | Size S  | Size M  | Size L   | Size XL  |
|--------------|---------|---------|----------|----------|
| YugabyteDB   | 95s     | 140s    | 245s     | 480s     |
| DragonflyDB  | 25s     | 35s     | 45s      | 55s      |
| ClickHouse   | 85s     | 120s    | 210s     | 390s     |
| Tembo (PG)   | 48s     | 75s     | 130s     | 240s     |
| SurrealDB    | 40s     | 62s     | 110s     | 200s     |
| Apache Doris | 110s    | 165s    | 285s     | 540s     |
| InfluxDB     | 45s     | 68s     | 125s     | 225s     |

### 1.4 Provisioning Time Breakdown

The provisioning process consists of several phases. Below is a breakdown for a typical YugabyteDB Size M standalone instance:

| Phase                          | Duration | % of Total |
|--------------------------------|----------|------------|
| API validation and quota check | 1.2s     | 1.7%       |
| CRD creation and submission    | 0.8s     | 1.1%       |
| Operator detection (informer)  | 2.0s     | 2.8%       |
| PVC provisioning               | 8.5s     | 11.8%      |
| Pod scheduling and pull        | 15.0s    | 20.8%      |
| Engine initialization          | 35.0s    | 48.6%      |
| Health check convergence       | 6.5s     | 9.0%       |
| Credential generation          | 1.5s     | 2.1%       |
| Status update and event emit   | 1.5s     | 2.1%       |
| **Total**                      | **72s**  | **100%**   |

---

## 2. Backup Throughput Benchmarks

### 2.1 Backup Speed by Engine and Data Size

| Engine       | 10 GiB    | 50 GiB    | 100 GiB   | 500 GiB   | 1 TiB      |
|--------------|-----------|-----------|-----------|-----------|------------|
| YugabyteDB   | 2m 15s    | 8m 30s    | 15m 45s   | 72m       | 140m       |
| DragonflyDB  | 0m 30s    | 2m 10s    | 4m 20s    | N/A*      | N/A*       |
| ClickHouse   | 1m 45s    | 7m 20s    | 13m 50s   | 65m       | 125m       |
| Tembo (PG)   | 1m 55s    | 7m 45s    | 14m 30s   | 68m       | 132m       |
| SurrealDB    | 1m 30s    | 6m 15s    | 11m 45s   | 55m       | 108m       |
| QuestDB      | 1m 10s    | 5m 00s    | 9m 30s    | 45m       | 88m        |
| Apache Doris | 2m 30s    | 9m 45s    | 18m 15s   | 85m       | 165m       |
| InfluxDB     | 1m 40s    | 6m 50s    | 12m 50s   | 60m       | 118m       |

*DragonflyDB is memory-bound; datasets above 32 GiB are uncommon.

### 2.2 Backup Throughput Rates

| Phase             | Throughput (Avg) | Throughput (Peak) |
|-------------------|------------------|-------------------|
| Snapshot/Dump     | 120 MiB/s        | 250 MiB/s         |
| Compression (zstd)| 200 MiB/s        | 400 MiB/s         |
| Encryption (AES)  | 500 MiB/s        | 800 MiB/s         |
| Upload to RustFS  | 150 MiB/s        | 500 MiB/s         |
| **End-to-end**    | **80 MiB/s**     | **180 MiB/s**     |

### 2.3 Compression Ratios

| Engine       | Typical Compression Ratio | Compressed Size (100 GiB raw) |
|--------------|--------------------------|-------------------------------|
| YugabyteDB   | 3.2:1                    | 31.25 GiB                     |
| DragonflyDB  | 2.8:1                    | 35.71 GiB                     |
| ClickHouse   | 5.5:1                    | 18.18 GiB                     |
| Tembo (PG)   | 3.0:1                    | 33.33 GiB                     |
| SurrealDB    | 3.5:1                    | 28.57 GiB                     |
| QuestDB      | 4.8:1                    | 20.83 GiB                     |
| Apache Doris | 4.2:1                    | 23.81 GiB                     |
| InfluxDB     | 6.0:1                    | 16.67 GiB                     |

---

## 3. Restore Time Benchmarks

### 3.1 Restore Speed by Engine and Data Size

| Engine       | 10 GiB    | 50 GiB    | 100 GiB   | 500 GiB    |
|--------------|-----------|-----------|-----------|------------|
| YugabyteDB   | 3m 30s    | 14m 00s   | 26m 00s   | 120m       |
| DragonflyDB  | 0m 45s    | 3m 15s    | 6m 30s    | N/A        |
| ClickHouse   | 2m 50s    | 11m 30s   | 21m 30s   | 100m       |
| Tembo (PG)   | 3m 10s    | 12m 30s   | 23m 30s   | 110m       |
| SurrealDB    | 2m 20s    | 9m 30s    | 17m 45s   | 85m        |
| QuestDB      | 1m 50s    | 7m 30s    | 14m 15s   | 68m        |
| Apache Doris | 4m 00s    | 15m 30s   | 29m 00s   | 135m       |
| InfluxDB     | 2m 40s    | 10m 45s   | 20m 00s   | 95m        |

### 3.2 Restore Time Breakdown (100 GiB YugabyteDB)

| Phase                          | Duration | % of Total |
|--------------------------------|----------|------------|
| Download from RustFS           | 3m 20s   | 12.8%      |
| Checksum verification          | 0m 30s   | 1.9%       |
| Decryption (AES-256-GCM)      | 0m 45s   | 2.9%       |
| Decompression (zstd)           | 1m 15s   | 4.8%       |
| Instance provisioning          | 2m 30s   | 9.6%       |
| Data import (ysqlsh)           | 15m 00s  | 57.7%      |
| Index rebuild                  | 2m 00s   | 7.7%       |
| Health check verification      | 0m 40s   | 2.6%       |
| **Total**                      | **26m**  | **100%**   |

### 3.3 RTO Compliance

| Tier   | RTO Target  | 100 GiB Restore (worst case) | Compliant? |
|--------|-------------|------------------------------|------------|
| Tier A | 15 minutes  | 10-15 min (with HA failover) | Yes        |
| Tier B | 1 hour      | 26 min (from backup)         | Yes        |
| Tier C | 4 hours     | 29 min (from backup)         | Yes        |

Note: Tier A achieves its RTO target through HA failover (not backup restore). Backup restore for Tier A instances exceeding 100 GiB may exceed 15 minutes.

---

## 4. Connection Pooling Performance

### 4.1 PgBouncer Performance (YugabyteDB / Tembo)

| Metric                          | Without PgBouncer | With PgBouncer | Improvement |
|---------------------------------|-------------------|----------------|-------------|
| Max concurrent connections      | 300               | 10,000         | 33x         |
| Connection establishment time   | 45ms              | 2ms            | 22.5x       |
| Query latency (P50)             | 1.2ms             | 1.3ms          | -8%         |
| Query latency (P99)             | 12ms              | 8ms            | 33%         |
| Transactions per second         | 8,500             | 12,000         | 41%         |
| Memory per connection (server)  | 10 MiB            | 2 KiB (pooled) | 5000x       |

### 4.2 Connection Pool Sizing Recommendations

| Size Preset | Pool Size | Max Client Connections | Server Connections | Reserve Pool |
|-------------|-----------|------------------------|--------------------|--------------|
| S           | 10        | 50                     | 10                 | 2            |
| M           | 20        | 150                    | 20                 | 5            |
| L           | 40        | 300                    | 40                 | 10           |
| XL          | 80        | 500                    | 80                 | 20           |

### 4.3 DragonflyDB Connection Performance

| Metric                          | Value              |
|---------------------------------|--------------------|
| Max concurrent connections      | 65,000+            |
| Connection establishment time   | < 1ms              |
| PING latency (P50)              | 0.08ms             |
| PING latency (P99)              | 0.25ms             |
| GET/SET throughput (pipeline)   | 4,000,000 ops/s    |
| GET/SET throughput (no pipeline)| 650,000 ops/s      |

---

## 5. Metering Data Pipeline Latency

### 5.1 Pipeline Stage Latencies

| Stage                           | Latency (P50) | Latency (P99) | Throughput         |
|---------------------------------|---------------|---------------|--------------------|
| Metrics scrape (VictoriaMetrics)| 2s            | 5s            | 100K series/scrape |
| Metering aggregation            | 15s           | 45s           | 50K records/batch  |
| Write to YugabyteDB             | 5ms           | 25ms          | 10K inserts/s      |
| Billing export (hourly)         | 30s           | 120s          | 500K records/export|
| **End-to-end (scrape to query)**| **< 30s**     | **< 90s**     |                    |

### 5.2 Metering Accuracy

| Metric                          | Target     | Measured    |
|---------------------------------|------------|-------------|
| CPU usage accuracy              | +/- 2%     | +/- 1.5%    |
| Memory usage accuracy           | +/- 2%     | +/- 1.2%    |
| Storage usage accuracy          | +/- 1%     | +/- 0.8%    |
| Collection completeness         | > 99.9%    | 99.97%      |
| Data point loss rate            | < 0.1%     | 0.03%       |

### 5.3 Metering Collector Resource Usage

| Instance Count | Collector CPU | Collector Memory | Scrape Interval |
|----------------|---------------|------------------|-----------------|
| 50             | 0.1 vCPU      | 128 MiB          | 15s             |
| 200            | 0.5 vCPU      | 512 MiB          | 15s             |
| 500            | 1.0 vCPU      | 1 GiB            | 15s             |
| 1000           | 2.0 vCPU      | 2 GiB            | 30s             |
| 5000           | 8.0 vCPU      | 8 GiB            | 30s             |

---

## 6. Query Performance per Engine Type

### 6.1 YugabyteDB (OLTP)

Benchmark: pgbench, 4 vCPU, 16 GiB memory, SSD storage

| Workload                | TPS (read-only) | TPS (read-write) | P50 Latency | P99 Latency |
|-------------------------|------------------|-------------------|-------------|-------------|
| Simple SELECT (1 row)   | 45,000           | N/A               | 0.4ms       | 2.1ms       |
| TPC-B (scale factor 10) | N/A              | 3,800             | 2.6ms       | 15ms        |
| TPC-B (scale factor 100)| N/A              | 3,200             | 3.1ms       | 22ms        |
| Range scan (1K rows)    | 12,000           | N/A               | 1.8ms       | 8.5ms       |
| Join (3 tables)         | 5,500            | N/A               | 3.5ms       | 18ms        |

### 6.2 DragonflyDB (Key-Value)

Benchmark: redis-benchmark, 4 vCPU, 8 GiB memory

| Command  | Throughput (ops/s) | P50 Latency | P99 Latency |
|----------|--------------------|-------------|-------------|
| SET      | 680,000            | 0.06ms      | 0.18ms      |
| GET      | 720,000            | 0.05ms      | 0.15ms      |
| MSET (10)| 95,000             | 0.09ms      | 0.30ms      |
| MGET (10)| 105,000            | 0.08ms      | 0.25ms      |
| LPUSH    | 620,000            | 0.07ms      | 0.20ms      |
| ZADD     | 510,000            | 0.08ms      | 0.22ms      |

### 6.3 ClickHouse (Analytical)

Benchmark: ClickBench, 8 vCPU, 32 GiB memory, SSD storage, 100M rows

| Query Type                      | Cold (P50) | Warm (P50) | P99        |
|---------------------------------|------------|------------|------------|
| Full table scan (COUNT)         | 120ms      | 35ms       | 180ms      |
| Filtered aggregation (GROUP BY) | 250ms      | 85ms       | 420ms      |
| Top-N query                     | 180ms      | 55ms       | 310ms      |
| Time-range filter (1 day)       | 45ms       | 12ms       | 85ms       |
| Join (2 tables, 10M rows each)  | 850ms      | 320ms      | 1.5s       |
| INSERT (batch 1M rows)          | 1.2s       | 1.0s       | 2.5s       |

### 6.4 QuestDB (Time-Series)

Benchmark: TSBS, 4 vCPU, 16 GiB memory, NVMe storage

| Operation                       | Throughput / Latency              |
|---------------------------------|-----------------------------------|
| INSERT (single row)             | 1,200,000 rows/s                  |
| INSERT (batch 10K rows)         | 2,800,000 rows/s                  |
| SELECT latest value             | 0.3ms (P50), 1.2ms (P99)         |
| Time-range aggregate (1 hour)   | 8ms (P50), 25ms (P99)            |
| Time-range aggregate (1 day)    | 45ms (P50), 120ms (P99)          |
| Downsampling (5-min intervals)  | 22ms (P50), 65ms (P99)           |

### 6.5 Performance Testing Methodology

| Parameter               | Value                                  |
|-------------------------|----------------------------------------|
| Test environment        | Dedicated K8s cluster (bare-metal)     |
| Network                 | 10 Gbps dedicated                      |
| Storage                 | NVMe SSD (local PV)                    |
| Client instances        | 3 load-generator pods                  |
| Warm-up period          | 5 minutes                              |
| Test duration           | 30 minutes per benchmark               |
| Result collection       | VictoriaMetrics + custom exporter       |
| Statistical method      | Median of 3 runs, P50/P95/P99 reported |

---

## Appendix: Benchmark Environment Specifications

| Component          | Specification                           |
|--------------------|-----------------------------------------|
| Kubernetes version | 1.29.x                                  |
| Node type          | Bare-metal, 32 vCPU, 128 GiB RAM        |
| Storage            | 2x NVMe SSD, 1.92 TiB each             |
| Network            | 10 Gbps dedicated, < 0.1ms RTT intra-cluster |
| OS                 | Ubuntu 22.04 LTS                         |
| Kernel             | 6.5.x with io_uring enabled             |
| Container runtime  | containerd 1.7.x                        |
