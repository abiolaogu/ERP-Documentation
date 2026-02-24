# ERP-AI Technical Design Document

| Field | Value |
|---|---|
| Module | ERP-AI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Agent Orchestrator Design

### 1.1 Agent Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> Pending: spawn request
    Pending --> Initializing: resources allocated
    Initializing --> Ready: memory loaded
    Ready --> Running: task assigned
    Running --> Waiting: awaiting sub-agent
    Waiting --> Running: sub-agent completed
    Running --> Completed: task finished
    Running --> Failed: error occurred
    Failed --> Running: retry
    Failed --> Terminated: max retries
    Completed --> Terminated: cleanup
    Terminated --> [*]
```

### 1.2 DAG Execution Engine

Agent chains are defined as Directed Acyclic Graphs (DAGs):

```typescript
interface AgentDAG {
  id: string;
  nodes: AgentNode[];
  edges: AgentEdge[];
  errorPolicy: 'fail_fast' | 'continue' | 'retry';
  timeout: number;
}

interface AgentNode {
  id: string;
  agentId: string;
  version: string;
  input: Record<string, any>;
  timeout: number;
  retryPolicy: { maxRetries: number; backoff: 'exponential' | 'linear' };
}

interface AgentEdge {
  from: string;
  to: string;
  condition?: string; // Expression to evaluate
  dataMapping?: Record<string, string>; // Output->Input mapping
}
```

### 1.3 Agent Memory Architecture

```mermaid
graph TB
    subgraph "Agent Memory Layers"
        STM[Short-Term Memory<br/>Redis TTL 1h<br/>Current conversation]
        LTM[Long-Term Memory<br/>Qdrant Vectors<br/>Historical context]
        WM[Working Memory<br/>In-process<br/>Current task state]
    end

    Agent[Agent Pod] --> WM
    Agent --> STM
    Agent --> LTM
```

---

## 2. NLP Pipeline Design

### 2.1 Processing Pipeline

```mermaid
graph LR
    A[Raw Text Input] --> B[Language Detection]
    B --> C[Tokenization]
    C --> D[Intent Classification]
    D --> E[Entity Extraction]
    E --> F[Sentiment Analysis]
    F --> G{Action Required?}
    G -->|Translation| H[Translation Engine]
    G -->|Summarization| I[Summarization Engine]
    G -->|Generation| J[Text Generation via Claude]
    G -->|Complete| K[Return NLP Result]
```

### 2.2 Intent Classification

Multi-class classification across ERP domains:

| Domain | Example Intents |
|---|---|
| Finance | create_invoice, approve_expense, forecast_revenue |
| CRM | log_interaction, score_lead, schedule_followup |
| HCM | request_leave, submit_timesheet, find_candidate |
| SCM | create_po, check_inventory, track_shipment |
| General | search, summarize, translate, explain |

### 2.3 Entity Extraction

| Entity Type | Examples |
|---|---|
| PERSON | Employee names, customer names |
| ORG | Company names, department names |
| DATE | Absolute/relative dates, ranges |
| MONEY | Currency amounts with codes |
| PRODUCT | Product names, SKUs |
| LOCATION | Addresses, regions, countries |
| CUSTOM | Domain-specific entities per module |

---

## 3. ML Pipeline Design

### 3.1 Training Pipeline

```mermaid
graph TB
    A[Training Request] --> B[Feature Retrieval]
    B --> C[Data Validation]
    C --> D[Train/Test Split]
    D --> E[Model Training]
    E --> F[Evaluation]
    F --> G{Meets Threshold?}
    G -->|Yes| H[Register in Model Registry]
    G -->|No| I[Adjust Hyperparameters]
    I --> E
    H --> J[Shadow Deployment]
    J --> K[A/B Testing]
    K --> L{Winner?}
    L -->|Yes| M[Full Deployment]
    L -->|No| N[Rollback]
