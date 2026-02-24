# ERP-BI API Reference

| Field | Value |
|---|---|
| Module | ERP-BI |
| Base URL | `https://api.erp.example.com/bi` |
| Version | v1 |
| Auth | Bearer JWT + X-Tenant-ID header |
| Last Updated | 2026-02-23 |

---

## 1. Authentication

All API requests require:
- `Authorization: Bearer <jwt_token>` - JWT issued by ERP-IAM
- `X-Tenant-ID: <tenant_id>` - Tenant context identifier

Missing `X-Tenant-ID` returns `400 Bad Request`:
```json
{"error": "missing X-Tenant-ID"}
```

---

## 2. Dashboard Service API

### 2.1 List Dashboards

```
GET /v1/dashboard
```

**Query Parameters**:
| Parameter | Type | Required | Description |
|---|---|---|---|
| page | integer | No | Page number (default: 1) |
| limit | integer | No | Items per page (default: 20, max: 100) |
| search | string | No | Search by name |
| tag | string | No | Filter by tag |

**Response** `200 OK`:
```json
{
  "items": [
    {
      "id": "dash_abc123",
      "name": "Sales Overview",
      "description": "Monthly sales KPIs",
      "widgets": 8,
      "createdAt": "2026-01-15T10:00:00Z",
      "updatedAt": "2026-02-20T14:30:00Z"
    }
  ],
  "total": 42,
  "page": 1,
  "limit": 20,
  "event_topic": "erp.bi.dashboard.listed"
}
```

### 2.2 Create Dashboard

```
POST /v1/dashboard
```

**Request Body**:
```json
{
  "name": "Q1 Finance Dashboard",
  "description": "Key financial metrics for Q1 2026",
  "layout": "grid",
  "widgets": [],
  "tags": ["finance", "quarterly"],
  "isPublic": false
}
```

**Response** `201 Created`:
```json
{
  "item": {
    "id": "dash_def456",
    "name": "Q1 Finance Dashboard",
    "createdAt": "2026-02-23T12:00:00Z"
  },
  "event_topic": "erp.bi.dashboard.created"
}
```

### 2.3 Get Dashboard

```
GET /v1/dashboard/{id}
```

**Response** `200 OK`:
```json
{
  "id": "dash_abc123",
  "name": "Sales Overview",
  "widgets": [...],
  "filters": [...],
  "refreshInterval": 30,
  "event_topic": "erp.bi.dashboard.read"
}
```

### 2.4 Update Dashboard

```
PUT /v1/dashboard/{id}
PATCH /v1/dashboard/{id}
```

**Response** `200 OK`:
```json
{
  "id": "dash_abc123",
  "item": { "name": "Updated Sales Overview" },
  "event_topic": "erp.bi.dashboard.updated"
}
```

### 2.5 Delete Dashboard

```
DELETE /v1/dashboard/{id}
```

**Response** `200 OK`:
```json
{
  "id": "dash_abc123",
  "event_topic": "erp.bi.dashboard.deleted"
}
```

---

## 3. Report Service API

### 3.1 List Reports
```
GET /v1/report
```

### 3.2 Create Report
```
POST /v1/report
```

**Request Body**:
```json
{
  "name": "Monthly Revenue Report",
  "type": "paginated",
  "layout": "tabular",
  "dataSource": "model_finance_revenue",
  "parameters": [
    {"name": "date_range", "type": "dateRange", "default": "last_30_days"},
    {"name": "department", "type": "select", "options": "dim_department"}
  ],
  "schedule": {
    "cron": "0 8 1 * *",
    "delivery": {
      "email": ["finance-team@company.com"],
      "slack": ["#finance-reports"],
      "webhook": null
    },
    "format": "pdf"
  }
}
```

### 3.3 Execute Report
```
POST /v1/report/{id}/execute
```

**Request Body**:
```json
{
  "parameters": {"date_range": "2026-01-01/2026-01-31", "department": "sales"},
  "format": "excel",
  "async": true
}
```

**Response** `202 Accepted`:
```json
{
  "executionId": "exec_789",
  "status": "queued",
  "estimatedTime": 15
}
```

### 3.4 Get/Update/Delete Report
```
GET /v1/report/{id}
PUT /v1/report/{id}
DELETE /v1/report/{id}
```

---

## 4. Query Engine API

### 4.1 Execute Query
```
POST /v1/query-engine
```

**Request Body**:
```json
{
  "model": "semantic_model_sales",
  "dimensions": ["product_category", "region"],
  "measures": ["total_revenue", "order_count"],
  "filters": [
    {"field": "order_date", "operator": "between", "value": ["2026-01-01", "2026-02-28"]}
  ],
  "orderBy": [{"field": "total_revenue", "direction": "desc"}],
  "limit": 100
}
```

