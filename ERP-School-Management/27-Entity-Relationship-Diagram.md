# ERP-School-Management -- Entity Relationship Diagram

**Product:** EduCore Pro
**Version:** 1.0.0
**Date:** 2026-02-23

---

## 1. Core Domain ERD

```mermaid
erDiagram
    schools {
        uuid id PK
        varchar name
        varchar code UK
        varchar type
        varchar country
        varchar timezone
        varchar currency_code
        int academic_year_start_month
        jsonb settings
        boolean is_active
        varchar subscription_tier
        timestamp subscription_expires_at
    }

    users {
        uuid id PK
        uuid school_id FK
        varchar email UK
        varchar password_hash
        varchar first_name
        varchar last_name
        varchar role
        boolean is_active
        boolean two_factor_enabled
        varchar two_factor_secret
        timestamp last_login_at
    }

    students {
        uuid id PK
        uuid school_id FK
        uuid user_id FK
        varchar student_number UK
        varchar first_name
        varchar last_name
        date date_of_birth
        varchar gender
        int grade_level
        varchar enrollment_status
        varchar primary_language
        boolean is_special_education
        jsonb custom_fields
    }

    guardians {
        uuid id PK
        uuid school_id FK
        uuid user_id FK
        varchar first_name
        varchar last_name
        varchar relationship
        varchar email
        varchar phone
        varchar communication_preference
    }

    student_guardians {
        uuid id PK
        uuid student_id FK
        uuid guardian_id FK
        varchar relationship
        boolean is_primary
        boolean is_emergency_contact
        int priority
    }

    staff {
        uuid id PK
        uuid school_id FK
        uuid user_id FK
        varchar employee_number UK
        varchar first_name
        varchar last_name
        varchar position
        varchar department
        date hire_date
        varchar employment_status
        decimal hourly_rate
        decimal annual_salary
        jsonb qualifications
    }

    schools ||--o{ users : "has"
    schools ||--o{ students : "enrolls"
    schools ||--o{ guardians : "registers"
    schools ||--o{ staff : "employs"
    users ||--o| students : "is"
    users ||--o| guardians : "is"
    users ||--o| staff : "is"
    students ||--o{ student_guardians : "linked via"
    guardians ||--o{ student_guardians : "linked via"
```

---

## 2. Academic Domain ERD

```mermaid
erDiagram
    academic_years {
        uuid id PK
        uuid school_id FK
        varchar name
        date start_date
        date end_date
        boolean is_current
    }

    terms {
        uuid id PK
        uuid school_id FK
        uuid academic_year_id FK
        varchar name
        varchar type
        int term_number
        date start_date
        date end_date
        date mid_break_start
        date mid_break_end
        boolean is_current
    }

    curricula {
        uuid id PK
        uuid school_id FK
        varchar name
        varchar code
        varchar type
        varchar language
        varchar country
        boolean is_active
    }

    grading_scales {
        uuid id PK
        uuid curriculum_id FK
        varchar name
        varchar type
        decimal passing_score
        boolean is_default
    }

    grade_levels {
        uuid id PK
        uuid grading_scale_id FK
        varchar grade
        varchar description
        decimal min_score
        decimal max_score
        decimal gpa_value
        int points
    }

    subjects {
        uuid id PK
        uuid school_id FK
        varchar name
        varchar code
        varchar category
        varchar subject_type
        int credit_hours
        decimal passing_score
        boolean is_active
    }

    classes {
        uuid id PK
        uuid school_id FK
        uuid academic_year_id FK
        varchar name
        varchar code
        int max_capacity
        int current_enrollment
        varchar room_number
    }

    class_students {
        uuid id PK
        uuid class_id FK
        varchar student_id FK
        int roll_number
        boolean is_active
    }

    timetable_slots {
        uuid id PK
        uuid school_id FK
        uuid class_id FK
        varchar day_of_week
        int period_number
        varchar start_time
        varchar end_time
        varchar subject_name
        varchar teacher_name
        varchar room
        boolean is_break
    }

    assessments {
        uuid id PK
        uuid school_id FK
        uuid subject_id FK
        uuid class_id FK
        uuid term_id FK
        varchar title
        varchar type
        decimal max_score
        decimal weight
        datetime scheduled_date
        datetime due_date
    }

    student_grades {
        uuid id PK
        varchar student_id FK
        uuid assessment_id FK
        decimal score
        varchar grade
        varchar feedback
        varchar status
        datetime graded_at
    }

    term_summaries {
        uuid id PK
        varchar student_id FK
        uuid subject_id FK
        uuid term_id FK
        decimal average_score
        varchar final_grade
        decimal gpa_value
        int class_rank
    }

    academic_years ||--o{ terms : "contains"
    curricula ||--o{ grading_scales : "defines"
    grading_scales ||--o{ grade_levels : "has"
    classes ||--o{ class_students : "enrolls"
    classes ||--o{ timetable_slots : "scheduled in"
    subjects ||--o{ assessments : "assessed via"
    terms ||--o{ assessments : "within"
    assessments ||--o{ student_grades : "graded as"
    subjects ||--o{ term_summaries : "summarized in"
    terms ||--o{ term_summaries : "for"
```

