# ERP-Marketing -- Entity Relationship Diagram

## 1. Core Entity Relationships

```mermaid
erDiagram
    audiences ||--o{ campaigns : "targeted_by"
    email_templates ||--o{ campaigns : "uses"
    campaigns {
        uuid id PK
        varchar name
        varchar subject
        varchar channel
        varchar status
        uuid audience_id FK
        uuid template_id FK
        varchar objective
        float budget
        int expected_reach
        varchar owner
        timestamptz scheduled_at
        timestamptz sent_at
        jsonb stats
        timestamptz created_at
    }
    audiences {
        uuid id PK
        varchar name
        text description
        jsonb filters
        int member_count
        timestamptz created_at
    }
    email_templates {
        uuid id PK
        varchar name
        varchar subject
        text html_content
        text text_content
        timestamptz created_at
    }
```

## 2. Contact and Lifecycle Entities

```mermaid
erDiagram
    marketing_contacts ||--o{ marketing_touchpoints : "generates"
    marketing_contacts ||--o{ marketing_form_submissions : "submits"
    marketing_contacts ||--o{ marketing_sequence_enrollments : "enrolled_in"
    marketing_contacts ||--o{ marketing_meetings : "attends"
    marketing_contacts ||--o{ marketing_tickets : "creates"
    marketing_contacts ||--o{ marketing_conversations : "participates"
    marketing_contacts {
        uuid id PK
        varchar email UK
        varchar first_name
        varchar last_name
        varchar company
        varchar job_title
        varchar lifecycle_stage
        int lead_score
        varchar consent_status
        jsonb tags
        jsonb traits
        timestamptz last_activity_at
        timestamptz created_at
        timestamptz updated_at
    }
    marketing_segments {
        uuid id PK
        varchar name
        text description
        jsonb rule_json
        int estimated_size
        varchar status
        timestamptz created_at
        timestamptz updated_at
    }
```

## 3. Journey and Automation Entities

```mermaid
erDiagram
    marketing_segments ||--o{ marketing_journeys : "entry_segment"
    marketing_journeys ||--o{ marketing_journey_steps : "contains"
    email_templates ||--o{ marketing_journey_steps : "uses"
    marketing_journeys ||--o{ marketing_touchpoints : "tracks"
    marketing_journeys {
        uuid id PK
        varchar name
        varchar goal
        varchar status
        uuid entry_segment_id FK
        jsonb channels
        jsonb settings
        varchar created_by
        timestamptz created_at
        timestamptz updated_at
    }
    marketing_journey_steps {
        uuid id PK
        uuid journey_id FK
        int position
        varchar step_type
        varchar channel
        uuid template_id FK
        int wait_minutes
        jsonb condition_json
        timestamptz created_at
    }
```

## 4. Attribution and Touchpoint Entities

```mermaid
erDiagram
    marketing_contacts ||--o{ marketing_touchpoints : "has"
    campaigns ||--o{ marketing_touchpoints : "attributed_to"
    marketing_journeys ||--o{ marketing_touchpoints : "attributed_to"
    marketing_touchpoints {
        uuid id PK
        uuid contact_id FK
        uuid campaign_id FK
        uuid journey_id FK
        varchar channel
        varchar event_type
        real attribution_weight
        jsonb metadata
        timestamptz occurred_at
    }
```

## 5. Experimentation Entities

```mermaid
erDiagram
    campaigns ||--o{ marketing_experiments : "tests"
    marketing_experiments {
        uuid id PK
        varchar name
        uuid campaign_id FK
        varchar status
        text hypothesis
        jsonb variants
        varchar winner_variant
        timestamptz started_at
        timestamptz ended_at
        jsonb results
        timestamptz created_at
    }
```

## 6. Account and Pipeline Entities

```mermaid
erDiagram
    marketing_accounts ||--o{ marketing_opportunities : "owns"
    marketing_contacts ||--o{ marketing_opportunities : "primary_contact"
    marketing_accounts ||--o{ marketing_tickets : "raises"
    marketing_opportunities ||--o{ marketing_meetings : "related_to"
    marketing_accounts {
        uuid id PK
        varchar name
        varchar industry
        varchar owner
        float arr
        int health_score
        float expansion_potential
        timestamptz created_at
        timestamptz updated_at
    }
    marketing_opportunities {
        uuid id PK
        uuid account_id FK
        uuid primary_contact_id FK
        varchar name
        varchar stage
        varchar status
        float amount
        int probability
        varchar source
        date close_date
        varchar owner
        text next_step
        timestamptz created_at
        timestamptz updated_at
    }
```

