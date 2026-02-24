# ERP-HCM Product Requirements Document (PRD)

## Document Control
| Field | Value |
|-------|-------|
| Product | ERP-HCM (Human Capital Management) |
| Version | 1.0.0 |
| Date | 2026-02-23 |
| Status | Approved |
| SKU | erp.hcm |

---

## 1. Product Vision

ERP-HCM is a comprehensive Human Capital Management platform that enables organizations to manage the entire employee lifecycle from hire to retire. It consolidates core HR, payroll, recruitment, performance, learning, attendance, benefits, and workforce planning into a unified, multi-tenant SaaS platform with first-class support for African markets (particularly Nigeria) while being globally capable.

### 1.1 Target Users

| Persona | Description | Key Needs |
|---------|-------------|-----------|
| Employee | Individual contributor | Self-service, payslips, leave, learning |
| Manager | Team lead / dept head | Approvals, team dashboards, performance reviews |
| HR Admin | HR department staff | Employee management, policy configuration, reporting |
| Payroll Admin | Finance/payroll staff | Payroll processing, statutory compliance, disbursement |
| C-Suite Executive | CEO, CHRO, CFO | Workforce analytics, headcount planning, cost analysis |
| IT Admin | System administrator | Configuration, integrations, security |

### 1.2 Target Market

- Mid-market to enterprise organizations (100-50,000 employees)
- Primary: Nigerian/African organizations
- Secondary: Global organizations with African presence
- Tertiary: Global organizations seeking open, configurable HCM

---

## 2. Competitive Analysis

### 2.1 Competitive Landscape

| Capability | ERP-HCM | Workday | SAP SuccessFactors | BambooHR |
|-----------|---------|---------|---------------------|----------|
| **Core HR** | Full | Full | Full | Full |
| **Nigerian Payroll** | Native PAYE/NHF/Pension | Third-party addon | Third-party addon | Not supported |
| **Multi-country Payroll** | NG native, US/UK framework | 60+ countries | 50+ countries | US only |
| **Recruitment/ATS** | Built-in + AI parsing | Built-in | Built-in | Built-in |
| **Performance (OKR/KPI)** | Full + 360 + 9-box | Full | Full | Basic |
| **LMS (SCORM)** | Built-in | Learning module | SuccessFactors Learning | Not included |
| **Time & Attendance** | Geofence + biometric | Time tracking | Time management | Time-off only |
| **Benefits/EWA** | Built-in + EWA | Benefits | Benefits | Benefits |
| **Workforce Planning** | Scenario modeling | Adaptive Planning | Workforce Analytics | Not included |
| **Self-hosted** | Yes | No (SaaS only) | No (SaaS only) | No (SaaS only) |
| **Open-source core** | Yes | No | No | No |
| **AIDD Guardrails** | Yes | No | No | No |
| **Multi-tenant** | Yes (X-Tenant-ID) | Yes | Yes | No |
| **API-first** | REST + Events | REST + SOAP | OData + REST | REST |
| **Pricing** | Competitive | $100+/user/mo | $80+/user/mo | $8+/user/mo |

### 2.2 Competitive Advantages

1. **Nigerian-first**: Only platform with native Nigerian PAYE, NHF, pension, NHIS, NSITF, ITF calculations
2. **Open architecture**: Self-hosted option, open-source core components
3. **EWA (Earned Wage Access)**: Built-in salary advance and earned wage access
4. **AIDD framework**: AI-driven operations with explicit governance guardrails
5. **Cost-effective**: Competitive pricing for African market
6. **Comprehensive**: All HR functions in one platform (vs. best-of-breed fragmentation)

---

## 3. Feature Requirements

### 3.1 P0 Features (Must-Have for v1.0)

#### 3.1.1 Employee Lifecycle Management
- **EMP-001**: Create employee profiles with personal, employment, bank, tax, pension information
- **EMP-002**: Employee number auto-generation with configurable format
- **EMP-003**: Department and position management with hierarchy
- **EMP-004**: Employee status transitions (active, on leave, suspended, terminated, resigned, retired)
- **EMP-005**: Probation management with confirmation workflow
- **EMP-006**: Employee document upload and management
- **EMP-007**: Emergency contact management
- **EMP-008**: Interactive org chart visualization
- **EMP-009**: Employee search and filtering with pagination
- **EMP-010**: Audit trail for all employee data changes

#### 3.1.2 Payroll Engine
- **PAY-001**: Nigerian PAYE tax calculation with graduated bands
- **PAY-002**: CRA (Consolidated Relief Allowance) computation
- **PAY-003**: Pension calculation (employee 8% + employer 10%)
- **PAY-004**: NHF (2.5%), NHIS, NSITF, ITF statutory deductions
- **PAY-005**: Payroll period management (create, process, approve, pay, close)
- **PAY-006**: Multiple run types (regular, off-cycle, bonus, correction, reversal, 13th month, back pay)
- **PAY-007**: Salary structure and component configuration (22 component types)
- **PAY-008**: Payslip generation (PDF)
- **PAY-009**: Multi-level approval workflow
- **PAY-010**: Year-to-date (YTD) tracking
- **PAY-011**: Proration for mid-month hires/exits
- **PAY-012**: Bank file generation (NIBSS format)

