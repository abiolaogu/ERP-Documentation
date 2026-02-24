# Enterprise Architecture -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Enterprise Context

ERP-Church-Management is one vertical module within the BillyRonks ERP ecosystem. It serves the Faith/Nonprofit sector, operating alongside horizontal modules (Finance, HCM, BI, CRM) and other vertical modules (Healthcare, School Management, Commerce, BSS-OSS).

---

## 2. Enterprise Integration Map

```mermaid
flowchart TB
    subgraph ERP_Platform_Layer["Platform Layer"]
        IAM["ERP-IAM\n(Identity & Access)"]
        PLT["ERP-Platform\n(Entitlements & Config)"]
        iPaaS["ERP-iPaaS\n(Integration Hub)"]
        AI["ERP-AI\n(ML/AI Services)"]
    end

    subgraph Horizontal_Modules["Horizontal Modules"]
        FIN["ERP-Finance"]
        HCM["ERP-HCM"]
        BII["ERP-BI"]
        CRM["ERP-CRM"]
        MKT["ERP-Marketing"]
        WS["ERP-Workspace"]
    end

    subgraph Vertical_Modules["Vertical Modules"]
        CHM["ERP-Church-Management"]
        SCH["ERP-School-Management"]
        HC["ERP-Healthcare"]
        COM["ERP-Commerce"]
    end

    CHM --> IAM
    CHM --> PLT
    CHM --> FIN
    CHM --> BII
    CHM --> MKT
    CHM --> iPaaS
    CHM --> AI

    FIN --> IAM
    HCM --> IAM
    BII --> IAM
    CRM --> IAM
```

---

## 3. Integration Patterns

### 3.1 Authentication Flow (ERP-IAM)

```mermaid
sequenceDiagram
    participant C as Client
    participant GW as Church Gateway
    participant IAM as ERP-IAM
    participant SVC as Microservice

    C->>IAM: POST /auth/token (credentials)
    IAM-->>C: JWT access token + refresh token
    C->>GW: GET /v1/member/123 (Bearer JWT)
    GW->>IAM: Validate JWT (JWKS)
    IAM-->>GW: Token valid, claims: {sub, role, tenant_id}
    GW->>SVC: Forward request + X-Tenant-ID
    SVC-->>GW: Response
    GW-->>C: Response
```

### 3.2 Entitlement Check (ERP-Platform)

```mermaid
sequenceDiagram
    participant GW as Church Gateway
    participant PLT as ERP-Platform

    GW->>PLT: GET /v1/entitlements/{tenant_id}
    PLT-->>GW: {"modules": ["erp.church_management", "erp.finance"]}
    Note over GW: Check if "erp.church_management" or "erp.full_suite" present
    GW->>GW: Allow or deny request
```

The gateway implements a graceful degradation pattern: if ERP-Platform is unreachable and `ALLOW_ON_ENTITLEMENT_FAILURE=true`, requests proceed. This ensures standalone operation.

### 3.3 Financial Integration (ERP-Finance)

```mermaid
sequenceDiagram
    participant GS as Giving Service
    participant RP as Redpanda
    participant FIN as ERP-Finance

    GS->>RP: Publish giving.recorded event
    RP->>FIN: Consume giving.recorded
    FIN->>FIN: Create journal entry
    FIN->>FIN: Update ledger
    Note over FIN: Tithe -> Revenue account<br/>Offering -> Revenue account<br/>Welfare -> Restricted fund
```

### 3.4 Analytics Integration (ERP-BI)

```mermaid
flowchart LR
    KS["kpi-service"] -->|KPI snapshots| RP["Redpanda"]
    RP -->|kpi.calculated events| BII["ERP-BI"]
    BII --> DW["Data Warehouse"]
    DW --> DASH["BI Dashboards"]
```

---

## 4. Data Flow Architecture

### 4.1 Member Lifecycle Data Flow

```mermaid
flowchart TD
    V["Visitor Registration"] -->|visitor.created| FS["Follow-up Service"]
    FS -->|Account Officer assigned| AO["Account Officer Workspace"]
    AO -->|72-hour contact| CS["Communication Service"]
    CS -->|Multi-channel outreach| CH["SMS/WhatsApp/Email/Telegram"]
    V -->|Visitor converts| MS["Member Service"]
    MS -->|member.created| DS["Discipleship Service"]
    DS -->|NBC enrollment| NBC["New Believer Class"]
    NBC -->|Completed| MEN["Mentorship (90-120 days)"]
    MEN -->|Completed| GRS["Group Service"]
    GRS -->|Small group placement| HF["Home Fellowship"]
    MS -->|member.created| GS["Giving Service"]
    GS -->|First tithe/offering| FIN["ERP-Finance"]
```

---

## 5. Capability Map

