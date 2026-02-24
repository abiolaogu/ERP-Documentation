# ERP-SCM API Reference

## 1. Overview

The ERP-SCM API is a RESTful JSON API built on FastAPI. All business endpoints require JWT authentication via ERP-IAM and a tenant context header (`X-Tenant-ID`). The API is versioned at `/v1/` and follows OpenAPI 3.1 specification.

**Base URL**: `https://{host}/api` (monolith) or `https://{host}/v1/{service}` (microservices)

---

## 2. Authentication

All requests must include:

```http
Authorization: Bearer <jwt_token>
X-Tenant-ID: <tenant_uuid>
```

### Auth Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/auth/register` | Register a new user |
| `POST` | `/api/auth/login` | Login and receive JWT token |

**Login Request**:
```json
{
  "email": "admin@scm.io",
  "password": "admin123"
}
```

**Login Response**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer"
}
```

---

## 3. Health & System

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/health` | Health check |
| `GET` | `/healthz` | Kubernetes liveness probe |
| `GET` | `/v1/capabilities` | List available module capabilities |

---

## 4. Products API

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/products` | List all products (paginated) |
| `POST` | `/api/products` | Create a new product |
| `GET` | `/api/products/{id}` | Get product by ID |
| `PUT` | `/api/products/{id}` | Update product |
| `DELETE` | `/api/products/{id}` | Soft-delete product |

### Create Product

```http
POST /api/products
Content-Type: application/json

{
  "sku": "WDG-1001",
  "name": "Industrial Widget A",
  "description": "High-grade industrial widget",
  "category_id": 1,
  "unit_price": 29.99,
  "unit_cost": 15.50,
  "weight_kg": 0.75
}
```

**Response** (201):
```json
{
  "id": 1,
  "sku": "WDG-1001",
  "name": "Industrial Widget A",
  "description": "High-grade industrial widget",
  "category_id": 1,
  "unit_price": 29.99,
  "unit_cost": 15.50,
  "weight_kg": 0.75,
  "is_active": true,
  "created_at": "2026-02-23T10:00:00Z"
}
```

---

## 5. Inventory API

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/inventory` | List inventory items |
| `POST` | `/api/inventory` | Create inventory record |
| `GET` | `/api/inventory/{id}` | Get inventory item |
| `PUT` | `/api/inventory/{id}` | Update inventory item |
| `POST` | `/api/inventory/{id}/optimize-reorder` | AI-optimize reorder point |

### AI Reorder Optimization

```http
POST /api/inventory/42/optimize-reorder
```

**Response** (200):
```json
{
  "product_id": 42,
  "current_reorder_point": 10,
  "ai_reorder_point": 23,
  "current_reorder_quantity": 50,
  "ai_reorder_quantity": 87,
  "safety_stock": 15,
  "service_level": 0.95,
  "method": "statistical_eoq"
}
```

---

## 6. Supplier API

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/suppliers` | List all suppliers |
| `POST` | `/api/suppliers` | Create a supplier |
| `GET` | `/api/suppliers/{id}` | Get supplier details |
| `PUT` | `/api/suppliers/{id}` | Update supplier |
| `GET` | `/api/suppliers/{id}/risk-report` | AI risk analysis report |
| `GET` | `/api/suppliers/ai/rankings` | AI-ranked supplier list |

### Supplier Risk Report

```http
GET /api/suppliers/5/risk-report
```

**Response** (200):
```json
{
  "supplier_id": 5,
  "supplier_name": "Acme Parts Co.",
  "overall_risk_score": 0.342,
  "risk_level": "moderate",
  "factors": {
    "delivery_reliability": 0.15,
    "quality_score": 0.22,
    "price_competitiveness": 0.45,
    "financial_stability": 0.20,
    "communication": 0.30
  },
  "recommendations": [
    "Negotiate pricing or evaluate alternative suppliers for cost optimization",
    "Establish regular check-ins and automated order status updates"
  ],
  "trend": "improving"
}
```

---

## 7. Orders API

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/orders` | List orders (filterable by type, status) |
| `POST` | `/api/orders` | Create purchase or sales order |
| `GET` | `/api/orders/{id}` | Get order details with items |
| `PUT` | `/api/orders/{id}` | Update order status/details |
| `POST` | `/api/orders/{id}/approve` | Approve a pending order |
| `POST` | `/api/orders/{id}/cancel` | Cancel an order |

