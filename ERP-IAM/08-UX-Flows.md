# ERP-IAM UX Flows

> **Document ID:** ERP-IAM-UX-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [06-Frontend-Documentation.md](./06-Frontend-Documentation.md), [07-Figma-Design-Prompts.md](./07-Figma-Design-Prompts.md)

---

## 1. Overview

This document defines the key user experience flows for ERP-IAM across all user personas. Each flow is described with a Mermaid diagram, happy path steps, error states, and accessibility considerations.

---

## 2. Authentication Flows

### 2.1 Standard Login Flow (Username + Password + MFA)

```mermaid
flowchart TD
    A["User navigates to login page"] --> B["Enter username/email"]
    B --> C{"Account exists?"}
    C -->|No| D["Error: Account not found<br/>(generic message for security)"]
    C -->|Yes| E{"Account locked?"}
    E -->|Yes| F["Error: Account locked<br/>Contact administrator"]
    E -->|No| G["Enter password"]
    G --> H{"Password correct?"}
    H -->|No| I["Increment failed attempts"]
    I --> J{"Threshold exceeded?"}
    J -->|Yes| K["Lock account<br/>Notify admin"]
    J -->|No| L["Error: Invalid credentials<br/>X attempts remaining"]
    H -->|Yes| M{"MFA required?"}
    M -->|No| N["Create session<br/>Redirect to app"]
    M -->|Yes| O{"MFA enrolled?"}
    O -->|No| P["MFA enrollment wizard"]
    O -->|Yes| Q["Select MFA method"]
    Q --> R["Verify MFA challenge"]
    R --> S{"MFA valid?"}
    S -->|No| T["Error: Invalid code<br/>Try again"]
    S -->|Yes| N
    P --> Q
```

### 2.2 Passwordless Login (FIDO2/WebAuthn)

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Keycloak
    participant Authenticator as FIDO2 Authenticator

    User->>Browser: Click "Sign in with Security Key"
    Browser->>Keycloak: POST /auth/passwordless/begin
    Keycloak-->>Browser: Challenge + allowed credentials
    Browser->>Authenticator: navigator.credentials.get(options)
    Authenticator->>User: Touch/biometric prompt
    User->>Authenticator: Touch / Face ID / Fingerprint
    Authenticator-->>Browser: Signed assertion
    Browser->>Keycloak: POST /auth/passwordless/complete
    Keycloak->>Keycloak: Verify signature against stored public key
    Keycloak-->>Browser: Session token + redirect
    Browser->>User: Redirect to application
```

### 2.3 Magic Link Authentication

```mermaid
flowchart TD
    A["User enters email"] --> B["Click 'Send Magic Link'"]
    B --> C["Server generates secure token<br/>TTL: 10 minutes"]
    C --> D["Send email with magic link"]
    D --> E["User opens email"]
    E --> F["Clicks magic link"]
    F --> G{"Token valid?"}
    G -->|Expired| H["Error: Link expired<br/>Request new link"]
    G -->|Already used| I["Error: Link already used<br/>Request new link"]
    G -->|Valid| J{"MFA required?"}
    J -->|No| K["Create session, redirect"]
    J -->|Yes| L["MFA challenge"]
    L --> K
```

### 2.4 Social Login Flow

```mermaid
sequenceDiagram
    participant User
    participant App
    participant Keycloak
    participant Social as Social IdP (Google/MS/Apple/FB)

    User->>App: Click "Sign in with Google"
    App->>Keycloak: Redirect to /auth/realms/{realm}/broker/google/login
    Keycloak->>Social: Redirect to Google OAuth consent
    Social->>User: Show consent screen
    User->>Social: Grant consent
    Social->>Keycloak: Redirect with authorization code
    Keycloak->>Social: Exchange code for tokens
    Social-->>Keycloak: Access token + ID token
    Keycloak->>Keycloak: Map claims to local user
    alt First-time social login
        Keycloak->>Keycloak: Create or link local account
    end
    Keycloak-->>App: Redirect with Keycloak session
    App->>User: Authenticated dashboard
