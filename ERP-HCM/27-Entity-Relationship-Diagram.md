# ERP-HCM Entity-Relationship Diagram

## Database Schema Documentation

---

## 1. Overview

ERP-HCM uses PostgreSQL 16 with schema-per-domain isolation. The database contains 30+ schemas with 47 migration files defining 80+ tables. All tables use UUID primary keys, `TIMESTAMPTZ` for timestamps, and include `tenant_id` for multi-tenancy (shared database, shared schema pattern with the `core` schema; the `employee` schema uses `legal_entity_id` as the scoping column).

### 1.1 Schema Organization

| Schema | Domain | Key Tables |
|--------|--------|------------|
| core | Multi-tenancy, Auth, Audit | tenants, users, sessions, audit_logs |
| employee | Employee Lifecycle | employees, departments, locations, legal_entities |
| payroll | Payroll Engine | payroll_runs, payroll_entries, salary_components |
| leave | Leave Management | leave_requests, leave_balances, leave_types |
| attendance | Time & Attendance | clock_records, geofences, shifts |
| auth | Authentication | user_accounts, roles, permissions, sessions |
| recruitment | ATS | job_requisitions, candidates, applications |
| performance | Performance Mgmt | okr_cycles, objectives, review_cycles, reviews |
| benefits | Benefits & EWA | plans, enrollments, claims, ewa_transactions |
| lms | Learning | courses, modules, enrollments, certificates |
| workforce | Workforce Planning | headcount_plans, scenarios |
| dms | Document Mgmt | documents, signatures |
| communication | Internal Comms | messages, channels |
| analytics | Analytics | dashboards, reports |
| notification | Notifications | templates, delivery_log |

---

## 2. Core Schema ERD

```mermaid
erDiagram
    TENANTS {
        uuid id PK
        varchar name
        varchar slug UK
        varchar domain
        varchar plan
        varchar status
        varchar isolation_mode
        jsonb settings
        jsonb branding
        varchar currency
        varchar timezone
        timestamptz created_at
        timestamptz updated_at
    }

    USERS {
        uuid id PK
        uuid tenant_id FK
        varchar email
        varchar password_hash
        varchar first_name
        varchar last_name
        varchar role
        text[] permissions
        boolean is_active
        boolean mfa_enabled
        varchar mfa_secret
        timestamptz last_login_at
        int failed_login_attempts
        timestamptz locked_until
        timestamptz created_at
    }

    SESSIONS {
        uuid id PK
        uuid user_id FK
        uuid tenant_id FK
        varchar refresh_token_hash
        text user_agent
        varchar ip_address
        timestamptz expires_at
        timestamptz revoked_at
    }

    API_KEYS {
        uuid id PK
        uuid tenant_id FK
        varchar name
        varchar key_hash
        varchar key_prefix
        text[] permissions
        int rate_limit
        timestamptz expires_at
    }

    AUDIT_LOGS {
        uuid id PK
        uuid tenant_id
        uuid user_id
        varchar action
        varchar resource_type
        varchar resource_id
        jsonb old_values
        jsonb new_values
        varchar ip_address
        timestamptz created_at
    }

    COMPLIANCE_LOGS {
        uuid id PK
        uuid tenant_id
        varchar event_type
        varchar data_subject
        varchar purpose
        varchar legal_basis
        jsonb details
        timestamptz created_at
    }

    TENANTS ||--o{ USERS : "has"
    TENANTS ||--o{ SESSIONS : "scopes"
    TENANTS ||--o{ API_KEYS : "owns"
    TENANTS ||--o{ AUDIT_LOGS : "generates"
    USERS ||--o{ SESSIONS : "creates"
```

---

## 3. Employee Schema ERD

