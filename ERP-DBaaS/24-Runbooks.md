# ERP-DBaaS Runbooks

Operational runbooks for diagnosing and resolving common issues in the ERP-DBaaS platform. Each runbook includes symptoms, diagnosis steps, resolution procedures, and escalation paths.

---

## Table of Contents

1. [RB-001: Instance Stuck in Provisioning](#rb-001-instance-stuck-in-provisioning)
2. [RB-002: Backup Failure](#rb-002-backup-failure)
3. [RB-003: YugabyteDB Tablet Imbalance](#rb-003-yugabytedb-tablet-imbalance)
4. [RB-004: DragonflyDB OOM (Out of Memory)](#rb-004-dragonflydb-oom-out-of-memory)
5. [RB-005: Credential Rotation Failure](#rb-005-credential-rotation-failure)
6. [RB-006: Plugin Validation Timeout](#rb-006-plugin-validation-timeout)
7. [RB-007: Quota Exceeded](#rb-007-quota-exceeded)

---

## RB-001: Instance Stuck in Provisioning

### Symptoms

- Instance status remains `provisioning` for more than 10 minutes.
- No `instance.provisioned` webhook event received.
- Tenant unable to use the provisioned instance.
- API returns instance with `status: "provisioning"` on GET request.

### Impact

- **Severity**: High
- **Affected**: Single tenant, single instance
- **SLA**: Provisioning should complete within 5 minutes for standalone, 10 minutes for HA

### Diagnosis Steps

**Step 1: Check instance status in YugabyteDB**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT id, engine, status, namespace, created_at, updated_at
      FROM service_instances WHERE status = 'provisioning'
      AND created_at < NOW() - INTERVAL '10 minutes';"
```

**Step 2: Check ServiceInstance CRD status**

```bash
# Get the CRD for the stuck instance
kubectl get si -n <namespace> -o yaml

# Check conditions
kubectl describe si <instance-name> -n <namespace>
```

**Step 3: Check operator pods**

```bash
# Check if the appropriate operator is running
kubectl get pods -n kubedb-system       # For YugabyteDB, DragonflyDB, MongoDB
kubectl get pods -n scylla-operator     # For ScyllaDB
kubectl get pods -n kube-system | grep clickhouse  # For ClickHouse

# Check operator logs
kubectl logs -n kubedb-system deploy/kubedb-provisioner --tail=100
```

**Step 4: Check pod scheduling**

```bash
# Check if pods are being scheduled
kubectl get pods -n <namespace> -o wide
kubectl describe pod <pod-name> -n <namespace>

# Check events
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

**Step 5: Check resource availability**

```bash
# Check node resource usage
kubectl top nodes
kubectl describe nodes | grep -A 5 "Allocated resources"
```

### Resolution

**Scenario A: Insufficient cluster resources**

```bash
# Scale the cluster (if using managed K8s)
# Or reduce the plan size:
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "UPDATE service_instances SET status = 'failed', updated_at = NOW()
      WHERE id = '<instance-id>';"
```

Notify the tenant to re-provision with a smaller plan or contact platform ops to scale the cluster.

**Scenario B: Operator not running**

```bash
# Restart the operator
kubectl rollout restart deploy/kubedb-provisioner -n kubedb-system

# Watch for recovery
kubectl get pods -n kubedb-system -w
```

**Scenario C: CRD not created**

```bash
# Check if the CRD was created by the API
kubectl get si -n <namespace>

# If missing, check API logs
kubectl logs deploy/dbaas-api -n dbaas-system --tail=200 | grep "instanceId"
```

**Scenario D: PVC stuck in pending**

```bash
# Check PVCs
kubectl get pvc -n <namespace>
kubectl describe pvc <pvc-name> -n <namespace>

# Check StorageClass
kubectl get storageclass
```

### Escalation

If the issue persists after 30 minutes:
1. Escalate to the Platform Engineering team.
2. Include: instance ID, namespace, operator logs, pod events, node resource status.

---

## RB-002: Backup Failure

### Symptoms

- Backup status remains `in_progress` for more than 30 minutes, or transitions to `failed`.
- `backup.failed` webhook event received.
- Instance status stuck in `backing_up`.
- RustFS storage path is null or size_bytes is 0.

### Impact

- **Severity**: High (data protection SLA violation)
- **Affected**: Single instance, potentially affects RPO compliance
- **SLA**: Full backup should complete within 30 minutes for 50Gi, 2 hours for 500Gi

### Diagnosis Steps

**Step 1: Check backup record**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT id, instance_id, type, status, storage_path, size_bytes, started_at
      FROM backup_records WHERE status IN ('in_progress', 'failed')
      ORDER BY started_at DESC LIMIT 10;"
```

**Step 2: Check RustFS connectivity**

```bash
# Test RustFS endpoint from the dbaas-api pod
kubectl exec -n dbaas-system deploy/dbaas-api -- \
  wget -qO- http://rustfs:9000/minio/health/live

# Check RustFS bucket exists
kubectl exec -n dbaas-system deploy/dbaas-api -- \
  wget -qO- "http://rustfs:9000/minio/health/cluster"
```

**Step 3: Check API logs for backup errors**

```bash
kubectl logs deploy/dbaas-api -n dbaas-system --tail=200 | grep -i "backup"
```

**Step 4: Check backup pod/job**

```bash
# Check for backup-related pods
kubectl get pods -n <instance-namespace> | grep backup
kubectl logs <backup-pod> -n <instance-namespace>
```

**Step 5: Check disk space**

```bash
# Check PVC usage on the instance
kubectl exec -n <namespace> <db-pod> -- df -h

# Check RustFS storage
kubectl exec -n rustfs-system deploy/rustfs -- df -h
```

### Resolution

**Scenario A: RustFS unavailable**

```bash
# Restart RustFS
kubectl rollout restart deploy/rustfs -n rustfs-system

# Wait for readiness
kubectl rollout status deploy/rustfs -n rustfs-system

# Retry the backup
curl -X POST http://localhost:8090/v1/dbaas/instances/<instanceId>/backup \
  -H "Content-Type: application/json" \
  -H "X-Dev-Tenant-ID: <tenantId>" \
  -d '{"type": "full", "retentionDays": 30}'
```

**Scenario B: Instance status stuck in backing_up**

```bash
# Reset instance status to running
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "UPDATE service_instances SET status = 'running', updated_at = NOW()
      WHERE id = '<instance-id>' AND status = 'backing_up';"

# Mark failed backup
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "UPDATE backup_records SET status = 'failed' WHERE id = '<backup-id>';"
```

**Scenario C: Insufficient storage**

Expand the RustFS storage or clean up expired backups:

```bash
# Check backup retention
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT COUNT(*), SUM(size_bytes) FROM backup_records
      WHERE completed_at < NOW() - (retention_days * INTERVAL '1 day');"
```

### Escalation

If backup failures are recurring:
1. Check for pattern (specific engine, specific region, specific time of day).
2. Review RustFS capacity planning.
3. Escalate to Storage team if RustFS is chronically overloaded.

---

## RB-003: YugabyteDB Tablet Imbalance

### Symptoms

- YugabyteDB instances show uneven query latency across replicas.
- Some TServer nodes report significantly higher CPU/memory usage.
- YugabyteDB Master UI shows uneven tablet distribution.
- Slow queries on the service registry (dbaas_registry database).

### Impact

- **Severity**: Medium
- **Affected**: All tenants using YugabyteDB instances, potentially the service registry
- **SLA**: Query latency should be < 50ms for simple lookups

### Diagnosis Steps

**Step 1: Check tablet distribution**

```bash
# Access YugabyteDB Master UI
# For service registry: http://localhost:7000

# Or via ysqlsh:
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT * FROM yb_servers();"
```

**Step 2: Check TServer metrics**

```bash
# Check TServer web UI for each node
# http://localhost:9000/tablets

# Check via kubectl for managed instances
kubectl exec -n <namespace> <tserver-pod> -- \
  curl -s localhost:9000/api/v1/tablet-servers
```

**Step 3: Check for hotspot tables**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT schemaname, tablename, n_tup_ins, n_tup_upd, n_tup_del
      FROM pg_stat_user_tables ORDER BY n_tup_ins + n_tup_upd + n_tup_del DESC;"
```

### Resolution

**Step 1: Trigger tablet rebalancing**

```bash
# Connect to the master and trigger load balancing
kubectl exec -n <namespace> <master-pod> -- \
  /home/yugabyte/bin/yb-admin -master_addresses <master-rpc> \
  set_load_balancer_enabled 1

# Or via the Master UI, navigate to: Configuration > Load Balancer > Enable
```

**Step 2: Manual tablet move (if needed)**

```bash
kubectl exec -n <namespace> <master-pod> -- \
  /home/yugabyte/bin/yb-admin -master_addresses <master-rpc> \
  move_replica <tablet-id> <from-ts-uuid> <to-ts-uuid>
```

**Step 3: Consider table splitting for hot tables**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "ALTER TABLE metering_events SPLIT AT VALUES (...);"
```

### Prevention

- Monitor tablet distribution via Prometheus metrics: `yb_tablet_server_tablet_count`.
- Set alerts for tablet count variance > 20% across TServer nodes.
- Use hash-partitioned tables for high-write tables (metering_events).

---

## RB-004: DragonflyDB OOM (Out of Memory)

### Symptoms

- Rate limiter stops enforcing limits (requests bypass rate limiting).
- API logs show: `DragonflyDB connection error`.
- DragonflyDB container restarts repeatedly.
- `docker compose logs dragonfly` shows OOM kill.

### Impact

- **Severity**: Medium (rate limiting is a defense-in-depth control)
- **Affected**: All tenants (rate limiting disabled, potential for abuse)
- **SLA**: Rate limiting should be operational > 99.9% of the time

### Diagnosis Steps

**Step 1: Check DragonflyDB status**

```bash
# Docker Compose
docker compose logs dragonfly --tail=50

# Kubernetes
kubectl logs deploy/dragonfly -n dragonfly-system --tail=50
kubectl describe pod -l app=dragonfly -n dragonfly-system
```

**Step 2: Check memory usage**

```bash
# Connect via redis-cli
redis-cli -h localhost -p 6379 INFO memory

# Key metrics:
# used_memory_human
# used_memory_peak_human
# maxmemory_human
```

**Step 3: Check key count and size**

```bash
redis-cli -h localhost -p 6379 DBSIZE
redis-cli -h localhost -p 6379 SCAN 0 MATCH "ratelimit:*" COUNT 100
```

### Resolution

**Step 1: Clear expired rate limit keys**

```bash
# Rate limit keys expire automatically, but if there is a buildup:
redis-cli -h localhost -p 6379 --scan --pattern "ratelimit:*" | \
  xargs -L 100 redis-cli DEL
```

**Step 2: Increase memory limit**

For Docker Compose, edit `infra/docker-compose.yaml`:

```yaml
dragonfly:
  image: docker.dragonflydb.io/dragonflydb/dragonfly:latest
  command: ["--maxmemory", "2gb", "--maxmemory-policy", "allkeys-lru"]
  deploy:
    resources:
      limits:
        memory: 2560m
```

For Kubernetes, update the DragonflyDB StatefulSet:

```bash
kubectl patch statefulset dragonfly -n dragonfly-system -p \
  '{"spec":{"template":{"spec":{"containers":[{"name":"dragonfly","resources":{"limits":{"memory":"2Gi"}}}]}}}}'
```

**Step 3: Restart DragonflyDB**

```bash
# Docker Compose
docker compose restart dragonfly

# Kubernetes
kubectl rollout restart statefulset/dragonfly -n dragonfly-system
```

### Prevention

- Set `maxmemory` to 80% of available container memory.
- Use `allkeys-lru` eviction policy to prevent OOM.
- Monitor memory usage with alerts at 70% and 90% thresholds.
- The API gracefully degrades when DragonflyDB is unavailable (rate limiting bypassed).

---

## RB-005: Credential Rotation Failure

### Symptoms

- Credential rotation status transitions to `failed`.
- `credentials.rotation_failed` webhook event received.
- API returns 409 `rotation_in_progress` for subsequent rotation attempts.
- Tenant unable to rotate credentials.

### Impact

- **Severity**: High (security-sensitive operation)
- **Affected**: Single tenant, single instance
- **SLA**: Credential rotation should complete within 60 seconds

### Diagnosis Steps

**Step 1: Check rotation records**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT id, instance_id, status, initiated_by, initiated_at, completed_at
      FROM credential_rotations
      WHERE instance_id = '<instance-id>'
      ORDER BY initiated_at DESC LIMIT 5;"
```

**Step 2: Check Vault connectivity**

```bash
# From dbaas-api pod
kubectl exec -n dbaas-system deploy/dbaas-api -- \
  wget -qO- http://vault:8200/v1/sys/health

# Check Vault seal status
kubectl exec -n vault-system deploy/vault -- vault status
```

**Step 3: Check API logs**

```bash
kubectl logs deploy/dbaas-api -n dbaas-system --tail=200 | grep -i "rotation\|credential\|vault"
```

**Step 4: Check Vault audit logs**

```bash
kubectl exec -n vault-system deploy/vault -- \
  vault audit list
```

### Resolution

**Scenario A: Vault unreachable**

```bash
# Unseal Vault if sealed
kubectl exec -n vault-system deploy/vault -- vault operator unseal <unseal-key>

# Restart Vault pod
kubectl rollout restart deploy/vault -n vault-system
```

**Scenario B: Stale rotation record blocking new rotations**

```bash
# Mark stuck rotations as failed
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "UPDATE credential_rotations SET status = 'failed', completed_at = NOW()
      WHERE instance_id = '<instance-id>'
      AND status IN ('pending', 'in_progress')
      AND initiated_at < NOW() - INTERVAL '5 minutes';"
```

**Scenario C: Engine-specific rotation failure**

Check the engine's user management:

```bash
# For YugabyteDB
PGPASSWORD=<current-password> psql -h <endpoint> -p 5433 -U <username> -c "\du"

# For MongoDB
kubectl exec -n <namespace> <mongo-pod> -- mongosh --eval "db.getUsers()"
```

### Escalation

1. If Vault is consistently unreachable, escalate to the Security/Infrastructure team.
2. If rotation fails at the engine level, escalate to the DBA team with engine-specific logs.
3. Never manually set credentials in the database without updating Vault.

---

## RB-006: Plugin Validation Timeout

### Symptoms

- Plugin status remains `pending_validation` for more than 5 minutes.
- No `plugin.validated` or `plugin.rejected` webhook event received.
- Plugin registration completed but plugin never becomes active.

### Impact

- **Severity**: Low
- **Affected**: Single plugin registration
- **SLA**: Plugin validation should complete within 60 seconds

### Diagnosis Steps

**Step 1: Check plugin record**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT id, name, engine, grpc_endpoint, status, registered_at, validated_at
      FROM plugin_registrations
      WHERE status = 'pending_validation';"
```

**Step 2: Check gRPC endpoint connectivity**

```bash
# From dbaas-api pod, test connectivity to the plugin's gRPC endpoint
kubectl exec -n dbaas-system deploy/dbaas-api -- \
  wget --spider -T 5 <grpc-endpoint>

# Or use grpcurl if available
grpcurl -plaintext <host>:<port> list
```

**Step 3: Check API logs for validation errors**

```bash
kubectl logs deploy/dbaas-api -n dbaas-system --tail=200 | grep -i "plugin\|validation\|grpc"
```

**Step 4: Check NetworkPolicy allows traffic to plugin namespace**

```bash
kubectl get networkpolicies -n dbaas-system
kubectl describe networkpolicy dbaas-api-egress -n dbaas-system
```

### Resolution

**Scenario A: gRPC endpoint unreachable**

```bash
# Check if the plugin pod is running
kubectl get pods -n <plugin-namespace> -l app=<plugin-name>

# Check the plugin's service
kubectl get svc -n <plugin-namespace>
kubectl describe svc <plugin-service> -n <plugin-namespace>
```

Fix the plugin deployment, then manually trigger re-validation:

```bash
# Reset plugin status to trigger re-validation
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "DELETE FROM plugin_registrations WHERE id = '<plugin-id>';"

# Re-register the plugin via the API
```

**Scenario B: NetworkPolicy blocking traffic**

Add an egress rule allowing traffic from dbaas-api to the plugin namespace:

```yaml
# Add to dbaas-api-egress NetworkPolicy
- to:
    - namespaceSelector:
        matchLabels:
          app.kubernetes.io/name: dbaas-plugins
    ports:
      - protocol: TCP
        port: 50051
```

**Scenario C: Validation scheduler not running**

```bash
# Check dbaas-api pod health
kubectl get pods -n dbaas-system -l app=dbaas-api

# Restart the API to reset the in-memory validation scheduler
kubectl rollout restart deploy/dbaas-api -n dbaas-system
```

---

## RB-007: Quota Exceeded

### Symptoms

- Provisioning requests return HTTP 429 with error `quota_exceeded`.
- Tenant reports they cannot create new instances despite having decommissioned old ones.
- `current_instances` count does not match actual running instances.

### Impact

- **Severity**: Medium
- **Affected**: Single tenant
- **SLA**: Quota should accurately reflect actual resource usage

### Diagnosis Steps

**Step 1: Check tenant quota**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT * FROM tenant_quotas WHERE tenant_id = '<tenant-id>';"
```

**Step 2: Count actual instances**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT status, COUNT(*)
      FROM service_instances
      WHERE tenant_id = '<tenant-id>'
      AND status NOT IN ('decommissioned')
      GROUP BY status;"
```

**Step 3: Compare quota with actual count**

```bash
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT
        tq.current_instances AS quota_count,
        (SELECT COUNT(*) FROM service_instances si
         WHERE si.tenant_id = tq.tenant_id
         AND si.status NOT IN ('decommissioned')) AS actual_count
      FROM tenant_quotas tq
      WHERE tq.tenant_id = '<tenant-id>';"
```

### Resolution

**Scenario A: Quota count drift (counter out of sync)**

```bash
# Recalculate and fix the quota counter
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "UPDATE tenant_quotas
      SET current_instances = (
        SELECT COUNT(*) FROM service_instances
        WHERE tenant_id = '<tenant-id>'
        AND status NOT IN ('decommissioned')
      ),
      updated_at = NOW()
      WHERE tenant_id = '<tenant-id>';"
```

**Scenario B: Instances stuck in non-terminal state**

```bash
# Find stuck instances
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "SELECT id, status, updated_at FROM service_instances
      WHERE tenant_id = '<tenant-id>'
      AND status IN ('provisioning', 'decommissioning', 'scaling')
      AND updated_at < NOW() - INTERVAL '1 hour';"

# Force transition to appropriate state
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "UPDATE service_instances SET status = 'failed', updated_at = NOW()
      WHERE tenant_id = '<tenant-id>'
      AND status = 'provisioning'
      AND updated_at < NOW() - INTERVAL '1 hour';"
```

**Scenario C: Tenant needs a tier upgrade**

```bash
# Upgrade tenant tier
PGPASSWORD=yugabyte psql -h localhost -p 5433 -U yugabyte -d dbaas_registry \
  -c "UPDATE tenant_quotas
      SET tier = 'B', max_instances = 20, max_cpu = '64',
          max_memory = '256Gi', max_storage = '2Ti', updated_at = NOW()
      WHERE tenant_id = '<tenant-id>';"
```

### Prevention

- Run a daily cron job to reconcile quota counters against actual instance counts.
- Add monitoring alerts when `current_instances` diverges from the actual count.
- Implement quota auditing in the metering pipeline.
