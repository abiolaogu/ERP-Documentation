# Compliance & Regulatory Matrix

**Product**: ERP-MessageBus
**Date**: 2026-03-03
**Review Cycle**: Quarterly
**Owner**: Security & Compliance Team

---

## 1. Applicability

ERP-MessageBus processes events that may contain PII, PHI, financial data, and payment card data depending on the originating module. The compliance posture must satisfy the strictest applicable standard.

## 2. Control Matrix

### GDPR (General Data Protection Regulation)

| # | Control | Implementation | Evidence Artifact | Owner | Status |
|---|---------|---------------|-------------------|-------|--------|
| GDPR-1 | Data minimization | Envelope schema enforces minimal PII; modules responsible for data content | Schema registry validation; pipeline review | Module Teams + Platform Eng | Implemented |
| GDPR-2 | Right to erasure | Connect pipeline to tombstone events by tenant_id; compacted topics support delete | Tombstone pipeline YAML; test evidence | Platform Eng | Implemented |
| GDPR-3 | Data portability | Export pipeline via Connect can extract all events for a tenant | Export pipeline template; test run logs | Platform Eng | Planned (v1.1) |
| GDPR-4 | Consent tracking | Event envelope includes consent_ref field; module responsibility | Envelope schema docs | Module Teams | Implemented |
| GDPR-5 | Data processing agreement | Redpanda self-hosted (no third-party processor) | Infrastructure architecture doc | Platform Eng | Implemented |
| GDPR-6 | Breach notification | AIOps detects unauthorized access patterns; alerts within 72h | AIOps alert rules; incident playbook | SRE + Security | Implemented |
| GDPR-7 | Cross-border transfer | All data stays in sovereign infrastructure (no cloud egress) | Infrastructure topology doc | Platform Eng | Implemented |

### SOC 2 (Type II)

| # | Control | Implementation | Evidence Artifact | Owner | Status |
|---|---------|---------------|-------------------|-------|--------|
| SOC2-1 | Access control (CC6.1) | SASL/SCRAM per-module auth; prefix ACLs; Console OIDC RBAC | ACL configs; OIDC config; access review logs | Platform Eng | Implemented |
| SOC2-2 | Logical access (CC6.2) | Module-scoped topic prefixes; no cross-module access without approval | ACL audit; approval workflow docs | Platform Eng + Security | Implemented |
| SOC2-3 | Encryption in transit (CC6.7) | mTLS between broker/clients (staging/prod); SASL/SCRAM auth | cert-manager configs; TLS verification tests | Platform Eng | Implemented |
| SOC2-4 | Encryption at rest (CC6.7) | Volume encryption on broker disks; RustFS server-side encryption | K8s StorageClass config; RustFS encryption config | Platform Eng | Implemented |
| SOC2-5 | Change management (CC8.1) | All config changes via PR + review; Fleet GitOps deployment | Git history; PR approvals; Fleet sync logs | Platform Eng | Implemented |
| SOC2-6 | Monitoring (CC7.2) | Prometheus metrics; Grafana dashboards; AIOps alerting | Dashboard screenshots; alert rule configs | SRE | Implemented |
| SOC2-7 | Incident response (CC7.3) | Runbooks for common incidents; escalation policy; post-mortem template | Runbook docs; incident log; post-mortem archive | SRE | Implemented |
| SOC2-8 | Availability (A1.2) | Multi-replica cluster; tiered storage backup; DR plan | DR plan doc; failover test results | Platform Eng + SRE | Implemented |

### HIPAA (Health Insurance Portability and Accountability Act)

| # | Control | Implementation | Evidence Artifact | Owner | Status |
|---|---------|---------------|-------------------|-------|--------|
| HIPAA-1 | Access controls (§164.312(a)) | SASL auth + prefix ACLs; healthcare module has dedicated ACL | ACL config for healthcare module | Platform Eng | Implemented |
| HIPAA-2 | Audit controls (§164.312(b)) | All topic operations logged; Redpanda audit log; Console access log | Audit log retention config; sample queries | Platform Eng | Implemented |
| HIPAA-3 | Integrity controls (§164.312(c)) | CRC32 checksums on segments; acks=all ensures write integrity | Redpanda config; producer config template | Platform Eng | Implemented |
| HIPAA-4 | Transmission security (§164.312(e)) | mTLS in staging/prod; SASL/SCRAM auth layer | TLS config; mTLS verification test | Platform Eng | Implemented |
| HIPAA-5 | PHI in events | Healthcare module events may contain PHI; encrypted in transit/at rest | Encryption configs; healthcare topic schema | Healthcare Team + Platform Eng | Implemented |
| HIPAA-6 | BAA coverage | Self-hosted infrastructure; no third-party BAA required | Infrastructure ownership docs | Legal + Platform Eng | Implemented |
| HIPAA-7 | Minimum necessary | Topic schemas enforce minimum necessary data; healthcare team reviews | Schema registry; healthcare topic review log | Healthcare Team | Implemented |

### PCI-DSS (Payment Card Industry Data Security Standard)

| # | Control | Implementation | Evidence Artifact | Owner | Status |
|---|---------|---------------|-------------------|-------|--------|
| PCI-1 | Network segmentation (Req 1) | K8s network policies isolate broker namespace; Istio mTLS | NetworkPolicy YAMLs; Istio AuthorizationPolicy | Platform Eng | Implemented |
| PCI-2 | Protect stored data (Req 3) | No PAN stored in events (tokenized by payment module before publishing) | Payment module schema; tokenization docs | Commerce Team + Platform Eng | Implemented |
| PCI-3 | Encrypt transmission (Req 4) | mTLS for all broker-client connections (staging/prod) | TLS config; cert-manager setup | Platform Eng | Implemented |
| PCI-4 | Access control (Req 7) | Least-privilege ACLs; payment topics restricted to commerce/finance modules | ACL config; access review log | Platform Eng + Security | Implemented |
| PCI-5 | Monitor access (Req 10) | Audit log for all topic operations; centralized in AIOps | Audit log config; monitoring dashboard | SRE + Security | Implemented |
| PCI-6 | Security testing (Req 11) | Quarterly vulnerability scans; annual penetration test | Scan reports; pentest reports | Security | In Progress |
| PCI-7 | Security policy (Req 12) | Documented security policy in SECURITY.md; reviewed quarterly | SECURITY.md; review log | Security | Implemented |

## 3. Evidence Repository

All compliance evidence is stored in:
- Git history (config changes, PR approvals)
- Grafana dashboards (monitoring evidence, exported quarterly)
- Audit log archive (2-year retention in YugabyteDB + RustFS)
- Incident post-mortem archive (Confluence/wiki)
- Quarterly access review exports

## 4. Review Schedule

| Review | Frequency | Owner | Output |
|--------|-----------|-------|--------|
| Access review | Quarterly | Security | Access review report |
| Compliance control assessment | Quarterly | Security + Platform Eng | Updated matrix |
| Penetration test | Annual | External vendor | Pentest report + remediation plan |
| DR drill | Semi-annual | SRE + Platform Eng | DR test report |
| Audit log integrity check | Monthly | SRE | Integrity verification report |

---

**Related**: [Security-Architecture](../01-architecture/Security-Architecture.md) | [DR-Plan](../03-quality-devops/DR-Plan.md)
