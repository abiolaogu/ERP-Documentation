# ERP-IAM Changelog

> **Document ID:** ERP-IAM-CL-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Active
> **Related Documents:** [02-Release-Notes.md](./02-Release-Notes.md)

---

## Format

All notable changes to ERP-IAM are documented in this file. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-02-23

### Added

#### Identity Service
- Keycloak-based multi-tenant identity provider with realm-per-tenant isolation
- OIDC/OAuth 2.0 authorization server with PKCE, device flow, client credentials
- SAML 2.0 service provider and identity provider with metadata exchange
- Social login integration: Google, Microsoft, Apple, Facebook
- Passwordless authentication: FIDO2/WebAuthn, magic links, OTP
- Multi-factor authentication: TOTP, SMS, push notification, hardware keys
- Adaptive risk-based authentication with IP reputation, device fingerprint, behavioral analysis
- Brute force protection with progressive delay escalation and configurable lockout
- RESTful API at `/v1/identity` with full CRUD operations

#### Directory Service
- Authentik-based user directory with organizational unit hierarchy
- Samba AD DC providing full Active Directory Domain Controller functionality
- Domain join support: Windows (native AD), macOS (directory binding), Linux (SSSD/Winbind)
- Group policy object distribution for Windows, macOS, and Linux
- Computer object management with OU-based placement
- LDAP v3 query interface with search filters, pagination, and StartTLS
- Directory synchronization with Azure AD, Google Workspace, on-premises AD
- RESTful API at `/v1/directory` with full CRUD operations

#### Provisioning Service
- SCIM 2.0 server (RFC 7643/7644 compliant) for inbound provisioning
- SCIM 2.0 client for outbound provisioning to downstream applications
- Joiner-Mover-Leaver lifecycle automation with configurable rules
- Declarative attribute mapping engine with JEXL transformation expressions
- Configurable full and delta sync scheduling with exponential backoff
- RESTful API at `/v1/provisioning` with full CRUD operations

#### Device Trust Service
- FleetDM + osquery integration for endpoint telemetry collection
- Zero-trust posture assessment: OS version, encryption, firewall, antivirus, patches
- Jailbreak/root detection for mobile platforms
- Trust score computation with configurable weight factors
- Conditional access policy engine with customizable grace periods
- RESTful API at `/v1/device-trust` with full CRUD operations

#### MDM Service
- Apple MDM via NanoMDM with DEP/ADE automated enrollment
- Android Enterprise managed device and work profile support
- Windows MDM enrollment via OMA-DM protocol
- Managed app deployment, configuration, and removal
- Remote wipe, lock, passcode reset, and restart commands
- RESTful API at `/v1/mdm` with full CRUD operations

#### Credential Vault Service
- Secure storage for OAuth2 tokens, API keys, database credentials, TLS certificates
- AES-256-GCM envelope encryption with per-tenant key hierarchy
- HSM integration via PKCS#11 for master key management
- Automatic credential rotation with configurable schedules and overlap periods
- Full access audit trail for every read/write operation
- RESTful API at `/v1/credential-vault` with full CRUD operations

#### Session Service
- Configurable concurrent session limits per user (default: 5)
- Idle timeout (30 min) and absolute timeout (8 hours) policies
- Admin-initiated forced logout (single or all sessions)
- Geolocation-based session restrictions with allowlist/blocklist
- Redis-backed session storage with sub-millisecond access
- RESTful API at `/v1/session` with full CRUD operations

#### Audit Service
- CloudEvents-compliant event logging for all IAM operations
- SIEM integration: Splunk (HEC), Elasticsearch (Bulk API), Datadog (Logs API)
- SOC 2 Type II evidence report generation
- ISO 27001 controls mapping
- Immutable audit log with SHA-256 chain verification
- Configurable real-time alerting via webhooks
- RESTful API at `/v1/audit` with full CRUD and report operations

#### Infrastructure
- Go 1.22 microservice architecture (8 services)
- YugabyteDB distributed SQL primary store
- Redis cluster for session cache
- NATS JetStream / Redpanda for event streaming
- Docker multi-stage build images (~15MB each)
- Kubernetes deployment manifests with HPA and PDB
- AIDD guardrails enforcement (autonomous/supervised/prohibited action categories)
- CloudEvents envelope for all event emission

#### Consolidation
- Merged ERP-IDaaS2 (identity provider, MFA authenticator, OAuth2 proxy, Flask webapp)
- Merged ERP-Directory (Samba AD DC configs, device trust policies)
- Merged IDaaS/IDaaS2 legacy codebase
- Unified module manifest under `erp.iam` SKU

### Changed
- N/A (initial release)

### Deprecated
- N/A (initial release)

### Removed
- N/A (initial release)

### Fixed
- N/A (initial release)

### Security
- All data encrypted at rest (AES-256) and in transit (TLS 1.3)
- Argon2id password hashing (64MB memory, 3 iterations)
- PKCE mandatory for public OAuth2 clients
- Row-level security on all database tables for tenant isolation
- AIDD guardrails prohibit cross-tenant data access and privilege escalation

---

## Commit History

| Hash | Date | Message |
|---|---|---|
| `603b11a` | 2026-02-23 | chore: isolate imported source trees as nested modules |
| `de0148e` | 2026-02-23 | feat: deep import selected source service directories for consolidation |
| `cbf3b92` | 2026-02-23 | feat: apply consolidation merger contracts and module scaffolds |
| `c4eda00` | 2026-02-23 | chore: initialize ERP-IAM |
