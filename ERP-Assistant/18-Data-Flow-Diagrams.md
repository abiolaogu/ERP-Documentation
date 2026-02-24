# ERP-Assistant Data Flow Diagrams

## 1. System-Level Data Flow

```mermaid
flowchart TB
    subgraph "Input Sources"
        WEB["Web Chat"]
        CMD["Command Palette"]
        VOICE["Voice Input"]
        WIDGET["Embeddable Widget"]
        MOBILE["Mobile App"]
        SDK["SDK Clients"]
    end

    subgraph "API Gateway :8090"
        AUTH["JWT Validation"]
        ROUTE["Request Router"]
        RATE["Rate Limiter"]
    end

    subgraph "Processing Pipeline"
        AC["assistant-core<br/>(NLP + Orchestration)"]
        CLAUDE["Claude API<br/>(External)"]
        MS["memory-service<br/>(Context Enrichment)"]
    end

    subgraph "Execution Pipeline"
        AE["action-engine<br/>(Guardrail Enforcement)"]
        CH["connector-hub<br/>(Connector Resolution)"]
    end

    subgraph "Data Targets"
        ERP["ERP Modules<br/>(Finance, CRM, HCM...)"]
        EXT["External Tools<br/>(Google, Slack, Jira...)"]
        BRIEF["briefing-service<br/>(Scheduled Briefings)"]
    end

    subgraph "Data Stores"
        PG["PostgreSQL<br/>(Conversations, Configs)"]
        RD["Redis<br/>(Session, Cache)"]
        QD["Qdrant<br/>(Vector Memory)"]
        KF["Redpanda<br/>(Events + Audit)"]
    end

    WEB & CMD & VOICE & WIDGET & MOBILE & SDK --> AUTH
    AUTH --> RATE --> ROUTE
    ROUTE --> AC
    AC <--> CLAUDE
    AC <--> MS
    AC --> AE
    AE --> CH
    CH --> ERP & EXT
    AC --> BRIEF
    AC --> PG & RD
    MS --> QD
    AE --> KF
    AC --> KF
```

## 2. Natural Language Command Data Flow

```mermaid
flowchart LR
    subgraph "Input"
        USER_INPUT["User Prompt<br/>(text string)"]
    end

    subgraph "Authentication"
        JWT["JWT Token<br/>(Bearer header)"]
        TENANT["Tenant ID<br/>(X-Tenant-ID header)"]
    end

    subgraph "Context Assembly"
        CONV_HIST["Conversation History<br/>(last 10 messages from Redis)"]
        USER_PREF["User Preferences<br/>(from Qdrant)"]
        TOOL_DEFS["Available Tools<br/>(from capabilities cache)"]
    end

    subgraph "NLP Processing"
        CLAUDE_REQ["Claude API Request<br/>(system prompt + context +<br/>user prompt + tools)"]
        CLAUDE_RESP["Claude API Response<br/>(intent + entities +<br/>tool calls)"]
    end

    subgraph "Action Resolution"
        RISK["Risk Classification"]
        EXEC["Execution or Confirmation"]
    end

    subgraph "Output"
        RESPONSE["AI Response<br/>(text + data + suggestions)"]
    end

    USER_INPUT --> CLAUDE_REQ
    JWT --> CONV_HIST & USER_PREF
    TENANT --> TOOL_DEFS
    CONV_HIST & USER_PREF & TOOL_DEFS --> CLAUDE_REQ
    CLAUDE_REQ --> CLAUDE_RESP
    CLAUDE_RESP --> RISK --> EXEC --> RESPONSE
```

## 3. OAuth2 Token Data Flow

```mermaid
flowchart TB
    subgraph "OAuth2 Authorization Flow"
        U["User clicks Connect"]
        AUTH_URL["Generate authorization URL<br/>(+ PKCE challenge + state token)"]
        PROVIDER["OAuth Provider<br/>(Google, Slack, Jira...)"]
        CALLBACK["Receive authorization code<br/>(validate state token)"]
        EXCHANGE["Exchange code for tokens<br/>(+ PKCE verifier)"]
    end

    subgraph "Token Storage Flow"
        ENCRYPT["Encrypt tokens<br/>(AES-256-GCM)"]
        STORE["Store to PostgreSQL<br/>(connector_auths table)"]
        CACHE["Cache in Redis<br/>(TTL = token expiry - 5min)"]
    end

    subgraph "Token Usage Flow"
        REQUEST["API request needs token"]
        CHECK_CACHE["Check Redis cache"]
        CHECK_DB["Read from PostgreSQL"]
        DECRYPT["Decrypt with AES-256-GCM"]
        REFRESH["Refresh if near expiry"]
        USE["Use token for API call"]
    end

    U --> AUTH_URL --> PROVIDER --> CALLBACK --> EXCHANGE
    EXCHANGE --> ENCRYPT --> STORE --> CACHE

    REQUEST --> CHECK_CACHE
    CHECK_CACHE -->|miss| CHECK_DB --> DECRYPT
    CHECK_CACHE -->|hit| USE
    DECRYPT --> REFRESH --> USE
```

## 4. Briefing Generation Data Flow

