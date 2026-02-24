# ERP-DBaaS Business Requirements Document (BRD)

## Document Control

| Field             | Value                                     |
|-------------------|-------------------------------------------|
| Document Title    | ERP-DBaaS Business Requirements Document  |
| Version           | 1.0.0                                    |
| Date              | 2026-02-24                                |
| Status            | Approved                                  |
| Sponsor           | VP of Engineering                         |
| Classification    | Internal - Business                       |

---

## 1. Business Context

### 1.1 Current State

The ERP ecosystem consists of 14+ modules, each requiring one or more database instances for development, staging, and production environments. Currently, database provisioning follows a manual, ticket-driven process:

**Current Process**:
1. Developer submits a Jira ticket requesting a database (Day 1)
2. Ticket is triaged and assigned to a DBA (Day 1-2)
3. DBA evaluates requirements and selects engine/configuration (Day 2-3)
4. DBA manually provisions the database via scripts or console (Day 3-4)
5. DBA configures backups, monitoring, and access controls (Day 4-5)
6. Credentials are shared with the requester via encrypted email (Day 5)
7. Developer validates connectivity and begins integration (Day 5+)

**Current Pain Points**:

| Pain Point                      | Business Impact                             | Affected Stakeholders |
|---------------------------------|---------------------------------------------|-----------------------|
| 2-5 day provisioning lead time  | Delays feature delivery by a full sprint    | Developers, Product   |
| Manual configuration errors     | 15% of new databases require re-work        | DBAs, Developers      |
| Inconsistent security settings  | 3 audit findings in the past year           | Security, Compliance  |
| No self-service capability      | DBA team is bottleneck (40% of time on provisioning) | DBAs, Management |
| No centralized visibility       | Cannot answer "how many databases exist?"   | Management, FinOps    |
| Incomplete backup coverage      | 30% of dev databases lack any backup        | All stakeholders      |
| No cost attribution             | Cannot attribute infrastructure costs       | FinOps, Management    |

### 1.2 Desired State

A self-service, policy-governed platform where any authorized team member can provision a production-grade database in under 5 minutes, with full backup coverage, security compliance, and cost visibility from day one.

---

## 2. Business Objectives

### 2.1 Primary Objectives

**BO-1: Reduce Database Provisioning Time from Days to Minutes**

| Metric                  | Current State | Target State | Improvement |
|-------------------------|---------------|--------------|-------------|
| Average provisioning time| 3.2 days     | < 5 minutes  | 99.9%       |
| Median provisioning time | 2.5 days     | < 3 minutes  | 99.9%       |
| P95 provisioning time   | 5.0 days     | < 10 minutes | 99.9%       |

**BO-2: Enforce Governance on 100% of Database Instances**

| Metric                       | Current State | Target State | Improvement |
|------------------------------|---------------|--------------|-------------|
| Instances meeting security standards | 70%    | 100%         | 30pp        |
| Instances with backup coverage | 70%          | 100% (prod)  | 30pp        |
| Instances with HA enabled (prod) | 45%        | 100%         | 55pp        |
| Audit compliance findings    | 3/year        | 0/year       | 100%        |

**BO-3: Reduce DBA Operational Burden by 80%**

| Metric                       | Current State    | Target State   | Improvement |
|------------------------------|------------------|----------------|-------------|
| DBA time on provisioning     | 40% of capacity  | 5% of capacity | 87.5%       |
| DBA time on routine backups  | 20% of capacity  | 2% of capacity | 90%         |
| DBA time on credential mgmt  | 10% of capacity  | 1% of capacity | 90%         |
| DBA time on strategic work   | 30% of capacity  | 92% of capacity| 207% increase|

**BO-4: Enable Full Cost Visibility and Attribution**

| Metric                       | Current State | Target State | Improvement |
|------------------------------|---------------|--------------|-------------|
| Cost attribution accuracy    | 0% (no tracking) | 100%      | Full        |
| Time to generate cost report | N/A           | < 1 minute   | Full        |
| Budget forecast accuracy     | N/A           | +/- 10%      | Full        |

### 2.2 Secondary Objectives

