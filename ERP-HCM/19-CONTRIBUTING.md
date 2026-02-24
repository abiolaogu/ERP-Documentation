# Contributing to ERP-HCM

Thank you for your interest in contributing to ERP-HCM. This guide covers the development workflow, coding standards, and contribution process.

---

## 1. Getting Started

### 1.1 Prerequisites

- Go 1.24+ (`go version`)
- Node.js 20 LTS (`node --version`)
- PostgreSQL 16+ (local or Docker)
- Redis 7+ (local or Docker)
- NATS Server 2.10+ (local or Docker)
- Docker and Docker Compose
- Git

### 1.2 Fork and Clone

```bash
# Fork the repository on GitHub
# Clone your fork
git clone https://github.com/YOUR_USERNAME/ERP-HCM.git
cd ERP-HCM

# Add upstream remote
git remote add upstream https://github.com/opensase/ERP-HCM.git
```

### 1.3 Branch Naming

```
feature/<ticket-id>-<short-description>    # New features
fix/<ticket-id>-<short-description>         # Bug fixes
chore/<short-description>                   # Maintenance tasks
docs/<short-description>                    # Documentation
refactor/<short-description>                # Code refactoring
```

Example: `feature/HCM-123-add-leave-accrual`

---

## 2. Development Workflow

### 2.1 Create a Branch

```bash
git checkout main
git pull upstream main
git checkout -b feature/HCM-123-add-leave-accrual
```

### 2.2 Make Changes

Follow the coding standards below. Run tests before committing.

### 2.3 Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Types**: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`, `ci`

**Scopes**: `employee`, `payroll`, `leave`, `recruitment`, `performance`, `attendance`, `benefits`, `learning`, `compensation`, `workforce`, `compliance`, `document`, `facilities`, `gateway`, `frontend`, `infra`

**Examples**:
```
feat(payroll): add NHIS statutory deduction calculation
fix(attendance): correct Haversine distance for southern hemisphere
chore(deps): update go-chi/chi to v5.0.13
docs(api): add OpenAPI spec for leave endpoints
test(payroll): add integration test for 13th month payroll run
```

### 2.4 Pull Request

```bash
git push origin feature/HCM-123-add-leave-accrual
```

Create a PR against `main` with:
- Clear title following conventional commit format
- Description of changes and motivation
- Link to related issue/ticket
- Screenshots for UI changes
- Test evidence (passing test output)

---

## 3. Coding Standards

### 3.1 Go Backend

**Package Structure** (per domain):
```
internal/<domain>/
    domain/         # Domain models, value objects, enums (no external deps)
    service/        # Business logic, orchestration
    engine/         # Calculation engines (stateless)
    repository/     # Data access (PostgreSQL via pgx/sqlx)
    handlers/       # HTTP handlers (Chi router)
    adapters/       # External service adapters
```

**Naming Conventions**:
- Files: `snake_case.go`
- Packages: lowercase, single word
- Exported types: `PascalCase`
- Unexported: `camelCase`
- Constants: `PascalCase` for exported, `camelCase` for package-level
- Interfaces: verb-noun (e.g., `EmployeeRepository`, `PayrollCalculator`)

**Error Handling**:
```go
// Always return errors, never panic in production code
result, err := service.ProcessPayroll(ctx, runID)
if err != nil {
    return fmt.Errorf("process payroll run %s: %w", runID, err)
}
```

**Multi-Tenancy**:
```go
// Always extract tenant from context
tenantID := middleware.TenantIDFromContext(ctx)
if tenantID == uuid.Nil {
    return ErrMissingTenantID
}
// Always include tenant in queries
query := "SELECT * FROM employees WHERE tenant_id = $1 AND id = $2"
```

**Financial Calculations**:
```go
// Always use shopspring/decimal, never float64
amount := decimal.NewFromInt(1000000) // 1,000,000 kobo = NGN 10,000
rate := decimal.NewFromFloat(0.08)
pension := amount.Mul(rate).Round(0) // Round to nearest kobo
```