```mermaid
flowchart TB
    TRIGGER["Trigger<br/>(6:00 AM schedule or manual)"]

    subgraph "Data Collection"
        FIN["ERP-Finance<br/>Revenue, AR, AP, Budgets"]
        CRM["ERP-CRM<br/>Pipeline, Deals Won/Lost"]
        HCM["ERP-HCM<br/>Headcount, Attendance"]
        CAL_G["Google Calendar<br/>Today's Events"]
        CAL_M["Microsoft 365<br/>Today's Events"]
        PROJ["ERP-Projects<br/>Milestones, Deadlines"]
    end

    subgraph "Processing"
        AGG["Aggregate Raw Data"]
        KPI["Calculate KPIs<br/>+ Change %"]
        ANOMALY["Anomaly Detection<br/>(statistical outliers)"]
        PRIORITIZE["Prioritize Items<br/>(urgency + user preferences)"]
    end

    subgraph "Output"
        JSON_BRIEF["JSON Briefing Object"]
        PG_STORE["Store in PostgreSQL"]
        NOTIF["Push Notification"]
        TTS_OPT["TTS Audio (optional)"]
    end

    TRIGGER --> FIN & CRM & HCM & CAL_G & CAL_M & PROJ
    FIN & CRM & HCM & CAL_G & CAL_M & PROJ --> AGG
    AGG --> KPI --> ANOMALY --> PRIORITIZE
    PRIORITIZE --> JSON_BRIEF
    JSON_BRIEF --> PG_STORE & NOTIF & TTS_OPT
```

## 5. Voice Pipeline Data Flow

```mermaid
flowchart LR
    subgraph "Input Pipeline"
        MIC["Microphone<br/>(PCM 16-bit 16kHz)"]
        WS_IN["WebSocket Frame<br/>(binary audio)"]
        WAKE["Wake Word Detection"]
        WHISPER["Whisper Large-v3<br/>(STT)"]
        TRANSCRIPT["Transcript Text<br/>(+ confidence score)"]
    end

    subgraph "Processing"
        NLP["assistant-core<br/>(same as text commands)"]
    end

    subgraph "Output Pipeline"
        AI_TEXT["AI Response Text"]
        TTS["ElevenLabs / Coqui<br/>(TTS)"]
        AUDIO_OUT["Audio Stream<br/>(PCM 16-bit 24kHz)"]
        WS_OUT["WebSocket Frame<br/>(binary audio)"]
        SPEAKER["Speaker Output"]
    end

    MIC --> WS_IN --> WAKE --> WHISPER --> TRANSCRIPT
    TRANSCRIPT --> NLP --> AI_TEXT
    AI_TEXT --> TTS --> AUDIO_OUT --> WS_OUT --> SPEAKER
```

## 6. Memory Service Data Flow

```mermaid
flowchart TB
    subgraph "Ingestion"
        MSG["New Message<br/>(from conversation)"]
        PREF["User Preference Update"]
        SHORTCUT["Shortcut Detection<br/>(repeated command)"]
    end

    subgraph "Embedding"
        TOKENIZE["Tokenize Text"]
        EMBED["Generate Embedding<br/>(sentence-transformers)"]
        VECTOR["384-dim Vector"]
    end

    subgraph "Storage"
        QD_UPSERT["Qdrant Upsert<br/>(collection: memory_{tenant})"]
        PG_META["PostgreSQL Metadata<br/>(shortcuts, preferences)"]
    end

    subgraph "Retrieval"
        QUERY["Semantic Query"]
        Q_EMBED["Query Embedding"]
        QD_SEARCH["Qdrant ANN Search<br/>(cosine similarity, top-k)"]
        RERANK["Re-rank by Recency<br/>+ Relevance"]
        CONTEXT["Enriched Context<br/>(for NLP pipeline)"]
    end

    MSG --> TOKENIZE --> EMBED --> VECTOR --> QD_UPSERT
    PREF --> PG_META
    SHORTCUT --> PG_META

    QUERY --> Q_EMBED --> QD_SEARCH --> RERANK --> CONTEXT
```

## 7. Event Flow Across Services

```mermaid
flowchart LR
    subgraph "Event Producers"
        P1["assistant-core"]
        P2["action-engine"]
        P3["connector-hub"]
        P4["briefing-service"]
        P5["voice-service"]
    end

    subgraph "Redpanda Topics"
        T1["command.received"]
        T2["command.classified"]
        T3["command.executed"]
        T4["action.requested"]
        T5["action.confirmed"]
        T6["action.executed"]
        T7["connector.connected"]
        T8["briefing.created"]
        T9["voice.transcribed"]
        T10["security.blocked"]
    end

    subgraph "Event Consumers"
        C1["Audit Logger<br/>(PostgreSQL)"]
        C2["Analytics<br/>(ClickHouse)"]
        C3["Notification<br/>(Push + Email)"]
        C4["memory-service<br/>(Context Update)"]
        C5["External Webhooks"]
    end

    P1 --> T1 & T2 & T3
    P2 --> T4 & T5 & T6 & T10
    P3 --> T7
    P4 --> T8
    P5 --> T9

    T1 & T2 & T3 & T4 & T5 & T6 & T7 & T8 & T9 & T10 --> C1
    T3 & T6 --> C2
    T5 & T8 & T10 --> C3
    T3 --> C4
    T3 & T6 & T7 --> C5
```

## 8. Tenant Data Isolation Flow

```mermaid
flowchart TB
    REQ["Request with X-Tenant-ID: A"]

    subgraph "Isolation Enforcement"
        GW["Gateway extracts tenant_id"]
        PG_RLS["PostgreSQL: SET app.current_tenant_id = 'A'<br/>RLS filters all queries"]
        QD_NS["Qdrant: collection = memory_A"]
        RD_NS["Redis: key prefix = A:"]
        KF_HDR["Redpanda: tenantid header = A"]
    end

    subgraph "Data Access (Tenant A only)"
        CONV_A["Conversations for Tenant A"]
        MEM_A["Memory vectors for Tenant A"]
        CACHE_A["Cache entries for Tenant A"]
        EVENTS_A["Events tagged Tenant A"]
    end

    REQ --> GW
    GW --> PG_RLS --> CONV_A
    GW --> QD_NS --> MEM_A
    GW --> RD_NS --> CACHE_A
    GW --> KF_HDR --> EVENTS_A
```
