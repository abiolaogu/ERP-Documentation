# ERP-BI Developer Guide

| Field | Value |
|---|---|
| Module | ERP-BI |
| Audience | Developers |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Development Environment Setup

### 1.1 Prerequisites

```bash
# Required tools
node >= 20.0.0
go >= 1.22
docker >= 24.0
```

### 1.2 Clone and Install

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-BI

# Install frontend dependencies
npm install

# Generate Prisma client
npm run db:generate

# Push database schema
npm run db:push

# Seed demo data
npm run db:seed
```

### 1.3 Run Development Server

```bash
# Start Next.js frontend
npm run dev

# Start a backend service (example)
cd services/dashboard-service
PORT=8080 go run main.go
```

---

## 2. Project Structure

```
ERP-BI/
├── src/                          # Next.js frontend
│   ├── app/                      # App Router pages
│   │   ├── page.tsx              # Dashboard (home)
│   │   ├── layout.tsx            # Root layout
│   │   ├── globals.css           # Global styles
│   │   ├── ai-insights/          # AI Insights page
│   │   ├── bom/                  # BOM page
│   │   ├── capacity/             # Capacity page
│   │   ├── products/             # Products page
│   │   ├── quality/              # Quality page
│   │   ├── settings/             # Settings page
│   │   ├── shop-floor/           # Shop Floor page
│   │   ├── work-orders/          # Work Orders page
│   │   └── api/                  # API routes
│   │       ├── ai/               # AI API endpoints
│   │       ├── bom/              # BOM API endpoints
│   │       ├── dashboard/        # Dashboard API
│   │       ├── products/         # Products API
│   │       ├── work-centers/     # Work Centers API
│   │       └── work-orders/      # Work Orders API
│   ├── components/               # React components
│   │   ├── layout/               # Header, Sidebar
│   │   └── ui/                   # KPI Card, Data Table, etc.
│   └── lib/                      # Utilities
│       ├── ai/                   # AI engines
│       │   ├── index.ts          # AI exports
│       │   ├── forecasting.ts    # Demand forecasting
│       │   ├── anomaly-detection.ts  # Anomaly detection
│       │   └── optimization.ts   # Schedule/capacity optimization
│       ├── db.ts                 # Prisma client
│       └── utils.ts              # Utility functions
├── services/                     # Go microservices
│   ├── dashboard-service/
│   ├── report-service/
│   ├── data-modeling-service/
│   ├── query-engine/
│   ├── data-warehouse-service/
│   ├── alert-service/
│   └── nlq-service/
├── prisma/
│   ├── schema.prisma             # Database schema
│   └── seed.ts                   # Seed data
├── erp/
│   └── module.manifest.yaml      # ERP module manifest
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── next.config.js
├── postcss.config.js
└── Makefile
```

---

## 3. Frontend Development

### 3.1 Component Pattern

Components use TypeScript with Tailwind CSS:

```typescript
interface KPICardProps {
  title: string;
  value: string | number;
  change?: number;
  changeLabel?: string;
  icon: React.ComponentType;
  iconColor?: string;
  trend?: 'up' | 'down';
}

export function KPICard({ title, value, change, ... }: KPICardProps) {
  return (
    <div className="card">
      {/* Component JSX */}
    </div>
  );
}
```

### 3.2 State Management

- **Server state**: TanStack Query (React Query) for API data fetching
- **Client state**: Zustand for UI state (sidebar open, filters, theme)
- **Form state**: Controlled components with Zod validation

### 3.3 Chart Development

Charts use Recharts library:

```typescript
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';

<ResponsiveContainer width="100%" height={280}>
  <BarChart data={data}>
    <CartesianGrid strokeDasharray="3 3" />
    <XAxis dataKey="name" />
    <YAxis />
    <Tooltip />
    <Bar dataKey="value" fill="#2563eb" />
  </BarChart>
</ResponsiveContainer>
```

---

## 4. Backend Service Development

### 4.1 Service Pattern

All Go services follow an identical pattern:

```go
func main() {
    port := os.Getenv("PORT")
    if port == "" { port = "8080" }

    mux := http.NewServeMux()
    mux.HandleFunc("/healthz", healthHandler)
    mux.HandleFunc("/v1/{resource}", resourceHandler)
    mux.HandleFunc("/v1/{resource}/", resourceByIDHandler)

    log.Fatal(http.ListenAndServe(":"+port, mux))
}
```

### 4.2 Tenant Validation

Every handler validates the `X-Tenant-ID` header:

```go
if r.Header.Get("X-Tenant-ID") == "" {
    writeJSON(w, http.StatusBadRequest, map[string]string{"error": "missing X-Tenant-ID"})
    return
}
```

### 4.3 Event Publishing

Services publish NATS events for every state change:

```go
writeJSON(w, http.StatusCreated, map[string]any{
    "item": body,
    "event_topic": "erp.bi.dashboard.created",
})
```

---

## 5. AI Engine Development

### 5.1 Forecasting Engine

The forecasting engine in `src/lib/ai/forecasting.ts` implements:
- Linear regression for trend detection
- Autocorrelation for seasonality detection
- Holt-Winters double exponential smoothing
- Weighted moving average
- Ensemble combination with configurable weights
- MAPE calculation for accuracy evaluation

### 5.2 Anomaly Detection Engine

The anomaly detection engine in `src/lib/ai/anomaly-detection.ts` implements:
- Z-Score based detection
- IQR (Interquartile Range) method
- Multi-metric anomaly scoring
- Predictive maintenance with degradation modeling

---

## 6. Testing

```bash
# Unit tests
make test

# Integration tests
make test-integration

# E2E tests
make test-e2e
```

---

## 7. Contributing Guidelines

1. Create a feature branch from `main`
2. Follow TypeScript strict mode, Go standard formatting
3. Add tests for new features
4. Update API documentation for endpoint changes
5. Run full test suite before submitting PR
