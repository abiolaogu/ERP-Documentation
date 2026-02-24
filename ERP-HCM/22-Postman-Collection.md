# ERP-HCM Postman Collection

## Collection Overview

This document describes the Postman collection for testing and exploring the ERP-HCM API. The collection covers all 14 services with pre-configured authentication, environment variables, and test scripts.

---

## 1. Environment Setup

### 1.1 Environment Variables

Create a Postman environment named `ERP-HCM Local` with the following variables:

| Variable | Initial Value | Description |
|----------|--------------|-------------|
| `base_url` | `http://localhost:8090` | Gateway base URL |
| `employee_service_url` | `http://localhost:8081` | Employee service direct URL |
| `payroll_service_url` | `http://localhost:8082` | Payroll service direct URL |
| `leave_service_url` | `http://localhost:8083` | Leave service direct URL |
| `tenant_id` | `00000000-0000-0000-0000-000000000001` | Test tenant UUID |
| `access_token` | (empty, auto-populated) | JWT access token |
| `refresh_token` | (empty, auto-populated) | JWT refresh token |
| `test_employee_id` | (empty, auto-populated) | Created employee UUID |
| `test_payroll_period_id` | (empty, auto-populated) | Created period UUID |
| `test_payroll_run_id` | (empty, auto-populated) | Created run UUID |
| `test_leave_request_id` | (empty, auto-populated) | Created leave request UUID |

### 1.2 Production Environment

| Variable | Initial Value |
|----------|--------------|
| `base_url` | `https://api.peopleforce.io` |
| `tenant_id` | (your tenant UUID) |

---

## 2. Collection Structure

```
ERP-HCM API/
  00-Health & Capabilities/
    GET Health Check
    GET Capabilities
  01-Authentication/
    POST Login
    POST MFA Verify
    POST Refresh Token
    POST Logout
  02-Employee Service/
    GET List Employees
    POST Create Employee
    GET Get Employee by ID
    PUT Update Employee
    DELETE Delete Employee
    POST Bulk Import
    GET Org Chart
    GET Employee Statistics
  03-Payroll Service/
    POST Create Period
    GET List Periods
    POST Create Payroll Run
    POST Process Payroll Run
    POST Approve Payroll Run
    GET Get Payslip
    POST Generate Bank File
    GET Payroll Summary
    GET YTD Report
  04-Leave Service/
    POST Submit Leave Request
    PUT Approve Leave Request
    PUT Reject Leave Request
    GET Get Leave Balances
    GET Leave Calendar
    GET Leave Types
  05-Time & Attendance/
    POST Clock In
    POST Clock Out
    GET Attendance Report
    GET Shift Schedule
  06-Recruitment/
    POST Create Job Requisition
    GET List Requisitions
    POST Create Candidate
    POST Submit Application
    PUT Move Pipeline Stage
    POST Schedule Interview
  07-Performance/
    POST Create OKR Cycle
    POST Create Objective
    PUT Update Key Result Progress
    POST Create Review Cycle
    POST Submit Review
    GET 9-Box Grid
  08-Benefits/
    POST Create Plan
    POST Enroll Employee
    POST Submit Claim
    POST Request EWA
  09-Learning/
    POST Create Course
    POST Enroll Learner
    POST Record Completion
    GET Get Certificate
  10-Compensation/
    POST Create Salary Band
    POST Create Comp Cycle
  11-Workforce Planning/
    POST Create Headcount Plan
    GET Workforce Analytics
  12-Compliance/
    GET Compliance Status
    POST Data Subject Access Request
    GET Audit Log
  13-Document Service/
    POST Upload Document
    GET Get Document
  14-Facilities/
    POST Book Room
    GET Check Availability
```

---

## 3. Request Definitions

### 3.1 Health Check

```json
{
  "name": "GET Health Check",
  "request": {
    "method": "GET",
    "header": [],
    "url": {
      "raw": "{{base_url}}/healthz",
      "host": ["{{base_url}}"],
      "path": ["healthz"]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test('Status code is 200', function () {",
          "    pm.response.to.have.status(200);",
          "});",
          "pm.test('Status is ok', function () {",
          "    var json = pm.response.json();",
          "    pm.expect(json.status).to.eql('ok');",
          "    pm.expect(json.module).to.eql('ERP-HCM');",
          "});"
        ]
      }
    }
  ]
}
```

### 3.2 Login

```json
{
  "name": "POST Login",
  "request": {
    "method": "POST",
    "header": [
      { "key": "Content-Type", "value": "application/json" }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"email\": \"admin@company.com\",\n  \"password\": \"SecureP@ss123\"\n}"
    },
    "url": {
      "raw": "{{base_url}}/v1/auth/login",
      "host": ["{{base_url}}"],
      "path": ["v1", "auth", "login"]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test('Status code is 200', function () {",
          "    pm.response.to.have.status(200);",
          "});",
          "pm.test('Token received', function () {",
          "    var json = pm.response.json();",
          "    pm.expect(json.access_token).to.be.a('string');",
          "    pm.environment.set('access_token', json.access_token);",
          "    pm.environment.set('refresh_token', json.refresh_token);",
          "});"
        ]
      }
    }
  ]
}
```

