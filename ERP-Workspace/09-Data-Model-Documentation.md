# ERP-Workspace Data Model Documentation

> **Document ID:** ERP-WS-DM-009
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## 1. Data Model Overview

ERP-Workspace uses PostgreSQL 16 as the primary relational database, with 85+ tables organized across 11 DDD bounded contexts. The schema supports multi-tenant isolation via `tenant_id` foreign keys and Row-Level Security (RLS) policies. The data model uses UUID primary keys, TIMESTAMPTZ for all temporal fields, JSONB for semi-structured data, and TEXT arrays for tags and multi-value fields.

---

## 2. Table Inventory by Bounded Context

### 2.1 Tenancy Context (3 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| tenants | Tenant organizations with plan and branding | 1K |
| domains | Email domains with DNS verification records | 5K |
| provisioning_events | Provisioning workflow audit trail | 50K |

### 2.2 Email Core Context (12 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| mailboxes | User email accounts with quotas | 100K |
| email_messages | Sent/received email tracking | 100M |
| email_events | Delivery events (open, click, bounce) | 500M |
| email_templates | Reusable email templates with versioning | 10K |
| email_campaigns | Marketing campaign management | 5K |
| email_suppressions | Bounce/complaint suppression list | 500K |
| email_provider_configs | Multi-provider routing configuration | 1K |
| email_ab_tests | A/B test variants and results | 2K |
| email_validations | Email address validation cache | 1M |
| ip_pools | Dedicated IP pool management | 100 |
| webhook_subscriptions | Event webhook endpoints | 5K |
| webhook_deliveries | Webhook delivery attempts | 50K |
| audit_logs | Email audit trail | 10M |

### 2.3 Contacts Context (7 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| contacts | Contact directory with metadata | 5M |
| contact_emails | Contact email addresses (multi-valued) | 8M |
| contact_phones | Contact phone numbers | 5M |
| contact_addresses | Contact postal addresses | 3M |
| contact_labels | Contact tag associations | 10M |
| contact_groups | Named contact groups | 100K |
| contact_group_members | Group membership join table | 1M |

### 2.4 Calendar Context (6 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| calendars | Calendar containers per user/team | 500K |
| calendar_events | Scheduled events with recurrence | 10M |
| calendar_event_attendees | Event RSVP tracking | 50M |
| calendar_event_reminders | Configurable reminders | 20M |
| meeting_rooms | Bookable room inventory | 10K |
| meeting_room_bookings | Room reservation records | 1M |

### 2.5 Tasks Context (5 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| task_boards | Kanban/list boards | 100K |
| task_board_members | Board collaboration access | 500K |
| task_items | Individual task records | 5M |
| task_comments | Task discussion threads | 10M |
| task_attachments | Task file attachments | 2M |

### 2.6 Storage Context (5 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| document_libraries | Team/personal file collections | 200K |
| file_items | Files and folders hierarchy | 50M |
| file_versions | File version history | 100M |
| file_shares | Per-user file permissions | 10M |
| share_links | Token-based share URLs | 5M |

### 2.7 Chat Context (6 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| teams | Organizational team groupings | 50K |
| team_members | Team membership | 500K |
| conversations | Chat channels and DMs | 1M |
| conversation_members | Conversation participants | 5M |
| chat_messages | Individual chat messages | 500M |
| message_reactions | Emoji reactions on messages | 50M |
| message_read_receipts | Read tracking per user/message | 1B |

### 2.8 Knowledge Base Context (9 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| notes | Personal/linked notes | 5M |
| wikis | Collaborative knowledge sites | 50K |
| wiki_members | Wiki access control | 200K |
| wiki_navigation_items | Sidebar/tree structure | 500K |
| wiki_pages | Individual wiki pages | 1M |
| wiki_page_blocks | Block-based page content | 10M |
| wiki_page_versions | Page version history | 5M |
| wiki_page_comments | Page discussions | 2M |
| custom_lists | User-defined structured lists | 100K |

### 2.9 Collaboration Context (7 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| distribution_groups | Mailing list groups | 10K |
| distribution_group_members | Group membership | 500K |
| public_folders | Shared organizational folders | 50K |
| public_folder_acl | Folder access control entries | 200K |
| collaboration_sessions | Real-time co-editing sessions | 1M |
| collaboration_participants | Session participants with cursor state | 5M |
| collaboration_operations | Append-only edit operation log | 100M |

