# ERP-AI Developer Guide

| Field | Value |
|---|---|
| Module | ERP-AI |
| Audience | Developers |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Development Environment Setup

### 1.1 Prerequisites

```bash
go >= 1.22
python >= 3.11  # For legacy agents
docker >= 24.0
kubectl >= 1.28
```

### 1.2 Clone and Build

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-AI

# Build a service
cd services/copilot-service
go build -o copilot-service main.go

# Run locally
PORT=8086 MODULE_NAME=ERP-AI ./copilot-service
```

---

## 2. Project Structure

```
ERP-AI/
├── cmd/
│   └── server/              # Server entry point
├── configs/
│   └── capabilities.json    # Module capabilities
├── erp/
│   └── module.manifest.yaml # ERP manifest
├── services/                # Go microservices
│   ├── agent-orchestrator/
│   │   ├── Dockerfile
│   │   ├── README.md
│   │   └── main.go
│   ├── agent-catalog/
│   ├── nlp-service/
│   ├── ml-pipeline-service/
│   ├── embedding-service/
│   ├── guardrail-service/
│   └── copilot-service/
├── imports/
│   ├── ai_legacy/           # Legacy agent code (Python)
│   │   ├── packages/
│   │   │   ├── agent_framework/
│   │   │   ├── integration_framework/
│   │   │   ├── vector_memory/
│   │   │   ├── multi_framework/
│   │   │   └── performance/
│   │   ├── services/
│   │   │   ├── lead_scoring_agent/
│   │   │   ├── email_campaign_agent/
│   │   │   ├── social_media_agent/
│   │   │   ├── brand_voice_consistency_agent/
│   │   │   ├── proposal_generation_agent/
│   │   │   ├── seo_agent/
│   │   │   ├── crm_data_entry_agent/
│   │   │   ├── meeting_scheduling_agent/
│   │   │   └── competitor_analysis_agent/
│   │   └── web/
│   └── ai_core/             # Core AI library
├── docs/
├── go.mod
├── Makefile
└── README.md
```

---

## 3. Service Development Pattern

Every Go service follows the same pattern:

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "os"
    "strings"
)

type payload map[string]any

func main() {
    port := os.Getenv("PORT")
    if port == "" { port = "8080" }

    mux := http.NewServeMux()
    mux.HandleFunc("/healthz", healthHandler)
    mux.HandleFunc("/v1/{resource}", collectionHandler)
    mux.HandleFunc("/v1/{resource}/", itemHandler)

    log.Fatal(http.ListenAndServe(":"+port, mux))
}
```

**Key patterns**:
- X-Tenant-ID validation on every request
- JSON response with event_topic field
- Health check at /healthz
- CRUD operations via HTTP methods

---

## 4. Building a Custom Agent

### 4.1 Agent Framework (Python Legacy)

```python
from agent_framework import BaseAgent

class CustomAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            name="custom-agent",
            domain="finance",
            capabilities=["anomaly_detection"]
        )

    async def execute(self, task, context):
        # Agent logic here
        result = await self.process(task, context)
        await self.store_memory(result)
        return result
```

### 4.2 Agent Docker Packaging

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "-m", "agent"]
```

### 4.3 Registering in Catalog

```bash
curl -X POST http://localhost:8081/v1/agent-catalog \
  -H "X-Tenant-ID: dev" \
  -H "Content-Type: application/json" \
  -d '{"name":"custom-agent","domain":"finance","capabilities":["anomaly_detection"]}'
```

---

## 5. Integration Development

### 5.1 Embedding Copilot in a New Module

```typescript
// Frontend integration
const copilotResponse = await fetch('/ai/v1/copilot', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'X-Tenant-ID': tenantId
  },
  body: JSON.stringify({
    module: 'your-module-name',
    context: { page, field, partial_input, form_data },
    type: 'autocomplete'
  })
});
```

---

## 6. Testing

```bash
make test              # Unit tests
make test-integration  # Integration tests
make test-e2e          # End-to-end tests
```

---

## 7. Legacy Agent Reference

The `imports/ai_legacy/` directory contains Python agents from the pre-consolidation era:

| Package | Purpose |
|---|---|
| agent_framework | Base agent class, loader, lifecycle |
| integration_framework | Credential manager, base connector |
| vector_memory | Qdrant manager for agent memory |
| multi_framework | Framework orchestrator for multi-agent |
| performance | Cache manager, optimization utilities |
