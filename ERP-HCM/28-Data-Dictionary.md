# ERP-HCM Data Dictionary

## Complete Field-Level Documentation for All Database Tables

---

## 1. Overview

This data dictionary documents all tables, columns, data types, constraints, and business rules for the ERP-HCM database. The database uses PostgreSQL 16 with 30+ schemas and 80+ tables across 47 migration files.

### 1.1 Conventions

| Convention | Standard |
|-----------|----------|
| Primary Keys | UUID v4 via `gen_random_uuid()` |
| Timestamps | `TIMESTAMPTZ` (timezone-aware) |
| Monetary Values | `BIGINT` in kobo (1 NGN = 100 kobo) in payroll; `DECIMAL(18,2)` in employee/org |
| Soft Delete | `deleted_at TIMESTAMPTZ` (NULL = active) |
| Multi-tenancy | `tenant_id UUID` or `legal_entity_id UUID` |
| JSONB Defaults | `'{}'::jsonb` for objects, `'[]'::jsonb` for arrays |
| Audit Fields | `created_at`, `updated_at`, `created_by`, `updated_by` |

---

## 2. Core Schema (`core`)

### 2.1 `core.tenants`

Tenant/organization record for multi-tenancy.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| name | VARCHAR(255) | NO | | Organization display name |
| slug | VARCHAR(100) | NO | | URL-safe unique identifier |
| domain | VARCHAR(255) | YES | | Primary email domain |
| custom_domain | VARCHAR(255) | YES | | Custom portal domain |
| plan | VARCHAR(50) | NO | 'free' | Subscription plan: free, professional, enterprise |
| status | VARCHAR(50) | NO | 'pending' | Account status: pending, active, suspended, cancelled |
| isolation_mode | VARCHAR(50) | NO | 'shared' | Data isolation: shared, schema, database |
| settings | JSONB | YES | '{}' | Tenant-specific configuration |
| branding | JSONB | YES | '{}' | Logo, colors, themes |
| limits | JSONB | YES | '{}' | Feature and usage limits |
| features | TEXT[] | YES | '{}' | Enabled feature flags |
| billing_email | VARCHAR(255) | YES | | Billing contact email |
| country | VARCHAR(3) | YES | | ISO 3166-1 alpha-3 country code |
| currency | VARCHAR(3) | YES | 'NGN' | Default currency (ISO 4217) |
| timezone | VARCHAR(50) | YES | 'Africa/Lagos' | Default timezone (IANA) |
| created_at | TIMESTAMPTZ | YES | NOW() | Record creation timestamp |
| updated_at | TIMESTAMPTZ | YES | NOW() | Last update timestamp |
| trial_ends_at | TIMESTAMPTZ | YES | | Trial expiration date |
| suspended_at | TIMESTAMPTZ | YES | | Suspension timestamp |
| suspended_reason | TEXT | YES | | Reason for suspension |

**Constraints**: `slug` is UNIQUE. **Indexes**: slug, custom_domain.

### 2.2 `core.users`

User authentication records.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| tenant_id | UUID | YES | | FK to core.tenants |
| email | VARCHAR(255) | NO | | User email address |
| password_hash | VARCHAR(255) | YES | | Argon2id hashed password |
| first_name | VARCHAR(100) | YES | | First name |
| last_name | VARCHAR(100) | YES | | Last name |
| role | VARCHAR(50) | NO | 'employee' | Base role: super_admin, hr_admin, manager, employee |
| permissions | TEXT[] | YES | '{}' | Direct permission grants |
| is_active | BOOLEAN | YES | true | Account active flag |
| email_verified | BOOLEAN | YES | false | Email verified via OTP |
| mfa_enabled | BOOLEAN | YES | false | MFA (TOTP) enabled |
| mfa_secret | VARCHAR(255) | YES | | Encrypted TOTP secret |
| last_login_at | TIMESTAMPTZ | YES | | Last successful login |
| last_login_ip | VARCHAR(50) | YES | | IP of last login |
| failed_login_attempts | INT | YES | 0 | Consecutive failures (resets on success) |
| locked_until | TIMESTAMPTZ | YES | | Lockout expiry (5 failures = 30min lock) |
| password_changed_at | TIMESTAMPTZ | YES | | Last password change |
| created_at | TIMESTAMPTZ | YES | NOW() | |
| updated_at | TIMESTAMPTZ | YES | NOW() | |

