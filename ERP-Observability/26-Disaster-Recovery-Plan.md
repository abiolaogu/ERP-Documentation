# ERP-Observability Disaster Recovery Plan

> **Document ID:** ERP-OBS-DR-026
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [24-Runbooks.md](./24-Runbooks.md), [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md)

---

## 1. Overview

This document defines the disaster recovery (DR) strategy for ERP-Observability, covering data protection, service recovery, and business continuity procedures for all observability infrastructure components.

### Recovery Objectives

| Metric | Target | Justification |
|---|---|---|
| **RTO (Recovery Time Objective)** | 30 minutes | Observability is critical for detecting issues in other ERP modules. Extended outage means blind operations. |
| **RPO (Recovery Point Objective)** | 5 minutes | Acceptable to lose up to 5 minutes of telemetry data. Metric data can be reconstructed from Prometheus WAL. Log data in transit is buffered by Fluent-Bit and OTel Collector. |
| **MTTR (Mean Time to Recovery)** | < 45 minutes | Including detection (5 min), diagnosis (10 min), and recovery (30 min). |

### Disaster Classification

| Level | Description | Example | Response |
|---|---|---|---|
| **D1 - Component Failure** | Single component unavailable | Quickwit searcher pod crash | Automatic (K8s self-healing) |
| **D2 - Service Degradation** | Multiple components degraded | RustFS partial failure causing Quickwit index corruption | Manual intervention, runbook |
| **D3 - Zone Failure** | Entire availability zone lost | AZ outage | Cross-AZ failover |
| **D4 - Region Failure** | Entire region unavailable | Region-wide cloud outage | Cross-region DR activation |

---

## 2. Component Recovery Strategies

### 2.1 VictoriaMetrics Data Replication

**Architecture:**

VictoriaMetrics cluster mode provides built-in replication:

- **vminsert:** Stateless write proxies, 3 replicas across AZs. No data recovery needed.
- **vmselect:** Stateless query proxies, 3 replicas. No data recovery needed.
- **vmstorage:** Stateful storage nodes with replication factor 2.

**Backup Strategy:**

```bash
# Automated daily snapshot to RustFS
# Cron job running on vmstorage nodes

# Create snapshot
curl -sf "http://vmstorage-0:8482/snapshot/create" | jq .
# Response: {"status":"ok","snapshot":"20260224T100000Z-abc123"}

# Upload to RustFS
aws s3 sync --endpoint-url http://rustfs:9000 \
  /vmstorage-data/snapshots/20260224T100000Z-abc123 \
  s3://erp-victoriametrics-snapshots/20260224T100000Z-abc123/

# Cleanup old snapshots (retain 7 days)
curl -sf "http://vmstorage-0:8482/snapshot/delete?snapshot=<old-snapshot>"
```

**Recovery Procedure:**

```bash
# Step 1: Stop vmstorage pods
kubectl scale statefulset vmstorage -n erp-observability --replicas=0

# Step 2: Download latest snapshot from RustFS
LATEST=$(aws s3 ls --endpoint-url http://rustfs:9000 \
  s3://erp-victoriametrics-snapshots/ | sort | tail -1 | awk '{print $2}')

aws s3 sync --endpoint-url http://rustfs:9000 \
  "s3://erp-victoriametrics-snapshots/${LATEST}" \
  /vmstorage-data/data/

# Step 3: Restart vmstorage pods
kubectl scale statefulset vmstorage -n erp-observability --replicas=3

# Step 4: Verify data integrity
curl -s "http://vmselect:8481/select/0/prometheus/api/v1/query?query=up" | jq '.data.result | length'
```

**Recovery Time:** 15-20 minutes (snapshot download + restart).

**RPO:** Last snapshot interval (daily = 24 hours max data loss for metrics older than Prometheus WAL retention).

### 2.2 Quickwit Index Backup to RustFS

**Architecture:**

Quickwit is natively S3-backed. All index data resides in RustFS:

- **Metastore:** `s3://erp-quickwit-metastore/` -- Index metadata, split metadata.
- **Index Data:** `s3://erp-quickwit-indexes/` -- Actual index splits (immutable files).

Since Quickwit data is already in RustFS, the primary DR concern is RustFS availability and cross-region replication.

**Backup Strategy:**

```bash
# RustFS bucket replication to secondary region
# Configured via RustFS bucket replication rules

# Primary region: africa-west-1
# DR region: europe-west-1

# RustFS replication configuration
mc admin bucket remote add primary/erp-quickwit-metastore \
  https://rustfs-dr.eu-west-1.erp.io/erp-quickwit-metastore \
  --service replication

mc admin bucket remote add primary/erp-quickwit-indexes \
  https://rustfs-dr.eu-west-1.erp.io/erp-quickwit-indexes \
  --service replication
```

