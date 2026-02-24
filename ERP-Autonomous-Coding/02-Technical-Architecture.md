# ERP-Autonomous-Coding -- Technical Architecture Document

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Autonomous-Coding |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |
| Status | Draft |
| Classification | Internal -- Engineering |

---

## 1. Architecture Overview

ERP-Autonomous-Coding follows a microservices architecture with six core backend services, four IDE plugin frontends, a CLI tool, and a Next.js web dashboard. The system operates in `standalone_plus_suite` mode, authenticating via ERP-IAM (OIDC/JWT) and resolving entitlements via ERP-Platform.

### 1.1 C4 Context Diagram

```mermaid
C4Context
    title ERP-Autonomous-Coding System Context

    Person(dev, "Developer", "Writes code, reviews PRs, manages repos")
    Person(mgr, "Engineering Manager", "Monitors team velocity and quality")
    Person(devops, "DevOps Engineer", "Manages CI/CD and infrastructure")

    System(ac, "ERP-Autonomous-Coding", "Autonomous AI coding platform")

    System_Ext(iam, "ERP-IAM", "Identity & Access Management")
    System_Ext(platform, "ERP-Platform", "Entitlements & Licensing")
    System_Ext(github, "GitHub", "Git hosting, PRs, Actions")
    System_Ext(gitlab, "GitLab", "Git hosting, MRs, CI/CD")
    System_Ext(bitbucket, "Bitbucket", "Git hosting, PRs, Pipelines")
    System_Ext(azdo, "Azure DevOps", "Git hosting, PRs, Pipelines, Work Items")
    System_Ext(claude, "Claude API (Anthropic)", "LLM reasoning engine")
    System_Ext(snyk, "Snyk", "Dependency vulnerability scanning")
    System_Ext(trivy, "Trivy", "Container/dependency scanning")
    System_Ext(trufflehog, "TruffleHog", "Secret detection")

    Rel(dev, ac, "Uses via IDE/CLI/Web", "HTTPS/WSS")
    Rel(mgr, ac, "Monitors via Dashboard", "HTTPS")
    Rel(devops, ac, "Configures sandboxes/pipelines", "HTTPS/CLI")
    Rel(ac, iam, "Authenticates", "OIDC/JWT")
    Rel(ac, platform, "Entitlements", "gRPC")
    Rel(ac, github, "Git operations", "REST/GraphQL/Webhooks")
    Rel(ac, gitlab, "Git operations", "REST/GraphQL/Webhooks")
    Rel(ac, bitbucket, "Git operations", "REST/Webhooks")
    Rel(ac, azdo, "Git operations", "REST/Webhooks")
    Rel(ac, claude, "LLM inference", "HTTPS")
    Rel(ac, snyk, "Vuln scanning", "HTTPS")
    Rel(ac, trivy, "Vuln scanning", "CLI/HTTPS")
    Rel(ac, trufflehog, "Secret scanning", "CLI")
```

### 1.2 C4 Container Diagram

```mermaid
C4Container
    title ERP-Autonomous-Coding Container Diagram

    Person(user, "User", "Developer / Manager / DevOps")

    Container_Boundary(ac, "ERP-Autonomous-Coding") {
        Container(gateway, "API Gateway", "Go", "Routes, auth, rate limiting")
        Container(agentcore, "Agent Core", "Python / FastAPI", "Autonomous coding agent with Claude API")
        Container(sandbox, "Sandbox Runtime", "Go", "Docker-based sandboxed code execution")
        Container(gitbridge, "Git Bridge", "Go", "Unified Git provider abstraction")
        Container(ideserver, "IDE Server", "TypeScript / Express", "LSP bridge over WebSocket")
        Container(reviewengine, "Review Engine", "Python / FastAPI", "SAST, security, quality analysis")
        Container(taskplanner, "Task Planner", "Python / FastAPI", "Task decomposition and scheduling")
        Container(dashboard, "Web Dashboard", "Next.js 14", "Agent monitoring and management UI")
        Container(cli, "CLI Tool", "Go", "Command-line interface")

        ContainerDb(postgres, "PostgreSQL 16", "Database", "Sessions, tasks, audit logs, configs")
        ContainerDb(redis, "Redis 7", "Cache", "Session state, sandbox pool, rate limits")
        Container(kafka, "Redpanda/Kafka", "Event Bus", "Async event streaming")
    }

    Rel(user, gateway, "HTTPS/WSS")
    Rel(user, cli, "Terminal")
    Rel(user, dashboard, "Browser")
    Rel(gateway, agentcore, "gRPC/REST")
    Rel(gateway, gitbridge, "gRPC/REST")
    Rel(gateway, ideserver, "WebSocket proxy")
    Rel(gateway, reviewengine, "gRPC/REST")
    Rel(gateway, taskplanner, "gRPC/REST")
    Rel(agentcore, sandbox, "gRPC")
    Rel(agentcore, gitbridge, "gRPC")
    Rel(agentcore, reviewengine, "gRPC")
    Rel(agentcore, taskplanner, "gRPC")
    Rel(agentcore, postgres, "SQL")
    Rel(agentcore, redis, "Cache")
    Rel(agentcore, kafka, "Publish events")
    Rel(sandbox, postgres, "Audit logs")
    Rel(gitbridge, kafka, "Webhook events")
    Rel(reviewengine, kafka, "Review results")
    Rel(taskplanner, postgres, "Task state")
```

