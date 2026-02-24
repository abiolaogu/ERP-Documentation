# ERP-School-Management -- Software Architecture

**Product:** EduCore Pro
**Version:** 1.0.0
**Date:** 2026-02-23

---

## 1. Architecture Style

EduCore Pro follows a **domain-driven microservices architecture** with event sourcing for cross-cutting concerns. The system is organized into bounded contexts aligned with educational domain capabilities.

```mermaid
graph TB
    subgraph "Bounded Contexts"
        BC1["Identity Context<br/>auth-service"]
        BC2["Student Context<br/>student-service"]
        BC3["Academic Context<br/>academic-service"]
        BC4["Finance Context<br/>finance-service"]
        BC5["Learning Context<br/>lms-service"]
        BC6["Admin Context<br/>admin-service"]
        BC7["Communication Context<br/>communication-service<br/>notification-service"]
        BC8["Innovation Context<br/>ai-service, blockchain-service<br/>gamification-service, iot-service"]
        BC9["Career Context<br/>placement-service<br/>scholarship-service<br/>research-service"]
    end

    BC1 --> BC2
    BC2 --> BC3
    BC3 --> BC5
    BC2 --> BC4
    BC4 --> BC7
    BC2 --> BC8
    BC3 --> BC9
```

---

## 2. Layered Architecture per Service

Each NestJS service follows a clean layered architecture:

```mermaid
graph TB
    subgraph "Service Architecture"
        A["Controller Layer<br/>REST endpoints, input validation"]
        B["Service Layer<br/>Business logic, orchestration"]
        C["Repository Layer<br/>Prisma ORM, data access"]
        D["Infrastructure Layer<br/>Event publishing, external APIs"]
    end

    A --> B
    B --> C
    B --> D
```

### 2.1 NestJS Service Structure

```
services/<service-name>/
  src/
    main.ts                 -- Application bootstrap
    app.module.ts           -- Root module
    controllers/            -- HTTP request handlers
    services/               -- Business logic
    repositories/           -- Data access layer
    dto/                    -- Data transfer objects
    entities/               -- Domain entities
    guards/                 -- Authentication/authorization guards
    interceptors/           -- Request/response interceptors
    pipes/                  -- Validation pipes
    generated/prisma/       -- Generated Prisma client
  prisma/
    schema.prisma           -- Database schema
    migrations/             -- Database migrations
  Dockerfile                -- Container image definition
  package.json              -- Dependencies
  tsconfig.json             -- TypeScript configuration
```

### 2.2 Rust Service Structure (placement-service, research-service)

```
services/<service-name>/
  src/
    main.rs                 -- Entry point
    lib.rs                  -- Library root
    handlers/               -- HTTP handlers
    models/                 -- Domain models
    services/               -- Business logic
    db/                     -- Database access
  Cargo.toml                -- Dependencies
  Dockerfile                -- Container image
```

### 2.3 Go Service Structure (scholarship-service)

```
services/<service-name>/
  cmd/
    main.go                 -- Entry point
  internal/
    handlers/               -- HTTP handlers
    models/                 -- Domain models
    services/               -- Business logic
    repository/             -- Data access
  go.mod                    -- Dependencies
  Dockerfile                -- Container image
```

---

## 3. API Gateway Architecture

The ERP gateway implements the API Gateway pattern, serving as the single entry point for all client applications.

```mermaid
sequenceDiagram
    participant C as Client App
    participant GW as ERP Gateway :8090
    participant IAM as ERP-IAM
    participant PLAT as ERP-Platform
    participant SVC as Target Service

    C->>GW: HTTP Request + Bearer JWT + X-Tenant-ID
    GW->>IAM: Validate JWT
    IAM-->>GW: Token Claims (user, roles)
    GW->>PLAT: Check Entitlements
    PLAT-->>GW: Feature Flags + Tier
    GW->>SVC: Proxy Request + Enriched Headers
    SVC-->>GW: Response
    GW-->>C: Response + CORS Headers
```

