# ERP-CRM Disaster Recovery Plan

## 1. Recovery Objectives

| Metric | Target | Justification |
|--------|--------|--------------|
| RTO (Recovery Time Objective) | < 1 hour | Business operations resume within 1 hour |
| RPO (Recovery Point Objective) | < 5 minutes | Maximum 5 minutes of data loss |
| MTTR (Mean Time To Recovery) | < 30 minutes | Average recovery time |

## 2. Disaster Scenarios

```mermaid
graph TB
    subgraph "Scenario Classification"
        S1["S1: Single Pod Failure<br/>Impact: Minimal<br/>Recovery: Automatic"]
        S2["S2: Database Failure<br/>Impact: Critical<br/>Recovery: 15-30 min"]
        S3["S3: Full Cluster Failure<br/>Impact: Total<br/>Recovery: 30-60 min"]
        S4["S4: Data Corruption<br/>Impact: Critical<br/>Recovery: 30-60 min"]
        S5["S5: Region/DC Failure<br/>Impact: Total<br/>Recovery: 1-4 hours"]
        S6["S6: Ransomware/Security<br/>Impact: Total<br/>Recovery: 2-8 hours"]
    end
```

## 3. Backup Strategy

### 3.1 PostgreSQL Backups

```mermaid
flowchart LR
    PG4["PostgreSQL 16"] --> CONTINUOUS["WAL Archiving<br/>(Continuous)"]
    PG4 --> DAILY["pg_dump<br/>(Daily Full)"]
    PG4 --> HOURLY["pg_basebackup<br/>(Hourly Incremental)"]

    CONTINUOUS --> S3_STORE["Object Storage<br/>(Encrypted, Off-site)"]
    DAILY --> S3_STORE
    HOURLY --> S3_STORE

    S3_STORE --> RETENTION["Retention Policy<br/>Daily: 30 days<br/>Weekly: 12 weeks<br/>Monthly: 12 months"]
```

**Backup Schedule:**

| Type | Frequency | Retention | Storage |
|------|-----------|-----------|---------|
| WAL Archive | Continuous | 7 days | Off-site object storage |
| Full Backup | Daily at 02:00 UTC | 30 days | Off-site object storage |
| Incremental | Hourly | 48 hours | Local + off-site |
| Point-in-Time | On demand | Until restored | Temporary |

**Backup Commands:**

```bash
# Full backup
pg_dump -Fc -h localhost -U postgres -d crm > /backups/crm_$(date +%Y%m%d_%H%M%S).dump

# Verify backup
pg_restore --list /backups/crm_latest.dump

# WAL archiving (postgresql.conf)
archive_mode = on
archive_command = 'test ! -f /backup/wal/%f && cp %p /backup/wal/%f'
```

### 3.2 Configuration Backups

```bash
# Backup all configuration
kubectl get configmap -n crm -o yaml > /backups/configmaps.yaml
kubectl get secret -n crm -o yaml > /backups/secrets.yaml
kubectl get deployment -n crm -o yaml > /backups/deployments.yaml

# Backup Pulsar topics configuration
cp eventing/pulsar/topics.yaml /backups/pulsar-topics.yaml

# Backup Quickwit index configuration
cp observability/quickwit/index-config.yaml /backups/quickwit-config.yaml
```

### 3.3 Event Store Backups

Pulsar provides built-in message retention. Configure:

```yaml
# Topic-level retention
retention_time_in_minutes: 10080  # 7 days
retention_size_in_mb: 10240       # 10 GB
```

## 4. Recovery Procedures

### 4.1 S1: Single Pod Failure

**Impact:** Minimal -- other replicas serve traffic
**Recovery:** Automatic via Kubernetes

```mermaid
flowchart TB
    FAIL1([Pod Crash]) --> K8S_DETECT["Kubernetes<br/>Detects Failure"]
    K8S_DETECT --> RESTART["Auto-Restart Pod<br/>(RestartPolicy: Always)"]
    RESTART --> HEALTH_PASS{Health<br/>Check?}
    HEALTH_PASS -->|Pass| TRAFFIC2["Pod Rejoins<br/>Load Balancer"]
    HEALTH_PASS -->|Fail 3x| CRASHLOOP["CrashLoopBackOff<br/>See RB-010"]
```

**Actions:** None required. Monitor pod restart count.

### 4.2 S2: Database Failure

**Impact:** Critical -- all CRUD operations fail
**Recovery:** 15-30 minutes

```bash
# Step 1: Assess damage
kubectl exec -n data statefulset/postgresql -- pg_isready -U postgres

# Step 2: If PostgreSQL is down but data intact
kubectl delete pod -n data postgresql-0  # Force recreate
kubectl wait --for=condition=ready pod/postgresql-0 -n data --timeout=120s

# Step 3: If data is corrupted, restore from backup
# Stop application traffic
kubectl scale deployment crm-core -n crm --replicas=0

# Restore from latest full backup
pg_restore -h localhost -U postgres -d crm --clean --create /backups/crm_latest.dump

# If point-in-time recovery needed
# Restore base backup, then replay WAL to target time
pg_restore --target-time="2026-02-23 10:00:00" ...

# Resume application traffic
kubectl scale deployment crm-core -n crm --replicas=2
```

