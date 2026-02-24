# ERP-School-Management -- Data Dictionary

**Product:** EduCore Pro
**Version:** 1.0.0
**Date:** 2026-02-23

---

## 1. Overview

This data dictionary catalogs all tables, columns, types, constraints, and business rules for the EduCore Pro database schema across all services.

---

## 2. Core Tables

### 2.1 schools

The root tenant entity. All school-scoped data references this table.

| Column | Type | Nullable | Default | Description |
|---|---|---|---|---|
| id | UUID | No | uuid_generate_v4() | Primary key |
| name | VARCHAR(255) | No | - | School display name |
| code | VARCHAR(50) | No | - | Unique school code (e.g., "DES001") |
| type | VARCHAR(50) | No | - | School type (elementary, middle, high, k12, private, charter, public) |
| address | TEXT | Yes | - | Full street address |
| city | VARCHAR(100) | Yes | - | City name |
| state | VARCHAR(100) | Yes | - | State/province |
| country | VARCHAR(100) | Yes | 'Nigeria' | Country name |
| postal_code | VARCHAR(20) | Yes | - | Postal/ZIP code |
| phone | VARCHAR(50) | Yes | - | School phone number |
| email | VARCHAR(255) | Yes | - | School email address |
| website | VARCHAR(255) | Yes | - | School website URL |
| logo_url | TEXT | Yes | - | School logo image URL |
| timezone | VARCHAR(100) | Yes | 'Africa/Lagos' | IANA timezone identifier |
| currency_code | VARCHAR(3) | Yes | 'NGN' | ISO 4217 currency code |
| academic_year_start_month | INT | Yes | 9 | Month academic year starts (1-12) |
| settings | JSONB | Yes | '{}' | Flexible school settings |
| is_active | BOOLEAN | Yes | true | Active/inactive flag |
| subscription_tier | VARCHAR(50) | Yes | 'professional' | Subscription level |
| subscription_expires_at | TIMESTAMP | Yes | - | Subscription expiry date |
| created_at | TIMESTAMP | Yes | CURRENT_TIMESTAMP | Record creation time |
| updated_at | TIMESTAMP | Yes | CURRENT_TIMESTAMP | Last update time |

**Indexes:** code (UNIQUE)
**Business Rules:** Code must be globally unique. Type must be one of the defined values.

---

### 2.2 users

User accounts for all roles in the system.

| Column | Type | Nullable | Default | Description |
|---|---|---|---|---|
| id | UUID | No | uuid_generate_v4() | Primary key |
| school_id | UUID | No | - | FK to schools.id |
| email | VARCHAR(255) | No | - | Login email (globally unique) |
| password_hash | VARCHAR(255) | No | - | Bcrypt hashed password |
| first_name | VARCHAR(100) | No | - | User first name |
| last_name | VARCHAR(100) | No | - | User last name |
| phone | VARCHAR(50) | Yes | - | Phone number |
| role | VARCHAR(50) | No | - | User role (super_admin, admin, principal, teacher, staff, parent, student) |
| is_active | BOOLEAN | Yes | true | Account active status |
| is_email_verified | BOOLEAN | Yes | false | Email verification status |
| last_login_at | TIMESTAMP | Yes | - | Last successful login |
| two_factor_enabled | BOOLEAN | Yes | false | MFA enabled flag |
| two_factor_secret | VARCHAR(255) | Yes | - | TOTP shared secret |
| profile_photo_url | TEXT | Yes | - | Profile picture URL |
| preferences | JSONB | Yes | '{}' | User preferences (language, notifications) |
| created_at | TIMESTAMP | Yes | CURRENT_TIMESTAMP | Record creation time |
| updated_at | TIMESTAMP | Yes | CURRENT_TIMESTAMP | Last update time |

**Indexes:** school_id, email (UNIQUE), role
**Business Rules:** Email must be globally unique. Role determines access permissions. Account locks after 5 failed login attempts.

---

### 2.3 students

Student information records.

