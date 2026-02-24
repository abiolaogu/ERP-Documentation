# ERP-Marketing -- Low-Level Design

## 1. Overview

This document provides detailed design specifications for the core components of ERP-Marketing, including class-level structures, database interaction patterns, API handler implementations, and event processing logic.

## 2. Domain Model Details

### 2.1 Campaign Aggregate

```mermaid
classDiagram
    class Campaign {
        -String id
        -String name
        -CampaignType campaign_type
        -CampaignStatus status
        -Option~String~ subject
        -String content
        -Option~Segment~ segment
        -Option~DateTime~ scheduled_at
        -Option~DateTime~ sent_at
        -CampaignStats stats
        -DateTime created_at
        -Vec~DomainEvent~ events
        +create(name: String, campaign_type: CampaignType) Campaign
        +id() &str
        +status() &CampaignStatus
        +stats() &CampaignStats
        +set_content(subject: String, content: String)
        +set_segment(segment: Segment)
        +schedule(at: DateTime) Result~(), CampaignError~
        +send() Result~(), CampaignError~
        +complete(stats: CampaignStats)
        +cancel()
        +take_events() Vec~DomainEvent~
        -raise_event(e: DomainEvent)
    }

    class CampaignStatus {
        <<enumeration>>
        Draft
        Scheduled
        Sending
        Sent
        Paused
        Cancelled
    }

    class CampaignStats {
        +u64 sent
        +u64 delivered
        +u64 opened
        +u64 clicked
        +u64 bounced
        +u64 unsubscribed
    }

    class CampaignError {
        <<enumeration>>
        NoContent
        NoSegment
    }

    Campaign --> CampaignStatus
    Campaign --> CampaignStats
    Campaign ..> CampaignError
```

### 2.2 Automation Aggregate

```mermaid
classDiagram
    class Automation {
        -String id
        -String name
        -AutomationStatus status
        -AutomationTrigger trigger
        -Vec~AutomationStep~ steps
        -u64 enrolled
        -u64 completed
        -DateTime created_at
        +create(name: String, trigger: AutomationTrigger) Automation
        +id() &str
        +status() &AutomationStatus
        +add_step(step: AutomationStep)
        +activate()
        +pause()
    }

    class AutomationStatus {
        <<enumeration>>
        Draft
        Active
        Paused
        Archived
    }

    class AutomationTrigger {
        <<enumeration>>
        FormSubmission(form_id: String)
        SegmentEntry(segment_id: String)
        DateBased(field: String)
        Manual
    }

    class AutomationStep {
        +String id
        +StepType step_type
        +Option~u32~ delay_hours
        +Vec~String~ conditions
    }

    class StepType {
        <<enumeration>>
        SendEmail(template_id: String)
        Wait(hours: u32)
        IfElse(condition: String)
        AddTag(tag: String)
        UpdateProperty(field: String, value: String)
    }

    Automation --> AutomationStatus
    Automation --> AutomationTrigger
    Automation --> AutomationStep
    AutomationStep --> StepType
```

### 2.3 Value Objects

```mermaid
classDiagram
    class CampaignType {
        <<enumeration>>
        Email
        Sms
        Push
        InApp
        Social
    }

    class Segment {
        +String id
        +String name
        +SegmentFilter filter
        +u64 contact_count
    }

    class SegmentFilter {
        +Vec~FilterCondition~ conditions
        +FilterLogic logic
    }

    class FilterLogic {
        <<enumeration>>
        And
        Or
    }

    class FilterCondition {
        +String field
        +String operator
        +String value
    }

    Segment --> SegmentFilter
    SegmentFilter --> FilterCondition
    SegmentFilter --> FilterLogic
```

## 3. API Handler Implementation Details

### 3.1 Request/Response Flow

```mermaid
sequenceDiagram
    participant Client
    participant axum as axum Router
    participant Handler as Handler Function
    participant State as AppState
    participant sqlx as sqlx PgPool
    participant PG as PostgreSQL

    Client->>axum: HTTP Request
    axum->>Handler: Route match + Extract(State, Path, Json)
    Handler->>State: Access db pool
    State->>sqlx: query_as::<_, Struct>
    sqlx->>PG: SQL query
    PG-->>sqlx: Row data
    sqlx-->>Handler: Deserialized struct
    Handler-->>Client: Json<Struct> + StatusCode
```

### 3.2 Create Campaign Handler (Detailed)

```rust
// Input validation
struct CreateCampaignRequest {
    name: String,           // Required, non-empty
    subject: Option<String>, // Optional for non-email channels
    channel: Option<String>, // Default: "email"
    audience_id: Option<Uuid>,
    template_id: Option<Uuid>,
}

// Handler logic:
// 1. Generate UUID v7 (time-ordered)
// 2. Default channel to "email" if not provided
// 3. Default status to "draft"
// 4. Default stats to empty JSON object
// 5. INSERT with RETURNING * for atomic create+read
// 6. Return 201 Created with full campaign object
```

### 3.3 AIDD Guardrail Evaluation Logic

