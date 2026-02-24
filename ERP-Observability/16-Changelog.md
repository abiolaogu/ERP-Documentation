# ERP-Observability Changelog

> **Document ID:** ERP-OBS-CL-016
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md)

---

All notable changes to the ERP-Observability module are documented in this file. This project follows [Semantic Versioning](https://semver.org/) and [Keep a Changelog](https://keepachangelog.com/) conventions.

---

## [1.0.0] - 2026-02-15

### Summary

General availability release. ERP-Observability is now a fully unified observability platform integrating VictoriaMetrics for metrics, Quickwit for logs/traces/search, Grafana for visualization, OTel Collector as the unified telemetry pipeline, Alertmanager for alert routing, Zabbix for infrastructure monitoring, OpenNMS for event correlation, DragonflyDB for caching, YugabyteDB for persistent storage, and RustFS for S3-compatible object storage. The frontend has been rebuilt on React + Refine.dev + Ant Design.

### Added

- **VictoriaMetrics Integration:** Replaced standalone Prometheus with VictoriaMetrics as the long-term metrics storage backend. VictoriaMetrics provides horizontal scalability, multi-tenancy via `vm-insert` and `vm-select` components, and full PromQL compatibility. Prometheus remains as a scrape agent forwarding via `remote_write` to VictoriaMetrics.
- **Zabbix Infrastructure Monitoring:** Integrated Zabbix 7.0 LTS for low-level infrastructure monitoring including host availability, hardware health (CPU, memory, disk, network), SNMP-based device monitoring, and auto-discovery of Kubernetes nodes. Zabbix Proxy deployed per availability zone for reduced latency. Zabbix metrics are forwarded to VictoriaMetrics via the Zabbix-to-Prometheus exporter.
- **OpenNMS Event Correlation:** Integrated OpenNMS Horizon for advanced event correlation, root cause analysis, and topology-aware monitoring. OpenNMS consumes events from Zabbix, VictoriaMetrics alerts, and Quickwit log anomalies to perform cross-signal correlation. Configured correlation rules for cascading failure detection across ERP modules.
- **Refine.dev Frontend:** Complete frontend rewrite using React 18, Refine.dev 4.x, and Ant Design 5.x. The new frontend provides a unified observability dashboard with real-time log streaming (SSE), interactive metric charts (via Grafana embedded panels), trace waterfall views, alert management, dashboard builder, and cross-module search. The frontend connects to the observability API via `graphql-request` and `graphql-ws` through the Hasura federation layer, with direct REST fallback for streaming endpoints.
- **Unified OTel Collector Pipeline:** All 20 ERP modules now emit telemetry exclusively through the OpenTelemetry Collector. The collector receives traces (OTLP gRPC on port 4317, OTLP HTTP on port 4318), processes them through tenant-aware attribute enrichment, and exports to Quickwit (traces), VictoriaMetrics (metrics), and the log pipeline (structured logs).
- **Multi-tenant Tenant API (Go):** New Go-based tenant-api service on port 8080 providing tenant lifecycle management, tenant-scoped configuration (retention policies, alert thresholds, dashboard templates), and tenant usage metering. Endpoints include `POST /api/tenants`, `GET /api/tenants/:id/config`, `PUT /api/tenants/:id/retention`, and `GET /api/tenants/:id/usage`.
- **RustFS S3 Object Storage:** Quickwit indexes, VictoriaMetrics snapshots, and Grafana dashboard exports are backed by RustFS (MinIO-compatible S3). Buckets: `erp-quickwit-metastore`, `erp-quickwit-indexes`, `erp-victoriametrics-snapshots`, `erp-grafana-exports`.
- **Network Policies:** Comprehensive Kubernetes NetworkPolicy resources for zero-trust networking within the `erp-observability` namespace. Default deny ingress with explicit allow rules for each component pair.
- **Helm Chart v1.0.0:** Production-ready Helm chart with values for dev, staging, and production environments. Includes resource limits, PodDisruptionBudgets, HorizontalPodAutoscalers, and anti-affinity rules.

### Changed

- **Gateway Architecture:** The Go gateway (port 8090) now serves as the unified API entry point, reverse-proxying to the Node.js observability-api (port 3000) and the Go tenant-api (port 8080). CORS handling moved to gateway level with configurable `CORS_ORIGINS` environment variable.
- **Database Backend:** Migrated from PostgreSQL to YugabyteDB (YSQL) for distributed SQL with automatic sharding and multi-region support. Connection pooling configured with 20 max connections, 30s idle timeout, 5s connection timeout.
- **Cache Backend:** Migrated from Redis to DragonflyDB for higher throughput and lower memory consumption. DragonflyDB is used for search result caching, log query caching, and rate limiting.
- **Alertmanager Configuration:** Expanded from 2 receivers (default, critical) to 5 receivers: `default-webhook`, `critical-pagerduty`, `database-team`, `infrastructure-team`, and `slo-alerts`. Added inhibition rules to suppress warning alerts when critical alerts are active for the same module/instance.
- **Prometheus Configuration:** Added scrape targets for all 20 ERP modules, 7 database systems (YugabyteDB, ScyllaDB, DragonflyDB, MongoDB, ClickHouse, TimescaleDB, QuestDB), and infrastructure services (Pulsar, Quickwit, RustFS, SeaweedFS, node-exporter, kube-state-metrics).
- **Log Retention:** Default log retention increased from 30 days to 90 days. Audit log retention set to 730 days (2 years) for compliance. Trace retention set to 30 days.

### Security

- JWT authentication via Authentik OIDC with JWKS validation and 1-hour cache TTL.
- Role-based access control: `admin` role required for creating/updating/deleting alert rules, silencing alerts, and triggering reindex operations.
- Tenant isolation enforced at query level via `buildTenantFilter()` (Quickwit) and `buildTenantSQLFilter()` (YugabyteDB).
- System-level roles (`platform_admin`, `super_admin`, `system`) can query across all tenants.
- Security headers: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`, `Cache-Control: no-store`.

---

## [0.8.0] - 2026-01-20

### Summary

Grafana dashboard release with pre-built dashboards for ERP module health, database health, and overall ERP overview. Alert rules expanded with SLO-based alerting.

### Added

- **Grafana Dashboards:** Three pre-built dashboards provisioned via Grafana's dashboard-as-code approach:
  - `erp-overview.json` -- High-level view of all 20 ERP modules with request rates, error rates, latency percentiles (p50/p95/p99), and uptime indicators.
  - `module-health.json` -- Deep-dive per-module dashboard with resource utilization, database connection pool metrics, cache hit rates, and custom module-specific panels.
  - `database-health.json` -- Unified database monitoring covering YugabyteDB tablet server metrics, DragonflyDB memory/throughput, and per-database replication lag.
- **SLO Alert Rules:** New `erp-slo-rules.yaml` with availability SLOs (99.9% target) and latency SLOs (p99 < 500ms) per module. Error budget burn rate alerts at 2x, 5x, and 10x burn rates.
- **Database Alert Rules:** New `database-rules.yaml` with alerts for connection pool exhaustion, replication lag, disk space, and query latency across all database backends.
- **Infrastructure Alert Rules:** New `infrastructure-rules.yaml` with alerts for Kubernetes node resource pressure, pod crash loops, PVC capacity, and message queue backlog.
- **Log Volume Statistics Endpoint:** `GET /v1/observability/logs/stats` returns log volume aggregated by level, module, and time bucket for a configurable time range.
- **Dashboard CRUD API:** Full CRUD for custom dashboards stored in YugabyteDB. Dashboards support panels of type: timeseries, stat, gauge, table, bar, pie. Each panel references a Prometheus or Quickwit datasource.
- **Grafana Datasource Provisioning:** Automatic provisioning of Prometheus and Quickwit datasources in Grafana via ConfigMap-mounted provisioning files.

### Changed

- **Prometheus Rule Files:** Reorganized from a single `alerts.yaml` into three category-specific files under `configs/prometheus/rules/`: `erp-slo-rules.yaml`, `database-rules.yaml`, `infrastructure-rules.yaml`.
- **Alertmanager Routing:** Added dedicated routes for database-team and infrastructure-team with category-based matching. SLO alerts now route to a dedicated `#erp-slo-alerts` Slack channel.
- **Quickwit Indexing Settings:** Tuned `split_num_docs_target` to 10M for the `erp-logs` index and 5M for `erp-search` and `erp-audit` indexes. Merge factor set to 10 with max merge factor 12.

### Fixed

- Fixed Grafana provisioning race condition where dashboards would fail to load if the Prometheus datasource was not yet available. Added health check dependency in the Grafana deployment.
- Fixed search index `erp-search` missing `tags` field mapping. Added `array<text>` type with raw tokenizer.
- Fixed alertmanager template rendering failure when alert annotations contained newlines.

---

## [0.5.0] - 2025-12-10

### Summary

Tenant API release introducing multi-tenant isolation for all observability data. The Go tenant-api service manages tenant lifecycle and configuration.

### Added

- **Tenant API Service (Go, port 8080):** New microservice for tenant management within the observability context. Endpoints:
  - `POST /api/tenants` -- Register a new tenant with configurable retention policies and alert thresholds.
  - `GET /api/tenants/:id` -- Retrieve tenant configuration including retention policies, usage quotas, and active integrations.
  - `PUT /api/tenants/:id/config` -- Update tenant-specific observability configuration.
  - `PUT /api/tenants/:id/retention` -- Configure per-signal retention (logs, traces, metrics, audit) per tenant.
  - `GET /api/tenants/:id/usage` -- Retrieve tenant usage metrics (log volume, trace count, metric cardinality, storage bytes).
  - `DELETE /api/tenants/:id` -- Soft-delete tenant and schedule data purge according to retention policy.
- **Tenant Context Middleware:** Express middleware that extracts `tenant_id` from JWT claims (`https://erp.io/jwt/claims.tenant_id`) or the `X-Tenant-ID` header. All downstream queries are automatically scoped to the tenant. System-level roles (`platform_admin`, `super_admin`, `system`) bypass tenant scoping.
- **Quickwit Tag Fields:** Added `tenant_id` as a tag field on all Quickwit indexes (`erp-logs`, `erp-traces`, `erp-audit`, `erp-search`, `erp-metrics`). Tag fields enable partition pruning for tenant-scoped queries, dramatically improving query performance.
- **Tenant SQL Isolation:** `buildTenantSQLFilter()` utility generates parameterized SQL WHERE clauses for YugabyteDB queries. System users get `1=1` (no restriction); regular users get `tenant_id = $N`.
- **Alert Rule Tenant Scoping:** Alert rules are now tenant-scoped. Each alert rule has a `tenant_id` column, and listing/creating/updating rules is restricted to the tenant context.
- **Dashboard Tenant Scoping:** Dashboards are tenant-scoped. The `dashboards` table includes a `tenant_id` column with an index for efficient tenant-filtered queries.

### Changed

- **Authentication Middleware:** Upgraded to support Authentik JWKS validation. In production, JWT tokens are verified against the Authentik JWKS endpoint with RS256 algorithm. JWKS is cached for 1 hour. In development mode, unauthenticated requests are allowed with a default `dev-user` identity and `admin` role.
- **Schema Migration:** Updated `001_initial_schema.sql` to add `tenant_id TEXT NOT NULL` column to `alert_rules`, `alert_silences`, and `dashboards` tables. Added indexes on `tenant_id` for all tables.
- **Log Query Parameters:** The `GET /v1/observability/logs` endpoint now accepts an optional `tenant_id` query parameter. If present and the user has system-level access, it overrides the JWT-derived tenant context for cross-tenant investigation.

### Security

- Non-system users without a `tenant_id` in their JWT or `X-Tenant-ID` header receive a `403 Forbidden` response.
- Alert rule creation, update, and deletion restricted to users with the `admin` role.
- Search reindex operations restricted to `admin` role.

---

## [0.4.0] - 2025-11-15

### Summary

Search and cross-module indexing release. Full-text search across all ERP modules powered by Quickwit.

### Added

- **Cross-Module Search Engine:** `GET /v1/observability/search` provides full-text search across all 20 ERP modules with relevance scoring, pagination, and module filtering. Backed by the `erp-search` Quickwit index.
- **Search Suggestions:** `GET /v1/observability/search/suggest` returns autocomplete suggestions for search queries. Results are cached in DragonflyDB with a 5-minute TTL.
- **Reindex Trigger:** `POST /v1/observability/search/reindex` triggers reindexing for a specific module/entity_type combination. Returns `202 Accepted` with a task ID for tracking progress.
- **Search Index Status Table:** New `search_index_status` table tracking per-module, per-entity indexing state including `last_indexed_at`, `document_count`, and `status`.
- **Quickwit Index: erp-search:** New index with fields: `id`, `module`, `entity_type`, `entity_id`, `tenant_id`, `title`, `content`, `tags`, `metadata`, `indexed_at`. Default search fields: `title`, `content`.

### Changed

- **Quickwit Configuration:** Updated `quickwit.yaml` to use RustFS for metastore and index storage. Added `s3://erp-quickwit-metastore/` and `s3://erp-quickwit-indexes/` bucket URIs with path-style access.
- **DragonflyDB Integration:** Extended DragonflyDB usage from trace cache to search result cache. Cache keys prefixed with `search:` for search results and `suggest:` for suggestions.

---

## [0.3.0] - 2025-10-20

### Summary

Distributed tracing and audit logging release.

### Added

- **Trace Querying:** `GET /v1/observability/traces` queries traces from the `erp-traces` Quickwit index with filters for service, module, tenant, duration range, time range, and error status.
- **Trace Detail:** `GET /v1/observability/traces/:traceId` returns the full trace with all spans, including span hierarchy, attributes, and events.
- **Trace Timeline:** `GET /v1/observability/traces/:traceId/timeline` returns a flattened timeline view of a trace suitable for waterfall visualization in the frontend.
- **Audit Log Index:** New `erp-audit` Quickwit index with strict mode (no dynamic fields). Fields: `timestamp`, `actor_id`, `actor_type`, `tenant_id`, `module`, `action`, `resource_type`, `resource_id`, `details`, `ip_address` (IP type), `user_agent`, `result`. Retention: 730 days.
- **Trace Index:** New `erp-traces` Quickwit index with fields: `trace_id`, `span_id`, `parent_span_id`, `operation_name`, `service_name`, `module`, `tenant_id`, `start_timestamp`, `duration_ms` (f64), `status_code`, `attributes` (JSON), `events` (JSON). Retention: 30 days.
- **OTel Collector Deployment:** Initial deployment of the OpenTelemetry Collector as a Kubernetes Deployment with OTLP gRPC (4317) and OTLP HTTP (4318) receivers.

### Changed

- **Fluent-Bit Configuration:** Added trace correlation by propagating `trace_id` and `span_id` from log records to the `erp-logs` Quickwit index, enabling log-to-trace linking.
- **Type Definitions:** Added `TraceSpan`, `Trace`, `TraceQueryParams`, `AuditEntry` TypeScript interfaces.

---

## [0.2.0] - 2025-09-15

### Summary

Alert management release with Alertmanager integration and alert rule CRUD.

### Added

- **Alert Management API:**
  - `GET /v1/observability/alerts` -- List active alerts from Alertmanager.
  - `GET /v1/observability/alerts/rules` -- List configured alert rules with optional module and enabled filters.
  - `POST /v1/observability/alerts/rules` -- Create a new alert rule with PromQL expression, severity, labels, and annotations.
  - `PUT /v1/observability/alerts/rules/:id` -- Partial update of an alert rule.
  - `DELETE /v1/observability/alerts/rules/:id` -- Delete an alert rule.
  - `POST /v1/observability/alerts/:id/silence` -- Silence an alert for a configurable duration (0.5 to 720 hours).
- **Alert Rules Table:** `alert_rules` table in YugabyteDB with UUID primary key, PromQL expression storage, severity enum, JSONB labels/annotations, and enabled flag.
- **Alert Silences Table:** `alert_silences` table with foreign key to `alert_rules`, time-bounded silences, and partial index on `ends_at` for efficient active silence queries.
- **Alertmanager Configuration:** Initial `alertmanager.yaml` with SMTP configuration, Slack integration, webhook receivers, and inhibition rules.
- **Input Validation:** All API endpoints use Zod schemas for request validation. Invalid requests return `400` with structured error detail.

### Changed

- **Metrics Router:** Added `/v1/observability/metrics/modules` endpoint for per-module metric summaries.
- **Error Handling:** Unified error response format across all endpoints: `{ error: string, detail: string, request_id?: string }`.

---

## [0.1.0] - 2025-08-01

### Summary

Initial release of ERP-Observability with Quickwit log aggregation and Prometheus metric collection.

### Added

- **Go API Gateway (port 8090):** Lightweight reverse-proxy gateway built with Go 1.22 standard library. Routes `/v1/observability/*` paths to the Node.js observability-api backend. Includes health check (`/healthz`), capabilities endpoint (`/v1/capabilities`), request ID propagation, structured logging, CORS handling, and security headers.
- **Node.js Observability API (port 3000):** Express.js API providing REST endpoints for log querying, metric querying, and health checking. Built with TypeScript, Zod validation, Pino logging.
- **Log Aggregation:** `GET /v1/observability/logs` queries the `erp-logs` Quickwit index with filters for module, level, tenant, time range, and free-text search. Supports pagination (limit/offset) and sort order (asc/desc).
- **Log Streaming:** `GET /v1/observability/logs/stream` provides Server-Sent Events (SSE) for real-time log tailing. Polls Quickwit every 2 seconds with heartbeat every 15 seconds.
- **Metric Querying:**
  - `GET /v1/observability/metrics/query` -- Instant PromQL query proxy to Prometheus.
  - `GET /v1/observability/metrics/range` -- Range PromQL query with configurable start, end, step, and timeout.
- **Quickwit Index: erp-logs:** Log index with dynamic mode, field mappings for `timestamp`, `level`, `message`, `module`, `service`, `tenant_id`, `request_id`, `trace_id`, `span_id`, `host`, `container`, `namespace`, `metadata`. Tag fields: `tenant_id`, `module`, `level`. Default search on `message` field.
- **Quickwit Index: erp-metrics:** Metrics index for long-term Prometheus remote_write storage.
- **Prometheus Scrape Configuration:** Initial scrape configs for ERP module gateways and infrastructure targets.
- **Fluent-Bit DaemonSet:** Log collection agent forwarding container logs to Quickwit.
- **Docker Infrastructure:** Initial `docker-compose.yaml` for local development with Quickwit, Prometheus, Grafana, Alertmanager, DragonflyDB, and YugabyteDB.
- **Kubernetes Manifests:** Initial K8s manifests for namespace, Quickwit cluster (control-plane, indexer StatefulSet, searcher Deployment), RustFS StatefulSet, Prometheus, Grafana, Fluent-Bit DaemonSet, OTel Collector, Alertmanager, API Deployment with HPA.
- **Capabilities Configuration:** `capabilities.json` declaring module capabilities: `log_aggregation`, `metric_collection`, `distributed_tracing`, `full_text_search`, `cross_module_search`, `alerting`, `dashboards`, `audit_trail`, `slo_monitoring`, `anomaly_detection`.
- **Module Dependencies:** `module_dependencies.yaml` declaring dependencies on ERP-Platform, ERP-IAM (identity), Vault (secrets), Pulsar (events), RustFS (storage), DragonflyDB (cache), YugabyteDB (registry).
- **Database Migration:** `001_initial_schema.sql` creating `alert_rules`, `alert_silences`, `dashboards`, and `search_index_status` tables in YugabyteDB (YSQL).

### Infrastructure

- Quickwit cluster: 1 control-plane node, 6 indexer nodes (StatefulSet), 3+ searcher nodes (Deployment with HPA).
- RustFS: 4-node distributed mode with 3 buckets (`erp-quickwit-metastore`, `erp-quickwit-indexes`, `erp-observability-backups`).
- Prometheus: Single instance with remote_write to Quickwit for long-term storage.
- Grafana: Single instance with provisioned datasources and dashboards.
- DragonflyDB: Single instance for caching layer.
- YugabyteDB: 3 masters + 3 tablet servers for distributed SQL.

---

## Version Compatibility Matrix

| ERP-Observability | Go  | Node.js | Quickwit | VictoriaMetrics | Grafana | Zabbix | OpenNMS | DragonflyDB | YugabyteDB | RustFS |
|---|---|---|---|---|---|---|---|---|---|---|
| 1.0.0 | 1.22 | 20.x | 0.8 | 1.96+ | 10.x | 7.0 LTS | 33+ | 1.x | 2.20+ | Latest |
| 0.8.0 | 1.22 | 20.x | 0.8 | N/A | 10.x | N/A | N/A | 1.x | 2.20+ | Latest |
| 0.5.0 | 1.22 | 20.x | 0.8 | N/A | 10.x | N/A | N/A | 1.x | 2.20+ | Latest |
| 0.1.0 | 1.22 | 20.x | 0.7 | N/A | 10.x | N/A | N/A | 1.x | 2.18+ | Latest |

---

## Migration Guides

### Migrating from 0.8.x to 1.0.0

1. **VictoriaMetrics Setup:** Deploy VictoriaMetrics cluster (vminsert, vmselect, vmstorage). Update Prometheus `remote_write` target from Quickwit to VictoriaMetrics. Quickwit `erp-metrics` index remains for historical data.
2. **Zabbix Integration:** Deploy Zabbix Server + Zabbix Proxy. Configure auto-discovery for Kubernetes nodes. Install `zabbix-prometheus-exporter` to forward Zabbix metrics to VictoriaMetrics.
3. **OpenNMS Integration:** Deploy OpenNMS Horizon. Configure event sources (Zabbix traps, VictoriaMetrics webhook, Quickwit log anomalies). Define correlation rules in `drools-engine.d/`.
4. **Frontend Migration:** Replace legacy dashboard with Refine.dev frontend. Update reverse proxy / ingress rules to serve the new SPA. Configure `REACT_APP_API_URL` to point to the gateway.
5. **Tenant API:** Deploy the Go tenant-api service. Update gateway route configuration to proxy `/api/tenants/*` to the tenant-api on port 8080.
6. **Network Policies:** Apply updated `network-policies.yaml` with rules for Zabbix Proxy, OpenNMS, and VictoriaMetrics components.

### Migrating from 0.5.x to 0.8.0

1. **Grafana Dashboards:** Apply dashboard ConfigMaps from `configs/grafana/dashboards/`. Grafana provisioning will auto-load the dashboards on restart.
2. **Alert Rule Reorganization:** Replace single `alerts.yaml` with the three new rule files in `configs/prometheus/rules/`. Restart Prometheus to reload rules.
3. **Dashboard API:** Run database migration to ensure `dashboards` table has `panels` and `tags` JSONB columns (included in `001_initial_schema.sql`).

### Migrating from 0.1.x to 0.5.0

1. **Schema Migration:** Run `001_initial_schema.sql` to add `tenant_id` columns and indexes if upgrading from the initial schema.
2. **Authentication:** Configure `AUTHENTIK_JWKS_URL` and `AUTHENTIK_ISSUER` environment variables for production JWT validation.
3. **Tenant API Deployment:** Deploy the Go tenant-api service alongside the existing gateway and observability-api.

---

## Deprecations

| Deprecated | Version | Replacement | Removal Target |
|---|---|---|---|
| Standalone Prometheus for long-term storage | 0.8.0 | VictoriaMetrics cluster | 2.0.0 |
| Redis cache client | 0.5.0 | DragonflyDB (ioredis client compatible) | 1.0.0 (completed) |
| PostgreSQL backend | 0.5.0 | YugabyteDB (YSQL, pg-compatible) | 1.0.0 (completed) |
| Legacy dashboard (vanilla React) | 0.8.0 | Refine.dev + Ant Design frontend | 1.0.0 (completed) |

---

## Contributors

| Version | Contributors |
|---|---|
| 1.0.0 | Platform Engineering, SRE Team, Frontend Team |
| 0.8.0 | Platform Engineering, SRE Team |
| 0.5.0 | Platform Engineering, Security Team |
| 0.1.0 | Platform Engineering |
