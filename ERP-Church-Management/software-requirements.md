# Software Requirements -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Runtime Dependencies

### 1.1 Server-Side (Target Architecture)

| Software | Version | Purpose | License |
|---|---|---|---|
| Go | 1.22+ | Gateway and microservice runtime | BSD-3-Clause |
| PostgreSQL | 16.x | Primary relational database | PostgreSQL License |
| Redis | 7.x | Caching, sessions, rate limiting | BSD-3-Clause |
| Redpanda | 24.2+ | Event streaming (Kafka-compatible) | BSL / Community |
| Docker | 24+ | Container runtime | Apache 2.0 |
| Docker Compose | 2.x | Local development orchestration | Apache 2.0 |
| Kubernetes | 1.28+ | Production orchestration | Apache 2.0 |

### 1.2 Server-Side (Source Monolith / Legacy)

| Software | Version | Purpose | License |
|---|---|---|---|
| Node.js | 18+ (LTS) | Application runtime | MIT |
| Express.js | 4.18.x | HTTP framework | MIT |
| Sequelize | 6.35.x | ORM for PostgreSQL | MIT |
| Socket.IO | 4.6.x | Real-time WebSocket server | MIT |
| jsonwebtoken | 9.0.x | JWT creation and validation | MIT |
| bcryptjs | 2.4.x | Password hashing | MIT |
| helmet | 7.1.x | HTTP security headers | MIT |
| morgan | 1.10.x | HTTP request logging | MIT |
| winston | 3.11.x | Application logging | MIT |
| node-cron | 3.0.x | Scheduled job execution | ISC |
| nodemailer | 6.9.x | Email sending (SMTP) | MIT |
| twilio | 4.19.x | SMS delivery | MIT |
| axios | 1.6.x | HTTP client (external API calls) | MIT |
| express-validator | 7.0.x | Request validation | MIT |
| multer | 1.4.x | File upload handling | MIT |
| pg | 8.11.x | PostgreSQL driver | MIT |
| pg-hstore | 2.3.x | PostgreSQL hstore support | MIT |
| umzug | 3.8.x | Database migration runner | MIT |
| cors | 2.8.x | CORS middleware | MIT |
| dotenv | 16.3.x | Environment variable loading | BSD-2-Clause |

### 1.3 Development Dependencies

| Software | Version | Purpose |
|---|---|---|
| Jest | 29.7.x | Unit and integration testing |
| Supertest | 6.3.x | HTTP assertion library |
| Nodemon | 3.0.x | Development auto-restart |

---

## 2. Frontend Dependencies

### 2.1 Web Application (React/Next.js)

| Software | Version | Purpose |
|---|---|---|
| Next.js | 14+ | React framework with SSR/RSC |
| React | 18+ | UI component library |
| TypeScript | 5+ | Type-safe JavaScript |
| Tailwind CSS | 3+ | Utility-first CSS framework |
| Zustand / React Query | Latest | State management and data fetching |
| Socket.IO Client | 4.6.x | Real-time communication (legacy) |
| QR Code Scanner | Latest | Browser-based QR scanning |
| Chart.js / Recharts | Latest | Dashboard visualizations |
| React Hook Form | Latest | Form management |
| Zod | Latest | Schema validation |

### 2.2 Mobile Application (Flutter)

| Software | Version | Purpose |
|---|---|---|
| Flutter SDK | 3.x | Cross-platform mobile framework |
| Dart | 3.x | Programming language |
| Provider | Latest | State management |
| Dio | Latest | HTTP client |
| qr_code_scanner | Latest | QR code scanning |
| flutter_local_notifications | Latest | Push notifications |
| shared_preferences | Latest | Local storage |
| flutter_secure_storage | Latest | Secure credential storage |
| intl | Latest | Internationalization |
| cached_network_image | Latest | Image caching |

---

## 3. External Service Dependencies

### 3.1 Communication APIs

| Service | Purpose | Required Credentials |
|---|---|---|
| Twilio | SMS delivery | Account SID, Auth Token, Phone Number |
| WhatsApp Business API | WhatsApp messaging | API Key, Phone Number ID |
| Telegram Bot API | Telegram messaging | Bot Token |
| Facebook Messenger API | Facebook messaging | Page Access Token, App Secret |
| SMTP Server (Gmail/SendGrid) | Email delivery | Host, Port, User, Password |