```

### 3.2 Model Registry Schema

```sql
CREATE TABLE models (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL,  -- classification, regression, embedding, etc.
    domain VARCHAR(50),          -- finance, crm, hcm, etc.
    tenant_id VARCHAR(50),
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE model_versions (
    id UUID PRIMARY KEY,
    model_id UUID REFERENCES models(id),
    version VARCHAR(20) NOT NULL,
    status VARCHAR(20) DEFAULT 'draft', -- draft, staging, production, archived
    metrics JSONB,           -- {accuracy: 0.95, f1: 0.92, ...}
    hyperparameters JSONB,
    artifact_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 3.3 Feature Store Design

```mermaid
graph LR
    subgraph "Feature Sources"
        ERP[ERP Module Events]
        BATCH[Batch Processing]
        STREAM[Real-time Stream]
    end

    subgraph "Feature Store"
        ONLINE[(Online Store<br/>Redis)]
        OFFLINE[(Offline Store<br/>PostgreSQL)]
    end

    subgraph "Consumers"
        TRAIN[Training Pipeline]
        INFER[Inference Service]
    end

    ERP --> STREAM --> ONLINE
    ERP --> BATCH --> OFFLINE
    OFFLINE --> TRAIN
    ONLINE --> INFER
```

---

## 4. Embedding Service Design

### 4.1 RAG Pipeline

```mermaid
sequenceDiagram
    participant User
    participant COP as Copilot Service
    participant EMB as Embedding Service
    participant QD as Qdrant
    participant CLAUDE as Claude API

    User->>COP: Ask question
    COP->>EMB: Generate query embedding
    EMB-->>COP: Query vector
    COP->>QD: Search similar vectors (top-k=5)
    QD-->>COP: Relevant documents
    COP->>CLAUDE: Question + relevant context
    CLAUDE-->>COP: Generated answer
    COP-->>User: Answer with citations
```

### 4.2 Embedding Models

| Content Type | Model | Dimensions | Metric |
|---|---|---|---|
| Documents | text-embedding-3-large | 1536 | Cosine |
| Code | code-search-ada-002 | 768 | Cosine |
| Images | CLIP ViT-L/14 | 512 | Cosine |

---

## 5. Guardrail Service Design

### 5.1 Policy Evaluation Engine

```mermaid
graph TB
    A[AI Action Request] --> B[Policy Lookup]
    B --> C[Context Assembly]
    C --> D{Classification}
    D -->|Autonomous| E[Execute Directly]
    D -->|Supervised| F[Queue for Human Review]
    D -->|Prohibited| G[Block + Log]

    E --> H[Post-execution Audit]
    F --> I{Human Approved?}
    I -->|Yes| J[Execute]
    I -->|No| K[Reject + Log]
    J --> H
    K --> H
    G --> H

    H --> L[NATS Audit Event]
```

### 5.2 Bias Detection

| Metric | Definition | Threshold |
|---|---|---|
| Demographic Parity | P(positive \| group_A) == P(positive \| group_B) | < 0.1 difference |
| Equal Opportunity | True positive rate equal across groups | < 0.05 difference |
| Predictive Parity | Precision equal across groups | < 0.05 difference |
| Disparate Impact | Ratio of selection rates | 0.8 - 1.25 |

---

## 6. Copilot Service Design

### 6.1 Context Assembly

```mermaid
graph LR
    A[User Request] --> B[Module Context]
    B --> C[User History]
    C --> D[Current Data State]
    D --> E[Similar Past Actions]
    E --> F[Relevant Documents RAG]
    F --> G[Assembled Prompt]
    G --> H[Claude API]
    H --> I[Response]
    I --> J[Guardrail Check]
    J --> K[Return to User]
```

### 6.2 Suggestion Types

| Type | Trigger | Latency Target |
|---|---|---|
| Inline suggestion | Keystroke + 300ms debounce | < 500ms |
| Auto-complete | Field focus | < 200ms |
| Smart default | Form open | < 100ms |
| Next-best-action | Page load | < 1s |
| Anomaly explanation | Anomaly detected | < 2s |

---

## 7. Error Handling

| Error | Service | Handling |
|---|---|---|
| Claude API timeout | NLP, Copilot | Retry 1x, return cached/fallback |
| Qdrant unreachable | Embedding, Orchestrator | Degrade gracefully, skip memory |
| Agent pod crash | Orchestrator | Auto-restart, retry task |
| Model serving failure | ML Pipeline | Fallback to previous version |
| Guardrail service down | All | Default to supervised mode |
