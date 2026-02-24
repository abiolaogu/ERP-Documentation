# ERP-Workspace Software Architecture

> **Document ID:** ERP-WS-SA-004
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved
> **Related Documents:** [01-Technical-Writeup.md](./01-Technical-Writeup.md), [12-High-Level-Design.md](./12-High-Level-Design.md)

---

## 1. Architecture Overview

ERP-Workspace follows a microservices architecture with seven core services, supported by external infrastructure components for video conferencing (LiveKit), document editing (ONLYOFFICE), file storage (Nextcloud/MinIO), and analytics (ClickHouse). The architecture employs a polyglot approach: Rust for the high-throughput mail server, Go for API services, and Python for AI-powered features.

---

## 2. C4 Model

### 2.1 System Context (Level 1)

```mermaid
C4Context
    title ERP-Workspace - System Context Diagram

    Person(worker, "Knowledge Worker", "Sends email, attends meetings, edits documents")
    Person(admin, "IT Administrator", "Manages workspace policies and users")
    Person(guest, "Guest", "External collaborator with limited access")

    System(workspace, "ERP-Workspace", "Unified Communication & Collaboration Suite")

    System_Ext(iam, "ERP-IAM", "SSO, RBAC, OIDC Provider")
    System_Ext(platform, "ERP-Platform", "Subscription & Entitlement Engine")
    System_Ext(ai_engine, "ERP-AI", "LLM/ML Inference Service")
    System_Ext(ipaas, "ERP-iPaaS", "External System Connectors")
    System_Ext(assistant, "ERP-Assistant", "AI Copilot Interface")

    System_Ext(ext_email, "External MTA", "Internet Email Servers")
    System_Ext(ext_caldav, "CalDAV Clients", "Thunderbird, Apple Calendar")
    System_Ext(ext_webrtc, "WebRTC Clients", "Browser, Mobile App")

    Rel(worker, workspace, "Uses daily", "HTTPS/WSS/WebRTC")
    Rel(admin, workspace, "Configures", "HTTPS")
    Rel(guest, workspace, "Limited access", "HTTPS/WSS")
    Rel(workspace, iam, "Auth", "OIDC/JWT")
    Rel(workspace, platform, "License check", "gRPC")
    Rel(workspace, ai_engine, "Inference", "gRPC")
    Rel(workspace, ipaas, "Sync", "HTTPS/Webhooks")
    Rel(workspace, assistant, "Copilot", "gRPC")
    Rel(workspace, ext_email, "Send/Receive", "SMTP/TLS")
    Rel(ext_caldav, workspace, "Sync", "CalDAV")
    Rel(ext_webrtc, workspace, "Join", "WebRTC/DTLS")
```

### 2.2 Container Diagram (Level 2)

