# ERP-Observability Capacity Planning Guide

> **Document ID:** ERP-OBS-CAP-030
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [14-Technical-Specifications.md](./14-Technical-Specifications.md), [27-Performance-Benchmarks.md](./27-Performance-Benchmarks.md), [28-Multi-Tenancy-Guide.md](./28-Multi-Tenancy-Guide.md)

---

## 1. Overview

This document provides a comprehensive capacity planning guide for the ERP-Observability platform. It includes sizing calculators, storage estimation formulas, resource scaling guidelines, and growth projection models to ensure the observability infrastructure scales smoothly alongside ERP module adoption and tenant growth.

### Capacity Planning Principles

| Principle | Description |
|-----------|-------------|
| **Plan for 2x headroom** | Always provision 2x current usage to absorb spikes and growth |
| **Scale horizontally first** | Add replicas before scaling up individual nodes |
| **Monitor the monitors** | Track observability platform resource usage with self-monitoring |
| **Tiered storage** | Use fast NVMe for hot data, object storage (RustFS) for warm/cold |
| **Retention drives cost** | Storage is the largest cost driver; optimize retention per tier |
| **Compress aggressively** | VictoriaMetrics and Quickwit both achieve 10-14x compression |

---

## 2. Sizing Calculator

### 2.1 Input Variables

| Variable | Symbol | Description | How to Estimate |
|----------|--------|-------------|-----------------|
| Number of ERP modules | `M` | Active modules emitting telemetry | Count of deployed ERP-* services |
| Number of tenants | `T` | Active tenants across all modules | Count from tenant-api |
| Metrics per module | `Mpm` | Unique metric series per module | Default: 200 base + 10 per tenant |
| Metric label cardinality | `Lc` | Average label combinations per metric | Default: 10 |
| Scrape interval | `Si` | Metric collection interval in seconds | Default: 15s |
| Logs per second (per module) | `Lps` | Log ingestion rate per module | Default: 5,000 logs/sec |
| Average log size | `Ls` | Average uncompressed log entry size | Default: 512 bytes |
| Traces per second (per module) | `Tps` | Span ingestion rate per module | Default: 2,500 spans/sec |
| Average span size | `Ss` | Average uncompressed span size | Default: 1,024 bytes |
| Metrics retention | `Rm` | How long metrics are stored | Default: 30 days |
| Logs retention | `Rl` | How long logs are stored | Default: 14 days |
| Traces retention | `Rt` | How long traces are stored | Default: 7 days |
| Compression ratio (metrics) | `Cm` | VictoriaMetrics compression | Default: 14:1 |
| Compression ratio (logs) | `Cl` | Quickwit log compression | Default: 12:1 |
| Compression ratio (traces) | `Ct` | Quickwit trace compression | Default: 10:1 |

### 2.2 Formulas

#### Active Metric Series

```
Total Active Series = M * (Mpm + (T * 10)) * Lc

Example (10 modules, 50 tenants):
= 10 * (200 + (50 * 10)) * 10
= 10 * 700 * 10
= 70,000 active series
```

#### Metric Data Points Per Day

```
Data Points/Day = Total_Active_Series * (86400 / Si)

Example:
= 70,000 * (86400 / 15)
= 70,000 * 5,760
= 403,200,000 data points/day (~403M)
```

#### Metric Storage (Compressed)

```
Metric Storage = Data_Points/Day * Rm * 0.8 bytes / Cm

Example (30-day retention):
= 403,200,000 * 30 * 0.8 / 14
= 691,200,000 bytes
= ~0.64 GB

Note: Includes index overhead. Real-world factor: multiply by 1.3
Adjusted: 0.64 * 1.3 = ~0.84 GB for 30 days
```

#### Total Log Volume Per Day

```
Log Volume/Day (raw) = M * Lps * 86400 * Ls

Example (10 modules, 5K logs/sec each):
= 10 * 5,000 * 86,400 * 512
= 2,211,840,000,000 bytes
= ~2.01 TB/day (raw)

Log Storage/Day (compressed) = Log_Volume/Day / Cl
= 2.01 TB / 12
= ~171 GB/day (compressed)
```

#### Total Log Storage

```
Total Log Storage = Log_Storage/Day * Rl

Example (14-day retention):
= 171 GB * 14
= ~2.39 TB
```