```

---

## 3. Self-Service Flows

### 3.1 Self-Service Password Reset

```mermaid
flowchart TD
    A["User clicks 'Forgot Password'"] --> B["Enter email address"]
    B --> C["Server sends reset email<br/>(always shows success for security)"]
    C --> D["User opens email"]
    D --> E["Click reset link"]
    E --> F{"Token valid?"}
    F -->|No| G["Error: Link expired"]
    F -->|Yes| H["Enter new password"]
    H --> I{"Meets password policy?"}
    I -->|No| J["Error: Policy violations listed"]
    I -->|Yes| K{"Different from last N passwords?"}
    K -->|No| L["Error: Cannot reuse recent passwords"]
    K -->|Yes| M["Password updated"]
    M --> N["Invalidate all existing sessions"]
    N --> O["Redirect to login"]
```

### 3.2 MFA Enrollment

```mermaid
flowchart TD
    A["User directed to MFA enrollment"] --> B["Select MFA method"]
    B --> C{"Method selected?"}

    C -->|TOTP| D1["Display QR code + secret"]
    D1 --> E1["User scans with authenticator app"]
    E1 --> F1["Enter verification code"]
    F1 --> G1{"Code valid?"}
    G1 -->|Yes| H["Generate recovery codes"]
    G1 -->|No| I1["Error: Invalid code, try again"]

    C -->|SMS| D2["Enter phone number"]
    D2 --> E2["Send verification SMS"]
    E2 --> F2["Enter SMS code"]
    F2 --> G2{"Code valid?"}
    G2 -->|Yes| H
    G2 -->|No| I2["Error: Invalid code"]

    C -->|Push| D3["Install authenticator app prompt"]
    D3 --> E3["Scan pairing QR"]
    E3 --> F3["Send test push"]
    F3 --> G3{"Push approved?"}
    G3 -->|Yes| H
    G3 -->|No| I3["Error: Push not received/denied"]

    C -->|FIDO2| D4["Insert security key prompt"]
    D4 --> E4["Touch key for registration"]
    E4 --> F4["Key registered"]
    F4 --> H

    H --> J["Display recovery codes<br/>User must save/print"]
    J --> K["Confirm codes saved"]
    K --> L["MFA enrollment complete"]
```

---

## 4. Administrative Flows

### 4.1 SSO Connection Setup (Admin)

```mermaid
flowchart TD
    A["Admin opens SSO Config"] --> B["Click 'Add Connection'"]
    B --> C["Step 1: Select Protocol"]
    C --> D{"Protocol?"}
    D -->|OIDC| E1["Enter Discovery URL"]
    E1 --> F1["Auto-populate from .well-known"]
    F1 --> G1["Enter Client ID + Secret"]
    D -->|SAML| E2["Upload/enter metadata URL"]
    E2 --> F2["Parse metadata"]
    F2 --> G2["Configure assertion settings"]
    D -->|LDAP| E3["Enter server details"]
    E3 --> F3["Test LDAP bind"]

    G1 --> H["Step 3: Attribute Mapping"]
    G2 --> H
    F3 --> H

    H --> I["Step 4: Test Connection"]
    I --> J{"Test passed?"}
    J -->|No| K["Show error details<br/>Allow edit and retry"]
    J -->|Yes| L["Step 5: Review + Activate"]
    L --> M["Toggle Active"]
    M --> N["SSO Connection live"]
```

### 4.2 Device Enrollment (Admin-Initiated)

```mermaid
sequenceDiagram
    participant Admin
    participant Console
    participant MDM as MDM Service
    participant DEP as Apple DEP/ABM
    participant Device

    Admin->>Console: Navigate to Device Inventory
    Admin->>Console: Click "Enroll New Device"
    Admin->>Console: Select platform (macOS/iOS/Windows/Android)

    alt Apple DEP Enrollment
        Console->>MDM: Create enrollment profile
        MDM->>DEP: Assign profile to serial numbers
        DEP-->>MDM: Assignment confirmed
        Note over Device: Device activates / resets
        Device->>DEP: Check for MDM enrollment
        DEP->>Device: Redirect to MDM server
        Device->>MDM: Enroll with enrollment profile
        MDM-->>Device: Configuration profiles pushed
    else Manual Enrollment
        Console->>MDM: Generate enrollment URL/QR
        MDM-->>Console: Display QR code / URL
        Admin->>Device: Share QR code with user
        Device->>MDM: Scan QR / open URL
        MDM-->>Device: Install MDM profile
    end

    MDM->>Console: Device enrolled event
    Console->>Admin: Device appears in inventory