```mermaid
mindmap
  root((ERP-Church-Management))
    Member Care
      Member CRUD
      Family Units
      Natural Groups
      Communication Preferences
      Absentee Detection
    Visitor Assimilation
      Registration
      72-Hour Protocol
      4Cs Workflow
      Welcome Gifts
      Conversion Pipeline
    Follow-up Ministry
      Account Officers
      6 Directorates
      Activity Tracking
      Escalation Rules
    Financial Stewardship
      Tithes & Offerings
      Pledges & Campaigns
      Tax Receipts
      Giving Statements
    Worship & Events
      Event Management
      Attendance Tracking
      QR/NFC Check-in
      Service Planning
    Community Groups
      Small Groups
      Home Fellowships
      Cell Groups
      Ministry Teams
    Spiritual Growth
      New Believer Class
      Mentorship
      Sunday School
      Progress Tracking
    Pastoral Care
      Welfare Cases
      Benevolence Fund
      Counseling
      Needs Tracking
    Communication
      SMS
      WhatsApp
      Telegram
      Facebook
      Email
      Push
      In-app
    Analytics
      KPI Dashboard
      Directorate Metrics
      Growth Trends
      Engagement Scores
    Operations
      Volunteer Management
      Facility Booking
      Equipment Tracking
      Campus Management
```

---

## 6. TOGAF Architecture Layers

### 6.1 Business Architecture

| Business Function | Process Owner | Supporting Service |
|---|---|---|
| Soul Winning | Senior Pastor | visitor-service, followup-service |
| Discipleship | Discipleship Director | discipleship-service |
| Worship | Worship Director | event-service |
| Finance | Finance Director | giving-service |
| Welfare | Welfare Director | welfare-service |
| Communication | Media Director | communication-service |
| Facilities | Facilities Manager | facility-service |
| Volunteer Coordination | Volunteer Coordinator | volunteer-service |

### 6.2 Application Architecture

| Application Component | Technology | Deployment |
|---|---|---|
| Web Application | React/Next.js | Vercel / Kubernetes |
| Mobile Application | Flutter | App Store / Play Store |
| API Gateway | Go | Kubernetes Pod |
| 12 Microservices | Go | Kubernetes Pods |
| Background Jobs | Kafka Consumers | Kubernetes Jobs |

### 6.3 Technology Architecture

| Layer | Component | Technology |
|---|---|---|
| Presentation | Web, Mobile, Kiosk | React, Flutter, React (kiosk mode) |
| API | Gateway, REST | Go net/http, httputil |
| Application | Business Logic | Go services |
| Data | Relational | PostgreSQL 16 |
| Data | Cache | Redis 7 |
| Data | Event Stream | Redpanda/Kafka |
| Infrastructure | Containers | Docker + Kubernetes |
| Infrastructure | CI/CD | GitHub Actions |
| Infrastructure | Monitoring | Prometheus + Grafana |

### 6.4 Information Architecture

```mermaid
flowchart TD
    subgraph Master_Data["Master Data"]
        MD_M["Members"]
        MD_T["Tenants/Campuses"]
        MD_U["Users/Roles"]
    end

    subgraph Transactional_Data["Transactional Data"]
        TD_V["Visitor Records"]
        TD_F["Follow-up Activities"]
        TD_G["Giving Transactions"]
        TD_A["Attendance Records"]
        TD_W["Welfare Cases"]
    end

    subgraph Reference_Data["Reference Data"]
        RD_D["Directorate Definitions"]
        RD_NG["Natural Group Categories"]
        RD_GT["Giving Types"]
        RD_ET["Event Types"]
    end

    subgraph Analytical_Data["Analytical Data"]
        AD_K["KPI Snapshots"]
        AD_E["Engagement Scores"]
        AD_T["Trend Data"]
    end

    Master_Data --> Transactional_Data
    Reference_Data --> Transactional_Data
    Transactional_Data --> Analytical_Data
```

---

## 7. Governance Model

### 7.1 Data Governance

| Principle | Implementation |
|---|---|
| Data Ownership | Each service owns its domain data |
| Data Privacy | GDPR-compliant erasure via choreographed saga |
| Data Quality | Validation at API layer (express-validator legacy, Go validators target) |
| Data Retention | Configurable per data class (giving: 7 years, communications: 1 year) |

### 7.2 Service Governance

| Principle | Implementation |
|---|---|
| API Contract | OpenAPI 3.0 specification per service |
| Versioning | URL-based (/v1/, /v2/) |
| SLA | 99.9% uptime, p95 < 200ms |
| Ownership | Each service has a designated team owner |

---

## 8. Disaster Recovery

| Component | RPO | RTO | Strategy |
|---|---|---|---|
| PostgreSQL | 1 hour | 4 hours | WAL archiving + daily snapshots |
| Redis | 24 hours | 1 hour | Rebuild from database |
| Redpanda | 0 (replicated) | 30 minutes | Multi-AZ replication |
| Application | 0 | 15 minutes | Kubernetes rolling restart |
| Frontend | 0 | 5 minutes | CDN + fallback origin |
