# Release Notes -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## Release 1.0.0 -- Foundation Release

**Release Date**: 2026-02-23
**Release Type**: Major
**Compatibility**: ERP-Platform 1.x, ERP-IAM 1.x

---

### What's New

#### 1. Microservices Architecture
- 12 domain microservices decomposed from the source monolith
- Go API gateway with JWT validation, tenant isolation, and entitlement checking
- Redpanda/Kafka event streaming for inter-service communication
- Docker Compose orchestration for local development

#### 2. Member Service
- Full CRUD operations for member records
- Auto-generated membership IDs (MEM000001 format)
- Member categories: New Believer, Member, Prospect, Worker, Minister, Pastor
- Natural Group segmentation: Youth, Men, Women, Elders, Teens, Children
- Family unit (Household) management
- Communication preferences per member
- Absentee detection (configurable threshold, default 3 weeks)
- Member statistics by type and natural group

#### 3. Visitor Service
- Visitor registration with multi-channel contact details
- First-timer tracking and identification
- 72-hour follow-up protocol with automated reminders (hourly cron)
- Visitor-to-member conversion with full data migration
- Welcome gift tracking and distribution logging
- 72-hour completion rate reporting

#### 4. Follow-up Service
- Account Officer for Souls assignment (auto and manual)
- 6-Directorate workflow: 1st Timer, Further Follow-up, Counselling, Natural Group, Development & Structuring, Welfare & Finance
- Follow-up activity recording with type, status, notes
- 4Cs Assimilation Workflow (Connect, Capture, Communicate, Convert)
- Directorate routing and escalation

#### 5. Giving Service
- Giving types: Tithes, Offerings, Donations, Special Seeds, Building Fund
- Pledge campaign management with fulfillment tracking
- Tax-deductible receipt generation
- Annual giving statement generation (PDF)
- Integration with ERP-Finance via Kafka events

#### 6. Event Service
- Event management with types: Sunday Service, Midweek, Outreach, Come and See, Go and Tell
- Attendance tracking per event per member
- QR code and NFC check-in support
- Service planning capabilities
- Recurring event support

#### 7. Group Service
- Small group management
- Home fellowship management
- Cell group management with multiplication tracking
- Ministry team management
- Group leader and co-leader assignment

#### 8. Discipleship Service
- New Believer Class (NBC) enrollment and tracking
- Mentorship program (90-120 day pairs)
- Sunday School management
- Progress tracking with milestones

#### 9. Welfare Service
- Welfare case management with full lifecycle
- Case categories: Financial, Medical, Housing, Food, Education, Emergency
- Urgency levels: Low, Medium, High, Critical
- Fund request approval workflow
- Disbursement tracking

#### 10. Communication Service
- 7-channel messaging: SMS (Twilio), WhatsApp Business API, Telegram Bot API, Facebook Messenger, Email (SMTP), Push notifications, In-app notifications
- Unified messaging service with channel fallback
- Message template system
- Delivery tracking and reporting
- Scheduled messaging

#### 11. KPI Service
- Quarterly Shepherding KPIs:
  - 72-hour contact completion rate (target: 90%)
  - NBC enrollment rate (target: 100%)
  - Mentorship completion rate (target: 85%)
  - Visitor conversion rate (target: 60%)
  - Welfare cases fulfilled (target: 20/month)
- Daily automated calculation (midnight cron)
- Traffic-light status indicators (Achieved, On Track, Behind)
- Directorate-specific KPI views

#### 12. Volunteer Service
- Volunteer profile management with skills and availability
- Shift scheduling and skill-based matching
- Team management
- Hours served tracking

#### 13. Facility Service
- Room and facility booking with conflict detection
- Equipment and asset management
- Campus management
- Booking rules configuration

#### 14. API Gateway
- Go-based reverse proxy gateway
- JWT Bearer token validation
- X-Tenant-ID enforcement for multi-campus isolation
- ERP-Platform entitlement verification
- Correlation ID generation and forwarding
- Graceful degradation when ERP-Platform unavailable

---

### Known Issues

| # | Issue | Severity | Workaround |
|---|---|---|---|
| 1 | JWT validation checks token length only, not signature | High | Pending JWKS integration with ERP-IAM |
| 2 | No rate limiting on gateway | Medium | Monitor traffic; implement in next release |
| 3 | KPI calculator does not include Sunday School enrollment KPI | Low | Manual tracking until next release |
| 4 | Socket.IO legacy not yet replaced with SSE | Low | Monolith Socket.IO still functional |

---

### Migration Notes

- **From source monolith**: The 79 Sequelize models are consolidated into 17 PostgreSQL tables
- **Data migration**: Run `/database/migrations/0001_initial_core.sql` before starting services
- **Environment variables**: See `.env.example` for required configuration
- **Breaking changes from monolith API**: URL paths changed from `/api/v1/{resource}` to `/v1/{service}/{resource}`

---

### Upgrade Path

This is the initial release. Future upgrades will follow semantic versioning:
- **Patch (1.0.x)**: Bug fixes, security patches
- **Minor (1.x.0)**: New features, non-breaking API changes
- **Major (x.0.0)**: Breaking API changes, architecture changes

---

### Contributors

- Architecture: AIDD System
- Source Monolith: RCCG Development Team
- Microservices Migration: BillyRonks Engineering
