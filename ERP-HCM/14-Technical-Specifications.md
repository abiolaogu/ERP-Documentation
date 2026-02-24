# ERP-HCM Technical Specifications

## Version 1.0.0 | Date: 2026-02-23

---

## 1. API Contracts

### 1.1 Base URL and Versioning
- Base: `https://api.{tenant}.erp-hcm.com/v1`
- Versioning: URI path (`/v1/`, `/v2/`)
- Content-Type: `application/json`
- Authentication: `Authorization: Bearer <JWT>`
- Tenant: `X-Tenant-ID: <UUID>`

### 1.2 Standard Response Envelope

```json
{
  "data": {},
  "meta": {
    "page": 1,
    "page_size": 20,
    "total": 150,
    "total_pages": 8
  },
  "errors": []
}
```

### 1.3 Employee API Contract

#### POST /v1/employee
```json
{
  "employee_number": "EMP-2026-0042",
  "first_name": "Adebayo",
  "last_name": "Ogundimu",
  "email": "adebayo@company.com",
  "employment_type": "permanent",
  "hire_date": "2026-03-01",
  "department_id": "uuid",
  "position_id": "uuid",
  "bank_details": {
    "bank_name": "GTBank",
    "bank_code": "058",
    "account_number": "0123456789",
    "account_name": "Adebayo Ogundimu"
  },
  "tax_info": {
    "tax_id": "TIN-12345678",
    "tax_office": "Lagos FIRS"
  },
  "pension_info": {
    "rsa_pin": "PEN100012345678",
    "pension_provider": "ARM Pension",
    "employee_rate": 0.08,
    "employer_rate": 0.10
  }
}
```

#### Response: 201 Created
```json
{
  "data": {
    "id": "b7e4c9a2-...",
    "tenant_id": "a1b2c3d4-...",
    "employee_number": "EMP-2026-0042",
    "first_name": "Adebayo",
    "last_name": "Ogundimu",
    "employment_status": "active",
    "created_at": "2026-02-23T10:00:00Z"
  }
}
```

### 1.4 Payroll API Contract

#### POST /v1/payroll/runs
```json
{
  "period_id": "uuid",
  "run_type": "regular",
  "description": "February 2026 Regular Payroll"
}
```

#### POST /v1/payroll/runs/{id}/process
```json
{
  "run_id": "uuid",
  "recalculate_all": true
}
```

#### Response: PayrollRunResult
```json
{
  "data": {
    "total_employees": 500,
    "processed_employees": 498,
    "failed_employees": 2,
    "total_gross": 150000000,
    "total_net": 105000000,
    "total_deductions": 45000000,
    "total_paye": 25000000,
    "total_pension_employee": 12000000,
    "total_pension_employer": 15000000,
    "total_nhf": 3750000,
    "currency": "NGN",
    "validation_passed": true
  }
}
```

---

## 2. Database Table Inventory

### 2.1 Schema Summary

| Schema | Tables | Description |
|--------|--------|-------------|
| `employee` | 15+ | Employee, departments, positions, locations, legal entities |
| `payroll` | 20+ | Pay grades, salary components, payroll runs, entries, items |
| `leave` | 8+ | Leave types, requests, balances, holidays |
| `attendance` | 10+ | Clock records, geofences, shifts, biometric templates |
| `recruitment` | 12+ | Requisitions, candidates, applications, assessments, pipeline |
| `performance` | 10+ | OKR cycles, objectives, review cycles, reviews, competencies |
| `benefits` | 8+ | Plans, enrollments, claims, EWA transactions |
| `lms` | 10+ | Courses, modules, enrollments, certificates, SCORM |
| `dms` | 6+ | Documents, templates, signatures, permissions |
| `workforce` | 6+ | Headcount plans, compensation cycles, proposals, pay bands |
| `analytics` | 5+ | Dashboards, reports, metrics |
| `auth` | 8+ | Users, roles, permissions, sessions, MFA tokens |
| `notification` | 4+ | Templates, queues, preferences, analytics |
| `communication` | 4+ | Channels, messages, announcements, events |
| `tracking` | 3+ | GPS tracks, field workforce, routes |
| `integrations` | 4+ | Connectors, webhooks, sync jobs, API keys |
| `plugins` | 3+ | Plugin registry, installations, configurations |
| `workflow` | 3+ | Workflow definitions, instances, tasks |

### 2.2 Key Indexes

