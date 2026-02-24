# ERP-DBaaS Security

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Security                     |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Confidential                |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [Encryption at Rest](#1-encryption-at-rest)
2. [Encryption in Transit](#2-encryption-in-transit)
3. [Credential Management and Rotation](#3-credential-management-and-rotation)
4. [Network Security](#4-network-security)
5. [RBAC for Database Operations](#5-rbac-for-database-operations)
6. [Audit Logging](#6-audit-logging)
7. [Compliance](#7-compliance)
8. [Vulnerability Scanning for Engine Images](#8-vulnerability-scanning-for-engine-images)

---

## 1. Encryption at Rest

### 1.1 Storage Encryption

All persistent data is encrypted at rest using volume-level encryption provided by the Kubernetes StorageClass and engine-level encryption where supported.

| Layer                | Encryption Method              | Key Management       |
|----------------------|--------------------------------|----------------------|
| Kubernetes PVCs      | dm-crypt / LUKS (volume level) | K8s KMS provider     |
| YugabyteDB           | Built-in AES-256 encryption    | Universe key rotation|
| DragonflyDB          | Volume-level only              | dm-crypt             |
| ClickHouse           | AES-256-CTR (disk encryption)  | Per-disk key         |
| Tembo (PostgreSQL)   | pgcrypto + volume-level        | Application key      |
| SurrealDB            | TiKV encryption at rest        | Master key rotation  |
| QuestDB              | Volume-level only              | dm-crypt             |
| Apache Doris         | Volume-level only              | dm-crypt             |
| InfluxDB             | Volume-level only              | dm-crypt             |

### 1.2 Backup Encryption

All backups are encrypted using AES-256-GCM with a two-tier key hierarchy before upload to RustFS.

```
┌──────────────────────────────────────────────────────┐
│                Key Hierarchy                          │
│                                                       │
│  Root Key (HSM-backed, never leaves HSM)              │
│    │                                                  │
│    └── Tenant Master Key (KEK)                        │
│          │  - Unique per tenant                       │
│          │  - Rotated annually                        │
│          │  - Stored in secrets manager               │
│          │                                            │
│          └── Data Encryption Key (DEK)                │
│               - Unique per backup                     │
│               - Generated at backup time              │
│               - Wrapped by KEK                        │
│               - Stored with backup metadata           │
└──────────────────────────────────────────────────────┘
```

### 1.3 etcd Encryption

Kubernetes Secrets (containing database credentials) are encrypted at rest in etcd using the Kubernetes encryption provider configuration.

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key-2026
              secret: <base64-encoded-32-byte-key>
      - identity: {}
```

---

## 2. Encryption in Transit

### 2.1 TLS Configuration

All network communication between components and database instances is encrypted using TLS 1.3.

| Communication Path                        | Protocol  | Min TLS Version | Certificate Source       |
|-------------------------------------------|-----------|-----------------|--------------------------|
| Client to Go Gateway                      | HTTPS     | TLS 1.3         | cert-manager (Let's Encrypt) |
| Go Gateway to Node.js API                 | mTLS      | TLS 1.3         | Internal CA (cert-manager)  |
| Node.js API to Hasura                     | HTTPS     | TLS 1.2         | Internal CA              |
| Application to Database Instance          | TLS       | TLS 1.2         | Internal CA              |
| Database Replication (inter-node)         | TLS       | TLS 1.2         | Internal CA              |
| Backup Controller to RustFS               | HTTPS     | TLS 1.3         | Internal CA              |
| Operator to K8s API Server                | mTLS      | TLS 1.3         | Service account token    |
| VictoriaMetrics scraping                  | HTTPS     | TLS 1.2         | Internal CA              |

### 2.2 Certificate Management

```
┌──────────────────────────────────────────────┐
│ cert-manager (Kubernetes)                     │
│                                               │
│  Issuers:                                     │
│  ├── letsencrypt-prod (external endpoints)    │
│  ├── internal-ca (inter-service mTLS)         │
│  └── database-ca (database connections)       │
│                                               │
│  Certificate Lifecycle:                       │
│  ├── Auto-renewal: 30 days before expiry      │
│  ├── Max validity: 90 days (external)         │
│  ├── Max validity: 365 days (internal)        │
│  └── Key type: ECDSA P-256                    │
└──────────────────────────────────────────────┘
```

### 2.3 Per-Engine TLS Configuration

**YugabyteDB:**
```yaml
# TLS flags for YugabyteDB TServer and Master
tserverFlags:
  use_node_to_node_encryption: "true"
  allow_insecure_connections: "false"
  use_client_to_server_encryption: "true"
  certs_dir: "/opt/certs"
masterFlags:
  use_node_to_node_encryption: "true"
  allow_insecure_connections: "false"
```

**DragonflyDB:**
```yaml
# DragonflyDB TLS configuration
args:
  - "--tls"
  - "--tls_cert_file=/certs/tls.crt"
  - "--tls_key_file=/certs/tls.key"
  - "--tls_ca_cert_file=/certs/ca.crt"
```

**ClickHouse:**
```xml
<!-- ClickHouse TLS configuration -->
<openSSL>
    <server>
        <certificateFile>/certs/tls.crt</certificateFile>
        <privateKeyFile>/certs/tls.key</privateKeyFile>
        <caConfig>/certs/ca.crt</caConfig>
        <verificationMode>strict</verificationMode>
    </server>
</openSSL>
```

---

## 3. Credential Management and Rotation

### 3.1 Password Policy

| Parameter               | Requirement                          |
|-------------------------|--------------------------------------|
| Minimum length          | 32 characters                        |
| Character classes       | Uppercase, lowercase, digits, special|
| Entropy                 | >= 192 bits                          |
| Generation method       | Cryptographically secure PRNG        |
| Storage                 | K8s Secrets (etcd-encrypted)         |
| Transmission            | Never logged or exposed in plain text|
| Clipboard expiry        | 60 seconds in UI                     |

### 3.2 Rotation Schedule

| Credential Type        | Default Rotation Period | Minimum Period | Grace Period |
|------------------------|-------------------------|----------------|--------------|
| Database user password | 90 days                 | 30 days        | 24 hours     |
| Service account token  | 180 days                | 90 days        | 48 hours     |
| TLS certificate        | Auto (cert-manager)     | N/A            | N/A          |
| RustFS access key      | 365 days                | 180 days       | 72 hours     |
| Encryption KEK         | 365 days                | 180 days       | 7 days       |

### 3.3 Rotation Process

```
1. Generate new credentials (cryptographically secure)
2. Create new credentials on the database engine
3. Verify connectivity with new credentials
4. Update K8s Secret with new credentials
5. Maintain old credentials for grace period
6. Emit credential.rotated event via Pulsar
7. Revoke old credentials after grace period
8. Record rotation in audit log
```

### 3.4 Emergency Credential Rotation

In the event of a suspected credential compromise:

1. Security officer triggers emergency rotation via API or dashboard.
2. Grace period is set to zero (immediate revocation of old credentials).
3. All applications consuming the affected credentials are force-restarted.
4. Incident is logged and escalated per the security incident response plan.
5. Forensic investigation is initiated to determine the scope of the compromise.

---

## 4. Network Security

### 4.1 Kubernetes NetworkPolicies

All tenant namespaces start with a default deny-all policy. Specific traffic patterns are allowed through explicit NetworkPolicy rules.

```yaml
# Default deny all traffic in tenant namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: dbaas-{tenant-id}
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### 4.2 Allowed Traffic Matrix

| Source              | Destination           | Port(s)     | Protocol | Purpose                |
|---------------------|-----------------------|-------------|----------|------------------------|
| dbaas-system        | Tenant namespace      | Engine ports| TCP      | Operator management    |
| Within namespace    | Within namespace      | All         | TCP      | Intra-tenant replication|
| Tenant pods         | kube-system           | 53          | UDP/TCP  | DNS resolution         |
| Backup agent        | RustFS subnet         | 443         | TCP      | Backup upload/download |
| Metrics exporter    | VictoriaMetrics       | 8428        | TCP      | Metrics scraping       |

### 4.3 External Access Controls

Database instances are not directly exposed outside the Kubernetes cluster. External access is controlled through:

| Access Method          | Security Controls                                      |
|------------------------|--------------------------------------------------------|
| DBaaS API (HTTPS)      | JWT authentication, rate limiting, WAF                 |
| K8s port-forward       | RBAC-restricted, audit logged                          |
| VPN tunnel             | WireGuard encryption, certificate-based auth           |
| SSH bastion (emergency)| MFA required, session recorded, time-limited access    |

### 4.4 DDoS Protection

| Layer              | Protection Mechanism                              |
|--------------------|---------------------------------------------------|
| Network (L3/L4)    | Cloud provider DDoS protection, connection limits |
| Application (L7)   | Go Gateway rate limiting (per-tenant, per-IP)     |
| DNS                | DNS rate limiting, DNSSEC                         |

---

## 5. RBAC for Database Operations

### 5.1 Platform Roles

| Role               | Permissions                                                    |
|--------------------|----------------------------------------------------------------|
| `platform_admin`   | Full access to all tenants, all operations, system config      |
| `platform_viewer`  | Read-only access to all tenants (for support)                  |
| `tenant_admin`     | Full CRUD on own tenant instances, quota view, billing         |
| `tenant_operator`  | Create, scale, backup instances; no delete, no billing         |
| `tenant_viewer`    | Read-only access to own tenant instances and backups           |
| `service_account`  | API access only, limited to specific operations                |

### 5.2 Permission Matrix

| Operation              | platform_admin | tenant_admin | tenant_operator | tenant_viewer |
|------------------------|:--------------:|:------------:|:---------------:|:-------------:|
| Create instance        | Yes            | Yes          | Yes             | No            |
| View instance          | Yes            | Yes          | Yes             | Yes           |
| Scale instance         | Yes            | Yes          | Yes             | No            |
| Delete instance        | Yes            | Yes          | No              | No            |
| Create backup          | Yes            | Yes          | Yes             | No            |
| Restore backup         | Yes            | Yes          | Yes             | No            |
| Delete backup          | Yes            | Yes          | No              | No            |
| Rotate credentials     | Yes            | Yes          | No              | No            |
| View credentials       | Yes            | Yes          | Yes             | No            |
| Install plugin         | Yes            | Yes          | Yes             | No            |
| Modify quota           | Yes            | No           | No              | No            |
| View audit logs        | Yes            | Yes          | Yes             | Yes           |

### 5.3 Authentik Integration

```
User authenticates via Authentik (OIDC)
    │
    ▼
JWT token contains:
  - sub: user ID
  - tenant_id: tenant identifier
  - roles: [tenant_admin, ...]
  - groups: [dbaas-admins, ...]
    │
    ▼
Go Gateway validates JWT:
  - Signature verification (RS256)
  - Expiry check
  - Issuer validation
  - Audience validation
    │
    ▼
Role extracted and mapped to permissions
  - X-Hasura-Role header set for Hasura
  - X-Hasura-Tenant-Id header set for row-level security
```

### 5.4 Database-Level RBAC

Within each provisioned database instance, DBaaS creates a role hierarchy:

| Database Role    | Privileges                                          | Used By            |
|------------------|-----------------------------------------------------|--------------------|
| `dbaas_admin`    | SUPERUSER (engine-level admin)                      | DBaaS operators    |
| `app_owner`      | CREATE, ALTER, DROP on application schemas           | Application admin  |
| `app_readwrite`  | SELECT, INSERT, UPDATE, DELETE on application tables | Application service|
| `app_readonly`   | SELECT only on application tables                    | Read replicas, BI  |
| `backup_user`    | REPLICATION, SELECT on all tables                    | Backup controller  |

---

## 6. Audit Logging

### 6.1 Audit Event Categories

| Category          | Events Captured                                            |
|-------------------|------------------------------------------------------------|
| Authentication    | Login, logout, failed login, token refresh                 |
| Authorization     | Permission granted, permission denied, role change         |
| Data Access       | Instance view, credential view, backup download            |
| Data Modification | Instance create/update/delete, backup create/delete        |
| Configuration     | Policy change, quota change, engine config update          |
| Security          | Credential rotation, TLS certificate renewal, policy violation |

### 6.2 Audit Log Fields

Every audit event contains the following fields:

```json
{
  "event_id": "UUID",
  "timestamp": "ISO-8601",
  "tenant_id": "string",
  "actor_id": "string",
  "actor_type": "user|service|system",
  "actor_ip": "IP address",
  "user_agent": "string",
  "action": "string",
  "resource_type": "instance|backup|credential|plugin|quota",
  "resource_id": "string",
  "result": "success|failure|denied",
  "details": {},
  "request_id": "UUID (correlation ID)"
}
```

### 6.3 Log Integrity

Audit logs are protected against tampering through:

| Control                   | Implementation                                    |
|---------------------------|---------------------------------------------------|
| Append-only storage       | Write-once storage backend                        |
| Hash chain                | Each log entry includes SHA-256 of previous entry |
| Separate storage          | Audit logs stored in a dedicated database          |
| Access restriction        | Only platform_admin can view, no one can modify    |
| Export to immutable store | Daily export to RustFS with integrity signatures   |
| Retention lock            | Configurable retention lock prevents early deletion|

---

## 7. Compliance

### 7.1 SOC 2 Type II

| Control Area           | DBaaS Implementation                                   |
|------------------------|---------------------------------------------------------|
| Access control         | RBAC, MFA via Authentik, principle of least privilege   |
| Change management      | GitOps, PR reviews, canary deployments                  |
| Risk assessment        | Quarterly security reviews, annual pen tests            |
| Monitoring             | VictoriaMetrics, audit logging, real-time alerts        |
| Incident response      | DR plan, escalation matrix, post-incident reviews       |
| Data protection        | Encryption at rest/transit, backup encryption, key mgmt |
| Vendor management      | Open-source engines with enterprise support contracts   |

### 7.2 PCI-DSS (for Financial Data)

ERP modules handling financial data (ERP-Finance, ERP-Billing) require PCI-DSS compliant database instances.

| PCI-DSS Requirement    | DBaaS Implementation                                   |
|------------------------|---------------------------------------------------------|
| Req 1: Firewall        | Kubernetes NetworkPolicies, default deny                |
| Req 2: No defaults     | Custom credentials generated per instance               |
| Req 3: Protect data    | AES-256 encryption at rest, column-level encryption     |
| Req 4: Encrypt transit | TLS 1.2+ for all database connections                   |
| Req 5: Anti-malware    | Trivy image scanning, no shell in production containers |
| Req 6: Secure systems  | Automated patching, vulnerability scanning              |
| Req 7: Restrict access | RBAC with role-based permissions                        |
| Req 8: Identify users  | Individual user accounts via Authentik, MFA             |
| Req 9: Physical access | Cloud provider responsibility (shared model)            |
| Req 10: Logging        | Comprehensive audit logging, 1-year retention           |
| Req 11: Testing        | Quarterly vulnerability scans, annual pen tests         |
| Req 12: Security policy| Documented security policies, annual review             |

### 7.3 GDPR Considerations

| Aspect                  | Implementation                                         |
|-------------------------|--------------------------------------------------------|
| Data residency          | Region-specific deployments, no cross-region PII       |
| Right to erasure        | Tenant data deletion API, backup purge capability      |
| Data portability        | Export API for database dumps                          |
| Privacy by design       | tenant_id isolation, minimal data collection           |
| Breach notification     | Automated detection, 72-hour notification pipeline     |

---

## 8. Vulnerability Scanning for Engine Images

### 8.1 Scanning Pipeline

All database engine images and operator images are scanned for vulnerabilities before deployment.

```
Image Build ──> Trivy Scan ──> Policy Gate ──> Registry Push
                    │
               ┌────┴────┐
               ▼         ▼
           PASS       FAIL (CRITICAL/HIGH)
               │         │
               ▼         ▼
           Push to    Block push,
           registry   alert team
```

### 8.2 Scan Policy

| Severity  | Action                                     | SLA for Remediation |
|-----------|--------------------------------------------|---------------------|
| CRITICAL  | Block deployment, immediate alert           | 24 hours            |
| HIGH      | Block deployment, alert                     | 7 days              |
| MEDIUM    | Allow deployment, create tracking issue     | 30 days             |
| LOW       | Allow deployment, batch review quarterly    | 90 days             |

### 8.3 Image Provenance

| Control                    | Implementation                               |
|----------------------------|----------------------------------------------|
| Base image source          | Official vendor images or distroless          |
| Image signing              | Cosign (Sigstore) signatures on all images   |
| SBOM generation            | Syft generates SBOMs for every image build   |
| Attestation                | SLSA Level 3 build provenance                |
| Registry access            | Pull-only for production, push requires CI   |
| Tag immutability           | SHA-based tags, no mutable tags in production|

### 8.4 Runtime Security

| Control                    | Implementation                               |
|----------------------------|----------------------------------------------|
| Pod security standards     | Restricted PSS enforced on all namespaces    |
| Read-only root filesystem  | Enabled on all operator and application pods |
| No privilege escalation    | `allowPrivilegeEscalation: false` on all pods|
| Non-root execution         | All containers run as non-root (UID >= 1000) |
| Seccomp profile            | RuntimeDefault profile on all pods            |
| Resource limits            | CPU and memory limits set on all containers   |
| Service mesh               | Optional Istio sidecar for mTLS enforcement  |

### 8.5 Regular Security Activities

| Activity                         | Frequency  | Owner                | Output                    |
|----------------------------------|------------|----------------------|---------------------------|
| Automated image scan             | Every build| CI pipeline          | Scan report               |
| Dependency audit (npm, Go mod)   | Weekly     | CI pipeline          | Dependency report         |
| Penetration testing              | Annual     | External vendor      | Pen test report           |
| Security architecture review     | Annual     | Security team        | Architecture review doc   |
| Tabletop DR exercise             | Quarterly  | SRE + Security       | Exercise findings         |
| Kubernetes CIS benchmark         | Monthly    | Platform team        | Compliance report         |
| Secrets rotation verification    | Monthly    | Automated + manual   | Rotation status report    |

---

## Appendix: Security Contact Information

| Role                      | Contact Method          | Response SLA    |
|---------------------------|-------------------------|-----------------|
| Security Incident Report  | security@erp.io         | 1 hour          |
| Vulnerability Disclosure  | security@erp.io (PGP)  | 24 hours        |
| Security Team On-Call     | PagerDuty               | 15 minutes      |
| Compliance Inquiries      | compliance@erp.io       | 2 business days |

## Appendix: Security Tooling Summary

| Tool               | Purpose                          | Integration Point     |
|--------------------|----------------------------------|-----------------------|
| Trivy              | Container image vulnerability scan| CI pipeline          |
| Cosign             | Image signing and verification   | CI pipeline + runtime|
| Syft               | SBOM generation                  | CI pipeline          |
| Falco              | Runtime security monitoring       | Kubernetes DaemonSet |
| cert-manager       | TLS certificate lifecycle         | Kubernetes           |
| Authentik          | Identity and access management    | Gateway              |
| OPA / Gatekeeper   | Policy enforcement                | Kubernetes admission |
| VictoriaMetrics    | Security metrics and alerting     | Monitoring stack     |
