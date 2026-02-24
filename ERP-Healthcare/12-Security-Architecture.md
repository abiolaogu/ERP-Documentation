# Security Architecture - AfriHealth ERP-Healthcare

## 1. Overview

AfriHealth implements a defense-in-depth security architecture protecting sensitive healthcare data (PHI/PII) across all layers. The security model addresses HIPAA, GDPR, NDPA, and POPIA requirements through encryption, access control, audit logging, and network segmentation.

---

## 2. Security Architecture Layers

```mermaid
graph TB
    subgraph "Layer 1: Perimeter Security"
        WAF[AWS WAF<br/>DDoS Protection]
        CDN[CloudFront CDN<br/>TLS Termination]
        SHIELD[AWS Shield Advanced<br/>DDoS Mitigation]
    end

    subgraph "Layer 2: Network Security"
        VPC[VPC af-south-1<br/>10.0.0.0/16]
        PUB[Public Subnets<br/>ALB Only]
        PRIV[Private Subnets<br/>Application Services]
        DATA[Data Subnets<br/>Databases Only]
        SG[Security Groups<br/>Per Service]
        NACL[Network ACLs<br/>Subnet Level]
    end

    subgraph "Layer 3: Application Security"
        GW[API Gateway<br/>Rate Limiting + Auth]
        JWT[JWT RS256 Tokens<br/>15-min Expiry]
        RBAC[RBAC + ABAC<br/>Role + Attribute]
        MTLS[mTLS Between Services]
        CORS[CORS Policies]
    end

    subgraph "Layer 4: Data Security"
        TLS[TLS 1.3<br/>In Transit]
        AES[AES-256<br/>At Rest]
        FLE[Field-Level Encryption<br/>SSN, National ID]
        HASH[Biometric Hashing<br/>One-Way bcrypt]
        KMS[AWS KMS<br/>Key Management]
    end

    subgraph "Layer 5: Monitoring & Response"
        AUDIT[Audit Logging<br/>All PHI Access]
        SIEM[SIEM Integration<br/>CloudWatch + GuardDuty]
        IDS[Intrusion Detection<br/>GuardDuty]
        RESP[Incident Response<br/>Automated Playbooks]
    end

    WAF --> CDN --> VPC
    VPC --> PUB --> PRIV --> DATA
    PUB --> GW
    GW --> JWT --> RBAC
    PRIV --> MTLS
    DATA --> AES --> KMS
    PRIV --> AUDIT --> SIEM
```

---

## 3. Authentication Architecture

### 3.1 Authentication Flow

```mermaid
sequenceDiagram
    participant Client as Client App
    participant GW as API Gateway
    participant Auth as Auth Service
    participant IDP as Identity Provider
    participant DB as User Database
    participant Redis as Token Cache

    Client->>GW: POST /auth/login {email, password}
    GW->>Auth: Forward request
    Auth->>DB: Lookup user by email
    DB-->>Auth: User record (hashed password)
    Auth->>Auth: Verify password (bcrypt)
    Auth->>Auth: Check MFA requirement
    alt MFA Required
        Auth-->>Client: 202 MFA Required {mfa_token}
        Client->>Auth: POST /auth/mfa/verify {mfa_token, otp_code}
        Auth->>Auth: Verify TOTP code
    end
    Auth->>Auth: Generate JWT (RS256, 15-min expiry)
    Auth->>Auth: Generate Refresh Token (7-day expiry)
    Auth->>Redis: Store refresh token (with rotation ID)
    Auth-->>Client: 200 {access_token, refresh_token, expires_in}

    Note over Client,Auth: Subsequent API Calls
    Client->>GW: GET /api/patients {Authorization: Bearer <JWT>}
    GW->>GW: Validate JWT signature (RS256 public key)
    GW->>GW: Check token expiry
    GW->>GW: Extract tenant_id, user_id, roles
    GW->>Auth: Forward to service with context
```

### 3.2 JWT Token Structure

