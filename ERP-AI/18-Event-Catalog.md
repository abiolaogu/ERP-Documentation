# ERP-AI Event Catalog

| Field | Value |
|---|---|
| Module | ERP-AI |
| Backbone | NATS JetStream |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Published Events

### 1.1 Agent Events

| Event | Payload | Publisher |
|---|---|---|
| erp.ai.agent-orchestrator.created | `{execution_id, agent_id, task, tenant_id}` | Orchestrator |
| erp.ai.agent-orchestrator.completed | `{execution_id, result, duration_ms, tokens}` | Orchestrator |
| erp.ai.agent-orchestrator.failed | `{execution_id, error, retries}` | Orchestrator |
| erp.ai.agent-orchestrator.terminated | `{execution_id, reason}` | Orchestrator |

### 1.2 Agent Catalog Events

| Event | Payload |
|---|---|
| erp.ai.agent-catalog.created | `{agent_id, name, domain, version}` |
| erp.ai.agent-catalog.updated | `{agent_id, changes}` |
| erp.ai.agent-catalog.deleted | `{agent_id}` |

### 1.3 NLP Events

| Event | Payload |
|---|---|
| erp.ai.nlp.created | `{request_id, operations, language, tenant_id}` |
| erp.ai.nlp.completed | `{request_id, results, duration_ms}` |

### 1.4 ML Pipeline Events

| Event | Payload |
|---|---|
| erp.ai.ml-pipeline.training.started | `{model_id, version, dataset_size}` |
| erp.ai.ml-pipeline.training.completed | `{model_id, version, metrics}` |
| erp.ai.ml-pipeline.deployed | `{model_id, version, strategy}` |
| erp.ai.ml-pipeline.drift.detected | `{model_id, drift_score, recommendation}` |

### 1.5 Embedding Events

| Event | Payload |
|---|---|
| erp.ai.embedding.indexed | `{document_id, collection, vector_count}` |
| erp.ai.embedding.searched | `{query_hash, results_count, latency_ms}` |

### 1.6 Guardrail Events

| Event | Payload |
|---|---|
| erp.ai.guardrail.evaluated | `{action, classification, policy_id}` |
| erp.ai.guardrail.blocked | `{action, reason, agent_id}` |
| erp.ai.guardrail.bias.detected | `{model_id, metric, value, threshold}` |
| erp.ai.aidd.audit | `{full audit record}` |

### 1.7 Copilot Events

| Event | Payload |
|---|---|
| erp.ai.copilot.suggestion | `{module, type, accepted, latency_ms}` |
| erp.ai.copilot.conversation | `{session_id, turns, duration_ms}` |

---

## 2. Consumed Events

| Source | Events | Purpose |
|---|---|---|
| ERP-Platform | `erp.platform.tenant.*` | Tenant lifecycle |
| ERP modules | `erp.{module}.{entity}.*` | Training data, feature updates |
| ERP-BI | `erp.bi.nlq.created` | NLQ request handling |