**Constraints**: UNIQUE(tenant_id, email). **Business Rules**: Account locks after 5 failed attempts for 30 minutes. Password minimum 8 characters with special character required. Password history tracks last 5 passwords.

### 2.3 `core.audit_logs`

Immutable audit trail for all data mutations.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | uuid_generate_v4() | Primary key |
| tenant_id | UUID | NO | | Tenant scope |
| user_id | UUID | YES | | Acting user (NULL for system) |
| action | VARCHAR(100) | NO | | Action: create, update, delete, approve, reject |
| resource_type | VARCHAR(100) | NO | | Entity type: employee, payroll_run, leave_request |
| resource_id | VARCHAR(100) | YES | | Entity UUID |
| old_values | JSONB | YES | | Previous field values (for updates) |
| new_values | JSONB | YES | | New field values |
| ip_address | VARCHAR(50) | YES | | Client IP address |
| user_agent | TEXT | YES | | Client user agent |
| request_id | VARCHAR(100) | YES | | Distributed trace ID |
| created_at | TIMESTAMPTZ | YES | NOW() | Event timestamp |

**Business Rules**: Append-only table. No UPDATE or DELETE allowed. Retained for 7 years per NDPR/SOC 2 requirements.

---

## 3. Employee Schema (`employee`)

### 3.1 `employee.employees`

Core employee record with 50+ fields covering personal, employment, and organizational data.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| user_account_id | UUID | YES | | FK to auth user_accounts |
| legal_entity_id | UUID | NO | | FK to legal_entities (tenant scope) |
| employee_number | VARCHAR(50) | NO | | Auto-generated: EMP-001 format |
| badge_number | VARCHAR(50) | YES | | Physical badge/access card number |
| title | VARCHAR(20) | YES | | Salutation: Mr, Mrs, Ms, Dr |
| first_name | VARCHAR(100) | NO | | **PII** - Field-level encrypted |
| middle_name | VARCHAR(100) | YES | | |
| last_name | VARCHAR(100) | NO | | **PII** - Field-level encrypted |
| preferred_name | VARCHAR(100) | YES | | Display name override |
| date_of_birth | DATE | YES | | **PII** |
| gender | VARCHAR(20) | YES | | male, female, non_binary, prefer_not_to_say |
| marital_status | VARCHAR(30) | YES | | single, married, divorced, widowed |
| nationality | VARCHAR(100) | YES | | |
| state_of_origin | VARCHAR(100) | YES | | Nigerian state of origin |
| lga_of_origin | VARCHAR(100) | YES | | Local Government Area |
| nin | VARCHAR(11) | YES | | **PII** - National Identification Number (11 digits) |
| bvn | VARCHAR(11) | YES | | **PII** - Bank Verification Number (11 digits) |
| personal_email | VARCHAR(255) | YES | | **PII** |
| work_email | VARCHAR(255) | NO | | Corporate email (UNIQUE) |
| personal_phone | VARCHAR(50) | YES | | **PII** |
| work_phone | VARCHAR(50) | YES | | |
| photo_url | VARCHAR(500) | YES | | Profile photo S3 URL |
| status | ENUM | NO | 'pre_boarding' | See Employee Status Enum below |
| employment_type | ENUM | NO | 'full_time' | See Employment Type Enum below |
| job_title_id | UUID | YES | | FK to job_titles |
| department_id | UUID | YES | | FK to departments |
| job_grade_id | UUID | YES | | FK to job_grades |
| location_id | UUID | YES | | FK to locations |
| cost_center_id | UUID | YES | | FK to cost_centers |
| reports_to_id | UUID | YES | | FK to employees (self-referential) |
| dotted_line_manager_id | UUID | YES | | FK to employees |
| hire_date | DATE | YES | | Original hire date |
| probation_end_date | DATE | YES | | Probation period end |
| confirmation_date | DATE | YES | | Confirmed employment date |
| contract_end_date | DATE | YES | | For contract employees |
| termination_date | DATE | YES | | Employment end date |
| current_salary | DECIMAL(18,2) | YES | | Current gross salary |
| salary_currency | VARCHAR(3) | YES | 'NGN' | ISO 4217 currency code |
| pay_frequency | VARCHAR(20) | YES | 'monthly' | weekly, bi-weekly, monthly |
| is_manager | BOOLEAN | YES | false | Has direct reports |
| full_name | VARCHAR(300) | GENERATED | | Concatenated first + middle + last |
| metadata | JSONB | YES | '{}' | Custom fields |
| created_at | TIMESTAMPTZ | NO | NOW() | |
| updated_at | TIMESTAMPTZ | NO | NOW() | |
| deleted_at | TIMESTAMPTZ | YES | | Soft delete timestamp |

