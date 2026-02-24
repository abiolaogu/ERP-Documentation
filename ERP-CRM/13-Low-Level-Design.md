# ERP-CRM Low-Level Design

## 1. Module Structure

### 1.1 Rust Core File Layout

```
src/
  main.rs              -- Entry point, router, all HTTP handlers, domain models
  lib.rs               -- Module registry, re-exports, legacy stubs
  config.rs            -- Environment-based configuration loading
  domain/
    mod.rs             -- Domain module root
    aggregates/
      mod.rs           -- Aggregate module root
      contact.rs       -- Contact aggregate (345 lines): lifecycle, scoring, tags, merge
      deal.rs          -- Deal aggregate (478 lines): stages, products, competitors
    value_objects/
      mod.rs           -- Value object module root, EntityId definition
      email.rs         -- Email value object with validation
      money.rs         -- Money value object with currency arithmetic
      phone.rs         -- Phone value object with formatting
      address.rs       -- Address value object with country codes
    events/
      mod.rs           -- DomainEvent, ContactEvent, DealEvent, AccountEvent enums
    services/
      mod.rs           -- LeadScoringService, ForecastService, ContactMergeService
  application/
    mod.rs             -- Application layer root
    commands/
      mod.rs           -- ContactService, DealService (use case orchestration)
    queries/
      mod.rs           -- Read model queries
    dto/
      mod.rs           -- DTOs: CreateContactCommand, PipelineView, ForecastView
  ports/
    mod.rs             -- Ports layer root
    inbound/
      mod.rs           -- ContactUseCases, DealUseCases traits
    outbound/
      mod.rs           -- ContactRepository, DealRepository, EventPublisher traits
  infrastructure/
    mod.rs             -- Infrastructure layer root
    persistence/
      mod.rs           -- PostgreSQL repository implementations
```

### 1.2 Go Microservice File Layout (per service)

```
services/{name}-service/
  main.go              -- HTTP server, handlers, tenant validation
  Dockerfile           -- Multi-stage build
  README.md            -- Service documentation
```

## 2. Detailed Class/Struct Design

### 2.1 Contact Aggregate

```mermaid
classDiagram
    class Contact {
        -EntityId id
        -Email email
        -String first_name
        -String last_name
        -Option~Phone~ phone
        -Option~Phone~ mobile
        -Option~String~ title
        -Option~String~ department
        -Option~Address~ address
        -Option~EntityId~ account_id
        -EntityId owner_id
        -LeadStatus lead_status
        -LeadScore lead_score
        -LifecycleStage lifecycle_stage
        -Vec~String~ tags
        -HashMap custom_fields
        -Option~DateTime~ last_activity_at
        -DateTime created_at
        -DateTime updated_at
        -Vec~DomainEvent~ events
        +create(email, first_name, last_name, owner_id) Contact
        +update_info(first_name, last_name, title, dept)
        +set_phone(phone)
        +set_mobile(mobile)
        +set_address(address)
        +link_to_account(account_id)
        +qualify() Result~ContactError~
        +disqualify(reason)
        +convert_to_customer() Result~ContactError~
        +update_lead_score(score)
        +add_tag(tag)
        +remove_tag(tag)
        +record_activity()
        +transfer_to(new_owner_id)
        +take_events() Vec~DomainEvent~
    }

    class LeadStatus {
        <<enum>>
        New
        Contacted
        Qualified
        Unqualified
        Nurturing
        Converted
    }

    class LifecycleStage {
        <<enum>>
        Subscriber
        Lead
        MarketingQualifiedLead
        SalesQualifiedLead
        Opportunity
        Customer
        Evangelist
    }

    class LeadScore {
        -u8 value
        +new(score) LeadScore
        +value() u8
        +is_hot() bool
        +is_warm() bool
        +is_cold() bool
    }

    Contact --> LeadStatus
    Contact --> LifecycleStage
    Contact --> LeadScore
```

### 2.2 Deal Aggregate

