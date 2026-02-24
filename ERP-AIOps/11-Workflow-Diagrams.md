# ERP-AIOps Workflow Diagrams

> **Document ID:** ERP-AIOPS-WF-011
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [01-Technical-Writeup.md](./01-Technical-Writeup.md), [12-High-Level-Design.md](./12-High-Level-Design.md)

---

## 1. Anomaly Detection Pipeline

The anomaly detection pipeline processes incoming telemetry metrics through a multi-stage flow, from raw ingestion to alert/suppression decision.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ANOMALY DETECTION PIPELINE                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐    ┌──────────────────┐    ┌─────────────────────────┐   │
│  │ ERP Modules   │    │  OTel Collector   │    │  AIOps Ingestion        │   │
│  │ (20+ sources) │───>│  Federation       │───>│  Pipeline (Rust)        │   │
│  │              │    │                  │    │  100K events/sec        │   │
│  └──────────────┘    └──────────────────┘    └───────────┬─────────────┘   │
│                                                           │                 │
│                                                           v                 │
│                                              ┌─────────────────────────┐   │
│                                              │  Feature Extraction      │   │
│                                              │  - Rolling mean/stddev   │   │
│                                              │  - Rate of change        │   │
│                                              │  - Seasonality decomp    │   │
│                                              │  - Percentile tracking   │   │
│                                              └───────────┬─────────────┘   │
│                                                           │                 │
│                              ┌────────────────────────────┼───────────┐     │
│                              │            │               │           │     │
│                              v            v               v           v     │
│                        ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│                        │ Z-Score  │ │   IQR    │ │ Isolation│ │   LSTM   │ │
│                        │ Detector │ │ Detector │ │  Forest  │ │ Detector │ │
│                        │  <10ms   │ │  <10ms   │ │  <50ms   │ │  <100ms  │ │
│                        └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ │
│                              │            │            │            │       │
│                              v            v            v            v       │
│                        ┌─────────────────────────────────────────────────┐ │
│                        │          SCORE FUSION ENGINE                     │ │
│                        │  Weighted ensemble: w1*z + w2*iqr + w3*if +     │ │
│                        │                     w4*lstm = final_score        │ │
│                        └──────────────────────┬──────────────────────────┘ │
│                                                │                           │
│                                                v                           │
│                        ┌─────────────────────────────────────────────────┐ │
│                        │        ADAPTIVE THRESHOLD ENGINE                 │ │
│                        │  Compare final_score against dynamic threshold   │ │
│                        └────────┬───────────────────────┬────────────────┘ │
│                                 │                       │                   │
│                    score >= threshold            score < threshold          │
│                                 │                       │                   │
│                                 v                       v                   │
│                        ┌──────────────┐        ┌──────────────┐            │
│                        │  ALERT       │        │  SUPPRESS     │            │
│                        │  Create      │        │  Log for      │            │
│                        │  anomaly     │        │  model         │            │
│                        │  record      │        │  retraining    │            │
│                        └──────┬───────┘        └──────────────┘            │
│                               │                                             │
│                               v                                             │
│                   ┌───────────────────────┐                                 │
│                   │  → Correlation Engine  │                                 │
│                   │  → Rule Evaluation     │                                 │
│                   │  → Incident Creation   │                                 │
│                   │  → Notification        │                                 │
│                   └───────────────────────┘                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Incident Lifecycle

