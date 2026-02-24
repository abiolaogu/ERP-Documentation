# ERP-CRM Data Dictionary

## Table: contacts

Primary entity for lead and customer management.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | Generated (v7) | Primary key, time-ordered |
| email | VARCHAR(255) | No | - | Contact email address, indexed |
| first_name | VARCHAR(100) | Yes | NULL | First name |
| last_name | VARCHAR(100) | Yes | NULL | Last name |
| phone | VARCHAR(50) | Yes | NULL | Phone number |
| company_id | UUID | Yes | NULL | FK to companies.id |
| lead_score | INTEGER | No | 0 | Score 0-100, hot >= 80, warm >= 50 |
| lifecycle_stage | VARCHAR(50) | No | 'subscriber' | subscriber, lead, mql, sql, opportunity, customer, evangelist |
| source | VARCHAR(100) | Yes | NULL | Lead source (website, referral, etc.) |
| owner_id | UUID | Yes | NULL | Assigned sales rep |
| tags | TEXT[] | No | '{}' | Array of tag strings |
| custom_fields | JSONB | No | '{}' | Unlimited custom key-value pairs |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp |
| updated_at | TIMESTAMPTZ | No | NOW() | Last modification timestamp |

**Indexes:** email (B-tree), company_id (B-tree), lifecycle_stage (B-tree), created_at (B-tree)

---

## Table: companies

Organization/account records.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | Generated (v7) | Primary key |
| name | VARCHAR(255) | No | - | Company name |
| domain | VARCHAR(255) | Yes | NULL | Company domain (e.g., acme.com) |
| industry | VARCHAR(100) | Yes | NULL | Industry classification |
| size | VARCHAR(50) | Yes | NULL | Employee count range (e.g., 100-500) |
| website | VARCHAR(500) | Yes | NULL | Company website URL |
| phone | VARCHAR(50) | Yes | NULL | Company phone number |
| address | JSONB | Yes | NULL | Structured address (street, city, state, country, postal_code) |
| owner_id | UUID | Yes | NULL | Assigned account manager |
| custom_fields | JSONB | No | '{}' | Unlimited custom key-value pairs |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp |
| updated_at | TIMESTAMPTZ | No | NOW() | Last modification timestamp |

**Indexes:** name (B-tree), domain (B-tree)

---

## Table: pipelines

Sales pipeline definitions.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| name | VARCHAR(100) | No | - | Pipeline name |
| description | TEXT | Yes | NULL | Pipeline description |
| is_default | BOOLEAN | No | FALSE | Default pipeline flag (only one per tenant) |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp |

**Seed Data:** Default pipeline "Sales Pipeline" with UUID `00000000-0000-0000-0000-000000000001`

---

## Table: pipeline_stages

Stages within a sales pipeline.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| pipeline_id | UUID | No | - | FK to pipelines.id (CASCADE on delete) |
| name | VARCHAR(100) | No | - | Stage name |
| position | INTEGER | No | - | Display order (1-based) |
| probability | INTEGER | No | 0 | Default win probability percentage (0-100) |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp |

**Indexes:** pipeline_id (B-tree)

**Seed Data:**

| Stage | Position | Probability | UUID |
|-------|----------|------------|------|
| Lead | 1 | 10% | ...0011 |
| Qualified | 2 | 25% | ...0012 |
| Proposal | 3 | 50% | ...0013 |
| Negotiation | 4 | 75% | ...0014 |
| Closed Won | 5 | 100% | ...0015 |
| Closed Lost | 6 | 0% | ...0016 |

---

## Table: deals

Sales opportunities/deals.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | Generated (v7) | Primary key |
| name | VARCHAR(255) | No | - | Deal name |
| contact_id | UUID | Yes | NULL | FK to contacts.id (SET NULL on delete) |
| company_id | UUID | Yes | NULL | FK to companies.id (SET NULL on delete) |
| pipeline_id | UUID | No | - | FK to pipelines.id |
| stage_id | UUID | No | - | FK to pipeline_stages.id |
| amount | BIGINT | Yes | NULL | Deal value in smallest currency unit |
| currency | VARCHAR(3) | No | 'NGN' | ISO 4217 currency code |
| probability | INTEGER | Yes | NULL | Win probability percentage (0-100) |
| expected_close_date | TIMESTAMPTZ | Yes | NULL | Target close date |
| owner_id | UUID | Yes | NULL | Assigned sales rep |
| custom_fields | JSONB | No | '{}' | Unlimited custom key-value pairs |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp |
| updated_at | TIMESTAMPTZ | No | NOW() | Last modification timestamp |
| closed_at | TIMESTAMPTZ | Yes | NULL | When the deal was closed (won or lost) |

**Indexes:** contact_id, company_id, pipeline_id, stage_id (all B-tree)

---

## Table: activities

Interaction records (calls, emails, meetings, tasks).

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | Generated (v7) | Primary key |
| activity_type | VARCHAR(50) | No | - | call, email, meeting, task |
| subject | VARCHAR(255) | No | - | Activity subject line |
| description | TEXT | Yes | NULL | Detailed description |
| contact_id | UUID | Yes | NULL | FK to contacts.id (CASCADE) |
| company_id | UUID | Yes | NULL | FK to companies.id (CASCADE) |
| deal_id | UUID | Yes | NULL | FK to deals.id (CASCADE) |
| owner_id | UUID | Yes | NULL | Who performed the activity |
| due_date | TIMESTAMPTZ | Yes | NULL | When the activity is due |
| completed_at | TIMESTAMPTZ | Yes | NULL | When the activity was completed |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp |

**Indexes:** contact_id, deal_id, due_date (all B-tree)

---

## Table: notes