---

## 3. Finance Domain ERD

```mermaid
erDiagram
    fee_structures {
        uuid id PK
        varchar school_id FK
        varchar name
        varchar academic_year
        varchar term
        varchar fee_type
        decimal amount
        varchar currency_code
        datetime due_date
        boolean allows_installment
        int max_installments
        decimal late_payment_fee
    }

    student_fees {
        uuid id PK
        varchar school_id FK
        varchar student_id FK
        uuid fee_structure_id FK
        decimal original_amount
        decimal discount_amount
        decimal penalty_amount
        decimal total_amount
        decimal amount_paid
        decimal balance
        varchar status
        datetime due_date
        boolean is_installment_plan
    }

    fee_installments {
        uuid id PK
        varchar school_id FK
        uuid student_fee_id FK
        int installment_number
        decimal amount
        decimal amount_paid
        datetime due_date
        varchar status
    }

    fee_payments {
        uuid id PK
        varchar school_id FK
        uuid student_fee_id FK
        uuid installment_id FK
        varchar receipt_number
        decimal amount
        varchar currency_code
        varchar payment_method
        varchar transaction_reference
        datetime payment_date
        boolean is_reversed
    }

    fee_discounts {
        uuid id PK
        varchar school_id FK
        varchar student_id FK
        uuid student_fee_id FK
        varchar discount_type
        decimal discount_value
        boolean is_percentage
        decimal calculated_amount
    }

    invoices {
        uuid id PK
        varchar school_id FK
        varchar student_id FK
        varchar invoice_number UK
        decimal total_amount
        decimal paid_amount
        decimal balance_amount
        varchar currency
        datetime due_date
        varchar status
    }

    invoice_items {
        uuid id PK
        uuid invoice_id FK
        uuid fee_structure_id FK
        varchar description
        decimal amount
        int quantity
        decimal total_amount
    }

    vendors {
        uuid id PK
        varchar school_id FK
        varchar name
        varchar code
        varchar category
        varchar status
    }

    assets {
        uuid id PK
        varchar school_id FK
        varchar asset_tag
        varchar name
        varchar category
        varchar status
        varchar condition
        decimal purchase_price
    }

    financial_aids {
        uuid id PK
        varchar school_id FK
        varchar name
        varchar type
        varchar criteria
        decimal amount_value
        boolean is_percentage
    }

    fee_structures ||--o{ student_fees : "generates"
    student_fees ||--o{ fee_installments : "split into"
    student_fees ||--o{ fee_payments : "paid via"
    fee_installments ||--o{ fee_payments : "paid via"
    student_fees ||--o{ fee_discounts : "discounted by"
    invoices ||--o{ invoice_items : "contains"
    vendors ||--o{ vendor_contacts : "has"
    assets ||--o{ maintenance_tasks : "maintained by"
```

---

## 4. LMS Domain ERD

```mermaid
erDiagram
    lms_courses {
        uuid id PK
        varchar region
        varchar slug UK
        varchar title
        varchar description
        varchar difficulty
        varchar status
        boolean is_free
        decimal price_usd
        int enrollment_count
        decimal avg_rating
    }

    lms_modules {
        uuid id PK
        varchar region
        uuid course_id FK
        varchar title
        int order_num
        int duration_minutes
    }

    lms_lessons {
        uuid id PK
        varchar region
        uuid module_id FK
        varchar title
        varchar type
        int order_num
        int duration_minutes
        varchar content_type
        boolean is_free
        boolean is_required
    }

    lms_enrollments {
        uuid id PK
        varchar region
        uuid user_id FK
        uuid course_id FK
        varchar status
        decimal progress_pct
        int lessons_completed
        int total_lessons
        datetime enrolled_at
        datetime completed_at
    }

    lms_certificates {
        uuid id PK
        varchar region
        uuid user_id FK
        uuid course_id FK
        uuid enrollment_id FK
        varchar certificate_number UK
        datetime issue_date
        varchar verification_url
    }

    lms_courses ||--o{ lms_modules : "contains"
    lms_modules ||--o{ lms_lessons : "contains"
    lms_courses ||--o{ lms_enrollments : "has"
    lms_enrollments ||--o| lms_certificates : "earns"
```

