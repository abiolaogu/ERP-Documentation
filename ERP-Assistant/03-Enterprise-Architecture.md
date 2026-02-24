# ERP-Assistant Enterprise Architecture

## 1. Strategic Context

ERP-Assistant is the intelligent conversational layer of the OpenSASE ERP platform. It occupies a unique position in the enterprise architecture as the **only module that connects to every other module**, serving as the universal access point for users who need to query, act on, or be briefed about data across Finance, CRM, HCM, Commerce, Healthcare, SCM, Projects, Workspace, and all external productivity tools.

### Business Capability Mapping

```mermaid
graph TB
    subgraph "Enterprise Capabilities"
        subgraph "Conversational Intelligence"
            NLQ["Natural Language Queries"]
            MTC["Multi-Turn Conversations"]
            IR["Intent Routing"]
            ER["Entity Resolution"]
        end
        subgraph "Cross-Module Orchestration"
            CMA["Cross-Module Actions"]
            WFA["Workflow Automation"]
            BSA["Bulk System Actions"]
        end
        subgraph "Proactive Intelligence"
            DB["Daily Briefings"]
            AD["Anomaly Detection"]
            PA["Pending Approvals"]
            DL["Deadline Tracking"]
        end
        subgraph "User Personalization"
            UP["User Preferences"]
            CM["Conversation Memory"]
            PS["Personalized Shortcuts"]
            LB["Learned Behaviors"]
        end
        subgraph "Multi-Modal Interface"
            CI["Chat Interface"]
            CP["Command Palette"]
            VI["Voice Interface"]
            EW["Embeddable Widget"]
            MA["Mobile App"]
        end
        subgraph "External Integration"
            PROD["Productivity Tools"]
            COMM["Communication Channels"]
            STOR["Cloud Storage"]
        end
    end
```

## 2. C4 Architecture Model

### Level 1: System Context

```mermaid
C4Context
    title ERP-Assistant System Context

    Person(user, "Enterprise User", "Employees, managers, executives")
    Person(admin, "System Administrator", "Configures connectors, policies")
    Person(dev, "Developer", "Integrates via SDK")

    System(assistant, "ERP-Assistant", "Personal AI Assistant connecting all ERP modules and external tools")

    System_Ext(iam, "ERP-IAM", "Identity & Access Management")
    System_Ext(platform, "ERP-Platform", "Tenant Management & Configuration")
    System_Ext(finance, "ERP-Finance", "Financial Management")
    System_Ext(crm, "ERP-CRM", "Customer Relationship Management")
    System_Ext(hcm, "ERP-HCM", "Human Capital Management")
    System_Ext(commerce, "ERP-Commerce", "Commerce & Inventory")
    System_Ext(google, "Google Workspace", "Gmail, Calendar, Drive")
    System_Ext(ms365, "Microsoft 365", "Outlook, Teams, OneDrive")
    System_Ext(slack, "Slack", "Team Communication")
    System_Ext(jira, "Jira", "Issue Tracking")
    System_Ext(claude, "Claude API", "NLP Processing")

    Rel(user, assistant, "Converses via chat, voice, command palette")
    Rel(admin, assistant, "Configures connectors, guardrails")
    Rel(dev, assistant, "Integrates via TypeScript/Python/Go SDK")
    Rel(assistant, iam, "Authenticates users, validates tokens")
    Rel(assistant, platform, "Resolves tenant configuration")
    Rel(assistant, finance, "Queries invoices, budgets, GL")
    Rel(assistant, crm, "Queries contacts, deals, pipelines")
    Rel(assistant, hcm, "Queries employees, payroll, leave")
    Rel(assistant, commerce, "Queries orders, inventory")
    Rel(assistant, google, "OAuth2: Gmail, Calendar, Drive")
    Rel(assistant, ms365, "OAuth2: Outlook, Teams")
    Rel(assistant, slack, "OAuth2: Messages, channels")
    Rel(assistant, jira, "OAuth2: Issues, sprints")
    Rel(assistant, claude, "NLP processing, tool calling")
```

### Level 2: Container View

