# ERP-AIOps Operational Runbooks

> **Document ID:** ERP-AIOPS-RB-024
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Audience:** SREs, DevOps Engineers, On-Call Engineers

---

## Runbook 1: AIOps Service Health Check

### Purpose

Verify that all AIOps components are healthy and processing data correctly.

### Frequency

Run after deployments, during on-call handoffs, or when dashboard shows stale data.

### Steps

**1. Check Go Gateway Health**

```bash
curl -s http://localhost:8090/health | jq .
# Expected: {"status":"healthy","version":"1.x.x","uptime":"..."}
```

**2. Check Rust API Health**

```bash
curl -s http://localhost:8080/health | jq .
# Expected: {"status":"healthy","version":"1.x.x","db":"connected","cache":"connected"}
```

**3. Check Python AI Brain Health**

```bash
curl -s http://localhost:8001/health | jq .
# Expected: {"status":"healthy","models_loaded":true,"gpu_available":false}
```

**4. Verify Database Connectivity**

```bash
# Check YugabyteDB
curl -s http://localhost:8080/health/db | jq .
# Expected: {"status":"connected","pool_active":5,"pool_idle":15,"latency_ms":2}
```

**5. Verify Cache Connectivity**

```bash
# Check DragonflyDB
curl -s http://localhost:8080/health/cache | jq .
# Expected: {"status":"connected","memory_used":"512MB","keys":45000}
```

**6. Verify Telemetry Ingestion**

```bash
# Check event ingestion rate (should be > 0)
curl -s http://localhost:8080/metrics | grep aiops_events_ingested_total
# Expected: aiops_events_ingested_total{tenant="..."} > 0 and increasing
```

**7. Verify WebSocket Server**

```bash
# Check active WebSocket connections
curl -s http://localhost:8080/health/ws | jq .
# Expected: {"active_connections":N,"messages_per_sec":M}
```

### Escalation

If any component is unhealthy, proceed to the relevant restart runbook below. If multiple components are unhealthy, check underlying infrastructure (Kubernetes node health, network, DNS).

---

## Runbook 2: Restarting Rust API

### Purpose

Restart the Rust API core service when it becomes unresponsive or exhibits degraded performance.

### Prerequisites

- Kubernetes cluster access with appropriate RBAC permissions.
- Confirm the issue is with the Rust API, not the gateway or AI brain.

### Steps

**1. Confirm the Issue**

```bash
# Check pod status
kubectl get pods -l app=aiops-api -n aiops
# Look for: CrashLoopBackOff, OOMKilled, or high restart count

# Check recent logs
kubectl logs -l app=aiops-api -n aiops --tail=100
```

**2. Graceful Restart (Rolling)**

```bash
kubectl rollout restart deployment/aiops-api -n aiops
kubectl rollout status deployment/aiops-api -n aiops --timeout=120s
```

**3. Verify Recovery**

```bash
# Wait 30 seconds for startup
sleep 30

# Health check
curl -s http://localhost:8080/health | jq .

# Verify event processing resumed
curl -s http://localhost:8080/metrics | grep aiops_events_ingested_total
```

**4. If Rolling Restart Fails**

```bash
# Check for resource exhaustion
kubectl top pods -l app=aiops-api -n aiops

# Check for OOM events
kubectl describe pods -l app=aiops-api -n aiops | grep -A 5 "Last State"

# If OOM, increase memory limit
kubectl set resources deployment/aiops-api -n aiops \
  --limits=memory=4Gi --requests=memory=2Gi
```

### Rollback

If the new version is causing issues:

```bash
kubectl rollout undo deployment/aiops-api -n aiops
kubectl rollout status deployment/aiops-api -n aiops --timeout=120s
```

---

## Runbook 3: Restarting Python AI Brain

### Purpose

Restart the Python AI brain service when ML inference fails or models are not loading correctly.

### Steps

**1. Confirm the Issue**

```bash
kubectl get pods -l app=aiops-ai-brain -n aiops
kubectl logs -l app=aiops-ai-brain -n aiops --tail=100

# Check model loading status
curl -s http://localhost:8001/health | jq .models_loaded
```

**2. Graceful Restart**

```bash
kubectl rollout restart deployment/aiops-ai-brain -n aiops
kubectl rollout status deployment/aiops-ai-brain -n aiops --timeout=180s
```

