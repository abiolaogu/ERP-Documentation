# ERP-HCM API Documentation

## OpenAPI 3.1 Specification

**Version**: 1.0.0
**Base URL**: `https://api.peopleforce.io/v1` (Production) | `http://localhost:8090/v1` (Development)

---

## 1. Overview

The ERP-HCM API is a RESTful HTTP API that provides programmatic access to the Human Capital Management platform. All business endpoints require JWT authentication and tenant scoping via the `X-Tenant-ID` header. The API follows resource-oriented design with consistent patterns across all 14 services.

### 1.1 API Conventions

| Convention | Detail |
|-----------|--------|
| Protocol | HTTPS (TLS 1.2+) in production |
| Content-Type | `application/json` |
| Authentication | `Authorization: Bearer <JWT>` (RS256) |
| Multi-Tenancy | `X-Tenant-ID: <UUID>` header on all business endpoints |
| Pagination | `?page=1&page_size=25` (default 25, max 100) |
| Sorting | `?sort=created_at&order=desc` |
| Filtering | `?status=active&department_id=<UUID>` |
| Date Format | ISO 8601 (`2026-02-23T14:30:00Z`) |
| Monetary Values | Integer (kobo/cents), not floating point |
| IDs | UUID v4 |

### 1.2 Authentication Flow

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant AuthService
    participant Service

    Client->>Gateway: POST /v1/auth/login {email, password}
    Gateway->>AuthService: Validate credentials
    AuthService-->>Gateway: {access_token, refresh_token}
    Gateway-->>Client: 200 {access_token, refresh_token}

    Client->>Gateway: GET /v1/employee (Bearer token + X-Tenant-ID)
    Gateway->>Gateway: Validate JWT (RS256 public key)
    Gateway->>Gateway: Extract tenant_id from JWT claims
    Gateway->>Service: Forward with tenant context
    Service-->>Gateway: 200 {data}
    Gateway-->>Client: 200 {data}
```

---

## 2. Core Endpoints

### 2.1 Health Check

```
GET /healthz
```

**Response** `200 OK`:
```json
{
  "status": "ok",
  "module": "ERP-HCM",
  "version": "1.0.0",
  "uptime": "72h15m30s"
}
```

### 2.2 Capabilities

```
GET /v1/capabilities
```

**Response** `200 OK`:
```json
{
  "module": "ERP-HCM",
  "capabilities": [
    "core_hr",
    "payroll",
    "recruitment",
    "performance",
    "attendance",
    "shift_scheduling",
    "field_workforce"
  ]
}
```

---

## 3. Authentication Endpoints

### 3.1 Login

```
POST /v1/auth/login
```

**Request Body**:
```json
{
  "email": "admin@company.com",
  "password": "SecureP@ss123"
}
```

**Response** `200 OK`:
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJSUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 900,
  "user": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "admin@company.com",
    "role": "hr_admin",
    "tenant_id": "00000000-0000-0000-0000-000000000001",
    "mfa_required": false
  }
}
```

**Error Responses**:
- `401 Unauthorized`: Invalid credentials
- `403 Forbidden`: Account locked (5 failed attempts, 30min lockout)
- `428 Precondition Required`: MFA verification required

### 3.2 MFA Verify

```
POST /v1/auth/mfa/verify
```

**Request Body**:
```json
{
  "mfa_token": "eyJhbGciOiJSUzI1...",
  "totp_code": "123456"
}
```

### 3.3 Refresh Token

```
POST /v1/auth/refresh
```

**Request Body**:
```json
{
  "refresh_token": "eyJhbGciOiJSUzI1NiIs..."
}
```

### 3.4 Logout

```
POST /v1/auth/logout
```

---

## 4. Employee Service API

**Base Path**: `/v1/employee`

### 4.1 List Employees

```
GET /v1/employee
```

**Query Parameters**:

| Parameter | Type | Description |
|-----------|------|-------------|
| page | int | Page number (default: 1) |
| page_size | int | Items per page (default: 25, max: 100) |
| status | string | Filter by status: `active`, `on_leave`, `terminated` |
| department_id | UUID | Filter by department |
| location_id | UUID | Filter by location |
| employment_type | string | `full_time`, `part_time`, `contract`, `intern` |
| search | string | Full-text search on name and employee number |
| sort | string | Sort field: `created_at`, `hire_date`, `last_name` |
| order | string | `asc` or `desc` |