The incident state machine governs how incidents progress from initial detection through to closure.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           INCIDENT LIFECYCLE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                         ┌───────────────┐                                   │
│  Anomaly/Rule/Manual───>│   DETECTED    │                                   │
│                         │               │                                   │
│                         │  • Auto-assign│                                   │
│                         │  • Notify     │                                   │
│                         │  • Start SLA  │                                   │
│                         └───────┬───────┘                                   │
│                                 │                                           │
│                        engineer accepts                                     │
│                                 │                                           │
│                                 v                                           │
│                         ┌───────────────┐                                   │
│                         │ ACKNOWLEDGED  │                                   │
│                         │               │                                   │
│                         │  • Owner set  │                                   │
│                         │  • SLA ack    │                                   │
│                         │    timer stop │                                   │
│                         └───────┬───────┘                                   │
│                                 │                                           │
│                      begins investigation                                   │
│                                 │                                           │
│                                 v                                           │
│                         ┌───────────────┐                                   │
│                         │ INVESTIGATING │                                   │
│                         │               │                                   │
│                         │  • RCA runs   │                                   │
│                         │  • Evidence   │                                   │
│                         │    collected  │                                   │
│                         │  • Topology   │                                   │
│                         │    analyzed   │                                   │
│                         └───────┬───────┘                                   │
│                                 │                                           │
│                       fix identified                                        │
│                                 │                                           │
│                                 v                                           │
│                         ┌───────────────┐                                   │
│                         │  MITIGATING   │──── remediation fails ───┐       │
│                         │               │                          │       │
│                         │  • Action     │                          │       │
│                         │    executing  │                          v       │
│                         │  • Monitoring │               ┌──────────────┐   │
│                         │    impact     │               │ INVESTIGATING │   │
│                         └───────┬───────┘               │  (re-enter)   │   │
│                                 │                       └──────────────┘   │
│                        fix confirmed                                       │
│                                 │                                           │
│                                 v                                           │
│                         ┌───────────────┐                                   │
│                         │   RESOLVED    │                                   │
│                         │               │                                   │
│                         │  • Resolution │                                   │
│                         │    summary    │                                   │
│                         │  • Cool-down  │                                   │
│                         │    period     │                                   │
│                         │    (24h)      │                                   │
│                         └───────┬───────┘                                   │
│                                 │                                           │
│                 issue recurs during     cool-down                           │
│                 cool-down?              expires                             │
│                    │                       │                                 │
│                    v                       v                                 │
│           ┌──────────────┐        ┌───────────────┐                        │
│           │ INVESTIGATING │        │    CLOSED     │                        │
│           │  (re-open)    │        │               │                        │
│           └──────────────┘        │  • Post-mortem│                        │
│                                   │    generated  │                        │
│                                   │  • Metrics    │                        │
│                                   │    finalized  │                        │
│                                   └───────────────┘                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

**SLA Timer Summary:**

| Severity | Acknowledge | Investigate | Resolve |
|----------|-------------|-------------|---------|
| P1 | 5 min | 15 min | 1 hour |
| P2 | 15 min | 30 min | 4 hours |
| P3 | 1 hour | 4 hours | 24 hours |
| P4 | 4 hours | 24 hours | 7 days |
| P5 | 24 hours | 7 days | 30 days |

---

## 3. RCA Workflow

Root Cause Analysis collects evidence from multiple sources, correlates findings, applies causal inference, and produces an actionable report.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           RCA WORKFLOW                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                         ┌───────────────┐                                   │
│                         │  RCA Trigger  │                                   │
│                         │  (P1/P2 auto, │                                   │
│                         │  manual btn)  │                                   │
│                         └───────┬───────┘                                   │
│                                 │                                           │
│              ┌──────────────────┼──────────────────┐                       │
│              v                  v                   v                       │
│  ┌───────────────────┐ ┌──────────────────┐ ┌──────────────────┐          │
│  │  Collect Metrics   │ │  Collect Logs    │ │  Collect Traces  │          │
│  │  (VictoriaMetrics) │ │  (Quickwit)      │ │  (OTel Store)    │          │
│  │                   │ │                  │ │                  │          │
│  │  • Affected svcs  │ │  • Error logs    │ │  • Slow traces   │          │
│  │  • Time window    │ │  • Change logs   │ │  • Failed spans  │          │
│  │  • Related metrics│ │  • Deploy logs   │ │  • Cross-svc     │          │
│  └────────┬──────────┘ └────────┬─────────┘ └────────┬─────────┘          │
│           │                     │                     │                     │
│           └─────────────────────┼─────────────────────┘                     │
│                                 v                                           │
│                     ┌───────────────────────┐                               │
│                     │  Evidence Correlation  │                               │
│                     │                       │                               │
│                     │  • Temporal alignment │                               │
│                     │  • Service grouping   │                               │
│                     │  • Event sequencing   │                               │
│                     └───────────┬───────────┘                               │
│                                 │                                           │
│                                 v                                           │
│                     ┌───────────────────────┐                               │
│                     │  Causal Inference     │                               │
│                     │  (Python AI Brain)    │                               │
│                     │                       │                               │
│                     │  • Granger causality  │                               │
│                     │  • Transfer entropy   │                               │
│                     │  • Topology-aware     │                               │
│                     │    path analysis      │                               │
│                     └───────────┬───────────┘                               │
│                                 │                                           │
│                                 v                                           │
│                     ┌───────────────────────┐                               │
│                     │  LLM Analysis         │                               │
│                     │                       │                               │
│                     │  • Summarize findings │                               │
│                     │  • Plain English      │                               │
│                     │    explanation        │                               │
│                     │  • Confidence score   │                               │
│                     └───────────┬───────────┘                               │
│                                 │                                           │
│                                 v                                           │
│                     ┌───────────────────────┐                               │
│                     │  RCA REPORT           │                               │
│                     │                       │                               │
│                     │  • Causal chain       │                               │
│                     │    visualization      │                               │
│                     │  • Contributing       │                               │
│                     │    factors ranked     │                               │
│                     │  • Similar past       │                               │
│                     │    incidents          │                               │
│                     │  • Remediation        │                               │
│                     │    recommendations    │                               │
│                     └───────────────────────┘                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Auto-Remediation Flow

