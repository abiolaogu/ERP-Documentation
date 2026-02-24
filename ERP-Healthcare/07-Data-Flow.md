# Data Flow Document - AfriHealth ERP-Healthcare

## 1. Overview

This document maps the data flows across all AfriHealth services, covering patient data lifecycle, clinical workflow data, financial transactions, AI/ML data pipelines, and inter-service event flows.

---

## 2. Patient Data Lifecycle

```mermaid
flowchart TB
    subgraph "Registration"
        A1[Patient Arrives] --> A2[Search Existing Records]
        A2 --> A3{Found?}
        A3 -->|No| A4[Create New Patient]
        A3 -->|Yes| A5[Retrieve Existing Record]
        A4 --> A6[Capture Biometrics]
        A6 --> A7[Generate MRN]
        A7 --> A8[Store in PostgreSQL]
        A8 --> A9[Cache in Redis]
        A9 --> A10[Index in Elasticsearch]
        A10 --> A11[Publish patient.created]
    end

    subgraph "Clinical Data Flow"
        B1[Check-in for Encounter] --> B2[Record Vitals]
        B2 --> B3[SOAP Documentation]
        B3 --> B4[Diagnosis Entry ICD-10]
        B4 --> B5[Order Labs/Imaging]
        B5 --> B6[Prescribe Medications]
        B6 --> B7[Complete Encounter]
    end

    subgraph "Data Distribution"
        A11 --> C1[Notification Service]
        A11 --> C2[Analytics Service]
        A11 --> C3[HMO Service]
        B7 --> C4[Billing Service]
        B7 --> C5[Quality Metrics]
        B7 --> C6[Surveillance Service]
    end

    A5 --> B1
    A4 --> B1
```

---

## 3. Clinical Encounter Data Flow

```mermaid
flowchart LR
    subgraph "Input Sources"
        I1[Provider Portal]
        I2[Mobile App]
        I3[IoT Devices]
        I4[Lab Analyzers]
        I5[Voice Transcript]
    end

    subgraph "Processing"
        P1[API Gateway<br/>Auth + Tenant]
        P2[Patient Service<br/>Demographics]
        P3[Encounter Service<br/>Visit Context]
        P4[Lab Service<br/>Orders + Results]
        P5[Pharmacy Service<br/>Prescriptions]
        P6[Clinical AI<br/>CDSS Alerts]
    end

    subgraph "Storage"
        S1[(Clinical Schema<br/>encounters, notes,<br/>diagnoses, vitals)]
        S2[(LIMS Schema<br/>lab_orders,<br/>lab_results)]
        S3[(Pharmacy Schema<br/>prescriptions,<br/>dispense_records)]
        S4[(Financial Schema<br/>charges, bills)]
    end

    subgraph "Output"
        O1[Patient Chart]
        O2[Lab Reports]
        O3[Prescriptions]
        O4[Bills/Invoices]
        O5[Clinical Alerts]
        O6[Analytics Events]
    end

    I1 & I2 --> P1
    I3 --> P1
    I4 --> P4
    I5 --> P6
    P1 --> P2 & P3
    P3 --> P4 & P5 & P6
    P2 --> S1
    P3 --> S1
    P4 --> S2
    P5 --> S3
    P3 --> S4
    S1 --> O1
    S2 --> O2
    S3 --> O3
    S4 --> O4
    P6 --> O5
    S1 & S2 & S3 & S4 --> O6
```

---

## 4. Financial Data Flow

