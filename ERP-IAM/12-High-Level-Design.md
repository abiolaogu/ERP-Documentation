# ERP-IAM High-Level Design (HLD)

> **Document ID:** ERP-IAM-HLD-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [04-Software-Architecture.md](./04-Software-Architecture.md), [13-Low-Level-Design.md](./13-Low-Level-Design.md)

---

## 1. Introduction

This High-Level Design document describes the overall system structure of ERP-IAM at the module and service boundary level. It focuses on how the eight microservices interact with each other, with external systems, and with the broader ERP ecosystem, without diving into implementation details of individual components.

---

## 2. System Decomposition

### 2.1 Module Boundary

```mermaid
flowchart TB
    subgraph "ERP-IAM Module Boundary"
        direction TB
        GW["API Gateway<br/>Authentication + Rate Limiting + Routing"]

        subgraph "Identity Domain"
            direction LR
            IDS["Identity Service<br/>(Keycloak Orchestrator)"]
            DRS["Directory Service<br/>(Authentik + Samba AD DC)"]
            PRS["Provisioning Service<br/>(SCIM 2.0 Engine)"]
        end

        subgraph "Device Domain"
            direction LR
            DTS["Device Trust Service<br/>(FleetDM + osquery)"]
            MDS["MDM Service<br/>(NanoMDM)"]
        end

        subgraph "Security Infrastructure"
            direction LR
            CVS["Credential Vault<br/>(AES-256 + HSM)"]
            SSS["Session Service<br/>(Redis-backed)"]
            AUS["Audit Service<br/>(Immutable Chain)"]
        end
    end

    subgraph "External Systems"
        ERP_PLAT["ERP-Platform"]
        ERP_MODS["Other ERP Modules"]
        SOCIAL["Social IdPs"]
        EXT_DIR["External Directories"]
        SIEM_EXT["SIEM Systems"]
        ENDPOINTS["Managed Endpoints"]
    end

    ERP_PLAT <-->|"Entitlements<br/>Module Registry"| GW
    ERP_MODS -->|"JWT Validation"| GW
    SOCIAL <-->|"OIDC/OAuth2"| IDS
    EXT_DIR <-->|"LDAP/SCIM/Graph"| DRS
    SIEM_EXT <--|"Audit Events"| AUS
    ENDPOINTS <-->|"osquery/MDM"| DTS & MDS
```

### 2.2 Service Responsibilities

| Service | Single Responsibility | Owns |
|---|---|---|
| identity-service | Authentication and token lifecycle | Keycloak realm management, protocol handling (OIDC/SAML/LDAP), MFA, social login, passwordless, risk engine |
| directory-service | User/group/OU data management | Authentik user store, Samba AD DC, LDAP interface, directory sync, group policies |
| provisioning-service | Identity lifecycle automation | SCIM 2.0 server+client, joiner-mover-leaver rules, attribute mapping, sync scheduling |
| device-trust-service | Endpoint compliance assessment | FleetDM orchestration, osquery policy, posture evaluation, trust scoring, conditional access |
| mdm-service | Device management commands | NanoMDM, enrollment profiles, app deployment, remote wipe/lock, configuration profiles |
| credential-vault-service | Secrets management | Encryption/decryption, key hierarchy, rotation schedules, HSM integration |
| session-service | Session lifecycle | Session creation/validation/termination, concurrent limits, timeout policies, geolocation |
| audit-service | Immutable event logging | Event ingestion, chain verification, SIEM forwarding, compliance reports |

---

## 3. Inter-Service Communication

### 3.1 Communication Matrix

```mermaid
flowchart LR
    IDS["Identity"] -->|"Sync: Create session"| SSS["Session"]
    IDS -->|"Async: Auth events"| AUS["Audit"]
    IDS -->|"Sync: Validate device"| DTS["Device Trust"]
    DRS["Directory"] -->|"Async: User events"| PRS["Provisioning"]
    DRS -->|"Async: Directory events"| AUS
    PRS -->|"Sync: Create user"| DRS
    PRS -->|"Async: Provisioning events"| AUS
    DTS -->|"Sync: Query device"| MDS["MDM"]
    DTS -->|"Async: Compliance events"| AUS
    MDS -->|"Async: MDM events"| AUS
    CVS["Vault"] -->|"Async: Rotation events"| AUS
    SSS -->|"Async: Session events"| AUS
```

### 3.2 Communication Patterns

| From | To | Pattern | Protocol | Use Case |
|---|---|---|---|---|
| identity-service | session-service | Synchronous | gRPC | Create/validate session on auth |
| identity-service | device-trust-service | Synchronous | gRPC | Check device posture during auth |
| identity-service | audit-service | Asynchronous | NATS | Emit auth events |
| directory-service | provisioning-service | Asynchronous | NATS | Trigger provisioning on user changes |
| provisioning-service | directory-service | Synchronous | HTTP | Create/update users in directory |
| device-trust-service | mdm-service | Synchronous | gRPC | Query MDM enrollment status |
| All services | audit-service | Asynchronous | NATS | Emit all state change events |
| audit-service | SIEM | Asynchronous | HTTP/Kafka | Forward events to external SIEM |

---

## 4. Authentication Flow (End-to-End)

