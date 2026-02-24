# ERP-AIOps Training Manual

> **Document ID:** ERP-AIOPS-TRAIN-008
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Audience:** SREs, DevOps Engineers, Platform Administrators, Security Analysts

---

## 1. Getting Started with AIOps Dashboard

### 1.1 First Login

1. Navigate to your AIOps instance (default: `https://aiops.yourdomain.com`).
2. Authenticate using your ERP-IAM credentials (SSO via Authentik is supported).
3. Upon successful login, the **AIOps Command Center** loads as your landing page.

### 1.2 Navigating the Interface

The left sidebar organizes features into logical groups:

| Section | Purpose | Key Actions |
|---------|---------|-------------|
| Command Center | Unified operational overview | View health scores, active incidents, anomaly trends |
| Incidents | Incident management | Create, investigate, resolve incidents |
| Anomalies | Anomaly detection results | Review detected anomalies, adjust thresholds |
| Topology | Service dependency map | Explore module relationships, identify blast radius |
| Rules | Detection and correlation rules | Create, edit, enable/disable rules |
| Remediation | Automation playbooks | Configure and trigger auto-remediation |
| Cost Center | Cost analysis | Review optimization recommendations |
| Security | Security findings | Triage vulnerabilities, review compliance |

### 1.3 Setting Up Your Profile

1. Click your avatar in the top-right corner and select **Settings**.
2. Configure notification preferences (email, Slack, PagerDuty).
3. Set your default timezone for incident timelines.
4. Select your preferred severity filter (e.g., show P1-P3 only).

---

## 2. Understanding Anomaly Detection

### 2.1 How Anomaly Detection Works

ERP-AIOps uses a multi-algorithm approach to detect anomalies in real-time:

```
Metrics Ingestion → Feature Extraction → Algorithm Ensemble → Score Fusion → Alert/Suppress
```

**Algorithms used:**

| Algorithm | Best For | Detection Speed |
|-----------|----------|-----------------|
| Z-Score | Gaussian-distributed metrics (CPU, memory) | <10ms |
| IQR (Interquartile Range) | Skewed distributions (request latency) | <10ms |
| Moving Average | Trend-based deviations (throughput) | <15ms |
| Isolation Forest | Multi-dimensional anomalies | <50ms |
| LSTM Neural Network | Complex temporal patterns | <100ms |

### 2.2 Reading Anomaly Scores

Each anomaly is assigned a score from 0.0 to 1.0:

- **0.0 - 0.3**: Low confidence, likely noise (suppressed by default)
- **0.3 - 0.6**: Medium confidence, review recommended
- **0.6 - 0.8**: High confidence, likely a real anomaly
- **0.8 - 1.0**: Critical confidence, strong deviation from baseline

### 2.3 Anomaly Detail View

When you click on an anomaly, the detail view shows:

1. **Metric Chart** - The actual metric values plotted against the predicted baseline with confidence bands.
2. **Score Sparkline** - How the anomaly score evolved over time.
3. **Contributing Factors** - Which features contributed most to the anomaly score.
4. **Related Anomalies** - Other anomalies detected on the same service within the time window.
5. **Recommended Actions** - AI-generated suggestions for investigation or remediation.

### 2.4 Adjusting Sensitivity

To adjust anomaly detection sensitivity for a specific metric:

1. Navigate to **Anomalies** > **Threshold Settings**.
2. Select the metric category (CPU, Memory, Latency, Error Rate, etc.).
3. Adjust the sensitivity slider (Lower = fewer alerts, Higher = more alerts).
4. Click **Apply** to save. Changes take effect within 60 seconds.

---

## 3. Creating and Managing Rules

### 3.1 Rule Types

| Rule Type | Description | Example |
|-----------|-------------|---------|
| Detection Rule | Triggers when a metric condition is met | CPU > 90% for 5 minutes |
| Correlation Rule | Links related events across services | Error spike in Module A correlates with latency in Module B |
| Suppression Rule | Prevents alerts during known windows | Suppress during maintenance |
| Escalation Rule | Escalates incidents based on time or severity | Auto-escalate P1 after 15 min |

### 3.2 Creating a Detection Rule

