# ERP-AIOps User Manual

## 1. Introduction

Welcome to the ERP-AIOps User Manual. This guide covers all features of the AIOps platform, from managing incidents and reviewing anomalies to configuring automation rules, viewing service topology, triggering remediation, analyzing costs, and reviewing security findings. ERP-AIOps is designed to bring AI-powered intelligence to the operations of the entire OpenSASE ERP suite.

### 1.1 Accessing ERP-AIOps

1. Navigate to the ERP-AIOps URL provided by your administrator (default: `https://aiops.yourdomain.com`).
2. Log in using your ERP-IAM credentials (SSO is supported).
3. Upon login, you will see the **AIOps Command Center** dashboard tailored to your role.

### 1.2 Navigation Overview

The left sidebar contains the following navigation sections:

| Section | Icon | Description |
|---------|------|-------------|
| Command Center | Dashboard | Unified operational overview |
| Incidents | Alert Bell | Incident management |
| Anomalies | Chart | Anomaly detection and review |
| Topology | Network Graph | Service dependency mapping |
| Rules | Gear | Detection and correlation rules |
| Remediation | Wrench | Automation playbooks and actions |
| Cost Center | Dollar | Cost analysis and optimization |
| Security | Shield | Security findings and compliance |
| Forecasts | Trend Line | Metric forecasting and capacity |
| Settings | Cog | Platform configuration |

---

## 2. AIOps Command Center

The Command Center is your primary operational dashboard, providing a unified view of operational health across all ERP modules.

### 2.1 Dashboard Components

**Health Score Panel** - Displays the overall operational health score (0-100) for the entire ERP ecosystem, calculated from active incident severity, anomaly count, service health status, and SLA compliance.

**Active Incidents Feed** - Real-time feed of active incidents, sorted by severity. Each card shows the incident title, severity badge (P1-P5), affected services, time since detection, and current assignee. Click any incident to navigate to its detail page.

**Anomaly Ticker** - Scrolling ticker of newly detected anomalies with their scores. High-scoring anomalies (>80) are highlighted in red.

**Module Health Grid** - Grid view of all 20+ ERP modules showing their current health status (healthy, degraded, critical, unknown). Click any module to see its detailed metrics and active incidents.

**Key Metrics** - Four key metric cards:
- **MTTR (Last 7 Days)**: Average time from incident detection to resolution.
- **Active Incidents**: Count of currently open incidents by severity.
- **Auto-Remediation Rate**: Percentage of incidents auto-resolved this month.
- **Noise Reduction**: Alert-to-incident compression ratio.

**Topology Mini-Map** - Simplified service dependency graph with health overlay. Red nodes indicate services with active incidents. Click to expand to full topology view.

### 2.2 Customizing the Command Center

1. Click the **Customize** button (top right).
2. Drag and drop dashboard widgets to rearrange layout.
3. Add or remove widgets from the widget catalog.
4. Save your layout. The layout is saved per-user and persists across sessions.
5. Use the **Time Range** selector (top right) to adjust the default time window (1h, 6h, 24h, 7d, 30d).

---

## 3. Managing Incidents

### 3.1 Incident List

Navigate to **Incidents** in the left sidebar. The incident list shows all incidents for your tenant, with the following default columns:

| Column | Description |
|--------|-------------|
| Severity | P1 (red), P2 (orange), P3 (yellow), P4 (blue), P5 (gray) badge |
| Title | Incident title (auto-generated or manual) |
| Status | Open, Investigating, Mitigating, Resolved, Closed |
| Source | Module or service that triggered the incident |
| Assignee | Currently assigned team or individual |
| Detected At | When the incident was first detected |
| Duration | Time since detection (or total duration if resolved) |

**Filtering**: Use the filter bar to filter by severity, status, module, assignee, tag, or date range. Filters can be combined and saved as named views.

**Sorting**: Click any column header to sort. Default sort is by severity (descending) then detection time (descending).

**Bulk Actions**: Select multiple incidents to perform bulk actions: assign, change severity, add tag, or close.

### 3.2 Incident Detail View

