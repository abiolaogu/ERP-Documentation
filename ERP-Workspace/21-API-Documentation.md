# ERP-Workspace API Documentation

> **Document ID:** ERP-WS-API-021
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## 1. API Overview

All workspace APIs follow RESTful conventions over HTTPS with JSON request/response bodies. Authentication is via Bearer JWT tokens from ERP-IAM. Multi-tenant isolation is enforced via the `X-Tenant-ID` header.

### Common Headers

| Header | Required | Description |
|--------|---------|-------------|
| `Authorization` | Yes | `Bearer <JWT>` from ERP-IAM |
| `X-Tenant-ID` | Yes | UUID of the authenticated tenant |
| `Content-Type` | For POST/PUT | `application/json` |
| `X-Request-ID` | Optional | Client-generated correlation ID |

### Common Response Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (invalid/expired JWT) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found |
| 429 | Rate Limited |
| 500 | Internal Server Error |

---

## 2. Email API

### 2.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/email` | List emails (paginated) |
| POST | `/v1/email` | Send/create email |
| GET | `/v1/email/{id}` | Get email by ID |
| PUT | `/v1/email/{id}` | Update email (labels, read status) |
| DELETE | `/v1/email/{id}` | Delete/trash email |
| POST | `/v1/email/{id}/snooze` | Snooze email until specified time |
| POST | `/v1/email/search` | Full-text search across emails |
| GET | `/v1/email/threads/{threadId}` | Get conversation thread |

### 2.2 Send Email Request

```json
POST /v1/email
{
  "from": "user@company.com",
  "to": ["recipient@example.com"],
  "cc": [],
  "bcc": [],
  "subject": "Project Update",
  "body_html": "<p>Hello, here is the update...</p>",
  "body_text": "Hello, here is the update...",
  "attachments": [
    {
      "file_id": "uuid",
      "filename": "report.pdf"
    }
  ],
  "reply_to_message_id": "uuid",
  "signature_id": "uuid",
  "schedule_at": "2026-02-24T09:00:00Z",
  "confidential": false
}
```

### 2.3 Email List Response

```json
{
  "items": [
    {
      "id": "uuid",
      "thread_id": "uuid",
      "from": {"name": "Alice", "email": "alice@example.com"},
      "to": [{"name": "Bob", "email": "bob@company.com"}],
      "subject": "Re: Project Update",
      "preview": "Looks good, let's discuss...",
      "status": "delivered",
      "labels": ["inbox", "important"],
      "is_read": false,
      "is_focused": true,
      "has_attachments": true,
      "sent_at": "2026-02-23T10:30:00Z"
    }
  ],
  "total": 1542,
  "page": 1,
  "per_page": 50,
  "event_topic": "erp.workspace.email.listed"
}
```

---

## 3. Calendar API

### 3.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/calendar` | List calendars |
| POST | `/v1/calendar` | Create calendar |
| GET | `/v1/calendar/{id}` | Get calendar |
| PUT | `/v1/calendar/{id}` | Update calendar |
| DELETE | `/v1/calendar/{id}` | Delete calendar |
| GET | `/v1/calendar/{id}/events` | List events in calendar |
| POST | `/v1/calendar/events` | Create event |
| GET | `/v1/calendar/events/{id}` | Get event |
| PUT | `/v1/calendar/events/{id}` | Update event |
| DELETE | `/v1/calendar/events/{id}` | Delete/cancel event |
| POST | `/v1/calendar/events/{id}/rsvp` | Respond to invitation |
| GET | `/v1/calendar/free-busy` | Query free/busy |
| GET | `/v1/calendar/rooms` | List meeting rooms |
| POST | `/v1/calendar/rooms/{id}/book` | Book meeting room |

### 3.2 Create Event Request

```json
POST /v1/calendar/events
{
  "calendar_id": "uuid",
  "title": "Sprint Planning",
  "description": "Bi-weekly sprint planning meeting",
  "start_time": "2026-02-24T10:00:00Z",
  "end_time": "2026-02-24T11:00:00Z",
  "timezone": "America/New_York",
  "all_day": false,
  "location": {"name": "Room 301", "room_id": "uuid"},
  "attendees": [
    {"email": "alice@company.com", "role": "required"},
    {"email": "bob@company.com", "role": "optional"}
  ],
  "recurrence": {
    "frequency": "weekly",
    "interval": 2,
    "by_day": ["MO"],
    "count": 26
  },
  "reminders": [
    {"type": "notification", "minutes_before": 15},
    {"type": "email", "minutes_before": 60}
  ]
}
```

---

## 4. Meet API

### 4.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/meet` | List meetings |
| POST | `/v1/meet` | Create meeting |
| GET | `/v1/meet/{id}` | Get meeting details |
| PUT | `/v1/meet/{id}` | Update meeting |
| DELETE | `/v1/meet/{id}` | End/cancel meeting |
| POST | `/v1/meet/{id}/join` | Join meeting (get room token) |
| POST | `/v1/meet/{id}/record` | Start/stop recording |
| GET | `/v1/meet/{id}/participants` | List participants |
| POST | `/v1/meet/{id}/breakout` | Create breakout rooms |