```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT",
    "kid": "afrihealth-prod-2024-key-1"
  },
  "payload": {
    "sub": "user-uuid-here",
    "iss": "afrihealth.com",
    "aud": "afrihealth-api",
    "exp": 1708000000,
    "iat": 1707999100,
    "tenant_id": "tenant-uuid-here",
    "roles": ["doctor", "department_head"],
    "department_id": "dept-uuid",
    "facility_id": "facility-uuid",
    "permissions": [
      "patient:read",
      "patient:write",
      "prescription:create",
      "lab:order"
    ]
  }
}
```

### 3.3 Token Refresh and Rotation

```mermaid
flowchart TB
    A[Access Token Expired] --> B[Client Sends Refresh Token]
    B --> C[Auth Service Validates Refresh Token]
    C --> D{Token Valid?}
    D -->|Yes| E[Generate New Access Token]
    E --> F[Generate New Refresh Token]
    F --> G[Invalidate Old Refresh Token]
    G --> H[Return New Token Pair]
    D -->|No| I{Reuse Detected?}
    I -->|Yes| J[Invalidate ALL Tokens for User]
    J --> K[Force Re-authentication]
    I -->|No| L[Return 401 Unauthorized]
```

---

## 4. Authorization Model (RBAC + ABAC)

### 4.1 Role Hierarchy

```mermaid
graph TB
    SA[System Admin] --> TA[Tenant Admin]
    TA --> HA[Hospital Admin]
    HA --> DH[Department Head]
    DH --> DOC[Doctor/Physician]
    DH --> NUR[Nurse]
    DH --> LAB[Lab Technician]
    DH --> PHR[Pharmacist]
    HA --> BIL[Billing Clerk]
    HA --> REG[Registration Clerk]
    HA --> HMO[HMO Officer]
    TA --> CC[Call Center Agent]
    TA --> PH[Public Health Officer]

    style SA fill:#c62828,color:#fff
    style TA fill:#e65100,color:#fff
    style HA fill:#1565c0,color:#fff
```

### 4.2 Permission Matrix

| Resource | System Admin | Tenant Admin | Doctor | Nurse | Lab Tech | Pharmacist | Patient |
|----------|-------------|-------------|--------|-------|----------|------------|---------|
| Patient Demographics | CRUD | CRUD | Read | Read | Read | Read | Own |
| Clinical Notes | CRUD | Read | CRUD (own dept) | Read | - | - | Own (read) |
| Lab Orders | CRUD | Read | Create/Read | Read | CRUD | Read | Own (read) |
| Prescriptions | CRUD | Read | Create/Read | Read | - | CRUD | Own (read) |
| Billing | CRUD | CRUD | Read | - | - | - | Own |
| Audit Logs | Read | Read | - | - | - | - | - |
| System Config | CRUD | Read | - | - | - | - | - |
| AI Results | CRUD | Read | Read/Override | Read | Read | - | Own (read) |

### 4.3 Attribute-Based Access Control (ABAC)

```go
// ABAC Policy Evaluation
type AccessPolicy struct {
    Subject    SubjectAttributes    // Who is requesting
    Resource   ResourceAttributes   // What is being accessed
    Action     string               // CRUD operation
    Environment EnvironmentAttributes // When/where
}

type SubjectAttributes struct {
    UserID       string
    TenantID     string
    Roles        []string
    DepartmentID string
    FacilityID   string
    Clearance    string // standard, elevated, emergency
}

type ResourceAttributes struct {
    ResourceType string   // patient, encounter, lab_result
    OwnerID      string   // patient who owns this data
    TenantID     string   // tenant this belongs to
    Department   string   // department context
    Sensitivity  string   // normal, restricted, confidential
}

// Policy rules examples:
// 1. Doctor can only access patients in their department
// 2. Lab tech can only see lab results, not full patient chart
// 3. Emergency override requires justification logging
// 4. Cross-tenant access is never allowed
// 5. Mental health records require explicit consent check
```

---

## 5. Break-the-Glass Emergency Access

