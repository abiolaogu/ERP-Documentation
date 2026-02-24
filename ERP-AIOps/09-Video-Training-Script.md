# ERP-AIOps Video Training Scripts

> **Document ID:** ERP-AIOPS-VIDEO-009
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Audience:** Training Content Creators, New Users, Onboarding Teams

---

## Episode 1: AIOps Overview (5 minutes)

### Scene 1: Introduction (0:00 - 0:45)

**Visual:** Animated OpenSASE ERP logo transitions into AIOps dashboard.

**Narration:**
"Welcome to ERP-AIOps, the AI-powered operations platform for the OpenSASE ERP suite. In this episode, we will explore what AIOps is, why it matters, and how it transforms the way your team manages 20+ ERP modules."

**Key Screenshot:** AIOps Command Center dashboard with health score prominently displayed.

### Scene 2: The Problem AIOps Solves (0:45 - 2:00)

**Visual:** Split screen showing a traditional NOC with hundreds of alerts vs. AIOps consolidated view.

**Narration:**
"Traditional operations teams face alert fatigue, siloed monitoring, and reactive firefighting. With 20+ ERP modules generating thousands of events per minute, manual correlation is impossible. AIOps applies machine learning to detect anomalies, correlate events across services, identify root causes, and even trigger automated remediation, all in sub-second timeframes."

**Key Screenshot:** Side-by-side comparison: raw alert stream vs. AIOps correlated incident view.

### Scene 3: Architecture at a Glance (2:00 - 3:30)

**Visual:** Animated architecture diagram showing three tiers: Go Gateway, Rust API Core, Python AI Brain.

**Narration:**
"The platform is built on three tiers. The Go gateway handles authentication and routing. The Rust API core provides high-throughput event processing, incident management, and rule evaluation. The Python AI brain powers anomaly detection with algorithms like Isolation Forest and LSTM neural networks, plus LLM-based root cause analysis. Data flows from all ERP modules through OpenTelemetry into the AIOps ingestion pipeline."

**Key Screenshot:** Mermaid architecture diagram from the technical documentation.

### Scene 4: Key Features Tour (3:30 - 4:30)

**Visual:** Quick montage of each major feature area in the UI.

**Narration:**
"Let us quickly tour the key features. The Command Center gives you a unified health overview. The Incidents page manages the full lifecycle from detection to resolution. Anomalies shows ML-detected deviations with confidence scores. The Topology view maps service dependencies. Rules let you define detection and correlation logic. Remediation automates recovery actions. The Cost Center identifies optimization opportunities. And Security continuously scans for vulnerabilities."

**Key Screenshot:** Sidebar navigation with each section highlighted in sequence.

### Scene 5: Closing (4:30 - 5:00)

**Visual:** Return to Command Center dashboard.

**Narration:**
"In the next episode, we will dive deep into anomaly detection, the core intelligence engine of AIOps. See you there."

---

## Episode 2: Anomaly Detection Deep Dive (10 minutes)

### Scene 1: Introduction (0:00 - 0:30)

**Visual:** Anomaly list page with several active anomalies.

**Narration:**
"Welcome back. In this episode, we take a deep dive into anomaly detection, the foundation of AIOps intelligence. You will learn how anomalies are detected, scored, and surfaced for your review."

### Scene 2: The Detection Pipeline (0:30 - 2:30)

**Visual:** Animated pipeline diagram: Metrics Ingestion, Feature Extraction, Algorithm Ensemble, Score Fusion, Alert/Suppress.

**Narration:**
"Every metric flowing into AIOps passes through a five-stage pipeline. First, raw metrics are ingested from the OpenTelemetry Collector federation at up to 100,000 events per second. Second, features are extracted, including rolling averages, standard deviations, rate of change, and seasonality components. Third, multiple detection algorithms run in parallel: Z-Score for Gaussian metrics, IQR for skewed distributions, Moving Average for trend deviations, Isolation Forest for multi-dimensional patterns, and LSTM networks for complex temporal sequences. Fourth, scores from all algorithms are fused into a single anomaly score using a weighted ensemble. Finally, the score is compared against the adaptive threshold to decide whether to alert or suppress."