**BO-5: Accelerate New ERP Module Delivery**
- New modules can start with production-grade data infrastructure in the first sprint
- Standardized database patterns reduce design decisions and integration time
- Expected delivery acceleration: 2-3 weeks per new module

**BO-6: Improve Developer Experience**
- Self-service eliminates dependency on DBA team for standard operations
- Guided wizard helps developers choose the right engine for their workload
- Automatic credential delivery removes manual coordination
- Target Developer NPS: >40

**BO-7: Establish Platform Engineering as a Capability**
- DBaaS demonstrates the value of internal platform products
- Plugin system enables community contributions from module teams
- Sets the pattern for future platform services (messaging, identity, etc.)

---

## 3. Scope

### 3.1 In Scope

| Area                    | Description                                                |
|-------------------------|------------------------------------------------------------|
| Database Provisioning   | Self-service provisioning of 8 database engines            |
| Lifecycle Management    | Scaling, backup, restore, credential rotation, decommission|
| Governance              | AIDD policy engine with strict and flexible profiles       |
| Multi-Tenancy           | Tenant isolation, quotas, and per-tenant metering          |
| Frontend Application    | React admin UI with Refine.dev framework                   |
| API Layer               | REST and GraphQL APIs for programmatic access              |
| Plugin System           | gRPC-based extensibility framework                         |
| Monitoring Integration  | Health monitoring, metrics export, alerting hooks          |

### 3.2 Out of Scope (v1.0.0)

| Area                    | Rationale                                                  |
|-------------------------|------------------------------------------------------------|
| Database Query Optimization | Domain of application teams and DBAs; not a platform concern |
| Schema Management       | Handled by application-level migration tools (Hasura, etc.)|
| Multi-Region Replication| Planned for v1.1.0; requires cross-cluster networking      |
| Serverless Databases    | Planned for v2.0.0; requires significant operator changes  |
| External Customer Access| Internal platform only; no external-facing APIs            |
| Database Engine Development | We consume upstream engines; do not fork or modify them |

---

## 4. Stakeholder Analysis

### 4.1 RACI Matrix

| Activity                        | Executive Sponsor | Product Owner | Platform Eng | DBA Team | Security | FinOps |
|---------------------------------|-------------------|---------------|-------------|----------|----------|--------|
| Define business objectives      | A                 | R             | C           | C        | C        | C      |
| Approve feature priorities      | A                 | R             | C           | C        | I        | I      |
| Design architecture             | I                 | C             | R           | C        | C        | I      |
| Implement platform              | I                 | C             | R           | C        | C        | I      |
| Define governance policies      | I                 | C             | C           | R        | A        | I      |
| Define quota structures         | I                 | C             | C           | I        | I        | R/A    |
| User acceptance testing         | I                 | A             | C           | R        | R        | C      |
| Production deployment           | I                 | A             | R           | C        | C        | I      |
| Ongoing operations              | I                 | I             | R           | C        | C        | C      |
| Cost reporting and optimization | I                 | I             | C           | I        | I        | R      |

*R = Responsible, A = Accountable, C = Consulted, I = Informed*

### 4.2 Stakeholder Benefits

| Stakeholder       | Key Benefit                                   | Quantified Value                |
|-------------------|-----------------------------------------------|---------------------------------|
| Application Developers | Self-service database in minutes          | Save 3+ days per request        |
| DBA Team          | Focus on strategic work, not provisioning     | Recover 70% of capacity         |
| Security Team     | Automated compliance enforcement              | Zero audit findings goal        |
| FinOps            | Full cost visibility and attribution          | Enable chargeback model         |
| Engineering Management | Faster delivery, lower costs              | 2-3 week acceleration per module|
| Executive Leadership | Reduced OpEx, improved agility             | Estimated $450K annual savings  |

---

## 5. Success Metrics and KPIs

### 5.1 Leading Indicators (measured monthly)

| KPI                                | Target          | Measurement Method             |
|------------------------------------|-----------------|--------------------------------|
| Provisioning requests via DBaaS    | >90% of total   | DBaaS metrics vs. Jira tickets |
| Average provisioning time          | < 3 minutes     | API metrics                    |
| Policy compliance rate             | 100% (prod)     | Policy engine reports          |
| Active tenants                     | +2/month        | Tenant registry                |
| API availability                   | > 99.95%        | Uptime monitoring              |