The remediation engine matches triggers to playbooks, checks approval policies, executes actions, and verifies outcomes.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       AUTO-REMEDIATION FLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐    ┌──────────────────────────────────────────┐          │
│  │  Trigger      │    │  Rule Match Engine                       │          │
│  │  Event        │───>│  Match incident/anomaly against          │          │
│  │  (incident,   │    │  playbook trigger conditions              │          │
│  │  anomaly,     │    └───────────────────┬──────────────────────┘          │
│  │  rule)        │                        │                                 │
│  └──────────────┘               ┌─────────┴─────────┐                      │
│                                 │                   │                      │
│                            match found         no match                    │
│                                 │                   │                      │
│                                 v                   v                      │
│                    ┌────────────────────┐   ┌──────────────┐               │
│                    │  Load Playbook     │   │  No action   │               │
│                    │  & Check Policy    │   │  (log only)  │               │
│                    └─────────┬──────────┘   └──────────────┘               │
│                              │                                              │
│              ┌───────────────┼───────────────┐                             │
│              │               │               │                             │
│         auto-approve    manual-approve   time-based                        │
│              │               │               │                             │
│              │               v               v                             │
│              │    ┌──────────────────┐  ┌─────────────────┐               │
│              │    │ Send Approval    │  │ Check Schedule  │               │
│              │    │ Request          │  │ (biz hours?)    │               │
│              │    │ (Slack/Teams/    │  └────┬───────┬────┘               │
│              │    │  Dashboard)      │       │       │                     │
│              │    └────┬───────┬─────┘   in-hours  off-hours              │
│              │         │       │          │       │                        │
│              │     approved  rejected     │       │                        │
│              │         │       │          │       v                        │
│              │         │       v          │  ┌──────────────┐             │
│              │         │  ┌─────────┐    │  │ Manual Approv │             │
│              │         │  │ Abort   │    │  └──────┬───────┘             │
│              │         │  │ & Log   │    │         │                      │
│              │         │  └─────────┘    │    approved/rejected           │
│              │         │                 │         │                      │
│              └─────────┴─────────────────┘         │                      │
│                         │                          │                      │
│                         v                          │                      │
│              ┌──────────────────────┐              │                      │
│              │  EXECUTE ACTION      │<─────────────┘ (if approved)       │
│              │                      │                                     │
│              │  • Target service    │                                     │
│              │  • Action parameters │                                     │
│              │  • Timeout: 5 min    │                                     │
│              │  • Retry: up to 3x   │                                     │
│              └──────────┬───────────┘                                     │
│                         │                                                  │
│              ┌──────────┴───────────┐                                     │
│              │                      │                                     │
│          succeeded              failed                                    │
│              │                      │                                     │
│              v                      v                                     │
│  ┌──────────────────┐  ┌──────────────────┐                              │
│  │ VERIFY OUTCOME   │  │ RETRY or ABORT   │                              │
│  │                  │  │                  │                              │
│  │ Check condition  │  │ If retries left: │                              │
│  │ (e.g., CPU <80%) │  │   retry action   │                              │
│  │ within timeout   │  │ Else:            │                              │
│  └───────┬──────────┘  │   abort & alert  │                              │
│          │              └──────────────────┘                              │
│    ┌─────┴──────┐                                                         │
│    │            │                                                         │
│  verified   not verified                                                  │
│    │            │                                                         │
│    v            v                                                         │
│ ┌────────┐  ┌────────────────┐                                           │
│ │SUCCESS │  │PARTIAL/FAILED  │                                           │
│ │Report  │  │Escalate to     │                                           │
│ │& close │  │human operator  │                                           │
│ └────────┘  └────────────────┘                                           │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Event Correlation Pipeline

