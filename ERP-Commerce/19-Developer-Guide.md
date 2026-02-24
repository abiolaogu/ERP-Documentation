# ERP-Commerce -- Developer Guide

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

## 1. Development Environment Setup

### 1.1 Prerequisites

| Tool         | Version  | Purpose                              |
|-------------|----------|--------------------------------------|
| Go           | 1.22+    | Core service development             |
| Python       | 3.12+    | AI/ML service development            |
| Rust         | 1.75+    | High-performance components          |
| Node.js      | 20+      | Frontend development                 |
| Docker       | 24+      | Container runtime                    |
| Docker Compose| 2.x     | Local multi-service orchestration    |
| Make         | 3.81+    | Build automation                     |
| kubectl      | Latest   | Kubernetes interaction               |

### 1.2 Repository Structure

```
ERP-Commerce/
  cmd/
    server/
      main.go              # Main gateway entry point
  configs/
    capabilities.json      # Module capability registry
  docs/                    # Internal docs
  erp/
    aidd.guardrails.yaml   # AI-driven development guardrails
    module.manifest.yaml   # Module manifest
  imports/
    omniroute_core/        # Imported OmniRoute source
  merge/
    MERGE_MANIFEST.yaml    # Consolidation manifest
  services/
    catalog-service/       # Product catalog + PIM
    order-service/         # Order orchestration + EDI
    pricing-service/       # Pricing engine
    inventory-service/     # Stock management
    trade-credit-service/  # Credit scoring + collections
    distribution-service/  # RTM + territories + van sales
    pos-service/           # Point of sale
    portal-service/        # 13 role portals
    logistics-service/     # Delivery + routing
    marketplace-service/   # B2B marketplace
  Makefile
  go.mod
  server                   # Compiled binary
```

### 1.3 Local Development

```bash
# Clone repository
git clone <repo-url> ERP-Commerce
cd ERP-Commerce

# Start infrastructure dependencies
docker-compose up -d postgres redis nats elasticsearch

# Run a specific service
cd services/catalog-service
PORT=8001 MODULE_NAME=ERP-Commerce go run main.go

# Run all services
make run-all

# Run tests
make test

# Build all services
make build
```

---

## 2. Service Development Patterns

### 2.1 Creating a New Endpoint

All services follow the same Go HTTP handler pattern:

```go
// 1. Define handler in http/handlers.go
func (h *Handler) CreateProduct(w http.ResponseWriter, r *http.Request) {
    // Extract tenant context
    tenantID := r.Header.Get("X-Tenant-ID")
    if tenantID == "" {
        writeJSON(w, http.StatusBadRequest, map[string]string{
            "error": "missing X-Tenant-ID",
        })
        return
    }

    // Decode request body
    var req CreateProductRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeJSON(w, http.StatusBadRequest, map[string]string{
            "error": "invalid request body",
        })
        return
    }

    // Execute business logic
    product, err := h.service.CreateProduct(r.Context(), tenantID, req)
    if err != nil {
        writeJSON(w, http.StatusInternalServerError, map[string]string{
            "error": err.Error(),
        })
        return
    }

    // Return response with event topic
    writeJSON(w, http.StatusCreated, map[string]any{
        "item":        product,
        "event_topic": "erp.commerce.catalog.created",
    })
}
```

### 2.2 Event Publishing Pattern

```go
// Publish CloudEvents-formatted event
func (p *EventPublisher) Publish(ctx context.Context, event DomainEvent) error {
    ce := cloudevents.NewEvent()
    ce.SetSpecVersion("1.0")
    ce.SetType(fmt.Sprintf("erp.commerce.%s.%s", event.Entity, event.Action))
    ce.SetSource("/erp-commerce/" + p.serviceName)
    ce.SetID(uuid.NewString())
    ce.SetTime(time.Now())
    ce.SetData(cloudevents.ApplicationJSON, event.Data)

    return p.natsClient.Publish(ce.Type(), ce)
}
```

### 2.3 Multi-Tenant Data Access

Every database query must include tenant isolation:

```go
// Repository pattern with tenant scoping
func (r *ProductRepo) FindByID(ctx context.Context, tenantID, productID string) (*Product, error) {
    var product Product
    err := r.db.QueryRowContext(ctx,
        `SELECT id, tenant_id, sku, name, status, created_at
         FROM products
         WHERE tenant_id = $1 AND id = $2`,
        tenantID, productID,
    ).Scan(&product.ID, &product.TenantID, &product.SKU,
        &product.Name, &product.Status, &product.CreatedAt)
    return &product, err
}
```