**Key Screenshot:** Pipeline diagram with each stage labeled.

### Scene 3: Understanding Algorithms (2:30 - 5:00)

**Visual:** Each algorithm visualized with example charts.

**Narration:**
"Let us look at each algorithm. Z-Score calculates how many standard deviations a data point is from the mean. It works best for metrics like CPU utilization that follow a normal distribution. IQR, or Interquartile Range, uses the spread between the 25th and 75th percentiles. It is more robust to outliers, making it ideal for latency metrics. Moving Average compares the current value against a rolling window average, great for detecting trend shifts in throughput. Isolation Forest is a tree-based algorithm that isolates anomalous points by randomly splitting feature space. It excels at finding multi-dimensional anomalies that single-metric methods miss. Finally, LSTM neural networks learn long-term temporal patterns, detecting anomalies in complex, seasonal workloads."

**Key Screenshot:** Chart showing each algorithm's detection boundary overlaid on example metric data.

### Scene 4: Anomaly Scores and Thresholds (5:00 - 7:00)

**Visual:** Anomaly detail page with score sparkline and threshold bands.

**Narration:**
"Every anomaly receives a score from 0.0 to 1.0. Scores below 0.3 are considered noise and suppressed by default. Scores between 0.3 and 0.6 warrant review. Scores above 0.6 are high-confidence anomalies, and scores above 0.8 are critical. The adaptive threshold engine continuously adjusts these boundaries based on historical patterns. If a metric is naturally volatile, the thresholds widen to reduce false positives. If a metric is normally stable, thresholds tighten to catch subtle deviations."

**Key Screenshot:** Anomaly detail view showing score sparkline, confidence bands, and contributing factors.

### Scene 5: Reviewing Anomalies in the UI (7:00 - 9:00)

**Visual:** Live walkthrough of the Anomalies page.

**Narration:**
"Let us walk through the Anomalies page. The list view shows all detected anomalies with their score, affected metric, service, and detection time. You can filter by severity, service, or time range. Clicking an anomaly opens the detail view. Here you see the metric chart with the predicted baseline and confidence bands overlaid. The score sparkline shows how the anomaly evolved. The contributing factors panel explains which features drove the detection. Related anomalies on the same or connected services are linked. And the recommended actions panel suggests next steps based on AI analysis."

**Key Screenshot:** Anomaly detail page with all panels visible.

### Scene 6: Adjusting Sensitivity (9:00 - 9:45)

**Visual:** Threshold settings page with sensitivity slider.

**Narration:**
"If you are receiving too many false positives for a specific metric, navigate to Anomalies, then Threshold Settings. Select the metric category and adjust the sensitivity slider. Moving it left reduces sensitivity, resulting in fewer alerts. Moving it right increases sensitivity. Changes take effect within 60 seconds."

**Key Screenshot:** Threshold settings page with slider highlighted.

### Scene 7: Closing (9:45 - 10:00)

**Visual:** Return to Anomaly list.

**Narration:**
"Now you understand how AIOps detects and scores anomalies. Next, we will cover incident management workflows."

---

## Episode 3: Incident Management Workflow (8 minutes)

### Scene 1: Introduction (0:00 - 0:30)

**Visual:** Incident list page with active incidents of varying severity.

**Narration:**
"In this episode, we cover the full incident management lifecycle in AIOps, from automated detection through investigation, mitigation, and resolution."

### Scene 2: Incident Creation (0:30 - 2:00)

**Visual:** Diagram showing three paths to incident creation.

**Narration:**
"Incidents are created in three ways. First, automatically when an anomaly or rule triggers an incident. The system sets the severity, assigns to the on-call engineer, and populates the initial timeline. Second, through event correlation, where multiple related events are grouped into a single incident, reducing noise. Third, manually, when an engineer observes an issue not yet detected by the system."