Click any incident to open the detail view. The detail page has four tabs:

**Overview Tab**
- Incident title, description, and severity classification
- Current status with state transition buttons (Acknowledge, Investigate, Mitigate, Resolve, Close)
- Assigned team and individual
- Tags and metadata
- RCA summary (if available) with confidence score and root cause identification
- Recommended remediation actions with one-click execution

**Timeline Tab**
- Chronological list of all events associated with this incident:
  - Original detection events (anomalies, rule triggers)
  - State transitions with actor and timestamp
  - Correlation events (newly correlated alerts added to incident)
  - RCA analysis results
  - Remediation actions (attempted, succeeded, failed, rolled back)
  - Comments and annotations
- Each timeline entry is expandable for full details

**Correlated Events Tab**
- Table of all individual events (alerts, anomalies, log entries) that were correlated into this incident
- Correlation reason for each event (temporal, topological, label-based, causal)
- Click any event to see its raw data
- Ability to manually add or remove events from the correlation group

**Evidence Tab**
- Metric charts for affected services during the incident window
- Relevant log entries
- Trace spans crossing affected services
- Topology snapshot at time of incident
- Attached screenshots and files

### 3.3 Creating Incidents Manually

1. Click **Create Incident** (top right of incident list).
2. Fill in the form:
   - **Title** (required): Brief description of the issue
   - **Description**: Detailed description, accepts Markdown
   - **Severity**: Select P1-P5
   - **Source**: Select the affected module or service
   - **Assignee**: Select team or individual
   - **Tags**: Add relevant tags
3. Click **Create**. The incident is created with status "Open".

### 3.4 Incident State Transitions

| Current State | Available Transitions | Description |
|--------------|----------------------|-------------|
| Open | Acknowledge, Investigate | New incident, not yet acknowledged |
| Investigating | Mitigate, Resolve | Team is actively investigating |
| Mitigating | Resolve | Remediation in progress |
| Resolved | Close, Reopen | Issue has been resolved |
| Closed | Reopen | Incident record finalized |

Each state transition requires a comment explaining the action taken.

### 3.5 Post-Incident Review

After resolving an incident, initiate a post-incident review:

1. Navigate to the resolved incident.
2. Click **Start Post-Incident Review**.
3. Complete the review template:
   - **Impact Summary**: What was affected and for how long
   - **Root Cause**: Confirmed root cause (may differ from AI-suggested)
   - **Timeline of Actions**: Key events and decisions during response
   - **What Went Well**: Effective response actions
   - **What Could Be Improved**: Areas for improvement
   - **Action Items**: Concrete follow-up tasks with owners and due dates
4. Click **Publish Review**. The review is attached to the incident and visible to all stakeholders.

---

## 4. Reviewing Anomalies

### 4.1 Anomaly Explorer

Navigate to **Anomalies** in the left sidebar. The anomaly explorer shows detected anomalies with interactive time series charts.

**List View**: Table of anomalies with columns for metric name, anomaly score, expected value, actual value, deviation, detector type, status, and detection time.

**Chart View**: Toggle to chart view to see anomaly markers overlaid on time series charts.

**Filtering**: Filter by metric name, score range, detector type (Z-score, Isolation Forest, LSTM), status (active, acknowledged, false positive), and date range.

### 4.2 Anomaly Detail

Click any anomaly to see its detail view:

**Time Series Chart**: Interactive chart showing the metric's actual values (solid line), expected baseline (dashed line), and adaptive threshold bands (shaded area). The anomaly period is highlighted in red.

**Anomaly Metadata**:
- **Score**: 0-100 composite score
- **Type**: Point, Contextual, or Collective anomaly
- **Detector**: Which algorithm detected it (Z-score, Isolation Forest, LSTM)
- **Deviation**: Magnitude of deviation from expected value
- **Baseline Window**: Training period used for baseline computation
- **Associated Incident**: Link to incident if the anomaly was correlated

**Actions**:
- **Acknowledge**: Mark as reviewed (removes from active feed)
- **Mark False Positive**: Provide feedback to improve model accuracy
- **Create Incident**: Manually create an incident from this anomaly
- **View Baseline**: See the full baseline computation details

