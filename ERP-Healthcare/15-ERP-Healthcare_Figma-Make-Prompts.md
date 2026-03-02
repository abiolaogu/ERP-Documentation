# Figma & Make Prompts -- ERP-Healthcare
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make (Integromat) automation prompts for the ERP-Healthcare module (AfriHealth Platform). ERP-Healthcare is the healthcare vertical of the consolidated ERP platform, covering patient management, hospital administration, appointment scheduling, laboratory services, pharmacy and prescriptions, telemedicine, HMO/insurance management, AI-powered diagnostics, maternal and child health (MCH), immunization tracking, population health surveillance, quality metrics, supply chain management, IoT device integration, call center operations, chatbot services, health information exchange (HIE), blockchain-based records, marketplace, analytics, notifications, multi-tenant organization management, and payment processing.

All prompts reference real services (patient-service, hospital-service, appointment-service, lab-service, pharmacy-service, telemedicine-service, hmo-service, ai-diagnosis-service, mch-service, immunization-service, population-health-service, quality-metrics-service, supply-chain-service, iot-service, call-center-service, chatbot-service, hie-service, blockchain-service, marketplace-service, analytics-service, notification-service, organization-service, tenant-service, payment-service, ml-service, ai-voice-service, asset-management-service, compliance-service, surveillance-service) and align with the AIDD guardrails in `erp/aidd.guardrails.yaml`.

The platform implements multi-tenancy via subdomain or `X-Tenant-ID` header (e.g., `lagoshealth.afrihealth.com`), supports hierarchical tenant relationships (hospital chains), and uses Go/Gin with PostgreSQL backend.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance; color contrast >= 4.5:1 text, >= 3:1 UI components
- Minimum 44x44px touch targets on all interactive elements
- Clear focus-visible outlines (2px solid) for keyboard navigation
- Plain-language medical labels with terminology tooltips (e.g., "Blood Pressure" with "Systolic/Diastolic mmHg" tooltip)
- Semantic HTML landmarks reflected in Figma layer naming
- Screen reader annotations for vital sign charts, lab result graphs, and medical imaging thumbnails
- Color-blind safe palettes for clinical status indicators (combine color + icon + label)
- HIPAA-compliant UI patterns: auto-lock after 5 min idle, PHI masking capability

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading; initial route bundle < 220 KB gzipped
- Skeleton UIs shown within 200ms; full clinical content within 1s on 4G
- Virtualized lists for patient registries > 100 rows
- Image assets: WebP, max 80 KB hero, DICOM viewer loaded on-demand
- Prefetch patient detail route from patient list
- Real-time data (vital signs, IoT telemetry) via WebSocket with reconnection logic

### 2.3 Reliability, Trust, And Safety
- Every clinical action (prescribe medication, modify diagnosis, discharge patient) requires confirmation with provider credential check
- Inline recovery paths on all error states ("Retry", "Go back", "Call support")
- Medication ordering: mandatory allergy cross-check and drug interaction warning before submission
- Trust signals: data source labels ("Lab verified", "Self-reported"), last-synced timestamp, EHR sync status
- Patient data access logged to compliance-service audit trail (HIPAA BAA compliance)
- Emergency override flow: "Break glass" access with mandatory justification and audit log entry

### 2.4 Observability And Testability
- Every CTA emits structured analytics: `{event, module, entity, action, user_role, tenant_id, timestamp}`
- Error boundaries at page and widget level with correlation_id
- Performance instrumentation: LCP, FID, CLS on all pages
- Feature flag integration annotated (e.g., "Visible when `telemedicine_enabled` = true")
- Clinical decision support audit: every AI diagnosis suggestion logged with confidence score

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens -- Healthcare Theme

```
Create a Figma page titled "Healthcare Design Tokens" containing the complete
token specification for the AfriHealth Healthcare platform.

COLOR PALETTE (Light Theme -- Clinical):
- Primary: #0E7490 (Medical Teal -- trust, care, clinical precision)
- Primary Hover: #0B5E75
- Primary Light: #E0F7FA
- Accent: #7C3AED (Purple -- AI features, advanced analytics)
- Success: #059669 (Stable vitals, Completed, Discharged, Normal range)
- Warning: #D97706 (Abnormal result, Pending review, Expiring)
- Danger: #DC2626 (Critical vitals, Emergency, Allergic, Overdue)
- Info: #2563EB (Informational, Scheduled, In Progress)
- Clinical-Blue: #1D4ED8 (Prescribed, Lab ordered)
- Clinical-Amber: #B45309 (Specimen collected, In processing)
- Neutral-50: #F8FAFC (Page background)
- Neutral-100: #F1F5F9 (Card background)
- Neutral-200: #E2E8F0 (Borders)
- Neutral-500: #64748B (Secondary text)
- Neutral-900: #0F172A (Primary text)
- Surface: #FFFFFF (Cards, modals)

COLOR PALETTE (Dark Theme -- NOC/Monitoring Mode):
- Background: #020617 (Near-black for extended monitoring)
- Surface: #0F172A
- Surface Elevated: #1E293B
- Primary: #22D3EE (Bright Teal for visibility on dark)
- Danger: #F87171 (Bright Red for critical alerts)
- Warning: #FBBF24
- Success: #34D399
- Text Primary: #F1F5F9
- Text Secondary: #94A3B8
- Border: #334155
Note: Dark theme is essential for NOC/monitoring dashboards, overnight nursing
stations, and telemedicine consultations in low-light environments.

CLINICAL STATUS COLOR SYSTEM (always paired with icon + label):
- Normal / Stable: #059669 + checkmark icon
- Abnormal / Warning: #D97706 + exclamation-triangle icon
- Critical / Emergency: #DC2626 + alert-octagon icon
- Pending / Processing: #2563EB + clock icon
- Inactive / Discharged: #64748B + arrow-right-circle icon

TYPOGRAPHY (Inter font family):
- Display: 30px/36px, semibold 600 -- page titles ("Patient Registry")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Lab Results")
- Heading 2: 20px/28px, medium 500 -- card titles ("Vital Signs")
- Heading 3: 16px/24px, medium 500 -- subsection headers
- Body: 14px/20px, regular 400 -- clinical notes, table cells
- Body Small: 12px/16px, regular 400 -- timestamps, dosage instructions, helper text
- Label: 14px/20px, medium 500 -- form labels, column headers
- Mono: 13px/20px, JetBrains Mono -- patient MRNs, medication codes, vital numbers
- Clinical Large: 36px/44px, bold 700 -- vital sign displays (BP: 120/80)

SPACING: 4px base (4, 8, 12, 16, 24, 32, 48px)
Card padding: 24px. Section gap: 32px. Page margin: 32px desktop, 16px mobile.

ELEVATION:
- Level 0: none
- Level 1: 0 1px 3px rgba(0,0,0,0.1) -- cards
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, clinical alerts
- Level 3: 0 8px 24px rgba(0,0,0,0.15) -- emergency alert overlays

BORDER RADIUS: 4px (badges), 8px (cards, inputs), 12px (modals), 9999px (avatars, status dots)

GRID:
- Desktop (1440px): 12-column, 72px col, 24px gutter, 280px sidebar
- Tablet (1024px): 12-column, 56px col, 16px gutter, collapsed sidebar
- Mobile (390px): 4-column, fluid, 16px gutter, bottom nav

Include both light clinical theme and dark NOC/monitoring theme side-by-side.
```

#### Prompt F-002: Component Library -- Healthcare Components