---

## 5. Communication Domain ERD

```mermaid
erDiagram
    messages {
        uuid id PK
        uuid school_id FK
        uuid sender_id FK
        uuid recipient_id FK
        varchar subject
        text body
        boolean is_read
        varchar priority
        uuid parent_message_id FK
        jsonb attachments
    }

    announcements {
        uuid id PK
        uuid school_id FK
        varchar title
        text content
        uuid author_id FK
        text_array target_audience
        int_array grade_levels
        varchar priority
        timestamp published_date
        timestamp expiry_date
        boolean is_published
    }

    messages ||--o{ messages : "replies to"
```

---

## 6. Auth & Security Domain ERD

```mermaid
erDiagram
    auth_users {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar role
        varchar status
        boolean mfa_enabled
        varchar mfa_secret
        text_array mfa_backup_codes
        int failed_login_attempts
        datetime account_locked_until
    }

    sessions {
        uuid id PK
        uuid user_id FK
        varchar refresh_token_hash
        varchar device_type
        varchar ip_address
        boolean is_revoked
        datetime expires_at
    }

    oauth_accounts {
        uuid id PK
        uuid user_id FK
        varchar provider
        varchar provider_user_id
        varchar access_token
        datetime token_expires_at
    }

    blockchain_credentials {
        uuid id PK
        uuid student_id FK
        varchar credential_type
        varchar title
        date issued_date
        varchar blockchain_hash UK
        varchar transaction_id
        bigint block_number
        varchar ipfs_hash
        varchar verification_url
        boolean is_revoked
    }

    audit_logs {
        uuid id PK
        uuid school_id FK
        uuid user_id FK
        varchar action
        varchar entity_type
        uuid entity_id
        jsonb old_values
        jsonb new_values
        inet ip_address
    }

    auth_users ||--o{ sessions : "has"
    auth_users ||--o{ oauth_accounts : "linked to"
```

---

## 7. Student Extended Domain ERD

```mermaid
erDiagram
    student_dietary_profiles {
        uuid id PK
        varchar school_id FK
        varchar student_id FK
        text_array dietary_restrictions
        text_array food_allergies
        jsonb allergy_details
        text_array food_preferences
        varchar medical_conditions
    }

    incidents {
        uuid id PK
        varchar school_id FK
        varchar incident_number UK
        varchar title
        text description
        varchar severity
        varchar status
        varchar student_id FK
        varchar category_id FK
    }

    incident_updates {
        uuid id PK
        uuid incident_id FK
        text note
        varchar status_change
    }

    academic_progress {
        uuid id PK
        varchar student_id FK
        varchar academic_year_id
        varchar term_id
        decimal gpa
        decimal cumulative_gpa
        int total_credits
        int credits_earned
        decimal attendance_percentage
        int class_rank
    }

    transcripts {
        uuid id PK
        varchar student_id FK
        varchar issued_to
        datetime issued_date
        boolean is_official
        varchar url
    }

    attendance_records {
        uuid id PK
        uuid school_id FK
        uuid student_id FK
        uuid section_id FK
        date date_val
        varchar status
        time arrival_time
        int minutes_late
        text reason
    }

    incidents ||--o{ incident_updates : "updated by"
```

---

## 8. Table Count Summary

| Domain | Tables | Key Entities |
|---|---|---|
| Core (Schools/Users) | 5 | schools, users, students, guardians, staff |
| Academic | 10 | curricula, academic_years, terms, classes, subjects, assessments, grades |
| Finance | 15 | fee_structures, invoices, payments, vendors, assets, financial_aid |
| LMS | 8 | courses, modules, lessons, enrollments, certificates |
| Communication | 3 | messages, announcements, payment_reminders |
| Auth/Security | 5 | sessions, oauth_accounts, blockchain_credentials, audit_logs |
| Student Extended | 6 | dietary_profiles, incidents, academic_progress, transcripts, attendance |
| **Total** | **52+** | |
