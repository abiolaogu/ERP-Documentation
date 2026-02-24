# ERP-AIOps Capacity Planning

> **Document ID:** ERP-AIOPS-CAP-030
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [27-Performance-Benchmarks.md](./27-Performance-Benchmarks.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## 1. Resource Requirements per ERP Module Monitored

Each ERP module monitored by AIOps contributes resource load across all AIOps components. The following table estimates the incremental cost of adding one module.

### Per-Module Resource Overhead

| Component | Metric | Per Module (Avg) | Notes |
|-----------|--------|-----------------|-------|
| OTel Gateway Collector | CPU | +0.05 cores | Processing telemetry from one module |
| OTel Gateway Collector | Memory | +25 MB | Batch buffers and metadata enrichment |
| OTel Gateway Collector | Network (inbound) | +2 MB/s | Metrics + logs + traces from one module |
| Rust API | CPU | +0.15 cores | Event processing, rule evaluation |
| Rust API | Memory | +50 MB | Active event buffers, rule state |
| Python AI Brain | CPU | +0.2 cores | Running detection ensemble per module metrics |
| Python AI Brain | Memory | +200 MB | Per-module model instances (IF, rolling stats) |
| YugabyteDB | Storage (monthly) | +2 GB | Incidents, anomalies, events, topology |
| YugabyteDB | IOPS | +50 IOPS | Read/write for module data |
| DragonflyDB | Memory | +100 MB | Cache entries for module metrics, scores |

### Total Resource Profile by Module Count

| Modules | Rust API (CPU/Mem) | AI Brain (CPU/Mem) | YugabyteDB (Storage) | DragonflyDB (Mem) |
|---------|-------------------|--------------------|---------------------|-------------------|
| 5 | 0.75 cores / 250 MB | 1.0 cores / 1 GB | 10 GB/month | 500 MB |
| 10 | 1.5 cores / 500 MB | 2.0 cores / 2 GB | 20 GB/month | 1 GB |
| 15 | 2.25 cores / 750 MB | 3.0 cores / 3 GB | 30 GB/month | 1.5 GB |
| 20 | 3.0 cores / 1 GB | 4.0 cores / 4 GB | 40 GB/month | 2 GB |
| 30 | 4.5 cores / 1.5 GB | 6.0 cores / 6 GB | 60 GB/month | 3 GB |
| 50 | 7.5 cores / 2.5 GB | 10.0 cores / 10 GB | 100 GB/month | 5 GB |

---

## 2. ML Model Memory Requirements

### Model Memory Footprint

| Model | Base Memory | Per-Tenant Overhead | Per-Metric Overhead |
|-------|-------------|--------------------|--------------------|
| Z-Score (rolling stats) | 10 MB | 500 KB | 1 KB per metric series |
| IQR (percentile tracker) | 15 MB | 800 KB | 2 KB per metric series |
| Moving Average | 5 MB | 200 KB | 0.5 KB per metric series |
| Isolation Forest | 200 MB | 50 MB per tenant model | N/A (uses feature vectors) |
| LSTM Base Model | 500 MB | 100 MB per fine-tuned variant | N/A |
| Prophet (per forecast) | 50 MB | 10 MB per active forecast | N/A |
| Ensemble Configuration | 5 MB | 1 MB | N/A |

### Total AI Brain Memory by Scale

| Scale | Tenants | Metrics/Tenant | Total AI Brain Memory |
|-------|---------|---------------|-----------------------|
| Small | 5 | 2,500 | 2.5 GB |
| Medium | 20 | 5,000 | 6.0 GB |
| Large | 50 | 10,000 | 14.0 GB |
| Enterprise | 100 | 15,000 | 28.0 GB |

### GPU Requirements

| Scale | GPU Needed | GPU Type | Usage |
|-------|-----------|----------|-------|
| Small (5 tenants) | No | N/A | CPU inference sufficient |
| Medium (20 tenants) | Optional | T4 | Accelerates LSTM inference by 5x |
| Large (50 tenants) | Recommended | A10G | Required for acceptable LSTM latency |
| Enterprise (100+ tenants) | Required | A100 | Multiple concurrent LSTM inferences |

---

## 3. YugabyteDB Storage Growth Projections

### Storage per Data Type

| Data Type | Avg Record Size | Records/Day (20 modules) | Daily Growth | Monthly Growth |
|-----------|----------------|--------------------------|-------------|----------------|
| Events (raw) | 500 bytes | 5,000,000 | 2.5 GB | 75 GB |
| Events (aggregated) | 200 bytes | 500,000 | 100 MB | 3 GB |
| Anomalies | 2 KB | 5,000 | 10 MB | 300 MB |
| Incidents | 5 KB | 200 | 1 MB | 30 MB |
| Rules | 3 KB | 10 (changes) | 30 KB | 900 KB |
| Topology | 1 KB per edge | 1,000 (snapshots) | 1 MB | 30 MB |
| RCA Reports | 50 KB | 50 | 2.5 MB | 75 MB |
| Security Findings | 3 KB | 500 | 1.5 MB | 45 MB |
| Cost Data | 500 bytes | 10,000 | 5 MB | 150 MB |
| Audit Logs | 1 KB | 50,000 | 50 MB | 1.5 GB |

### Retention Policy Impact

| Retention Policy | Raw Events | Aggregated | Anomalies | Total at Steady State |
|-----------------|------------|------------|-----------|----------------------|
| 7 days (Free tier) | 17.5 GB | 700 MB | 70 MB | ~20 GB |
| 30 days (Standard) | 75 GB | 3 GB | 300 MB | ~85 GB |
| 90 days (Enterprise) | 225 GB | 9 GB | 900 MB | ~250 GB |
| 1 year (with downsampling) | 75 GB raw + 36 GB downsampled | 36 GB | 3.6 GB | ~165 GB |

### Storage Growth Projection (Standard Tier, 20 Modules)

| Month | Cumulative Storage | With Compaction | Notes |
|-------|-------------------|-----------------|-------|
| 1 | 85 GB | 70 GB | Initial data |
| 3 | 170 GB | 140 GB | Steady growth, retention kicks in |
| 6 | 170 GB | 140 GB | Steady state (30-day retention) |
| 12 | 175 GB | 145 GB | Slight growth from non-expiring data |

---

## 4. DragonflyDB Cache Sizing

### Cache Key Categories

| Category | Key Pattern | Avg Value Size | TTL | Keys (20 modules) |
|----------|-----------|---------------|-----|-------------------|
| Anomaly Scores | `{tenant}:anomaly:score:{svc}:{metric}` | 500 bytes | 15 min | 50,000 |
| Rule Eval Cache | `{tenant}:rule:eval:{rule_id}:{hash}` | 200 bytes | 5 min | 20,000 |
| Topology Graph | `{tenant}:topology:graph` | 50 KB | 5 min | Per tenant |
| Incident Cache | `{tenant}:incident:{id}` | 5 KB | 10 min | 5,000 |
| Session Data | `{tenant}:session:{user_id}` | 2 KB | 30 min | 1,000 |
| Rate Limit Counters | `ratelimit:{tenant}:{window}` | 8 bytes | 2 min | 5,000 |
| Event Streams | `events:{tenant}` | Variable | 1 hour | Per tenant |

### Memory Sizing by Scale

| Scale | Total Keys | Avg Memory | Recommended Allocation | Headroom |
|-------|-----------|------------|----------------------|----------|
| Small (5 tenants, 5 modules) | 50,000 | 400 MB | 1 GB | 60% |
| Medium (20 tenants, 15 modules) | 500,000 | 2 GB | 4 GB | 50% |
| Large (50 tenants, 20 modules) | 2,000,000 | 6 GB | 8 GB | 25% |
| Enterprise (100 tenants, 30 modules) | 5,000,000 | 12 GB | 16 GB | 25% |

### Eviction Policy

DragonflyDB is configured with `maxmemory-policy allkeys-lru`. When memory pressure occurs:

1. Least recently used keys are evicted first.
2. Rate limit counters and session data are evicted before anomaly scores.
3. Topology graph and incident caches are rebuilt on demand from YugabyteDB.

---

## 5. Network Bandwidth for Telemetry Ingestion

### Per-Module Bandwidth

| Signal | Avg Bandwidth | Peak Bandwidth | Protocol |
|--------|--------------|----------------|----------|
| Metrics | 500 KB/s | 2 MB/s | OTLP gRPC (compressed) |
| Logs | 1 MB/s | 5 MB/s | OTLP gRPC (compressed) |
| Traces | 500 KB/s | 3 MB/s | OTLP gRPC (compressed) |
| **Total per module** | **2 MB/s** | **10 MB/s** | - |

### Aggregate Bandwidth by Scale

| Modules | Sustained Bandwidth | Peak Bandwidth | Network Requirement |
|---------|-------------------|----------------|---------------------|
| 5 | 10 MB/s | 50 MB/s | 1 Gbps (10% utilization) |
| 10 | 20 MB/s | 100 MB/s | 1 Gbps (20% utilization) |
| 20 | 40 MB/s | 200 MB/s | 1 Gbps (40% utilization) |
| 30 | 60 MB/s | 300 MB/s | 1 Gbps (60% utilization) |
| 50 | 100 MB/s | 500 MB/s | 10 Gbps recommended |

### Internal Service-to-Service Traffic

| Path | Avg Bandwidth | Peak |
|------|--------------|------|
| Gateway -> Rust API | 5 MB/s | 20 MB/s |
| Rust API -> AI Brain | 10 MB/s | 40 MB/s |
| Rust API -> YugabyteDB | 8 MB/s | 30 MB/s |
| Rust API -> DragonflyDB | 15 MB/s | 50 MB/s |
| AI Brain -> VictoriaMetrics | 5 MB/s | 20 MB/s |
| AI Brain -> Quickwit | 2 MB/s | 10 MB/s |
| WebSocket (outbound to clients) | 1 MB/s | 5 MB/s |

---

## 6. Scaling Tiers

### Tier 1: Small (5 Modules)

Suitable for initial AIOps deployment monitoring a subset of ERP modules.

| Component | Replicas | CPU (per) | Memory (per) | Storage |
|-----------|----------|-----------|-------------|---------|
| Go Gateway | 2 | 1 core | 256 MB | - |
| Rust API | 2 | 2 cores | 1 GB | - |
| Python AI Brain | 1 | 2 cores | 4 GB | 20 GB (models) |
| YugabyteDB | 3 (RF=3) | 2 cores | 4 GB | 50 GB SSD each |
| DragonflyDB | 1 | 1 core | 1 GB | - |
| OTel Gateway Collector | 1 | 1 core | 512 MB | - |
| **Total** | **10 pods** | **17 cores** | **19 GB** | **170 GB** |

**Estimated Monthly Cost (cloud):** $800 - $1,200

### Tier 2: Medium (15 Modules)

Suitable for most ERP deployments with full module coverage.

| Component | Replicas | CPU (per) | Memory (per) | Storage |
|-----------|----------|-----------|-------------|---------|
| Go Gateway | 2 | 2 cores | 512 MB | - |
| Rust API | 3 | 4 cores | 2 GB | - |
| Python AI Brain | 2 | 4 cores | 8 GB | 40 GB each |
| YugabyteDB | 3 (RF=3) | 4 cores | 8 GB | 200 GB SSD each |
| DragonflyDB | 1 | 2 cores | 4 GB | - |
| OTel Gateway Collector | 2 | 2 cores | 1 GB | - |
| **Total** | **13 pods** | **44 cores** | **58 GB** | **680 GB** |

**Estimated Monthly Cost (cloud):** $2,500 - $3,800

### Tier 3: Large (30+ Modules)

Suitable for large-scale ERP deployments or multi-tenant SaaS.

| Component | Replicas | CPU (per) | Memory (per) | Storage |
|-----------|----------|-----------|-------------|---------|
| Go Gateway | 4 | 2 cores | 512 MB | - |
| Rust API | 5 | 4 cores | 4 GB | - |
| Python AI Brain | 3 + GPU | 8 cores | 16 GB | 100 GB each |
| YugabyteDB | 5 (RF=3) | 8 cores | 16 GB | 500 GB SSD each |
| DragonflyDB | 2 | 4 cores | 8 GB | - |
| OTel Gateway Collector | 3 | 4 cores | 2 GB | - |
| RustFS | 3 | 2 cores | 4 GB | 1 TB each |
| **Total** | **25 pods** | **110 cores** | **150 GB** | **6 TB** |

**Estimated Monthly Cost (cloud):** $7,000 - $12,000

### Scaling Decision Matrix

| Indicator | Current Threshold | Action |
|-----------|------------------|--------|
| Event ingestion backlog | > 10,000 events | Scale Rust API replicas |
| Anomaly detection p99 > 200ms | Sustained 10 min | Scale AI Brain replicas |
| YugabyteDB CPU > 70% | Sustained 15 min | Add tablet server node |
| DragonflyDB memory > 80% | Current | Increase memory or add node |
| API p95 > 100ms | Sustained 10 min | Scale Rust API replicas |
| OTel Collector drop rate > 1% | Current | Scale collector replicas |
| Gateway p99 > 10ms | Sustained 5 min | Scale gateway replicas |

### Capacity Planning Review Schedule

| Activity | Frequency | Owner |
|----------|-----------|-------|
| Review resource utilization trends | Weekly | SRE team |
| Update capacity projections | Monthly | Platform team |
| Load testing against next tier | Quarterly | SRE + Platform team |
| Full capacity planning review | Semi-annually | Engineering leadership |