```
Create a Figma page titled "Healthcare Component Library" with the following
reusable components. Show all states and light/dark variants.

PATIENT CARD:
- 340px wide. Patient photo (or initials avatar), full name, MRN (Medical Record Number),
  date of birth, age auto-calculated, gender icon, blood type badge, allergy alert badge (red)
- Status: Active (green), Admitted (blue), Discharged (gray), Emergency (red pulsing)
- HMO/Insurance badge: "NHIS - Standard" or "Private - AXA Mansard"
- Quick actions: View Record, Schedule Appointment, Send Message
- Example: "Chioma Okafor | MRN-2024-00847 | DOB: 1985-03-12 | Age: 40 | Female |
  Blood: O+ | Allergy: Penicillin | Active | NHIS Standard"

VITAL SIGNS PANEL:
- Grid of vital sign tiles (2x3):
  -- Heart Rate: 72 bpm (green, normal range indicator)
  -- Blood Pressure: 128/82 mmHg (amber, slightly elevated)
  -- Temperature: 36.8C (green, normal)
  -- SpO2: 98% (green)
  -- Respiratory Rate: 16 breaths/min (green)
  -- Blood Glucose: 142 mg/dL (amber, elevated)
- Each tile: large number, unit, trend sparkline (last 24h), status dot
- IoT badge: "Live from BedMonitor-ICU-04" when iot-service connected
- Last updated timestamp

MEDICATION ORDER CARD:
- Drug name: "Amoxicillin 500mg"
- Route: Oral | Frequency: TDS (Three times daily) | Duration: 7 days
- Prescriber: Dr. Emeka Okafor | Date: Feb 23, 2026
- Status: Active (green), Completed (gray), Discontinued (red with strikethrough)
- Allergy warning overlay (red border + icon if patient has known allergy)
- Drug interaction warning badge (amber if interaction detected)

APPOINTMENT CARD:
- Patient name + photo, appointment type (Consultation, Follow-up, Lab, Imaging)
- Date: Feb 24, 2026 | Time: 09:30 AM | Duration: 30 min
- Provider: Dr. Adaeze Nwosu, Cardiology
- Status: Scheduled (blue), Checked In (green), In Progress (amber), Completed (green), No Show (red)
- Location: "Lagos State Teaching Hospital, Room 204" or "Telemedicine" (video icon)
- Actions: Check In, Reschedule, Cancel, Start Video Call

LAB RESULT CARD:
- Test name: "Complete Blood Count (CBC)"
- Ordered by: Dr. Emeka Okafor | Date ordered: Feb 20, 2026
- Status: Ordered -> Specimen Collected -> Processing -> Results Ready -> Reviewed
  (progress stepper visualization)
- Results table (when ready):
  | Parameter | Result | Unit | Reference Range | Flag |
  | WBC       | 11.2   | x10^9/L | 4.5-11.0   | H (High, amber) |
  | RBC       | 4.8    | x10^12/L| 4.5-5.5    | N (Normal, green) |
  | Hemoglobin| 13.5   | g/dL    | 12.0-16.0  | N |
  | Platelets | 180    | x10^9/L | 150-400    | N |
- Abnormal values highlighted with amber/red background

TELEMEDICINE WIDGET:
- Video call interface card (16:9 preview)
- Provider camera feed (large), patient camera feed (small PiP)
- Controls: Mute, Camera On/Off, Screen Share, Chat, End Call
- Connection quality indicator: Excellent/Good/Poor
- "Waiting room" state: "Dr. Nwosu will join shortly" with estimated wait time

CLINICAL TIMELINE:
- Vertical timeline for patient encounter history
- Each entry: timestamp, actor (provider avatar + name), action description, linked records
- Example entries:
  -- "Dr. Emeka Okafor admitted patient to Cardiology Ward" -- Feb 23, 10:30
  -- "Lab ordered: Troponin, CBC, BMP" -- Feb 23, 10:45
  -- "Medication prescribed: Aspirin 81mg daily" -- Feb 23, 11:00
  -- "Vital signs recorded: BP 128/82, HR 72" -- Feb 23, 11:30
  -- "Lab results ready: Troponin within normal limits" -- Feb 23, 14:00

AI DIAGNOSIS SUGGESTION CARD (ai-diagnosis-service):
- "AI Clinical Decision Support" header with AI badge
- Suggested diagnosis: "Possible Hypertension Stage 1"
- Confidence: 87% (progress bar)
- Supporting evidence: list of contributing factors
- "Accept", "Modify", "Dismiss" actions
- Disclaimer: "AI-assisted suggestion. Clinical judgment required."
- Audit link: "View reasoning chain"

SIDEBAR NAVIGATION (Healthcare):
- Sections: Dashboard, Patients, Appointments, Lab Orders, Pharmacy, Telemedicine,
  Admissions, MCH, Immunizations, HMO Claims, Supply Chain, Reports, Settings
- Role-based visibility annotations
- Badge counts: Appointments Today (24), Pending Lab (8), HMO Claims (5)
- Tenant branding area at top (logo from tenant-service branding config)

EMERGENCY ALERT BANNER:
- Full-width, red background, pulsing animation
- "EMERGENCY: Code Blue -- ICU Bed 12 -- Patient MRN-2024-00847"
- "Acknowledge" button + "View Details" link
- Auto-dismiss after acknowledgement with audit log
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Clinical Dashboard -- Provider View (1440px)

```
Design a desktop Clinical Dashboard at 1440x900px for the AfriHealth platform.
Target user: Doctor / Clinical Provider at a hospital tenant (Lagos State Teaching Hospital).

HEADER:
- Tenant branding: Hospital logo (left), "Lagos State Teaching Hospital" tenant name
- User: "Dr. Emeka Okafor, Cardiology" with avatar
- Global search (Cmd+K): search patients, orders, medications
- Notification bell (7 unread), dark/light theme toggle, settings gear

SIDEBAR (280px, collapsible):
Navigation per component library spec. Active: "Dashboard".

CONTENT -- ROW 1 (KPI Cards, 4 across):
- Card 1: "Today's Appointments" -- 24 scheduled -- 6 completed, 18 remaining
- Card 2: "Admitted Patients" -- 142 -- 8 new today, 5 discharged
- Card 3: "Pending Lab Results" -- 18 -- 3 critical flagged
- Card 4: "Telemedicine Queue" -- 5 patients waiting -- avg wait 8 min

CONTENT -- ROW 2 (Two-column, 60/40):
- Left (60%): "My Schedule Today" -- Timeline view (07:00 - 18:00)
  07:00-07:30: Ward Round (Cardiology Ward, 12 patients)
  09:00-09:30: Chioma Okafor -- Follow-up Consultation -- Room 204
  09:30-10:00: Adamu Bello -- New Patient -- Chest pain evaluation
  10:00-10:30: Break
  10:30-11:00: Telemedicine -- Fatima Ibrahim -- Post-op review (video icon)
  11:00-12:00: MDT Meeting -- Oncology case discussion
  14:00-14:30: Oluwaseun Ade -- Pre-op assessment
  14:30-15:00: Telemedicine -- Ibrahim Musa -- Medication review
  15:00-16:00: Admin time

  Each slot: patient name (clickable), type badge, status indicator, location/video link

- Right (40%): "Critical Alerts" card
  -- "Lab Critical: Chioma Okafor -- Troponin elevated 0.8 ng/mL (ref <0.04)" -- 30 min ago (red)
  -- "Vital Alert: Bed ICU-04 -- BP 180/110 mmHg" -- 45 min ago (red)
  -- "Drug Interaction: Adamu Bello -- Warfarin + Aspirin flagged" -- 1 hour ago (amber)
  -- "HMO Pre-auth required: Oluwaseun Ade -- Surgery" -- 2 hours ago (amber)
  -- "Acknowledge All" button at bottom

