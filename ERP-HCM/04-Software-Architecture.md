# ERP-HCM Software Architecture (C4 Model)

## Version: 1.0.0 | Date: 2026-02-23

---

## 1. System Context (Level 1)

```mermaid
C4Context
    title ERP-HCM System Context Diagram

    Person(emp, "Employee", "Self-service HR portal")
    Person(mgr, "Manager", "Team management and approvals")
    Person(hra, "HR Admin", "HR operations and configuration")
    Person(pra, "Payroll Admin", "Payroll processing and compliance")

    System(hcm, "ERP-HCM", "Human Capital Management<br/>14 microservices")

    System_Ext(iam, "ERP-IAM", "Identity Provider<br/>OIDC/JWT")
    System_Ext(plat, "ERP-Platform", "Subscription & Entitlements")
    System_Ext(fin, "ERP-Finance", "General Ledger & AP")
    System_Ext(nats, "NATS JetStream", "Event Backbone")
    System_Ext(vault, "HashiCorp Vault", "Key Management")
    System_Ext(s3, "AWS S3", "File Storage")
    System_Ext(pay_gw, "Payment Gateways", "Flutterwave, Remita")

    Rel(emp, hcm, "Uses", "HTTPS")
    Rel(mgr, hcm, "Uses", "HTTPS")
    Rel(hra, hcm, "Uses", "HTTPS")
    Rel(pra, hcm, "Uses", "HTTPS")

    Rel(hcm, iam, "Authenticates via", "OIDC/JWT")
    Rel(hcm, plat, "Checks entitlements", "REST")
    Rel(hcm, fin, "Posts GL entries", "NATS")
    Rel(hcm, nats, "Publishes/subscribes events", "TCP")
    Rel(hcm, vault, "Key management", "HTTPS")
    Rel(hcm, s3, "Document storage", "HTTPS")
    Rel(hcm, pay_gw, "Salary disbursement", "HTTPS")
```

---

## 2. Container Diagram (Level 2)

```mermaid
C4Container
    title ERP-HCM Container Diagram

    Person(user, "User", "Employee/Manager/Admin")

    Container(web, "Web Application", "Next.js 14", "Employee self-service portal<br/>React, Tailwind, Radix UI")
    Container(mobile, "Mobile App", "Flutter", "Mobile HR app with biometric")

    Container(gw, "API Gateway", "Go/Chi", "Authentication, routing,<br/>rate limiting, tenant context")

    Container(emp_svc, "Employee Service", "Go", "Employee lifecycle,<br/>onboarding, org chart")
    Container(pay_svc, "Payroll Service", "Go", "Multi-country payroll,<br/>salary, disbursement")
    Container(rec_svc, "Recruitment Service", "Go", "ATS, candidate pipeline,<br/>AI resume parsing")
    Container(perf_svc, "Performance Service", "Go", "OKR, KPI, 360 reviews,<br/>calibration")
    Container(lv_svc, "Leave Service", "Go", "Leave management,<br/>policies, approvals")
    Container(ta_svc, "Time & Attendance", "Go", "Clock-in/out, geofence,<br/>biometric, shifts")
    Container(ben_svc, "Benefits Service", "Go", "Benefits plans,<br/>enrollment, EWA")
    Container(lms_svc, "Learning Service", "Go", "LMS, SCORM/xAPI,<br/>certifications")
    Container(comp_svc, "Compensation Service", "Go", "Salary bands, pay grades,<br/>compensation cycles")
    Container(wfp_svc, "Workforce Planning", "Go", "Headcount planning,<br/>scenario modeling")
    Container(comply_svc, "Compliance Service", "Go", "Labor law, GDPR/NDPR,<br/>audit trails")
    Container(dms_svc, "Document Service", "Go", "Document mgmt,<br/>digital signatures")
    Container(fac_svc, "Facilities Service", "Go", "Room/desk booking")

    ContainerDb(pg, "PostgreSQL 16", "Database", "Primary data store<br/>30+ schemas")
    ContainerDb(ts, "TimescaleDB", "Time-Series DB", "Attendance events")
    ContainerDb(redis, "Redis 7", "Cache", "Sessions, rate limits")
    ContainerDb(ms, "Meilisearch", "Search Engine", "Full-text search")

    Container(nats, "NATS JetStream", "Message Broker", "Event backbone")

    Rel(user, web, "Uses", "HTTPS")
    Rel(user, mobile, "Uses", "HTTPS")
    Rel(web, gw, "API calls", "HTTPS/JSON")
    Rel(mobile, gw, "API calls", "HTTPS/JSON")

    Rel(gw, emp_svc, "Routes", "HTTP")
    Rel(gw, pay_svc, "Routes", "HTTP")
    Rel(gw, rec_svc, "Routes", "HTTP")
    Rel(gw, perf_svc, "Routes", "HTTP")
    Rel(gw, lv_svc, "Routes", "HTTP")
    Rel(gw, ta_svc, "Routes", "HTTP")
    Rel(gw, ben_svc, "Routes", "HTTP")
    Rel(gw, lms_svc, "Routes", "HTTP")
    Rel(gw, comp_svc, "Routes", "HTTP")
    Rel(gw, wfp_svc, "Routes", "HTTP")
    Rel(gw, comply_svc, "Routes", "HTTP")
    Rel(gw, dms_svc, "Routes", "HTTP")
    Rel(gw, fac_svc, "Routes", "HTTP")

    Rel(emp_svc, pg, "Reads/Writes", "pgx")
    Rel(pay_svc, pg, "Reads/Writes", "pgx")
    Rel(ta_svc, ts, "Writes events", "pgx")
    Rel(gw, redis, "Session/cache", "TCP")
    Rel(rec_svc, ms, "Search index", "HTTP")

    Rel(emp_svc, nats, "Publishes events", "TCP")
    Rel(pay_svc, nats, "Publishes events", "TCP")
```