```mermaid
classDiagram
    class Deal {
        -EntityId id
        -String name
        -Money amount
        -EntityId pipeline_id
        -EntityId stage_id
        -Probability probability
        -Option~NaiveDate~ expected_close_date
        -Option~NaiveDate~ actual_close_date
        -Option~EntityId~ contact_id
        -Option~EntityId~ account_id
        -EntityId owner_id
        -DealType deal_type
        -DealStatus status
        -Option~String~ lost_reason
        -Option~String~ next_step
        -Vec~Competitor~ competitors
        -Vec~DealProduct~ products
        -Vec~StageChange~ stage_history
        -Vec~DomainEvent~ events
        +create(name, amount, pipeline_id, stage_id, owner_id) Deal
        +move_to_stage(stage_id, probability) Result~DealError~
        +update_amount(new_amount) Result~DealError~
        +close_won() Result~DealError~
        +close_lost(reason) Result~DealError~
        +reopen() Result~DealError~
        +add_product(product)
        +remove_product(product_id)
        +link_contact(contact_id)
        +link_account(account_id)
        +weighted_value() Decimal
        +days_in_stage() i64
    }

    class DealStatus {
        <<enum>>
        Open
        Won
        Lost
    }

    class DealType {
        <<enum>>
        NewBusiness
        Renewal
        Upsell
        CrossSell
    }

    class Probability {
        -u8 value
        +new(value) Probability
        +value() u8
    }

    class StageChange {
        +EntityId from_stage
        +EntityId to_stage
        +DateTime changed_at
    }

    class DealProduct {
        +EntityId product_id
        +String name
        +u32 quantity
        +Decimal unit_price
        +Decimal discount_percent
        +total() Decimal
    }

    class Competitor {
        +String name
        +Vec~String~ strengths
        +Vec~String~ weaknesses
    }

    Deal --> DealStatus
    Deal --> DealType
    Deal --> Probability
    Deal *-- StageChange
    Deal *-- DealProduct
    Deal *-- Competitor
```

### 2.3 Value Objects

```mermaid
classDiagram
    class Email {
        -String value
        +new(value) Result~EmailError~
        +new_unchecked(value) Email
        +as_str() str
        +domain() Option~str~
        +local_part() Option~str~
    }

    class Money {
        -Decimal amount
        -Currency currency
        +new(amount, currency) Money
        +from_cents(cents, currency) Money
        +zero(currency) Money
        +usd(amount) Money
        +add(other) Result~MoneyError~
        +subtract(other) Result~MoneyError~
        +multiply(factor) Money
        +is_positive() bool
        +is_negative() bool
        +is_zero() bool
        +abs() Money
        +same_currency(other) bool
    }

    class Currency {
        <<enum>>
        USD
        EUR
        GBP
        CAD
        AUD
        NGN
        JPY
        CNY
        INR
        Other(String)
        +code() str
        +from_code(code) Currency
    }

    class EntityId {
        -String value
        +new() EntityId
        +from_string(id) EntityId
        +as_str() str
    }

    Money --> Currency
```

## 3. API Handler Design

### 3.1 Handler Function Signatures

```rust
// Health
async fn health_check() -> impl IntoResponse

// Readiness (database check)
async fn readiness_check(State(state): State<AppState>) -> impl IntoResponse

// Contact CRUD
async fn list_contacts(
    State(state): State<AppState>,
    Query(params): Query<ListParams>,
) -> Result<Json<PaginatedResponse<Contact>>, (StatusCode, String)>

async fn get_contact(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
) -> Result<Json<Contact>, (StatusCode, String)>

async fn create_contact(
    State(state): State<AppState>,
    Json(req): Json<CreateContactRequest>,
) -> Result<(StatusCode, Json<Contact>), (StatusCode, String)>

async fn update_contact(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
    Json(req): Json<UpdateContactRequest>,
) -> Result<Json<Contact>, (StatusCode, String)>

async fn delete_contact(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
) -> Result<StatusCode, (StatusCode, String)>
```

### 3.2 Request/Response Flow

```mermaid
sequenceDiagram
    participant Client
    participant Middleware as TraceLayer + CorsLayer
    participant Router as axum::Router
    participant Handler as Handler Function
    participant Validator as validator::Validate
    participant DB as sqlx::PgPool
    participant NATS as async_nats::Client

    Client->>Middleware: HTTP Request
    Middleware->>Router: Route matching
    Router->>Handler: Extract State, Path, Query, Json
    Handler->>Validator: Validate request DTO
    alt Validation Fails
        Validator-->>Handler: Err(ValidationErrors)
        Handler-->>Client: 400 Bad Request
    else Validation Passes
        Handler->>DB: sqlx::query_as (compile-time checked)
        DB-->>Handler: Result<T>
        alt DB Error
            Handler-->>Client: 500 Internal Server Error
        else Success
            Handler->>NATS: Publish event (optional)
            Handler-->>Client: 200/201 + JSON body
        end
    end
```

## 4. Database Access Patterns

### 4.1 Pagination Implementation

```rust
// Standard pagination pattern used across all list handlers
let page = params.page.unwrap_or(1).max(1);
let per_page = params.per_page.unwrap_or(20).min(100);
let offset = ((page - 1) * per_page) as i64;

// Data query
let items = sqlx::query_as::<_, T>(
    "SELECT * FROM {table} ORDER BY created_at DESC LIMIT $1 OFFSET $2"
)
.bind(per_page as i64)
.bind(offset)
.fetch_all(&state.db)
.await?;

// Count query
let total: (i64,) = sqlx::query_as("SELECT COUNT(*) FROM {table}")
    .fetch_one(&state.db)
    .await?;
```

