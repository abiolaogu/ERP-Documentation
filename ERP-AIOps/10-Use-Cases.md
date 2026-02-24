# ERP-AIOps Use Cases

> **Document ID:** ERP-AIOPS-UC-010
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [01-Technical-Writeup.md](./01-Technical-Writeup.md), [08-Training-Manual.md](./08-Training-Manual.md)

---

## Use Case 1: Detecting CPU Anomaly and Auto-Scaling

### Scenario

The ERP-Commerce module experiences a sudden surge in traffic during a flash sale event. CPU utilization spikes beyond the normal seasonal pattern.

### Detection

1. Metrics from the ERP-Commerce pods flow through the OTel Collector federation into the AIOps ingestion pipeline.
2. The Python AI brain's anomaly detection ensemble runs in parallel:
   - **Z-Score**: Flags CPU at 4.2 standard deviations above the rolling mean.
   - **Isolation Forest**: Confirms the multi-dimensional anomaly (CPU + request rate + queue depth).
   - **LSTM**: Compares against the seasonal pattern and confirms this exceeds the expected peak.
3. The fused anomaly score is 0.87 (critical confidence).

### Correlation

4. The Rust correlation engine identifies concurrent anomalies on the ERP-Commerce database connection pool and the downstream payment gateway latency.
5. A correlated incident is created: **"ERP-Commerce CPU Spike with Cascade Effects"** (Severity P2).

### Auto-Remediation

6. A pre-configured playbook triggers: **"Auto-Scale on CPU Spike"**.
7. The approval policy allows auto-execution during business hours.
8. The remediation engine scales ERP-Commerce from 3 to 6 replicas.
9. Verification step confirms CPU drops below 75% within 3 minutes.

### Outcome

| Metric | Without AIOps | With AIOps |
|--------|--------------|------------|
| Detection Time | 8-15 minutes (manual) | 12 seconds |
| Response Time | 20-30 minutes (human scale) | 45 seconds (auto-scale) |
| User Impact Duration | 30-45 minutes | 3 minutes |
| Revenue Protected | N/A | Estimated $45K (flash sale period) |

---

## Use Case 2: Correlating Cascade Failures Across ERP Modules

### Scenario

A misconfigured rate limiter on the ERP-IAM authentication service causes intermittent 503 errors. Because all ERP modules depend on IAM for authentication, a cascade of failures spreads across the ecosystem.

### Detection

1. Within 30 seconds, AIOps detects error rate anomalies on 8 different ERP modules.
2. Without correlation, this would generate 8+ separate incidents, overwhelming the on-call team.

### Correlation

3. The temporal correlation engine identifies that all error spikes started within a 15-second window.
4. The topological correlation engine traces all affected services back to ERP-IAM as the common upstream dependency.
5. A single correlated incident is created: **"IAM Authentication Cascade Failure"** (Severity P1).
6. The incident timeline shows the propagation path: IAM -> CRM, Finance, HCM, Commerce, SCM, Procurement, Projects, Analytics.

### Root Cause Analysis

7. The Python AI brain collects evidence:
   - IAM access logs from Quickwit showing 503 responses starting at 14:32:07 UTC.
   - IAM configuration change log showing a rate limit modification deployed at 14:31:45 UTC.
   - Metrics showing IAM request rejection rate jumping from 0.1% to 34%.
8. The LLM-powered RCA generates: *"Root cause: Rate limiter configuration change deployed to ERP-IAM at 14:31:45 UTC reduced the per-tenant request limit from 10,000/min to 100/min. This caused legitimate authentication requests from all downstream modules to be rejected."*

### Outcome

| Metric | Without AIOps | With AIOps |
|--------|--------------|------------|
| Incidents Created | 8+ separate incidents | 1 correlated incident |
| Root Cause Identification | 45-90 minutes (manual log analysis) | 2 minutes (automated RCA) |
| MTTR | 60-120 minutes | 8 minutes (config rollback) |

---

## Use Case 3: Predicting Disk Space Exhaustion 48 Hours Ahead

### Scenario

The ERP-Finance module's YugabyteDB instance is gradually consuming disk space due to an unnoticed increase in transaction logging volume after a recent audit compliance update.

