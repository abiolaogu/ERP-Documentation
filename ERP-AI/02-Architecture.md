# ERP-AI Architecture Document

| Field | Value |
|---|---|
| Module | ERP-AI (AI Intelligence Layer) |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Architecture Overview

ERP-AI follows a microservices architecture with seven core services, integrating Anthropic Claude as the primary LLM, Qdrant for vector storage, and Kubernetes for agent container orchestration. The module serves as the AI backbone for all ERP modules.

```mermaid
graph TB
    subgraph "Consumer Modules"
        BI[ERP-BI]
        FIN[ERP-Finance]
        CRM[ERP-CRM]
        HCM[ERP-HCM]
        SCM[ERP-SCM]
        COM[ERP-Commerce]
        MKT[ERP-Marketing]
    end

    subgraph "API Gateway"
        GW[Gateway<br/>Authentication + Routing]
    end

    subgraph "ERP-AI Service Mesh"
        AO[Agent Orchestrator<br/>Go :8080]
        AC[Agent Catalog<br/>Go :8081]
        NLP[NLP Service<br/>Go :8082]
        ML[ML Pipeline<br/>Go :8083]
        EMB[Embedding Service<br/>Go :8084]
        GR[Guardrail Service<br/>Go :8085]
        COP[Copilot Service<br/>Go :8086]
    end

    subgraph "AI Runtime"
        K8S[Kubernetes<br/>Agent Pods]
        CLAUDE[Anthropic Claude<br/>API]
    end

    subgraph "Data Stores"
        QDRANT[(Qdrant<br/>Vector DB)]
        PG[(PostgreSQL<br/>Metadata)]
        REDIS[(Redis<br/>Cache + State)]
    end

    subgraph "Platform"
        NATS[NATS JetStream]
        IAM[ERP-IAM]
        PLAT[ERP-Platform]
    end

    BI & FIN & CRM & HCM & SCM & COM & MKT --> GW
    GW --> AO & AC & NLP & ML & EMB & GR & COP

    AO --> K8S
    AO --> AC
    AO --> GR
    COP --> AO & NLP & EMB
    NLP --> CLAUDE
    COP --> CLAUDE
    EMB --> QDRANT
    AO --> QDRANT & PG & REDIS
    ML --> PG & REDIS
    AC --> PG
    GR --> PG & NATS
    AO --> NATS
    GW --> IAM & PLAT
```

---

## 2. Service Decomposition

### 2.1 Agent Orchestrator

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/agent-orchestrator` |
| Port | 8080 |
| Health Check | `/healthz` |
| Event Topics | `erp.ai.agent-orchestrator.*` |

**Responsibilities**: Agent lifecycle management (spawn/route/chain/parallelize/terminate), multi-agent collaboration, DAG execution, agent memory management via Qdrant, agent versioning, health monitoring, performance metrics.

```mermaid
graph LR
    subgraph "Agent Lifecycle"
        A[Spawn] --> B[Route]
        B --> C[Execute]
        C --> D{Chain?}
        D -->|Yes| E[Next Agent]
        D -->|No| F[Return Result]
        E --> C
        C --> G{Error?}
        G -->|Yes| H[Retry / Fallback]
        G -->|No| F
        F --> I[Terminate]
    end
```

### 2.2 Agent Catalog

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/agent-catalog` |
| Port | 8081 |

**Responsibilities**: Agent registration, discovery by capability/domain/task, health dashboard, performance metrics aggregation, category management (29 business categories).

**Agent Categories**: Finance, CRM, HCM, SCM, Commerce, Marketing, Healthcare, Education, Legal, Compliance, Project Management, Customer Support, Sales, Procurement, Manufacturing, Quality, Logistics, Warehouse, IT Operations, Security, Data Analytics, Content, Communication, Scheduling, Document Processing, Risk Assessment, Forecasting, Reporting, Administration.

### 2.3 NLP Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/nlp` |
| Port | 8082 |

**Responsibilities**: Intent classification, named entity extraction, sentiment analysis, language detection (100+ languages), translation, summarization, text generation via Claude.

### 2.4 ML Pipeline Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/ml-pipeline` |
| Port | 8083 |

**Responsibilities**: Model training orchestration, evaluation (precision/recall/F1/AUC), deployment (shadow/canary/full), monitoring (accuracy/drift/latency), automated retraining, feature store, model registry with A/B testing.

### 2.5 Embedding Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/embedding` |
| Port | 8084 |

**Responsibilities**: Vector generation (document/code/image), Qdrant index management, semantic search, RAG pipeline, embedding model management.

### 2.6 Guardrail Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/guardrail` |
| Port | 8085 |

**Responsibilities**: AIDD policy evaluation, action classification (autonomous/supervised/prohibited), human-in-the-loop workflow, bias detection, fairness monitoring, explainability report generation, compliance audit trail.

