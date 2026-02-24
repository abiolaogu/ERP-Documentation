# ERP-AIOps Disaster Recovery Plan

> **Document ID:** ERP-AIOPS-DR-026
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [24-Runbooks.md](./24-Runbooks.md), [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md)

---

## 1. Service Priority Matrix

Not all AIOps components are equally critical. The priority matrix defines the order in which services should be restored during a disaster recovery event.

### Priority Tiers

| Priority | Component | RTO | RPO | Justification |
|----------|-----------|-----|-----|---------------|
| P0 | Go Gateway | 5 min | 0 (stateless) | Entry point for all traffic; without it, the platform is completely inaccessible |
| P0 | YugabyteDB | 15 min | < 1 min | Primary data store; all state depends on it |
| P1 | Rust API Core | 10 min | 0 (stateless) | Core business logic; incidents and rules stop processing without it |
| P1 | DragonflyDB | 10 min | < 5 min | Cache and streaming; degraded performance without it but not data loss |
| P2 | Python AI Brain | 30 min | 0 (stateless) | ML inference; anomaly detection pauses but alerting via rules continues |
| P2 | RustFS | 1 hour | < 1 hour | Object storage for models and reports; models cached locally in AI brain |
| P3 | Frontend | 15 min | 0 (static) | Dashboard; API remains accessible for integrations |

### RTO/RPO Definitions

- **RTO (Recovery Time Objective):** Maximum acceptable time to restore service.
- **RPO (Recovery Point Objective):** Maximum acceptable data loss measured in time.

---

## 2. Data Backup Strategy

### 2.1 YugabyteDB Backups

| Backup Type | Frequency | Retention | Storage Location |
|-------------|-----------|-----------|-----------------|
| Continuous WAL Replication | Real-time | 72 hours | Cross-region YugabyteDB replica |
| Full Logical Backup | Daily (02:00 UTC) | 30 days | RustFS (S3-compatible) in separate region |
| Point-in-Time Recovery | Continuous | 72 hours | WAL archive |
| Weekly Snapshot | Sunday 04:00 UTC | 90 days | Cold storage (Glacier-equivalent) |

**Backup Automation:**

```bash
# Daily logical backup (automated via CronJob)
ysql_dump -h $YB_HOST -U aiops_admin aiops_db | \
  gzip | \
  aws s3 cp - s3://aiops-backups/db/daily/aiops_$(date +%Y%m%d).sql.gz

# Verify backup integrity
aws s3 cp s3://aiops-backups/db/daily/aiops_$(date +%Y%m%d).sql.gz - | \
  gunzip | \
  head -5  # Should show valid SQL
```

### 2.2 ML Model Weight Backups

| Artifact | Frequency | Retention | Storage |
|----------|-----------|-----------|---------|
| Isolation Forest models | On retrain | Last 10 versions | RustFS + cross-region copy |
| LSTM model weights | On retrain | Last 10 versions | RustFS + cross-region copy |
| Prophet models | On retrain | Last 10 versions | RustFS + cross-region copy |
| Ensemble configuration | On change | Last 20 versions | Git repository |
| Training data snapshots | Weekly | Last 4 weeks | Cold storage |

**Model Versioning:**

```
rustfs://aiops-models/
├── isolation_forest/
│   ├── v1.0.0/model.pkl          (2026-01-15)
│   ├── v1.1.0/model.pkl          (2026-02-01)
│   └── v1.2.0/model.pkl          (2026-02-15)  ← current
├── lstm/
│   ├── v1.0.0/weights.pt         (2026-01-20)
│   └── v1.1.0/weights.pt         (2026-02-10)  ← current
└── prophet/
    └── v1.0.0/model.json         (2026-02-01)  ← current
```

### 2.3 Configuration Backups

All configuration is stored as code in Git:

- Helm chart values (deployment configuration)
- OTel Collector configuration
- Rule definitions (exported nightly as JSON)
- Playbook definitions (exported nightly as JSON)
- Integration configurations (encrypted)

---

## 3. Recovery Procedures

### 3.1 Single Component Failure

**Scenario:** One AIOps component crashes but infrastructure is intact.

**Procedure:**

1. Kubernetes automatically restarts the failed pod (restartPolicy: Always).
2. If restart loops occur (CrashLoopBackOff), follow the relevant runbook in [24-Runbooks.md](./24-Runbooks.md).
3. If the pod cannot start on the current node, Kubernetes reschedules to another node.
4. Estimated recovery: 1-5 minutes (automatic).

### 3.2 Database Failure

**Scenario:** YugabyteDB primary becomes unavailable.

**Procedure:**

1. **Automatic Failover (< 30 seconds):** YugabyteDB's Raft consensus protocol automatically promotes a replica to primary.
2. **If automatic failover fails:**
   ```bash
   # Check cluster status
   kubectl exec -it yb-master-0 -n aiops -- yb-admin -master_addresses yb-master-0:7100 list_all_masters

   # Force leader election if stuck
   kubectl exec -it yb-master-0 -n aiops -- yb-admin -master_addresses yb-master-0:7100 change_master_config REMOVE_SERVER yb-master-2
   ```
3. **If cluster is unrecoverable:**
   ```bash
   # Restore from latest backup
   kubectl exec -it yb-tserver-0 -n aiops -- ysqlsh -c "DROP DATABASE IF EXISTS aiops_db;"
   kubectl exec -it yb-tserver-0 -n aiops -- ysqlsh -c "CREATE DATABASE aiops_db;"
   aws s3 cp s3://aiops-backups/db/daily/aiops_latest.sql.gz - | \
     gunzip | \
     kubectl exec -i yb-tserver-0 -n aiops -- ysqlsh -d aiops_db
   ```

### 3.3 Cache Failure

**Scenario:** DragonflyDB becomes unavailable.

**Procedure:**

1. The Rust API degrades gracefully, falling back to direct database queries.
2. Performance will be degraded (API latency increases 3-5x) but functionality is preserved.
3. Restart DragonflyDB:
   ```bash
   kubectl rollout restart statefulset/aiops-dragonfly -n aiops
   ```
4. Cache warms up automatically over 10-15 minutes as requests flow through.

### 3.4 Complete Region Failure

**Scenario:** The entire primary region is unavailable.

**Procedure:**

1. **Activate DR Region (RTO: 30 minutes)**
   ```bash
   # Switch DNS to DR region
   aws route53 change-resource-record-sets --hosted-zone-id $ZONE_ID \
     --change-batch '{"Changes":[{"Action":"UPSERT","ResourceRecordSet":{"Name":"aiops.erp.internal","Type":"CNAME","TTL":60,"ResourceRecordValues":[{"Value":"aiops-dr.erp.internal"}]}}]}'
   ```

2. **Verify DR Region Services**
   ```bash
   curl -s https://aiops-dr.erp.internal/health | jq .
   ```

3. **Restore Database from Cross-Region Replica**
   - YugabyteDB cross-region replica should be in sync (RPO < 1 minute).
   - Promote the DR replica to primary.

4. **Deploy Latest AI Models**
   ```bash
   # Models are pre-synced to DR region RustFS
   curl -s -X POST http://aiops-dr:8001/admin/reload-models | jq .
   ```

5. **Verify Full Functionality**
   - Run health checks on all components.
   - Verify telemetry ingestion from ERP modules (update OTel Collector endpoints if needed).
   - Confirm anomaly detection and incident creation are working.

---

## 4. Failover to Degraded Mode

When full recovery is not immediately possible, AIOps can operate in degraded mode with reduced functionality.

### Degraded Mode Levels

