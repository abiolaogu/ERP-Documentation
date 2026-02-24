# ERP-CRM Entity-Relationship Diagram

## Complete ER Diagram

```mermaid
erDiagram
    contacts {
        UUID id PK
        VARCHAR email "NOT NULL, indexed"
        VARCHAR first_name
        VARCHAR last_name
        VARCHAR phone
        UUID company_id FK "nullable"
        INTEGER lead_score "DEFAULT 0"
        VARCHAR lifecycle_stage "DEFAULT subscriber"
        VARCHAR source
        UUID owner_id
        TEXT_ARRAY tags "DEFAULT {}"
        JSONB custom_fields "DEFAULT {}"
        TIMESTAMPTZ created_at "NOT NULL"
        TIMESTAMPTZ updated_at "NOT NULL"
    }

    companies {
        UUID id PK
        VARCHAR name "NOT NULL, indexed"
        VARCHAR domain "indexed"
        VARCHAR industry
        VARCHAR size
        VARCHAR website
        VARCHAR phone
        JSONB address
        UUID owner_id
        JSONB custom_fields "DEFAULT {}"
        TIMESTAMPTZ created_at "NOT NULL"
        TIMESTAMPTZ updated_at "NOT NULL"
    }

    pipelines {
        UUID id PK
        VARCHAR name "NOT NULL"
        TEXT description
        BOOLEAN is_default "DEFAULT FALSE"
        TIMESTAMPTZ created_at "NOT NULL"
    }

    pipeline_stages {
        UUID id PK
        UUID pipeline_id FK "NOT NULL"
        VARCHAR name "NOT NULL"
        INTEGER position "NOT NULL"
        INTEGER probability "DEFAULT 0"
        TIMESTAMPTZ created_at "NOT NULL"
    }

    deals {
        UUID id PK
        VARCHAR name "NOT NULL"
        UUID contact_id FK "nullable"
        UUID company_id FK "nullable"
        UUID pipeline_id FK "NOT NULL"
        UUID stage_id FK "NOT NULL"
        BIGINT amount
        VARCHAR currency "DEFAULT NGN"
        INTEGER probability
        TIMESTAMPTZ expected_close_date
        UUID owner_id
        JSONB custom_fields "DEFAULT {}"
        TIMESTAMPTZ created_at "NOT NULL"
        TIMESTAMPTZ updated_at "NOT NULL"
        TIMESTAMPTZ closed_at
    }

    activities {
        UUID id PK
        VARCHAR activity_type "NOT NULL"
        VARCHAR subject "NOT NULL"
        TEXT description
        UUID contact_id FK "nullable"
        UUID company_id FK "nullable"
        UUID deal_id FK "nullable"
        UUID owner_id
        TIMESTAMPTZ due_date
        TIMESTAMPTZ completed_at
        TIMESTAMPTZ created_at "NOT NULL"
    }

    notes {
        UUID id PK
        TEXT content "NOT NULL"
        UUID contact_id FK "nullable"
        UUID company_id FK "nullable"
        UUID deal_id FK "nullable"
        UUID created_by
        TIMESTAMPTZ created_at "NOT NULL"
        TIMESTAMPTZ updated_at "NOT NULL"
    }

    tickets {
        UUID id PK
        VARCHAR ticket_number "UNIQUE, NOT NULL"
        VARCHAR subject "NOT NULL"
        TEXT description "NOT NULL"
        VARCHAR customer_email "NOT NULL, indexed"
        VARCHAR customer_name
        VARCHAR priority "DEFAULT medium"
        VARCHAR status "DEFAULT open, indexed"
        VARCHAR category
        UUID assignee_id
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
        TIMESTAMPTZ resolved_at
    }

    ticket_comments {
        UUID id PK
        UUID ticket_id FK "NOT NULL"
        VARCHAR author_type "NOT NULL"
        UUID author_id
        VARCHAR author_email "NOT NULL"
        TEXT content "NOT NULL"
        BOOLEAN is_internal "DEFAULT FALSE"
        TIMESTAMPTZ created_at
    }

    kb_categories {
        UUID id PK
        VARCHAR name "NOT NULL"
        VARCHAR slug "UNIQUE, NOT NULL"
        TEXT description
        TIMESTAMPTZ created_at
    }

    kb_articles {
        UUID id PK
        VARCHAR title "NOT NULL"
        VARCHAR slug "UNIQUE, NOT NULL"
        TEXT content "NOT NULL"
        UUID category_id FK "nullable"
        VARCHAR status "DEFAULT draft"
        INTEGER view_count "DEFAULT 0"
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    forms {
        UUID id PK
        VARCHAR name "NOT NULL"
        TEXT description
        VARCHAR slug "UNIQUE, NOT NULL, indexed"
        JSONB fields "NOT NULL, DEFAULT []"
        JSONB settings "DEFAULT {}"
        VARCHAR status "DEFAULT active"
        INTEGER submission_count "DEFAULT 0"
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    submissions {
        UUID id PK
        UUID form_id FK "NOT NULL, indexed"
        JSONB data "NOT NULL"
        JSONB metadata "DEFAULT {}"
        TIMESTAMPTZ created_at
    }

    companies ||--o{ contacts : "employs"
    contacts ||--o{ deals : "has deals"
    companies ||--o{ deals : "has deals"
    pipelines ||--|{ pipeline_stages : "contains"
    pipeline_stages ||--o{ deals : "stages deals"
    pipelines ||--o{ deals : "organizes"
    contacts ||--o{ activities : "has activities"
    companies ||--o{ activities : "has activities"
    deals ||--o{ activities : "has activities"
    contacts ||--o{ notes : "has notes"
    companies ||--o{ notes : "has notes"
    deals ||--o{ notes : "has notes"
    tickets ||--|{ ticket_comments : "has comments"
    kb_categories ||--o{ kb_articles : "contains"
    forms ||--|{ submissions : "receives"
```

