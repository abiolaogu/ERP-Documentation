# ERP-AIOps Release Notes

## Version 1.0.0 - Initial Release

**Release Date:** February 2026
**Classification:** General Availability (GA)
**Platform Compatibility:** OpenSASE ERP Suite v4.0+

---

### Release Overview

ERP-AIOps v1.0.0 is the inaugural release of the AI-powered operations platform for the OpenSASE ERP suite. This release delivers a comprehensive AIOps solution that ingests telemetry from all 20+ ERP modules via OpenTelemetry Collector federation and applies machine learning, large language models, and rule-based analysis to detect anomalies, correlate events, identify root causes, and automate remediation. The platform is built on a Rust core (40+ crates via Axum) for high-throughput event processing, a Python AI brain (FastAPI) for ML inference, and a Go gateway for routing and tenant isolation.

---

### New Features

#### Incident Management

- **Incident Lifecycle Management**: Full lifecycle tracking from detection through correlation, triage, investigation, remediation, and resolution. Each incident maintains a complete audit trail of all state transitions, assignments, and actions taken.
- **Incident Severity Classification**: Automatic severity classification (P1-P5) based on impact scope, affected services, SLA risk, and historical incident patterns. Manual override with audit logging.
- **Incident Deduplication**: Intelligent deduplication engine that merges duplicate alerts into a single incident using fingerprinting based on source, metric, labels, and temporal proximity. Reduces incident volume by up to 70%.
- **Incident Assignment and Escalation**: Rule-based assignment to on-call teams with automatic escalation policies. Integration with ERP-Workspace for real-time notifications via chat and push notifications.
- **Incident Timeline**: Chronological timeline view showing all events, alerts, actions, and state changes associated with an incident. Enables rapid understanding of incident progression.
- **Incident Collaboration**: Multi-user incident collaboration with threaded comments, evidence attachments (screenshots, logs, traces), and real-time status updates.
- **Incident Templates**: Configurable incident templates for common incident types (database outage, network partition, memory leak, etc.) with pre-populated runbooks and checklists.
- **Post-Incident Review**: Structured post-incident review workflow with blameless postmortem templates, action item tracking, and knowledge base integration.

#### Anomaly Detection with Adaptive Thresholds

- **Statistical Anomaly Detection**: Z-score and modified Z-score algorithms for detecting point anomalies in time series metrics. Configurable sensitivity with automatic outlier exclusion during baseline computation.
- **Machine Learning Anomaly Detection**: Isolation Forest models trained per-metric per-tenant for detecting complex, multivariate anomalies that statistical methods miss. Models automatically retrain on a configurable schedule (default: weekly).
- **Deep Learning Anomaly Detection**: LSTM autoencoder models for detecting subtle temporal pattern anomalies in high-dimensional time series data. Particularly effective for detecting gradual degradation and novel failure modes.
- **Adaptive Threshold Engine**: ML-powered threshold engine that learns normal behavior patterns including hourly, daily, weekly, and seasonal cycles. Automatically adjusts detection boundaries as workload patterns evolve. Eliminates the need for manual threshold configuration in most cases.
- **Anomaly Scoring**: Each detected anomaly receives a composite score (0-100) based on deviation magnitude, duration, recurrence pattern, and topological impact. Scores drive prioritization and routing decisions.
- **Anomaly Grouping**: Related anomalies across multiple metrics are automatically grouped into anomaly clusters, reducing noise and surfacing correlated degradation patterns.
- **Baseline Management**: Automatic baseline computation with configurable training windows (7-90 days). Supports manual baseline overrides for planned maintenance windows and known seasonal events.

#### LLM-Powered Root Cause Analysis

- **Automated Root Cause Identification**: LLM-based analysis that examines incident context (correlated events, topology, recent changes, historical patterns) to identify probable root causes. Provides confidence scores and reasoning chains.
- **Causal Inference Engine**: DoWhy-based causal inference that constructs directed acyclic graphs (DAGs) from telemetry data to establish causal relationships between events. Supports counterfactual analysis ("what would have happened if X had not occurred?").
- **Change Correlation**: Automatic correlation of incidents with recent changes (deployments, configuration changes, infrastructure modifications) detected via the ERP module APIs. Highlights changes with high causal probability.
- **Historical Pattern Matching**: Similarity search against historical incidents using embedding-based retrieval to surface past incidents with similar signatures and their resolutions.
- **Natural Language Summaries**: LLM-generated human-readable summaries of incident context, probable root cause, and recommended remediation steps. Summaries are tailored to the recipient's role (SRE, manager, executive).
- **Evidence Collection**: Automatic collection and organization of supporting evidence (relevant logs, metric charts, trace spans, topology snapshots) into a structured RCA report.

#### Event Correlation Engine

