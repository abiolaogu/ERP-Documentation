# Entity Relationship Diagram (ERD) - AfriHealth ERP-Healthcare

## 1. Overview

AfriHealth uses 8 PostgreSQL domain schemas containing 100+ tables. This document presents the entity relationships across all domains.

---

## 2. Schema 1: Clinical Core ERD

```mermaid
erDiagram
    patients ||--o{ encounters : "has"
    patients ||--o{ diagnoses : "has"
    patients ||--o{ medications : "takes"
    patients ||--o{ allergies : "has"
    patients ||--o{ immunizations : "receives"
    patients ||--o{ vital_signs : "records"
    patients ||--o{ care_plans : "follows"
    patients ||--o{ consents : "grants"
    patients ||--o{ patient_identifiers : "has"
    patients ||--o{ emergency_contacts : "has"
    patients ||--o{ chronic_conditions : "has"

    encounters ||--o{ clinical_notes : "contains"
    encounters ||--o{ diagnoses : "yields"
    encounters ||--o{ medications : "prescribes"
    encounters ||--o{ vital_signs : "records"
    encounters ||--o{ laboratory_orders : "orders"
    encounters ||--o{ radiology_orders : "orders"
    encounters ||--o{ procedures : "performs"
    encounters ||--o{ care_plans : "creates"

    patients {
        uuid id PK
        uuid tenant_id
        varchar medical_record_number UK
        varchar first_name
        varchar last_name
        date date_of_birth
        varchar gender
        varchar phone_primary
        varchar email
        varchar national_id
        text biometric_fingerprint_hash
        boolean is_active
        tsvector search_vector
    }

    encounters {
        uuid id PK
        uuid tenant_id
        uuid patient_id FK
        varchar encounter_number UK
        varchar encounter_class
        varchar status
        timestamp actual_start
        timestamp actual_end
        uuid attending_physician_id
        uuid facility_id
        text chief_complaint
        integer readmission_risk_score
    }

    clinical_notes {
        uuid id PK
        uuid encounter_id FK
        uuid patient_id FK
        varchar note_type
        text subjective
        text objective
        text assessment
        text plan
        boolean ai_generated
        text ai_summary
        jsonb cdss_alerts
        uuid author_id
        varchar status
    }

    diagnoses {
        uuid id PK
        uuid patient_id FK
        uuid encounter_id FK
        varchar diagnosis_code
        varchar diagnosis_code_system
        text diagnosis_description
        varchar severity
        varchar clinical_status
        boolean is_chronic
        boolean ai_suggested
        decimal ai_confidence
    }

    medications {
        uuid id PK
        uuid patient_id FK
        uuid encounter_id FK
        varchar medication_name
        varchar dose
        varchar route
        varchar frequency
        varchar status
        uuid prescribed_by
        boolean allergies_checked
        boolean interactions_checked
        text ai_dosing_recommendation
        jsonb ai_interaction_warnings
    }

    vital_signs {
        uuid id PK
        uuid patient_id FK
        uuid encounter_id FK
        decimal temperature
        integer heart_rate
        integer blood_pressure_systolic
        integer blood_pressure_diastolic
        decimal oxygen_saturation
        decimal weight
        decimal height
        integer pain_score
        integer gcs_total
        boolean is_abnormal
    }

    allergies {
        uuid id PK
        uuid patient_id FK
        varchar allergen_type
        varchar allergen_code
        varchar allergen_name
        varchar severity
        varchar status
    }

    immunizations {
        uuid id PK
        uuid patient_id FK
        varchar vaccine_code
        varchar vaccine_name
        integer dose_number
        timestamp administered_date
        boolean adverse_event_reported
        boolean reported_to_registry
    }

    consents {
        uuid id PK
        uuid patient_id FK
        varchar consent_type
        varchar decision
        timestamp effective_date
        text blockchain_transaction_id
        text blockchain_hash
    }

    audit_log {
        uuid id PK
        uuid tenant_id
        varchar event_type
        varchar table_name
        uuid record_id
        uuid user_id
        uuid patient_id
        jsonb old_values
        jsonb new_values
    }
```

---

## 3. Schema 2: HIMS ERD

```mermaid
erDiagram
    organizations ||--o{ organization_departments : "has"
    organizations ||--o{ organization_facilities : "has"
    organizations ||--o{ organization_addresses : "located_at"
    organizations ||--o{ organization_contacts : "contacted_via"
    organizations ||--o{ staff_members : "employs"
    organizations ||--o{ organization_accreditations : "holds"
    organizations ||--o{ organization_relationships : "relates_to"
    organizations ||--o{ data_exchange_agreements : "agrees_to"
    organizations ||--o{ quality_measurements : "measures"
    organizations ||--o{ compliance_assessments : "undergoes"
    organizations ||--o{ medical_assets : "owns"

    organization_departments ||--o{ staff_members : "staffed_by"
    organizations ||--o{ patient_referrals : "refers_from"
    organizations ||--o{ patient_referrals : "refers_to"

    organizations {
        uuid id PK
        uuid tenant_id
        uuid parent_org_id FK
        varchar name
        organization_type type
        varchar registration_no UK
        integer bed_count
        integer staff_count
        accreditation_status accreditation_status
        boolean emergency_services
        jsonb specializations
    }

    staff_members {
        uuid id PK
        uuid organization_id FK
        uuid department_id FK
        varchar employee_id
        varchar first_name
        varchar last_name
        varchar staff_type
        varchar specialty
        varchar license_number
        boolean can_prescribe
        boolean telehealth_enabled
        varchar employment_status
    }

    quality_indicators {
        uuid id PK
        varchar code UK
        varchar name
        varchar category
        varchar domain
        varchar direction
        decimal benchmark_national
        decimal target_value
    }

    quality_measurements {
        uuid id PK
        uuid organization_id FK
        uuid indicator_id FK
        varchar reporting_period
        decimal value
        decimal target
        varchar performance_status
        varchar trend_direction
    }
```

