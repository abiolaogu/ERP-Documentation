# Project Charter

**Project**: ERP-MessageBus — Shared Event Streaming Platform
**Sponsor**: VP Engineering
**PM**: Platform Engineering Lead
**Date**: 2026-03-03
**Status**: Active

---

## 1. Project Purpose

Establish a centralized, governed, multi-tenant event streaming platform to replace per-module broker deployments across the Sovereign ERP ecosystem. Deliver cost savings, operational simplicity, data governance, and real-time cross-module capabilities.

## 2. Timeline & Milestones

| Milestone | Target Date | Status | Owner |
|-----------|------------|--------|-------|
| M1: Cluster deployed in shared-infra | 2026-01-15 | Complete | Platform Eng |
| M2: Topic catalog + naming convention | 2026-02-01 | Complete | Platform Eng |
| M3: SASL/SCRAM + ACLs for all modules | 2026-02-15 | Complete | Platform Eng |
| M4: Console OIDC + RBAC live | 2026-03-01 | Complete | Platform Eng |
| M5: First 5 modules migrated | 2026-03-31 | In Progress | Module Teams |
| M6: All 25 modules migrated | 2026-05-30 | Planned | Module Teams |
| M7: Legacy brokers decommissioned | 2026-06-15 | Planned | Platform Eng |
| M8: Schema Registry GA | 2026-07-31 | Planned | Platform Eng |
| M9: Cross-module subscriptions GA | 2026-09-30 | Planned | Platform Eng |

## 3. RACI Matrix

| Activity | Platform Eng | Module Teams | Security | SRE | VP Eng |
|----------|:---:|:---:|:---:|:---:|:---:|
| Cluster deployment | R/A | I | C | C | I |
| Topic governance framework | R/A | C | C | I | I |
| Module migration | C | R/A | I | C | I |
| ACL management | R/A | C | C | I | I |
| Security review | C | I | R/A | I | I |
| Incident response | C | I | I | R/A | I |
| Budget approval | C | I | I | I | R/A |
| Connect pipeline review | R/A | R | C | I | I |

*R=Responsible, A=Accountable, C=Consulted, I=Informed*

## 4. Budget Approach

| Item | Annual Cost | Notes |
|------|-----------|-------|
| Redpanda cluster (5 nodes) | $14,400 | Harvester HCI bare-metal |
| RustFS tiered storage | $2,400 | 10 TB cold storage |
| Operational overhead (0.5 FTE) | $75,000 | Shared with platform team |
| **Total** | **$91,800** | vs. $192,000 for 25 separate brokers |
| **Net savings** | **$100,200/yr** | 52% reduction |

## 5. Success Criteria

| Criterion | Measure | Target |
|-----------|---------|--------|
| All modules migrated | Module count on shared cluster | 25/25 |
| Zero message loss | Audit reconciliation | 0 lost events |
| Operational simplicity | Incident count (broker-related) | < 2/month |
| Developer satisfaction | Quarterly survey | > 4.0/5.0 |
| Compliance | Audit findings | 0 critical/high |

## 6. Communication Plan

| Audience | Channel | Frequency | Content |
|----------|---------|-----------|---------|
| VP Engineering | Executive summary | Monthly | Status, risks, budget |
| Module Teams | Slack #erp-messagebus | Continuous | Updates, migration guides |
| All Engineering | Town Hall | Quarterly | Roadmap, demos |
| Security | Security review meeting | Bi-weekly | Compliance progress |
| SRE | On-call handoff | Weekly | Operational status |

## 7. Constraints

- No per-module broker deployments after Phase 2 complete
- All events must use the standard envelope format
- Topic names must pass automated validation in CI
- No plaintext credentials in source control
- Connect pipelines must be stateless (no local state)

## 8. Risks & Escalation

See [BRD Risk Register](./BRD.md#9-risks). Escalation path:
1. Module Team Lead → Platform Engineering Lead
2. Platform Engineering Lead → VP Engineering
3. VP Engineering → CTO (for budget/timeline changes)

---

**Related**: [BRD](./BRD.md) | [PRD](./PRD.md) | [EA-Roadmap](./EA-Roadmap.md)
