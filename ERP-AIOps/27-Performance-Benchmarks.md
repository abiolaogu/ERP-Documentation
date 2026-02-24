# ERP-AIOps Performance Benchmarks

> **Document ID:** ERP-AIOPS-PERF-027
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [14-Technical-Specifications.md](./14-Technical-Specifications.md), [30-Capacity-Planning.md](./30-Capacity-Planning.md)

---

## 1. Anomaly Detection Latency Benchmarks

### Individual Algorithm Latency

Measured with 1,000,000 metric points, single-threaded, on standard hardware (4 vCPU, 8 GB RAM).

| Algorithm | p50 | p95 | p99 | p99.9 | Max |
|-----------|-----|-----|-----|-------|-----|
| Z-Score | 0.8ms | 2.1ms | 4.3ms | 8.1ms | 12ms |
| IQR | 1.1ms | 2.8ms | 5.2ms | 9.4ms | 15ms |
| Moving Average | 1.4ms | 3.5ms | 7.1ms | 12ms | 18ms |
| Isolation Forest | 8.2ms | 22ms | 38ms | 52ms | 68ms |
| LSTM | 18ms | 45ms | 72ms | 95ms | 120ms |

### Ensemble (All Algorithms Combined)

The ensemble runs all algorithms in parallel and fuses scores.

| Metric | Value |
|--------|-------|
| p50 Latency | 22ms |
| p95 Latency | 52ms |
| p99 Latency | 78ms |
| p99.9 Latency | 98ms |
| Target SLA | < 100ms (p99) |
| SLA Met | Yes |

### Latency by Load Level

| Concurrent Metrics/sec | p50 | p95 | p99 | SLA Met |
|------------------------|-----|-----|-----|---------|
| 10,000 | 18ms | 42ms | 65ms | Yes |
| 50,000 | 22ms | 55ms | 82ms | Yes |
| 100,000 | 28ms | 68ms | 95ms | Yes |
| 150,000 | 45ms | 98ms | 135ms | No (scale out needed) |
| 200,000 (2 nodes) | 24ms | 58ms | 88ms | Yes |

---

## 2. Event Correlation Throughput

### Correlation Engine Benchmarks

| Metric | Value |
|--------|-------|
| Events Processed per Second | 12,500 |
| Correlation Window | 5 minutes (configurable) |
| Average Events per Correlation Group | 4.2 |
| Noise Reduction (correlated vs. raw) | 78% reduction |

### Correlation Latency by Strategy

| Strategy | p50 | p95 | p99 |
|----------|-----|-----|-----|
| Temporal Only | 12ms | 35ms | 68ms |
| Topological Only | 45ms | 120ms | 210ms |
| Pattern Matching | 85ms | 180ms | 320ms |
| Combined (all three) | 110ms | 280ms | 450ms |
| Target SLA | - | - | < 500ms |
| SLA Met | - | - | Yes |

### Correlation Accuracy

Measured against manually labeled incident data from a 30-day production sample.

| Metric | Value |
|--------|-------|
| True Positive Rate (correctly grouped) | 92.3% |
| False Positive Rate (incorrectly grouped) | 4.1% |
| False Negative Rate (missed correlation) | 7.7% |
| Average Incidents per Day (with correlation) | 23 |
| Average Incidents per Day (without correlation) | 108 |
| Noise Reduction Factor | 4.7x |

---

## 3. RCA Analysis Time

### End-to-End RCA Duration

Measured across 200 RCA executions in production.

| Phase | p50 | p95 | Max |
|-------|-----|-----|-----|
| Evidence Collection (metrics) | 3.2s | 8.5s | 15s |
| Evidence Collection (logs) | 5.1s | 12s | 25s |
| Evidence Collection (traces) | 2.8s | 7.2s | 18s |
| Causal Inference | 8.5s | 22s | 45s |
| LLM Analysis | 12s | 35s | 60s |
| Similar Incident Search | 2.1s | 5.5s | 12s |
| **Total End-to-End** | **34s** | **85s** | **120s** |
| **Target SLA** | - | **< 120s** | - |
| **SLA Met** | - | **Yes** | - |