```mermaid
flowchart TB
    subgraph "Charge Capture"
        A[Encounter Completed] --> B[Auto-generate Charges]
        B --> C[Apply Fee Schedule]
        C --> D[Create Bill with Line Items]
    end

    subgraph "Insurance Processing"
        D --> E{Insurance?}
        E -->|Yes| F[Verify Eligibility]
        F --> G[Submit Claim]
        G --> H{Adjudication}
        H -->|Approved| I[Apply Insurance Payment]
        H -->|Denied| J[Patient Responsibility]
        I --> K[Calculate Patient Share<br/>Copay + Deductible]
    end

    subgraph "Payment Collection"
        K --> L[Patient Bill]
        J --> L
        E -->|No| L
        L --> M{Payment Method}
        M -->|Card| N[Paystack API]
        M -->|Mobile Money| O[Flutterwave API]
        M -->|Cash| P[Cashier Entry]
        M -->|Bank Transfer| Q[Bank Reconciliation]
        N & O & P & Q --> R[Payment Recorded]
        R --> S[Update Patient Account]
        S --> T[Update Aging Buckets]
    end

    subgraph "Revenue Analytics"
        R --> U[payment.completed Event]
        U --> V[Analytics Service]
        V --> W[Revenue Dashboard]
    end
```

---

## 5. AI/ML Data Pipeline

```mermaid
flowchart TB
    subgraph "Data Sources"
        D1[Chest X-rays<br/>DICOM/PNG]
        D2[Patient Vitals<br/>Real-time Stream]
        D3[Clinical Notes<br/>Text/Transcript]
        D4[Voice Recordings<br/>Audio Stream]
        D5[Lab Results<br/>Structured Data]
        D6[Climate Data<br/>External API]
    end

    subgraph "AI Processing Pipeline"
        P1[Imaging AI<br/>TB Detection<br/>EfficientNetB4]
        P2[Clinical AI<br/>Sepsis Prediction<br/>SOFA/qSOFA + ML]
        P3[Clinical AI<br/>Drug Safety<br/>Interaction Model]
        P4[Voice Mental Health<br/>Acoustic Analysis<br/>Depression Detection]
        P5[Clinical NLP<br/>Note Generation<br/>ICD Code Extraction]
        P6[Climate Health<br/>Outbreak Prediction]
        P7[Disease Prediction<br/>Risk Stratification]
    end

    subgraph "Output Actions"
        O1[TB Alert + Grad-CAM]
        O2[Sepsis Alert<br/>ICU Transfer Rec]
        O3[Drug Interaction<br/>Warning/Block]
        O4[Depression Score<br/>Crisis Detection]
        O5[Structured SOAP Note<br/>ICD-10 Suggestions]
        O6[Outbreak Early Warning]
        O7[Patient Risk Score]
    end

    D1 --> P1 --> O1
    D2 & D5 --> P2 --> O2
    D5 --> P3 --> O3
    D4 --> P4 --> O4
    D3 --> P5 --> O5
    D6 & D5 --> P6 --> O6
    D2 & D5 --> P7 --> O7
```

---

## 6. Event Streaming Data Flow (Redpanda)

```mermaid
flowchart LR
    subgraph "Producers"
        direction TB
        PR1[Patient Service<br/>patient.created<br/>patient.updated]
        PR2[Lab Service<br/>lab.order.created<br/>lab.result.critical]
        PR3[Pharmacy Service<br/>prescription.created<br/>drug.dispensed]
        PR4[Payment Service<br/>payment.completed<br/>payment.failed]
        PR5[Telemedicine<br/>consultation.started<br/>consultation.completed]
        PR6[HMO Service<br/>claim.submitted<br/>claim.adjudicated]
        PR7[IoT Service<br/>device.reading<br/>vital.abnormal]
        PR8[AI Services<br/>tb.detected<br/>crisis.detected]
    end

    subgraph "Redpanda Cluster"
        direction TB
        T1[clinical.patient.*]
        T2[clinical.lab.*]
        T3[clinical.pharmacy.*]
        T4[financial.payment.*]
        T5[clinical.telemedicine.*]
        T6[financial.claims.*]
        T7[iot.device.*]
        T8[ai.diagnosis.*]
    end

    subgraph "Consumers"
        direction TB
        C1[Notification Service<br/>SMS/Email/Push]
        C2[Analytics Service<br/>Dashboards/Reports]
        C3[Surveillance Service<br/>Disease Tracking]
        C4[Blockchain Service<br/>Consent Audit]
        C5[ML Service<br/>Model Retraining]
        C6[Quality Metrics<br/>KPI Calculation]
    end

    PR1 --> T1
    PR2 --> T2
    PR3 --> T3
    PR4 --> T4
    PR5 --> T5
    PR6 --> T6
    PR7 --> T7
    PR8 --> T8

    T1 --> C1 & C2
    T2 --> C1 & C2 & C3
    T3 --> C1 & C4
    T4 --> C2
    T5 --> C2 & C5
    T6 --> C2
    T7 --> C1 & C2
    T8 --> C1 & C3 & C6
```