```mermaid
erDiagram
    LEGAL_ENTITIES {
        uuid id PK
        uuid parent_entity_id FK
        varchar code UK
        varchar name
        varchar entity_type
        varchar registration_number
        varchar tax_id
        varchar country_code
        varchar currency_code
        boolean is_active
    }

    LOCATIONS {
        uuid id PK
        uuid legal_entity_id FK
        varchar code
        varchar name
        varchar location_type
        varchar city
        varchar state
        decimal latitude
        decimal longitude
        varchar timezone
        boolean is_headquarters
    }

    DEPARTMENTS {
        uuid id PK
        uuid legal_entity_id FK
        uuid parent_department_id FK
        uuid cost_center_id FK
        varchar code
        varchar name
        int level
        varchar path
        uuid head_id
        int headcount_budget
    }

    COST_CENTERS {
        uuid id PK
        uuid legal_entity_id FK
        varchar code
        varchar name
        uuid manager_id
        decimal budget_amount
        varchar budget_currency
    }

    JOB_GRADES {
        uuid id PK
        uuid legal_entity_id FK
        varchar code
        varchar name
        int level
        decimal min_salary
        decimal mid_salary
        decimal max_salary
        varchar currency_code
    }

    JOB_TITLES {
        uuid id PK
        uuid legal_entity_id FK
        uuid department_id FK
        uuid job_grade_id FK
        varchar code
        varchar title
        varchar job_family
        boolean is_management
    }

    EMPLOYEES {
        uuid id PK
        uuid legal_entity_id FK
        varchar employee_number UK
        varchar first_name
        varchar last_name
        varchar work_email UK
        date date_of_birth
        varchar gender
        varchar nationality
        varchar nin
        varchar bvn
        enum status
        enum employment_type
        uuid job_title_id FK
        uuid department_id FK
        uuid location_id FK
        uuid reports_to_id FK
        date hire_date
        decimal current_salary
        varchar salary_currency
        boolean is_manager
        timestamptz created_at
    }

    EMPLOYEE_BANK_ACCOUNTS {
        uuid id PK
        uuid employee_id FK
        varchar bank_code
        varchar bank_name
        varchar account_number
        varchar account_name
        boolean is_primary
        boolean is_salary_account
    }

    EMPLOYEE_TAX_INFO {
        uuid id PK
        uuid employee_id FK
        varchar tax_id
        varchar tax_state
        varchar tax_status
        boolean has_nhf_contribution
        boolean has_pension_contribution
        date effective_date
    }

    EMPLOYEE_PENSION_INFO {
        uuid id PK
        uuid employee_id FK
        varchar pfa_code
        varchar pfa_name
        varchar rsa_pin UK
        decimal employee_contribution_rate
        decimal employer_contribution_rate
    }

    EMPLOYEE_EMERGENCY_CONTACTS {
        uuid id PK
        uuid employee_id FK
        varchar contact_name
        varchar relationship
        varchar phone_primary
        int priority
    }

    EMPLOYEE_DEPENDENTS {
        uuid id PK
        uuid employee_id FK
        varchar dependent_type
        varchar first_name
        varchar last_name
        varchar relationship
        boolean is_next_of_kin
    }

    EMPLOYEE_DOCUMENTS {
        uuid id PK
        uuid employee_id FK
        uuid category_id FK
        varchar document_name
        varchar document_type
        varchar file_path
        varchar storage_provider
        date expiry_date
        varchar status
        boolean is_confidential
    }

    JOB_CHANGES {
        uuid id PK
        uuid employee_id FK
        enum change_type
        uuid previous_job_title_id FK
        uuid new_job_title_id FK
        decimal previous_salary
        decimal new_salary
        varchar status
        date effective_date
    }

    EMPLOYEE_TIMELINE {
        uuid id PK
        uuid employee_id FK
        varchar event_type
        varchar event_category
        timestamptz event_date
        varchar title
        varchar field_changed
    }

    LEGAL_ENTITIES ||--o{ LOCATIONS : "has"
    LEGAL_ENTITIES ||--o{ DEPARTMENTS : "contains"
    LEGAL_ENTITIES ||--o{ COST_CENTERS : "manages"
    LEGAL_ENTITIES ||--o{ EMPLOYEES : "employs"
    DEPARTMENTS ||--o{ EMPLOYEES : "includes"
    DEPARTMENTS ||--o{ DEPARTMENTS : "parent-child"
    JOB_TITLES ||--o{ EMPLOYEES : "assigned to"
    JOB_GRADES ||--o{ EMPLOYEES : "graded"
    LOCATIONS ||--o{ EMPLOYEES : "located at"
    EMPLOYEES ||--o{ EMPLOYEES : "reports to"
    EMPLOYEES ||--o{ EMPLOYEE_BANK_ACCOUNTS : "has"
    EMPLOYEES ||--o{ EMPLOYEE_TAX_INFO : "has"
    EMPLOYEES ||--o{ EMPLOYEE_PENSION_INFO : "has"
    EMPLOYEES ||--o{ EMPLOYEE_EMERGENCY_CONTACTS : "has"
    EMPLOYEES ||--o{ EMPLOYEE_DEPENDENTS : "has"
    EMPLOYEES ||--o{ EMPLOYEE_DOCUMENTS : "uploads"
    EMPLOYEES ||--o{ JOB_CHANGES : "undergoes"
    EMPLOYEES ||--o{ EMPLOYEE_TIMELINE : "generates"
```