CONTENT -- ROW 3 (Two-column, 50/50):
- Left: "Recent Patient Activity" timeline (last 10 items)
  -- "Discharged: Mrs. Akinola -- Improved, follow-up in 2 weeks" -- 1 hour ago
  -- "Admission: Adamu Bello -- Cardiology Ward, Bed 8A" -- 2 hours ago
  -- "Prescription filled: Chioma Okafor -- Metoprolol 50mg" -- 3 hours ago

- Right: "Department Statistics" mini-charts
  -- Bed Occupancy: 87% (horizontal bar, amber threshold at 90%)
  -- Average Length of Stay: 4.2 days (trend line, last 30 days)
  -- Patient Satisfaction: 4.3/5 (star rating + trend)
  -- Readmission Rate (30-day): 3.8% (green, below 5% target)

CONTENT -- ROW 4 (Full width):
- "Ward Overview" -- Grid showing bed status for Cardiology Ward:
  Each bed cell: Bed number, patient name (if occupied), status dot
  Green=available, Blue=occupied, Amber=reserved, Red=critical patient
  8 beds x 4 rows = 32 beds displayed. Click to view patient.

STATES: Loading skeleton, empty ("No appointments today"), error with retry.
Both light theme and dark NOC monitoring theme.
```

#### Prompt F-004: Patient Registry and EHR (1440px)

```
Design two connected desktop screens at 1440x900px for the patient-service.

SCREEN A -- PATIENT REGISTRY:
- Top: "Patients" title, "Register Patient" primary button, search bar
  (search by name, MRN, phone number), filter chips
- Filters: Gender, Age Range, HMO Provider, Status (Active/Admitted/Discharged),
  Ward, Primary Diagnosis, Registration Date Range
- Active filter chips with dismiss

TABLE:
| MRN            | Patient Name    | Age | Gender | Phone          | HMO          | Primary Dx     | Status    | Last Visit | Actions |
|----------------|-----------------|-----|--------|----------------|--------------|----------------|-----------|------------|---------|
| MRN-2024-00847 | Chioma Okafor   | 40  | F      | +234 803 456 7890 | NHIS Standard | Hypertension | Active    | Feb 23     | ...     |
| MRN-2024-01203 | Adamu Bello     | 55  | M      | +234 802 123 4567 | AXA Mansard  | Chest Pain   | Admitted  | Feb 23     | ...     |
| MRN-2023-00412 | Fatima Ibrahim  | 32  | F      | +234 805 678 9012 | Leadway      | Pregnancy    | Active    | Feb 20     | ...     |
| MRN-2024-00099 | Oluwaseun Ade   | 48  | M      | +234 801 234 5678 | Private      | Hernia       | Active    | Feb 22     | ...     |
| MRN-2022-00651 | Ngozi Eze       | 67  | F      | +234 807 890 1234 | NHIS Premium | Diabetes T2  | Active    | Feb 18     | ...     |

- Pagination: "Showing 1-25 of 4,832 patients"
- Bulk actions: Export, Send Bulk Message

SCREEN B -- PATIENT ELECTRONIC HEALTH RECORD (EHR):
Top: Back breadcrumb "Patients > Chioma Okafor", patient banner (always visible):
  Photo, Name, MRN, DOB (1985-03-12), Age (40), Gender (F), Blood Type (O+),
  Allergies (Penicillin - red badge), HMO (NHIS Standard), Status (Active)

MAIN CONTENT (Tabbed, below patient banner):

Tab 1 "Overview":
- Two-column: Left=Vital Signs panel (current), Right=Active medications list
- Below: Problem list / Active diagnoses, Recent encounters timeline

Tab 2 "Encounters":
- List of all clinical encounters with date, type, provider, summary
- Click to expand: full encounter notes, orders, prescriptions

Tab 3 "Lab Results":
- Table of all lab orders with status stepper
- Trend view: select a parameter (e.g., Blood Glucose) to see graph over time
  Data points: Jan 130, Feb 142, trend line with reference range shading

Tab 4 "Medications":
- Active medications list with dosage, frequency, prescriber, start date
- Past medications (gray, with end date and reason for discontinuation)
- Drug interaction checker panel

Tab 5 "Imaging":
- Radiology orders with thumbnail previews
- DICOM viewer integration placeholder (loads on-demand)

Tab 6 "MCH" (Maternal & Child Health -- visible if applicable, from mch-service):
- Antenatal visits timeline, gestational age calculator
- Growth charts for pediatric patients

Tab 7 "Immunizations" (immunization-service):
- Vaccination schedule with completed (green check) and due (amber clock) markers
- National immunization calendar reference

Tab 8 "Billing & HMO":
- HMO eligibility status, pre-authorization requests
- Invoice history, outstanding balance
- Claims submitted to hmo-service

Tab 9 "Documents":
- Uploaded clinical documents, consent forms, referral letters
- "Upload Document" button

RIGHT SIDEBAR (320px, collapsible):
- AI Clinical Decision Support card (ai-diagnosis-service):
  "Based on recent vitals and lab results, consider: Hypertension Stage 1 (87% confidence)"
- Care team: list of providers involved with roles
- Upcoming appointments
- Recent notes from care team

Both light and dark themes. All PHI fields annotatable for masking capability.
```

#### Prompt F-005: Appointment Scheduling (1440px)

```
Design a desktop Appointment Scheduling page at 1440x900px for the appointment-service.
Target users: Front desk staff, patients (self-service), and providers.

HEADER:
- "Appointments" title
- "New Appointment" primary button
- View toggle: Calendar | List | Provider Schedule
- Date navigation: < Today > with date picker

CALENDAR VIEW (Default -- Week view):
- Left column: Time slots (07:00-18:00, 30-min increments)
- Column per provider (show 5 providers, horizontal scroll for more):
  Dr. Emeka Okafor (Cardiology) | Dr. Adaeze Nwosu (Pediatrics) |
  Dr. Tunde Bakare (Surgery) | Dr. Amina Yusuf (OB/GYN) | Dr. Kemi Ola (GP)

Appointment blocks (colored by type):
  Blue: Consultation | Green: Follow-up | Purple: Telemedicine | Red: Emergency
  Each block shows: Patient name, type, duration
  Example: 09:00-09:30 block under Dr. Okafor: "Chioma Okafor | Follow-up" (blue)

Availability: Open slots shown as subtle dashed borders (bookable)
Blocked time: Gray hatched pattern (lunch, meetings)

NEW APPOINTMENT MODAL (600px wide):
- Patient search (autocomplete from patient-service): "Search by name, MRN, or phone"
  Selected: "Chioma Okafor | MRN-2024-00847 | F, 40"
- Appointment Type: Consultation, Follow-up, Lab Visit, Imaging, Telemedicine, Emergency
- Department: dropdown with provider auto-filter
- Provider: dropdown (filtered by department + availability)
- Date picker with availability indicators (green=available, amber=few slots, red=full)
- Time slot picker (30-min intervals, only available slots shown)
- Duration: 15/30/45/60 min
- Reason: textarea
- HMO Pre-authorization toggle (if applicable, links to hmo-service)
- Telemedicine toggle: "This is a video consultation" (links to telemedicine-service)
- Reminders: SMS (24h before), Email (48h before), WhatsApp (2h before)
- "Book Appointment" primary button