```mermaid
flowchart TB
    A[Emergency Situation] --> B[Provider Requests Emergency Access]
    B --> C[System Displays Warning]
    C --> D[Provider Enters Justification]
    D --> E[System Grants Temporary Access<br/>4-Hour Window]
    E --> F[All Actions Logged with<br/>EMERGENCY_ACCESS Flag]
    F --> G[Automatic Alert to:<br/>Privacy Officer +<br/>Department Head +<br/>Compliance Team]
    G --> H[Post-Incident Review<br/>Within 24 Hours]
    H --> I{Justified?}
    I -->|Yes| J[Document + Close]
    I -->|No| K[Disciplinary Action +<br/>Report to Regulator]
```

---

## 6. Encryption Architecture

### 6.1 Encryption at Rest

| Data Type | Encryption Method | Key Management |
|-----------|------------------|----------------|
| RDS Database | AES-256 (RDS encryption) | AWS KMS (CMK) |
| S3 Objects (images, documents) | AES-256 (SSE-KMS) | AWS KMS (CMK) |
| EBS Volumes | AES-256 (EBS encryption) | AWS KMS (CMK) |
| Redis Cache | TLS + at-rest encryption | ElastiCache encryption |
| National ID / SSN fields | AES-256-GCM (field-level) | Application-managed via KMS |
| Biometric data | bcrypt hash (one-way) | N/A (irreversible) |
| Blockchain ledger | Fabric channel encryption | Fabric MSP certificates |

### 6.2 Encryption in Transit

```mermaid
flowchart LR
    subgraph "External Traffic"
        C[Client] -->|TLS 1.3| ALB[Application<br/>Load Balancer]
    end

    subgraph "Internal Traffic"
        ALB -->|TLS 1.3| GW[API Gateway]
        GW -->|mTLS| S1[Service A]
        S1 -->|mTLS| S2[Service B]
        S1 -->|TLS| DB[(PostgreSQL)]
        S1 -->|TLS| RD[(Redis)]
        S1 -->|TLS| RP[Redpanda]
    end

    subgraph "Cross-Region"
        DB -->|TLS 1.3 + VPN| REP[(Read Replica<br/>DR Region)]
    end
```

### 6.3 Key Rotation Policy

| Key Type | Rotation Period | Mechanism |
|----------|----------------|-----------|
| KMS Master Key | Annual (automatic) | AWS KMS auto-rotation |
| JWT Signing Key (RSA) | Quarterly | Key ID in JWT header (kid) |
| Database Encryption Key | Annual | RDS re-encryption |
| TLS Certificates | 90 days | ACM auto-renewal |
| Service mTLS Certificates | 30 days | cert-manager auto-renewal |
| API Keys | 90 days | Force rotation via API |
| Refresh Tokens | 7 days | Token rotation on use |

---

## 7. Network Security

### 7.1 VPC Architecture

```mermaid
graph TB
    subgraph "VPC 10.0.0.0/16"
        subgraph "Public Subnets (10.0.1.0/24, 10.0.2.0/24)"
            ALB[Application Load Balancer]
            NAT[NAT Gateway]
        end

        subgraph "Private Subnets (10.0.10.0/24, 10.0.11.0/24)"
            EKS[EKS Worker Nodes]
            SVC[Application Services]
        end

        subgraph "Data Subnets (10.0.20.0/24, 10.0.21.0/24)"
            RDS[(RDS PostgreSQL<br/>Multi-AZ)]
            REDIS[(ElastiCache Redis)]
            ES[(Elasticsearch)]
        end
    end

    INET[Internet] --> ALB
    ALB --> EKS
    EKS --> SVC
    SVC --> RDS & REDIS & ES
    SVC --> NAT --> INET
```

### 7.2 Security Group Rules

