# ERP-Projects -- API Reference

## Document Control

| Field         | Value                                          |
|---------------|------------------------------------------------|
| Module        | ERP-Projects                                   |
| Version       | 1.0                                            |
| Date          | 2026-02-23                                     |
| Base URL      | `https://api.erp.example.com/v1`               |

---

## 1. Authentication

All API requests require a valid JWT token from ERP-IAM and a tenant identifier.

### Required Headers

| Header          | Required | Description                           |
|-----------------|----------|---------------------------------------|
| Authorization   | Yes      | `Bearer <jwt-token>`                  |
| X-Tenant-ID     | Yes      | Tenant UUID for data isolation        |
| Content-Type    | Yes      | `application/json`                    |
| X-Request-ID    | No       | Correlation ID for distributed tracing|

### Error Response Format

```json
{
  "error": {
    "code": "ERR_NOT_FOUND",
    "message": "Project with id 'abc-123' not found",
    "details": {},
    "request_id": "req-uuid"
  }
}
```

### Standard Error Codes

| HTTP Code | Error Code              | Description                     |
|-----------|-------------------------|---------------------------------|
| 400       | ERR_VALIDATION          | Request validation failed       |
| 400       | ERR_MISSING_TENANT      | X-Tenant-ID header missing      |
| 401       | ERR_UNAUTHORIZED        | Invalid or expired JWT          |
| 403       | ERR_FORBIDDEN           | Insufficient permissions        |
| 404       | ERR_NOT_FOUND           | Resource not found              |
| 409       | ERR_CONFLICT            | Resource conflict (duplicate)   |
| 422       | ERR_BUSINESS_RULE       | Business rule violation         |
| 429       | ERR_RATE_LIMITED         | Rate limit exceeded             |
| 500       | ERR_INTERNAL            | Internal server error           |

---

## 2. Project Management API

### 2.1 List Projects

```
GET /v1/project
```

**Query Parameters:**

| Parameter   | Type    | Default | Description                          |
|-------------|---------|---------|--------------------------------------|
| page        | integer | 1       | Page number                          |
| per_page    | integer | 50      | Items per page (max 100)             |
| status      | string  |         | Filter by status                     |
| priority    | string  |         | Filter by priority                   |
| owner_id    | uuid    |         | Filter by owner                      |
| type        | string  |         | Filter by project type               |
| search      | string  |         | Full-text search on name/description |
| sort        | string  | created_at | Sort field                        |
| order       | string  | desc    | Sort order (asc/desc)                |

**Response:** `200 OK`

```json
{
  "items": [
    {
      "id": "proj-uuid",
      "name": "ERP Platform Migration",
      "status": "ACTIVE",
      "priority": "HIGH",
      "type": "DEVELOPMENT",
      "startDate": "2026-01-15",
      "endDate": "2026-06-30",
      "budget": 250000.00,
      "spentAmount": 85000.00,
      "currency": "USD",
      "healthScore": 78,
      "healthStatus": "GOOD",
      "completionPct": 34.5,
      "owner": {
        "id": "user-uuid",
        "name": "Jane Smith",
        "email": "jane@example.com"
      },
      "createdAt": "2026-01-10T08:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "perPage": 50,
    "total": 127,
    "totalPages": 3
  }
}
```

### 2.2 Create Project

```
POST /v1/project
```

**Request Body:**

```json
{
  "name": "Website Redesign",
  "description": "Complete redesign of corporate website",
  "clientName": "Acme Corp",
  "clientEmail": "contact@acme.com",
  "type": "DESIGN",
  "priority": "HIGH",
  "startDate": "2026-03-01",
  "endDate": "2026-07-31",
  "budget": 75000.00,
  "currency": "USD",
  "ownerId": "user-uuid",
  "tags": "website,redesign,q2"
}
```

**Response:** `201 Created`

### 2.3 Get Project

```
GET /v1/project/:id
```

**Response:** `200 OK` -- Full project object with nested counts for tasks, milestones, resources.

### 2.4 Update Project

```
PUT /v1/project/:id
```

### 2.5 Delete Project

```
DELETE /v1/project/:id
```

**Response:** `200 OK`

---

## 3. Task Management API

### 3.1 List Tasks

```
GET /v1/task
```

**Query Parameters:**

