# ERP-CRM Runbooks

## Runbook Index

| ID | Scenario | Severity | MTTR Target |
|----|----------|----------|-------------|
| RB-001 | Service Not Starting | P1 | 15 min |
| RB-002 | Database Connection Failure | P1 | 10 min |
| RB-003 | High API Latency | P2 | 30 min |
| RB-004 | NATS Connection Lost | P3 | 30 min |
| RB-005 | Disk Space Exhaustion | P2 | 15 min |
| RB-006 | Memory Pressure | P2 | 20 min |
| RB-007 | Certificate Expiry | P2 | 30 min |
| RB-008 | Database Migration Failure | P1 | 30 min |
| RB-009 | Pulsar Topic Backlog | P3 | 60 min |
| RB-010 | Pod CrashLoop | P1 | 15 min |

---

## RB-001: Service Not Starting

### Symptoms
- Health endpoint returns non-200 or connection refused
- Container exits immediately after start
- Logs show panic or fatal error

### Diagnosis Flow

```mermaid
flowchart TB
    START3([Service Not Starting]) --> CHECK_LOGS["Check container logs"]
    CHECK_LOGS --> DB_ERR{Database<br/>error?}
    DB_ERR -->|Yes| RB002["Go to RB-002"]
    DB_ERR -->|No| CONFIG_ERR{Config<br/>error?}
    CONFIG_ERR -->|Yes| FIX_CONFIG["Fix environment<br/>variables"]
    CONFIG_ERR -->|No| PORT_ERR{Port<br/>conflict?}
    PORT_ERR -->|Yes| FIX_PORT["Change PORT<br/>or kill conflicting process"]
    PORT_ERR -->|No| BINARY_ERR{Binary<br/>missing?}
    BINARY_ERR -->|Yes| REBUILD["Rebuild Docker<br/>image"]
    BINARY_ERR -->|No| ESCALATE["Escalate to<br/>Engineering"]
```

### Resolution Steps

1. Check container logs:
   ```bash
   docker compose logs crm --tail 50
   # or for Kubernetes:
   kubectl logs -n crm deployment/crm-core --tail 50
   ```

2. Verify environment variables:
   ```bash
   docker compose exec crm env | grep -E "DATABASE_URL|PORT|NATS"
   ```

3. Verify database is accessible:
   ```bash
   docker compose exec db pg_isready -U postgres
   ```

4. Check port conflicts:
   ```bash
   lsof -i :8081
   ```

5. If binary is missing, rebuild:
   ```bash
   docker compose build crm
   docker compose up -d crm
   ```

---

## RB-002: Database Connection Failure

### Symptoms
- Readiness endpoint (`/ready`) returns 503
- Logs show "Connection refused" or "connection pool exhausted"
- API requests return 500 errors

### Resolution Steps

1. Check PostgreSQL status:
   ```bash
   docker compose exec db pg_isready -U postgres
   # Kubernetes:
   kubectl exec -n data statefulset/postgresql -- pg_isready -U postgres
   ```

2. Check connection count:
   ```sql
   SELECT count(*) FROM pg_stat_activity WHERE datname = 'crm';
   SELECT max_connections FROM pg_settings WHERE name = 'max_connections';
   ```

3. If pool exhausted, increase max_connections:
   ```bash
   export DATABASE_MAX_CONNECTIONS=20
   # Restart the CRM service
   docker compose restart crm
   ```

4. If PostgreSQL is down:
   ```bash
   docker compose restart db
   # Wait for health check
   docker compose exec db pg_isready -U postgres
   # Restart CRM after DB is healthy
   docker compose restart crm
   ```

5. Check disk space on database volume:
   ```bash
   docker system df
   docker volume inspect erp-crm_postgres_data
   ```

---

## RB-003: High API Latency

### Symptoms
- p95 latency exceeds 50ms for CRUD operations
- p95 latency exceeds 200ms for dashboard queries
- User-reported slowness

### Diagnosis

```mermaid
flowchart TB
    SLOW([High Latency]) --> CHECK_DB["Check DB query<br/>performance"]
    CHECK_DB --> SLOW_QUERY{Slow<br/>queries?}
    SLOW_QUERY -->|Yes| ANALYZE["Run EXPLAIN ANALYZE<br/>on slow queries"]
    SLOW_QUERY -->|No| CHECK_CONN{Connection<br/>pool full?}
    CHECK_CONN -->|Yes| INCREASE_POOL["Increase<br/>DATABASE_MAX_CONNECTIONS"]
    CHECK_CONN -->|No| CHECK_CPU{CPU<br/>saturated?}
    CHECK_CPU -->|Yes| SCALE["Scale horizontally<br/>or vertically"]
    CHECK_CPU -->|No| CHECK_NETWORK{Network<br/>latency?}
    CHECK_NETWORK -->|Yes| FIX_NETWORK["Check DNS,<br/>network policies"]
    CHECK_NETWORK -->|No| PROFILE["Run perf profile<br/>scripts/perf/profile.sh"]
```

### Resolution Steps

