# ERP-DBaaS

Sovereign ERP 2026 DBaaS control-plane module.

## Local Development

- Primary URL: http://localhost:5178
- Auth policy: dev-token-fallback

Copy `.env.example` to `.env.local` and configure MongoDB credentials.

Start local MongoDB (Docker example):

```bash
docker run --name erp-dbaas-mongo -p 27017:27017 -d mongo:7
```

Run in this exact order for immediate readiness:

```bash
npm install --no-audit --no-fund
npm run db:indexes
npm run db:seed
npm run dev
```

Validation commands:

```bash
npm run test
npm run typecheck
npm run build
```

## MongoDB Configuration

Required server variables:

- `MONGODB_URI`
- `MONGODB_DB_NAME`
- `MONGODB_MAX_POOL_SIZE` (default `50`)
- `MONGODB_MIN_POOL_SIZE` (default `5`)
- `MONGODB_MAX_IDLE_TIME_MS` (default `30000`)

## API Endpoints

- `GET /api/health` (MongoDB ping-backed readiness/liveness)
- `GET /api/control-center/cards` (tenant-scoped card list via `x-tenant-id`)
- `POST /api/control-center/cards/upsert` (tenant-scoped upsert for admin/testing)

### Curl Examples

```bash
curl -s http://localhost:5178/api/health
```

```bash
curl -s -H "x-tenant-id: tenant-default" http://localhost:5178/api/control-center/cards
```

```bash
curl -s -X POST http://localhost:5178/api/control-center/cards/upsert \
  -H "content-type: application/json" \
  -H "x-tenant-id: tenant-default" \
  -d '{"id":"qps","title":"Query Throughput","value":"11.2k/s","delta":"+6.1%"}'
```

## Architecture

- Next.js 16 App Router + TypeScript
- MongoDB (official Node.js driver) with tuned connection pooling
- Zod request/response validation for API contracts
- Workik feature slices under `src/features/*`
- TanStack Query default staleTime of 5 minutes
- Realtime invalidation via Centrifugo topic:
  `${NEXT_PUBLIC_ENV}.${NEXT_PUBLIC_ORG}.${NEXT_PUBLIC_MODULE}.${tenant}.ui.invalidate`

## Documentation Index
- Start here: [docs/README.md](./docs/README.md)
- Architecture: [docs/01-architecture/SAD.md](./docs/01-architecture/SAD.md)
- DevOps and quality: [docs/03-quality-devops/CI-CD.md](./docs/03-quality-devops/CI-CD.md)