### 4.3 Managing Baselines

Navigate to **Anomalies > Baselines** to view and manage adaptive threshold baselines:

1. **Baseline List**: Table of all metrics with computed baselines, showing training window, last updated, and model type.
2. **Baseline Detail**: Click to see the decomposed baseline (trend, daily cycle, weekly cycle, seasonal component).
3. **Override Baseline**: Manually set a temporary baseline override (useful during planned maintenance or known seasonal events).
4. **Reset Baseline**: Force recomputation of a baseline from scratch.
5. **Exclude Period**: Mark a period to exclude from baseline training (e.g., a known outage that should not influence normal behavior).

---

## 5. Configuring Rules

### 5.1 Detection Rules

Navigate to **Rules > Detection Rules**. Detection rules define conditions that trigger alerts or anomaly evaluations.

**Creating a Detection Rule**:
1. Click **Create Rule**.
2. Configure the rule:
   - **Name**: Descriptive rule name
   - **Type**: Static Threshold, Rate of Change, Absence, or Composite
   - **Metric**: Select the metric to evaluate (supports PromQL-style selectors)
   - **Condition**: Define the trigger condition (e.g., `value > 95` for CPU threshold)
   - **Duration**: How long the condition must persist before triggering (e.g., 5 minutes)
   - **Severity**: Default severity for incidents created by this rule
   - **Labels**: Tags applied to generated alerts
3. Click **Save**. The rule is immediately active.

**Rule Types**:
| Type | Description | Example |
|------|-------------|---------|
| Static Threshold | Triggers when metric crosses a fixed value | CPU > 95% for 5 minutes |
| Rate of Change | Triggers when metric changes rate exceeds threshold | Error rate increases >200% in 10 minutes |
| Absence | Triggers when expected metric stops reporting | Heartbeat metric absent for >60 seconds |
| Composite | Triggers when multiple conditions are met simultaneously | CPU > 80% AND Memory > 85% AND Error Rate > 5% |

### 5.2 Correlation Rules

Navigate to **Rules > Correlation Rules**. Correlation rules define how events are grouped.

**Default Correlation Rules** (active out-of-box):
- Temporal: Events within 5-minute window are candidates for correlation
- Topological: Events in upstream/downstream services are correlated
- Label: Events sharing environment, region, and cluster labels are correlated

**Custom Correlation Rules**:
1. Click **Create Correlation Rule**.
2. Define the correlation criteria:
   - **Time Window**: Maximum time between events for temporal correlation (default: 5 min)
   - **Topology Depth**: How many hops in the dependency graph to consider (default: 3)
   - **Label Matching**: Which labels must match for label-based correlation
   - **Minimum Events**: Minimum number of events to form a correlation group
3. Click **Save**.

### 5.3 Remediation Rules

Navigate to **Rules > Remediation Rules**. These rules link incident patterns to remediation playbooks.

1. Click **Create Remediation Rule**.
2. Define trigger conditions (incident severity, source module, tags, RCA category).
3. Select the playbook to execute when conditions match.
4. Configure approval requirements (auto-approve, team lead, VP).
5. Set cooldown period to prevent rapid re-execution.
6. Click **Save**.

---

## 6. Viewing Service Topology

### 6.1 Topology Map

Navigate to **Topology** in the left sidebar. The interactive service dependency graph shows all discovered services and their relationships.

**Graph Controls**:
- **Zoom**: Mouse wheel or pinch to zoom in/out
- **Pan**: Click and drag background to pan
- **Select**: Click a node to select and view details
- **Multi-Select**: Hold Shift and click multiple nodes
- **Layout**: Toggle between force-directed, hierarchical, and circular layouts
- **Filter**: Filter by module, namespace, health status, or service type

**Health Overlay**: Each node is color-coded by health status:
- Green: Healthy (no active incidents or anomalies)
- Yellow: Degraded (active anomalies or P3-P5 incidents)
- Red: Critical (active P1-P2 incidents)
- Gray: Unknown (no recent telemetry)

