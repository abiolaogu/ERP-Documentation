# ERP-Workspace Data Flow Diagrams

> **Document ID:** ERP-WS-DFD-008
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## 1. Level 0 - Context Diagram

```mermaid
flowchart TB
    user["External Entity:<br/>Workspace User"]
    admin["External Entity:<br/>IT Admin"]
    ext_mta["External Entity:<br/>Internet Email (MTA)"]
    ext_cal["External Entity:<br/>CalDAV Client"]
    iam["External Entity:<br/>ERP-IAM"]
    platform["External Entity:<br/>ERP-Platform"]

    workspace(("ERP-Workspace<br/>Process 0"))

    user -->|"emails, messages,<br/>documents, meetings"| workspace
    workspace -->|"notifications,<br/>content, media"| user
    admin -->|"policies, config,<br/>provisioning"| workspace
    workspace -->|"reports, audit logs"| admin
    ext_mta <-->|"SMTP inbound/outbound"| workspace
    ext_cal <-->|"CalDAV sync"| workspace
    iam -->|"JWT tokens, OIDC"| workspace
    platform -->|"entitlements"| workspace
```

---

## 2. Level 1 - Major Process Decomposition

```mermaid
flowchart TB
    user["User"]

    subgraph p1["1.0 Email Processing"]
        p1a["1.1 Compose & Send"]
        p1b["1.2 Receive & Store"]
        p1c["1.3 Thread & Index"]
        p1d["1.4 DLP Scan"]
    end

    subgraph p2["2.0 Calendar Management"]
        p2a["2.1 Event CRUD"]
        p2b["2.2 Scheduling"]
        p2c["2.3 Room Booking"]
    end

    subgraph p3["3.0 Video Meeting"]
        p3a["3.1 Room Management"]
        p3b["3.2 Media Routing"]
        p3c["3.3 Recording"]
    end

    subgraph p4["4.0 Chat Messaging"]
        p4a["4.1 Channel Management"]
        p4b["4.2 Message Delivery"]
        p4c["4.3 Thread & Reaction"]
    end

    subgraph p5["5.0 Document Editing"]
        p5a["5.1 Document CRUD"]
        p5b["5.2 Real-time Co-edit"]
        p5c["5.3 Version Control"]
    end

    subgraph p6["6.0 File Storage"]
        p6a["6.1 Upload/Download"]
        p6b["6.2 Sharing"]
        p6c["6.3 Search"]
    end

    subgraph stores["Data Stores"]
        ds1[("D1: PostgreSQL")]
        ds2[("D2: MinIO")]
        ds3[("D3: Redis")]
        ds4[("D4: Redpanda")]
        ds5[("D5: Quickwit")]
        ds6[("D6: ClickHouse")]
    end

    user --> p1a & p2a & p3a & p4a & p5a & p6a
    p1a --> ds1 & ds4
    p1b --> ds1 & ds2
    p1c --> ds5
    p1d --> ds1
    p2a --> ds1 & ds4
    p3a --> ds1
    p3c --> ds2
    p4b --> ds1 & ds3 & ds4
    p5b --> ds1
    p6a --> ds2
    p6b --> ds1
    p6c --> ds5
```

---

## 3. Email Data Flows

### 3.1 Outbound Email Flow

```mermaid
flowchart LR
    user["User"] -->|"compose"| compose["Compose Service"]
    compose -->|"draft"| pg[("PostgreSQL<br/>email_messages")]
    compose -->|"AI assist"| ai["AI Service<br/>(smart compose)"]
    ai -->|"suggestions"| compose
    compose -->|"DLP check"| dlp["DLP Engine"]
    dlp -->|"PII scan"| privacy[("privacy_pii_detections")]
    dlp -->|"pass/warn/block"| compose
    compose -->|"submit"| delivery["Delivery Queue<br/>(Redpanda)"]
    delivery -->|"dequeue"| smtp["Rust SMTP<br/>Outbound"]
    smtp -->|"sign"| dkim["DKIM/SPF<br/>Signer"]
    dkim -->|"relay"| ext["External MTA"]
    smtp -->|"status"| events["Email Events<br/>(email_events)"]
    events -->|"sync"| clickhouse[("ClickHouse<br/>Analytics")]
```

### 3.2 Inbound Email Flow

```mermaid
flowchart LR
    ext["External MTA"] -->|"SMTP"| smtp_in["Rust SMTP<br/>Inbound"]
    smtp_in -->|"verify"| auth["SPF/DKIM/DMARC<br/>Verification"]
    auth -->|"classify"| spam["Spam Filter"]
    spam -->|"store"| jmap["JMAP Store"]
    jmap -->|"metadata"| pg[("PostgreSQL<br/>email_messages")]
    jmap -->|"blob"| minio[("MinIO<br/>Mail Blobs")]
    jmap -->|"index"| quickwit[("Quickwit<br/>Search Index")]
    jmap -->|"rules"| rules["Rules Engine"]
    rules -->|"apply labels/move"| pg
    jmap -->|"event"| bus["Redpanda<br/>erp.workspace.email.created"]
    bus -->|"classify"| ai["AI Classification<br/>(ai_email_classifications)"]
    bus -->|"notify"| notif["Notification Hub"]
```

---

## 4. Meeting Data Flow