- **Temporal Correlation**: Groups events that occur within configurable time windows (default: 5 minutes) into correlation clusters. Uses exponential decay weighting to prioritize temporally proximate events.
- **Topological Correlation**: Leverages the service dependency graph to correlate events across upstream and downstream services. An error in Service A is automatically correlated with degradation in Service B if A depends on B.
- **Label-Based Correlation**: Correlates events sharing common labels (environment, region, cluster, namespace, pod) to identify infrastructure-level issues affecting multiple services.
- **Causal Correlation**: Uses causal inference to establish directional relationships between events, distinguishing root causes from symptoms.
- **Cross-Module Correlation**: Correlates events across different ERP modules (e.g., a database connection pool exhaustion in ERP-Finance correlated with increased latency in ERP-Commerce) via the OTel Collector federation.
- **Noise Reduction**: The correlation engine reduces raw alert volume by 80-90% by grouping related alerts into a single actionable incident with full context.

#### Auto-Remediation Framework

- **Action Catalog**: Library of pre-built remediation actions including pod restart, service scaling, cache flush, connection pool reset, config rollback, DNS failover, and traffic shift.
- **Playbook Engine**: Visual playbook editor for composing multi-step remediation workflows with conditional branching, approval gates, rollback conditions, and notification steps.
- **Execution Engine**: Secure action execution engine with credential management, timeout handling, retry logic, and comprehensive audit logging. Supports SSH, HTTP/REST, Kubernetes API, and custom script execution.
- **Approval Workflows**: Configurable approval policies ranging from fully automatic (for low-risk, well-understood remediation) to multi-level human approval (for high-risk actions). Approval requests are delivered via ERP-Workspace.
- **Rollback Support**: Automatic rollback capability for remediation actions that fail health checks post-execution. Each action captures a pre-execution snapshot for safe rollback.
- **Impact Assessment**: Pre-execution impact assessment that evaluates the blast radius of proposed remediation actions using the service topology graph.

#### Cost Optimization

- **Resource Cost Attribution**: Attributes infrastructure costs to individual ERP modules, tenants, teams, and services based on actual resource consumption (CPU, memory, storage, network).
- **Cost Anomaly Detection**: Detects unexpected cost spikes and gradual cost drift using the same anomaly detection algorithms applied to operational metrics.
- **Right-Sizing Recommendations**: ML-based analysis of resource utilization patterns to recommend optimal resource allocations, identifying over-provisioned and under-provisioned services.
- **Idle Resource Detection**: Identifies unused or underutilized resources (idle databases, orphaned storage volumes, oversized caches) with estimated monthly savings.
- **Cost Forecasting**: Time series forecasting of infrastructure costs based on historical trends, seasonal patterns, and planned growth projections.
- **Budget Alerting**: Configurable budget thresholds with alerts when projected costs exceed defined limits.

#### Security Scanning

- **Vulnerability Assessment**: Continuous scanning of ERP module dependencies, container images, and infrastructure configurations for known vulnerabilities (CVE database).
- **Configuration Drift Detection**: Monitors security-relevant configurations (TLS settings, RBAC policies, network policies, secret management) and alerts on unauthorized changes.
- **Compliance Monitoring**: Automated compliance checks against common frameworks (SOC 2, ISO 27001, HIPAA, PCI DSS) with gap analysis and remediation guidance.
- **Security Posture Scoring**: Composite security score per module and per tenant based on vulnerability count, configuration compliance, patch currency, and access control hygiene.
- **Threat Detection**: Behavioral analysis of API access patterns, authentication events, and data access to detect potential security threats (brute force, privilege escalation, data exfiltration).

#### Topology Mapping

- **Auto-Discovery**: Automatic service dependency discovery from distributed traces, network connections, and service mesh configuration.
- **Dependency Graph**: Interactive service dependency graph with real-time health overlay, showing request flows, latency, and error rates between services.
- **Impact Analysis**: Click-on-service impact analysis showing all upstream consumers and downstream dependencies, with estimated blast radius for outage scenarios.
- **Change Visualization**: Topology diff view showing how the service dependency graph has changed over time, highlighting new dependencies, removed connections, and modified communication patterns.

#### Frontend (React + Refine.dev + Ant Design)