**Response** `200 OK`:
```json
{
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440001",
      "employee_number": "EMP-001",
      "first_name": "Adebayo",
      "last_name": "Okonkwo",
      "work_email": "adebayo.okonkwo@company.com",
      "status": "active",
      "employment_type": "full_time",
      "department": {
        "id": "...",
        "name": "Engineering"
      },
      "job_title": {
        "id": "...",
        "title": "Senior Software Engineer"
      },
      "location": {
        "id": "...",
        "name": "Lagos HQ"
      },
      "hire_date": "2024-03-15",
      "reports_to": {
        "id": "...",
        "name": "Funke Adeyemi"
      }
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 25,
    "total": 142,
    "total_pages": 6
  }
}
```

### 4.2 Create Employee

```
POST /v1/employee
```

**Request Body**:
```json
{
  "first_name": "Adebayo",
  "last_name": "Okonkwo",
  "work_email": "adebayo.okonkwo@company.com",
  "personal_email": "adebayo@gmail.com",
  "personal_phone": "+2348012345678",
  "date_of_birth": "1990-05-15",
  "gender": "male",
  "nationality": "Nigerian",
  "state_of_origin": "Lagos",
  "employment_type": "full_time",
  "department_id": "550e8400-e29b-41d4-a716-446655440010",
  "job_title_id": "550e8400-e29b-41d4-a716-446655440020",
  "location_id": "550e8400-e29b-41d4-a716-446655440030",
  "hire_date": "2026-03-01",
  "reports_to_id": "550e8400-e29b-41d4-a716-446655440040",
  "current_salary": 500000000,
  "salary_currency": "NGN"
}
```

**Response** `201 Created`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "employee_number": "EMP-143",
  "status": "pre_boarding",
  "event_topic": "erp.hcm.employee.created"
}
```

### 4.3 Get Employee

```
GET /v1/employee/{id}
```

### 4.4 Update Employee

```
PUT /v1/employee/{id}
```

### 4.5 Delete Employee (Soft Delete)

```
DELETE /v1/employee/{id}
```

**Response** `200 OK`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440001",
  "deleted_at": "2026-02-23T14:30:00Z",
  "event_topic": "erp.hcm.employee.deleted"
}
```

### 4.6 Bulk Import

```
POST /v1/employee/bulk-import
Content-Type: multipart/form-data
```

**Form Fields**:
- `file`: CSV or Excel file
- `import_type`: `employees` | `updates`
- `dry_run`: `true` | `false`

### 4.7 Org Chart

```
GET /v1/employee/org-chart?root_id={id}&depth=3
```

---

## 5. Payroll Service API

**Base Path**: `/v1/payroll`

### 5.1 Create Payroll Period

```
POST /v1/payroll/periods
```

**Request Body**:
```json
{
  "period_name": "February 2026",
  "period_year": 2026,
  "period_month": 2,
  "start_date": "2026-02-01",
  "end_date": "2026-02-28",
  "pay_date": "2026-02-28",
  "payroll_group_id": "550e8400-e29b-41d4-a716-446655440050"
}
```

### 5.2 Initiate Payroll Run

```
POST /v1/payroll/runs
```

**Request Body**:
```json
{
  "period_id": "550e8400-e29b-41d4-a716-446655440060",
  "run_type": "regular",
  "description": "February 2026 regular payroll"
}
```

**Response** `201 Created`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440070",
  "run_number": "PR-2026-02-001",
  "status": "draft",
  "total_employees": 142
}
```

### 5.3 Process Payroll Run

```
POST /v1/payroll/runs/{id}/process
```

Triggers the Nigerian payroll calculation engine:
- CRA computation: Higher of (NGN 200,000 or 1% of gross) + 20% of gross
- PAYE tax bands: 7%, 11%, 15%, 19%, 21%, 24%
- Pension: Employee 8% + Employer 10%
- NHF: 2.5% of basic salary
- NSITF and NHIS statutory deductions

**Response** `200 OK`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440070",
  "status": "pending_approval",
  "summary": {
    "total_gross": 71200000000,
    "total_net": 55840000000,
    "total_paye": 8320000000,
    "total_pension_employee": 5696000000,
    "total_pension_employer": 7120000000,
    "total_nhf": 1780000000,
    "processed_employees": 142,
    "failed_employees": 0,
    "currency": "NGN"
  }
}
```