LIST VIEW:
| Time  | Patient          | MRN            | Type          | Provider        | Department | Status     | Actions |
|-------|------------------|----------------|---------------|-----------------|------------|------------|---------|
| 09:00 | Chioma Okafor    | MRN-2024-00847 | Follow-up     | Dr. Emeka Okafor| Cardiology | Checked In | Start   |
| 09:30 | Adamu Bello      | MRN-2024-01203 | Consultation  | Dr. Emeka Okafor| Cardiology | Scheduled  | Check In|
| 10:00 | Grace Nwosu      | MRN-2024-00345 | Telemedicine  | Dr. Emeka Okafor| Cardiology | Waiting    | Join    |
| 10:30 | -- Open Slot --  | --             | --            | Dr. Emeka Okafor| Cardiology | Available  | Book    |

PROVIDER SCHEDULE VIEW:
- Single provider full-day view with detailed time blocks
- Patient queue panel (right): Checked-in patients waiting to be seen
  -- "Chioma Okafor -- Checked in 08:55 -- Waiting 5 min" (green)
  -- "Adamu Bello -- Checked in 09:20 -- Waiting 10 min" (amber if > 15 min)

PATIENT SELF-SERVICE BOOKING (separate view):
- Step 1: Select department/specialty
- Step 2: Select provider (with photo, rating, availability summary)
- Step 3: Select date and time (calendar with available slots)
- Step 4: Confirm and provide reason
- Step 5: Confirmation with calendar add button and reminder setup
```

#### Prompt F-006: Pharmacy and Prescription Management (1440px)

```
Design a desktop Pharmacy Dashboard at 1440x900px for the pharmacy-service.
Target user: Hospital Pharmacist.

HEADER:
- "Pharmacy" title with queue count badge
- "New Prescription" button, search bar, filter by status

KPI ROW (4 cards):
- Pending Prescriptions: 23 -- 3 urgent (red badge)
- Dispensed Today: 89
- Low Stock Alerts: 7 items
- Drug Interaction Alerts: 2 active (amber badge)

PRESCRIPTION QUEUE TABLE:
| # | Priority | Patient          | MRN            | Prescriber      | Medications         | Ordered   | Status      | Actions        |
|---|----------|------------------|----------------|-----------------|---------------------|-----------|-------------|----------------|
| 1 | URGENT   | Adamu Bello      | MRN-2024-01203 | Dr. Emeka Okafor| Aspirin 81mg, Metoprolol 50mg, Atorvastatin 20mg | 10:30 AM | Pending | Dispense, Hold |
| 2 | Normal   | Chioma Okafor    | MRN-2024-00847 | Dr. Emeka Okafor| Amlodipine 5mg, Metformin 500mg | 09:45 AM | Processing  | Complete       |
| 3 | Normal   | Fatima Ibrahim   | MRN-2023-00412 | Dr. Amina Yusuf | Folic Acid 5mg, Ferrous Sulfate 200mg | 09:00 AM | Ready | Handoff |
| 4 | STAT     | ICU Bed 04       | MRN-2024-02100 | Dr. Tunde Bakare| Adrenaline 1mg IV stat | 10:45 AM | Pending | DISPENSE NOW |

- STAT orders: red row highlight, pulsing indicator
- URGENT: amber highlight
- Status flow: Pending -> Processing -> Ready for Pickup -> Dispensed

DISPENSE SIDE PANEL (480px, opens on "Dispense" click):
- Patient header with allergies prominently displayed (red banner if allergies exist)
- Medication list with each line item:
  -- Drug: Amlodipine 5mg
  -- Form: Tablet
  -- Dose: 1 tablet
  -- Route: Oral
  -- Frequency: Once daily
  -- Duration: 30 days
  -- Quantity to dispense: 30 tablets
  -- Stock status: "In Stock (450 available)" (green) or "Low Stock (12 remaining)" (amber)
  -- Substitution available: "Generic available: Norvasc -> Amlodipine (Cost: NGN 800 vs NGN 5,500)"
- Drug interaction check results:
  -- "No interactions detected" (green) or
  -- "WARNING: Interaction detected between Warfarin and Aspirin -- increased bleeding risk" (red alert)
- Allergy cross-check: "Patient allergic to Penicillin -- No penicillin-class drugs in this order" (green)
- Patient counseling notes field
- "Confirm Dispense" button (requires pharmacist PIN)
- "Hold and Notify Prescriber" button

INVENTORY SECTION (bottom or tab):
- Low stock table:
  | Drug Name | Current Stock | Min Level | Reorder Qty | Supplier | Action |
  | Amoxicillin 500mg | 45 | 100 | 500 | MedSupply NG | Reorder |
  | Insulin Glargine | 8 | 20 | 50 | PharmaCorp | URGENT |
- Supply chain integration link (supply-chain-service)
- Expiry tracking: items expiring within 30 days

Dark theme variant for 24-hour pharmacy operations.
```

#### Prompt F-007: Telemedicine Console (1440px)

```
Design a desktop Telemedicine Console at 1440x900px for the telemedicine-service.
Target user: Doctor conducting a video consultation.

FULL-SCREEN VIDEO LAYOUT:
- Main video area (70% of screen): Patient camera feed
  -- Patient name overlay (bottom-left): "Fatima Ibrahim"
  -- Connection quality indicator (top-right): green/amber/red dots
  -- Recording indicator: red dot + "Recording" (if consent given)

- Provider self-view (PiP, bottom-right): Draggable, resizable (default 200x150px)

- CONTROL BAR (bottom center, 480px wide, floating):
  -- Microphone toggle (with level indicator)
  -- Camera toggle
  -- Screen share
  -- Chat panel toggle
  -- Virtual background toggle
  -- End Call (red button)
  -- More options: Recording, Settings, Report Issue

RIGHT PANEL (380px, toggleable):
Three tabs: "Patient Info" | "Notes" | "Chat"

Patient Info tab:
- Patient card (compact): Chioma Okafor, MRN, age, allergies
- Current vitals (from last recorded or IoT real-time if available)
- Active medications list
- Reason for visit: "3-month follow-up for hypertension management"
- Previous encounter summary

Notes tab:
- Structured note template:
  -- Chief Complaint: [editable]
  -- History of Present Illness: [editable]
  -- Assessment: [editable with ICD-10 autocomplete]
  -- Plan: [editable]
- "Use AI Scribe" button (ai-voice-service): auto-transcribe and structure notes
- "Save Notes" button

Chat tab:
- Text chat with patient (for sharing links, instructions)
- File sharing: lab results, educational materials
- Chat history persisted

POST-CALL SCREEN:
- "Consultation Summary" auto-generated
- Actions: "Create Prescription", "Order Lab", "Schedule Follow-up", "Refer"
- Rating: "How was the consultation quality?" (1-5 stars)
- "Sign and Close" button

WAITING ROOM VIEW (before joining):
- Queue: 3 patients waiting
  -- "Fatima Ibrahim -- Scheduled 10:30 -- Waiting 5 min" [Join]
  -- "Ibrahim Musa -- Scheduled 11:00 -- Waiting 2 min" [Join]
  -- "Grace Nwosu -- Walk-in -- Waiting 12 min" [Join]
- System check: Camera OK, Microphone OK, Internet Good
```

#### Prompt F-008: Population Health and Surveillance Dashboard (1440px)

```
Design a desktop Population Health Dashboard at 1440x900px for the
population-health-service and surveillance-service.
Target user: Public Health Official / Hospital Administrator.

HEADER:
- "Population Health & Disease Surveillance" title
- Region selector: "Lagos State" dropdown (or "All Regions")
- Date range: "Last 30 days" with custom range picker
- "Export Report" button, "Alert Settings" button

ROW 1 (KPI Cards, 5 across):
- Total Population Covered: 2,450,000
- Active Disease Alerts: 3 (red badge)
- Immunization Coverage: 78.4%
- Maternal Mortality Rate: 12.3 per 100,000 (trending down, green arrow)
- Under-5 Mortality Rate: 18.7 per 1,000 (trending down, green arrow)

