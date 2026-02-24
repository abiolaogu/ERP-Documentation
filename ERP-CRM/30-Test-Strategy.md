# ERP-CRM Test Strategy

## 1. Testing Philosophy

The ERP-CRM module follows a multi-layered testing strategy that aligns with the system's Domain-Driven Design (DDD) and Hexagonal Architecture. Testing is organized concentrically: domain logic is tested in pure isolation (no I/O), application services are tested against port interfaces, and integration tests verify real database and messaging behavior. The goal is fast, reliable feedback loops that catch regressions early and give high confidence in business rule correctness.

```mermaid
graph TB
    subgraph "Test Pyramid"
        E2E["End-to-End Tests<br/>~5% of total<br/>Full HTTP workflow"]
        INTEGRATION["Integration Tests<br/>~25% of total<br/>DB + NATS + HTTP"]
        UNIT["Unit Tests<br/>~70% of total<br/>Domain logic, value objects, services"]
    end

    E2E --> INTEGRATION --> UNIT

    subgraph "Coverage Targets"
        DOMAIN_COV["Domain Layer: >= 95%"]
        APP_COV["Application Layer: >= 85%"]
        INFRA_COV["Infrastructure Layer: >= 75%"]
        HTTP_COV["HTTP Handlers: >= 80%"]
    end
```

## 2. Test Categories

### 2.1 Unit Tests

Unit tests form the foundation of the test strategy. They run without any external dependencies (no database, no NATS, no network). The Rust compiler and type system already eliminate entire categories of bugs, so unit tests focus on business logic correctness.

```mermaid
flowchart TB
    subgraph "Unit Test Scope"
        VO["Value Objects<br/>Email, Money, Phone,<br/>Address, EntityId,<br/>LeadScore, Probability"]
        AGG["Aggregates<br/>Contact, Deal, Ticket"]
        SVC["Domain Services<br/>LeadScoringService,<br/>ForecastService,<br/>ContactMergeService"]
        ENUM["Enumerations<br/>LeadStatus, DealStatus,<br/>LifecycleStage, DealType,<br/>TicketStatus, Priority"]
        EVT["Domain Events<br/>ContactEvent, DealEvent,<br/>AccountEvent"]
    end

    VO --> |"Validation rules"| ASSERT1["Assert valid/invalid inputs"]
    AGG --> |"State transitions"| ASSERT2["Assert invariants hold"]
    SVC --> |"Calculations"| ASSERT3["Assert correct outputs"]
    ENUM --> |"Serialization"| ASSERT4["Assert roundtrip fidelity"]
    EVT --> |"Event emission"| ASSERT5["Assert correct events raised"]
```

**Existing Unit Tests (from source):**

| Module | Test | Purpose |
|--------|------|---------|
| `contact.rs` | `test_contact_creation` | Verifies factory method sets defaults (LeadStatus::New, LeadScore 0) |
| `contact.rs` | `test_contact_created_event` | Verifies ContactEvent::Created is raised on creation |
| `contact.rs` | `test_qualify_contact` | Verifies status transitions to Qualified, lifecycle to SQL |
| `contact.rs` | `test_cannot_qualify_twice` | Verifies AlreadyQualified error returned |
| `contact.rs` | `test_convert_to_customer` | Verifies lifecycle stage transition to Customer |
| `contact.rs` | `test_lead_score` | Verifies LeadScore value object (hot >= 80) |
| `contact.rs` | `test_tags` | Verifies add/remove tag operations, deduplication |
| `deal.rs` | `test_deal_creation` | Verifies factory method, initial DealStatus::Open |
| `deal.rs` | `test_move_stage` | Verifies stage transition, probability update, history recording |
| `deal.rs` | `test_close_won` | Verifies status=Won, probability=100, closed_at set |
| `deal.rs` | `test_close_lost` | Verifies status=Lost, probability=0, reason stored |
| `deal.rs` | `test_cannot_modify_closed_deal` | Verifies DealNotOpen error for closed deals |
| `deal.rs` | `test_weighted_value` | Verifies amount * probability / 100 calculation |
| `email.rs` | `test_valid_email` | Verifies Email::new accepts valid emails |
| `email.rs` | `test_invalid_email_no_at` | Verifies rejection of missing @ |
| `email.rs` | `test_invalid_email_no_domain` | Verifies rejection of missing domain |
| `email.rs` | `test_email_normalization` | Verifies lowercase normalization |
| `email.rs` | `test_email_display` | Verifies Display trait implementation |
| `email.rs` | `test_email_equality` | Verifies Eq based on normalized value |
| `money.rs` | `test_money_creation` | Verifies Money::usd factory |
| `money.rs` | `test_money_addition` | Verifies same-currency addition |
| `money.rs` | `test_money_currency_mismatch` | Verifies cross-currency rejection |
| `money.rs` | `test_zero_money` | Verifies Money::zero creates $0 |
| `money.rs` | `test_money_display` | Verifies formatted display |
| `services/mod.rs` | `test_lead_scoring` | Verifies score calculation with activity inputs |
| `services/mod.rs` | `test_weighted_pipeline` | Verifies ForecastService pipeline calculation |

