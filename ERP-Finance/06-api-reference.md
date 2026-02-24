# ERP-Finance API Reference

## Document Information

| Field | Value |
|-------|-------|
| Module | ERP-Finance |
| Document Type | API Reference |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |
| Base URL | `https://api.erp.example.com/finance` |

## Authentication

All API requests require a valid JWT Bearer token from ERP-IAM:

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
X-Tenant-ID: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json
```

## API Architecture

```mermaid
flowchart LR
    Client["API Client"] --> GW["API Gateway<br/>:8090"]

    GW --> |"/v1/general-ledger"| GL["GL Service"]
    GW --> |"/v1/accounts-payable"| AP["AP Service"]
    GW --> |"/v1/accounts-receivable"| AR["AR Service"]
    GW --> |"/api/v1/plans<br/>/api/v1/subscriptions<br/>/api/v1/invoices"| BILL["Billing Service<br/>:8089"]
    GW --> |"/api/v1/payments<br/>/api/v1/transactions<br/>/api/v1/wallets"| PAY["Payments Service<br/>:8084"]
    GW --> |"/api/v1/assets<br/>/api/v1/maintenance<br/>/api/v1/ai"| ASSET["Asset Mgmt<br/>:8096"]
    GW --> |"/v1/tax-management"| TAX["Tax Service"]
    GW --> |"/v1/expense-management"| EXP["Expense Service"]
    GW --> |"/v1/treasury"| TREAS["Treasury Service"]
    GW --> |"/v1/budget"| BUD["Budget Service"]
```

## Gateway Endpoints

### Health Check

```
GET /healthz
```

**Response 200:**
```json
{
  "status": "ok",
  "module": "ERP-Finance"
}
```

### Module Capabilities

```
GET /v1/capabilities
```

**Response 200:**
```json
{
  "module": "ERP-Finance",
  "capabilities": [
    "billing", "payments", "fixed_assets", "general_ledger",
    "ap", "ar", "tax", "expense", "treasury", "budgeting_forecasting"
  ]
}
```

## Billing Service API

### Plans

#### List Plans
```
GET /api/v1/plans
```

**Response 200:**
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "name": "Pro Plan",
    "description": "Professional tier",
    "price": 9900,
    "currency": "NGN",
    "billing_period": "monthly",
    "features": ["basic_security", "ztna", "casb"],
    "limits": {"users": 50, "bandwidth_gb": 100},
    "status": "active",
    "created_at": "2026-02-23T10:00:00Z"
  }
]
```

#### Create Plan
```
POST /api/v1/plans
```

**Request Body:**
```json
{
  "name": "Enterprise Plan",
  "description": "Full enterprise features",
  "price": 49900,
  "billing_period": "monthly",
  "features": ["basic_security", "ztna", "casb", "dlp", "siem"],
  "limits": {"users": 500, "bandwidth_gb": 1000}
}
```

**Response 201:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440002",
  "name": "Enterprise Plan",
  "description": "Full enterprise features",
  "price": 49900,
  "currency": "NGN",
  "billing_period": "monthly",
  "features": ["basic_security", "ztna", "casb", "dlp", "siem"],
  "limits": {"users": 500, "bandwidth_gb": 1000},
  "status": "active",
  "created_at": "2026-02-23T10:00:00Z"
}
```

### Subscriptions

#### Create Subscription
```
POST /api/v1/subscriptions
```

**Request Body:**
```json
{
  "tenant_id": "550e8400-e29b-41d4-a716-446655440000",
  "plan_id": "550e8400-e29b-41d4-a716-446655440001"
}
```

**Response 201:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440003",
  "tenant_id": "550e8400-e29b-41d4-a716-446655440000",
  "plan_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "active",
  "current_period_start": "2026-02-23T10:00:00Z",
  "current_period_end": "2026-03-25T10:00:00Z",
  "cancel_at_period_end": false,
  "created_at": "2026-02-23T10:00:00Z",
  "updated_at": "2026-02-23T10:00:00Z"
}
```

#### Cancel Subscription
```
POST /api/v1/subscriptions/:id/cancel
```

### Usage Metering

#### Record Usage
```
POST /api/v1/usage
```

**Request Body:**
```json
{
  "subscription_id": "550e8400-e29b-41d4-a716-446655440003",
  "metric": "api_requests",
  "quantity": 1500,
  "metadata": {"endpoint": "/v1/data", "status": 200}
}
```