Note: The AI brain takes longer to start (60-90 seconds) because it loads ML models into memory on startup.

**3. Verify Model Loading**

```bash
sleep 90

curl -s http://localhost:8001/health | jq .
# Verify: models_loaded = true

# Test anomaly detection
curl -s -X POST http://localhost:8001/api/v1/detect \
  -H "Content-Type: application/json" \
  -d '{"metric":"cpu_usage","values":[45,46,44,95,93,97],"timestamps":["..."]}' | jq .
```

**4. If Models Fail to Load**

```bash
# Check RustFS connectivity (model artifact storage)
curl -s http://localhost:8001/health/storage | jq .

# Re-download models from RustFS
curl -s -X POST http://localhost:8001/admin/reload-models

# Check disk space
kubectl exec -it $(kubectl get pod -l app=aiops-ai-brain -n aiops -o jsonpath='{.items[0].metadata.name}') \
  -n aiops -- df -h /models
```

---

## Runbook 4: Clearing DragonflyDB Cache

### Purpose

Clear the DragonflyDB cache when stale data is causing incorrect behavior, or after a schema change that invalidates cached data.

### Impact

- Temporary increase in database query load as the cache warms up.
- WebSocket streams may briefly disconnect and reconnect.
- Anomaly detection may produce slightly different scores until rolling statistics are rebuilt (approximately 1 hour).

### Steps

**1. Assess Impact**

Before flushing, check current cache utilization:

```bash
kubectl exec -it $(kubectl get pod -l app=aiops-dragonfly -n aiops -o jsonpath='{.items[0].metadata.name}') \
  -n aiops -- redis-cli INFO memory
```

**2. Selective Flush (Preferred)**

Flush only specific key patterns:

```bash
# Flush anomaly score cache
kubectl exec -it $DRAGONFLY_POD -n aiops -- \
  redis-cli --scan --pattern "anomaly:score:*" | xargs redis-cli DEL

# Flush topology cache
kubectl exec -it $DRAGONFLY_POD -n aiops -- \
  redis-cli --scan --pattern "topology:*" | xargs redis-cli DEL

# Flush rule evaluation cache
kubectl exec -it $DRAGONFLY_POD -n aiops -- \
  redis-cli --scan --pattern "rule:eval:*" | xargs redis-cli DEL
```

**3. Full Flush (Last Resort)**

```bash
kubectl exec -it $DRAGONFLY_POD -n aiops -- redis-cli FLUSHALL

# Verify
kubectl exec -it $DRAGONFLY_POD -n aiops -- redis-cli DBSIZE
# Expected: (integer) 0
```

**4. Monitor Cache Warm-up**

```bash
# Watch cache key count rebuild over the next 10 minutes
watch -n 10 'kubectl exec -it $DRAGONFLY_POD -n aiops -- redis-cli DBSIZE'
```

---

## Runbook 5: Database Migration Procedure

### Purpose

Apply database schema migrations to YugabyteDB.

### Prerequisites

- Database migration files in `ERP-AIOps/migrations/`.
- Database admin credentials.
- Maintenance window communicated to stakeholders.

### Steps

**1. Pre-Migration Backup**

```bash
# Create YugabyteDB snapshot
kubectl exec -it $(kubectl get pod -l app=aiops-yugabyte-master -n aiops -o jsonpath='{.items[0].metadata.name}') \
  -n aiops -- ysql_dump -h localhost -U aiops_admin aiops_db > backup_$(date +%Y%m%d_%H%M%S).sql
```

**2. Review Migration**

```bash
# List pending migrations
ls -la ERP-AIOps/migrations/

# Review the migration SQL
cat ERP-AIOps/migrations/XXX_migration_name.sql
```

**3. Apply Migration (Staging First)**

```bash
# Apply to staging
kubectl exec -it $YUGABYTE_POD -n aiops-staging -- \
  ysqlsh -h localhost -U aiops_admin -d aiops_db -f /migrations/XXX_migration_name.sql

# Verify staging
kubectl exec -it $YUGABYTE_POD -n aiops-staging -- \
  ysqlsh -h localhost -U aiops_admin -d aiops_db -c "\dt"
```

**4. Apply Migration (Production)**

```bash
kubectl exec -it $YUGABYTE_POD -n aiops -- \
  ysqlsh -h localhost -U aiops_admin -d aiops_db -f /migrations/XXX_migration_name.sql
```

