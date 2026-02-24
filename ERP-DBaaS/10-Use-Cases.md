# ERP-DBaaS Use Cases

## Document Control

| Field             | Value                          |
|-------------------|--------------------------------|
| Document Title    | ERP-DBaaS Use Cases            |
| Version           | 1.0.0                         |
| Date              | 2026-02-24                     |
| Classification    | Internal - Engineering         |
| Author            | Platform Engineering Team      |

---

## Table of Contents

1. [UC-001: Provisioning a New YugabyteDB Cluster](#uc-001-provisioning-a-new-yugabytedb-cluster)
2. [UC-002: Scaling a DragonflyDB Cache](#uc-002-scaling-a-dragonflydb-cache)
3. [UC-003: Setting Up Automated Backups with Encryption](#uc-003-setting-up-automated-backups-with-encryption)
4. [UC-004: Credential Rotation for Compliance](#uc-004-credential-rotation-for-compliance)
5. [UC-005: Multi-Region Deployment with HA](#uc-005-multi-region-deployment-with-ha)
6. [UC-006: Plugin Management for Custom Extensions](#uc-006-plugin-management-for-custom-extensions)
7. [UC-007: Quota Enforcement for Tenant Tiers](#uc-007-quota-enforcement-for-tenant-tiers)
8. [UC-008: Disaster Recovery Scenarios](#uc-008-disaster-recovery-scenarios)

---

## 1. Overview

This document catalogs the primary use cases for the ERP-DBaaS platform. Each use case describes a real-world scenario that the platform must support, including the actors involved, preconditions, main flow, alternative flows, and postconditions. These use cases serve as the foundation for feature development, acceptance testing, and stakeholder communication.

---

## UC-001: Provisioning a New YugabyteDB Cluster

### Actors
- **Primary**: ERP Module Developer (e.g., Finance module team)
- **Secondary**: DBaaS Platform (automated), K8s Operator

### Preconditions
- Tenant has an active subscription with available quota
- Target namespace exists in the Kubernetes cluster
- YugabyteDB engine is enabled in the engine catalog

### Main Flow

1. Developer navigates to the DBaaS dashboard and selects "Create Instance."
2. Developer selects YugabyteDB from the engine catalog.
3. Developer chooses a size preset (S/M/L/XL) and topology (standalone or HA with 3 replicas).
4. Developer configures database name, replication factor, and tablespace settings.
5. System validates the request against tenant quota limits (CPU, memory, storage, instance count).
6. System generates a `YugabyteDBInstance` CRD and submits it to the Kubernetes API server.
7. The YugabyteDB operator detects the new CRD and begins provisioning.
8. Operator creates StatefulSets for TServer and Master pods, PVCs for storage, and Services for connectivity.
9. Operator runs health checks until all pods are ready and the cluster is initialized.
10. System generates credentials, stores them in a Kubernetes Secret, and registers the instance in the dbaas_registry.
11. Developer receives a connection string and credentials via the dashboard.

### Alternative Flows
- **Quota Exceeded**: System rejects the request at step 5 with a quota violation error and suggests upgrading the tenant tier.
- **Provisioning Timeout**: If provisioning exceeds 10 minutes, the operator marks the instance as `failed` and triggers an alert via Apache Pulsar.
- **Engine Unavailable**: If YugabyteDB is under maintenance, the system displays an availability notice with an estimated restoration time.

### Postconditions
- A fully operational YugabyteDB cluster is running in the tenant namespace.
- Connection credentials are stored in a Kubernetes Secret.
- The instance appears in the tenant dashboard with status `running`.
- Metering begins recording resource consumption for billing.

---

## UC-002: Scaling a DragonflyDB Cache

### Actors
- **Primary**: Application Developer
- **Secondary**: DBaaS Platform, DragonflyDB Operator

### Preconditions
- An existing DragonflyDB instance is running with a 1GB memory allocation.
- The tenant has sufficient quota headroom for the target size.

### Main Flow

1. Developer opens the instance detail page for their DragonflyDB cache.
2. Developer selects "Scale" and chooses the target memory allocation (e.g., 8GB).
3. System validates the scaling request against tenant quota and cluster capacity.
4. System performs a pre-scale health check to verify the instance is in a healthy state.
5. System updates the `DragonflyDBInstance` CRD with the new resource specification.
6. The DragonflyDB operator detects the CRD change and initiates the scaling operation.
7. For vertical scaling, the operator updates the pod resource limits and triggers a rolling restart.
8. The operator monitors memory usage stabilization and confirms data integrity.
9. System updates metering records to reflect the new resource allocation.
10. Developer receives a notification that scaling is complete.

### Alternative Flows
- **Insufficient Cluster Capacity**: System suggests scheduling the scale operation during off-peak hours or requests cluster administrator intervention.
- **Data Migration Required**: For horizontal scaling (adding replicas), the operator redistributes keys across shards before marking the operation as complete.
- **Rollback on Failure**: If the scaled instance fails health checks within 5 minutes, the operator reverts to the previous resource allocation.

### Postconditions
- The DragonflyDB instance is running with 8GB memory allocation.
- All cached data is intact (no data loss during scaling).
- Metering reflects the updated resource consumption.

---

## UC-003: Setting Up Automated Backups with Encryption

### Actors
- **Primary**: Database Administrator or Application Developer
- **Secondary**: Backup Controller, RustFS Storage Backend

### Preconditions
- A database instance exists and is in `running` status.
- RustFS storage backend is configured and accessible.
- Encryption keys are available in the secrets management system.

### Main Flow

1. Developer navigates to the instance detail page and selects the "Backups" tab.
2. Developer enables automated backups and configures the schedule (e.g., daily at 02:00 UTC).
3. Developer selects encryption settings: AES-256-GCM with a tenant-specific key.
4. Developer sets the retention policy (e.g., 30 days for daily, 90 days for weekly).
5. System validates the backup configuration and estimates storage requirements.
6. System creates a `BackupSchedule` CRD linked to the database instance.
7. At the scheduled time, the backup controller triggers a consistent snapshot of the database.
8. The snapshot is compressed using zstd compression.
9. The compressed snapshot is encrypted using the specified encryption algorithm and key.
10. The encrypted backup is uploaded to RustFS with metadata (timestamp, size, checksum, encryption key ID).
11. System records the backup in the backup catalog with verification status.
12. Old backups exceeding the retention policy are automatically purged.

### Alternative Flows
- **Backup Failure**: If the snapshot fails, the system retries up to 3 times with exponential backoff. On final failure, an alert is sent via the notification pipeline.
- **Storage Quota Warning**: If backup storage approaches 80% of the allocated quota, the system notifies the tenant and suggests adjusting retention policies.
- **Point-in-Time Recovery**: For engines that support WAL archiving (YugabyteDB, PostgreSQL via Tembo), continuous WAL backups are streamed to RustFS alongside periodic full snapshots.

### Postconditions
- Automated backup schedule is active and producing encrypted backups.
- Backups are stored in RustFS with integrity checksums.
- Retention policy is enforced automatically.
- Backup history is visible in the dashboard with restore options.

---

## UC-004: Credential Rotation for Compliance

### Actors
- **Primary**: Security Officer or Automated Compliance System
- **Secondary**: Credential Manager, K8s Secrets, Application Pods

### Preconditions
- Database instance is running with active credentials.
- Credential rotation policy is defined (e.g., every 90 days).
- Applications consuming the database use Kubernetes Secret references (not hardcoded credentials).

### Main Flow

1. The rotation trigger fires (scheduled policy or manual request from security officer).
2. Credential Manager generates new credentials conforming to the password policy (minimum 32 characters, mixed case, numbers, special characters).
3. System creates the new credentials on the database engine (e.g., `ALTER ROLE` for YugabyteDB).
4. System verifies connectivity using the new credentials.
5. System updates the Kubernetes Secret with the new credentials.
6. Applications using Secret volume mounts or environment variable references automatically pick up the new credentials on the next pod restart or Secret refresh cycle.
7. System maintains the old credentials in an active state for a grace period (default: 24 hours) to prevent disruption.
8. After the grace period, old credentials are revoked on the database engine.
9. System records the rotation event in the audit log with timestamps, key IDs, and actor information.

### Alternative Flows
- **Rotation Failure**: If new credentials fail verification at step 4, the system aborts the rotation and retains the existing credentials. An alert is raised for manual investigation.
- **Application Not Updated**: If applications are still using old credentials after the grace period, the system extends the grace period and notifies the application team before revoking.
- **Emergency Rotation**: In the event of a credential compromise, the security officer can trigger an immediate rotation with a zero-second grace period, forcing all applications to restart.

### Postconditions
- New credentials are active on the database and in Kubernetes Secrets.
- Old credentials are revoked after the grace period.
- Audit log contains a complete rotation record for compliance reporting.
- No application downtime occurred during the rotation window.

---

## UC-005: Multi-Region Deployment with HA

### Actors
- **Primary**: Platform Architect or SRE
- **Secondary**: DBaaS Control Plane, Regional K8s Clusters

### Preconditions
- Multiple Kubernetes clusters are available across regions (e.g., us-east-1, eu-west-1, ap-southeast-1).
- Cross-region networking (VPN or service mesh) is established.
- The database engine supports multi-region replication (YugabyteDB, ClickHouse).

### Main Flow

1. Architect selects "Multi-Region HA" during instance creation or upgrades an existing instance.
2. Architect configures the region topology: primary region, replica regions, and replication mode (synchronous or asynchronous).
3. System validates cross-region connectivity and latency requirements.
4. System provisions the primary instance in the designated primary region.
5. System provisions read replicas in each secondary region using the engine-specific replication mechanism.
6. System configures DNS-based routing with health checks for automatic failover.
7. System establishes monitoring for replication lag, cross-region latency, and node health.
8. System performs a failover drill to validate the HA configuration.

### Alternative Flows
- **Region Unavailable**: If a target region is unreachable during setup, the system provisions available regions and queues the unavailable region for retry.
- **Replication Lag Exceeds Threshold**: If asynchronous replication lag exceeds the configured threshold (e.g., 30 seconds), the system triggers an alert and optionally throttles writes to allow replicas to catch up.
- **Automatic Failover**: When the primary region fails health checks for 3 consecutive intervals, DNS routing shifts to the highest-priority replica, which is promoted to primary.

### Postconditions
- Database is operational across multiple regions with configured replication.
- Automatic failover is enabled and tested.
- Monitoring dashboards show cross-region replication status.
- RTO and RPO targets are documented and validated.

---

## UC-006: Plugin Management for Custom Extensions

### Actors
- **Primary**: Database Administrator or Application Developer
- **Secondary**: Plugin Registry, Engine Operator

### Preconditions
- The target database engine supports plugins or extensions (e.g., YugabyteDB/PostgreSQL extensions, ClickHouse plugins).
- The plugin is available in the DBaaS plugin registry and has passed security scanning.

### Main Flow

1. Developer browses the plugin registry for available extensions compatible with their engine and version.
2. Developer selects a plugin (e.g., `pg_trgm` for trigram-based text search) and reviews its compatibility matrix and resource impact.
3. Developer initiates the plugin installation for a specific database instance.
4. System validates plugin compatibility with the current engine version and existing plugins.
5. System checks that the plugin does not violate tenant security policies or resource quotas.
6. The engine operator downloads the plugin artifact from the registry.
7. The operator installs the plugin on the database engine (e.g., `CREATE EXTENSION pg_trgm`).
8. The operator runs validation tests to confirm the plugin is functional.
9. System records the installed plugin in the instance metadata.

### Alternative Flows
- **Incompatible Version**: System rejects the installation and displays the compatible version range for the plugin.
- **Security Policy Violation**: If the plugin requires elevated privileges that the tenant policy disallows, the request is escalated to a platform administrator for approval.
- **Plugin Rollback**: If validation tests fail after installation, the operator uninstalls the plugin and restores the previous state.

### Postconditions
- The plugin is installed and operational on the target instance.
- Instance metadata reflects the installed plugin and its version.
- Plugin appears in the instance detail view with management options (update, uninstall).

---

## UC-007: Quota Enforcement for Tenant Tiers

### Actors
- **Primary**: Tenant Administrator
- **Secondary**: Quota Enforcement Engine, Billing System

### Preconditions
- Tenant is assigned to a tier (A, B, or C) with defined resource limits.
- Quota limits are configured in the tenant profile (max instances, max CPU, max memory, max storage).

### Main Flow

1. Tenant administrator reviews current quota usage on the dashboard (progress bars showing consumption vs. limits).
2. When a resource request is made (new instance, scaling, backup storage), the quota enforcement engine evaluates the request.
3. The engine checks the request against the tenant tier limits:
   - **Tier A (Enterprise)**: 50 instances, 200 vCPU, 512GB memory, 10TB storage.
   - **Tier B (Professional)**: 20 instances, 80 vCPU, 256GB memory, 5TB storage.
   - **Tier C (Starter)**: 5 instances, 16 vCPU, 32GB memory, 500GB storage.
4. If within limits, the request proceeds.
5. If the request would exceed limits, the system returns a detailed quota violation indicating which resource is exhausted.
6. The tenant administrator can request a tier upgrade through the billing portal.
7. Upon tier upgrade approval, quota limits are updated in real time and the previously blocked request can be retried.

### Alternative Flows
- **Soft Limit Warning**: When usage reaches 80% of any quota dimension, the system sends proactive notifications to the tenant administrator.
- **Grace Period Overage**: For Tier A tenants, a 10% overage grace is permitted for 72 hours while an upgrade is processed.
- **Quota Reconciliation**: A nightly job reconciles actual resource usage with the quota ledger to correct any drift caused by failed provisioning or scaling operations.

### Postconditions
- Resource requests are approved or denied based on tenant tier limits.
- Quota usage is accurately tracked and visible on the dashboard.
- Tier upgrades are reflected immediately in quota enforcement.

---

## UC-008: Disaster Recovery Scenarios

### Actors
- **Primary**: Site Reliability Engineer (SRE)
- **Secondary**: DR Controller, Backup System, Regional Clusters

### Preconditions
- Backup schedule is active with recent successful backups.
- DR plan is documented and has been tested within the last 90 days.
- Secondary region infrastructure is provisioned and on standby.

### Main Flow: Single Instance Recovery

1. SRE detects an instance failure (data corruption, accidental deletion, or infrastructure failure).
2. SRE initiates a recovery operation from the DBaaS dashboard, selecting the target backup point.
3. System retrieves the encrypted backup from RustFS and verifies its integrity checksum.
4. System decrypts the backup using the tenant encryption key.
5. System provisions a new instance with the same engine, version, and configuration as the original.
6. System restores the backup data to the new instance.
7. System updates DNS and service records to point to the recovered instance.
8. SRE validates application connectivity and data integrity.

### Main Flow: Regional Failover

1. Monitoring detects that an entire region is unreachable (outage confirmed for >5 minutes).
2. DR controller initiates the regional failover procedure automatically (or SRE triggers manually).
3. For each affected database instance, the controller identifies the most recent backup or replica in the secondary region.
4. Controller promotes read replicas to primary (for engines with replication) or restores from backup (for standalone instances).
5. DNS records are updated to route traffic to the secondary region.
6. Controller notifies all affected tenants of the failover event and estimated recovery timeline.
7. Once the primary region recovers, the controller orchestrates a controlled failback with data reconciliation.

### Alternative Flows
- **Backup Corruption**: If the most recent backup fails integrity checks, the system falls back to the next most recent valid backup and alerts the SRE.
- **Partial Recovery**: For large databases where full recovery exceeds the RTO target, the system supports table-level or schema-level selective recovery.
- **Data Reconciliation Conflict**: During failback, if writes occurred in both regions, the system flags conflicts for manual resolution by the application team.

### Postconditions
- Affected instances are restored and operational within the defined RTO.
- Data loss is within the defined RPO bounds.
- Incident report is generated with timeline, actions taken, and data loss assessment.
- DR plan is updated with lessons learned from the actual recovery event.

---

## Appendix: Use Case Traceability Matrix

| Use Case | Related Requirements | Priority | Status      |
|----------|---------------------|----------|-------------|
| UC-001   | PRD-001, PRD-005    | P0       | Implemented |
| UC-002   | PRD-003, PRD-008    | P0       | Implemented |
| UC-003   | PRD-010, PRD-012    | P0       | Implemented |
| UC-004   | PRD-015, PRD-016    | P1       | Implemented |
| UC-005   | PRD-020, PRD-021    | P1       | In Progress |
| UC-006   | PRD-025, PRD-026    | P2       | Implemented |
| UC-007   | PRD-030, PRD-031    | P0       | Implemented |
| UC-008   | PRD-035, PRD-036    | P0       | Tested      |