---

## 4. Payroll Schema ERD

```mermaid
erDiagram
    SALARY_COMPONENT_TYPES {
        uuid id PK
        uuid tenant_id FK
        varchar code UK
        varchar name
        varchar category
        varchar sub_category
        varchar calculation_type
        decimal percentage_value
        boolean is_taxable
        boolean is_pensionable
        boolean is_statutory
        varchar statutory_type
        bigint min_amount
        bigint max_amount
    }

    SALARY_GRADES {
        uuid id PK
        uuid tenant_id FK
        varchar code
        varchar name
        int level
        bigint min_salary
        bigint mid_salary
        bigint max_salary
        varchar currency
    }

    PAYROLL_GROUPS {
        uuid id PK
        uuid tenant_id FK
        varchar code
        varchar name
        varchar pay_frequency
        int pay_day
        boolean requires_approval
        int approval_levels
    }

    EMPLOYEE_PAYROLL_CONFIGS {
        uuid id PK
        uuid tenant_id FK
        uuid employee_id FK
        uuid payroll_group_id FK
        uuid salary_grade_id FK
        bigint gross_salary
        bigint basic_salary
        varchar tax_state
        varchar tin
        varchar rsa_pin
        decimal pension_employee_rate
        decimal pension_employer_rate
        varchar payment_method
        date effective_date
    }

    PAYROLL_PERIODS {
        uuid id PK
        uuid tenant_id FK
        varchar period_name
        int period_year
        int period_month
        date start_date
        date end_date
        date pay_date
        varchar status
        boolean is_locked
    }

    PAYROLL_RUNS {
        uuid id PK
        uuid tenant_id FK
        uuid period_id FK
        varchar run_number
        varchar run_type
        varchar status
        int total_employees
        bigint total_gross
        bigint total_net
        bigint total_paye
        bigint total_pension_employee
        bigint total_pension_employer
        bigint total_nhf
        varchar currency
        uuid created_by
        uuid approved_by
    }

    PAYROLL_RUN_APPROVALS {
        uuid id PK
        uuid run_id FK
        int approval_level
        uuid approver_id
        varchar status
        text comments
    }

    PAYROLL_ENTRIES {
        uuid id PK
        uuid tenant_id FK
        uuid run_id FK
        uuid period_id FK
        uuid employee_id FK
        bigint gross_salary
        bigint basic_salary
        bigint total_earnings
        bigint total_deductions
        bigint net_pay
        bigint taxable_income
        bigint cra_amount
        bigint paye_amount
        bigint pension_employee
        bigint pension_employer
        bigint nhf_amount
        bigint nsitf_amount
        bigint ytd_gross
        bigint ytd_paye
        varchar payment_status
    }

    PAYROLL_ENTRY_ITEMS {
        uuid id PK
        uuid entry_id FK
        uuid component_type_id FK
        varchar component_code
        varchar component_name
        varchar category
        bigint amount
        decimal rate
        boolean is_taxable
        boolean is_pensionable
    }

    PAYROLL_PERIODS ||--o{ PAYROLL_RUNS : "contains"
    PAYROLL_RUNS ||--o{ PAYROLL_ENTRIES : "generates"
    PAYROLL_RUNS ||--o{ PAYROLL_RUN_APPROVALS : "requires"
    PAYROLL_ENTRIES ||--o{ PAYROLL_ENTRY_ITEMS : "itemized"
    SALARY_COMPONENT_TYPES ||--o{ PAYROLL_ENTRY_ITEMS : "defines"
    PAYROLL_GROUPS ||--o{ EMPLOYEE_PAYROLL_CONFIGS : "groups"
    SALARY_GRADES ||--o{ EMPLOYEE_PAYROLL_CONFIGS : "grades"
```

