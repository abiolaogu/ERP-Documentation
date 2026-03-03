# Business Requirements Document (BRD)

| Field            | Value                                                        |
|------------------|--------------------------------------------------------------|
| **Document**     | Business Requirements Document                               |
| **Product**      | ERP-MessageBus -- Shared Multi-Tenant Event Streaming Platform|
| **Version**      | 2.0                                                          |
| **Date**         | 2026-03-03                                                   |
| **Author**       | Platform Engineering Team                                    |
| **Reviewers**    | VP Engineering, Security Lead, Compliance Officer            |
| **Status**       | Approved                                                     |

---

## 1. Executive Summary

Sovereign ERP 2026 is a modular, multi-tenant SaaS platform composed of 24+ domain modules (CRM, HRMS, Finance, Inventory, Manufacturing, and others). Each module is an independently deployable microservice with its own PostgreSQL database, Go backend, and React frontend. Today every module that needs asynchronous messaging deploys its own broker -- a mix of RabbitMQ, Redis Streams, and single-node Kafka instances. This fragmentation creates compounding operational cost, prevents cross-module real-time workflows, and leaves the platform without a unified audit trail for data-in-motion.

ERP-MessageBus replaces this heterogeneous broker landscape with a single, governed, multi-tenant Redpanda cluster that serves as the canonical event streaming data plane for the entire Sovereign ERP platform. The project delivers topic governance, tenant-scoped access control, CDC pipeline orchestration, tiered long-term storage, schema management, and a self-service developer experience -- all operated as a shared platform service.

This document captures the business requirements that justify, scope, and constrain the ERP-MessageBus initiative. It is the authoritative input for the [Product Requirements Document](./PRD.md), the [Enterprise Architecture Roadmap](./EA-Roadmap.md), and all downstream design artifacts.

---

## 2. Business Goals

| ID   | Goal                                        | Description                                                                                                                                          | Measurable Outcome                                                            |
|------|---------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| BG-1 | Unified Event Backbone                      | Establish a single, shared Redpanda cluster as the canonical asynchronous communication channel for all 24 ERP modules.                              | Zero standalone broker instances across all modules by end of Q2 2026.        |
| BG-2 | Reduced Operational Cost                    | Eliminate the cost of maintaining 24 independent broker deployments (patching, monitoring, capacity planning, incident response).                     | 60% reduction in broker-related infrastructure spend ($48K/yr to $19K/yr).    |
| BG-3 | Regulatory Compliance Readiness             | Provide encryption in transit and at rest, immutable audit logging for all topic operations, and data retention controls that satisfy GDPR, SOC 2, HIPAA, and PCI-DSS obligations. | 100% of topic CRUD operations auditable; encryption coverage verified quarterly. |
| BG-4 | Multi-Tenant Data Isolation                 | Guarantee that tenant data flowing through the bus cannot be observed, modified, or inferred by other tenants or unauthorized module service accounts.| Zero cross-tenant data leaks validated by quarterly penetration testing.       |
| BG-5 | Self-Service Topic Governance               | Enable module teams to provision topics, register schemas, and submit Connect pipelines through a GitOps-driven self-service workflow with guardrails.| New topic creation SLA under 5 minutes from PR merge to topic availability.   |
| BG-6 | Cross-Module Real-Time Workflows            | Unlock event-driven integration patterns (saga orchestration, CQRS projections, domain event fanout) that were previously impossible without a shared bus. | End-to-end event latency p99 under 50 ms between any two modules.            |
| BG-7 | Accelerated Module Onboarding               | Reduce the time for a new ERP module to begin producing and consuming events from approximately two days of bespoke broker setup to a streamlined, documented process. | Onboarding time under 30 minutes measured by onboarding checklist completion. |

---

## 3. Stakeholders

