# ERP-School-Management -- High-Level Design

**Product:** EduCore Pro
**Version:** 1.0.0
**Date:** 2026-02-23

---

## 1. System Context (C4 Level 1)

```mermaid
graph TB
    subgraph "External Systems"
        IAM["ERP-IAM<br/>Identity Provider"]
        PLAT["ERP-Platform<br/>Subscription Hub"]
        PAY["Payment Gateways<br/>Stripe / Paystack / Flutterwave"]
        SMS["SMS Providers<br/>Twilio / AfricasTalking"]
        EMAIL["Email Services<br/>SendGrid / SES"]
        OAUTH["OAuth Providers<br/>Google / Microsoft"]
        BC_NET["Blockchain Network<br/>Ethereum / IPFS"]
    end

    subgraph "EduCore Pro"
        EP["EduCore Pro<br/>School Management Platform"]
    end

    subgraph "Users"
        STU["Students"]
        PAR["Parents"]
        TEA["Teachers"]
        ADM["School Admins"]
        SAD["Super Admins"]
        DRV["Bus Drivers"]
    end

    STU & PAR & TEA & ADM & SAD & DRV -->|Use| EP
    EP -->|Authenticate| IAM
    EP -->|Check entitlements| PLAT
    EP -->|Process payments| PAY
    EP -->|Send SMS| SMS
    EP -->|Send email| EMAIL
    EP -->|Social login| OAUTH
    EP -->|Issue certificates| BC_NET
```

---

## 2. Container Diagram (C4 Level 2)

```mermaid
graph TB
    subgraph "Frontend Containers"
        WEB["Web App<br/>Next.js 14<br/>Port: 3000"]
        MOB["Mobile App<br/>Flutter"]
        PAPP["Parent App<br/>Flutter"]
        TAPP["Teacher App<br/>Flutter"]
        BAPP["Bus Tracker<br/>Flutter"]
    end

    subgraph "API Layer"
        GW["ERP Gateway<br/>NestJS<br/>Port: 8090"]
    end

    subgraph "Core Domain Services"
        AUTH["auth-service<br/>NestJS<br/>Port: 8080"]
        STU["student-service<br/>NestJS<br/>Port: 8080"]
        ACA["academic-service<br/>NestJS<br/>Port: 8080"]
        FIN["finance-service<br/>NestJS<br/>Port: 8080"]
        LMS["lms-service<br/>NestJS<br/>Port: 8080"]
        ADM["admin-service<br/>NestJS<br/>Port: 8080"]
    end

    subgraph "Support Services"
        COMM["communication-service<br/>NestJS"]
        NOTIF["notification-service<br/>NestJS"]
        FILE["file-service<br/>NestJS"]
        SEARCH["search-service<br/>NestJS"]
        INTEG["integration-service<br/>NestJS"]
        EVT["event-service<br/>NestJS"]
        MIG["migration-service<br/>NestJS"]
        SUB["subscription-service<br/>NestJS"]
    end

    subgraph "Innovation Services"
        AI["ai-service<br/>Python"]
        BC["blockchain-service<br/>NestJS"]
        GAME["gamification-service<br/>NestJS"]
        IOT["iot-service<br/>NestJS"]
        AIOPS["aiops-service<br/>NestJS"]
        ANA["analytics-service<br/>NestJS"]
        PLACE["placement-service<br/>Rust"]
        SCHOL["scholarship-service<br/>Go"]
        RES["research-service<br/>Rust"]
    end

    subgraph "Data Stores"
        PG["LumaDB<br/>PostgreSQL 16<br/>Port: 5432"]
        RP["Redpanda<br/>Kafka-compatible<br/>Port: 9092"]
    end

    subgraph "Observability Stack"
        OTEL["OTel Collector<br/>Port: 4317/4318"]
        GF["Grafana<br/>Port: 3000"]
        SS["Apache Superset<br/>Port: 8088"]
    end

    WEB & MOB & PAPP & TAPP & BAPP --> GW
    GW --> AUTH & STU & ACA & FIN & LMS & ADM
    GW --> COMM & NOTIF & FILE & SEARCH & INTEG & EVT & MIG & SUB
    GW --> AI & BC & GAME & IOT & AIOPS & ANA & PLACE & SCHOL & RES

    AUTH & STU & ACA & FIN & LMS & ADM --> PG
    COMM & NOTIF & FILE & SEARCH & ANA --> PG
    BC & GAME & IOT --> PG

    AUTH & STU & ACA & FIN --> RP
    RP --> OTEL --> GF
    PG --> SS
```

---

## 3. Domain Model

