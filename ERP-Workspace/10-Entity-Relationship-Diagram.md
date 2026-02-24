# ERP-Workspace Entity-Relationship Diagram

> **Document ID:** ERP-WS-ERD-010
> **Version:** 1.0.0
> **Last Updated:** 2026-02-23
> **Status:** Approved

---

## 1. Master ERD Overview

The full ERP-Workspace schema spans 85+ tables across 11 bounded contexts. This document presents ER diagrams for each context.

---

## 2. Tenancy & Email Core Context

```mermaid
erDiagram
    tenants {
        uuid tenant_id PK
        text name
        text primary_domain
        text plan_tier
        text status
        jsonb branding
        jsonb locale_settings
        timestamptz created_at
    }

    domains {
        uuid domain_id PK
        uuid tenant_id FK
        varchar domain_name
        text dmarc_policy
        text spf_record
        varchar dkim_selector
    }

    mailboxes {
        uuid mailbox_id PK
        uuid tenant_id FK
        text user_principal
        integer quota_mb
        boolean active
        timestamptz created_at
    }

    email_messages {
        uuid message_id PK
        uuid tenant_id FK
        uuid campaign_id FK
        uuid template_id FK
        varchar from_email
        varchar to_email
        text subject
        varchar status
        varchar provider
        timestamptz sent_at
        timestamptz snoozed_until
        jsonb labels
        boolean is_focused
    }

    email_events {
        uuid event_id PK
        uuid message_id FK
        uuid tenant_id FK
        varchar event_type
        varchar recipient
        timestamptz timestamp
        jsonb metadata
    }

    email_templates {
        uuid template_id PK
        uuid tenant_id FK
        varchar name
        text subject
        text html_body
        text text_body
        varchar language
        integer version
        jsonb variables
    }

    email_campaigns {
        uuid campaign_id PK
        uuid tenant_id FK
        varchar name
        uuid template_id FK
        varchar status
        timestamptz scheduled_at
        integer total_recipients
        integer sent_count
        integer opened_count
    }

    email_suppressions {
        uuid suppression_id PK
        uuid tenant_id FK
        varchar email
        varchar reason
        varchar bounce_type
    }

    email_provider_configs {
        uuid config_id PK
        uuid tenant_id FK
        varchar provider_type
        jsonb config
        boolean is_primary
        integer priority
    }

    email_ab_tests {
        uuid test_id PK
        uuid campaign_id FK
        uuid tenant_id FK
        varchar test_name
        jsonb variants
        varchar winner_variant
    }

    ip_pools {
        uuid pool_id PK
        uuid tenant_id FK
        varchar name
        text pool_type
    }

    email_validations {
        uuid validation_id PK
        varchar email
        boolean valid
        decimal score
    }

    webhook_subscriptions {
        uuid subscription_id PK
        uuid tenant_id FK
        text url
        text[] events
        varchar secret
        boolean enabled
    }

    webhook_deliveries {
        uuid delivery_id PK
        uuid subscription_id FK
        uuid event_id
        varchar status
        integer attempt
    }

    audit_logs {
        bigserial log_id PK
        uuid tenant_id FK
        varchar actor
        varchar action
        varchar target
        jsonb metadata
    }

    tenants ||--o{ domains : "has"
    tenants ||--o{ mailboxes : "provisions"
    tenants ||--o{ email_messages : "contains"
    tenants ||--o{ email_templates : "owns"
    tenants ||--o{ email_campaigns : "runs"
    tenants ||--o{ email_suppressions : "maintains"
    tenants ||--o{ email_provider_configs : "configures"
    tenants ||--o{ webhook_subscriptions : "registers"
    tenants ||--o{ audit_logs : "generates"
    email_messages ||--o{ email_events : "produces"
    email_templates ||--o{ email_messages : "renders"
    email_campaigns ||--o{ email_messages : "sends"
    email_campaigns ||--o{ email_ab_tests : "tests"
    webhook_subscriptions ||--o{ webhook_deliveries : "dispatches"
```

---

## 3. Contacts Context

```mermaid
erDiagram
    contacts {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        varchar first_name
        varchar last_name
        varchar display_name
        varchar company
        varchar job_title
        varchar department
        text notes
        boolean is_favorite
        boolean is_deleted
        timestamptz last_contacted_at
        integer version
    }

    contact_emails {
        uuid id PK
        uuid contact_id FK
        varchar address
        varchar label
        boolean is_primary
    }

    contact_phones {
        uuid id PK
        uuid contact_id FK
        varchar number
        varchar phone_type
        boolean is_primary
    }

    contact_addresses {
        uuid id PK
        uuid contact_id FK
        varchar street
        varchar city
        varchar state
        varchar postal_code
        varchar country
        varchar address_type
    }

    contact_labels {
        uuid contact_id FK
        varchar label
    }

    contact_groups {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        varchar name
        varchar group_type
    }

    contact_group_members {
        uuid group_id FK
        uuid contact_id FK
        timestamptz added_at
    }

    contact_group_shares {
        uuid group_id FK
        uuid user_id
        timestamptz shared_at
    }

    canned_responses {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        varchar name
        text body_html
        varchar shortcut
        varchar category
        boolean shared
    }

    contacts ||--o{ contact_emails : "has"
    contacts ||--o{ contact_phones : "has"
    contacts ||--o{ contact_addresses : "has"
    contacts ||--o{ contact_labels : "tagged"
    contact_groups ||--o{ contact_group_members : "contains"
    contact_groups ||--o{ contact_group_shares : "shared_with"
    contacts ||--o{ contact_group_members : "member_of"
```