---

## 2. Service Architecture

### 2.1 Service Inventory

| Service | Language | Framework | Port | Responsibility |
|---------|----------|-----------|------|----------------|
| `autonomous-coding-api` | Go 1.22 | stdlib/chi | 8090 (ext: 8095) | API gateway, routing, auth middleware |
| `agent-core` | Python 3.12 | FastAPI | 8080 (ext: 8205) | Autonomous coding agent orchestration |
| `sandbox-runtime` | Go 1.22 | stdlib | Internal | Docker container lifecycle management |
| `git-bridge` | Go 1.22 | stdlib | Internal | Git provider abstraction layer |
| `ide-server` | TypeScript | Express | 8080 (ext: 8207) | LSP bridge, WebSocket server |
| `review-engine` | Python 3.12 | FastAPI | 8080 (ext: 8206) | Code quality and security analysis |
| `task-planner` | Python 3.12 | FastAPI | 8080 (ext: 8209) | Task decomposition and scheduling |

### 2.2 Agent Core Architecture

```mermaid
flowchart TB
    subgraph "Agent Core"
        direction TB
        API["FastAPI Endpoints"]
        ORCH["Agent Orchestrator"]
        CLAUDE["Claude API Client"]
        TOOLS["Tool Registry"]
        MEMORY["Context Memory"]
        TRACE["Reasoning Trace Logger"]

        API --> ORCH
        ORCH --> CLAUDE
        ORCH --> TOOLS
        ORCH --> MEMORY
        ORCH --> TRACE

        subgraph "Tools"
            T1["FileEditor"]
            T2["TerminalExecutor"]
            T3["GitOperations"]
            T4["TestRunner"]
            T5["CodeSearcher"]
            T6["DocGenerator"]
        end

        TOOLS --> T1
        TOOLS --> T2
        TOOLS --> T3
        TOOLS --> T4
        TOOLS --> T5
        TOOLS --> T6
    end

    subgraph "Iteration Loop"
        direction LR
        GEN["Generate Code"] --> TEST["Run Tests"]
        TEST --> CHECK{Tests Pass?}
        CHECK -->|No| FIX["Fix Issues"]
        FIX --> GEN
        CHECK -->|Yes| VERIFY["Verify & Review"]
        VERIFY --> SUBMIT["Submit PR"]
    end

    ORCH --> GEN
    T2 --> SANDBOX["Sandbox Runtime"]
    T3 --> GITBRIDGE["Git Bridge"]
```

### 2.3 Sandbox Runtime Architecture

```mermaid
flowchart TB
    subgraph "Sandbox Runtime"
        MGR["Container Manager"]
        POOL["Warm Pool Controller"]
        IMAGES["Image Registry"]
        LIMITS["Resource Limiter"]
        SNAP["Filesystem Snapshotter"]
        NET["Network Policy Engine"]

        MGR --> POOL
        MGR --> IMAGES
        MGR --> LIMITS
        MGR --> SNAP
        MGR --> NET
    end

    subgraph "Container Pool"
        C1["Go 1.22 Container"]
        C2["Python 3.12 Container"]
        C3["Node.js 20 Container"]
        C4["Rust 1.77 Container"]
        C5["Java 21 Container"]
        C6[".NET 8 Container"]
    end

    POOL --> C1
    POOL --> C2
    POOL --> C3
    POOL --> C4
    POOL --> C5
    POOL --> C6

    subgraph "Resource Limits"
        CPU["CPU: 2 cores max"]
        MEM["Memory: 4GB max"]
        DISK["Disk: 10GB max"]
        NETTL["Network: configurable"]
    end

    LIMITS --> CPU
    LIMITS --> MEM
    LIMITS --> DISK
    LIMITS --> NETTL
```