The event correlation pipeline groups related events from multiple sources into unified incidents.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      EVENT CORRELATION PIPELINE                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ Anomaly  │  │ Alert    │  │ Rule     │  │ External │                  │
│  │ Events   │  │ Events   │  │ Match    │  │ Webhook  │                  │
│  │          │  │          │  │ Events   │  │ Events   │                  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘                  │
│       │              │             │              │                        │
│       └──────────────┴──────┬──────┴──────────────┘                        │
│                              v                                              │
│                  ┌───────────────────────┐                                  │
│                  │  EVENT INTAKE QUEUE   │                                  │
│                  │  (DragonflyDB Stream) │                                  │
│                  └───────────┬───────────┘                                  │
│                              │                                              │
│              ┌───────────────┼───────────────┐                             │
│              v               v               v                             │
│  ┌──────────────────┐ ┌─────────────────┐ ┌──────────────────┐           │
│  │ TEMPORAL          │ │ TOPOLOGICAL     │ │ PATTERN          │           │
│  │ CORRELATION       │ │ CORRELATION     │ │ CORRELATION      │           │
│  │                  │ │                 │ │                  │           │
│  │ Group events     │ │ Group events    │ │ Match against    │           │
│  │ within sliding   │ │ on connected    │ │ known failure    │           │
│  │ time window      │ │ services per    │ │ patterns from    │           │
│  │ (configurable    │ │ topology graph  │ │ historical       │           │
│  │  default: 5 min) │ │                 │ │ incidents        │           │
│  └────────┬─────────┘ └────────┬────────┘ └────────┬─────────┘           │
│           │                    │                    │                      │
│           └────────────────────┼────────────────────┘                      │
│                                v                                           │
│                   ┌────────────────────────┐                               │
│                   │  CORRELATION SCORER    │                               │
│                   │                        │                               │
│                   │  Combine temporal,     │                               │
│                   │  topological, and      │                               │
│                   │  pattern scores into   │                               │
│                   │  correlation confidence │                               │
│                   └───────────┬────────────┘                               │
│                               │                                            │
│                    ┌──────────┴──────────┐                                 │
│                    │                     │                                 │
│             confidence >= 0.7      confidence < 0.7                       │
│                    │                     │                                 │
│                    v                     v                                 │
│         ┌──────────────────┐  ┌──────────────────┐                       │
│         │ CREATE/MERGE     │  │ INDIVIDUAL       │                       │
│         │ CORRELATED       │  │ INCIDENT PER     │                       │
│         │ INCIDENT         │  │ EVENT            │                       │
│         │                  │  │ (or suppress)    │                       │
│         │ Single incident  │  └──────────────────┘                       │
│         │ with all events  │                                              │
│         │ as evidence      │                                              │
│         └──────────────────┘                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Cost Analysis Workflow

The cost analysis workflow processes resource utilization data to generate optimization recommendations.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       COST ANALYSIS WORKFLOW                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────┐     ┌──────────────────────┐                        │
│  │ Resource Metrics  │     │ Pricing/Billing Data │                        │
│  │ (VictoriaMetrics) │     │ (Cloud Provider API) │                        │
│  └────────┬─────────┘     └──────────┬───────────┘                        │
│           │                          │                                     │
│           └───────────┬──────────────┘                                     │
│                       v                                                     │
│           ┌───────────────────────┐                                        │
│           │ DATA AGGREGATION      │                                        │
│           │                       │                                        │
│           │ • Hourly/daily/weekly │                                        │
│           │   utilization stats   │                                        │
│           │ • Cost per resource   │                                        │
│           │ • Trend calculation   │                                        │
│           └───────────┬───────────┘                                        │
│                       │                                                     │
│        ┌──────────────┼──────────────┬──────────────┐                     │
│        v              v              v              v                     │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐            │
│  │ RIGHT-    │  │ IDLE      │  │ RESERVED  │  │ STORAGE   │            │
│  │ SIZING    │  │ RESOURCE  │  │ INSTANCE  │  │ TIER      │            │
│  │ ANALYSIS  │  │ DETECTION │  │ ANALYSIS  │  │ ANALYSIS  │            │
│  │           │  │           │  │           │  │           │            │
│  │ Compare   │  │ Find      │  │ Compare   │  │ Find data │            │
│  │ allocated │  │ resources │  │ on-demand │  │ on wrong  │            │
│  │ vs actual │  │ with zero │  │ vs RI     │  │ storage   │            │
│  │ usage     │  │ or near-  │  │ savings   │  │ tier      │            │
│  │           │  │ zero use  │  │           │  │           │            │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘            │
│        │              │              │              │                     │
│        └──────────────┴──────┬───────┴──────────────┘                     │
│                              v                                             │
│                  ┌───────────────────────┐                                 │
│                  │ RECOMMENDATION ENGINE │                                 │
│                  │                       │                                 │
│                  │ • Calculate savings   │                                 │
│                  │ • Assess risk level   │                                 │
│                  │ • Set confidence score│                                 │
│                  │ • Generate rollback   │                                 │
│                  │   plan                │                                 │
│                  └───────────┬───────────┘                                 │
│                              │                                             │
│                              v                                             │
│                  ┌───────────────────────┐                                 │
│                  │ DASHBOARD DISPLAY     │                                 │
│                  │                       │                                 │
│                  │ • Recommendations     │                                 │
│                  │   ranked by savings   │                                 │
│                  │ • Apply button with   │                                 │
│                  │   confirmation        │                                 │
│                  │ • Savings tracker     │                                 │
│                  │   (actual vs target)  │                                 │
│                  └───────────────────────┘                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```