---

## 4. Calendar & Tasks Context

```mermaid
erDiagram
    calendars {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        varchar name
        varchar calendar_type
        varchar color
        varchar visibility
        varchar timezone
        boolean is_default
    }

    calendar_shares {
        uuid id PK
        uuid calendar_id FK
        varchar user_id
        boolean can_edit
        boolean can_invite
    }

    calendar_events {
        uuid id PK
        uuid tenant_id
        uuid calendar_id FK
        uuid organizer_id
        varchar title
        text description
        timestamptz start_time
        timestamptz end_time
        boolean all_day
        varchar timezone
        jsonb recurrence_rule
        uuid room_id FK
        boolean is_cancelled
        boolean show_as_busy
    }

    calendar_event_attendees {
        uuid id PK
        uuid event_id FK
        varchar user_id
        varchar email
        varchar role
        varchar status
        timestamptz responded_at
    }

    calendar_event_reminders {
        uuid id PK
        uuid event_id FK
        varchar reminder_type
        integer minutes_before
    }

    meeting_rooms {
        uuid id PK
        uuid tenant_id
        varchar name
        varchar floor
        varchar building
        integer capacity
        text[] amenities
        jsonb booking_policy
        boolean is_active
    }

    meeting_room_bookings {
        uuid id PK
        uuid room_id FK
        uuid event_id FK
        timestamptz start_time
        timestamptz end_time
    }

    task_boards {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        varchar name
        varchar board_type
        jsonb columns
        boolean is_archived
    }

    task_items {
        uuid id PK
        uuid tenant_id
        uuid board_id FK
        varchar title
        varchar status
        varchar priority
        varchar column_name
        text[] assignee_ids
        timestamptz due_date
        jsonb checklist
    }

    task_comments {
        uuid id PK
        uuid task_id FK
        varchar author_id
        text content
    }

    task_attachments {
        uuid id PK
        uuid task_id FK
        varchar file_name
        varchar storage_key
        bigint file_size
    }

    calendars ||--o{ calendar_shares : "shared_with"
    calendars ||--o{ calendar_events : "contains"
    calendar_events ||--o{ calendar_event_attendees : "invites"
    calendar_events ||--o{ calendar_event_reminders : "has"
    calendar_events ||--o{ meeting_room_bookings : "books"
    meeting_rooms ||--o{ meeting_room_bookings : "reserved_by"
    task_boards ||--o{ task_items : "contains"
    task_items ||--o{ task_comments : "discussed"
    task_items ||--o{ task_attachments : "attached"
```

---

## 5. Storage & Chat Context

```mermaid
erDiagram
    document_libraries {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        varchar owner_type
        varchar name
        bigint quota_bytes
        bigint used_bytes
        boolean is_active
    }

    file_items {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        uuid library_id FK
        varchar name
        varchar file_type
        varchar content_type
        bigint file_size
        varchar storage_key
        uuid parent_id FK
        text path
        integer current_version
        boolean is_trashed
    }

    file_versions {
        uuid id PK
        uuid file_id FK
        integer version_number
        varchar storage_key
        bigint file_size
        varchar created_by
    }

    file_shares {
        uuid id PK
        uuid file_id FK
        varchar user_id
        varchar permission
        varchar shared_by
    }

    share_links {
        uuid id PK
        uuid file_id FK
        varchar token
        varchar permission
        timestamptz expires_at
        integer max_downloads
        integer download_count
    }

    teams {
        uuid id PK
        uuid tenant_id
        varchar name
        text description
        boolean is_public
        boolean allow_guest_access
        boolean is_archived
    }

    team_members {
        uuid id PK
        uuid team_id FK
        varchar user_id
        varchar role
        varchar display_name
    }

    conversations {
        uuid id PK
        uuid tenant_id
        uuid team_id FK
        varchar name
        varchar conversation_type
        boolean is_archived
    }

    conversation_members {
        uuid id PK
        uuid conversation_id FK
        varchar user_id
        varchar role
        boolean is_muted
    }

    chat_messages {
        uuid id PK
        uuid tenant_id
        uuid conversation_id FK
        varchar sender_id
        text content_text
        varchar message_type
        varchar parent_message_id
        integer thread_reply_count
        text[] mentioned_user_ids
        boolean is_edited
        boolean is_deleted
    }

    message_reactions {
        uuid id PK
        uuid message_id FK
        varchar user_id
        varchar emoji
    }

    message_read_receipts {
        uuid id PK
        uuid message_id FK
        varchar user_id
        timestamptz read_at
    }

    document_libraries ||--o{ file_items : "contains"
    file_items ||--o{ file_versions : "versioned"
    file_items ||--o{ file_shares : "shared"
    file_items ||--o{ share_links : "linked"
    file_items ||--o{ file_items : "parent_child"
    teams ||--o{ team_members : "has"
    teams ||--o{ conversations : "organizes"
    conversations ||--o{ conversation_members : "has"
    conversations ||--o{ chat_messages : "contains"
    chat_messages ||--o{ message_reactions : "reacted"
    chat_messages ||--o{ message_read_receipts : "read_by"
```

