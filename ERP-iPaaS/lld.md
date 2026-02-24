# Low-Level Design -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This Low-Level Design (LLD) document provides implementation-level detail for ERP-iPaaS components, including class/module diagrams, function signatures, data structures, algorithm descriptions, and configuration specifics.

## 2. Go Microservice Implementation

### 2.1 Service Skeleton

Each Go microservice follows an identical pattern implemented in `main.go`:

```mermaid
graph TB
    subgraph "main.go Structure"
        ENV[Environment Variables<br/>PORT, MODULE_NAME]
        MUX[http.ServeMux]
        H[Health Handler<br/>/healthz]
        COL[Collection Handler<br/>/v1/{service}]
        RES[Resource Handler<br/>/v1/{service}/{id}]
        JSON[writeJSON helper]
    end

    ENV --> MUX
    MUX --> H
    MUX --> COL
    MUX --> RES
    COL --> JSON
    RES --> JSON
```

### 2.2 Handler Implementation Detail

```go
// Common payload type for all services
type payload map[string]any

// JSON response writer with content-type header
func writeJSON(w http.ResponseWriter, code int, v any) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(code)
    _ = json.NewEncoder(w).Encode(v)
}

// Tenant validation middleware (inline)
if r.Header.Get("X-Tenant-ID") == "" {
    writeJSON(w, http.StatusBadRequest,
        map[string]string{"error": "missing X-Tenant-ID"})
    return
}
```

### 2.3 Event Topic Convention

Each service emits events following the pattern:
```
erp.ipaas.{service-name}.{action}
```

| Service | Created | Updated | Deleted | Listed | Read |
|---------|---------|---------|---------|--------|------|
| workflow-engine | `erp.ipaas.workflow-engine.created` | `erp.ipaas.workflow-engine.updated` | `erp.ipaas.workflow-engine.deleted` | `erp.ipaas.workflow-engine.listed` | `erp.ipaas.workflow-engine.read` |
| connector-framework | `erp.ipaas.connector-framework.created` | `erp.ipaas.connector-framework.updated` | `erp.ipaas.connector-framework.deleted` | `erp.ipaas.connector-framework.listed` | `erp.ipaas.connector-framework.read` |
| event-backbone | `erp.ipaas.event-backbone.created` | `erp.ipaas.event-backbone.updated` | `erp.ipaas.event-backbone.deleted` | `erp.ipaas.event-backbone.listed` | `erp.ipaas.event-backbone.read` |
| api-management | `erp.ipaas.api-management.created` | `erp.ipaas.api-management.updated` | `erp.ipaas.api-management.deleted` | `erp.ipaas.api-management.listed` | `erp.ipaas.api-management.read` |
| etl | `erp.ipaas.etl.created` | `erp.ipaas.etl.updated` | `erp.ipaas.etl.deleted` | `erp.ipaas.etl.listed` | `erp.ipaas.etl.read` |
| webhook | `erp.ipaas.webhook.created` | `erp.ipaas.webhook.updated` | `erp.ipaas.webhook.deleted` | `erp.ipaas.webhook.listed` | `erp.ipaas.webhook.read` |

## 3. Nexum Flow Engine Detail

### 3.1 Class Diagram

```mermaid
classDiagram
    class NexumEngine {
        -redis: RedisClient
        -executions: Map~string, ExecutionRecord~
        -opts: EngineOptions
        +run(context: ExecutionContext): ExecutionRecord
        +getExecution(id: string): ExecutionRecord
        +listExecutions(tenantId: string): ExecutionRecord[]
    }

    class EngineOptions {
        +redisUrl: string
        +executors: Map~string, NodeExecutor~
    }

    class ExecutionContext {
        +workflow: WorkflowDefinition
        +tenantId: string
        +inputs: any
    }

    class ExecutionRecord {
        +id: string
        +workflowId: string
        +tenantId: string
        +status: string
        +startedAt: string
        +finishedAt: string
        +output: any
        +error: string
    }

    class NodeExecutor {
        <<interface>>
        +execute(ctx, node, payload): NodeResult
    }

    class HTTPNode {
        +execute(): NodeResult
    }

    class LLMNode {
        +execute(): NodeResult
    }

    class MCPNode {
        +execute(): NodeResult
    }

    class TemporalNode {
        +execute(): NodeResult
    }

    NexumEngine --> EngineOptions
    NexumEngine --> ExecutionRecord
    NexumEngine ..> ExecutionContext
    EngineOptions --> NodeExecutor
    HTTPNode ..|> NodeExecutor
    LLMNode ..|> NodeExecutor
    MCPNode ..|> NodeExecutor
    TemporalNode ..|> NodeExecutor
```