| Parameter     | Type    | Default | Description                        |
|---------------|---------|---------|-------------------------------------|
| project_id    | uuid    |         | Filter by project (required)        |
| status        | string  |         | Filter by status                    |
| priority      | string  |         | Filter by priority                  |
| assignee_id   | uuid    |         | Filter by assigned user             |
| parent_task_id| uuid    |         | Get subtasks of a parent            |
| sprint_id     | uuid    |         | Filter by sprint                    |
| epic_id       | uuid    |         | Filter by epic                      |
| is_overdue    | boolean |         | Filter overdue tasks                |
| search        | string  |         | Full-text search                    |
| page          | integer | 1       | Page number                         |
| per_page      | integer | 50      | Items per page                      |

### 3.2 Create Task

```
POST /v1/task
```

**Request Body:**

```json
{
  "projectId": "proj-uuid",
  "parentTaskId": null,
  "title": "Design wireframes for homepage",
  "description": "Create low-fidelity wireframes for the homepage redesign",
  "priority": "HIGH",
  "estimatedHours": 16,
  "startDate": "2026-03-05",
  "dueDate": "2026-03-12",
  "tags": "design,wireframe",
  "assignees": [
    { "userId": "user-uuid", "role": "ASSIGNEE" }
  ]
}
```

### 3.3 Add Task Dependency

```
POST /v1/task/:id/dependencies
```

**Request Body:**

```json
{
  "dependencyTaskId": "predecessor-task-uuid",
  "type": "FINISH_TO_START",
  "lagDays": 0
}
```

### 3.4 Bulk Operations

```
POST /v1/task/bulk
```

**Request Body:**

```json
{
  "operation": "UPDATE_STATUS",
  "taskIds": ["task-1", "task-2", "task-3"],
  "payload": {
    "status": "IN_PROGRESS"
  }
}
```

**Supported Operations:** `UPDATE_STATUS`, `ASSIGN`, `MOVE_PROJECT`, `SET_PRIORITY`, `DELETE`

---

## 4. Board API

### 4.1 Get Board

```
GET /v1/board/:projectId
```

**Response:**

```json
{
  "projectId": "proj-uuid",
  "type": "KANBAN",
  "columns": [
    {
      "id": "col-1",
      "name": "To Do",
      "statusMapping": "TODO",
      "wipLimit": 10,
      "cards": [
        {
          "taskId": "task-uuid",
          "title": "Design wireframes",
          "priority": "HIGH",
          "assignees": [{ "id": "user-uuid", "name": "Jane", "avatarUrl": "..." }],
          "dueDate": "2026-03-12",
          "storyPoints": 5,
          "tags": ["design"]
        }
      ]
    }
  ],
  "swimlanes": {
    "enabled": true,
    "groupBy": "PRIORITY"
  }
}
```

### 4.2 Move Card

```
PUT /v1/board/:projectId/card/:taskId
```

**Request Body:**

```json
{
  "targetColumnId": "col-2",
  "position": 3,
  "swimlaneId": "high-priority"
}
```

---

## 5. Timeline / Gantt API

### 5.1 Get Timeline Data

```
GET /v1/timeline/:projectId
```

**Query Parameters:**

| Parameter   | Type   | Description                         |
|-------------|--------|--------------------------------------|
| start_date  | date   | Timeline viewport start             |
| end_date    | date   | Timeline viewport end               |
| zoom        | string | day/week/month/quarter/year          |

**Response:**

```json
{
  "projectId": "proj-uuid",
  "tasks": [
    {
      "id": "task-uuid",
      "title": "Design wireframes",
      "startDate": "2026-03-05",
      "endDate": "2026-03-12",
      "progress": 45,
      "isCriticalPath": true,
      "isMilestone": false,
      "dependencies": [
        { "taskId": "dep-uuid", "type": "FINISH_TO_START", "lag": 0 }
      ],
      "resourceIds": ["user-uuid"]
    }
  ],
  "milestones": [
    {
      "id": "ms-uuid",
      "title": "Design Complete",
      "date": "2026-03-15",
      "status": "PENDING"
    }
  ],
  "criticalPath": ["task-1", "task-3", "task-7", "task-12"],
  "baselines": [
    { "id": "bl-uuid", "name": "Original Plan", "createdAt": "2026-03-01T00:00:00Z" }
  ]
}
```

### 5.2 Save Baseline

```
POST /v1/timeline/:projectId/baseline
```

### 5.3 Auto-Schedule

```
POST /v1/timeline/:projectId/auto-schedule
```

**Request Body:**

```json
{
  "startDate": "2026-03-01",
  "respectDependencies": true,
  "levelResources": true,
  "workingDays": [1, 2, 3, 4, 5],
  "hoursPerDay": 8
}
```

---