---

## 6. Knowledge Base & Collaboration Context

```mermaid
erDiagram
    wikis {
        uuid id PK
        uuid tenant_id
        varchar name
        varchar site_type
        uuid home_page_id
        boolean is_public
        boolean is_archived
    }

    wiki_pages {
        uuid id PK
        uuid tenant_id
        uuid wiki_id FK
        varchar title
        varchar slug
        uuid parent_page_id FK
        integer current_version
        boolean is_published
    }

    wiki_page_blocks {
        uuid id PK
        uuid page_id FK
        varchar block_type
        text content
        integer sort_order
    }

    wiki_page_versions {
        uuid id PK
        uuid page_id FK
        integer version_number
        varchar edited_by
        varchar title
    }

    notes {
        uuid id PK
        uuid tenant_id
        uuid owner_id
        varchar title
        text content_html
        varchar color
        varchar folder
        boolean is_pinned
        boolean is_archived
    }

    distribution_groups {
        uuid id PK
        uuid tenant_id
        varchar name
        varchar email_address
        varchar moderation
        boolean allow_external_senders
    }

    distribution_group_members {
        uuid id PK
        uuid group_id FK
        varchar email
        varchar role
        varchar delivery_mode
    }

    collaboration_sessions {
        uuid id PK
        uuid file_id
        uuid tenant_id
        varchar status
        integer current_revision
        integer max_participants
    }

    collaboration_participants {
        uuid id PK
        uuid session_id FK
        varchar user_id
        integer cursor_position
        varchar color
        boolean is_active
    }

    collaboration_operations {
        uuid id PK
        uuid session_id FK
        varchar user_id
        varchar operation_type
        integer position
        text content
        integer revision
    }

    wikis ||--o{ wiki_pages : "contains"
    wiki_pages ||--o{ wiki_page_blocks : "composed_of"
    wiki_pages ||--o{ wiki_page_versions : "versioned"
    wiki_pages ||--o{ wiki_pages : "parent_child"
    distribution_groups ||--o{ distribution_group_members : "has"
    collaboration_sessions ||--o{ collaboration_participants : "joined_by"
    collaboration_sessions ||--o{ collaboration_operations : "records"
```

---

## 7. AI / Privacy / Innovation Context

```mermaid
erDiagram
    ai_email_classifications {
        uuid id PK
        uuid tenant_id
        uuid message_id
        varchar category
        varchar sentiment
        integer priority
        float confidence
    }

    ai_summaries {
        uuid id PK
        uuid tenant_id
        varchar source_type
        uuid source_id
        text summary
        jsonb key_points
        jsonb action_items
    }

    knowledge_graph_nodes {
        uuid id PK
        uuid tenant_id
        varchar node_type
        varchar name
        jsonb properties
        integer mention_count
    }

    knowledge_graph_edges {
        uuid id PK
        uuid tenant_id
        uuid source_node_id FK
        uuid target_node_id FK
        varchar relationship
        float weight
    }

    privacy_pii_detections {
        uuid id PK
        uuid tenant_id
        uuid email_id
        varchar pii_type
        float confidence
        boolean redacted
    }

    email_health_scores {
        uuid id PK
        uuid tenant_id
        varchar domain
        float overall_score
        varchar spf_status
        varchar dkim_status
        varchar dmarc_status
        float bounce_rate
    }

    email_action_extractions {
        uuid id PK
        uuid tenant_id
        uuid email_id
        varchar action_type
        varchar title
        float confidence
        varchar status
    }

    email_triage_model {
        uuid id PK
        uuid tenant_id
        uuid user_id
        integer model_version
        jsonb feature_weights
        float accuracy
    }

    knowledge_graph_nodes ||--o{ knowledge_graph_edges : "source"
    knowledge_graph_nodes ||--o{ knowledge_graph_edges : "target"
```

---

## 8. Index Strategy Summary

| Index Type | Count | Purpose |
|-----------|-------|---------|
| B-tree (standard) | 80+ | Equality and range lookups |
| Covering (INCLUDE) | 15+ | Avoid heap lookups for listing queries |
| BRIN (block range) | 3 | Space-efficient time-series filtering |
| Partial | 10+ | Filter on common predicates (is_deleted, status) |
| GIN | 5+ | JSONB and array containment queries |
| Unique | 30+ | Enforce business uniqueness constraints |

---

*For data flow details, see [08-Data-Flow-Diagrams.md](./08-Data-Flow-Diagrams.md). For data model documentation, see [09-Data-Model-Documentation.md](./09-Data-Model-Documentation.md).*