1. Navigate to **Rules** > **Create Rule**.
2. Select rule type: **Detection**.
3. Define the condition using the visual condition builder:
   - **Metric**: Select from the metric catalog (e.g., `cpu_usage_percent`).
   - **Operator**: Choose comparison (`>`, `<`, `>=`, `<=`, `==`, `!=`).
   - **Threshold**: Enter the threshold value.
   - **Duration**: How long the condition must persist before triggering.
4. Configure the action (create incident, send notification, trigger remediation).
5. Set the severity level (P1 through P5).
6. Click **Save & Enable**.

### 3.3 Creating a Correlation Rule

1. Navigate to **Rules** > **Create Rule** > **Correlation**.
2. Define the **source event** pattern (e.g., error rate anomaly on service A).
3. Define the **correlated event** pattern (e.g., latency anomaly on service B).
4. Set the **time window** (how close in time the events must occur).
5. Optionally define **topology constraints** (events must be on connected services).
6. Define the resulting action (create correlated incident, merge into existing incident).

### 3.4 Managing Existing Rules

- **Enable/Disable**: Toggle the switch on the rules table to enable or disable a rule without deleting it.
- **Edit**: Click the rule name to open the editor.
- **Clone**: Use the context menu to duplicate a rule as a starting template.
- **Delete**: Use the context menu. Deleted rules are soft-deleted and can be recovered within 30 days.
- **Audit History**: Click **History** to see who modified the rule and when.

---

## 4. Working with Incidents and RCA

### 4.1 Incident Lifecycle

```
Detected → Acknowledged → Investigating → Mitigating → Resolved → Closed
```

### 4.2 Investigating an Incident

1. Open the incident from the **Incidents** list or Command Center.
2. Review the **Timeline** tab showing all events in chronological order.
3. Check the **Root Cause Analysis** tab for AI-generated insights:
   - **Causal Chain**: Visual representation of the likely causal sequence.
   - **Contributing Factors**: Ranked list of factors with confidence scores.
   - **Similar Past Incidents**: Historical incidents that match the current pattern.
4. Use the **Topology Impact** tab to see the blast radius on the service map.
5. Add investigation notes using the **Activity** tab.

### 4.3 Triggering Root Cause Analysis

RCA runs automatically for P1 and P2 incidents. For lower severities:

1. Open the incident detail page.
2. Click **Run RCA** in the top action bar.
3. The Python AI brain collects evidence from logs (Quickwit), metrics (VictoriaMetrics), and traces.
4. Results appear within 30-120 seconds depending on the data volume.

### 4.4 Resolving Incidents

1. After mitigation, click **Resolve** on the incident detail page.
2. Fill in the resolution summary (what was the root cause, what was done).
3. Optionally tag the resolution category (configuration change, code bug, infrastructure, external dependency).
4. The incident moves to **Resolved** state and auto-closes after the configured cool-down period (default: 24h).

---

## 5. Using Service Topology View

### 5.1 Overview

The topology view provides a graph visualization of all ERP modules and their dependencies. Nodes represent services; edges represent communication paths (HTTP, gRPC, message queue).

### 5.2 Navigating the Topology

- **Zoom**: Scroll wheel or pinch gesture to zoom in/out.
- **Pan**: Click and drag on empty space to pan.
- **Select**: Click a node to view its health status, recent anomalies, and active incidents.
- **Filter**: Use the sidebar filter to show only specific module groups or health states.

### 5.3 Health Indicators

| Node Color | Meaning |
|------------|---------|
| Green | Healthy, no active issues |
| Yellow | Warning, minor anomalies detected |
| Orange | Degraded, active P3-P4 incidents |
| Red | Critical, active P1-P2 incidents |
| Gray | Unknown, no telemetry received |

### 5.4 Blast Radius Analysis

1. Right-click a service node and select **Analyze Blast Radius**.
2. The system highlights all downstream services that would be affected if the selected service fails.
3. A summary panel shows the estimated user impact percentage.

---

## 6. Cost Optimization Recommendations

### 6.1 Accessing Cost Center

Navigate to **Cost Center** from the left sidebar. The dashboard shows:

- **Total Monthly Cost** across all monitored ERP modules.
- **Cost Trend** chart (last 30/60/90 days).
- **Top Cost Drivers** ranked by monthly spend.
- **Optimization Opportunities** with estimated savings.