### 2.7 Copilot Service

| Attribute | Value |
|---|---|
| Language | Go 1.22 |
| Base Path | `/v1/copilot` |
| Port | 8086 |

**Responsibilities**: Module-embedded AI assistance, inline suggestions, context-aware auto-complete, smart defaults, predictive actions (next-best-action), anomaly explanations, conversation management.

---

## 3. Agent Architecture

### 3.1 Agent Execution Model

```mermaid
sequenceDiagram
    participant Client as Consumer Module
    participant AO as Agent Orchestrator
    participant GR as Guardrail Service
    participant AC as Agent Catalog
    participant K8S as Kubernetes
    participant Agent as Agent Pod
    participant QD as Qdrant (Memory)

    Client->>AO: POST /v1/agent-orchestrator (task)
    AO->>GR: Classify action
    GR-->>AO: Classification (autonomous/supervised)
    AO->>AC: Find best agent for task
    AC-->>AO: Agent definition
    AO->>QD: Load agent memory (context)
    QD-->>AO: Historical context
    AO->>K8S: Spawn agent pod
    K8S-->>Agent: Container started
    Agent->>Agent: Execute task with context
    Agent->>QD: Store results in memory
    Agent-->>AO: Task result
    AO->>GR: Audit log
    AO-->>Client: Response
```

### 3.2 Multi-Agent Collaboration

```mermaid
graph TB
    subgraph "Agent Chain"
        A1[Lead Scoring Agent] --> A2[Email Campaign Agent]
        A2 --> A3[Social Media Agent]
    end

    subgraph "Parallel Execution"
        B1[Competitor Analysis Agent]
        B2[SEO Agent]
        B3[Brand Voice Agent]
        B1 & B2 & B3 --> AGG[Result Aggregator]
    end

    subgraph "Routing"
        R[Router Agent] --> |CRM task| C1[CRM Data Entry Agent]
        R --> |Meeting task| C2[Meeting Scheduling Agent]
        R --> |Proposal task| C3[Proposal Generation Agent]
    end
```

---

## 4. Data Architecture

### 4.1 Vector Storage (Qdrant)

| Collection | Content | Dimensions | Index |
|---|---|---|---|
| agent_memory | Agent conversation history | 1536 | HNSW |
| document_embeddings | ERP document vectors | 1536 | HNSW |
| code_embeddings | Source code vectors | 768 | HNSW |
| image_embeddings | Image/diagram vectors | 512 | HNSW |

### 4.2 PostgreSQL Schema

| Table | Purpose |
|---|---|
| agents | Agent definitions and metadata |
| agent_versions | Version history per agent |
| agent_executions | Execution log with metrics |
| models | ML model registry |
| model_versions | Model version history |
| model_metrics | Training/evaluation metrics |
| features | Feature store definitions |
| feature_values | Feature value snapshots |
| guardrail_policies | AIDD policy definitions |
| audit_log | Compliance audit trail |

---

## 5. Integration Architecture

```mermaid
graph LR
    subgraph "ERP Modules"
        M1[ERP-BI] -->|NLQ| COP
        M2[ERP-CRM] -->|Lead Scoring| AO
        M3[ERP-Finance] -->|Anomaly| AO
        M4[ERP-HCM] -->|Candidate Matching| AO
        M5[ERP-SCM] -->|Demand Forecast| AO
        M6[ERP-Marketing] -->|Content Gen| COP
    end

    COP[Copilot Service]
    AO[Agent Orchestrator]
```

---

## 6. Deployment Architecture

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "ai-system namespace"
            AO_D[Agent Orchestrator x3]
            AC_D[Agent Catalog x2]
            NLP_D[NLP Service x3]
            ML_D[ML Pipeline x2]
            EMB_D[Embedding Service x3]
            GR_D[Guardrail Service x2]
            COP_D[Copilot Service x4]
        end

        subgraph "ai-agents namespace"
            AP[Agent Pods<br/>Dynamic scaling]
        end

        subgraph "ai-data namespace"
            QD_S[Qdrant StatefulSet x3]
            PG_S[PostgreSQL StatefulSet x2]
            RD_S[Redis StatefulSet x3]
        end
    end

    subgraph "External"
        CLAUDE_E[Anthropic Claude API]
    end
```

---

## 7. Security Architecture

| Layer | Mechanism |
|---|---|
| Authentication | JWT from ERP-IAM, X-Tenant-ID |
| Authorization | RBAC per agent category |
| Agent isolation | Per-tenant Kubernetes namespaces |
| Model isolation | Separate model artifacts per tenant |
| Data isolation | Qdrant collection partitioning |
| LLM safety | Guardrail Service pre/post filtering |
| Audit | Full trail via NATS events |
| Compliance | AIDD guardrails enforcement |
