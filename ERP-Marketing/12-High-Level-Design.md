# ERP-Marketing -- High-Level Design

## 1. System Overview

ERP-Marketing is decomposed into a Rust API gateway, nine Go domain microservices, a React SPA command center, and shared infrastructure services (PostgreSQL, Apache Pulsar, Quickwit). The system supports multi-tenant operation with tenant isolation enforced at every layer.

## 2. System Context Diagram

```mermaid
C4Context
    title ERP-Marketing System Context
    Person(marketer, "Marketing User")
    Person(admin, "Administrator")
    System(marketing, "ERP-Marketing")
    System_Ext(iam, "ERP-IAM")
    System_Ext(crm, "ERP-CRM")
    System_Ext(finance, "ERP-Finance")
    System_Ext(platform, "ERP-Platform")
    System_Ext(email, "Email Provider")
    System_Ext(social, "Social APIs")
    System_Ext(ads, "Ad Networks")
    Rel(marketer, marketing, "Uses")
    Rel(admin, marketing, "Configures")
    Rel(marketing, iam, "AuthN/AuthZ")
    Rel(marketing, crm, "Contact Sync")
    Rel(marketing, finance, "Revenue Attribution")
    Rel(marketing, platform, "Config/Feature Flags")
    Rel(marketing, email, "Sends emails")
    Rel(marketing, social, "Publishes posts")
    Rel(marketing, ads, "Manages ads")
```

## 3. Container Diagram

```mermaid
C4Container
    title ERP-Marketing Containers
    Person(user, "Marketing User")
    Container(spa, "Web SPA", "React + Ant Design + Refine", "Marketing command center")
    Container(api, "API Gateway", "Rust / axum", "REST API, domain logic, AIDD guardrails")
    Container(campaign_svc, "Campaign Service", "Go", "Campaign CRUD + events")
    Container(email_svc, "Email Service", "Go", "Email template + delivery")
    Container(journey_svc, "Journey Service", "Go", "Journey orchestration")
    Container(social_svc, "Social Service", "Go", "Social media management")
    Container(ads_svc, "Ads Service", "Go", "Ad campaign management")
    Container(content_svc, "Content Service", "Go", "CMS blog + landing pages")
    Container(attr_svc, "Attribution Service", "Go", "Multi-touch attribution")
    Container(segment_svc, "Segment Service", "Go", "Dynamic segmentation")
    Container(analytics_svc, "Analytics Service", "Go", "Reporting + dashboards")
    ContainerDb(pg, "PostgreSQL 16", "Database", "25+ tables, JSONB, indexed")
    Container(pulsar, "Apache Pulsar", "Message Broker", "4 topics x 6 partitions")
    Container(quickwit, "Quickwit", "Log Search", "Structured log indexing")
    Rel(user, spa, "HTTPS")
    Rel(spa, api, "REST/JSON")
    Rel(api, campaign_svc, "HTTP")
    Rel(api, email_svc, "HTTP")
    Rel(api, journey_svc, "HTTP")
    Rel(api, social_svc, "HTTP")
    Rel(api, ads_svc, "HTTP")
    Rel(api, content_svc, "HTTP")
    Rel(api, attr_svc, "HTTP")
    Rel(api, segment_svc, "HTTP")
    Rel(api, analytics_svc, "HTTP")
    Rel(api, pg, "sqlx")
    Rel(campaign_svc, pulsar, "Publish events")
    Rel(journey_svc, pulsar, "Publish events")
    Rel(pulsar, quickwit, "Ingest logs")
```

## 4. Component Responsibilities

| Component | Technology | Responsibility | Port |
|---|---|---|---|
| API Gateway | Rust (axum) | HTTP routing, DB access, AIDD evaluation, CORS, tracing | 8086 |
| Campaign Service | Go | Campaign CRUD, lifecycle events | 8080 |
| Email Marketing Service | Go | Template management, send orchestration | 8080 |
| Journey Service | Go | Journey CRUD, step execution, enrollment | 8080 |
| Social Service | Go | Social post CRUD, platform publishing | 8080 |
| Ads Service | Go | Ad CRUD, network sync, spend tracking | 8080 |
| Content Service | Go | CMS CRUD, slug management, SEO | 8080 |
| Attribution Service | Go | Touchpoint aggregation, model calculation | 8080 |
| Segment Service | Go | Segment evaluation, contact matching | 8080 |
| Analytics Service | Go | KPI aggregation, report generation | 8080 |
| PostgreSQL | PostgreSQL 16 | Relational data storage | 5432 |
| Apache Pulsar | Pulsar 3.x | Async event messaging | 6650 |
| Quickwit | Quickwit 0.8+ | Log search and indexing | 7280 |
| Web SPA | React 18 | User interface | 3000 (dev) |