### 3.2 Execution Algorithm

```
function run(context):
    id = generateUUID()
    record = new ExecutionRecord(id, context)
    record.status = "running"

    try:
        payload = context.inputs
        for each node in context.workflow.nodes:
            executor = executors.get(node.type)
            if executor is null:
                throw Error("No executor for " + node.type)

            result = executor.execute(context, node, payload)
            payload = { previous: payload, output: result.output }

            // Cache intermediate results in Redis (1h TTL)
            redis.set("nexum:{id}:{node.id}", result.output, TTL=3600)

            emit("node:completed", { executionId: id, nodeId: node.id })

        record.status = "completed"
        record.output = payload.output
        emit("execution:completed", record)
    catch error:
        record.status = "failed"
        record.error = error.message
        emit("execution:failed", record)
        throw error

    return record
```

## 4. Temporal Workflow Implementation

### 4.1 Lead Intake Workflow Detail

```mermaid
graph TD
    subgraph "Lead Intake Workflow"
        START[Start] --> A1[upsertLeadActivity<br/>timeout: 5m, retries: 5]
        A1 --> A2[llmActivity<br/>timeout: 5m, retries: 5]
        A2 --> A3[notifyChannel<br/>timeout: 5m, retries: 5]
        A3 --> END[Complete]
    end

    subgraph "Activity Config"
        CFG[startToCloseTimeout: 5 minutes<br/>heartbeatTimeout: 30 seconds<br/>initialInterval: 5 seconds<br/>maximumAttempts: 5<br/>backoffCoefficient: 2]
    end

    A1 -.-> CFG
    A2 -.-> CFG
    A3 -.-> CFG
```

### 4.2 Activity Implementations

| Activity | Module | Purpose |
|----------|--------|---------|
| `upsertLeadActivity` | `temporal/src/activities/crm.ts` | Upsert lead record in CRM |
| `llmActivity` | `temporal/src/activities/llm.ts` | Generate LLM-powered content |
| `notifyChannel` | `temporal/src/activities/notifications.ts` | Send notifications (Slack, email, etc.) |
| `throttleActivity` | `temporal/src/activities/throttling.ts` | Rate-limit external API calls |

### 4.3 Worker Configuration

```mermaid
graph LR
    subgraph "Temporal Worker"
        W[Worker Process<br/>temporal/src/worker.ts]
        REG[Register Workflows<br/>register-workflows.ts]
        ACT[Register Activities]
    end

    subgraph "Task Queues"
        TQ1[ipaas-workflows]
        TQ2[ipaas-activities]
    end

    W --> REG
    W --> ACT
    REG --> TQ1
    ACT --> TQ2
```

## 5. Activepieces Custom Piece Implementation

### 5.1 Piece Structure

```mermaid
graph TB
    subgraph "Custom Piece (e.g., ClickHouse)"
        IDX[index.ts<br/>Piece registration]
        SCH[schema.ts<br/>Type definitions]
        SRC[src/index.ts<br/>Action implementations]
    end

    subgraph "Actions"
        IRA[insertRows action<br/>Batch insert data]
        QTA[queryTable action<br/>Execute SELECT]
    end

    IDX --> SRC
    SRC --> IRA
    SRC --> QTA
    IDX --> SCH
```

### 5.2 Shared Utilities