**Recovery Procedure (Index Corruption):**

```bash
# Step 1: Identify corrupted index
curl -s "http://quickwit-control:7280/api/v1/indexes" | jq '.[] | {index_id, num_splits, num_docs}'

# Step 2: Delete corrupted index
curl -X DELETE "http://quickwit-control:7280/api/v1/indexes/erp-logs"

# Step 3: Recreate index from schema
curl -X POST "http://quickwit-control:7280/api/v1/indexes" \
  -H "Content-Type: application/yaml" \
  --data-binary @configs/quickwit/index-erp-logs.yaml

# Step 4: Trigger re-ingestion from Fluent-Bit buffer
kubectl rollout restart daemonset fluent-bit -n erp-observability

# Note: Historical data in RustFS splits is preserved. Only the index metadata is recreated.
# Quickwit will automatically discover existing splits in S3.
```

**Recovery Time:** 5-10 minutes for index metadata recreation. Historical data available immediately if S3 splits are intact.

### 2.3 YugabyteDB Failover

**Architecture:**

YugabyteDB cluster: 3 masters + 3 tablet servers across 3 AZs.

- **Replication factor:** 3 (synchronous Raft consensus).
- **Automatic failover:** If a tablet server fails, surviving replicas elect a new leader within seconds.
- **Multi-region:** Configured with geo-partitioned tables for cross-region deployment.

**Backup Strategy:**

```bash
# Automated daily backup via ysql_dump
ysql_dump -h yb-master-0 -p 5433 -U erp -d erp_observability \
  --format=custom --file=/backups/erp_observability_$(date +%Y%m%d).dump

# Upload to RustFS
aws s3 cp --endpoint-url http://rustfs:9000 \
  /backups/erp_observability_$(date +%Y%m%d).dump \
  s3://erp-observability-backups/yugabytedb/

# Retain 30 days of backups
aws s3 ls --endpoint-url http://rustfs:9000 \
  s3://erp-observability-backups/yugabytedb/ | \
  while read -r line; do
    createDate=$(echo "$line" | awk '{print $1}')
    if [[ $(date -d "$createDate" +%s) -lt $(date -d "30 days ago" +%s) ]]; then
      fileName=$(echo "$line" | awk '{print $4}')
      aws s3 rm --endpoint-url http://rustfs:9000 \
        "s3://erp-observability-backups/yugabytedb/$fileName"
    fi
  done
```

**Recovery Procedure (Full Cluster Rebuild):**

```bash
# Step 1: Deploy fresh YugabyteDB cluster
kubectl apply -f infra/k8s/yugabytedb/

# Step 2: Wait for cluster initialization
kubectl exec -n erp-data yb-master-0 -- yb-admin list_all_masters

# Step 3: Download latest backup
aws s3 cp --endpoint-url http://rustfs:9000 \
  s3://erp-observability-backups/yugabytedb/erp_observability_latest.dump \
  /tmp/restore.dump

# Step 4: Restore
pg_restore -h yb-tserver-0 -p 5433 -U erp -d erp_observability \
  --clean --if-exists /tmp/restore.dump

# Step 5: Verify
psql -h yb-tserver-0 -p 5433 -U erp -d erp_observability \
  -c "SELECT count(*) FROM alert_rules; SELECT count(*) FROM dashboards;"
```

**Recovery Time:** 10-15 minutes for full restore.

**RPO:** Last backup interval (daily = up to 24 hours for alert rules and dashboards).

### 2.4 Grafana Dashboard Export

**Backup Strategy:**

Grafana dashboards are stored in two places:

1. **Git repository:** Pre-built dashboards in `configs/grafana/dashboards/*.json` (source of truth for default dashboards).
2. **YugabyteDB:** Custom tenant dashboards stored in the `dashboards` table (backed up with YugabyteDB).

```bash
# Export all Grafana dashboards via API
for uid in $(curl -s "http://grafana:3001/api/search?type=dash-db" \
  -H "Authorization: Bearer ${GRAFANA_API_KEY}" | jq -r '.[].uid'); do
  curl -s "http://grafana:3001/api/dashboards/uid/$uid" \
    -H "Authorization: Bearer ${GRAFANA_API_KEY}" \
    > "/backups/grafana/dashboard-${uid}.json"
done

# Upload to RustFS
aws s3 sync --endpoint-url http://rustfs:9000 \
  /backups/grafana/ \
  s3://erp-observability-backups/grafana/$(date +%Y%m%d)/
```

**Recovery Procedure:**