```mermaid
C4Container
    title ERP-Workspace - Container Diagram

    Person(user, "User")

    System_Boundary(workspace, "ERP-Workspace") {
        Container(web, "Web App", "React/TypeScript", "SPA for all workspace features")
        Container(mobile, "Mobile App", "Flutter", "iOS/Android native app")
        Container(desktop, "Desktop App", "Electron", "Windows/macOS/Linux")

        Container(gateway, "API Gateway", "Go", "Authentication, routing, rate limiting")

        Container(email_svc, "email-service", "Go", "Email CRUD, JMAP proxy")
        Container(cal_svc, "calendar-service", "Go", "Calendar CRUD, scheduling")
        Container(meet_svc, "meet-service", "Go", "Meeting lifecycle, recording")
        Container(chat_svc, "chat-service", "Go", "Channels, messages, reactions")
        Container(docs_svc, "docs-service", "Go", "Document management, WOPI host")
        Container(drive_svc, "drive-service", "Go", "File management, sharing")
        Container(contacts_svc, "contacts-service", "Go", "Contact directory")

        Container(smtp_server, "SMTP/JMAP Server", "Rust", "100K msg/sec mail engine")
        Container(ai_svc, "AI Features", "Python/FastAPI", "Smart compose, triage, summaries")

        ContainerDb(pg, "PostgreSQL", "Database", "85+ tables, 11 DDD contexts")
        ContainerDb(redis, "Redis", "Cache", "Sessions, rate limits, queues")
        ContainerDb(minio, "MinIO", "Object Store", "File blobs, attachments")
        ContainerDb(clickhouse, "ClickHouse", "OLAP", "Email analytics, audit logs")

        Container(kafka, "Redpanda", "Event Bus", "CloudEvents streaming")
        Container(quickwit, "Quickwit", "Search", "Full-text search engine")
    }

    System_Ext(livekit, "LiveKit SFU", "Video/Audio routing")
    System_Ext(onlyoffice, "ONLYOFFICE DS", "Document editing engine")
    System_Ext(nextcloud, "Nextcloud", "File sync platform")

    Rel(user, web, "HTTPS")
    Rel(user, mobile, "HTTPS")
    Rel(user, desktop, "HTTPS")
    Rel(web, gateway, "REST/WSS")
    Rel(mobile, gateway, "REST/WSS")
    Rel(desktop, gateway, "REST/WSS")

    Rel(gateway, email_svc, "gRPC/HTTP")
    Rel(gateway, cal_svc, "gRPC/HTTP")
    Rel(gateway, meet_svc, "gRPC/HTTP")
    Rel(gateway, chat_svc, "gRPC/HTTP")
    Rel(gateway, docs_svc, "gRPC/HTTP")
    Rel(gateway, drive_svc, "gRPC/HTTP")
    Rel(gateway, contacts_svc, "gRPC/HTTP")

    Rel(email_svc, smtp_server, "JMAP/LMTP")
    Rel(email_svc, ai_svc, "gRPC")
    Rel(meet_svc, livekit, "LiveKit SDK")
    Rel(docs_svc, onlyoffice, "WOPI")
    Rel(drive_svc, nextcloud, "WebDAV")
    Rel(drive_svc, minio, "S3 API")

    Rel(email_svc, pg, "SQL")
    Rel(cal_svc, pg, "SQL")
    Rel(chat_svc, pg, "SQL")
    Rel(contacts_svc, pg, "SQL")
    Rel(email_svc, redis, "Cache")
    Rel(chat_svc, redis, "Pub/Sub")
    Rel(email_svc, kafka, "Events")
    Rel(chat_svc, kafka, "Events")
    Rel(email_svc, quickwit, "Index")
```

### 2.3 Component Diagram - Email Service (Level 3)

```mermaid
flowchart TB
    subgraph email_service["email-service"]
        handler["HTTP Handlers<br/>/v1/email/*"]
        jmap_proxy["JMAP Proxy<br/>Conversation Threading"]
        rules_engine["Rules Engine<br/>Filter Evaluation"]
        dlp_engine["DLP Engine<br/>PII Detection"]
        delivery["Delivery Manager<br/>Queue + Retry"]
        delegation["Delegation Manager<br/>Send-As / On-Behalf"]
        shared_mb["Shared Mailbox<br/>Manager"]
        dist_list["Distribution List<br/>Manager"]
        archive["Archive Manager<br/>Retention + eDiscovery"]
        smime["S/MIME Handler<br/>Encrypt/Sign/Verify"]
    end

    subgraph rust_mail["Rust SMTP/JMAP Server"]
        smtp_in["SMTP Inbound<br/>MX Receiver"]
        smtp_out["SMTP Outbound<br/>Relay Sender"]
        jmap_server["JMAP Server<br/>RFC 8620/8621"]
        store["Mail Store<br/>Blob + Metadata"]
    end

    client["Web/Mobile/Desktop Client"]
    ai["AI Features Service"]
    pg["PostgreSQL"]
    minio_s["MinIO"]
    kafka_s["Redpanda"]

    client --> handler
    handler --> jmap_proxy --> jmap_server
    handler --> rules_engine
    handler --> dlp_engine --> ai
    handler --> delivery --> smtp_out
    handler --> delegation
    handler --> shared_mb
    handler --> dist_list
    handler --> archive
    handler --> smime

    smtp_in --> store --> pg
    smtp_in --> store --> minio_s
    jmap_server --> store
    handler --> kafka_s
```

