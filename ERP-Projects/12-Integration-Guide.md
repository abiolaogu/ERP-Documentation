# ERP-Projects -- Integration Guide

## Document Control

| Field         | Value                                          |
|---------------|------------------------------------------------|
| Module        | ERP-Projects                                   |
| Version       | 1.0                                            |
| Date          | 2026-02-23                                     |

---

## 1. Integration Overview

```mermaid
flowchart TB
    subgraph "ERP-Projects"
        PS["project-service"]
        TTS["time-tracking-service"]
        BDS["budget-service"]
        RS["resource-service"]
    end

    subgraph "ERP Suite Modules"
        IAM["ERP-IAM<br/>Authentication"]
        PLT["ERP-Platform<br/>Entitlements"]
        HCM["ERP-HCM<br/>HR & Payroll"]
        FIN["ERP-Finance<br/>General Ledger"]
        CRM["ERP-CRM<br/>Client Management"]
        WS["ERP-Workspace<br/>Collaboration"]
        BI["ERP-BI<br/>Business Intelligence"]
    end

    PS -->|"OIDC/JWT auth"| IAM
    PS -->|"Feature gating"| PLT
    TTS -->|"Timesheet export"| HCM
    BDS -->|"Cost posting"| FIN
    PS -->|"Client link"| CRM
    RS -->|"Employee data"| HCM
    PS -->|"Notifications"| WS
    PS -->|"Analytics data"| BI
```

---

## 2. ERP-IAM Integration

### 2.1 Authentication Flow

```mermaid
sequenceDiagram
    participant U as User Browser
    participant FE as Next.js Frontend
    participant GW as API Gateway
    participant IAM as ERP-IAM
    participant SVC as ERP-Projects Service

    U->>FE: Access ERP-Projects
    FE->>IAM: Redirect to /authorize
    IAM->>U: Login form
    U->>IAM: Credentials
    IAM->>FE: Authorization code
    FE->>IAM: POST /token (code exchange)
    IAM-->>FE: Access token + Refresh token
    FE->>GW: API request + Bearer token
    GW->>IAM: GET /userinfo (validate)
    IAM-->>GW: User claims
    GW->>SVC: Forward request + X-Tenant-ID + X-User-ID
    SVC-->>GW: Response
    GW-->>FE: Response
```

### 2.2 JWT Claims Structure

```json
{
  "sub": "user-uuid",
  "email": "jane@example.com",
  "name": "Jane Smith",
  "tenant_id": "tenant-uuid",
  "roles": ["MANAGER"],
  "permissions": [
    "projects:read",
    "projects:write",
    "tasks:read",
    "tasks:write",
    "budget:read",
    "resources:read",
    "resources:write"
  ],
  "iss": "https://iam.erp.example.com",
  "aud": "erp-projects",
  "exp": 1740326400,
  "iat": 1740322800
}
```

### 2.3 Permission Mapping

| Permission            | Description                        | Default Roles        |
|----------------------|-------------------------------------|----------------------|
| `projects:read`     | View projects                       | All                  |
| `projects:write`    | Create/update/delete projects       | ADMIN, MANAGER       |
| `tasks:read`        | View tasks                          | All                  |
| `tasks:write`       | Create/update tasks                 | ADMIN, MANAGER, MEMBER|
| `budget:read`       | View budget data                    | ADMIN, MANAGER       |
| `budget:write`      | Modify budgets                      | ADMIN                |
| `resources:read`    | View resource allocations           | ADMIN, MANAGER       |
| `resources:write`   | Manage resource allocations         | ADMIN, MANAGER       |
| `timesheets:approve`| Approve team timesheets             | ADMIN, MANAGER       |
| `portfolio:read`    | View portfolio data                 | ADMIN                |
| `portfolio:write`   | Manage portfolios                   | ADMIN                |

---

## 3. ERP-Platform Integration

### 3.1 Entitlement Check

