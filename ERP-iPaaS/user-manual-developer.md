# User Manual for Developers -- ERP-iPaaS
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This manual provides comprehensive guidance for developers building integrations on the ERP-iPaaS platform, covering SDK usage, connector development, Temporal workflow authoring, API consumption, and testing.

## 2. Development Environment Setup

### 2.1 Prerequisites

| Tool | Version | Install Command |
|------|---------|----------------|
| Node.js | 18+ | `nvm install 18` |
| Go | 1.21+ | `brew install go` |
| Docker | 24+ | Docker Desktop |
| Helm | 3.12+ | `brew install helm` |
| kubectl | 1.28+ | `brew install kubectl` |
| kind | 0.20+ | `brew install kind` |

### 2.2 Local Development Stack

```bash
# Clone the repository
git clone <repo-url> && cd ERP-iPaaS

# Start local infrastructure
docker compose up -d

# Bootstrap local Kubernetes cluster
make bootstrap

# Verify services
curl http://localhost:8080/healthz  # Activepieces
curl http://localhost:7233          # Temporal
curl http://localhost:9092          # Redpanda (Kafka)
curl http://localhost:8123          # ClickHouse
curl http://localhost:3000          # Grafana
```

### 2.3 Project Structure

```
packages/
  integration-layer-ts/    # TypeScript SDK
  integration-layer-go/    # Go SDK
  connector-cli/           # CLI for connector development
  temporal/                # Temporal workflow SDK wrapper
  nexum-flow/              # DAG execution engine
  mcp-host/                # AI agent gateway
  llm-utils/               # LLM utility functions
```

## 3. Using the TypeScript SDK

### 3.1 Installation

```bash
npm install @billyronks/integration-layer-ts
```

### 3.2 Client Initialization

```typescript
import { IntegrationClient } from '@billyronks/integration-layer-ts';

const client = new IntegrationClient({
  baseUrl: 'https://api.<DOMAIN>/integration',
  tenantId: 'your-tenant-id',
  accessToken: 'your-oauth2-token',
});
```

### 3.3 Common Operations

```typescript
// List connectors
const connectors = await client.connectors.list({ page: 1, pageSize: 20 });

// Create a connector
const connector = await client.connectors.create({
  name: 'my-connector',
  description: 'My custom connector',
  semanticVersion: '1.0.0',
  owner: 'my-team',
  categories: ['CRM'],
  schemas: [{ name: 'lead-schema', version: '1.0' }],
});

// Publish an event
const ack = await client.events.publish({
  topic: 'workflow.events',
  payload: { status: 'completed' },
  idempotencyKey: 'unique-key',
});
```

## 4. Using the Go SDK

### 4.1 Installation

```bash
go get github.com/billyronks/integration-layer-go
```

### 4.2 Client Usage

```go
package main

import (
    ipaas "github.com/billyronks/integration-layer-go"
)

func main() {
    client := ipaas.NewClient(
        "https://api.<DOMAIN>/integration",
        "your-tenant-id",
        "your-oauth2-token",
    )

    connectors, err := client.ListConnectors(1, 20)
    if err != nil {
        log.Fatal(err)
    }
}
```

## 5. Building Custom Connectors

### 5.1 Scaffold a New Connector

```bash
npx @billyronks/connector-cli scaffold my-connector
cd my-connector
```

### 5.2 Connector Structure

```
my-connector/
  package.json
  tsconfig.json
  src/
    index.ts         # Connector definition
    auth.ts          # Authentication handler
    actions/
      create.ts      # Action: create resource
      list.ts        # Action: list resources
    triggers/
      webhook.ts     # Trigger: webhook listener
  schemas/
    input.json       # Input schema (JSON Schema)
    output.json      # Output schema (JSON Schema)
  tests/
    actions.test.ts  # Unit tests
```

### 5.3 Implementing a Connector

```typescript
// src/index.ts
import { createPiece } from '@activepieces/pieces-sdk';
import { createAction } from './actions/create';
import { listAction } from './actions/list';
import { webhookTrigger } from './triggers/webhook';

export const myConnector = createPiece({
  name: 'my-connector',
  displayName: 'My Connector',
  description: 'Connects to My Service',
  auth: {
    type: 'OAUTH2',
    authUrl: 'https://my-service.com/oauth/authorize',
    tokenUrl: 'https://my-service.com/oauth/token',
    scope: ['read', 'write'],
  },
  actions: [createAction, listAction],
  triggers: [webhookTrigger],
});
```