| Stakeholder                    | Role              | Interest & Responsibilities                                                                                                 | Engagement Level |
|--------------------------------|--------------------|-----------------------------------------------------------------------------------------------------------------------------|------------------|
| VP Engineering                 | Executive Sponsor  | Approves budget and strategic direction; accountable for cost reduction and platform reliability targets.                    | Inform           |
| Platform Engineering Lead      | Product Owner      | Owns the MessageBus backlog; defines acceptance criteria; resolves cross-team conflicts; accountable for delivery milestones.| Responsible      |
| Platform Architect             | Technical Lead     | Designs the cluster topology, governance model, and integration contracts; reviews all architectural decisions.              | Accountable      |
| Module Team Leads (x24)        | Consumers          | Integrate their modules with the bus; adopt naming conventions, envelope schema, and ACL policies; provide feedback.         | Consulted        |
| Security & AppSec Team         | Reviewer           | Reviews encryption posture, ACL policies, credential rotation procedures, and penetration test results.                     | Consulted        |
| Compliance Officer             | Reviewer           | Validates that controls satisfy GDPR, SOC 2, HIPAA, and PCI-DSS requirements; approves evidence artifacts.                 | Consulted        |
| SRE / DevOps Team              | Operator           | Operates the cluster day-to-day; executes runbooks; manages alerting, capacity planning, and incident response.             | Responsible      |
| Finance                        | Reviewer           | Tracks total cost of ownership; validates budget alignment; approves infrastructure procurement.                            | Inform           |
| QA Lead                        | Reviewer           | Validates integration test coverage for event flows; reviews chaos engineering results.                                     | Consulted        |

---

## 4. Problem Statement

The Sovereign ERP platform today consists of 24 independently deployed modules. Over time, module teams that needed asynchronous messaging each provisioned their own broker technology:

- **CRM** and **HRMS** use RabbitMQ on dedicated Docker containers.
- **Finance**, **Inventory**, and **Manufacturing** use single-node Kafka instances.
- **Fleet Management**, **Notifications**, and **Analytics** use Redis Streams.
- Several modules have no event infrastructure at all and rely on synchronous REST polling.

This fragmentation causes five critical problems:

1. **Operational Sprawl**: The SRE team maintains 18 distinct broker deployments across different technologies. Each requires its own monitoring dashboards, alerting rules, upgrade cadences, and incident runbooks. A single Kafka CVE requires patching 6 different clusters independently.

2. **Inconsistent Schemas and Contracts**: There is no shared schema registry. Module teams define their own message formats -- some use JSON, some use Protobuf, some use unstructured strings. Breaking schema changes propagate silently and cause downstream failures that are difficult to diagnose.

3. **No Cross-Module Event Flows**: Because each module operates its own isolated broker, there is no mechanism for module A to react to events published by module B. Cross-module integration is achieved through synchronous GraphQL queries or manual database polling, creating tight coupling and latency.

4. **No Audit Trail**: None of the existing broker deployments log topic creation, ACL changes, or message metadata in a centralized, queryable format. This is a compliance gap for GDPR (right to erasure tracking), SOC 2 (change management), and HIPAA (access logging).

5. **No Tiered Storage or Retention Policy**: Broker-local storage is ephemeral. Messages older than the broker's in-memory or disk retention window are permanently lost. There is no cold-tier archival for compliance-mandated retention periods (e.g., 7 years for financial events under SOX-adjacent requirements).

6. **Cost Duplication**: Running 18 broker instances with their associated compute, storage, and networking consumes approximately $4,000/month ($48,000/year). The majority of these brokers are underutilized, with average throughput below 5% of provisioned capacity.

---

## 5. Scope

### 5.1 In Scope

| Area                          | Description                                                                                                        |
|-------------------------------|--------------------------------------------------------------------------------------------------------------------|
| Shared Redpanda Cluster       | Multi-broker Redpanda deployment on Harvester HCI via Rancher, with NVMe-backed storage and rack-aware replication.|
| Topic Catalog & Governance    | Centralized topic registry (`TOPICS.md` + machine-readable YAML) with enforced naming conventions and ownership.   |
| ACL & RBAC Enforcement        | SASL/SCRAM service accounts per module with prefix-based ACL scoping; Redpanda Console RBAC via Authentik OIDC.    |
| Connect Pipeline Farm         | Shared Redpanda Connect deployment for CDC from YugabyteDB/PostgreSQL sources and event transformation pipelines.  |
| Redpanda Console              | Web-based cluster management UI with OIDC SSO, module-scoped topic visibility, and consumer group monitoring.      |
| Tiered Storage                | Automatic offload of cold log segments to RustFS (S3-compatible) for long-term retention at reduced cost.          |
| Schema Registry               | Centralized Avro, Protobuf, and JSON Schema management with compatibility mode enforcement (BACKWARD default).    |
| Quota Management              | Per-module produce/consume rate limits (default 10 MB/s in, 20 MB/s out) to prevent noisy-neighbor effects.       |
| Observability Integration     | Prometheus metrics export, Grafana dashboards, alerting rules, and integration with ERP-AIOps anomaly detection.   |
| Migration Tooling             | Scripts, runbooks, and dual-write adapters to migrate modules from legacy brokers with zero message loss.          |
| GitOps Deployment             | Fleet-managed Kubernetes manifests for all MessageBus infrastructure; PR-driven topic and pipeline provisioning.   |
| Developer Documentation       | Onboarding guides, SDK examples, troubleshooting runbooks, and architecture decision records.                     |