---

## 7. Blockchain Data Flow

```mermaid
flowchart TB
    subgraph "Consent Management"
        A1[Patient Grants Consent] --> A2[Blockchain Service]
        A2 --> A3[Hyperledger Fabric<br/>ConsentContract.GrantConsent]
        A3 --> A4[Consent Stored on Ledger]
        A5[Provider Requests Data] --> A6[Blockchain Service]
        A6 --> A7[ConsentContract.VerifyConsent]
        A7 --> A8{Valid?}
        A8 -->|Yes| A9[Allow Data Access]
        A8 -->|No| A10[Deny Access]
    end

    subgraph "Drug Supply Chain"
        B1[Manufacturer Registers Drug] --> B2[RegisterDrug on Ledger]
        B2 --> B3[Drug at Manufacturer]
        B3 --> B4[TransferDrug to Distributor]
        B4 --> B5[TransferDrug to Hospital]
        B5 --> B6[Pharmacist Verifies]
        B6 --> B7[VerifyDrug on Ledger]
        B7 --> B8{Authentic?}
        B8 -->|Yes| B9[Accept into Inventory]
        B8 -->|No| B10[ALERT: Counterfeit]
    end

    subgraph "Medical Credentials"
        C1[Issue Credential NFT] --> C2[IssueCredential on Ledger]
        C2 --> C3[Credential Active]
        C4[Verify Provider] --> C5[VerifyCredential]
        C5 --> C6{Valid + Not Expired?}
        C6 -->|Yes| C7[Provider Authorized]
        C6 -->|No| C8[Provider Not Authorized]
    end
```

---

## 8. Multi-Tenant Data Isolation Flow

```mermaid
flowchart TB
    A[API Request] --> B[API Gateway]
    B --> C[Extract X-Tenant-ID Header]
    C --> D[Validate Tenant Exists]
    D --> E[Set tenant_id in Context]
    E --> F[Service Layer]
    F --> G[Repository Layer]
    G --> H[Add tenant_id WHERE clause]
    H --> I[(PostgreSQL<br/>RLS Policy Enforced)]
    I --> J[Results filtered by tenant]
    J --> K[Response to Client]

    style C fill:#f96,stroke:#333
    style H fill:#f96,stroke:#333
```

---

## 9. Cross-Border Health Data Exchange

```mermaid
flowchart TB
    subgraph "Country A (Nigeria)"
        A1[Hospital A<br/>Lagos]
        A2[AfriHealth Instance<br/>Tenant A]
    end

    subgraph "HIE Layer"
        H1[Data Exchange Agreement]
        H2[Patient Consent Check<br/>Blockchain]
        H3[FHIR R4 Bundle Creation]
        H4[Encryption AES-256]
        H5[Secure Transfer TLS 1.3]
        H6[Audit Log]
    end

    subgraph "Country B (Kenya)"
        B1[AfriHealth Instance<br/>Tenant B]
        B2[Hospital B<br/>Nairobi]
    end

    A1 --> A2
    A2 --> H1
    H1 --> H2
    H2 --> H3
    H3 --> H4
    H4 --> H5
    H5 --> B1
    B1 --> B2
    H5 --> H6
```
