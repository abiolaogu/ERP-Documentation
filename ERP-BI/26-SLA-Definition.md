# ERP-BI Service Level Agreement (SLA)

| Field | Value |
|---|---|
| Module | ERP-BI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Service Level Objectives (SLOs)

| SLO | Target | Measurement Window |
|---|---|---|
| Availability | 99.95% | Monthly |
| Dashboard Load Latency (p95) | < 2 seconds | Rolling 7 days |
| Query Latency (p95) | < 5 seconds | Rolling 7 days |
| NLQ Response (p95) | < 3 seconds | Rolling 7 days |
| Error Rate | < 0.1% | Monthly |
| CDC Data Freshness | < 5 minutes | Continuous |
| Report Delivery Success | > 99.5% | Monthly |
| Alert Evaluation Success | > 99.9% | Monthly |

---

## 2. SLI Definitions

| SLI | Formula |
|---|---|
| Availability | (Total requests - 5xx errors) / Total requests |
| Latency | Time from request received to response sent |
| Error Rate | 5xx responses / Total responses |
| Freshness | Current time - Last successful CDC sync time |
| Delivery Success | Delivered reports / Scheduled reports |

---

## 3. Error Budget

| SLO | Monthly Budget | Allowed Downtime |
|---|---|---|
| 99.95% availability | 0.05% | 21.9 minutes/month |
| 99.9% availability | 0.1% | 43.8 minutes/month |
| 99.5% availability | 0.5% | 3.65 hours/month |

---

## 4. Tier-Specific SLAs

| Metric | Free | Professional | Enterprise |
|---|---|---|---|
| Availability | 99.5% | 99.9% | 99.95% |
| Support Response | 48 hours | 4 hours | 1 hour |
| Data Freshness | 30 minutes | 5 minutes | 1 minute |
| Concurrent Queries | 5 | 25 | 100 |

---

## 5. Incident Severity Levels

| Level | Definition | Response Time | Resolution Time |
|---|---|---|---|
| P1 - Critical | Complete service outage | 15 minutes | 4 hours |
| P2 - Major | Significant feature degradation | 30 minutes | 8 hours |
| P3 - Minor | Minor feature issue | 4 hours | 24 hours |
| P4 - Low | Cosmetic or enhancement | 24 hours | 5 business days |

---

## 6. Maintenance Windows

| Type | Schedule | Duration | Notice |
|---|---|---|---|
| Planned maintenance | Sunday 02:00-06:00 UTC | Up to 4 hours | 72 hours advance |
| Emergency maintenance | As needed | As needed | Best effort |
| ClickHouse upgrades | Quarterly | 2 hours | 1 week advance |

---

## 7. Compensation

| Availability | Credit |
|---|---|
| 99.95% - 99.0% | None |
| 99.0% - 95.0% | 10% monthly credit |
| 95.0% - 90.0% | 25% monthly credit |
| < 90.0% | 50% monthly credit |