## 7. Social, Ads, and Content Entities

```mermaid
erDiagram
    campaigns ||--o{ marketing_social_posts : "promotes"
    audiences ||--o{ marketing_ads : "targets"
    marketing_ads {
        uuid id PK
        varchar name
        varchar network
        varchar objective
        varchar status
        float budget
        float spend
        int impressions
        int clicks
        int conversions
        uuid audience_id FK
        varchar owner
        timestamptz created_at
        timestamptz updated_at
    }
    marketing_social_posts {
        uuid id PK
        varchar channel
        varchar title
        text content
        varchar status
        timestamptz scheduled_at
        timestamptz published_at
        uuid campaign_id FK
        varchar owner
        jsonb engagement_json
        timestamptz created_at
        timestamptz updated_at
    }
    marketing_content_assets {
        uuid id PK
        varchar asset_type
        varchar title
        varchar slug UK
        varchar status
        text body
        jsonb seo_keywords
        jsonb cta_json
        varchar owner
        timestamptz created_at
        timestamptz updated_at
    }
```

## 8. Service and Operations Entities

```mermaid
erDiagram
    marketing_tickets ||--o{ marketing_conversations : "contains"
    marketing_sequences ||--o{ marketing_sequence_enrollments : "enrolls"
    marketing_contacts ||--o{ marketing_sequence_enrollments : "enrolled"
    marketing_tickets {
        uuid id PK
        uuid account_id FK
        uuid contact_id FK
        varchar subject
        varchar priority
        varchar status
        varchar pipeline
        varchar owner
        timestamptz sla_due_at
        text resolution_notes
        timestamptz created_at
        timestamptz updated_at
    }
    marketing_sequences {
        uuid id PK
        varchar name
        varchar status
        varchar trigger_type
        jsonb steps_json
        int enrollments
        varchar owner
        timestamptz created_at
        timestamptz updated_at
    }
```

## 9. Governance Entities

```mermaid
erDiagram
    marketing_scoring_models {
        uuid id PK
        varchar name
        varchar status
        jsonb rules_json
        text threshold_sql
        timestamptz created_at
        timestamptz updated_at
    }
    marketing_aidd_guardrail_events {
        uuid id PK
        varchar entity_type
        uuid entity_id
        varchar action
        real confidence
        int blast_radius
        float monetary_value
        varchar risk_level
        varchar decision
        varchar approved_by
        text rationale
        jsonb payload
        timestamptz created_at
    }
```

## 10. Complete Table Summary

| Table | Records (Seed) | Primary Relationships |
|---|---|---|
| audiences | 2 | campaigns, ads |
| email_templates | 2 | campaigns, journey_steps |
| campaigns | 2 | audiences, templates, touchpoints, experiments, social_posts |
| marketing_contacts | 3 | touchpoints, submissions, enrollments, meetings, tickets |
| marketing_segments | 2 | journeys |
| marketing_journeys | 2 | segments, steps, touchpoints |
| marketing_journey_steps | 3 | journeys, templates |
| marketing_touchpoints | 2 | contacts, campaigns, journeys |
| marketing_forms | 1 | submissions |
| marketing_form_submissions | 0 | forms, contacts |
| marketing_experiments | 1 | campaigns |
| marketing_scoring_models | 1 | standalone |
| marketing_accounts | 2 | opportunities, tickets |
| marketing_opportunities | 2 | accounts, contacts, meetings |
| marketing_tasks | 2 | any entity (polymorphic) |
| marketing_ads | 2 | audiences |
| marketing_social_posts | 2 | campaigns |
| marketing_content_assets | 2 | standalone |
| marketing_sequences | 2 | enrollments |
| marketing_sequence_enrollments | 1 | sequences, contacts |
| marketing_meetings | 2 | contacts, opportunities |
| marketing_tickets | 2 | accounts, contacts, conversations |
| marketing_conversations | 2 | tickets, contacts |
| marketing_knowledge_articles | 2 | standalone |
| marketing_playbooks | 2 | standalone |
| marketing_data_sync_jobs | 2 | standalone |
| marketing_aidd_guardrail_events | 0+ | any entity (polymorphic) |