**Key Screenshot:** Incident creation form and auto-created incident comparison.

### Scene 3: Incident Lifecycle (2:00 - 3:30)

**Visual:** State machine diagram: Detected, Acknowledged, Investigating, Mitigating, Resolved, Closed.

**Narration:**
"Each incident progresses through defined states. Detected means the system created the incident but no human has acknowledged it. Acknowledged means an engineer has accepted ownership. Investigating means active troubleshooting is underway. Mitigating means a fix is being applied. Resolved means the issue is fixed and under observation. Closed means the incident is complete with a post-mortem documented. SLA timers track how long each transition takes, ensuring your team meets response and resolution targets."

**Key Screenshot:** Incident detail page showing state transition buttons.

### Scene 4: Investigation with RCA (3:30 - 5:30)

**Visual:** RCA results page with causal chain visualization.

**Narration:**
"The Root Cause Analysis tab is where AIOps truly shines. For P1 and P2 incidents, RCA runs automatically. The AI brain collects evidence from three sources: metrics from VictoriaMetrics, logs from Quickwit, and traces from the telemetry pipeline. It then applies causal inference algorithms to build a causal chain, showing which event likely caused which downstream effect. The LLM summarizes findings in plain English, and similar past incidents are surfaced for comparison."

**Key Screenshot:** RCA page with causal chain diagram, contributing factors, and similar incidents panel.

### Scene 5: Collaboration Features (5:30 - 6:30)

**Visual:** Activity tab with comments and status updates.

**Narration:**
"The Activity tab serves as the collaboration hub. Engineers can post notes, attach evidence like screenshots or log snippets, and tag team members. Integration with Slack and Teams means updates posted in AIOps appear in your incident channel and vice versa. All activity is timestamped and attributed, creating a full audit trail for post-incident review."

**Key Screenshot:** Activity tab with threaded comments.

### Scene 6: Resolution and Post-Mortem (6:30 - 7:30)

**Visual:** Resolution dialog and post-mortem template.

**Narration:**
"When the issue is fixed, click Resolve and fill in the resolution summary. Categorize the root cause: was it a configuration change, a code bug, infrastructure failure, or an external dependency? After the cool-down period, the incident auto-closes. A post-mortem template is generated from the incident timeline, RCA results, and resolution notes, giving you a head start on your review."

**Key Screenshot:** Resolution dialog and generated post-mortem template.

### Scene 7: Closing (7:30 - 8:00)

**Visual:** Incident list showing a mix of resolved and active incidents.

**Narration:**
"You now know how to manage incidents end-to-end in AIOps. In the next episode, we explore the service topology and dependency mapping."

---

## Episode 4: Service Topology and Dependencies (7 minutes)

### Scene 1: Introduction (0:00 - 0:30)

**Visual:** Full topology graph view with all ERP modules.

**Narration:**
"Understanding how your ERP modules connect and depend on each other is critical for fast incident response. In this episode, we explore the service topology view."

### Scene 2: Topology Discovery (0:30 - 2:00)

**Visual:** Animated diagram showing OTel traces being assembled into a topology graph.

**Narration:**
"AIOps builds the topology automatically from OpenTelemetry trace data. Every inter-service call, whether HTTP, gRPC, or message queue, creates an edge in the dependency graph. The system discovers new services and connections without manual configuration, keeping the map always up to date."

**Key Screenshot:** Topology graph with service nodes and labeled edges.

### Scene 3: Navigating the Graph (2:00 - 3:30)

**Visual:** Live UI walkthrough of zoom, pan, select, and filter operations.

**Narration:**
"Use the scroll wheel to zoom, click and drag to pan, and click a node to select it. The sidebar filter lets you show only specific module groups or health states. Selected nodes display a summary card with health status, active incidents, recent anomalies, and key metrics like request rate and error rate."