---

## 5. Auth Schema ERD

```mermaid
erDiagram
    USER_ACCOUNTS {
        uuid id PK
        uuid organization_id FK
        varchar email UK
        varchar password_hash
        varchar password_algorithm
        varchar status
        boolean mfa_enabled
        varchar sso_provider
        int failed_login_attempts
        timestamptz last_login_at
    }

    ROLES {
        uuid id PK
        uuid organization_id FK
        varchar name
        varchar slug UK
        varchar role_type
        uuid parent_role_id FK
        int level
        varchar scope
        boolean is_system
    }

    PERMISSIONS {
        uuid id PK
        varchar name
        varchar slug UK
        varchar module
        varchar resource
        varchar action
        varchar scope
        jsonb field_restrictions
        boolean is_system
    }

    ROLE_PERMISSIONS {
        uuid id PK
        uuid role_id FK
        uuid permission_id FK
        varchar scope_override
    }

    USER_ROLES {
        uuid id PK
        uuid user_account_id FK
        uuid role_id FK
        uuid organization_id
        varchar scope_type
        uuid scope_id
        boolean is_active
    }

    USER_SESSIONS {
        uuid id PK
        uuid user_account_id FK
        varchar access_token_hash
        varchar refresh_token_hash
        varchar session_type
        varchar ip_address
        boolean is_active
        timestamptz access_token_expires_at
    }

    MFA_TOTP_SECRETS {
        uuid id PK
        uuid user_account_id FK
        varchar secret_encrypted
        int digits
        int period
        boolean is_verified
        jsonb backup_codes
    }

    PASSWORD_POLICIES {
        uuid id PK
        uuid organization_id FK
        int min_length
        boolean require_uppercase
        boolean require_special_chars
        int password_history_count
        int password_expiry_days
        int max_failed_attempts
        int lockout_duration_minutes
    }

    AUTH_AUDIT_LOG {
        uuid id PK
        uuid organization_id
        uuid user_account_id
        varchar event_type
        varchar event_category
        varchar event_status
        varchar ip_address
        int risk_score
    }

    USER_ACCOUNTS ||--o{ USER_ROLES : "assigned"
    USER_ACCOUNTS ||--o{ USER_SESSIONS : "has"
    USER_ACCOUNTS ||--o| MFA_TOTP_SECRETS : "configures"
    ROLES ||--o{ ROLE_PERMISSIONS : "grants"
    ROLES ||--o{ USER_ROLES : "assigned via"
    PERMISSIONS ||--o{ ROLE_PERMISSIONS : "included in"
    USER_ACCOUNTS ||--o{ AUTH_AUDIT_LOG : "generates"
```

---

## 6. Leave & Attendance Schema ERD