### 5.2 Lagging Indicators (measured quarterly)

| KPI                                | Target          | Measurement Method             |
|------------------------------------|-----------------|--------------------------------|
| DBA provisioning hours saved       | 80% reduction   | Time tracking comparison       |
| Security audit findings            | 0 database-related| Audit reports                |
| Module delivery acceleration       | 2 weeks faster  | Sprint velocity comparison     |
| Developer NPS                      | > 40            | Quarterly survey               |
| Cost attribution coverage          | 100%            | FinOps reports                 |
| Annual cost savings                | $450K+          | Total cost comparison          |

### 5.3 Operational Metrics

| Metric                             | Target          | Alert Threshold                |
|------------------------------------|-----------------|--------------------------------|
| Platform uptime                    | 99.95%          | < 99.9% (monthly)             |
| Backup success rate                | 99.9%           | < 99% (daily)                 |
| Failed provisioning rate           | < 1%            | > 3% (daily)                  |
| Operator restart rate              | < 1/week        | > 3/day                       |
| API error rate (5xx)               | < 0.1%          | > 1% (hourly)                 |

---

## 6. Cost Model

### 6.1 Implementation Costs

| Cost Category          | Estimate       | Notes                              |
|------------------------|----------------|------------------------------------|
| Engineering (6 months) | $800,000       | 4 senior engineers + 2 mid-level   |
| Infrastructure (annual)| $180,000       | K8s cluster, storage, networking   |
| Platform services      | $60,000/year   | YugabyteDB, DragonflyDB, RustFS licenses (if applicable) |
| Training and adoption  | $30,000        | Materials, workshops, documentation|
| **Total Year 1**       | **$1,070,000** |                                    |
| **Annual Recurring**   | **$240,000**   | Infra + platform + 1 FTE maintenance |

### 6.2 Expected Savings

| Savings Category       | Annual Estimate | Calculation                        |
|------------------------|-----------------|------------------------------------|
| DBA time recovered     | $280,000        | 2.5 FTE x $112K average            |
| Developer time saved   | $150,000        | 500 requests x 3 days x $100/hour  |
| Incident reduction     | $50,000         | 5 fewer incidents x $10K avg cost  |
| Audit remediation      | $30,000         | 3 fewer findings x $10K remediation|
| **Total Annual Savings** | **$510,000**  |                                    |

### 6.3 ROI Analysis

| Metric                 | Value           |
|------------------------|-----------------|
| Year 1 Net Cost        | $560,000        |
| Year 2 Net Savings     | $270,000        |
| Year 3 Net Savings     | $270,000        |
| 3-Year TCO             | $1,790,000      |
| 3-Year Savings         | $1,530,000      |
| 3-Year Net             | -$260,000       |
| Breakeven              | Month 26        |
| 5-Year ROI             | 58%             |

*Note: ROI calculations do not include intangible benefits such as improved developer satisfaction, faster time-to-market, and reduced security risk.*

### 6.4 Chargeback Model

For internal cost attribution, the following chargeback rates apply:

| Resource         | Unit            | Rate (Monthly)   |
|------------------|-----------------|------------------|
| vCPU             | Per vCPU        | $25              |
| Memory           | Per GB          | $8               |
| Storage (SSD)    | Per GB          | $0.15            |
| Backup Storage   | Per GB          | $0.05            |
| Network Egress   | Per GB          | $0.02            |
| Platform Fee     | Per instance    | $10              |

---

## 7. Risk Assessment

### 7.1 Business Risks

| Risk ID | Risk Description                        | Likelihood | Impact  | Mitigation                               |
|---------|------------------------------------------|-----------|---------|------------------------------------------|
| BR-01   | Low adoption by teams                    | Medium    | High    | Executive sponsorship, training program, quick wins |
| BR-02   | Insufficient ROI to justify investment   | Low       | High    | Phased delivery, early value demonstration |
| BR-03   | Platform instability causes incidents    | Medium    | Critical| Extensive testing, HA deployment, runbooks |
| BR-04   | Compliance gaps in AIDD policies         | Low       | High    | Security team co-ownership of policy rules |
| BR-05   | Budget overrun on implementation         | Medium    | Medium  | Agile delivery, scope prioritization      |
| BR-06   | Key person dependency                    | Medium    | Medium  | Knowledge sharing, documentation, pair programming |