```sql
-- Employee lookups
CREATE INDEX idx_employees_tenant_status ON employee.employees(tenant_id, employment_status);
CREATE INDEX idx_employees_tenant_department ON employee.employees(tenant_id, department_id);
CREATE INDEX idx_employees_tenant_manager ON employee.employees(tenant_id, manager_id);
CREATE INDEX idx_employees_email ON employee.employees(tenant_id, email);

-- Payroll lookups
CREATE INDEX idx_payroll_runs_period ON payroll_runs(period_id, status);
CREATE INDEX idx_payroll_entries_run ON payroll_entries(run_id, employee_id);
CREATE INDEX idx_payroll_entries_employee_ytd ON payroll_entries(tenant_id, employee_id, created_at);

-- Attendance lookups
CREATE INDEX idx_clock_records_employee_date ON attendance.clock_records(employee_id, clock_date);
CREATE INDEX idx_clock_records_tenant_date ON attendance.clock_records(tenant_id, clock_date);

-- Leave lookups
CREATE INDEX idx_leave_requests_employee ON leave.leave_requests(employee_id, status);
CREATE INDEX idx_leave_balances_employee ON leave.leave_balances(employee_id, leave_type_id);
```

---

## 3. Event Schemas (CloudEvents)

### 3.1 Event Envelope

```json
{
  "specversion": "1.0",
  "id": "uuid-v4",
  "type": "erp.hcm.employee.created",
  "source": "/employee-service",
  "subject": "employee-uuid",
  "time": "2026-02-23T10:00:00Z",
  "datacontenttype": "application/json",
  "data": {
    "tenant_id": "uuid",
    "employee_id": "uuid",
    "employee_number": "EMP-2026-0042",
    "action": "created",
    "actor_id": "uuid",
    "timestamp": "2026-02-23T10:00:00Z"
  }
}
```

### 3.2 Event Type Catalog

| Event Type | Publisher | Consumers |
|------------|-----------|-----------|
| `erp.hcm.employee.created` | employee-service | payroll, benefits, learning, notification |
| `erp.hcm.employee.updated` | employee-service | payroll, compliance, notification |
| `erp.hcm.employee.terminated` | employee-service | payroll, benefits, assets, notification |
| `erp.hcm.payroll.processed` | payroll-service | notification, analytics |
| `erp.hcm.payroll.approved` | payroll-service | disbursement, notification |
| `erp.hcm.payroll.paid` | payroll-service | ERP-Finance, notification |
| `erp.hcm.leave.requested` | leave-service | notification |
| `erp.hcm.leave.approved` | leave-service | attendance, notification |
| `erp.hcm.leave.rejected` | leave-service | notification |
| `erp.hcm.recruitment.created` | recruitment-service | notification |
| `erp.hcm.recruitment.offer_accepted` | recruitment-service | employee, notification |
| `erp.hcm.performance.review_completed` | performance-service | compensation, notification |
| `erp.hcm.time-attendance.clock_in` | attendance-service | analytics |
| `erp.hcm.time-attendance.anomaly` | attendance-service | notification, compliance |
| `erp.hcm.benefits.enrolled` | benefits-service | payroll, notification |
| `erp.hcm.learning.completed` | learning-service | performance, notification |
| `erp.hcm.compensation.updated` | compensation-service | payroll, notification |
| `erp.hcm.compliance.violation` | compliance-service | notification, audit |
| `erp.hcm.document.signed` | document-service | compliance, notification |

---

## 4. Environment Variables

### 4.1 Server Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PF_SERVER_HOST` | `0.0.0.0` | HTTP server bind address |
| `PF_SERVER_PORT` | `8080` | HTTP server port |
| `PF_SERVER_READ_TIMEOUT` | `30s` | HTTP read timeout |
| `PF_SERVER_WRITE_TIMEOUT` | `30s` | HTTP write timeout |
| `PF_SERVER_IDLE_TIMEOUT` | `120s` | HTTP idle timeout |
| `PF_SERVER_SHUTDOWN_TIMEOUT` | `15s` | Graceful shutdown timeout |
| `PF_SERVER_TLS_ENABLED` | `false` | Enable TLS |
| `PF_ENVIRONMENT` | `development` | Environment (development/production) |