#### Total Trace Volume Per Day

```
Trace Volume/Day (raw) = M * Tps * 86400 * Ss

Example (10 modules, 2.5K spans/sec each):
= 10 * 2,500 * 86,400 * 1,024
= 2,211,840,000,000 bytes
= ~2.01 TB/day (raw)

Trace Storage/Day (compressed) = Trace_Volume/Day / Ct
= 2.01 TB / 10
= ~206 GB/day (compressed)
```

#### Total Trace Storage

```
Total Trace Storage = Trace_Storage/Day * Rt

Example (7-day retention):
= 206 GB * 7
= ~1.44 TB
```

### 2.3 Quick Reference Sizing Table

| Deployment Size | Modules | Tenants | Active Series | Metrics Storage (30d) | Log Storage (14d) | Trace Storage (7d) | Total Storage |
|----------------|---------|---------|---------------|----------------------|-------------------|-------------------|---------------|
| **Tiny** | 3 | 5 | 7,500 | 0.1 GB | 72 GB | 43 GB | ~115 GB |
| **Small** | 5 | 10 | 15,000 | 0.2 GB | 120 GB | 72 GB | ~192 GB |
| **Medium** | 10 | 50 | 70,000 | 0.8 GB | 2.4 TB | 1.4 TB | ~3.8 TB |
| **Large** | 15 | 100 | 225,000 | 2.6 GB | 3.6 TB | 2.2 TB | ~5.8 TB |
| **Enterprise** | 20+ | 200+ | 600,000+ | 7.0 GB | 4.8 TB+ | 2.9 TB+ | ~7.7 TB+ |

---

## 3. Storage Estimates by Component

### 3.1 VictoriaMetrics Storage

VictoriaMetrics achieves exceptional compression. Storage is primarily driven by active series count and retention period.

| Factor | Impact on Storage |
|--------|-------------------|
| Active series count | Linear: 2x series = 2x storage |
| Retention period | Linear: 2x retention = 2x storage |
| Scrape interval | Inverse: 30s vs 15s = 0.5x data points |
| Label cardinality | Linear: more labels = more series |
| Deduplication | -10% with `dedup.minScrapeInterval=15s` |
| Downsampling | -70% for data older than 7 days (with recording rules) |

**Storage Formula:**

```
VM Storage (GB) = Active_Series * (86400/Si) * Retention_Days * 0.8 bytes * 1.3 / (Cm * 1e9)

Detailed Example (Medium deployment):
= 70,000 * 5,760 * 30 * 0.8 * 1.3 / (14 * 1,000,000,000)
= 70,000 * 5,760 * 30 * 1.04 / 14,000,000,000
= 12,579,840,000 / 14,000,000,000
= ~0.9 GB
```

**NVMe vs Object Storage Split:**

| Data Age | Storage Tier | Percentage |
|----------|-------------|------------|
| 0-7 days | NVMe (fast) | 100% of queries |
| 7-30 days | NVMe (warm) | 30% of queries |
| 30-90 days | RustFS (cold) | 5% of queries |
| 90-365 days | RustFS (archive) | < 1% of queries |

### 3.2 Quickwit Storage (Logs + Traces)

Quickwit stores data in splits on S3-compatible object storage (RustFS), with local caches on NVMe for fast access.

**Log Storage Formula:**

```
Quickwit Log Storage (GB) = M * Lps * 86400 * Ls * Rl / (Cl * 1e9)

Medium deployment:
= 10 * 5,000 * 86,400 * 512 * 14 / (12 * 1,000,000,000)
= 30,965,760,000,000 / 12,000,000,000
= ~2,580 GB = ~2.5 TB
```

**Trace Storage Formula:**

```
Quickwit Trace Storage (GB) = M * Tps * 86400 * Ss * Rt / (Ct * 1e9)

Medium deployment:
= 10 * 2,500 * 86,400 * 1,024 * 7 / (10 * 1,000,000,000)
= 15,482,880,000,000 / 10,000,000,000
= ~1,548 GB = ~1.5 TB
```

**Local Cache Requirements:**

| Component | Cache Purpose | Recommended Size |
|-----------|--------------|-----------------|
| Indexer split cache | Buffers before upload | 10 GB NVMe |
| Searcher split footer cache | Split metadata | 1 GB RAM |
| Searcher fast field cache | Filter/sort acceleration | 10 GB RAM |
| Searcher split cache | Recently accessed splits | 50 GB NVMe |

