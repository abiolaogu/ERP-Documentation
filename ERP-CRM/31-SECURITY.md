# ERP-CRM Security Policy & Architecture

## 1. Security Overview

The ERP-CRM module implements a defense-in-depth security architecture that protects customer data across all system layers. Security is not an afterthought but a foundational design principle, enforced through compile-time guarantees (Rust's type system and sqlx compile-time query verification), architectural isolation (tenant-scoped data access, hexagonal port boundaries), and operational controls (immutable audit logs, encrypted communication, automated vulnerability scanning).

```mermaid
graph TB
    subgraph "Defense in Depth"
        L1["Layer 1: Network<br/>TLS 1.3, CORS, Rate Limiting"]
        L2["Layer 2: Authentication<br/>JWT (RS256/ES256), OIDC via ERP-IAM"]
        L3["Layer 3: Authorization<br/>RBAC, Tenant Isolation (X-Tenant-ID)"]
        L4["Layer 4: Input Validation<br/>Value Objects, validator crate, sqlx compile-time"]
        L5["Layer 5: Data Protection<br/>AES-256 at rest, PII masking in logs"]
        L6["Layer 6: Audit & Monitoring<br/>Immutable Pulsar audit trail, Quickwit logs"]
    end

    L1 --> L2 --> L3 --> L4 --> L5 --> L6
```

## 2. Supported Versions

| Version | Supported | Notes |
|---------|-----------|-------|
| 0.1.x (current) | Yes | Active development, rolling security updates |
| Pre-release | No | Not recommended for production use |

This repository follows rolling security updates on the active branch (`main`). Security patches are applied as soon as they are available and backported to the latest release.

## 3. Reporting a Vulnerability

### 3.1 Responsible Disclosure Process

```mermaid
flowchart TB
    DISCOVER([Vulnerability<br/>Discovered]) --> REPORT["Report Privately<br/>to Maintainers"]
    REPORT --> TRIAGE["Security Team<br/>Triages (24h)"]
    TRIAGE --> ASSESS{Severity?}

    ASSESS -->|Critical/High| FIX_URGENT["Fix within 48h<br/>CVE assignment"]
    ASSESS -->|Medium| FIX_NORMAL["Fix within 7 days"]
    ASSESS -->|Low| FIX_LOW["Fix in next release"]

    FIX_URGENT --> PATCH["Release Patch<br/>Security Advisory"]
    FIX_NORMAL --> PATCH
    FIX_LOW --> PATCH

    PATCH --> DISCLOSE["Coordinated<br/>Public Disclosure"]
```

### 3.2 What to Include in a Report

- Impact summary describing the vulnerability and its potential consequences
- Detailed reproduction steps with exact inputs and environment details
- Affected version or commit hash
- Any suggested remediation if known
- Your contact information for follow-up

### 3.3 What NOT to Do

- Do not disclose publicly until remediation guidance is available
- Do not exploit the vulnerability against production systems
- Do not access data belonging to other users or tenants
- Do not perform denial-of-service testing against production

## 4. Authentication Architecture

### 4.1 JWT-Based Authentication

```mermaid
sequenceDiagram
    participant Client
    participant CRM as ERP-CRM API
    participant IAM as ERP-IAM
    participant DB as PostgreSQL

    Client->>IAM: POST /auth/token (credentials)
    IAM->>IAM: Validate credentials
    IAM->>Client: JWT Token (RS256 signed)

    Client->>CRM: GET /api/v1/contacts<br/>Authorization: Bearer {jwt}
    CRM->>CRM: Validate JWT signature
    CRM->>CRM: Extract tenant_id, user_id, roles
    CRM->>DB: Query with tenant scope
    DB->>CRM: Results
    CRM->>Client: 200 OK + data
```

### 4.2 Token Structure

```json
{
    "header": {
        "alg": "RS256",
        "typ": "JWT",
        "kid": "erp-iam-key-001"
    },
    "payload": {
        "sub": "user-uuid-v7",
        "iss": "erp-iam",
        "aud": "erp-crm",
        "exp": 1708704000,
        "iat": 1708700400,
        "tenant_id": "tenant-uuid",
        "roles": ["sales_rep", "crm_user"],
        "permissions": [
            "contacts:read",
            "contacts:write",
            "deals:read",
            "deals:write"
        ]
    }
}
```

### 4.3 Token Validation Rules

| Check | Implementation | Failure Response |
|-------|---------------|-----------------|
| Signature verification | RS256 public key from ERP-IAM JWKS | 401 Unauthorized |
| Expiration (`exp`) | `chrono::Utc::now() < exp` | 401 Token Expired |
| Issuer (`iss`) | Must equal `erp-iam` | 401 Invalid Issuer |
| Audience (`aud`) | Must contain `erp-crm` | 401 Invalid Audience |
| Tenant ID | Must be present and valid UUID | 400 Missing Tenant |
| Not Before (`nbf`) | `chrono::Utc::now() >= nbf` | 401 Token Not Yet Valid |

## 5. Authorization Model

### 5.1 Role-Based Access Control (RBAC)

```mermaid
graph TB
    subgraph "Roles"
        ADMIN["Admin<br/>Full access"]
        MANAGER["Sales Manager<br/>Team + pipeline access"]
        REP["Sales Rep<br/>Own contacts + deals"]
        SUPPORT["Support Agent<br/>Tickets + KB"]
        READONLY["Read-Only<br/>View only"]
    end

    subgraph "Permissions"
        P1["contacts:read"]
        P2["contacts:write"]
        P3["contacts:delete"]
        P4["deals:read"]
        P5["deals:write"]
        P6["deals:delete"]
        P7["tickets:read"]
        P8["tickets:write"]
        P9["kb:manage"]
        P10["pipeline:manage"]
        P11["reports:read"]
        P12["admin:manage"]
    end

    ADMIN --> P1 & P2 & P3 & P4 & P5 & P6 & P7 & P8 & P9 & P10 & P11 & P12
    MANAGER --> P1 & P2 & P4 & P5 & P7 & P10 & P11
    REP --> P1 & P2 & P4 & P5 & P7
    SUPPORT --> P1 & P7 & P8 & P9
    READONLY --> P1 & P4 & P7 & P11
```

### 5.2 Ownership-Based Access

Beyond role permissions, data access is further scoped by ownership:

| Role | Contact Access | Deal Access | Ticket Access |
|------|---------------|-------------|---------------|
| Admin | All tenant contacts | All tenant deals | All tenant tickets |
| Manager | Team members' contacts | Team members' deals | All tenant tickets |
| Sales Rep | Own contacts only | Own deals only | Own assigned tickets |
| Support Agent | Read all contacts | Read all deals | Own assigned tickets |
| Read-Only | All tenant contacts (read) | All tenant deals (read) | All tenant tickets (read) |

### 5.3 Tenant Isolation

```mermaid
flowchart TB
    REQUEST([API Request]) --> EXTRACT["Extract X-Tenant-ID<br/>from JWT or Header"]
    EXTRACT --> VALIDATE{Valid<br/>Tenant?}

    VALIDATE -->|No| REJECT["400 Bad Request<br/>Missing tenant context"]
    VALIDATE -->|Yes| SCOPE["Scope All Queries<br/>WHERE tenant_id = $tenant"]

    SCOPE --> QUERY["Execute Query<br/>(Tenant-isolated)"]
    QUERY --> RESPONSE["Return Response<br/>(Only tenant data)"]
```

Every database query is automatically scoped to the tenant extracted from the authentication context. The Go microservices enforce this via the `X-Tenant-ID` header:

```go
func requireTenantID(r *http.Request) (string, error) {
    tenantID := r.Header.Get("X-Tenant-ID")
    if tenantID == "" {
        return "", fmt.Errorf("X-Tenant-ID header is required")
    }
    return tenantID, nil
}
```

## 6. Input Validation & Injection Prevention

### 6.1 Compile-Time SQL Injection Prevention

The ERP-CRM uses `sqlx` with compile-time query verification. All SQL queries are checked against the actual PostgreSQL schema at compile time, eliminating SQL injection as a category of vulnerability.

```rust
// SAFE: sqlx compile-time verified, parameterized query
let contact = sqlx::query_as!(
    Contact,
    "SELECT * FROM contacts WHERE id = $1 AND tenant_id = $2",
    contact_id,    // Parameter $1 - automatically escaped
    tenant_id      // Parameter $2 - automatically escaped
)
.fetch_optional(&state.db)
.await?;
```

### 6.2 Value Object Validation

All domain inputs pass through validated value objects before reaching the database:

```mermaid
flowchart LR
    INPUT["Raw Input<br/>(JSON String)"] --> DESER["Deserialize<br/>(serde)"]
    DESER --> VALIDATE["Validate<br/>(Value Object)"]
    VALIDATE -->|Valid| DOMAIN["Domain<br/>Operation"]
    VALIDATE -->|Invalid| REJECT2["400 Bad Request<br/>Validation Error"]
```

| Value Object | Validation Rules | Prevents |
|-------------|-----------------|----------|
| `Email` | Contains `@`, domain has dot, non-empty, lowercase normalized | Invalid email storage, case-sensitivity bugs |
| `Phone` | Non-empty, valid phone characters only | Invalid phone storage |
| `Address` | Structured fields (street, city, state, country, postal_code) | Unstructured address data |
| `Money` | Non-negative amount, valid ISO 4217 currency | Negative deal values, invalid currencies |
| `EntityId` | Valid UUID format | Path traversal, injection |
| `LeadScore` | Integer 0-100, clamped at boundaries | Score overflow, invalid ranges |
| `Probability` | Integer 0-100, clamped at boundaries | Probability overflow |

### 6.3 Request Body Validation

```rust
#[derive(Debug, Deserialize, Validate)]
pub struct CreateContactRequest {
    #[validate(email)]
    pub email: String,

    #[validate(length(min = 1, max = 100))]
    pub first_name: Option<String>,

    #[validate(length(min = 1, max = 100))]
    pub last_name: Option<String>,

    #[validate(length(max = 50))]
    pub phone: Option<String>,

    #[validate(range(min = 0, max = 100))]
    pub lead_score: Option<i32>,
}
```

## 7. Encryption

### 7.1 Encryption at Rest

```mermaid
graph TB
    subgraph "Data at Rest Encryption"
        PG_TDE["PostgreSQL<br/>Transparent Data Encryption<br/>AES-256"]
        BACKUP_ENC["Backup Files<br/>AES-256-GCM<br/>Encrypted before transfer"]
        VOLUME_ENC["Mayastor Volumes<br/>dm-crypt / LUKS<br/>Block-level encryption"]
        PULSAR_ENC["Pulsar Persistent<br/>Storage Encryption<br/>AES-256"]
    end
```

| Data Store | Encryption Method | Key Management |
|-----------|-------------------|----------------|
| PostgreSQL 16 | TDE (AES-256) | Master key in ERP-IAM vault |
| Backup dumps | AES-256-GCM | Separate backup encryption key |
| Mayastor volumes | dm-crypt LUKS | Node-level key via Harvester |
| Pulsar bookkeeper | AES-256 | Pulsar key management |

### 7.2 Encryption in Transit

```mermaid
graph TB
    subgraph "In-Transit Encryption"
        API_TLS["API Communication<br/>TLS 1.3<br/>Certificate pinning"]
        NATS_TLS["NATS JetStream<br/>TLS mutual auth<br/>Client certificates"]
        PULSAR_TLS["Apache Pulsar<br/>TLS 1.3<br/>mTLS between brokers"]
        DB_SSL["PostgreSQL<br/>SSL required<br/>sslmode=require"]
        INTER_SVC["Inter-Service<br/>mTLS via service mesh"]
    end
```

| Communication Path | Protocol | Minimum Version |
|-------------------|----------|----------------|
| Client to API Gateway | HTTPS | TLS 1.3 |
| API Gateway to CRM | HTTPS | TLS 1.2+ |
| CRM to PostgreSQL | PostgreSQL SSL | sslmode=require |
| CRM to NATS | NATS TLS | TLS 1.2+ |
| CRM to Pulsar | Pulsar TLS | TLS 1.3 |
| Go services to Go services | mTLS | TLS 1.2+ |

### 7.3 Application-Level Cryptography

| Purpose | Algorithm | Key Size | Usage |
|---------|-----------|----------|-------|
| JWT signing | RS256 (RSA + SHA-256) | 2048-bit RSA | Authentication tokens |
| JWT signing (alternative) | ES256 (ECDSA + SHA-256) | P-256 curve | Compact tokens |
| Webhook HMAC | HMAC-SHA256 | 256-bit | Webhook payload verification |
| Password hashing | Argon2id | N/A | User password storage (ERP-IAM) |
| API key generation | CSPRNG | 256-bit | Service-to-service auth |

## 8. Secrets Management

### 8.1 Secret Storage Policy

```mermaid
flowchart TB
    subgraph "Secrets Lifecycle"
        CREATE["Create Secret<br/>(Generated or provisioned)"]
        STORE["Store Securely<br/>(K8s Secret / Vault)"]
        INJECT["Inject at Runtime<br/>(Environment variable)"]
        ROTATE["Rotate Periodically<br/>(Automated schedule)"]
        REVOKE["Revoke on<br/>Compromise"]
    end

    CREATE --> STORE --> INJECT --> ROTATE
    STORE --> REVOKE
```

| Secret Type | Storage Location | Rotation Period | Access |
|------------|-----------------|----------------|--------|
| Database credentials | K8s Secret / Vault | 90 days | CRM application only |
| JWT signing keys | ERP-IAM Vault | 365 days | ERP-IAM only |
| NATS credentials | K8s Secret | 90 days | CRM application only |
| Pulsar auth tokens | K8s Secret | 90 days | CRM application only |
| GHCR token | GitHub Secrets | On change | CI/CD only |
| API keys | K8s Secret / Vault | 180 days | Service consumers |

### 8.2 Secret Handling Rules

1. **Never commit secrets** to source control. Use environment variables exclusively.
2. **No secrets in Docker images.** All secrets are injected via environment at runtime.
3. **No secrets in logs.** The structured logging schema explicitly excludes secret fields.
4. **No secrets in error messages.** Error responses never expose internal credentials.
5. **Rotate on exposure.** Any suspected compromise triggers immediate rotation.

```bash
# .env.example (committed, no real secrets)
DATABASE_URL=postgres://postgres:postgres@localhost:5432/crm
NATS_URL=nats://localhost:4222
RUST_LOG=debug
PORT=8081

# .env (NOT committed, contains real values)
DATABASE_URL=postgres://crm_user:REAL_PASSWORD@prod-db:5432/crm
NATS_URL=nats://crm_nats:REAL_TOKEN@prod-nats:4222
```

## 9. Audit Logging

### 9.1 Audit Event Architecture

```mermaid
flowchart TB
    subgraph "Audit Event Flow"
        API_ACTION["API Operation<br/>(CRUD action)"]
        DOMAIN_EVT["Domain Event<br/>Raised by Aggregate"]
        CLOUD_EVT["CloudEvent<br/>Formatted"]
        NATS_PUB["NATS JetStream<br/>Real-time publish"]
        PULSAR_PUB["Pulsar Audit Topic<br/>Durable storage"]
        QUICKWIT["Quickwit<br/>Searchable index"]
    end

    API_ACTION --> DOMAIN_EVT --> CLOUD_EVT
    CLOUD_EVT --> NATS_PUB
    CLOUD_EVT --> PULSAR_PUB --> QUICKWIT
```

### 9.2 Audited Events (60+ Types)

| Category | Events | Sensitivity |
|----------|--------|------------|
| Contact | created, updated, deleted, qualified, converted, merged, ownership_transferred, lead_score_changed | High |
| Deal | created, updated, deleted, stage_changed, won, lost, amount_changed, reopened | High |
| Account | created, updated, deleted, contact_linked, deal_linked | Medium |
| Ticket | created, updated, assigned, commented, solved, closed, reopened, escalated | Medium |
| KB Article | created, updated, published, unpublished, deleted | Low |
| Form | created, updated, deleted, submission_received | Medium |
| Auth | login, logout, token_refresh, failed_login, password_change | Critical |
| Admin | user_created, user_deleted, role_changed, permission_changed | Critical |

### 9.3 Audit Event Schema (CloudEvents v1.0)

```json
{
    "specversion": "1.0",
    "type": "erp.crm.contact.created",
    "source": "/erp-crm/contacts",
    "id": "evt-uuid-v7",
    "time": "2026-02-23T10:30:00Z",
    "datacontenttype": "application/json",
    "tenantid": "tenant-uuid",
    "data": {
        "contact_id": "contact-uuid",
        "email": "user@example.com",
        "owner_id": "owner-uuid",
        "performed_by": "user-uuid",
        "ip_address": "192.168.1.1"
    }
}
```

### 9.4 Audit Log Retention

| Log Type | Retention | Immutability | Storage |
|----------|-----------|-------------|---------|
| Security events (auth) | 7 years | Immutable (Pulsar) | Off-site archive |
| Data modification events | 7 years | Immutable (Pulsar) | Off-site archive |
| Access events (read) | 1 year | Immutable (Pulsar) | Standard storage |
| System events | 90 days | Append-only (Quickwit) | Local cluster |

## 10. PII Protection

### 10.1 PII Data Flow

```mermaid
flowchart TB
    subgraph "PII Handling"
        INPUT_PII["PII Input<br/>(email, phone, name, address)"]
        VALIDATE_PII["Validate via<br/>Value Objects"]
        STORE_PII["Store Encrypted<br/>(PostgreSQL TDE)"]
        LOG_PII["Log Masked<br/>(PII stripped)"]
        EXPORT_PII["Export on Request<br/>(GDPR Article 15)"]
        DELETE_PII["Delete on Request<br/>(GDPR Article 17)"]
    end

    INPUT_PII --> VALIDATE_PII
    VALIDATE_PII --> STORE_PII
    VALIDATE_PII --> LOG_PII
    STORE_PII --> EXPORT_PII
    STORE_PII --> DELETE_PII
```

### 10.2 PII Classification

| Field | Sensitivity | Encrypted at Rest | Masked in Logs | Exportable | Deletable |
|-------|-----------|-------------------|----------------|-----------|-----------|
| Email | High PII | Yes (TDE) | Yes | Yes | Yes (CASCADE) |
| Phone | High PII | Yes (TDE) | Yes | Yes | Yes (CASCADE) |
| First/Last Name | High PII | Yes (TDE) | Yes | Yes | Yes (CASCADE) |
| Address | High PII | Yes (TDE) | Yes | Yes | Yes (CASCADE) |
| Company Name | Medium PII | Yes (TDE) | No | Yes | Yes |
| Job Title | Medium PII | Yes (TDE) | No | Yes | Yes |
| IP Address | Medium PII | Yes (TDE) | Yes | Yes | Yes (set NULL) |
| Deal Amount | Financial | Yes (TDE) | No | Yes | No (7yr retention) |

### 10.3 Log Masking Implementation

```rust
/// Mask PII fields in log output
fn mask_pii(email: &str) -> String {
    if let Some(at_pos) = email.find('@') {
        let local = &email[..at_pos];
        let domain = &email[at_pos..];
        if local.len() <= 2 {
            format!("**{}", domain)
        } else {
            format!("{}...{}{}", &local[..1], &local[local.len()-1..], domain)
        }
    } else {
        "***".to_string()
    }
}

// Usage in structured logging
tracing::info!(
    contact_id = %contact.id,
    email = %mask_pii(&contact.email),
    action = "contact.created",
    "Contact created"
);
// Output: contact_id=abc-123 email=j...e@example.com action=contact.created Contact created
```

## 11. Network Security

### 11.1 CORS Configuration

```rust
// Tower HTTP CORS layer
let cors = CorsLayer::new()
    .allow_origin(AllowOrigin::list([
        "https://app.example.com".parse().unwrap(),
        "https://admin.example.com".parse().unwrap(),
    ]))
    .allow_methods([Method::GET, Method::POST, Method::PUT, Method::DELETE, Method::OPTIONS])
    .allow_headers([CONTENT_TYPE, AUTHORIZATION, HeaderName::from_static("x-tenant-id")])
    .max_age(Duration::from_secs(3600));
```

### 11.2 Rate Limiting

| Endpoint Category | Rate Limit | Window | Burst |
|------------------|-----------|--------|-------|
| Authentication | 10 req | 1 minute | 15 |
| Contact CRUD | 100 req | 1 minute | 150 |
| List/Search | 60 req | 1 minute | 100 |
| Bulk operations | 10 req | 1 minute | 15 |
| Health/Ready | Unlimited | N/A | N/A |

### 11.3 Network Isolation

```mermaid
graph TB
    subgraph "External Network"
        CLIENT["Client<br/>(Browser/Mobile)"]
    end

    subgraph "DMZ"
        LB["Load Balancer<br/>+ WAF"]
        GATEWAY["API Gateway<br/>TLS Termination"]
    end

    subgraph "Application Network"
        CRM["CRM Core<br/>(Port 8081)"]
        GO_SVC["Go Microservices<br/>(Ports 8090-8101)"]
    end

    subgraph "Data Network (Isolated)"
        PG["PostgreSQL<br/>(Port 5432)"]
        NATS2["NATS<br/>(Port 4222)"]
        PULSAR2["Pulsar<br/>(Port 6650)"]
    end

    CLIENT -->|HTTPS| LB -->|HTTPS| GATEWAY
    GATEWAY -->|HTTP/2| CRM
    GATEWAY -->|HTTP/2| GO_SVC
    CRM -->|SSL| PG
    CRM -->|TLS| NATS2
    CRM -->|TLS| PULSAR2
    GO_SVC -->|SSL| PG
```

## 12. Dependency Security

### 12.1 Supply Chain Security

```mermaid
flowchart TB
    subgraph "Dependency Security Pipeline"
        AUDIT_DEP["cargo audit<br/>CVE database check"]
        DENY_DEP["cargo deny<br/>License + advisory"]
        UPDATE_DEP["Dependabot<br/>Automated updates"]
        REVIEW_DEP["Manual Review<br/>New dependencies"]
    end

    AUDIT_DEP --> CI_GATE["CI Gate<br/>Block on critical CVE"]
    DENY_DEP --> CI_GATE
    UPDATE_DEP --> PR_REVIEW["PR Review"]
    REVIEW_DEP --> PR_REVIEW
```

### 12.2 Dependency Audit Commands

```bash
# Check for known vulnerabilities
cargo audit

# Check license compliance and advisories
cargo deny check

# Update dependencies to latest compatible versions
cargo update

# Check for outdated dependencies
cargo outdated
```

### 12.3 Approved Dependency Categories

| Category | Approved Crates | Justification |
|----------|----------------|---------------|
| Async runtime | tokio | Industry standard async runtime |
| HTTP framework | axum | Tower-based, well-maintained |
| Database | sqlx | Compile-time verified queries |
| Serialization | serde, serde_json | De facto standard |
| Datetime | chrono | Comprehensive timezone support |
| UUID | uuid | RFC 4122 compliance |
| Error handling | thiserror, anyhow | Ergonomic error types |
| Logging | tracing | Structured, async-aware |
| Validation | validator | Derive-based validation |
| Decimal | rust_decimal | Financial precision |
| Concurrency | dashmap | Thread-safe HashMap |

## 13. Container Security

### 13.1 Docker Image Hardening

```mermaid
flowchart TB
    subgraph "Multi-Stage Build Security"
        BUILDER["Stage 1: Builder<br/>rust:1.75-bookworm<br/>Full toolchain (discarded)"]
        RUNTIME["Stage 2: Runtime<br/>debian:bookworm-slim<br/>~50MB, minimal attack surface"]
    end

    subgraph "Runtime Hardening"
        NO_ROOT["Non-root user<br/>appuser (UID 10001)"]
        NO_SHELL["No shell<br/>(optional: scratch base)"]
        READ_ONLY["Read-only filesystem<br/>(K8s securityContext)"]
        MIN_DEPS["Minimal dependencies<br/>ca-certificates, libssl3"]
    end

    BUILDER --> RUNTIME
    RUNTIME --> NO_ROOT & NO_SHELL & READ_ONLY & MIN_DEPS
```

**Dockerfile Security Features:**

```dockerfile
# Multi-stage build - builder stage is discarded
FROM rust:1.75-bookworm AS builder
WORKDIR /app
# ... build steps ...
RUN cargo build --release

# Minimal runtime image
FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates libssl3 \
    && rm -rf /var/lib/apt/lists/*

# Non-root user
RUN useradd -r -u 10001 -U appuser
USER appuser

COPY --from=builder /app/target/release/sase-crm /usr/local/bin/
COPY --from=builder /app/migrations /app/migrations

ENTRYPOINT ["/usr/local/bin/sase-crm"]
```

### 13.2 Kubernetes Security Context

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crm-core
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        fsGroup: 10001
      containers:
        - name: crm
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: [ALL]
          resources:
            limits:
              memory: "512Mi"
              cpu: "500m"
            requests:
              memory: "256Mi"
              cpu: "250m"
```

## 14. Incident Response

### 14.1 Security Incident Classification

| Severity | Description | Response Time | Examples |
|----------|-----------|--------------|---------|
| P1 - Critical | Active data breach, system compromise | 15 min | Unauthorized data access, credential leak |
| P2 - High | Vulnerability with exploit potential | 1 hour | SQL injection discovery, auth bypass |
| P3 - Medium | Security misconfiguration | 4 hours | CORS misconfiguration, missing rate limit |
| P4 - Low | Minor security improvement | Next sprint | Dependency update, log improvement |

### 14.2 Incident Response Flow

```mermaid
flowchart TB
    DETECT2([Incident<br/>Detected]) --> CONTAIN2["1. Contain<br/>(15 min)"]
    CONTAIN2 --> ISOLATE["Isolate affected<br/>pods/services"]
    ISOLATE --> ASSESS3["2. Assess<br/>(30 min)"]
    ASSESS3 --> SCOPE["Determine scope:<br/>- Data affected?<br/>- Tenants impacted?<br/>- Duration of exposure?"]
    SCOPE --> NOTIFY2["3. Notify<br/>(1 hour)"]
    NOTIFY2 --> NOTIFY_INT2["Internal: Security team,<br/>management, legal"]
    NOTIFY_INT2 --> GDPR_CHECK{EU Data<br/>Subjects?}
    GDPR_CHECK -->|Yes| DPA_NOTIFY["Notify DPA<br/>within 72 hours"]
    GDPR_CHECK -->|No| SKIP
    DPA_NOTIFY --> REMEDIATE2["4. Remediate"]
    SKIP --> REMEDIATE2
    REMEDIATE2 --> FIX["Fix vulnerability,<br/>rotate credentials,<br/>patch & deploy"]
    FIX --> POSTMORTEM2["5. Post-Mortem<br/>(within 5 days)"]
    POSTMORTEM2 --> IMPROVE3["6. Improve<br/>Update controls,<br/>add tests"]
```

## 15. Compliance Control Mapping

### 15.1 SOC 2 Type II Controls

| Control | Category | ERP-CRM Implementation |
|---------|----------|----------------------|
| CC6.1 | Logical Access | JWT authentication, RBAC permissions |
| CC6.2 | Access Provisioning | User provisioning via ERP-Directory |
| CC6.3 | Access Modification | Ownership transfer with audit events |
| CC6.6 | Access Revocation | Token expiry, immediate revocation |
| CC7.1 | System Monitoring | Quickwit centralized logging |
| CC7.2 | Anomaly Detection | Event pattern monitoring |
| CC7.3 | Incident Response | Documented runbooks, escalation matrix |
| CC8.1 | Change Management | Git-tracked, PR reviews, CI gates |

### 15.2 HIPAA Controls (if handling PHI)

| Control | Requirement | Implementation |
|---------|------------|---------------|
| 164.312(a)(1) | Access Control | RBAC + tenant isolation |
| 164.312(b) | Audit Controls | Immutable Pulsar audit trail |
| 164.312(c)(1) | Integrity | Event immutability, signed payloads |
| 164.312(d) | Person Authentication | MFA via ERP-IAM |
| 164.312(e)(1) | Transmission Security | TLS 1.3 everywhere |

### 15.3 PCI-DSS Controls

| Requirement | Control | Implementation |
|------------|---------|---------------|
| 7.1 | Restrict Access | Role-based, need-to-know |
| 8.2 | Authentication | JWT RS256 signing |
| 10.1 | Audit Trail | All CRUD operations logged |
| 10.5 | Log Integrity | Pulsar immutable topics |
| 10.7 | Log Retention | 1+ year (configurable) |

## 16. Security Checklist

### 16.1 Development Checklist

- [ ] All SQL queries use parameterized placeholders (`$1`, `$2`)
- [ ] All inputs validated through value objects or `validator` crate
- [ ] No secrets hardcoded in source code
- [ ] All new endpoints have authentication/authorization checks
- [ ] PII fields masked in all log statements
- [ ] Error responses do not leak internal details
- [ ] CORS headers properly configured for new endpoints
- [ ] Rate limiting applied to new endpoints

### 16.2 Deployment Checklist

- [ ] TLS certificates valid and not expiring within 30 days
- [ ] Database credentials rotated per schedule
- [ ] Container running as non-root user
- [ ] Read-only filesystem enabled
- [ ] Resource limits configured (CPU/memory)
- [ ] Network policies restrict traffic to required paths only
- [ ] Audit logging verified for all operations
- [ ] Health and readiness probes responding

### 16.3 Periodic Review Checklist

- [ ] `cargo audit` shows no critical vulnerabilities (weekly)
- [ ] Dependency versions current (monthly)
- [ ] Access reviews completed for all roles (quarterly)
- [ ] Penetration test conducted (annually)
- [ ] Disaster recovery drill completed (semi-annually)
- [ ] Security training completed by all developers (annually)

## 17. Evidence Artifacts

| Artifact | Location | Purpose |
|----------|----------|------------|
| Security Policy | `/ERP-CRM/SECURITY.md` | This document |
| Compliance Mapping | `/ERP-CRM/COMPLIANCE.md` | Regulatory control mapping |
| CI Pipeline | `/ERP-CRM/.github/workflows/ci.yml` | Automated security checks |
| Dockerfile | `/ERP-CRM/Dockerfile` | Container hardening evidence |
| Audit Topic Config | `/ERP-CRM/eventing/pulsar/topics.yaml` | Audit trail configuration |
| Log Schema | `/ERP-CRM/observability/log-schema.json` | Structured log format |
