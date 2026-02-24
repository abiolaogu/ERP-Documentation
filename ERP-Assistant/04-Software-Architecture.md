# ERP-Assistant Software Architecture

## 1. Architectural Style

ERP-Assistant employs a **polyglot microservices architecture** with an event-driven backbone. Go services handle orchestration, connection management, and business logic; Python services handle machine learning workloads (vector search, speech processing). All services communicate through a combination of synchronous HTTP/gRPC calls and asynchronous CloudEvents via Redpanda/Kafka.

### Architectural Principles

1. **Single Responsibility**: Each service owns exactly one domain (NLP, connections, actions, memory, briefings, voice)
2. **Polyglot by Design**: Language chosen per-domain constraint -- Go for performance-critical orchestration, Python for ML ecosystem
3. **AIDD-First**: Every action pipeline incorporates governance checks before execution
4. **Tenant Isolation**: All data paths enforce tenant context via `X-Tenant-ID` propagation
5. **Connector Auto-Discovery**: New ERP modules are automatically integrated via capabilities.json scanning
6. **Fail-Open for Reads, Fail-Closed for Writes**: Read operations degrade gracefully; write operations require explicit confirmation

## 2. Service Architecture

```mermaid
graph TB
    subgraph "API Layer"
        GW["API Gateway<br/>Go :8090<br/>JWT Validation, Routing"]
    end

    subgraph "Orchestration Layer"
        AC["assistant-core<br/>Go<br/>Claude API, Intent Routing,<br/>Entity Resolution, Tool Calling"]
    end

    subgraph "Execution Layer"
        AE["action-engine<br/>Go<br/>AIDD Guardrails,<br/>Cross-system Execution"]
        CH["connector-hub<br/>Go<br/>OAuth2, Token Vault,<br/>Auto-discovery"]
    end

    subgraph "Intelligence Layer"
        MS["memory-service<br/>Python/FastAPI<br/>Qdrant Vectors,<br/>Preferences"]
        BS["briefing-service<br/>Go<br/>KPI Aggregation,<br/>Anomaly Detection"]
    end

    subgraph "Interface Layer"
        VS["voice-service<br/>Python/FastAPI<br/>Whisper STT,<br/>ElevenLabs TTS"]
    end

    GW --> AC
    GW --> VS
    AC --> AE
    AC --> CH
    AC --> MS
    AC --> BS
    AE --> CH
```

## 3. Domain Model

### Core Domain Entities

```mermaid
erDiagram
    TENANT ||--o{ USER : "has"
    USER ||--o{ CONVERSATION : "creates"
    CONVERSATION ||--o{ MESSAGE : "contains"
    MESSAGE ||--o{ ACTION : "triggers"
    ACTION ||--o{ CONFIRMATION : "requires"
    USER ||--o{ PREFERENCE : "configures"
    USER ||--o{ CONNECTOR_AUTH : "authorizes"
    TENANT ||--o{ CONNECTOR_CONFIG : "configures"
    CONNECTOR_CONFIG ||--o{ CONNECTOR_AUTH : "enables"
    USER ||--o{ BRIEFING : "receives"
    BRIEFING ||--o{ BRIEFING_SECTION : "contains"
    USER ||--o{ VOICE_SESSION : "initiates"
    VOICE_SESSION ||--o{ TRANSCRIPT : "produces"
    USER ||--o{ SHORTCUT : "creates"

    TENANT {
        uuid id PK
        string name
        jsonb settings
    }
    USER {
        uuid id PK
        uuid tenant_id FK
        string email
        string display_name
        jsonb preferences
    }
    CONVERSATION {
        uuid id PK
        uuid user_id FK
        uuid tenant_id FK
        string title
        timestamp created_at
        timestamp updated_at
        string status
    }
    MESSAGE {
        uuid id PK
        uuid conversation_id FK
        string role
        text content
        jsonb tool_calls
        jsonb metadata
        timestamp created_at
    }
    ACTION {
        uuid id PK
        uuid message_id FK
        string action_type
        string target_module
        string target_entity
        jsonb parameters
        string risk_level
        string status
        timestamp executed_at
    }
    CONFIRMATION {
        uuid id PK
        uuid action_id FK
        string decision
        text reason
        timestamp decided_at
    }
    PREFERENCE {
        uuid id PK
        uuid user_id FK
        string key
        jsonb value
    }
    CONNECTOR_AUTH {
        uuid id PK
        uuid user_id FK
        uuid connector_config_id FK
        bytea encrypted_token
        timestamp expires_at
    }
    CONNECTOR_CONFIG {
        uuid id PK
        uuid tenant_id FK
        string connector_type
        string provider
        jsonb settings
        boolean enabled
    }
    BRIEFING {
        uuid id PK
        uuid user_id FK
        string type
        date briefing_date
        jsonb content
        timestamp generated_at
    }
    BRIEFING_SECTION {
        uuid id PK
        uuid briefing_id FK
        string section_type
        jsonb data
        integer sort_order
    }
    VOICE_SESSION {
        uuid id PK
        uuid user_id FK
        timestamp started_at
        timestamp ended_at
    }
    TRANSCRIPT {
        uuid id PK
        uuid session_id FK
        text content
        float confidence
        timestamp created_at
    }
    SHORTCUT {
        uuid id PK
        uuid user_id FK
        string label
        string command
        integer usage_count
    }
```