**Response** `201 Created`:
```json
{
  "item": {
    "columns": ["product_category", "region", "total_revenue", "order_count"],
    "rows": [
      ["Electronics", "North America", 1250000, 3420],
      ["Clothing", "Europe", 890000, 5120]
    ],
    "metadata": {
      "rowCount": 24,
      "executionTimeMs": 142,
      "cacheHit": false,
      "queryId": "qry_abc123"
    }
  },
  "event_topic": "erp.bi.query-engine.created"
}
```

---

## 5. Data Modeling Service API

### 5.1 List Models
```
GET /v1/data-modeling
```

### 5.2 Create Semantic Model
```
POST /v1/data-modeling
```

**Request Body**:
```json
{
  "name": "Sales Analytics Model",
  "tables": [
    {"name": "fact_sales", "schema": "analytics", "type": "fact"},
    {"name": "dim_product", "schema": "analytics", "type": "dimension"}
  ],
  "joins": [
    {"from": "fact_sales.product_id", "to": "dim_product.id", "type": "left"}
  ],
  "measures": [
    {"name": "total_revenue", "expression": "SUM(fact_sales.amount)", "format": "currency"},
    {"name": "order_count", "expression": "COUNT(DISTINCT fact_sales.order_id)", "format": "number"}
  ],
  "dimensions": [
    {"name": "product_category", "column": "dim_product.category", "hierarchy": "product_hierarchy"}
  ],
  "rls": {
    "enabled": true,
    "policy": "iam_department_filter"
  }
}
```

---

## 6. Data Warehouse Service API

### 6.1 List Data Sources
```
GET /v1/data-warehouse
```

### 6.2 Trigger Sync
```
POST /v1/data-warehouse
```

**Request Body**:
```json
{
  "source": "erp-finance",
  "entities": ["invoices", "payments", "journal_entries"],
  "mode": "incremental",
  "since": "2026-02-22T00:00:00Z"
}
```

### 6.3 Get Data Lineage
```
GET /v1/data-warehouse/{id}
```

---

## 7. Alert Service API

### 7.1 List Alerts
```
GET /v1/alert
```

### 7.2 Create Alert Rule
```
POST /v1/alert
```

**Request Body**:
```json
{
  "name": "Revenue Drop Alert",
  "type": "threshold",
  "condition": {
    "metric": "daily_revenue",
    "model": "semantic_model_sales",
    "operator": "less_than",
    "value": 50000,
    "window": "24h"
  },
  "schedule": "*/15 * * * *",
  "notifications": {
    "email": ["cfo@company.com"],
    "slack": ["#revenue-alerts"],
    "webhook": "https://hooks.example.com/alert"
  },
  "escalation": {
    "timeout": "30m",
    "escalateTo": ["ceo@company.com"]
  }
}
```

---

## 8. NLQ Service API

### 8.1 Natural Language Query
```
POST /v1/nlq
```

**Request Body**:
```json
{
  "question": "What were the top 10 products by revenue last quarter?",
  "context": "semantic_model_sales",
  "visualize": true
}
```

**Response** `201 Created`:
```json
{
  "item": {
    "question": "What were the top 10 products by revenue last quarter?",
    "generatedSQL": "SELECT p.name, SUM(s.amount) as revenue FROM fact_sales s JOIN dim_product p ON s.product_id = p.id WHERE s.order_date BETWEEN '2025-10-01' AND '2025-12-31' GROUP BY p.name ORDER BY revenue DESC LIMIT 10",
    "results": {
      "columns": ["name", "revenue"],
      "rows": [["Widget Pro", 450000], ["Gadget X", 380000]]
    },
    "suggestedChart": "bar",
    "confidence": 0.94
  },
  "event_topic": "erp.bi.nlq.created"
}
```

---

## 9. Common Response Codes

| Code | Meaning |
|---|---|
| 200 | Success |
| 201 | Created |
| 202 | Accepted (async operation) |
| 400 | Bad Request (missing tenant ID, invalid params) |
| 401 | Unauthorized (invalid/missing JWT) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 429 | Too Many Requests (governor limit) |
| 500 | Internal Server Error |

---

## 10. Rate Limits

| Tier | Requests/min | Concurrent Queries | Max Rows |
|---|---|---|---|
| Free | 60 | 5 | 10,000 |
| Professional | 300 | 25 | 100,000 |
| Enterprise | 1,000 | 100 | 1,000,000 |
| Unlimited | No limit | 500 | 10,000,000 |

---

## 11. Health Check

All services expose:
```
GET /healthz
```

**Response**:
```json
{
  "status": "healthy",
  "module": "ERP-BI",
  "service": "dashboard-service"
}
```