### 2.2 Unit Test Patterns

**Contact Aggregate State Transition Tests:**

```rust
#[cfg(test)]
mod contact_state_machine_tests {
    use super::*;

    // Test the full lifecycle: New -> Contacted -> Qualified -> Converted
    #[test]
    fn test_full_lifecycle_progression() {
        let mut contact = create_test_contact();
        assert_eq!(contact.lead_status(), &LeadStatus::New);

        contact.qualify().unwrap();
        assert_eq!(contact.lead_status(), &LeadStatus::Qualified);
        assert_eq!(contact.lifecycle_stage(), &LifecycleStage::SalesQualifiedLead);

        contact.convert_to_customer().unwrap();
        assert_eq!(contact.lifecycle_stage(), &LifecycleStage::Customer);
        assert_eq!(contact.lead_status(), &LeadStatus::Converted);
    }

    // Test invalid transition: Unqualified -> Qualified should fail
    #[test]
    fn test_cannot_qualify_unqualified() {
        let mut contact = create_test_contact();
        contact.disqualify("No budget".to_string());

        assert!(matches!(
            contact.qualify(),
            Err(ContactError::CannotQualifyUnqualified)
        ));
    }

    // Test idempotent tag operations
    #[test]
    fn test_add_tag_idempotent() {
        let mut contact = create_test_contact();
        contact.add_tag("vip");
        contact.add_tag("vip"); // Duplicate
        assert_eq!(contact.tags().len(), 1);
    }

    // Test ownership transfer emits event
    #[test]
    fn test_ownership_transfer_event() {
        let mut contact = create_test_contact();
        contact.take_events(); // Clear creation event

        let new_owner = EntityId::new();
        contact.transfer_to(new_owner);

        let events = contact.take_events();
        assert_eq!(events.len(), 1);
        assert!(matches!(
            events[0],
            DomainEvent::Contact(ContactEvent::OwnershipTransferred { .. })
        ));
    }
}
```

**Deal Aggregate Business Rule Tests:**

```rust
#[cfg(test)]
mod deal_business_rule_tests {
    use super::*;

    // Test product-based amount recalculation
    #[test]
    fn test_deal_amount_recalculates_with_products() {
        let mut deal = create_test_deal();

        deal.add_product(DealProduct {
            product_id: EntityId::new(),
            name: "Enterprise License".into(),
            quantity: 10,
            unit_price: Decimal::new(1000, 0),
            discount_percent: Decimal::new(10, 0),
        });

        // 10 * $1000 - 10% = $9,000
        assert_eq!(deal.amount().amount(), Decimal::new(9000, 0));
    }

    // Test reopen after close_lost
    #[test]
    fn test_reopen_deal() {
        let mut deal = create_test_deal();
        deal.close_lost("Lost to competitor").unwrap();
        assert!(deal.is_lost());

        deal.reopen().unwrap();
        assert!(deal.is_open());
        assert_eq!(deal.probability().value(), 10); // Reset to initial
    }

    // Test currency mismatch prevention
    #[test]
    fn test_currency_mismatch_on_update() {
        let mut deal = create_test_deal(); // USD deal
        let eur_amount = Money::new(Decimal::new(50000, 0), Currency::EUR);

        assert!(matches!(
            deal.update_amount(eur_amount),
            Err(DealError::CurrencyMismatch)
        ));
    }
}
```

**Value Object Validation Tests:**

```rust
#[cfg(test)]
mod value_object_tests {
    use super::*;

    // LeadScore clamping
    #[test]
    fn test_lead_score_clamped_at_100() {
        let score = LeadScore::new(150);
        assert_eq!(score.value(), 100);
    }

    #[test]
    fn test_lead_score_categories() {
        assert!(LeadScore::new(85).is_hot());
        assert!(LeadScore::new(60).is_warm());
        assert!(LeadScore::new(30).is_cold());
    }

    // Probability clamping
    #[test]
    fn test_probability_clamped_at_100() {
        let prob = Probability::new(200);
        assert_eq!(prob.value(), 100);
    }

    // Email validation edge cases
    #[test]
    fn test_email_edge_cases() {
        assert!(Email::new("").is_err());
        assert!(Email::new("@domain.com").is_err());
        assert!(Email::new("user@").is_err());
        assert!(Email::new("user@domain").is_err());
        assert!(Email::new("user@domain.com").is_ok());
        assert!(Email::new("USER@Domain.COM").unwrap().as_str() == "user@domain.com");
    }
}
```