### 6.2 Understanding Recommendations

Each recommendation includes:

| Field | Description |
|-------|-------------|
| Category | Right-sizing, Reserved Instances, Idle Resources, Storage Optimization |
| Affected Resource | The specific service or resource targeted |
| Current Cost | Current monthly cost |
| Projected Savings | Estimated monthly savings after optimization |
| Confidence | How confident the system is in the recommendation |
| Risk Level | Low, Medium, High - impact risk of applying the recommendation |

### 6.3 Applying Recommendations

1. Click a recommendation to view the full analysis.
2. Review the historical resource utilization charts.
3. Click **Apply** to trigger the optimization (requires appropriate RBAC permissions).
4. The system tracks the outcome and reports actual savings vs. projected savings.

---

## 7. Security Scanning Results Interpretation

### 7.1 Security Dashboard

The Security section displays findings from continuous scanning of ERP containers, configurations, and dependencies.

### 7.2 Finding Severity Levels

| Severity | Description | SLA |
|----------|-------------|-----|
| Critical | Actively exploitable, high impact | Remediate within 24 hours |
| High | Exploitable with moderate effort | Remediate within 7 days |
| Medium | Requires specific conditions to exploit | Remediate within 30 days |
| Low | Informational or best-practice deviation | Remediate within 90 days |

### 7.3 Reading a Security Finding

Each finding contains:

1. **CVE ID** (if applicable) - The Common Vulnerabilities and Exposures identifier.
2. **Affected Component** - Container image, package, or configuration file.
3. **Description** - What the vulnerability is and how it could be exploited.
4. **CVSS Score** - Numerical severity score (0.0-10.0).
5. **Remediation Guidance** - Specific steps to fix the issue (e.g., upgrade package version).
6. **Evidence** - Scan output showing where the vulnerability was found.

### 7.4 Triaging Findings

1. Review findings sorted by severity on the **Security** > **Findings** page.
2. For each finding, select an action: **Accept Risk**, **Create Ticket**, **Mark False Positive**, or **Remediate**.
3. Accepted risks require a justification and an expiry date for periodic re-review.

---

## 8. Auto-Remediation Configuration

### 8.1 Overview

Auto-remediation allows AIOps to execute predefined actions in response to detected incidents or anomalies, reducing MTTR without human intervention.

### 8.2 Supported Remediation Actions

| Action | Description | Risk Level |
|--------|-------------|------------|
| Restart Service | Restart a failing ERP module pod | Medium |
| Scale Out | Add replicas to handle increased load | Low |
| Scale In | Remove excess replicas during low traffic | Low |
| Rollback Deployment | Revert to previous known-good deployment | High |
| Clear Cache | Flush DragonflyDB cache for a specific service | Low |
| DNS Failover | Switch traffic to backup instance | High |
| Config Rollback | Restore previous configuration | Medium |

### 8.3 Creating a Remediation Playbook

1. Navigate to **Remediation** > **Playbooks** > **Create**.
2. Name the playbook and provide a description.
3. Define the **trigger condition** (incident severity, anomaly type, affected service).
4. Add **remediation steps** in sequence:
   - Select an action from the catalog.
   - Configure action parameters (target service, timeout, retry count).
   - Add optional **verification step** (check if the action resolved the issue).
5. Set the **approval policy**:
   - **Auto-approve**: Execute immediately (recommended only for low-risk actions).
   - **Manual approve**: Require human approval via Slack/Teams/Dashboard.
   - **Time-based**: Auto-approve during business hours, require manual approval off-hours.
6. Click **Save & Activate**.

### 8.4 Monitoring Remediation Activity

1. Navigate to **Remediation** > **Activity Log**.
2. Review executed actions with their status (Success, Failed, Pending Approval, Rolled Back).
3. Click any entry to see the full execution log, duration, and outcome verification result.

---

## Appendix: Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `G` then `D` | Go to Dashboard |
| `G` then `I` | Go to Incidents |
| `G` then `A` | Go to Anomalies |
| `G` then `T` | Go to Topology |
| `G` then `R` | Go to Rules |
| `/` | Open global search |
| `?` | Show keyboard shortcuts help |
| `Esc` | Close modal or panel |