---

## 4. Schema 3: Financial/RCM ERD

```mermaid
erDiagram
    patient_accounts ||--o{ bills : "receives"
    bills ||--o{ bill_items : "contains"
    bills ||--o{ payments : "paid_by"
    bills ||--o{ insurance_claims : "claimed_via"
    payments ||--o| refunds : "refunded"

    patient_accounts {
        uuid id PK
        uuid patient_id
        varchar account_number UK
        varchar account_type
        decimal total_charges
        decimal total_payments
        decimal current_balance
        decimal balance_0_30_days
        decimal balance_31_60_days
        decimal balance_61_90_days
        decimal balance_91_plus_days
        boolean in_collections
    }

    bills {
        uuid id PK
        uuid patient_id
        uuid hospital_id
        varchar bill_number UK
        decimal total_amount
        decimal paid_amount
        decimal balance
        varchar currency
        varchar status
        date due_date
    }

    bill_items {
        uuid id PK
        uuid bill_id FK
        varchar item_type
        varchar description
        integer quantity
        decimal unit_price
        decimal discount
        decimal total_price
    }

    payments {
        uuid id PK
        uuid patient_id
        uuid bill_id FK
        decimal amount
        varchar currency
        varchar payment_method
        varchar payment_provider
        varchar status
        varchar transaction_ref UK
    }

    insurance_claims {
        uuid id PK
        uuid bill_id FK
        uuid patient_id
        varchar claim_number UK
        decimal claim_amount
        decimal approved_amount
        varchar status
        varchar policy_number
    }
```

---

## 5. Schema 4: LIMS ERD

```mermaid
erDiagram
    laboratory_locations ||--o{ laboratory_departments : "contains"
    laboratory_departments ||--o{ test_catalog : "offers"
    test_catalog ||--o{ test_panel_components : "grouped_in"
    test_panels ||--o{ test_panel_components : "consists_of"

    lab_tests ||--o{ lab_results : "produces"
    lab_tests ||--o{ specimens : "requires"

    laboratory_locations {
        uuid id PK
        varchar location_code UK
        varchar location_name
        varchar location_type
        jsonb accreditation
    }

    test_catalog {
        uuid id PK
        varchar test_code UK
        varchar test_name
        varchar specimen_type
        varchar loinc_code
        varchar cpt_code
        decimal price
        integer tat_hours
    }

    lab_tests {
        uuid id PK
        uuid patient_id
        uuid doctor_id
        varchar test_code
        varchar test_name
        varchar status
        varchar priority
        boolean is_critical
        boolean is_abnormal
    }

    lab_results {
        uuid id PK
        uuid test_id FK
        varchar parameter_name
        varchar parameter_code
        varchar value
        varchar unit
        varchar reference_range
        boolean is_abnormal
        boolean is_critical
        varchar flag
    }

    specimens {
        uuid id PK
        uuid test_id FK
        uuid patient_id
        varchar specimen_number UK
        varchar specimen_type
        varchar status
        varchar condition
    }
```

---

## 6. Schema 5: HMO/Insurance ERD

```mermaid
erDiagram
    providers ||--o{ insurance_plans : "offers"
    insurance_plans ||--o{ enrollments : "has"
    enrollments ||--o{ verifications : "verified_by"
    enrollments ||--o{ claims : "generates"

    providers {
        uuid id PK
        varchar name
        varchar code UK
        varchar type
        boolean is_active
    }

    insurance_plans {
        uuid id PK
        uuid provider_id FK
        varchar plan_name
        varchar plan_code UK
        varchar plan_type
        varchar coverage
        decimal monthly_premium
        decimal coverage_limit
        decimal deductible
        decimal copayment
        jsonb benefits
        jsonb exclusions
    }

    enrollments {
        uuid id PK
        uuid patient_id
        uuid plan_id FK
        varchar enrollment_number UK
        varchar policy_number
        varchar member_type
        varchar status
        decimal utilized_amount
        decimal remaining_balance
    }

    claims {
        uuid id PK
        uuid enrollment_id FK
        uuid patient_id
        varchar claim_number UK
        varchar claim_type
        decimal total_amount
        decimal approved_amount
        decimal paid_amount
        varchar status
        jsonb diagnosis
        jsonb procedures
    }
```

