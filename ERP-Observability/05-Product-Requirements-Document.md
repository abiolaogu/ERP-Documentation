# ERP-Observability Product Requirements Document

## 1. Product Vision

ERP-Observability aims to be the definitive unified observability platform for the OpenSASE ERP suite, providing a single pane of glass for metrics, logs, traces, alerts, and infrastructure monitoring across all 20+ ERP modules. The platform replaces fragmented monitoring tools with a cohesive, multi-tenant observability stack that enables SREs to reduce Mean Time to Resolution (MTTR) by 60%, DevOps engineers to proactively detect issues before user impact, and platform administrators to manage observability-as-a-service for all tenant organizations.

## 2. User Personas

### 2.1 Persona Profiles

| Persona | Name | Role | Experience | Primary Goals |
|---------|------|------|-----------|---------------|
| SRE | Amara O. | Site Reliability Engineer | 5+ years in production operations | Maintain SLOs, reduce MTTR, automate incident response |
| DevOps | Tunde K. | DevOps Engineer | 3+ years in CI/CD and infrastructure | Monitor deployments, optimize pipeline health, capacity planning |
| Platform Admin | Chioma N. | Platform Administrator | 7+ years in enterprise IT | Manage tenants, configure retention, control costs |
| Module Dev | Emeka A. | ERP Module Developer | 2+ years building ERP modules | Debug application issues, trace request flows, analyze logs |
| Security Eng | Fatima B. | Security Engineer | 4+ years in security operations | Audit log review, anomaly detection, compliance reporting |

### 2.2 Persona Details

**Amara O. -- Site Reliability Engineer**
- Manages SLOs for 20+ ERP modules across 50+ tenants
- Needs real-time alerting with low false-positive rate
- Requires cross-module correlation to identify cascading failures
- Uses PromQL daily for ad-hoc metric analysis
- Wants automated runbook execution for common incidents

**Tunde K. -- DevOps Engineer**
- Monitors deployment health across staging and production
- Needs infrastructure metrics (CPU, memory, disk, network) at a glance
- Requires log tailing during deployments for rapid feedback
- Wants Zabbix integration for bare-metal and network device monitoring
- Builds custom dashboards for team-specific views

**Chioma N. -- Platform Administrator**
- Provisions observability stacks for new tenants
- Manages retention policies to control storage costs
- Configures notification channels (Slack, email, PagerDuty)
- Reviews usage metrics for capacity planning and billing
- Ensures compliance with data retention regulations

**Emeka A. -- ERP Module Developer**
- Searches logs for specific error messages during debugging
- Traces request flows across microservices to find bottlenecks
- Monitors API latency and error rates after code changes
- Needs simple, self-service access without SRE involvement
- Wants to create custom alerts for application-specific conditions

**Fatima B. -- Security Engineer**
- Reviews audit logs for unauthorized access attempts
- Monitors authentication and authorization events from ERP-IAM
- Detects anomalous patterns in API usage
- Ensures observability data meets compliance retention requirements
- Needs immutable audit trails for regulatory audits

## 3. Competitive Analysis

### 3.1 Feature Comparison

| Feature | Datadog | New Relic | Grafana Cloud | Elastic Stack | ERP-Observability |
|---------|---------|-----------|---------------|---------------|-------------------|
| **Pricing** | $15-34/host/mo | $0.30/GB | $0-299/mo | Self-hosted (free) | Self-hosted (free) |
| **Metrics** | Excellent | Excellent | Excellent | Good | Excellent (VM) |
| **Logs** | Excellent | Good | Good | Excellent | Excellent (Quickwit) |
| **Traces** | Excellent | Excellent | Good | Good | Good (Quickwit) |
| **Alerts** | Excellent | Good | Excellent | Good | Excellent (AM) |
| **Dashboards** | Excellent | Good | Excellent | Good | Excellent (Grafana) |
| **Infrastructure** | Excellent | Good | Basic | Basic | Excellent (Zabbix) |
| **Event Correlation** | Good | Good | Basic | Basic | Excellent (OpenNMS) |
| **Multi-Tenant** | Yes | Yes | Yes | Limited | Yes (native) |
| **Self-Hosted** | No | No | Partial | Yes | **Yes** |
| **OTel Native** | Partial | Partial | Yes | Partial | **Yes** |
| **Data Sovereignty** | Cloud only | Cloud only | Cloud option | Self-hosted | **Full control** |
| **Open Source** | No | No | Partial | Elastic License | **Yes (Apache 2.0)** |

### 3.2 Competitive Advantages