### 3.3 RustFS Object Storage

RustFS serves as the primary long-term storage backend for both VictoriaMetrics snapshots and Quickwit splits.

**Total RustFS Storage:**

```
RustFS Total = VM_Cold_Storage + QW_Log_Storage + QW_Trace_Storage + VM_Snapshots

Medium deployment:
= 0 GB (VM cold, if <30d retention) + 2,500 GB + 1,500 GB + 50 GB (daily snapshots * 7d)
= ~4,050 GB = ~4 TB
```

**RustFS Sizing Recommendations:**

| Deployment | Raw Capacity | With Replication (3x) | IOPS Required |
|-----------|-------------|----------------------|---------------|
| Tiny | 200 GB | 600 GB | 100 |
| Small | 500 GB | 1.5 TB | 500 |
| Medium | 4 TB | 12 TB | 2,000 |
| Large | 8 TB | 24 TB | 5,000 |
| Enterprise | 20+ TB | 60+ TB | 10,000+ |

### 3.4 YugabyteDB Storage

YugabyteDB stores configuration, tenant metadata, alert state, and dashboard metadata.

| Data Category | Estimated Size |
|--------------|---------------|
| Tenant configurations | ~1 KB per tenant |
| Alert rules | ~2 KB per rule, ~50 rules per module |
| Alert history | ~500 bytes per event, ~1000 events/day |
| Dashboard metadata | ~10 KB per dashboard |
| Audit logs | ~200 bytes per entry, ~10,000 entries/day |
| API keys | ~500 bytes per key |
| **Total (Medium, 30d)** | **~2-5 GB** |

### 3.5 DragonflyDB Memory

DragonflyDB caches query results, session state, and rate limiting counters.

| Cache Type | Estimated Memory |
|-----------|-----------------|
| Grafana query cache (5m TTL) | 500 MB - 2 GB |
| Tenant configuration cache | 50 MB |
| Rate limiter state | 100 MB |
| Session tokens | 200 MB |
| **Total (Medium)** | **~1-4 GB** |

---

## 4. CPU and Memory Scaling Guidelines

### 4.1 CPU Scaling

#### VictoriaMetrics

```
vminsert CPU = ceil(Ingestion_Rate / 250,000) cores
vmselect CPU = ceil(Concurrent_Queries * Avg_Series_Per_Query / 100,000) cores
vmstorage CPU = ceil(Active_Series / 500,000) * 2 cores (for compaction)

Medium deployment (70K series, ~400M points/day):
vminsert:  ceil(403M / 86400 / 250,000) = ceil(0.019) = 1 core (minimum 2 recommended)
vmselect:  ceil(10 * 1,000 / 100,000) = 1 core (minimum 2 recommended)
vmstorage: ceil(70,000 / 500,000) * 2 = 2 cores (minimum 4 recommended)
```

#### Quickwit

```
Indexer CPU = ceil(Total_Ingestion_Bytes_Per_Sec / 50_MB_per_core)
Searcher CPU = ceil(Concurrent_Searches * Avg_Splits_Scanned / 10)

Medium deployment:
Indexer: ceil((50,000 logs * 512 + 25,000 spans * 1024) / 50,000,000) = ceil(1.02) = 2 cores (min 4 rec)
Searcher: ceil(20 * 5 / 10) = 10 cores (across replicas, min 4 per replica)
```

#### OTel Collector

```
Collector CPU = ceil((Metrics_Rate + Logs_Rate * 10 + Spans_Rate * 10) / 500,000)

Medium deployment:
= ceil((4,667 + 50,000 * 10 + 25,000 * 10) / 500,000)
= ceil(1.51) = 2 cores (minimum 4 recommended for mixed workloads)
```

### 4.2 Memory Scaling

#### VictoriaMetrics

```
vmstorage Memory = Active_Series * 1 KB (for cache) + 4 GB (base overhead)

Medium deployment:
= 70,000 * 1,024 + 4,294,967,296
= 71,680,000 + 4,294,967,296
= ~4.4 GB (recommend 8 GB for headroom)
```

**Key Factor:** The `-memory.allowedPercent` parameter controls how much RAM VictoriaMetrics uses for caching. At 80%, a node with 16 GB RAM allocates 12.8 GB for cache, dramatically improving query performance.

