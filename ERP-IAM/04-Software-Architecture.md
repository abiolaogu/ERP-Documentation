# ERP-IAM Software Architecture Document

> **Document ID:** ERP-IAM-SA-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [03-Technical-Writeup.md](./03-Technical-Writeup.md), [12-High-Level-Design.md](./12-High-Level-Design.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md)

---

## 1. Introduction

### 1.1 Purpose

This document defines the software architecture for ERP-IAM, the Identity and Access Management module of the ERP suite. It describes the system's structural decomposition, communication patterns, data flows, and design decisions using the C4 model (Context, Container, Component, Code).

### 1.2 Scope

ERP-IAM encompasses identity provider services, directory management, user provisioning, device trust assessment, mobile device management, credential vaulting, session governance, and audit logging. The architecture supports both standalone deployment and suite-integrated mode.

### 1.3 Architectural Drivers

| Driver | Description |
|---|---|
| **Multi-Tenancy** | Complete tenant isolation at the data, compute, and network layers |
| **Zero Trust** | Every request is authenticated, authorized, and device-trust verified |
| **Protocol Breadth** | Support for OIDC, OAuth 2.0, SAML 2.0, LDAP, SCIM, WebAuthn simultaneously |
| **Horizontal Scale** | Each service scales independently based on its specific load characteristics |
| **Compliance** | SOC 2 Type II and ISO 27001 controls baked into the architecture |

---

## 2. C4 Model

### 2.1 Level 1: System Context

```mermaid
C4Context
    title ERP-IAM System Context Diagram

    Person(admin, "IT Administrator", "Manages identity infrastructure")
    Person(user, "End User", "Authenticates and accesses resources")
    Person(auditor, "Compliance Auditor", "Reviews security posture")

    System(iam, "ERP-IAM", "Unified Identity and Access Management")

    System_Ext(erp_platform, "ERP-Platform", "Control plane, entitlements, subscriptions")
    System_Ext(erp_modules, "ERP Modules", "CRM, Finance, HCM, SCM, etc.")
    System_Ext(social_idps, "Social Identity Providers", "Google, Microsoft, Apple, Facebook")
    System_Ext(external_dirs, "External Directories", "Azure AD, Google Workspace, On-prem AD")
    System_Ext(siem, "SIEM Systems", "Splunk, ELK, Datadog")
    System_Ext(endpoints, "Managed Endpoints", "Windows, macOS, Linux, iOS, Android")

    Rel(admin, iam, "Configures policies, manages users")
    Rel(user, iam, "Authenticates, enrolls devices")
    Rel(auditor, iam, "Reviews audit logs, generates reports")
    Rel(iam, erp_platform, "Registers module, receives entitlements")
    Rel(erp_modules, iam, "Validates JWT tokens, checks access")
    Rel(iam, social_idps, "Federates authentication")
    Rel(iam, external_dirs, "Syncs directories")
    Rel(iam, siem, "Streams audit events")
    Rel(endpoints, iam, "Reports posture, receives MDM commands")
```

### 2.2 Level 2: Container Diagram

```mermaid
flowchart TB
    subgraph "ERP-IAM Module"
        GW["API Gateway<br/>Go / net/http<br/>Port: 8080"]

        subgraph "Identity Domain"
            IDS["identity-service<br/>Go + Keycloak<br/>OIDC/OAuth2/SAML2/MFA"]
            DRS["directory-service<br/>Go + Authentik + Samba AD DC<br/>LDAP/AD/Groups/OUs"]
            PRS["provisioning-service<br/>Go<br/>SCIM 2.0 Server+Client"]
        end

        subgraph "Device Domain"
            DTS["device-trust-service<br/>Go + FleetDM + osquery<br/>Posture/Compliance/Conditional Access"]
            MDS["mdm-service<br/>Go + NanoMDM<br/>Apple/Android/Windows MDM"]
        end

        subgraph "Security Domain"
            CVS["credential-vault-service<br/>Go + HSM<br/>Secrets/Encryption/Rotation"]
            SSS["session-service<br/>Go + Redis<br/>Sessions/Timeouts/Geo"]
            AUS["audit-service<br/>Go<br/>Logging/SIEM/Compliance"]
        end
    end

    subgraph "Data Stores"
        YDB[("YugabyteDB<br/>Primary SQL Store")]
        RDS[("Redis Cluster<br/>Session Cache")]
        NATS["NATS / Redpanda<br/>Event Bus"]
    end

    GW --> IDS
    GW --> DRS
    GW --> PRS
    GW --> DTS
    GW --> MDS
    GW --> CVS
    GW --> SSS
    GW --> AUS

    IDS --> YDB
    DRS --> YDB
    PRS --> YDB
    DTS --> YDB
    MDS --> YDB
    CVS --> YDB
    SSS --> RDS
    AUS --> YDB

    IDS --> NATS
    DRS --> NATS
    PRS --> NATS
    DTS --> NATS
    MDS --> NATS
    CVS --> NATS
    SSS --> NATS
    AUS --> NATS
```