### Gateway Routing

| Route Pattern | Target | Description |
|---|---|---|
| `GET /healthz` | Gateway | Health check endpoint |
| `GET /v1/capabilities` | Gateway | Feature discovery |
| `ALL /v1/academic/*` | academic-service | Academic operations |
| `ALL /v1/students/*` | student-service | Student operations |
| `ALL /v1/auth/*` | auth-service | Authentication |
| `ALL /v1/finance/*` | finance-service | Financial operations |
| `ALL /v1/lms/*` | lms-service | Learning management |
| `ALL /v1/admin/*` | admin-service | Administration |
| `ALL /v1/events` | event-service | Event publishing |
| `ALL /v1/:service/*` | Dynamic routing | Catch-all service proxy |

---

## 4. Data Architecture

### 4.1 Database Per Service (Logical Separation)

All services share a single PostgreSQL instance (LumaDB) with logical schema separation. Each service's Prisma schema defines its own models.

```mermaid
graph LR
    subgraph "LumaDB (PostgreSQL 16)"
        S1["Auth Schema<br/>users, sessions, oauth_accounts"]
        S2["Student Schema<br/>students, guardians, enrollments, incidents"]
        S3["Academic Schema<br/>curricula, subjects, classes, grades"]
        S4["Finance Schema<br/>fee_structures, payments, invoices, vendors"]
        S5["LMS Schema<br/>courses, modules, lessons, certificates"]
        S6["Core Schema<br/>schools, audit_logs"]
    end
```

### 4.2 Schema Relationships

```mermaid
erDiagram
    schools ||--o{ users : "employs"
    schools ||--o{ students : "enrolls"
    schools ||--o{ academic_years : "defines"
    academic_years ||--o{ terms : "contains"
    schools ||--o{ courses : "offers"
    courses ||--o{ sections : "divided into"
    sections ||--o{ enrollments : "has"
    students ||--o{ enrollments : "participates"
    sections }o--|| staff : "taught by"
    sections ||--o{ assignments : "contains"
    assignments ||--o{ submissions : "receives"
    submissions ||--o{ grades : "evaluated as"
    students ||--o{ grades : "earns"
    schools ||--o{ fee_types : "defines"
    students ||--o{ invoices : "billed"
    invoices ||--o{ payments : "settled by"
    students ||--o{ blockchain_credentials : "verified by"
    students ||--o{ attendance_records : "tracked"
    users ||--o{ messages : "sends"
    schools ||--o{ announcements : "publishes"
```

### 4.3 CQRS Pattern (Analytics)

```mermaid
graph LR
    subgraph "Command Side"
        C["Services"] --> W["Write to LumaDB"]
        C --> E["Publish Events to Redpanda"]
    end

    subgraph "Query Side"
        E --> P["Event Processors"]
        P --> A["Analytics Store"]
        A --> S["Apache Superset"]
    end
```

---

## 5. Authentication & Authorization Architecture

### 5.1 Authentication Flow

```mermaid
sequenceDiagram
    participant U as User
    participant APP as Client App
    participant AUTH as Auth Service
    participant IAM as ERP-IAM

    U->>APP: Login (email + password)
    APP->>AUTH: POST /v1/auth/login
    AUTH->>AUTH: Validate credentials
    AUTH->>AUTH: Check MFA requirement
    alt MFA Enabled
        AUTH-->>APP: 200 + MFA challenge
        U->>APP: Enter TOTP code
        APP->>AUTH: POST /v1/auth/mfa/verify
    end
    AUTH->>AUTH: Generate JWT + Refresh Token
    AUTH->>AUTH: Create Session record
    AUTH-->>APP: 200 + {accessToken, refreshToken}
    APP->>APP: Store tokens securely
```

### 5.2 Role-Based Access Control