#### Quickwit

```
Indexer Memory = Number_of_Pipelines * Pipeline_Memory + Heap_Size
Searcher Memory = Fast_Field_Cache + Split_Footer_Cache + Query_Buffer

Medium deployment:
Indexer:  2 pipelines * 2 GB + 8 GB heap = 12 GB (recommend 16 GB)
Searcher: 10 GB + 1 GB + 2 GB = 13 GB (recommend 16 GB)
```

#### OTel Collector

```
Collector Memory = Batch_Buffer + Send_Queue_Buffer + Processing_Overhead
= (Batch_Size * Avg_Entry_Size) + (Queue_Size * Avg_Entry_Size) + 2 GB

Medium deployment:
= (10,000 * 1 KB) + (10,000 * 1 KB) + 2 GB
= 10 MB + 10 MB + 2 GB
= ~2 GB (recommend 4 GB with memory_limiter)
```

### 4.3 Memory Scaling Summary

| Component | Formula | Tiny | Small | Medium | Large | Enterprise |
|-----------|---------|------|-------|--------|-------|------------|
| vmstorage | Series * 1KB + 4GB | 4.1 GB | 4.2 GB | 4.4 GB | 8 GB | 16 GB |
| vminsert | 1 GB per 250K samples/sec | 1 GB | 1 GB | 2 GB | 4 GB | 8 GB |
| vmselect | 2 GB per 10 concurrent queries | 2 GB | 2 GB | 4 GB | 8 GB | 16 GB |
| QW Indexer | Pipelines * 2GB + heap | 6 GB | 8 GB | 16 GB | 32 GB | 64 GB |
| QW Searcher | Cache + buffers | 6 GB | 8 GB | 16 GB | 32 GB | 64 GB |
| OTel Collector | Batch + queue + 2GB | 2 GB | 2 GB | 4 GB | 8 GB | 16 GB |
| Grafana | 512MB base + 256MB per 100 users | 1 GB | 1 GB | 2 GB | 4 GB | 8 GB |
| DragonflyDB | Cache data + 512MB overhead | 1 GB | 2 GB | 4 GB | 8 GB | 16 GB |
| YugabyteDB | 1GB per tablet peer | 2 GB | 4 GB | 8 GB | 16 GB | 32 GB |

---

## 5. Network Bandwidth Requirements

### 5.1 Ingestion Bandwidth

```
Metrics Bandwidth = Ingestion_Rate * Avg_Sample_Size (with labels, ~200 bytes per sample)
Logs Bandwidth = Log_Rate * Avg_Log_Size
Traces Bandwidth = Span_Rate * Avg_Span_Size

Medium deployment:
Metrics: 4,667 samples/sec * 200 bytes = ~0.9 MB/sec = ~7.5 Mbps
Logs:    50,000 logs/sec * 512 bytes = ~24.4 MB/sec = ~195 Mbps
Traces:  25,000 spans/sec * 1,024 bytes = ~24.4 MB/sec = ~195 Mbps

Total ingestion: ~400 Mbps
```

### 5.2 Internal Communication Bandwidth

| Path | Estimated Bandwidth | Protocol |
|------|-------------------|----------|
| OTel Collector -> VictoriaMetrics | 10-50 Mbps | Prometheus Remote Write (HTTP) |
| OTel Collector -> Quickwit | 200-500 Mbps | OTLP gRPC |
| Quickwit Indexer -> RustFS | 100-300 Mbps | S3 HTTP PUT |
| Quickwit Searcher -> RustFS | 50-200 Mbps | S3 HTTP GET |
| Grafana -> VictoriaMetrics | 10-50 Mbps | PromQL HTTP |
| Grafana -> Quickwit | 10-50 Mbps | Search API HTTP |
| Gateway -> APIs | 5-20 Mbps | HTTP/gRPC |

### 5.3 Total Network Requirements

| Deployment | Ingestion | Internal | Client (UI) | Total Recommended |
|-----------|-----------|----------|-------------|-------------------|
| Tiny | 50 Mbps | 100 Mbps | 10 Mbps | 1 Gbps |
| Small | 100 Mbps | 300 Mbps | 20 Mbps | 1 Gbps |
| Medium | 400 Mbps | 1 Gbps | 50 Mbps | 5 Gbps |
| Large | 800 Mbps | 3 Gbps | 100 Mbps | 10 Gbps |
| Enterprise | 2+ Gbps | 8+ Gbps | 200+ Mbps | 25 Gbps |