### 2.3 Integration Tests

Integration tests verify that the application correctly interacts with external systems. They require a running PostgreSQL instance (provided by Docker or CI service containers).

```mermaid
flowchart TB
    subgraph "Integration Test Architecture"
        TEST_RUNNER["Test Runner<br/>(cargo test --features integration)"]

        subgraph "Test Fixtures"
            SETUP["Database Setup<br/>Run migrations"]
            SEED["Seed Data<br/>Pipelines, stages"]
            CLEANUP["Cleanup<br/>Truncate tables"]
        end

        subgraph "Systems Under Test"
            DB["PostgreSQL 16<br/>(Test Database)"]
            NATS_TEST["NATS<br/>(Optional)"]
            HTTP_TEST["HTTP Server<br/>(Test Instance)"]
        end
    end

    TEST_RUNNER --> SETUP --> SEED --> CLEANUP
    SETUP --> DB
    TEST_RUNNER --> HTTP_TEST --> DB
    TEST_RUNNER -.-> NATS_TEST
```

**Database Integration Tests:**

```rust
#[cfg(test)]
#[cfg(feature = "integration")]
mod integration_tests {
    use sqlx::PgPool;

    async fn setup_test_db() -> PgPool {
        let url = std::env::var("DATABASE_URL")
            .unwrap_or_else(|_| "postgres://postgres:postgres@localhost:5432/crm_test".into());
        let pool = PgPool::connect(&url).await.unwrap();
        sqlx::migrate!("./migrations").run(&pool).await.unwrap();
        pool
    }

    #[tokio::test]
    async fn test_contact_crud_cycle() {
        let pool = setup_test_db().await;

        // CREATE
        let id = Uuid::now_v7();
        sqlx::query!(
            "INSERT INTO contacts (id, email, lifecycle_stage, lead_score, tags, custom_fields)
             VALUES ($1, $2, 'subscriber', 0, '{}', '{}')",
            id, "integration@test.com"
        ).execute(&pool).await.unwrap();

        // READ
        let contact = sqlx::query_as!(Contact,
            "SELECT * FROM contacts WHERE id = $1", id
        ).fetch_one(&pool).await.unwrap();
        assert_eq!(contact.email, "integration@test.com");

        // UPDATE
        sqlx::query!("UPDATE contacts SET lead_score = 75 WHERE id = $1", id)
            .execute(&pool).await.unwrap();

        // DELETE
        sqlx::query!("DELETE FROM contacts WHERE id = $1", id)
            .execute(&pool).await.unwrap();
    }

    #[tokio::test]
    async fn test_cascade_delete_contact() {
        let pool = setup_test_db().await;
        let contact_id = Uuid::now_v7();

        // Insert contact
        sqlx::query!(
            "INSERT INTO contacts (id, email, lifecycle_stage, lead_score, tags, custom_fields)
             VALUES ($1, 'cascade@test.com', 'subscriber', 0, '{}', '{}')",
            contact_id
        ).execute(&pool).await.unwrap();

        // Insert activity linked to contact
        let activity_id = Uuid::now_v7();
        sqlx::query!(
            "INSERT INTO activities (id, activity_type, subject, contact_id)
             VALUES ($1, 'call', 'Test call', $2)",
            activity_id, contact_id
        ).execute(&pool).await.unwrap();

        // Delete contact - activities should cascade
        sqlx::query!("DELETE FROM contacts WHERE id = $1", contact_id)
            .execute(&pool).await.unwrap();

        // Verify cascade
        let count = sqlx::query_scalar!(
            "SELECT COUNT(*) FROM activities WHERE id = $1", activity_id
        ).fetch_one(&pool).await.unwrap();
        assert_eq!(count, Some(0));
    }
}
```

### 2.4 HTTP API Tests

API tests verify the HTTP layer including request parsing, validation, response serialization, and error handling.

```mermaid
flowchart TB
    subgraph "API Test Matrix"
        subgraph "Contact Endpoints"
            C_CREATE["POST /api/v1/contacts<br/>201 Created<br/>400 Validation Error<br/>409 Duplicate Email"]
            C_LIST["GET /api/v1/contacts<br/>200 OK<br/>Pagination params"]
            C_GET["GET /api/v1/contacts/:id<br/>200 OK<br/>404 Not Found"]
            C_UPDATE["PUT /api/v1/contacts/:id<br/>200 OK<br/>404 Not Found"]
            C_DELETE["DELETE /api/v1/contacts/:id<br/>204 No Content<br/>404 Not Found"]
        end

        subgraph "Deal Endpoints"
            D_CREATE["POST /api/v1/deals<br/>201 Created<br/>400 Validation Error"]
            D_LIST["GET /api/v1/deals<br/>200 OK<br/>Pipeline filter"]
            D_UPDATE["PUT /api/v1/deals/:id<br/>200 OK"]
            D_DELETE["DELETE /api/v1/deals/:id<br/>204 No Content"]
        end

        subgraph "Infrastructure"
            HEALTH["GET /health<br/>200 OK (always)"]
            READY["GET /ready<br/>200 OK / 503 DB down"]
            METRICS["GET /metrics<br/>200 OK Prometheus"]
        end
    end
```

