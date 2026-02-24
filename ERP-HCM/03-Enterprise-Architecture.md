# ERP-HCM Enterprise Architecture

## TOGAF-Aligned Enterprise Architecture Document

### Version: 1.0.0
### Date: 2026-02-23
### Classification: Internal

---

## 1. Architecture Vision

ERP-HCM delivers a unified Human Capital Management platform within the broader ERP product-line architecture. The vision is to provide organizations with a single, comprehensive system for managing the entire employee lifecycle -- from recruitment and onboarding through payroll, performance, learning, and retirement -- while maintaining the flexibility to operate standalone or as part of the integrated ERP suite.

### 1.1 Business Context

```mermaid
C4Context
    title ERP-HCM Business Context

    Person(employee, "Employee", "Self-service HR tasks")
    Person(manager, "Manager", "Team management, approvals")
    Person(hr_admin, "HR Admin", "HR operations, policy management")
    Person(payroll_admin, "Payroll Admin", "Payroll processing, compliance")
    Person(executive, "Executive", "Workforce analytics, planning")

    System(hcm, "ERP-HCM", "Human Capital Management Platform")

    System_Ext(erp_platform, "ERP-Platform", "Entitlements & subscription hub")
    System_Ext(erp_iam, "ERP-IAM/Directory", "Identity & access management")
    System_Ext(erp_finance, "ERP-Finance", "General ledger, accounts payable")
    System_Ext(payment_gw, "Payment Gateways", "Flutterwave, Remita")
    System_Ext(govt, "Government Portals", "FIRS, PenCom, NHIS")

    Rel(employee, hcm, "Self-service")
    Rel(manager, hcm, "Approvals, team view")
    Rel(hr_admin, hcm, "HR operations")
    Rel(payroll_admin, hcm, "Payroll runs")
    Rel(executive, hcm, "Analytics")

    Rel(hcm, erp_platform, "Entitlement checks")
    Rel(hcm, erp_iam, "SSO, JWT tokens")
    Rel(hcm, erp_finance, "GL postings")
    Rel(hcm, payment_gw, "Salary disbursement")
    Rel(hcm, govt, "Statutory filings")
```

### 1.2 Architecture Principles

| Principle | Rationale |
|-----------|-----------|
| Multi-tenant by default | Single deployment serves multiple organizations |
| Event-driven integration | Loose coupling between modules via NATS JetStream |
| AIDD guardrails | AI-driven operations governed by explicit risk tiers |
| Domain isolation | Each service owns its data and business logic |
| API-first design | REST APIs with CloudEvents for async communication |
| Nigerian-first compliance | First-class support for Nigerian tax, pension, labor law |

---

## 2. Business Architecture

### 2.1 Business Capability Map

```mermaid
mindmap
  root((ERP-HCM))
    Core HR
      Employee Lifecycle
      Organization Structure
      Department Management
      Position Management
      Org Chart
    Talent Acquisition
      Job Requisitions
      Candidate Pipeline
      AI Resume Parsing
      Interview Management
      Offer Management
      Onboarding
    Compensation & Benefits
      Salary Structures
      Pay Grades
      Compensation Cycles
      Benefits Plans
      Enrollment Management
      EWA
    Payroll
      Multi-Country Engine
      Nigerian PAYE/NHF/Pension
      Payslip Generation
      Disbursement
      Statutory Filing
    Time & Attendance
      Clock In/Out
      Geofencing
      Biometric
      Shift Scheduling
      Overtime
    Performance
      OKR Cycles
      KPI Tracking
      360 Reviews
      Calibration
      9-Box Grid
    Learning
      Course Catalog
      SCORM/xAPI
      Certifications
      Learning Paths
    Workforce Planning
      Headcount Planning
      Scenario Modeling
      Budget Allocation
    Compliance
      Labor Law
      GDPR/NDPR
      Audit Trails
```

### 2.2 Value Stream: Hire-to-Retire

