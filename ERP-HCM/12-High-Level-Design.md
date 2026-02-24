# ERP-HCM High-Level Design (HLD)

## Version 1.0.0 | Date: 2026-02-23

---

## 1. System Overview

ERP-HCM is a multi-tenant, microservices-based Human Capital Management platform built with Go 1.24+ backend services and a Next.js 14 frontend. The system manages the complete employee lifecycle from recruitment through retirement, with particular depth in Nigerian payroll compliance.

### 1.1 Component Overview

```mermaid
flowchart TB
    subgraph "Client Layer"
        WEB[Next.js 14 Web App<br/>React 18, Tailwind, Radix UI]
        MOB[Flutter Mobile App<br/>Biometric, GPS]
    end

    subgraph "Gateway Layer"
        GW[API Gateway<br/>Go/Chi Router<br/>Auth, Rate Limit, Tenant]
    end

    subgraph "Application Layer"
        direction LR
        subgraph "Core Services"
            EMP[Employee<br/>Service]
            PAY[Payroll<br/>Service]
            LV[Leave<br/>Service]
        end
        subgraph "Talent Services"
            REC[Recruitment<br/>Service]
            PERF[Performance<br/>Service]
            LMS[Learning<br/>Service]
        end
        subgraph "Operations Services"
            TA[Attendance<br/>Service]
            BEN[Benefits<br/>Service]
            COMP[Compensation<br/>Service]
        end
        subgraph "Platform Services"
            WFP[Workforce<br/>Planning]
            COMPLY[Compliance<br/>Service]
            DMS[Document<br/>Service]
            FAC[Facilities<br/>Service]
        end
    end

    subgraph "Data Layer"
        PG[(PostgreSQL 16<br/>30+ Schemas)]
        TS[(TimescaleDB<br/>Time-Series)]
        RD[(Redis 7<br/>Cache)]
        MS[(Meilisearch<br/>Search)]
    end

    subgraph "Integration Layer"
        NATS[NATS JetStream<br/>Event Backbone]
        S3[AWS S3<br/>File Storage]
        VAULT[HashiCorp Vault<br/>Key Management]
    end

    WEB --> GW
    MOB --> GW
    GW --> EMP & PAY & LV & REC & PERF & LMS & TA & BEN & COMP & WFP & COMPLY & DMS & FAC

    EMP & PAY & LV --> PG
    TA --> TS
    GW --> RD
    REC --> MS

    EMP & PAY --> NATS
    DMS --> S3
    EMP --> VAULT
```

---

## 2. Data Flow Architecture

### 2.1 Request Flow

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant Service
    participant DB as PostgreSQL
    participant Cache as Redis
    participant Events as NATS

    Client->>Gateway: HTTPS Request + JWT + X-Tenant-ID
    Gateway->>Gateway: CORS Check
    Gateway->>Gateway: CSRF Validation
    Gateway->>Gateway: Rate Limit Check
    Gateway->>Cache: Validate JWT (session cache)
    Cache-->>Gateway: Session valid
    Gateway->>Gateway: Extract Tenant ID
    Gateway->>Gateway: Audit Log Entry
    Gateway->>Service: Route to service + context
    Service->>DB: Query with tenant_id scope
    DB-->>Service: Result set
    Service->>Events: Publish domain event
    Service-->>Gateway: JSON response
    Gateway-->>Client: HTTPS Response