---

## 6. Retention Policy Impact on Storage

### 6.1 Storage vs Retention Matrix

For a Medium deployment (10 modules, 50 tenants):

| Retention | Metrics Storage | Log Storage | Trace Storage | Total |
|-----------|----------------|-------------|---------------|-------|
| 3 days | 0.09 GB | 513 GB | 618 GB | ~1.1 TB |
| 7 days | 0.21 GB | 1.2 TB | 1.4 TB | ~2.6 TB |
| 14 days | 0.42 GB | 2.4 TB | 2.9 TB | ~5.3 TB |
| 30 days | 0.9 GB | 5.1 TB | 6.2 TB | ~11.3 TB |
| 90 days | 2.7 GB | 15.4 TB | 18.5 TB | ~34 TB |
| 365 days | 11 GB | 62.4 TB | 75.2 TB | ~138 TB |

### 6.2 Cost-Optimized Retention Strategy

| Data Type | Hot (NVMe) | Warm (SSD) | Cold (RustFS) | Archive |
|-----------|-----------|-----------|---------------|---------|
| **Metrics** | 0-7 days | 7-30 days | 30-90 days | 90-365 days (downsampled) |
| **Logs** | 0-3 days | 3-14 days | 14-90 days | - (delete) |
| **Traces** | 0-1 day | 1-7 days | 7-30 days | - (delete) |

**Downsampling Strategy for Long-Term Metrics:**

```yaml
# VictoriaMetrics recording rules for downsampling
groups:
  - name: downsample_5m
    interval: 5m
    rules:
      # Keep 5-minute averages for metrics older than 7 days
      - record: downsampled:5m:erp_http_requests_total:rate
        expr: rate(erp_http_requests_total[5m])
      - record: downsampled:5m:erp_http_request_duration:p99
        expr: histogram_quantile(0.99, rate(erp_http_request_duration_seconds_bucket[5m]))

  - name: downsample_1h
    interval: 1h
    rules:
      # Keep 1-hour averages for metrics older than 30 days
      - record: downsampled:1h:erp_http_requests_total:rate
        expr: avg_over_time(downsampled:5m:erp_http_requests_total:rate[1h])
```

### 6.3 Per-Tenant Retention Override

Tenant tier determines retention (see Multi-Tenancy Guide):

```sql
-- Retention policies per tenant tier
SELECT
    t.tenant_id,
    t.tier,
    CASE t.tier
        WHEN 'free'       THEN '7d metrics, 3d logs, 1d traces'
        WHEN 'standard'   THEN '30d metrics, 14d logs, 7d traces'
        WHEN 'enterprise' THEN '90d metrics, 90d logs, 30d traces'
    END as retention_policy,
    CASE t.tier
        WHEN 'free'       THEN estimated_storage_gb * 0.23
        WHEN 'standard'   THEN estimated_storage_gb * 1.0
        WHEN 'enterprise' THEN estimated_storage_gb * 3.2
    END as estimated_storage_multiplier
FROM tenants t;
```

---

## 7. Growth Projection Formulas

### 7.1 Module Growth Model

As new ERP modules are deployed, observability resource requirements grow linearly:

```
Resources(M+1) = Resources(M) + Incremental_Per_Module

Where Incremental_Per_Module:
  CPU:     +4 cores (2 for indexing, 1 for query, 1 for pipeline)
  Memory:  +6 GB (3 for indexing, 2 for query cache, 1 for pipeline)
  Storage: +(Lps_module * 86400 * Ls * Rl / Cl) for logs
           +(Tps_module * 86400 * Ss * Rt / Ct) for traces
  Network: +(Lps_module * Ls + Tps_module * Ss) * 8 bits
```

### 7.2 Tenant Growth Model

Adding tenants increases cardinality more than volume:

```
Impact_Per_Tenant:
  Active Series:  +10 per metric * Mpm metrics = +2,000 series (for 200 base metrics)
  Log Volume:     +2-5% increase (new tenant generates proportional logs)
  Trace Volume:   +2-5% increase (new tenant generates proportional traces)
  VM Memory:      +2 MB (2,000 series * 1 KB)
  QW Storage:     Proportional to tenant's log/trace volume
```