**Employee Status Enum**: `pre_boarding`, `onboarding`, `active`, `on_leave`, `suspended`, `terminated`, `resigned`, `retired`, `deceased`

**Employment Type Enum**: `full_time`, `part_time`, `contract`, `intern`, `consultant`, `temporary`, `probation`

**PII Fields**: first_name, last_name, date_of_birth, nin, bvn, personal_email, personal_phone, residential address fields, bank account details. All encrypted with AES-256-GCM via the encryption service.

### 3.2 `employee.employee_bank_accounts`

Employee bank account details for salary payments.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| employee_id | UUID | NO | | FK to employees |
| bank_code | VARCHAR(20) | NO | | Nigerian bank code (CBN) |
| bank_name | VARCHAR(255) | NO | | Bank display name |
| account_number | VARCHAR(20) | NO | | **PII** - NUBAN account number (10 digits) |
| account_name | VARCHAR(255) | NO | | **PII** - Name on account |
| account_type | VARCHAR(50) | YES | 'savings' | savings, current |
| is_verified | BOOLEAN | YES | false | Bank account verified via Paystack/Flutterwave |
| is_primary | BOOLEAN | YES | false | Primary salary account |
| is_salary_account | BOOLEAN | YES | false | Designated for salary payments |
| payment_split_type | VARCHAR(20) | YES | | percentage, fixed, remainder |
| payment_split_value | DECIMAL(18,2) | YES | | Split amount or percentage |

### 3.3 `employee.employee_pension_info`

Nigerian pension contribution configuration.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| employee_id | UUID | NO | | FK to employees |
| pfa_code | VARCHAR(20) | NO | | Pension Fund Administrator code |
| pfa_name | VARCHAR(255) | NO | | PFA display name |
| rsa_pin | VARCHAR(15) | NO | | **PII** - Retirement Savings Account PIN (UNIQUE) |
| employee_contribution_rate | DECIMAL(5,2) | YES | 8.00 | Employee contribution % (PenCom minimum: 8%) |
| employer_contribution_rate | DECIMAL(5,2) | YES | 10.00 | Employer contribution % (PenCom minimum: 10%) |
| voluntary_contribution_rate | DECIMAL(5,2) | YES | 0 | Additional voluntary contribution % |
| effective_date | DATE | NO | CURRENT_DATE | When this config takes effect |

---

## 4. Payroll Schema

### 4.1 `payroll_runs`