### Detection

1. The Python AI brain's forecasting module (Prophet + ARIMA ensemble) analyzes the disk usage time series for all monitored storage volumes.
2. The forecast predicts that the ERP-Finance database volume will reach 95% capacity in approximately 46 hours at the current growth rate.
3. A predictive anomaly is generated with score 0.72 (high confidence).

### Proactive Action

4. An incident is created: **"Predicted Disk Exhaustion: ERP-Finance DB in 46h"** (Severity P3).
5. The RCA module identifies the root cause: transaction log volume increased 340% after the audit compliance update deployed 5 days ago.
6. Recommendations are generated:
   - Short-term: Expand the volume by 50% (auto-remediation available).
   - Medium-term: Implement log rotation with 30-day retention.
   - Long-term: Review audit logging granularity to reduce volume.

### Outcome

| Metric | Without AIOps | With AIOps |
|--------|--------------|------------|
| Discovery | After disk full (outage) | 46 hours before exhaustion |
| Impact | Full database outage, Finance module down | Zero downtime |
| Response | Emergency expansion under pressure | Planned expansion with optimization |

---

## Use Case 4: Automated Incident Creation from Correlated Alerts

### Scenario

During a routine deployment of ERP-SCM (Supply Chain Management), a memory leak in the new version causes gradual degradation.

### Detection Flow

1. **14:00** - Deployment completes. No immediate issues.
2. **14:15** - Memory usage anomaly detected on ERP-SCM (score 0.41, medium confidence).
3. **14:22** - Response latency anomaly detected on ERP-SCM API (score 0.55).
4. **14:28** - Garbage collection pause anomaly detected (score 0.67).
5. **14:30** - Error rate increase on ERP-SCM downstream: ERP-Procurement (score 0.52).

### Correlation and Incident Creation

6. The correlation engine groups all four anomalies based on:
   - Temporal proximity (all within 30-minute window).
   - Topological relationship (SCM is upstream of Procurement).
   - Causal pattern matching (memory -> latency -> GC pauses -> downstream errors follows known memory leak pattern).
7. A single incident is created: **"ERP-SCM Memory Leak Post-Deployment"** (Severity P2).
8. The incident automatically includes all four anomalies as contributing evidence.

### Resolution

9. The deployment rollback playbook triggers with manual approval.
10. On-call engineer approves. Rollback completes in 90 seconds.
11. All anomaly scores return to baseline within 5 minutes.

---

## Use Case 5: Root Cause Analysis for Latency Spikes

### Scenario

Users report slow response times across multiple ERP modules. There is no obvious single point of failure.

### Investigation

1. A P2 incident is created manually by the on-call engineer.
2. The engineer triggers RCA from the incident detail page.
3. The AI brain collects evidence across all ERP modules:

**Metrics Analysis:**
- 12 ERP modules showing p99 latency increases of 2-5x.
- Network metrics show no anomalies.
- CPU and memory within normal ranges.

**Log Analysis (Quickwit):**
- Database connection pool exhaustion warnings across multiple modules.
- YugabyteDB slow query log showing queries taking 10-50x longer than baseline.

**Trace Analysis:**
- Distributed traces show the shared YugabyteDB cluster as the common bottleneck.
- Specific table scans on the `audit_log` table taking 15 seconds instead of 50ms.

4. The RCA report identifies: *"Root cause: Missing index on the `audit_log.created_at` column in the shared YugabyteDB cluster. A recent migration added an audit query that performs a full table scan. The table grew past 50M rows, causing query times to degrade and exhaust connection pools across all modules using this database."*

### Resolution

5. The DBA applies the missing index. Query times drop from 15s to 12ms.
6. All 12 module latencies return to baseline within 2 minutes.
7. Total time from report to resolution: 18 minutes.

---

## Use Case 6: Cost Optimization for Over-Provisioned Services

### Scenario

After a seasonal peak (year-end financial close), several ERP modules remain scaled up beyond what is needed.

### Analysis

1. The cost analysis engine reviews 90 days of resource utilization data for all ERP modules.
2. It identifies over-provisioned services:

| Service | Current Replicas | Peak Usage | Avg Usage | Recommended |
|---------|-----------------|------------|-----------|-------------|
| ERP-Finance | 8 | 85% (Dec) | 22% (Jan-Feb) | 3 |
| ERP-HCM | 6 | 78% (Dec) | 18% (Jan-Feb) | 2 |
| ERP-Analytics | 10 | 90% (Dec) | 15% (Jan-Feb) | 4 |
| ERP-Procurement | 4 | 72% (Dec) | 30% (Jan-Feb) | 2 |

3. Total estimated monthly savings: $12,400.

### Recommendation Details

4. Each recommendation includes:
   - Historical utilization chart showing the seasonal drop-off.
   - Confidence score (0.89 for all four, high confidence).
   - Risk assessment: Low (utilization consistently below 35% for 6 weeks).
   - Rollback plan: Auto-scale back up if utilization exceeds 70%.

### Outcome

| Metric | Value |
|--------|-------|
| Recommendations Generated | 4 |
| Estimated Monthly Savings | $12,400 |
| Actual Monthly Savings (after apply) | $11,800 |
| Performance Impact | None (zero incidents post-scaling) |

---

## Use Case 7: Security Vulnerability Detection in ERP Containers

### Scenario

A critical CVE is published affecting a widely-used Go standard library component present in multiple ERP module container images.

### Detection

1. AIOps security scanner updates its vulnerability database within 4 hours of CVE publication.
2. Automated image scanning identifies the vulnerable component in 14 out of 20 ERP module images.
3. 14 security findings are created, each classified as **Critical** (CVSS 9.8).

### Triage and Response

4. The security findings are grouped by CVE and presented as a single campaign: **"CVE-2026-XXXX: Go stdlib vulnerability affecting 14 modules"**.
5. For each affected module, the finding includes:
   - Exact image tag and layer containing the vulnerable package.
   - Specific version installed vs. patched version.
   - Remediation: Update Go base image to version X.Y.Z.

### Automated Remediation

6. A security remediation playbook triggers:
   - Rebuild all 14 images with the patched base image.
   - Run integration tests on each rebuilt image.
   - Deploy patched images using canary rollout (10% -> 50% -> 100%).
7. Manual approval required for production deployment.

### Outcome

| Metric | Without AIOps | With AIOps |
|--------|--------------|------------|
| Time to Detect | 1-7 days (manual scan schedule) | 4 hours |
| Time to Identify Scope | 2-4 hours (manual audit) | Immediate (automated scan) |
| Time to Remediate | 2-5 days | 8 hours (including approval gates) |
| Modules Left Vulnerable | Unknown for days | Zero after 8 hours |

---

## Use Case 8: GitHub Integration for Incident Tracking

### Scenario

The platform team uses GitHub Issues for tracking operational work and wants incidents to automatically sync between AIOps and GitHub.

### Configuration

1. An administrator configures the GitHub integration in **Settings** > **Integrations** > **GitHub**.
2. Configuration includes:
   - Repository: `org/erp-operations`
   - Auto-create issues for: P1 and P2 incidents
   - Label mapping: P1 -> `critical`, P2 -> `high-priority`, `aiops-incident`
   - Assignee mapping: On-call engineer -> GitHub username

### Workflow

3. When a P1 incident is created in AIOps:
   - A GitHub issue is automatically created with the incident title, severity, affected services, and a link back to the AIOps incident.
   - The issue body includes the initial RCA summary if available.
   - Labels are applied per the mapping.
4. Bidirectional sync:
   - Comments added in GitHub appear in the AIOps incident Activity tab.
   - Status changes in AIOps update the GitHub issue (e.g., Resolved closes the issue).
   - Labels added in GitHub are reflected in AIOps tags.
5. When the incident is resolved in AIOps:
   - The GitHub issue is closed with a resolution comment.
   - A post-mortem link is added to the issue.

### Outcome

| Benefit | Description |
|---------|-------------|
| Single Source of Truth | Engineers work in their preferred tool without losing context |
| Audit Trail | Full incident history preserved across both platforms |
| Automation | No manual issue creation or status synchronization |
| Reporting | GitHub project boards can track AIOps incidents alongside development work |
