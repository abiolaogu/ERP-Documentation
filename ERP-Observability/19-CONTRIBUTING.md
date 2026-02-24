# Contributing to ERP-Observability

> **Document ID:** ERP-OBS-CTR-019
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md), [30-Test-Strategy.md](./30-Test-Strategy.md)

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [Development Workflow](#development-workflow)
3. [Code Standards](#code-standards)
4. [Pull Request Process](#pull-request-process)
5. [Testing Requirements](#testing-requirements)
6. [Documentation Requirements](#documentation-requirements)
7. [Commit Conventions](#commit-conventions)
8. [Review Guidelines](#review-guidelines)

---

## 1. Getting Started

### Prerequisites

Ensure you have the following installed:

- **Go 1.22+** -- For the gateway and tenant-api
- **Node.js 20.x LTS** -- For the observability-api
- **Docker Desktop 4.25+** -- For local infrastructure
- **Make 3.81+** -- For build automation

See [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md) for detailed installation instructions.

### First-Time Setup

```bash
# Clone the repository
git clone https://github.com/your-org/ERP-Observability.git
cd ERP-Observability

# Install Node.js dependencies
cd services/observability-api && npm install && cd ../..

# Start infrastructure
docker-compose up -d

# Create Quickwit indexes
./scripts/create-indexes.sh

# Run database migrations
psql -h localhost -p 5433 -U erp -d erp_observability -f migrations/001_initial_schema.sql

# Verify everything is running
./scripts/verify-observability.sh
```

### Repository Layout

| Directory | Language | Purpose |
|---|---|---|
| `cmd/server/` | Go | API gateway (port 8090) |
| `services/observability-api/` | TypeScript | Observability REST API (port 3000) |
| `configs/` | YAML/JSON | Infrastructure configuration |
| `migrations/` | SQL | Database schema migrations |
| `infra/` | Dockerfile/YAML | Docker and Kubernetes manifests |
| `scripts/` | Bash | Operational scripts |
| `docs/` | Markdown | Internal documentation |
| `hasura/` | YAML | Hasura metadata |

---

## 2. Development Workflow

### Branch Strategy

We use a trunk-based development model with short-lived feature branches:

```
main (protected)
  |
  +-- feat/OBS-123-add-trace-sampling
  +-- fix/OBS-456-search-cache-ttl
  +-- chore/OBS-789-update-quickwit-config
  +-- docs/OBS-101-update-api-docs
```

**Branch naming convention:** `<type>/<ticket>-<short-description>`

| Type | Purpose |
|---|---|
| `feat/` | New feature or enhancement |
| `fix/` | Bug fix |
| `chore/` | Maintenance, dependency updates, config changes |
| `docs/` | Documentation only |
| `refactor/` | Code restructuring without behavior change |
| `perf/` | Performance improvement |
| `test/` | Test additions or fixes |

### Development Loop

1. Create a branch from `main`:
   ```bash
   git checkout main && git pull
   git checkout -b feat/OBS-123-add-trace-sampling
   ```

2. Make changes with tests.

3. Run local validation:
   ```bash
   # Go
   go vet ./...
   go test ./...

   # TypeScript
   cd services/observability-api
   npm run lint
   npm test
   ```

4. Commit using conventional commit format (see [Commit Conventions](#commit-conventions)).

5. Push and create a pull request.

---

## 3. Code Standards

### Go (Gateway, Tenant API)

**Style:**
- Follow the [Effective Go](https://go.dev/doc/effective_go) guidelines.
- Use `gofmt` (enforced by CI).
- Use `go vet` and `staticcheck` for linting.
- No global mutable state except for atomic counters and package-level loggers.
- Error handling: Always check errors. Do not use `_` to discard errors except in deferred close operations.

**Naming:**
- Package names: lowercase, single word (e.g., `server`, `tenant`, `config`).
- Exported functions: PascalCase with descriptive names.
- Unexported functions: camelCase.
- Interfaces: Do not use `I` prefix. Use `-er` suffix for single-method interfaces (e.g., `Writer`, `Reader`).
- Constants: PascalCase for exported, camelCase for unexported.

**Structure:**
```go
// Standard library imports first, then third-party, then internal
import (
    "fmt"
    "net/http"

    "github.com/some/pkg"

    "erp/erp_observability/internal/config"
)
```

**HTTP Handlers:**
- Use `http.HandlerFunc` pattern.
- Always set `Content-Type` header before writing response body.
- Use the `writeJSON` helper for JSON responses.
- Log with structured fields: `method`, `path`, `status`, `duration_ms`, `tenant`, `request_id`.

**Example:**
```go
func handleListTenants(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        writeJSON(w, http.StatusMethodNotAllowed, map[string]string{"error": "method not allowed"})
        return
    }

    tenants, err := store.ListTenants(r.Context())
    if err != nil {
        log.Printf("list tenants failed: %v", err)
        writeJSON(w, http.StatusInternalServerError, map[string]string{
            "error": "internal_error",
            "detail": "Failed to list tenants",
        })
        return
    }

    writeJSON(w, http.StatusOK, map[string]any{
        "data":  tenants,
        "total": len(tenants),
    })
}
```

### TypeScript (Observability API)

**Style:**
- Use TypeScript strict mode (`"strict": true` in tsconfig.json).
- Use ESLint with the project configuration (enforced by CI).
- Use Prettier for formatting (4-space indent, single quotes, trailing commas).
- Prefer `const` over `let`. Never use `var`.
- Use explicit return types for all exported functions.
- Use Zod for runtime input validation on all API endpoints.

**Naming:**
- Files: kebab-case (e.g., `log-aggregator.ts`, `tenant-context.ts`).
- Interfaces: PascalCase without `I` prefix (e.g., `TenantContext`, `LogEntry`).
- Enums: PascalCase for enum name, UPPER_SNAKE_CASE for enum values (e.g., `Module.FINANCE`).
- Functions: camelCase.
- Constants: UPPER_SNAKE_CASE for module-level constants, camelCase for function-scoped.

**Structure:**
```typescript
// External imports first, then internal, then types
import { Router, Request, Response } from 'express';
import { z } from 'zod';

import { LogAggregator } from '../controllers/log-aggregator';
import { Module } from '../types';
import type { TenantContext } from '../middleware/tenant-context';
```

**Route Handler Pattern:**
```typescript
router.get('/', async (req: Request, res: Response) => {
  try {
    // 1. Validate input with Zod
    const params = querySchema.parse(req.query);

    // 2. Extract tenant context
    const tenantCtx = req.tenantContext as TenantContext;

    // 3. Execute business logic
    const results = await service.query(params, tenantCtx);

    // 4. Return response
    res.json(results);
  } catch (err) {
    // 5. Handle validation errors distinctly
    if (err instanceof z.ZodError) {
      res.status(400).json({
        error: 'validation_error',
        detail: err.errors.map(e => `${e.path.join('.')}: ${e.message}`).join('; '),
      });
      return;
    }

    // 6. Log and return generic error
    req.app.locals.logger.error({ err }, 'Operation failed');
    res.status(500).json({ error: 'operation_failed', detail: 'An error occurred' });
  }
});
```

**Error Response Format:**
All API errors must follow this structure:
```json
{
  "error": "error_code",
  "detail": "Human-readable description",
  "request_id": "req-1234567890-1"
}
```

### React / Frontend

**Style:**
- Use functional components with hooks exclusively. No class components.
- Use Refine.dev hooks for data operations (`useList`, `useOne`, `useCreate`, `useUpdate`, `useDelete`).
- Use Ant Design components for UI. Do not use raw HTML elements where an Ant Design equivalent exists.
- Colocate component-specific styles using CSS Modules or Ant Design's style overrides.

**Naming:**
- Component files: PascalCase (e.g., `LogViewer.tsx`, `AlertRuleForm.tsx`).
- Hook files: camelCase with `use` prefix (e.g., `useLogStream.ts`).
- Utility files: camelCase (e.g., `formatDuration.ts`).

**Component Structure:**
```tsx
import React from 'react';
import { useList } from '@refinedev/core';
import { Table, Tag } from 'antd';
import type { LogEntry } from '../types';

interface LogTableProps {
  module?: string;
  level?: string;
}

export const LogTable: React.FC<LogTableProps> = ({ module, level }) => {
  const { data, isLoading } = useList<LogEntry>({
    resource: 'observability/logs',
    filters: [
      { field: 'module', operator: 'eq', value: module },
      { field: 'level', operator: 'eq', value: level },
    ],
  });

  return (
    <Table
      dataSource={data?.data}
      loading={isLoading}
      rowKey="request_id"
      columns={[
        { title: 'Timestamp', dataIndex: 'timestamp' },
        { title: 'Level', dataIndex: 'level', render: (val) => <Tag>{val}</Tag> },
        { title: 'Message', dataIndex: 'message' },
      ]}
    />
  );
};
```

### SQL Migrations

- File naming: `NNN_description.sql` (e.g., `001_initial_schema.sql`, `002_add_webhook_config.sql`).
- All tables must include `tenant_id TEXT NOT NULL` (except system tables like `search_index_status`).
- All tables with `tenant_id` must have an index on `tenant_id`.
- Use `UUID` primary keys generated via `gen_random_uuid()`.
- Use `TIMESTAMPTZ` for all timestamp columns.
- Include `created_at` and `updated_at` columns on all mutable tables.
- Use `JSONB` for semi-structured data (labels, annotations, panels, metadata).
- Migrations must be idempotent (`CREATE TABLE IF NOT EXISTS`, `CREATE INDEX IF NOT EXISTS`).

### Configuration Files (YAML)

- Use 2-space indentation.
- Include comments for non-obvious settings.
- Use environment variable substitution (`${VAR_NAME}`) for secrets.
- Never commit actual secrets or credentials.

---

## 4. Pull Request Process

### Before Opening a PR

1. Ensure all tests pass locally.
2. Ensure linting passes (`go vet`, `npm run lint`).
3. Update documentation if the change affects API contracts, configuration, or architecture.
4. Rebase onto latest `main` to avoid merge conflicts.

### PR Template

When creating a PR, use the following template:

```markdown
## Summary

Brief description of what this PR does and why.

## Changes

- [ ] List of specific changes

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated (if applicable)
- [ ] Manual testing performed

## Documentation

- [ ] API documentation updated (if API changes)
- [ ] Configuration documentation updated (if config changes)
- [ ] Changelog entry added (if user-facing)

## Related Issues

Closes #<issue-number>
```

### Review Requirements

| Change Type | Required Reviewers | Approval Count |
|---|---|---|
| Feature (new endpoint, new capability) | 2 engineers + 1 architect | 2 |
| Bug fix | 1 engineer | 1 |
| Configuration change | 1 engineer + 1 SRE | 2 |
| Security-related change | 1 engineer + 1 security reviewer | 2 |
| Documentation only | 1 engineer | 1 |
| Infrastructure (K8s, Docker, CI) | 1 engineer + 1 SRE | 2 |
| Database migration | 1 engineer + 1 DBA | 2 |

### Merge Strategy

- **Squash and merge** for feature branches with multiple work-in-progress commits.
- **Rebase and merge** for clean, single-commit branches.
- **Never** use merge commits on `main`.

---

## 5. Testing Requirements

### Minimum Requirements

| Component | Minimum Coverage | Test Framework |
|---|---|---|
| Go gateway | 70% | Go testing |
| TypeScript API | 80% | Vitest |
| React frontend | 70% | Vitest + React Testing Library |
| SQL migrations | Manual verification | psql on YugabyteDB |

### Test Categories

**Unit Tests (required for all changes):**
- Test individual functions and methods in isolation.
- Mock external dependencies (database, Quickwit, Prometheus).
- Test validation schemas (Zod) with valid and invalid inputs.
- Test error handling paths.

**Integration Tests (required for API changes):**
- Test full request/response cycle through Express routes.
- Use test database (YugabyteDB) with isolated test data.
- Test authentication and tenant isolation.
- Test Quickwit query construction.

**E2E Tests (required for frontend changes):**
- Playwright tests for critical user flows.
- Test log viewing, search, alert management, dashboard creation.

See [30-Test-Strategy.md](./30-Test-Strategy.md) for the comprehensive testing strategy.

### Running Tests

```bash
# Go tests
go test ./... -v -race

# TypeScript unit tests
cd services/observability-api && npm test

# TypeScript tests with coverage
cd services/observability-api && npm test -- --coverage

# Lint
cd services/observability-api && npm run lint

# Frontend tests
cd frontend && npm test

# E2E tests (requires running services)
cd frontend && npx playwright test
```

---

## 6. Documentation Requirements

### When to Update Documentation

| Change Type | Documentation Required |
|---|---|
| New API endpoint | [21-API-Documentation.md](./21-API-Documentation.md), [22-Postman-Collection.md](./22-Postman-Collection.md) |
| New configuration option | [17-README.md](./17-README.md) environment variables table |
| New infrastructure component | [17-README.md](./17-README.md) architecture, [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md) |
| Database schema change | [27-Entity-Relationship-Diagram.md](./27-Entity-Relationship-Diagram.md), [28-Data-Dictionary.md](./28-Data-Dictionary.md), migration file |
| Security change | [31-SECURITY.md](./31-SECURITY.md) |
| User-facing feature | [16-Changelog.md](./16-Changelog.md) |
| Operational procedure | [24-Runbooks.md](./24-Runbooks.md) |
| Webhook change | [23-Webhook-Specifications.md](./23-Webhook-Specifications.md) |

### Documentation Style

- Use present tense ("adds support for" not "added support for").
- Include code examples for all API changes.
- Include mermaid diagrams for architectural changes.
- Keep sentences concise and technical.
- Do not use emojis in documentation.

---

## 7. Commit Conventions

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types:**

| Type | Description |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation changes |
| `style` | Code style changes (formatting, no logic change) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `chore` | Build, CI, dependency updates |
| `ci` | CI/CD configuration changes |

**Scopes:**

| Scope | Description |
|---|---|
| `gateway` | Go gateway (cmd/server/) |
| `api` | Node.js observability-api |
| `tenant-api` | Go tenant-api |
| `frontend` | React frontend |
| `config` | Configuration files |
| `infra` | Docker/K8s infrastructure |
| `migration` | Database migrations |
| `quickwit` | Quickwit index configuration |
| `grafana` | Grafana dashboards |
| `alerting` | Alertmanager/alert rules |

**Examples:**

```
feat(api): add trace sampling rate configuration endpoint

fix(gateway): handle upstream timeout with proper 504 response

chore(infra): update Quickwit image to 0.8.1

docs(api): update search endpoint documentation with new filters

test(api): add integration tests for alert rule CRUD

refactor(api): extract tenant filter into reusable middleware
```

---

## 8. Review Guidelines

### For Reviewers

When reviewing a PR, check the following:

**Correctness:**
- Does the code do what it claims to do?
- Are edge cases handled (empty inputs, null values, large payloads)?
- Are error messages clear and actionable?

**Security:**
- Is tenant isolation maintained? All queries must use `buildTenantFilter()` or `buildTenantSQLFilter()`.
- Are admin-only operations protected by role checks?
- Are inputs validated with Zod before use?
- Are SQL queries parameterized (no string interpolation)?
- Are secrets excluded from logs?

**Performance:**
- Are database queries using appropriate indexes?
- Is caching used for expensive operations?
- Are Quickwit queries using tag fields for partition pruning?
- Are large result sets paginated?

**Maintainability:**
- Is the code self-documenting with clear variable and function names?
- Are complex operations commented?
- Is the code DRY without being overly abstracted?
- Are TypeScript types explicit (no `any` except in legacy integration code)?

**Tests:**
- Are the right things being tested (behavior, not implementation)?
- Are error paths tested?
- Are mocks appropriate (not mocking the thing under test)?

### For Authors

- Keep PRs small and focused. Aim for <400 lines of code changes.
- Respond to all review comments, even if just to acknowledge.
- Do not force-push over review comments (push new commits instead).
- Request re-review after addressing feedback.

---

## Code of Conduct

All contributors are expected to adhere to the ERP platform's Code of Conduct. We are committed to providing a welcoming and respectful environment for all contributors regardless of background or experience level.

---

## Questions?

If you have questions about contributing, reach out to the Platform Engineering team via the `#erp-observability` Slack channel or open a discussion in the repository.