---

## 3. Data Flow Architecture

### 3.1 Email Send Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant GW as API Gateway
    participant ES as email-service
    participant DLP as DLP Engine
    participant AI as AI Service
    participant SMTP as Rust SMTP
    participant PG as PostgreSQL
    participant KB as Redpanda

    C->>GW: POST /v1/email/send
    GW->>GW: Validate JWT + X-Tenant-ID
    GW->>ES: Forward request
    ES->>DLP: Scan for PII
    DLP->>AI: Classify content
    AI-->>DLP: Classification result
    DLP-->>ES: Scan result (pass/warn/block)
    ES->>PG: Store message record
    ES->>SMTP: Submit for delivery
    SMTP->>SMTP: SPF/DKIM sign
    SMTP-->>ES: Accepted
    ES->>KB: Publish erp.workspace.email.created
    ES-->>GW: 201 Created
    GW-->>C: Message sent
```

### 3.2 Video Meeting Join Flow

```mermaid
sequenceDiagram
    participant C as Client (WebRTC)
    participant GW as API Gateway
    participant MS as meet-service
    participant LK as LiveKit SFU
    participant PG as PostgreSQL
    participant KB as Redpanda

    C->>GW: POST /v1/meet/{id}/join
    GW->>GW: Validate JWT
    GW->>MS: Forward join request
    MS->>PG: Load meeting config
    MS->>MS: Check waiting room policy
    MS->>LK: Generate room token
    LK-->>MS: JWT room token
    MS->>PG: Add participant record
    MS->>KB: Publish erp.workspace.meet.updated
    MS-->>GW: Room token + SFU URL
    GW-->>C: Join credentials
    C->>LK: Connect WebRTC (DTLS/SRTP)
    LK-->>C: Media streams
```

---

## 4. Infrastructure Architecture

```mermaid
flowchart TB
    subgraph k8s["Kubernetes Cluster"]
        subgraph ns_ws["namespace: erp-workspace"]
            subgraph svc_pods["Service Pods"]
                ep["email-service<br/>x3 replicas"]
                cp["calendar-service<br/>x2 replicas"]
                mp["meet-service<br/>x2 replicas"]
                chp["chat-service<br/>x3 replicas"]
                dp["docs-service<br/>x2 replicas"]
                drp["drive-service<br/>x2 replicas"]
                cop["contacts-service<br/>x2 replicas"]
            end

            subgraph infra_pods["Infrastructure Pods"]
                rust_mail["Rust SMTP/JMAP<br/>x3 replicas"]
                ai_pod["AI Features (FastAPI)<br/>x2 replicas"]
                gw_pod["API Gateway<br/>x3 replicas"]
            end
        end

        subgraph ns_data["namespace: erp-data"]
            pg_primary["PostgreSQL Primary"]
            pg_replica["PostgreSQL Replica x2"]
            redis_cluster["Redis Cluster x3"]
            minio_cluster["MinIO Cluster x4"]
        end

        subgraph ns_stream["namespace: erp-streaming"]
            redpanda["Redpanda x3"]
            quickwit_n["Quickwit x2"]
            clickhouse_n["ClickHouse x2"]
        end

        subgraph ns_collab["namespace: erp-collaboration"]
            livekit_n["LiveKit SFU x3"]
            onlyoffice_n["ONLYOFFICE DS x2"]
            nextcloud_n["Nextcloud x2"]
        end
    end

    lb["Load Balancer<br/>(Ingress Controller)"]
    lb --> gw_pod
    gw_pod --> svc_pods
    svc_pods --> ns_data
    svc_pods --> ns_stream
    svc_pods --> ns_collab
    rust_mail --> pg_primary
    rust_mail --> minio_cluster