**Edge Information**: Hover over edges to see request rate, average latency, and error rate between services.

### 6.2 Impact Analysis

1. Right-click a service node and select **Impact Analysis**.
2. The view highlights:
   - **Upstream Impact**: All services that depend on the selected service (consumers)
   - **Downstream Impact**: All services that the selected service depends on (providers)
   - **Blast Radius**: Estimated number of users/requests affected if this service goes down
3. The impact analysis considers the full transitive dependency chain, not just direct neighbors.

### 6.3 Topology Diff

1. Click the **Diff** button (top toolbar).
2. Select two time points to compare (e.g., "1 week ago" vs. "now").
3. The diff view shows:
   - **Added nodes** (new services): Green outline
   - **Removed nodes** (decommissioned services): Red outline with strikethrough
   - **Added edges** (new dependencies): Green dashed line
   - **Removed edges** (removed dependencies): Red dashed line
4. Click any diff element to see details (when added/removed, by which deployment).

---

## 7. Triggering Remediation

### 7.1 Action Catalog

Navigate to **Remediation > Action Catalog**. Browse the library of available remediation actions:

| Action | Risk Level | Target Type | Description |
|--------|-----------|-------------|-------------|
| Restart Pod | Low | Kubernetes | Restart a specific pod in a deployment |
| Scale Deployment | Medium | Kubernetes | Increase/decrease replica count |
| Flush Cache | Low | DragonflyDB | Clear cache entries for a service |
| Reset Connection Pool | Low | Database | Reset database connection pool |
| Rollback Deployment | High | Kubernetes | Rollback to previous deployment revision |
| DNS Failover | High | DNS | Switch DNS to failover endpoint |
| Traffic Shift | Medium | Load Balancer | Shift traffic percentage between endpoints |
| Execute Script | Variable | SSH/HTTP | Run custom remediation script |

### 7.2 Executing Remediation from an Incident

1. Open an incident detail page.
2. Scroll to the **Recommended Actions** section.
3. The AI has pre-selected the most appropriate remediation based on the root cause analysis.
4. Click **Execute** on the recommended action.
5. Review the pre-execution checklist:
   - Target service and environment
   - Expected impact (blast radius)
   - Rollback plan
   - Approval status
6. If approval is required, the request is sent to the appropriate approver.
7. Once approved (or auto-approved), the action executes.
8. Monitor execution progress in real-time:
   - Pre-execution health snapshot captured
   - Action executing (with live output)
   - Post-execution health check running
   - Result: Success / Failed / Rolled Back
9. The execution result is recorded on the incident timeline.

### 7.3 Managing Playbooks

Navigate to **Remediation > Playbooks**.

**Creating a Playbook**:
1. Click **Create Playbook**.
2. Name the playbook and add a description.
3. Define trigger conditions (incident severity, tags, source, RCA category).
4. Build the workflow using the visual editor:
   - Drag actions from the catalog onto the canvas
   - Connect actions to define execution order
   - Add conditions (if/else branches based on health check results)
   - Add approval gates between high-risk steps
   - Add notification steps (notify team before/after execution)
   - Add rollback steps for failure scenarios
5. Configure execution parameters:
   - **Timeout**: Maximum execution time (default: 5 minutes)
   - **Cooldown**: Minimum time between executions (default: 30 minutes)
   - **Auto-approve**: Enable auto-approval for matching severity levels
6. Save and enable the playbook.

### 7.4 Remediation History

Navigate to **Remediation > History** to view all past remediation executions:

| Column | Description |
|--------|-------------|
| Playbook | Name of the executed playbook |
| Incident | Associated incident link |
| Trigger | How it was triggered (automatic, manual, approval) |
| Status | Succeeded, Failed, Rolled Back, In Progress |
| Duration | Execution time |
| Executed By | User or "system" for auto-remediation |
| Executed At | Timestamp |

Click any execution to see the full step-by-step log with input/output for each action.

---

## 8. Analyzing Costs

### 8.1 Cost Dashboard

Navigate to **Cost Center** in the left sidebar. The cost dashboard provides comprehensive cost visibility.