### 5.2 Out of Scope

| Area                          | Rationale                                                                                              |
|-------------------------------|--------------------------------------------------------------------------------------------------------|
| Application-level event handlers | Each module team owns their consumer/producer logic and business event semantics.                    |
| Business logic in Connect pipelines | Modules define transformation logic; the platform provides the runtime and guardrails.             |
| Database schema changes       | Consuming modules manage their own persistence layer independently.                                    |
| Frontend UI changes           | Module frontends are not affected by the MessageBus migration.                                         |
| GraphQL Federation            | The Cosmo Router / Hasura federation layer is a separate concern managed by ERP-Gateway.               |
| General-purpose messaging     | The bus serves only Sovereign ERP modules, not external or third-party workloads.                      |
| Streaming analytics engine    | Real-time analytics and BI are handled by the dedicated ERP-BI module consuming from the bus.          |

---

## 6. Non-Goals

1. **Replace the GraphQL Federation Layer**: Cosmo Router handles synchronous API composition; the MessageBus handles asynchronous event flows. These are complementary, not competing.
2. **Serve as a Primary Database**: The bus is a transit and short/medium-term retention layer, not a system of record. Modules must not use topic compaction as a substitute for a proper database.
3. **Support Non-ERP Workloads**: The cluster is sized, governed, and secured exclusively for Sovereign ERP modules. External systems integrate through dedicated gateway modules.
4. **Provide Real-Time Streaming Analytics**: The ERP-BI module consumes events from the bus and performs analytics. The bus itself does not run analytical queries or materializations.
5. **Replace Synchronous APIs for Request-Response Patterns**: Modules should continue to use GraphQL for synchronous queries. The bus is for fire-and-forget events, sagas, and CDC streams.

---

## 7. Success Metrics

| Metric                            | Baseline (Pre-MessageBus)         | Target (Post-MessageBus)        | Measurement Method                          | Review Cadence |
|-----------------------------------|-----------------------------------|---------------------------------|---------------------------------------------|----------------|
| Standalone broker instance count  | 18 instances across 6 technologies| 0 (single shared cluster)       | Infrastructure inventory audit              | Monthly        |
| Monthly infrastructure cost       | $4,000/month                      | $1,600/month                    | Cloud billing reports                       | Monthly        |
| Topic creation SLA                | 1-2 days (manual provisioning)    | < 5 minutes (GitOps automated)  | Time from PR merge to topic availability    | Per event      |
| Cluster availability              | ~99.5% (varied by module)         | 99.99% (< 4.4 min/month)        | Uptime monitoring via Prometheus/Alertmanager| Monthly        |
| Produce latency (p99)             | 15-200 ms (varied by technology)  | < 10 ms                         | Prometheus histogram (redpanda_produce_latency)| Continuous   |
| End-to-end event latency (p99)    | N/A (no cross-module events)      | < 50 ms                         | Distributed tracing (OpenTelemetry)         | Continuous     |
| Cross-tenant data leaks           | Unknown (no isolation controls)   | Zero                            | Quarterly penetration tests + ACL audits    | Quarterly      |
| Audit trail coverage              | 0% of topic operations            | 100% of topic CRUD + ACL changes| Audit log completeness checks               | Monthly        |
| Module onboarding time            | ~2 days                           | < 30 minutes                    | Onboarding checklist timestamps             | Per onboarding |
| Schema compatibility violations   | N/A (no registry)                 | Zero breaking changes in prod   | Schema Registry CI checks                   | Per deployment |

---

## 8. Assumptions & Constraints

### 8.1 Assumptions