1. Check slow queries:
   ```sql
   SELECT pid, now() - pg_stat_activity.query_start AS duration, query
   FROM pg_stat_activity
   WHERE state = 'active' AND (now() - pg_stat_activity.query_start) > interval '1 second';
   ```

2. Analyze specific query:
   ```sql
   EXPLAIN ANALYZE SELECT * FROM contacts ORDER BY created_at DESC LIMIT 20 OFFSET 0;
   ```

3. Check for missing indexes:
   ```sql
   SELECT schemaname, tablename, indexname FROM pg_indexes WHERE tablename IN ('contacts', 'deals', 'activities');
   ```

4. Check connection pool:
   ```sql
   SELECT count(*), state FROM pg_stat_activity WHERE datname = 'crm' GROUP BY state;
   ```

5. Scale if needed:
   ```bash
   # Kubernetes HPA
   kubectl autoscale deployment crm-core --min=2 --max=10 --cpu-percent=70
   ```

---

## RB-004: NATS Connection Lost

### Symptoms
- Logs show "Failed to connect to NATS"
- Events not being published
- No impact on CRUD operations (graceful degradation)

### Resolution Steps

1. This is **non-critical** -- CRM continues to function without NATS:
   ```
   WARN: Failed to connect to NATS: ... Running without events.
   ```

2. Check NATS status:
   ```bash
   docker compose logs nats --tail 20
   curl http://localhost:8222/varz  # NATS monitoring
   ```

3. Restart NATS:
   ```bash
   docker compose restart nats
   ```

4. Restart CRM to re-establish connection:
   ```bash
   docker compose restart crm
   ```

---

## RB-005: Disk Space Exhaustion

### Symptoms
- Database writes fail
- Container logs fill up
- Pulsar/Quickwit ingestion stops

### Resolution Steps

1. Check disk usage:
   ```bash
   df -h
   docker system df
   ```

2. Clean Docker:
   ```bash
   docker system prune -f
   docker volume prune -f  # CAUTION: only if safe
   ```

3. Rotate logs:
   ```bash
   docker compose logs --no-log-prefix crm | tail -1000 > /tmp/crm-latest.log
   # Truncate container logs (Docker Desktop)
   ```

4. Check PostgreSQL WAL:
   ```sql
   SELECT pg_size_pretty(pg_database_size('crm'));
   ```

5. Vacuum if needed:
   ```sql
   VACUUM FULL ANALYZE;
   ```

---

## RB-006: Memory Pressure

### Symptoms
- OOMKilled container restarts
- Slow query responses
- Connection pool timeouts

### Resolution Steps

1. Check memory usage:
   ```bash
   docker stats --no-stream
   # Kubernetes:
   kubectl top pods -n crm
   ```

2. Check for memory leaks:
   ```bash
   # Monitor RSS over time
   while true; do docker stats --no-stream crm; sleep 10; done
   ```

3. Increase container memory limits:
   ```yaml
   # docker-compose.yml
   services:
     crm:
       deploy:
         resources:
           limits:
             memory: 512M
   ```

---

## RB-008: Database Migration Failure

### Symptoms
- Service starts but immediately fails with migration error
- Tables missing or schema inconsistent

### Resolution Steps

1. Check migration status:
   ```bash
   sqlx migrate info --source ./migrations
   ```

2. Run migration manually:
   ```bash
   export DATABASE_URL=postgres://postgres:postgres@localhost:5432/crm
   sqlx migrate run --source ./migrations
   ```

3. If migration is corrupt, check the `_sqlx_migrations` table:
   ```sql
   SELECT * FROM _sqlx_migrations ORDER BY installed_on;
   ```

4. For fresh start (DESTROYS DATA):
   ```bash
   docker compose down -v
   docker compose up -d
   ```

---

## RB-010: Pod CrashLoop

### Symptoms
- Kubernetes pod status shows CrashLoopBackOff
- Multiple restarts in short period

### Resolution Steps

1. Check pod events:
   ```bash
   kubectl describe pod -n crm <pod-name>
   ```

2. Check previous container logs:
   ```bash
   kubectl logs -n crm <pod-name> --previous
   ```

3. Check resource limits:
   ```bash
   kubectl get pod -n crm <pod-name> -o yaml | grep -A5 resources
   ```

4. Common fixes:
   - Increase memory/CPU limits
   - Fix environment variable configuration
   - Ensure database is accessible from pod network
   - Check image pull status

---

## SLO Targets

| Metric | Target | Alert Threshold |
|--------|--------|----------------|
| Availability | 99.9% | < 99.5% |
| API p95 latency | < 50ms | > 100ms |
| API p99 latency | < 200ms | > 500ms |
| Error rate | < 0.1% | > 1% |
| Database connection pool | < 80% utilization | > 90% |
| Event publish latency | < 10ms | > 100ms |

## Escalation Matrix

| Severity | Response Time | Escalation Path |
|----------|--------------|-----------------|
| P1 (Critical) | 15 min | On-call engineer -> Team lead -> CTO |
| P2 (High) | 30 min | On-call engineer -> Team lead |
| P3 (Medium) | 2 hours | On-call engineer |
| P4 (Low) | Next business day | Ticket queue |