### 4.2 Join Meeting Response

```json
{
  "room_id": "uuid",
  "token": "eyJ...",
  "sfu_url": "wss://livekit.workspace.erp/",
  "permissions": {
    "can_publish": true,
    "can_subscribe": true,
    "can_share_screen": true
  },
  "meeting": {
    "title": "Sprint Planning",
    "host_id": "uuid",
    "waiting_room_enabled": false,
    "recording_allowed": true
  }
}
```

---

## 5. Chat API

### 5.1 Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/chat/conversations` | List conversations |
| POST | `/v1/chat/conversations` | Create channel/DM |
| GET | `/v1/chat/conversations/{id}` | Get conversation |
| PUT | `/v1/chat/conversations/{id}` | Update conversation |
| DELETE | `/v1/chat/conversations/{id}` | Archive conversation |
| GET | `/v1/chat/conversations/{id}/messages` | List messages |
| POST | `/v1/chat/conversations/{id}/messages` | Send message |
| PUT | `/v1/chat/messages/{id}` | Edit message |
| DELETE | `/v1/chat/messages/{id}` | Delete message |
| POST | `/v1/chat/messages/{id}/reactions` | Add reaction |
| DELETE | `/v1/chat/messages/{id}/reactions/{emoji}` | Remove reaction |
| POST | `/v1/chat/messages/{id}/pin` | Pin/unpin message |

### 5.2 WebSocket Connection

```
WSS /v1/chat/ws?token=<JWT>

// Client sends:
{"type": "message", "conversation_id": "uuid", "content": "Hello team!"}
{"type": "typing", "conversation_id": "uuid"}
{"type": "read_receipt", "message_id": "uuid"}

// Server pushes:
{"type": "message", "data": {...}}
{"type": "typing", "user_id": "uuid", "conversation_id": "uuid"}
{"type": "reaction", "message_id": "uuid", "emoji": "thumbsup", "user_id": "uuid"}
{"type": "presence", "user_id": "uuid", "status": "online"}
```

---

## 6. Docs API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/docs` | List documents |
| POST | `/v1/docs` | Create document |
| GET | `/v1/docs/{id}` | Get document metadata |
| PUT | `/v1/docs/{id}` | Update document metadata |
| DELETE | `/v1/docs/{id}` | Delete document |
| GET | `/v1/docs/{id}/edit` | Get ONLYOFFICE editor URL |
| GET | `/v1/docs/{id}/versions` | List version history |
| POST | `/v1/docs/{id}/versions/{ver}/restore` | Restore version |

---

## 7. Drive API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/drive` | List files/folders |
| POST | `/v1/drive/upload` | Upload file (multipart) |
| GET | `/v1/drive/{id}` | Get file metadata |
| GET | `/v1/drive/{id}/download` | Download file |
| PUT | `/v1/drive/{id}` | Update file metadata |
| DELETE | `/v1/drive/{id}` | Trash file |
| POST | `/v1/drive/{id}/share` | Share file |
| GET | `/v1/drive/{id}/versions` | List file versions |
| POST | `/v1/drive/{id}/copy` | Copy file |
| POST | `/v1/drive/{id}/move` | Move file |
| POST | `/v1/drive/folders` | Create folder |
| POST | `/v1/drive/search` | Search files |

---

## 8. Contacts API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/contacts` | List contacts |
| POST | `/v1/contacts` | Create contact |
| GET | `/v1/contacts/{id}` | Get contact |
| PUT | `/v1/contacts/{id}` | Update contact |
| DELETE | `/v1/contacts/{id}` | Delete contact |
| GET | `/v1/contacts/groups` | List contact groups |
| POST | `/v1/contacts/groups` | Create group |
| POST | `/v1/contacts/search` | Search contacts |
| POST | `/v1/contacts/import` | Import contacts (vCard/CSV) |
| GET | `/v1/contacts/export` | Export contacts |

---

## 9. Health & Capabilities

| Method | Path | Description |
|--------|------|-------------|
| GET | `/healthz` | Health check (no auth required) |
| GET | `/v1/capabilities` | List enabled capabilities |

---

## 10. Rate Limiting

| Endpoint Group | Limit | Window |
|---------------|-------|--------|
| Email send | 100 req/min per user | Sliding window |
| Email list/read | 1000 req/min per user | Sliding window |
| Chat messages | 300 req/min per user | Sliding window |
| File upload | 50 req/min per user | Sliding window |
| Search | 100 req/min per user | Sliding window |
| General API | 5000 req/min per tenant | Sliding window |

Rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`

---

*For technical specifications behind these APIs, see [14-Technical-Specifications.md](./14-Technical-Specifications.md).*
