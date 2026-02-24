# ERP-AI Security Architecture

| Field | Value |
|---|---|
| Module | ERP-AI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Security Model Overview

```mermaid
graph TB
    subgraph "Perimeter"
        WAF[WAF + Rate Limiting]
        LB[Load Balancer + TLS]
    end

    subgraph "Authentication"
        JWT[JWT Validation via ERP-IAM]
        TENANT[Tenant ID Enforcement]
    end

    subgraph "AI Safety"
        GR[Guardrail Service]
        AIDD[AIDD Policy Engine]
        HITL[Human-in-the-Loop]
    end

    subgraph "Data Safety"
        PII[PII Detection + Masking]
        PROMPT[Prompt Injection Defense]
        OUTPUT[Output Filtering]
    end

    subgraph "Infrastructure"
        MTLS[mTLS Between Services]
        NET[Network Policies]
        ENC[AES-256 At-Rest]
    end

    WAF --> LB --> JWT --> TENANT
    TENANT --> GR --> AIDD
    AIDD --> HITL
    GR --> PII & PROMPT & OUTPUT
```

---

## 2. LLM Security

### 2.1 Prompt Injection Prevention
- System prompts are hardcoded, never user-modifiable
- User input is sanitized and placed in designated user message blocks
- Output is validated against expected format before returning
- Content filtering blocks harmful/toxic outputs

### 2.2 PII Protection
- PII is detected and redacted before sending to Claude API
- Redacted PII is re-injected after response generation
- PII patterns: SSN, credit card, email, phone, address

### 2.3 Data Leakage Prevention
- Tenant data never sent to LLM without explicit policy approval
- Cross-tenant context mixing is architecturally impossible
- All LLM interactions logged in audit trail

---

## 3. Agent Security

### 3.1 Agent Isolation
- Each agent runs in its own Kubernetes pod
- Network policies restrict agent-to-agent communication
- Agents only access their designated Qdrant collections
- Agent credentials are injected via Kubernetes secrets

### 3.2 Agent Permissions
```mermaid
graph TB
    A[Agent Execution Request] --> B[Guardrail Classification]
    B --> C{Classification}
    C -->|Autonomous| D[Execute with Read-Only Access]
    C -->|Supervised| E[Queue for Human Approval]
    C -->|Prohibited| F[Block Execution]
    D --> G[Scoped API Access Only]
    E --> H{Approved?}
    H -->|Yes| I[Execute with Requested Access]
    H -->|No| J[Reject]
```

---

## 4. Audit Trail

All AI actions are logged with 7-year retention:

```json
{
  "event_type": "erp.ai.aidd.audit",
  "action": "agent_execution",
  "classification": "supervised",
  "agent_id": "lead-scoring-agent",
  "user_id": "user_123",
  "tenant_id": "tenant_001",
  "input_hash": "sha256:abc...",
  "output_hash": "sha256:def...",
  "approved": true,
  "approver_id": "manager_456",
  "duration_ms": 2340,
  "tokens_used": 1500,
  "timestamp": "2026-02-23T10:00:00Z"
}
```

---

## 5. Model Security

| Concern | Mitigation |
|---|---|
| Model theft | Encrypted artifact storage, access logging |
| Training data poisoning | Data validation, provenance tracking |
| Adversarial inputs | Input validation, anomaly detection |
| Model bias | Bias detection, fairness monitoring |
| Model drift | Continuous monitoring, auto-retraining triggers |