| ID   | Assumption                                                                                                                                     |
|------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| A-1  | All 24 ERP modules will complete migration to the shared Redpanda cluster by end of Q2 2026.                                                  |
| A-2  | Redpanda v24.3+ supports the multi-tenant ACL model with wildcard prefix matching required for module isolation.                               |
| A-3  | RustFS provides S3-compatible object storage with sufficient throughput (>100 MB/s) for tiered storage segment offloading.                     |
| A-4  | Authentik OIDC integration supports group-to-role mapping that enables per-module Console RBAC scoping.                                       |
| A-5  | Harvester HCI nodes have sufficient NVMe-backed storage IOPS (>50,000 IOPS per node) for Redpanda's Raft consensus and log-structured storage.|
| A-6  | Module teams will adopt the standard envelope schema and topic naming convention without requiring backwards-compatible shim layers.           |
| A-7  | The platform team has sufficient headcount (3 engineers + 1 SRE) dedicated to the MessageBus initiative for the duration of 2026.             |
| A-8  | Redpanda Console's enterprise features (RBAC, audit logging) are available under the current licensing arrangement.                            |

### 8.2 Constraints

| ID   | Constraint                                                                                                                                     |
|------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| C-1  | The cluster must run on Harvester HCI infrastructure managed by Rancher; public cloud managed Kafka/Redpanda services are not approved.        |
| C-2  | All inter-service communication must use SASL/SCRAM authentication; plaintext connections are prohibited even in development environments.     |
| C-3  | Data residency requirements mandate that all message storage (local and tiered) remains within the organization's on-premise data centers.     |
| C-4  | The project must be delivered within the existing platform engineering budget; no incremental headcount or capital expenditure is approved.      |
| C-5  | Topic naming must follow the convention `[env].[org].erp.[module].[topic_name]` to enable prefix-based ACL scoping.                           |
| C-6  | Schema compatibility mode must default to BACKWARD to protect consumers from breaking changes.                                                 |
| C-7  | All GitOps manifests must be managed via Rancher Fleet; Helm-only or manual kubectl deployments are not permitted.                             |

---

## 9. Risks & Mitigations

| ID   | Risk                                              | Likelihood | Impact   | Mitigation Strategy                                                                                                                                           |
|------|---------------------------------------------------|------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| R-1  | Noisy-neighbor effect: a single module saturates the shared cluster with excessive produce throughput | Medium | High | Enforce per-module quotas (default 10 MB/s produce, 20 MB/s consume) via Redpanda quota configuration. Implement priority tiers for critical modules (Finance, HRMS). Monitor quota utilization via Prometheus alerts. |
| R-2  | Data loss during migration from legacy brokers     | Low        | Critical | Execute a dual-write period where both legacy and Redpanda receive events. Reconcile consumer offsets. Validate message counts before decommissioning legacy brokers. Maintain rollback capability for 30 days post-migration. |
| R-3  | SASL/SCRAM credential leak or compromise           | Low        | High     | Store credentials in Kubernetes Secrets managed by External Secrets Operator syncing from HashiCorp Vault. Enforce 90-day credential rotation. Monitor for unauthorized authentication attempts. Maintain credential revocation runbook. |
| R-4  | Tiered storage data corruption or unavailability   | Low        | Medium   | Enable checksummed log segments for integrity verification. Maintain local retention window (7 days) as fallback. Test restore-from-tiered-storage quarterly. Monitor RustFS health independently. |
| R-5  | Authentik OIDC outage blocks Console access         | Medium     | Low      | Maintain emergency admin bypass credentials stored in a break-glass vault. Console unavailability does not affect produce/consume paths (SASL/SCRAM is independent of OIDC). Document manual cluster administration via rpk CLI as fallback. |
| R-6  | Schema evolution breaks downstream consumers       | Medium     | High     | Enforce BACKWARD compatibility mode by default in the Schema Registry. Run schema compatibility checks in CI before merge. Provide consumer SDK helpers for schema migration. Maintain a schema changelog per topic. |
| R-7  | Insufficient Harvester HCI IOPS for Redpanda workload | Low     | High     | Benchmark NVMe performance before deployment with Redpanda's qualification tool (rpk iotune). Reserve dedicated storage pools for broker nodes. Plan capacity for 2x peak throughput headroom. |
| R-8  | Module team adoption resistance                    | Medium     | Medium   | Provide comprehensive onboarding documentation, SDK examples, and office hours. Demonstrate ROI through early-adopter success stories. Assign a platform engineer as liaison to each wave of migrating modules. |
| R-9  | Regulatory audit findings due to incomplete controls | Low       | High     | Map all controls to GDPR, SOC 2, HIPAA, and PCI-DSS requirements before go-live (see [Compliance-Regulatory-Matrix](./Compliance-Regulatory-Matrix.md)). Conduct internal audit dry-run in Q1 2026. Engage external auditor for gap assessment. |

---

## 10. Dependencies