**HTTP Handler Test Examples:**

```rust
#[cfg(test)]
mod api_tests {
    use axum::http::StatusCode;
    use axum_test::TestServer;

    async fn test_server() -> TestServer {
        let state = create_test_state().await;
        let app = create_router(state);
        TestServer::new(app).unwrap()
    }

    #[tokio::test]
    async fn test_create_contact_valid() {
        let server = test_server().await;
        let response = server.post("/api/v1/contacts")
            .json(&serde_json::json!({
                "email": "test@example.com",
                "first_name": "Jane",
                "last_name": "Doe"
            }))
            .await;

        assert_eq!(response.status_code(), StatusCode::CREATED);
        let body: serde_json::Value = response.json();
        assert!(body["id"].is_string());
        assert_eq!(body["email"], "test@example.com");
    }

    #[tokio::test]
    async fn test_create_contact_invalid_email() {
        let server = test_server().await;
        let response = server.post("/api/v1/contacts")
            .json(&serde_json::json!({
                "email": "not-an-email"
            }))
            .await;

        assert_eq!(response.status_code(), StatusCode::BAD_REQUEST);
    }

    #[tokio::test]
    async fn test_get_contact_not_found() {
        let server = test_server().await;
        let response = server
            .get(&format!("/api/v1/contacts/{}", Uuid::new_v4()))
            .await;

        assert_eq!(response.status_code(), StatusCode::NOT_FOUND);
    }

    #[tokio::test]
    async fn test_list_contacts_pagination() {
        let server = test_server().await;
        let response = server
            .get("/api/v1/contacts?limit=10&offset=0")
            .await;

        assert_eq!(response.status_code(), StatusCode::OK);
        let body: serde_json::Value = response.json();
        assert!(body["data"].is_array());
        assert!(body["total"].is_number());
    }

    #[tokio::test]
    async fn test_health_endpoint() {
        let server = test_server().await;
        let response = server.get("/health").await;
        assert_eq!(response.status_code(), StatusCode::OK);
    }

    #[tokio::test]
    async fn test_ready_endpoint() {
        let server = test_server().await;
        let response = server.get("/ready").await;
        assert_eq!(response.status_code(), StatusCode::OK);
    }
}
```

### 2.5 Go Microservice Tests

Each of the 12 Go microservices follows an identical CRUD pattern and testing strategy.

```mermaid
flowchart TB
    subgraph "Go Test Matrix (per service)"
        HEALTH_TEST["Test /healthz<br/>200 OK"]
        CREATE_TEST["Test POST /v1/{entity}<br/>201 Created + validation"]
        READ_TEST["Test GET /v1/{entity}/{id}<br/>200 OK + 404"]
        UPDATE_TEST["Test PUT /v1/{entity}/{id}<br/>200 OK + 404"]
        DELETE_TEST["Test DELETE /v1/{entity}/{id}<br/>204 + 404"]
        LIST_TEST["Test GET /v1/{entity}<br/>200 OK + pagination"]
        TENANT_TEST["Test X-Tenant-ID<br/>400 if missing"]
    end
```

**Go Service Test Example:**

```go
// contact-service/main_test.go
package main

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestHealthz(t *testing.T) {
    req := httptest.NewRequest("GET", "/healthz", nil)
    w := httptest.NewRecorder()

    mux := setupRouter()
    mux.ServeHTTP(w, req)

    if w.Code != http.StatusOK {
        t.Errorf("expected 200, got %d", w.Code)
    }
}

func TestCreateContactRequiresTenantID(t *testing.T) {
    body := map[string]string{"email": "test@example.com"}
    jsonBody, _ := json.Marshal(body)

    req := httptest.NewRequest("POST", "/v1/contact", bytes.NewBuffer(jsonBody))
    req.Header.Set("Content-Type", "application/json")
    // No X-Tenant-ID header

    w := httptest.NewRecorder()
    mux := setupRouter()
    mux.ServeHTTP(w, req)

    if w.Code != http.StatusBadRequest {
        t.Errorf("expected 400 without tenant ID, got %d", w.Code)
    }
}

func TestCreateContactValid(t *testing.T) {
    body := map[string]string{"email": "test@example.com", "first_name": "John"}
    jsonBody, _ := json.Marshal(body)

    req := httptest.NewRequest("POST", "/v1/contact", bytes.NewBuffer(jsonBody))
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("X-Tenant-ID", "tenant-001")

    w := httptest.NewRecorder()
    mux := setupRouter()
    mux.ServeHTTP(w, req)

    if w.Code != http.StatusCreated {
        t.Errorf("expected 201, got %d", w.Code)
    }
}
```

