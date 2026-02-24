# ERP-Platform Dependency Manifest

> **Document ID:** ERP-PLAT-DM-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Related Documents:** [31-SECURITY.md](./31-SECURITY.md), [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md)

---

## 1. Dependency Philosophy

ERP-Platform follows a **zero-dependency core** philosophy for its Go microservices. The subscription-hub and all platform services use only the Go standard library, resulting in no direct third-party Go module dependencies. This approach:

- Eliminates supply chain vulnerability risk from Go dependencies.
- Reduces dependency update churn.
- Produces minimal binary sizes (8-15MB).
- Ensures long-term stability (stdlib is backward compatible).

---

## 2. Go Direct Dependencies

### 2.1 Subscription Hub

```
module erp-platform/subscription-hub
go 1.22
```

| Package | Source | License | Purpose |
|---------|--------|---------|---------|
| encoding/json | Go stdlib | BSD-3-Clause | JSON serialization/deserialization |
| errors | Go stdlib | BSD-3-Clause | Error creation and wrapping |
| fmt | Go stdlib | BSD-3-Clause | String formatting |
| log | Go stdlib | BSD-3-Clause | Structured logging |
| net/http | Go stdlib | BSD-3-Clause | HTTP server and client |
| os | Go stdlib | BSD-3-Clause | Environment variable access, file reading |
| path | Go stdlib | BSD-3-Clause | File path manipulation |
| strings | Go stdlib | BSD-3-Clause | String manipulation |
| sync | Go stdlib | BSD-3-Clause | RWMutex for concurrent access |

**Third-party Go dependencies: NONE**

### 2.2 Other Platform Services (Common)

All non-hub services share the same dependency profile:

| Package | Source | License | Purpose |
|---------|--------|---------|---------|
| encoding/json | Go stdlib | BSD-3-Clause | JSON serialization |
| log | Go stdlib | BSD-3-Clause | Logging |
| net/http | Go stdlib | BSD-3-Clause | HTTP server |
| os | Go stdlib | BSD-3-Clause | Environment variables |
| strings | Go stdlib | BSD-3-Clause | Path string manipulation |

**Third-party Go dependencies: NONE**

---

## 3. Infrastructure Dependencies

### 3.1 Container Base Images

| Image | Version | License | Vulnerability Status | Size |
|-------|---------|---------|---------------------|------|
| golang:1.22-alpine | 1.22.x | BSD-3-Clause (Go), MIT (Alpine) | Scan on every build via Trivy | ~300MB (build only) |
| alpine:3.20 | 3.20.x | MIT | Scan on every build via Trivy | ~7MB (runtime) |

### 3.2 Database and Middleware

| Dependency | Version | License | Purpose | Vulnerability Monitoring |
|-----------|---------|---------|---------|------------------------|
| PostgreSQL | 16.x | PostgreSQL License (MIT-like) | Primary relational database | postgresql.org/security |
| Redis | 7.x | RSALv2 + SSPLv1 (server), BSD-3-Clause (client) | Caching, session, rate limiting | redis.io/security |
| NATS Server | 2.10.x | Apache-2.0 | Event streaming (JetStream) | nats.io/advisories |

### 3.3 Docker Compose Services

From `infra/docker-compose.platform.yml`:

| Service | Image | Version | License | Ports |
|---------|-------|---------|---------|-------|
| postgres | postgres:16 | 16.x | PostgreSQL License | 5432 |
| redis | redis:7 | 7.x | RSALv2 | 6379 |
| nats | nats:2.10 | 2.10.x | Apache-2.0 | 4222 |

---

## 4. Frontend Dependencies

### 4.1 Activation Console (imports/bac_activation/frontend)

| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| react | ^18.x | MIT | UI component library |
| react-dom | ^18.x | MIT | React DOM renderer |
| typescript | ^5.x | Apache-2.0 | Type-safe JavaScript |
| vite | ^5.x | MIT | Build tool and dev server |
| tailwindcss | ^3.x | MIT | Utility-first CSS framework |
| postcss | ^8.x | MIT | CSS processing |
| autoprefixer | ^10.x | MIT | CSS vendor prefixing |
| msw | ^2.x | MIT | Mock Service Worker (dev only) |
| eslint | ^8.x | MIT | Code linting |

### 4.2 Admin Console (Next.js)

| Package | Version | License | Purpose |
|---------|---------|---------|---------|
| next | ^14.x | MIT | React framework with SSR |
| react | ^18.x | MIT | UI library |
| react-dom | ^18.x | MIT | DOM rendering |
| typescript | ^5.x | Apache-2.0 | Type safety |