## 6. Resource Management API

### 6.1 List Allocations

```
GET /v1/resource/allocations
```

### 6.2 Create Allocation

```
POST /v1/resource/allocations
```

### 6.3 Get User Availability

```
GET /v1/resource/availability/:userId
```

**Response:**

```json
{
  "userId": "user-uuid",
  "totalCapacity": 100,
  "allocations": [
    { "projectId": "proj-1", "projectName": "ERP Migration", "percentage": 60, "startDate": "2026-01-01", "endDate": "2026-06-30" },
    { "projectId": "proj-2", "projectName": "Website Redesign", "percentage": 30, "startDate": "2026-03-01", "endDate": "2026-07-31" }
  ],
  "availableCapacity": 10,
  "overAllocated": false
}
```

### 6.4 Get Workload

```
GET /v1/resource/workload
```

### 6.5 Capacity Planning

```
GET /v1/resource/capacity
```

---

## 7. Time Tracking API

### 7.1 Timer Operations

```
POST /v1/time-tracking/timer/start
POST /v1/time-tracking/timer/stop
```

### 7.2 Create Time Entry

```
POST /v1/time-tracking/entries
```

**Request Body:**

```json
{
  "projectId": "proj-uuid",
  "taskId": "task-uuid",
  "description": "Wireframe iteration based on feedback",
  "hours": 3.5,
  "date": "2026-03-10",
  "billable": true
}
```

### 7.3 Timesheet Operations

```
GET  /v1/time-tracking/timesheet/:userId?week=2026-W10
POST /v1/time-tracking/timesheet/submit
POST /v1/time-tracking/timesheet/approve
POST /v1/time-tracking/timesheet/reject
```

---

## 8. Budget API

### 8.1 Get EVM Metrics

```
GET /v1/budget/:projectId/evm
```

**Response:**

```json
{
  "projectId": "proj-uuid",
  "asOfDate": "2026-03-15",
  "plannedValue": 100000.00,
  "earnedValue": 85000.00,
  "actualCost": 92000.00,
  "cpi": 0.924,
  "spi": 0.85,
  "costVariance": -7000.00,
  "scheduleVariance": -15000.00,
  "eac": 270834.00,
  "etc": 178834.00,
  "vac": -20834.00,
  "tcpi": 1.09,
  "healthIndicator": "WARNING"
}
```

---

## 9. Portfolio API

### 9.1 Portfolio Dashboard

```
GET /v1/portfolio/:id/dashboard
```

### 9.2 What-If Scenario

```
POST /v1/portfolio/:id/what-if
```

**Request Body:**

```json
{
  "scenario": "DELAY_PROJECT",
  "parameters": {
    "projectId": "proj-uuid",
    "delayWeeks": 4
  }
}
```

---

## 10. Agile API

### 10.1 Sprint Management

```
POST /v1/agile/sprint
POST /v1/agile/sprint/:id/start
POST /v1/agile/sprint/:id/complete
GET  /v1/agile/velocity/:projectId
GET  /v1/agile/burndown/:sprintId
GET  /v1/agile/backlog/:projectId
```

### 10.2 Retrospective

```
POST /v1/agile/retrospective
```

**Request Body:**

```json
{
  "sprintId": "sprint-uuid",
  "wentWell": ["Good team collaboration", "Automated tests caught bugs early"],
  "toImprove": ["Sprint planning accuracy", "Code review turnaround time"],
  "actionItems": [
    { "description": "Add estimation calibration session", "assigneeId": "user-uuid" }
  ]
}
```

---

## 11. Pagination

All list endpoints support cursor-based or offset pagination:

```
GET /v1/project?page=2&per_page=25
```

**Response envelope:**

```json
{
  "items": [...],
  "pagination": {
    "page": 2,
    "perPage": 25,
    "total": 127,
    "totalPages": 6,
    "hasNext": true,
    "hasPrev": true
  }
}
```

---

## 12. Rate Limiting

| Tier        | Requests/min | Burst |
|-------------|-------------|-------|
| Starter     | 100         | 20    |
| Professional| 500         | 50    |
| Business    | 2000        | 200   |
| Enterprise  | 10000       | 1000  |

Rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

---

## 13. Webhooks

```
POST /v1/webhooks
```

```json
{
  "url": "https://your-app.com/webhook",
  "events": ["erp.projects.task.created", "erp.projects.task.updated"],
  "secret": "webhook-signing-secret"
}
```

Webhook payloads follow the CloudEvents 1.0 specification with HMAC-SHA256 signature validation.
