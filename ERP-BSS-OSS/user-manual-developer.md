# Developer User Manual -- ERP-BSS-OSS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Introduction

This manual covers how to set up a local development environment, understand the codebase structure, build and test services, consume APIs, and contribute to the ERP-BSS-OSS platform.

---

## 2. Development Environment Setup

### 2.1 Prerequisites

| Tool | Version | Installation |
|------|---------|-------------|
| Rust | 1.83+ | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Go | 1.22+ | `brew install go` or download from golang.org |
| Docker | 24+ | `brew install --cask docker` |
| Docker Compose | 2.0+ | Included with Docker Desktop |
| PostgreSQL client | 16 | `brew install postgresql@16` |
| Redis CLI | 7 | `brew install redis` |

### 2.2 Clone and Build

```bash
git clone https://github.com/abiolaogu/BSS-OSS.git
cd BSS-OSS

# Start infrastructure dependencies
docker-compose up -d

# Build all Rust crates
cargo build

# Run all tests
cargo test --all

# Start the API server
cargo run --bin bss-api-server
```

### 2.3 Local Development Architecture

```mermaid
graph TB
    DEV["Developer Machine"]
    subgraph "Docker Compose"
        PG["PostgreSQL :5432"]
        RD["Redis :6379"]
        MG["MongoDB :27017"]
        RMQ["RabbitMQ :5672 / :15672"]
        KF["Kafka :9092"]
        ZK["ZooKeeper :2181"]
        JAEGER["Jaeger :16686"]
        PROM["Prometheus :9091"]
        GRAFANA["Grafana :3000"]
    end

    DEV -->|"cargo run"| API["bss-api :8080"]
    API --> PG & RD & MG & RMQ & KF
    API --> JAEGER
```

---

## 3. Project Structure

```
ERP-BSS-OSS/
  Cargo.toml              # Workspace manifest
  crates/
    bss-core/             # Shared infrastructure (DB, cache, MQ)
    bss-ddd/              # Domain-driven design building blocks
    bss-api/              # REST API server (Axum)
    bss-billing/          # Billing domain logic
    bss-crm/              # CRM domain logic
    bss-ordering/         # Order management domain logic
    bss-inventory/        # Resource inventory domain logic
    bss-analytics/        # Analytics (ClickHouse)
    bss-integration/      # External system adapters
  services/
    billing-rating-service/     # Go microservice
    customer-management-service/
    order-management-service/
    ... (30 services)
  web/                    # Frontend (React)
  flutter/                # Mobile app
  android/                # Android native
  ios/                    # iOS native
  infrastructure/         # Terraform, Helm charts
  kubernetes/             # K8s manifests
  scripts/                # Utility scripts
  tests/                  # Integration tests
```

---

## 4. API Development Guide

### 4.1 Adding a New API Endpoint (Rust)

1. Define the route in the appropriate crate handler module
2. Implement the handler function:

```rust
use axum::{extract::Path, Json};
use uuid::Uuid;

pub async fn get_customer(
    Path(id): Path<Uuid>,
) -> Result<Json<Customer>, AppError> {
    let customer = customer_repo.find_by_id(id).await?;
    Ok(Json(customer))
}
```

3. Register the route in the router:

```rust
Router::new()
    .route("/customers/:id", get(get_customer))
    .route("/customers", post(create_customer))
```

4. Add tests:

```rust
#[tokio::test]
async fn test_get_customer() {
    let app = create_test_app().await;
    let response = app.get("/customers/uuid-here").await;
    assert_eq!(response.status(), 200);
}
```

### 4.2 Adding a New Go Microservice

Each Go microservice follows a standard pattern:

```go
package main

import (
    "net/http"
    "os"
)

func main() {
    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    mux := http.NewServeMux()

    // Health check (required)
    mux.HandleFunc("/healthz", healthHandler)

    // CRUD endpoints
    base := "/v1/<entity-name>"
    mux.HandleFunc(base, listOrCreateHandler)
    mux.HandleFunc(base+"/", getUpdateDeleteHandler)

    http.ListenAndServe(":"+port, mux)
}
```

**Key requirements:**
- Always require `X-Tenant-ID` header
- Always include `event_topic` in responses
- Follow naming convention: `erp.bss_oss.<entity>.<action>`

---

## 5. Database Development

### 5.1 Adding a Migration

```bash
# Create a new migration
sqlx migrate add create_new_table

# Edit the generated file
vi migrations/YYYYMMDDHHMMSS_create_new_table.sql

# Run migrations
sqlx migrate run --database-url postgresql://postgres:postgres@localhost/bss
```

### 5.2 Compile-Time Query Checking

SQLx validates queries at compile time:

```rust
let customer = sqlx::query_as!(
    Customer,
    r#"
    SELECT id, name, status, customer_type, created_at
    FROM customers
    WHERE id = $1 AND deleted_at IS NULL
    "#,
    customer_id
)
.fetch_optional(&pool)
.await?;
```

---

## 6. Event Development

### 6.1 Publishing an Event

```rust
use rdkafka::producer::FutureProducer;

pub async fn publish_event(
    producer: &FutureProducer,
    topic: &str,
    key: &str,
    payload: &impl Serialize,
) -> Result<(), BssError> {
    let json = serde_json::to_string(payload)?;
    let record = FutureRecord::to(topic)
        .key(key)
        .payload(&json)
        .headers(OwnedHeaders::new()
            .insert(Header { key: "ce-specversion", value: Some("1.0") })
            .insert(Header { key: "ce-type", value: Some(topic) })
            .insert(Header { key: "ce-source", value: Some("/v1/billing-rating") })
        );
    producer.send(record, Duration::from_secs(5)).await?;
    Ok(())
}
```

### 6.2 Consuming Events

```rust
use rdkafka::consumer::StreamConsumer;

pub async fn consume_events(consumer: &StreamConsumer) {
    consumer.subscribe(&["erp.bss_oss.order-management.created"]).unwrap();
    loop {
        match consumer.recv().await {
            Ok(message) => {
                let payload = message.payload_view::<str>().unwrap().unwrap();
                let order: Order = serde_json::from_str(payload).unwrap();
                handle_new_order(order).await;
            }
            Err(e) => tracing::error!("Kafka error: {}", e),
        }
    }
}
```

---

## 7. Testing Guide

### 7.1 Unit Tests

```bash
cargo test --lib                  # Unit tests only
cargo test --lib -p bss-billing   # Single crate
```

### 7.2 Integration Tests

```bash
# Requires Docker Compose running
cargo test --test integration_tests
```

### 7.3 Load Testing

```bash
# Install k6
brew install k6

# Run load test
k6 run scripts/load-tests/billing-api.js
```

---

## 8. Debugging

### 8.1 Viewing Traces

1. Open Jaeger UI: `http://localhost:16686`
2. Select service (e.g., `bss-api`)
3. Search traces by operation or trace ID
4. View span waterfall for request flow

### 8.2 Viewing Metrics

1. Open Grafana: `http://localhost:3000`
2. Login: admin/admin
3. Navigate to pre-built dashboards

### 8.3 Log Format

All services emit structured JSON logs:

```json
{
    "timestamp": "2026-02-23T10:00:00.000Z",
    "level": "INFO",
    "target": "bss_api::handlers::customer",
    "message": "Customer created",
    "trace_id": "abc123",
    "span_id": "def456",
    "customer_id": "uuid",
    "tenant_id": "uuid"
}
```

---

## 9. Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Write code with tests
4. Run `cargo clippy` and `cargo fmt`
5. Submit a pull request
6. Pass CI checks (lint, test, build)
7. Get code review approval
8. Merge
