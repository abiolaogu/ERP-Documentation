# Product Requirements Document (PRD)

**Product**: ERP-MessageBus
**Version**: 1.0
**Date**: 2026-03-03
**Owner**: Platform Engineering Team
**Status**: Approved

---

## 1. Overview

ERP-MessageBus is the shared event streaming backbone for the Sovereign ERP 2026 platform. It provides topic governance, pipeline orchestration, console access control, and schema management as a platform service consumed by all 25 ERP modules.

## 2. Personas

| Persona | Description | Key Needs |
|---------|-------------|-----------|
| **Module Developer** | Backend/frontend engineer on an ERP module team | Produce/consume events, submit pipelines, view topic data |
| **Platform Engineer** | Infra/platform team member | Manage cluster, ACLs, quotas, troubleshoot, capacity plan |
| **Team Lead** | Module team technical lead | Monitor team's topics, review pipeline submissions, track SLOs |
| **Security Engineer** | AppSec / compliance team | Audit access, review encryption, validate compliance controls |
| **SRE** | Site reliability engineer | Respond to incidents, monitor health, execute runbooks |
| **Product Manager** | Non-technical stakeholder | Understand event flows, track feature enablement via events |

## 3. User Journeys

### Journey 1: Module Developer Produces First Event
1. Developer reads topic catalog (`TOPICS.md`)
2. Finds their module's topic prefix in the registry
3. Configures producer with SASL credentials from K8s secret
4. Sends event using standard envelope format
5. Verifies delivery in Redpanda Console (module-scoped view)

### Journey 2: Platform Engineer Onboards New Module
1. Creates SASL user: `svc-erp-<module>`
2. Creates ACL entry granting prefix access to `*.*.erp.<module>.*`
3. Registers topics in topic catalog
4. Sets producer/consumer quotas
5. Provides credentials via K8s secret
6. Module team starts producing within 30 minutes

### Journey 3: SRE Responds to Consumer Lag Alert
1. Receives PagerDuty alert: consumer group lag > threshold
2. Opens Redpanda Console, navigates to consumer group
3. Identifies lagging partitions
4. Checks consumer health via module's health endpoint
5. Executes remediation: restart consumer pod or increase replicas
6. Verifies lag recovery in dashboard

### Journey 4: Security Engineer Audits Topic Access
1. Opens Console with OIDC login
2. Navigates to ACL management view
3. Reviews per-module access patterns
4. Cross-references with audit log for unauthorized access attempts
5. Generates compliance report

## 4. Feature Requirements

### 4.1 Functional Requirements

| ID | Feature | Priority | Description | Acceptance Criteria |
|----|---------|----------|-------------|---------------------|
| FR-01 | Topic Catalog | P0 | Centralized registry of all topics with ownership and schema | All 25 modules have registered topics; searchable in Console |
| FR-02 | SASL/SCRAM Auth | P0 | Per-module service account authentication | Each module has unique credentials; unauthorized access denied |
| FR-03 | Prefix-Based ACLs | P0 | Module isolation via topic prefix scoping | Module can only read/write topics matching its prefix |
| FR-04 | Redpanda Console OIDC | P0 | SSO login for Console via Authentik | Users authenticate via Authentik; roles determine visibility |
| FR-05 | Console RBAC | P0 | Role-based access control in Console | infra-admin sees all; team-X sees only their prefix |
| FR-06 | Tiered Storage | P1 | Offload cold segments to RustFS | Segments older than local retention automatically offloaded |
| FR-07 | Connect Pipeline Farm | P1 | Shared Redpanda Connect deployment for CDC/transforms | Teams submit pipelines via PR; deployed automatically |
| FR-08 | Schema Registry | P1 | Centralized Avro/Protobuf/JSON schema management | Schemas registered per topic; compatibility enforced |
| FR-09 | Quota Management | P1 | Per-module produce/consume rate limits | Default 10 MB/s in, 20 MB/s out; configurable per module |
| FR-10 | Dead Letter Queue | P2 | DLQ pattern for failed message processing | Standard DLQ topic per module; Connect routes failures |
| FR-11 | Cross-Module Events | P2 | Curated cross-module event subscriptions | AIOps reads all; modules can request cross-module read ACLs |
| FR-12 | Audit Logging | P0 | All topic CRUD and ACL changes logged | Queryable audit trail; retained for 2 years |

### 4.2 Non-Functional Requirements

| ID | Requirement | Target | Rationale |
|----|-------------|--------|-----------|
| NFR-01 | Throughput | 500 MB/s aggregate cluster | Support all 25 modules at peak |
| NFR-02 | Latency (p99) | < 10ms produce, < 50ms end-to-end | Real-time cross-module flows |
| NFR-03 | Availability | 99.95% (< 22 min/month downtime) | Business-critical event flows |
| NFR-04 | Durability | Zero message loss (acks=all, min.insync=2) | Financial and healthcare events |
| NFR-05 | Scalability | Linear scaling to 50 modules | Growth headroom |
| NFR-06 | Recovery | RPO=0, RTO<5 min | Replicated + tiered storage |
| NFR-07 | Security | Encryption in transit (mTLS) and at rest | Compliance: SOC2, HIPAA |
| NFR-08 | Observability | Full metrics, logs, traces | SLO monitoring and alerting |

## 5. MVP vs Later

### MVP (v1.0)
- Shared Redpanda cluster with SASL/SCRAM
- Topic catalog with naming enforcement
- Prefix-based ACLs for all 25 modules
- Console with OIDC login + RBAC
- Tiered storage to RustFS
- 3 reference Connect pipelines
- Basic quota management

### v1.1
- Schema Registry with compatibility modes
- Advanced quota tiers (burst, priority)
- Self-service topic creation via PR workflow
- Cross-module event subscription approval flow

### v2.0
- Schema evolution CI validation
- Real-time data quality scoring
- Connect pipeline auto-scaling
- Multi-region replication

## 6. User Flows

### Produce Event Flow
```
Developer → Configure SASL credentials → Create producer
→ Serialize message with envelope stamp → Produce to topic
→ Broker validates ACL → Acknowledges → Console shows message
```

### Pipeline Submission Flow
```
Developer → Write Connect YAML → PR to ERP-MessageBus
→ CI validates YAML + naming + quotas → Reviewer approves
→ Fleet deploys pipeline → Connect processes events
→ Developer monitors in Console
```

---

**Dependencies**: [BRD](./BRD.md)
**Next**: [EA-Roadmap](./EA-Roadmap.md) → [Architecture](../01-architecture/SAD.md)