ROW 2 (Full width -- Disease Surveillance Map):
- Choropleth map of Nigeria showing disease incidence by state/LGA
- Color gradient: green (low) -> yellow -> orange -> red (high)
- Current alert overlays: malaria hotspots (Lagos mainland, Abeokuta)
- Tooltip on hover: "Lagos Mainland -- Malaria cases: 342 this month (+18% vs last month)"
- Disease selector: Malaria, Cholera, Measles, COVID-19, Lassa Fever, Meningitis

ROW 3 (Two-column, 50/50):
- Left: "Disease Trend Lines" -- Multi-line chart
  Lines: Malaria (red), Cholera (blue), Measles (purple)
  X-axis: Weeks (last 12 weeks)
  Y-axis: Case count
  Malaria data: 280, 310, 295, 320, 340, 315, 330, 350, 342, 360, 375, 390
  Threshold line (dashed red) at 350 = "outbreak threshold"
  When line crosses threshold: annotation "Alert triggered"

- Right: "Immunization Coverage by Antigen" -- Horizontal bar chart
  BCG: 92% (green)
  OPV-3: 84% (green)
  Pentavalent-3: 81% (amber -- below 85% target)
  Measles-1: 78% (amber)
  Yellow Fever: 71% (red -- below 75% critical)
  Target line at 85% (dashed)

ROW 4 (Two-column, 60/40):
- Left (60%): "Active Alerts" table
  | Alert ID | Disease | Region | Cases | Trend | Severity | Status | Actions |
  | SRV-001 | Malaria | Lagos Mainland | 390 | +18% | HIGH | Active | Investigate |
  | SRV-002 | Cholera | Abeokuta | 45 | +120% | CRITICAL | Active | Respond |
  | SRV-003 | Measles | Kano South | 28 | +35% | MEDIUM | Monitoring | Review |

- Right (40%): "MCH Indicators" (mch-service)
  -- Antenatal visits (4+ visits): 65% (target: 80%) -- bar chart
  -- Skilled birth attendance: 72%
  -- Postpartum care within 48h: 58%
  -- Low birth weight rate: 8.2%

ROW 5 (Full width):
- "Facility Performance" heat map table:
  Rows: Top 10 facilities | Columns: Bed Occupancy, Avg Wait Time, Patient Satisfaction,
  Readmission Rate, Mortality Rate
  Cell colors: green (good), amber (warning), red (poor)
  -- Lagos State Teaching Hospital: 87%, 22min, 4.3/5, 3.8%, 1.2%
  -- Abuja National Hospital: 92%, 35min, 3.9/5, 5.1%, 1.8%

DARK THEME VARIANT (essential for NOC monitoring stations):
Design the entire dashboard in dark NOC mode with high-contrast alerts,
ambient background, and reduced eye strain for extended monitoring sessions.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Clinical Dashboard (390px)

```
Design a mobile Clinical Dashboard at 390x844px for doctors on the go.

NAVIGATION:
- Bottom tab bar: Home, Patients, Schedule, Alerts (badge: 7), Profile
- Top bar: Hospital logo, search icon, notification bell

HOME TAB:
- Greeting: "Good morning, Dr. Okafor" + date "Monday, Feb 23, 2026"
- Quick Actions (horizontal scroll of circular icons):
  New Patient, Schedule, Lab Orders, Prescribe, Telemedicine, Emergency

- "Today's Schedule" card:
  -- Compact timeline (next 3 appointments visible, "See All" link)
  -- "09:00 Chioma Okafor -- Follow-up -- Room 204" [Start]
  -- "09:30 Adamu Bello -- New -- Chest pain" [View]
  -- "10:00 Telemedicine -- Fatima Ibrahim" [Join Call]

- "Critical Alerts" card (red accent border):
  -- "Troponin elevated: Chioma Okafor (0.8 ng/mL)" -- Tap to view
  -- "Vital: ICU Bed 04 -- BP 180/110" -- Tap to view
  Count: "2 critical, 5 total alerts"

- "My Ward" summary card:
  -- Admitted: 12 patients | Critical: 2 | Stable: 10
  -- Pending orders: 5 lab, 2 imaging
  -- "View Ward" button

- "Pending Tasks" list:
  -- "Sign discharge summary: Mrs. Akinola" [Sign]
  -- "Review lab result: Adamu Bello CBC" [Review]
  -- "Approve pharmacy order: ICU Bed 04" [Approve]

All touch targets >= 44px. Pull-to-refresh. Skeleton loading.
```

#### Prompt F-010: Mobile Patient Lookup and Quick Actions (390px)

```
Design a mobile Patient Lookup flow at 390x844px.

SCREEN 1 -- "Patients" tab:
- Large search bar (top): "Search patients by name, MRN, or phone"
- Recent patients (horizontal scroll): 5 circular avatars with names below
  "Chioma O.", "Adamu B.", "Fatima I.", "Oluwaseun A.", "Ngozi E."
- "My Patients" section: list of assigned patients
  Each card: Avatar (40px), Name, MRN, Primary Dx, Ward/Bed (if admitted)
  Status dot: green (stable), amber (needs attention), red (critical)

SCREEN 2 -- "Patient Summary" (tapped from list):
- Sticky patient banner: Photo, Name, MRN, Age, Allergies (red if present)
- Quick action buttons (horizontal row):
  [Vitals] [Lab Order] [Prescribe] [Notes] [Refer] [More...]

- "Latest Vitals" card:
  2x3 grid of vital tiles (compact):
  HR: 72 | BP: 128/82 | Temp: 36.8 | SpO2: 98% | RR: 16 | Glucose: 142
  "Updated 30 min ago" | "Record New Vitals" link

- "Active Medications" (collapsible list):
  -- Amlodipine 5mg -- 1 tab OD -- Active
  -- Metformin 500mg -- 1 tab BD -- Active

- "Recent Lab Results" (collapsible):
  -- CBC: Feb 23 -- WBC slightly elevated (amber flag)
  -- Lipid Panel: Feb 20 -- All normal (green)

- "Upcoming Appointments" (collapsible):
  -- Follow-up: Mar 15, 2026, Dr. Okafor

SCREEN 3 -- "Quick Vitals Entry" (bottom sheet, 85% height):
- Large input fields optimized for one-handed entry:
  Heart Rate: [___] bpm (numeric keyboard)
  BP Systolic: [___] / Diastolic: [___] mmHg
  Temperature: [___] C
  SpO2: [___] %
  Respiratory Rate: [___] /min
  Blood Glucose: [___] mg/dL
- Auto-range validation: highlights if out of normal range
- "Save Vitals" full-width sticky button

SCREEN 4 -- "Quick Prescription" (bottom sheet):
- Drug search with autocomplete: "Amlo..." -> "Amlodipine 5mg, 10mg"
- Selected drug form, dose, frequency pickers (scroll wheels)
- Duration picker
- Allergy check result displayed
- "Prescribe" button (requires biometric/PIN confirmation)
```

#### Prompt F-011: Mobile Telemedicine Patient View (390px)

