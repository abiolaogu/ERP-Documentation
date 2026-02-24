# ERP-SCM Developer Guide

## 1. Overview

This guide helps developers set up their local environment, understand the codebase structure, follow coding conventions, and contribute to ERP-SCM effectively.

---

## 2. Project Structure

```
ERP-SCM/
├── backend/                    # Python FastAPI backend
│   ├── app/
│   │   ├── ai/                # AI/ML engines
│   │   │   ├── demand_forecaster.py    # ES + RF ensemble forecasting
│   │   │   ├── supplier_risk.py        # Risk scoring + Isolation Forest
│   │   │   ├── anomaly_detector.py     # Z-score + rule-based detection
│   │   │   ├── route_optimizer.py      # NN + 2-opt routing
│   │   │   └── insights_engine.py      # BI and KPI generation
│   │   ├── api/routes/        # REST API endpoint definitions
│   │   │   ├── auth.py
│   │   │   ├── products.py
│   │   │   ├── inventory.py
│   │   │   ├── suppliers.py
│   │   │   ├── orders.py
│   │   │   ├── shipments.py
│   │   │   └── ai_insights.py
│   │   ├── models/            # SQLAlchemy ORM models
│   │   │   └── models.py
│   │   ├── schemas/           # Pydantic request/response schemas
│   │   │   └── schemas.py
│   │   ├── core/              # Configuration and security
│   │   │   ├── config.py
│   │   │   └── security.py
│   │   ├── db/                # Database session management
│   │   │   ├── base.py
│   │   │   └── session.py
│   │   ├── services/          # Business logic layer
│   │   └── main.py           # FastAPI application entry point
│   ├── tests/                 # Test suite
│   ├── seed_data.py          # Demo data seeder
│   ├── requirements.txt      # Python dependencies
│   └── Dockerfile
├── frontend/                  # React + TypeScript frontend
│   ├── src/
│   │   ├── pages/            # Page components
│   │   │   ├── Dashboard.tsx
│   │   │   ├── Products.tsx
│   │   │   ├── Inventory.tsx
│   │   │   ├── Suppliers.tsx
│   │   │   ├── Orders.tsx
│   │   │   ├── Logistics.tsx
│   │   │   ├── AICenter.tsx
│   │   │   └── Alerts.tsx
│   │   ├── services/         # API client
│   │   │   └── api.ts
│   │   ├── hooks/            # Custom React hooks
│   │   │   └── useApi.ts
│   │   ├── types/            # TypeScript interfaces
│   │   │   └── index.ts
│   │   ├── utils/            # Formatting utilities
│   │   │   └── format.ts
│   │   ├── App.tsx           # Root component with routing
│   │   └── main.tsx          # Vite entry point
│   ├── package.json
│   ├── tsconfig.json
│   ├── vite.config.ts
│   └── tailwind.config.js
├── services/                  # Microservice definitions
│   ├── procurement-service/
│   ├── inventory-service/
│   ├── warehouse-service/
│   ├── manufacturing-service/
│   ├── demand-planning-service/
│   ├── logistics-service/
│   ├── quality-service/
│   ├── fleet-service/
│   └── supplier-portal-service/
├── docs/                      # Project docs
├── configs/                   # Configuration files
├── docker-compose.yml         # Container orchestration
├── Makefile                   # Build commands
└── README.md
```

---

## 3. Local Development Setup

### 3.1 Backend

```bash
# Navigate to project
cd /Users/AbiolaOgunsakin1/ERP/ERP-SCM

# Create virtual environment
cd backend
python3.11 -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Seed demo data
python -m seed_data

# Run the development server
uvicorn app.main:app --reload --port 8000

# Verify: http://localhost:8000/docs
# Login: admin@scm.io / admin123
```

### 3.2 Frontend

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-SCM/frontend
npm install
npm run dev

# Verify: http://localhost:5173
```

### 3.3 Docker

```bash
cd /Users/AbiolaOgunsakin1/ERP/ERP-SCM
docker-compose up --build

# Frontend: http://localhost:3000
# Backend: http://localhost:8000/docs
```

---

## 4. Coding Conventions

### 4.1 Python (Backend)

```python
# File naming: snake_case.py
# Class naming: PascalCase
# Function naming: snake_case
# Constant naming: UPPER_SNAKE_CASE
# Variable naming: snake_case

# Type hints required on all functions
def calculate_risk_score(
    supplier_id: int,
    weights: dict[str, float],
) -> dict[str, Any]:
    ...

# Docstrings on all public classes and functions
class DemandForecaster:
    """
    AI Demand Forecasting Engine.

    Uses multiple statistical and ML models to predict future demand:
    - Exponential Smoothing (Holt-Winters)
    - Random Forest ensemble
    - Weighted model averaging for robust predictions
    """
    ...

# Pydantic schemas for all API inputs/outputs
class ProductCreate(BaseModel):
    sku: str = Field(..., min_length=1, max_length=100)
    name: str = Field(..., min_length=1, max_length=255)
    unit_price: float = Field(..., gt=0)
    unit_cost: float = Field(..., gt=0)