### 3.3 List Employees

```json
{
  "name": "GET List Employees",
  "request": {
    "method": "GET",
    "header": [
      { "key": "Authorization", "value": "Bearer {{access_token}}" },
      { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
    ],
    "url": {
      "raw": "{{base_url}}/v1/employee?page=1&page_size=25&status=active",
      "host": ["{{base_url}}"],
      "path": ["v1", "employee"],
      "query": [
        { "key": "page", "value": "1" },
        { "key": "page_size", "value": "25" },
        { "key": "status", "value": "active" }
      ]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test('Status code is 200', function () {",
          "    pm.response.to.have.status(200);",
          "});",
          "pm.test('Returns array of employees', function () {",
          "    var json = pm.response.json();",
          "    pm.expect(json.data).to.be.an('array');",
          "    pm.expect(json.pagination).to.have.property('total');",
          "});"
        ]
      }
    }
  ]
}
```

### 3.4 Create Employee

```json
{
  "name": "POST Create Employee",
  "request": {
    "method": "POST",
    "header": [
      { "key": "Authorization", "value": "Bearer {{access_token}}" },
      { "key": "X-Tenant-ID", "value": "{{tenant_id}}" },
      { "key": "Content-Type", "value": "application/json" }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"first_name\": \"Adebayo\",\n  \"last_name\": \"Okonkwo\",\n  \"work_email\": \"adebayo.okonkwo@company.com\",\n  \"personal_email\": \"adebayo@gmail.com\",\n  \"personal_phone\": \"+2348012345678\",\n  \"date_of_birth\": \"1990-05-15\",\n  \"gender\": \"male\",\n  \"nationality\": \"Nigerian\",\n  \"state_of_origin\": \"Lagos\",\n  \"employment_type\": \"full_time\",\n  \"department_id\": \"{{test_department_id}}\",\n  \"job_title_id\": \"{{test_job_title_id}}\",\n  \"location_id\": \"{{test_location_id}}\",\n  \"hire_date\": \"2026-03-01\",\n  \"current_salary\": 500000000,\n  \"salary_currency\": \"NGN\"\n}"
    },
    "url": {
      "raw": "{{base_url}}/v1/employee",
      "host": ["{{base_url}}"],
      "path": ["v1", "employee"]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test('Status code is 201', function () {",
          "    pm.response.to.have.status(201);",
          "});",
          "pm.test('Employee created', function () {",
          "    var json = pm.response.json();",
          "    pm.expect(json.id).to.be.a('string');",
          "    pm.expect(json.employee_number).to.be.a('string');",
          "    pm.environment.set('test_employee_id', json.id);",
          "});"
        ]
      }
    }
  ]
}
```

### 3.5 Create Payroll Run

```json
{
  "name": "POST Create Payroll Run",
  "request": {
    "method": "POST",
    "header": [
      { "key": "Authorization", "value": "Bearer {{access_token}}" },
      { "key": "X-Tenant-ID", "value": "{{tenant_id}}" },
      { "key": "Content-Type", "value": "application/json" }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"period_id\": \"{{test_payroll_period_id}}\",\n  \"run_type\": \"regular\",\n  \"description\": \"February 2026 Regular Payroll\"\n}"
    },
    "url": {
      "raw": "{{base_url}}/v1/payroll/runs",
      "host": ["{{base_url}}"],
      "path": ["v1", "payroll", "runs"]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test('Status code is 201', function () {",
          "    pm.response.to.have.status(201);",
          "});",
          "pm.test('Payroll run created', function () {",
          "    var json = pm.response.json();",
          "    pm.expect(json.id).to.be.a('string');",
          "    pm.expect(json.status).to.eql('draft');",
          "    pm.environment.set('test_payroll_run_id', json.id);",
          "});"
        ]
      }
    }
  ]
}
```

### 3.6 Process Payroll Run

```json
{
  "name": "POST Process Payroll Run",
  "request": {
    "method": "POST",
    "header": [
      { "key": "Authorization", "value": "Bearer {{access_token}}" },
      { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
    ],
    "url": {
      "raw": "{{base_url}}/v1/payroll/runs/{{test_payroll_run_id}}/process",
      "host": ["{{base_url}}"],
      "path": ["v1", "payroll", "runs", "{{test_payroll_run_id}}", "process"]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test('Status code is 200', function () {",
          "    pm.response.to.have.status(200);",
          "});",
          "pm.test('Payroll processed', function () {",
          "    var json = pm.response.json();",
          "    pm.expect(json.status).to.be.oneOf(['pending_approval', 'processed']);",
          "    pm.expect(json.summary.total_gross).to.be.above(0);",
          "    pm.expect(json.summary.total_paye).to.be.at.least(0);",
          "});"
        ]
      }
    }
  ]
}
```

