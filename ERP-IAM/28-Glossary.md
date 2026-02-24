# ERP-IAM Glossary

> **Document ID:** ERP-IAM-GL-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** All ERP-IAM documents

---

## Terms and Definitions

| Term | Definition |
|---|---|
| **ABAC** | Attribute-Based Access Control -- authorization model that evaluates attributes of the user, resource, action, and environment to make access decisions |
| **AD DC** | Active Directory Domain Controller -- a server that responds to authentication requests and manages domain-joined device policies (Samba implementation) |
| **ADE** | Apple Device Enrollment (formerly DEP) -- Apple's zero-touch enrollment program for MDM |
| **AIDD** | AI-Integrated Development Discipline -- guardrail framework governing autonomous, supervised, and prohibited actions for AI-driven operations |
| **Argon2id** | Password hashing algorithm combining Argon2i (side-channel resistant) and Argon2d (GPU resistant), specified in RFC 9106 |
| **Authentik** | Open-source identity provider and user directory used as ERP-IAM's cloud-native user store |
| **CAE** | Continuous Access Evaluation -- real-time session revocation based on policy changes or risk signals |
| **CloudEvents** | CNCF specification for describing event data in a common, interoperable way (used for all ERP-IAM events) |
| **Conditional Access** | Policy framework that evaluates conditions (user, device, location, risk) to make access allow/block decisions |
| **CTAP** | Client to Authenticator Protocol -- FIDO2 protocol for communication between browser and external authenticator |
| **DEK** | Data Encryption Key -- symmetric key used to encrypt individual secrets, itself encrypted by a KEK |
| **DEP** | Device Enrollment Program -- Apple's automated MDM enrollment (now called ADE) |
| **DRS** | Directory Replication Service -- Active Directory protocol for replicating directory data between domain controllers |
| **Envelope Encryption** | Encryption pattern where data is encrypted with a DEK, the DEK is encrypted with a KEK, and the KEK is protected by an HSM |
| **FIDO2** | Fast IDentity Online 2 -- authentication standard using public key cryptography, comprising WebAuthn and CTAP |
| **FleetDM** | Open-source osquery management server for endpoint visibility and compliance |
| **GPO** | Group Policy Object -- collection of settings used to define user and computer configurations in Active Directory |
| **HSM** | Hardware Security Module -- dedicated cryptographic processor for key generation, storage, and operations (FIPS 140-2 certified) |
| **JML** | Joiner-Mover-Leaver -- identity lifecycle management pattern for employee onboarding, role changes, and offboarding |
| **JWT** | JSON Web Token -- compact, URL-safe token format for claims between two parties (RFC 7519) |
| **KEK** | Key Encryption Key -- symmetric key used to encrypt DEKs, itself protected by the HSM master key |
| **Keycloak** | Open-source identity and access management server by Red Hat, used as ERP-IAM's identity provider |
| **LDAP** | Lightweight Directory Access Protocol -- standard protocol for accessing directory information services (RFC 4511) |
| **MFA** | Multi-Factor Authentication -- requiring two or more verification methods (something you know, have, or are) |
| **NanoMDM** | Lightweight open-source Apple MDM server used by ERP-IAM for iOS/macOS device management |
| **NATS** | Open-source messaging system for cloud-native applications, used for event streaming via JetStream |
| **OIDC** | OpenID Connect -- identity layer on top of OAuth 2.0 for authentication (ID tokens, UserInfo endpoint) |
| **OMA-DM** | Open Mobile Alliance Device Management -- protocol used for Windows MDM |
| **osquery** | Open-source SQL-based endpoint agent exposing operating system state as virtual SQL tables |
| **OTP** | One-Time Password -- single-use code for authentication, delivered via SMS, email, or authenticator app |
| **OU** | Organizational Unit -- container in Active Directory for organizing users, groups, computers, and policies |
| **PKCE** | Proof Key for Code Exchange -- OAuth 2.0 extension preventing authorization code interception (RFC 7636) |
| **PKCS#11** | Standard API for cryptographic token interfaces (smart cards, HSMs) defined by OASIS |
| **RBAC** | Role-Based Access Control -- authorization model assigning permissions to roles, which are assigned to users |
| **Realm** | Keycloak concept representing a tenant's isolated identity domain with its own users, clients, and identity providers |
| **RLS** | Row-Level Security -- PostgreSQL feature restricting row access based on the executing user/role |
| **SAML** | Security Assertion Markup Language -- XML-based standard for exchanging authentication and authorization data (OASIS) |
| **SCEP** | Simple Certificate Enrollment Protocol -- certificate management protocol used for MDM device certificate distribution |
| **SCIM** | System for Cross-domain Identity Management -- standard for automating user provisioning (RFC 7643/7644) |
| **SPI** | Service Provider Interface -- Keycloak extension mechanism for custom authenticators, user storage, event listeners |
| **SSSD** | System Security Services Daemon -- Linux service for accessing remote directories and authentication mechanisms |
| **STRIDE** | Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege -- threat modeling framework |
| **SYSVOL** | System Volume -- shared directory in Active Directory storing group policy templates and logon scripts |
| **TDE** | Transparent Data Encryption -- database-level encryption where data is encrypted/decrypted transparently |
| **TOTP** | Time-based One-Time Password -- OTP algorithm using current time and a shared secret (RFC 6238) |
| **Trust Score** | Numerical value (0-100) representing a device's security posture, computed from weighted compliance checks |
| **WebAuthn** | Web Authentication API -- W3C standard enabling FIDO2 authentication in browsers |
| **YugabyteDB** | Distributed SQL database compatible with PostgreSQL, used as ERP-IAM's primary data store |
| **Zero Trust** | Security model assuming no implicit trust, requiring continuous verification of every access request |
