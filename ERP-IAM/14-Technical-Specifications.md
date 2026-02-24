# ERP-IAM Technical Specifications

> **Document ID:** ERP-IAM-TS-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [04-Software-Architecture.md](./04-Software-Architecture.md), [12-High-Level-Design.md](./12-High-Level-Design.md), [13-Low-Level-Design.md](./13-Low-Level-Design.md)

---

## 1. System Specifications

### 1.1 Runtime Environment

| Component | Specification |
|---|---|
| **Language (Services)** | Go 1.22+ (standard library net/http) |
| **Language (Webapp)** | Python 3.11+ (Flask 3.x) |
| **Language (Mobile)** | Dart 3.x (Flutter 3.x) |
| **Container Runtime** | Docker 24+, OCI-compliant |
| **Orchestration** | Kubernetes 1.28+ |
| **Service Mesh** | Istio 1.20+ or Linkerd 2.14+ (optional) |

### 1.2 Data Store Specifications

| Store | Version | Protocol | Purpose |
|---|---|---|---|
| **YugabyteDB** | 2.20+ | PostgreSQL wire (5432) | Primary relational store |
| **Redis** | 7.x | RESP3 (6379) | Session cache, rate limiting |
| **NATS** | 2.10+ | NATS protocol (4222) | Event streaming (JetStream) |
| **Redpanda** | 24.x | Kafka protocol (9092) | Alternative event streaming |

### 1.3 Infrastructure Component Specifications

| Component | Version | Protocol | Port |
|---|---|---|---|
| **Keycloak** | 24+ | HTTP/HTTPS | 8080/8443 |
| **Authentik** | 2024.2+ | HTTP/HTTPS | 9000/9443 |
| **Samba AD DC** | 4.19+ | LDAP/LDAPS/Kerberos/DNS | 389/636/88/53 |
| **FleetDM** | 4.x | HTTPS | 8412 |
| **osquery** | 5.x | TLS (to FleetDM) | N/A (agent) |
| **NanoMDM** | 0.5+ | HTTPS | 9000 |

---

## 2. Protocol Specifications

### 2.1 OIDC/OAuth 2.0

| Parameter | Value |
|---|---|
| **Authorization endpoint** | `/auth/realms/{realm}/protocol/openid-connect/auth` |
| **Token endpoint** | `/auth/realms/{realm}/protocol/openid-connect/token` |
| **UserInfo endpoint** | `/auth/realms/{realm}/protocol/openid-connect/userinfo` |
| **JWKS endpoint** | `/auth/realms/{realm}/protocol/openid-connect/certs` |
| **Discovery** | `/auth/realms/{realm}/.well-known/openid-configuration` |
| **Supported grant types** | authorization_code, client_credentials, refresh_token, device_code |
| **PKCE** | Required for public clients (S256) |
| **Token format** | JWT (RS256 or ES256 signing) |
| **Access token lifetime** | 300-3600 seconds (configurable, default: 3600) |
| **Refresh token lifetime** | 1800-86400 seconds (configurable, default: 86400) |
| **ID token lifetime** | 300-3600 seconds (matches access token) |
| **Supported scopes** | openid, profile, email, groups, roles, offline_access |

### 2.2 SAML 2.0

| Parameter | Value |
|---|---|
| **SSO endpoint** | `/auth/realms/{realm}/protocol/saml` |
| **SLO endpoint** | `/auth/realms/{realm}/protocol/saml` |
| **Metadata** | `/auth/realms/{realm}/protocol/saml/descriptor` |
| **Name ID formats** | persistent, transient, email, unspecified |
| **Assertion signing** | RSA-SHA256 |
| **Assertion encryption** | AES-256-CBC (optional) |
| **Binding** | HTTP-POST, HTTP-Redirect |
| **Assertion lifetime** | 300 seconds |

### 2.3 LDAP