```bash
# Step 1: Restore Grafana provisioned dashboards from Git
kubectl create configmap grafana-dashboards \
  --from-file=configs/grafana/dashboards/ \
  -n erp-observability --dry-run=client -o yaml | kubectl apply -f -

kubectl rollout restart deployment grafana -n erp-observability

# Step 2: Restore custom dashboards from backup
for f in /backups/grafana/dashboard-*.json; do
  curl -X POST "http://grafana:3001/api/dashboards/db" \
    -H "Authorization: Bearer ${GRAFANA_API_KEY}" \
    -H "Content-Type: application/json" \
    -d "{\"dashboard\": $(jq '.dashboard' "$f"), \"overwrite\": true}"
done
```

---

## 3. Cross-Region Recovery Procedure

### Scenario: Primary Region (africa-west-1) Complete Outage

**Pre-conditions:**
- DR region (europe-west-1) has standby infrastructure deployed.
- RustFS bucket replication is active.
- YugabyteDB xCluster replication is configured.
- DNS is managed via Route 53 / CloudFlare with health checks.

**Recovery Steps:**

| Step | Action | Time | Cumulative |
|---|---|---|---|
| 1 | Detect outage (automated health check failure) | 2 min | 2 min |
| 2 | Alert SRE team (PagerDuty) | 1 min | 3 min |
| 3 | Confirm primary region is down (manual verification) | 2 min | 5 min |
| 4 | Decision to activate DR (SRE Lead approval) | 3 min | 8 min |
| 5 | Switch DNS to DR region | 2 min | 10 min |
| 6 | Promote DR YugabyteDB to primary | 3 min | 13 min |
| 7 | Verify Quickwit indexes in DR RustFS | 5 min | 18 min |
| 8 | Scale up DR application deployments | 5 min | 23 min |
| 9 | Verify health endpoints | 2 min | 25 min |
| 10 | Verify Grafana dashboards load | 3 min | 28 min |
| 11 | Notify stakeholders | 2 min | 30 min |

**Total estimated recovery time: 30 minutes** (within RTO target).

---

## 4. Data Loss Assessment

| Component | Data Type | RPO | Backup Frequency | Acceptable Loss |
|---|---|---|---|---|
| VictoriaMetrics | Metrics (time series) | 5 min | Daily snapshot + Prometheus WAL | Yes (can re-scrape) |
| Quickwit erp-logs | Application logs | 5 min | Continuous (S3-native) | Yes (logs are transient) |
| Quickwit erp-traces | Traces | 5 min | Continuous (S3-native) | Yes (traces are transient) |
| Quickwit erp-audit | Audit logs | 1 min | Continuous (S3-native) | Minimal (compliance req) |
| YugabyteDB | Alert rules, dashboards | 24 hours | Daily backup | Moderate (can recreate) |
| Grafana | Dashboard configs | 24 hours | Daily export + Git | Low (Git is source of truth) |
| DragonflyDB | Cache data | N/A | No backup (ephemeral) | None (cache is ephemeral) |

---

## 5. DR Testing Schedule

| Test Type | Frequency | Description |
|---|---|---|
| Backup Verification | Weekly | Verify backup files exist and are non-zero size |
| Restore Test | Monthly | Restore YugabyteDB backup to test cluster |
| Component Failover | Monthly | Kill a Quickwit indexer/searcher and verify recovery |
| Zone Failover | Quarterly | Simulate AZ failure, verify cross-AZ recovery |
| Full DR Drill | Semi-annually | Activate DR region, verify all services, measure RTO/RPO |

### DR Test Procedure

```bash
# 1. Weekly backup verification
./scripts/verify-backups.sh
# Checks: RustFS bucket sizes, snapshot timestamps, dump file integrity

# 2. Monthly restore test
./scripts/dr-restore-test.sh
# Restores YugabyteDB to test cluster, runs SQL checks, reports delta

# 3. Quarterly zone failover
./scripts/dr-zone-failover.sh --zone az-1 --dry-run
./scripts/dr-zone-failover.sh --zone az-1 --execute

# 4. Semi-annual full DR drill
./scripts/dr-full-drill.sh --activate-dr-region
```

---

## 6. Communication Plan

### During a DR Event

| Audience | Channel | Timing | Message |
|---|---|---|---|
| SRE Team | PagerDuty + Slack #erp-critical | Immediate | Automated alert |
| Engineering Teams | Slack #erp-incidents | Within 5 min | Impact assessment |
| Product/Business | Email + Slack #erp-status | Within 15 min | Service impact, ETA |
| Customers | Status page | Within 20 min | Public status update |

### Post-Recovery

1. Update status page to "Resolved."
2. Send all-clear notification to all channels.
3. Schedule post-incident review within 48 hours.
4. Document lessons learned and update DR plan if needed.