### 2.3 Level 3: Component Diagram (Identity Service)

```mermaid
flowchart TB
    subgraph "identity-service"
        API["HTTP API Handler<br/>net/http ServeMux"]
        AUTH["Authentication Engine<br/>Protocol Router"]
        OIDC["OIDC Provider<br/>Authorization Code + PKCE"]
        SAML["SAML Provider<br/>SP + IdP Roles"]
        MFA["MFA Engine<br/>TOTP/SMS/Push/FIDO2"]
        SOCIAL["Social Login Manager<br/>Google/MS/Apple/FB"]
        RISK["Risk Engine<br/>Adaptive Auth Scoring"]
        BF["Brute Force Protector<br/>Rate Limiter + Lockout"]
        KC["Keycloak Client<br/>Admin REST API"]
    end

    API --> AUTH
    AUTH --> OIDC
    AUTH --> SAML
    AUTH --> MFA
    AUTH --> SOCIAL
    AUTH --> RISK
    AUTH --> BF
    OIDC --> KC
    SAML --> KC
    MFA --> KC
    SOCIAL --> KC
```

### 2.4 Level 3: Component Diagram (Directory Service)

```mermaid
flowchart TB
    subgraph "directory-service"
        API["HTTP API Handler"]
        UM["User Manager<br/>CRUD + Attributes"]
        GM["Group Manager<br/>Membership + Nesting"]
        OUM["OU Manager<br/>Hierarchy + Placement"]
        LDAP["LDAP Interface<br/>Search/Bind/Modify"]
        AD["AD Controller<br/>Samba AD DC Client"]
        GPO["GPO Manager<br/>Policy Distribution"]
        SYNC["Directory Sync Engine<br/>Azure AD/Google/On-Prem"]
        AK["Authentik Client<br/>User Store API"]
    end

    API --> UM
    API --> GM
    API --> OUM
    API --> LDAP
    UM --> AK
    UM --> AD
    GM --> AK
    GM --> AD
    OUM --> AD
    LDAP --> AD
    GPO --> AD
    SYNC --> AD
    SYNC --> AK
```

---

## 3. Architectural Patterns

### 3.1 Microservices with Domain Boundaries

The eight services are organized into three bounded contexts:

1. **Identity Domain**: `identity-service`, `directory-service`, `provisioning-service` -- owns user identity, authentication, and lifecycle
2. **Device Domain**: `device-trust-service`, `mdm-service` -- owns endpoint management and compliance
3. **Security Domain**: `credential-vault-service`, `session-service`, `audit-service` -- owns runtime security infrastructure

### 3.2 Event-Driven Architecture

```mermaid
flowchart LR
    subgraph "Producers"
        IDS["identity-service"]
        DRS["directory-service"]
        PRS["provisioning-service"]
        DTS["device-trust-service"]
    end

    subgraph "Event Bus"
        NATS["NATS / Redpanda<br/>Topic: erp.iam.*"]
    end

    subgraph "Consumers"
        AUS["audit-service<br/>(all events)"]
        PRS2["provisioning-service<br/>(identity events)"]
        SSS["session-service<br/>(identity events)"]
        SIEM["External SIEM"]
        PLAT["ERP-Platform"]
    end

    IDS --> NATS
    DRS --> NATS
    PRS --> NATS
    DTS --> NATS
    NATS --> AUS
    NATS --> PRS2
    NATS --> SSS
    NATS --> SIEM
    NATS --> PLAT
```