| ID   | Dependency                                    | Provider               | Type          | Status   | Impact if Unavailable                                                    |
|------|-----------------------------------------------|------------------------|---------------|----------|--------------------------------------------------------------------------|
| D-1  | Redpanda cluster runtime (brokers + Raft)     | Platform Engineering   | Infrastructure| Active   | Complete project blocker; no event streaming capability.                 |
| D-2  | RustFS S3-compatible object storage           | Shared Infrastructure  | Infrastructure| Active   | Tiered storage unavailable; increased local storage costs.               |
| D-3  | Authentik OIDC identity provider              | Shared Infrastructure  | Authentication| Active   | Console RBAC unavailable; fallback to manual ACL management.             |
| D-4  | YugabyteDB / PostgreSQL (CDC sources)         | Module Teams           | Data Source   | Active   | CDC pipelines cannot capture changes; modules use manual event publish.  |
| D-5  | ERP-AIOps observability platform              | ERP-AIOps Module       | Monitoring    | Active   | Anomaly detection unavailable; rely on threshold-based alerting only.    |
| D-6  | cert-manager (mTLS certificates)              | Kubernetes Platform    | Security      | Active   | Inter-broker and client-broker encryption unavailable.                   |
| D-7  | Rancher Fleet (GitOps delivery)               | Platform Engineering   | Deployment    | Active   | Manual deployment required; GitOps workflow broken.                      |
| D-8  | External Secrets Operator + Vault             | Shared Infrastructure  | Secrets       | Active   | SASL credentials must be managed manually in K8s Secrets.                |
| D-9  | Harvester HCI bare-metal nodes                | Infrastructure Team    | Compute       | Active   | No compute substrate for Redpanda brokers.                               |
| D-10 | Module team migration readiness               | All 24 Module Teams    | Organizational| In Progress | Delayed module migrations extend dual-broker operational overhead.    |

---

## 11. Business Process Impact

### 11.1 Processes Improved

| Process                          | Current State                                     | Future State with MessageBus                                              |
|----------------------------------|--------------------------------------------------|---------------------------------------------------------------------------|
| Order-to-Cash                    | Synchronous REST calls between Sales, Inventory, Finance | Event-driven saga: OrderPlaced -> InventoryReserved -> InvoiceGenerated |
| Employee Onboarding              | Manual data entry across HRMS, IAM, Asset Management | HRMS publishes EmployeeCreated; IAM and AssetMgmt consume and auto-provision |
| Compliance Reporting             | Manual export from each module's database         | Continuous event stream to ERP-BI for real-time compliance dashboards     |
| Incident Notification            | Each module has its own notification logic         | Centralized notification pipeline consuming events from all modules       |
| Audit Trail Construction         | No unified audit trail exists                     | All topic operations logged; immutable audit trail queryable for 2+ years |

### 11.2 Processes Unchanged

- Synchronous GraphQL API queries between modules (handled by Cosmo Router federation).
- Module-internal database transactions (remain within each module's PostgreSQL).
- Frontend user interactions (no change to React/Refine.dev frontends).

---

## 12. Approval

| Role                    | Name              | Approval Status | Date       |
|-------------------------|-------------------|-----------------|------------|
| Executive Sponsor       | VP Engineering    | Approved        | 2026-02-15 |
| Product Owner           | Platform Eng Lead | Approved        | 2026-02-15 |
| Security Lead           | AppSec Lead       | Approved        | 2026-02-18 |
| Compliance Officer      | Compliance Lead   | Approved        | 2026-02-18 |
| Finance                 | Finance Lead      | Approved        | 2026-02-20 |

---

## 13. Document History

| Version | Date       | Author               | Changes                                         |
|---------|------------|-----------------------|-------------------------------------------------|
| 1.0     | 2026-01-15 | Platform Eng Lead     | Initial draft                                   |
| 1.1     | 2026-02-01 | Platform Architect    | Added risk register and dependency matrix       |
| 1.2     | 2026-02-10 | Compliance Officer    | Added compliance requirements and audit scope   |
| 2.0     | 2026-03-03 | Platform Eng Team     | Comprehensive rewrite; expanded all sections    |

---

**Related Documents**:
- [Product Requirements Document](./PRD.md)
- [Enterprise Architecture Roadmap](./EA-Roadmap.md)
- [Project Charter](./Project-Charter.md)
- [Compliance & Regulatory Matrix](./Compliance-Regulatory-Matrix.md)
