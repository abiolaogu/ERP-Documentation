# ERP-AIOps Technical Specifications

> **Document ID:** ERP-AIOPS-SPEC-014
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [13-Low-Level-Design.md](./13-Low-Level-Design.md), [21-API-Documentation.md](./21-API-Documentation.md)

---

## 1. API Endpoint Specifications (Rust Axum Routes)

### 1.1 Incident Endpoints

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/v1/incidents` | List incidents with filtering and pagination | Yes |
| POST | `/api/v1/incidents` | Create a new incident manually | Yes |
| GET | `/api/v1/incidents/:id` | Get incident detail | Yes |
| PUT | `/api/v1/incidents/:id` | Update incident fields | Yes |
| POST | `/api/v1/incidents/:id/transition` | Transition incident state | Yes |
| POST | `/api/v1/incidents/:id/rca` | Trigger RCA for an incident | Yes |
| GET | `/api/v1/incidents/:id/timeline` | Get incident timeline events | Yes |
| POST | `/api/v1/incidents/:id/comments` | Add a comment to an incident | Yes |
| GET | `/api/v1/incidents/stats` | Get incident statistics and SLA metrics | Yes |

### 1.2 Anomaly Endpoints

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/v1/anomalies` | List detected anomalies | Yes |
| GET | `/api/v1/anomalies/:id` | Get anomaly detail with contributing factors | Yes |
| PUT | `/api/v1/anomalies/:id/status` | Update anomaly status (reviewed, dismissed) | Yes |
| GET | `/api/v1/anomalies/thresholds` | Get adaptive threshold configuration | Yes |
| PUT | `/api/v1/anomalies/thresholds` | Update threshold sensitivity | Yes |
| GET | `/api/v1/anomalies/stats` | Anomaly statistics (counts by severity, service) | Yes |

### 1.3 Rule Endpoints

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/v1/rules` | List all rules | Yes |
| POST | `/api/v1/rules` | Create a new rule | Yes |
| GET | `/api/v1/rules/:id` | Get rule detail | Yes |
| PUT | `/api/v1/rules/:id` | Update a rule | Yes |
| DELETE | `/api/v1/rules/:id` | Soft-delete a rule | Yes |
| POST | `/api/v1/rules/:id/toggle` | Enable/disable a rule | Yes |
| POST | `/api/v1/rules/:id/clone` | Clone a rule | Yes |
| GET | `/api/v1/rules/:id/history` | Get rule audit history | Yes |

### 1.4 Topology Endpoints

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/v1/topology` | Get the full service topology graph | Yes |
| GET | `/api/v1/topology/services/:id` | Get service detail with health status | Yes |
| GET | `/api/v1/topology/services/:id/blast-radius` | Compute blast radius for a service | Yes |
| GET | `/api/v1/topology/services/:id/dependencies` | Get upstream and downstream dependencies | Yes |

### 1.5 Remediation Endpoints

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/v1/remediation/playbooks` | List playbooks | Yes |
| POST | `/api/v1/remediation/playbooks` | Create a playbook | Yes |
| GET | `/api/v1/remediation/playbooks/:id` | Get playbook detail | Yes |
| PUT | `/api/v1/remediation/playbooks/:id` | Update a playbook | Yes |
| DELETE | `/api/v1/remediation/playbooks/:id` | Delete a playbook | Yes |
| GET | `/api/v1/remediation/activity` | Get remediation activity log | Yes |
| POST | `/api/v1/remediation/approve/:id` | Approve a pending remediation | Yes |
| POST | `/api/v1/remediation/reject/:id` | Reject a pending remediation | Yes |
| GET | `/api/v1/remediation/actions` | List available action catalog | Yes |

### 1.6 Cost Endpoints

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/v1/cost/dashboard` | Get cost dashboard data | Yes |
| GET | `/api/v1/cost/recommendations` | List optimization recommendations | Yes |
| GET | `/api/v1/cost/recommendations/:id` | Get recommendation detail | Yes |
| POST | `/api/v1/cost/recommendations/:id/apply` | Apply a recommendation | Yes |
| POST | `/api/v1/cost/recommendations/:id/reject` | Reject a recommendation | Yes |
| GET | `/api/v1/cost/savings` | Get savings tracker data | Yes |

### 1.7 Security Endpoints

| Method | Path | Description | Auth Required |
|--------|------|-------------|---------------|
| GET | `/api/v1/security/findings` | List security findings | Yes |
| GET | `/api/v1/security/findings/:id` | Get finding detail | Yes |
| PUT | `/api/v1/security/findings/:id/triage` | Triage a finding | Yes |
| GET | `/api/v1/security/posture` | Get security posture metrics | Yes |
| POST | `/api/v1/security/scan` | Trigger a manual scan | Yes |

