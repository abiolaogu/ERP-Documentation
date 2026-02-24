# ERP-Platform Data Privacy & Compliance

> **Document ID:** ERP-PLAT-DPC-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Classification:** Confidential
> **Related Documents:** [31-SECURITY.md](./31-SECURITY.md), [06-Business-Requirements-Document.md](./06-Business-Requirements-Document.md)

---

## 1. GDPR Article Mapping

| GDPR Article | Requirement | ERP-Platform Implementation | Status |
|-------------|-------------|---------------------------|--------|
| Art. 5(1)(a) | Lawfulness, fairness, transparency | Subscription consent at sign-up; privacy policy displayed | Implemented |
| Art. 5(1)(b) | Purpose limitation | Data used only for stated platform services | Implemented |
| Art. 5(1)(c) | Data minimization | Only necessary data collected per table schema | Implemented |
| Art. 5(1)(d) | Accuracy | Tenant self-service update via admin console | Implemented |
| Art. 5(1)(e) | Storage limitation | Retention policies per data classification | Implemented |
| Art. 5(1)(f) | Integrity and confidentiality | TLS 1.3, AES-256, RLS, AIDD guardrails | Implemented |
| Art. 6 | Lawful basis for processing | Contract performance + legitimate interest | Documented |
| Art. 13 | Information to data subjects | Privacy notice at registration | Implemented |
| Art. 15 | Right of access | Data export API (`GET /v1/data-export/{tenant_id}`) | Planned v1.1 |
| Art. 16 | Right to rectification | Tenant update endpoints (PUT /v1/tenant-provisioner/{id}) | Implemented |
| Art. 17 | Right to erasure | Tenant decommissioning with data purge | Implemented |
| Art. 18 | Right to restriction | Tenant suspension capability | Implemented |
| Art. 20 | Right to data portability | JSON export of all tenant data | Planned v1.1 |
| Art. 25 | Data protection by design | RLS, tenant isolation, encryption, AIDD | Implemented |
| Art. 28 | Processor obligations | DPA template for sub-processors | Documented |
| Art. 30 | Records of processing | Audit logs maintained for all operations | Implemented |
| Art. 32 | Security of processing | Encryption, access control, monitoring | Implemented |
| Art. 33 | Breach notification | Incident response plan with 72-hour notification | Documented |
| Art. 35 | Data protection impact assessment | PIA completed for platform data processing | Documented |

---

## 2. SOC 2 Type II Controls

### 2.1 Trust Services Criteria Mapping

| TSC | Criteria | ERP-Platform Control | Evidence |
|-----|----------|---------------------|----------|
| CC1.1 | Control environment | AIDD guardrails, code review process | AIDD_GUARDRAILS.md, PR reviews |
| CC2.1 | Information and communication | Audit logging, notification hub | Audit service, CloudEvents |
| CC3.1 | Risk assessment | AIDD confidence thresholds, blast-radius controls | AIDD policy engine |
| CC4.1 | Monitoring | Module health checks (30s), structured logging | Module registry, Prometheus |
| CC5.1 | Control activities | JWT authentication, RBAC, RLS | ERP-IAM integration, PostgreSQL RLS |
| CC6.1 | Logical access | JWT + X-Tenant-ID header validation | API gateway, service middleware |
| CC6.2 | Provisioned access | Role-based access: platform-admin, tenant-admin, module-admin, viewer | RBAC implementation |
| CC6.3 | Removed access | Tenant decommissioning, user deactivation | Tenant provisioner DELETE |
| CC6.6 | System boundaries | Network segmentation (DMZ, app, data subnets) | Kubernetes namespaces |
| CC7.1 | System monitoring | Health checks, structured logging, Prometheus metrics | /healthz, log aggregation |
| CC7.2 | Anomaly detection | AIDD guardrail blocks, error rate monitoring | Audit service, alerting |
| CC7.3 | Incident response | Runbook procedures (P1-P4), escalation matrix | Runbooks document |
| CC8.1 | Change management | GitHub PR process, CI/CD pipeline | GitHub Actions, doc-gen |
| CC9.1 | Risk mitigation | Multi-region DR, automated backups, encryption | DR plan |
| A1.1 | Availability commitment | 99.95% SLA, multi-replica deployment, HPA | Kubernetes HPA |
| A1.2 | Disaster recovery | RPO 1h, RTO 15min, automated failover | DR plan |
| PI1.1 | Processing integrity | Input validation, business rule enforcement | Subscription hub validation |

---

## 3. Data Classification

### 3.1 Classification Levels

| Level | Definition | Examples | Controls |
|-------|-----------|----------|----------|
| **Public** | Information freely available | Product catalog, public API docs | Integrity protection |
| **Internal** | Business information for internal use | Module health status, subscription counts | Access control, encryption in transit |
| **Confidential** | Sensitive business information | Tenant configurations, user details, notification content | Encryption at rest + in transit, RLS, access logging |
| **Restricted** | Highly sensitive, regulated data | Audit logs, AIDD decisions, financial data, healthcare/education data | Encryption, RLS, immutable storage, access approval, compliance controls |

### 3.2 Data Classification by Table

