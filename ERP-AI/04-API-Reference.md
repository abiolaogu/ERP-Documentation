# ERP-AI API Reference

| Field | Value |
|---|---|
| Module | ERP-AI |
| Base URL | `https://api.erp.example.com/ai` |
| Version | v1 |
| Auth | Bearer JWT + X-Tenant-ID header |
| Last Updated | 2026-02-23 |

---

## 1. Authentication

All requests require:
- `Authorization: Bearer <jwt_token>` from ERP-IAM
- `X-Tenant-ID: <tenant_id>` -- tenant context

---

## 2. Agent Orchestrator API

### 2.1 List Agent Executions
```
GET /v1/agent-orchestrator
```

### 2.2 Execute Agent Task
```
POST /v1/agent-orchestrator
```

**Request**:
```json
{
  "task": "Score and prioritize leads from last week's campaign",
  "domain": "crm",
  "agents": ["lead-scoring-agent"],
  "context": {
    "campaign_id": "camp_123",
    "date_range": "2026-02-16/2026-02-23"
  },
  "chain": ["lead-scoring-agent", "email-campaign-agent"],
  "timeout": 30
}
```

**Response** `201 Created`:
```json
{
  "item": {
    "execution_id": "exec_abc123",
    "status": "running",
    "agents": ["lead-scoring-agent"],
    "started_at": "2026-02-23T10:00:00Z"
  },
  "event_topic": "erp.ai.agent-orchestrator.created"
}
```

### 2.3 Get Execution Status
```
GET /v1/agent-orchestrator/{id}
```

### 2.4 Terminate Execution
```
DELETE /v1/agent-orchestrator/{id}
```

---

## 3. Agent Catalog API

### 3.1 List Agents
```
GET /v1/agent-catalog
```

**Query Parameters**:
| Param | Type | Description |
|---|---|---|
| domain | string | Filter by business domain (crm, finance, etc.) |
| capability | string | Filter by capability |
| status | string | Filter by health status |
| page | int | Page number |
| limit | int | Items per page |

### 3.2 Register Agent
```
POST /v1/agent-catalog
```

**Request**:
```json
{
  "name": "Invoice Anomaly Detector",
  "domain": "finance",
  "capabilities": ["anomaly_detection", "fraud_detection"],
  "version": "1.0.0",
  "runtime": "python:3.11",
  "config": {
    "model": "claude-3-5-sonnet",
    "memory_collection": "finance_anomalies"
  }
}
```

### 3.3 Get Agent Details
```
GET /v1/agent-catalog/{id}
```

### 3.4 Update Agent
```
PUT /v1/agent-catalog/{id}
```

### 3.5 Deregister Agent
```
DELETE /v1/agent-catalog/{id}
```

---

## 4. NLP Service API

### 4.1 Analyze Text
```
POST /v1/nlp
```

**Request**:
```json
{
  "text": "I need to schedule a meeting with the finance team next Tuesday to discuss Q1 budget overruns",
  "operations": ["intent", "entities", "sentiment", "language"]
}
```

**Response** `201 Created`:
```json
{
  "item": {
    "intent": {"label": "schedule_meeting", "confidence": 0.94},
    "entities": [
      {"text": "finance team", "type": "ORG", "confidence": 0.92},
      {"text": "next Tuesday", "type": "DATE", "value": "2026-02-24", "confidence": 0.98},
      {"text": "Q1 budget overruns", "type": "TOPIC", "confidence": 0.87}
    ],
    "sentiment": {"label": "neutral", "score": 0.05},
    "language": {"code": "en", "name": "English", "confidence": 0.99}
  },
  "event_topic": "erp.ai.nlp.created"
}
```

### 4.2 Translate
```
POST /v1/nlp
```

**Request**:
```json
{
  "text": "Invoice #1234 is overdue by 15 days",
  "operations": ["translate"],
  "targetLanguage": "es"
}
```

### 4.3 Summarize
```
POST /v1/nlp
```

**Request**:
```json
{
  "text": "<long document text>",
  "operations": ["summarize"],
  "maxLength": 200
}
```

---

## 5. ML Pipeline API

### 5.1 List Models
```
GET /v1/ml-pipeline
```

### 5.2 Train Model
```
POST /v1/ml-pipeline
```

**Request**:
```json
{
  "action": "train",
  "model_name": "lead_scoring_v2",
  "type": "classification",
  "dataset": "crm_leads_2025",
  "hyperparameters": {
    "learning_rate": 0.01,
    "epochs": 100,
    "batch_size": 32
  },
  "evaluation": {
    "metrics": ["accuracy", "f1", "auc"],
    "test_split": 0.2
  }
}
```

### 5.3 Deploy Model
```
PUT /v1/ml-pipeline/{id}
```

**Request**:
```json
{
  "action": "deploy",
  "strategy": "canary",
  "canary_percentage": 10,
  "success_threshold": {"accuracy": 0.9}
}
```

---

## 6. Embedding Service API

### 6.1 Generate Embeddings
```
POST /v1/embedding
```

**Request**:
```json
{
  "action": "embed",
  "content": "Quarterly financial report showing 15% revenue growth",
  "content_type": "document",
  "collection": "finance_docs",
  "metadata": {"department": "finance", "quarter": "Q4-2025"}
}
```

### 6.2 Semantic Search
```
POST /v1/embedding
```

**Request**:
```json
{
  "action": "search",
  "query": "revenue growth trends",
  "collection": "finance_docs",
  "top_k": 5,
  "filters": {"department": "finance"}
}
```

**Response**:
```json
{
  "item": {
    "results": [
      {"id": "doc_123", "score": 0.94, "text": "Q4 revenue grew 15%...", "metadata": {}},
      {"id": "doc_456", "score": 0.87, "text": "Annual revenue summary...", "metadata": {}}
    ]
  }
}
```

---

## 7. Guardrail Service API

### 7.1 Evaluate Action
```
POST /v1/guardrail
```

**Request**:
```json
{
  "action": "auto_approve_expense",
  "agent_id": "expense-approval-agent",
  "context": {
    "amount": 15000,
    "currency": "USD",
    "category": "travel"
  }
}
```

**Response**:
```json
{
  "item": {
    "classification": "supervised",
    "reason": "Expense amount exceeds $10,000 autonomous threshold",
    "required_approver": "manager",
    "policy_id": "pol_expense_001"
  }
}
```

---

## 8. Copilot Service API

### 8.1 Get Suggestions
```
POST /v1/copilot
```

**Request**:
```json
{
  "module": "erp-crm",
  "context": {
    "page": "lead_form",
    "field": "company_name",
    "partial_input": "Acm",
    "form_data": {"industry": "technology"}
  },
  "type": "autocomplete"
}
```

**Response**:
```json
{
  "item": {
    "suggestions": [
      {"text": "Acme Corporation", "confidence": 0.95},
      {"text": "Acme Technologies", "confidence": 0.82}
    ],
    "source": "historical_data"
  }
}
```

---

## 9. Health Check

All services:
```
GET /healthz
```

Response:
```json
{"status": "healthy", "module": "ERP-AI", "service": "agent-orchestrator"}
```

---

## 10. Capabilities Endpoint

```
GET /v1/capabilities
```

Response:
```json
{
  "module": "ERP-AI",
  "capabilities": ["agent_orchestration", "copilot", "nlq", "workflow_automation", "risk_scoring"]
}
```