---

## 3. Component Diagram (Level 3) -- Payroll Service

```mermaid
C4Component
    title Payroll Service - Component Diagram

    Container_Boundary(pay, "Payroll Service") {
        Component(handler, "HTTP Handlers", "Go/Chi", "REST API endpoints<br/>/v1/payroll/*")
        Component(service, "Payroll Service", "Go", "Business logic orchestration")
        Component(engine, "Payroll Engine", "Go", "Calculation engine")
        Component(ng_calc, "Nigeria Calculator", "Go", "PAYE, CRA, Pension,<br/>NHF, NHIS, NSITF")
        Component(tax_eng, "Tax Engine", "Go", "Multi-country tax<br/>calculation framework")
        Component(pension_eng, "Pension Calculator", "Go", "Employee + employer<br/>contribution calculator")
        Component(payslip_gen, "Payslip Generator", "Go", "PDF payslip generation")
        Component(bank_gen, "Bank File Generator", "Go", "NIBSS-format bank<br/>instruction files")
        Component(approval, "Approval Workflow", "Go", "Multi-level payroll<br/>approval chain")
        Component(analytics, "Payroll Analytics", "Go", "Variance analysis,<br/>period comparison")
        Component(ewa_eng, "EWA Engine", "Go", "Earned Wage Access<br/>calculation")
        Component(repo, "Repository", "Go/pgx", "PostgreSQL data access")
        Component(disbursement, "Disbursement Service", "Go", "Payment processing")
        Component(flw, "Flutterwave Adapter", "Go", "Flutterwave integration")
        Component(rem, "Remita Adapter", "Go", "Remita integration")
    }

    Rel(handler, service, "Delegates")
    Rel(service, engine, "Calculates")
    Rel(engine, ng_calc, "Nigerian payroll")
    Rel(engine, tax_eng, "Tax calculation")
    Rel(engine, pension_eng, "Pension calc")
    Rel(service, payslip_gen, "Generates payslips")
    Rel(service, bank_gen, "Generates bank files")
    Rel(service, approval, "Manages approvals")
    Rel(service, analytics, "Analyzes")
    Rel(service, ewa_eng, "EWA processing")
    Rel(service, repo, "Persists")
    Rel(service, disbursement, "Disburses")
    Rel(disbursement, flw, "Flutterwave")
    Rel(disbursement, rem, "Remita")
```