| Parameter | Value |
|---|---|
| **Protocol** | LDAPv3 (RFC 4511) |
| **Port** | 389 (LDAP), 636 (LDAPS) |
| **TLS** | StartTLS on 389, native TLS on 636 |
| **Base DN** | `dc={domain},dc={tld}` per tenant |
| **Bind methods** | Simple bind, SASL (GSSAPI/EXTERNAL) |
| **Search** | Base, one-level, subtree scopes |
| **Pagination** | Simple Paged Results (RFC 2696), max 1000 per page |
| **Referrals** | Supported (configurable chase depth) |

### 2.4 SCIM 2.0

| Parameter | Value |
|---|---|
| **Base URL** | `/v1/provisioning/scim/v2` |
| **Spec compliance** | RFC 7643 (Core Schema), RFC 7644 (Protocol) |
| **Supported resources** | Users, Groups |
| **Authentication** | Bearer token |
| **Filtering** | SCIM filter syntax (eq, ne, co, sw, ew, pr, gt, ge, lt, le, and, or, not) |
| **Sorting** | Supported on userName, name.familyName, meta.created |
| **Pagination** | startIndex + count (1-based indexing) |
| **Bulk operations** | Max 500 operations per request |
| **Patch** | RFC 7644 Section 3.5.2 (Add, Remove, Replace) |
| **ETags** | Supported for optimistic concurrency |

### 2.5 WebAuthn/FIDO2

| Parameter | Value |
|---|---|
| **Spec** | WebAuthn Level 2 (W3C) |
| **RP ID** | Configurable per tenant (domain) |
| **Attestation** | none, indirect, direct (configurable) |
| **User verification** | required, preferred, discouraged (configurable) |
| **Authenticator attachment** | platform, cross-platform, or any |
| **Algorithms** | ES256 (-7), RS256 (-257) |
| **Credential storage** | Server-side public key storage |
| **Discoverable credentials** | Supported (resident keys) |

---

## 3. Security Specifications

### 3.1 Cryptographic Standards

| Purpose | Algorithm | Key Size | Standard |
|---|---|---|---|
| Token signing | RS256 or ES256 | 2048-bit RSA / P-256 ECDSA | RFC 7518 |
| Secret encryption | AES-256-GCM | 256-bit | NIST SP 800-38D |
| Password hashing | Argon2id | 64MB memory, 3 iterations | RFC 9106 |
| Audit chain | SHA-256 | 256-bit | FIPS 180-4 |
| TLS | TLS 1.3 | ECDHE + AES-256-GCM | RFC 8446 |
| HSM key management | PKCS#11 | Various | OASIS PKCS#11 v3.0 |
| TOTP | HMAC-SHA1 / HMAC-SHA256 | 160/256-bit | RFC 6238 |

### 3.2 Password Policy Defaults

| Policy | Default Value | Range |
|---|---|---|
| Minimum length | 12 | 8-128 |
| Require uppercase | Yes | Boolean |
| Require lowercase | Yes | Boolean |
| Require digit | Yes | Boolean |
| Require special character | Yes | Boolean |
| Password history | 12 (cannot reuse last 12) | 0-24 |
| Maximum age | 90 days | 0 (disabled) - 365 |
| Minimum age | 1 day | 0-30 |
| Lockout threshold | 5 failed attempts | 3-20 |
| Lockout duration | 30 minutes | 1-1440 minutes |
| Progressive delay | 1s, 2s, 4s, 8s, 16s | Configurable steps |

### 3.3 Rate Limiting

| Endpoint Category | Rate Limit | Window |
|---|---|---|
| Authentication | 10 requests per account | 1 minute |
| Token issuance | 30 requests per client | 1 minute |
| SCIM operations | 100 requests per connection | 1 minute |
| LDAP searches | 50 requests per bind | 1 minute |
| Admin API | 200 requests per admin | 1 minute |
| Health checks | Unlimited | N/A |

---

## 4. Performance Specifications

### 4.1 Latency SLOs

| Operation | p50 Target | p99 Target | Max |
|---|---|---|---|
| OIDC token issuance | < 30ms | < 150ms | 500ms |
| SAML assertion generation | < 40ms | < 200ms | 500ms |
| LDAP simple bind | < 10ms | < 80ms | 200ms |
| LDAP search (100 results) | < 20ms | < 100ms | 300ms |
| SCIM user creation | < 50ms | < 300ms | 1000ms |
| Session validation | < 5ms | < 20ms | 50ms |
| Device posture check | < 100ms | < 500ms | 2000ms |
| Audit log write | < 5ms | < 25ms | 100ms |
| Credential decryption | < 10ms | < 50ms | 200ms |