### 2.6 End-to-End Tests

End-to-end tests verify complete workflows across the full stack.

```mermaid
flowchart LR
    subgraph "E2E Scenarios"
        S1["Lead-to-Close<br/>Create contact -> Qualify<br/>-> Create deal -> Win"]
        S2["Ticket Lifecycle<br/>Create ticket -> Assign<br/>-> Comment -> Solve"]
        S3["Form Submission<br/>Submit form -> Create<br/>contact -> Score lead"]
        S4["Ownership Transfer<br/>Transfer contact<br/>-> Verify events"]
    end

    subgraph "Verification"
        DB_CHECK["Database State"]
        EVENT_CHECK["Event Published"]
        API_CHECK["API Response"]
    end

    S1 --> DB_CHECK & EVENT_CHECK & API_CHECK
    S2 --> DB_CHECK & EVENT_CHECK & API_CHECK
    S3 --> DB_CHECK & EVENT_CHECK & API_CHECK
    S4 --> DB_CHECK & EVENT_CHECK & API_CHECK
```

**E2E Test: Lead-to-Close Workflow:**

```rust
#[cfg(test)]
#[cfg(feature = "e2e")]
mod e2e_tests {
    #[tokio::test]
    async fn test_lead_to_close_workflow() {
        let client = reqwest::Client::new();
        let base_url = "http://localhost:8081";

        // 1. Create a contact (lead)
        let contact_resp = client.post(&format!("{}/api/v1/contacts", base_url))
            .json(&serde_json::json!({
                "email": "lead@enterprise.com",
                "first_name": "Enterprise",
                "last_name": "Buyer"
            }))
            .send().await.unwrap();
        assert_eq!(contact_resp.status(), 201);
        let contact: serde_json::Value = contact_resp.json().await.unwrap();
        let contact_id = contact["id"].as_str().unwrap();

        // 2. Update lead score (qualify)
        let update_resp = client.put(&format!("{}/api/v1/contacts/{}", base_url, contact_id))
            .json(&serde_json::json!({
                "lead_score": 85,
                "lifecycle_stage": "sql"
            }))
            .send().await.unwrap();
        assert_eq!(update_resp.status(), 200);

        // 3. Create a deal linked to this contact
        let deal_resp = client.post(&format!("{}/api/v1/deals", base_url))
            .json(&serde_json::json!({
                "name": "Enterprise License Deal",
                "contact_id": contact_id,
                "pipeline_id": "00000000-0000-0000-0000-000000000001",
                "stage_id": "00000000-0000-0000-0000-000000000011",
                "amount": 10000000,
                "currency": "USD"
            }))
            .send().await.unwrap();
        assert_eq!(deal_resp.status(), 201);
        let deal: serde_json::Value = deal_resp.json().await.unwrap();
        let deal_id = deal["id"].as_str().unwrap();

        // 4. Progress through stages
        let stage_resp = client.put(&format!("{}/api/v1/deals/{}", base_url, deal_id))
            .json(&serde_json::json!({
                "stage_id": "00000000-0000-0000-0000-000000000015",
                "probability": 100
            }))
            .send().await.unwrap();
        assert_eq!(stage_resp.status(), 200);

        // 5. Verify deal is in Closed Won stage
        let verify_resp = client.get(&format!("{}/api/v1/deals/{}", base_url, deal_id))
            .send().await.unwrap();
        let final_deal: serde_json::Value = verify_resp.json().await.unwrap();
        assert_eq!(final_deal["stage_id"], "00000000-0000-0000-0000-000000000015");

        // 6. Clean up
        client.delete(&format!("{}/api/v1/deals/{}", base_url, deal_id)).send().await.unwrap();
        client.delete(&format!("{}/api/v1/contacts/{}", base_url, contact_id)).send().await.unwrap();
    }
}
```

## 3. Domain Service Test Coverage

### 3.1 LeadScoringService Tests

```mermaid
flowchart TB
    subgraph "Lead Scoring Test Cases"
        T1["CEO title<br/>score += 25"]
        T2["VP title<br/>score += 20"]
        T3["Manager title<br/>score += 15"]
        T4["No title<br/>score += 0"]
        T5["Has account<br/>score += 10"]
        T6["10 activities<br/>score += 20"]
        T7["20 email opens<br/>score += 20"]
        T8["20 page views<br/>score += 10"]
        T9["Max score<br/>capped at 100"]
    end
```