| Level | ML Available | Rules Available | Alerting | Remediation | Dashboard |
|-------|-------------|-----------------|----------|-------------|-----------|
| Full | Yes | Yes | Yes | Yes | Yes |
| Degraded L1 | No | Yes | Yes | Manual only | Yes |
| Degraded L2 | No | Yes | Yes | No | Limited |
| Emergency | No | No | Webhook only | No | No |

### Activating Degraded Mode

**Degraded L1 (ML Down, Rules Active):**

```bash
# Set feature flag to disable ML-dependent features
kubectl set env deployment/aiops-api -n aiops FEATURE_ML_ENABLED=false

# Anomaly detection falls back to static threshold rules
# RCA is disabled; incident creation continues via rules
```

**Degraded L2 (API Partially Available):**

```bash
# Enable read-only mode
kubectl set env deployment/aiops-api -n aiops FEATURE_READONLY=true

# Dashboard shows cached data; no new incidents created
# Existing incidents can be viewed but not updated
```

**Emergency (Webhook-Only Alerting):**

When the entire AIOps platform is down, pre-configured OTel Collector alerting rules in the ERP-Observability module can send basic alerts directly to Slack/PagerDuty, bypassing AIOps entirely.

### Returning to Full Mode

```bash
# Re-enable ML
kubectl set env deployment/aiops-api -n aiops FEATURE_ML_ENABLED=true

# Re-enable read-write mode
kubectl set env deployment/aiops-api -n aiops FEATURE_READONLY=false

# Verify AI brain is healthy and models loaded
curl -s http://localhost:8001/health | jq .
```

---

## 5. Communication Plan

### Notification Matrix

| Severity | Who to Notify | Channel | Frequency |
|----------|--------------|---------|-----------|
| P0 (Full Outage) | VP Engineering, SRE Lead, All SREs | Slack + PagerDuty + Email | Every 15 min |
| P1 (Major Degradation) | SRE Lead, On-Call SREs | Slack + PagerDuty | Every 30 min |
| P2 (Partial Degradation) | On-Call SRE | Slack | Every 1 hour |
| P3 (Minor Issue) | On-Call SRE | Slack | On resolution |

### Communication Templates

**Initial Notification:**
```
[AIOps DR Event - P{severity}]
Status: {Investigating|Mitigating|Resolved}
Impact: {description of user impact}
Component: {affected component}
Start Time: {timestamp UTC}
ETA: {estimated resolution time}
Incident Commander: {name}
Next Update: {timestamp UTC}
```

**Update Template:**
```
[AIOps DR Update #{number}]
Status: {current status}
Progress: {what has been done since last update}
Remaining: {what still needs to be done}
ETA: {revised estimate}
Next Update: {timestamp UTC}
```

---

## 6. Post-Incident Review Process

### Timeline

| Action | Deadline |
|--------|----------|
| Incident resolved | T+0 |
| Initial incident report | T+24 hours |
| Post-incident review meeting | T+3 business days |
| Final report with action items | T+5 business days |
| Action items completed | T+30 days |

### Review Template

1. **Summary**: One paragraph describing what happened.
2. **Impact**: User impact, duration, affected tenants.
3. **Timeline**: Minute-by-minute reconstruction of events.
4. **Root Cause**: Technical root cause analysis.
5. **What Went Well**: Effective responses and detections.
6. **What Went Wrong**: Gaps in monitoring, runbooks, or response.
7. **Action Items**: Specific, assignable tasks with deadlines.
8. **Process Improvements**: Changes to DR procedures.

### DR Plan Testing Schedule

| Test Type | Frequency | Scope |
|-----------|-----------|-------|
| Tabletop Exercise | Quarterly | Walk through DR scenarios with the team |
| Component Failover | Monthly | Intentionally kill one component, verify recovery |
| Database Recovery | Quarterly | Restore from backup, verify data integrity |
| Full DR Activation | Annually | Activate DR region, run full test workload |
| Chaos Engineering | Monthly | Random failure injection via Chaos Mesh |