**Summary Cards**:
- **Total Monthly Cost**: Current month's total infrastructure cost
- **Month-over-Month Change**: Percentage change from previous month
- **Projected Monthly Cost**: Forecasted end-of-month cost
- **Potential Savings**: Total savings from unimplemented recommendations

**Cost Breakdown Charts**:
- **By Module**: Bar chart showing cost per ERP module
- **By Resource Type**: Pie chart (Compute, Storage, Network, Database, Cache)
- **By Tenant**: Table showing cost per tenant
- **Over Time**: Line chart showing daily cost trend

### 8.2 Cost Recommendations

Navigate to **Cost Center > Recommendations**. Each recommendation card shows:

- **Title**: e.g., "Right-size ERP-Finance database from 8 vCPU to 4 vCPU"
- **Category**: Right-sizing, Idle Resource, Reserved Instance, Architecture
- **Estimated Monthly Savings**: Dollar amount
- **Risk Level**: Low, Medium, High
- **Evidence**: Charts showing utilization data supporting the recommendation
- **Implementation Steps**: Step-by-step guide to implement

Actions on each recommendation:
- **Implement**: Mark as implementing (tracks progress)
- **Dismiss**: Dismiss with reason (noise, already planned, risk too high)
- **Snooze**: Snooze for 30/60/90 days

### 8.3 Cost Anomalies

Cost anomalies appear in the Cost Center when unexpected spending patterns are detected:

- **Spike Anomalies**: Sudden cost increase (e.g., runaway query, resource misconfiguration)
- **Drift Anomalies**: Gradual unexplained cost increase over time
- **Pattern Break**: Cost behavior deviates from normal weekly/monthly patterns

Each anomaly shows the affected resource, expected cost, actual cost, deviation percentage, and probable cause.

---

## 9. Reviewing Security Findings

### 9.1 Security Dashboard

Navigate to **Security** in the left sidebar. The security dashboard provides an overview of the security posture.

**Security Score**: Overall security posture score (0-100) with breakdown by category:
- Vulnerability Management (CVE patching)
- Configuration Compliance (TLS, RBAC, network policies)
- Access Control Hygiene (unused accounts, overprivileged roles)
- Patch Currency (OS and dependency versions)

**Module Security Heatmap**: Grid showing security score for each ERP module, color-coded from green (>85) to red (<50).

### 9.2 Security Findings List

Navigate to **Security > Findings**. Table of all security findings:

| Column | Description |
|--------|-------------|
| Severity | Critical, High, Medium, Low, Informational |
| Title | Finding description |
| Category | Vulnerability, Configuration, Compliance, Threat |
| Module | Affected ERP module |
| Status | Open, In Progress, Resolved, Accepted Risk |
| First Detected | When the finding was first identified |
| Age | Days since first detection |

**Filtering**: Filter by severity, category, module, status, and age. Save filters as named views.

### 9.3 Finding Detail

Click any finding to see its detail:

- **Description**: Detailed explanation of the finding
- **Impact**: What happens if exploited or left unresolved
- **Affected Resources**: List of specific resources affected
- **Remediation Guidance**: Step-by-step instructions to resolve
- **References**: Links to CVE records, compliance framework sections
- **History**: Timeline of status changes and notes

**Actions**:
- **Assign**: Assign to a team or individual for remediation
- **Accept Risk**: Accept the risk with documented justification (requires approval)
- **Create Incident**: Create an incident for urgent findings
- **Auto-Remediate**: If a remediation playbook exists, execute it

### 9.4 Compliance Reports

Navigate to **Security > Compliance**. Generate compliance reports:

1. Select the compliance framework (SOC 2, ISO 27001, HIPAA, PCI DSS).
2. Select the scope (all modules, specific modules).
3. Click **Generate Report**.
4. The report shows:
   - Overall compliance percentage
   - Per-control compliance status (pass/fail/partial)
   - Gap analysis with specific findings per control
   - Remediation recommendations for gaps
5. Export as PDF or CSV for auditors.

---

## 10. Forecasting