---

## 5. CI/CD Dependencies

### 5.1 GitHub Actions

| Action | Version | License | Purpose |
|--------|---------|---------|---------|
| actions/checkout | v4 | MIT | Repository checkout |
| actions/setup-python | v5 | MIT | Python setup for doc-gen |
| actions/setup-go | v5 | MIT | Go toolchain setup |

### 5.2 Development Tools

| Tool | Version | License | Purpose |
|------|---------|---------|---------|
| Python | 3.11 | PSF License | Documentation generation |
| golangci-lint | Latest | GPL-3.0 | Go linting |
| gosec | Latest | Apache-2.0 | Security static analysis |
| govulncheck | Latest | BSD-3-Clause | Go vulnerability check |
| Trivy | Latest | Apache-2.0 | Container and filesystem scanning |
| k6 | Latest | AGPL-3.0 | Load testing |
| jq | 1.7+ | MIT | JSON processing |

---

## 6. License Summary

| License | Count | Risk Level | Notes |
|---------|-------|-----------|-------|
| BSD-3-Clause | 15+ | Low | Go stdlib, Go tools |
| MIT | 20+ | Low | Alpine, React, Vite, Tailwind, most npm packages |
| Apache-2.0 | 5+ | Low | TypeScript, NATS, gosec, Trivy |
| PostgreSQL License | 1 | Low | Similar to MIT/BSD |
| RSALv2 / SSPLv1 | 1 | Medium | Redis server (OK for internal use; not for competing service) |
| AGPL-3.0 | 1 | Medium | k6 load testing (dev tool only, not distributed) |
| GPL-3.0 | 1 | Medium | golangci-lint (dev tool only, not distributed) |

### License Compliance Notes

- **No GPL/AGPL-licensed code** is included in production Docker images.
- GPL/AGPL tools (golangci-lint, k6) are development-only dependencies.
- Redis RSALv2 license permits use as an internal caching service.
- All production dependencies are MIT, BSD, Apache, or PostgreSQL licensed.

---

## 7. Vulnerability Status

### 7.1 Current Status (2026-02-23)

| Category | Scan Tool | Critical | High | Medium | Low |
|----------|----------|----------|------|--------|-----|
| Go Dependencies | govulncheck | 0 | 0 | 0 | 0 |
| Container Images | Trivy | 0 | 0 | TBD | TBD |
| Static Analysis | gosec | 0 | 0 | TBD | TBD |
| Secret Detection | Trivy FS | 0 | 0 | 0 | 0 |

### 7.2 Vulnerability Resolution SLA

| Severity | Resolution Timeline |
|----------|-------------------|
| Critical | 24 hours |
| High | 7 days |
| Medium | 30 days |
| Low | Next scheduled update |

---

## 8. Update Policy

### 8.1 Go Toolchain

- Patch updates applied within 48 hours of release.
- Minor version updates evaluated within 1 week.
- Major version updates evaluated during quarterly planning.

### 8.2 Infrastructure Images

- Security patches: Rebuild and redeploy within 1 week.
- Feature updates: Evaluate during monthly maintenance window.
- Major version upgrades: Plan and execute during quarterly release cycle.

### 8.3 Frontend Dependencies

- Security patches via `npm audit fix`: Applied within 48 hours.
- Minor updates: Monthly via `npm update`.
- Major updates: Quarterly planning and migration.

---

## 9. Lock File Strategy

| Ecosystem | Lock File | Location | Strategy |
|-----------|----------|----------|----------|
| Go | go.sum | services/subscription-hub/go.sum | Auto-generated; committed to repo |
| npm | package-lock.json | imports/bac_activation/frontend/package-lock.json | Committed to repo; exact versions |
| Docker | Image digests | CI/CD pipeline | Pin to SHA256 digest in production manifests |
| GitHub Actions | Action versions | .github/workflows/*.yml | Pin to major version (v4, v5) |

---

## 10. Transitive Dependency Analysis

### Go Services

Since ERP-Platform core services have **zero third-party Go dependencies**, there are **zero transitive dependencies** to track. This is a deliberate architectural decision (see [ADR-007](./18-Architecture-Decision-Records.md)).

### Frontend

Transitive dependency count for activation console:

| Direct Dependencies | Transitive Dependencies | Total |
|--------------------|------------------------|-------|
| ~10 | ~200 (typical for React + Vite) | ~210 |

All transitive dependencies are scanned via `npm audit` in the CI pipeline.

---

*For security policy, see [31-SECURITY.md](./31-SECURITY.md). For deployment pipeline, see [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md).*