1. **Self-hosted data sovereignty**: Unlike Datadog and New Relic, all observability data stays within the organization's infrastructure.
2. **Unified multi-tenant**: Native tenant isolation at every layer, unlike bolted-on multi-tenancy in Elastic Stack.
3. **AIDD-compliant stack**: Purpose-built with VictoriaMetrics, Quickwit, and DragonflyDB for optimal price/performance.
4. **Infrastructure depth**: Zabbix + OpenNMS provide enterprise-grade infrastructure monitoring that cloud observability platforms lack.
5. **No per-host/per-GB pricing**: Eliminates the cost explosion that comes with Datadog/New Relic as infrastructure grows.
6. **ERP-native**: Pre-built dashboards and alerts tailored for each ERP module, not generic templates.

## 4. Functional Requirements

### 4.1 Metrics Collection & Query

| ID | Requirement | Priority | Status |
|----|------------|----------|--------|
| FR-MET-001 | Ingest metrics via OTel Collector OTLP receiver | P0 | Done |
| FR-MET-002 | Store metrics in VictoriaMetrics with tenant isolation | P0 | Done |
| FR-MET-003 | PromQL query API with X-Scope-OrgID tenant scoping | P0 | Done |
| FR-MET-004 | Metric series metadata (label names, label values) | P0 | Done |
| FR-MET-005 | Configurable retention per tenant (7d to 5y) | P0 | Done |
| FR-MET-006 | Downsampling for long-range queries | P1 | Done |
| FR-MET-007 | Recording rules for pre-computed aggregations | P1 | Done |
| FR-MET-008 | Metric explorer UI with autocomplete | P0 | Done |
| FR-MET-009 | Prometheus scrape endpoint support (legacy) | P1 | Done |
| FR-MET-010 | Metric cardinality analysis and limits | P2 | Planned |

### 4.2 Log Management

| ID | Requirement | Priority | Status |
|----|------------|----------|--------|
| FR-LOG-001 | Ingest logs via OTel Collector OTLP exporter | P0 | Done |
| FR-LOG-002 | Store logs in Quickwit with per-tenant indexes | P0 | Done |
| FR-LOG-003 | Full-text log search with structured field filtering | P0 | Done |
| FR-LOG-004 | Real-time log tailing via WebSocket | P0 | Done |
| FR-LOG-005 | Log aggregations (count, histogram, percentiles) | P1 | Done |
| FR-LOG-006 | Configurable retention per tenant (30d to 2y) | P0 | Done |
| FR-LOG-007 | Log-to-trace correlation via trace_id field | P0 | Done |
| FR-LOG-008 | Log pattern detection and grouping | P2 | Planned |
| FR-LOG-009 | Log-based alerting via Quickwit search queries | P1 | Done |
| FR-LOG-010 | Long-term archival to RustFS S3 storage | P1 | Done |

### 4.3 Distributed Tracing

| ID | Requirement | Priority | Status |
|----|------------|----------|--------|
| FR-TRC-001 | Ingest traces via OTel Collector OTLP exporter | P0 | Done |
| FR-TRC-002 | Store traces in Quickwit with per-tenant indexes | P0 | Done |
| FR-TRC-003 | Trace search by service, operation, duration, status | P0 | Done |
| FR-TRC-004 | Trace waterfall visualization | P0 | Done |
| FR-TRC-005 | Service map generation from trace data | P1 | Done |
| FR-TRC-006 | Trace-to-log correlation via span_id field | P0 | Done |
| FR-TRC-007 | Tail sampling for error-biased trace collection | P1 | Done |
| FR-TRC-008 | Latency analysis (p50, p95, p99) per service/operation | P0 | Done |
| FR-TRC-009 | Error propagation tracking across services | P1 | Done |
| FR-TRC-010 | Configurable retention per tenant (3d to 30d) | P0 | Done |

### 4.4 Alerting

| ID | Requirement | Priority | Status |
|----|------------|----------|--------|
| FR-ALT-001 | PromQL-based alert rule creation and evaluation | P0 | Done |
| FR-ALT-002 | Alert routing with grouping, deduplication, silencing | P0 | Done |
| FR-ALT-003 | Multi-channel notifications (email, Slack, webhook, PD, OG) | P0 | Done |
| FR-ALT-004 | Alert history with timeline view | P0 | Done |
| FR-ALT-005 | Silence management for maintenance windows | P0 | Done |
| FR-ALT-006 | Escalation policies with configurable timeouts | P1 | Done |
| FR-ALT-007 | Alert correlation with infrastructure events | P1 | Done |
| FR-ALT-008 | Alert rule templates for common ERP patterns | P1 | Done |
| FR-ALT-009 | Per-tenant alert rule management | P0 | Done |
| FR-ALT-010 | SLO-based alerting (burn rate, error budget) | P2 | Planned |

