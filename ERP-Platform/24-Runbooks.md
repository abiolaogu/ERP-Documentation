# ERP-Platform Operational Runbooks

> **Document ID:** ERP-PLAT-RB-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Audience:** Platform Engineers, SRE, On-Call
> **Related Documents:** [26-Disaster-Recovery-Plan.md](./26-Disaster-Recovery-Plan.md), [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md)

---

## RB-001: Service Restart

### When to Use
A platform service is unresponsive, returning errors, or in a degraded state.

### Procedure

**Kubernetes:**
```bash
# Identify the problematic pod
kubectl get pods -n erp-platform -l app=subscription-hub

# Check pod logs
kubectl logs -n erp-platform deployment/subscription-hub --tail=100

# Restart via rollout
kubectl rollout restart deployment/subscription-hub -n erp-platform

# Verify restart
kubectl rollout status deployment/subscription-hub -n erp-platform
kubectl get pods -n erp-platform -l app=subscription-hub
```

**Docker Compose:**
```bash
# Restart specific service
docker compose -f infra/docker-compose.platform.yml restart subscription-hub

# Verify
docker compose -f infra/docker-compose.platform.yml ps
curl -s http://localhost:8091/healthz | jq
```

### Verification
- Health check returns 200 with expected status.
- No error logs in the last 60 seconds.
- Subscription and entitlement queries return expected data.

---

## RB-002: Database Failover

### When to Use
PostgreSQL primary is unreachable or experiencing severe performance degradation.

### Procedure

```bash
# 1. Verify primary is down
pg_isready -h primary-host -p 5432
# Expected: no response or connection refused

# 2. Check replica status
psql -h replica-host -p 5432 -U erp -c "SELECT pg_is_in_recovery();"
# Expected: true (it is a replica)

# 3. Promote replica to primary
psql -h replica-host -p 5432 -U erp -c "SELECT pg_promote();"

# 4. Verify promotion
psql -h replica-host -p 5432 -U erp -c "SELECT pg_is_in_recovery();"
# Expected: false (now primary)

# 5. Update connection strings
# Update DATABASE_URL in all service configurations to point to new primary

# 6. Restart services to pick up new connection
kubectl rollout restart deployment -n erp-platform -l tier=service

# 7. Verify services
for svc in subscription-hub tenant-provisioner entitlement-engine; do
    kubectl exec -n erp-platform deployment/$svc -- wget -q -O - http://localhost:8080/healthz
done
```

### Post-Failover
- Set up new replica from promoted primary.
- Update monitoring and alerting to reflect new topology.
- Create incident report.

---

## RB-003: Scaling Procedures

### Horizontal Scale-Up

```bash
# Scale a specific service
kubectl scale deployment/subscription-hub -n erp-platform --replicas=5

# Or adjust HPA
kubectl patch hpa subscription-hub-hpa -n erp-platform \
  --type merge -p '{"spec":{"maxReplicas":15}}'

# Verify scaling
kubectl get pods -n erp-platform -l app=subscription-hub
kubectl get hpa -n erp-platform
```

### Horizontal Scale-Down

```bash
# Ensure current load supports fewer replicas
kubectl top pods -n erp-platform -l app=subscription-hub

# Scale down
kubectl scale deployment/subscription-hub -n erp-platform --replicas=2

# Verify no request failures during scale-down
kubectl logs -n erp-platform -l app=subscription-hub --since=5m | grep -i error
```

---

## RB-004: Incident Response

### P1 -- Critical (Platform Down)

| Step | Action | Timeline |
|------|--------|----------|
| 1 | Page on-call + tech lead + VP Engineering | Immediate |
| 2 | Open war room (video call link in PagerDuty) | < 5 min |
| 3 | Update status page to "Major Outage" | < 10 min |
| 4 | Identify affected services and scope | < 15 min |
| 5 | Apply emergency fix or failover | < 30 min |
| 6 | Verify service restoration | < 45 min |
| 7 | Update status page to "Monitoring" | < 60 min |
| 8 | Post-mortem within 48 hours | 48 hours |

### P2 -- High (Service Degraded)

| Step | Action | Timeline |
|------|--------|----------|
| 1 | Page on-call + tech lead | Immediate |
| 2 | Identify root cause | < 30 min |
| 3 | Apply fix or workaround | < 2 hours |
| 4 | Verify resolution | < 4 hours |
| 5 | Post-incident review within 1 week | 1 week |

### P3 -- Medium (Feature Impacted)

| Step | Action | Timeline |
|------|--------|----------|
| 1 | Assign to on-call engineer | < 1 hour |
| 2 | Diagnose and plan fix | < 4 hours |
| 3 | Implement and deploy fix | < 24 hours |

### P4 -- Low (Cosmetic/Minor)

| Step | Action | Timeline |
|------|--------|----------|
| 1 | Create ticket in backlog | < 24 hours |
| 2 | Schedule fix in next sprint | 5 business days |