| Security Group | Inbound | Outbound |
|---------------|---------|----------|
| ALB-SG | 443 from 0.0.0.0/0 | 8080 to EKS-SG |
| EKS-SG | 8080 from ALB-SG | 5432 to RDS-SG, 6379 to Redis-SG |
| RDS-SG | 5432 from EKS-SG | None |
| Redis-SG | 6379 from EKS-SG | None |
| Bastion-SG | 22 from VPN CIDR | All to VPC CIDR |

---

## 8. API Security

### 8.1 Rate Limiting

| Endpoint Category | Rate Limit | Window |
|-------------------|-----------|--------|
| Authentication | 10 requests | per minute per IP |
| Patient API | 100 requests | per minute per user |
| Lab API | 200 requests | per minute per user |
| AI Inference | 20 requests | per minute per user |
| Bulk Operations | 5 requests | per minute per user |
| Webhook Callbacks | 50 requests | per minute per tenant |

### 8.2 Input Validation and Sanitization

```go
// Input validation middleware
func ValidateRequest(c *gin.Context) {
    // 1. Content-Type validation
    if c.ContentType() != "application/json" {
        c.AbortWithStatusJSON(415, gin.H{"error": "Unsupported media type"})
        return
    }

    // 2. Request body size limit (1MB for standard, 50MB for image uploads)
    c.Request.Body = http.MaxBytesReader(c.Writer, c.Request.Body, maxBodySize)

    // 3. SQL injection prevention (parameterized queries via GORM)
    // 4. XSS prevention (HTML escaping in responses)
    // 5. Path traversal prevention (whitelist allowed paths)
    // 6. SSRF prevention (whitelist allowed external URLs)

    c.Next()
}
```

### 8.3 CORS Configuration

```go
corsConfig := cors.Config{
    AllowOrigins:     []string{"https://app.afrihealth.com", "https://admin.afrihealth.com"},
    AllowMethods:     []string{"GET", "POST", "PUT", "PATCH", "DELETE"},
    AllowHeaders:     []string{"Authorization", "Content-Type", "X-Tenant-ID", "X-Request-ID"},
    ExposeHeaders:    []string{"X-Request-ID", "X-RateLimit-Remaining"},
    AllowCredentials: true,
    MaxAge:           12 * time.Hour,
}
```

---

## 9. Audit and Compliance Logging

### 9.1 Audit Log Schema

```sql
CREATE TABLE audit_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    event_type      VARCHAR(50) NOT NULL,  -- CREATE, READ, UPDATE, DELETE
    table_name      VARCHAR(100) NOT NULL,
    record_id       UUID,
    user_id         UUID NOT NULL,
    user_role       VARCHAR(50),
    old_values      JSONB,
    new_values      JSONB,
    ip_address      INET,
    user_agent      TEXT,
    session_id      VARCHAR(100),
    access_type     VARCHAR(20),  -- NORMAL, EMERGENCY, SYSTEM
    justification   TEXT,         -- Required for emergency access
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Indexed for compliance queries
CREATE INDEX idx_audit_log_tenant_date ON audit_log (tenant_id, created_at);
CREATE INDEX idx_audit_log_user ON audit_log (user_id, created_at);
CREATE INDEX idx_audit_log_record ON audit_log (table_name, record_id);
CREATE INDEX idx_audit_log_type ON audit_log (event_type, created_at);
```

### 9.2 Audited Operations

| Operation | Audit Level | Data Captured |
|-----------|-------------|---------------|
| Patient record access | Full | User, record, timestamp, IP |
| Clinical note creation/edit | Full | Old values, new values, user |
| Lab result viewing | Full | User, result ID, timestamp |
| Prescription creation | Full | Full prescription details |
| Login/logout | Full | User, IP, device, success/failure |
| Permission change | Full | Old roles, new roles, approver |
| Data export/download | Full | Export type, record count, user |
| Emergency access | Enhanced | Justification, duration, all actions |
| Failed access attempts | Full | User, resource, reason for denial |
| System configuration changes | Full | Setting, old value, new value |

---

## 10. Vulnerability Management

### 10.1 Scanning Pipeline