### 3.7 Clock In

```json
{
  "name": "POST Clock In",
  "request": {
    "method": "POST",
    "header": [
      { "key": "Authorization", "value": "Bearer {{access_token}}" },
      { "key": "X-Tenant-ID", "value": "{{tenant_id}}" },
      { "key": "Content-Type", "value": "application/json" }
    ],
    "body": {
      "mode": "raw",
      "raw": "{\n  \"latitude\": 6.5244,\n  \"longitude\": 3.3792,\n  \"accuracy_meters\": 15.0,\n  \"device_id\": \"postman-test-device\"\n}"
    },
    "url": {
      "raw": "{{base_url}}/v1/time-attendance/clock-in",
      "host": ["{{base_url}}"],
      "path": ["v1", "time-attendance", "clock-in"]
    }
  },
  "event": [
    {
      "listen": "test",
      "script": {
        "exec": [
          "pm.test('Status code is 200', function () {",
          "    pm.response.to.have.status(200);",
          "});",
          "pm.test('Clock in recorded', function () {",
          "    var json = pm.response.json();",
          "    pm.expect(json.geofence_status).to.eql('inside');",
          "});"
        ]
      }
    }
  ]
}
```

---

## 4. Pre-request Scripts

### 4.1 Collection-Level Pre-request Script

Applied to every request in the collection:

```javascript
// Auto-refresh token if expired
if (pm.environment.get('access_token')) {
    try {
        var token = pm.environment.get('access_token');
        var payload = JSON.parse(atob(token.split('.')[1]));
        var expiry = payload.exp * 1000;
        var now = Date.now();

        if (now > expiry - 60000) { // Refresh 60s before expiry
            console.log('Token expiring soon, refreshing...');
            pm.sendRequest({
                url: pm.environment.get('base_url') + '/v1/auth/refresh',
                method: 'POST',
                header: {
                    'Content-Type': 'application/json'
                },
                body: {
                    mode: 'raw',
                    raw: JSON.stringify({
                        refresh_token: pm.environment.get('refresh_token')
                    })
                }
            }, function (err, res) {
                if (!err && res.code === 200) {
                    var json = res.json();
                    pm.environment.set('access_token', json.access_token);
                    pm.environment.set('refresh_token', json.refresh_token);
                }
            });
        }
    } catch (e) {
        console.log('Token parse error:', e.message);
    }
}
```

---

## 5. Collection Runner Workflow

### 5.1 End-to-End Payroll Flow

Run these requests in order to test the complete payroll workflow:

```
1. POST Login                        -> Sets access_token
2. POST Create Employee              -> Sets test_employee_id
3. POST Create Payroll Period         -> Sets test_payroll_period_id
4. POST Create Payroll Run            -> Sets test_payroll_run_id
5. POST Process Payroll Run           -> Calculates PAYE, pension, NHF
6. POST Approve Payroll Run           -> Advances to approved status
7. GET Get Payslip                    -> Retrieves calculated payslip
8. POST Generate Bank File            -> Generates NIBSS payment file
```

### 5.2 Employee Lifecycle Flow

```
1. POST Login
2. POST Create Employee               -> pre_boarding status
3. PUT Update Employee (onboarding)    -> onboarding status
4. PUT Update Employee (activate)      -> active status
5. POST Submit Leave Request
6. PUT Approve Leave Request
7. POST Clock In
8. POST Clock Out
9. DELETE Delete Employee              -> soft delete
```

---

## 6. Importing the Collection

### 6.1 Import via Postman

1. Open Postman
2. Click **Import** in the top-left
3. Select **Raw text** and paste the collection JSON
4. Import the environment variables separately
5. Select the `ERP-HCM Local` environment

### 6.2 Import via Newman (CLI)

```bash
# Install Newman
npm install -g newman

# Run the collection
newman run ERP-HCM.postman_collection.json \
  --environment ERP-HCM-Local.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export report.html

# Run specific folder
newman run ERP-HCM.postman_collection.json \
  --folder "02-Employee Service" \
  --environment ERP-HCM-Local.postman_environment.json
```

---

## 7. Common Headers Template

Every business request requires these headers:

```
Authorization: Bearer {{access_token}}
X-Tenant-ID: {{tenant_id}}
Content-Type: application/json
```

Optional headers:

| Header | Purpose |
|--------|---------|
| `X-Request-ID` | Distributed tracing correlation ID |
| `X-Idempotency-Key` | Prevent duplicate mutations |
| `Accept-Language` | Localization (`en`, `fr`, `es`, `ar`) |
