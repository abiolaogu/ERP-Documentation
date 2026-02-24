# Contributing to ERP-CRM

Thank you for your interest in contributing to ERP-CRM. This document provides guidelines and processes for contributing to the project.

## Code of Conduct

All contributors are expected to adhere to the project's Code of Conduct. Be respectful, constructive, and collaborative.

## Getting Started

### Prerequisites

```bash
# Rust toolchain
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup component add clippy rustfmt

# Go (for microservices)
# Install from https://go.dev/dl/

# Docker and Docker Compose
# Install from https://docs.docker.com/get-docker/

# PostgreSQL client tools
# macOS: brew install postgresql
# Ubuntu: apt install postgresql-client

# sqlx CLI (for migrations)
cargo install sqlx-cli --no-default-features --features postgres
```

### Development Setup

```bash
# Clone the repository
git clone https://github.com/opensase/ERP-CRM.git
cd ERP-CRM

# Start infrastructure
docker compose up -d db nats

# Set environment
export DATABASE_URL=postgres://postgres:postgres@localhost:5432/crm

# Run migrations
sqlx migrate run --source ./migrations

# Run tests
cargo test --all-features

# Start the development server
cargo run
```

## Development Workflow

```mermaid
flowchart LR
    FORK["Fork Repo"] --> BRANCH["Create Branch"]
    BRANCH --> DEVELOP["Develop + Test"]
    DEVELOP --> LINT["cargo fmt + clippy"]
    LINT --> PR["Open PR"]
    PR --> REVIEW["Code Review"]
    REVIEW --> MERGE["Merge to main"]
```

### Branch Naming

| Type | Format | Example |
|------|--------|---------|
| Feature | `feature/{description}` | `feature/contact-import` |
| Bug Fix | `fix/{description}` | `fix/deal-update-handler` |
| Documentation | `docs/{description}` | `docs/api-documentation` |
| Performance | `perf/{description}` | `perf/contact-query-index` |
| Refactor | `refactor/{description}` | `refactor/deal-aggregate` |

### Commit Messages

Follow the Conventional Commits specification:

```
type(scope): description

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

Examples:
```
feat(contacts): add contact merge service
fix(deals): prevent stage change on closed deals
docs(api): document pagination parameters
test(domain): add deal weighted value tests
perf(queries): add index on contacts.lifecycle_stage
```

## Coding Standards

### Rust Code

1. **Format**: Run `cargo fmt` before committing (enforced in CI)
2. **Lint**: Run `cargo clippy -- -D warnings` (enforced in CI)
3. **Tests**: All new business logic must have unit tests
4. **Documentation**: Add `//!` doc comments to modules and `///` to public functions
5. **Error handling**: Use domain-specific error types, not `anyhow` in domain layer
6. **Architecture**: Follow hexagonal architecture -- domain logic in `domain/`, not in handlers

### Domain-Driven Design Rules

```mermaid
graph TB
    subgraph "Allowed Dependencies"
        DOM3["Domain Layer"] -->|"depends on"| NOTHING["Nothing<br/>(pure Rust + serde)"]
        APP3["Application Layer"] -->|"depends on"| DOM3
        PORTS3["Ports Layer"] -->|"depends on"| DOM3
        INFRA3["Infrastructure"] -->|"depends on"| PORTS3
        HANDLERS["HTTP Handlers"] -->|"depends on"| APP3
    end

    subgraph "Forbidden Dependencies"
        DOM3 -.->|"NEVER depends on"| INFRA3
        DOM3 -.->|"NEVER depends on"| HANDLERS
    end
```

- Aggregates must encapsulate business rules (no logic in handlers)
- Value objects must be immutable
- Domain events must be raised within aggregates
- Repository interfaces defined in `ports/outbound/`
- Use case interfaces defined in `ports/inbound/`

### Go Code

1. **Format**: Run `gofmt` before committing
2. **Lint**: Run `go vet` before committing
3. **Pattern**: Follow the standard Go microservice template
4. **Tenant Isolation**: Always validate `X-Tenant-ID` header

### SQL Migrations

1. Use sequential numbering: `001_initial_schema.sql`, `002_add_territories.sql`
2. Always use `CREATE TABLE IF NOT EXISTS`
3. Always include `ON CONFLICT DO NOTHING` for seed data
4. Always add indexes for foreign keys and common query patterns
5. Use `TIMESTAMPTZ` for all timestamps (not `TIMESTAMP`)

## Testing

### Test Pyramid

```mermaid
graph TB
    UNIT["Unit Tests<br/>Domain aggregates, value objects, services<br/>(Fast, no I/O)"]
    INTEGRATION["Integration Tests<br/>Repository implementations, API handlers<br/>(Requires PostgreSQL)"]
    E2E["End-to-End Tests<br/>Full API workflows<br/>(Requires all services)"]

    UNIT -->|"Most tests"| INTEGRATION
    INTEGRATION -->|"Some tests"| E2E
```

### Running Tests

```bash
# Unit tests (no external dependencies)
cargo test --lib

# All tests (requires PostgreSQL)
cargo test --all-features

# Specific test module
cargo test domain::aggregates::contact

# With output
cargo test -- --nocapture

# Coverage report
cargo tarpaulin --all-features --out html
```

### Writing Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    // Helper factory for test data
    fn create_test_contact() -> Contact {
        let email = Email::new("test@example.com").unwrap();
        Contact::create(email, "John", "Doe", EntityId::new())
    }

    #[test]
    fn test_descriptive_name() {
        // Arrange
        let mut contact = create_test_contact();

        // Act
        contact.qualify().unwrap();

        // Assert
        assert_eq!(contact.lead_status(), &LeadStatus::Qualified);
    }
}
```

## Pull Request Process

1. **Before submitting:**
   - All tests pass: `cargo test --all-features`
   - Code is formatted: `cargo fmt`
   - No clippy warnings: `cargo clippy -- -D warnings`
   - Commit messages follow conventions

2. **PR Description:**
   - Describe what changed and why
   - Reference related issues
   - Include screenshots for UI changes
   - Note any breaking changes

3. **Review Process:**
   - At least one maintainer approval required
   - CI must pass (tests, fmt, clippy, Docker build)
   - All review comments must be resolved

4. **After merge:**
   - Delete the source branch
   - Update documentation if needed
   - Update CHANGELOG.md for significant changes

## Architecture Decision Records

For significant architectural changes, create an ADR:

1. Copy the ADR template from `docs/ADR/`
2. Number sequentially (e.g., `003-new-decision.md`)
3. Include context, decision, alternatives considered, and consequences
4. Submit as part of the PR implementing the change

## Reporting Issues

Use GitHub Issues with these labels:

| Label | Description |
|-------|------------|
| `bug` | Something is not working |
| `feature` | New feature request |
| `enhancement` | Improvement to existing feature |
| `documentation` | Documentation updates |
| `performance` | Performance improvement |
| `security` | Security concern |

## Questions?

- Check existing documentation in `docs/`
- Open a GitHub Discussion for questions
- Review ADRs for architectural context