### 5.4 Validate and Publish

```bash
# Validate connector
npx @billyronks/connector-cli validate --strict

# Run tests
npm test

# Publish to marketplace
npx @billyronks/connector-cli publish --version 1.0.0
```

## 6. Temporal Workflow Development

### 6.1 Creating a Workflow

```typescript
// workflows/my-workflow.ts
import { proxyActivities } from '@temporalio/workflow';

const { fetchData, transformData, loadData } = proxyActivities({
  startToCloseTimeout: '5 minutes',
  retry: {
    initialInterval: '5 seconds',
    maximumAttempts: 5,
    backoffCoefficient: 2,
  },
});

export async function myWorkflow(input: { tenantId: string }) {
  const rawData = await fetchData({ tenantId: input.tenantId });
  const transformed = await transformData({ data: rawData });
  await loadData({ tenantId: input.tenantId, data: transformed });
}
```

### 6.2 Creating Activities

```typescript
// activities/my-activities.ts
export async function fetchData(input: { tenantId: string }) {
  const response = await fetch(`https://api.example.com/data`, {
    headers: { 'X-Tenant-ID': input.tenantId },
  });
  return response.json();
}

export async function transformData(input: { data: any }) {
  // Transform logic here
  return input.data.map((item: any) => ({
    ...item,
    processedAt: new Date().toISOString(),
  }));
}
```

### 6.3 Registering Workers

```typescript
// worker.ts
import { Worker } from '@temporalio/worker';
import * as activities from './activities/my-activities';

async function run() {
  const worker = await Worker.create({
    workflowsPath: require.resolve('./workflows/my-workflow'),
    activities,
    taskQueue: 'my-task-queue',
  });
  await worker.run();
}

run().catch(console.error);
```

### 6.4 Running Workflows

```typescript
import { Client } from '@temporalio/client';

const client = new Client();
const handle = await client.workflow.start('myWorkflow', {
  taskQueue: 'my-task-queue',
  workflowId: 'my-workflow-' + Date.now(),
  args: [{ tenantId: 'tenant-123' }],
});
```

## 7. API Development

### 7.1 OpenAPI Specification

The API is defined at `api/openapi/integration-layer.yaml`. Use Redocly CLI to validate:

```bash
make openapi-validate
```

### 7.2 Testing APIs

```bash
# Health check
curl http://localhost:8080/healthz

# List workflows (with tenant header)
curl -H "X-Tenant-ID: test-tenant" http://localhost:8080/v1/workflow-engine

# Create a workflow
curl -X POST -H "X-Tenant-ID: test-tenant" \
  -H "Content-Type: application/json" \
  -d '{"name":"test","type":"activepieces"}' \
  http://localhost:8080/v1/workflow-engine
```

## 8. Testing

### 8.1 Unit Tests (Vitest)

```bash
# Run all tests
make smoke

# Run specific test
npx vitest run tests/lead-intake.test.ts
```

### 8.2 Temporal Workflow Tests

```typescript
// tests/my-workflow.test.ts
import { TestWorkflowEnvironment } from '@temporalio/testing';
import { myWorkflow } from '../workflows/my-workflow';

describe('myWorkflow', () => {
  let testEnv: TestWorkflowEnvironment;

  beforeAll(async () => {
    testEnv = await TestWorkflowEnvironment.create();
  });

  afterAll(async () => {
    await testEnv.teardown();
  });

  it('should complete successfully', async () => {
    const result = await testEnv.client.workflow.execute(myWorkflow, {
      taskQueue: 'test',
      args: [{ tenantId: 'test-tenant' }],
    });
    expect(result).toBeDefined();
  });
});
```

## 9. Debugging

### 9.1 Local Debugging Tools

| Tool | URL | Purpose |
|------|-----|---------|
| Temporal Web UI | http://localhost:8088 | Workflow execution history |
| Grafana | http://localhost:3000 | Metrics and dashboards |
| Redpanda Console | http://localhost:8082 | Event stream inspection |
| ClickHouse | http://localhost:8123 | SQL queries on analytics |

### 9.2 Trace Correlation

All requests carry `trace_id` via OpenTelemetry. Use Grafana Tempo to search by trace ID and see the full request chain across services.