```mermaid
graph LR
    A[Workforce Planning] --> B[Requisition]
    B --> C[Sourcing]
    C --> D[Screening]
    D --> E[Interview]
    E --> F[Offer]
    F --> G[Onboarding]
    G --> H[Active Employment]
    H --> I[Performance Mgmt]
    H --> J[Learning & Dev]
    H --> K[Compensation Review]
    H --> L[Benefits Admin]
    I --> M{Career Decision}
    M -->|Promote| H
    M -->|Transfer| H
    M -->|Exit| N[Offboarding]
    N --> O[Alumni]
```

### 2.3 Business Process Architecture

| Process | Owner | Frequency | SLA |
|---------|-------|-----------|-----|
| New Hire Onboarding | HR Admin | Per event | 3 business days |
| Monthly Payroll Run | Payroll Admin | Monthly | T-2 before pay date |
| Leave Approval | Manager | Per event | 24 hours |
| Performance Review | HR Admin | Quarterly/Annual | 2 weeks |
| Benefits Enrollment | HR Admin | Annual + life events | 30 days |
| Compliance Audit | Compliance Officer | Quarterly | 5 business days |

---

## 3. Information Systems Architecture

### 3.1 Application Architecture

```mermaid
flowchart TB
    subgraph "Presentation Layer"
        WEB[Next.js 14 Web App]
        MOB[Flutter Mobile App]
        PORTAL[Employee Self-Service Portal]
    end

    subgraph "API Gateway Layer"
        GW[API Gateway<br/>Chi Router + Middleware]
    end

    subgraph "Service Layer"
        EMP[Employee Service]
        PAY[Payroll Service]
        REC[Recruitment Service]
        PERF[Performance Service]
        LV[Leave Service]
        TA[Time & Attendance Service]
        BEN[Benefits Service]
        LMS[Learning Service]
        COMP[Compensation Service]
        WFP[Workforce Planning Service]
        COMPLY[Compliance Service]
        DMS[Document Service]
        FAC[Facilities Service]
        FM[Facilities Management]
    end

    subgraph "Data Layer"
        PG[(PostgreSQL 16)]
        TS[(TimescaleDB)]
        RD[(Redis 7)]
        MS[(Meilisearch)]
    end

    subgraph "Integration Layer"
        NATS[NATS JetStream]
        VAULT[HashiCorp Vault]
        S3[AWS S3]
    end

    subgraph "External Systems"
        IAM[ERP-IAM]
        PLAT[ERP-Platform]
        FIN[ERP-Finance]
        FW[Flutterwave]
        REM[Remita]
    end

    WEB --> GW
    MOB --> GW
    PORTAL --> GW

    GW --> EMP
    GW --> PAY
    GW --> REC
    GW --> PERF
    GW --> LV
    GW --> TA
    GW --> BEN
    GW --> LMS
    GW --> COMP
    GW --> WFP
    GW --> COMPLY
    GW --> DMS
    GW --> FAC
    GW --> FM

    EMP --> PG
    PAY --> PG
    TA --> TS
    EMP --> RD
    REC --> MS

    EMP --> NATS
    PAY --> NATS
    PAY --> FW
    PAY --> REM
    DMS --> S3
    GW --> IAM
    GW --> PLAT
    PAY --> FIN
    EMP --> VAULT
```

### 3.2 Data Architecture

```mermaid
graph TB
    subgraph "Operational Data"
        EMP_DB[Employee Schema<br/>legal_entities, locations, departments,<br/>employees, positions, cost_centers]
        PAY_DB[Payroll Schema<br/>pay_grades, salary_components,<br/>payroll_periods, payroll_runs,<br/>payroll_entries, payslips]
        LV_DB[Leave Schema<br/>leave_types, leave_balances,<br/>leave_requests, holidays]
        REC_DB[Recruitment Schema<br/>job_requisitions, candidates,<br/>applications, assessments]
        PERF_DB[Performance Schema<br/>okr_cycles, objectives,<br/>review_cycles, reviews]
        ATT_DB[Attendance Schema<br/>clock_records, geofences,<br/>shifts, biometric_templates]
    end

    subgraph "Analytical Data"
        ANA_DB[Analytics Schema<br/>dashboards, reports,<br/>metrics, KPIs]
        TS_DB[Time-Series<br/>attendance_events,<br/>location_pings]
    end

    subgraph "Security Data"
        AUTH_DB[Auth Schema<br/>users, sessions, roles,<br/>permissions, mfa_tokens]
        AUDIT_DB[Audit Schema<br/>audit_logs, data_changes,<br/>compliance_records]
    end

    EMP_DB --> ANA_DB
    PAY_DB --> ANA_DB
    ATT_DB --> TS_DB
```

