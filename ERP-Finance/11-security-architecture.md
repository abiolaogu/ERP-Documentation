# ERP-Finance Security Architecture

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | Security Architecture |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

## Security Overview

ERP-Finance handles the most sensitive data in the enterprise -- financial transactions, payment credentials, bank account details, and tax information. The security architecture is designed to meet PCI-DSS Level 1, SOX compliance, and GDPR requirements simultaneously.

## Security Architecture

```mermaid
flowchart TB
    subgraph Perimeter["Perimeter Security"]
        WAF["Web Application Firewall"]
        DDoS["DDoS Protection"]
        GW["API Gateway + Rate Limiting"]
    end

    subgraph AuthN["Authentication Layer"]
        OIDC["OIDC/JWT (ERP-IAM)"]
        MTLS["mTLS Service Mesh"]
        APIKEY["API Key Management"]
        MFA["Multi-Factor Authentication"]
    end

    subgraph AuthZ["Authorization Layer"]
        RBAC["Role-Based Access Control"]
        ABAC["Attribute-Based Access Control"]
        SOD["Separation of Duties Engine"]
        ENTL["Entitlement Checks (ERP-Platform)"]
    end

    subgraph DataSec["Data Security Layer"]
        TDE["Transparent Data Encryption (AES-256)"]
        FLE["Field-Level Encryption"]
        TOKEN["Payment Tokenization"]
        RLS["Row-Level Security (Tenant Isolation)"]
        MASK["Dynamic Data Masking"]
    end

    subgraph AuditSec["Audit & Monitoring"]
        AUDIT["Immutable Audit Trail"]
        SIEM["SIEM Integration"]
        IDS["Intrusion Detection"]
        ANOM["Anomaly Detection"]
    end

    Perimeter --> AuthN --> AuthZ --> DataSec
    DataSec --> AuditSec
```

## Authentication

### JWT Token Validation

All requests to business endpoints must include a valid JWT from ERP-IAM:

```
Authorization: Bearer <jwt>
X-Tenant-ID: <tenant_uuid>
```

Token claims are validated:
- `iss`: Must match ERP-IAM issuer URL
- `aud`: Must include `erp-finance`
- `exp`: Must not be expired
- `tenant_id`: Must match `X-Tenant-ID` header
- `sub`: User identifier for audit trail

### Service-to-Service Authentication

Internal services use mutual TLS (mTLS) with certificates managed by the service mesh. NATS connections authenticate via NKey or JWT.

## Authorization Model

### Role Hierarchy

```mermaid
flowchart TD
    SUPER["Super Admin"]
    SUPER --> CFO["CFO / Finance Director"]
    CFO --> CTRL["Controller"]
    CFO --> TREAS2["Treasurer"]
    CTRL --> AP_MGR["AP Manager"]
    CTRL --> AR_MGR["AR Manager"]
    CTRL --> TAX_MGR["Tax Manager"]
    AP_MGR --> AP_CLERK["AP Clerk"]
    AR_MGR --> AR_CLERK["AR Clerk"]
    TREAS2 --> CASH_MGR["Cash Manager"]
    CFO --> BUDGET_MGR["Budget Manager"]
    CFO --> ASSET_MGR["Asset Manager"]
    CFO --> EXPENSE_APPR["Expense Approver"]
```

### Permission Matrix

| Permission | CFO | Controller | AP Clerk | AR Clerk | Auditor |
|-----------|-----|-----------|----------|----------|---------|
| GL: View journals | Yes | Yes | Read own | Read own | Read all |
| GL: Post journal | Yes | Yes | No | No | No |
| GL: Close period | Yes | Yes | No | No | No |
| AP: Create invoice | Yes | Yes | Yes | No | No |
| AP: Approve payment | Yes | Yes | No | No | No |
| AP: Execute payment run | Yes | No | No | No | No |
| AR: Issue invoice | Yes | Yes | No | Yes | No |
| AR: Apply credit | Yes | Yes | No | Yes | No |
| Billing: Manage plans | Yes | No | No | No | No |
| Payments: Process refund | Yes | Yes | No | No | No |
| Assets: Dispose asset | Yes | Yes | No | No | No |
| Reports: All | Yes | Yes | Limited | Limited | Yes |

