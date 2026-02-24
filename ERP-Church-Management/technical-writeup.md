# Technical Write-up -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Technical Overview

ERP-Church-Management is architected as a multi-tenant microservices system transitioning from a Node.js/Express monolith to 12 Go-based domain services. The system manages church operations including member care, visitor assimilation, giving, discipleship, and multi-channel communication. This document details the implementation approach, technical decisions, and migration strategy.

---

## 2. Migration Architecture: Monolith to Microservices

### 2.1 Source Monolith Stack

The original application consists of:
- **Runtime**: Node.js with Express.js framework
- **Database**: PostgreSQL with Sequelize ORM (79 models)
- **Real-time**: Socket.IO for live notifications
- **Communication**: Twilio (SMS), WhatsApp Business API, Telegram Bot API, Facebook Messenger API, Nodemailer (email)
- **Jobs**: node-cron for scheduled tasks (follow-up reminders, absentee checker, KPI calculator)
- **Validation**: express-validator
- **Security**: JWT (jsonwebtoken), bcryptjs, helmet
- **Logging**: Winston

### 2.2 Target Microservices Stack

```mermaid
flowchart LR
    subgraph Target["Target Architecture"]
        GW["Go Gateway\n(net/http, httputil)"]
        SVC["12 Go Services\n(stdlib + pgx + kafka-go)"]
        PG["PostgreSQL 16\n(17 core tables)"]
        RD["Redis 7"]
        RP["Redpanda / Kafka"]
    end
```

### 2.3 Strangler Fig Pattern

The migration follows the Strangler Fig pattern, gradually routing traffic from the monolith to new microservices:

```mermaid
flowchart TD
    LB["Load Balancer"] --> GW["Go Gateway"]
    GW --> ROUTE{"Service available?"}
    ROUTE -->|Yes| SVC["New Microservice"]
    ROUTE -->|No| MONO["Legacy Monolith"]
    SVC --> PG["Shared PostgreSQL"]
    MONO --> PG
```

Phase 1: Gateway + service skeletons (CURRENT)
Phase 2: Business logic port (member, visitor, followup, giving)
Phase 3: Business logic port (event, group, discipleship, welfare)
Phase 4: Business logic port (communication, kpi, volunteer, facility)
Phase 5: Decommission monolith

---

## 3. Gateway Implementation Details

The Go API gateway (`gateway/main.go`) implements five middleware layers chained via `http.Handler` wrapping:

```go
var h http.Handler = buildMux(doc)
h = withCorrelationID(h)
h = requireTenant(h)
h = requireJWT(h)
h = requireEntitlement("erp.church_management", h)
```

**Execution order** (outermost first):
1. `requireEntitlement` -- Calls ERP-Platform to verify tenant has `erp.church_management` or `erp.full_suite` entitlement
2. `requireJWT` -- Validates Bearer token presence (minimum 16 characters)
3. `requireTenant` -- Enforces `X-Tenant-ID` header on business routes
4. `withCorrelationID` -- Generates or forwards `X-Correlation-ID`
5. `buildMux` -- Routes to `/healthz`, `/v1/capabilities`, or reverse proxy

### 3.1 Graceful Degradation

The entitlement check includes a `ALLOW_ON_ENTITLEMENT_FAILURE` flag (default: `true`). When ERP-Platform is unavailable, the gateway allows requests through. This supports standalone operation without the broader ERP suite.

---

## 4. Database Architecture

### 4.1 Multi-Tenancy Implementation

Every table includes a `tenant_id` column. All queries include `WHERE tenant_id = ?` to enforce data isolation. The gateway extracts `tenant_id` from the JWT claims or `X-Tenant-ID` header and forwards it to services.

```sql
-- Example: Member query with tenant isolation
SELECT * FROM members
WHERE tenant_id = $1 AND id = $2;

-- Index ensures tenant queries are efficient
CREATE INDEX idx_members_tenant ON members (tenant_id);
```

### 4.2 Model Count: 79 to 17

The source monolith has 79 Sequelize models which are consolidated into 17 core tables:

| Target Table | Sequelize Models Consolidated |
|---|---|
| tenants | Tenant, Campus |
| users | User |
| members | Member, MemberEngagementScore, SpiritualJourney, SpiritualMilestone, Household |
| visitors | Visitor |
| followups | FollowUpActivity, AccountOfficerAssignment, Directorate |
| donations | Donation, GivingStatement, BlockchainDonation |
| pledges | Pledge, PledgeCampaign |
| events | Event, Sermon, SermonSeries |
| attendance | Attendance, ChildCheckin |
| groups | SmallGroup, Ministry |
| discipleship_programs | DiscipleshipPath, NewBelieverClass |
| discipleship_progress | DiscipleshipEnrollment, Mentorship |
| welfare_cases | WelfareCase, CareRequest, CounselingCase, CareActivity, CareTeam |
| communications | Communication, DirectMessage, Conversation, Comment |
| kpis | KPI, AnalyticsDashboard, AnalyticsReport, AIInsight |
| volunteers | Volunteer, VolunteerRole, VolunteerShift, VolunteerTraining, VolunteerTrainingRecord |
| facilities | Facility, FacilityBooking, Asset, AssetCheckout, MaintenanceRequest, Vehicle, VehicleBooking |

JSONB columns absorb many auxiliary models (e.g., milestones, training records, booking rules).

---

## 5. Real-time Communication

### 5.1 Legacy (Socket.IO)