### 2.4 Git Bridge Architecture

```mermaid
flowchart TB
    subgraph "Git Bridge"
        ABSTRACT["Unified Git Interface"]
        WEBHOOK["Webhook Processor"]
        AIDD["AIDD Approval Gate"]
        QUEUE["Event Queue"]

        ABSTRACT --> WEBHOOK
        ABSTRACT --> AIDD
        WEBHOOK --> QUEUE
    end

    subgraph "Provider Adapters"
        GH["GitHub Adapter"]
        GL["GitLab Adapter"]
        BB["Bitbucket Adapter"]
        AZ["Azure DevOps Adapter"]
    end

    ABSTRACT --> GH
    ABSTRACT --> GL
    ABSTRACT --> BB
    ABSTRACT --> AZ

    subgraph "GitHub Adapter"
        GH_APP["GitHub App"]
        GH_WH["Webhooks"]
        GH_REST["REST API v3"]
        GH_GQL["GraphQL API v4"]
        GH_ACT["Actions Integration"]
        GH_COP["Copilot Bridge"]
    end

    GH --> GH_APP
    GH --> GH_WH
    GH --> GH_REST
    GH --> GH_GQL
    GH --> GH_ACT
    GH --> GH_COP

    subgraph "GitLab Adapter"
        GL_OAUTH["OAuth2"]
        GL_REST2["REST API v4"]
        GL_GQL2["GraphQL API"]
        GL_CI["CI/CD Integration"]
        GL_MR["Merge Requests"]
    end

    GL --> GL_OAUTH
    GL --> GL_REST2
    GL --> GL_GQL2
    GL --> GL_CI
    GL --> GL_MR

    subgraph "Operations"
        OP1["Clone"]
        OP2["Branch"]
        OP3["Commit"]
        OP4["Push"]
        OP5["Create PR/MR"]
        OP6["Respond to Reviews"]
        OP7["Approve"]
        OP8["Merge"]
    end
```

---

## 3. Data Architecture

### 3.1 Database Schema (PostgreSQL)

```mermaid
erDiagram
    TENANT ||--o{ WORKSPACE : has
    WORKSPACE ||--o{ REPOSITORY : contains
    WORKSPACE ||--o{ AGENT_SESSION : runs
    REPOSITORY ||--o{ AGENT_SESSION : targets
    AGENT_SESSION ||--o{ TASK : decomposes_into
    AGENT_SESSION ||--o{ REASONING_TRACE : logs
    AGENT_SESSION ||--o{ SANDBOX_EXECUTION : uses
    TASK ||--o{ TASK_DEPENDENCY : has
    AGENT_SESSION ||--o{ CODE_REVIEW : produces
    AGENT_SESSION ||--o{ PULL_REQUEST : creates
    PULL_REQUEST ||--o{ REVIEW_COMMENT : has
    PULL_REQUEST ||--o{ AIDD_APPROVAL : requires

    TENANT {
        uuid id PK
        string name
        string plan_tier
        jsonb settings
        timestamp created_at
    }

    WORKSPACE {
        uuid id PK
        uuid tenant_id FK
        string name
        jsonb git_connections
        timestamp created_at
    }

    REPOSITORY {
        uuid id PK
        uuid workspace_id FK
        string provider
        string owner
        string name
        string default_branch
        jsonb settings
    }

    AGENT_SESSION {
        uuid id PK
        uuid workspace_id FK
        uuid repository_id FK
        string prompt
        string status
        string agent_model
        integer iteration_count
        jsonb result
        timestamp started_at
        timestamp completed_at
    }

    TASK {
        uuid id PK
        uuid session_id FK
        string title
        string status
        integer order_index
        boolean parallelizable
        jsonb plan
    }

    REASONING_TRACE {
        uuid id PK
        uuid session_id FK
        integer step_number
        string action_type
        text reasoning
        jsonb tool_calls
        timestamp created_at
    }

    SANDBOX_EXECUTION {
        uuid id PK
        uuid session_id FK
        string container_id
        string image
        string status
        jsonb resource_usage
        jsonb filesystem_diff
        timestamp started_at
        timestamp terminated_at
    }

    CODE_REVIEW {
        uuid id PK
        uuid session_id FK
        string status
        jsonb findings
        float security_score
        float quality_score
        float coverage_delta
    }

    PULL_REQUEST {
        uuid id PK
        uuid session_id FK
        uuid repository_id FK
        string provider_pr_id
        string branch
        string status
        string url
    }

    AIDD_APPROVAL {
        uuid id PK
        uuid pull_request_id FK
        uuid approver_id
        string decision
        text reason
        timestamp decided_at
    }

    TASK_DEPENDENCY {
        uuid id PK
        uuid task_id FK
        uuid depends_on_task_id FK
    }

    REVIEW_COMMENT {
        uuid id PK
        uuid pull_request_id FK
        string file_path
        integer line_number
        text comment
        string severity
    }
```