```

### 2.2 Payroll Calculation Data Flow

```mermaid
flowchart LR
    subgraph "Input Data"
        EMP_DATA[Employee Records<br/>Salary, Bank, Tax Info]
        SAL_STRUCT[Salary Structures<br/>Components, Rates]
        ATT_DATA[Attendance Data<br/>Hours, Overtime]
        LV_DATA[Leave Records<br/>Unpaid Leave Days]
        LOAN_DATA[Loan/Advance<br/>Deductions]
    end

    subgraph "Calculation Engine"
        CALC[Payroll Processor]
        NG_TAX[Nigeria Tax Engine<br/>PAYE, CRA]
        PENSION[Pension Calculator<br/>8%/10%]
        NHF_CALC[NHF Calculator<br/>2.5%]
        OT_CALC[Overtime Calculator]
        PRORATE[Proration Engine]
    end

    subgraph "Output"
        ENTRIES[Payroll Entries<br/>Per Employee]
        PAYSLIPS[PDF Payslips]
        BANK_FILE[NIBSS Bank File]
        REPORTS[Statutory Reports]
        EVENTS[NATS Events]
    end

    EMP_DATA --> CALC
    SAL_STRUCT --> CALC
    ATT_DATA --> CALC
    LV_DATA --> CALC
    LOAN_DATA --> CALC

    CALC --> NG_TAX
    CALC --> PENSION
    CALC --> NHF_CALC
    CALC --> OT_CALC
    CALC --> PRORATE

    NG_TAX --> ENTRIES
    PENSION --> ENTRIES
    NHF_CALC --> ENTRIES
    OT_CALC --> ENTRIES
    PRORATE --> ENTRIES

    ENTRIES --> PAYSLIPS
    ENTRIES --> BANK_FILE
    ENTRIES --> REPORTS
    ENTRIES --> EVENTS
```

---

## 3. Service Architecture

### 3.1 Service Registry

| Service | API Prefix | Database Schema | Event Namespace |
|---------|------------|-----------------|-----------------|
| employee-service | `/v1/employee` | `employee.*` | `erp.hcm.employee.*` |
| payroll-service | `/v1/payroll` | `payroll.*` | `erp.hcm.payroll.*` |
| leave-service | `/v1/leave` | `leave.*` | `erp.hcm.leave.*` |
| recruitment-service | `/v1/recruitment` | `recruitment.*` | `erp.hcm.recruitment.*` |
| performance-service | `/v1/performance` | `performance.*` | `erp.hcm.performance.*` |
| time-attendance-service | `/v1/time-attendance` | `attendance.*` | `erp.hcm.time-attendance.*` |
| benefits-service | `/v1/benefits` | `benefits.*` | `erp.hcm.benefits.*` |
| learning-service | `/v1/learning` | `lms.*` | `erp.hcm.learning.*` |
| compensation-service | `/v1/compensation` | `workforce.*` | `erp.hcm.compensation.*` |
| workforce-planning-service | `/v1/workforce-planning` | `workforce.*` | `erp.hcm.workforce-planning.*` |
| compliance-service | `/v1/compliance` | `compliance.*` | `erp.hcm.compliance.*` |
| document-service | `/v1/document` | `dms.*` | `erp.hcm.document.*` |
| facilities-service | `/v1/facilities` | `facilities.*` | `erp.hcm.facilities.*` |

### 3.2 Internal Package Structure (per service)

```
internal/<domain>/
    domain/         # Domain models, value objects, enums
    service/        # Business logic, orchestration
    engine/         # Calculation engines (payroll, tax, etc.)
    repository/     # Data access layer (PostgreSQL)
    handlers/       # HTTP handlers (Chi router)
    adapters/       # External service adapters
```

---

## 4. Security Architecture

### 4.1 Authentication Flow

```mermaid
sequenceDiagram
    participant User
    participant Gateway
    participant AuthService
    participant Redis
    participant DB as PostgreSQL

    User->>Gateway: POST /auth/login {email, password}
    Gateway->>AuthService: Forward credentials
    AuthService->>DB: Lookup user by email
    DB-->>AuthService: User record (hashed password)
    AuthService->>AuthService: Verify bcrypt hash
    AuthService->>AuthService: Check account lockout
    alt MFA Enabled
        AuthService-->>User: MFA challenge
        User->>AuthService: TOTP code
        AuthService->>AuthService: Verify TOTP
    end
    AuthService->>AuthService: Sign JWT (RS256)
    AuthService->>Redis: Store session
    AuthService-->>User: {access_token, refresh_token}
