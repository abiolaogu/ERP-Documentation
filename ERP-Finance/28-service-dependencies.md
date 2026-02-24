# ERP-Finance Service Dependencies

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | Service Dependencies |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

## Internal Service Dependency Graph

```mermaid
flowchart TB
    GW["Gateway<br/>:8090"] --> GL["General Ledger<br/>:8091"]
    GW --> AP["Accounts Payable<br/>:8093"]
    GW --> AR["Accounts Receivable<br/>:8095"]
    GW --> BILL["Billing<br/>:8089"]
    GW --> PAY["Payments<br/>:8084"]
    GW --> ASSET["Asset Management<br/>:8096"]
    GW --> TAX["Tax Management<br/>:8098"]
    GW --> EXP["Expense Management<br/>:8100"]
    GW --> TREAS["Treasury<br/>:8101"]
    GW --> BUD["Budget<br/>:8102"]

    BILL --> PAY
    BILL --> GL
    AP --> GL
    AR --> GL
    EXP --> GL
    EXP --> AP
    ASSET --> GL
    TAX --> GL
    TREAS --> GL
    BUD --> GL

    PAY --> GL
    AR --> PAY
    TREAS --> PAY
```

## Dependency Matrix

| Service | Depends On | Depended By |
|---------|-----------|-------------|
| Gateway | All services | External clients |
| General Ledger | PostgreSQL, Redis, NATS | AP, AR, Billing, Payments, Asset, Tax, Expense, Treasury, Budget |
| Accounts Payable | GL, PostgreSQL, NATS, OCR | Expense Management |
| Accounts Receivable | GL, Payments, PostgreSQL, NATS | - |
| Billing | Payments, GL, PostgreSQL, Redis, NATS | - |
| Payments | GL, PostgreSQL, NATS, Payment Providers | Billing, AR, Treasury |
| Asset Management | GL, PostgreSQL, Qdrant, Claude API | - |
| Tax Management | GL, PostgreSQL, NATS, Avalara/Vertex | - |
| Expense Management | GL, AP, PostgreSQL, MinIO, NATS, OCR | - |
| Treasury | GL, Payments, PostgreSQL, NATS, Bank APIs | - |
| Budget | GL, PostgreSQL, ClickHouse, NATS | - |

## External Dependency Map

```mermaid
flowchart LR
    subgraph Critical["Critical Dependencies"]
        PG["PostgreSQL<br/>(Primary Database)"]
        NATS2["NATS JetStream<br/>(Event Bus)"]
    end

    subgraph Important["Important Dependencies"]
        REDIS2["Redis<br/>(Cache)"]
        IAM2["ERP-IAM<br/>(Authentication)"]
        PLAT["ERP-Platform<br/>(Entitlements)"]
    end

    subgraph Optional["Optional / Degradable"]
        CLAUDE["Claude API<br/>(AI Analysis)"]
        AVA2["Avalara<br/>(Tax Calc)"]
        OCR2["Document AI<br/>(OCR)"]
        CH2["ClickHouse<br/>(Analytics)"]
    end

    subgraph PayProviders["Payment Providers (Failover)"]
        STR["Stripe"]
        ADY["Adyen"]
        PST["Paystack"]
        FLW["Flutterwave"]
        MPESA2["M-Pesa"]
    end

    PG --> |"Required"| Services["All Services"]
    NATS2 --> |"Required"| Services
    REDIS2 --> |"Degraded mode<br/>without cache"| Services
    IAM2 --> |"Suite mode only"| Services
    PLAT --> |"Suite mode only"| Services
    CLAUDE --> |"Feature disabled<br/>without key"| ASSET2["Asset Mgmt"]
    AVA2 --> |"Fallback to<br/>native engine"| TAX2["Tax Mgmt"]
    OCR2 --> |"Manual entry<br/>fallback"| AP2["AP + Expense"]
    CH2 --> |"Reports slower<br/>from PostgreSQL"| BUD2["Budget + Treasury"]
    STR & ADY & PST & FLW & MPESA2 --> |"Failover between<br/>providers"| PAY2["Payments"]
```

## Failure Impact Analysis

| Dependency Failure | Impact | Mitigation |
|-------------------|--------|-----------|
| PostgreSQL down | Full outage -- all services unable to persist | HA cluster with auto-failover (< 60s) |
| NATS down | Events queued; eventual consistency delayed | NATS cluster (3 nodes); services continue with local queue |
| Redis down | Cache miss; increased DB load | Graceful degradation; direct DB queries |
| Claude API down | AI features unavailable | Features disabled; no impact on core finance |
| Stripe down | Payments via Stripe fail | Automatic failover to Adyen/Paystack |
| Paystack down | NGN payments via Paystack fail | Failover to Flutterwave |
| Avalara down | Tax calculation unavailable | Fallback to built-in tax engine |
| ClickHouse down | Analytics queries slower | Fallback to PostgreSQL for reporting |
| MinIO down | Document upload/download fails | Queued for retry; no impact on transactions |
| ERP-IAM down | Suite auth fails | Standalone mode fallback if configured |

## Service Startup Order

Services must start in dependency order:

```mermaid
flowchart TD
    L1["Layer 1: Infrastructure"]
    L2["Layer 2: Core Data"]
    L3["Layer 3: Financial Core"]
    L4["Layer 4: Sub-Ledgers"]
    L5["Layer 5: Gateway"]

    L1 --> L2 --> L3 --> L4 --> L5

    L1 -.- PG2["PostgreSQL"] & RD["Redis"] & NT["NATS"]
    L2 -.- GL2["General Ledger"]
    L3 -.- BILL2["Billing"] & PAY3["Payments"]
    L4 -.- AP3["AP"] & AR3["AR"] & TAX3["Tax"] & ASSET3["Asset"] & EXP3["Expense"] & TREAS3["Treasury"] & BUD3["Budget"]
    L5 -.- GW2["Gateway"]
```

## Circuit Breaker Configuration

| Dependency | Failure Threshold | Recovery Time | Fallback |
|-----------|-------------------|---------------|----------|
| Stripe API | 5 failures / 30s | 60s half-open | Route to Adyen |
| Paystack API | 5 failures / 30s | 60s half-open | Route to Flutterwave |
| Avalara API | 3 failures / 60s | 120s half-open | Native tax engine |
| Claude API | 2 failures / 60s | 300s half-open | Feature disabled |
| Bank Feed API | 3 failures / 300s | 600s half-open | Manual import |

## Health Check Dependencies

Each service health check verifies its critical dependencies:

```json
{
  "service": "billing-service",
  "status": "healthy",
  "dependencies": {
    "postgresql": {"status": "up", "latency_ms": 2},
    "redis": {"status": "up", "latency_ms": 1},
    "nats": {"status": "up", "latency_ms": 1}
  },
  "uptime_seconds": 86400
}
```