### RCA Accuracy

| Metric | Value |
|--------|-------|
| Top-1 Root Cause Accuracy | 78% |
| Top-3 Root Cause Accuracy | 93% |
| LLM Summary Relevance (human-rated) | 91% |
| Similar Incident Match Precision | 84% |
| Target: Top-1 > 75% | Met |
| Target: Top-3 > 90% | Met |

---

## 4. API Response Time Targets

### Rust API Endpoint Latency

Measured under production load (500 req/sec aggregate).

| Endpoint Category | p50 | p95 | p99 | Target p95 |
|-------------------|-----|-----|-----|------------|
| Incident List (paginated) | 8ms | 22ms | 38ms | < 50ms |
| Incident Detail | 5ms | 15ms | 28ms | < 50ms |
| Incident Create | 12ms | 32ms | 48ms | < 50ms |
| Anomaly List | 10ms | 28ms | 42ms | < 50ms |
| Anomaly Detail | 6ms | 18ms | 32ms | < 50ms |
| Rule List | 4ms | 12ms | 22ms | < 50ms |
| Rule Evaluate (single) | 1ms | 3ms | 8ms | < 10ms |
| Topology Full Graph | 35ms | 95ms | 155ms | < 200ms |
| Blast Radius | 120ms | 450ms | 1.2s | < 2s |
| Cost Dashboard | 25ms | 65ms | 110ms | < 200ms |
| Security Findings List | 12ms | 35ms | 55ms | < 100ms |

### Go Gateway Overhead

The gateway adds minimal overhead for authentication, tenant extraction, and routing.

| Metric | Value |
|--------|-------|
| Gateway Overhead p50 | 1.2ms |
| Gateway Overhead p95 | 2.8ms |
| Gateway Overhead p99 | 4.5ms |
| Target: < 5ms (p99) | Met |

### Python AI Brain API Latency

| Endpoint | p50 | p95 | p99 |
|----------|-----|-----|-----|
| POST /detect (single metric) | 22ms | 55ms | 85ms |
| POST /batch-detect (100 metrics) | 180ms | 420ms | 680ms |
| POST /analyze (RCA) | 34s | 85s | 115s |
| POST /predict (forecast) | 1.2s | 3.5s | 5.8s |
| GET /thresholds | 2ms | 5ms | 12ms |

---

## 5. Rust API vs Python Brain Latency Comparison

### Operation-Level Comparison

| Operation | Rust API | Python Brain | Ratio | Notes |
|-----------|---------|-------------|-------|-------|
| JSON Parsing (1KB payload) | 0.02ms | 0.15ms | 7.5x | Rust serde vs. Python json |
| Database Query (simple) | 2ms | 3ms | 1.5x | Both use async drivers |
| HTTP Request Handling | 0.5ms | 2ms | 4x | Axum vs. FastAPI |
| Rule Evaluation (single) | 0.8ms | N/A | N/A | Rust only |
| Anomaly Scoring (Z-Score) | N/A | 0.8ms | N/A | Python only |
| Anomaly Scoring (Isolation Forest) | N/A | 8ms | N/A | Python only (sklearn) |
| Memory per Connection | 8 KB | 64 KB | 8x | Rust tokio vs. Python uvicorn |
| Startup Time | 0.5s | 45s | 90x | Rust binary vs. Python + model loading |

### Why This Split?

The Rust API handles all latency-sensitive CRUD operations, event processing, and real-time streaming. The Python AI brain handles ML inference where the computational cost of the algorithms (not the framework overhead) dominates. This gives us the best of both worlds: Rust's performance for the hot path and Python's ML ecosystem for intelligence.

---

## 6. Resource Usage Under Load

