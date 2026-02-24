# ERP-AIOps Contributing Guide

> **Document ID:** ERP-AIOPS-CG-019
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md), [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md)

---

## Welcome

Thank you for your interest in contributing to ERP-AIOps. This document provides guidelines for contributing code, documentation, ML models, and operational runbooks to the AIOps platform. Following these guidelines ensures consistency, quality, and maintainability across the Rust, Python, and Go codebases.

---

## Table of Contents

1. [Getting Started](#1-getting-started)
2. [Development Workflow](#2-development-workflow)
3. [Rust Coding Standards](#3-rust-coding-standards)
4. [Python Style Guide](#4-python-style-guide)
5. [Go Coding Standards](#5-go-coding-standards)
6. [Frontend Conventions](#6-frontend-conventions)
7. [Adding a New Rust Crate](#7-adding-a-new-rust-crate)
8. [ML Model Contribution](#8-ml-model-contribution)
9. [Testing Requirements](#9-testing-requirements)
10. [Code Review Process](#10-code-review-process)
11. [Documentation Standards](#11-documentation-standards)
12. [Commit Message Convention](#12-commit-message-convention)

---

## 1. Getting Started

### Prerequisites

Complete the [20-Local-Environment-Setup.md](./20-Local-Environment-Setup.md) guide to set up your development environment.

### First-Time Setup

```bash
# Clone the repository
git clone https://github.com/your-org/ERP-AIOps.git
cd ERP-AIOps

# Install Rust toolchain components
rustup component add clippy rustfmt
cargo install cargo-watch cargo-nextest cargo-deny cargo-audit

# Set up Python virtual environment
cd ai-brain
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt

# Install Go tools
cd ../gateway
go install golang.org/x/tools/cmd/goimports@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Install frontend dependencies
cd ../frontend
npm install

# Install pre-commit hooks
cd ..
./scripts/install-hooks.sh
```

### Finding Work

- **Good First Issues**: Search for issues labeled `good-first-issue` in the issue tracker.
- **Help Wanted**: Issues labeled `help-wanted` are suitable for contributors with some project familiarity.
- **Feature Requests**: Discuss feature requests in `#erp-aiops-dev` before starting implementation.
- **Bug Reports**: Bug fixes are always welcome. Verify the bug exists on the `main` branch before fixing.

---

## 2. Development Workflow

### Branch Naming

```
feature/AIOPS-{ticket}-{short-description}    # New features
fix/AIOPS-{ticket}-{short-description}         # Bug fixes
refactor/AIOPS-{ticket}-{short-description}    # Refactoring
docs/AIOPS-{ticket}-{short-description}        # Documentation
model/AIOPS-{ticket}-{short-description}       # ML model changes
perf/AIOPS-{ticket}-{short-description}        # Performance improvements
```

**Examples:**
```
feature/AIOPS-142-add-topology-auto-discovery
fix/AIOPS-198-correlation-engine-memory-leak
model/AIOPS-205-improve-anomaly-autoencoder
```

### Workflow Steps

1. **Create branch** from `main`:
   ```bash
   git checkout main && git pull origin main
   git checkout -b feature/AIOPS-142-add-topology-auto-discovery
   ```

2. **Implement changes** following the coding standards below.

3. **Write tests** for all new functionality (see [Testing Requirements](#9-testing-requirements)).

4. **Run local checks**:
   ```bash
   # Rust
   cargo fmt --all --check
   cargo clippy --workspace -- -D warnings
   cargo test --workspace
   cargo deny check

   # Python
   cd ai-brain
   ruff check .
   ruff format --check .
   mypy app/
   pytest tests/ -v

   # Go
   cd gateway
   goimports -l .
   golangci-lint run
   go test ./...

   # Frontend
   cd frontend
   npm run lint
   npm run typecheck
   npm test
   ```

5. **Commit** using the [Commit Message Convention](#12-commit-message-convention).

6. **Push** and create a Pull Request.

7. **Address review feedback** and ensure CI passes.

---

## 3. Rust Coding Standards

### Formatting

- Use `rustfmt` with the project's `rustfmt.toml` configuration. All Rust code must be formatted before committing.
  ```bash
  cargo fmt --all
  ```

- **`rustfmt.toml`:**
  ```toml
  edition = "2021"
  max_width = 100
  use_small_heuristics = "Max"
  imports_granularity = "Module"
  group_imports = "StdExternalCrate"
  ```

### Linting

- All code must pass `cargo clippy` with no warnings:
  ```bash
  cargo clippy --workspace -- -D warnings
  ```

- Enable additional clippy lint groups in `Cargo.toml`:
  ```toml
  [lints.clippy]
  pedantic = "warn"
  nursery = "warn"
  unwrap_used = "deny"
  expect_used = "warn"
  panic = "deny"
  ```

### Error Handling

- **Never use `.unwrap()` in production code.** Use `.expect("reason")` only in tests or truly unreachable code paths.
- Use the `thiserror` crate for defining error types:
  ```rust
  use thiserror::Error;

  #[derive(Error, Debug)]
  pub enum IncidentError {
      #[error("incident not found: {id}")]
      NotFound { id: String },

      #[error("invalid severity: {value}")]
      InvalidSeverity { value: String },

      #[error("database error")]
      Database(#[from] sqlx::Error),

      #[error("unauthorized: {reason}")]
      Unauthorized { reason: String },
  }
  ```

- Use `anyhow::Result` in application code (binary crates) and `thiserror` in library crates.
- Map errors to appropriate HTTP status codes in Axum handlers using the `IntoResponse` trait.

### Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| Crate names | `aiops-{domain}` | `aiops-anomaly`, `aiops-incidents` |
| Struct names | PascalCase | `IncidentService`, `AnomalyDetector` |
| Function names | snake_case | `create_incident`, `detect_anomalies` |
| Constants | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT_MS` |
| Type parameters | Single uppercase or descriptive | `T`, `E`, `Req`, `Res` |
| Module names | snake_case | `incident_handler`, `anomaly_engine` |

### Async/Await Best Practices

- Use `tokio::spawn` for truly concurrent tasks, not for sequential operations.
- Avoid holding locks across `.await` points to prevent deadlocks.
- Use `tokio::select!` for racing multiple futures with cancellation safety.
- Prefer bounded channels (`tokio::sync::mpsc::channel`) over unbounded channels to prevent memory exhaustion.
- Always set timeouts on external calls:
  ```rust
  use tokio::time::{timeout, Duration};

  let result = timeout(Duration::from_secs(5), ai_brain_client.analyze(request))
      .await
      .map_err(|_| AiOpsError::AiBrainTimeout)?;
  ```

### Documentation

- All public items must have doc comments (`///`).
- Include usage examples in doc comments for complex functions:
  ```rust
  /// Detects anomalies in a time series using the adaptive threshold algorithm.
  ///
  /// # Arguments
  /// * `values` - Time series values ordered by timestamp
  /// * `config` - Detection configuration (sensitivity, min_duration, etc.)
  ///
  /// # Returns
  /// A vector of detected anomalies with scores and time ranges.
  ///
  /// # Example
  /// ```
  /// let anomalies = detect_anomalies(&values, &config).await?;
  /// for anomaly in &anomalies {
  ///     println!("Anomaly at {}: score={:.2}", anomaly.timestamp, anomaly.score);
  /// }
  /// ```
  pub async fn detect_anomalies(
      values: &[f64],
      config: &DetectionConfig,
  ) -> Result<Vec<Anomaly>, AnomalyError> { ... }
  ```

### Multi-Tenancy

- All database queries MUST include `tenant_id` in the WHERE clause.
- Extract `tenant_id` from the JWT token using the `TenantId` Axum extractor:
  ```rust
  async fn list_incidents(
      TenantId(tenant_id): TenantId,
      State(db): State<DbPool>,
      Query(params): Query<ListParams>,
  ) -> Result<Json<PaginatedResponse<Incident>>, ApiError> {
      let incidents = sqlx::query_as!(
          Incident,
          "SELECT * FROM incidents WHERE tenant_id = $1 ORDER BY created_at DESC LIMIT $2 OFFSET $3",
          tenant_id,
          params.per_page,
          params.offset()
      )
      .fetch_all(&db)
      .await?;
      // ...
  }
  ```

### Dependency Management

- Use workspace-level dependencies in the root `Cargo.toml`:
  ```toml
  [workspace.dependencies]
  axum = "0.7"
  sqlx = { version = "0.7", features = ["runtime-tokio", "postgres", "uuid", "chrono"] }
  tokio = { version = "1", features = ["full"] }
  serde = { version = "1", features = ["derive"] }
  ```
- Reference workspace dependencies in crate `Cargo.toml`:
  ```toml
  [dependencies]
  axum = { workspace = true }
  sqlx = { workspace = true }
  ```
- Run `cargo deny check` to verify license compliance and ban known-vulnerable dependencies.
- Run `cargo audit` regularly to check for security advisories.

---

## 4. Python Style Guide

### Formatting and Linting

- Use `ruff` for both linting and formatting (replaces `black`, `isort`, `flake8`):
  ```bash
  ruff check .        # Lint
  ruff format .       # Format
  ```

- **`pyproject.toml` configuration:**
  ```toml
  [tool.ruff]
  target-version = "py312"
  line-length = 100

  [tool.ruff.lint]
  select = ["E", "F", "W", "I", "N", "UP", "S", "B", "A", "C4", "DTZ", "T10", "ISC", "ICN", "PIE", "PT", "RSE", "RET", "SLF", "SIM", "TID", "TCH", "ARG", "PTH", "ERA"]

  [tool.ruff.lint.isort]
  known-first-party = ["app"]
  ```

### Type Hints

- All functions must have complete type annotations:
  ```python
  from typing import Optional

  async def analyze_anomaly(
      metric_data: list[float],
      timestamps: list[datetime],
      sensitivity: float = 0.95,
      method: str = "adaptive",
  ) -> AnomalyAnalysisResult:
      ...
  ```

- Use `mypy` for static type checking:
  ```bash
  mypy app/ --strict
  ```

### Pydantic Models

- Use Pydantic v2 models for all request/response schemas:
  ```python
  from pydantic import BaseModel, Field

  class AnomalyDetectionRequest(BaseModel):
      metric_name: str = Field(..., min_length=1, max_length=255)
      values: list[float] = Field(..., min_length=2)
      timestamps: list[datetime] = Field(..., min_length=2)
      detection_method: str = Field(default="adaptive", pattern="^(adaptive|isolation_forest|lstm|ensemble)$")
      sensitivity: float = Field(default=0.95, ge=0.0, le=1.0)
      tenant_id: str = Field(..., min_length=1)

      model_config = {"json_schema_extra": {"examples": [...]}}
  ```

### FastAPI Conventions

- Use dependency injection for shared resources (database, cache, model registry):
  ```python
  from fastapi import Depends

  async def get_model_registry() -> ModelRegistry:
      return app.state.model_registry

  @router.post("/analyze/anomaly")
  async def analyze_anomaly(
      request: AnomalyDetectionRequest,
      registry: ModelRegistry = Depends(get_model_registry),
  ) -> AnomalyAnalysisResponse:
      ...
  ```

- Use routers to organize endpoints by domain:
  ```python
  # app/routers/anomaly.py
  router = APIRouter(prefix="/analyze", tags=["anomaly"])
  ```

### Error Handling

- Use custom exception classes with HTTP status mapping:
  ```python
  class ModelNotFoundError(Exception):
      def __init__(self, model_name: str):
          self.model_name = model_name
          super().__init__(f"Model not found: {model_name}")

  @app.exception_handler(ModelNotFoundError)
  async def model_not_found_handler(request: Request, exc: ModelNotFoundError):
      return JSONResponse(
          status_code=404,
          content={"type": "model_not_found", "detail": str(exc)},
      )
  ```

### Logging

- Use `structlog` for structured logging:
  ```python
  import structlog
  logger = structlog.get_logger()

  logger.info("anomaly_detected", metric=metric_name, score=0.95, tenant_id=tenant_id)
  ```

---

## 5. Go Coding Standards

### Formatting

- Use `goimports` (superset of `gofmt`):
  ```bash
  goimports -w .
  ```

### Linting

- Use `golangci-lint` with the project configuration:
  ```bash
  golangci-lint run
  ```

### Conventions

- Follow [Effective Go](https://go.dev/doc/effective_go) and [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments).
- Use descriptive variable names; avoid single-letter variables except in short loops.
- Return errors instead of panicking. Use `fmt.Errorf` with `%w` for error wrapping:
  ```go
  if err != nil {
      return fmt.Errorf("failed to proxy request to rust api: %w", err)
  }
  ```
- Use context propagation (`context.Context`) for all HTTP handlers and downstream calls.
- Define interfaces at the consumer, not the producer.

### Project Structure

```
gateway/
├── cmd/server/main.go       # Entry point
├── internal/
│   ├── routes/routes.go     # Route definitions
│   ├── middleware/
│   │   ├── auth.go          # JWT validation
│   │   ├── ratelimit.go     # Rate limiting
│   │   ├── logging.go       # Request/response logging
│   │   └── cors.go          # CORS handling
│   ├── proxy/
│   │   ├── rust_api.go      # Rust API reverse proxy
│   │   └── ai_brain.go      # AI Brain reverse proxy
│   ├── health/
│   │   └── aggregator.go    # Health check aggregation
│   └── config/
│       └── config.go        # Configuration loading
├── go.mod
├── go.sum
└── Dockerfile
```

---

## 6. Frontend Conventions

### Framework and Libraries

- **Framework**: Refine.dev 4.x with Ant Design 5.x
- **Language**: TypeScript (strict mode)
- **Build Tool**: Vite
- **State Management**: Refine.dev's built-in data hooks
- **GraphQL Client**: `graphql-request` + `graphql-ws`

### Component Structure

```typescript
// pages/incidents/list.tsx
import { List, useTable, TextField, TagField, DateField } from "@refinedev/antd";
import { Table } from "antd";

export const IncidentList: React.FC = () => {
  const { tableProps } = useTable<IIncident>({
    resource: "incidents",
    sorters: { initial: [{ field: "created_at", order: "desc" }] },
  });

  return (
    <List>
      <Table {...tableProps} rowKey="id">
        <Table.Column dataIndex="title" title="Title" render={(value) => <TextField value={value} />} />
        <Table.Column dataIndex="severity" title="Severity" render={(value) => <TagField value={value} />} />
        <Table.Column dataIndex="created_at" title="Created" render={(value) => <DateField value={value} />} />
      </Table>
    </List>
  );
};
```

### Theme

- Primary color: `#7c3aed` (purple)
- Use the shared theme configuration from `src/theme/index.ts`
- Do not hardcode colors; use Ant Design's token system

### Naming

| Item | Convention | Example |
|------|-----------|---------|
| Components | PascalCase | `IncidentList.tsx`, `AnomalyChart.tsx` |
| Hooks | camelCase with `use` prefix | `useIncidents.ts`, `useAnomalyStream.ts` |
| Utilities | camelCase | `formatSeverity.ts`, `parseTimestamp.ts` |
| Types/Interfaces | PascalCase with `I` prefix | `IIncident`, `IAnomaly` |

---

## 7. Adding a New Rust Crate

When adding a new domain or infrastructure crate to the workspace:

### Step 1: Create the Crate

```bash
# Create the crate directory
mkdir -p crates/aiops-{name}/src

# Initialize Cargo.toml
cat > crates/aiops-{name}/Cargo.toml << 'EOF'
[package]
name = "aiops-{name}"
version = "0.1.0"
edition = "2021"
description = "AIOps {Description}"

[dependencies]
aiops-core = { path = "../aiops-core" }
aiops-models = { path = "../aiops-models" }
aiops-db = { path = "../aiops-db" }
serde = { workspace = true }
tokio = { workspace = true }
tracing = { workspace = true }
thiserror = { workspace = true }

[dev-dependencies]
tokio-test = "0.4"
sqlx = { workspace = true, features = ["runtime-tokio", "postgres"] }
EOF

# Create lib.rs
cat > crates/aiops-{name}/src/lib.rs << 'EOF'
//! AIOps {Name} crate.
//!
//! Provides {description of functionality}.

pub mod error;
pub mod service;

pub use error::{Error, Result};
pub use service::{Name}Service;
EOF
```

### Step 2: Register in Workspace

Add the new crate to the root `Cargo.toml`:

```toml
[workspace]
members = [
    # ... existing crates ...
    "crates/aiops-{name}",
]
```

### Step 3: Add API Routes

If the crate exposes HTTP endpoints, add routes in `crates/aiops-api/src/routes/`:

```rust
// crates/aiops-api/src/routes/{name}.rs
use axum::{Router, routing::{get, post}};

pub fn routes() -> Router<AppState> {
    Router::new()
        .route("/", get(list).post(create))
        .route("/:id", get(get_by_id).put(update).delete(delete))
}
```

Register the routes in `crates/aiops-api/src/routes/mod.rs`:

```rust
pub fn api_routes() -> Router<AppState> {
    Router::new()
        // ... existing routes ...
        .nest("/api/v1/{name}", {name}::routes())
}
```

### Step 4: Add Database Migration

Create a new migration file in `migrations/`:

```sql
-- migrations/NNN_{name}_tables.sql
CREATE TABLE {name} (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id TEXT NOT NULL,
    -- domain-specific columns
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_{name}_tenant_id ON {name} (tenant_id);
CREATE INDEX idx_{name}_tenant_created ON {name} (tenant_id, created_at DESC);
```

### Step 5: Add Tests

Create test files following the testing requirements (see [Testing Requirements](#9-testing-requirements)):

```bash
mkdir -p crates/aiops-{name}/tests
touch crates/aiops-{name}/tests/integration_test.rs
```

### Step 6: Verify

```bash
# Build the new crate
cargo build -p aiops-{name}

# Run tests
cargo test -p aiops-{name}

# Check lints
cargo clippy -p aiops-{name} -- -D warnings

# Verify workspace compiles
cargo check --workspace
```

---

## 8. ML Model Contribution

### Model Development Workflow

1. **Research Phase**: Explore the problem in Jupyter notebooks under `ai-brain/notebooks/`.
2. **Prototype**: Implement the model in `ai-brain/app/models/`.
3. **Evaluate**: Run evaluation metrics on test datasets.
4. **Package**: Export the model to ONNX format (preferred) or pickle.
5. **Integrate**: Add inference endpoint in `ai-brain/app/routers/`.
6. **Test**: Write unit and integration tests.

### Model Requirements

| Requirement | Details |
|-------------|---------|
| **Format** | ONNX (preferred), pickle (scikit-learn only), PyTorch state dict |
| **Size Limit** | Max 500MB per model artifact |
| **Inference Latency** | P99 < 500ms for single inference |
| **Batch Support** | Must support batch inference for throughput |
| **Versioning** | Semantic versioning (e.g., `anomaly-detector-v1.2.0.onnx`) |
| **Documentation** | Model card documenting architecture, training data, metrics, limitations |

### Model Card Template

Every model contribution must include a model card (`model_card.md`):

```markdown
# Model Card: {Model Name}

## Overview
- **Task**: {e.g., Anomaly Detection, Forecasting, Correlation}
- **Architecture**: {e.g., Isolation Forest, LSTM Autoencoder, GNN}
- **Version**: {semantic version}
- **Training Date**: {date}

## Training Data
- **Dataset**: {description of training data}
- **Size**: {number of samples}
- **Features**: {list of input features}
- **Label Distribution**: {class balance information}

## Performance Metrics
| Metric | Value |
|--------|-------|
| Precision | {value} |
| Recall | {value} |
| F1 Score | {value} |
| AUC-ROC | {value} |
| P99 Inference Latency | {value} |

## Limitations
- {Known limitations and failure modes}

## Ethical Considerations
- {Bias analysis, fairness considerations}
```

### Model Storage

- Model artifacts are stored in RustFS under the `aiops-models` bucket.
- Directory structure: `s3://aiops-models/{model-type}/{version}/{artifact}`
- Example: `s3://aiops-models/anomaly-detector/v1.2.0/model.onnx`

---

## 9. Testing Requirements

### Minimum Coverage Requirements

| Component | Unit Test Coverage | Integration Test Coverage |
|-----------|-------------------|--------------------------|
| Rust crates | 80% line coverage | Critical paths covered |
| Python AI Brain | 75% line coverage | All endpoints covered |
| Go Gateway | 70% line coverage | All routes covered |
| Frontend | 60% line coverage | Core user flows covered |

### Rust Testing

```bash
# Run all tests
cargo test --workspace

# Run with coverage (using cargo-llvm-cov)
cargo llvm-cov --workspace --html

# Run with nextest (faster, better output)
cargo nextest run --workspace
```

**Unit Test Example:**

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_anomaly_score_calculation() {
        let values = vec![100.0, 102.0, 98.0, 101.0, 250.0];
        let baseline = 100.25;
        let std_dev = 1.71;

        let score = calculate_anomaly_score(values[4], baseline, std_dev);
        assert!(score > 0.9, "Expected high anomaly score for large deviation");
    }

    #[tokio::test]
    async fn test_create_incident() {
        let db = setup_test_db().await;
        let service = IncidentService::new(db);

        let incident = service.create(CreateIncident {
            tenant_id: "test-tenant".to_string(),
            title: "Test Incident".to_string(),
            severity: Severity::High,
            ..Default::default()
        }).await.unwrap();

        assert_eq!(incident.title, "Test Incident");
        assert_eq!(incident.status, Status::Open);
    }
}
```

### Python Testing

```bash
# Run all tests
cd ai-brain && pytest tests/ -v

# Run with coverage
pytest tests/ --cov=app --cov-report=html

# Run specific test file
pytest tests/test_anomaly_detection.py -v
```

**Test Example:**

```python
import pytest
from app.services.anomaly import AnomalyDetectionService

class TestAnomalyDetection:
    @pytest.fixture
    def service(self):
        return AnomalyDetectionService()

    def test_adaptive_threshold_detects_spike(self, service):
        normal_values = [100.0] * 100
        spike_values = normal_values + [500.0, 520.0, 510.0]

        anomalies = service.detect(spike_values, method="adaptive")

        assert len(anomalies) > 0
        assert anomalies[0].score > 0.8

    def test_seasonal_adjustment(self, service):
        # Create data with daily seasonality
        daily_pattern = [100 + 50 * math.sin(2 * math.pi * i / 24) for i in range(168)]  # 7 days

        anomalies = service.detect(daily_pattern, method="adaptive")
        assert len(anomalies) == 0, "Seasonal pattern should not trigger anomalies"
```

### Integration Testing

Integration tests require a running Docker Compose stack:

```bash
# Start test infrastructure
docker compose -f infra/docker-compose.test.yml up -d

# Run integration tests
./scripts/run-tests.sh integration

# Cleanup
docker compose -f infra/docker-compose.test.yml down -v
```

### ML Model Validation Tests

```python
class TestAnomalyModel:
    def test_model_precision_above_threshold(self, model, test_dataset):
        predictions = model.predict(test_dataset.X)
        precision = precision_score(test_dataset.y, predictions)
        assert precision >= 0.85, f"Model precision {precision} below threshold 0.85"

    def test_model_inference_latency(self, model, sample_input):
        start = time.perf_counter()
        for _ in range(100):
            model.predict(sample_input)
        avg_latency = (time.perf_counter() - start) / 100
        assert avg_latency < 0.005, f"Average inference latency {avg_latency}s exceeds 5ms"

    def test_model_deterministic(self, model, sample_input):
        result1 = model.predict(sample_input)
        result2 = model.predict(sample_input)
        np.testing.assert_array_equal(result1, result2)
```

---

## 10. Code Review Process

### Pull Request Requirements

- [ ] All CI checks pass (build, lint, test, security scan)
- [ ] Code coverage does not decrease
- [ ] New public APIs have documentation
- [ ] New features have tests
- [ ] Database migrations are backward-compatible
- [ ] Multi-tenant isolation is maintained (`tenant_id` in all queries)
- [ ] No secrets or credentials in the diff
- [ ] Changelog entry added for user-facing changes

### Review Checklist

**Reviewers should verify:**

1. **Correctness**: Does the code do what it claims?
2. **Security**: Are there SQL injection, XSS, or tenant isolation issues?
3. **Performance**: Are there N+1 queries, unbounded allocations, or missing indexes?
4. **Error Handling**: Are all error cases handled? Are errors propagated correctly?
5. **Testing**: Are edge cases tested? Are tests meaningful (not just testing mocks)?
6. **Documentation**: Are public APIs documented? Are complex algorithms explained?
7. **Multi-Tenancy**: Is `tenant_id` correctly scoped in all database operations?

### Approval Requirements

| Change Type | Required Approvals | Required Reviewers |
|-------------|-------------------|--------------------|
| Bug fix | 1 | Any team member |
| Feature | 2 | At least 1 senior engineer |
| Database migration | 2 | DBA + senior engineer |
| Security-related | 2 | Security team member + senior engineer |
| ML model | 2 | ML engineer + senior engineer |
| Architecture change | 3 | Architecture board member |

---

## 11. Documentation Standards

- All user-facing features must be documented.
- API endpoints must have OpenAPI/Swagger documentation (auto-generated via Axum/FastAPI).
- Runbooks must be updated for operational changes (see [24-Runbooks.md](./24-Runbooks.md)).
- Architecture Decision Records for significant technical decisions (see [18-Architecture-Decision-Records.md](./18-Architecture-Decision-Records.md)).

---

## 12. Commit Message Convention

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code refactoring (no behavior change) |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `docs` | Documentation changes |
| `chore` | Build, CI, tooling changes |
| `model` | ML model changes |
| `security` | Security-related changes |

### Scopes

| Scope | Description |
|-------|-------------|
| `incidents` | Incident management |
| `anomaly` | Anomaly detection |
| `rca` | Root cause analysis |
| `correlation` | Event correlation |
| `topology` | Topology mapping |
| `remediation` | Auto-remediation |
| `cost` | Cost optimization |
| `security` | Security scanning |
| `forecasting` | Forecasting |
| `ai-brain` | Python AI Brain |
| `gateway` | Go Gateway |
| `frontend` | Frontend application |
| `db` | Database/migrations |
| `api` | API changes |
| `infra` | Infrastructure/Docker |

### Examples

```
feat(anomaly): add LSTM autoencoder for sequence anomaly detection

Implemented an LSTM autoencoder model in the AI Brain for detecting
anomalies in sequential metric data. The model supports variable-length
input sequences and outputs reconstruction error as anomaly score.

Closes AIOPS-205
```

```
fix(correlation): resolve memory leak in event buffer

The correlation engine's event buffer was not being drained when
correlation groups were finalized, causing unbounded memory growth
under sustained high event volume. Added explicit buffer cleanup
in the group finalization path.

Fixes AIOPS-312
```

---

## Questions and Support

- **Slack**: `#erp-aiops-dev`
- **Office Hours**: Tuesdays and Thursdays 2-3 PM UTC
- **Architecture Questions**: `#erp-aiops-architecture`

---

*Thank you for contributing to ERP-AIOps. Your contributions make the platform better for everyone.*