| Role | Access Level | Typical Operations |
|---|---|---|
| SUPER_ADMIN | Full system | Multi-school management, system configuration |
| SCHOOL_ADMIN | School-wide | School settings, staff management, reports |
| PRINCIPAL | School-wide (read-heavy) | Dashboards, approvals, reports |
| VICE_PRINCIPAL | Department-level | Department oversight, schedule management |
| TEACHER | Class-level | Gradebook, attendance, assignments, LMS content |
| STUDENT | Self + enrolled classes | View grades, submit assignments, access LMS |
| PARENT | Child's data | View grades, pay fees, communicate with teachers |
| ACCOUNTANT | Financial data | Fee management, payment processing, reports |
| LIBRARIAN | Library data | Catalog management, book loans |
| IT_ADMIN | Technical systems | User provisioning, system health |
| RECEPTIONIST | Front desk | Visitor management, basic student lookup |

---

## 6. Event Architecture

### 6.1 CloudEvents Envelope

```json
{
  "specversion": "1.0",
  "type": "erp.school_management.student.enrolled",
  "source": "/services/student-service",
  "id": "uuid-v4",
  "time": "2026-02-23T10:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    "studentId": "uuid",
    "schoolId": "uuid",
    "gradeLevel": "Grade 10",
    "academicYearId": "uuid"
  }
}
```

### 6.2 Event Catalog

| Event Type | Producer | Consumers |
|---|---|---|
| `student.enrolled` | student-service | finance, academic, notification |
| `student.graduated` | academic-service | blockchain, placement, analytics |
| `grade.published` | academic-service | notification, analytics, ai |
| `payment.completed` | finance-service | notification, analytics |
| `attendance.marked` | academic-service | notification, analytics, ai |
| `assignment.submitted` | lms-service | academic, notification, gamification |
| `certificate.issued` | blockchain-service | notification, analytics |
| `assessment.graded` | academic-service | notification, analytics, gamification |

---

## 7. Shared Packages Architecture

```mermaid
graph TB
    subgraph "Shared Packages"
        CFG["@educore/config<br/>Shared configuration"]
        DB["@educore/database<br/>Database utilities"]
        LOG["@educore/logger<br/>Structured logging"]
        SA["@educore/superset-adapter<br/>BI integration"]
        TYP["@educore/types<br/>TypeScript types"]
        UI["@educore/ui<br/>Shared components"]
        UTL["@educore/utils<br/>Common utilities"]
    end

    subgraph "Consumers"
        SVC["25 Services"]
        APP["5 Frontend Apps"]
    end

    CFG & LOG & TYP & UTL --> SVC
    UI & TYP & UTL --> APP
    DB --> SVC
    SA --> SVC
```

---

## 8. Error Handling Strategy

### 8.1 Standard Error Response

```json
{
  "statusCode": 400,
  "error": "VALIDATION_ERROR",
  "message": "Invalid student enrollment data",
  "details": [
    {
      "field": "dateOfBirth",
      "constraint": "Must be a valid date in the past"
    }
  ],
  "traceId": "otel-trace-uuid",
  "timestamp": "2026-02-23T10:30:00Z"
}
```

### 8.2 Error Categories

| Category | HTTP Code | Handling |
|---|---|---|
| Validation | 400 | Return field-level details |
| Authentication | 401 | Redirect to login |
| Authorization | 403 | Log access attempt |
| Not Found | 404 | Return entity type |
| Conflict | 409 | Return conflicting field |
| Rate Limit | 429 | Return retry-after header |
| Internal | 500 | Log full trace, return generic message |

---

## 9. Scalability Patterns

| Pattern | Implementation |
|---|---|
| Horizontal Scaling | Kubernetes HPA on CPU/memory metrics |
| Database Read Replicas | PostgreSQL streaming replication |
| Event-Driven Decoupling | Redpanda for async processing |
| Caching | Redis (planned) for session and query caching |
| CDN | Static assets served via CDN |
| Connection Pooling | PgBouncer for database connections |
| Build Caching | Turborepo remote caching |