**Testing**:
```go
// Table-driven tests preferred
func TestCalculatePAYE(t *testing.T) {
    tests := []struct {
        name     string
        gross    decimal.Decimal
        expected decimal.Decimal
    }{
        {"minimum wage", decimal.NewFromInt(360000), decimal.NewFromInt(0)},
        {"mid range", decimal.NewFromInt(5000000), decimal.NewFromInt(XXX)},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := CalculatePAYE(tt.gross, pension, nhf, zero)
            assert.True(t, result.Equal(tt.expected))
        })
    }
}
```

### 3.2 Frontend (Next.js/TypeScript)

**File Naming**:
- Components: `PascalCase.tsx`
- Pages: `page.tsx` (App Router convention)
- Hooks: `use-something.ts`
- Utils: `kebab-case.ts`
- Types: `types.ts` or `<domain>.ts`

**Component Pattern**:
```tsx
// Prefer server components by default
// Use "use client" only when needed (interactivity, browser APIs)
export default function EmployeePage() {
  return <EmployeeList />;
}
```

**State Management**:
- Server state: React Query (`@tanstack/react-query`)
- Client state: Zustand (minimal stores)
- Form state: React Hook Form + Zod

### 3.3 SQL Migrations

**Naming**: `{sequence}_{description}.{up|down}.sql`
- Example: `000003_add_leave_accrual.up.sql`

**Rules**:
- Every `up` migration must have a corresponding `down` migration
- Migrations must be idempotent (use `IF NOT EXISTS`, `IF EXISTS`)
- Always include `tenant_id` in new tables
- Add indexes for common query patterns
- Use `TIMESTAMPTZ` for all timestamps

---

## 4. AIDD Guardrails Compliance

All contributions must respect the AIDD guardrails:

```yaml
autonomous_actions:
  - read_only_queries
  - low_risk_notifications

supervised_actions:
  - data_mutations
  - workflow_automation
  - bulk_operations

prohibited_actions:
  - cross_tenant_data_access
  - irreversible_delete_without_backup
  - privilege_escalation
```

- Never write code that accesses data without tenant scoping
- Never implement hard deletes (use soft delete with `deleted_at`)
- Never bypass the approval workflow for payroll or financial operations
- Always log data mutations for audit trail

---

## 5. Testing Requirements

### 5.1 Minimum Coverage

- New domain logic: 80%+ unit test coverage
- Financial calculations: 100% coverage with edge cases
- API handlers: Integration tests for happy path and error cases
- Frontend components: Snapshot tests + interaction tests

### 5.2 Running Tests

```bash
# Backend unit tests
make test

# Backend integration tests (requires Docker)
make test-integration

# Frontend tests
cd imports/hrms_core/web/frontend
npm test

# E2E tests
npm run test:e2e
```

---

## 6. Code Review Checklist

Reviewers should verify:

- [ ] Follows conventional commit message format
- [ ] Includes tests for new functionality
- [ ] Tenant ID is enforced in all queries
- [ ] Financial calculations use decimal, not float
- [ ] PII fields are encrypted
- [ ] Error handling is comprehensive (no swallowed errors)
- [ ] No hardcoded secrets or credentials
- [ ] Migrations are reversible
- [ ] API changes are backward compatible (within v1)
- [ ] AIDD guardrails are respected
- [ ] Documentation is updated if applicable

---

## 7. Release Process

1. Create a release branch: `release/v1.x.y`
2. Update version numbers and changelog
3. Run full test suite
4. Create GitHub Release with tag `v1.x.y`
5. Docker images auto-built and pushed
6. Helm chart updated

---

## 8. Communication

- **Issues**: GitHub Issues for bug reports and feature requests
- **Discussions**: GitHub Discussions for questions and proposals
- **ADRs**: Architecture decisions documented in `docs/ADR/`
- **RFC Process**: Major changes require an RFC (Request for Comments) as a GitHub Discussion before implementation