### 4.5 Infrastructure Monitoring

| ID | Requirement | Priority | Status |
|----|------------|----------|--------|
| FR-INF-001 | Agent-based host monitoring (Zabbix) | P0 | Done |
| FR-INF-002 | SNMP network device monitoring | P1 | Done |
| FR-INF-003 | Auto-discovery of new hosts and services | P1 | Done |
| FR-INF-004 | Host health dashboard with CPU, memory, disk, network | P0 | Done |
| FR-INF-005 | Trigger-based alerting to Alertmanager | P0 | Done |
| FR-INF-006 | Event correlation via OpenNMS | P1 | Done |
| FR-INF-007 | Network topology visualization | P2 | Done |
| FR-INF-008 | Service availability polling (OpenNMS) | P1 | Done |
| FR-INF-009 | Zabbix proxy for distributed monitoring | P2 | Planned |
| FR-INF-010 | IPMI monitoring for bare-metal servers | P2 | Planned |

### 4.6 Dashboards & Visualization

| ID | Requirement | Priority | Status |
|----|------------|----------|--------|
| FR-DSH-001 | Pre-built dashboard templates for all ERP modules | P0 | Done |
| FR-DSH-002 | Custom dashboard creation via Grafana | P0 | Done |
| FR-DSH-003 | Dashboard-as-code provisioning | P1 | Done |
| FR-DSH-004 | Grafana org-per-tenant isolation | P0 | Done |
| FR-DSH-005 | Embedded Grafana panels in React frontend | P0 | Done |
| FR-DSH-006 | Annotation support for deployments and incidents | P1 | Done |
| FR-DSH-007 | Dashboard sharing with public links | P2 | Planned |
| FR-DSH-008 | PDF report generation from dashboards | P2 | Planned |

### 4.7 Tenant Management

| ID | Requirement | Priority | Status |
|----|------------|----------|--------|
| FR-TNT-001 | Tenant CRUD operations | P0 | Done |
| FR-TNT-002 | Automated tenant provisioning (Grafana org, VM namespace, QW index, Zabbix group) | P0 | Done |
| FR-TNT-003 | Per-tenant retention configuration | P0 | Done |
| FR-TNT-004 | Per-tenant usage metering (metrics count, log volume) | P1 | Done |
| FR-TNT-005 | Tenant decommissioning with data cleanup | P1 | Done |
| FR-TNT-006 | RBAC per tenant (Viewer, Editor, Admin) | P0 | Done |

## 5. Non-Functional Requirements

| ID | Requirement | Target |
|----|------------|--------|
| NFR-001 | Metric ingestion throughput | 1M+ data points/sec |
| NFR-002 | Log ingestion throughput | 500K+ lines/sec |
| NFR-003 | PromQL query latency (1h range, p99) | < 100ms |
| NFR-004 | Log search latency (p99) | < 200ms |
| NFR-005 | Trace lookup latency (p99) | < 50ms |
| NFR-006 | Alert evaluation interval | < 10s |
| NFR-007 | Dashboard load time | < 2s |
| NFR-008 | Availability | 99.95% uptime |
| NFR-009 | Storage efficiency (metrics) | 10x compression ratio |
| NFR-010 | Storage efficiency (logs) | 80% reduction vs. ELK |
| NFR-011 | Concurrent users | 1,000+ simultaneous |
| NFR-012 | Metric retention | Up to 5 years |
| NFR-013 | Log retention | Up to 2 years |
| NFR-014 | Browser support | Chrome, Firefox, Safari, Edge (latest 2 versions) |
| NFR-015 | Compliance | SOC2, HIPAA, PCI-DSS, GDPR |

## 6. Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Mean Time to Resolution (MTTR) | 60% reduction from baseline | Incident management data |
| False positive alert rate | < 5% | Alert review process |
| Dashboard load time p95 | < 2 seconds | Frontend performance metrics |
| Query latency (PromQL) p99 | < 100ms | VictoriaMetrics internal metrics |
| Log search p99 | < 200ms | Quickwit internal metrics |
| Tenant onboarding time | < 5 minutes (automated) | Provisioning metrics |
| Module coverage | 100% of ERP modules instrumented | OTel SDK integration tracking |
| Documentation completeness | 15/15 docs (Phase 1) | This document set |