### Create Purchase Order

```http
POST /api/orders
Content-Type: application/json

{
  "order_type": "purchase",
  "supplier_id": 5,
  "warehouse_id": 1,
  "expected_delivery": "2026-03-15T00:00:00Z",
  "notes": "Urgent restock",
  "items": [
    {"product_id": 1, "quantity": 100, "unit_price": 15.50},
    {"product_id": 3, "quantity": 50, "unit_price": 22.00}
  ]
}
```

---

## 8. Shipments API

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/shipments` | List shipments |
| `POST` | `/api/shipments` | Create shipment |
| `GET` | `/api/shipments/{id}` | Get shipment with tracking |
| `PUT` | `/api/shipments/{id}` | Update shipment |
| `POST` | `/api/shipments/ai/optimize-route` | AI route optimization |
| `POST` | `/api/shipments/ai/consolidate` | AI shipment consolidation |

### Route Optimization

```http
POST /api/shipments/ai/optimize-route
Content-Type: application/json

{
  "stops": [
    {"id": "WH-1", "lat": 40.7128, "lng": -74.0060, "name": "NYC Warehouse"},
    {"id": "C-1", "lat": 40.7580, "lng": -73.9855, "name": "Customer A"},
    {"id": "C-2", "lat": 40.6892, "lng": -74.0445, "name": "Customer B"},
    {"id": "C-3", "lat": 40.7282, "lng": -73.7949, "name": "Customer C"}
  ]
}
```

**Response** (200):
```json
{
  "optimized_order": [
    {"id": "WH-1", "lat": 40.7128, "lng": -74.0060, "name": "NYC Warehouse"},
    {"id": "C-2", "lat": 40.6892, "lng": -74.0445, "name": "Customer B"},
    {"id": "C-1", "lat": 40.7580, "lng": -73.9855, "name": "Customer A"},
    {"id": "C-3", "lat": 40.7282, "lng": -73.7949, "name": "Customer C"}
  ],
  "total_distance_km": 42.15,
  "estimated_time_hours": 1.05,
  "stops_count": 4
}
```

---

## 9. AI & Analytics API

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/ai/dashboard/kpis` | Dashboard KPI summary |
| `GET` | `/api/ai/dashboard/health` | Supply chain health score |
| `GET` | `/api/ai/insights` | AI-generated business insights |
| `POST` | `/api/ai/forecast/{product_id}` | Generate demand forecast |
| `POST` | `/api/ai/anomalies/detect` | Run anomaly detection |
| `GET` | `/api/ai/alerts` | List AI alerts |
| `PUT` | `/api/ai/alerts/{id}/resolve` | Resolve an alert |

### Dashboard KPIs

```http
GET /api/ai/dashboard/kpis
```

**Response** (200):
```json
{
  "total_products": 156,
  "total_suppliers": 24,
  "total_orders": 1842,
  "pending_orders": 127,
  "total_inventory_value": 2456789.50,
  "low_stock_items": 12,
  "active_shipments": 45,
  "avg_supplier_score": 0.823,
  "total_revenue": 5678900.00,
  "total_cost": 3456789.00,
  "fulfillment_rate": 94.5,
  "ai_alerts_count": 8
}
```

### Demand Forecast

```http
POST /api/ai/forecast/42?horizon_days=30
```

**Response** (200):
```json
[
  {
    "id": 1,
    "product_id": 42,
    "forecast_date": "2026-02-24",
    "predicted_quantity": 127.5,
    "lower_bound": 89.3,
    "upper_bound": 165.7,
    "confidence": 0.847,
    "model_used": "ensemble_es_rf",
    "created_at": "2026-02-23T10:00:00Z"
  }
]
```

---

## 10. Procurement Extended API

| Method | Path | Description |
|---|---|---|
| `GET` | `/v1/procurement/requisitions` | List purchase requisitions |
| `POST` | `/v1/procurement/requisitions` | Create requisition |
| `POST` | `/v1/procurement/requisitions/{id}/approve` | Approve requisition |
| `GET` | `/v1/procurement/rfqs` | List RFQ events |
| `POST` | `/v1/procurement/rfqs` | Create RFQ |
| `POST` | `/v1/procurement/rfqs/{id}/responses` | Submit RFQ response |
| `POST` | `/v1/procurement/rfqs/{id}/evaluate` | AI vendor selection |
| `GET` | `/v1/procurement/contracts` | List contracts |
| `POST` | `/v1/procurement/three-way-match` | Execute 3-way match |