```

### 4.2 Encryption Architecture

```mermaid
flowchart TB
    subgraph "Application Layer"
        SVC[Service] --> ENC[Encryption Service]
    end

    subgraph "Key Management"
        ENC --> CACHE[Key Cache<br/>In-Memory TTL]
        CACHE --> VAULT[HashiCorp Vault<br/>Transit Engine]
        VAULT --> MASTER[Master Key<br/>HSM-Backed]
    end

    subgraph "Data Layer"
        ENC --> DB[(PostgreSQL<br/>Encrypted Fields)]
    end

    subgraph "Encrypted Fields"
        F1[Bank Account Number]
        F2[Tax ID / TIN]
        F3[National ID]
        F4[RSA PIN]
        F5[Passport Number]
    end

    DB --> F1 & F2 & F3 & F4 & F5
```

---

## 5. Multi-Tenancy Architecture

```mermaid
flowchart TB
    REQ[HTTP Request] --> GW[Gateway]
    GW --> JWT_DECODE[Decode JWT<br/>Extract tenant_id]
    JWT_DECODE --> TENANT_CTX[Store in Context<br/>tenantContextKey]

    subgraph "Super Admin Override"
        JWT_DECODE --> SA_CHECK{Is Super Admin?}
        SA_CHECK -->|Yes| HEADER_CHECK{X-Tenant-ID<br/>Header Present?}
        HEADER_CHECK -->|Yes| OVERRIDE[Use Header Tenant ID]
        HEADER_CHECK -->|No| USE_JWT[Use JWT Tenant ID]
        SA_CHECK -->|No| USE_JWT
    end

    TENANT_CTX --> SERVICE[Service Layer]
    SERVICE --> REPO[Repository Layer]
    REPO --> QUERY[SQL Query<br/>WHERE tenant_id = $1]
    QUERY --> DB[(PostgreSQL)]
```

---

## 6. Caching Strategy

```mermaid
flowchart TB
    subgraph "Cache Tiers"
        L1[L1: In-Process<br/>Go sync.Map<br/>TTL: 1 min]
        L2[L2: Redis<br/>Distributed Cache<br/>TTL: 5-60 min]
        L3[L3: PostgreSQL<br/>Source of Truth]
    end

    subgraph "Cached Entities"
        C1[Session Data - 15min TTL]
        C2[Rate Limit Counters - 1min Window]
        C3[Configuration - 1hr TTL]
        C4[Employee Summary - 5min TTL]
        C5[Payroll Templates - 1hr TTL]
    end

    L1 --> L2 --> L3
    C1 --> L2
    C2 --> L2
    C3 --> L1
    C4 --> L2
    C5 --> L1
```

---

## 7. High Availability Design

```mermaid
flowchart TB
    subgraph "Region: Primary"
        LB1[Load Balancer]
        subgraph "K8s Cluster"
            GW1[Gateway x3]
            SVC1[Services x2 each]
        end
        PG1[(PG Primary)]
        RD1[(Redis Primary)]
        NATS1[(NATS Cluster x3)]
    end

    subgraph "Region: DR"
        LB2[Load Balancer - Standby]
        subgraph "K8s Cluster DR"
            GW2[Gateway x2]
            SVC2[Services x1 each]
        end
        PG2[(PG Standby<br/>Streaming Replication)]
        RD2[(Redis Replica)]
    end

    PG1 -->|WAL Streaming| PG2
    RD1 -->|Replication| RD2
    LB1 -.->|DNS Failover| LB2
```

---

## 8. Monitoring and Observability

```mermaid
flowchart LR
    subgraph "Application"
        APP[Go Services]
        FE[Next.js Frontend]
    end

    subgraph "Collection"
        PROM[Prometheus<br/>Metrics Scraping]
        OTEL[OpenTelemetry<br/>Trace Collection]
        LOG[zerolog<br/>JSON Logs]
    end

    subgraph "Storage"
        PROM_DB[(Prometheus TSDB)]
        JAEGER[(Jaeger/Tempo)]
        LOKI[(Loki/ELK)]
    end

    subgraph "Visualization"
        GRAF[Grafana<br/>Dashboards]
        ALERT[AlertManager<br/>PagerDuty]
    end

    APP --> PROM
    APP --> OTEL
    APP --> LOG

    PROM --> PROM_DB
    OTEL --> JAEGER
    LOG --> LOKI

    PROM_DB --> GRAF
    JAEGER --> GRAF
    LOKI --> GRAF
    PROM_DB --> ALERT
```