## Relationship Summary

| Parent | Child | Cardinality | ON DELETE |
|--------|-------|-------------|----------|
| companies | contacts | 1:N | SET NULL (company_id) |
| contacts | deals | 1:N | SET NULL (contact_id) |
| companies | deals | 1:N | SET NULL (company_id) |
| pipelines | pipeline_stages | 1:N | CASCADE |
| pipelines | deals | 1:N | RESTRICT |
| pipeline_stages | deals | 1:N | RESTRICT |
| contacts | activities | 1:N | CASCADE |
| companies | activities | 1:N | CASCADE |
| deals | activities | 1:N | CASCADE |
| contacts | notes | 1:N | CASCADE |
| companies | notes | 1:N | CASCADE |
| deals | notes | 1:N | CASCADE |
| tickets | ticket_comments | 1:N | CASCADE |
| kb_categories | kb_articles | 1:N | SET NULL |
| forms | submissions | 1:N | CASCADE |

## Index Inventory

| Table | Index Name | Column(s) | Type |
|-------|-----------|-----------|------|
| contacts | idx_contacts_email | email | B-tree |
| contacts | idx_contacts_company_id | company_id | B-tree |
| contacts | idx_contacts_lifecycle_stage | lifecycle_stage | B-tree |
| contacts | idx_contacts_created_at | created_at | B-tree |
| companies | idx_companies_name | name | B-tree |
| companies | idx_companies_domain | domain | B-tree |
| pipeline_stages | idx_pipeline_stages_pipeline_id | pipeline_id | B-tree |
| deals | idx_deals_contact_id | contact_id | B-tree |
| deals | idx_deals_company_id | company_id | B-tree |
| deals | idx_deals_pipeline_id | pipeline_id | B-tree |
| deals | idx_deals_stage_id | stage_id | B-tree |
| activities | idx_activities_contact_id | contact_id | B-tree |
| activities | idx_activities_deal_id | deal_id | B-tree |
| activities | idx_activities_due_date | due_date | B-tree |
| tickets | idx_tickets_status | status | B-tree |
| tickets | idx_tickets_customer | customer_email | B-tree |
| ticket_comments | idx_comments_ticket | ticket_id | B-tree |
| submissions | idx_submissions_form | form_id | B-tree |
| forms | idx_forms_slug | slug | B-tree |

## Domain Model Relationships (DDD)

```mermaid
graph TB
    subgraph "CRM Bounded Context"
        CONTACT_BC["Contact Aggregate<br/>Root: Contact<br/>VOs: Email, Phone, Address, LeadScore"]
        DEAL_BC["Deal Aggregate<br/>Root: Deal<br/>VOs: Money, Probability<br/>Entities: DealProduct, StageChange, Competitor"]
        ACCOUNT_BC["Account (Company)<br/>Root: Company"]
    end

    subgraph "Support Bounded Context"
        TICKET_BC["Ticket Aggregate<br/>Root: Ticket<br/>Entities: Comment<br/>VOs: Priority, TicketType, SlaPolicy"]
        KB_BC["Knowledge Base<br/>Root: Article<br/>Parent: Category"]
    end

    subgraph "Forms Bounded Context"
        FORM_BC["Form Aggregate<br/>Root: Form<br/>Entities: Submission"]
    end

    CONTACT_BC -.->|"linked via contact_id"| DEAL_BC
    ACCOUNT_BC -.->|"linked via company_id"| CONTACT_BC
    ACCOUNT_BC -.->|"linked via company_id"| DEAL_BC
    CONTACT_BC -.->|"linked via customer_email"| TICKET_BC
    FORM_BC -.->|"web-to-lead"| CONTACT_BC
```