### 4.2 Throughput SLOs

| Operation | Per Node | Cluster (3 nodes) |
|---|---|---|
| Authentications/sec | 5,000 | 15,000 |
| Token validations/sec | 20,000 | 60,000 |
| LDAP operations/sec | 10,000 | 30,000 |
| SCIM operations/sec | 2,000 | 6,000 |
| Session operations/sec | 20,000 | 60,000 |
| Audit writes/sec | 30,000 | 90,000 |

### 4.3 Availability SLOs

| Service Tier | Uptime Target | Monthly Downtime Budget |
|---|---|---|
| Authentication (P0) | 99.99% | 4.32 minutes |
| Directory (P0) | 99.99% | 4.32 minutes |
| Session (P0) | 99.99% | 4.32 minutes |
| Provisioning (P1) | 99.95% | 21.6 minutes |
| Device Trust (P1) | 99.95% | 21.6 minutes |
| MDM (P1) | 99.95% | 21.6 minutes |
| Audit (P1) | 99.95% | 21.6 minutes |
| Credential Vault (P0) | 99.99% | 4.32 minutes |

---

## 5. API Specifications

### 5.1 Request/Response Format

- **Content-Type**: `application/json` for all request/response bodies
- **Character Encoding**: UTF-8
- **Date Format**: ISO 8601 with timezone (`2026-02-23T10:00:00Z`)
- **ID Format**: UUID v4 (lowercase, hyphenated: `550e8400-e29b-41d4-a716-446655440000`)
- **Pagination**: Offset-based (`page` + `page_size`) with total count
- **Sorting**: `sort` parameter with `+` (asc) or `-` (desc) prefix
- **Filtering**: SCIM-style filter expressions on SCIM endpoints, query parameters elsewhere

### 5.2 HTTP Status Codes

| Code | Meaning | Usage |
|---|---|---|
| 200 | OK | Successful GET, PUT, PATCH, DELETE |
| 201 | Created | Successful POST creating a resource |
| 204 | No Content | Successful DELETE with no response body |
| 400 | Bad Request | Invalid input, missing required fields |
| 401 | Unauthorized | Missing or invalid JWT token |
| 403 | Forbidden | Valid token but insufficient permissions |
| 404 | Not Found | Resource does not exist |
| 409 | Conflict | Duplicate resource (e.g., username already exists) |
| 423 | Locked | Account locked due to brute force protection |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server error |

---

## 6. Configuration Specifications

### 6.1 Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | HTTP server listen port |
| `MODULE_NAME` | `ERP-IAM` | Module identifier |
| `YUGABYTE_DSN` | `postgres://localhost:5433/iam` | YugabyteDB connection string |
| `REDIS_URL` | `redis://localhost:6379` | Redis connection URL |
| `NATS_URL` | `nats://localhost:4222` | NATS server URL |
| `KEYCLOAK_URL` | `http://localhost:8080/auth` | Keycloak base URL |
| `KEYCLOAK_ADMIN_USER` | `admin` | Keycloak admin username |
| `KEYCLOAK_ADMIN_PASS` | (required) | Keycloak admin password |
| `AUTHENTIK_URL` | `http://localhost:9000` | Authentik base URL |
| `SAMBA_DC_HOST` | `localhost` | Samba AD DC hostname |
| `FLEETDM_URL` | `https://localhost:8412` | FleetDM server URL |
| `NANOMDM_URL` | `https://localhost:9000` | NanoMDM server URL |
| `HSM_PKCS11_LIB` | `/usr/lib/softhsm/libsofthsm2.so` | PKCS#11 library path |
| `HSM_SLOT` | `0` | HSM slot number |
| `HSM_PIN` | (required) | HSM user PIN |
| `LOG_LEVEL` | `info` | Log level (debug, info, warn, error) |
| `LOG_FORMAT` | `json` | Log format (json, text) |
