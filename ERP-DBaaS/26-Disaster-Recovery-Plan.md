# ERP-DBaaS Disaster Recovery Plan

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Disaster Recovery Plan       |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Confidential                |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [RPO/RTO Targets per Tier](#1-rporto-targets-per-tier)
2. [Backup Verification Procedures](#2-backup-verification-procedures)
3. [Cross-Region Replication Setup](#3-cross-region-replication-setup)
4. [Failover Procedures per Engine](#4-failover-procedures-per-engine)
5. [Data Recovery from RustFS](#5-data-recovery-from-rustfs)
6. [Communication Plan During Outages](#6-communication-plan-during-outages)

---

## 1. RPO/RTO Targets per Tier

### 1.1 Recovery Objectives

| Tier   | RPO                            | RTO                           | Backup Frequency     |
|--------|--------------------------------|-------------------------------|----------------------|
| Tier A | 1 hour                         | 15 minutes                    | Hourly incremental + daily full |
| Tier B | 6 hours                        | 1 hour                        | Every 6h incremental + daily full |
| Tier C | 24 hours                       | 4 hours                       | Daily full           |

### 1.2 Disaster Severity Levels

| Level  | Description                                          | Examples                                    | Response Time |
|--------|------------------------------------------------------|---------------------------------------------|---------------|
| SEV-1  | Complete platform outage, all tenants affected       | Control plane failure, K8s cluster down      | Immediate     |
| SEV-2  | Single region outage, multiple tenants affected      | AZ failure, storage backend unavailable      | 5 minutes     |
| SEV-3  | Single engine outage, subset of tenants affected     | Operator crash loop, engine-specific bug     | 15 minutes    |
| SEV-4  | Single instance failure                               | Pod eviction, PVC corruption                 | 30 minutes    |

### 1.3 Recovery Priority Matrix

| Priority | Criteria                                              | Order of Recovery               |
|----------|-------------------------------------------------------|---------------------------------|
| P0       | Tier A instances with financial/transactional data    | Restored first                  |
| P1       | Tier A remaining + Tier B instances with HA           | Restored second                 |
| P2       | Tier B remaining instances                             | Restored third                  |
| P3       | Tier C instances                                       | Restored last                   |

---

## 2. Backup Verification Procedures

### 2.1 Automated Verification Schedule

| Verification Type       | Frequency | Scope                        | Automation Level |
|-------------------------|-----------|------------------------------|------------------|
| Checksum validation     | Every backup | All backups               | Fully automated  |
| Restore test (sample)   | Daily     | 5% random sample of backups  | Fully automated  |
| Full restore drill      | Weekly    | 1 instance per engine type   | Semi-automated   |
| Cross-region restore    | Monthly   | 1 Tier A instance per region | Semi-automated   |
| Complete DR drill       | Quarterly | All Tier A instances         | Manual + automated|

### 2.2 Checksum Validation Process

```
Backup Upload ──> Calculate SHA-256 ──> Store in backup_catalog
                                              │
                                     ┌────────┘
                                     ▼
                  Nightly Verification Job
                         │
                         ▼
                  Download from RustFS ──> Recalculate SHA-256
                         │
                         ▼
                  Compare with stored checksum
                         │
                    ┌────┴────┐
                    ▼         ▼
                  MATCH     MISMATCH
                    │         │
                    ▼         ▼
                  Pass      Alert + Re-backup
```

### 2.3 Restore Test Procedure

```bash
#!/bin/bash
# Automated restore test script (runs daily via CronJob)

# 1. Select random backup from catalog
BACKUP_ID=$(psql -t -c "
  SELECT id FROM backup_catalog
  WHERE status = 'completed'
  AND verified_at < NOW() - INTERVAL '7 days'
  ORDER BY RANDOM() LIMIT 1;
")

# 2. Provision ephemeral test instance
TEST_INSTANCE="restore-test-$(date +%s)"
dbaas-cli instance create --name ${TEST_INSTANCE} \
  --engine $(get_backup_engine ${BACKUP_ID}) \
  --size S --namespace dbaas-dr-test

# 3. Restore backup to test instance
dbaas-cli backup restore --backup-id ${BACKUP_ID} \
  --target ${TEST_INSTANCE} --wait

# 4. Validate data integrity
dbaas-cli instance verify --name ${TEST_INSTANCE} \
  --checks schema,row-count,checksum

# 5. Record verification result
psql -c "UPDATE backup_catalog SET
  verified_at = NOW(),
  verification_status = 'passed'
  WHERE id = '${BACKUP_ID}';"

# 6. Cleanup test instance
dbaas-cli instance delete --name ${TEST_INSTANCE} --force
```

---

## 3. Cross-Region Replication Setup

### 3.1 Replication Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Primary Region (us-east-1)                   │
│                                                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ YugabyteDB│  │ ClickHouse│  │ Tembo PG │  │ DragonflyDB│      │
│  │ Primary   │  │ Primary   │  │ Primary  │  │ Primary    │      │
│  └─────┬────┘  └─────┬────┘  └────┬─────┘  └─────┬─────┘      │
│        │             │            │               │              │
└────────┼─────────────┼────────────┼───────────────┼──────────────┘
         │ sync/async  │ async      │ streaming     │ async
         │ replication │ replication│ replication    │ replication
┌────────┼─────────────┼────────────┼───────────────┼──────────────┐
│        │             │            │               │              │
│  ┌─────▼────┐  ┌─────▼────┐  ┌───▼──────┐  ┌────▼──────┐      │
│  │ YugabyteDB│  │ ClickHouse│  │ Tembo PG │  │ DragonflyDB│      │
│  │ Replica   │  │ Replica   │  │ Standby  │  │ Replica    │      │
│  └──────────┘  └──────────┘  └──────────┘  └───────────┘       │
│                                                                   │
│                     DR Region (eu-west-1)                        │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Replication Configuration per Engine

| Engine      | Replication Method         | Mode Options    | Max Lag Target | Failover Method           |
|-------------|----------------------------|-----------------|----------------|---------------------------|
| YugabyteDB  | xCluster replication       | Sync / Async    | < 1s (sync)    | Automatic (Raft)          |
| DragonflyDB | Built-in replication       | Async           | < 5s           | Sentinel-based promotion  |
| ClickHouse  | ReplicatedMergeTree + ZK   | Async           | < 30s          | Manual table promotion    |
| Tembo (PG)  | Streaming replication      | Sync / Async    | < 1s (sync)    | pg_promote()              |
| SurrealDB   | TiKV Raft replication      | Sync            | < 1s           | Automatic (Raft)          |
| QuestDB     | Backup-based (no native)   | N/A             | = RPO          | Restore from backup       |
| Apache Doris| BDBJE replication (FE)     | Sync            | < 5s           | FE follower promotion     |
| InfluxDB    | Anti-entropy replication   | Async           | < 30s          | Manual re-election        |

### 3.3 Network Requirements

| Parameter                    | Specification                    |
|------------------------------|----------------------------------|
| Cross-region bandwidth       | Minimum 1 Gbps dedicated         |
| Cross-region latency         | Maximum 100ms RTT                |
| VPN tunnel                   | WireGuard mesh between regions   |
| MTU                          | 1400 bytes (accounting for VPN)  |
| Encryption                   | WireGuard ChaCha20-Poly1305      |

---

## 4. Failover Procedures per Engine

### 4.1 YugabyteDB Failover

**Automatic Failover (HA mode):**
YugabyteDB uses Raft consensus for automatic leader election. No manual intervention is required for single-node failures within a cluster.

**Regional Failover:**

```
1. Detect primary region failure (health checks fail for 3 consecutive intervals)
2. Verify xCluster replication lag is within acceptable bounds
3. Pause xCluster replication to prevent split-brain
4. Promote DR region cluster:
   $ yb-admin -master_addresses <dr-masters> set_universe_replication_enabled 0
5. Update DNS to point to DR region service endpoint
6. Verify application connectivity
7. Monitor for data consistency
```

### 4.2 DragonflyDB Failover

```
1. Sentinel detects primary is unreachable (quorum: 2 of 3 sentinels agree)
2. Sentinel selects replica with least replication lag
3. Sentinel promotes selected replica:
   $ redis-cli -h <sentinel-host> SENTINEL FAILOVER <master-name>
4. Sentinel reconfigures remaining replicas to follow new primary
5. K8s Service endpoint is updated automatically
6. Verify client reconnection (clients using Sentinel-aware drivers reconnect automatically)
```

### 4.3 ClickHouse Failover

```
1. Detect shard replica failure via system.replicas health check
2. ZooKeeper maintains quorum awareness
3. Remaining replicas continue serving reads
4. For writes, the distributed table routes to healthy replicas
5. To promote a specific replica as preferred:
   $ clickhouse-client --query "SYSTEM RESTART REPLICA <table>"
6. Once the failed node recovers, ZooKeeper triggers data sync
```

### 4.4 Tembo (PostgreSQL) Failover

```
1. Detect primary failure via pg_isready or streaming replication monitor
2. Select standby with least replication lag:
   $ psql -c "SELECT client_addr, sent_lsn, replay_lsn,
              sent_lsn - replay_lsn AS lag FROM pg_stat_replication;"
3. Promote the selected standby:
   $ psql -c "SELECT pg_promote();"
   -- or --
   $ pg_ctl promote -D /var/lib/postgresql/data
4. Update PgBouncer configuration to point to the new primary
5. Reconfigure remaining standbys to follow the new primary
6. Update K8s Service endpoints
```

### 4.5 Generic Backup-Based Recovery

For engines without native replication (QuestDB standalone), or when replication is unavailable:

```
1. Identify the most recent valid backup from backup_catalog
2. Provision a new instance with matching engine, version, and configuration
3. Download and decrypt the backup from RustFS
4. Restore the backup to the new instance
5. Apply any available WAL/incremental backups to minimize data loss
6. Update DNS and service records to point to the new instance
7. Verify data integrity and application connectivity
```

---

## 5. Data Recovery from RustFS

### 5.1 Recovery Prerequisites

| Requirement               | Description                                         |
|---------------------------|-----------------------------------------------------|
| RustFS accessibility      | Verify endpoint reachability and credentials         |
| Encryption keys           | Tenant encryption keys must be available             |
| Target K8s cluster        | Sufficient capacity for the restored instance        |
| Backup catalog access     | Required to locate the correct backup artifacts      |

### 5.2 Step-by-Step Recovery

```bash
#!/bin/bash
# Manual recovery from RustFS

TENANT_ID="$1"
INSTANCE_ID="$2"
BACKUP_ID="$3"

# Step 1: Locate backup in catalog
BACKUP_INFO=$(psql -t -A -c "
  SELECT storage_path, encryption_key_id, engine, checksum_sha256
  FROM backup_catalog
  WHERE id = '${BACKUP_ID}' AND tenant_id = '${TENANT_ID}';
")

STORAGE_PATH=$(echo $BACKUP_INFO | cut -d'|' -f1)
KEY_ID=$(echo $BACKUP_INFO | cut -d'|' -f2)
ENGINE=$(echo $BACKUP_INFO | cut -d'|' -f3)
EXPECTED_CHECKSUM=$(echo $BACKUP_INFO | cut -d'|' -f4)

# Step 2: Download from RustFS
mc cp rustfs/dbaas-backups/${STORAGE_PATH} /tmp/restore/backup.zst.enc

# Step 3: Verify checksum
ACTUAL_CHECKSUM=$(sha256sum /tmp/restore/backup.zst.enc | cut -d' ' -f1)
if [ "$ACTUAL_CHECKSUM" != "$EXPECTED_CHECKSUM" ]; then
  echo "ERROR: Checksum mismatch! Backup may be corrupted."
  exit 1
fi

# Step 4: Retrieve decryption key
DEK=$(vault kv get -field=dek secret/dbaas/keys/${TENANT_ID}/${KEY_ID})

# Step 5: Decrypt
openssl enc -d -aes-256-gcm -K ${DEK} -in /tmp/restore/backup.zst.enc \
  -out /tmp/restore/backup.zst

# Step 6: Decompress
zstd -d /tmp/restore/backup.zst -o /tmp/restore/backup.dump

# Step 7: Restore to target instance (engine-specific)
case $ENGINE in
  yugabytedb) ysqlsh -h $TARGET_HOST -f /tmp/restore/backup.dump ;;
  clickhouse) clickhouse-client --host $TARGET_HOST --query "$(cat /tmp/restore/backup.dump)" ;;
  tembo)      pg_restore -h $TARGET_HOST -d $DB_NAME /tmp/restore/backup.dump ;;
  dragonflydb) redis-cli -h $TARGET_HOST --rdb /tmp/restore/backup.dump ;;
esac

# Step 8: Cleanup
rm -rf /tmp/restore/
```

### 5.3 RustFS Failure Contingency

If RustFS itself is unavailable:

| Scenario                    | Mitigation                                            |
|-----------------------------|-------------------------------------------------------|
| Single RustFS node failure  | RustFS erasure coding ensures data durability          |
| RustFS cluster failure      | Restore from cross-region RustFS replica               |
| Total RustFS loss           | Reconstruct from engine-level snapshots (if available) |
| Network partition to RustFS | Wait for network restoration; backups continue locally |

---

## 6. Communication Plan During Outages

### 6.1 Communication Channels

| Channel                    | Audience                    | Update Frequency    |
|----------------------------|-----------------------------|---------------------|
| Status page (status.erp.io)| All tenants, stakeholders   | Every 15 minutes    |
| Slack #dbaas-incidents     | Engineering team            | Real-time           |
| PagerDuty                  | On-call SRE team            | Immediate on SEV-1/2|
| Email notification         | Affected tenant admins      | At incident start, every hour, at resolution |
| Apache Pulsar events       | Automated systems           | Real-time           |

### 6.2 Communication Templates

**Initial Notification (within 5 minutes of detection):**

```
Subject: [DBaaS Incident] {SEVERITY} - {Brief Description}

We are investigating an issue affecting {scope}.

Impact: {description of what is affected}
Start Time: {UTC timestamp}
Current Status: Investigating

We will provide updates every {15 minutes / 1 hour} until resolved.
```

**Update Notification:**

```
Subject: [DBaaS Incident Update] {SEVERITY} - {Brief Description}

Status: {Investigating / Identified / Monitoring / Resolved}
Impact: {current impact assessment}
Root Cause: {if identified}
Mitigation: {actions taken or in progress}
ETA: {estimated time to resolution, if known}

Next update in {timeframe}.
```

**Resolution Notification:**

```
Subject: [DBaaS Incident Resolved] {Brief Description}

The incident affecting {scope} has been resolved.

Duration: {total duration}
Root Cause: {brief root cause}
Resolution: {what was done}
Data Impact: {any data loss or RPO deviation}

A full post-incident review will be published within 48 hours.
```

### 6.3 Escalation Matrix

| Time Elapsed | Action                                                     |
|--------------|-------------------------------------------------------------|
| 0 min        | Primary on-call SRE paged via PagerDuty                    |
| 5 min        | Status page updated, Slack incident channel created         |
| 15 min       | Secondary on-call SRE engaged if primary unresponsive       |
| 30 min       | Engineering manager notified                                |
| 1 hour       | Director of Engineering notified, war room established      |
| 2 hours      | VP of Engineering notified, executive stakeholder briefing  |
| 4 hours      | CTO notified, customer success team briefed for Tier A tenants |

### 6.4 Post-Incident Review

All SEV-1 and SEV-2 incidents require a post-incident review (PIR) completed within 48 hours of resolution. The PIR document includes:

1. **Timeline**: Minute-by-minute account of the incident
2. **Root Cause Analysis**: 5 Whys analysis to identify the underlying cause
3. **Impact Assessment**: Number of tenants affected, data loss (if any), SLA violations
4. **Action Items**: Concrete steps to prevent recurrence, assigned owners, and due dates
5. **Lessons Learned**: What worked well and what can be improved in the DR process

---

## Appendix: DR Contact List

| Role                    | Primary Contact       | Backup Contact         | Phone           |
|-------------------------|-----------------------|------------------------|-----------------|
| SRE On-Call             | Rotates weekly        | See PagerDuty schedule | PagerDuty       |
| Engineering Manager     | {Name}                | {Name}                 | {Number}        |
| Director of Engineering | {Name}                | {Name}                 | {Number}        |
| RustFS Admin            | {Name}                | {Name}                 | {Number}        |
| K8s Cluster Admin       | {Name}                | {Name}                 | {Number}        |
| Network Operations      | {Name}                | {Name}                 | {Number}        |