### 4.3 S3: Full Cluster Failure

**Impact:** Total outage
**Recovery:** 30-60 minutes

```mermaid
flowchart TB
    CLUSTER_FAIL([Cluster Failure]) --> ASSESS["Assess Cluster<br/>State"]
    ASSESS --> NEW_CLUSTER{Can Recover<br/>Existing?}
    NEW_CLUSTER -->|Yes| REPAIR["Repair Cluster<br/>Nodes"]
    NEW_CLUSTER -->|No| PROVISION["Provision New<br/>Cluster"]

    REPAIR --> VERIFY_STORAGE["Verify Persistent<br/>Volumes"]
    PROVISION --> RESTORE_CONFIG["Restore K8s<br/>Configurations"]

    VERIFY_STORAGE --> DEPLOY_DB["Deploy PostgreSQL<br/>StatefulSet"]
    RESTORE_CONFIG --> DEPLOY_DB

    DEPLOY_DB --> RESTORE_DATA["Restore Database<br/>from Backup"]
    RESTORE_DATA --> DEPLOY_APP["Deploy Application<br/>Services"]
    DEPLOY_APP --> VERIFY_HEALTH["Verify Health<br/>& Readiness"]
    VERIFY_HEALTH --> RESTORE_TRAFFIC["Restore DNS<br/>& Traffic"]
```

### 4.4 S4: Data Corruption

```bash
# Step 1: Identify corruption scope
# Check PostgreSQL logs for errors
kubectl logs -n data statefulset/postgresql --tail 100

# Step 2: Determine restore point
# Query audit events to find last known good state
# Check _sqlx_migrations for migration integrity

# Step 3: Point-in-time recovery
kubectl scale deployment crm-core -n crm --replicas=0

# Create new database from backup
createdb -h localhost -U postgres crm_restored
pg_restore -h localhost -U postgres -d crm_restored /backups/crm_latest.dump

# Verify restored data
psql -h localhost -U postgres -d crm_restored -c "SELECT count(*) FROM contacts;"

# Swap databases
psql -h localhost -U postgres -c "ALTER DATABASE crm RENAME TO crm_corrupted;"
psql -h localhost -U postgres -c "ALTER DATABASE crm_restored RENAME TO crm;"

# Resume traffic
kubectl scale deployment crm-core -n crm --replicas=2
```

## 5. Testing Schedule

| Test Type | Frequency | Description |
|-----------|-----------|------------|
| Backup Verification | Daily | Automated restore test to staging |
| Pod Failure Drill | Monthly | Kill a pod, verify auto-recovery |
| Database Failover | Quarterly | Simulate DB failure, time recovery |
| Full DR Drill | Semi-annual | Full cluster recovery from backups |
| Tabletop Exercise | Annual | Walk through all scenarios with team |

## 6. Communication Plan

### Incident Communication Flow

```mermaid
sequenceDiagram
    participant Monitor as Monitoring
    participant OnCall as On-Call Engineer
    participant Lead as Team Lead
    participant Status as Status Page
    participant Users as Users

    Monitor->>OnCall: Alert triggered
    OnCall->>OnCall: Assess severity
    OnCall->>Lead: Escalate if P1/P2
    OnCall->>Status: Update status page
    Status->>Users: Notification sent
    OnCall->>OnCall: Execute runbook
    OnCall->>Status: Update: investigating
    OnCall->>OnCall: Resolve incident
    OnCall->>Status: Update: resolved
    Status->>Users: Resolution notification
    OnCall->>Lead: Post-mortem scheduled
```

### Contact List

| Role | Contact Method | Response Time |
|------|---------------|--------------|
| On-Call Engineer | PagerDuty | 15 min (P1) |
| Team Lead | Phone + Slack | 30 min |
| Database Admin | Phone | 30 min |
| Infrastructure Lead | Phone | 30 min |

## 7. Data Classification for Recovery Priority

| Data | Priority | RPO | Recovery Order |
|------|----------|-----|---------------|
| Contact/Company records | Critical | 5 min | 1st |
| Deal/Pipeline data | Critical | 5 min | 1st |
| Ticket/Support data | High | 15 min | 2nd |
| Activities/Notes | Medium | 30 min | 3rd |
| Form submissions | Medium | 30 min | 3rd |
| Chat transcripts | Low | 1 hour | 4th |
| Analytics/Reports | Low | Regenerate | 5th |
| Audit events | Critical | 0 (immutable) | 1st (Pulsar) |