```
Design a mobile Telemedicine view at 390x844px for patients joining a video consultation.

SCREEN 1 -- "My Appointments" (from patient app):
- Upcoming appointment card with "Join Video Call" button (active 10 min before)
  "Dr. Emeka Okafor | Cardiology | Today, 10:30 AM"
  "Join Video Call" (green button, 44px height, full width)
  "Prepare for Visit" checklist:
  -- Have your medications list ready
  -- Be in a quiet, well-lit space
  -- Test your camera and microphone

SCREEN 2 -- "Pre-Call Check":
- Camera preview (self-view, large)
- Microphone level indicator
- Internet speed test result: "Good connection" (green)
- "Join Consultation" full-width button

SCREEN 3 -- "Video Consultation" (full screen):
- Provider video feed (full screen)
- Self-view PiP (top-right, 100x75px, draggable)
- Bottom control bar: Mute, Camera, Chat, End Call
- Chat overlay (slides up from bottom):
  -- Text messages with provider
  -- "Dr. Okafor shared: Lab Results PDF" with download
- Call duration: "00:12:34" (top bar)
- Connection quality dot (top bar)

SCREEN 4 -- "Post-Consultation Summary":
- "Your Visit Summary" card:
  -- Date: Feb 23, 2026 | Duration: 25 min
  -- Provider: Dr. Emeka Okafor, Cardiology
  -- Diagnosis: Hypertension Stage 1
  -- Prescriptions: Amlodipine 5mg (tap to view details)
  -- Follow-up: Scheduled for Mar 15, 2026
  -- Instructions: "Continue current medication. Monitor blood pressure daily."
- "Download Summary PDF" button
- "Rate Your Experience" (1-5 stars)
- "Book Follow-up" button
- "Go to Pharmacy" button (if prescriptions issued)
```

#### Prompt F-012: Mobile HMO/Insurance (390px)

```
Design a mobile HMO/Insurance management view at 390x844px for patients
interacting with the hmo-service.

SCREEN 1 -- "My Insurance":
- Insurance card visual (340x200px, styled like a physical card):
  -- NHIS logo (top-left)
  -- "National Health Insurance Scheme"
  -- Member: Chioma Okafor
  -- ID: NHIS-2024-LAG-0847
  -- Plan: Premium
  -- Employer: TechCorp Nigeria Ltd
  -- Valid: Jan 1 - Dec 31, 2026
  -- Barcode/QR code (bottom)

- Coverage summary cards (horizontal scroll):
  -- Outpatient: NGN 500,000 annual limit | NGN 320,000 used | NGN 180,000 remaining
  -- Inpatient: NGN 2,000,000 annual limit | NGN 0 used
  -- Dental: NGN 150,000 | NGN 45,000 used
  -- Optical: NGN 100,000 | NGN 0 used

- "Find a Provider" button (opens network search)
- "Pre-Authorization" button

SCREEN 2 -- "My Claims":
- Segment: "Active" | "Completed" | "Denied"
- Each claim card:
  -- Claim #CLM-2026-0234
  -- Date: Feb 20, 2026
  -- Provider: Lagos State Teaching Hospital
  -- Service: Cardiology Consultation + Lab Tests
  -- Amount: NGN 35,000
  -- Status: "Under Review" (amber) with progress stepper
  -- Tap to expand: itemized breakdown

SCREEN 3 -- "Request Pre-Authorization":
- Procedure type search
- Provider selection
- Estimated cost
- Supporting documents upload
- "Submit Request" button
- Expected response time: "2-4 business hours"

SCREEN 4 -- "Provider Network" (map + list):
- Map with pins for nearby in-network providers
- Filter by specialty, distance, rating
- List view below map with distance, rating, "Get Directions" link
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Tablet Clinical Dashboard (1024px)

```
Design a tablet-adapted Clinical Dashboard at 1024x768px.

ADAPTATIONS FROM DESKTOP (1440px):
- Sidebar collapsed to icon-only (72px) with tooltip labels
- KPI cards: 2x2 grid instead of 4 across
- Schedule and Alerts: stacked vertically instead of side-by-side
- Ward overview: 4 beds per row instead of 8, horizontal scroll for more
- Charts: full width, reduced height (280px instead of 380px)
- Table columns: hide non-essential columns, expand on row tap

SPECIFIC LAYOUT:
Row 1: 2x2 KPI grid
Row 2: "Today's Schedule" (full width, timeline view, scrollable)
Row 3: "Critical Alerts" (full width, scrollable list)
Row 4: "Department Stats" (2x2 mini-chart grid)
Row 5: "Ward Overview" (scrollable bed grid)

Touch optimized: all targets >= 44px, swipe gestures for schedule navigation.
Support landscape (1024x768) and portrait (768x1024).
Split-view annotation for iPadOS multitasking.
```

#### Prompt F-014: Tablet Patient EHR (1024px)

```
Design a tablet Patient EHR view at 1024x768px.

ADAPTATIONS:
- Patient banner stays sticky at top (compact: single row with key info)
- Tab bar becomes scrollable horizontal (not all tabs visible at once)
- Vital signs panel: 3x2 grid instead of full-width strip
- Lab results table: fewer columns visible, swipe-to-see-more
- Timeline: full-width, vertically scrollable
- Side panel (AI suggestions, care team): accessible via bottom sheet instead of side panel
- Medication list: swipeable cards instead of table

PORTRAIT MODE (768x1024):
- Single column layout
- Patient banner: two-row (name+MRN on row 1, vitals summary on row 2)
- Tabs become a bottom sheet selector
- Full-width cards for each content section

CRITICAL: Patient allergy banner must remain visible in all viewport orientations
and scroll positions. Red persistent bar at top below patient name.
```

---

## 4. Make Automation Prompts

### Prompt M-001: Patient Appointment Reminder Workflow

```
Create a Make scenario titled "Healthcare -- Appointment Reminders".

TRIGGER: Scheduled every hour.

STEP 1 -- Fetch Upcoming Appointments:
- Call appointment-service: GET /v1/appointments?status=scheduled&within=48h
- Filter to appointments that haven't received reminders yet

STEP 2 -- Classify Reminder Type:
- 48 hours before: Email reminder with preparation instructions
- 24 hours before: SMS reminder
- 2 hours before: WhatsApp/Push notification with directions and video link (if telemedicine)

STEP 3 -- Send Reminders:
- Call notification-service: POST /v1/notifications/bulk
  Payload per patient: { patient_id, channel: "sms"|"email"|"whatsapp"|"push",
  template: "appointment_reminder", data: { patient_name, provider_name,
  appointment_date, appointment_time, location_or_video_link, preparation_notes }}
- For telemedicine: include video call link and system check URL

STEP 4 -- Track Delivery:
- Log reminder sent status
- If delivery fails (invalid phone, bounced email): flag for front desk review
- Update appointment record with reminder timestamps

STEP 5 -- No-Show Follow-up:
- 30 minutes after scheduled time, if status != "Checked In":
  Send: "We missed you today. Would you like to reschedule?" with reschedule link
- Log no-show event for analytics-service

ERROR HANDLING: Retry 3x, alert front desk on persistent failures.
VOLUME: ~200 reminders/day per facility.
TENANT-AWARE: All operations scoped to X-Tenant-ID.
```

### Prompt M-002: Lab Result Critical Alert Pipeline

```
Create a Make scenario titled "Healthcare -- Critical Lab Result Alerts".

TRIGGER: Webhook from lab-service when result status changes to "RESULTED".

STEP 1 -- Evaluate Results:
- Parse lab result payload: { order_id, patient_id, test_name, parameters[] }
- For each parameter: compare value against critical range
  -- e.g., Potassium > 6.0 mEq/L = CRITICAL
  -- e.g., Hemoglobin < 7.0 g/dL = CRITICAL
  -- e.g., Troponin > 0.04 ng/mL = CRITICAL
  -- e.g., Blood Glucose < 50 mg/dL = CRITICAL

STEP 2 -- Route by Severity:
- CRITICAL: Immediate notification to ordering provider + on-call physician
- ABNORMAL: Notification to ordering provider (within 1 hour)
- NORMAL: Standard result available notification