```rust
#[cfg(test)]
mod lead_scoring_tests {
    use super::*;

    #[test]
    fn test_ceo_gets_highest_demographic_score() {
        let mut contact = create_test_contact();
        contact.update_info(None, None, Some("CEO".to_string()), None);

        let score = LeadScoringService::calculate_score(&contact, 0, 0, 0);
        assert_eq!(score, 25);
    }

    #[test]
    fn test_max_engagement_score() {
        let contact = create_test_contact();
        // Max activities (10*2=20) + max opens (20) + max views (20/2=10) = 50
        let score = LeadScoringService::calculate_score(&contact, 100, 100, 100);
        assert_eq!(score, 50); // Capped engagement scores
    }

    #[test]
    fn test_score_never_exceeds_100() {
        let mut contact = create_test_contact();
        contact.update_info(None, None, Some("CEO".to_string()), None);
        contact.link_to_account(EntityId::new());
        contact.update_lead_score(85); // Makes is_hot() true

        let score = LeadScoringService::calculate_score(&contact, 100, 100, 100);
        assert!(score <= 100);
    }
}
```

### 3.2 ForecastService Tests

```rust
#[cfg(test)]
mod forecast_tests {
    use super::*;

    fn create_deals_portfolio() -> Vec<Deal> {
        let mut deals = vec![];

        // Open deal at 50% probability, $100K
        let mut deal1 = Deal::create("Deal A", Money::usd(dec!(100000)),
            EntityId::new(), EntityId::new(), EntityId::new());
        deal1.move_to_stage(EntityId::new(), 50).unwrap();
        deals.push(deal1);

        // Won deal, $75K
        let mut deal2 = Deal::create("Deal B", Money::usd(dec!(75000)),
            EntityId::new(), EntityId::new(), EntityId::new());
        deal2.close_won().unwrap();
        deals.push(deal2);

        // Lost deal, $50K (should be excluded from pipeline)
        let mut deal3 = Deal::create("Deal C", Money::usd(dec!(50000)),
            EntityId::new(), EntityId::new(), EntityId::new());
        deal3.close_lost("Budget").unwrap();
        deals.push(deal3);

        deals
    }

    #[test]
    fn test_weighted_pipeline_excludes_closed() {
        let deals = create_deals_portfolio();
        let weighted = ForecastService::calculate_weighted_pipeline(&deals);
        // Only Deal A: $100K * 50% = $50K
        assert_eq!(weighted, dec!(50000));
    }

    #[test]
    fn test_total_pipeline_open_only() {
        let deals = create_deals_portfolio();
        let total = ForecastService::calculate_total_pipeline(&deals);
        assert_eq!(total, dec!(100000)); // Only Deal A
    }

    #[test]
    fn test_closed_won_value() {
        let deals = create_deals_portfolio();
        let closed_won = ForecastService::calculate_closed_won(&deals);
        assert_eq!(closed_won, dec!(75000)); // Only Deal B
    }

    #[test]
    fn test_at_risk_deals_identifies_stale() {
        let deals = create_deals_portfolio();
        let at_risk = ForecastService::identify_at_risk_deals(&deals, 30);
        // New deals just created, days_in_stage = 0, not at risk
        assert_eq!(at_risk.len(), 0);
    }
}
```

## 4. CI Pipeline Quality Gates

```mermaid
flowchart LR
    subgraph "CI Test Pipeline"
        FMT["cargo fmt<br/>-- --check"]
        CLIPPY["cargo clippy<br/>-- -D warnings"]
        UNIT_CI["cargo test<br/>--all-features"]
        BUILD_CI["cargo build<br/>--release"]
    end

    FMT -->|Pass| CLIPPY -->|Pass| UNIT_CI -->|Pass| BUILD_CI
    FMT -->|Fail| BLOCK1["Block Merge"]
    CLIPPY -->|Fail| BLOCK2["Block Merge"]
    UNIT_CI -->|Fail| BLOCK3["Block Merge"]
```

### 4.1 CI Configuration (GitHub Actions)

The CI pipeline runs on every push to `main`/`develop` and every PR targeting `main`:

| Step | Command | Failure Action |
|------|---------|---------------|
| Format Check | `cargo fmt -- --check` | Block merge |
| Lint Check | `cargo clippy -- -D warnings` | Block merge |
| Unit Tests | `cargo test --all-features` | Block merge |
| Integration Tests | `cargo test --features integration` | Block merge (requires PostgreSQL service) |
| Release Build | `cargo build --release` | Block merge |
| Docker Build | `docker/build-push-action@v5` | Block deploy (main only) |

### 4.2 Test Environment Configuration