**Key Screenshot:** Topology view with a selected node showing the summary card.

### Scene 4: Health Visualization (3:30 - 4:30)

**Visual:** Topology with nodes in different health states (green, yellow, orange, red).

**Narration:**
"Node colors indicate health. Green means healthy with no active issues. Yellow indicates warning-level anomalies. Orange means degraded with P3 or P4 incidents. Red signals critical issues with P1 or P2 incidents. Gray means no telemetry is being received, which itself may indicate a problem."

**Key Screenshot:** Topology showing a cascade failure with multiple red and orange nodes.

### Scene 5: Blast Radius Analysis (4:30 - 6:00)

**Visual:** Right-click context menu and blast radius highlight.

**Narration:**
"One of the most powerful features is blast radius analysis. Right-click any service node and select Analyze Blast Radius. The system traces all downstream dependencies and highlights them. A summary panel shows the estimated user impact percentage based on traffic patterns. This helps you prioritize which failures to address first and understand the potential scope of an outage before it cascades."

**Key Screenshot:** Blast radius visualization with downstream services highlighted and impact summary.

### Scene 6: Topology in Incident Context (6:00 - 6:40)

**Visual:** Incident detail page with Topology Impact tab.

**Narration:**
"Within any incident, the Topology Impact tab shows the same graph but focused on the affected services. Impacted nodes are highlighted and the causal path from RCA is overlaid on the graph, making it immediately clear how the failure propagated."

**Key Screenshot:** Incident topology tab with causal path overlay.

### Scene 7: Closing (6:40 - 7:00)

**Visual:** Full topology view.

**Narration:**
"The topology view turns complex dependencies into an intuitive visual map. Next, we will learn how to configure auto-remediation."

---

## Episode 5: Auto-Remediation Setup (10 minutes)

### Scene 1: Introduction (0:00 - 0:30)

**Visual:** Remediation section landing page.

**Narration:**
"Auto-remediation is the capstone of AIOps intelligence, taking action on detected issues without waiting for human intervention. In this episode, we cover how to set up, configure, and monitor automated remediation."

### Scene 2: Remediation Concepts (0:30 - 2:30)

**Visual:** Diagram showing: Trigger, Playbook, Approval, Execution, Verification.

**Narration:**
"Remediation follows a five-step flow. A trigger activates the playbook, this could be an incident of a certain severity, a specific anomaly type, or a rule match. The playbook defines the steps to execute. The approval policy decides whether execution is automatic or requires human sign-off. Execution runs the action against the target service. Verification checks whether the action resolved the issue."

**Key Screenshot:** Remediation flow diagram.

### Scene 3: Supported Actions (2:30 - 4:00)

**Visual:** Action catalog table.

**Narration:**
"The action catalog includes: Restart Service to bounce a failing pod, Scale Out to add replicas under load, Scale In to remove excess capacity, Rollback Deployment to revert to the last known-good version, Clear Cache to flush DragonflyDB, DNS Failover to reroute traffic, and Config Rollback to restore a previous configuration. Each action has an assigned risk level. Low-risk actions like scaling are safe to auto-approve. High-risk actions like rollback should typically require manual approval."

**Key Screenshot:** Action catalog with risk levels.

### Scene 4: Creating a Playbook (4:00 - 7:00)

**Visual:** Step-by-step walkthrough of the playbook creation form.

**Narration:**
"Let us create a playbook. Navigate to Remediation, then Playbooks, then Create. Name it, for example, Auto-Scale on CPU Spike. Set the trigger to: incident severity P2 or higher, anomaly type CPU, affected service any. Add the first step: Scale Out with a target replica count of current plus two. Add a verification step: check CPU utilization drops below 80% within 5 minutes. Set the approval policy to Auto-approve during business hours and Manual approve off-hours. Finally, save and activate the playbook."

**Key Screenshot:** Completed playbook creation form.

### Scene 5: Approval Workflows (7:00 - 8:30)