---

## 2. WebSocket Event Stream Protocol

### 2.1 Connection

```
ws://localhost:8080/ws/events?token=<jwt>&tenant_id=<tenant>
```

### 2.2 Client-to-Server Messages

```json
{
  "type": "subscribe",
  "channels": ["incidents", "anomalies", "remediation"],
  "filters": {
    "severity": ["P1", "P2"],
    "services": ["erp-crm", "erp-finance"]
  }
}
```

```json
{
  "type": "unsubscribe",
  "channels": ["anomalies"]
}
```

```json
{
  "type": "ping"
}
```

### 2.3 Server-to-Client Messages

**Incident Event:**
```json
{
  "type": "incident",
  "action": "created|updated|resolved|closed",
  "timestamp": "2026-02-24T10:30:00Z",
  "data": {
    "id": "inc-uuid",
    "title": "ERP-CRM Latency Spike",
    "severity": "P2",
    "state": "detected",
    "affected_services": ["erp-crm"],
    "anomaly_count": 3
  }
}
```

**Anomaly Event:**
```json
{
  "type": "anomaly",
  "action": "detected|updated|resolved",
  "timestamp": "2026-02-24T10:30:00Z",
  "data": {
    "id": "anom-uuid",
    "metric": "cpu_usage_percent",
    "service": "erp-crm",
    "score": 0.87,
    "algorithms": {
      "zscore": 0.82,
      "isolation_forest": 0.91,
      "lstm": 0.85
    }
  }
}
```

**Remediation Event:**
```json
{
  "type": "remediation",
  "action": "started|completed|failed|pending_approval",
  "timestamp": "2026-02-24T10:30:00Z",
  "data": {
    "id": "remed-uuid",
    "playbook_name": "Auto-Scale on CPU Spike",
    "action_type": "scale_out",
    "target_service": "erp-crm",
    "status": "completed",
    "duration_ms": 12500
  }
}
```

### 2.4 Protocol Details

| Parameter | Value |
|-----------|-------|
| Heartbeat Interval | 30 seconds (server sends ping) |
| Client Timeout | 60 seconds (disconnect if no pong) |
| Max Message Size | 64 KB |
| Max Subscriptions | 10 channels per connection |
| Reconnect Policy | Client should reconnect with exponential backoff (1s, 2s, 4s, max 30s) |

---

## 3. ML Model Specifications and Accuracy Targets

### 3.1 Anomaly Detection Models

| Model | Type | Training Data | Accuracy Target | Retrain Frequency |
|-------|------|--------------|-----------------|-------------------|
| Z-Score | Statistical | Rolling 1h window | Precision > 85%, Recall > 90% | Continuous (rolling) |
| IQR | Statistical | Rolling 1h window | Precision > 80%, Recall > 85% | Continuous (rolling) |
| Moving Average | Statistical | Rolling 1h window | Precision > 75%, Recall > 80% | Continuous (rolling) |
| Isolation Forest | ML | 7 days historical | Precision > 90%, Recall > 85% | Weekly |
| LSTM | Deep Learning | 30 days historical | Precision > 92%, Recall > 88% | Bi-weekly |
| Ensemble (fused) | Combined | N/A | Precision > 93%, Recall > 90% | N/A |

### 3.2 Forecasting Models

| Model | Horizon | MAE Target | MAPE Target | Retrain Frequency |
|-------|---------|-----------|-------------|-------------------|
| Prophet | 48 hours | < 5% of mean | < 8% | Weekly |
| ARIMA | 24 hours | < 3% of mean | < 5% | Daily |
| N-BEATS | 72 hours | < 7% of mean | < 10% | Bi-weekly |

### 3.3 RCA Model

| Component | Target | Measurement |
|-----------|--------|-------------|
| Causal Inference | Top-1 accuracy > 75% | Correct root cause in first suggestion |
| Causal Inference | Top-3 accuracy > 90% | Correct root cause in top 3 suggestions |
| LLM Summary | Relevance > 90% | Human-rated relevance of generated summary |
| Similar Incident Match | Precision > 80% | Relevant historical incidents surfaced |

---

## 4. Supported Remediation Actions Catalog