### 4.2 Database Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PF_DATABASE_HOST` | `localhost` | PostgreSQL host |
| `PF_DATABASE_PORT` | `5432` | PostgreSQL port |
| `PF_DATABASE_USER` | `peopleforce` | Database user |
| `PF_DATABASE_PASSWORD` | (required) | Database password |
| `PF_DATABASE_DATABASE` | `peopleforce` | Database name |
| `PF_DATABASE_SSL_MODE` | `disable` | SSL mode (disable/require/verify-full) |
| `PF_DATABASE_MAX_OPEN_CONNS` | `100` | Max open connections |
| `PF_DATABASE_MAX_IDLE_CONNS` | `25` | Max idle connections |
| `PF_DATABASE_CONN_LIFETIME` | `1h` | Connection max lifetime |

### 4.3 Redis Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PF_REDIS_HOST` | `localhost` | Redis host |
| `PF_REDIS_PORT` | `6379` | Redis port |
| `PF_REDIS_PASSWORD` | (empty) | Redis password |
| `PF_REDIS_DB` | `0` | Redis database number |
| `PF_REDIS_POOL_SIZE` | `100` | Connection pool size |

### 4.4 NATS Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PF_NATS_URL` | `nats://localhost:4222` | NATS server URL |
| `PF_NATS_TOKEN` | (empty) | NATS auth token |
| `PF_NATS_CLIENT_ID` | `hrms-gateway` | NATS client identifier |
| `PF_NATS_MAX_RECONNECTS` | `-1` | Max reconnection attempts (-1 = unlimited) |

### 4.5 Authentication Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PF_AUTH_RSA_PRIVATE_KEY` | (required for prod) | RSA private key PEM |
| `PF_AUTH_RSA_PUBLIC_KEY` | (required for prod) | RSA public key PEM |
| `PF_AUTH_JWT_ISSUER` | `peopleforce` | JWT issuer claim |
| `PF_AUTH_ACCESS_TOKEN_EXPIRY` | `15m` | Access token TTL |
| `PF_AUTH_REFRESH_TOKEN_EXPIRY` | `168h` | Refresh token TTL (7 days) |
| `PF_AUTH_MAX_LOGIN_ATTEMPTS` | `5` | Before account lockout |
| `PF_AUTH_LOCKOUT_DURATION` | `30m` | Lockout duration |
| `PF_AUTH_MAX_CONCURRENT_SESSIONS` | `5` | Per user |
| `PF_AUTH_PASSWORD_MIN_LENGTH` | `8` | Minimum password length |
| `PF_AUTH_PASSWORD_REQUIRE_SPECIAL` | `true` | Require special chars |
| `PF_AUTH_MFA_ISSUER` | `PeopleForce` | MFA TOTP issuer name |

### 4.6 CORS Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PF_CORS_ALLOWED_ORIGINS` | `http://localhost:3000` | Comma-separated origins |
| `PF_CORS_ALLOWED_METHODS` | `GET,POST,PUT,PATCH,DELETE,OPTIONS` | HTTP methods |
| `PF_CORS_ALLOWED_HEADERS` | `Authorization,Content-Type,X-CSRF-Token,X-Tenant-ID` | Headers |
| `PF_CORS_MAX_AGE` | `86400` | Preflight cache (seconds) |

### 4.7 Logging Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `PF_LOG_LEVEL` | `info` | Log level (debug/info/warn/error) |
| `PF_LOG_FORMAT` | `json` | Log format (json/console) |

---

## 5. Rate Limiting Specifications

| Endpoint Category | Rate Limit | Window | Scope |
|------------------|-----------|--------|-------|
| Authentication | 10 req | 1 min | Per IP |
| Read operations | 1000 req | 1 min | Per tenant |
| Write operations | 200 req | 1 min | Per tenant |
| Payroll processing | 5 req | 5 min | Per tenant |
| Bulk import | 2 req | 10 min | Per tenant |
| File upload | 50 req | 1 min | Per user |

---

## 6. Data Validation Rules

| Field | Validation | Example |
|-------|-----------|---------|
| `email` | RFC 5322, max 255 chars | `user@company.com` |
| `employee_number` | Required, max 50, unique per tenant | `EMP-2026-0042` |
| `first_name` | Required, max 100 chars | `Adebayo` |
| `phone` | E.164 format, max 50 | `+2348012345678` |
| `currency` | ISO 4217, 3 chars | `NGN` |
| `country_code` | ISO 3166-1 alpha-3 | `NGA` |
| `employment_type` | Enum | `permanent\|contract\|intern\|part_time` |
| `employment_status` | Enum | `active\|inactive\|on_leave\|suspended\|terminated` |
| `payroll_period_month` | 1-12 | `2` |
| `page_size` | 1-100, default 20 | `20` |