The monolith uses Socket.IO for:
- `new-visitor` -- Real-time notification when visitor registers
- `checkin-update` -- Child check-in/out events
- `paging-alert` -- Parent paging during children's ministry
- `new-care-request` -- Care team notifications
- `new-message` -- Direct messaging

### 5.2 Target (SSE + Kafka Bridge)

```mermaid
flowchart LR
    SVC["Microservice"] -->|Publish| RP["Redpanda"]
    RP -->|Consume| BRIDGE["SSE Bridge Service"]
    BRIDGE -->|Server-Sent Events| CLIENT["Web/Mobile Client"]
```

The SSE bridge subscribes to relevant Kafka topics and fans out to connected clients, filtered by tenant_id and user role.

---

## 6. Background Job Architecture

### 6.1 Cron Jobs (Source Monolith)

| Job | Schedule | Purpose |
|---|---|---|
| followUpReminder | Every hour (`0 * * * *`) | Check for visitors needing 72-hour follow-up |
| absenteeChecker | Daily at midnight | Detect members absent > 3 weeks |
| kpiCalculator | Daily at midnight (`0 0 * * *`) | Calculate and store KPI snapshots |

### 6.2 Target: Kafka Consumer Workers

In the microservices architecture, cron jobs are replaced by:
- **Kafka consumers** that react to domain events
- **Kubernetes CronJobs** for scheduled calculations

```mermaid
flowchart TD
    subgraph Cron_Jobs["Kubernetes CronJobs"]
        CJ1["followup-reminder\n(every hour)"]
        CJ2["absentee-checker\n(daily midnight)"]
        CJ3["kpi-calculator\n(daily midnight)"]
    end

    subgraph Event_Consumers["Kafka Consumers"]
        EC1["followup-consumer\n(visitor.created)"]
        EC2["communication-consumer\n(followup.assigned)"]
        EC3["kpi-consumer\n(giving.recorded, event.checkin)"]
    end

    CJ1 --> FU["followup-service API"]
    CJ2 --> MS["member-service API"]
    CJ3 --> KPI["kpi-service API"]

    EC1 --> FU
    EC2 --> CS["communication-service"]
    EC3 --> KPI
```

---

## 7. Communication Integration Layer

### 7.1 Unified Messaging Service

The source monolith implements a unified messaging service that abstracts 5 communication channels:

```mermaid
flowchart TD
    UMS["Unified Messaging Service"] --> PREF["Check recipient preferences"]
    PREF --> PRI["Channel priority queue"]
    PRI --> A1["Adapter: Twilio SMS"]
    PRI --> A2["Adapter: WhatsApp Business API"]
    PRI --> A3["Adapter: Telegram Bot API"]
    PRI --> A4["Adapter: Facebook Messenger API"]
    PRI --> A5["Adapter: Nodemailer Email"]
    A1 & A2 & A3 & A4 & A5 --> RESULT["Delivery result"]
    RESULT --> FALLBACK{"Delivered?"}
    FALLBACK -->|No| NEXT["Try next channel"]
    FALLBACK -->|Yes| LOG["Log success"]
```

### 7.2 Message Template System

```json
{
  "template_id": "welcome_visitor",
  "channels": {
    "sms": "Welcome to {church_name}! Your Account Officer {officer_name} will reach out soon. God bless you!",
    "whatsapp": "Welcome to {church_name}! ...",
    "email": {
      "subject": "Welcome to {church_name}",
      "body": "Dear {visitor_name}, ..."
    }
  }
}
```

---

## 8. Performance Characteristics

### 8.1 Observed Monolith Performance

| Metric | Value |
|---|---|
| Cold start time | ~3 seconds |
| Avg response time (CRUD) | 50-150ms |
| Avg response time (search) | 100-300ms |
| Avg response time (KPI query) | 200-500ms |
| Max concurrent connections (tested) | 200 |
| Database query count per request | 1-5 (with Sequelize eager loading) |

### 8.2 Target Microservices Performance

| Metric | Target |
|---|---|
| Cold start time (Go binary) | < 500ms |
| Avg response time (CRUD) | < 50ms |
| Avg response time (search) | < 100ms |
| Avg response time (KPI query) | < 150ms (with Redis cache) |
| Max concurrent connections | 2000+ (per service) |
| Database connections per service | 10-25 (connection pool) |

---

## 9. Security Implementation

### 9.1 Authentication Chain

```
Client -> TLS 1.3 -> Gateway -> JWT Validation -> Entitlement Check -> Tenant Isolation -> Service
```

### 9.2 Password Hashing

The monolith uses bcryptjs with default cost factor. The target uses bcrypt with cost factor 12:
```
$2b$12$[salt][hash]
```

### 9.3 API Security Headers

- `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `Content-Security-Policy: default-src 'self'`
- `X-XSS-Protection: 1; mode=block`

Helmet middleware in the monolith sets these headers. The Go gateway will set equivalent headers.

---

## 10. Docker Compose Environment

The docker-compose.yml defines 15 containers:
- 1 gateway (port 8093:8090)
- 12 microservices (internal port 8080 each)
- 1 PostgreSQL 16 (port 5435:5432)
- 1 Redis 7 (port 6381:6379)
- 1 Redpanda (port 19094:9092)

All services share environment variables:
- `DATABASE_URL=postgres://erp:erp@postgres:5432/erp_church_management`
- `REDIS_ADDR=redis:6379`
- `KAFKA_BROKERS=redpanda:9092`