### 5.4 Approve Payroll Run

```
POST /v1/payroll/runs/{id}/approve
```

### 5.5 Get Payslip

```
GET /v1/payroll/payslips/{employee_id}?period_id={period_id}
```

### 5.6 Generate Bank File

```
POST /v1/payroll/runs/{id}/bank-file
```

**Query Parameters**:
- `format`: `nibss` | `csv`

---

## 6. Leave Service API

**Base Path**: `/v1/leave`

### 6.1 Submit Leave Request

```
POST /v1/leave/requests
```

**Request Body**:
```json
{
  "leave_type_id": "550e8400-e29b-41d4-a716-446655440080",
  "start_date": "2026-03-10",
  "end_date": "2026-03-14",
  "day_type": "full_day",
  "reason": "Family vacation",
  "delegate_to_id": "550e8400-e29b-41d4-a716-446655440090"
}
```

### 6.2 Approve/Reject Leave

```
PUT /v1/leave/requests/{id}/approve
PUT /v1/leave/requests/{id}/reject
```

### 6.3 Get Leave Balances

```
GET /v1/leave/balances/{employee_id}
```

**Response** `200 OK`:
```json
{
  "employee_id": "550e8400-e29b-41d4-a716-446655440001",
  "balances": [
    {
      "leave_type": "Annual Leave",
      "entitled": 20.0,
      "used": 5.0,
      "pending": 3.0,
      "available": 12.0,
      "carry_forward": 2.0
    },
    {
      "leave_type": "Sick Leave",
      "entitled": 10.0,
      "used": 1.0,
      "pending": 0.0,
      "available": 9.0,
      "carry_forward": 0.0
    }
  ]
}
```

---

## 7. Time & Attendance Service API

**Base Path**: `/v1/time-attendance`

### 7.1 Clock In

```
POST /v1/time-attendance/clock-in
```

**Request Body**:
```json
{
  "latitude": 6.5244,
  "longitude": 3.3792,
  "accuracy_meters": 15.0,
  "device_id": "device-abc-123"
}
```

**Validation**:
- Haversine distance check against assigned geofence (configurable radius)
- GPS accuracy must be <= 50 meters
- Teleport detection: rejects if speed exceeds 100 km/hr from last known location
- Multi-device detection: flags if different device_id from recent clock events

**Response** `200 OK`:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440100",
  "clock_in_time": "2026-02-23T08:02:15Z",
  "geofence_status": "inside",
  "distance_meters": 42.5,
  "office": "Lagos HQ"
}
```

### 7.2 Clock Out

```
POST /v1/time-attendance/clock-out
```

---

## 8. Additional Service APIs

### 8.1 Recruitment

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/recruitment/requisitions` | Create job requisition |
| GET | `/v1/recruitment/requisitions` | List requisitions |
| POST | `/v1/recruitment/candidates` | Create candidate |
| POST | `/v1/recruitment/applications` | Submit application |
| PUT | `/v1/recruitment/applications/{id}/stage` | Move pipeline stage |

### 8.2 Performance

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/performance/okr-cycles` | Create OKR cycle |
| POST | `/v1/performance/objectives` | Create objective |
| PUT | `/v1/performance/key-results/{id}/progress` | Update key result |
| POST | `/v1/performance/review-cycles` | Create review cycle |
| POST | `/v1/performance/reviews/{id}/submit` | Submit review |

### 8.3 Benefits

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/benefits/plans` | Create benefit plan |
| POST | `/v1/benefits/enrollments` | Enroll employee |
| POST | `/v1/benefits/claims` | Submit claim |
| POST | `/v1/benefits/ewa/request` | Request earned wage access |

### 8.4 Learning (LMS)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/learning/courses` | Create course |
| POST | `/v1/learning/enrollments` | Enroll learner |
| POST | `/v1/learning/completions` | Record completion |
| GET | `/v1/learning/certificates/{id}` | Get certificate |