---

## 4. Technology Architecture

### 4.1 Infrastructure View

```mermaid
graph TB
    subgraph "CDN / Edge"
        CF[Cloudflare / AWS CloudFront]
    end

    subgraph "Load Balancer"
        LB[Application Load Balancer]
    end

    subgraph "Kubernetes Cluster"
        subgraph "Namespace: erp-hcm"
            GW_POD[Gateway Pods x3]
            EMP_POD[Employee Pods x2]
            PAY_POD[Payroll Pods x2]
            REC_POD[Recruitment Pods x2]
            PERF_POD[Performance Pods x2]
            LV_POD[Leave Pods x2]
            TA_POD[Attendance Pods x2]
            BEN_POD[Benefits Pods x2]
            LMS_POD[Learning Pods x2]
            OTHER_POD[Other Services...]
        end
    end

    subgraph "Data Tier"
        PG_PRI[(PostgreSQL Primary)]
        PG_REP[(PostgreSQL Replica)]
        RD_CL[(Redis Cluster)]
        NATS_CL[(NATS Cluster)]
    end

    CF --> LB
    LB --> GW_POD
    GW_POD --> EMP_POD
    GW_POD --> PAY_POD
    GW_POD --> REC_POD
    GW_POD --> PERF_POD
    GW_POD --> LV_POD
    GW_POD --> TA_POD
    GW_POD --> BEN_POD
    GW_POD --> LMS_POD
    GW_POD --> OTHER_POD

    EMP_POD --> PG_PRI
    PAY_POD --> PG_PRI
    PG_PRI --> PG_REP
    EMP_POD --> RD_CL
    EMP_POD --> NATS_CL
```

### 4.2 Technology Standards

| Layer | Standard | Implementation |
|-------|----------|----------------|
| Language | Go 1.24+ | Microservices backend |
| Frontend | Next.js 14 | Server-side rendered web app |
| Mobile | Flutter | Cross-platform mobile |
| Database | PostgreSQL 16 | Primary data store |
| Time-Series | TimescaleDB | Attendance events |
| Cache | Redis 7 | Session, rate-limit, config cache |
| Messaging | NATS JetStream | Event-driven integration |
| Search | Meilisearch | Full-text employee/candidate search |
| API | REST + CloudEvents | Synchronous + asynchronous |
| Auth | RS256 JWT + MFA | Token-based authentication |
| Encryption | AES-256-GCM | Field-level PII encryption |
| Key Management | HashiCorp Vault | Encryption key rotation |
| Monitoring | Prometheus + OTel | Metrics and tracing |
| Logging | zerolog (JSON) | Structured logging |
| Container | Docker (Alpine) | < 20MB images |
| Orchestration | Kubernetes | Production deployment |

---

## 5. Governance

### 5.1 Architecture Decision Records

All significant architectural decisions are documented as ADRs in `docs/ADR/`. Key decisions include:

- **ADR-001**: Go selected as primary backend language for performance and concurrency
- **ADR-002**: PostgreSQL 16 selected for relational data with JSONB support

### 5.2 AIDD Governance Framework

The AIDD guardrails file (`erp/aidd.guardrails.yaml`) establishes three tiers of operational governance:

```mermaid
graph TB
    subgraph "AIDD Risk Tiers"
        A[Autonomous<br/>Read queries, notifications]
        S[Supervised<br/>Data mutations, workflows, bulk ops]
        P[Prohibited<br/>Cross-tenant access, irreversible delete,<br/>privilege escalation]
    end

    subgraph "Controls"
        HITL[Human-in-the-loop for high-risk]
        LOG[Decision logging required]
        RB[24-hour rollback window]
    end

    A --> LOG
    S --> HITL
    S --> LOG
    S --> RB
    P -.->|BLOCKED| X[Rejected]
```

### 5.3 Compliance Framework

| Standard | Scope | Implementation |
|----------|-------|----------------|
| GDPR | EU employee data | DSR handler, data minimization, encryption |
| NDPR | Nigerian data protection | Consent management, breach notification |
| SOC 2 | Trust services | Audit logging, access controls, encryption |
| PCI DSS | Payment card data | Tokenization via payment gateways |
| NPC | Nigerian pension compliance | Pension calculator, PenCom reporting |

---

## 6. Integration Architecture

### 6.1 Integration Pattern: Event-Driven

```mermaid
sequenceDiagram
    participant EMP as Employee Service
    participant NATS as NATS JetStream
    participant PAY as Payroll Service
    participant FIN as ERP-Finance
    participant NOTIFY as Notification Service

    EMP->>NATS: Publish erp.hcm.employee.created
    NATS->>PAY: Deliver (subscription)
    PAY->>PAY: Create payroll record
    NATS->>FIN: Deliver (cross-module)
    FIN->>FIN: Create GL journal entry
    NATS->>NOTIFY: Deliver
    NOTIFY->>NOTIFY: Send welcome email
```

### 6.2 Integration Catalog

| Source | Target | Pattern | Protocol | Event |
|--------|--------|---------|----------|-------|
| Employee | Payroll | Event | NATS | employee.created/updated |
| Payroll | Finance | Event | NATS | payroll.paid |
| Employee | IAM | Sync | REST | employee.created |
| Payroll | Payment GW | Request | REST | disbursement |
| Attendance | Payroll | Event | NATS | attendance.summary |
| Leave | Attendance | Event | NATS | leave.approved |
| Recruitment | Employee | Event | NATS | offer.accepted |
| All Services | Analytics | Event | NATS | *.created/*.updated |

---

## 7. Migration and Transition Architecture

### 7.1 Consolidation Transition

```mermaid
gantt
    title ERP-HCM Consolidation Timeline
    dateFormat  YYYY-MM-DD
    section Phase 1: Foundation
    Initialize ERP-HCM module           :done, p1a, 2026-02-23, 1d
    Apply consolidation contracts        :done, p1b, 2026-02-23, 1d
    Deep import source directories       :done, p1c, 2026-02-23, 1d
    Isolate nested modules               :done, p1d, 2026-02-23, 1d
    section Phase 2: Integration
    Unified API gateway                  :active, p2a, 2026-02-24, 14d
    Event backbone wiring                :p2b, 2026-02-24, 14d
    Cross-service integration tests      :p2c, 2026-03-10, 7d
    section Phase 3: Production
    Staging deployment                   :p3a, 2026-03-17, 7d
    Performance benchmarking             :p3b, 2026-03-17, 7d
    GA release                           :milestone, p3c, 2026-03-24, 0d
```

### 7.2 Legacy Migration Strategy

Organizations migrating from legacy HR systems should follow a phased approach:

1. **Assessment**: Inventory existing HR data and processes
2. **Data Migration**: Use bulk import engines with validation
3. **Parallel Run**: Run both systems for 1-2 payroll cycles
4. **Cutover**: Switch to ERP-HCM as system of record
5. **Decommission**: Archive legacy system data

---

## 8. Risk Assessment

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Data loss during migration | High | Low | Preserved source snapshots, rollback window |
| Nigerian tax law changes | Medium | Medium | Configurable tax bands, rapid deployment |
| Cross-tenant data leakage | Critical | Low | Tenant middleware, field-level encryption, AIDD guardrails |
| Service outage during payroll | High | Low | Multi-replica deployment, automated failover |
| Compliance violation | High | Low | Audit logging, DSR automation, encryption |
