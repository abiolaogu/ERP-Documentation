# Contributing to ERP-Platform

> **Document ID:** ERP-PLAT-CONTRIB-001
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Related Documents:** [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md), [30-Test-Strategy.md](./30-Test-Strategy.md)

---

## Code of Conduct

All contributors are expected to adhere to our Code of Conduct. We are committed to providing a welcoming and inclusive experience for everyone. Harassment, discrimination, and disruptive behavior will not be tolerated.

---

## Development Workflow

### 1. Fork and Clone

```bash
# Fork the repository via GitHub UI
git clone https://github.com/<your-username>/ERP-Platform.git
cd ERP-Platform
git remote add upstream https://github.com/org/ERP-Platform.git
```

### 2. Create a Feature Branch

```bash
git checkout -b feat/your-feature-name
# Branch naming conventions:
#   feat/description     - New features
#   fix/description      - Bug fixes
#   docs/description     - Documentation changes
#   refactor/description - Code refactoring
#   chore/description    - Maintenance tasks
#   test/description     - Test additions/fixes
```

### 3. Make Changes

- Follow the coding standards below.
- Write tests for new functionality.
- Update documentation if behavior changes.
- Ensure AIDD guardrails compliance for any AI-related changes.

### 4. Commit Changes

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types:** `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`

**Scopes:** `subscription-hub`, `tenant-provisioner`, `entitlement-engine`, `module-registry`, `marketplace`, `audit-service`, `notification-hub`, `web-hosting`, `activation-wizard`, `catalog`, `infra`, `docs`

**Examples:**
```
feat(subscription-hub): add PostgreSQL persistence layer
fix(entitlement-engine): correct cache invalidation on subscription update
docs(api): update OpenAPI spec with new webhook endpoints
test(marketplace): add integration tests for module installation flow
chore(infra): upgrade PostgreSQL from 15 to 16 in docker-compose
```

### 5. Push and Create Pull Request

```bash
git push origin feat/your-feature-name
```

Then create a Pull Request via GitHub against the `main` branch.

---

## Pull Request Template

```markdown
## Description
<!-- What does this PR do? Why is this change needed? -->

## Type of Change
- [ ] New feature (non-breaking change adding functionality)
- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] Breaking change (fix or feature causing existing functionality to change)
- [ ] Documentation update
- [ ] Refactoring (no functional changes)
- [ ] Infrastructure/CI change

## Affected Services
- [ ] subscription-hub
- [ ] tenant-provisioner
- [ ] entitlement-engine
- [ ] module-registry
- [ ] marketplace
- [ ] audit-service
- [ ] notification-hub
- [ ] web-hosting
- [ ] activation-wizard
- [ ] catalog
- [ ] infrastructure

## Testing
- [ ] Unit tests pass (`make test`)
- [ ] Integration tests pass (`make test-integration`)
- [ ] Manual testing completed
- [ ] No regression in existing tests

## AIDD Compliance
- [ ] Not applicable (no AI-related changes)
- [ ] AIDD guardrails reviewed and maintained
- [ ] Guardrail thresholds not modified
- [ ] If thresholds modified: AI Ethics Board approval attached

## Documentation
- [ ] README updated (if needed)
- [ ] API docs updated (if endpoints changed)
- [ ] ADR created (if architectural decision)
- [ ] Changelog entry added

## Checklist
- [ ] Code follows project coding standards
- [ ] Self-review completed
- [ ] No hardcoded credentials or secrets
- [ ] No TODO comments without linked issue
- [ ] Commit messages follow Conventional Commits
```

---

## Coding Standards

### Go Code

- **Formatting**: Run `gofmt` and `go vet` before committing.
- **Linting**: Use `golangci-lint` with the project's configuration.
- **Error Handling**: Always check errors; never use `_` for error returns in production code.
- **Naming**: Follow Go conventions (exported names are PascalCase, unexported are camelCase).
- **Dependencies**: Prefer standard library. External dependencies require Architecture Council approval.
- **Testing**: Minimum 80% coverage for new code. Use table-driven tests.
- **Documentation**: Every exported function and type must have a GoDoc comment.

```go
// Good: Descriptive function with error handling
func (s *Store) get(tenantID string) (SubscriptionRecord, bool) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    rec, ok := s.recs[tenantID]
    return rec, ok
}
```

### JSON/YAML

- JSON files: 4-space indentation, trailing newline.
- YAML files: 2-space indentation, no tabs.
- Catalog JSON: All SKUs lowercase with hyphens (e.g., `erp-crm`).

### Docker

- Multi-stage builds required.
- Use `golang:1.22-alpine` for build stage, `alpine:3.20` for runtime.
- No sensitive data in Dockerfiles.
- Expose only required ports.

---

## Review Process

1. **Automated Checks**: CI runs lint, test, security scan on every PR.
2. **Peer Review**: Minimum one approval from a team member.
3. **Architecture Review**: Required for:
   - New services
   - API contract changes
   - Database schema changes
   - External dependency additions
   - AIDD guardrail modifications
4. **Security Review**: Required for:
   - Authentication/authorization changes
   - Encryption changes
   - Tenant isolation changes
5. **Merge**: Squash-and-merge to maintain clean history.

---

## Release Process

### Versioning

We follow [Semantic Versioning](https://semver.org/):
- **MAJOR**: Breaking API changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

### Release Steps

1. Create release branch: `release/v1.x.x`
2. Update changelog and version references.
3. Run full test suite including e2e.
4. Create GitHub Release with release notes.
5. CI/CD automatically builds and deploys to staging.
6. QA validation on staging.
7. Promote to production via CD pipeline.
8. Tag the release: `git tag v1.x.x`

---

## Getting Help

- **Documentation**: See the [documentation suite](./17-README.md) for comprehensive references.
- **Issues**: Open a GitHub Issue for bug reports or feature requests.
- **Discussions**: Use GitHub Discussions for questions and proposals.
- **Slack**: #erp-platform-dev channel for real-time discussion.

---

*For local setup instructions, see [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md). For test strategy, see [30-Test-Strategy.md](./30-Test-Strategy.md).*