| Action ID | Name | Description | Risk Level | Timeout | Rollback |
|-----------|------|-------------|------------|---------|----------|
| `restart_service` | Restart Service | Restart pod(s) for the target service | Medium | 120s | Auto-restart previous version |
| `scale_out` | Scale Out | Increase replica count | Low | 300s | Scale back to original count |
| `scale_in` | Scale In | Decrease replica count | Low | 300s | Scale back to original count |
| `rollback_deploy` | Rollback Deployment | Revert to previous deployment revision | High | 600s | Re-deploy current version |
| `clear_cache` | Clear Cache | Flush DragonflyDB keys for target service | Low | 30s | N/A (cache rebuilds) |
| `dns_failover` | DNS Failover | Switch traffic to backup instance | High | 60s | Revert DNS to primary |
| `config_rollback` | Config Rollback | Restore previous configuration version | Medium | 120s | Re-apply current config |
| `circuit_break` | Circuit Breaker | Enable circuit breaker on failing dependency | Medium | 10s | Disable circuit breaker |
| `rate_limit` | Rate Limit | Apply emergency rate limiting | Low | 10s | Remove rate limit |
| `drain_node` | Drain Node | Drain workloads from a failing node | High | 600s | Uncordon node |

---

## 5. Alert Severity Classification Criteria

| Severity | Label | Criteria | Response SLA | Examples |
|----------|-------|----------|-------------|---------|
| P1 | Critical | Service fully down or data loss risk; multiple modules affected; user-facing impact > 50% | Ack: 5 min, Resolve: 1h | Database cluster failure, auth service down |
| P2 | High | Major feature degraded; single module significantly impacted; user-facing impact 10-50% | Ack: 15 min, Resolve: 4h | High latency on Commerce, memory leak in Finance |
| P3 | Medium | Minor feature degraded; performance below SLA but service functional; impact < 10% | Ack: 1h, Resolve: 24h | Elevated error rate, slow background jobs |
| P4 | Low | Non-urgent issue; potential future problem; informational anomaly | Ack: 4h, Resolve: 7d | Predicted disk exhaustion in 7 days, minor config drift |
| P5 | Info | Informational only; no immediate impact; optimization opportunity | Ack: 24h, Resolve: 30d | Cost optimization suggestion, security best practice |

---

## 6. Tenant Resource Limits

| Resource | Free Tier | Standard Tier | Enterprise Tier |
|----------|-----------|--------------|-----------------|
| Monitored Services | 5 | 20 | Unlimited |
| Events per Second | 1,000 | 10,000 | 100,000 |
| Detection Rules | 10 | 50 | Unlimited |
| Correlation Rules | 5 | 25 | Unlimited |
| Remediation Playbooks | 3 | 15 | Unlimited |
| Concurrent Remediations | 1 | 5 | 20 |
| Data Retention (days) | 7 | 30 | 90 |
| RCA per Day | 5 | 25 | Unlimited |
| Forecast Horizon | 24h | 48h | 72h |
| API Rate Limit (req/min) | 60 | 300 | 3,000 |
| WebSocket Connections | 2 | 10 | 50 |

---

## 7. Performance Targets

| Operation | Target Latency | Percentile | Throughput |
|-----------|---------------|------------|------------|
| Anomaly Detection (single metric) | < 100ms | p99 | 100K events/sec/node |
| Event Correlation | < 500ms | p99 | 10K events/sec |
| Rule Evaluation | < 10ms | p99 | 50K evaluations/sec |
| RCA Completion | < 120s | p95 | 10 concurrent |
| Incident CRUD | < 50ms | p95 | 1K req/sec |
| Topology Query | < 200ms | p95 | 100 req/sec |
| Blast Radius Computation | < 2s | p95 | 20 req/sec |
| Cost Recommendation Generation | < 30s | p95 | Batch (daily) |
| Security Scan (per image) | < 60s | p95 | 10 concurrent |
| WebSocket Message Delivery | < 100ms | p99 | 10K messages/sec |
| Forecasting (single metric) | < 5s | p95 | 50 concurrent |
| API Gateway Overhead | < 5ms | p99 | Transparent |

### Resource Budget per Node

| Component | CPU | Memory | Disk |
|-----------|-----|--------|------|
| Go Gateway | 2 cores | 512 MB | Minimal (stateless) |
| Rust API Core | 4 cores | 2 GB | 10 GB (local cache) |
| Python AI Brain | 4 cores + GPU optional | 8 GB | 20 GB (model artifacts) |
| YugabyteDB (per node) | 4 cores | 8 GB | 100 GB SSD |
| DragonflyDB | 2 cores | 4 GB | N/A (in-memory) |
| RustFS | 2 cores | 2 GB | 500 GB |