### 2.10 AI / Search / Analytics Context (7 tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| ai_compose_cache | Smart compose suggestion cache | 10M |
| ai_email_classifications | Email category/sentiment labels | 100M |
| ai_smart_replies | Pre-generated reply suggestions | 50M |
| ai_summaries | Meeting/document/thread summaries | 5M |
| search_index_metadata | Search index tracking | 50M |
| search_query_log | Query analytics | 100M |
| analytics_reports | Persisted analytics snapshots | 100K |
| analytics_anomaly_alerts | Anomaly detection alerts | 50K |
| email_event_staging | ClickHouse sync staging | 500M |

### 2.11 Privacy & Innovation Context (15+ tables)

| Table | Purpose | Row Estimate |
|-------|---------|-------------|
| email_action_extractions | AI-extracted action items | 10M |
| email_behavior_patterns | Communication patterns per contact | 5M |
| email_follow_up_predictions | Follow-up reminders | 10M |
| email_triage_signals | User behavior signals for ML | 100M |
| email_triage_model | Per-user triage ML model weights | 100K |
| email_sentiment_timeline | Sentiment tracking per contact | 50M |
| email_sentiment_alerts | Sentiment degradation alerts | 1M |
| knowledge_graph_nodes | Entity nodes (person, org, topic) | 10M |
| knowledge_graph_edges | Entity relationships | 50M |
| knowledge_graph_mentions | Entity-email associations | 100M |
| privacy_pii_detections | PII detection results | 10M |
| privacy_compliance_scores | Per-email compliance scores | 100M |
| privacy_policies | Tenant DLP policy configuration | 1K |
| email_thread_analysis | Thread topic/drift analysis | 5M |
| email_digest_config | Digest mode user preferences | 100K |
| email_digests | Generated digest bundles | 1M |
| email_digest_held | Emails held for digest delivery | 50M |
| email_health_scores | Domain health assessments | 10K |
| email_health_history | Health score time series | 500K |
| collaborative_drafts | Multi-author email drafts | 500K |
| collaborative_draft_comments | Draft review comments | 2M |
| collaborative_draft_versions | Draft version history | 1M |
| canned_responses | Quick reply templates | 500K |
| mailbox_rules | User-defined email rules | 1M |
| shared_mailbox_members | Shared mailbox access | 50K |
| contact_group_shares | Group sharing permissions | 100K |
| user_preferences | Per-user UI/notification settings | 100K |
| translation_overrides | Custom i18n translations | 50K |

---

## 3. Key Design Patterns

### 3.1 Multi-Tenant Isolation

Every business table includes a `tenant_id UUID NOT NULL` column. Row-Level Security (RLS) policies enforce that queries only return rows matching the authenticated tenant. The `X-Tenant-ID` header is extracted by the API gateway and injected into the database session via `SET app.current_tenant = '...'`.

### 3.2 Soft Deletes

Most entities use soft deletes via `is_deleted BOOLEAN DEFAULT FALSE` or `is_trashed BOOLEAN DEFAULT FALSE` with corresponding `deleted_at`/`trashed_at` timestamps. Partial indexes exclude soft-deleted rows for query efficiency.

### 3.3 Optimistic Concurrency

Entities that support concurrent editing include a `version INTEGER DEFAULT 0` column. Updates increment the version and include a `WHERE version = ?` condition to detect conflicts.

### 3.4 JSONB for Semi-Structured Data

Configuration, metadata, and variable-schema fields use JSONB columns. Examples: `recurrence_rule`, `booking_policy`, `checklist`, `feature_weights`, `branding`. This avoids EAV anti-patterns while maintaining queryability via GIN indexes.

### 3.5 Time-Series Optimization

High-volume append-only tables (`email_events`, `chat_messages`, `collaboration_operations`) use BRIN indexes on timestamp columns for space-efficient time-range queries. Covering indexes with INCLUDE clauses avoid heap lookups for common query patterns.

---

*For visual ER diagrams, see [10-Entity-Relationship-Diagram.md](./10-Entity-Relationship-Diagram.md).*