Internal notes on CRM entities.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| content | TEXT | No | - | Note content (Markdown supported) |
| contact_id | UUID | Yes | NULL | FK to contacts.id (CASCADE) |
| company_id | UUID | Yes | NULL | FK to companies.id (CASCADE) |
| deal_id | UUID | Yes | NULL | FK to deals.id (CASCADE) |
| created_by | UUID | Yes | NULL | Author user ID |
| created_at | TIMESTAMPTZ | No | NOW() | Record creation timestamp |
| updated_at | TIMESTAMPTZ | No | NOW() | Last modification timestamp |

---

## Table: tickets

Support tickets (from opensase-support).

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| ticket_number | VARCHAR(50) | No | - | Unique human-readable ticket number |
| subject | VARCHAR(255) | No | - | Ticket subject |
| description | TEXT | No | - | Detailed issue description |
| customer_email | VARCHAR(255) | No | - | Requester email, indexed |
| customer_name | VARCHAR(100) | Yes | NULL | Requester name |
| priority | VARCHAR(20) | No | 'medium' | low, medium, high, urgent |
| status | VARCHAR(50) | No | 'open' | new, open, pending, on_hold, solved, closed |
| category | VARCHAR(100) | Yes | NULL | Ticket category |
| assignee_id | UUID | Yes | NULL | Assigned agent |
| created_at | TIMESTAMPTZ | No | NOW() | Ticket creation time |
| updated_at | TIMESTAMPTZ | No | NOW() | Last update time |
| resolved_at | TIMESTAMPTZ | Yes | NULL | Resolution time |

**Indexes:** status (B-tree), customer_email (B-tree)

---

## Table: ticket_comments

Comments/replies on support tickets.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| ticket_id | UUID | No | - | FK to tickets.id (CASCADE) |
| author_type | VARCHAR(20) | No | - | agent, customer, system |
| author_id | UUID | Yes | NULL | Author user ID |
| author_email | VARCHAR(255) | No | - | Author email |
| content | TEXT | No | - | Comment content |
| is_internal | BOOLEAN | No | FALSE | Internal note (not visible to customer) |
| created_at | TIMESTAMPTZ | No | NOW() | Comment timestamp |

**Indexes:** ticket_id (B-tree)

---

## Table: kb_categories

Knowledge base article categories.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| name | VARCHAR(100) | No | - | Category name |
| slug | VARCHAR(100) | No | - | URL-friendly identifier (unique) |
| description | TEXT | Yes | NULL | Category description |
| created_at | TIMESTAMPTZ | No | NOW() | Creation timestamp |

---

## Table: kb_articles

Knowledge base articles.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| title | VARCHAR(255) | No | - | Article title |
| slug | VARCHAR(255) | No | - | URL-friendly identifier (unique) |
| content | TEXT | No | - | Article body (Markdown) |
| category_id | UUID | Yes | NULL | FK to kb_categories.id |
| status | VARCHAR(50) | No | 'draft' | draft, published |
| view_count | INTEGER | No | 0 | Number of views |
| created_at | TIMESTAMPTZ | No | NOW() | Creation timestamp |
| updated_at | TIMESTAMPTZ | No | NOW() | Last update timestamp |

---

## Table: forms

Form builder form definitions.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| name | VARCHAR(255) | No | - | Form name |
| description | TEXT | Yes | NULL | Form description |
| slug | VARCHAR(255) | No | - | URL-friendly identifier (unique, indexed) |
| fields | JSONB | No | '[]' | Array of field definitions |
| settings | JSONB | No | '{}' | Form settings (notifications, redirects) |
| status | VARCHAR(50) | No | 'active' | active, inactive |
| submission_count | INTEGER | No | 0 | Total submissions received |
| created_at | TIMESTAMPTZ | No | NOW() | Creation timestamp |
| updated_at | TIMESTAMPTZ | No | NOW() | Last update timestamp |

**Indexes:** slug (B-tree)

---

## Table: submissions

Form submission records.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | - | Primary key |
| form_id | UUID | No | - | FK to forms.id (CASCADE) |
| data | JSONB | No | - | Submitted field values |
| metadata | JSONB | No | '{}' | IP address, user agent, referrer, timestamp |
| created_at | TIMESTAMPTZ | No | NOW() | Submission timestamp |

**Indexes:** form_id (B-tree)

---

## Domain Value Types

| Value Object | Rust Type | Validation | Storage |
|-------------|-----------|-----------|---------|
| Email | Email(String) | Contains @, domain has dot, non-empty | VARCHAR(255) |
| Money | Money { amount: Decimal, currency: Currency } | Non-negative, valid currency | BIGINT (amount) + VARCHAR(3) (currency) |
| Phone | Phone(String) | Non-empty, valid characters | VARCHAR(50) |
| Address | Address { street, city, state, country, postal_code } | Non-empty fields | JSONB |
| EntityId | EntityId(String) | UUID format | UUID |
| LeadScore | LeadScore(u8) | 0-100, capped at 100 | INTEGER |
| Probability | Probability(u8) | 0-100, capped at 100 | INTEGER |

## Enumeration Types

### LeadStatus
`New` | `Contacted` | `Qualified` | `Unqualified` | `Nurturing` | `Converted`

### LifecycleStage
`Subscriber` | `Lead` | `MarketingQualifiedLead` | `SalesQualifiedLead` | `Opportunity` | `Customer` | `Evangelist`

### DealStatus
`Open` | `Won` | `Lost`

### DealType
`NewBusiness` | `Renewal` | `Upsell` | `CrossSell`

### TicketStatus
`New` | `Open` | `Pending` | `OnHold` | `Solved` | `Closed`

### Priority
`Low` | `Normal` | `High` | `Urgent`

### Currency
`USD` | `EUR` | `GBP` | `CAD` | `AUD` | `NGN` | `JPY` | `CNY` | `INR` | `Other(String)`