### Invoices

#### Generate Invoices
```
POST /api/v1/invoices/generate
```

**Response 200:**
```json
{
  "generated": 42
}
```

## Payments Service API

### Payment Initiation

```
POST /api/v1/payments/initiate
```

**Request Body:**
```json
{
  "amount": 50000,
  "currency": "NGN",
  "email": "customer@example.com",
  "customer_id": "550e8400-e29b-41d4-a716-446655440010",
  "payment_method": "card",
  "callback_url": "https://app.example.com/payment/callback",
  "metadata": {"invoice_id": "INV-001234"}
}
```

**Response 200:**
```json
{
  "reference": "TXN-019523a4-5678-7def-8901-234567890abc",
  "authorization_url": "https://checkout.paystack.com/TXN-019523a4...",
  "status": "pending"
}
```

### Refunds

```
POST /api/v1/refunds
```

**Request Body:**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440020",
  "amount": 25000,
  "reason": "Customer dissatisfied with service"
}
```

### Wallets

#### Create Wallet
```
POST /api/v1/wallets
```

**Request Body:**
```json
{
  "customer_id": "550e8400-e29b-41d4-a716-446655440010"
}
```

#### Wallet Transfer
```
POST /api/v1/transfers
```

**Request Body:**
```json
{
  "from_wallet_id": "550e8400-e29b-41d4-a716-446655440030",
  "to_wallet_id": "550e8400-e29b-41d4-a716-446655440031",
  "amount": 10000,
  "description": "Settlement transfer"
}
```

## Asset Management API

### Assets

#### Create Asset
```
POST /api/v1/assets
```

**Request Body:**
```json
{
  "name": "Dell Server R750",
  "asset_tag": "IT-SRV-001",
  "description": "Primary database server",
  "category": "it_equipment",
  "manufacturer": "Dell",
  "model_number": "PowerEdge R750",
  "serial_number": "SN-12345",
  "location": "Data Center A, Rack 12",
  "department": "IT Operations",
  "purchase_price": 15000.00,
  "purchase_date": "2025-01-15",
  "warranty_expiry": "2028-01-15",
  "salvage_value": 1500.00,
  "useful_life_years": 5,
  "depreciation_method": "straight_line"
}
```

### AI Endpoints

#### Asset Health Analysis
```
POST /api/v1/ai/health/:asset_id
```

**Response 200:**
```json
{
  "analysis_type": "asset_health",
  "summary": "Asset is in good condition with moderate risk due to approaching warranty expiry",
  "recommendations": [
    "Renew warranty before January 2028",
    "Schedule preventative maintenance for Q3 2026",
    "Consider RAM upgrade for performance optimization"
  ],
  "risk_level": "medium",
  "confidence_score": 0.87,
  "details": {
    "health_score": 82,
    "maintenance_compliance": "Good - 90% on-time completion",
    "depreciation_status": "On track, 40% depreciated",
    "remaining_value_assessment": "Above expected curve",
    "failure_risk_factors": ["Warranty expiry approaching", "High utilization hours"],
    "optimization_opportunities": ["Extend lifecycle with memory upgrade"]
  }
}
```

## Error Codes

| HTTP Status | Error Code | Description |
|-------------|-----------|-------------|
| 400 | BAD_REQUEST | Invalid request parameters |
| 401 | UNAUTHORIZED | Missing or invalid JWT token |
| 403 | FORBIDDEN | Insufficient permissions |
| 404 | NOT_FOUND | Resource not found |
| 409 | CONFLICT | Resource conflict (duplicate) |
| 422 | VALIDATION_ERROR | Business rule validation failure |
| 429 | RATE_LIMITED | Too many requests |
| 500 | INTERNAL_ERROR | Unexpected server error |

## Pagination

All list endpoints support cursor-based pagination:

```
GET /api/v1/transactions?page=2&per_page=50&status=completed&from_date=2026-01-01T00:00:00Z
```

| Parameter | Type | Default | Max |
|-----------|------|---------|-----|
| page | integer | 1 | - |
| per_page | integer | 20 | 100 |
| status | string | - | - |
| from_date | ISO 8601 | - | - |
| to_date | ISO 8601 | - | - |
