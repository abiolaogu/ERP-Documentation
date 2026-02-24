# ERP-AI Data Flow Documentation

| Field | Value |
|---|---|
| Module | ERP-AI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. End-to-End Data Flow

```mermaid
graph TB
    subgraph "Consumer Modules"
        M1[ERP-BI<br/>NLQ Requests]
        M2[ERP-CRM<br/>Lead Scoring]
        M3[ERP-Finance<br/>Anomaly Detection]
        M4[ERP-HCM<br/>Candidate Matching]
        M5[ERP-Marketing<br/>Content Generation]
    end

    subgraph "API Gateway"
        GW[Auth + Routing]
    end

    subgraph "Copilot Path"
        COP[Copilot Service]
        CTX[Context Assembly]
        RAG[RAG Pipeline]
    end

    subgraph "Agent Path"
        AO[Agent Orchestrator]
        AC[Agent Catalog]
        GR[Guardrail Service]
        AP[Agent Pods]
    end

    subgraph "NLP Path"
        NLP[NLP Service]
        CLAUDE[Claude API]
    end

    subgraph "ML Path"
        ML[ML Pipeline]
        FS[Feature Store]
        MR[Model Registry]
    end

    subgraph "Data Stores"
        QD[(Qdrant)]
        PG[(PostgreSQL)]
        RD[(Redis)]
    end

    M1 & M2 & M3 & M4 & M5 --> GW
    GW --> COP & AO

    COP --> CTX --> RAG
    RAG --> QD
    COP --> NLP --> CLAUDE

    AO --> GR
    AO --> AC --> PG
    AO --> AP
    AP --> QD & RD

    ML --> FS --> RD
    ML --> MR --> PG

    AO --> NATS[NATS Events]
    GR --> NATS
```

---

## 2. Copilot Request Flow

```mermaid
sequenceDiagram
    participant User
    participant Module as ERP Module
    participant COP as Copilot Service
    participant EMB as Embedding Service
    participant QD as Qdrant
    participant NLP as NLP Service
    participant CLAUDE as Claude API
    participant GR as Guardrail Service

    User->>Module: Type in form field
    Module->>COP: POST /v1/copilot (context + partial input)
    COP->>COP: Assemble context (module, page, field, history)
    COP->>EMB: Generate query embedding
    EMB->>QD: Semantic search (top-k=5)
    QD-->>EMB: Relevant context documents
    EMB-->>COP: Context results
    COP->>NLP: Classify intent
    NLP-->>COP: Intent + entities
    COP->>CLAUDE: Prompt (context + intent + relevant docs)
    CLAUDE-->>COP: Suggestion
    COP->>GR: Validate suggestion
    GR-->>COP: Approved
    COP-->>Module: Suggestion(s)
    Module-->>User: Display inline suggestion
```

---

## 3. Agent Execution Flow

```mermaid
sequenceDiagram
    participant Client
    participant AO as Agent Orchestrator
    participant GR as Guardrail Service
    participant AC as Agent Catalog
    participant K8S as Kubernetes
    participant Agent as Agent Pod
    participant QD as Qdrant (Memory)
    participant NATS as NATS

    Client->>AO: POST /v1/agent-orchestrator
    AO->>GR: Classify action
    GR-->>AO: supervised
    AO->>AC: Lookup agent by capability
    AC-->>AO: Agent definition + version
    AO->>QD: Load agent long-term memory
    QD-->>AO: Historical context vectors
    AO->>K8S: Spawn agent pod
    K8S->>Agent: Start container
    Agent->>Agent: Execute task
    Agent->>QD: Store new memories
    Agent-->>AO: Result
    AO->>GR: Post-execution audit
    AO->>NATS: Publish execution event
    AO-->>Client: Response
```

---

## 4. ML Training Flow

```mermaid
sequenceDiagram
    participant Dev as ML Developer
    participant ML as ML Pipeline Service
    participant FS as Feature Store (Redis)
    participant PG as PostgreSQL
    participant K8S as Kubernetes
    participant MR as Model Registry

    Dev->>ML: POST /v1/ml-pipeline (train request)
    ML->>FS: Retrieve features for dataset
    FS-->>ML: Feature vectors
    ML->>K8S: Launch training job
    K8S->>K8S: Train model
    K8S-->>ML: Training complete + metrics
    ML->>PG: Evaluate (accuracy, F1, AUC)
    ML->>MR: Register model version
    MR->>PG: Store version + metrics
    ML-->>Dev: Training report

    Note over ML,K8S: Deployment follows
    Dev->>ML: Deploy (canary 10%)
    ML->>K8S: Deploy model pod (canary)
    ML->>ML: Monitor A/B metrics
    ML->>K8S: Promote to full or rollback
```

---

## 5. Guardrail Audit Flow

```mermaid
sequenceDiagram
    participant AI as AI Service
    participant GR as Guardrail Service
    participant PG as PostgreSQL
    participant NATS as NATS
    participant DASH as Compliance Dashboard

    AI->>GR: Evaluate action
    GR->>PG: Load applicable policies
    PG-->>GR: Policy rules
    GR->>GR: Classify action
    GR->>PG: Write audit_log entry
    GR->>NATS: Publish erp.ai.aidd.audit
    NATS->>DASH: Audit event
    GR-->>AI: Classification result
```

---

## 6. Embedding Indexing Flow

```mermaid
graph LR
    A[Document Upload] --> B[Chunking<br/>512 tokens]
    B --> C[Embedding Generation<br/>Claude/OpenAI]
    C --> D[Qdrant Upsert]
    D --> E[Metadata Index]

    F[Search Query] --> G[Query Embedding]
    G --> H[Qdrant ANN Search]
    H --> I[Re-ranking]
    I --> J[Results]
```