STEP 3 -- Critical Alert Path:
- Push notification to provider mobile app (high priority, bypass DND)
- SMS to provider: "CRITICAL LAB: {patient_name} {test} {value} {unit}. Call {lab_phone}"
- Phone call via ai-voice-service if not acknowledged within 10 minutes
- Escalate to department head if not acknowledged within 30 minutes

STEP 4 -- Audit and Compliance:
- Log notification chain to compliance-service: who was notified, when, acknowledgment time
- Calculate turnaround time: result ready -> provider acknowledged
- Flag for quality-metrics-service if TAT exceeds threshold

STEP 5 -- Patient Notification:
- For non-critical results: notify patient via app/SMS that results are available
- Do NOT send critical values to patients directly (provider must communicate)
- For normal results: "Your lab results are ready. View in your health portal."

HIPAA COMPLIANCE: No PHI in SMS/push bodies beyond patient name. Detail accessible via secure app link only.
```

### Prompt M-003: HMO Pre-Authorization Workflow

```
Create a Make scenario titled "Healthcare -- HMO Pre-Authorization".

TRIGGER: Webhook from hmo-service when pre-authorization is requested.

STEP 1 -- Validate Request:
- Parse: { request_id, patient_id, hmo_provider, procedure_code, icd10_code,
           estimated_cost, requesting_provider, supporting_documents[] }
- Verify patient has active coverage (hmo-service)
- Verify procedure is within plan coverage

STEP 2 -- Auto-Approve Rules:
- If procedure is in auto-approve list (routine consultations, basic labs): approve immediately
- If estimated cost < NGN 50,000 and procedure is outpatient: approve immediately
- Route auto-approved to Step 5

STEP 3 -- Submit to HMO (for manual review):
- Call HMO provider API (via hie-service integration):
  POST to insurer endpoint with standardized NHIA format
- Attach supporting documents
- Set expected response SLA: 4 hours for urgent, 48 hours for elective

STEP 4 -- Track and Escalate:
- Poll HMO response every 30 minutes
- If urgent and no response in 4 hours: auto-escalate to HMO liaison officer
- If elective and no response in 48 hours: send follow-up

STEP 5 -- Complete Authorization:
- Update hmo-service with approval/denial and authorization code
- Notify requesting provider (push + email)
- Notify patient (app notification)
- If denied: include denial reason and appeal instructions

ERROR HANDLING: Retry 3x for API failures. Manual queue for unresolvable cases.
```

### Prompt M-004: Immunization Schedule Monitor

```
Create a Make scenario titled "Healthcare -- Immunization Due Reminders".

TRIGGER: Scheduled daily at 07:00 UTC.

STEP 1 -- Identify Due Immunizations:
- Call immunization-service: GET /v1/immunizations/due?within=14_days
- Cross-reference with patient-service for active patients
- Group by: Overdue (past due date), Due This Week, Due Next Week

STEP 2 -- Generate Reminders:
- For each patient with due immunization:
  -- Identify parent/guardian contact (for pediatric patients)
  -- Template: "Dear {parent_name}, {child_name}'s {vaccine_name} vaccination is
     due on {due_date}. Visit {nearest_facility} for immunization."
  -- Include facility address and operating hours from hospital-service

STEP 3 -- Send Multi-Channel:
- SMS to primary phone
- WhatsApp if registered
- In-app push notification
- For overdue (> 7 days): add phone call via call-center-service queue

STEP 4 -- Report:
- Daily immunization coverage report to Public Health unit
- Weekly trend report for population-health-service dashboard
- Flag facilities with declining immunization rates for surveillance-service

STEP 5 -- Track Completion:
- Monitor for immunization recording events
- Mark reminder as acted upon when vaccine administered
- Generate completion certificates (document service)
```

### Prompt M-005: Supply Chain Reorder Automation

```
Create a Make scenario titled "Healthcare -- Pharmacy Auto-Reorder".

TRIGGER: Webhook from supply-chain-service when stock falls below minimum level.

STEP 1 -- Validate Stock Alert:
- Parse: { item_id, item_name, current_stock, minimum_level, reorder_quantity,
           preferred_supplier, tenant_id, facility_id }
- Verify alert is not duplicate (check last order within 24h)
- Check if item is on formulary (pharmacy-service)

STEP 2 -- Auto-Order Rules:
- If current_stock < critical_level (25% of minimum): URGENT order
- If current_stock < minimum_level: STANDARD order
- If item has substitute and substitute is in stock: alert pharmacist, don't order

STEP 3 -- Generate Purchase Order:
- Call supply-chain-service: POST /v1/purchase-orders
  { items: [{ item_id, quantity, urgency }], supplier_id, facility_id }
- For orders > NGN 500,000: route to Procurement Manager for approval (supervised action per AIDD guardrails)
- For URGENT: flag for expedited shipping

STEP 4 -- Notify Stakeholders:
- Pharmacist: "Reorder placed for {item_name} x{quantity}"
- Procurement: notification for orders requiring approval
- Supplier: automated PO via email/API (marketplace-service integration)

STEP 5 -- Track Fulfillment:
- Monitor delivery status
- Update inventory on receipt confirmation
- Quality check reminder on delivery

TENANT-SCOPED: All inventory operations scoped to tenant_id.
```

### Prompt M-006: Telemedicine Session Orchestration

```
Create a Make scenario titled "Healthcare -- Telemedicine Session Setup".

TRIGGER: Webhook from appointment-service 15 minutes before a telemedicine appointment.

STEP 1 -- Prepare Session:
- Call telemedicine-service: POST /v1/sessions/create
  { appointment_id, provider_id, patient_id, scheduled_time, duration }
- Generate unique video room URL
- Pre-load patient record summary for provider

STEP 2 -- Notify Participants:
- Patient: Push + SMS with join link and system check URL
  "Your video consultation with Dr. Okafor starts in 15 minutes. Tap to join: {link}"
- Provider: In-app notification with patient summary and join link
- Include: "Please ensure camera and microphone are working"

STEP 3 -- Session Monitoring:
- Track join events (provider joined, patient joined)
- If patient hasn't joined 5 min after start: send reminder
- If patient hasn't joined 10 min after start: alert front desk for phone outreach
- Monitor connection quality via iot-service telemetry

STEP 4 -- Post-Session:
- Capture session duration, recording (if consented), chat transcript
- Trigger encounter documentation workflow
- Generate consultation summary (ai-voice-service transcription if enabled)
- Send post-visit summary to patient

STEP 5 -- Billing:
- Create billing entry in payment-service
- If HMO patient: auto-submit claim to hmo-service
- If private: generate invoice and send to patient

STEP 6 -- Quality Tracking:
- Log session metrics to quality-metrics-service:
  { wait_time, session_duration, connection_quality, patient_satisfaction, no_show_flag }
```

---

## 5. Prompt Usage Guidelines

| Step | Action | Tool |
|------|--------|------|
| 1 | Copy the prompt text from Section 3 or 4 | Clipboard |
| 2 | Open Figma and activate the Make Design / AI plugin | Figma |
| 3 | Paste the prompt into the plugin input field | Plugin UI |
| 4 | Review generated output; adjust clinical color coding, vital sign ranges | Manual |
| 5 | Connect components to shared Healthcare Design System library | Figma Libraries |
| 6 | Verify HIPAA-compliant patterns (PHI masking, auto-lock, audit links) | Manual |
| 7 | Export developer-ready assets using naming convention below | Figma Export |

**Clinical data guidelines:**
- Replace sample patient data with anonymized but realistic data for demo environments
- Verify medical terminology accuracy with clinical SME
- Ensure drug names, dosages, and reference ranges are medically accurate
- Dark/NOC theme must be validated for extended monitoring use cases

---

## 6. Output Packaging Convention

```
Deliverable naming:
  HC-{PromptID}-{Breakpoint}-{Theme}-v{Version}.fig
  Example: HC-F003-1440-light-v1.fig
  Example: HC-F008-1440-dark-noc-v1.fig