```mermaid
flowchart LR
    subgraph "Build-Time Scanning"
        A[Source Code] --> B[SAST<br/>Semgrep/gosec]
        C[Dependencies] --> D[SCA<br/>Snyk/Dependabot]
        E[Docker Images] --> F[Container Scan<br/>Trivy]
        G[IaC Templates] --> H[IaC Scan<br/>Checkov/tfsec]
    end

    subgraph "Runtime Scanning"
        I[Running Containers] --> J[Runtime Scan<br/>Falco]
        K[API Endpoints] --> L[DAST<br/>OWASP ZAP]
        M[Infrastructure] --> N[Cloud Posture<br/>AWS Security Hub]
    end

    subgraph "Response"
        B & D & F & H --> O[Block Build<br/>on Critical/High]
        J & L & N --> P[Alert SOC<br/>+ Auto-Remediate]
    end
```

### 10.2 Vulnerability SLA

| Severity | Detection to Patch SLA | Scope |
|----------|----------------------|-------|
| Critical (CVSS 9.0+) | 24 hours | Immediate hotfix |
| High (CVSS 7.0-8.9) | 7 days | Next sprint |
| Medium (CVSS 4.0-6.9) | 30 days | Planned release |
| Low (CVSS 0.1-3.9) | 90 days | Backlog |

---

## 11. Data Loss Prevention (DLP)

### 11.1 PHI Detection Patterns

| Data Type | Pattern | Action |
|-----------|---------|--------|
| National ID (Nigeria) | `[0-9]{11}` (NIN) | Encrypt at field level |
| Phone Number | `+234[0-9]{10}` | Mask in logs |
| Medical Record Number | `MRN-[A-Z0-9]+` | Never log |
| Credit Card | `[0-9]{13,19}` | Block from storage |
| Email Address | Standard email regex | Mask in non-auth contexts |
| Date of Birth | Date fields in patient context | Restrict access |

### 11.2 Log Sanitization

```go
// Sanitize sensitive fields before logging
func SanitizeForLogging(data map[string]interface{}) map[string]interface{} {
    sensitiveFields := []string{
        "password", "ssn", "national_id", "biometric_hash",
        "credit_card", "bank_account", "jwt_token",
    }
    for _, field := range sensitiveFields {
        if _, exists := data[field]; exists {
            data[field] = "[REDACTED]"
        }
    }
    // Mask phone numbers: +234801****567
    if phone, ok := data["phone"].(string); ok && len(phone) > 6 {
        data["phone"] = phone[:6] + "****" + phone[len(phone)-3:]
    }
    return data
}
```

---

## 12. Incident Response Plan

### 12.1 Response Workflow

```mermaid
flowchart TB
    A[Security Event Detected] --> B{Severity Assessment}
    B -->|Critical| C[P1: Activate Incident Commander<br/>Response in 15 min]
    B -->|High| D[P2: Security Team Lead<br/>Response in 1 hour]
    B -->|Medium| E[P3: Security Engineer<br/>Response in 4 hours]
    B -->|Low| F[P4: Ticket Created<br/>Response in 24 hours]

    C --> G[Contain Threat]
    G --> H[Preserve Evidence]
    H --> I[Eradicate Root Cause]
    I --> J[Recover Systems]
    J --> K[Post-Incident Review]
    K --> L[Update Playbooks]

    C --> M[Notify:<br/>CISO, CTO, Legal]
    M --> N{Breach Confirmed?}
    N -->|Yes| O[72-Hour Regulatory Notification<br/>HIPAA + NDPA + POPIA]
    N -->|Yes| P[Patient Notification<br/>if PHI exposed]
```

### 12.2 Contact Escalation

| Level | Role | Response Time | Contact |
|-------|------|--------------|---------|
| P1 | Incident Commander / CISO | 15 minutes | Phone + SMS |
| P2 | Security Team Lead | 1 hour | Phone + Email |
| P3 | On-call Security Engineer | 4 hours | PagerDuty |
| P4 | Security Team Queue | Next business day | Jira ticket |