### 7.3 Time-Based Growth Projection

```
Storage(t) = Storage(t0) * (1 + monthly_growth_rate) ^ months

Where:
  monthly_growth_rate = (new_modules_per_month * per_module_storage
                        + new_tenants_per_month * per_tenant_storage)
                        / current_total_storage

Example:
  Current: 10 modules, 50 tenants, 4 TB total
  Growth: 1 new module/quarter, 5 new tenants/month

  Per-month storage growth rate:
  = (0.33 * 400 GB + 5 * 20 GB) / 4,000 GB
  = (132 + 100) / 4,000
  = 5.8% per month

  6-month projection: 4 TB * (1.058)^6 = ~5.6 TB
  12-month projection: 4 TB * (1.058)^12 = ~7.8 TB
```

### 7.4 Growth Projection Table

Starting from Medium deployment (10 modules, 50 tenants, ~4 TB):

| Month | Modules | Tenants | Active Series | Total Storage | CPU Cores | Memory |
|-------|---------|---------|---------------|---------------|-----------|--------|
| 0 | 10 | 50 | 70,000 | 4.0 TB | 36 | 72 GB |
| 3 | 11 | 65 | 91,000 | 4.7 TB | 40 | 78 GB |
| 6 | 12 | 80 | 112,000 | 5.6 TB | 44 | 84 GB |
| 9 | 13 | 95 | 133,000 | 6.5 TB | 48 | 90 GB |
| 12 | 14 | 110 | 154,000 | 7.8 TB | 52 | 96 GB |
| 18 | 15 | 140 | 196,000 | 10.8 TB | 56 | 108 GB |
| 24 | 17 | 170 | 252,000 | 15.2 TB | 64 | 120 GB |

---

## 8. Scaling Triggers and Automation

### 8.1 Horizontal Scaling Triggers

| Component | Metric | Scale-Up Threshold | Scale-Down Threshold | Action |
|-----------|--------|-------------------|---------------------|--------|
| vminsert | CPU usage | > 70% for 10m | < 30% for 30m | Add/remove replica |
| vmselect | Query latency p99 | > 500ms for 5m | < 100ms for 30m | Add/remove replica |
| vmstorage | Disk usage | > 70% | < 40% | Add node + rebalance |
| QW Indexer | Indexing lag | > 60s | < 5s for 30m | Add/remove replica |
| QW Searcher | Search latency p99 | > 1s for 5m | < 200ms for 30m | Add/remove replica |
| OTel Collector | Memory usage | > 80% limit | < 40% limit for 30m | Add/remove replica |
| OTel Collector | Dropped data | > 0 for 5m | N/A | Immediate add replica |
| Grafana | Dashboard load p95 | > 5s for 10m | < 1s for 30m | Add replica |

### 8.2 Kubernetes HPA Configuration

```yaml
# VictoriaMetrics vminsert HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vminsert-hpa
  namespace: observability
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vminsert
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: vm_rows_inserted_total_rate
        target:
          type: AverageValue
          averageValue: "250000"  # Scale at 250K samples/sec per pod
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 2
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Pods
          value: 1
          periodSeconds: 120

---
# Quickwit Indexer HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: quickwit-indexer-hpa
  namespace: observability
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: quickwit-indexer
  minReplicas: 2
  maxReplicas: 8
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Pods
      pods:
        metric:
          name: quickwit_indexing_pipeline_pending_docs
        target:
          type: AverageValue
          averageValue: "50000"  # Scale when pipeline has 50K pending docs
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 120
      policies:
        - type: Pods
          value: 1
          periodSeconds: 120
    scaleDown:
      stabilizationWindowSeconds: 600
```

### 8.3 Capacity Planning Alerts