```mermaid
sequenceDiagram
    participant GW as API Gateway
    participant PLT as ERP-Platform
    participant SVC as Service

    GW->>PLT: GET /v1/entitlements?tenant=uuid&sku=erp.projects
    PLT-->>GW: Entitled features + plan tier
    GW->>GW: Gate features based on entitlements
    GW->>SVC: Forward (if entitled)
```

### 3.2 Feature Gating by Plan

| Feature                 | Starter | Professional | Business | Enterprise |
|------------------------|---------|-------------|----------|------------|
| Projects (max)         | 10      | 50          | Unlimited| Unlimited  |
| Tasks per project      | 500     | 5,000       | Unlimited| Unlimited  |
| Gantt chart            | No      | Yes         | Yes      | Yes        |
| Critical path          | No      | Yes         | Yes      | Yes        |
| EVM metrics            | No      | No          | Yes      | Yes        |
| Portfolio management   | No      | No          | Yes      | Yes        |
| What-if scenarios      | No      | No          | No       | Yes        |
| AI insights            | No      | No          | Basic    | Full       |
| Custom fields          | 5       | 20          | 50       | Unlimited  |
| API rate limit (rpm)   | 100     | 500         | 2,000    | 10,000     |

---

## 4. ERP-HCM Integration

### 4.1 Timesheet to Payroll Sync

```mermaid
sequenceDiagram
    participant TTS as time-tracking-service
    participant NATS as NATS JetStream
    participant HCM as ERP-HCM Payroll

    TTS->>NATS: Publish erp.projects.time-tracking.timesheet_approved
    Note over NATS: Event payload includes user_id, hours, date range
    NATS->>HCM: Deliver event
    HCM->>HCM: Map project hours to payroll entries
    HCM->>HCM: Calculate overtime, billable premiums
    HCM-->>NATS: Publish erp.hcm.payroll.entry_created
```

### 4.2 Employee Data Sync

| Direction    | Data                              | Frequency  | Trigger              |
|-------------|-----------------------------------|------------|----------------------|
| HCM -> Projects | Employee profiles, departments | Real-time  | Employee created/updated |
| HCM -> Projects | Working calendar, holidays     | Daily      | Calendar update      |
| HCM -> Projects | Skill profiles                 | On change  | Skill assessment     |
| Projects -> HCM | Approved timesheets            | On approval| Timesheet approval   |
| Projects -> HCM | Project allocation hours       | Weekly     | Scheduled sync       |

### 4.3 Payroll Data Format

```json
{
  "type": "erp.projects.time-tracking.timesheet_approved",
  "data": {
    "userId": "user-uuid",
    "employeeId": "emp-12345",
    "weekStartDate": "2026-03-09",
    "weekEndDate": "2026-03-13",
    "entries": [
      {
        "date": "2026-03-09",
        "projectId": "proj-uuid",
        "projectName": "Website Redesign",
        "hours": 8.0,
        "billable": true,
        "hourlyRate": 150.00,
        "costCenter": "CC-1001"
      }
    ],
    "totalHours": 40.0,
    "totalBillableHours": 32.0,
    "totalNonBillableHours": 8.0,
    "approvedBy": "manager-uuid",
    "approvedAt": "2026-03-14T09:00:00Z"
  }
}
```

---

## 5. ERP-Finance Integration

### 5.1 Project Cost Posting

```mermaid
flowchart LR
    BDS["budget-service"] -->|"Project costs"| NATS["NATS"]
    NATS -->|"erp.projects.budget.cost_accrued"| FIN["ERP-Finance"]
    FIN -->|"Journal entry"| GL["General Ledger"]
```

### 5.2 GL Mapping

| Project Cost Type  | GL Account       | Debit/Credit |
|-------------------|------------------|--------------|
| Labor cost        | 5100-Labor       | Debit        |
| Software licenses | 5200-Software    | Debit        |
| Travel expenses   | 5300-Travel      | Debit        |
| Contractor fees   | 5400-Contractors | Debit        |
| Project revenue   | 4100-Revenue     | Credit       |

### 5.3 Invoice Sync

When project invoices are created in ERP-Projects, they sync to ERP-Finance for accounts receivable tracking:

```json
{
  "type": "erp.projects.invoice.created",
  "data": {
    "invoiceId": "inv-uuid",
    "invoiceNumber": "INV-2026-0042",
    "projectId": "proj-uuid",
    "clientName": "Acme Corp",
    "totalAmount": 12500.00,
    "currency": "USD",
    "issueDate": "2026-03-15",
    "dueDate": "2026-04-14",
    "lineItems": [
      {
        "description": "Website Design - March 2026",
        "quantity": 80,
        "unitPrice": 150.00,
        "amount": 12000.00
      },
      {
        "description": "Design tool licenses",
        "quantity": 1,
        "unitPrice": 500.00,
        "amount": 500.00
      }
    ]
  }
}
```

---

## 6. ERP-CRM Integration

### 6.1 Client-Project Linkage

| CRM Entity    | Projects Entity | Relationship                      |
|--------------|-----------------|-----------------------------------|
| Account      | Project.clientName | Client associated with project  |
| Contact      | Project.clientEmail| Primary contact for project     |
| Opportunity  | Project          | Won opportunity creates project  |
| Deal         | Project.budget   | Deal value maps to project budget|

### 6.2 Opportunity-to-Project Flow

```mermaid
sequenceDiagram
    participant CRM as ERP-CRM
    participant NATS as NATS
    participant PS as project-service

    CRM->>NATS: erp.crm.opportunity.won
    NATS->>PS: Deliver event
    PS->>PS: Auto-create project from opportunity data
    PS->>NATS: erp.projects.project.created
    Note over PS: Project linked to CRM account
```

---

## 7. Third-Party Integrations

### 7.1 Webhook Configuration

```json
{
  "webhooks": [
    {
      "id": "wh-uuid",
      "url": "https://external-system.com/webhook",
      "events": [
        "erp.projects.task.created",
        "erp.projects.task.completed"
      ],
      "secret": "hmac-sha256-secret",
      "active": true,
      "retryPolicy": {
        "maxRetries": 3,
        "backoff": "exponential"
      }
    }
  ]
}
```

### 7.2 Webhook Delivery

```mermaid
flowchart LR
    SVC["Service"] -->|event| NATS["NATS"]
    NATS --> WD["Webhook Dispatcher"]
    WD -->|POST + HMAC-SHA256| EXT["External System"]
    EXT -->|200 OK| WD
    WD -->|Retry on failure| DLQ["Dead Letter Queue"]
```

### 7.3 Supported Import/Export Formats

| Format              | Import | Export | Use Case                          |
|---------------------|--------|--------|-----------------------------------|
| Microsoft Project XML | Yes  | Yes    | MS Project interoperability       |
| CSV                 | Yes    | Yes    | Spreadsheet import/export         |
| JSON                | Yes    | Yes    | API data exchange                 |
| iCalendar (.ics)    | No     | Yes    | Calendar export                   |
| PDF                 | No     | Yes    | Report export                     |
| JIRA JSON           | Yes    | No     | JIRA migration                    |

---

## 8. Integration Testing

### 8.1 Contract Testing

```mermaid
flowchart TB
    PS["project-service<br/>(Provider)"] -->|"Publishes contracts"| PACT["Pact Broker"]
    PACT -->|"Validates"| TTS["time-tracking-service<br/>(Consumer)"]
    PACT -->|"Validates"| HCM["ERP-HCM<br/>(Consumer)"]
```

### 8.2 Integration Test Matrix

| Integration           | Test Type         | Frequency  | Environment |
|----------------------|-------------------|------------|-------------|
| ERP-IAM Auth         | Contract + E2E    | Per commit | Staging     |
| ERP-Platform Entitle | Contract          | Per commit | Staging     |
| ERP-HCM Timesheet    | Contract + E2E    | Daily      | Integration |
| ERP-Finance GL       | Contract + E2E    | Daily      | Integration |
| ERP-CRM Client       | Contract          | Weekly     | Integration |
| Webhooks             | Integration       | Per commit | Staging     |