```yaml
# CI Service Containers
services:
  postgres:
    image: postgres:16
    env:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: crm_test
    ports:
      - 5432:5432
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5

# Test Environment Variables
env:
  DATABASE_URL: postgres://postgres:postgres@localhost:5432/crm_test
  CARGO_TERM_COLOR: always
  RUST_BACKTRACE: 1
```

## 5. Test Data Management

### 5.1 Seed Data

The system uses deterministic seed data for pipeline stages, ensuring consistent test environments.

| Entity | ID Pattern | Purpose |
|--------|-----------|---------|
| Default Pipeline | `00000000-...-000000000001` | Consistent pipeline reference |
| Lead Stage | `00000000-...-000000000011` | Initial deal stage |
| Qualified Stage | `00000000-...-000000000012` | Qualified deals |
| Proposal Stage | `00000000-...-000000000013` | Proposal phase |
| Negotiation Stage | `00000000-...-000000000014` | Negotiation phase |
| Closed Won | `00000000-...-000000000015` | Won deals |
| Closed Lost | `00000000-...-000000000016` | Lost deals |

### 5.2 Test Data Factories

```rust
/// Test data factory module
#[cfg(test)]
pub mod test_factories {
    use super::*;

    pub fn contact(email: &str) -> Contact {
        Contact::create(
            Email::new(email).unwrap(),
            "Test",
            "User",
            EntityId::new(),
        )
    }

    pub fn deal(name: &str, amount: i64) -> Deal {
        Deal::create(
            name,
            Money::usd(Decimal::new(amount, 0)),
            EntityId::new(),
            EntityId::new(),
            EntityId::new(),
        )
    }

    pub fn hot_contact(email: &str) -> Contact {
        let mut c = contact(email);
        c.update_lead_score(85);
        c.qualify().unwrap();
        c
    }
}
```

## 6. Performance Testing

```mermaid
flowchart TB
    subgraph "Performance Test Suite"
        LOAD["Load Testing<br/>(k6 / drill)"]
        STRESS["Stress Testing<br/>(Sustained high load)"]
        SPIKE["Spike Testing<br/>(Sudden bursts)"]
        SOAK["Soak Testing<br/>(Extended duration)"]
    end

    subgraph "Benchmarks"
        B1["Contact CRUD: < 50ms p99"]
        B2["Deal CRUD: < 50ms p99"]
        B3["List with pagination: < 100ms p99"]
        B4["Search/filter: < 200ms p99"]
        B5["Event publish: < 10ms p99"]
    end

    LOAD --> B1 & B2 & B3 & B4 & B5
```

### 6.1 Load Test Script (k6)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
    stages: [
        { duration: '2m', target: 50 },    // Ramp up
        { duration: '5m', target: 50 },    // Sustain
        { duration: '2m', target: 100 },   // Peak
        { duration: '5m', target: 100 },   // Sustain peak
        { duration: '2m', target: 0 },     // Ramp down
    ],
    thresholds: {
        http_req_duration: ['p(99)<200'],   // 99th percentile < 200ms
        http_req_failed: ['rate<0.01'],     // < 1% error rate
    },
};

export default function () {
    // Create contact
    const createResp = http.post('http://localhost:8081/api/v1/contacts', JSON.stringify({
        email: `load-test-${__VU}-${__ITER}@example.com`,
        first_name: 'Load',
        last_name: 'Test',
    }), { headers: { 'Content-Type': 'application/json' } });

    check(createResp, { 'create: status 201': (r) => r.status === 201 });

    if (createResp.status === 201) {
        const contactId = JSON.parse(createResp.body).id;

        // Read contact
        const getResp = http.get(`http://localhost:8081/api/v1/contacts/${contactId}`);
        check(getResp, { 'get: status 200': (r) => r.status === 200 });

        // List contacts
        const listResp = http.get('http://localhost:8081/api/v1/contacts?limit=20&offset=0');
        check(listResp, { 'list: status 200': (r) => r.status === 200 });

        // Delete contact
        const delResp = http.del(`http://localhost:8081/api/v1/contacts/${contactId}`);
        check(delResp, { 'delete: status 204': (r) => r.status === 204 });
    }

    sleep(0.5);
}
```

### 6.2 Performance Targets

| Endpoint | p50 | p95 | p99 | Max Concurrent Users |
|----------|-----|-----|-----|---------------------|
| `POST /api/v1/contacts` | < 10ms | < 30ms | < 50ms | 100 |
| `GET /api/v1/contacts/:id` | < 5ms | < 15ms | < 30ms | 200 |
| `GET /api/v1/contacts` (list) | < 20ms | < 50ms | < 100ms | 100 |
| `PUT /api/v1/contacts/:id` | < 10ms | < 30ms | < 50ms | 100 |
| `DELETE /api/v1/contacts/:id` | < 10ms | < 25ms | < 50ms | 100 |
| `POST /api/v1/deals` | < 15ms | < 40ms | < 60ms | 100 |
| `GET /health` | < 1ms | < 3ms | < 5ms | 500 |
| `GET /ready` | < 5ms | < 15ms | < 30ms | 500 |

## 7. Security Testing

```mermaid
flowchart TB
    subgraph "Security Test Areas"
        SQL_INJ["SQL Injection<br/>Prevention<br/>(compile-time sqlx)"]
        INPUT_VAL["Input Validation<br/>(validator crate)"]
        AUTH_TEST["Authentication<br/>JWT verification"]
        TENANT_ISO["Tenant Isolation<br/>X-Tenant-ID enforcement"]
        CORS_TEST["CORS Policy<br/>Verification"]
    end

    subgraph "Tools"
        CLIPPY_SEC["cargo clippy<br/>security lints"]
        AUDIT["cargo audit<br/>vulnerability scan"]
        DENY["cargo deny<br/>license + advisory"]
    end

    SQL_INJ --> CLIPPY_SEC
    INPUT_VAL --> CLIPPY_SEC
    AUTH_TEST --> AUDIT
    TENANT_ISO --> AUDIT