### 3.2 ERP Suite Dependencies

| Service | Purpose | Integration |
|---|---|---|
| ERP-IAM | Authentication (OIDC/JWT) | JWKS endpoint for token validation |
| ERP-Platform | Entitlements and configuration | REST API `/v1/entitlements/{tenant_id}` |
| ERP-Finance | Giving journal entries | Kafka event consumption |
| ERP-BI | Analytics data warehouse | Kafka event consumption |

---

## 4. Infrastructure Software

### 4.1 CI/CD Pipeline

| Tool | Purpose |
|---|---|
| GitHub Actions | CI/CD pipeline execution |
| Docker BuildKit | Multi-stage container builds |
| Trivy | Container vulnerability scanning |
| ESLint / golangci-lint | Code linting |
| Jest / go test | Unit testing |

### 4.2 Monitoring Stack

| Tool | Version | Purpose |
|---|---|---|
| Prometheus | 2.x | Metrics collection |
| Grafana | 10+ | Dashboards and alerting |
| Loki | 2.x | Log aggregation |
| Jaeger | 1.x | Distributed tracing |
| OpenTelemetry Collector | Latest | Telemetry pipeline |

### 4.3 Security Tools

| Tool | Purpose |
|---|---|
| cert-manager | TLS certificate management (Let's Encrypt) |
| External Secrets Operator | Secret injection from cloud providers |
| Falco | Runtime security monitoring |
| OPA/Gatekeeper | Kubernetes policy enforcement |

---

## 5. Environment Configuration

### 5.1 Required Environment Variables

```
# Server
PORT=8080
NODE_ENV=production
API_VERSION=v1

# Database
DATABASE_URL=postgres://user:password@host:5432/erp_church_management
DB_POOL_MIN=5
DB_POOL_MAX=25

# Cache
REDIS_ADDR=redis:6379
REDIS_PASSWORD=<secret>

# Event Streaming
KAFKA_BROKERS=broker1:9092,broker2:9092,broker3:9092

# Authentication
JWT_SECRET=<secret>
JWT_EXPIRE=7d
ERP_IAM_JWKS_URL=https://iam.erp.example.com/.well-known/jwks.json

# ERP Integration
ERP_PLATFORM_BASE_URL=http://erp-platform:8091
ALLOW_ON_ENTITLEMENT_FAILURE=false

# Communication
TWILIO_ACCOUNT_SID=<secret>
TWILIO_AUTH_TOKEN=<secret>
TWILIO_PHONE_NUMBER=+1234567890
WHATSAPP_API_KEY=<secret>
WHATSAPP_PHONE_NUMBER_ID=<secret>
TELEGRAM_BOT_TOKEN=<secret>
FACEBOOK_PAGE_ACCESS_TOKEN=<secret>
FACEBOOK_APP_SECRET=<secret>
EMAIL_HOST=smtp.sendgrid.net
EMAIL_PORT=587
EMAIL_USER=<secret>
EMAIL_PASSWORD=<secret>

# Church Configuration
CHURCH_NAME=Church Name
CHURCH_EMAIL=info@church.org
FOLLOWUP_DEADLINE_HOURS=72
ENABLE_CRON_JOBS=true
```

---

## 6. Compatibility Matrix

| Component | Minimum | Recommended | Maximum Tested |
|---|---|---|---|
| Node.js | 18.0 | 20.x LTS | 22.x |
| Go | 1.22 | 1.23 | 1.24 |
| PostgreSQL | 14 | 16 | 17 |
| Redis | 6.2 | 7.x | 7.4 |
| Docker | 20.10 | 24+ | 27 |
| Kubernetes | 1.26 | 1.28+ | 1.31 |
| Flutter | 3.16 | 3.24+ | Latest stable |
| React/Next.js | 14.0 | 14.2+ | 15.x |

---

## 7. License Compliance

All software dependencies use permissive open-source licenses (MIT, BSD, Apache 2.0, ISC) compatible with commercial distribution. Redpanda Community Edition uses the Business Source License (BSL) which permits free use for non-competing purposes. For production deployments exceeding BSL limits, a Redpanda Enterprise license or migration to Apache Kafka is recommended.