All inter-service communication for non-synchronous operations uses the event bus. This ensures:
- **Loose coupling**: Services only know about event topics, not each other
- **Audit completeness**: The audit service subscribes to all events for immutable logging
- **Replay capability**: Event replay from NATS JetStream / Redpanda for recovery scenarios

### 3.3 Zero Trust Request Flow

Every API request passes through the following zero-trust verification chain:

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant Identity
    participant DeviceTrust
    participant Session
    participant Service

    Client->>Gateway: Request + JWT + X-Tenant-ID
    Gateway->>Identity: Validate JWT signature + expiry
    Identity-->>Gateway: Token valid + claims
    Gateway->>Session: Check session active + within limits
    Session-->>Gateway: Session valid
    Gateway->>DeviceTrust: Check device posture (if conditional access enabled)
    DeviceTrust-->>Gateway: Device compliant
    Gateway->>Service: Forward request with verified context
    Service-->>Client: Response
```

### 3.4 Multi-Tenancy Model

```mermaid
flowchart TB
    subgraph "Tenant A"
        RA["Keycloak Realm A"]
        DA["Directory Partition A"]
        SA["Session Namespace A"]
    end

    subgraph "Tenant B"
        RB["Keycloak Realm B"]
        DB["Directory Partition B"]
        SB["Session Namespace B"]
    end

    subgraph "Shared Infrastructure"
        KC["Keycloak Cluster"]
        YDB["YugabyteDB"]
        RDS["Redis Cluster"]
    end

    RA --> KC
    RB --> KC
    DA --> YDB
    DB --> YDB
    SA --> RDS
    SB --> RDS
```

Tenant isolation is enforced at multiple levels:
- **Keycloak**: Realm-per-tenant with isolated user stores, client registrations, and identity providers
- **Database**: Row-level security with `tenant_id` column on all tables; YugabyteDB tablespace-per-tenant option for large tenants
- **Redis**: Namespace prefixing (`tenant:{id}:session:{session_id}`) for session isolation
- **Events**: Tenant ID included in all CloudEvents metadata for event routing and filtering
- **API**: `X-Tenant-ID` header validation on every request with JWT claim cross-verification

---

## 4. Communication Patterns

### 4.1 Synchronous (Request-Response)

| Pattern | Use Case | Protocol |
|---|---|---|
| REST API | Client-to-service, service-to-service queries | HTTP/2 + JSON |
| LDAP | Directory queries, LDAP bind authentication | LDAP v3 + StartTLS |
| gRPC | High-throughput internal service communication | Protocol Buffers over HTTP/2 |

### 4.2 Asynchronous (Event-Driven)

| Pattern | Use Case | Protocol |
|---|---|---|
| Pub/Sub | Authentication events, provisioning triggers, audit logging | NATS JetStream |
| Queue | MDM command delivery, credential rotation jobs | NATS Queue Groups |
| Stream | SIEM integration, compliance report generation | Redpanda (Kafka protocol) |

### 4.3 Federation

| Pattern | Use Case | Protocol |
|---|---|---|
| OIDC Federation | Cross-realm SSO, social login | OpenID Connect |
| SAML Federation | Enterprise SSO with legacy IdPs | SAML 2.0 |
| SCIM Provisioning | User lifecycle sync with external systems | SCIM 2.0 over HTTPS |
| AD Replication | Multi-site directory sync | DRS (Directory Replication Service) |

---

## 5. Security Architecture

### 5.1 Defense in Depth

```mermaid
flowchart TB
    subgraph "Layer 1: Network"
        WAF["Web Application Firewall"]
        TLS["TLS 1.3 Termination"]
        NP["Network Policies (K8s)"]
    end

    subgraph "Layer 2: Authentication"
        JWT["JWT Validation"]
        MFA_L["MFA Enforcement"]
        RISK_L["Risk Assessment"]
    end

    subgraph "Layer 3: Authorization"
        RBAC["Role-Based Access Control"]
        ABAC["Attribute-Based Access Control"]
        CA["Conditional Access Policies"]
    end

    subgraph "Layer 4: Data"
        ENC["AES-256-GCM Encryption"]
        RLS["Row-Level Security"]
        MASK["Data Masking"]
    end

    subgraph "Layer 5: Monitoring"
        AUDIT["Immutable Audit Logs"]
        SIEM_L["SIEM Integration"]
        ALERT["Real-Time Alerting"]
    end

    WAF --> TLS --> NP
    NP --> JWT --> MFA_L --> RISK_L
    RISK_L --> RBAC --> ABAC --> CA
    CA --> ENC --> RLS --> MASK
    MASK --> AUDIT --> SIEM_L --> ALERT