---

## 3. Frontend Development

### 3.1 Portal Development

Portals are built as Next.js micro-frontends:

```
apps/web/
  shell/                   # Shared app shell
    app/
      layout.tsx           # Root layout with auth
      providers.tsx        # Context providers
  portals/
    manufacturer/          # Manufacturer portal pages
    distributor/           # Distributor portal pages
    retailer/              # Retailer portal pages
    ...                    # 13 total portals
  packages/
    ui/                    # Shared UI components
    api-client/            # Generated API client
    hooks/                 # Shared React hooks
```

### 3.2 Component Development

```tsx
// Example: Product Card Component
import { Card, Badge, Button } from 'antd';
import { useProductPrice } from '@erp/hooks';

interface ProductCardProps {
  product: Product;
  onAddToCart: (productId: string, quantity: number) => void;
}

export function ProductCard({ product, onAddToCart }: ProductCardProps) {
  const { price, loading } = useProductPrice(product.id);

  return (
    <Card
      cover={<img src={product.media[0]?.url} alt={product.name} />}
      actions={[
        <Button type="primary" onClick={() => onAddToCart(product.id, 1)}>
          Add to Cart
        </Button>,
      ]}
    >
      <Card.Meta
        title={product.name}
        description={
          <>
            <div>SKU: {product.sku}</div>
            <div>
              {loading ? 'Loading...' : `${price.currency} ${price.amount}`}
            </div>
            <Badge
              status={product.stock > 0 ? 'success' : 'error'}
              text={product.stock > 0 ? 'In Stock' : 'Out of Stock'}
            />
          </>
        }
      />
    </Card>
  );
}
```

---

## 4. AI/ML Development

### 4.1 Credit Scoring Model

```python
# credit-scoring service structure
credit_scoring/
    main.py              # FastAPI app
    models/
        credit_model.py  # ML model definition
        features.py      # Feature engineering
    api/
        routes.py        # API endpoints
    training/
        train.py         # Model training pipeline
        evaluate.py      # Model evaluation
    data/
        preprocessing.py # Data cleaning
```

### 4.2 Route Optimizer

```python
# VRP solver using Google OR-Tools
from ortools.constraint_solver import routing_enums_pb2, pywrapcp

def solve_vrp(delivery_jobs, vehicles, depot):
    manager = pywrapcp.RoutingIndexManager(
        len(delivery_jobs), len(vehicles), depot
    )
    routing = pywrapcp.RoutingModel(manager)

    # Add capacity constraint
    routing.AddDimensionWithVehicleCapacity(
        weight_callback, 0, vehicle_capacities, True, 'Capacity'
    )

    # Add time window constraints
    routing.AddDimension(time_callback, 30, max_time, False, 'Time')

    # Solve
    solution = routing.SolveWithParameters(search_params)
    return extract_routes(manager, routing, solution)
```

---

## 5. Coding Standards

### 5.1 Go Standards

- Follow Go standard project layout
- Use `golangci-lint` with project configuration
- All exported types and functions must have doc comments
- Error handling: wrap errors with context using `fmt.Errorf("action: %w", err)`
- Context propagation: always pass `context.Context` as first parameter
- No panics in production code

### 5.2 Python Standards

- Follow PEP 8 with `ruff` linter
- Type hints required for all function signatures
- Use `pydantic` for request/response validation
- Async handlers for I/O-bound operations

### 5.3 Frontend Standards

- TypeScript strict mode enabled
- ESLint + Prettier configuration enforced
- Component naming: PascalCase
- Hook naming: camelCase with `use` prefix
- Props interface naming: `{ComponentName}Props`

---

## 6. AIDD Guardrails for Development

Per `erp/aidd.guardrails.yaml`, developers must respect:

1. **Autonomous actions** (AI can perform without approval): read-only queries, low-risk notifications
2. **Supervised actions** (require human review): data mutations, workflow automation, bulk operations
3. **Prohibited actions** (never permitted): cross-tenant data access, irreversible delete without backup, privilege escalation
4. **Controls**: human-in-the-loop for high-risk, decision logging enabled, 24-hour rollback window