**Visual:** Slack notification showing approval request and dashboard approval queue.

**Narration:**
"When a playbook requires manual approval, notifications are sent to the configured channels, Slack, Teams, or the AIOps dashboard. Approvers see the trigger details, proposed action, risk assessment, and estimated impact. They can approve, reject, or modify the action directly from the notification. Approval timeouts can be configured: if no one responds within the timeout, the action either auto-approves or auto-rejects based on your policy."

**Key Screenshot:** Slack approval notification and dashboard approval queue.

### Scene 6: Monitoring Remediation Activity (8:30 - 9:30)

**Visual:** Remediation Activity Log page.

**Narration:**
"The Activity Log shows all remediation executions. Each entry includes the playbook name, trigger event, action taken, execution status, duration, and verification result. Click any entry for the full execution log. Use filters to view only failed executions, pending approvals, or actions on a specific service."

**Key Screenshot:** Activity Log with mixed statuses.

### Scene 7: Closing (9:30 - 10:00)

**Visual:** Return to Remediation landing page.

**Narration:**
"Auto-remediation dramatically reduces MTTR by taking action within seconds of detection. In the next episode, we cover cost optimization."

---

## Episode 6: Cost Optimization (8 minutes)

### Scene 1: Introduction (0:00 - 0:30)

**Visual:** Cost Center dashboard.

**Narration:**
"AIOps does not just detect problems, it also identifies ways to save money. In this episode, we explore the Cost Center and how to act on optimization recommendations."

### Scene 2: Cost Dashboard Overview (0:30 - 2:00)

**Visual:** Walkthrough of Cost Center dashboard components.

**Narration:**
"The Cost Center dashboard shows four key panels. Total Monthly Cost aggregates spend across all ERP modules. The Cost Trend chart tracks spending over the last 30, 60, or 90 days. Top Cost Drivers ranks services by monthly spend. And Optimization Opportunities lists actionable recommendations with estimated savings."

**Key Screenshot:** Full Cost Center dashboard.

### Scene 3: How Cost Analysis Works (2:00 - 4:00)

**Visual:** Data flow diagram: resource utilization metrics, pricing data, ML analysis, recommendations.

**Narration:**
"The cost analysis engine combines resource utilization metrics from the telemetry pipeline with pricing data for your infrastructure. Machine learning models analyze utilization patterns to identify right-sizing opportunities, where resources are over-provisioned relative to actual usage. The system also detects idle resources, unused reserved capacity, and storage that could be moved to cheaper tiers."

**Key Screenshot:** Right-sizing recommendation detail with utilization chart.

### Scene 4: Reviewing Recommendations (4:00 - 5:30)

**Visual:** Recommendation list and detail view.

**Narration:**
"Each recommendation includes the category, affected resource, current cost, projected savings, confidence level, and risk assessment. Click a recommendation to see the full analysis, including historical utilization charts that justify the suggestion. Recommendations are refreshed daily as new utilization data comes in."

**Key Screenshot:** Recommendation detail with utilization chart and savings projection.

### Scene 5: Applying Recommendations (5:30 - 7:00)

**Visual:** Apply workflow with confirmation dialog and tracking view.

**Narration:**
"To apply a recommendation, click Apply and confirm in the dialog. The system executes the change, whether it is resizing a pod, adjusting replica counts, or migrating storage. After applying, the system tracks actual savings versus projected savings and reports the results in the Savings Tracker panel. If a change causes performance degradation, the system automatically flags it and offers a rollback."

**Key Screenshot:** Apply confirmation dialog and Savings Tracker panel.

### Scene 6: Cost Alerts (7:00 - 7:40)

**Visual:** Cost alert configuration page.

**Narration:**
"You can also configure cost alerts to be notified when spending exceeds a threshold. Set budget limits per service or per module group, and receive alerts at 80%, 90%, and 100% of the budget."

**Key Screenshot:** Cost alert configuration form.

### Scene 7: Closing (7:40 - 8:00)