Payroll processing run record.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| tenant_id | UUID | NO | | FK to tenants |
| company_id | UUID | NO | | FK to companies |
| period_id | UUID | NO | | FK to payroll_periods |
| run_number | VARCHAR(50) | NO | | Auto-generated: PR-YYYY-MM-NNN |
| run_type | VARCHAR(50) | NO | 'regular' | regular, off_cycle, bonus, correction, reversal, 13th_month, back_pay |
| status | VARCHAR(50) | YES | 'draft' | draft, processing, validation_failed, pending_approval, approved, payment_processing, paid, reversed |
| total_employees | INT | YES | 0 | Count of employees in run |
| total_gross | BIGINT | YES | 0 | Sum of all gross pay (kobo) |
| total_net | BIGINT | YES | 0 | Sum of all net pay (kobo) |
| total_deductions | BIGINT | YES | 0 | Sum of all deductions (kobo) |
| total_paye | BIGINT | YES | 0 | Sum of PAYE tax (kobo) |
| total_pension_employee | BIGINT | YES | 0 | Sum of employee pension (kobo) |
| total_pension_employer | BIGINT | YES | 0 | Sum of employer pension (kobo) |
| total_nhf | BIGINT | YES | 0 | Sum of NHF deductions (kobo) |
| currency | VARCHAR(3) | YES | 'NGN' | Payment currency |
| current_approval_level | INT | YES | 0 | Current approval step |
| required_approval_levels | INT | YES | 1 | Total approvals needed |
| created_by | UUID | NO | | User who created the run |
| approved_by | UUID | YES | | Final approver |
| validation_errors | JSONB | YES | | Array of validation failure details |

**Business Rules**: Runs progress through a strict state machine: draft -> processing -> pending_approval -> approved -> paid. Reversal creates a new correction run. All monetary values stored in kobo (int64) to avoid floating-point errors.

### 4.2 `payroll_entries`

Individual employee payroll calculation result.

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | NO | gen_random_uuid() | Primary key |
| run_id | UUID | NO | | FK to payroll_runs |
| employee_id | UUID | NO | | FK to employees |
| gross_salary | BIGINT | NO | | Gross salary (kobo) |
| basic_salary | BIGINT | NO | | Basic salary (kobo) |
| total_earnings | BIGINT | YES | 0 | Sum of all earnings (kobo) |
| total_deductions | BIGINT | YES | 0 | Sum of all deductions (kobo) |
| net_pay | BIGINT | YES | 0 | Take-home pay (kobo) |
| taxable_income | BIGINT | YES | 0 | Income subject to PAYE (kobo) |
| cra_amount | BIGINT | YES | 0 | Consolidated Relief Allowance (kobo) |
| paye_amount | BIGINT | YES | 0 | PAYE income tax (kobo) |
| effective_tax_rate | DECIMAL(5,2) | YES | | Effective tax percentage |
| pension_employee | BIGINT | YES | 0 | Employee pension contribution (kobo) |
| pension_employer | BIGINT | YES | 0 | Employer pension contribution (kobo) |
| nhf_amount | BIGINT | YES | 0 | National Housing Fund (kobo) |
| nsitf_amount | BIGINT | YES | 0 | NSITF contribution (kobo) |
| nhis_amount | BIGINT | YES | 0 | NHIS contribution (kobo) |
| ytd_gross | BIGINT | YES | 0 | Year-to-date gross (kobo) |
| ytd_paye | BIGINT | YES | 0 | Year-to-date PAYE (kobo) |
| ytd_net_pay | BIGINT | YES | 0 | Year-to-date net pay (kobo) |
| payment_status | VARCHAR(50) | YES | 'pending' | pending, scheduled, processing, paid, failed |
| is_prorated | BOOLEAN | YES | false | Mid-month hire/exit proration |
| proration_factor | DECIMAL(10,6) | YES | 1.0 | Proration multiplier (0.0 - 1.0) |

**Nigerian Tax Calculation**: CRA = Higher of (NGN 200,000 or 1% of gross) + 20% of gross. Taxable income = Gross - CRA - Pension - NHF. PAYE bands: first NGN 300K at 7%, next NGN 300K at 11%, next NGN 500K at 15%, next NGN 500K at 19%, next NGN 1.6M at 21%, above NGN 3.2M at 24%.

---

## 5. Enumerated Types

### 5.1 Employee Status