| Column | Type | Nullable | Default | Description |
|---|---|---|---|---|
| id | UUID | No | uuid_generate_v4() | Primary key |
| school_id | UUID | No | - | FK to schools.id |
| user_id | UUID | Yes | - | FK to users.id (optional login account) |
| student_number | VARCHAR(50) | No | - | Unique student identifier |
| first_name | VARCHAR(100) | No | - | Student first name |
| middle_name | VARCHAR(100) | Yes | - | Student middle name |
| last_name | VARCHAR(100) | No | - | Student last name |
| preferred_name | VARCHAR(100) | Yes | - | Preferred/nickname |
| date_of_birth | DATE | No | - | Date of birth |
| gender | VARCHAR(20) | Yes | - | Gender (male, female, other, prefer_not_to_say) |
| grade_level | INT | No | - | Current grade (0-12) |
| enrollment_status | VARCHAR(50) | Yes | 'active' | Status (active, inactive, graduated, transferred, withdrawn, suspended, expelled) |
| email | VARCHAR(255) | Yes | - | Student email |
| phone | VARCHAR(50) | Yes | - | Student phone |
| address | TEXT | Yes | - | Home address |
| city | VARCHAR(100) | Yes | - | City |
| state | VARCHAR(100) | Yes | - | State |
| country | VARCHAR(100) | Yes | - | Country |
| profile_photo_url | TEXT | Yes | - | Photo URL |
| is_special_education | BOOLEAN | Yes | false | Special education flag |
| is_english_learner | BOOLEAN | Yes | false | ELL flag |
| is_gifted_talented | BOOLEAN | Yes | false | Gifted/talented flag |
| primary_language | VARCHAR(50) | Yes | 'English' | Primary language |
| custom_fields | JSONB | Yes | '{}' | School-defined custom fields |
| notes | TEXT | Yes | - | Administrative notes |
| created_at | TIMESTAMP | Yes | CURRENT_TIMESTAMP | Record creation time |
| updated_at | TIMESTAMP | Yes | CURRENT_TIMESTAMP | Last update time |

**Indexes:** school_id, student_number (UNIQUE), grade_level, enrollment_status, name trigram (GIN)
**Business Rules:** Student number must be unique. Grade level must be 0-12. Enrollment status transitions are audited.

---

## 3. Enumeration Types

### 3.1 User Roles

| Value | Description | Access Level |
|---|---|---|
| SUPER_ADMIN | Platform administrator | Full system access |
| SCHOOL_ADMIN | School administrator | School-wide access |
| PRINCIPAL | School principal | School-wide read, limited write |
| VICE_PRINCIPAL | Vice principal | Department-level |
| TEACHER | Classroom teacher | Class-level |
| STUDENT | Student | Self-access |
| PARENT | Parent/guardian | Child-access |
| ACCOUNTANT | Financial officer | Finance module |
| LIBRARIAN | Library staff | Library module |
| IT_ADMIN | IT administrator | Technical systems |
| RECEPTIONIST | Front desk | Visitor management |
| STAFF | General staff | Limited access |

### 3.2 Curriculum Types

| Value | Description | Region |
|---|---|---|
| WAEC | West African Examinations Council | West Africa |
| NECO | National Examinations Council | Nigeria |
| KCPE | Kenya Certificate of Primary Education | Kenya |
| KCSE | Kenya Certificate of Secondary Education | Kenya |
| ZIMSEC | Zimbabwe Schools Examination Council | Southern Africa |
| CAMBRIDGE_IGCSE | Cambridge International GCSE | Global |
| CAMBRIDGE_AS | Cambridge Advanced Subsidiary | Global |
| CAMBRIDGE_A_LEVEL | Cambridge Advanced Level | Global |
| IB_PYP | IB Primary Years Programme | Global |
| IB_MYP | IB Middle Years Programme | Global |
| IB_DP | IB Diploma Programme | Global |
| COMMON_CORE | US Common Core Standards | USA |
| AP | Advanced Placement | USA |
| GCSE | General Certificate of Secondary Education | UK |
| A_LEVEL | UK Advanced Level | UK |
| CUSTOM | School-defined curriculum | Any |

### 3.3 Assessment Types