## 5. Data Flow Architecture

```mermaid
flowchart LR
    subgraph "Ingress"
        USER[User Action] --> API[API Gateway]
        WEBHOOK[External Webhook] --> API
        CRON[Cron Scheduler] --> SYNC[Data Sync Service]
    end
    subgraph "Processing"
        API --> CMD[Command Handler]
        CMD --> AIDD{AIDD Guardrail}
        AIDD -->|Approved| DOMAIN[Domain Logic]
        AIDD -->|Blocked| REJECT[Reject + Log]
        DOMAIN --> DB[(PostgreSQL)]
        DOMAIN --> EVT[Domain Event]
    end
    subgraph "Eventing"
        EVT --> PULSAR[Pulsar Topic]
        PULSAR --> CONSUMER[Event Consumer]
        CONSUMER --> SIDE_EFFECT[Side Effect<br/>Email, SMS, Social API]
        PULSAR --> AUDIT[Audit Topic]
        AUDIT --> QW[Quickwit Index]
    end
    subgraph "Egress"
        DB --> QUERY[Query Handler]
        QUERY --> RESPONSE[API Response]
        QW --> SEARCH[Observability Search]
    end
```

## 6. Scalability Model

```mermaid
flowchart TB
    subgraph "Horizontal Scaling"
        API_1[API Pod 1]
        API_2[API Pod 2]
        API_N[API Pod N]
    end
    subgraph "Service Scaling"
        CS_1[Campaign Svc 1]
        CS_2[Campaign Svc 2]
        JS_1[Journey Svc 1]
        JS_2[Journey Svc 2]
    end
    subgraph "Data Scaling"
        PG_PRIMARY[PG Primary]
        PG_REPLICA[PG Read Replica]
        PULSAR_PART[Pulsar<br/>6 Partitions/Topic]
    end
    LB[Load Balancer] --> API_1 & API_2 & API_N
    API_1 & API_2 & API_N --> CS_1 & CS_2 & JS_1 & JS_2
    CS_1 & CS_2 --> PG_PRIMARY
    JS_1 & JS_2 --> PG_PRIMARY
    API_1 & API_2 & API_N --> PG_REPLICA
```

## 7. Security Boundaries

```mermaid
flowchart TB
    subgraph "DMZ"
        LB[Load Balancer<br/>TLS Termination]
    end
    subgraph "Application Zone"
        API[API Gateway<br/>AuthN/AuthZ Check]
        SVC[Domain Services<br/>Tenant Validation]
    end
    subgraph "Data Zone"
        DB[(PostgreSQL<br/>Encrypted at Rest)]
        MQ[Pulsar<br/>Topic ACLs]
    end
    subgraph "Monitoring Zone"
        QW[Quickwit<br/>Read-Only Access]
    end
    LB --> API
    API --> SVC
    SVC --> DB & MQ
    MQ --> QW
```

## 8. Technology Decisions Summary

| Decision | Choice | Alternative Considered | Rationale |
|---|---|---|---|
| API Language | Rust | Go, Node.js | Memory safety, zero-cost abstractions, sub-ms latency |
| Service Language | Go | Rust, Java | Fast compilation, simple deployment, team velocity |
| Frontend | React + Ant Design + Refine | Vue + Vuetify, Angular | Enterprise component library, data-provider abstraction |
| Database | PostgreSQL 16 | MySQL, MongoDB | JSONB support, ACID, mature ecosystem |
| Event Broker | Apache Pulsar | Kafka, RabbitMQ | Multi-tenant, persistent, exactly-once semantics |
| Log Search | Quickwit | Elasticsearch, Loki | Sub-second search, Rust-native, lower resource usage |
| Storage | Mayastor/Vitastor | Longhorn, Rook-Ceph | Low-latency replicated block storage for HCI |