**5. Post-Migration Verification**

```bash
# Check table structure
kubectl exec -it $YUGABYTE_POD -n aiops -- \
  ysqlsh -h localhost -U aiops_admin -d aiops_db -c "\dt"

# Run API health check
curl -s http://localhost:8080/health/db | jq .
```

### Rollback

If the migration fails or causes issues, apply the corresponding rollback migration:

```bash
kubectl exec -it $YUGABYTE_POD -n aiops -- \
  ysqlsh -h localhost -U aiops_admin -d aiops_db -f /migrations/XXX_migration_name_rollback.sql
```

---

## Runbook 6: Model Retraining Procedure

### Purpose

Retrain ML models when detection accuracy degrades or when significantly new traffic patterns emerge.

### When to Retrain

- Anomaly detection false positive rate exceeds 20%.
- New ERP modules have been added.
- Significant infrastructure changes (e.g., migration to new cluster).
- Quarterly scheduled retraining.

### Steps

**1. Assess Current Model Performance**

```bash
curl -s http://localhost:8001/api/v1/models/metrics | jq .
# Check: precision, recall, f1_score for each model
```

**2. Export Training Data**

```bash
# Export metrics from VictoriaMetrics (last 30 days)
curl -s "http://victoriametrics:8428/api/v1/export?match[]={__name__=~'.+'}&start=-30d" \
  > training_data_$(date +%Y%m%d).jsonl
```

**3. Trigger Retraining**

```bash
# Retrain Isolation Forest
curl -s -X POST http://localhost:8001/admin/retrain \
  -H "Content-Type: application/json" \
  -d '{"model":"isolation_forest","data_range_days":30}' | jq .

# Retrain LSTM
curl -s -X POST http://localhost:8001/admin/retrain \
  -H "Content-Type: application/json" \
  -d '{"model":"lstm","data_range_days":30}' | jq .
```

**4. Validate New Models**

```bash
# Run validation against holdout set
curl -s -X POST http://localhost:8001/admin/validate \
  -H "Content-Type: application/json" \
  -d '{"model":"isolation_forest"}' | jq .
# Expected: precision > 0.90, recall > 0.85

curl -s -X POST http://localhost:8001/admin/validate \
  -H "Content-Type: application/json" \
  -d '{"model":"lstm"}' | jq .
# Expected: precision > 0.92, recall > 0.88
```

**5. Deploy New Models (Canary)**

```bash
curl -s -X POST http://localhost:8001/admin/deploy \
  -H "Content-Type: application/json" \
  -d '{"model":"isolation_forest","strategy":"canary","traffic_percent":10}' | jq .
```

**6. Monitor and Promote**

Monitor the canary for 24 hours. If metrics are acceptable, promote to 100%:

```bash
curl -s -X POST http://localhost:8001/admin/promote \
  -H "Content-Type: application/json" \
  -d '{"model":"isolation_forest"}' | jq .
```

---

## Runbook 7: Emergency Incident Response

### Purpose

Guide on-call engineers through a high-severity AIOps platform incident.

### Trigger

AIOps platform itself is experiencing a P1 or P2 incident (not an incident detected by AIOps).

### Steps

**1. Assess Scope (First 5 Minutes)**

```bash
# Quick health check of all components
for svc in 8090 8080 8001; do
  echo "Port $svc: $(curl -s -o /dev/null -w '%{http_code}' http://localhost:$svc/health)"
done

# Check Kubernetes pod status
kubectl get pods -n aiops
```

**2. Identify the Failing Component**

| Symptom | Likely Component | Next Step |
|---------|-----------------|-----------|
| Dashboard not loading | Go Gateway | Runbook 2 (restart API) or check Ingress |
| No new anomalies | Python AI Brain | Runbook 3 (restart AI brain) |
| Incidents not updating | Rust API | Runbook 2 (restart API) |
| Stale data everywhere | DragonflyDB | Runbook 4 (clear cache) or check DB |
| All services unhealthy | YugabyteDB | Check database cluster health |

**3. Communicate**

- Post in `#aiops-incidents` Slack channel: "AIOps platform incident in progress. Investigating [component]. ETA for update: 15 min."
- If customer-facing: Update status page.

**4. Mitigate**