```

### 5.2 Encryption Strategy

| Layer | Algorithm | Key Management |
|---|---|---|
| Transport | TLS 1.3 (ECDHE + AES-256-GCM) | Let's Encrypt / cert-manager |
| Credential at rest | AES-256-GCM | HSM-backed master key, envelope encryption |
| Database at rest | AES-256 (YugabyteDB TDE) | YugabyteDB key management |
| Session tokens | HMAC-SHA256 | Rotating signing keys (72-hour lifecycle) |
| Audit log chain | SHA-256 | Hash chain with previous entry linkage |

---

## 6. Deployment Architecture

### 6.1 Kubernetes Deployment Model

```mermaid
flowchart TB
    subgraph "Kubernetes Cluster"
        subgraph "Namespace: erp-iam"
            subgraph "Deployments"
                D1["identity-service<br/>Replicas: 3-10"]
                D2["directory-service<br/>Replicas: 2-8"]
                D3["provisioning-service<br/>Replicas: 2-6"]
                D4["device-trust-service<br/>Replicas: 2-6"]
                D5["mdm-service<br/>Replicas: 2-4"]
                D6["credential-vault-service<br/>Replicas: 2-4"]
                D7["session-service<br/>Replicas: 3-10"]
                D8["audit-service<br/>Replicas: 2-6"]
            end

            subgraph "StatefulSets"
                KC["Keycloak Cluster<br/>Replicas: 3"]
                AK["Authentik<br/>Replicas: 2"]
                SA["Samba AD DC<br/>Replicas: 2"]
            end

            subgraph "Infrastructure"
                FL["FleetDM Server"]
                NM["NanoMDM Server"]
            end
        end

        subgraph "Namespace: erp-data"
            YDB["YugabyteDB<br/>Replicas: 3"]
            RDS["Redis Cluster<br/>Replicas: 6"]
        end

        subgraph "Namespace: erp-messaging"
            NATS_D["NATS Cluster<br/>Replicas: 3"]
        end
    end
```

### 6.2 High Availability

- **Active-Active**: All services run in active-active configuration across availability zones
- **Pod Anti-Affinity**: Service replicas spread across different nodes/zones
- **Health Checks**: Liveness (`/healthz`) and readiness probes on all pods
- **Circuit Breakers**: Resilience patterns (retry, circuit breaker, bulkhead) via service mesh
- **Auto-Scaling**: HPA with custom metrics (auth rate, LDAP query rate, SCIM sync backlog)

---

## 7. Technology Decision Records

### 7.1 ADR-001: Language Choice

**Decision**: Polyglot -- Go for API services, Python/Flask for web admin, Dart/Flutter for mobile MFA app.

**Rationale**: Go provides the performance and concurrency characteristics needed for high-throughput identity operations. Python/Flask is retained from the IDaaS2 webapp for admin UI. Flutter enables a single MFA authenticator codebase targeting both iOS and Android.

### 7.2 ADR-002: Database Selection

**Decision**: YugabyteDB primary, Redis cache.

**Rationale**: YugabyteDB provides PostgreSQL wire compatibility (critical for Keycloak/Authentik) with distributed SQL for global replication and horizontal scaling. Redis provides sub-millisecond session lookups with built-in TTL for automatic session expiry.