```mermaid
C4Container
    title ERP-Assistant Container Diagram

    Person(user, "User")

    Container_Boundary(frontend, "Frontend Layer") {
        Container(web, "Web App", "Next.js 14", "Chat + Command Palette")
        Container(widget, "Widget", "React 18", "Embeddable component")
        Container(mobile, "Mobile App", "Flutter", "iOS/Android")
    }

    Container_Boundary(api, "API Layer") {
        Container(gateway, "API Gateway", "Go 1.22", "Routes, auth, rate limiting")
    }

    Container_Boundary(services, "Service Layer") {
        Container(core, "assistant-core", "Go", "NLP orchestration, Claude API")
        Container(hub, "connector-hub", "Go", "OAuth2, auto-discovery")
        Container(engine, "action-engine", "Go", "AIDD-governed execution")
        Container(memory, "memory-service", "Python/FastAPI", "Vector memory")
        Container(briefing, "briefing-service", "Go", "AI briefings")
        Container(voice, "voice-service", "Python/FastAPI", "STT + TTS")
    }

    Container_Boundary(data, "Data Layer") {
        ContainerDb(pg, "PostgreSQL 16", "RDBMS", "Conversations, configs")
        ContainerDb(redis, "Redis 7", "Cache", "Session state, rate limits")
        ContainerDb(qdrant, "Qdrant", "Vector DB", "Semantic memory")
        ContainerDb(kafka, "Redpanda", "Event Bus", "CloudEvents")
    }

    Rel(user, web, "HTTPS")
    Rel(user, widget, "HTTPS")
    Rel(user, mobile, "HTTPS")
    Rel(web, gateway, "REST/WebSocket")
    Rel(widget, gateway, "REST/WebSocket")
    Rel(mobile, gateway, "REST/WebSocket")
    Rel(gateway, core, "gRPC/HTTP")
    Rel(core, hub, "HTTP")
    Rel(core, engine, "HTTP")
    Rel(core, memory, "HTTP")
    Rel(core, briefing, "HTTP")
    Rel(gateway, voice, "WebSocket")
    Rel(hub, pg, "SQL")
    Rel(core, redis, "Redis Protocol")
    Rel(memory, qdrant, "gRPC")
    Rel(engine, kafka, "Produce")
```

### Level 3: Component View -- assistant-core

```mermaid
flowchart TB
    subgraph "assistant-core Components"
        API["HTTP Handler Layer"]
        NLP["NLP Engine<br/>(Claude API Client)"]
        INTENT["Intent Classifier"]
        ENTITY["Entity Resolver"]
        TOOLS["Tool Registry<br/>(Dynamic from capabilities.json)"]
        CONV["Conversation Manager<br/>(Per-user, Per-tenant)"]
        CONFIRM["Confirmation Engine"]
        ROUTER["Action Router"]
    end

    API --> NLP
    NLP --> INTENT
    NLP --> ENTITY
    NLP --> TOOLS
    INTENT --> ROUTER
    ENTITY --> ROUTER
    ROUTER --> CONFIRM
    CONV --> NLP
    TOOLS --> NLP

    ROUTER -->|read| AE_READ["action-engine (auto)"]
    ROUTER -->|write| AE_WRITE["action-engine (confirm)"]
    ROUTER -->|delete| AE_DELETE["action-engine (always confirm)"]
```

## 3. Integration Architecture

### Integration Topology

```mermaid
graph LR
    subgraph "ERP-Assistant Hub"
        CH["connector-hub"]
    end

    subgraph "ERP Internal (Auto-Discovery)"
        F["ERP-Finance"]
        C["ERP-CRM"]
        H["ERP-HCM"]
        CO["ERP-Commerce"]
        HC["ERP-Healthcare"]
        SM["ERP-School-Mgmt"]
        CM["ERP-Church-Mgmt"]
        BO["ERP-BSS-OSS"]
        PL["ERP-Platform"]
        WS["ERP-Workspace"]
    end

    subgraph "External (OAuth2)"
        GW["Google Workspace"]
        MS["Microsoft 365"]
        NO["Notion"]
        SL["Slack"]
        JI["Jira"]
        AS["Asana"]
        TR["Trello"]
        LI["Linear"]
        TD["Todoist"]
        CL["Calendly"]
    end

    subgraph "Messaging (API)"
        WA["WhatsApp"]
        TG["Telegram"]
        DC["Discord"]
        SMS["SMS"]
    end

    subgraph "Storage (SDK)"
        DB["Dropbox"]
        BX["Box"]
        S3["Amazon S3"]
    end

    CH --- F & C & H & CO & HC & SM & CM & BO & PL & WS
    CH --- GW & MS & NO & SL & JI & AS & TR & LI & TD & CL
    CH --- WA & TG & DC & SMS
    CH --- DB & BX & S3
```

### Authentication Flow