Apply the relevant component-specific runbook. If the root cause is unclear:

```bash
# Restart all components in order
kubectl rollout restart deployment/aiops-gateway -n aiops
sleep 10
kubectl rollout restart deployment/aiops-api -n aiops
sleep 10
kubectl rollout restart deployment/aiops-ai-brain -n aiops
```

**5. Verify Recovery**

Run the full health check from Runbook 1. Confirm telemetry ingestion, anomaly detection, and dashboard are functional.

**6. Post-Incident**

- Update Slack with resolution summary.
- Create a post-incident review ticket.
- Document any manual actions taken.

---

## Runbook 8: Performance Degradation Troubleshooting

### Purpose

Diagnose and resolve performance issues in the AIOps platform.

### Symptoms

- API response times > 200ms (normally < 50ms).
- Anomaly detection latency > 500ms (normally < 100ms).
- Dashboard loading slowly.
- WebSocket messages delayed.

### Steps

**1. Identify the Bottleneck**

```bash
# Check API latency metrics
curl -s http://localhost:8080/metrics | grep http_request_duration_seconds

# Check AI brain latency
curl -s http://localhost:8001/metrics | grep inference_duration_seconds

# Check database query latency
curl -s http://localhost:8080/metrics | grep db_query_duration_seconds

# Check cache hit rate
curl -s http://localhost:8080/metrics | grep cache_hit_ratio
```

**2. Common Causes and Remediation**

| Cause | Indicator | Fix |
|-------|-----------|-----|
| Database slow queries | `db_query_duration_seconds` > 100ms | Check for missing indexes, run `ANALYZE` |
| Cache miss storm | `cache_hit_ratio` < 0.5 | Increase cache TTL, check eviction policy |
| Memory pressure | Pod memory usage > 90% | Increase memory limits, restart |
| CPU saturation | Pod CPU > 90% sustained | Scale horizontally, optimize hot paths |
| Network latency | Inter-service latency spikes | Check network policies, DNS resolution |
| Event backlog | `events_pending_processing` growing | Scale Rust API replicas |

**3. Database Performance**

```bash
# Check slow queries
kubectl exec -it $YUGABYTE_POD -n aiops -- \
  ysqlsh -c "SELECT query, calls, mean_time FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 10;"

# Run ANALYZE on key tables
kubectl exec -it $YUGABYTE_POD -n aiops -- \
  ysqlsh -c "ANALYZE incidents; ANALYZE anomalies; ANALYZE rules; ANALYZE events;"
```

---

## Runbook 9: Log Analysis for Debugging

### Purpose

Guide engineers through effective log analysis when troubleshooting AIOps issues.

### Log Locations

| Component | Log Source | Access |
|-----------|-----------|--------|
| Go Gateway | stdout (structured JSON) | `kubectl logs -l app=aiops-gateway -n aiops` |
| Rust API | stdout (structured JSON, tracing) | `kubectl logs -l app=aiops-api -n aiops` |
| Python AI Brain | stdout (structured JSON, uvicorn) | `kubectl logs -l app=aiops-ai-brain -n aiops` |
| Ingested Logs | Quickwit | Quickwit UI or API |

### Common Log Queries

**Find errors in the last hour:**

```bash
kubectl logs -l app=aiops-api -n aiops --since=1h | \
  grep '"level":"ERROR"' | jq -r '[.timestamp, .message, .error] | @tsv'
```

**Trace a specific request:**

```bash
# Using the trace ID from the response header
kubectl logs -l app=aiops-api -n aiops --since=1h | \
  grep '"trace_id":"abc123"' | jq .
```

**Find database connection errors:**

```bash
kubectl logs -l app=aiops-api -n aiops --since=1h | \
  grep -i "connection\|pool\|timeout" | jq .
```

**Check AI brain inference errors:**

```bash
kubectl logs -l app=aiops-ai-brain -n aiops --since=1h | \
  grep '"level":"ERROR"' | jq -r '[.timestamp, .model, .error] | @tsv'
```

### Log Retention

| Level | Retention | Storage |
|-------|-----------|---------|
| ERROR and above | 90 days | Quickwit (indexed) |
| WARN | 30 days | Quickwit (indexed) |
| INFO | 14 days | Quickwit (indexed) |
| DEBUG | 3 days | Local only (not shipped) |