---

## RB-005: Log Analysis

### Viewing Service Logs

```bash
# Kubernetes
kubectl logs -n erp-platform deployment/subscription-hub --tail=200 -f

# Docker Compose
docker compose -f infra/docker-compose.platform.yml logs -f subscription-hub

# Filter for errors
kubectl logs -n erp-platform deployment/audit-service --since=1h | grep -i "error\|fatal\|panic"
```

### Common Log Patterns

| Pattern | Meaning | Action |
|---------|---------|--------|
| `catalog load failed` | products.json not found or invalid | Check ERP_CATALOG_PATH env var |
| `missing X-Tenant-ID` | Client not sending required header | Client-side fix required |
| `unknown sku` | SKU not in product catalog | Verify catalog version |
| `connection refused` | Downstream service unreachable | Check service health |
| `context deadline exceeded` | Request timeout | Check service load, scale if needed |

---

## RB-006: Performance Troubleshooting

### High Latency Investigation

```bash
# 1. Check pod resource usage
kubectl top pods -n erp-platform

# 2. Check for CPU throttling
kubectl describe pod <pod-name> -n erp-platform | grep -A5 "Limits\|Requests"

# 3. Check database connection pool
psql -h db-host -U erp -c "SELECT count(*) FROM pg_stat_activity WHERE datname='erp_platform';"

# 4. Check Redis latency
redis-cli --latency -h redis-host

# 5. Check NATS queue depth
nats stream info PLATFORM_EVENTS
```

### Memory Issues

```bash
# Check for memory pressure
kubectl top pods -n erp-platform --sort-by=memory

# Check OOMKill events
kubectl get events -n erp-platform --field-selector reason=OOMKilled

# Increase memory limits if needed
kubectl patch deployment subscription-hub -n erp-platform \
  --type json -p '[{"op":"replace","path":"/spec/template/spec/containers/0/resources/limits/memory","value":"512Mi"}]'
```

---

## RB-007: Backup and Restore

### PostgreSQL Backup

```bash
# Full backup
pg_dump -h db-host -U erp erp_platform -F c -f erp_platform_$(date +%Y%m%d_%H%M%S).dump

# Verify backup
pg_restore --list erp_platform_*.dump | head -20
```

### PostgreSQL Restore

```bash
# Restore to new database
createdb -h db-host -U erp erp_platform_restored
pg_restore -h db-host -U erp -d erp_platform_restored erp_platform_backup.dump

# Verify restore
psql -h db-host -U erp -d erp_platform_restored -c "SELECT count(*) FROM tenants;"
```

### Redis Backup

```bash
# Trigger RDB snapshot
redis-cli -h redis-host BGSAVE

# Copy RDB file
scp redis-host:/data/dump.rdb ./redis_backup_$(date +%Y%m%d).rdb
```

---

## RB-008: Certificate Renewal

### Let's Encrypt SSL Renewal

```bash
# Check certificate expiry
echo | openssl s_client -servername api.erp-platform.example.com \
  -connect api.erp-platform.example.com:443 2>/dev/null | openssl x509 -noout -enddate

# If using cert-manager (Kubernetes)
kubectl get certificates -n erp-platform
kubectl describe certificate erp-platform-tls -n erp-platform

# Force renewal
kubectl delete certificate erp-platform-tls -n erp-platform
# cert-manager will auto-recreate

# Verify new certificate
kubectl get certificates -n erp-platform
```

---

## RB-009: Secret Rotation

### Database Password Rotation

```bash
# 1. Generate new password
NEW_PASSWORD=$(openssl rand -base64 32)

# 2. Update PostgreSQL password
psql -h db-host -U postgres -c "ALTER USER erp PASSWORD '$NEW_PASSWORD';"

# 3. Update Kubernetes secret
kubectl create secret generic erp-db-credentials -n erp-platform \
  --from-literal=password=$NEW_PASSWORD --dry-run=client -o yaml | kubectl apply -f -

# 4. Rolling restart services to pick up new secret
kubectl rollout restart deployment -n erp-platform -l tier=service

# 5. Verify connectivity
kubectl logs -n erp-platform deployment/subscription-hub --tail=10
```

### Webhook Secret Rotation

```bash
# 1. Generate new webhook signing secret
NEW_SECRET=$(openssl rand -hex 32)

# 2. Update in platform settings
curl -X PUT https://api.erp-platform.example.com/v1/settings/webhook-secret \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"secret\": \"$NEW_SECRET\"}"

# 3. Notify webhook consumers to update their verification secret
# 4. Dual-signing period: verify with both old and new secret for 24 hours
```

---

*For disaster recovery plan, see [26-Disaster-Recovery-Plan.md](./26-Disaster-Recovery-Plan.md). For deployment pipeline, see [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md).*