**Visual:** Cost Center dashboard showing savings achieved.

**Narration:**
"The Cost Center helps you operate efficiently without sacrificing reliability. In our final episode, we cover security scanning."

---

## Episode 7: Security Scanning (7 minutes)

### Scene 1: Introduction (0:00 - 0:30)

**Visual:** Security section landing page.

**Narration:**
"Security is integral to operations. In this final episode, we explore how AIOps continuously scans your ERP environment for vulnerabilities and misconfigurations."

### Scene 2: What Gets Scanned (0:30 - 2:00)

**Visual:** Diagram showing scan targets: container images, dependencies, configurations, runtime.

**Narration:**
"AIOps security scanning covers four areas. Container images are scanned for known CVEs using vulnerability databases. Application dependencies are checked for outdated or vulnerable packages. Configuration files are analyzed against security best practices and CIS benchmarks. Runtime behavior is monitored for anomalous patterns that could indicate compromise."

**Key Screenshot:** Security overview dashboard with scan coverage metrics.

### Scene 3: Understanding Findings (2:00 - 3:30)

**Visual:** Findings list with severity badges.

**Narration:**
"Findings are classified by severity: Critical, High, Medium, and Low. Each finding includes the CVE ID if applicable, affected component, a description of the vulnerability, the CVSS score, remediation guidance with specific steps to fix, and evidence from the scan. SLAs define how quickly each severity must be addressed: Critical within 24 hours, High within 7 days, Medium within 30 days, and Low within 90 days."

**Key Screenshot:** Finding detail page with all fields visible.

### Scene 4: Triaging Findings (3:30 - 5:00)

**Visual:** Triage workflow in the UI.

**Narration:**
"When reviewing findings, you have four options. Create Ticket sends the finding to your issue tracker, such as GitHub, for remediation. Accept Risk acknowledges the finding but defers action, requiring a justification and expiry date. Mark False Positive flags the finding as incorrect, removing it from active counts. Remediate triggers an automated fix if one is available. Triage status is tracked and audited."

**Key Screenshot:** Triage action menu and accept risk dialog.

### Scene 5: Security Posture Over Time (5:00 - 6:00)

**Visual:** Security posture trend chart.

**Narration:**
"The Security Posture panel tracks your finding count over time by severity. A downward trend in critical and high findings indicates improving security. The Mean Time to Remediate metric shows how quickly your team addresses findings. Use this data in compliance reporting and security reviews."

**Key Screenshot:** Security posture trend chart with MTTR metric.

### Scene 6: Integration with Remediation (6:00 - 6:40)

**Visual:** Auto-remediation playbook triggered by critical security finding.

**Narration:**
"For critical findings with known fixes, AIOps can trigger auto-remediation. For example, if a critical CVE is detected in a container image, a playbook can automatically rebuild the image with the patched base and redeploy, all with appropriate approval gates."

**Key Screenshot:** Security-triggered remediation in the Activity Log.

### Scene 7: Closing (6:40 - 7:00)

**Visual:** Return to AIOps Command Center.

**Narration:**
"That concludes our video training series. You now have a comprehensive understanding of ERP-AIOps, from anomaly detection through incident management, topology mapping, auto-remediation, cost optimization, and security scanning. Thank you for watching."

---

## Production Notes

| Episode | Duration | Slides | Screen Recordings | Animations |
|---------|----------|--------|-------------------|------------|
| 1: AIOps Overview | 5 min | 3 | 2 | 2 |
| 2: Anomaly Detection | 10 min | 4 | 3 | 3 |
| 3: Incident Management | 8 min | 3 | 3 | 2 |
| 4: Service Topology | 7 min | 2 | 4 | 2 |
| 5: Auto-Remediation | 10 min | 3 | 4 | 2 |
| 6: Cost Optimization | 8 min | 2 | 3 | 1 |
| 7: Security Scanning | 7 min | 2 | 3 | 2 |