```

### 4.2 TypeScript (Frontend)

```typescript
// File naming: PascalCase.tsx for components, camelCase.ts for utilities
// Component naming: PascalCase
// Function naming: camelCase
// Interface naming: PascalCase (no "I" prefix)
// Constants: UPPER_SNAKE_CASE

// Explicit typing on all exports
interface Product {
  id: number;
  sku: string;
  name: string;
  unitPrice: number;
  unitCost: number;
  isActive: boolean;
}

// Functional components with explicit return types
export default function ProductList(): JSX.Element {
  const { data, loading } = useApi<Product[]>(() => api.getProducts(), []);
  // ...
}

// Custom hooks with "use" prefix
function useApi<T>(fetcher: () => Promise<T>, deps: any[]) {
  // ...
}
```

### 4.3 SQL & Migrations

```sql
-- Table naming: snake_case, plural
-- Column naming: snake_case
-- Index naming: ix_{table}_{columns}
-- Constraint naming: chk_{table}_{description}
-- Foreign key naming: fk_{table}_{ref_table}

-- Always include tenant_id
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    ...
    CONSTRAINT chk_products_price CHECK (unit_price >= 0)
);

CREATE INDEX ix_products_tenant_sku ON products(tenant_id, sku);
```

---

## 5. Adding a New Feature

### Step 1: Define the Model

```python
# backend/app/models/models.py
class NewEntity(Base):
    __tablename__ = "new_entities"

    id = Column(Integer, primary_key=True, index=True)
    tenant_id = Column(UUID, nullable=False, index=True)
    name = Column(String(255), nullable=False)
    # ... fields
    created_at = Column(DateTime, default=utcnow)
```

### Step 2: Create Pydantic Schemas

```python
# backend/app/schemas/schemas.py
class NewEntityCreate(BaseModel):
    name: str

class NewEntityResponse(BaseModel):
    id: int
    name: str
    created_at: datetime

    class Config:
        from_attributes = True
```

### Step 3: Create API Route

```python
# backend/app/api/routes/new_entity.py
router = APIRouter(prefix="/new-entities", tags=["New Entity"])

@router.get("/", response_model=List[NewEntityResponse])
def list_entities(db: Session = Depends(get_db)):
    return db.query(NewEntity).all()

@router.post("/", response_model=NewEntityResponse, status_code=201)
def create_entity(data: NewEntityCreate, db: Session = Depends(get_db)):
    entity = NewEntity(**data.dict())
    db.add(entity)
    db.commit()
    db.refresh(entity)
    return entity
```

### Step 4: Register the Route

```python
# backend/app/main.py
from app.api.routes import new_entity
app.include_router(new_entity.router, prefix="/api")
```

### Step 5: Add Frontend Page

```typescript
// frontend/src/pages/NewEntity.tsx
export default function NewEntity() {
  const { data, loading } = useApi(() => api.getNewEntities(), []);
  // ... render table and forms
}
```

### Step 6: Write Tests

```python
# backend/tests/test_new_entity.py
def test_create_new_entity(client, db_session):
    response = client.post("/api/new-entities", json={"name": "Test"})
    assert response.status_code == 201
    assert response.json()["name"] == "Test"
```

---

## 6. Running Tests

```bash
# All tests
cd /Users/AbiolaOgunsakin1/ERP/ERP-SCM/backend
pytest tests/ -v

# With coverage
pytest tests/ -v --cov=app --cov-report=html

# Specific test file
pytest tests/test_demand_forecaster.py -v

# Frontend tests
cd /Users/AbiolaOgunsakin1/ERP/ERP-SCM/frontend
npm test
```

---

## 7. API Documentation

The API is self-documenting via FastAPI:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc
- **OpenAPI JSON**: http://localhost:8000/openapi.json

---

## 8. Git Workflow

```mermaid
gitgraph
    commit id: "main"
    branch feature/scm-123-add-quality-plans
    checkout feature/scm-123-add-quality-plans
    commit id: "Add quality plan model"
    commit id: "Add quality plan API"
    commit id: "Add quality plan UI"
    commit id: "Add tests"
    checkout main
    merge feature/scm-123-add-quality-plans
    commit id: "Release v1.1.0"
```

### Branch Naming

```
feature/SCM-{ticket}-{short-description}
fix/SCM-{ticket}-{short-description}
chore/SCM-{ticket}-{short-description}
```

### Commit Messages

```
feat(procurement): add 3-way matching workflow

- Implement PO/receipt/invoice comparison logic
- Add tolerance-based auto-matching (2% default)
- Create exception handling for mismatches
- Add API endpoint POST /v1/procurement/three-way-match

Closes SCM-456
```

---

## 9. Useful Commands

```bash
# Backend
make run-backend       # Start backend server
make test              # Run all tests
make lint              # Run ruff linter
make seed              # Seed demo data

# Frontend
make run-frontend      # Start frontend dev server
make build-frontend    # Production build

# Docker
make up                # docker-compose up --build
make down              # docker-compose down
make logs              # docker-compose logs -f
```