### 8.5 Compensation

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/compensation/bands` | Create salary band |
| POST | `/v1/compensation/cycles` | Create comp cycle |
| POST | `/v1/compensation/proposals` | Submit proposal |

### 8.6 Workforce Planning

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/workforce-planning/plans` | Create headcount plan |
| POST | `/v1/workforce-planning/scenarios` | Create scenario |
| GET | `/v1/workforce-planning/analytics` | Get workforce analytics |

### 8.7 Compliance

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/compliance/status` | Compliance dashboard |
| POST | `/v1/compliance/dsar` | Data subject access request |
| GET | `/v1/compliance/audit-log` | Audit trail |

### 8.8 Document Service

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/document/upload` | Upload document |
| GET | `/v1/document/{id}` | Get document |
| POST | `/v1/document/{id}/sign` | Digital signature |

### 8.9 Facilities

| Method | Path | Description |
|--------|------|-------------|
| POST | `/v1/facilities/rooms/book` | Book room |
| POST | `/v1/facilities/desks/book` | Book desk |
| GET | `/v1/facilities/availability` | Check availability |

---

## 9. Error Response Format

All errors follow a consistent format:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "must be a valid email address"
      }
    ],
    "request_id": "req-abc-123"
  }
}
```

### 9.1 Error Codes

| HTTP Status | Code | Description |
|-------------|------|-------------|
| 400 | VALIDATION_ERROR | Request body validation failed |
| 400 | MISSING_TENANT_ID | X-Tenant-ID header missing |
| 401 | UNAUTHORIZED | Missing or invalid JWT |
| 403 | FORBIDDEN | Insufficient permissions |
| 404 | NOT_FOUND | Resource not found |
| 409 | CONFLICT | Resource already exists |
| 422 | BUSINESS_RULE_VIOLATION | Business logic constraint violated |
| 429 | RATE_LIMIT_EXCEEDED | Too many requests |
| 500 | INTERNAL_ERROR | Server error |

---

## 10. Rate Limiting

Rate limits are enforced per tenant using a token bucket algorithm:

| Tier | Requests/Minute | Burst |
|------|----------------|-------|
| Free | 60 | 10 |
| Professional | 300 | 50 |
| Enterprise | 1000 | 200 |

Response headers include:
```
X-RateLimit-Limit: 300
X-RateLimit-Remaining: 297
X-RateLimit-Reset: 1708700400
```

---

## 11. Event Notifications

All mutating operations publish CloudEvents to NATS JetStream:

```json
{
  "specversion": "1.0",
  "id": "evt-550e8400-e29b-41d4-a716-446655440000",
  "source": "erp.hcm.employee-service",
  "type": "erp.hcm.employee.created",
  "time": "2026-02-23T14:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    "tenant_id": "00000000-0000-0000-0000-000000000001",
    "employee_id": "550e8400-e29b-41d4-a716-446655440001",
    "employee_number": "EMP-143"
  }
}
```

### 11.1 Event Topics

| Topic | Description |
|-------|-------------|
| `erp.hcm.employee.created` | New employee created |
| `erp.hcm.employee.updated` | Employee record updated |
| `erp.hcm.employee.deleted` | Employee soft-deleted |
| `erp.hcm.payroll.created` | Payroll run created |
| `erp.hcm.payroll.updated` | Payroll run processed/approved |
| `erp.hcm.leave.created` | Leave request submitted |
| `erp.hcm.leave.updated` | Leave request approved/rejected |
| `erp.hcm.time-attendance.created` | Clock event recorded |
| `erp.hcm.recruitment.created` | Candidate/requisition created |
| `erp.hcm.performance.created` | Review/OKR created |
| `erp.hcm.benefits.created` | Enrollment/claim created |

---

## 12. SDK Examples

### 12.1 Go Client

```go
import "github.com/peopleforce/hrms/sdk"

client := sdk.NewClient(
    sdk.WithBaseURL("https://api.peopleforce.io"),
    sdk.WithToken("your-jwt-token"),
    sdk.WithTenantID("00000000-0000-0000-0000-000000000001"),
)

employees, err := client.Employees.List(ctx, &sdk.ListParams{
    Page:     1,
    PageSize: 25,
    Status:   "active",
})
```

### 12.2 cURL

```bash
curl -X GET "https://api.peopleforce.io/v1/employee?status=active&page=1" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIs..." \
  -H "X-Tenant-ID: 00000000-0000-0000-0000-000000000001" \
  -H "Content-Type: application/json"
```