```yaml
# Proactive capacity alerts
groups:
  - name: capacity_planning
    rules:
      - alert: VMStorageNearingCapacity
        expr: |
          (vm_data_size_bytes / vm_storage_capacity_bytes) > 0.7
        for: 1h
        labels:
          severity: warning
          category: capacity
        annotations:
          summary: "VictoriaMetrics storage at {{ $value | humanizePercentage }}"
          action: "Plan storage expansion within 2 weeks"

      - alert: VMStorageCritical
        expr: |
          (vm_data_size_bytes / vm_storage_capacity_bytes) > 0.85
        for: 30m
        labels:
          severity: critical
          category: capacity
        annotations:
          summary: "VictoriaMetrics storage at {{ $value | humanizePercentage }}"
          action: "Immediate storage expansion or retention reduction required"

      - alert: QuickwitStorageGrowthRate
        expr: |
          predict_linear(quickwit_storage_used_bytes[7d], 30*86400)
          > quickwit_storage_capacity_bytes * 0.9
        for: 6h
        labels:
          severity: warning
          category: capacity
        annotations:
          summary: "Quickwit storage projected to reach 90% in 30 days"
          action: "Plan storage expansion or review retention policies"

      - alert: HighCardinalityGrowth
        expr: |
          rate(vm_new_timeseries_created_total[1h]) > 1000
        for: 30m
        labels:
          severity: warning
          category: capacity
        annotations:
          summary: "New metric series being created at {{ $value }}/sec"
          action: "Investigate potential cardinality explosion"

      - alert: TenantStorageQuotaNearing
        expr: |
          tenant_storage_used_bytes / on(tenant_id) tenant_storage_quota_bytes > 0.8
        for: 1h
        labels:
          severity: warning
          category: capacity
        annotations:
          summary: "Tenant {{ $labels.tenant_id }} at {{ $value | humanizePercentage }} of storage quota"
```

---

## 9. Capacity Planning Spreadsheet Template

### 9.1 Input Parameters Worksheet

| Parameter | Your Value | Default |
|-----------|-----------|---------|
| Number of ERP modules | ___ | 10 |
| Number of tenants | ___ | 50 |
| Average metrics per module | ___ | 200 |
| Average label combinations | ___ | 10 |
| Scrape interval (seconds) | ___ | 15 |
| Logs per second per module | ___ | 5,000 |
| Average log size (bytes) | ___ | 512 |
| Spans per second per module | ___ | 2,500 |
| Average span size (bytes) | ___ | 1,024 |
| Metrics retention (days) | ___ | 30 |
| Logs retention (days) | ___ | 14 |
| Traces retention (days) | ___ | 7 |
| Expected module growth/year | ___ | 4 |
| Expected tenant growth/month | ___ | 5 |
| Headroom multiplier | ___ | 2.0 |

### 9.2 Output Summary Worksheet

| Resource | Calculated | With Headroom (2x) |
|----------|-----------|-------------------|
| Active metric series | ___ | ___ |
| Total CPU cores | ___ | ___ |
| Total RAM (GB) | ___ | ___ |
| Metric storage (GB) | ___ | ___ |
| Log storage (TB) | ___ | ___ |
| Trace storage (TB) | ___ | ___ |
| Object storage total (TB) | ___ | ___ |
| Network bandwidth (Gbps) | ___ | ___ |
| 12-month projected storage (TB) | ___ | ___ |
| 24-month projected storage (TB) | ___ | ___ |

---

## 10. Cost Estimation

### 10.1 Infrastructure Cost Model

| Resource | Unit | Estimated Cost/Month | Notes |
|----------|------|---------------------|-------|
| CPU Core | per core | $25-50 | Varies by cloud/on-prem |
| RAM | per GB | $5-10 | ECC required for data stores |
| NVMe SSD | per TB | $80-150 | High IOPS for hot storage |
| HDD/SSD (warm) | per TB | $20-40 | Standard SSD for warm tier |
| Object Storage (RustFS) | per TB | $10-20 | Self-hosted, 3x replication |
| Network | per Gbps sustained | $50-100 | Internal data center fabric |

### 10.2 Total Cost Estimate by Tier

| Deployment | Monthly Compute | Monthly Storage | Monthly Network | **Total Monthly** |
|-----------|----------------|-----------------|----------------|-------------------|
| Tiny | $600 | $200 | $50 | **$850** |
| Small | $1,500 | $500 | $100 | **$2,100** |
| Medium | $4,000 | $2,000 | $500 | **$6,500** |
| Large | $8,000 | $5,000 | $1,000 | **$14,000** |
| Enterprise | $20,000+ | $15,000+ | $2,500+ | **$37,500+** |

*Note: Costs are estimates for self-hosted infrastructure. Cloud-hosted costs may differ significantly.*
