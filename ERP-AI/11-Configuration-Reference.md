# ERP-AI Configuration Reference

| Field | Value |
|---|---|
| Module | ERP-AI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Core Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| PORT | int | 8080 | HTTP server port |
| MODULE_NAME | string | ERP-AI | Module identifier |
| LOG_LEVEL | string | info | debug/info/warn/error |
| ENV | string | production | development/staging/production |

## 2. Claude API Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| ANTHROPIC_API_KEY | string | - | API key for Claude |
| CLAUDE_MODEL | string | claude-3-5-sonnet-20241022 | Default model |
| CLAUDE_MAX_TOKENS | int | 4096 | Max response tokens |
| CLAUDE_TEMPERATURE | float | 0.7 | Generation temperature |
| CLAUDE_TIMEOUT_MS | int | 30000 | Request timeout |

## 3. Qdrant Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| QDRANT_URL | string | - | Qdrant gRPC endpoint |
| QDRANT_API_KEY | string | - | API key (if auth enabled) |
| QDRANT_COLLECTION_PREFIX | string | erp_ | Collection name prefix |
| EMBEDDING_DIMENSIONS | int | 1536 | Vector dimensions |
| SEARCH_TOP_K | int | 5 | Default result count |

## 4. Agent Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| AGENT_NAMESPACE | string | ai-agents | K8s namespace for agent pods |
| AGENT_MAX_CONCURRENT | int | 100 | Max concurrent agents per tenant |
| AGENT_TIMEOUT_S | int | 60 | Default agent execution timeout |
| AGENT_RETRY_MAX | int | 3 | Max retries on failure |
| AGENT_MEMORY_TTL_H | int | 24 | Short-term memory TTL (hours) |

## 5. ML Pipeline Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| ML_TRAINING_NAMESPACE | string | ai-training | K8s namespace for training jobs |
| ML_MAX_TRAINING_TIME_H | int | 24 | Max training duration |
| ML_CANARY_PERCENTAGE | int | 10 | Default canary deployment % |
| ML_ACCURACY_THRESHOLD | float | 0.8 | Min accuracy for deployment |
| FEATURE_STORE_TTL_M | int | 15 | Online feature cache TTL |

## 6. Guardrail Configuration

```yaml
guardrails:
  default_classification: supervised
  autonomous_threshold: 0.95  # Confidence for auto-approval
  human_review_timeout: 300   # Seconds before escalation
  prohibited_actions:
    - delete_production_data
    - modify_financial_records
    - access_cross_tenant_data
    - disable_audit_logging
  bias_detection:
    enabled: true
    evaluation_frequency: daily
    threshold_demographic_parity: 0.1
    threshold_disparate_impact: 0.8
  audit:
    retention_years: 7
    log_input: true
    log_output: true
    hash_pii: true
```

## 7. Rate Limits

| Tier | Copilot req/min | Agent exec/min | NLP req/min | Embeddings/min |
|---|---|---|---|---|
| Free | 30 | 5 | 20 | 10 |
| Professional | 300 | 50 | 200 | 100 |
| Enterprise | 1000 | 200 | 1000 | 500 |
| Unlimited | No limit | No limit | No limit | No limit |