| Value | Description | Typical Weight |
|---|---|---|
| QUIZ | Short test (10-15 mins) | 5-10% |
| TEST | Standard test (30-45 mins) | 10-15% |
| EXAM | Formal exam (1-3 hours) | 20-40% |
| MIDTERM | Mid-term examination | 15-25% |
| FINAL | End-of-term examination | 25-40% |
| ASSIGNMENT | Take-home work | 5-10% |
| PROJECT | Long-term project | 10-20% |
| PRESENTATION | Oral presentation | 5-10% |
| PRACTICAL | Hands-on assessment | 10-15% |
| LAB_WORK | Laboratory work | 5-10% |
| CLASSWORK | In-class exercises | 5-10% |
| HOMEWORK | Daily homework | 5-10% |
| PARTICIPATION | Class participation | 5% |
| ATTENDANCE | Attendance-based | 5% |
| CONTINUOUS_ASSESSMENT | Ongoing evaluation | 30-40% |

### 3.4 Payment Methods

| Value | Description | Gateway |
|---|---|---|
| CASH | Physical cash | Manual |
| BANK_TRANSFER | Bank wire transfer | Manual reconciliation |
| CARD | Credit/debit card | Stripe/Paystack/Flutterwave |
| MOBILE_MONEY | Mobile money (M-Pesa, etc.) | Flutterwave/Paystack |
| CHECK | Physical check | Manual |
| WALLET | Feeding wallet debit | Internal |
| SCHOLARSHIP | Scholarship credit | Internal |
| WAIVER | Fee waiver | Internal |
| STRIPE | Stripe payment | Stripe |
| PAYSTACK | Paystack payment | Paystack |
| FLUTTERWAVE | Flutterwave payment | Flutterwave |
| OTHER | Other methods | Manual |

### 3.5 Fee Types

| Value | Description | Typical Frequency |
|---|---|---|
| TUITION | Main school fees | Termly/Annual |
| REGISTRATION | One-time registration | One-time |
| EXAMINATION | Exam fees | Termly |
| LABORATORY | Science lab fees | Termly |
| LIBRARY | Library access fee | Annual |
| SPORTS | Sports/PE fee | Annual |
| TRANSPORTATION | Bus/transport fee | Termly |
| UNIFORM | School uniform | One-time |
| BOOKS | Textbook fee | Annual |
| FEEDING | Cafeteria/meal plan | Monthly |
| EXCURSION | Field trip fee | Per trip |
| PTA | Parent-Teacher Association | Annual |
| DEVELOPMENT | Development levy | Annual |
| OTHER | Miscellaneous | Varies |

---

## 4. JSONB Column Schemas

### 4.1 schools.settings

```json
{
  "grading": {
    "showGPA": true,
    "showRank": true,
    "decimalPlaces": 2
  },
  "attendance": {
    "autoNotifyAbsent": true,
    "notifyAfterMinutes": 30
  },
  "finance": {
    "autoGenerateInvoices": true,
    "reminderDaysBefore": [7, 3, 1],
    "lateFeesEnabled": true
  },
  "communication": {
    "defaultChannels": ["email", "sms"],
    "urgentOverride": true
  }
}
```

### 4.2 staff.qualifications

```json
[
  {
    "degree": "B.Ed",
    "institution": "University of Lagos",
    "year": 2015,
    "major": "Mathematics Education"
  }
]
```

### 4.3 sections.schedule

```json
{
  "monday": [{ "period": 1, "start": "08:00", "end": "08:45" }],
  "tuesday": [{ "period": 1, "start": "08:00", "end": "08:45" }]
}
```

---

## 5. Data Integrity Constraints

| Constraint | Table | Rule |
|---|---|---|
| Unique student per section | enrollments | UNIQUE(student_id, section_id) |
| Unique attendance | attendance_records | UNIQUE(student_id, section_id, date, period) |
| Unique grade per assessment | student_grades | UNIQUE(student_id, assessment_id) |
| Valid grade level | students | CHECK(grade_level BETWEEN 0 AND 12) |
| Valid term number | terms | CHECK(term_number BETWEEN 1 AND 4) |
| Positive amounts | fee_structures | amount > 0 |
| Balance consistency | invoices | balance = total_amount - paid_amount |

---

## 6. Audit Trail

All tables with `updated_at` columns have automatic triggers that set the timestamp on UPDATE. The `audit_logs` table captures INSERT, UPDATE, and DELETE operations with full before/after snapshots in JSONB format, including:
- User who made the change
- IP address of the request
- Timestamp of the operation
- Entity type and ID
- Old and new values as JSONB