```mermaid
flowchart TB
    host["Host"] -->|"create meeting"| meet_svc["meet-service"]
    meet_svc -->|"store config"| pg[("PostgreSQL")]
    meet_svc -->|"generate token"| livekit["LiveKit SFU"]

    participant["Participant"] -->|"join"| meet_svc
    meet_svc -->|"check waiting room"| pg
    meet_svc -->|"issue token"| livekit

    livekit -->|"WebRTC media"| participant
    livekit -->|"WebRTC media"| host

    host -->|"start recording"| meet_svc
    meet_svc -->|"record API"| livekit
    livekit -->|"recording blob"| minio[("MinIO")]

    minio -->|"transcript"| ai["AI Service"]
    ai -->|"meeting notes"| pg
    ai -->|"action items"| pg
```

---

## 5. Chat Real-time Data Flow

```mermaid
flowchart LR
    sender["Sender"] -->|"POST message"| chat_svc["chat-service"]
    chat_svc -->|"store"| pg[("PostgreSQL<br/>chat_messages")]
    chat_svc -->|"publish"| redis["Redis Pub/Sub"]
    chat_svc -->|"event"| bus["Redpanda"]

    redis -->|"push"| ws1["WebSocket<br/>Recipient 1"]
    redis -->|"push"| ws2["WebSocket<br/>Recipient 2"]
    redis -->|"push"| wsN["WebSocket<br/>Recipient N"]

    bus -->|"index"| quickwit[("Quickwit")]
    bus -->|"notify"| notif["Push Notifications"]

    ws1 -->|"read receipt"| chat_svc
    chat_svc -->|"store"| pg
```

---

## 6. Document Collaboration Data Flow

```mermaid
flowchart TB
    user1["Editor 1"] -->|"open document"| docs_svc["docs-service"]
    user2["Editor 2"] -->|"open document"| docs_svc

    docs_svc -->|"fetch file"| minio[("MinIO")]
    docs_svc -->|"WOPI token"| onlyoffice["ONLYOFFICE DS"]

    user1 -->|"edits (OT ops)"| onlyoffice
    user2 -->|"edits (OT ops)"| onlyoffice

    onlyoffice -->|"save callback"| docs_svc
    docs_svc -->|"new version"| minio
    docs_svc -->|"version record"| pg[("PostgreSQL<br/>file_versions")]
    docs_svc -->|"session record"| pg2[("collaboration_sessions")]
    docs_svc -->|"event"| bus["Redpanda<br/>erp.workspace.docs.updated"]
```

---

## 7. File Upload and Share Data Flow

```mermaid
flowchart LR
    user["User"] -->|"upload file"| drive_svc["drive-service"]
    drive_svc -->|"store blob"| minio[("MinIO<br/>S3 API")]
    drive_svc -->|"metadata"| pg[("PostgreSQL<br/>file_items")]
    drive_svc -->|"index"| quickwit[("Quickwit")]
    drive_svc -->|"event"| bus["Redpanda"]

    user -->|"share"| drive_svc
    drive_svc -->|"ACL record"| pg2[("file_shares")]
    drive_svc -->|"share link"| pg3[("share_links")]
    drive_svc -->|"notify"| notif["Notification Hub"]

    recipient["Recipient"] -->|"access link"| drive_svc
    drive_svc -->|"verify token"| pg3
    drive_svc -->|"serve file"| minio
```

---

## 8. Search Data Flow

```mermaid
flowchart TB
    user["User"] -->|"search query"| gateway["API Gateway"]
    gateway -->|"route"| search_svc["Search Aggregator"]

    search_svc -->|"email search"| quickwit[("Quickwit<br/>Email Index")]
    search_svc -->|"chat search"| quickwit2[("Quickwit<br/>Chat Index")]
    search_svc -->|"file search"| quickwit3[("Quickwit<br/>File Index")]
    search_svc -->|"contact search"| pg[("PostgreSQL<br/>contacts")]

    quickwit --> search_svc
    quickwit2 --> search_svc
    quickwit3 --> search_svc
    pg --> search_svc

    search_svc -->|"log query"| pg2[("search_query_log")]
    search_svc -->|"ranked results"| gateway
    gateway -->|"results"| user
```

---

## 9. AI Pipeline Data Flow

```mermaid
flowchart TB
    bus["Redpanda<br/>Event Bus"]

    bus -->|"new email"| classify["AI Classifier"]
    classify -->|"category, sentiment"| pg1[("ai_email_classifications")]

    bus -->|"email thread"| summarize["AI Summarizer"]
    summarize -->|"summary, action items"| pg2[("ai_summaries")]

    bus -->|"outbound email"| compose["AI Smart Compose"]
    compose -->|"suggestions"| cache[("ai_compose_cache")]

    bus -->|"email content"| triage["AI Triage"]
    triage -->|"priority score"| pg3[("email_triage_model")]

    bus -->|"meeting recording"| transcribe["AI Transcription"]
    transcribe -->|"notes"| pg2

    bus -->|"email entities"| kg["Knowledge Graph Builder"]
    kg -->|"nodes, edges"| pg4[("knowledge_graph_nodes<br/>knowledge_graph_edges")]
```

---

*For implementation details of each data flow, see [14-Technical-Specifications.md](./14-Technical-Specifications.md).*
