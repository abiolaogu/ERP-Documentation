# ERP-IAM Release Notes

> **Document ID:** ERP-IAM-RN-001
> **Version:** 1.0.0
> **Release Date:** 2026-02-23
> **Status:** Released
> **Related Documents:** [01-PRD.md](./01-PRD.md), [16-Changelog.md](./16-Changelog.md), [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md)

---

## Release v1.0.0 -- "Zero Trust, Zero Friction"

### Release Summary

ERP-IAM v1.0.0 is the inaugural general-availability release of the unified Identity and Access Management module for the ERP suite. This release consolidates the previously separate ERP-IDaaS2 and ERP-Directory modules into a single, cohesive identity platform delivering eight microservices: identity, directory, provisioning, device trust, MDM, credential vault, session management, and audit.

---

## New Features

### Identity Service (`identity-service`)
- **Keycloak Multi-Tenant IdP**: Realm-per-tenant isolation with automated realm provisioning on tenant creation
- **Protocol Coverage**: Full OIDC/OAuth 2.0 authorization server with PKCE, device flow, and client credentials grant types
- **SAML 2.0**: Service provider and identity provider roles with automated metadata exchange
- **Social Login**: Google, Microsoft, Apple, and Facebook identity providers with configurable scope mappings
- **Passwordless Authentication**: FIDO2/WebAuthn credential registration and authentication, magic link email flow, OTP via SMS/email
- **Multi-Factor Authentication**: TOTP (Google Authenticator compatible), SMS OTP, push notification, hardware security key (YubiKey/SoloKey)
- **Adaptive Risk-Based Auth**: IP reputation scoring, device fingerprint analysis, impossible travel detection, behavioral anomaly flagging
- **Brute Force Protection**: Progressive delay escalation (1s, 2s, 4s, 8s, 16s), configurable lockout thresholds, IP-based and account-based rate limiting
- **Account Lockout**: Configurable lockout duration, automatic unlock scheduling, admin manual unlock via API

### Directory Service (`directory-service`)
- **Authentik User Directory**: Cloud-native user store with organizational unit (OU) hierarchy, custom attributes, nested groups
- **Samba AD DC**: Full Active Directory Domain Controller with DNS, Kerberos KDC, LDAP, and Group Policy support
- **Domain Join**: Native Windows domain join, macOS directory binding via Open Directory, Linux SSSD/Winbind configuration
- **Group Policy Objects**: Windows GPO distribution, macOS configuration profiles, Linux policy enforcement via SSSD
- **Computer Objects**: Automatic computer account creation on domain join, OU-based placement rules
- **LDAP Interface**: Standard LDAP v3 query interface with search filters, pagination, referrals, and StartTLS
- **Directory Sync**: Bidirectional sync with Azure AD, Google Workspace, and on-premises Active Directory

### Provisioning Service (`provisioning-service`)
- **SCIM 2.0 Server**: RFC 7643/7644 compliant SCIM server accepting inbound User and Group provisioning
- **SCIM 2.0 Client**: Outbound SCIM provisioning to downstream SaaS applications (Salesforce, Google Workspace, Slack, etc.)
- **Lifecycle Automation**: Joiner (auto-provision on HR hire event), Mover (auto-update on role/department change), Leaver (auto-deprovision on termination)
- **Attribute Mapping**: Declarative attribute mapping engine with JEXL transformation expressions
- **Sync Scheduling**: Configurable full and delta sync intervals with exponential backoff on failure

### Device Trust Service (`device-trust-service`)
- **FleetDM Integration**: Fleet server for osquery management with automatic agent enrollment
- **Posture Assessment**: OS version compliance, disk encryption (FileVault/BitLocker/LUKS), firewall status, antivirus presence
- **Patch Level Compliance**: OS and application patch currency verification against configurable thresholds
- **Jailbreak Detection**: iOS jailbreak and Android root detection via device attestation
- **Conditional Access**: Policy engine denying access from non-compliant devices with customizable grace periods

### MDM Service (`mdm-service`)
- **Apple MDM**: NanoMDM-based Apple device management with DEP/ADE automated enrollment
- **Android Enterprise**: Managed device and work profile provisioning via Android Management API
- **Windows MDM**: OMA-DM protocol support for Windows 10/11 device management
- **App Deployment**: Managed app installation, configuration, and removal across all platforms
- **Remote Commands**: Remote wipe, remote lock, passcode reset, device restart, lost mode activation