### 3.2 Event-Driven Architecture

```mermaid
flowchart LR
    subgraph "Event Producers"
        AC["Agent Core"]
        GB["Git Bridge"]
        RE["Review Engine"]
        TP["Task Planner"]
        SR["Sandbox Runtime"]
    end

    subgraph "Redpanda / Kafka"
        T1["erp.autonomous_coding.session.started"]
        T2["erp.autonomous_coding.session.completed"]
        T3["erp.autonomous_coding.task.planned"]
        T4["erp.autonomous_coding.review.completed"]
        T5["erp.autonomous_coding.sandbox.created"]
        T6["erp.autonomous_coding.sandbox.destroyed"]
        T7["erp.autonomous_coding.pr.created"]
        T8["erp.autonomous_coding.pr.merged"]
        T9["erp.autonomous_coding.pr.approval_required"]
        T10["erp.autonomous_coding.webhook.received"]
    end

    subgraph "Event Consumers"
        DASH["Dashboard (SSE)"]
        AUDIT["Audit Service"]
        NOTIFY["Notification Service"]
        METRICS["Metrics Collector"]
    end

    AC --> T1
    AC --> T2
    TP --> T3
    RE --> T4
    SR --> T5
    SR --> T6
    GB --> T7
    GB --> T8
    GB --> T9
    GB --> T10

    T1 --> DASH
    T2 --> DASH
    T2 --> METRICS
    T4 --> DASH
    T7 --> NOTIFY
    T9 --> NOTIFY
    T8 --> METRICS
    T1 --> AUDIT
    T2 --> AUDIT
    T7 --> AUDIT
    T8 --> AUDIT
    T9 --> AUDIT
```

---

## 4. Inter-Service Communication

### 4.1 Communication Patterns

| Source | Target | Protocol | Pattern | Purpose |
|--------|--------|----------|---------|---------|
| API Gateway | Agent Core | gRPC | Request/Response | Session creation, status queries |
| Agent Core | Sandbox Runtime | gRPC | Request/Response | Container lifecycle, code execution |
| Agent Core | Git Bridge | gRPC | Request/Response | Clone, commit, push, PR operations |
| Agent Core | Review Engine | gRPC | Request/Response | Code review, SAST scanning |
| Agent Core | Task Planner | gRPC | Request/Response | Task decomposition |
| Git Bridge | Agent Core | Kafka | Event-driven | Webhook notifications (PR events, reviews) |
| IDE Server | Agent Core | WebSocket | Bidirectional stream | Real-time agent interaction |
| Dashboard | API Gateway | SSE | Server-push | Real-time session updates |
| All services | Kafka | Kafka | Pub/Sub | Event distribution |

### 4.2 Request Flow -- Autonomous Coding Session

