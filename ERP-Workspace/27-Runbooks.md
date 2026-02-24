# ERP-Workspace Runbooks

> **Document ID:** ERP-WS-RB-027
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## RB-01: Email Queue Overflow

**Alert:** `ws_smtp_queue_depth > 100,000`
**Severity:** Critical
**On-Call Response Time:** 15 minutes

### Diagnosis
1. Check SMTP server health: `curl http://smtp-service:8080/healthz`
2. Check outbound SMTP connectivity: `telnet relay.smtp.provider 25`
3. Check Redpanda consumer lag: `rpk group describe ws-email-delivery`
4. Check for provider outage (SendGrid, AWS SES, Postmark status pages)

### Remediation
1. If provider outage: failover to secondary provider by updating `email_provider_configs`
2. If consumer stuck: restart consumer pods
3. If sustained volume spike: scale email-service to 5+ replicas
4. If spam/abuse: identify offending tenant and apply rate limit

### Escalation
If unresolved after 30 minutes, escalate to Engineering Lead.

---

## RB-02: Database Connection Exhaustion

**Alert:** `pg_pool_active > 0.95 * pg_pool_max`
**Severity:** Critical

### Diagnosis
1. Check PgBouncer stats: `SHOW POOLS;` via admin console
2. Identify connection-heavy services: `SELECT application_name, count(*) FROM pg_stat_activity GROUP BY 1;`
3. Check for long-running queries: `SELECT * FROM pg_stat_activity WHERE state = 'active' AND now() - query_start > '30s';`

### Remediation
1. Kill long-running queries: `SELECT pg_cancel_backend(pid);`
2. Increase PgBouncer pool size temporarily
3. If specific service is leaking connections, restart that service
4. If sustained, add read replica and route reads to it

---

## RB-03: Email Delivery Bounce Rate Spike

**Alert:** `bounce_rate > 5%` for 1 hour
**Severity:** Warning

### Diagnosis
1. Query recent bounces: `SELECT bounce_type, count(*) FROM email_suppressions WHERE created_at > now() - '1 hour' GROUP BY bounce_type;`
2. Check email health scores: `SELECT domain, overall_score, spf_status, dkim_status, dmarc_status FROM email_health_scores;`
3. Check IP reputation on blacklists

### Remediation
1. If hard bounces: clean recipient lists, add to suppression
2. If soft bounces: check sending IP reputation, warm up new IPs
3. If DKIM/SPF failures: verify DNS records and key rotation
4. If blacklisted: submit delisting requests, switch to clean IP pool

---

## RB-04: Video Meeting Quality Degradation

**Alert:** `ws_meet_packet_loss_percent > 3%` for 5 minutes
**Severity:** Warning

### Diagnosis
1. Check LiveKit SFU resource usage: CPU, memory, network
2. Check participant counts vs SFU capacity
3. Check network path between SFU nodes (latency, jitter)

### Remediation
1. Scale SFU nodes if overloaded
2. If network issue: reroute traffic via alternative path
3. Reduce simulcast layers temporarily to conserve bandwidth
4. Advise affected users to lower video quality

---

## RB-05: Chat Message Delivery Delay

**Alert:** WebSocket delivery latency > 5s for 5 minutes
**Severity:** Warning

### Diagnosis
1. Check Redis Pub/Sub lag
2. Check chat-service pod health and CPU usage
3. Check WebSocket connection count per pod
4. Verify Redpanda consumer lag for notification delivery

### Remediation
1. Scale chat-service pods if WebSocket connections are at limit
2. Restart Redis node if Pub/Sub is stuck
3. If Redpanda lag, increase consumer partition assignment

---

## RB-06: Document Editing Session Failure

**Alert:** ONLYOFFICE callback errors > 10/min
**Severity:** Warning

### Diagnosis
1. Check ONLYOFFICE Document Server health
2. Check WOPI token validity and expiration
3. Check MinIO connectivity for document storage
4. Review ONLYOFFICE logs for conversion or session errors

### Remediation
1. Restart ONLYOFFICE DS pods
2. If token issue: clear WOPI token cache
3. If MinIO issue: verify MinIO cluster health
4. Users should save work locally and re-open documents

---

## RB-07: Search Index Degradation

**Alert:** Search query latency > 500ms P99 for 10 minutes
**Severity:** Warning

### Diagnosis
1. Check Quickwit cluster health
2. Check index segment count and merge status
3. Check disk usage on Quickwit nodes

### Remediation
1. Force segment merge if segment count is high
2. Scale Quickwit nodes for more capacity
3. If index corrupted: rebuild from Redpanda events (1-4 hours)

---

## RB-08: Storage Quota Alert

**Alert:** `tenant_storage_used > 90%`
**Severity:** Warning

### Diagnosis
1. Identify largest files and folders per tenant
2. Check for orphaned file versions
3. Review retention policies

### Remediation
1. Notify tenant administrator
2. Clean up old file versions if retention allows
3. Empty trash for the tenant
4. If legitimate growth: increase quota

---

## RB-09: Full Service Restart Procedure

**When:** After critical infrastructure change or widespread issue

### Steps
1. Scale all services to 0: `kubectl scale deployment -l app=erp-workspace --replicas=0`
2. Verify no active database connections
3. Run any pending migrations
4. Start data layer first: PostgreSQL, Redis, MinIO, Redpanda
5. Verify data layer health
6. Start services in order: contacts, email, calendar, chat, drive, docs, meet
7. Verify each service health via `/healthz`
8. Start API gateway
9. Run smoke tests
10. Monitor error rate for 30 minutes

---

## RB-10: Tenant Data Export (GDPR)

**When:** Data subject access request or tenant offboarding

### Steps
1. Identify all data for the tenant across all bounded contexts
2. Export email messages in EML format
3. Export calendar events in ICS format
4. Export contacts in vCard format
5. Export chat messages in JSON format
6. Export files from MinIO bucket
7. Export analytics data from ClickHouse
8. Package all exports into encrypted archive
9. Deliver to requesting party via secure channel
10. Log the export in audit trail

---

*For alert configurations, see [20-Monitoring-Observability.md](./20-Monitoring-Observability.md). For disaster recovery procedures, see [26-Disaster-Recovery.md](./26-Disaster-Recovery.md).*