| Utility | Path | Purpose |
|---------|------|---------|
| HTTP Client | `src/activepieces/pieces/_shared/http-client.ts` | Centralized HTTP with retry |
| Tenant Context | `src/activepieces/pieces/_shared/tenant-context.ts` | Extract tenant from execution context |

## 6. MCP Host Implementation

### 6.1 Module Structure

```mermaid
classDiagram
    class MCPHost {
        +registry: Registry
        +server: Server
        +start(): void
    }

    class Registry {
        +tools: Map~string, Tool~
        +register(tool: Tool): void
        +lookup(name: string): Tool
    }

    class Server {
        +listen(port: number): void
        +handleRequest(req): Response
    }

    class Invokers {
        +invokeHTTP(config): Result
        +invokeTemporal(config): Result
        +invokeLLM(config): Result
    }

    MCPHost --> Registry
    MCPHost --> Server
    MCPHost --> Invokers
```

## 7. Circuit Breaker Implementation

```mermaid
stateDiagram-v2
    [*] --> Closed: Initial state

    Closed --> Closed: Success (reset failure count)
    Closed --> Open: Failure count >= threshold (5)

    Open --> Open: Reject requests immediately
    Open --> HalfOpen: After cooldown period (30s)

    HalfOpen --> Closed: Probe request succeeds
    HalfOpen --> Open: Probe request fails
```

Implementation in `src/temporal/workers/circuitBreaker.ts`:

| Parameter | Default Value | Description |
|-----------|--------------|-------------|
| failureThreshold | 5 | Failures before opening circuit |
| cooldownPeriod | 30000ms | Time before half-open probe |
| successThreshold | 1 | Successes in half-open to close |
| timeout | 10000ms | Per-request timeout |

## 8. LLM Utilities Detail

### 8.1 Module Structure

| File | Purpose |
|------|---------|
| `src/lib/llm/prompts.ts` | Prompt templates for various LLM tasks |
| `src/lib/llm/redaction.ts` | PII redaction before sending to LLM |
| `src/lib/llm/retry.ts` | LLM-specific retry with rate limit handling |
| `src/lib/llm/validators.ts` | Output validation and structured parsing |

### 8.2 PII Redaction Flow

```mermaid
graph LR
    INPUT[User Input] --> DETECT[Detect PII Fields]
    DETECT --> REDACT[Redact Sensitive Data]
    REDACT --> LLM[Send to LLM]
    LLM --> RESTORE[Restore Redacted Fields]
    RESTORE --> OUTPUT[Clean Output]
```

## 9. Configuration Details

### 9.1 Docker Compose Port Mapping

| Service | Host Port | Container Port |
|---------|-----------|---------------|
| Traefik | 80, 443 | 80, 443 |
| Keycloak | 8081 | 8080 |
| PostgreSQL | 5432 | 5432 |
| Redpanda (Kafka) | 9092 | 9092 |
| Redpanda (Schema Registry) | 8081 | 8081 |
| Redpanda (HTTP Proxy) | 8082 | 8082 |
| Activepieces | 8080 | 80 |
| Temporal | 7233 | 7233 |
| Temporal Web UI | 8088 | 8088 |
| MinIO API | 9000 | 9000 |
| MinIO Console | 9001 | 9001 |
| ClickHouse | 8123 | 8123 |
| Grafana | 3000 | 3000 |
| Dragonfly (Redis) | 6379 | 6379 |

### 9.2 Environment Variables

| Variable | Service | Default | Description |
|----------|---------|---------|-------------|
| PORT | All Go services | 8080 | HTTP listen port |
| MODULE_NAME | All Go services | ERP-iPaaS | Module identifier |
| AP_API_KEY | Activepieces | dev | API authentication key |
| DB | Temporal | postgresql | Database backend |
| GF_SECURITY_ADMIN_PASSWORD | Grafana | admin | Admin password |
| MINIO_ROOT_USER | MinIO | waas | Root username |
| MINIO_ROOT_PASSWORD | MinIO | waaswaas | Root password |