| Table | Classification | Justification |
|-------|---------------|---------------|
| products | Public | Product catalog is publicly accessible |
| modules | Internal | Module health status is operational data |
| tenants | Confidential | Contains organization identifiers and metadata |
| subscriptions | Confidential | Billing-related subscription details |
| entitlements | Confidential | Access control grants |
| audit_logs | Restricted | Compliance evidence, immutable records |
| notifications | Confidential | May contain sensitive communication |
| health_checks | Internal | Operational monitoring data |
| marketplace_listings | Public | Published marketplace content |
| marketplace_installations | Confidential | Tenant installation records |
| web_hosting_configs | Confidential | Domain and SSL configuration |
| activation_wizard_states | Confidential | Onboarding configuration data |
| webhook_endpoints | Confidential | Integration credentials |
| webhook_deliveries | Internal | Delivery tracking records |

---

## 4. Data Retention Policies

| Data Category | Active Retention | Archive Retention | Total | Deletion Method |
|--------------|-----------------|-------------------|-------|----------------|
| Tenant records | Lifetime of tenant | 90 days post-decommission | Tenant lifetime + 90d | Secure delete + audit record |
| Subscription records | Lifetime of subscription | 7 years post-cancellation | 7+ years | Archive to cold storage |
| Entitlement records | Lifetime of subscription | 7 years | 7+ years | Archive to cold storage |
| Audit logs | 1 year (hot storage) | 6 years (cold storage) | 7 years minimum | Archive to Glacier, then purge |
| Health check history | 90 days | None | 90 days | Auto-purge (scheduled job) |
| Notifications | 30 days | None | 30 days | Auto-purge (scheduled job) |
| Webhook deliveries | 30 days | None | 30 days | Auto-purge (scheduled job) |
| Product catalog versions | Indefinite | N/A | Indefinite | Git history |
| Session data (Redis) | 24 hours | None | 24 hours | Redis TTL expiry |

---

## 5. Data Subject Rights Implementation

### 5.1 Right of Access (Art. 15)

**Process:**
1. Data subject submits access request via privacy portal or admin console.
2. System verifies identity through ERP-IAM authentication.
3. System generates JSON export of all data associated with the data subject's tenant_id.
4. Export includes: tenant record, subscription, entitlements, audit logs (filtered to non-security-sensitive entries), notifications, web hosting configuration.
5. Export delivered via secure download link (72-hour expiry).
6. Audit record created for the access request.

### 5.2 Right to Rectification (Art. 16)

**Process:**
1. Tenant admin updates data via `PUT /v1/tenant-provisioner/{id}`.
2. System validates the update.
3. Previous values preserved in audit log (for compliance trail).
4. Confirmation notification sent.

### 5.3 Right to Erasure (Art. 17)

**Process:**
1. Tenant admin or data subject requests erasure.
2. System evaluates if erasure is legally permitted (retention obligations may override).
3. If permitted, system initiates tenant decommissioning.
4. 30-day grace period for data recovery.
5. After grace period: all tenant data purged from active databases.
6. Audit log entry for the erasure request retained (legal basis: legitimate interest for compliance).
7. Backups containing the data are overwritten within the backup rotation cycle.

### 5.4 Right to Data Portability (Art. 20)

**Process:**
1. Tenant admin requests data export.
2. System generates machine-readable JSON export.
3. Export includes structured data in standard formats.
4. Export delivered via secure download.

---

## 6. Data Processing Agreement (DPA) Template

### Key Clauses

1. **Subject Matter and Duration**: Processing of tenant business data for the purpose of providing ERP platform services for the duration of the subscription agreement.

2. **Nature and Purpose**: Storage, processing, and display of tenant configuration, subscription, entitlement, audit, and operational data to deliver unified ERP administration.

3. **Types of Personal Data**: Organization names, administrator email addresses, domain names, IP addresses in audit logs, user activity records.

4. **Categories of Data Subjects**: Tenant administrators, module users, API integrators.

5. **Sub-Processor List**: Cloud infrastructure provider (AWS/GCP/Azure), email delivery service, SMS gateway, CDN provider, payment processor.

6. **Security Measures**: TLS 1.3, AES-256 at rest, PostgreSQL RLS, AIDD guardrails, SOC 2 Type II certified operations.

7. **Breach Notification**: Within 72 hours of becoming aware of a personal data breach.

8. **Data Return/Deletion**: JSON export within 30 days of request; data deletion within 90 days of contract termination.

---

## 7. Privacy Impact Assessment Summary

### 7.1 Processing Activities Assessed

| Activity | Risk Level | Mitigations |
|----------|-----------|-------------|
| Tenant data storage | Medium | Encryption, RLS, access controls |
| Subscription management | Low | Standard business data processing |
| Audit log collection | Medium | Data minimization, retention policies |
| Notification delivery | Medium | Encryption in transit, no PII in logs |
| Health check monitoring | Low | No personal data collected |
| AIDD decision logging | High | Strict access control, immutable storage |
| Webhook payload delivery | Medium | HMAC signing, HTTPS enforcement |
| Cross-module entitlement checks | Medium | Tenant isolation, minimal data exchange |

### 7.2 Residual Risks

| Risk | Likelihood | Impact | Residual Rating |
|------|-----------|--------|----------------|
| Cross-tenant data leakage | Very Low | High | Medium (mitigated by RLS + header validation) |
| Insider access to audit logs | Low | High | Medium (mitigated by RBAC + logging) |
| Third-party sub-processor breach | Low | High | Medium (mitigated by DPA + security review) |
| AIDD guardrail bypass | Very Low | High | Low (mitigated by policy-as-code) |

---

*For security policy, see [31-SECURITY.md](./31-SECURITY.md). For business requirements, see [06-Business-Requirements-Document.md](./06-Business-Requirements-Document.md).*