```

---

## 5. Provisioning Flows

### 5.1 Joiner-Mover-Leaver Lifecycle

```mermaid
flowchart LR
    subgraph "JOINER"
        J1["HR creates new hire"] --> J2["Provisioning event fired"]
        J2 --> J3["Create identity<br/>(username, email)"]
        J3 --> J4["Assign to OU<br/>based on department"]
        J4 --> J5["Add to groups<br/>based on role"]
        J5 --> J6["Provision apps<br/>(Slack, GitHub, etc.)"]
        J6 --> J7["Send welcome email<br/>with activation link"]
    end

    subgraph "MOVER"
        M1["HR updates role/<br/>department"] --> M2["Provisioning event fired"]
        M2 --> M3["Update OU placement"]
        M3 --> M4["Update group<br/>memberships"]
        M4 --> M5["Provision/deprovision<br/>apps per new role"]
        M5 --> M6["Notify user and<br/>new manager"]
    end

    subgraph "LEAVER"
        L1["HR terminates<br/>employee"] --> L2["Provisioning event fired"]
        L2 --> L3["Disable identity<br/>(grace period)"]
        L3 --> L4["Terminate all<br/>active sessions"]
        L4 --> L5["Revoke app access"]
        L5 --> L6["Transfer data to<br/>manager (if configured)"]
        L6 --> L7["Delete identity<br/>after retention period"]
    end
```

---

## 6. Device Trust Evaluation Flow

```mermaid
flowchart TD
    A["User attempts resource access"] --> B["Gateway intercepts request"]
    B --> C["Validate JWT + session"]
    C --> D{"Conditional access policy<br/>requires device trust?"}
    D -->|No| E["Forward to resource"]
    D -->|Yes| F["Extract device ID from token"]
    F --> G["Query device posture"]
    G --> H{"Device registered?"}
    H -->|No| I["Block: Unregistered device<br/>Prompt to enroll"]
    H -->|Yes| J{"Last posture check<br/>within threshold?"}
    J -->|No| K["Trigger real-time<br/>posture check"]
    J -->|Yes| L{"Device compliant?"}
    K --> L
    L -->|Yes| E
    L -->|No| M{"Grace period active?"}
    M -->|Yes| N["Allow with warning<br/>User must remediate"]
    M -->|No| O["Block: Non-compliant<br/>Show remediation steps"]
```

---

## 7. Credential Rotation Flow

```mermaid
sequenceDiagram
    participant Scheduler
    participant Vault as Credential Vault
    participant HSM
    participant Target as Target System
    participant Audit

    Scheduler->>Vault: Check rotation schedules
    Vault->>Vault: Identify credentials due for rotation
    Vault->>Target: Generate new credential (API key, password, etc.)
    Target-->>Vault: New credential value
    Vault->>HSM: Encrypt new credential with DEK
    HSM-->>Vault: Encrypted blob
    Vault->>Vault: Store new version, mark old as previous
    Vault->>Vault: Update consumers with new credential reference
    Vault->>Audit: Log rotation event
    Note over Vault: Old credential remains valid for overlap period
    Scheduler->>Vault: After overlap period, revoke old credential
    Vault->>Target: Revoke old credential
    Vault->>Audit: Log revocation event
```

---

## 8. Error States and Recovery

### 8.1 Common Error Handling Patterns

| Scenario | User-Facing Message | Recovery Action |
|---|---|---|
| Invalid credentials | "The email or password you entered is incorrect" | Show attempt counter, link to password reset |
| Account locked | "Your account has been temporarily locked" | Display lockout duration, admin contact info |
| MFA code expired | "This code has expired. Request a new code." | Auto-resend option with cooldown timer |
| Device non-compliant | "Your device does not meet security requirements" | Checklist of failed posture checks with remediation links |
| Session expired | "Your session has expired. Please sign in again." | Redirect to login, preserve intended destination |
| Rate limited | "Too many requests. Please wait before trying again." | Display retry-after countdown |
| Network error | "Unable to connect. Check your internet connection." | Retry button with exponential backoff |