#### 3.1.3 Leave Management
- **LV-001**: Configurable leave types with policies
- **LV-002**: Leave request submission with approval workflow
- **LV-003**: Leave balance tracking and accrual
- **LV-004**: Half-day and hourly leave support
- **LV-005**: Holiday calendar management per location
- **LV-006**: Leave delegation management

#### 3.1.4 Authentication & Security
- **SEC-001**: RS256 JWT authentication with configurable expiry
- **SEC-002**: Multi-factor authentication (TOTP)
- **SEC-003**: Role-based access control (RBAC)
- **SEC-004**: Multi-tenant isolation (X-Tenant-ID)
- **SEC-005**: AES-256-GCM encryption for PII
- **SEC-006**: Audit logging on all mutations
- **SEC-007**: Rate limiting and CORS

### 3.2 P1 Features (Should-Have for v1.0)

#### 3.2.1 Recruitment & ATS
- **REC-001**: Job requisition creation with approval workflow
- **REC-002**: Candidate pipeline with configurable stages
- **REC-003**: Resume parsing framework
- **REC-004**: Assessment management
- **REC-005**: Interview scheduling
- **REC-006**: Offer management with approval chain
- **REC-007**: Talent pool management
- **REC-008**: Automated onboarding task creation on offer acceptance

#### 3.2.2 Performance Management
- **PERF-001**: OKR cycle management (quarterly, semi-annual, annual)
- **PERF-002**: Key result progress tracking with scoring
- **PERF-003**: Performance review cycles with multiple components
- **PERF-004**: 360-degree feedback (self, manager, peer, upward)
- **PERF-005**: Calibration workflow
- **PERF-006**: Competency framework management
- **PERF-007**: 9-box talent grid

#### 3.2.3 Time & Attendance
- **TA-001**: Clock-in/out with GPS coordinates
- **TA-002**: Geofence validation (Haversine distance)
- **TA-003**: Anti-spoofing (GPS accuracy, teleport detection)
- **TA-004**: Shift scheduling and management
- **TA-005**: Attendance reports
- **TA-006**: Payroll integration (attendance-based deductions)

#### 3.2.4 Benefits Administration
- **BEN-001**: Benefits plan configuration
- **BEN-002**: Open enrollment management
- **BEN-003**: Claims processing
- **BEN-004**: HMO integration adapters

### 3.3 P2 Features (Nice-to-Have, Post v1.0)

#### 3.3.1 Learning Management
- **LMS-001**: Course catalog with categories
- **LMS-002**: SCORM/xAPI package hosting
- **LMS-003**: Certificate generation
- **LMS-004**: Learning path creation
- **LMS-005**: Mandatory training assignment

#### 3.3.2 Advanced Features
- **ADV-001**: Workforce planning with scenario modeling
- **ADV-002**: Compensation cycle management
- **ADV-003**: Employee engagement surveys
- **ADV-004**: Internal communication hub
- **ADV-005**: IT asset management
- **ADV-006**: Background check integration
- **ADV-007**: Immigration/visa tracking
- **ADV-008**: Contractor management
- **ADV-009**: PEO/EOR service management
- **ADV-010**: Equity/stock option management
- **ADV-011**: Facilities management (room/desk booking)
- **ADV-012**: Flutter mobile app
- **ADV-013**: GraphQL API (Hasura)
- **ADV-014**: AI chat assistant for HR queries

---

## 4. Non-Functional Requirements

### 4.1 Performance
| Metric | Target |
|--------|--------|
| API P99 latency (read) | < 200ms |
| API P99 latency (write) | < 500ms |
| Payroll run (10K employees) | < 5 minutes |
| Concurrent users per tenant | 5,000+ |
| Search response time | < 100ms |

### 4.2 Scalability
- Horizontal scaling via Kubernetes pod autoscaling
- Database read replicas for report queries
- Redis cluster for distributed caching
- NATS cluster for event processing

### 4.3 Availability
- 99.9% uptime SLA (excludes planned maintenance)
- Multi-region deployment capability
- Automated failover for database and messaging

### 4.4 Security
- SOC 2 Type II compliance
- GDPR compliance for EU data
- NDPR compliance for Nigerian data
- Penetration testing annually
- Vulnerability scanning (automated)

### 4.5 Localization
- Languages: English, French, Spanish, Arabic
- Currency: Multi-currency with exchange rates
- Date/time: Timezone-aware per location
- Tax: Country-specific statutory engines

---

## 5. Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Employee self-service adoption | > 80% within 3 months | DAU/MAU ratio |
| Payroll processing time reduction | > 50% vs manual | Time per payroll run |
| Leave request turnaround | < 24 hours | Approval SLA |
| Recruitment time-to-hire | < 30 days | Pipeline metrics |
| Employee satisfaction (CSAT) | > 4.0/5.0 | Quarterly survey |
| System uptime | > 99.9% | Infrastructure monitoring |

---

## 6. Release Plan

| Phase | Scope | Target Date |
|-------|-------|-------------|
| v1.0-alpha | Core HR + Payroll + Leave | 2026-Q1 |
| v1.0-beta | + Recruitment + Performance + Attendance | 2026-Q1 |
| v1.0-GA | Full P0 + P1 feature set | 2026-Q1 |
| v1.1 | LMS + Workforce Planning + Benefits | 2026-Q2 |
| v1.2 | Mobile app + GraphQL + AI features | 2026-Q3 |
| v2.0 | Multi-country payroll expansion + Advanced analytics | 2026-Q4 |