Layer naming:
  {Page}/{Section}/{Component}-{State}
  Example: ClinicalDashboard/KPICards/TodayAppointments-Default

Asset export:
  /exports/healthcare/{breakpoint}/{page}/
  Example: /exports/healthcare/1440/dashboard/ward-overview.png

Handoff tokens:
  /tokens/healthcare-tokens.json (Style Dictionary format)

Clinical icons:
  /exports/healthcare/icons/{category}/
  Example: /exports/healthcare/icons/vitals/heart-rate.svg
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Initial route bundle size | < 220 KB gzipped | Webpack bundle analyzer |
| Largest Contentful Paint (LCP) | < 1.5s | Lighthouse CI |
| First Input Delay (FID) | < 100ms | Web Vitals |
| Cumulative Layout Shift (CLS) | < 0.1 | Web Vitals |
| Time to Interactive (TTI) | < 2.5s | Lighthouse CI |
| Real-time vital update latency | < 500ms | WebSocket roundtrip |
| Patient record load (full EHR) | < 2s | Custom perf mark |
| Lab result table render (500 rows) | < 300ms | Custom perf mark |
| Telemedicine video join time | < 3s | Custom perf mark |
| API read latency (p99) | < 200ms | perf/targets.yaml SLO |
| API write latency (p99) | < 500ms | perf/targets.yaml SLO |
| Availability | >= 99.95% | perf/targets.yaml SLO |

---

## 8. AIDD Handoff Gate Template

```markdown
## Design Handoff Checklist -- ERP-Healthcare

### Visual Completeness
- [ ] All pages designed for 1440px, 1024px, and 390px breakpoints
- [ ] Light clinical theme and dark NOC/monitoring theme complete
- [ ] All component states documented (default, hover, active, disabled, error, loading, empty)
- [ ] Realistic clinical data used (no "Lorem ipsum" or "Jane Doe")
- [ ] Tenant branding placeholder areas marked

### Clinical Safety
- [ ] Drug interaction warnings visible and non-dismissible without acknowledgment
- [ ] Allergy alerts persistent and visible across all patient-context screens
- [ ] Critical lab value alerts designed with escalation visual hierarchy
- [ ] Emergency override ("break glass") flow documented
- [ ] Medication ordering includes mandatory allergy cross-check step

### Accessibility
- [ ] Color contrast ratios verified (>= 4.5:1 text, >= 3:1 UI)
- [ ] Clinical status indicators use color + icon + label (color-blind safe)
- [ ] Touch targets >= 44x44px
- [ ] Focus order annotated
- [ ] Screen reader annotations for vital sign charts and lab graphs

### HIPAA/Compliance
- [ ] PHI masking capability annotated on all patient data fields
- [ ] Auto-lock screen after 5 min idle annotated
- [ ] Audit trail access points visible
- [ ] Data source labels present ("Lab verified", "Self-reported")
- [ ] Patient consent capture points marked

### Developer Handoff
- [ ] Design tokens exported (Style Dictionary JSON)
- [ ] API data mapping documented (which service populates which component)
- [ ] WebSocket endpoints annotated for real-time components (vitals, IoT)
- [ ] Feature flag annotation points marked
- [ ] Multi-tenant branding injection points documented
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial comprehensive prompt set covering clinical dashboard, patient EHR, appointments, pharmacy, telemedicine, population health/surveillance, mobile provider and patient views, tablet adaptations, and 6 Make automation workflows |

<!-- SOVEREIGN_FIGMA_MAKE_EXPANSION_2026_03 -->
## 2026-03 Shared Infra + AIDD + GA Expansion Pack

### Scope
This section upgrades the prompt pack to align with:
- Shared infrastructure (`Hasura + ERP-DBaaS + ERP-IAM + ERP-Observability`)
- AIDD guardrails (Protected, Supervised, Autonomous execution modes)
- GA deployment quality expectations (accessibility, performance, observability, rollback-safe UX)

### Global Prompt Rules (Apply To Every Screen)
- Use production-safe copy and deterministic states for loading/error/empty/success.
- Include tenant context, role context, and policy context in all critical admin flows.
- Require explicit confirmation UX for destructive actions; show blast radius and rollback hint.
- Include instrumentation notes: event name, trace ID propagation, KPI target, and SLO linkage.
- Ensure WCAG 2.1 AA contrast and keyboard focus maps in every frame set.

### Prompt F-900: Shared Infra Control Plane Screen
```
Design a control plane view for ERP-Healthcare that shows:
1. Hasura GraphQL connectivity (latency, success %, schema drift status)
2. ERP-DBaaS health (connection pool, replica lag, failover posture)
3. ERP-IAM integration (OIDC issuer health, token validation errors, session invalidations)
4. ERP-Observability status (OTLP export rate, tracing availability, alert health)

Include: command surface, timeline, runbook links, and one-click diagnostics panel.
Add states for: fully healthy, degraded IAM, degraded DB, degraded GraphQL, global outage.
```

### Prompt F-901: Role-Aware Workspace + Guardrail Surface
```
Generate a role-aware workspace for ERP-Healthcare with:
- Role lenses: Operator, Manager, Auditor, Admin
- Guardrail panel with mode badge (Protected / Supervised / Autonomous)
- Inline policy rationale for blocked/supervised actions
- Approval request flow for supervised actions

Ensure each role sees distinct navigation and action priorities.
```

### Prompt F-902: Incident + Recovery UX
```
Design a full incident-response flow for ERP-Healthcare:
- Alert ingestion to triage board
- Impacted tenant view and blast-radius visualization
- Action timeline with runbook steps and rollback controls
- Post-incident report generator modal

Must include: trace-id copy, audit evidence export, and SLA breach indicators.
```

### Prompt F-903: Mobile-Responsive Executive Snapshot
```
Create mobile and tablet executive dashboards for ERP-Healthcare:
- KPI cards (availability, p95 latency, error budget burn, throughput)
- AI-generated narrative summary with confidence score
- Risk highlights and next-best-action CTA

Use adaptive layout breakpoints and preserve semantic heading structure.
```

### Prompt F-904: Figma Make -> Engineering Handoff Packet
```
For each completed flow, generate a handoff packet containing:
- Component inventory and token usage map
- API contract touchpoints (GraphQL operations and IAM claims)
- Test matrix (unit/integration/e2e/accessibility/performance)
- Observability event catalog and dashboard mapping
- GA release checklist references
```

### Prompt M-900: Make Automation Blueprint (Shared Infra)
```
Create a Make scenario for ERP-Healthcare that:
1. Triggers on deployment/webhook events
2. Validates GraphQL, IAM, DB, and OTLP health in sequence
3. Creates incident ticket when thresholds are breached
4. Posts status updates to operations channels
5. Writes audit trail to persistent store

Add branching for: transient failures, policy violations, and hard-stop conditions.
```

### Prompt M-901: Tenant Onboarding Orchestration
```
Generate a Make scenario for tenant onboarding in ERP-Healthcare:
- Validate tenant metadata and compliance profile
- Provision IAM roles and claims mapping
- Apply Hasura metadata/migrations checks
- Seed tenant defaults and dashboards
- Emit onboarding completion event and SLA timer
```

### Completion Criteria
- Every screen has loading/error/empty/success/access-denied variants.
- Every critical action includes telemetry mapping and guardrail annotation.
- Every flow includes desktop + tablet + mobile-responsive frames.
- Every high-risk flow includes rollback and operator escalation UX.
