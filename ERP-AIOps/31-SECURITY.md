# ERP-AIOps Security

> **Document ID:** ERP-AIOPS-SEC-031
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [28-Multi-Tenancy-Guide.md](./28-Multi-Tenancy-Guide.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## 1. API Authentication (JWT + Service Tokens)

### JWT Authentication (User Requests)

All user-facing API requests must include a valid JWT token issued by Authentik (the ERP-IAM identity provider).

**Token Flow:**

```
User → Authentik (login) → JWT issued → Include in Authorization header → AIOps Gateway validates
```

**JWT Structure:**

```json
{
  "header": {
    "alg": "RS256",
    "kid": "authentik-signing-key-001"
  },
  "payload": {
    "sub": "user-uuid-001",
    "iss": "https://auth.erp.internal/application/o/aiops/",
    "aud": "aiops-client-id",
    "exp": 1708776600,
    "iat": 1708773000,
    "tenant_id": "tenant-001",
    "roles": ["aiops_admin", "incident_manager"],
    "email": "jane.doe@example.com",
    "name": "Jane Doe"
  }
}
```

**Validation Steps (Go Gateway):**

1. Extract the `Authorization: Bearer <token>` header.
2. Decode the JWT header to get the `kid` (key ID).
3. Fetch the public key from Authentik's JWKS endpoint (`/.well-known/jwks.json`), cached locally.
4. Verify the RS256 signature.
5. Validate claims: `iss`, `aud`, `exp` (not expired), `iat` (not in future).
6. Extract `tenant_id` and `roles` for downstream use.
7. Reject if any validation step fails (HTTP 401).

### Service-to-Service Authentication (Service Tokens)

Internal service-to-service communication (e.g., Rust API to Python AI Brain) uses pre-shared service tokens.

```yaml
# Service token configuration
service_auth:
  tokens:
    rust_api_to_ai_brain: "${SVC_TOKEN_API_BRAIN}"
    rust_api_to_observability: "${SVC_TOKEN_API_OBS}"
    gateway_to_rust_api: "${SVC_TOKEN_GW_API}"
  header: "X-Service-Token"
  validation: "HMAC-SHA256"
```

**Token Rotation:**

- Service tokens are rotated every 90 days.
- Rotation is performed via Kubernetes Secret updates with zero-downtime deployment.
- Both the old and new token are accepted during a 24-hour overlap window.

---

## 2. RBAC for AIOps Operations

### Role Definitions

| Role | Description | Permissions |
|------|-------------|-------------|
| `aiops_viewer` | Read-only access to dashboards and data | View incidents, anomalies, topology, cost, security findings |
| `aiops_operator` | Operational access for on-call engineers | All viewer permissions + acknowledge/investigate incidents, review anomalies, approve remediation |
| `aiops_admin` | Full administrative access | All operator permissions + create/edit/delete rules, playbooks, integrations, manage thresholds |
| `aiops_security` | Security-focused role | All viewer permissions + triage security findings, trigger security scans, manage risk acceptance |
| `aiops_cost_manager` | Cost optimization role | View cost data, apply/reject cost recommendations |
| `aiops_super_admin` | Platform administration | All permissions + manage tenants, quotas, system configuration |

### Permission Matrix

| Action | Viewer | Operator | Admin | Security | Cost Mgr | Super Admin |
|--------|--------|----------|-------|----------|----------|-------------|
| View incidents | Yes | Yes | Yes | Yes | No | Yes |
| Create incidents | No | Yes | Yes | No | No | Yes |
| Transition incidents | No | Yes | Yes | No | No | Yes |
| Trigger RCA | No | Yes | Yes | No | No | Yes |
| View anomalies | Yes | Yes | Yes | Yes | No | Yes |
| Adjust thresholds | No | No | Yes | No | No | Yes |
| Create rules | No | No | Yes | No | No | Yes |
| Edit rules | No | No | Yes | No | No | Yes |
| View topology | Yes | Yes | Yes | Yes | No | Yes |
| Create playbooks | No | No | Yes | No | No | Yes |
| Approve remediation | No | Yes | Yes | No | No | Yes |
| View cost data | Yes | Yes | Yes | No | Yes | Yes |
| Apply cost recommendations | No | No | No | No | Yes | Yes |
| View security findings | Yes | Yes | Yes | Yes | No | Yes |
| Triage security findings | No | No | No | Yes | No | Yes |
| Trigger security scans | No | No | No | Yes | No | Yes |
| Manage integrations | No | No | Yes | No | No | Yes |
| Manage tenants | No | No | No | No | No | Yes |

### RBAC Enforcement

```rust
// Middleware: Check permissions for the current route
pub async fn rbac_middleware(
    Extension(tenant_ctx): Extension<TenantContext>,
    Extension(user_ctx): Extension<UserContext>,
    req: Request<Body>,
    next: Next,
) -> Result<Response, StatusCode> {
    let required_permission = route_to_permission(req.method(), req.uri().path());

    let has_permission = user_ctx.roles.iter().any(|role| {
        ROLE_PERMISSIONS
            .get(role.as_str())
            .map(|perms| perms.contains(&required_permission))
            .unwrap_or(false)
    });

    if !has_permission {
        return Err(StatusCode::FORBIDDEN);
    }

    Ok(next.run(req).await)
}
```

---

## 3. Tenant Data Isolation Verification

### Isolation Testing Strategy

Tenant data isolation is verified through automated tests run in the CI pipeline and periodic production audits.

**CI Pipeline Tests:**

```python
class TestTenantIsolation:
    """Verify that tenant A cannot access tenant B's data."""

    def test_incident_isolation(self, tenant_a_client, tenant_b_client):
        # Create incident as tenant A
        incident = tenant_a_client.post("/incidents", json={...})
        incident_id = incident.json()["id"]

        # Verify tenant B cannot access it
        response = tenant_b_client.get(f"/incidents/{incident_id}")
        assert response.status_code == 404

    def test_anomaly_isolation(self, tenant_a_client, tenant_b_client):
        # List anomalies as tenant A
        a_anomalies = tenant_a_client.get("/anomalies").json()

        # List anomalies as tenant B
        b_anomalies = tenant_b_client.get("/anomalies").json()

        # No overlap in IDs
        a_ids = {a["id"] for a in a_anomalies["data"]}
        b_ids = {b["id"] for b in b_anomalies["data"]}
        assert a_ids.isdisjoint(b_ids)

    def test_cache_isolation(self, tenant_a_client, tenant_b_client):
        # Populate cache as tenant A
        tenant_a_client.get("/topology")

        # Verify tenant B gets independent cache
        # (This is verified by inspecting cache keys)
        pass

    def test_missing_tenant_header_rejected(self, raw_client):
        # Request without X-Tenant-ID should be rejected
        response = raw_client.get("/incidents", headers={})
        assert response.status_code == 403
```

**Production Audit (Monthly):**

1. Run cross-tenant query detection script against YugabyteDB query logs.
2. Verify all queries include `tenant_id` filter.
3. Scan DragonflyDB keys to confirm tenant prefix convention.
4. Review access logs for any 200 responses to cross-tenant requests.

---

## 4. Remediation Action Authorization

Automated remediation actions are high-risk operations that require additional authorization beyond standard RBAC.

### Authorization Layers

```
Layer 1: RBAC Role Check
  └─ User must have aiops_operator or aiops_admin role

Layer 2: Playbook Approval Policy
  └─ Auto-approve: System executes immediately
  └─ Manual-approve: Requires explicit human approval
  └─ Time-based: Different policy for business hours vs. off-hours

Layer 3: Action Risk Assessment
  └─ Low-risk actions: Scale out, clear cache
  └─ Medium-risk actions: Restart service, config rollback
  └─ High-risk actions: Rollback deployment, DNS failover

Layer 4: Tenant Quota Check
  └─ Max concurrent remediations per tenant tier
  └─ Cooldown period between same action on same target
```

### Remediation Audit Trail

Every remediation action is logged with full context:

```json
{
  "action": "scale_out",
  "target_service": "erp-crm",
  "tenant_id": "tenant-001",
  "triggered_by": "playbook:pb-001",
  "trigger_event": "incident:inc-001",
  "approval": {
    "policy": "manual_approve",
    "approved_by": "user-001 (jane.doe@example.com)",
    "approved_at": "2026-02-24T10:30:15Z"
  },
  "execution": {
    "started_at": "2026-02-24T10:30:16Z",
    "completed_at": "2026-02-24T10:31:30Z",
    "result": "success",
    "parameters": {
      "from_replicas": 3,
      "to_replicas": 6
    }
  },
  "ip_address": "10.0.1.45",
  "user_agent": "AIOps-Dashboard/1.0"
}
```

---

## 5. Audit Logging for All Actions

### What is Logged

Every state-changing action in the AIOps platform is captured in an immutable audit log.

| Category | Actions Logged |
|----------|---------------|
| Authentication | Login, logout, token refresh, failed login attempts |
| Incidents | Create, update, transition, assign, comment, delete |
| Rules | Create, update, enable, disable, delete, clone |
| Remediation | Create playbook, execute action, approve, reject, rollback |
| Cost | Apply recommendation, reject recommendation |
| Security | Triage finding, accept risk, trigger scan |
| Configuration | Change threshold, update integration, modify tenant settings |
| Access | API access (all endpoints), WebSocket connections |

### Audit Log Schema

```sql
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id TEXT NOT NULL,
    actor_id TEXT NOT NULL,           -- User or system identifier
    actor_type TEXT NOT NULL,         -- 'user', 'system', 'service'
    action TEXT NOT NULL,             -- 'incident.create', 'rule.update', etc.
    resource_type TEXT NOT NULL,      -- 'incident', 'rule', 'playbook', etc.
    resource_id TEXT,                 -- ID of the affected resource
    details JSONB,                    -- Action-specific details
    ip_address INET,
    user_agent TEXT,
    timestamp TIMESTAMPTZ DEFAULT NOW(),

    -- Partitioned by month for efficient querying and retention
    PRIMARY KEY (tenant_id, timestamp, id)
) PARTITION BY RANGE (timestamp);

-- Create monthly partitions
CREATE TABLE audit_log_2026_02 PARTITION OF audit_log
    FOR VALUES FROM ('2026-02-01') TO ('2026-03-01');

CREATE INDEX idx_audit_tenant_action ON audit_log(tenant_id, action, timestamp DESC);
CREATE INDEX idx_audit_tenant_resource ON audit_log(tenant_id, resource_type, resource_id);
CREATE INDEX idx_audit_actor ON audit_log(tenant_id, actor_id, timestamp DESC);
```

### Audit Log Retention

| Tier | Retention | Storage |
|------|-----------|---------|
| Free | 30 days | YugabyteDB |
| Standard | 90 days | YugabyteDB |
| Enterprise | 1 year | YugabyteDB (hot) + Cold storage (archive) |
| Compliance | 7 years | Cold storage (Glacier-equivalent) |

### Audit Log Access

- Audit logs are accessible via **Settings > Audit Log** in the dashboard (admin role required).
- Programmatic access via `GET /api/v1/audit-log` with filtering by action, actor, resource, and time range.
- Audit logs are read-only; they cannot be modified or deleted through the API.

---

## 6. ML Model Integrity Verification

### Model Signing

All trained ML models are cryptographically signed before storage to prevent tampering.

```python
import hashlib
import hmac

def sign_model(model_bytes: bytes, signing_key: str) -> str:
    """Generate HMAC-SHA256 signature for model artifact."""
    return hmac.new(
        signing_key.encode(),
        model_bytes,
        hashlib.sha256
    ).hexdigest()

def verify_model(model_bytes: bytes, signature: str, signing_key: str) -> bool:
    """Verify model artifact has not been tampered with."""
    expected = sign_model(model_bytes, signing_key)
    return hmac.compare_digest(expected, signature)
```

### Model Metadata

Each model artifact is stored with metadata:

```json
{
  "model_name": "isolation_forest",
  "version": "v1.2.0",
  "tenant_id": "tenant-001",
  "trained_at": "2026-02-15T03:00:00Z",
  "training_data_hash": "sha256:abc123...",
  "training_data_range": "2026-01-15 to 2026-02-15",
  "artifact_hash": "sha256:def456...",
  "signature": "hmac-sha256:ghi789...",
  "metrics": {
    "precision": 0.92,
    "recall": 0.87,
    "f1_score": 0.89
  },
  "signed_by": "model-training-pipeline",
  "approved_by": "ml-review-pipeline"
}
```

### Model Loading Verification

On startup and at each model reload, the AI brain verifies:

1. The model artifact hash matches the stored hash.
2. The HMAC signature is valid.
3. The model version matches the expected active version.
4. The model was trained within the acceptable freshness window (default: 30 days).

If verification fails, the AI brain refuses to load the model and falls back to the previous known-good version, logging a critical security alert.

---

## 7. Secure Communication (mTLS Between Services)

### mTLS Architecture

All internal service-to-service communication within the AIOps platform uses mutual TLS (mTLS).

```
┌─────────────┐         mTLS          ┌──────────────┐
│ Go Gateway  │ ◄────────────────────► │  Rust API    │
│             │  Client cert: gateway  │              │
│             │  Server cert: api      │              │
└─────────────┘                        └──────┬───────┘
                                              │
                                         mTLS │
                                              │
                                       ┌──────┴───────┐
                                       │ Python AI    │
                                       │ Brain        │
                                       │              │
                                       └──────────────┘
```

### Certificate Management

| Component | Certificate CN | Issuer | Rotation |
|-----------|---------------|--------|----------|
| Go Gateway | `aiops-gateway.aiops.svc` | Internal CA | 90 days (auto) |
| Rust API | `aiops-api.aiops.svc` | Internal CA | 90 days (auto) |
| Python AI Brain | `aiops-ai-brain.aiops.svc` | Internal CA | 90 days (auto) |
| YugabyteDB | `aiops-yugabyte.aiops.svc` | Internal CA | 90 days (auto) |
| DragonflyDB | `aiops-dragonfly.aiops.svc` | Internal CA | 90 days (auto) |

### Certificate Rotation

Certificates are managed by cert-manager in Kubernetes with automatic rotation:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: aiops-api-cert
  namespace: aiops
spec:
  secretName: aiops-api-tls
  duration: 2160h    # 90 days
  renewBefore: 360h  # Renew 15 days before expiry
  issuerRef:
    name: aiops-internal-ca
    kind: ClusterIssuer
  dnsNames:
    - aiops-api.aiops.svc
    - aiops-api.aiops.svc.cluster.local
  usages:
    - server auth
    - client auth
```

### External Communication

| Path | Protocol | Certificate |
|------|----------|-------------|
| Users -> Gateway | HTTPS (TLS 1.3) | Public CA (Let's Encrypt or organizational CA) |
| AIOps -> GitHub API | HTTPS | GitHub's public certificate |
| AIOps -> Slack/Teams | HTTPS | Service provider's public certificate |
| AIOps -> PagerDuty | HTTPS | PagerDuty's public certificate |
| OTel Collectors -> AIOps | OTLP over mTLS | Internal CA |

---

## 8. Compliance Considerations

### Regulatory Frameworks

| Framework | Relevance | Key Requirements |
|-----------|-----------|-----------------|
| SOC 2 Type II | High | Audit logging, access controls, encryption, monitoring |
| GDPR | Medium | Data minimization, right to erasure, data processing records |
| HIPAA | Conditional | If monitoring healthcare ERP modules: encryption, access controls, audit trails |
| PCI DSS | Conditional | If monitoring payment processing modules: network segmentation, access controls |
| ISO 27001 | High | Information security management, risk assessment, security controls |

### Data Classification

| Data Type | Classification | Handling |
|-----------|---------------|----------|
| Telemetry metrics | Internal | Encrypted at rest and in transit |
| Application logs | Confidential | May contain PII; encrypted, access-controlled |
| Incident details | Internal | Encrypted at rest and in transit |
| RCA reports | Confidential | May reference sensitive data; access-controlled |
| Security findings | Confidential | Vulnerability details; restricted access |
| Audit logs | Restricted | Immutable, long-term retention |
| ML model artifacts | Internal | Signed, integrity-verified |
| Tenant configuration | Confidential | Encrypted, access-controlled |

### Encryption Standards

| Context | Algorithm | Key Size |
|---------|-----------|----------|
| Data at rest (YugabyteDB) | AES-256-GCM | 256-bit |
| Data at rest (RustFS) | AES-256-GCM | 256-bit |
| Data in transit (internal) | TLS 1.3 | ECDHE P-256 |
| Data in transit (external) | TLS 1.3 | ECDHE P-256 |
| JWT signing | RS256 | 2048-bit RSA |
| Webhook signing | HMAC-SHA256 | 256-bit |
| Model signing | HMAC-SHA256 | 256-bit |
| Password hashing | Argon2id | N/A |

### Security Scanning Schedule

| Scan Type | Frequency | Tool | Scope |
|-----------|-----------|------|-------|
| Container image scan | On every build | Trivy | All Docker images |
| Dependency scan | Daily | Dependabot/Renovate | Cargo.toml, requirements.txt, package.json |
| SAST (Static Analysis) | On every PR | CodeQL, Clippy, Ruff | Source code |
| DAST (Dynamic Analysis) | Weekly | OWASP ZAP | Running API endpoints |
| Infrastructure scan | Weekly | Checkov | Helm charts, Kubernetes manifests |
| Penetration test | Annually | External vendor | Full platform |

### Incident Response for Security Events

| Event Type | Response SLA | Notification |
|------------|-------------|-------------|
| Unauthorized access attempt | Investigate within 1 hour | Security team + CISO |
| Data breach suspected | Investigate within 30 minutes | Security team + CISO + Legal |
| Vulnerability (Critical CVE) | Patch within 24 hours | Engineering + Security team |
| Insider threat indicator | Investigate within 2 hours | Security team + HR + Legal |
| Model tampering detected | Halt AI brain within 5 minutes | Engineering + Security team |