### 4.2 Event Publishing Pattern

```rust
// CloudEvents-compatible event format
if let Some(ref nats) = state.nats {
    let event = serde_json::json!({
        "type": "com.opensase.crm.{entity}.{action}",
        "source": "opensase-crm",
        "id": Uuid::new_v4().to_string(),
        "time": Utc::now().to_rfc3339(),
        "data": { /* entity-specific payload */ }
    });
    let _ = nats.publish(
        "com.opensase.crm.{entity}.{action}".to_string(),
        serde_json::to_vec(&event).unwrap().into(),
    ).await;
}
```

## 5. Configuration Design

### 5.1 Environment Variables

```mermaid
graph TB
    subgraph "Required"
        DB_URL["DATABASE_URL<br/>PostgreSQL connection string"]
    end

    subgraph "Optional with Defaults"
        HOST["HOST<br/>Default: 0.0.0.0"]
        PORT2["PORT<br/>Default: 8081"]
        MAX_CONN["DATABASE_MAX_CONNECTIONS<br/>Default: 10"]
        RUST_LOG["RUST_LOG<br/>Default: info"]
    end

    subgraph "Optional Integrations"
        NATS_URL["NATS_URL<br/>NATS connection"]
        SASE_API["SASE_API_URL<br/>Platform API"]
        SASE_KEY["SASE_API_KEY<br/>Platform auth"]
        SASE_TENANT["SASE_TENANT_ID<br/>Tenant identifier"]
    end
```

### 5.2 Config Loading Flow

```rust
// Config loads from environment variables with fallbacks
Config {
    server: ServerConfig {
        host: env::var("HOST").unwrap_or("0.0.0.0"),
        port: env::var("PORT").parse().unwrap_or(8081),
    },
    database: DatabaseConfig {
        url: env::var("DATABASE_URL").expect("required"),
        max_connections: env::var("DATABASE_MAX_CONNECTIONS")
            .parse().unwrap_or(10),
    },
    nats: env::var("NATS_URL").ok().map(|url| NatsConfig { url }),
    sase: load_sase_config(), // All three vars must be present
}
```

## 6. Error Handling Design

### 6.1 Error Hierarchy

```mermaid
graph TB
    subgraph "Domain Errors"
        CE["ContactError<br/>AlreadyQualified<br/>CannotQualifyUnqualified<br/>AlreadyCustomer"]
        DE2["DealError<br/>DealNotOpen<br/>AlreadyOpen<br/>AlreadyInStage<br/>CurrencyMismatch"]
        TE["TicketError<br/>AlreadySolved"]
    end

    subgraph "Application Errors"
        UCE["UseCaseError<br/>NotFound<br/>ValidationError<br/>DomainError<br/>RepositoryError<br/>Unauthorized"]
    end

    subgraph "Infrastructure Errors"
        RE["RepositoryError<br/>NotFound<br/>DuplicateKey<br/>ConnectionError<br/>QueryError<br/>SerializationError"]
        VE["EmailError<br/>Empty<br/>InvalidFormat"]
        ME["MoneyError<br/>CurrencyMismatch<br/>NegativeAmount"]
    end

    subgraph "HTTP Errors"
        H400["400 Bad Request<br/>Validation failures"]
        H404["404 Not Found<br/>Entity not found"]
        H500["500 Internal Server Error<br/>Database/system errors"]
    end

    CE & DE2 --> UCE
    RE --> UCE
    VE & ME --> H400
    UCE --> H400 & H404 & H500
```

## 7. Testing Design

### 7.1 Test Structure

The codebase includes unit tests co-located with their implementations:

- `domain/aggregates/contact.rs` -- 7 tests: creation, events, qualify, convert, score, tags
- `domain/aggregates/deal.rs` -- 6 tests: creation, stage move, close won/lost, closed modification, weighted value
- `domain/value_objects/email.rs` -- 6 tests: valid, lowercase, trim, empty, no-at, no-domain
- `domain/value_objects/money.rs` -- 5 tests: creation, from-cents, add, currency-mismatch, multiply
- `domain/services/mod.rs` -- 2 tests: lead scoring, weighted pipeline
- `imports/support_core/domain/aggregates/ticket.rs` -- 1 test: ticket workflow

### 7.2 Test Execution

```bash
# Run all unit tests
cargo test --all-features

# Run tests with output
cargo test -- --nocapture

# Run specific test module
cargo test domain::aggregates::contact

# Run with coverage
cargo tarpaulin --all-features
```