```mermaid
sequenceDiagram
    participant U as User
    participant GW as API Gateway
    participant IAM as ERP-IAM
    participant AC as assistant-core
    participant CH as connector-hub

    U->>GW: POST /v1/command (Bearer token + X-Tenant-ID)
    GW->>IAM: Validate JWT
    IAM-->>GW: Token valid, scopes: [assistant.read, assistant.write]
    GW->>AC: Forward request with validated claims
    AC->>CH: Need data from ERP-Finance
    CH->>CH: Load encrypted token from vault
    CH->>CH: Decrypt with AES-256-GCM
    CH-->>AC: Finance data response
    AC-->>GW: AI-generated response
    GW-->>U: JSON response
```

## 4. Data Architecture

### Data Flow Matrix

| Data Domain | Source | Storage | Access Pattern |
|------------|--------|---------|----------------|
| Conversations | User input | PostgreSQL | Write-heavy, per-tenant partitioned |
| Conversation context | assistant-core | Redis | Ephemeral, TTL-based |
| User preferences | memory-service | Qdrant + PostgreSQL | Read-heavy, personalized |
| OAuth tokens | connector-hub | PostgreSQL (encrypted) | Read-heavy, cached in Redis |
| Briefing data | briefing-service | PostgreSQL | Scheduled generation, read-heavy |
| Voice transcripts | voice-service | PostgreSQL + S3 | Write-once, read-many |
| Audit logs | action-engine | PostgreSQL + Redpanda | Append-only, compliance |
| Module capabilities | ERP-* modules | Redis (cached) | Periodic refresh |

### Tenant Isolation Model

```mermaid
graph TB
    subgraph "Tenant A"
        A_CONV["Conversations A"]
        A_MEM["Memory A"]
        A_BRIEF["Briefings A"]
        A_TOKENS["OAuth Tokens A"]
    end
    subgraph "Tenant B"
        B_CONV["Conversations B"]
        B_MEM["Memory B"]
        B_BRIEF["Briefings B"]
        B_TOKENS["OAuth Tokens B"]
    end
    PG["PostgreSQL<br/>(Row-Level Security)"]
    QD["Qdrant<br/>(Collection-per-tenant)"]

    A_CONV --> PG
    A_BRIEF --> PG
    A_TOKENS --> PG
    A_MEM --> QD
    B_CONV --> PG
    B_BRIEF --> PG
    B_TOKENS --> PG
    B_MEM --> QD
```

## 5. Governance & Compliance

### AIDD Guardrails Integration

The Enterprise Architecture enforces AIDD governance at every layer:

1. **API Gateway**: Validates that requests include tenant context and valid authentication
2. **assistant-core**: Classifies intent risk level before routing
3. **action-engine**: Enforces guardrail policies from `aidd.guardrails.yaml`
4. **Audit trail**: Every decision and action is logged to Redpanda for compliance

### Risk Classification Matrix

| Risk Level | Actions | Governance |
|-----------|---------|-----------|
| Low | Read queries, list operations | Autonomous execution |
| Medium | Non-sensitive writes, notifications | Logged, autonomous |
| High | Sensitive writes, workflow automation | Human confirmation required |
| Critical | Delete operations, bulk mutations | Always confirm with preview |
| Prohibited | Cross-tenant access, privilege escalation | Blocked, alert raised |

## 6. Disaster Recovery & Business Continuity

| Component | RPO | RTO | Strategy |
|-----------|-----|-----|----------|
| PostgreSQL | 1 minute | 15 minutes | Streaming replication + WAL archiving |
| Redis | 5 minutes | 5 minutes | Redis Sentinel + AOF persistence |
| Qdrant | 1 hour | 30 minutes | Snapshot + restore from S3 |
| Redpanda | 0 (replicated) | 5 minutes | Multi-broker replication |
| Services | N/A | 2 minutes | Kubernetes rolling deployment |

## 7. Technology Radar

| Technology | Status | Rationale |
|-----------|--------|-----------|
| Claude API | Adopt | Best tool-calling accuracy for ERP use cases |
| Qdrant | Adopt | Purpose-built vector search, gRPC native |
| Whisper Large-v3 | Adopt | Best open-source STT accuracy |
| ElevenLabs TTS | Trial | Natural voice quality, evaluate cost at scale |
| Coqui TTS | Assess | Open-source alternative for self-hosted deployments |
| WebSocket streaming | Adopt | Required for real-time voice and chat |
| Flutter | Adopt | Single codebase mobile with native performance |