```mermaid
flowchart TB
    INPUT[Action Request<br/>confidence, blast_radius, monetary_value] --> CHECK_PROHIBITED{Is Action<br/>Prohibited?}
    CHECK_PROHIBITED -->|Yes| BLOCK[Return BLOCKED]
    CHECK_PROHIBITED -->|No| CHECK_CONF{confidence >= min_confidence<br/>0.64?}
    CHECK_CONF -->|No| BLOCK
    CHECK_CONF -->|Yes| CHECK_BLAST{blast_radius <= max_blast_radius<br/>15000?}
    CHECK_BLAST -->|No| NEEDS_REVIEW[Return NEEDS_REVIEW]
    CHECK_BLAST -->|Yes| CHECK_VALUE{monetary_value >= high_value_amount<br/>250000?}
    CHECK_VALUE -->|Yes| NEEDS_REVIEW
    CHECK_VALUE -->|No| CHECK_MED{confidence >= medium_confidence<br/>0.78?}
    CHECK_MED -->|No| NEEDS_REVIEW
    CHECK_MED -->|Yes| APPROVE[Return APPROVED]

    BLOCK --> PERSIST[INSERT INTO marketing_aidd_guardrail_events]
    NEEDS_REVIEW --> PERSIST
    APPROVE --> PERSIST
```

## 4. Database Access Patterns

### 4.1 Connection Pool Configuration

```rust
PgPoolOptions::new()
    .max_connections(10)          // Max concurrent connections
    .connect(&database_url)       // From DATABASE_URL env var
    .await?;
```

### 4.2 Migration Execution

```rust
sqlx::migrate!("./migrations")   // Compile-time migration embedding
    .run(&db)                     // Run on startup
    .await?;
```

### 4.3 Query Patterns

| Pattern | SQL | Usage |
|---|---|---|
| List with ordering | `SELECT * FROM campaigns ORDER BY created_at DESC` | List endpoints |
| Get by ID | `SELECT * FROM campaigns WHERE id = $1` | Detail endpoints |
| Insert with return | `INSERT INTO ... VALUES ($1, ...) RETURNING *` | Create endpoints |
| Update with return | `UPDATE ... SET ... WHERE id = $1 RETURNING *` | Update endpoints |
| Count aggregation | `SELECT COUNT(*) FROM contacts WHERE lifecycle_stage = 'mql'` | Dashboard KPIs |
| Weighted aggregation | `SELECT channel, SUM(attribution_weight) FROM touchpoints GROUP BY channel` | Attribution |

## 5. Go Microservice Internal Design

### 5.1 Service Structure

```mermaid
classDiagram
    class ServiceHandler {
        +handleList(w, r)
        +handleCreate(w, r)
        +handleRead(w, r)
        +handleUpdate(w, r)
        +handleDelete(w, r)
    }

    class TenantMiddleware {
        +validateTenantID(r) error
    }

    class EventEmitter {
        +emit(topic: string, payload: any) error
    }

    class JSONWriter {
        +writeJSON(w, code, v)
    }

    ServiceHandler --> TenantMiddleware
    ServiceHandler --> EventEmitter
    ServiceHandler --> JSONWriter
```

### 5.2 Tenant Validation

Every Go service handler validates the `X-Tenant-ID` header as the first operation:

```go
if r.Header.Get("X-Tenant-ID") == "" {
    writeJSON(w, http.StatusBadRequest, map[string]string{
        "error": "missing X-Tenant-ID",
    })
    return
}
```

### 5.3 Event Topic Naming

Events follow the convention `erp.marketing.<entity>.<action>`:

| Entity | Actions | Example Topic |
|---|---|---|
| campaign | created, read, listed, updated, deleted | `erp.marketing.campaign.created` |
| journey | created, read, listed, updated, deleted | `erp.marketing.journey.activated` |
| social | created, read, listed, updated, deleted | `erp.marketing.social.published` |
| ads | created, read, listed, updated, deleted | `erp.marketing.ads.launched` |
| content | created, read, listed, updated, deleted | `erp.marketing.content.published` |
| segment | created, read, listed, updated, deleted | `erp.marketing.segment.evaluated` |
| attribution | created, read, listed, updated, deleted | `erp.marketing.attribution.calculated` |
| email-marketing | created, read, listed, updated, deleted | `erp.marketing.email-marketing.sent` |
| analytics | created, read, listed, updated, deleted | `erp.marketing.analytics.generated` |

## 6. Frontend API Client Design

### 6.1 Request Pipeline

```mermaid
flowchart LR
    CALL[API Function Call] --> BUILD[Build Request<br/>URL + Headers + Body]
    BUILD --> FETCH[fetch() with<br/>Content-Type: application/json]
    FETCH --> CHECK{Response OK?}
    CHECK -->|No| ERROR[throw ApiClientError<br/>status + message]
    CHECK -->|Yes| STATUS{204 No Content?}
    STATUS -->|Yes| UNDEFINED[Return undefined]
    STATUS -->|No| PARSE[Parse JSON]
    PARSE --> TRANSFORM[Transform snake_case<br/>to camelCase]
    TRANSFORM --> RETURN[Return typed object]
```

### 6.2 Type Transformation Pattern

Each API entity follows a consistent transformation pattern:

1. Define `*Raw` type matching backend snake_case JSON
2. Define public `*` type in camelCase for frontend consumption
3. Write `to*` transformation function
4. Export `fetch*` function that calls API and transforms response

## 7. Error Handling Chain

```mermaid
flowchart TB
    subgraph "Rust Backend"
        DOMAIN_ERR[Domain Error<br/>CampaignError::NoContent]
        DB_ERR[Database Error<br/>sqlx::Error]
        INFRA_ERR[Infrastructure Error<br/>anyhow::Error]
    end
    subgraph "HTTP Layer"
        HTTP_ERR[HTTP Error<br/>StatusCode + String]
    end
    subgraph "Frontend"
        API_ERR[ApiClientError<br/>status + message]
        UI_ERR[UI Error Display<br/>Ant Design notification]
    end
    DOMAIN_ERR --> HTTP_ERR
    DB_ERR --> HTTP_ERR
    INFRA_ERR --> HTTP_ERR
    HTTP_ERR --> API_ERR
    API_ERR --> UI_ERR
```
