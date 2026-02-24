# ERP-Workspace High-Level Design

> **Document ID:** ERP-WS-HLD-012
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## 1. Design Philosophy

ERP-Workspace is designed around four core principles: (1) **Unified Experience** -- a single pane of glass for all communication and collaboration; (2) **Polyglot Performance** -- the right language for each job (Rust for email throughput, Go for API services, Python for AI); (3) **Open Standards** -- SMTP, JMAP, CalDAV, WebDAV, WebRTC, CloudEvents; (4) **Self-Sovereign Data** -- full control over data location, encryption, and retention.

---

## 2. System Decomposition

```mermaid
flowchart TB
    subgraph presentation["Presentation Layer"]
        web["React Web App"]
        mobile["Flutter Mobile"]
        desktop["Electron Desktop"]
        ext_clients["CalDAV/SMTP Clients"]
    end

    subgraph edge_layer["Edge Layer"]
        cdn["CDN / Static Assets"]
        waf["WAF / DDoS Protection"]
        lb["Load Balancer"]
    end

    subgraph api_layer["API Layer"]
        gateway["API Gateway<br/>Auth, Rate Limit, Routing"]
    end

    subgraph service_layer["Service Layer"]
        email["email-service"]
        calendar["calendar-service"]
        meet["meet-service"]
        chat["chat-service"]
        docs["docs-service"]
        drive["drive-service"]
        contacts["contacts-service"]
    end

    subgraph infra_layer["Infrastructure Services"]
        smtp["Rust SMTP/JMAP"]
        ai["Python AI/ML"]
        livekit["LiveKit SFU"]
        onlyoffice["ONLYOFFICE DS"]
        nextcloud["Nextcloud"]
    end

    subgraph data_layer["Data Layer"]
        pg["PostgreSQL 16"]
        redis["Redis 7"]
        minio["MinIO"]
        redpanda["Redpanda"]
        quickwit["Quickwit"]
        clickhouse["ClickHouse"]
    end

    subgraph platform_layer["ERP Platform Integration"]
        iam["ERP-IAM"]
        platform["ERP-Platform"]
        erp_ai["ERP-AI"]
        ipaas["ERP-iPaaS"]
    end

    presentation --> edge_layer --> api_layer --> service_layer
    service_layer --> infra_layer
    service_layer --> data_layer
    service_layer --> platform_layer
```

---

## 3. Service Interaction Design

### 3.1 Synchronous Communication

All client-to-service communication uses REST over HTTPS with JSON payloads. Internal service-to-service calls use gRPC for type-safe, high-performance RPC where latency is critical (entitlement checks, AI inference). The API gateway handles:

- JWT token validation (RS256) via ERP-IAM
- X-Tenant-ID header enforcement
- Rate limiting per tenant (Redis-backed token bucket)
- Request routing to appropriate service
- Response caching for read-heavy endpoints

### 3.2 Asynchronous Communication

Event-driven communication uses Redpanda (Kafka-compatible) with CloudEvents envelope format. Every state change publishes an event to the `erp.workspace.<entity>.<action>` topic hierarchy. Consumer groups include:

```mermaid
flowchart LR
    subgraph producers["Producers"]
        p1["email-service"]
        p2["calendar-service"]
        p3["chat-service"]
        p4["docs-service"]
        p5["drive-service"]
    end

    bus["Redpanda"]

    subgraph consumers["Consumer Groups"]
        cg1["search-indexer<br/>(Quickwit)"]
        cg2["analytics-pipeline<br/>(ClickHouse)"]
        cg3["ai-pipeline<br/>(Classification, Summary)"]
        cg4["notification-dispatcher"]
        cg5["audit-logger"]
        cg6["sync-connector<br/>(ERP-iPaaS)"]
    end

    producers --> bus --> consumers
```

### 3.3 Real-time Communication

WebSocket connections provide real-time updates for:
- Chat message delivery and typing indicators
- Document co-editing cursor positions
- Meeting participant state changes
- Inbox notification push

Redis Pub/Sub distributes real-time events across service replicas. The WebSocket connection lifecycle:

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant Redis
    participant Service

    Client->>Gateway: WSS handshake + JWT
    Gateway->>Gateway: Validate JWT
    Gateway->>Redis: SUBSCRIBE user:{id}
    Service->>Redis: PUBLISH user:{id} (new message)
    Redis->>Gateway: Message
    Gateway->>Client: WSS frame
```

---

## 4. Multi-Tenant Architecture

### 4.1 Tenant Isolation Model

```mermaid
flowchart TB
    request["Incoming Request"]
    request -->|"extract"| tenant_id["X-Tenant-ID Header"]
    tenant_id -->|"validate"| jwt["JWT Claims<br/>tenant_id must match"]
    jwt -->|"set session"| pg_session["SET app.current_tenant = '{id}'"]
    pg_session -->|"enforce"| rls["PostgreSQL RLS Policy<br/>WHERE tenant_id = current_setting('app.current_tenant')"]
    rls -->|"query"| data["Tenant-Scoped Data"]