```mermaid
sequenceDiagram
    participant User
    participant Gateway as API Gateway
    participant Agent as Agent Core
    participant Planner as Task Planner
    participant Sandbox as Sandbox Runtime
    participant Git as Git Bridge
    participant Review as Review Engine

    User->>Gateway: POST /v1/sessions {prompt, repo}
    Gateway->>Gateway: Validate JWT, check entitlements
    Gateway->>Agent: CreateSession(prompt, repo)
    Agent->>Git: CloneRepository(repo)
    Git-->>Agent: Local clone path
    Agent->>Planner: DecomposeTask(prompt, codebase)
    Planner-->>Agent: TaskPlan{tasks[], dependencies[]}

    loop For each task
        Agent->>Agent: Generate code via Claude API
        Agent->>Sandbox: Execute(code, tests, image)
        Sandbox-->>Agent: ExecutionResult{stdout, stderr, exit_code}
        alt Tests fail
            Agent->>Agent: Analyze failure, fix code
            Agent->>Sandbox: Re-execute(fixed_code, tests)
            Sandbox-->>Agent: ExecutionResult
        end
    end

    Agent->>Review: ReviewCode(diff)
    Review-->>Agent: ReviewResult{findings[], scores}
    Agent->>Git: CreateBranch, Commit, Push
    Agent->>Git: CreatePullRequest(branch, description)
    Git-->>Agent: PR URL
    Agent->>Agent: Emit session.completed event
    Agent-->>Gateway: SessionResult{pr_url, review, trace}
    Gateway-->>User: 200 OK {session_id, pr_url}
```

---

## 5. Deployment Architecture

### 5.1 Docker Compose (Development)

The development environment uses Docker Compose with 8 services as defined in `docker-compose.yml`:

| Service | Build Context | Exposed Port |
|---------|--------------|--------------|
| autonomous-coding-api | `.` | 8095 |
| agent-core | `./services/agent-core` | 8205 |
| review-engine | `./services/review-engine` | 8206 |
| ide-server | `./services/ide-server` | 8207 |
| task-planner | `./services/task-planner` | 8209 |
| sandbox-runtime | `./services/sandbox-runtime` | Internal |
| git-bridge | `./services/git-bridge` | Internal |
| redis | `redis:7` (image) | Internal |

### 5.2 Kubernetes (Production)

```mermaid
flowchart TB
    subgraph "Kubernetes Cluster"
        subgraph "Ingress"
            ING["NGINX Ingress Controller"]
        end

        subgraph "Application Namespace"
            GW["API Gateway<br/>Deployment (3 replicas)"]
            AC["Agent Core<br/>Deployment (5-20 replicas, HPA)"]
            GB["Git Bridge<br/>Deployment (3 replicas)"]
            IS["IDE Server<br/>Deployment (3-10 replicas, HPA)"]
            RE["Review Engine<br/>Deployment (3-10 replicas, HPA)"]
            TP["Task Planner<br/>Deployment (3 replicas)"]
            SR["Sandbox Runtime<br/>DaemonSet (all nodes)"]
            DASH["Dashboard<br/>Deployment (2 replicas)"]
        end

        subgraph "Data Namespace"
            PG["PostgreSQL<br/>StatefulSet (HA)"]
            RD["Redis<br/>StatefulSet (Sentinel)"]
            RP["Redpanda<br/>StatefulSet (3 brokers)"]
        end

        subgraph "Monitoring Namespace"
            PROM["Prometheus"]
            GRAF["Grafana"]
            JAEG["Jaeger"]
        end
    end

    ING --> GW
    GW --> AC
    GW --> GB
    GW --> IS
    GW --> RE
    GW --> TP
    AC --> SR
    AC --> PG
    AC --> RD
    AC --> RP
```

---

## 6. Technology Stack Summary

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| API Gateway | Go | 1.22 | Routing, auth, rate limiting |
| Agent Core | Python, FastAPI | 3.12 | Autonomous agent orchestration |
| Sandbox | Go, Docker Engine | 1.22 / 25.x | Container lifecycle |
| Git Bridge | Go | 1.22 | Git provider abstraction |
| IDE Server | TypeScript, Express | ES2022 | LSP bridge |
| Review Engine | Python, FastAPI | 3.12 | Code analysis |
| Task Planner | Python, FastAPI | 3.12 | Task decomposition |
| Dashboard | Next.js | 14 | Web UI |
| CLI | Go | 1.22 | Command-line tool |
| Database | PostgreSQL | 16 | Primary data store |
| Cache | Redis | 7 | Session state, rate limits |
| Event Bus | Redpanda/Kafka | Latest | Event streaming |
| LLM | Claude API (Anthropic) | Latest | AI reasoning engine |
| SAST | Snyk, Trivy | Latest | Vulnerability scanning |
| Secrets | TruffleHog | Latest | Secret detection |
| Container Runtime | Docker / gVisor | 25.x | Sandboxed execution |
| Observability | Prometheus, Grafana, Jaeger | Latest | Metrics, dashboards, tracing |