## 11. Manufacturing API

| Method | Path | Description |
|---|---|---|
| `GET` | `/v1/manufacturing/boms` | List BOMs |
| `POST` | `/v1/manufacturing/boms` | Create BOM |
| `GET` | `/v1/manufacturing/boms/{id}/explode` | Multi-level BOM explosion |
| `POST` | `/v1/manufacturing/production-orders` | Create production order |
| `POST` | `/v1/manufacturing/mrp/run` | Execute MRP run |
| `GET` | `/v1/manufacturing/work-centers` | List work centers |
| `GET` | `/v1/manufacturing/schedule` | Get production schedule (Gantt data) |

## 12. Quality API

| Method | Path | Description |
|---|---|---|
| `POST` | `/v1/quality/inspections` | Create inspection |
| `POST` | `/v1/quality/inspections/{id}/results` | Record inspection results |
| `POST` | `/v1/quality/ncrs` | Create NCR |
| `POST` | `/v1/quality/ncrs/{id}/capa` | Create CAPA from NCR |
| `GET` | `/v1/quality/spc/{product_id}` | Get SPC data points |

## 13. Fleet API

| Method | Path | Description |
|---|---|---|
| `GET` | `/v1/fleet/vehicles` | List vehicles |
| `POST` | `/v1/fleet/vehicles` | Register vehicle |
| `GET` | `/v1/fleet/vehicles/{id}/maintenance` | Get maintenance history |
| `POST` | `/v1/fleet/trips` | Create trip |
| `GET` | `/v1/fleet/drivers` | List drivers |
| `GET` | `/v1/fleet/fuel/summary` | Fuel consumption summary |

## 14. Warehouse API

| Method | Path | Description |
|---|---|---|
| `GET` | `/v1/warehouse/layout/{warehouse_id}` | Get warehouse layout |
| `POST` | `/v1/warehouse/receiving` | Create receiving order |
| `POST` | `/v1/warehouse/pick-waves` | Create pick wave |
| `GET` | `/v1/warehouse/pick-waves/{id}/tasks` | Get pick tasks |
| `POST` | `/v1/warehouse/pack/{task_id}` | Record packing |
| `POST` | `/v1/warehouse/ship/{shipment_id}` | Confirm shipping |

## 15. Supplier Portal API

| Method | Path | Description |
|---|---|---|
| `GET` | `/v1/supplier-portal/purchase-orders` | View assigned POs |
| `POST` | `/v1/supplier-portal/purchase-orders/{id}/acknowledge` | Acknowledge PO |
| `POST` | `/v1/supplier-portal/asn` | Submit ASN |
| `POST` | `/v1/supplier-portal/invoices` | Submit invoice |
| `GET` | `/v1/supplier-portal/payments` | Check payment status |

---

## 16. Common Query Parameters

| Parameter | Type | Description | Example |
|---|---|---|---|
| `page` | int | Page number (1-based) | `?page=2` |
| `page_size` | int | Items per page (max 100) | `?page_size=50` |
| `sort` | string | Sort field | `?sort=-created_at` |
| `search` | string | Full-text search | `?search=widget` |
| `status` | string | Filter by status | `?status=pending` |
| `date_from` | datetime | Date range start | `?date_from=2026-01-01` |
| `date_to` | datetime | Date range end | `?date_to=2026-03-01` |

## 17. Error Codes

| Code | HTTP Status | Description |
|---|---|---|
| `AUTH_REQUIRED` | 401 | Missing or invalid JWT |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 422 | Request body validation failed |
| `INVENTORY_INSUFFICIENT` | 409 | Not enough stock |
| `DUPLICATE_ENTITY` | 409 | SKU/code already exists |
| `STALE_DATA` | 409 | Optimistic lock conflict |
| `AI_MODEL_UNAVAILABLE` | 503 | ML model temporarily unavailable |

## 18. Rate Limiting

| Tier | Limit | Window |
|---|---|---|
| Standard | 1000 requests | Per minute |
| AI endpoints | 100 requests | Per minute |
| Bulk operations | 10 requests | Per minute |
| Supplier portal | 500 requests | Per minute |