## 4. Request Processing Pipeline

### Natural Language Command Flow

```mermaid
sequenceDiagram
    participant U as User
    participant GW as Gateway
    participant AC as assistant-core
    participant NLP as Claude API
    participant MS as memory-service
    participant CH as connector-hub
    participant AE as action-engine
    participant MOD as ERP Module

    U->>GW: POST /v1/command {"prompt": "What's my revenue this quarter?"}
    GW->>GW: Validate JWT + X-Tenant-ID
    GW->>AC: Forward authenticated request

    AC->>MS: Fetch user context & preferences
    MS-->>AC: User prefers Finance dashboard format

    AC->>NLP: Send prompt + conversation history + tool definitions
    NLP-->>AC: Intent: QUERY, Module: Finance, Entity: Revenue, Period: Q1 2026

    AC->>CH: Get Finance connector
    CH-->>AC: Finance connector with valid token

    AC->>AE: Execute read action (risk: low)
    AE->>AE: Check AIDD guardrails (read = autonomous)
    AE->>MOD: GET /v1/finance/revenue?period=Q1-2026
    MOD-->>AE: Revenue data

    AE-->>AC: Action result
    AC->>NLP: Format response with data
    NLP-->>AC: "Your Q1 2026 revenue is $2.4M, up 12% from last quarter."

    AC->>MS: Store interaction for memory
    AC-->>GW: Formatted response
    GW-->>U: JSON response with AI message
```

### Write Action with Confirmation

```mermaid
sequenceDiagram
    participant U as User
    participant AC as assistant-core
    participant AE as action-engine
    participant MOD as ERP Module

    U->>AC: "Approve PO-2024-0891"
    AC->>AE: Execute write action (risk: high)
    AE->>AE: Check AIDD guardrails (sensitive write = confirm)
    AE-->>AC: Confirmation required

    AC-->>U: "I'll approve PO-2024-0891 for $45,000 from Vendor XYZ. Confirm?"

    U->>AC: "Yes, approve it"
    AC->>AE: Execute confirmed action
    AE->>MOD: POST /v1/finance/purchase-orders/PO-2024-0891/approve
    MOD-->>AE: Approved
    AE->>AE: Log decision to audit trail
    AE-->>AC: Action completed
    AC-->>U: "PO-2024-0891 has been approved."
```

## 5. assistant-core Internal Architecture

```mermaid
flowchart TB
    subgraph "Input Processing"
        REQ["HTTP Request Handler"]
        AUTH["Auth Middleware"]
        RATE["Rate Limiter"]
    end

    subgraph "NLP Pipeline"
        CTX["Context Assembler<br/>(conversation history +<br/>user preferences +<br/>available tools)"]
        CLAUDE["Claude API Client<br/>(streaming, tool calling)"]
        INTENT["Intent Classifier<br/>(query, action, briefing,<br/>navigation, workflow)"]
        ENTITY["Entity Extractor<br/>(module, entity type,<br/>identifiers, filters)"]
    end

    subgraph "Tool Registry"
        DISC["Auto-Discovery Engine"]
        TOOL_DEF["Tool Definition Builder"]
        TOOL_EXEC["Tool Execution Proxy"]
    end

    subgraph "State Management"
        CONV_MGR["Conversation Manager"]
        REDIS_STATE["Redis Session Store"]
        PG_HIST["PostgreSQL History"]
    end

    REQ --> AUTH --> RATE --> CTX
    CTX --> CLAUDE
    CLAUDE --> INTENT
    CLAUDE --> ENTITY
    INTENT --> TOOL_EXEC
    ENTITY --> TOOL_EXEC
    DISC --> TOOL_DEF --> CLAUDE
    CONV_MGR --> CTX
    CONV_MGR --> REDIS_STATE
    CONV_MGR --> PG_HIST
```

## 6. connector-hub Architecture

### OAuth2 Connection Lifecycle

```mermaid
statechart-v2
```

```mermaid
flowchart LR
    DISC["Discovery<br/>(capabilities.json scan)"] --> INIT["Initialization<br/>(OAuth2 config loaded)"]
    INIT --> AUTH["Authorization<br/>(User grants access)"]
    AUTH --> ACTIVE["Active<br/>(Token stored, encrypted)"]
    ACTIVE --> REFRESH["Token Refresh<br/>(Auto before expiry)"]
    REFRESH --> ACTIVE
    ACTIVE --> REVOKED["Revoked<br/>(User disconnects)"]
    ACTIVE --> EXPIRED["Expired<br/>(Refresh failed)"]
    EXPIRED --> AUTH
```