- **AIOps Command Center**: Unified dashboard showing operational health across all ERP modules with real-time incident feed, anomaly alerts, remediation status, and key operational metrics.
- **Purple Theme (#7c3aed)**: Consistent purple-themed UI with dark mode support, following Ant Design's enterprise component library patterns.
- **Real-Time Updates**: WebSocket-based real-time updates for incident status changes, new anomaly detections, and remediation execution progress.
- **Responsive Design**: Fully responsive layout optimized for desktop monitoring stations, laptops, and tablet devices.
- **Role-Based Views**: Customizable dashboards and navigation based on user role (SRE, DevOps, Security Analyst, Cost Analyst, Manager).

---

### API Changes

#### New Endpoints

| Method | Path | Description |
|--------|------|-------------|
| POST | /api/v1/events | Ingest operational events |
| GET | /api/v1/incidents | List incidents with filtering and pagination |
| POST | /api/v1/incidents | Create manual incident |
| GET | /api/v1/incidents/:id | Get incident detail with timeline |
| PUT | /api/v1/incidents/:id | Update incident state |
| POST | /api/v1/incidents/:id/rca | Trigger root cause analysis |
| GET | /api/v1/anomalies | List detected anomalies |
| GET | /api/v1/anomalies/:id | Get anomaly detail with charts |
| GET | /api/v1/rules | List detection and remediation rules |
| POST | /api/v1/rules | Create detection or remediation rule |
| GET | /api/v1/topology | Get service dependency graph |
| POST | /api/v1/remediation/execute | Execute remediation action |
| GET | /api/v1/remediation/history | Get remediation execution history |
| GET | /api/v1/costs/summary | Get cost summary by module/tenant |
| GET | /api/v1/costs/recommendations | Get cost optimization recommendations |
| GET | /api/v1/security/findings | List security findings |
| GET | /api/v1/security/score | Get security posture score |
| GET | /api/v1/forecasts/:metric | Get metric forecast |
| GET | /api/v1/thresholds/:metric | Get adaptive threshold for metric |
| POST | /api/v1/ai/analyze | Submit ad-hoc AI analysis request |

---

### Database Schema

The initial schema includes the following core tables:

- `incidents` - Incident records with severity, status, assignee, and metadata
- `incident_events` - Events associated with incidents (timeline)
- `incident_comments` - Collaboration comments on incidents
- `anomalies` - Detected anomalies with scores and metadata
- `anomaly_baselines` - Computed baselines for adaptive thresholds
- `correlation_rules` - Correlation engine configuration
- `correlation_groups` - Correlated event groups
- `detection_rules` - Static and dynamic detection rules
- `remediation_actions` - Action catalog definitions
- `remediation_playbooks` - Multi-step remediation workflows
- `remediation_executions` - Execution history and audit trail
- `topology_nodes` - Service nodes in the dependency graph
- `topology_edges` - Dependency relationships between services
- `cost_records` - Resource cost attribution records
- `cost_recommendations` - Generated cost optimization recommendations
- `security_findings` - Security scan findings
- `security_configurations` - Monitored security configurations
- `ml_models` - ML model registry and metadata
- `tenant_settings` - Per-tenant configuration overrides

---

### Configuration

#### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| AIOPS_GATEWAY_PORT | 8090 | Go gateway listen port |
| AIOPS_CORE_PORT | 8080 | Rust core API listen port |
| AIOPS_AI_PORT | 8001 | Python AI brain listen port |
| AIOPS_DB_URL | postgresql://... | YugabyteDB connection string |
| AIOPS_CACHE_URL | redis://localhost:6379 | DragonflyDB connection string |
| AIOPS_STORAGE_URL | http://localhost:9000 | RustFS endpoint |
| AIOPS_OTEL_ENDPOINT | http://localhost:4317 | OTel Collector gRPC endpoint |
| AIOPS_LLM_ENDPOINT | https://api.openai.com/v1 | LLM API endpoint for RCA |
| AIOPS_LLM_MODEL | gpt-4 | LLM model for analysis |
| AIOPS_LOG_LEVEL | info | Logging level (debug, info, warn, error) |
| AIOPS_TENANT_HEADER | X-Tenant-ID | Tenant identification header name |

---

### Known Issues

| ID | Severity | Description | Workaround |
|----|----------|-------------|------------|
| AIOPS-001 | Medium | LSTM anomaly detection models require minimum 14 days of training data before producing reliable results | Use statistical detection (Z-score) for new tenants until sufficient data is collected |
| AIOPS-002 | Low | Topology auto-discovery may show stale edges for services that have been decommissioned | Manually remove stale edges via the topology editor |
| AIOPS-003 | Low | Cost attribution for shared infrastructure (databases, caches) uses proportional allocation which may not reflect actual usage in all cases | Configure custom attribution rules for shared resources |
| AIOPS-004 | Medium | LLM-based RCA requires network access to external LLM endpoint; air-gapped deployments need a local LLM | Deploy a local LLM (e.g., Ollama with Llama 3) and configure AIOPS_LLM_ENDPOINT |

---

### Upgrade Notes

This is the initial release. No upgrade path is required.

---

### Deprecation Notices

No deprecations in this release.

---

### Contributors

- AIOps Platform Team
- Observability Team (OTel Collector federation)
- IAM Team (authentication integration)
- Frontend Platform Team (Refine.dev framework)

---

## Version History

| Version | Date | Type | Summary |
|---------|------|------|---------|
| 1.0.0 | February 2026 | GA | Initial release with full AIOps capability set |