```mermaid
erDiagram
    LEAVE_TYPES {
        uuid id PK
        uuid tenant_id FK
        varchar name
        varchar code
        int default_days
        boolean is_paid
        boolean requires_approval
        boolean requires_document
        boolean is_carry_forward
        int max_carry_forward_days
    }

    LEAVE_REQUESTS {
        uuid id PK
        uuid tenant_id FK
        uuid employee_id FK
        uuid leave_type_id FK
        date start_date
        date end_date
        varchar day_type
        decimal total_days
        varchar status
        varchar reason
        uuid approver_id
        uuid delegate_to_id
    }

    LEAVE_BALANCES {
        uuid id PK
        uuid tenant_id FK
        uuid employee_id FK
        uuid leave_type_id FK
        int year
        decimal entitled
        decimal used
        decimal pending
        decimal available
        decimal carry_forward
    }

    HOLIDAYS {
        uuid id PK
        uuid tenant_id FK
        varchar name
        date holiday_date
        varchar location_scope
        boolean is_optional
    }

    CLOCK_RECORDS {
        uuid id PK
        uuid tenant_id FK
        uuid employee_id FK
        timestamptz clock_in_time
        timestamptz clock_out_time
        decimal latitude_in
        decimal longitude_in
        decimal accuracy_in
        varchar geofence_status
        decimal distance_meters
        varchar device_id
        varchar spoofing_status
    }

    GEOFENCES {
        uuid id PK
        uuid tenant_id FK
        uuid office_id FK
        varchar name
        decimal center_latitude
        decimal center_longitude
        int radius_meters
        boolean is_active
    }

    SHIFTS {
        uuid id PK
        uuid tenant_id FK
        varchar name
        time start_time
        time end_time
        int break_minutes
        boolean is_night_shift
    }

    LEAVE_TYPES ||--o{ LEAVE_REQUESTS : "of type"
    LEAVE_TYPES ||--o{ LEAVE_BALANCES : "tracks"
    LEAVE_REQUESTS }o--|| LEAVE_BALANCES : "reduces"
    GEOFENCES ||--o{ CLOCK_RECORDS : "validates"
```

---

## 7. Cross-Domain Relationships

```mermaid
erDiagram
    EMPLOYEES ||--o{ PAYROLL_ENTRIES : "paid via"
    EMPLOYEES ||--o{ LEAVE_REQUESTS : "submits"
    EMPLOYEES ||--o{ CLOCK_RECORDS : "clocks"
    EMPLOYEES ||--o{ JOB_CHANGES : "transitions"
    EMPLOYEES ||--o{ EMPLOYEE_DOCUMENTS : "uploads"
    EMPLOYEES ||--o{ EMPLOYEE_BANK_ACCOUNTS : "banks with"
    EMPLOYEES ||--o{ EMPLOYEE_PENSION_INFO : "pension"
    EMPLOYEES ||--o{ EMPLOYEE_TAX_INFO : "tax"

    PAYROLL_RUNS ||--o{ PAYROLL_ENTRIES : "contains"
    PAYROLL_PERIODS ||--o{ PAYROLL_RUNS : "period"

    USER_ACCOUNTS ||--|| EMPLOYEES : "linked"
    TENANTS ||--o{ LEGAL_ENTITIES : "operates"
    LEGAL_ENTITIES ||--o{ EMPLOYEES : "employs"
    LEGAL_ENTITIES ||--o{ DEPARTMENTS : "structures"
    DEPARTMENTS ||--o{ EMPLOYEES : "staffs"
```

---

## 8. Database Extensions

| Extension | Purpose |
|-----------|---------|
| uuid-ossp | UUID v4 generation (`uuid_generate_v4()`) |
| pgcrypto | Cryptographic functions (`gen_random_uuid()`) |
| pg_trgm | Trigram-based text search (full-name search) |
| TimescaleDB | Time-series hypertable for attendance data (optional) |

---

## 9. Indexing Strategy

### 9.1 Composite Indexes (Multi-Tenant)

All tables with `tenant_id` have composite indexes:
```sql
CREATE INDEX idx_employees_entity ON employee.employees(legal_entity_id);
CREATE INDEX idx_payroll_entries_employee ON payroll_entries(tenant_id, employee_id);
CREATE INDEX idx_payroll_periods_company ON payroll_periods(tenant_id, company_id);
```

### 9.2 Partial Indexes

```sql
CREATE INDEX idx_employees_active ON employee.employees(status) WHERE status = 'active';
CREATE INDEX idx_employees_deleted ON employee.employees(deleted_at) WHERE deleted_at IS NULL;
CREATE INDEX idx_employee_bank_accounts_primary ON employee.employee_bank_accounts(employee_id, is_primary) WHERE is_primary = true;
```

### 9.3 Full-Text Search Indexes

```sql
CREATE INDEX idx_employees_full_name ON employee.employees USING gin(to_tsvector('english', full_name));
```