```

---

## 5. Event-Driven Architecture

### 5.1 Event Topology

```mermaid
flowchart LR
    subgraph producers["Event Producers"]
        es["email-service"]
        cs["calendar-service"]
        ms["meet-service"]
        chs["chat-service"]
        ds["docs-service"]
        drs["drive-service"]
        cos["contacts-service"]
    end

    bus["Redpanda<br/>Event Bus"]

    subgraph consumers["Event Consumers"]
        search["Search Indexer<br/>(Quickwit)"]
        analytics["Analytics Pipeline<br/>(ClickHouse)"]
        notif["Notification Hub<br/>(ERP-Platform)"]
        audit["Audit Service<br/>(ERP-Platform)"]
        ai_c["AI Pipeline<br/>(Classification, Summaries)"]
        sync["External Sync<br/>(ERP-iPaaS)"]
    end

    es -->|"erp.workspace.email.*"| bus
    cs -->|"erp.workspace.calendar.*"| bus
    ms -->|"erp.workspace.meet.*"| bus
    chs -->|"erp.workspace.chat.*"| bus
    ds -->|"erp.workspace.docs.*"| bus
    drs -->|"erp.workspace.drive.*"| bus
    cos -->|"erp.workspace.contacts.*"| bus

    bus --> search
    bus --> analytics
    bus --> notif
    bus --> audit
    bus --> ai_c
    bus --> sync
```

### 5.2 Event Schema (CloudEvents)

All events follow the CloudEvents v1.0 specification:

```json
{
  "specversion": "1.0",
  "type": "erp.workspace.email.created",
  "source": "/erp-workspace/email-service",
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "time": "2026-02-23T10:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    "tenant_id": "...",
    "message_id": "...",
    "subject": "...",
    "from": "...",
    "to": ["..."]
  }
}
```

---

## 6. Security Architecture

```mermaid
flowchart TB
    subgraph client_layer["Client Layer"]
        web["Web App<br/>HTTPS/TLS 1.3"]
        mobile["Mobile App<br/>Certificate Pinning"]
        desktop["Desktop App<br/>HTTPS/TLS 1.3"]
    end

    subgraph edge["Edge Layer"]
        waf["WAF<br/>OWASP Rules"]
        lb["Load Balancer<br/>TLS Termination"]
        rate["Rate Limiter<br/>Per-tenant quotas"]
    end

    subgraph auth_layer["Authentication Layer"]
        jwt["JWT Validation<br/>RS256/ES256"]
        iam_check["ERP-IAM OIDC<br/>Token Introspection"]
        tenant_check["Tenant Isolation<br/>X-Tenant-ID enforcement"]
    end

    subgraph service_layer["Service Layer"]
        mtls["mTLS<br/>Service-to-Service"]
        rbac["RBAC<br/>Role-Based Access"]
        rls["RLS<br/>Row-Level Security"]
    end

    subgraph data_layer["Data Layer"]
        encrypt_rest["Encryption at Rest<br/>AES-256"]
        encrypt_transit["Encryption in Transit<br/>TLS 1.3"]
        smime_layer["S/MIME<br/>Email E2E"]
        dtls_layer["DTLS/SRTP<br/>Media E2E"]
    end

    client_layer --> edge --> auth_layer --> service_layer --> data_layer
```

---

## 7. Technology Decision Records

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Mail server language | Rust | Memory safety + zero-GC pauses for 100K msg/sec |
| API services language | Go | Fast compilation, low memory, stdlib HTTP |
| AI features language | Python | Rich ML ecosystem, Anthropic SDK |
| Video infrastructure | LiveKit SFU | Open-source, scalable, WebRTC-native |
| Document editing | ONLYOFFICE | Full OOXML compatibility, real-time OT |
| File storage | Nextcloud + MinIO | WebDAV + S3 API, enterprise features |
| Primary database | PostgreSQL 16 | JSONB, RLS, GiST indexes, mature ecosystem |
| Event streaming | Redpanda | Kafka-compatible, no JVM, lower latency |
| Search engine | Quickwit | Rust-based, sub-second search, cost-efficient |
| Analytics | ClickHouse | Column-oriented, 100x faster analytics |

---

*For detailed API specifications, see [21-API-Documentation.md](./21-API-Documentation.md). For deployment topology, see [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md).*
