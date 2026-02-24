# User Manual for Developers -- FusionCommerce (ERP-eCommerce)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This manual guides developers through the FusionCommerce codebase, covering local setup, service development, API integration, event system usage, theme development, testing, and deployment workflows.

## 2. Development Environment Setup

### 2.1 Prerequisites

| Tool | Version | Installation |
|------|---------|-------------|
| Node.js | 20 LTS | `nvm install 20` |
| npm | 10.x | Bundled with Node.js |
| Docker Desktop | 4.x | docker.com |
| Git | 2.40+ | System package manager |

### 2.2 Repository Setup

```bash
git clone https://gitlab.com/opensase/erp-ecommerce.git
cd erp-ecommerce
npm install              # Install all workspace dependencies
cp .env.example .env     # Configure environment
npm run build            # Compile all TypeScript
npm run test             # Run all unit tests
```

### 2.3 Local Development Modes

```mermaid
flowchart TD
    DEV{Development Mode?}
    DEV -->|In-Memory| IM["USE_IN_MEMORY_BUS=true<br/>No Kafka needed<br/>In-memory repositories"]
    DEV -->|Docker Compose| DC["docker compose up<br/>Full stack with Redpanda"]
    DEV -->|Hybrid| HY["Docker for Kafka + DBs<br/>Local Node.js for services"]
```

**In-Memory Mode** (fastest, no dependencies):
```bash
USE_IN_MEMORY_BUS=true npm run dev --workspace=@fusioncommerce/catalog-service
```

**Docker Compose Mode** (full stack):
```bash
docker compose up --build
```

## 3. Service Development

### 3.1 Creating a New Service

Follow this template structure:

```
services/my-new-service/
  src/
    index.ts              # Bootstrap, lifecycle
    app.ts                # Fastify routes and middleware
    my-new-service.ts     # Business logic class
    my-new-repository.ts  # Data access layer
    types.ts              # TypeScript types and interfaces
  __tests__/
    my-new-service.test.ts
    app.test.ts
  migrations/             # Knex database migrations
  Dockerfile              # Multi-stage container build
  package.json            # Workspace package config
  tsconfig.json           # TypeScript config (extends base)
  jest.config.cjs         # Jest config (extends preset)
```

### 3.2 Service Bootstrap Pattern

```typescript
// src/index.ts
import { buildApp } from './app';
import { EventBusFactory } from '@fusioncommerce/event-bus';

async function main() {
  const eventBus = EventBusFactory.create({
    brokers: process.env.KAFKA_BROKERS?.split(',') ?? ['localhost:9092'],
    useInMemory: process.env.USE_IN_MEMORY_BUS === 'true',
    clientId: 'my-new-service',
    groupId: 'my-new-group',
  });

  const app = buildApp(eventBus);
  const port = Number(process.env.PORT ?? 3020);

  await app.listen({ port, host: '0.0.0.0' });
  console.log(`my-new-service running on port ${port}`);

  const shutdown = async () => {
    await app.close();
    await eventBus.disconnect();
    process.exit(0);
  };
  process.on('SIGTERM', shutdown);
  process.on('SIGINT', shutdown);
}

main().catch((err) => { console.error(err); process.exit(1); });
```

### 3.3 Adding Routes

```typescript
// src/app.ts
import fastify, { FastifyInstance } from 'fastify';
import { EventBus } from '@fusioncommerce/event-bus';
import { MyNewService } from './my-new-service';
import { InMemoryRepository } from './my-new-repository';

export function buildApp(eventBus: EventBus): FastifyInstance {
  const app = fastify({ logger: true });
  const repository = new InMemoryRepository();
  const service = new MyNewService(repository, eventBus);

  // Health check (required for all services)
  app.get('/health', async () => ({ status: 'ok' }));

  // Domain endpoints
  app.post('/v1/resources', async (request, reply) => {
    const resource = await service.create(request.body as CreateInput);
    reply.status(201).send(resource);
  });

  app.get('/v1/resources', async (request, reply) => {
    const resources = await service.list();
    reply.send({ data: resources });
  });

  return app;
}
```

### 3.4 Publishing Events

```typescript
import { TOPICS } from '@fusioncommerce/contracts';

export class MyNewService {
  constructor(
    private repository: MyRepository,
    private eventBus: EventBus,
  ) {}

  async create(input: CreateInput): Promise<Resource> {
    const resource = { id: crypto.randomUUID(), ...input, createdAt: new Date() };
    await this.repository.save(resource);
    await this.eventBus.publish(TOPICS.MY_RESOURCE_CREATED, {
      resourceId: resource.id,
      tenantId: input.tenantId,
      timestamp: new Date().toISOString(),
    });
    return resource;
  }
}
```

### 3.5 Consuming Events

```typescript
// In service bootstrap (index.ts)
await eventBus.subscribe(TOPICS.ORDER_CREATED, async (payload) => {
  const { orderId, items } = payload as OrderCreatedPayload;
  await service.processOrder(orderId, items);
});
```

## 4. Working with the Event Bus

### 4.1 Event Bus Interface

```typescript
interface EventBus {
  publish(topic: string, payload: unknown): Promise<void>;
  subscribe(topic: string, handler: (payload: unknown) => Promise<void>): Promise<void>;
  disconnect(): Promise<void>;
}
```

### 4.2 Adding New Event Types

1. Define the topic constant in `@fusioncommerce/contracts`:

```typescript
// packages/contracts/src/index.ts
export const TOPICS = {
  // ... existing topics
  MY_RESOURCE_CREATED: 'my_resource.created',
} as const;
```

2. Define the payload interface:

```typescript
export interface MyResourceCreatedPayload {
  resourceId: string;
  tenantId: string;
  timestamp: string;
  // ... domain-specific fields
}
```

3. Rebuild contracts: `npm run build --workspace=@fusioncommerce/contracts`

## 5. Database Operations

### 5.1 Creating Migrations

```bash
npx knex migrate:make create_my_table \
  --knexfile services/my-new-service/knexfile.ts
```

```typescript
// migrations/20260223000000_create_my_table.ts
import { Knex } from 'knex';

export async function up(knex: Knex): Promise<void> {
  await knex.schema.createTable('my_resources', (table) => {
    table.uuid('id').primary();
    table.string('tenant_id').notNullable().index();
    table.string('name').notNullable();
    table.decimal('value', 12, 2).notNullable();
    table.timestamp('created_at').defaultTo(knex.fn.now());
    table.timestamp('updated_at').defaultTo(knex.fn.now());
  });
}

export async function down(knex: Knex): Promise<void> {
  await knex.schema.dropTable('my_resources');
}
```

### 5.2 Repository Pattern

Always implement the Repository interface. Provide both InMemory and Postgres implementations:

```mermaid
classDiagram
    class Repository {
        <<interface>>
        +save(entity): Promise~void~
        +findById(id): Promise~Entity | null~
        +findAll(filters): Promise~Entity[]~
    }
    class InMemoryRepo {
        -store: Map
    }
    class PostgresRepo {
        -knex: Knex
    }
    Repository <|.. InMemoryRepo
    Repository <|.. PostgresRepo
```

## 6. Theme Development

### 6.1 Theme Structure

```
themes/my-theme/
  config/
    settings_schema.json    # Theme settings definition
    settings_data.json      # Default values
  layouts/
    theme.liquid            # Main layout template
  templates/
    index.liquid            # Homepage
    product.liquid          # Product detail page
    collection.liquid       # Category listing
    cart.liquid             # Shopping cart
    checkout.liquid         # Checkout flow
  sections/
    header.liquid           # Header section
    hero-banner.liquid      # Hero banner
    product-grid.liquid     # Product grid
    footer.liquid           # Footer section
  assets/
    theme.css               # Stylesheet
    theme.js                # JavaScript
  locales/
    en.json                 # English translations
```

### 6.2 Liquid Template Variables

| Variable | Description | Available On |
|----------|-------------|-------------|
| `product` | Current product object | Product page |
| `product.title` | Product name | Product page |
| `product.price` | Formatted price | Product page |
| `product.images` | Image array | Product page |
| `product.variants` | Variant array | Product page |
| `collection` | Current collection | Collection page |
| `collection.products` | Products in collection | Collection page |
| `cart` | Shopping cart object | All pages |
| `cart.items` | Cart line items | Cart page |
| `cart.total_price` | Cart total | Cart/Checkout |
| `customer` | Logged-in customer | All pages (if auth) |

## 7. Testing

### 7.1 Running Tests

```bash
npm run test                                    # All tests
npm run test --workspace=@fusioncommerce/catalog-service  # Single service
npm run test -- --watch                         # Watch mode
npm run test -- --coverage                      # Coverage report
```

### 7.2 Writing Service Tests

```typescript
import { buildApp } from '../src/app';
import { InMemoryEventBus } from '@fusioncommerce/event-bus';

describe('CatalogService API', () => {
  const eventBus = new InMemoryEventBus();
  const app = buildApp(eventBus);

  afterAll(() => app.close());

  it('POST /v1/products creates a product', async () => {
    const response = await app.inject({
      method: 'POST',
      url: '/v1/products',
      payload: { name: 'Test', sku: 'TST-001', price: 99.99, currency: 'USD' },
    });
    expect(response.statusCode).toBe(201);
    expect(JSON.parse(response.body)).toHaveProperty('id');
  });

  it('GET /health returns ok', async () => {
    const response = await app.inject({ method: 'GET', url: '/health' });
    expect(response.statusCode).toBe(200);
    expect(JSON.parse(response.body)).toEqual({ status: 'ok' });
  });
});
```

## 8. Deployment

### 8.1 Building Docker Images

```bash
# Single service
docker build -t fusioncommerce/catalog-service:latest \
  -f services/catalog/Dockerfile .

# All services via CI
npm run ci   # lint + test + build across all workspaces
```

### 8.2 Multi-Stage Dockerfile

```dockerfile
# Builder stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
COPY tsconfig.base.json ./
COPY packages/ packages/
COPY services/catalog/ services/catalog/
RUN npm ci --workspace=@fusioncommerce/catalog-service
RUN npm run build --workspace=@fusioncommerce/catalog-service

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/services/catalog/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## 9. API Reference Quick Guide

### 9.1 Authentication

All API requests require a Bearer JWT token in the Authorization header:
```
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

Tokens are issued by ERP-IAM via OIDC flow.

### 9.2 Common Response Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Resource created |
| 204 | Deleted (no content) |
| 400 | Validation error (check error.details) |
| 401 | Authentication required |
| 403 | Insufficient permissions |
| 404 | Resource not found |
| 409 | Conflict (duplicate SKU, etc.) |
| 422 | Business rule violation |
| 429 | Rate limit exceeded |
| 500 | Internal server error |