```

### 4.2 Data Isolation Layers

| Layer | Mechanism | Granularity |
|-------|-----------|-------------|
| Network | Kubernetes network policies | Namespace |
| Application | X-Tenant-ID header validation | Request |
| Database | Row-Level Security (RLS) | Row |
| Storage | MinIO bucket per tenant | Object |
| Cache | Redis key prefix per tenant | Key |
| Events | Tenant ID in CloudEvents | Event |

---

## 5. Email Architecture (High-Level)

```mermaid
flowchart TB
    subgraph inbound["Inbound Path"]
        mx["MX Record<br/>→ SMTP Inbound"]
        spf["SPF Check"]
        dkim_v["DKIM Verify"]
        dmarc_v["DMARC Check"]
        spam["Spam Filter"]
        rules["Rules Engine"]
        store_in["JMAP Store"]
    end

    subgraph outbound["Outbound Path"]
        compose["Compose"]
        dlp["DLP Scan"]
        dkim_s["DKIM Sign"]
        queue["Delivery Queue"]
        relay["SMTP Relay"]
    end

    subgraph access["Access Path"]
        jmap_api["JMAP API"]
        smtp_api["SMTP/IMAP"]
        web_api["REST API"]
    end

    mx --> spf --> dkim_v --> dmarc_v --> spam --> rules --> store_in
    compose --> dlp --> dkim_s --> queue --> relay
    store_in --> jmap_api & smtp_api & web_api
```

---

## 6. Video Meeting Architecture

```mermaid
flowchart TB
    subgraph client["Client Layer"]
        browser["Browser<br/>WebRTC"]
        native["Mobile/Desktop<br/>WebRTC"]
    end

    subgraph signaling["Signaling Layer"]
        meet_api["meet-service<br/>REST API"]
        token_gen["Token Generator"]
    end

    subgraph media["Media Layer"]
        sfu1["LiveKit SFU Node 1"]
        sfu2["LiveKit SFU Node 2"]
        sfu3["LiveKit SFU Node 3"]
    end

    subgraph recording["Recording Layer"]
        rec_worker["Recording Worker"]
        rec_store["MinIO<br/>Recording Blobs"]
    end

    subgraph ai_layer["AI Layer"]
        caption["Speech-to-Text<br/>Live Captions"]
        notes["Meeting Notes<br/>Summarizer"]
    end

    client --> meet_api
    meet_api --> token_gen
    token_gen --> client
    client <-->|"WebRTC"| media
    media --> rec_worker --> rec_store
    media --> caption
    rec_store --> notes
```

---

## 7. Storage Architecture

```mermaid
flowchart LR
    client["Client"] --> drive_svc["drive-service"]
    drive_svc --> nextcloud_layer["Nextcloud<br/>WebDAV Interface"]
    drive_svc --> minio_layer["MinIO<br/>S3 Object Store"]
    drive_svc --> pg["PostgreSQL<br/>file_items metadata"]

    nextcloud_layer --> minio_layer

    subgraph minio_cluster["MinIO Cluster"]
        node1["Node 1<br/>Erasure Set A"]
        node2["Node 2<br/>Erasure Set A"]
        node3["Node 3<br/>Erasure Set B"]
        node4["Node 4<br/>Erasure Set B"]
    end

    minio_layer --> minio_cluster
```

---

## 8. AI Feature Architecture

```mermaid
flowchart TB
    subgraph triggers["Triggers"]
        new_email["New Email Received"]
        compose_req["Compose Request"]
        meeting_end["Meeting Ended"]
        thread_update["Thread Updated"]
    end

    subgraph ai_services["AI Services (Python/FastAPI)"]
        classifier["Email Classifier<br/>Category + Sentiment"]
        triage["Triage Engine<br/>Priority Scoring"]
        smart_compose["Smart Compose<br/>Draft Generation"]
        summarizer["Summarizer<br/>Notes + Action Items"]
        pii_detector["PII Detector<br/>DLP Engine"]
        kg_builder["Knowledge Graph<br/>Entity Extraction"]
    end

    subgraph models["Model Backend"]
        llm["ERP-AI<br/>LLM Inference"]
        embeddings["Embedding Service"]
    end

    subgraph storage["AI Storage"]
        pg["PostgreSQL<br/>AI tables"]
        cache["Redis<br/>Suggestion cache"]
    end

    new_email --> classifier & triage & pii_detector & kg_builder
    compose_req --> smart_compose
    meeting_end --> summarizer
    thread_update --> summarizer

    classifier & triage & smart_compose & summarizer & pii_detector & kg_builder --> models
    ai_services --> storage
```

---

## 9. Deployment Topology

| Component | Replicas | CPU | Memory | Storage |
|-----------|---------|-----|--------|---------|
| API Gateway | 3 | 1 core | 512MB | - |
| email-service | 3 | 2 cores | 1GB | - |
| calendar-service | 2 | 1 core | 512MB | - |
| meet-service | 2 | 1 core | 512MB | - |
| chat-service | 3 | 2 cores | 1GB | - |
| docs-service | 2 | 1 core | 512MB | - |
| drive-service | 2 | 1 core | 512MB | - |
| contacts-service | 2 | 1 core | 512MB | - |
| Rust SMTP/JMAP | 3 | 4 cores | 4GB | - |
| AI Features (FastAPI) | 2 | 2 cores | 2GB | - |
| PostgreSQL Primary | 1 | 8 cores | 32GB | 500GB SSD |
| PostgreSQL Replica | 2 | 4 cores | 16GB | 500GB SSD |
| Redis Cluster | 3 | 2 cores | 8GB | - |
| MinIO Cluster | 4 | 2 cores | 4GB | 10TB HDD |
| Redpanda | 3 | 4 cores | 8GB | 200GB SSD |
| Quickwit | 2 | 4 cores | 8GB | 500GB SSD |
| ClickHouse | 2 | 4 cores | 16GB | 1TB SSD |
| LiveKit SFU | 3 | 8 cores | 8GB | - |
| ONLYOFFICE DS | 2 | 4 cores | 4GB | - |
| Nextcloud | 2 | 2 cores | 4GB | - |

---

*For detailed component specifications, see [14-Technical-Specifications.md](./14-Technical-Specifications.md). For deployment procedures, see [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md).*