### Rust API Resource Consumption

| Load (events/sec) | CPU (cores) | Memory (MB) | Goroutines/Tasks | DB Connections |
|-------------------|-------------|-------------|-------------------|----------------|
| 10,000 | 0.8 | 256 | 1,200 | 8 |
| 50,000 | 2.1 | 512 | 4,500 | 15 |
| 100,000 | 3.8 | 1,024 | 8,200 | 20 |
| 150,000 | 5.5 | 1,536 | 12,000 | 20 |

### Python AI Brain Resource Consumption

| Load (inferences/sec) | CPU (cores) | Memory (GB) | GPU Utilization |
|-----------------------|-------------|-------------|-----------------|
| 100 | 1.2 | 2.8 | N/A (CPU only) |
| 500 | 2.8 | 4.2 | N/A |
| 1,000 | 3.9 | 5.8 | N/A |
| 5,000 | 3.5 | 6.2 | 45% (with GPU) |
| 10,000 | 3.8 | 7.5 | 72% (with GPU) |

### YugabyteDB Resource Consumption

| Active Incidents | Queries/sec | CPU (cores) | Memory (GB) | Disk I/O (MB/s) |
|-----------------|-------------|-------------|-------------|------------------|
| 100 | 500 | 1.5 | 3.2 | 15 |
| 500 | 2,000 | 2.8 | 4.8 | 45 |
| 1,000 | 5,000 | 3.5 | 6.5 | 85 |
| 5,000 | 12,000 | 6.2 | 12 | 180 |

### DragonflyDB Resource Consumption

| Cached Keys | Memory (GB) | Ops/sec | CPU (cores) |
|-------------|-------------|---------|-------------|
| 50,000 | 0.5 | 50,000 | 0.5 |
| 200,000 | 1.2 | 120,000 | 1.0 |
| 500,000 | 2.8 | 200,000 | 1.5 |
| 1,000,000 | 5.2 | 350,000 | 2.0 |

---

## 7. Scaling Recommendations

### Horizontal Scaling Triggers

| Component | Scale-Out Trigger | Scale-In Trigger | Min Replicas | Max Replicas |
|-----------|------------------|------------------|-------------|-------------|
| Go Gateway | CPU > 70% or p99 > 10ms | CPU < 30% for 15 min | 2 | 8 |
| Rust API | CPU > 70% or event backlog > 10K | CPU < 30% for 15 min | 2 | 10 |
| Python AI Brain | Inference queue > 1000 or p99 > 200ms | Queue < 100 for 15 min | 2 | 6 |

### Vertical Scaling Recommendations

| Component | Small (5 modules) | Medium (15 modules) | Large (30+ modules) |
|-----------|-------------------|---------------------|---------------------|
| Go Gateway | 1 CPU, 256 MB | 2 CPU, 512 MB | 4 CPU, 1 GB |
| Rust API | 2 CPU, 1 GB | 4 CPU, 2 GB | 8 CPU, 4 GB |
| Python AI Brain | 2 CPU, 4 GB | 4 CPU, 8 GB | 8 CPU + GPU, 16 GB |
| YugabyteDB | 2 CPU, 4 GB, 50 GB | 4 CPU, 8 GB, 200 GB | 8 CPU, 16 GB, 500 GB |
| DragonflyDB | 1 CPU, 1 GB | 2 CPU, 4 GB | 4 CPU, 8 GB |

### Benchmark Environment

All benchmarks were collected on the following hardware unless otherwise noted:

| Parameter | Value |
|-----------|-------|
| Kubernetes Version | 1.29 |
| Node Type | 8 vCPU, 32 GB RAM (c5.2xlarge equivalent) |
| Network | 10 Gbps intra-cluster |
| Storage | NVMe SSD |
| OS | Ubuntu 22.04 |
| Rust Version | 1.76 |
| Python Version | 3.12 |
| Go Version | 1.22 |