### Connector Interface

```go
type Connector interface {
    // Discovery
    Capabilities() CapabilityDoc
    HealthCheck(ctx context.Context) error

    // Authentication
    AuthorizationURL(state string) string
    ExchangeToken(ctx context.Context, code string) (*Token, error)
    RefreshToken(ctx context.Context, token *Token) (*Token, error)

    // Operations
    Execute(ctx context.Context, action Action) (*Result, error)
    Search(ctx context.Context, query SearchQuery) (*SearchResult, error)
}
```

## 7. memory-service Architecture

### Vector Pipeline

```mermaid
flowchart LR
    INPUT["User Interaction<br/>(message, preference,<br/>shortcut)"] --> EMBED["Embedding<br/>(sentence-transformers)"]
    EMBED --> STORE["Qdrant Upsert<br/>(collection per tenant)"]

    QUERY["Semantic Query"] --> Q_EMBED["Query Embedding"]
    Q_EMBED --> SEARCH["Qdrant Search<br/>(cosine similarity)"]
    SEARCH --> RANK["Re-rank Results"]
    RANK --> RESPONSE["Contextual Memory"]
```

### Memory Types

| Type | Storage | TTL | Update Frequency |
|------|---------|-----|-----------------|
| Conversation history | Qdrant + PostgreSQL | 90 days | Every message |
| User preferences | Qdrant + PostgreSQL | Permanent | On change |
| Shortcuts | PostgreSQL | Permanent | On usage |
| Module context | Redis | 1 hour | On query |
| Embeddings | Qdrant | 90 days | On ingestion |

## 8. Error Handling Strategy

```mermaid
flowchart TB
    ERR["Error Occurs"] --> CLASSIFY["Classify Error"]
    CLASSIFY -->|Transient| RETRY["Retry with Backoff<br/>(max 3 attempts)"]
    CLASSIFY -->|Auth| REAUTH["Re-authenticate<br/>(token refresh)"]
    CLASSIFY -->|Rate Limit| BACKOFF["Exponential Backoff<br/>(respect Retry-After)"]
    CLASSIFY -->|Permanent| FAIL["Fail Gracefully<br/>(user-friendly message)"]
    CLASSIFY -->|Connector Down| DEGRADE["Degrade Gracefully<br/>(skip unavailable source)"]

    RETRY -->|Success| CONTINUE["Continue Pipeline"]
    RETRY -->|Exhausted| FAIL
    REAUTH -->|Success| CONTINUE
    REAUTH -->|Failed| FAIL
    BACKOFF --> RETRY
    DEGRADE --> CONTINUE
```

## 9. Performance Architecture

### Caching Strategy

| Layer | Cache | TTL | Invalidation |
|-------|-------|-----|-------------|
| API Gateway | Redis | 60s | On write event |
| Conversation context | Redis | 30 min | Session end |
| Module capabilities | Redis | 5 min | On module restart |
| OAuth tokens | Redis | Token expiry - 5min | On refresh |
| Briefing data | Redis | 1 hour | On regeneration |
| User preferences | Redis | 15 min | On preference change |

### Concurrency Model

- **assistant-core**: Go goroutine per request, bounded by semaphore (max 1000 concurrent)
- **connector-hub**: Connection pool per connector (max 50 per external service)
- **action-engine**: Serialized per-user for write operations, parallel for reads
- **memory-service**: Python asyncio with FastAPI, uvicorn workers (4x CPU cores)
- **voice-service**: WebSocket connections with asyncio, one goroutine per stream

## 10. Observability

### Metrics

| Metric | Type | Labels |
|--------|------|--------|
| `assistant_command_duration_seconds` | Histogram | intent, module, status |
| `assistant_action_total` | Counter | action_type, risk_level, outcome |
| `assistant_connector_health` | Gauge | connector, provider |
| `assistant_conversation_active` | Gauge | tenant_id |
| `assistant_briefing_generation_seconds` | Histogram | type, sections |
| `assistant_voice_transcription_seconds` | Histogram | language, confidence_bucket |

### Distributed Tracing

All services propagate OpenTelemetry trace context through HTTP headers. Trace spans cover:

1. Gateway request handling
2. NLP pipeline execution (Claude API call)
3. Entity resolution
4. Connector invocation
5. Action execution
6. Memory storage
7. Response formatting

```mermaid
gantt
    title Typical Command Trace (p50)
    dateFormat X
    axisFormat %Lms
    section Gateway
        Auth validation       :0, 5
        Route dispatch        :5, 7
    section assistant-core
        Context assembly      :7, 15
        Claude API call       :15, 180
        Entity resolution     :180, 200
    section connector-hub
        Token retrieval       :200, 205
        Module API call       :205, 250
    section action-engine
        Guardrail check       :250, 255
        Execute action        :255, 290
    section Response
        Format response       :290, 310
        Send to client        :310, 315
```