| Value | Description | Transitions From |
|-------|-------------|-----------------|
| pre_boarding | Hired but not started | (initial) |
| onboarding | In onboarding process | pre_boarding |
| active | Actively employed | onboarding, on_leave, suspended |
| on_leave | On approved leave | active |
| suspended | Temporarily suspended | active |
| terminated | Employment terminated | active, suspended |
| resigned | Voluntary resignation | active |
| retired | Retired | active |
| deceased | Deceased | any |

### 5.2 Payroll Run Type

| Value | Description |
|-------|-------------|
| regular | Monthly salary payment |
| off_cycle | Unscheduled payment |
| bonus | Performance/year-end bonus |
| correction | Adjustment to prior run |
| reversal | Full reversal of a run |
| 13th_month | 13th month salary (December) |
| back_pay | Retroactive pay adjustment |

### 5.3 Job Change Type

| Value | Description |
|-------|-------------|
| promotion | Grade/title increase |
| demotion | Grade/title decrease |
| lateral_transfer | Same-level move |
| department_change | Move to different department |
| location_change | Relocation to different office |
| manager_change | New reporting line |
| salary_adjustment | Salary change without title change |
| contract_conversion | e.g., intern to full-time |

---

## 6. JSONB Field Schemas

### 6.1 `employees.metadata`

```json
{
  "custom_fields": {
    "shirt_size": "L",
    "emergency_language": "Yoruba"
  },
  "onboarding_notes": "Requires ergonomic chair",
  "tags": ["remote", "engineering", "senior"]
}
```

### 6.2 `payroll_entries.validation_errors`

```json
[
  {
    "code": "MISSING_TAX_STATE",
    "field": "tax_state",
    "message": "Employee has no tax state configured",
    "severity": "error"
  },
  {
    "code": "SALARY_BELOW_MINIMUM",
    "field": "gross_salary",
    "message": "Gross salary below national minimum wage",
    "severity": "warning"
  }
]
```

### 6.3 `tenants.settings`

```json
{
  "locale": "en-NG",
  "date_format": "DD/MM/YYYY",
  "fiscal_year_start": 1,
  "default_leave_days": 20,
  "probation_months": 3,
  "payroll_approval_levels": 2,
  "enable_geofencing": true,
  "geofence_radius_meters": 200,
  "enable_mfa": true,
  "password_expiry_days": 90
}
```

---

## 7. Foreign Key Relationships

| Parent Table | Child Table | FK Column | On Delete |
|-------------|-------------|-----------|-----------|
| core.tenants | core.users | tenant_id | RESTRICT |
| core.users | core.sessions | user_id | CASCADE |
| employee.legal_entities | employee.employees | legal_entity_id | RESTRICT |
| employee.employees | employee.employee_bank_accounts | employee_id | CASCADE |
| employee.employees | employee.employee_tax_info | employee_id | CASCADE |
| employee.employees | employee.employee_pension_info | employee_id | CASCADE |
| employee.employees | employee.employee_emergency_contacts | employee_id | CASCADE |
| employee.employees | employee.employee_dependents | employee_id | CASCADE |
| employee.employees | employee.employee_documents | employee_id | CASCADE |
| employee.employees | employee.employee_timeline | employee_id | CASCADE |
| employee.departments | employee.departments | parent_department_id | RESTRICT |
| employee.employees | employee.employees | reports_to_id | SET NULL |
| payroll_runs | payroll_entries | run_id | RESTRICT |
| payroll_entries | payroll_entry_items | entry_id | CASCADE |
| payroll_periods | payroll_runs | period_id | RESTRICT |

---

## 8. Data Sensitivity Classification

| Classification | Fields | Encryption | Access |
|---------------|--------|------------|--------|
| Highly Sensitive | nin, bvn, rsa_pin, bank_account_number, password_hash | AES-256-GCM field-level | HR Admin only |
| Sensitive | first_name, last_name, date_of_birth, personal_email, personal_phone, address, tax_id | AES-256-GCM field-level | HR + Manager |
| Internal | work_email, salary, department, job_title, hire_date | Database-level encryption | Authenticated users |
| Public | employee_number, department name, location name | None | Any authenticated user |