```mermaid
sequenceDiagram
    participant User
    participant App
    participant Gateway as API Gateway
    participant Identity as Identity Service
    participant Directory as Directory Service
    participant DeviceTrust as Device Trust
    participant Session as Session Service
    participant Audit as Audit Service

    User->>App: Access protected resource
    App->>Gateway: Redirect to /auth/authorize
    Gateway->>Identity: Forward auth request
    Identity->>Directory: Validate credentials (LDAP bind)
    Directory-->>Identity: Credentials valid
    Identity->>Identity: Evaluate risk score
    Identity->>Identity: Check MFA requirement
    Identity->>User: MFA challenge (if required)
    User->>Identity: MFA response
    Identity->>Identity: Verify MFA
    Identity->>DeviceTrust: Check device posture (if conditional access)
    DeviceTrust-->>Identity: Device compliant
    Identity->>Session: Create session
    Session-->>Identity: Session token
    Identity->>Audit: Emit auth.success event
    Identity-->>App: Authorization code
    App->>Identity: Exchange code for tokens
    Identity-->>App: Access + Refresh + ID tokens
    App-->>User: Grant access
```

---

## 5. Provisioning Flow (End-to-End)

```mermaid
sequenceDiagram
    participant HR as HR System
    participant SCIM as Provisioning Service
    participant Directory as Directory Service
    participant Identity as Identity Service
    participant Apps as Downstream Apps
    participant Audit as Audit Service

    HR->>SCIM: POST /scim/v2/Users (new hire)
    SCIM->>SCIM: Validate SCIM payload
    SCIM->>SCIM: Apply attribute mapping rules
    SCIM->>Directory: Create user in directory
    Directory-->>SCIM: User created with ID
    SCIM->>Identity: Create authentication identity (Keycloak)
    Identity-->>SCIM: Identity created
    SCIM->>Directory: Add user to groups (based on role rules)
    Directory-->>SCIM: Groups updated
    SCIM->>Apps: SCIM POST to Slack, GitHub, etc.
    Apps-->>SCIM: Provisioning confirmed
    SCIM->>Audit: Emit provisioning.lifecycle.joiner event
    SCIM->>SCIM: Send welcome email notification
```

---

## 6. Device Trust Flow (End-to-End)

```mermaid
sequenceDiagram
    participant Device
    participant osquery
    participant FleetDM
    participant DeviceTrust as Device Trust Service
    participant Policy as Policy Engine
    participant Audit as Audit Service

    Device->>osquery: Run scheduled queries
    osquery->>FleetDM: Report query results
    FleetDM->>DeviceTrust: Forward posture data
    DeviceTrust->>Policy: Evaluate against compliance policies
    Policy-->>DeviceTrust: Compliance result + trust score
    alt Device Compliant
        DeviceTrust->>DeviceTrust: Update device status: compliant
        DeviceTrust->>Audit: Emit device-trust.compliant event
    else Device Non-Compliant
        DeviceTrust->>DeviceTrust: Update device status: non-compliant
        DeviceTrust->>Audit: Emit device-trust.non-compliant event
        DeviceTrust->>DeviceTrust: Trigger conditional access re-evaluation
    end
```

---

## 7. Data Flow Overview

```mermaid
flowchart TB
    subgraph "Ingress"
        A["User Requests"]
        B["SCIM Inbound"]
        C["Device Telemetry"]
        D["MDM Check-ins"]
    end

    subgraph "Processing"
        E["API Gateway"]
        F["Service Mesh (mTLS)"]
        G["Business Logic"]
    end

    subgraph "State"
        H["YugabyteDB<br/>(Persistent)"]
        I["Redis<br/>(Ephemeral)"]
        J["NATS JetStream<br/>(Events)"]
    end

    subgraph "Egress"
        K["JWT Tokens"]
        L["SCIM Outbound"]
        M["MDM Commands"]
        N["SIEM Events"]
        O["Notifications"]
    end

    A & B & C & D --> E --> F --> G
    G --> H & I & J
    G --> K & L & M
    J --> N & O
```

---

## 8. Failure Modes and Recovery

| Failure Scenario | Impact | Detection | Recovery |
|---|---|---|---|
| Keycloak cluster failure | No new authentications | Health check + Kubernetes restart | Stateless pods restart, sessions preserved in Redis |
| YugabyteDB node failure | Degraded write performance | Raft consensus detects | Automatic re-replication to surviving nodes |
| Redis node failure | Session lookups degrade | Sentinel/Cluster detection | Failover to replica, sessions reconstructed from DB |
| NATS cluster failure | Event delivery paused | Health check | JetStream replay from disk on recovery |
| FleetDM failure | Device posture stale | Health check + stale data detection | Grace period for cached posture, alert admin |
| SIEM connector failure | Audit forwarding paused | Delivery failure counter | NATS retains events, replay on reconnection |
| HSM failure | Cannot encrypt new credentials | HSM health check | Failover to backup HSM, cached KEKs |

---

## 9. Capacity Planning

### 9.1 Sizing Guidelines

| Component | Small (1K users) | Medium (10K users) | Large (100K users) | Enterprise (1M users) |
|---|---|---|---|---|
| identity-service pods | 2 | 3 | 6 | 15 |
| directory-service pods | 2 | 2 | 4 | 10 |
| provisioning-service pods | 1 | 2 | 3 | 6 |
| device-trust-service pods | 1 | 2 | 4 | 8 |
| session-service pods | 2 | 3 | 6 | 15 |
| audit-service pods | 1 | 2 | 4 | 8 |
| YugabyteDB nodes | 3 | 3 | 5 | 9 |
| Redis nodes | 3 | 6 | 6 | 12 |
| NATS nodes | 3 | 3 | 5 | 7 |
| Keycloak nodes | 2 | 3 | 5 | 10 |