### Credential Vault Service (`credential-vault-service`)
- **Secrets Storage**: Secure storage for OAuth2 refresh tokens, API keys, database connection strings, TLS certificates
- **Encryption**: AES-256-GCM envelope encryption with per-tenant data encryption keys (DEKs)
- **Key Hierarchy**: Master Key (HSM) > Tenant Key Encryption Key (KEK) > Data Encryption Key (DEK) > Secret
- **Auto-Rotation**: Configurable rotation schedules with zero-downtime key rollover
- **HSM Support**: PKCS#11 interface for hardware security module integration (AWS CloudHSM, Azure Dedicated HSM, Thales Luna)

### Session Service (`session-service`)
- **Concurrent Limits**: Configurable maximum concurrent sessions per user (default: 5)
- **Timeout Policies**: Idle timeout (default: 30 minutes), absolute timeout (default: 8 hours), sliding window refresh
- **Forced Logout**: Admin-initiated single session or all-session revocation via API
- **Geolocation Restrictions**: Country/region-based session blocking with allowlist/blocklist configuration
- **Session Introspection**: Real-time session viewer with device, IP, location, and activity metadata

### Audit Service (`audit-service`)
- **Event Logging**: All authentication, authorization, provisioning, and administrative events captured with CloudEvents envelope
- **SIEM Integration**: Native connectors for Splunk (HEC), Elasticsearch (Bulk API), Datadog (Logs API)
- **Compliance Reports**: Automated SOC 2 Type II evidence collection and ISO 27001 controls mapping
- **Immutable Logs**: Append-only audit log with SHA-256 chain verification for tamper detection
- **Real-Time Alerts**: Webhook-based alerting for configurable event patterns (e.g., 10+ failed logins in 5 minutes)

---

## API Endpoints

| Service | Base Path | Methods | Auth Required |
|---|---|---|---|
| identity-service | `/v1/identity` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |
| directory-service | `/v1/directory` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |
| provisioning-service | `/v1/provisioning` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |
| device-trust-service | `/v1/device-trust` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |
| mdm-service | `/v1/mdm` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |
| credential-vault-service | `/v1/credential-vault` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |
| session-service | `/v1/session` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |
| audit-service | `/v1/audit` | GET, POST, PUT, PATCH, DELETE | JWT + X-Tenant-ID |

All services expose `GET /healthz` for liveness checks without authentication.

---

## Event Topics

All events follow the `erp.iam.<entity>.<action>` convention with CloudEvents payload envelope:

- `erp.iam.identity.{created|read|updated|deleted|listed}`
- `erp.iam.directory.{created|read|updated|deleted|listed}`
- `erp.iam.provisioning.{created|read|updated|deleted|listed}`
- `erp.iam.device-trust.{created|read|updated|deleted|listed}`
- `erp.iam.mdm.{created|read|updated|deleted|listed}`
- `erp.iam.credential-vault.{created|read|updated|deleted|listed}`
- `erp.iam.session.{created|read|updated|deleted|listed}`
- `erp.iam.audit.{created|read|updated|deleted|listed}`

---

## Known Limitations

1. **Keycloak Custom Themes**: Custom login page theming requires Keycloak theme deployment; hot-reload is not yet supported
2. **Samba AD DC Replication**: Multi-region AD replication is limited to two-site topology in v1.0.0
3. **Android Enterprise**: Zero-touch enrollment requires Android 8.0+ devices
4. **SCIM Bulk Operations**: Bulk SCIM operations are limited to 500 resources per request in v1.0.0

---

## Upgrade Path

This is the initial release. Future upgrade paths will be documented in subsequent release notes.

---

## Dependencies

- **Runtime**: Go 1.22+, Python 3.11+ (Flask webapp)
- **Infrastructure**: Keycloak 24+, Authentik 2024.2+, Samba 4.19+, FleetDM 4.x, NanoMDM 0.5+
- **Data**: YugabyteDB 2.20+, Redis 7.x, NATS 2.10+ / Redpanda 24.x
- **Platform**: ERP-Platform (entitlements), Kubernetes 1.28+, Docker 24+