### 7.2 Technical Risks

| Risk ID | Risk Description                        | Likelihood | Impact  | Mitigation                               |
|---------|------------------------------------------|-----------|---------|------------------------------------------|
| TR-01   | K8s operator bugs causing data loss      | Low       | Critical| Comprehensive testing, backup-first policy |
| TR-02   | Performance degradation at scale         | Medium    | High    | Load testing, capacity planning, auto-scaling |
| TR-03   | Security vulnerability in plugin system  | Medium    | High    | Plugin sandboxing, security scanning      |
| TR-04   | Upstream engine breaking changes         | Medium    | Medium  | Version pinning, compatibility testing    |
| TR-05   | RustFS reliability for backup storage    | Low       | High    | Erasure coding, multi-node deployment     |

---

## 8. Assumptions and Constraints

### 8.1 Assumptions

1. Kubernetes cluster with sufficient capacity is available and maintained by infrastructure team
2. Authentik identity provider is operational and configured for all ERP teams
3. Network connectivity between control plane and data plane namespaces is reliable
4. Storage classes with dynamic provisioning are available for all required storage types
5. Teams are willing to adopt self-service model with training support
6. DBA team supports transition from manual to platform-driven operations

### 8.2 Constraints

1. Must operate within existing Kubernetes infrastructure (no new cloud accounts)
2. Must integrate with Authentik for authentication (no alternative identity providers)
3. Must comply with existing data classification and retention policies
4. Budget is capped at $1.1M for Year 1 (implementation + infrastructure)
5. Must be production-ready within 6 months of project initiation
6. Must support all 8 specified database engines at GA (no phased engine rollout)

---

## 9. Acceptance Criteria

### 9.1 Business Acceptance Criteria

| ID    | Criterion                                                  | Verification Method     |
|-------|------------------------------------------------------------|-----------------------|
| BAC-1 | Any authorized user can provision a database in < 5 minutes | Timed user test        |
| BAC-2 | All production instances pass AIDD strict policy            | Policy compliance report|
| BAC-3 | Backup coverage is 100% for production instances            | Backup status dashboard |
| BAC-4 | Cost attribution is available per tenant and instance       | Metering report        |
| BAC-5 | DBA team confirms 50%+ time savings in first quarter        | DBA team survey        |
| BAC-6 | At least 3 ERP modules onboarded at GA                      | Tenant registry        |
| BAC-7 | Zero critical security findings in pre-launch assessment    | Security assessment    |

### 9.2 Technical Acceptance Criteria

| ID    | Criterion                                                  | Verification Method     |
|-------|------------------------------------------------------------|-----------------------|
| TAC-1 | All 8 database engines provision successfully              | Automated E2E tests    |
| TAC-2 | Backup and restore complete successfully for all engines    | Automated E2E tests    |
| TAC-3 | Platform maintains 99.95% uptime over 2-week soak test     | Monitoring data        |
| TAC-4 | API p99 latency < 500ms under load                          | Load test results      |
| TAC-5 | Credential rotation completes without instance downtime      | Rotation test          |
| TAC-6 | Tenant isolation prevents cross-tenant data access           | Penetration test       |
| TAC-7 | Plugin system handles plugin failure without platform impact | Chaos test             |

---

## 10. Approval

| Role                | Name                | Decision | Date       |
|---------------------|---------------------|----------|------------|
| Executive Sponsor   |                     | Approved | 2026-02-24 |
| Product Owner       |                     | Approved | 2026-02-24 |
| Engineering Lead    |                     | Approved | 2026-02-24 |
| DBA Lead            |                     | Approved | 2026-02-24 |
| Security Lead       |                     | Approved | 2026-02-24 |
| FinOps Lead         |                     | Approved | 2026-02-24 |

---

*This BRD is a living document maintained by the Product Owner. Changes to business objectives or scope require re-approval from the Executive Sponsor.*