---

## 7. Schema 6: Mental Health ERD

```mermaid
erDiagram
    mental_health_patients ||--o{ mental_health_assessments : "takes"
    mental_health_patients ||--o{ crisis_risk_assessments : "assessed_for"
    mental_health_patients ||--o{ chatbot_conversations : "has"
    mental_health_patients ||--o{ patient_therapy_enrollments : "enrolled_in"
    mental_health_patients ||--o{ voice_biomarkers : "analyzed"
    mental_health_patients ||--o{ safety_plans : "creates"
    mental_health_patients ||--o{ psychiatric_medications : "takes"

    chatbot_conversations ||--o{ chatbot_messages : "contains"
    chatbot_conversations ||--o{ chatbot_context : "maintains"

    digital_therapy_programs ||--o{ patient_therapy_enrollments : "enrolls"
    patient_therapy_enrollments ||--o{ therapy_sessions : "completes"

    support_groups ||--o{ support_group_members : "has"
    support_groups ||--o{ peer_messages : "contains"

    mental_health_patients {
        uuid id PK
        uuid patient_id FK
        varchar anonymous_user_id
        boolean is_anonymous
        boolean consent_for_research
        varchar language_preference
    }

    mental_health_assessments {
        uuid id PK
        uuid patient_id FK
        varchar assessment_type
        integer score
        varchar severity
        jsonb questions_responses
        varchar administered_by
    }

    crisis_risk_assessments {
        uuid id PK
        uuid patient_id FK
        varchar risk_level
        decimal suicide_risk_score
        text[] self_harm_indicators
        boolean immediate_intervention_triggered
        boolean emergency_services_contacted
    }

    voice_biomarkers {
        uuid id PK
        uuid patient_id FK
        decimal pitch_mean
        decimal speech_rate
        decimal jitter
        decimal shimmer
        decimal depression_probability
        decimal anxiety_probability
    }
```

---

## 8. Schema 7: Pharmacy ERD

```mermaid
erDiagram
    drugs ||--o{ inventory : "stocked"
    drugs ||--o{ prescription_items : "prescribed"
    prescriptions ||--o{ prescription_items : "contains"
    prescriptions ||--o| dispense_records : "dispensed_as"
    inventory ||--o{ dispense_records : "used_from"

    drugs {
        uuid id PK
        varchar name
        varchar generic_name
        varchar brand_name
        varchar category
        varchar form_type
        varchar strength
        boolean requires_prescription
        boolean is_controlled
        varchar blockchain_hash
    }

    prescriptions {
        uuid id PK
        uuid patient_id
        uuid doctor_id
        varchar status
        date valid_until
        text diagnosis
    }

    inventory {
        uuid id PK
        uuid drug_id FK
        uuid location_id
        varchar batch_number
        integer quantity
        integer reorder_level
        decimal unit_price
        decimal selling_price
        date expiry_date
        varchar status
    }
```

---

## 9. Schema 8: Supply Chain and Assets ERD

```mermaid
erDiagram
    suppliers ||--o{ purchase_orders : "fulfills"
    purchase_orders ||--o{ purchase_order_items : "contains"
    products ||--o{ purchase_order_items : "ordered_as"
    products ||--o{ stock_movements : "tracked_by"

    medical_assets ||--o{ maintenance_work_orders : "maintained_by"
    medical_assets ||--o{ asset_inspections : "inspected_via"
    medical_assets ||--o{ asset_calibrations : "calibrated_by"
    medical_assets ||--o{ asset_transfers : "transferred"
    medical_assets ||--o{ asset_disposals : "disposed"
    equipment_vendors ||--o{ service_contracts : "provides"

    products {
        uuid id PK
        varchar sku UK
        varchar name
        varchar category
        integer current_stock
        integer reorder_point
        decimal unit_cost
    }

    purchase_orders {
        uuid id PK
        varchar order_number UK
        uuid supplier_id FK
        varchar status
        decimal total_amount
    }

    medical_assets {
        uuid id PK
        uuid organization_id FK
        varchar asset_tag UK
        varchar serial_number
        varchar name
        asset_category category
        asset_status status
        varchar fda_classification
        decimal purchase_price
        decimal current_value
        boolean calibration_required
    }
```

---

## 10. Cross-Schema Relationships

```mermaid
graph TB
    subgraph "Clinical Core"
        P[patients]
        E[encounters]
    end

    subgraph "Financial"
        PA[patient_accounts]
        B[bills]
        PAY[payments]
    end

    subgraph "LIMS"
        LO[lab_orders]
        LR[lab_results]
    end

    subgraph "HMO"
        EN[enrollments]
        CL[claims]
    end

    subgraph "Pharmacy"
        RX[prescriptions]
        DIS[dispense_records]
    end

    subgraph "Mental Health"
        MHP[mental_health_patients]
        ASS[assessments]
    end

    P --> PA
    P --> EN
    P --> MHP
    E --> LO
    E --> RX
    E --> B
    LO --> LR
    B --> PAY
    B --> CL
    EN --> CL
    RX --> DIS
```