### Separation of Duties

Critical financial operations require different users for initiation and approval:

| Operation | Initiator | Approver |
|-----------|-----------|----------|
| Journal entry > $100K | Any user with GL access | Controller or CFO |
| Payment run execution | AP Clerk | AP Manager or CFO |
| Vendor master data change | AP Clerk | AP Manager |
| Credit note > $10K | AR Clerk | AR Manager |
| Budget approval | Budget Analyst | CFO |
| Asset disposal | Asset Manager | Controller |

## Data Protection

### Encryption Standards

| Data Class | At Rest | In Transit | Key Management |
|-----------|---------|-----------|---------------|
| Financial transactions | AES-256 (TDE) | TLS 1.3 | AWS KMS / Vault |
| Payment card data | AES-256 + tokenization | TLS 1.3 | PCI HSM |
| Bank account numbers | Field-level AES-256 | TLS 1.3 | Vault |
| Tax identifiers (TIN) | Field-level AES-256 | TLS 1.3 | Vault |
| Employee SSN/NIN | Field-level AES-256 | TLS 1.3 | Vault |
| Audit logs | AES-256 (immutable) | TLS 1.3 | AWS KMS |

### PCI-DSS Compliance

The payments service is designed for PCI-DSS Level 1 compliance:

- **No raw card data stored**: All card data tokenized via payment providers
- **Network segmentation**: Payments service runs in isolated Kubernetes namespace
- **Access logging**: All access to payment data logged with immutable audit trail
- **Vulnerability scanning**: Quarterly ASV scans, annual penetration testing
- **Encryption**: TLS 1.3 for all payment data in transit

### Row-Level Security

PostgreSQL RLS policies enforce tenant isolation:

```sql
CREATE POLICY tenant_isolation ON journal_entries
    USING (tenant_id = current_setting('app.current_tenant')::uuid);

ALTER TABLE journal_entries ENABLE ROW LEVEL SECURITY;
```

## Audit Trail

### Immutable Audit Log

Every financial operation creates an immutable audit record:

```json
{
  "audit_id": "aud_019523a4...",
  "timestamp": "2026-02-23T10:00:00Z",
  "tenant_id": "uuid",
  "user_id": "uuid",
  "action": "gl.journal_entry.posted",
  "resource_type": "journal_entry",
  "resource_id": "uuid",
  "changes": {
    "before": {},
    "after": {"status": "posted", "amount": 50000}
  },
  "ip_address": "192.168.1.100",
  "user_agent": "ERP-Finance/1.0",
  "session_id": "sess_abc123"
}
```

### Retention Policy

- Financial audit logs: 7 years (regulatory requirement)
- Access logs: 3 years
- API request logs: 1 year
- Debug/trace logs: 90 days

## AIDD Security Guardrails

```yaml
autonomous_actions:
  - read_only_queries
  - low_risk_notifications
supervised_actions:
  - data_mutations
  - workflow_automation
  - bulk_operations
prohibited_actions:
  - cross_tenant_data_access
  - irreversible_delete_without_backup
  - privilege_escalation
controls:
  require_human_in_the_loop_for_high_risk: true
  decision_logging: true
  rollback_window_hours: 24
```

## Threat Model

```mermaid
flowchart LR
    subgraph Threats["Key Threats"]
        T1["Unauthorized access<br/>to financial data"]
        T2["Payment fraud"]
        T3["Data exfiltration"]
        T4["Cross-tenant<br/>data leakage"]
        T5["Insider threat"]
        T6["SQL injection"]
    end

    subgraph Mitigations["Mitigations"]
        M1["JWT + RBAC + MFA"]
        M2["Fraud scoring + velocity checks"]
        M3["DLP + encryption + masking"]
        M4["RLS + tenant context enforcement"]
        M5["SoD + audit trail + anomaly detection"]
        M6["Parameterized queries + SQLx compile-time checks"]
    end

    T1 --> M1
    T2 --> M2
    T3 --> M3
    T4 --> M4
    T5 --> M5
    T6 --> M6
```
