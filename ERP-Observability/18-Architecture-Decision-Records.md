# ERP-Observability Architecture Decision Records

> **Document ID:** ERP-OBS-ADR-018
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [16-Changelog.md](./16-Changelog.md), [17-README.md](./17-README.md)

---

## Table of Contents

1. [ADR-001: VictoriaMetrics over Prometheus for Long-Term Metrics Storage](#adr-001)
2. [ADR-002: Quickwit over ELK Stack for Log/Trace/Search](#adr-002)
3. [ADR-003: DragonflyDB over Redis for Caching](#adr-003)
4. [ADR-004: YugabyteDB over PostgreSQL for Persistent Storage](#adr-004)
5. [ADR-005: OTel Collector as Unified Telemetry Pipeline](#adr-005)
6. [ADR-006: Zabbix for Infrastructure Monitoring](#adr-006)
7. [ADR-007: OpenNMS for Event Correlation](#adr-007)
8. [ADR-008: Refine.dev for Frontend Framework](#adr-008)

---

## ADR-001: VictoriaMetrics over Prometheus for Long-Term Metrics Storage {#adr-001}

### Status

**Accepted** -- 2026-01-15

### Context

ERP-Observability needed a long-term metrics storage backend capable of handling metrics from all 20 ERP modules, 7 database systems, and infrastructure services. The initial deployment used Prometheus as both the scraping agent and the storage backend. As the ERP suite scaled to production with multiple tenants, several limitations emerged:

- **Storage scalability:** Prometheus is a single-node system. Its TSDB does not support horizontal scaling. With 20 modules each emitting 500+ metric series and 15-second scrape intervals, the single Prometheus instance reached memory limits at approximately 2M active time series.
- **Multi-tenancy:** Prometheus has no native multi-tenant support. Tenant isolation for metric queries required external proxying and label-based filtering, which was error-prone and did not prevent cross-tenant data leakage at the storage level.
- **High availability:** Prometheus HA requires running duplicate instances, doubling resource costs. Deduplication is handled externally (e.g., Thanos Sidecar), adding architectural complexity.
- **Retention:** Prometheus local storage retention is limited by disk capacity of a single node. Our requirement is 13 months of metric retention for capacity planning and SLO burn rate analysis.

### Decision

We adopted **VictoriaMetrics** (cluster mode) as the long-term metrics storage backend. Prometheus continues as the scraping agent, forwarding metrics via `remote_write` to VictoriaMetrics.

### Rationale

| Criterion | Prometheus | VictoriaMetrics | Decision Factor |
|---|---|---|---|
| Horizontal scaling | No (single-node TSDB) | Yes (vminsert/vmselect/vmstorage) | Required for 2M+ series |
| Multi-tenancy | None | Native via accountID header | Mandatory for ERP multi-tenant |
| PromQL compatibility | Native | Full compatibility | No query migration needed |
| Compression ratio | ~1.3 bytes/sample | ~0.4 bytes/sample | 3x storage savings |
| Remote write speed | N/A (is the target) | Optimized bulk insert | Handles burst traffic |
| High availability | Duplicate instances + Thanos | Built-in replication factor | Simpler architecture |
| AIDD compliance | Open-source, well-known | Open-source, AIDD-approved | Both acceptable |
| Memory efficiency | ~3 bytes/series (active) | ~1 byte/series (active) | 3x memory savings |
| Long-term retention | Disk-limited per node | Distributed storage, S3 tiering | 13-month retention feasible |

**Performance benchmarks (internal testing):**

- Ingestion rate: VictoriaMetrics sustained 1.2M samples/sec vs. Prometheus 500K samples/sec on equivalent hardware.
- Query latency (p99) for 1-hour range query over 100K series: VictoriaMetrics 120ms vs. Prometheus 340ms.
- Storage for 30 days of 2M series at 15s interval: VictoriaMetrics 48GB vs. Prometheus 156GB.

### Consequences

**Positive:**
- Horizontal scaling eliminates single-node bottleneck.
- Native multi-tenancy with `accountID` maps directly to ERP `tenant_id`.
- 3x reduction in storage costs.
- Full PromQL compatibility means zero migration for existing alert rules and Grafana dashboards.

**Negative:**
- Additional operational complexity with 3-component cluster (vminsert, vmselect, vmstorage).
- Team needs to learn VictoriaMetrics-specific operational procedures (backup, restore, cluster scaling).
- Prometheus remains as a scrape agent, creating a two-tier architecture.

**Risks:**
- VictoriaMetrics is less mature than Prometheus ecosystem. Mitigated by using cluster mode which has been production-tested by large enterprises.

---

## ADR-002: Quickwit over ELK Stack for Log/Trace/Search {#adr-002}

### Status

**Accepted** -- 2025-07-20

### Context

The ERP platform needed a unified backend for log aggregation, distributed tracing, audit logging, and cross-module full-text search. The traditional choice would be the ELK (Elasticsearch, Logstash, Kibana) stack or its OpenSearch fork. However, several factors drove us to evaluate alternatives:

- **AIDD compliance:** The ERP platform follows AI-Integrated Development Discipline (AIDD) governance. AIDD mandates that infrastructure choices minimize operational complexity, reduce resource consumption, and consolidate tooling where possible. Running separate systems for logs (ELK), traces (Jaeger/Tempo), and search (Elasticsearch/Solr) violates the consolidation principle.
- **Cost at scale:** Elasticsearch requires significant memory (JVM heap) and disk I/O. For 20 modules generating ~100GB/day of logs, the estimated Elasticsearch cluster would require 12 data nodes with 64GB RAM each.
- **S3-native storage:** AIDD mandates that stateful services use object storage (RustFS/S3) for data persistence where possible, to leverage the durability and cost-efficiency of object storage.
- **Trace storage:** Jaeger with Elasticsearch backend adds another Elasticsearch dependency. Tempo uses object storage but does not support full-text search on trace attributes.

### Decision

We adopted **Quickwit** as the unified indexing and search engine for logs, traces, audit trails, and cross-module search.

### Rationale

| Criterion | ELK Stack | Quickwit | Decision Factor |
|---|---|---|---|
| Unified logs + traces + search | Requires separate configs/indexes | Single engine, multiple indexes | AIDD consolidation |
| Storage backend | Local disk (expensive EBS) | S3/object storage (RustFS) | 10x cost reduction |
| Memory per node | 32-64GB JVM heap | 2-8GB (no JVM) | 8x memory savings |
| Indexing throughput | High (but memory-hungry) | High (Rust-native, efficient) | Comparable |
| Search latency (simple queries) | <50ms | <50ms | Comparable |
| Multi-tenancy | Index-per-tenant or alias routing | Tag-field partition pruning | Quickwit simpler |
| Schema enforcement | Dynamic by default | Dynamic or strict per index | Both supported |
| S3-native | No (requires snapshot/restore) | Yes (native read/write) | AIDD requirement |
| License | Elastic License 2.0 / AGPL | AGPL 3.0 | Both acceptable |
| Operational complexity | High (master, data, ingest nodes) | Medium (control, indexer, searcher) | Quickwit simpler |
| Tantivy-based FTS | No (Lucene) | Yes (Tantivy, Rust) | Performance |

**Quickwit index architecture for ERP-Observability:**

| Index | Mode | Retention | Tag Fields | Use Case |
|---|---|---|---|---|
| `erp-logs` | dynamic | 90 days | tenant_id, module, level | Application logs |
| `erp-traces` | dynamic | 30 days | tenant_id, module, service_name | Distributed traces |
| `erp-audit` | strict | 730 days | tenant_id, module, action | Audit trail |
| `erp-search` | dynamic | None | tenant_id, module, entity_type | Cross-module search |
| `erp-metrics` | dynamic | 90 days | tenant_id, module | Long-term metric storage (remote_write) |

### Consequences

**Positive:**
- Single search engine for all observability signals eliminates operational overhead of managing ELK + Jaeger.
- S3-native storage reduces storage costs by 10x compared to EBS-backed Elasticsearch.
- Tag field partition pruning provides efficient multi-tenant query isolation.
- Rust implementation provides memory efficiency and predictable latency.

**Negative:**
- Quickwit ecosystem is smaller than ELK. Fewer third-party integrations and community plugins.
- Team needs to learn Quickwit-specific query syntax (Tantivy query language) alongside PromQL.
- Kibana-equivalent visualization requires Grafana with Quickwit datasource plugin.

---

## ADR-003: DragonflyDB over Redis for Caching {#adr-003}

### Status

**Accepted** -- 2025-08-15

### Context

ERP-Observability requires a caching layer for:
- Search result caching (cross-module search queries are expensive, 200-500ms)
- Log query caching (frequently accessed log patterns)
- Rate limiting (API rate limiting per tenant)
- Session state (SSE connection tracking)

Redis is the de-facto standard for caching in microservice architectures. However, AIDD governance requires evaluation of alternatives that offer improved resource efficiency.

### Decision

We adopted **DragonflyDB** as the caching backend, using the standard `ioredis` client library (DragonflyDB is wire-compatible with Redis).

### Rationale

| Criterion | Redis | DragonflyDB | Decision Factor |
|---|---|---|---|
| Protocol compatibility | Native | Full Redis wire protocol | Drop-in replacement |
| Client library | ioredis | ioredis (same) | Zero code change |
| Memory efficiency | Single-threaded, ~1.5x overhead | Multi-threaded, ~0.8x overhead | 2x memory savings |
| Throughput (ops/sec) | ~100K (single-threaded) | ~1M (multi-threaded) | 10x throughput |
| Persistence | RDB/AOF | RDB/AOF compatible | Both supported |
| License | BSD 3-Clause | BSL 1.1 | Both acceptable for internal use |
| AIDD compliance | Standard | Preferred (efficiency) | DragonflyDB wins |
| Cluster mode | Redis Cluster (complex) | Single-node multi-threaded | Simpler operations |
| Eviction policies | Multiple (LRU, LFU, etc.) | Multiple (Redis-compatible) | Both supported |

**Key AIDD consideration:** AIDD mandates that infrastructure components maximize efficiency per resource unit. DragonflyDB's multi-threaded architecture delivers 10x the throughput of Redis on the same hardware, meaning fewer nodes and lower infrastructure cost.

### Consequences

**Positive:**
- Zero application code changes (wire-compatible with Redis).
- 10x throughput improvement enables single-node deployment for current scale.
- Memory efficiency reduces cache infrastructure cost.

**Negative:**
- DragonflyDB is newer with a smaller community than Redis.
- BSL 1.1 license prevents offering DragonflyDB as a managed service (not relevant for internal use).
- Some Redis modules (RedisGraph, RedisTimeSeries) are not available in DragonflyDB.

---

## ADR-004: YugabyteDB over PostgreSQL for Persistent Storage {#adr-004}

### Status

**Accepted** -- 2025-08-15

### Context

ERP-Observability stores alert rules, alert silences, dashboards, and search index status in a relational database. The initial design used PostgreSQL. As the ERP platform expanded to multi-region deployment, several requirements emerged:

- **Distributed SQL:** Alert rules and dashboards must be available across regions with low-latency reads.
- **Automatic sharding:** With thousands of tenants, the alert_rules table could grow to millions of rows. Manual sharding in PostgreSQL is operationally expensive.
- **High availability:** The observability platform must maintain 99.99% availability. PostgreSQL HA with streaming replication requires manual failover or patroni, adding complexity.
- **AIDD compliance:** AIDD governance prefers distributed databases that provide automatic fault tolerance.

### Decision

We adopted **YugabyteDB** (YSQL mode) as the relational database backend. YugabyteDB provides a PostgreSQL-compatible SQL layer over a distributed, automatically sharded storage engine.

### Rationale

| Criterion | PostgreSQL | YugabyteDB (YSQL) | Decision Factor |
|---|---|---|---|
| SQL compatibility | Native | PostgreSQL wire-compatible | Existing SQL works unchanged |
| Driver compatibility | pg (node-postgres) | pg (same driver, port 5433) | Zero code change |
| Horizontal scaling | Manual sharding | Automatic sharding | Required for multi-region |
| Multi-region | Streaming replication | Synchronous multi-region | Required |
| HA failover | Manual / Patroni | Automatic (Raft consensus) | Reduced operational burden |
| Strong consistency | Single-node ACID | Distributed ACID | Required for alert rules |
| Connection pooling | PgBouncer / internal | Internal (compatible with pg pool) | Both supported |
| JSONB support | Full | Full | Required for panels, labels |
| GIN indexes | Full | Full | Required for tags |
| AIDD compliance | Standard | Preferred (distributed) | YugabyteDB wins |

**Schema compatibility verification:** All SQL statements in `001_initial_schema.sql` execute without modification on YugabyteDB YSQL, including `gen_random_uuid()`, `TIMESTAMPTZ`, `JSONB`, `GIN` indexes, partial indexes, and foreign keys.

### Consequences

**Positive:**
- Zero SQL migration required from PostgreSQL.
- Automatic sharding eliminates manual table partitioning.
- Built-in multi-region replication for disaster recovery.
- Automatic leader election eliminates HA failover complexity.

**Negative:**
- Higher operational complexity than a single PostgreSQL instance.
- YugabyteDB has higher write latency (~5ms vs. ~1ms for PostgreSQL) due to Raft consensus.
- Requires 3+ nodes minimum for HA (3 masters + 3 tablet servers).

---

## ADR-005: OTel Collector as Unified Telemetry Pipeline {#adr-005}

### Status

**Accepted** -- 2025-09-01

### Context

With 20 ERP modules emitting telemetry (logs, metrics, traces), the platform needed a standardized telemetry pipeline. Options considered:

1. **Direct integration:** Each module sends data directly to Quickwit/Prometheus. This creates tight coupling and requires each module to implement retry logic, batching, and connection management.
2. **Multiple agents:** Use Fluent-Bit for logs, Prometheus for metrics, and Jaeger Agent for traces. This requires three agent deployments and three configuration surfaces.
3. **Unified collector:** Use a single telemetry pipeline that receives all signals and routes them to appropriate backends.

### Decision

We adopted the **OpenTelemetry Collector** as the unified telemetry pipeline for traces and logs, with Prometheus continuing as the metrics scraping agent (since Prometheus pull-based scraping is more reliable for metric collection than push-based OTLP).

### Rationale

| Criterion | Direct Integration | Multi-Agent | OTel Collector |
|---|---|---|---|
| Module coupling | Tight (knows backends) | Medium | Loose (OTLP standard) |
| Configuration surface | Per-module | 3 agents | 1 collector |
| Protocol standardization | Backend-specific | Mixed | OTLP (vendor-neutral) |
| Attribute enrichment | Per-module | Per-agent | Centralized processors |
| Retry / backpressure | Per-module | Per-agent | Centralized |
| Tenant ID injection | Per-module | Per-agent | Centralized processor |
| AIDD compliance | Poor (duplication) | Medium | Good (consolidation) |

**OTel Collector configuration:**

- **Receivers:** OTLP gRPC (port 4317), OTLP HTTP (port 4318)
- **Processors:** Batch processor (200ms timeout, 8192 batch size), attribute enrichment (tenant_id, module, environment), memory limiter (80% limit, 85% spike)
- **Exporters:** Quickwit (traces, logs), Prometheus remote_write (metrics if OTLP metrics enabled)

### Consequences

**Positive:**
- All 20 ERP modules emit telemetry via a single OTLP endpoint, simplifying SDKs.
- Centralized tenant_id attribute injection ensures consistent tenant tagging.
- Backend changes (e.g., switching from Quickwit to another trace store) require only collector reconfiguration, not module changes.
- Backpressure handling at the collector level protects backends.

**Negative:**
- OTel Collector is a single point of failure. Mitigated by running as a Deployment with HPA.
- Additional network hop adds 1-2ms latency to telemetry delivery.
- Prometheus pull-based scraping still runs separately for metrics.

---

## ADR-006: Zabbix for Infrastructure Monitoring {#adr-006}

### Status

**Accepted** -- 2026-01-10

### Context

The ERP platform runs on Kubernetes across multiple availability zones. While Prometheus/VictoriaMetrics effectively monitors application-level metrics (HTTP latency, error rates, throughput), several infrastructure monitoring gaps remained:

- **Hardware health:** Prometheus node-exporter provides CPU/memory/disk metrics but lacks deep hardware monitoring (IPMI, disk SMART, RAID status).
- **SNMP monitoring:** Network switches, load balancers, and storage appliances expose metrics via SNMP, which Prometheus handles poorly.
- **Auto-discovery:** New Kubernetes nodes, VMs, and network devices should be automatically discovered and monitored.
- **Agent-based monitoring:** Some monitoring requires an agent on the host (e.g., custom scripts, log file monitoring outside containers).

### Decision

We integrated **Zabbix 7.0 LTS** for infrastructure monitoring, deployed with Zabbix Proxies per availability zone.

### Rationale

| Criterion | Prometheus Only | Zabbix | Decision Factor |
|---|---|---|---|
| SNMP monitoring | Limited (exporter) | Native, mature | Network devices |
| Agent-based monitoring | No | Yes (Zabbix Agent 2) | Host-level scripts |
| Auto-discovery | Kubernetes SD only | Multi-protocol (SNMP, DNS, IPMI) | Network devices |
| IPMI/hardware health | No | Native | Server hardware |
| Alerting | Basic (Alertmanager) | Advanced (escalation, maintenance) | Infrastructure team needs |
| Web monitoring | No | Built-in | External URL checks |
| Maps/topology | No | Built-in | Network visualization |
| Integration with VictoriaMetrics | Native | Via Zabbix-to-Prometheus exporter | Unified metric view |

### Consequences

**Positive:**
- Comprehensive infrastructure visibility including SNMP, IPMI, and hardware health.
- Zabbix Proxy per AZ reduces monitoring latency and provides local buffering.
- Zabbix metrics exported to VictoriaMetrics enables unified dashboarding in Grafana.
- Auto-discovery reduces manual monitoring configuration for infrastructure changes.

**Negative:**
- Additional infrastructure component to operate (Zabbix Server, Zabbix Proxy, Zabbix Agent 2).
- Zabbix has its own alerting system; we must ensure alerts flow to Alertmanager to avoid alert fragmentation.
- Learning curve for Zabbix templates and LLD (Low-Level Discovery) rules.

---

## ADR-007: OpenNMS for Event Correlation {#adr-007}

### Status

**Accepted** -- 2026-01-12

### Context

With metrics in VictoriaMetrics, logs/traces in Quickwit, and infrastructure events in Zabbix, the ERP platform generates thousands of alerts and events per day across 20 modules. Operators faced alert fatigue from correlated but independently firing alerts. For example:

- A YugabyteDB tablet server failure generates alerts from: Zabbix (host down), VictoriaMetrics (DB connection errors for 5+ modules), Quickwit (error log spike), and Alertmanager (SLO violation for dependent modules).
- A network partition generates alerts from every module in the affected availability zone simultaneously.

The platform needed an event correlation engine to deduplicate, correlate, and identify root causes across heterogeneous event sources.

### Decision

We integrated **OpenNMS Horizon** for event correlation, root cause analysis, and topology-aware monitoring.

### Rationale

| Criterion | Manual Correlation | OpenNMS | Decision Factor |
|---|---|---|---|
| Cross-source correlation | Manual operator work | Automated (Drools rules) | Reduces MTTR |
| Topology awareness | None | Service-dependency graph | Root cause analysis |
| Event deduplication | Alertmanager inhibition (basic) | Advanced (time-window, topology) | Reduced alert fatigue |
| Alarm lifecycle | Fire/resolve only | Open/acknowledge/clear/escalate | Operations workflow |
| Integration sources | N/A | SNMP traps, syslog, REST webhooks | Multi-protocol |
| Historical analysis | Alert history in Alertmanager | Event archive with correlation data | Post-incident review |

**OpenNMS integration points:**
1. **Zabbix traps:** Zabbix forwards infrastructure events to OpenNMS via SNMP traps.
2. **Alertmanager webhook:** VictoriaMetrics alerts route through Alertmanager to OpenNMS webhook receiver.
3. **Quickwit anomalies:** Log anomaly detection results posted to OpenNMS REST API.
4. **Topology graph:** OpenNMS maintains a dependency graph of ERP modules and infrastructure components, enabling impact analysis.

### Consequences

**Positive:**
- Reduced alert fatigue through cross-source correlation (estimated 60% reduction in actionable alert volume).
- Root cause identification through topology-aware correlation.
- Unified event lifecycle management with acknowledgment and escalation workflows.

**Negative:**
- OpenNMS is a complex platform with significant operational overhead.
- Correlation rules require ongoing tuning as the ERP architecture evolves.
- Additional Java-based service (memory-intensive).

---

## ADR-008: Refine.dev for Frontend Framework {#adr-008}

### Status

**Accepted** -- 2025-12-01

### Context

The ERP-Observability frontend needed to provide:
- Real-time log streaming with filtering
- Interactive metric charts
- Trace waterfall visualization
- Alert management (CRUD, silence)
- Dashboard builder
- Cross-module search
- Multi-tenant context switching

The ERP platform standardized on React for all frontend modules. The question was which React meta-framework or admin framework to use.

### Decision

We adopted **Refine.dev 4.x** with **Ant Design 5.x** as the frontend framework, consistent with all other ERP modules.

### Rationale

| Criterion | Custom React | Refine.dev + Ant Design | Decision Factor |
|---|---|---|---|
| CRUD scaffolding | Manual | Built-in (useList, useCreate, useUpdate, useDelete) | 70% less boilerplate |
| Data provider | Manual fetch logic | graphql-request + REST data providers | Consistent across ERP |
| Real-time | Manual WebSocket/SSE | useSubscription + graphql-ws | Built-in |
| Table/list views | Manual | Ant Design Table + Refine integration | Production-ready |
| Form handling | Manual | Ant Design Form + Refine hooks | Validation included |
| Authentication | Manual | authProvider interface | Authentik integration |
| Access control | Manual | accessControlProvider | Role-based UI |
| Routing | React Router manual | Refine router provider | Convention-based |
| Theming | Manual | Ant Design theme tokens | ERP design system |
| Platform consistency | N/A | All 20 ERP modules use Refine.dev | Mandatory |
| Developer velocity | Slow (all manual) | Fast (hooks + providers) | 3x faster development |

**Refine.dev + ERP-Observability integration:**

```typescript
// Data providers
- GraphQL (graphql-request) for CRUD operations via Hasura federation
- REST data provider for streaming endpoints (logs/stream)
- graphql-ws for real-time subscriptions (alerts, dashboard updates)

// Auth provider
- Authentik OIDC login flow
- JWT token management
- Role extraction from JWT claims

// Access control
- admin: full CRUD on alert rules, dashboards, reindex
- viewer: read-only access to logs, metrics, traces, dashboards
- operator: read access + alert silence capability
```

### Consequences

**Positive:**
- Consistent developer experience across all 20 ERP modules.
- Built-in CRUD hooks reduce observability-specific frontend code by ~70%.
- Ant Design provides production-ready table, chart, and form components.
- Refine.dev's data provider abstraction simplifies GraphQL + REST hybrid API consumption.

**Negative:**
- Refine.dev's opinionated patterns may not fit all observability visualizations (e.g., trace waterfall requires custom component).
- Ant Design's bundle size is larger than minimal UI libraries.
- Refine.dev version upgrades across 20 modules must be coordinated.

---

## ADR Index

| ADR | Title | Status | Date |
|---|---|---|---|
| ADR-001 | VictoriaMetrics over Prometheus | Accepted | 2026-01-15 |
| ADR-002 | Quickwit over ELK Stack | Accepted | 2025-07-20 |
| ADR-003 | DragonflyDB over Redis | Accepted | 2025-08-15 |
| ADR-004 | YugabyteDB over PostgreSQL | Accepted | 2025-08-15 |
| ADR-005 | OTel Collector as Unified Pipeline | Accepted | 2025-09-01 |
| ADR-006 | Zabbix for Infrastructure Monitoring | Accepted | 2026-01-10 |
| ADR-007 | OpenNMS for Event Correlation | Accepted | 2026-01-12 |
| ADR-008 | Refine.dev for Frontend Framework | Accepted | 2025-12-01 |