```

### 7.1 Security Test Cases

| Test | Category | Expected Behavior |
|------|----------|-------------------|
| SQL injection in email field | Input validation | Rejected by Email value object validation |
| SQL injection in query params | Input validation | Parameterized queries prevent injection |
| Missing X-Tenant-ID | Tenant isolation | 400 Bad Request |
| Cross-tenant data access | Tenant isolation | 403 Forbidden, no data returned |
| Oversized request body | DoS prevention | 413 Payload Too Large |
| Invalid UUID in path | Input validation | 400 Bad Request |
| CORS preflight | Security headers | Correct Access-Control-Allow headers |
| JWT with expired token | Authentication | 401 Unauthorized |
| JWT with wrong signing key | Authentication | 401 Unauthorized |

## 8. Test Execution Commands

```bash
# Run all unit tests
cargo test --all-features

# Run unit tests with output
cargo test --all-features -- --nocapture

# Run specific test module
cargo test domain::aggregates::contact::tests

# Run integration tests (requires PostgreSQL)
DATABASE_URL=postgres://postgres:postgres@localhost:5432/crm_test \
  cargo test --features integration

# Run with coverage (via cargo-tarpaulin)
cargo tarpaulin --all-features --out Html

# Run format check
cargo fmt -- --check

# Run clippy lints
cargo clippy -- -D warnings

# Run security audit
cargo audit

# Run Go microservice tests
cd services/contact-service && go test -v ./...
cd services/helpdesk-service && go test -v ./...

# Run load tests (k6)
k6 run tests/load/contacts.js

# Run E2E tests (requires running services)
cargo test --features e2e
```

## 9. Test Reporting

```mermaid
flowchart TB
    subgraph "Test Artifacts"
        RESULTS["Test Results<br/>(JUnit XML)"]
        COVERAGE["Coverage Report<br/>(HTML + Cobertura)"]
        PERF_REPORT["Performance Report<br/>(k6 JSON)"]
    end

    subgraph "CI Integration"
        GH_CHECK["GitHub Check<br/>Pass/Fail"]
        GH_COMMENT["PR Comment<br/>Coverage delta"]
        GH_ARTIFACT["Build Artifact<br/>Reports upload"]
    end

    RESULTS --> GH_CHECK
    COVERAGE --> GH_COMMENT
    PERF_REPORT --> GH_ARTIFACT
```

### 9.1 Coverage Thresholds

| Layer | Current Target | Enforcement |
|-------|---------------|-------------|
| Domain Aggregates | >= 95% | CI fails below threshold |
| Domain Value Objects | >= 95% | CI fails below threshold |
| Domain Services | >= 90% | CI fails below threshold |
| Domain Events | >= 85% | CI fails below threshold |
| HTTP Handlers | >= 80% | CI warns below threshold |
| Repository Layer | >= 75% | CI warns below threshold |
| Configuration | >= 60% | Informational only |
| Overall | >= 80% | CI fails below threshold |

## 10. Test Maintenance

### 10.1 Test Review Checklist

- Each new domain method must have at least one unit test for the happy path and one for error cases
- Each new endpoint must have API tests for all response codes
- Value objects must have boundary/edge case tests
- Domain events must verify correct event type and payload
- Integration tests must clean up after themselves (no test data leakage)
- Performance tests must be updated when SLA targets change

### 10.2 Flaky Test Policy

1. Flaky tests are flagged immediately via CI annotation
2. Flaky tests must be fixed within 48 hours or quarantined
3. Quarantined tests run in a separate non-blocking job
4. Root cause analysis required before un-quarantining
5. Time-dependent tests must use `chrono::Utc::now()` mocking via a clock trait