### 10.1 Metric Forecasts

Navigate to **Forecasts** in the left sidebar.

1. Select a metric from the metric picker (or type a PromQL-style selector).
2. Select the forecast horizon (24 hours, 7 days, 30 days).
3. The forecast chart shows:
   - Historical values (solid line)
   - Forecasted values (dashed line)
   - Confidence interval (shaded band)
4. The forecast considers daily, weekly, and seasonal patterns.

### 10.2 Capacity Alerts

Configure capacity alerts based on forecasts:

1. Navigate to **Forecasts > Capacity Alerts**.
2. Click **Create Alert**.
3. Select the metric and threshold (e.g., "disk usage > 90%").
4. Select the forecast horizon ("alert if projected to breach within 7 days").
5. Configure notification recipients.
6. Click **Save**.

---

## 11. Settings

### 11.1 Notification Settings

Navigate to **Settings > Notifications**. Configure how you receive alerts:

| Channel | Configuration |
|---------|---------------|
| In-App | Always enabled. Real-time WebSocket notifications in the AIOps UI |
| ERP-Workspace | Configure which chat channels receive incident notifications |
| Email | Set email address and severity threshold (e.g., P1-P2 only) |
| Webhook | Configure external webhook endpoints for custom integrations |

### 11.2 Personal Preferences

Navigate to **Settings > Preferences**:
- **Theme**: Light, Dark, or System (follows OS setting)
- **Time Zone**: Display timezone for all timestamps
- **Date Format**: ISO 8601, US, EU format options
- **Default Dashboard**: Choose which dashboard loads on login
- **Notification Sound**: Enable/disable browser notification sounds

### 11.3 Tenant Settings (Admin Only)

Navigate to **Settings > Tenant Configuration** (requires admin role):
- **Detection Sensitivity**: Global sensitivity multiplier for anomaly detection (0.5 = less sensitive, 2.0 = more sensitive)
- **Correlation Window**: Default time window for event correlation
- **Auto-Remediation**: Enable/disable auto-remediation globally
- **Retention Periods**: Configure data retention per data class
- **API Keys**: Manage API keys for external integrations
- **LLM Configuration**: Configure LLM endpoint and model for RCA

---

## 12. Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `G` then `D` | Go to Command Center (Dashboard) |
| `G` then `I` | Go to Incidents |
| `G` then `A` | Go to Anomalies |
| `G` then `T` | Go to Topology |
| `G` then `R` | Go to Rules |
| `G` then `C` | Go to Cost Center |
| `G` then `S` | Go to Security |
| `/` | Focus search bar |
| `?` | Show keyboard shortcuts help |
| `Esc` | Close modal/panel |
| `J` / `K` | Next/previous item in lists |
| `Enter` | Open selected item |

---

## 13. Troubleshooting

| Issue | Solution |
|-------|----------|
| Dashboard not loading | Check network connectivity. Clear browser cache. Verify ERP-IAM session is active. |
| No anomalies detected | Ensure OTel telemetry is flowing. Check that adaptive thresholds have sufficient training data (14+ days). Verify detection rules are enabled. |
| RCA shows "Insufficient context" | Ensure the LLM endpoint is configured and accessible. Check that correlated events include logs and traces (not just metrics). |
| Topology shows stale services | Run topology refresh from Settings > Topology. Remove stale nodes manually via the topology editor. |
| Auto-remediation not triggering | Check that auto-remediation is enabled at the tenant level. Verify remediation rules match incident patterns. Check approval queue for pending approvals. |
| Cost data missing | Verify cloud API credentials are configured. Cost data may have a 1-hour delay from collection to display. |
| Security scan incomplete | Verify network access to CVE databases. Check scan schedule in Settings > Security. |

---

## 14. Getting Help

- **In-App Help**: Click the `?` icon in the top navigation bar for contextual help.
- **Documentation**: Access the full documentation at `https://docs.yourdomain.com/aiops`.
- **Support**: Contact the AIOps platform team via ERP-Workspace channel `#aiops-support`.
- **Training**: See the Training Manual for structured learning materials and video tutorials.