---

## 4. Component Diagram (Level 3) -- Employee Service

```mermaid
flowchart TB
    subgraph "Employee Service"
        H[HTTP Handlers<br/>handlers.go]
        EH[Enhanced Handlers<br/>enhanced_handlers.go]
        OH[Onboarding Handler<br/>onboarding_handler.go]

        SVC[Employee Service<br/>service.go]
        ANA[Analytics Service<br/>analytics_service.go]

        OE[Onboarding Engine<br/>onboarding_engine.go]
        DE[Document Engine<br/>document_engine.go]
        BE[Bulk Operations Engine<br/>bulk_operations_engine.go]
        TE[Timeline Engine<br/>timeline_engine.go]
        VE[Verification Engine<br/>verification_engine.go]

        ER[Employee Repository<br/>employee_repository.go]
        DR[Document Repository<br/>document_repository.go]
        AR[Asset Repository<br/>asset_repository.go]
        DiR[Disciplinary Repository<br/>disciplinary_repository.go]
        OnR[Onboarding Repository<br/>onboarding_repository.go]
        OcR[Org Chart Repository<br/>org_chart.go]

        IMP[Import Engine<br/>import.go]
        NG[Number Generator<br/>number_generator.go]
        FF[Field Filter<br/>field_filter.go]
        EVT[Event Publisher<br/>events.go]
    end

    H --> SVC
    EH --> SVC
    OH --> OE
    SVC --> ER
    SVC --> ANA
    SVC --> OE
    SVC --> DE
    SVC --> BE
    SVC --> TE
    SVC --> VE
    SVC --> IMP
    SVC --> NG
    SVC --> FF
    SVC --> EVT
    OE --> OnR
    DE --> DR
    SVC --> AR
    SVC --> DiR
    SVC --> OcR
```

---

## 5. Component Diagram (Level 3) -- Time & Attendance Service

```mermaid
flowchart TB
    subgraph "Time & Attendance Service"
        HND[HTTP Handlers]
        ENH[Enhanced Handlers]
        RPT_H[Report Handler]

        CS[Clock Service<br/>clock_service.go]
        GS[Geofence Service<br/>geofence_service.go]
        BS[Biometric Service<br/>biometric_service.go]
        RS[Report Service<br/>report_service.go]
        ACO[Auto ClockOut<br/>auto_clockout.go]

        PS[Policy Service<br/>policy_service.go]
        SS[Shift Service<br/>shift_service.go]
        GB[Geofencing + Biometric<br/>geofencing_biometric.go]
        PYS[Payroll Sync<br/>payroll_sync.go]

        DOM[Domain Models<br/>Haversine, AntiSpoofing,<br/>Geofence Validation]

        REPO[Repository<br/>repository.go]
        RRPT[Report Repository<br/>report_repository.go]
    end

    HND --> CS
    HND --> GS
    ENH --> BS
    RPT_H --> RS

    CS --> DOM
    GS --> DOM
    CS --> PS
    CS --> SS
    CS --> GB
    CS --> ACO
    CS --> PYS

    CS --> REPO
    RS --> RRPT
```

---

## 6. Code Diagram (Level 4) -- Nigerian PAYE Calculator

```mermaid
classDiagram
    class PAYEResult {
        +AnnualGross decimal.Decimal
        +CRA decimal.Decimal
        +Pension decimal.Decimal
        +NHF decimal.Decimal
        +OtherDeductions decimal.Decimal
        +TaxableIncome decimal.Decimal
        +AnnualPAYE decimal.Decimal
        +MonthlyPAYE decimal.Decimal
        +BandBreakdown []PAYEBandDetail
    }

    class PAYEBandDetail {
        +BandNumber int
        +BandWidth decimal.Decimal
        +Rate decimal.Decimal
        +TaxableInBand decimal.Decimal
        +TaxInBand decimal.Decimal
    }

    class taxBand {
        +upperLimit decimal.Decimal
        +rate decimal.Decimal
    }

    class NigeriaCalculator {
        +CalculateCRA(annualGross) decimal.Decimal
        +CalculatePAYE(gross, pension, nhf, other) decimal.Decimal
        +CalculatePAYEDetailed(gross, pension, nhf, other) PAYEResult
        +CalculateMinimumTax(annualGross) decimal.Decimal
        -applyTaxBands(taxableIncome) (decimal.Decimal, []PAYEBandDetail)
    }

    PAYEResult --> PAYEBandDetail : contains
    NigeriaCalculator --> PAYEResult : produces
    NigeriaCalculator --> taxBand : uses
```

