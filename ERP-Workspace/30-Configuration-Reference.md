# ERP-Workspace Configuration Reference

> **Document ID:** ERP-WS-CR-030
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## 1. Module Manifest

Source: `erp/module.manifest.yaml`

```yaml
api_version: v1
module_id: erp_workspace
repository: ERP-Workspace
sku: erp.workspace
subscription:
  standalone: true
  suite: true
integration:
  control_plane: ERP-Platform
  identity_provider: ERP-Directory
  event_backbone: NATS
aidd:
  guardrails_file: erp/aidd.guardrails.yaml
```

---

## 2. Capabilities Configuration

Source: `configs/capabilities.json`

```json
{
  "module": "ERP-Workspace",
  "capabilities": ["email", "calendar", "meetings", "chat", "docs", "sheets", "slides", "drive"]
}
```

Each capability can be individually entitled via ERP-Platform subscription management.

---

## 3. Environment Variables

### 3.1 Common Service Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `PORT` | int | `8080` | HTTP listen port |
| `MODULE_NAME` | string | `ERP-Workspace` | Module identifier |
| `LOG_LEVEL` | string | `info` | Logging level (debug, info, warn, error) |
| `LOG_FORMAT` | string | `json` | Log output format (json, text) |

### 3.2 Database Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `DATABASE_URL` | string | Yes | PostgreSQL connection string |
| `DATABASE_MAX_CONNS` | int | No | Max connection pool size (default: 20) |
| `DATABASE_MIN_CONNS` | int | No | Min connection pool size (default: 5) |
| `DATABASE_CONN_MAX_LIFETIME` | duration | No | Max connection lifetime (default: 1h) |

### 3.3 Redis Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `REDIS_URL` | string | Yes | Redis connection URL |
| `REDIS_CLUSTER_ENABLED` | bool | No | Enable cluster mode (default: false) |
| `REDIS_PASSWORD` | string | No | Redis authentication password |

### 3.4 MinIO Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `MINIO_ENDPOINT` | string | Yes | MinIO server endpoint |
| `MINIO_ACCESS_KEY` | string | Yes | MinIO access key |
| `MINIO_SECRET_KEY` | string | Yes | MinIO secret key |
| `MINIO_USE_SSL` | bool | No | Enable TLS (default: true) |
| `MINIO_BUCKET_PREFIX` | string | No | Bucket name prefix (default: ws) |

### 3.5 Event Bus Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `REDPANDA_BROKERS` | string | Yes | Comma-separated broker addresses |
| `REDPANDA_TOPIC_PREFIX` | string | No | Topic prefix (default: erp.workspace) |
| `REDPANDA_CONSUMER_GROUP` | string | No | Consumer group ID |

### 3.6 Authentication Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `IAM_JWKS_URL` | string | Yes | ERP-IAM JWKS endpoint |
| `IAM_ISSUER` | string | Yes | Expected JWT issuer |
| `IAM_AUDIENCE` | string | Yes | Expected JWT audience |

### 3.7 LiveKit Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `LIVEKIT_URL` | string | Yes | LiveKit server WebSocket URL |
| `LIVEKIT_API_KEY` | string | Yes | LiveKit API key |
| `LIVEKIT_API_SECRET` | string | Yes | LiveKit API secret |

### 3.8 ONLYOFFICE Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `ONLYOFFICE_URL` | string | Yes | ONLYOFFICE Document Server URL |
| `ONLYOFFICE_JWT_SECRET` | string | Yes | JWT secret for WOPI tokens |

### 3.9 AI Service Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `AI_SERVICE_URL` | string | No | ERP-AI inference endpoint |
| `AI_ENABLED` | bool | No | Enable AI features (default: true) |
| `AI_TIMEOUT_MS` | int | No | AI request timeout (default: 10000) |

### 3.10 Search Configuration

| Variable | Type | Required | Description |
|----------|------|---------|-------------|
| `QUICKWIT_URL` | string | Yes | Quickwit search endpoint |
| `QUICKWIT_INDEX_PREFIX` | string | No | Index name prefix (default: ws) |

---

## 4. Tenant Configuration

Stored in the `tenants` table, configurable per tenant:

| Setting | JSONB Path | Default | Description |
|---------|-----------|---------|-------------|
| Logo URL | `branding.logo_url` | null | Custom logo image |
| Primary Color | `branding.primary_color` | `#2563EB` | Brand primary color |
| Theme Mode | `branding.theme_mode` | `light` | light, dark, system |
| Default Language | `locale_settings.language` | `en` | ISO 639-1 code |
| Default Timezone | `locale_settings.timezone` | `UTC` | IANA timezone |
| Date Format | `locale_settings.date_format` | `YYYY-MM-DD` | Date display format |

---

## 5. Feature Flags

| Flag | Default | Description |
|------|---------|-------------|
| `feature.ai.smart_compose` | true | Enable AI compose suggestions |
| `feature.ai.email_triage` | false | Enable Focus/Other inbox |
| `feature.ai.meeting_notes` | true | Enable AI meeting summaries |
| `feature.dlp.auto_scan` | true | Enable outbound PII scanning |
| `feature.dlp.auto_redact` | false | Enable automatic PII redaction |
| `feature.meet.waiting_room` | false | Default waiting room to on |
| `feature.meet.recording` | true | Allow meeting recording |
| `feature.chat.guest_access` | false | Allow external guests in chat |
| `feature.drive.share_links` | true | Allow creating public share links |

---

## 6. Rate Limit Configuration

| Endpoint Group | Variable | Default |
|---------------|----------|---------|
| Email send | `RATE_LIMIT_EMAIL_SEND` | 100/min/user |
| Chat message | `RATE_LIMIT_CHAT_MESSAGE` | 300/min/user |
| File upload | `RATE_LIMIT_FILE_UPLOAD` | 50/min/user |
| Search query | `RATE_LIMIT_SEARCH` | 100/min/user |
| API general | `RATE_LIMIT_API_GENERAL` | 5000/min/tenant |

---

*For deployment details, see [25-Deployment-Pipeline.md](./25-Deployment-Pipeline.md). For security configuration, see [16-Security-Architecture.md](./16-Security-Architecture.md).*
