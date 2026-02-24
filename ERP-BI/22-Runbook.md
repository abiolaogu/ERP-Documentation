# ERP-BI Operational Runbook

| Field | Value |
|---|---|
| Module | ERP-BI |
| Audience | SRE / DevOps |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Service Health Checks

```bash
# Check all services
for svc in dashboard-service report-service data-modeling-service query-engine data-warehouse-service alert-service nlq-service; do
  echo "Checking $svc..."
  curl -s http://$svc:8080/healthz | jq .
done
```

Expected response:
```json
{"status": "healthy", "module": "ERP-BI", "service": "<service-name>"}
```

---

## 2. Common Incidents

### 2.1 ClickHouse Unresponsive

**Symptoms**: Dashboard queries timing out, 503 errors from Query Engine

**Diagnosis**:
```bash
# Check ClickHouse process
kubectl exec -it clickhouse-0 -n bi-data -- clickhouse-client -q "SELECT 1"

# Check system metrics
kubectl exec -it clickhouse-0 -n bi-data -- clickhouse-client -q \
  "SELECT metric, value FROM system.metrics WHERE metric IN ('Query', 'MergesMutations')"

# Check query log for stuck queries
kubectl exec -it clickhouse-0 -n bi-data -- clickhouse-client -q \
  "SELECT query_id, elapsed, query FROM system.processes ORDER BY elapsed DESC LIMIT 10"
```

**Resolution**:
1. Kill long-running queries: `KILL QUERY WHERE elapsed > 60`
2. Restart ClickHouse pod if unresponsive: `kubectl delete pod clickhouse-0 -n bi-data`
3. Scale up Query Engine replicas to reduce per-instance load
4. Review governor limits

### 2.2 CDC Ingestion Lag

**Symptoms**: Data in dashboards is stale, DWS lag metric > 5 minutes

**Diagnosis**:
```bash
# Check NATS consumer lag
nats consumer info erp-cdc bi-ingestion

# Check DWS logs
kubectl logs -l app=data-warehouse-service -n bi-system --tail=100

# Check dead letter queue
kubectl exec -it data-warehouse-0 -- curl localhost:8084/v1/data-warehouse/dlq/count
```

**Resolution**:
1. Scale DWS replicas: `kubectl scale deployment data-warehouse-service --replicas=4 -n bi-system`
2. Check for schema mismatch errors in logs
3. Replay failed events from DLQ after fixing root cause
4. Verify NATS JetStream health

### 2.3 NLQ Service Failure

**Symptoms**: NLQ queries returning errors, ERP-AI unreachable

**Diagnosis**:
```bash
# Check NLQ health
curl http://nlq-service:8080/healthz

# Check ERP-AI connectivity
curl http://erp-ai:8080/healthz

# Check NLQ logs
kubectl logs -l app=nlq-service -n bi-system --tail=50
```

**Resolution**:
1. Verify ERP-AI service is running
2. Check AI_URL environment variable
3. Restart NLQ service if stuck
4. Verify Claude API quota/billing

### 2.4 Report Delivery Failure

**Symptoms**: Scheduled reports not arriving, delivery error count increasing

**Diagnosis**:
```bash
# Check report service logs
kubectl logs -l app=report-service -n bi-system --tail=50

# Check SMTP connectivity
kubectl exec -it report-service-0 -- nc -zv smtp.example.com 587

# Check Slack webhook
kubectl exec -it report-service-0 -- curl -X POST $SLACK_WEBHOOK -d '{"text":"test"}'
```

**Resolution**:
1. Verify SMTP credentials
2. Check Slack webhook URL validity
3. Verify S3 bucket access for file storage
4. Re-trigger failed report executions

---

## 3. Scaling Procedures

### 3.1 Horizontal Scaling

```bash
# Scale Query Engine for high dashboard load
kubectl scale deployment query-engine --replicas=8 -n bi-system

# Scale DWS for ingestion backlog
kubectl scale deployment data-warehouse-service --replicas=4 -n bi-system

# Scale Report Service for batch generation
kubectl scale deployment report-service --replicas=4 -n bi-system
```

### 3.2 ClickHouse Scaling

```bash
# Add a read replica (update StatefulSet)
kubectl patch statefulset clickhouse -n bi-data -p '{"spec":{"replicas":4}}'

# Verify cluster health
kubectl exec -it clickhouse-0 -n bi-data -- clickhouse-client -q \
  "SELECT host_name, is_active FROM system.clusters"
```

---

## 4. Maintenance Procedures

### 4.1 ClickHouse Partition Management

```sql
-- Check partition sizes
SELECT partition, formatReadableSize(sum(bytes_on_disk)) as size
FROM system.parts
WHERE table = 'fact_sales' AND active
GROUP BY partition ORDER BY partition;

-- Drop old partitions (older than 5 years)
ALTER TABLE fact_sales DROP PARTITION '202001';
```

### 4.2 Redis Cache Flush

```bash
# Flush cache for specific tenant
redis-cli -c KEYS "tenant_001:*" | xargs redis-cli DEL

# Flush all BI cache (use cautiously)
redis-cli -c FLUSHDB
```

### 4.3 Database Backup

```bash
# PostgreSQL backup
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql

# ClickHouse backup
clickhouse-backup create bi_backup_$(date +%Y%m%d)
```

---

## 5. Emergency Procedures

### 5.1 Service Degradation Mode

If ClickHouse is completely unavailable, enable degradation mode:
1. Query Engine serves from Redis cache only (stale data)
2. Dashboards show "Data may be stale" banner
3. NLQ is disabled
4. Alerts are paused

### 5.2 Data Recovery

1. Identify last known good state from backups
2. Restore ClickHouse from backup
3. Replay CDC events from NATS JetStream
4. Verify data consistency
5. Re-enable services
