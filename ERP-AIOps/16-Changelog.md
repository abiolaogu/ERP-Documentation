# ERP-AIOps Changelog

> **Document ID:** ERP-AIOPS-CL-016
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md)

---

## Overview

This document provides the complete version history for ERP-AIOps, the AI-powered operations platform for the ERP ecosystem. All notable changes, including new features, enhancements, bug fixes, deprecations, and breaking changes, are documented here following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) principles and [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## Version Summary

| Version | Release Date | Milestone |
|---------|-------------|-----------|
| v1.0.0  | 2026-02-15  | Full AIOps Platform GA |
| v0.12.0 | 2026-02-01  | Security Scanning & Compliance |
| v0.11.0 | 2026-01-18  | Cost Optimization Engine |
| v0.10.0 | 2026-01-05  | Auto-Remediation Framework |
| v0.9.0  | 2025-12-20  | Forecasting & Capacity Planning |
| v0.8.0  | 2025-12-05  | Topology Mapping & Visualization |
| v0.7.0  | 2025-11-20  | Event Correlation Engine |
| v0.6.0  | 2025-11-05  | LLM-Powered Root Cause Analysis |
| v0.5.0  | 2025-10-20  | Adaptive Anomaly Detection |
| v0.4.0  | 2025-10-05  | Multi-Tenant Isolation & RBAC |
| v0.3.0  | 2025-09-20  | AI Brain (Python FastAPI) Integration |
| v0.2.0  | 2025-09-05  | Go Gateway & API Routing |
| v0.1.1  | 2025-08-25  | Bug Fixes & Stability |
| v0.1.0  | 2025-08-10  | Core Incident Management |

---

## [v1.0.0] - 2026-02-15 -- Full AIOps Platform GA

### Highlights

The 1.0.0 release marks the General Availability of ERP-AIOps as a fully integrated, production-ready AI-powered operations platform. All subsystems -- incident management, anomaly detection, root cause analysis, event correlation, auto-remediation, cost optimization, security scanning, topology mapping, and forecasting -- are unified under a single, multi-tenant, enterprise-grade platform.

### Added

- **Unified AIOps Dashboard**: Single-pane-of-glass dashboard aggregating incident status, anomaly counts, cost trends, security posture, and topology health into a real-time operational view.
- **Cross-Module Correlation**: AI-driven correlation engine that links incidents, anomalies, and security findings across all ERP modules (CRM, HCM, Finance, SCM, etc.) to provide holistic operational intelligence.
- **Model Registry & Versioning**: Centralized ML model registry with version tracking, A/B testing support, rollback capabilities, and model performance monitoring.
- **SLA Management**: SLA definition, tracking, and breach alerting with customizable thresholds per tenant and service tier.
- **Runbook Automation Library**: Pre-built library of 50+ runbooks covering common operational scenarios across all ERP modules.
- **Executive Reporting**: Automated weekly and monthly executive reports with KPIs including MTTD (Mean Time to Detect), MTTR (Mean Time to Resolve), availability percentages, cost savings, and security posture scores.
- **Webhook Delivery System**: Reliable webhook delivery with configurable retry policies, dead-letter queues, and delivery status tracking (see [23-Webhook-Specifications.md](./23-Webhook-Specifications.md)).
- **Grafana Dashboard Templates**: Pre-built Grafana dashboards for all AIOps subsystems with drill-down capabilities.
- **OpenTelemetry Integration**: Full OpenTelemetry support for traces, metrics, and logs across Rust, Python, and Go components.
- **Multi-Region Support**: Cross-region deployment with data sovereignty controls and region-aware routing.

### Changed

- **API Versioning**: All API endpoints now use `/api/v1/` prefix consistently across Rust API, AI Brain, and Go Gateway.
- **Error Response Format**: Standardized error responses across all services to RFC 7807 (Problem Details for HTTP APIs) format.
- **Authentication Flow**: Migrated from per-service JWT validation to centralized Authentik-based authentication with token introspection caching in DragonflyDB.
- **Database Migrations**: Consolidated all migration scripts into a single ordered migration pipeline with rollback support.
- **Frontend Theme**: Finalized purple (#7c3aed) design system with dark mode support and accessibility compliance (WCAG 2.1 AA).

### Fixed

- **Memory Leak in Correlation Engine**: Resolved a memory leak in the Rust correlation engine caused by unbounded event buffers during high-throughput periods. Events are now processed with bounded channels and backpressure signaling.
- **AI Brain Cold Start**: Reduced AI Brain cold start time from 45 seconds to 8 seconds by implementing lazy model loading and pre-warming critical inference paths.
- **Topology Stale Data**: Fixed race condition in topology node updates that could cause stale dependency graphs when multiple services reported status simultaneously.
- **Cost Report Timezone**: Corrected timezone handling in cost reports that was causing 1-day offset in daily cost aggregations for tenants in negative UTC offsets.

### Security

- **CVE-2026-0142**: Updated `rustls` to 0.23.x to address a potential denial-of-service via malformed TLS ClientHello.
- **CVE-2026-0198**: Patched `pydantic` dependency in AI Brain to address arbitrary code execution via crafted model validation.
- **Audit Log Integrity**: Added cryptographic chaining (SHA-256 hash chains) to audit log entries to ensure tamper-evidence.

### Performance

- **Anomaly Detection Latency**: P99 anomaly detection latency reduced from 120ms to 35ms through SIMD-accelerated statistical computations in Rust.
- **Incident Query Performance**: Incident list queries improved 4x via composite indexes on `(tenant_id, status, severity, created_at)`.
- **AI Brain Throughput**: AI Brain inference throughput increased from 200 req/s to 850 req/s through batched inference and ONNX Runtime integration.

---

## [v0.12.0] - 2026-02-01 -- Security Scanning & Compliance

### Added

- **Security Scanning Engine**: Rust-based security scanning crate (`aiops-security`) that analyzes infrastructure configurations, API patterns, and access logs for security vulnerabilities.
- **Vulnerability Assessment**: Automated vulnerability scanning of container images, dependencies, and runtime configurations with CVE database integration.
- **Compliance Frameworks**: Built-in compliance checks for SOC 2 Type II, ISO 27001, HIPAA, and PCI-DSS with automated evidence collection.
- **Security Findings API**: New REST endpoints (`/api/v1/security/findings`, `/api/v1/security/scans`) for managing security scan results.
- **Security Score**: Tenant-level security posture score (0-100) calculated from weighted findings across severity levels.
- **Integration with Incident Management**: Critical and high-severity security findings automatically create incidents with `security` classification.
- **SBOM Generation**: Software Bill of Materials generation for all deployed components in CycloneDX and SPDX formats.

### Changed

- **RBAC Permissions**: Added `security:read`, `security:write`, `security:scan` permissions to the RBAC model.
- **Dashboard**: Added Security Posture widget to the main AIOps dashboard with trend visualization.

### Fixed

- **False Positive Reduction**: Tuned security scanning rules to reduce false positives by 40% through contextual analysis of infrastructure configurations.

---

## [v0.11.0] - 2026-01-18 -- Cost Optimization Engine

### Added

- **Cost Optimization Crate**: New Rust crate (`aiops-cost`) providing cost data ingestion, analysis, and optimization recommendation engine.
- **Cost Anomaly Detection**: ML-powered detection of unusual spending patterns using isolation forests trained on historical cost data.
- **Rightsizing Recommendations**: AI-driven compute and storage rightsizing recommendations based on actual resource utilization patterns.
- **Reserved Instance Analysis**: Analysis of on-demand vs. reserved instance usage with break-even calculations and savings projections.
- **Cost Allocation Tags**: Hierarchical cost tagging system supporting department, project, environment, and custom dimensions.
- **Budget Alerts**: Configurable budget thresholds with progressive alerts at 50%, 75%, 90%, and 100% of budget.
- **Cost Reports API**: Endpoints for generating and retrieving cost reports (`/api/v1/cost/reports`, `/api/v1/cost/recommendations`).
- **Forecasted Spend**: 30/60/90-day cost forecasting using ARIMA and Prophet models in the AI Brain.
- **Savings Dashboard**: Frontend dashboard showing realized savings, pending recommendations, and cost trends.

### Changed

- **AI Brain**: Added `/analyze/cost` and `/forecast/cost` endpoints for cost-specific ML analysis.
- **Database Schema**: Added `cost_reports` table with partitioning by `tenant_id` and `report_month`.

### Fixed

- **Data Ingestion Pipeline**: Resolved data loss during cost data ingestion when upstream cost APIs returned paginated results exceeding 10,000 records.

---

## [v0.10.0] - 2026-01-05 -- Auto-Remediation Framework

### Added

- **Remediation Engine**: Rust-based auto-remediation crate (`aiops-remediation`) providing a safe, auditable framework for automated incident response.
- **Action Library**: 30+ pre-built remediation actions including service restart, scaling, traffic shifting, cache flush, DNS failover, and certificate renewal.
- **Remediation Policies**: Policy engine for defining conditions under which remediation actions can execute automatically vs. requiring human approval.
- **Approval Workflows**: Configurable approval chains for high-impact remediation actions with Slack/Teams integration.
- **Dry Run Mode**: All remediation actions support dry-run execution that simulates the action and reports expected outcomes without making changes.
- **Blast Radius Analysis**: AI-powered blast radius estimation that predicts the impact of remediation actions on dependent services using the topology graph.
- **Remediation History**: Complete audit trail of all remediation actions with before/after state snapshots and rollback support.
- **Cooldown Periods**: Configurable cooldown periods to prevent remediation loops (e.g., repeated restarts of a fundamentally broken service).
- **Remediation API**: New endpoints (`/api/v1/remediation/actions`, `/api/v1/remediation/policies`, `/api/v1/remediation/execute`).

### Changed

- **Incident Workflow**: Incidents now include a `remediation_status` field tracking whether automated remediation was attempted, succeeded, or requires escalation.
- **Webhook Events**: Added `remediation.executed`, `remediation.failed`, `remediation.approval_required` webhook events.

### Security

- **Least Privilege Execution**: Remediation actions execute with scoped credentials using short-lived tokens with minimum required permissions.
- **Action Signing**: All remediation action definitions are cryptographically signed to prevent tampering.

---

## [v0.9.0] - 2025-12-20 -- Forecasting & Capacity Planning

### Added

- **Forecasting Crate**: New Rust crate (`aiops-forecasting`) for time-series forecasting of metrics, capacity, and workload patterns.
- **AI Brain Forecasting Models**: Prophet, ARIMA, and LSTM-based forecasting models in the Python AI Brain for multi-horizon predictions.
- **Capacity Planning**: Automated capacity planning that predicts when resources will be exhausted based on growth trends and seasonal patterns.
- **What-If Analysis**: Simulation engine for modeling the impact of infrastructure changes on capacity and performance.
- **Forecast API**: Endpoints for generating and retrieving forecasts (`/api/v1/forecasting/predict`, `/api/v1/forecasting/capacity`).
- **AI Brain `/forecast` Endpoint**: New FastAPI endpoint for submitting time-series data and receiving multi-step predictions with confidence intervals.
- **Seasonality Detection**: Automatic detection of daily, weekly, and monthly seasonal patterns in operational metrics.
- **Anomaly-Aware Forecasting**: Forecasting models automatically exclude anomalous data points to prevent forecast contamination.

### Changed

- **Dashboard**: Added Forecasting & Capacity widget with interactive time-horizon selector and confidence band visualization.
- **AI Brain Dependencies**: Added `prophet`, `statsmodels`, and `torch` to AI Brain Python dependencies.

---

## [v0.8.0] - 2025-12-05 -- Topology Mapping & Visualization

### Added

- **Topology Engine**: Rust crate (`aiops-topology`) for building and maintaining real-time service dependency graphs.
- **Auto-Discovery**: Automatic service discovery via OpenTelemetry trace analysis, network flow logs, and API gateway access patterns.
- **Dependency Mapping**: Directed graph representation of service dependencies with edge weights indicating call frequency, latency, and error rates.
- **Impact Analysis**: Graph traversal algorithms (BFS/DFS) for determining the blast radius of a service failure by walking the dependency tree.
- **Topology Visualization**: Interactive D3.js-based topology visualization in the frontend with zoom, filter, and drill-down capabilities.
- **Health Propagation**: Service health status propagation through the dependency graph -- a degraded upstream service automatically flags all downstream dependents.
- **Topology API**: Endpoints for querying topology (`/api/v1/topology/nodes`, `/api/v1/topology/edges`, `/api/v1/topology/impact/{node_id}`).
- **Change Detection**: Automatic detection of topology changes (new services, removed dependencies, changed communication patterns) with alerting.

### Changed

- **Correlation Engine**: Event correlation now uses topology information to improve correlation accuracy by considering service dependencies.
- **Database Schema**: Added `topology_nodes` and `topology_edges` tables with graph-optimized indexing.

---

## [v0.7.0] - 2025-11-20 -- Event Correlation Engine

### Added

- **Correlation Engine**: Rust crate (`aiops-correlation`) implementing multi-strategy event correlation to reduce alert noise and identify related events.
- **Temporal Correlation**: Time-window based correlation that groups events occurring within configurable time windows (default: 5 minutes).
- **Topological Correlation**: Correlation based on service dependency topology -- events from services in the same dependency chain are grouped together.
- **Causal Correlation**: AI-powered causal analysis using Granger causality tests and transfer entropy to identify cause-effect relationships between event streams.
- **Alert Deduplication**: Intelligent alert deduplication that reduces alert volume by 60-80% by identifying and merging duplicate alerts.
- **Correlation Rules**: Rule-based correlation engine supporting user-defined correlation patterns with CEP (Complex Event Processing) syntax.
- **Correlation Groups**: Correlated events are grouped into correlation groups with a primary event, contributing events, and a computed root cause hypothesis.
- **AI Brain `/correlate` Endpoint**: New FastAPI endpoint for ML-based event correlation using graph neural networks.
- **Correlation API**: Endpoints for managing correlation rules and viewing correlation groups (`/api/v1/correlation/rules`, `/api/v1/correlation/groups`).

### Changed

- **Incident Creation**: Correlated event groups can optionally auto-create incidents, reducing mean time to incident creation.
- **Anomaly Pipeline**: Anomaly events are now fed into the correlation engine before generating alerts, reducing noise.

### Performance

- **Streaming Correlation**: Implemented streaming correlation using Rust async channels, enabling real-time correlation with sub-millisecond latency for individual events.
- **Batch Processing**: Added batch correlation mode for historical event analysis processing up to 100,000 events/second.

---

## [v0.6.0] - 2025-11-05 -- LLM-Powered Root Cause Analysis

### Added

- **RCA Engine**: Rust crate (`aiops-rca`) orchestrating LLM-powered root cause analysis for incidents and anomaly clusters.
- **LLM Integration**: Integration with OpenAI GPT-4, Anthropic Claude, and self-hosted LLaMA models for generating natural-language root cause hypotheses.
- **Context Assembly**: Automated assembly of RCA context including incident timeline, related anomalies, topology information, recent deployments, configuration changes, and historical similar incidents.
- **Hypothesis Ranking**: Generated root cause hypotheses are ranked by confidence score using a combination of LLM confidence and historical accuracy data.
- **Evidence Linking**: Each hypothesis is linked to supporting evidence (log entries, metric anomalies, trace spans) with relevance scores.
- **RCA Reports**: Automated generation of structured RCA reports in Markdown format suitable for post-incident reviews.
- **AI Brain `/analyze/rca` Endpoint**: New FastAPI endpoint accepting incident context and returning ranked root cause hypotheses.
- **Similar Incident Search**: Vector similarity search (using embeddings) to find historically similar incidents and their resolutions.
- **Feedback Loop**: Operators can confirm or reject RCA hypotheses, and this feedback is used to improve future RCA accuracy.

### Changed

- **Incident Detail View**: Incident detail page now includes an "AI Root Cause Analysis" tab displaying hypotheses, evidence, and confidence scores.
- **AI Brain**: Added `langchain`, `openai`, `anthropic`, and `sentence-transformers` to Python dependencies.

### Security

- **Prompt Injection Prevention**: Implemented input sanitization and output validation to prevent prompt injection attacks in LLM-powered RCA.
- **Data Redaction**: PII and sensitive data (credentials, tokens, IP addresses) are automatically redacted before being sent to external LLM providers.

---

## [v0.5.0] - 2025-10-20 -- Adaptive Anomaly Detection

### Added

- **Anomaly Detection Engine**: Rust crate (`aiops-anomaly`) implementing multi-algorithm anomaly detection for time-series metrics.
- **Adaptive Thresholds**: Dynamic threshold computation using exponential moving averages, seasonal decomposition, and percentile-based bounds that automatically adapt to changing baselines.
- **Statistical Methods**: Z-score, Modified Z-score (MAD-based), IQR, Grubbs' test, and DBSCAN-based anomaly detection algorithms implemented in Rust for high performance.
- **ML-Based Detection**: Integration with AI Brain for ML-powered anomaly detection using Isolation Forest, Autoencoders, and LSTM-based sequence models.
- **Multi-Metric Correlation**: Detection of anomalies across correlated metrics (e.g., CPU spike + latency increase + error rate increase detected as a single composite anomaly).
- **Seasonality Awareness**: Anomaly detection automatically adjusts for daily, weekly, and monthly seasonal patterns to reduce false positives.
- **Anomaly Scoring**: Each anomaly is scored (0.0-1.0) based on deviation magnitude, duration, and impact on dependent services.
- **Anomaly API**: Full CRUD endpoints for anomaly management (`/api/v1/anomalies`, `/api/v1/anomalies/{id}`, `/api/v1/anomalies/{id}/acknowledge`).
- **AI Brain `/analyze/anomaly` Endpoint**: FastAPI endpoint for ML-based anomaly detection and classification.
- **Suppression Rules**: Anomaly suppression rules for known maintenance windows and expected deviations.

### Changed

- **AIOps Rules Engine**: The rules engine now supports anomaly-based trigger conditions in addition to threshold-based triggers.
- **Dashboard**: Added Anomaly Detection widget with real-time anomaly stream and historical anomaly heatmap.

---

## [v0.4.0] - 2025-10-05 -- Multi-Tenant Isolation & RBAC

### Added

- **Multi-Tenant Isolation**: All database tables, API endpoints, and AI Brain operations are scoped by `tenant_id TEXT` to ensure strict data isolation between tenants.
- **Row-Level Security (RLS)**: YugabyteDB RLS policies enforcing tenant isolation at the database level, preventing cross-tenant data access even in the event of application-level bugs.
- **RBAC Integration**: Integration with Authentik for role-based access control with the following roles: `aiops_admin`, `aiops_operator`, `aiops_viewer`, `aiops_analyst`.
- **Permission Model**: Fine-grained permissions for each AIOps capability (e.g., `incidents:write`, `anomalies:acknowledge`, `remediation:execute`, `rules:manage`).
- **Tenant Provisioning**: Automated tenant onboarding that creates tenant-specific database partitions, initializes default AIOps rules, and configures default anomaly detection thresholds.
- **API Key Management**: Tenant-scoped API key generation and management for service-to-service authentication.
- **Audit Logging**: All AIOps operations are audit-logged with tenant context, user identity, action type, and before/after state.

### Changed

- **Database Schema**: All tables updated to include `tenant_id TEXT NOT NULL` column with composite primary keys and indexes.
- **API Authentication**: All API endpoints now require valid JWT tokens with tenant claims.
- **AI Brain**: AI Brain endpoints now accept and validate `tenant_id` in request headers.

### Security

- **Tenant Boundary Testing**: Added comprehensive integration tests verifying that no API endpoint leaks data across tenant boundaries.

---

## [v0.3.0] - 2025-09-20 -- AI Brain (Python FastAPI) Integration

### Added

- **AI Brain Service**: Python FastAPI service (`ai-brain/`) providing ML inference capabilities for the AIOps platform.
- **FastAPI Application**: Production-grade FastAPI application with automatic OpenAPI documentation, request validation via Pydantic, and async request handling.
- **Health Check Endpoints**: `/health` and `/ready` endpoints for Kubernetes liveness and readiness probes.
- **ML Model Serving**: Framework for loading, serving, and hot-reloading ML models using ONNX Runtime and scikit-learn.
- **Analysis Endpoints**: Initial `/analyze/metrics` and `/analyze/logs` endpoints for metric and log analysis.
- **gRPC Interface**: gRPC interface between the Rust API and Python AI Brain for low-latency, high-throughput ML inference calls.
- **Model Storage**: Integration with RustFS (S3-compatible) for model artifact storage and versioning.
- **AI Brain Docker Image**: Multi-stage Docker image with CUDA support for GPU-accelerated inference.

### Changed

- **Rust API**: Added `aiops-ai-client` crate for communicating with the AI Brain service via gRPC and REST.
- **Docker Compose**: Added AI Brain service to the development Docker Compose configuration.

### Performance

- **Connection Pooling**: Implemented connection pooling for Rust-to-AI-Brain gRPC connections with configurable pool size and idle timeout.
- **Request Batching**: AI Brain supports request batching for bulk metric analysis, processing up to 1,000 metrics per batch.

---

## [v0.2.0] - 2025-09-05 -- Go Gateway & API Routing

### Added

- **Go API Gateway**: High-performance Go gateway service (`gateway/`) providing unified API routing, load balancing, and protocol translation.
- **Route Configuration**: Declarative route configuration mapping external API paths to internal Rust API and AI Brain endpoints.
- **Rate Limiting**: Per-tenant rate limiting with configurable burst and sustained rate limits using DragonflyDB as the backing store.
- **Request/Response Logging**: Structured request/response logging with configurable log levels and sampling rates.
- **Circuit Breaker**: Circuit breaker pattern implementation for Rust API and AI Brain backends with configurable failure thresholds and recovery timeouts.
- **Health Aggregation**: Gateway aggregates health status from all backend services and exposes a unified health endpoint.
- **CORS Configuration**: Configurable CORS policies for frontend access with support for per-tenant origin whitelisting.
- **Prometheus Metrics**: Gateway exposes Prometheus metrics for request rate, latency, error rate, and circuit breaker state.

### Changed

- **Frontend**: Frontend proxy configuration updated to route API calls through the Go Gateway.
- **Docker Compose**: Gateway added as the external-facing entry point, replacing direct Rust API exposure.

---

## [v0.1.1] - 2025-08-25 -- Bug Fixes & Stability

### Fixed

- **Incident Pagination**: Fixed off-by-one error in incident list pagination that caused the last item on each page to be duplicated on the next page.
- **Severity Validation**: Added server-side validation for incident severity values, rejecting invalid severity levels instead of defaulting to `info`.
- **Connection Pool Exhaustion**: Fixed connection pool exhaustion under high concurrency by implementing connection timeout and retry logic for YugabyteDB connections.
- **Timestamp Serialization**: Fixed inconsistent timestamp serialization between Rust API (RFC 3339) and frontend (ISO 8601) by standardizing on RFC 3339 with millisecond precision.

### Changed

- **Error Messages**: Improved error messages for 400 Bad Request responses to include specific field validation errors.
- **Logging**: Migrated from `println!` statements to structured logging using `tracing` crate with JSON output format.

---

## [v0.1.0] - 2025-08-10 -- Core Incident Management

### Highlights

Initial release of ERP-AIOps providing core incident management capabilities. This release establishes the foundational Rust workspace architecture, database schema, API layer, and frontend application.

### Added

- **Rust Workspace**: Established Cargo workspace with initial crates: `aiops-core`, `aiops-api`, `aiops-db`, `aiops-models`, `aiops-config`.
- **Axum API Server**: HTTP API server built on Axum with Tower middleware for logging, CORS, and request tracing.
- **Incident Management**: Complete CRUD operations for incident lifecycle management including creation, updates, status transitions, assignment, and resolution.
- **Incident Model**: Incident data model with fields: `id`, `tenant_id`, `title`, `description`, `severity` (critical/high/medium/low/info), `status` (open/investigating/identified/monitoring/resolved/closed), `source`, `assigned_to`, `tags`, `created_at`, `updated_at`, `resolved_at`.
- **YugabyteDB Integration**: Database layer using `sqlx` with compile-time query verification, connection pooling, and migration support.
- **DragonflyDB Caching**: Redis-compatible caching layer using DragonflyDB for incident metadata caching and session management.
- **Migration System**: SQL migration framework with versioned migration files and rollback support.
- **Initial Schema**: `incidents` table with appropriate indexes, constraints, and partition-readiness.
- **Refine.dev Frontend**: React-based frontend using Refine.dev framework with Ant Design components, featuring incident list, detail, create, and edit views.
- **Purple Theme**: Custom Ant Design theme with primary color `#7c3aed` (purple) for the AIOps module identity.
- **Docker Compose**: Development Docker Compose configuration for YugabyteDB, DragonflyDB, RustFS, Rust API, and frontend.
- **Configuration Management**: Environment-based configuration using `config` crate with support for TOML files and environment variable overrides.
- **Health Checks**: `/healthz` and `/readyz` endpoints for container orchestration integration.

### Technical Decisions

- **Rust over Go for Core**: Chose Rust for the core platform for memory safety without garbage collection, predictable latency, and high throughput (see [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md)).
- **Axum over Actix-web**: Selected Axum for its Tower ecosystem integration, type-safe extractors, and alignment with the Tokio runtime.
- **YugabyteDB over PostgreSQL**: Selected YugabyteDB for distributed SQL capabilities, horizontal scaling, and built-in replication for disaster recovery.

---

## Deprecation Notices

### Planned Deprecations (v1.1.0)

| Feature | Replacement | Deprecation Date | Removal Date |
|---------|------------|-------------------|--------------|
| `/api/v1/anomalies/detect` (sync) | `/api/v1/anomalies/detect-async` | v1.1.0 | v2.0.0 |
| Static threshold rules | Adaptive threshold policies | v1.1.0 | v2.0.0 |
| Basic text search for incidents | Full-text search with Tantivy | v1.1.0 | v2.0.0 |

---

## Migration Guides

### Upgrading from v0.x to v1.0.0

1. **Database Migration**: Run all pending migrations. The v1.0.0 migration consolidates indexes and adds new tables.
   ```bash
   cargo run --bin aiops-migrate -- up
   ```

2. **Configuration Changes**: The configuration format has been updated. Key changes:
   - `ai_brain.url` renamed to `ai_brain.base_url`
   - `cache.redis_url` renamed to `cache.dragonfly_url`
   - Added required `security.scanning.enabled` configuration key

3. **API Breaking Changes**:
   - `GET /api/v1/incidents` response envelope changed from `{ data: [...] }` to `{ items: [...], total: N, page: N, per_page: N }`
   - Incident `status` enum: `acknowledged` renamed to `investigating` for ITIL alignment
   - All datetime fields now use RFC 3339 with millisecond precision

4. **Frontend**:
   - Update `@refinedev/core` to `4.x` and `@refinedev/antd` to `5.x`
   - Update API data provider configuration to match new response envelope format

---

## Contributors

The following teams and individuals contributed to the ERP-AIOps releases:

- **AIOps Core Team** -- Rust platform development, API design, database architecture
- **AI/ML Team** -- Python AI Brain, ML model development, forecasting algorithms
- **Platform Engineering** -- Go Gateway, CI/CD pipeline, infrastructure
- **Frontend Team** -- Refine.dev dashboard, data visualization, UX design
- **Security Team** -- Security scanning engine, compliance frameworks, penetration testing
- **SRE Team** -- Runbooks, monitoring, incident response procedures

---

*For questions about specific releases, contact the AIOps team or file an issue in the ERP-AIOps repository.*