```mermaid
classDiagram
    class School {
        +UUID id
        +String name
        +String code
        +String type
        +String currency_code
        +String timezone
        +Boolean is_active
    }

    class User {
        +UUID id
        +String email
        +UserRole role
        +Boolean mfa_enabled
    }

    class Student {
        +UUID id
        +String student_number
        +String first_name
        +String last_name
        +Date date_of_birth
        +EnrollmentStatus status
    }

    class Guardian {
        +UUID id
        +String first_name
        +String last_name
        +String phone
        +GuardianRelationship relationship
    }

    class AcademicYear {
        +UUID id
        +String name
        +Date start_date
        +Date end_date
        +Boolean is_current
    }

    class Term {
        +UUID id
        +String name
        +TermType type
        +Int term_number
    }

    class Class {
        +UUID id
        +String name
        +String code
        +Int max_capacity
    }

    class Subject {
        +UUID id
        +String name
        +String code
        +Boolean is_core
    }

    class Assessment {
        +UUID id
        +String title
        +AssessmentType type
        +Decimal max_score
        +Decimal weight
    }

    class FeeStructure {
        +UUID id
        +FeeType fee_type
        +Decimal amount
        +String currency_code
    }

    class Invoice {
        +UUID id
        +String invoice_number
        +Decimal total_amount
        +InvoiceStatus status
    }

    School "1" --> "*" User
    School "1" --> "*" Student
    School "1" --> "*" AcademicYear
    AcademicYear "1" --> "*" Term
    School "1" --> "*" Class
    School "1" --> "*" Subject
    Subject "1" --> "*" Assessment
    Student "1" --> "*" Guardian
    School "1" --> "*" FeeStructure
    Student "1" --> "*" Invoice
```

---

## 4. High-Level Data Flow

```mermaid
flowchart LR
    subgraph "Data Ingestion"
        A["Client Apps"]
        B["IoT Sensors"]
        C["Third-party APIs"]
    end

    subgraph "Processing"
        D["API Gateway"]
        E["Microservices"]
        F["Event Processors"]
    end

    subgraph "Storage"
        G["LumaDB<br/>(PostgreSQL)"]
        H["Redpanda<br/>(Events)"]
        I["IPFS<br/>(Documents)"]
    end

    subgraph "Output"
        J["Dashboards"]
        K["Reports"]
        L["Notifications"]
        M["Certificates"]
    end

    A --> D --> E --> G
    E --> H --> F
    B --> E
    C --> E
    F --> G
    G --> J & K
    E --> L
    E --> I --> M
```

---

## 5. Service Interaction Matrix

| Source Service | Target Service | Protocol | Purpose |
|---|---|---|---|
| auth-service | student-service | REST | User-student linking |
| student-service | finance-service | Event | Fee generation on enrollment |
| academic-service | notification-service | Event | Grade publication alerts |
| finance-service | communication-service | Event | Payment confirmations |
| lms-service | gamification-service | Event | Progress milestones |
| academic-service | blockchain-service | Event | Certificate issuance |
| ai-service | analytics-service | REST | Prediction data |
| iot-service | notification-service | Event | Environmental alerts |
| admin-service | all services | REST | Configuration propagation |
| search-service | student/academic/lms | REST | Index synchronization |

---

## 6. Deployment Topology

```mermaid
graph TB
    subgraph "Region: Africa (Primary)"
        subgraph "Availability Zone 1"
            K8S_1["K8s Node Pool 1<br/>Core Services"]
            PG_PRIMARY["PostgreSQL Primary"]
        end
        subgraph "Availability Zone 2"
            K8S_2["K8s Node Pool 2<br/>Support Services"]
            PG_REPLICA["PostgreSQL Replica"]
        end
    end

    subgraph "Region: Europe (GDPR)"
        K8S_EU["K8s Node Pool<br/>EU Data Partition"]
        PG_EU["PostgreSQL<br/>EU Partition"]
    end

    subgraph "Region: Americas"
        K8S_US["K8s Node Pool<br/>US Data Partition"]
        PG_US["PostgreSQL<br/>US Partition"]
    end

    subgraph "CDN"
        CF["CloudFront / Cloudflare"]
    end

    CF --> K8S_1 & K8S_EU & K8S_US
    PG_PRIMARY --> PG_REPLICA
```

---

## 7. Technology Choices Rationale

| Decision | Choice | Rationale |
|---|---|---|
| Primary Language | TypeScript (NestJS) | Team expertise, type safety, ecosystem |
| High-Performance Services | Rust | Memory safety, zero-cost abstractions for career/research |
| Infrastructure Services | Go | Concurrency, small binaries for scholarship processing |
| AI/ML | Python | ML library ecosystem, model deployment ease |
| Database | PostgreSQL 16 | JSONB, extensions, reliability, cost |
| Event Streaming | Redpanda | Kafka-compatible, simpler operations, lower resource usage |
| Frontend Web | Next.js 14 | SSR, RSC, performance, SEO |
| Frontend Mobile | Flutter | Cross-platform, single codebase, performance |
| ORM | Prisma | Type-safe, migration management, introspection |
| Monorepo | Turborepo | Fast builds, dependency graph, remote caching |
| Observability | OpenTelemetry + Grafana | Vendor-neutral, unified traces/metrics/logs |
| BI | Apache Superset | Open source, SQL-native, self-hosted |
