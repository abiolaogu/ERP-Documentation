# ERP-Finance Disaster Recovery Plan

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | Disaster Recovery Plan |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

## Recovery Objectives

| Metric | Target | Justification |
|--------|--------|---------------|
| RPO (Recovery Point Objective) | < 1 minute | Synchronous replication for financial data |
| RTO (Recovery Time Objective) | < 15 minutes | Automated failover with pre-provisioned standby |
| MTTR (Mean Time to Recovery) | < 30 minutes | Including diagnosis and verification |

## DR Architecture

```mermaid
flowchart TB
    subgraph Primary["Primary Region"]
        subgraph PrimK8s["Kubernetes Cluster"]
            PrimSvc["Finance Services<br/>(15 services)"]
        end
        subgraph PrimData["Data Layer"]
            PrimPG["PostgreSQL Primary<br/>+ 2 Sync Replicas"]
            PrimRedis["Redis Sentinel"]
            PrimNATS["NATS JetStream"]
        end
        PrimSvc --> PrimPG
        PrimSvc --> PrimRedis
        PrimSvc --> PrimNATS
    end

    subgraph DR["DR Region"]
        subgraph DRK8s["Kubernetes Cluster (Standby)"]
            DRSvc["Finance Services<br/>(2 replicas each, warm)"]
        end
        subgraph DRData["Data Layer"]
            DRPG["PostgreSQL Async Replica"]
            DRRedis["Redis Replica"]
            DRNATS["NATS Mirror"]
        end
        DRSvc --> DRPG
        DRSvc --> DRRedis
        DRSvc --> DRNATS
    end

    PrimPG -->|"Async WAL Streaming<br/>(< 1 min lag)"| DRPG
    PrimRedis -->|"Replication"| DRRedis
    PrimNATS -->|"Stream Mirror"| DRNATS
    PrimPG -->|"WAL Archive<br/>(S3 Cross-Region)"| S3["S3 Archive<br/>(Point-in-Time Recovery)"]
```

## Failure Scenarios

### Scenario 1: Single Service Failure

**Impact**: One financial service unavailable.
**Detection**: Health check fails, Kubernetes restarts pod.
**Recovery**: Automatic (Kubernetes self-healing). RTO: < 30 seconds.

### Scenario 2: Database Primary Failure

**Impact**: All write operations blocked.
**Detection**: Patroni/CloudNativePG detects primary failure.
**Recovery**:
1. Patroni promotes synchronous replica to primary (automatic)
2. Services reconnect to new primary
3. RTO: < 60 seconds

### Scenario 3: Full Region Failure

**Impact**: Complete loss of primary region.
**Detection**: Multi-region health checks fail.
**Recovery**:

```mermaid
flowchart TD
    Detect["Detect region failure<br/>(multi-region health checks)"] --> Decide["Incident commander<br/>authorizes failover"]
    Decide --> DNS["Update DNS to DR region"]
    DNS --> Promote["Promote DR PostgreSQL<br/>to primary"]
    Promote --> Verify["Verify data integrity"]
    Verify --> Scale["Scale DR services<br/>to production capacity"]
    Scale --> Smoke["Run smoke tests"]
    Smoke --> Announce["Announce recovery"]
    Announce --> Monitor["Enhanced monitoring<br/>for 24 hours"]
```

RTO: < 15 minutes.

### Scenario 4: Data Corruption

**Impact**: Financial data integrity compromised.
**Detection**: Reconciliation checks, anomaly detection.
**Recovery**:
1. Identify scope of corruption
2. Point-in-time recovery from WAL archive to last known good state
3. Replay events from NATS JetStream for gap
4. Manual verification of financial balances
5. RTO: 1-4 hours depending on scope

## Backup Strategy

### Backup Schedule

| Data | Method | Frequency | Retention | Storage |
|------|--------|-----------|-----------|---------|
| PostgreSQL (full) | pg_basebackup | Every 6 hours | 30 days | S3 Cross-Region |
| PostgreSQL (WAL) | Continuous archiving | Continuous | 30 days | S3 Cross-Region |
| Redis (RDB) | Snapshot | Hourly | 7 days | S3 |
| MinIO (documents) | Cross-region replication | Continuous | 1 year | S3 Cross-Region |
| NATS streams | JetStream replication | Continuous | 90 days | Local + Remote |
| Configuration | Git repository | Every change | Forever | Git remote |

### Backup Verification

- **Weekly**: Automated restore test to isolated environment
- **Monthly**: Full DR drill restoring from backups
- **Quarterly**: Complete failover exercise to DR region

## Financial Data Protection

### Immutable Ledger Protection

The General Ledger's immutable posting design provides inherent protection:
- Posted journal entries cannot be modified or deleted
- Any correction requires a reversing entry
- Full audit trail maintained
- This means even partial data recovery preserves financial integrity

### Reconciliation After Recovery

After any recovery event:
1. Run trial balance -- verify debits equal credits
2. Reconcile GL to sub-ledgers (AP, AR, Asset)
3. Compare bank balances to GL cash accounts
4. Verify subscription billing state matches invoice records
5. Check payment transaction status with providers

## Communication Plan

| Audience | Channel | Timing | Responsible |
|----------|---------|--------|-------------|
| Engineering | Slack #finance-incident | Immediate | On-call SRE |
| Finance team | Email + Slack | Within 15 minutes | Product lead |
| Customers (if impacted) | Status page | Within 30 minutes | Communications |
| Executive team | Email + call | Within 1 hour (SEV-1) | VP Engineering |
| Auditors | Formal incident report | Within 48 hours | Compliance team |

## DR Testing Schedule

| Test Type | Frequency | Duration | Participants |
|-----------|-----------|----------|-------------|
| Tabletop exercise | Quarterly | 2 hours | Engineering + Finance |
| Automated failover test | Monthly | 30 minutes | SRE team |
| Full DR failover | Semi-annually | 4 hours | All teams |
| Backup restore verification | Weekly | Automated | CI/CD pipeline |
