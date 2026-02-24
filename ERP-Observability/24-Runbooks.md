# ERP-Observability Operational Runbooks

> **Document ID:** ERP-OBS-RB-024
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [26-Disaster-Recovery-Plan.md](./26-Disaster-Recovery-Plan.md)

---

## Table of Contents

1. [RB-001: VictoriaMetrics High Memory Usage](#rb-001)
2. [RB-002: Quickwit Indexing Lag](#rb-002)
3. [RB-003: Grafana Slow Dashboard Queries](#rb-003)
4. [RB-004: OTel Collector Backpressure](#rb-004)
5. [RB-005: DragonflyDB High Cache Miss Rate](#rb-005)
6. [RB-006: Alertmanager Notification Failure](#rb-006)
7. [RB-007: Zabbix Proxy Connectivity Issues](#rb-007)
8. [RB-008: OpenNMS Event Storm](#rb-008)

---

## RB-001: VictoriaMetrics High Memory Usage {#rb-001}

### Alert

```
AlertName: VictoriaMetricsHighMemory
Expression: process_resident_memory_bytes{job="victoriametrics"} / node_memory_MemTotal_bytes > 0.80
Severity: warning (>80%), critical (>90%)
```

### Symptoms

- VictoriaMetrics `vmstorage` pods showing high memory usage in Kubernetes resource metrics.
- Slow metric query responses (p99 > 2s).
- Prometheus remote_write queue backlog increasing.
- Grafana dashboards timing out or showing "No data" intermittently.

### Investigation

**Step 1: Check current memory usage**

```bash
# Check vmstorage pod memory
kubectl top pods -n erp-observability -l app=vmstorage

# Check VictoriaMetrics internal metrics
curl -s "http://vmstorage:8482/metrics" | grep -E "vm_active_merges|vm_rows_added_total|vm_cache"

# Check active time series count
curl -s "http://vmselect:8481/select/0/prometheus/api/v1/status/tsdb" | jq .
```

**Step 2: Identify high-cardinality metrics**

```bash
# Top 20 metrics by cardinality
curl -s "http://vmselect:8481/select/0/prometheus/api/v1/status/tsdb" | jq '.data.seriesCountByMetricName[:20]'

# Check for label cardinality explosion
curl -s "http://vmselect:8481/select/0/prometheus/api/v1/status/tsdb" | jq '.data.seriesCountByLabelValuePair[:20]'
```

**Step 3: Check merge activity**

```bash
# Active merges indicate compaction pressure
curl -s "http://vmstorage:8482/metrics" | grep vm_active_merges
```

### Remediation

**Immediate (< 5 min):**

1. If memory > 95%, scale vmstorage horizontally:
   ```bash
   kubectl scale statefulset vmstorage -n erp-observability --replicas=4
   ```

2. Drop high-cardinality metrics temporarily via relabeling:
   ```yaml
   # Add to Prometheus remote_write config
   write_relabel_configs:
     - source_labels: [__name__]
       regex: "high_cardinality_metric_.*"
       action: drop
   ```

**Short-term (< 1 hour):**

3. Identify and fix cardinality explosion at source:
   - Check if a module is emitting unbounded label values (e.g., request IDs, user IDs in metric labels).
   - Add relabeling rules to drop or aggregate high-cardinality labels.

4. Adjust retention or downsampling:
   ```bash
   # Reduce retention (e.g., from 13 months to 6 months)
   # Update vmstorage --retentionPeriod flag
   kubectl edit statefulset vmstorage -n erp-observability
   # Change: --retentionPeriod=6
   ```

**Long-term:**

5. Implement metric usage policies per ERP module.
6. Deploy VictoriaMetrics downsampling for data older than 30 days.
7. Set up cardinality alerts per module.

### Prevention

- Monitor `vm_rows_added_total` rate for unexpected spikes.
- Set per-tenant metric cardinality limits.
- Review new metrics in PR review process.

---

## RB-002: Quickwit Indexing Lag {#rb-002}

### Alert

```
AlertName: QuickwitIndexingLag
Expression: quickwit_indexing_pipeline_pending_docs > 1000000
Severity: warning (>1M pending), critical (>10M pending)
```

### Symptoms

- Recent logs/traces/audit entries not appearing in search results.
- Quickwit `/health/readyz` returning `not_ready` for indexer nodes.
- Log streaming (SSE) showing stale data.
- `erp-logs` index `last_indexed_at` falling behind real time.

### Investigation

**Step 1: Check indexer status**

```bash
# Check Quickwit indexer pod status
kubectl get pods -n erp-observability -l app=quickwit,role=indexer

# Check indexer logs for errors
kubectl logs -n erp-observability -l app=quickwit,role=indexer --tail=100

# Check pending splits
curl -s "http://quickwit-control:7280/api/v1/erp-logs/describe" | jq .
```

**Step 2: Check RustFS connectivity**

```bash
# Quickwit writes splits to RustFS. Check connectivity.
kubectl exec -n erp-observability quickwit-indexer-0 -- \
  curl -s http://rustfs.erp-storage.svc.cluster.local:9000/minio/health/ready
```

**Step 3: Check resource utilization**

```bash
# Check indexer CPU and memory
kubectl top pods -n erp-observability -l app=quickwit,role=indexer

# Check disk usage (local split store)
kubectl exec -n erp-observability quickwit-indexer-0 -- df -h /quickwit/data
```

**Step 4: Check ingest rate**

```bash
# Check ingest rate from Fluent-Bit / OTel Collector
kubectl logs -n erp-observability -l app=fluent-bit --tail=50
kubectl logs -n erp-observability -l app=otel-collector --tail=50
```

### Remediation

**Immediate (< 5 min):**

1. If indexer pods are in CrashLoopBackOff, check for OOM kills:
   ```bash
   kubectl describe pod quickwit-indexer-0 -n erp-observability | grep -A5 "Last State"
   ```

   Increase memory limits:
   ```bash
   kubectl patch statefulset quickwit-indexer -n erp-observability \
     -p '{"spec":{"template":{"spec":{"containers":[{"name":"quickwit","resources":{"limits":{"memory":"8Gi"}}}]}}}}'
   ```

2. Scale indexers:
   ```bash
   kubectl scale statefulset quickwit-indexer -n erp-observability --replicas=8
   ```

**Short-term (< 1 hour):**

3. If RustFS is the bottleneck, check RustFS health:
   ```bash
   kubectl get pods -n erp-storage -l app=rustfs
   kubectl logs -n erp-storage -l app=rustfs --tail=100
   ```

4. Force merge stale splits:
   ```bash
   # Trigger merge on erp-logs index
   curl -X POST "http://quickwit-control:7280/api/v1/erp-logs/merge"
   ```

5. If ingest volume is unexpectedly high, identify the source:
   ```bash
   # Check log volume by module
   curl -s "http://quickwit-searcher:7280/api/v1/erp-logs/search" \
     -H "Content-Type: application/json" \
     -d '{"query":"*","max_hits":0,"aggs":{"module":{"terms":{"field":"module","size":20}}}}'
   ```

**Long-term:**

6. Tune indexing settings (increase `split_num_docs_target`, adjust `commit_timeout_secs`).
7. Implement per-module ingest rate limiting.
8. Add horizontal pod autoscaler for indexer StatefulSet.

---

## RB-003: Grafana Slow Dashboard Queries {#rb-003}

### Alert

```
AlertName: GrafanaDashboardSlowQuery
Expression: grafana_api_dashboard_loading_duration_seconds{quantile="0.99"} > 10
Severity: warning
```

### Symptoms

- Dashboard panels showing loading spinners for > 10 seconds.
- Grafana UI becomes unresponsive.
- Users reporting "Timeout" errors on dashboard panels.

### Investigation

**Step 1: Identify slow queries**

```bash
# Check Grafana logs for slow queries
kubectl logs -n erp-observability -l app=grafana --tail=200 | grep -i "slow\|timeout\|error"

# Check Grafana datasource connectivity
curl -s "http://grafana:3001/api/datasources/proxy/1/api/v1/query?query=up" \
  -H "Authorization: Bearer <grafana-api-key>"
```

**Step 2: Check Prometheus/VictoriaMetrics query performance**

```bash
# Check Prometheus query log (if enabled)
kubectl logs -n erp-observability -l app=prometheus --tail=100 | grep "slow\|exceeded"

# Run the problematic query directly against Prometheus
time curl -s "http://prometheus:9090/api/v1/query?query=<the-slow-query>"

# Check VictoriaMetrics slow query log
kubectl logs -n erp-observability -l app=vmselect --tail=100 | grep "slow"
```

**Step 3: Check Quickwit query performance (for log-based panels)**

```bash
# Check Quickwit searcher load
kubectl top pods -n erp-observability -l app=quickwit,role=searcher

# Check concurrent search count
curl -s "http://quickwit-searcher:7280/metrics" | grep quickwit_search
```

### Remediation

**Immediate:**

1. Identify the specific dashboard/panel causing issues:
   - Open Grafana > Settings > Inspect > Query to see which queries are slow.
   - Check if queries use `{__name__=~".*"}` or other expensive regex patterns.

2. Add query caching in Grafana datasource settings:
   ```
   Grafana > Configuration > Data Sources > Prometheus > Query Options > Max data points: 1000
   ```

3. Reduce dashboard time range or increase step interval.

**Short-term:**

4. Optimize slow PromQL queries:
   - Replace `rate()` over large ranges with recording rules.
   - Replace regex label matchers with exact matchers where possible.
   - Use `topk()` or `limit_ratio()` to reduce result set size.

5. Add recording rules for frequently used queries:
   ```yaml
   groups:
     - name: erp-recording-rules
       rules:
         - record: module:http_request_rate5m
           expr: sum(rate(http_requests_total[5m])) by (module)
   ```

6. Scale Quickwit searcher replicas if log panel queries are slow:
   ```bash
   kubectl scale deployment quickwit-searcher -n erp-observability --replicas=5
   ```

---

## RB-004: OTel Collector Backpressure {#rb-004}

### Alert

```
AlertName: OTelCollectorBackpressure
Expression: otelcol_exporter_queue_size / otelcol_exporter_queue_capacity > 0.80
Severity: warning (>80%), critical (>95%)
```

### Symptoms

- ERP modules experiencing increased latency on OTLP gRPC connections.
- Trace and log data delayed or dropped.
- OTel Collector logs showing `sending queue is full` errors.
- `otelcol_exporter_send_failed_spans_total` increasing.

### Investigation

**Step 1: Check collector pod status**

```bash
kubectl get pods -n erp-observability -l app=otel-collector
kubectl top pods -n erp-observability -l app=otel-collector
```

**Step 2: Check exporter queue status**

```bash
curl -s "http://otel-collector:8888/metrics" | grep -E "otelcol_exporter_queue|otelcol_exporter_send_failed"
```

**Step 3: Identify the bottleneck exporter**

```bash
# Check which exporter is failing (Quickwit, VictoriaMetrics, etc.)
kubectl logs -n erp-observability -l app=otel-collector --tail=200 | grep -i "error\|failed\|timeout"
```

### Remediation

**Immediate:**

1. Scale OTel Collector horizontally:
   ```bash
   kubectl scale deployment otel-collector -n erp-observability --replicas=4
   ```

2. If Quickwit is the bottleneck, temporarily increase batch size:
   ```yaml
   # Update OTel Collector config
   exporters:
     otlphttp/quickwit:
       sending_queue:
         queue_size: 10000
       retry_on_failure:
         max_interval: 30s
   ```

3. If a specific backend is completely down, enable the collector's memory limiter to prevent OOM:
   ```yaml
   processors:
     memory_limiter:
       limit_mib: 4096
       spike_limit_mib: 1024
       check_interval: 5s
   ```

**Short-term:**

4. Check and fix the failing backend (Quickwit indexer, VictoriaMetrics vminsert).

5. Implement tail-based sampling to reduce trace volume:
   ```yaml
   processors:
     tail_sampling:
       decision_wait: 10s
       policies:
         - name: error-traces
           type: status_code
           status_code: {status_codes: [ERROR]}
         - name: slow-traces
           type: latency
           latency: {threshold_ms: 1000}
         - name: sample-rest
           type: probabilistic
           probabilistic: {sampling_percentage: 10}
   ```

---

## RB-005: DragonflyDB High Cache Miss Rate {#rb-005}

### Alert

```
AlertName: DragonflyDBHighCacheMissRate
Expression: (dragonfly_keyspace_misses / (dragonfly_keyspace_hits + dragonfly_keyspace_misses)) > 0.50
Severity: warning
```

### Symptoms

- Search API response times increasing (cache misses hit Quickwit directly).
- Log query API slower than baseline.
- DragonflyDB `info` showing low hit rate.

### Investigation

**Step 1: Check DragonflyDB metrics**

```bash
# Connect to DragonflyDB
kubectl exec -n erp-cache dragonfly-0 -- redis-cli info stats | grep -E "keyspace_hits|keyspace_misses|used_memory"

# Check memory usage
kubectl exec -n erp-cache dragonfly-0 -- redis-cli info memory

# Check evictions
kubectl exec -n erp-cache dragonfly-0 -- redis-cli info stats | grep evicted_keys
```

**Step 2: Analyze cache key patterns**

```bash
# Sample keys to understand what is cached
kubectl exec -n erp-cache dragonfly-0 -- redis-cli --scan --pattern "search:*" | head -20
kubectl exec -n erp-cache dragonfly-0 -- redis-cli --scan --pattern "suggest:*" | head -20
```

**Step 3: Check if cache TTLs are appropriate**

```bash
# Check TTL of sample keys
kubectl exec -n erp-cache dragonfly-0 -- redis-cli TTL "search:finance:invoice:1"
```

### Remediation

**Immediate:**

1. If evictions are high, increase DragonflyDB memory limit:
   ```bash
   kubectl patch deployment dragonfly -n erp-cache \
     -p '{"spec":{"template":{"spec":{"containers":[{"name":"dragonfly","resources":{"limits":{"memory":"4Gi"}}}]}}}}'
   ```

2. Adjust maxmemory-policy if needed:
   ```bash
   kubectl exec -n erp-cache dragonfly-0 -- redis-cli config set maxmemory-policy allkeys-lfu
   ```

**Short-term:**

3. Review cache TTLs in the observability API code. Increase TTLs for stable data:
   - Search results: increase from 5 min to 15 min for non-real-time searches.
   - Suggestions: increase from 5 min to 30 min.
   - Module summaries: increase from 1 min to 5 min.

4. Implement cache warming for frequently accessed dashboards and search patterns.

---

## RB-006: Alertmanager Notification Failure {#rb-006}

### Alert

```
AlertName: AlertmanagerNotificationFailure
Expression: rate(alertmanager_notifications_failed_total[5m]) > 0
Severity: critical
```

### Symptoms

- Alerts firing in Prometheus/VictoriaMetrics but notifications not reaching Slack/webhook endpoints.
- Alertmanager UI showing "Error" in notification history.
- `alertmanager_notifications_failed_total` counter increasing.

### Investigation

**Step 1: Check Alertmanager logs**

```bash
kubectl logs -n erp-observability -l app=alertmanager --tail=200

# Look for notification errors
kubectl logs -n erp-observability -l app=alertmanager --tail=200 | grep -i "error\|failed\|notify"
```

**Step 2: Check notification targets**

```bash
# Check Slack webhook connectivity
kubectl exec -n erp-observability alertmanager-0 -- \
  wget -q -O- --timeout=5 "https://hooks.slack.com/health" 2>&1

# Check internal webhook targets
kubectl exec -n erp-observability alertmanager-0 -- \
  wget -q -O- --timeout=5 "http://notification-service.erp-platform.svc.cluster.local:8080/healthz" 2>&1
```

**Step 3: Check Alertmanager cluster status**

```bash
# Check alertmanager cluster peers
curl -s "http://alertmanager-0:9093/api/v2/status" | jq '.cluster'
```

### Remediation

**Immediate:**

1. If Slack webhook URL is expired or changed, update the configuration:
   ```bash
   kubectl edit configmap alertmanager-config -n erp-observability
   # Update SLACK_WEBHOOK_URL

   kubectl rollout restart deployment alertmanager -n erp-observability
   ```

2. If the internal notification service is down, check its status:
   ```bash
   kubectl get pods -n erp-platform -l app=notification-service
   kubectl logs -n erp-platform -l app=notification-service --tail=50
   ```

3. If DNS resolution is failing, check CoreDNS:
   ```bash
   kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50
   ```

**Short-term:**

4. Add a fallback notification channel. Configure a secondary webhook receiver in alertmanager.yaml.

5. Implement Alertmanager HA with 2+ replicas and gossip clustering (already configured in production).

---

## RB-007: Zabbix Proxy Connectivity Issues {#rb-007}

### Alert

```
AlertName: ZabbixProxyOffline
Expression: zabbix_proxy_last_seen_seconds > 300
Severity: critical
```

### Symptoms

- Zabbix Server showing proxy as "Offline" in the web interface.
- Infrastructure monitoring data gap for the affected availability zone.
- No hardware health metrics from hosts behind the offline proxy.

### Investigation

**Step 1: Check Zabbix Proxy pod status**

```bash
kubectl get pods -n erp-monitoring -l app=zabbix-proxy

# Check proxy logs
kubectl logs -n erp-monitoring -l app=zabbix-proxy --tail=200
```

**Step 2: Check network connectivity to Zabbix Server**

```bash
# From proxy pod, test connectivity to Zabbix Server
kubectl exec -n erp-monitoring zabbix-proxy-0 -- \
  nc -zv zabbix-server.erp-monitoring.svc.cluster.local 10051

# Check for network policy blocks
kubectl get networkpolicies -n erp-monitoring
```

**Step 3: Check proxy configuration**

```bash
# Verify proxy configuration
kubectl exec -n erp-monitoring zabbix-proxy-0 -- cat /etc/zabbix/zabbix_proxy.conf | grep -E "Server=|Hostname=|DBHost="
```

**Step 4: Check proxy database**

```bash
# If using SQLite, check database integrity
kubectl exec -n erp-monitoring zabbix-proxy-0 -- sqlite3 /var/lib/zabbix/zabbix_proxy.db "PRAGMA integrity_check;"

# Check proxy database size (large DB can indicate data not being synced)
kubectl exec -n erp-monitoring zabbix-proxy-0 -- ls -lh /var/lib/zabbix/zabbix_proxy.db
```

### Remediation

**Immediate:**

1. Restart the proxy pod:
   ```bash
   kubectl delete pod zabbix-proxy-0 -n erp-monitoring
   ```

2. If network connectivity is the issue, check and fix network policies:
   ```bash
   kubectl describe networkpolicy -n erp-monitoring | grep -A20 "zabbix"
   ```

**Short-term:**

3. If proxy database is corrupted or oversized, recreate it:
   ```bash
   kubectl exec -n erp-monitoring zabbix-proxy-0 -- rm /var/lib/zabbix/zabbix_proxy.db
   kubectl delete pod zabbix-proxy-0 -n erp-monitoring
   # Proxy will recreate the database on startup
   ```

4. Check if Zabbix Server has sufficient capacity to process proxy data:
   ```bash
   # Check Zabbix Server queue
   kubectl exec -n erp-monitoring zabbix-server-0 -- zabbix_server -R queue
   ```

---

## RB-008: OpenNMS Event Storm {#rb-008}

### Alert

```
AlertName: OpenNMSEventStorm
Expression: rate(opennms_events_received_total[5m]) > 10000
Severity: critical
```

### Symptoms

- OpenNMS web UI slow or unresponsive.
- Event correlation engine backing up (correlation rules executing slowly).
- OpenNMS database (PostgreSQL) showing high CPU and connection saturation.
- Operators receiving hundreds of uncorrelated alarms.

### Investigation

**Step 1: Check OpenNMS health**

```bash
kubectl get pods -n erp-monitoring -l app=opennms
kubectl top pods -n erp-monitoring -l app=opennms
```

**Step 2: Identify event source**

```bash
# Check OpenNMS event log
kubectl logs -n erp-monitoring -l app=opennms --tail=200 | grep "Event"

# Check event rate by source
curl -s "http://opennms:8980/opennms/rest/events?limit=100&orderBy=eventTime&order=desc" \
  -u admin:admin | jq '.[].eventSource' | sort | uniq -c | sort -rn | head -20
```

**Step 3: Check if this is caused by a cascading failure**

```bash
# Check if multiple ERP modules are down
curl -s "http://localhost:8090/v1/observability/health" | jq '.components'

# Check VictoriaMetrics for module status
curl -s "http://vmselect:8481/select/0/prometheus/api/v1/query?query=up{job='erp-modules'}" | jq '.data.result[] | {module: .metric.module, value: .value[1]}'
```

### Remediation

**Immediate:**

1. Enable OpenNMS event throttling:
   ```bash
   # SSH into OpenNMS pod and enable threshold
   kubectl exec -n erp-monitoring opennms-0 -- \
     /opt/opennms/bin/opennms -Devent.throttle.rate=1000 restart
   ```

2. If a single source is flooding events, suppress it temporarily:
   - In the OpenNMS web UI, navigate to Admin > Event Configuration.
   - Add a suppression rule for the flooding source/UEI.

3. If the root cause is an actual infrastructure failure (not a false storm):
   - Acknowledge the root cause alarm in OpenNMS.
   - Let correlation rules suppress downstream alarms.

**Short-term:**

4. Tune correlation rules in `drools-engine.d/` to better handle cascading failures.

5. Add event deduplication rules:
   ```xml
   <!-- In eventconf.xml -->
   <event>
     <uei>uei.opennms.org/erp/moduleFlapDetected</uei>
     <event-label>ERP Module Flap Detected</event-label>
     <severity>Major</severity>
     <alarm-data reduction-key="%uei%:%dpname%:%module%" auto-clean="true" />
   </event>
   ```

6. Implement rate limiting on event sources (Zabbix traps, VictoriaMetrics webhooks).

---

## Runbook Index

| ID | Title | Trigger | Severity |
|---|---|---|---|
| RB-001 | VictoriaMetrics High Memory | Memory > 80% | Warning/Critical |
| RB-002 | Quickwit Indexing Lag | Pending docs > 1M | Warning/Critical |
| RB-003 | Grafana Slow Queries | Dashboard load > 10s | Warning |
| RB-004 | OTel Collector Backpressure | Queue > 80% capacity | Warning/Critical |
| RB-005 | DragonflyDB Cache Miss Rate | Miss rate > 50% | Warning |
| RB-006 | Alertmanager Notification Failure | Failed notifications > 0 | Critical |
| RB-007 | Zabbix Proxy Connectivity | Proxy offline > 5m | Critical |
| RB-008 | OpenNMS Event Storm | Event rate > 10K/5min | Critical |

---

## Escalation Matrix

| Severity | Response Time | Notification | Escalation |
|---|---|---|---|
| Critical | 15 minutes | PagerDuty + Slack #erp-critical | SRE Lead -> VP Engineering |
| Warning | 1 hour | Slack #erp-alerts | On-call SRE |
| Info | Next business day | Email digest | N/A |