---

## 7. Deployment Diagram

```mermaid
graph TB
    subgraph "Production Environment"
        subgraph "Kubernetes Cluster"
            subgraph "Ingress"
                ING[NGINX Ingress Controller]
            end

            subgraph "erp-hcm namespace"
                GW1[Gateway Pod 1]
                GW2[Gateway Pod 2]
                GW3[Gateway Pod 3]

                EMP1[Employee Pod 1]
                EMP2[Employee Pod 2]
                PAY1[Payroll Pod 1]
                PAY2[Payroll Pod 2]
                REC1[Recruitment Pod 1]
                PERF1[Performance Pod 1]
                LV1[Leave Pod 1]
                TA1[Attendance Pod 1]
                BEN1[Benefits Pod 1]
                LMS1[Learning Pod 1]
            end
        end

        subgraph "Data Tier"
            PG_P[(PostgreSQL Primary)]
            PG_R[(PostgreSQL Read Replica)]
            RD_M[(Redis Master)]
            RD_S[(Redis Sentinel)]
            NATS1[(NATS Node 1)]
            NATS2[(NATS Node 2)]
            NATS3[(NATS Node 3)]
        end
    end

    ING --> GW1
    ING --> GW2
    ING --> GW3

    GW1 --> EMP1
    GW2 --> PAY1
    GW3 --> REC1

    EMP1 --> PG_P
    EMP2 --> PG_R
    PAY1 --> PG_P
    EMP1 --> RD_M
    EMP1 --> NATS1

    PG_P -->|Streaming Replication| PG_R
    RD_M -->|Replication| RD_S
    NATS1 -->|Raft Consensus| NATS2
    NATS2 -->|Raft Consensus| NATS3
```

---

## 8. Cross-Cutting Concerns

### 8.1 Middleware Pipeline

```mermaid
flowchart LR
    REQ[HTTP Request] --> CORS[CORS]
    CORS --> CSRF[CSRF]
    CSRF --> SEC[Security Headers]
    SEC --> RL[Rate Limiter]
    RL --> JWT[JWT Auth]
    JWT --> TEN[Tenant Context]
    TEN --> AUDIT[Audit Logger]
    AUDIT --> SVC[Service Handler]
    SVC --> RES[HTTP Response]
```

### 8.2 Event Flow Pattern

```mermaid
flowchart LR
    SVC[Service] -->|Publish| NATS[NATS JetStream]
    NATS -->|CloudEvents Envelope| Q1[Consumer 1]
    NATS -->|CloudEvents Envelope| Q2[Consumer 2]
    NATS -->|CloudEvents Envelope| Q3[Consumer 3]

    subgraph "CloudEvents Payload"
        CE["{ specversion: 1.0,<br/>type: erp.hcm.employee.created,<br/>source: /employee-service,<br/>subject: employee-uuid,<br/>data: { ... } }"]
    end
```

### 8.3 Domain-Driven Design Layering

```mermaid
flowchart TB
    subgraph "Each Microservice"
        CMD[cmd/main.go<br/>Entry point]
        HDL[handlers/<br/>HTTP handlers]
        SVC[service/<br/>Business logic]
        ENG[engine/<br/>Calculation engines]
        DOM[domain/<br/>Domain models + types]
        REPO[repository/<br/>Data access layer]
    end

    CMD --> HDL
    HDL --> SVC
    SVC --> ENG
    SVC --> DOM
    SVC --> REPO
    ENG --> DOM
    REPO --> DOM
```
