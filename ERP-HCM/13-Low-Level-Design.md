# ERP-HCM Low-Level Design (LLD)

## Version 1.0.0 | Date: 2026-02-23

---

## 1. Employee Service - Sequence Diagrams

### 1.1 Create Employee

```mermaid
sequenceDiagram
    participant Client
    participant Handler as handlers.go
    participant Service as service.go
    participant NumGen as number_generator.go
    participant Validator as validator
    participant Repo as employee_repository.go
    participant Events as events.go
    participant Encrypt as encryption/service.go
    participant DB as PostgreSQL

    Client->>Handler: POST /v1/employee {CreateEmployeeRequest}
    Handler->>Handler: Parse JSON body
    Handler->>Handler: Extract tenantID from context
    Handler->>Validator: Validate request (required, email, max)
    Validator-->>Handler: Validation result

    alt Validation Failed
        Handler-->>Client: 400 Bad Request {errors}
    end

    Handler->>Service: CreateEmployee(ctx, tenantID, request)
    Service->>NumGen: GenerateEmployeeNumber(tenantID, format)
    NumGen->>DB: SELECT MAX(employee_number)
    DB-->>NumGen: Last number
    NumGen-->>Service: "EMP-2026-0042"

    Service->>Encrypt: EncryptField(bankAccountNumber)
    Encrypt-->>Service: Encrypted value
    Service->>Encrypt: EncryptField(taxID)
    Encrypt-->>Service: Encrypted value

    Service->>Repo: Create(ctx, employee)
    Repo->>DB: INSERT INTO employee.employees (...)
    DB-->>Repo: Employee record with UUID
    Repo-->>Service: *Employee

    Service->>Events: PublishEmployeeCreated(employee)
    Events-->>Service: Published to NATS

    Service-->>Handler: *Employee
    Handler-->>Client: 201 Created {employee}
```

### 1.2 Bulk Import Employees

```mermaid
sequenceDiagram
    participant Client
    participant Handler
    participant Import as import.go
    participant Validator
    participant Service
    participant DB as PostgreSQL

    Client->>Handler: POST /v1/employee/import {multipart/csv}
    Handler->>Import: ProcessCSV(file)
    Import->>Import: Parse CSV rows

    loop For each row
        Import->>Validator: ValidateRow(data)
        alt Row Invalid
            Import->>Import: Add to errors list
        else Row Valid
            Import->>Service: CreateEmployee(ctx, data)
            Service->>DB: INSERT INTO employees
        end
    end

    Import-->>Handler: ImportResult {imported: N, errors: [...]}
    Handler-->>Client: 200 OK {result}
```

---

## 2. Payroll Service - Sequence Diagrams

### 2.1 Process Payroll Run

```mermaid
sequenceDiagram
    participant Handler as payroll_handler.go
    participant Service as payroll_service.go
    participant Engine as payroll_engine.go
    participant Nigeria as nigeria/calculator.go
    participant Pension as pension_calculator.go
    participant Repo as payroll_repository.go
    participant DB as PostgreSQL

    Handler->>Service: ProcessPayrollRun(ctx, runID)
    Service->>Repo: GetPayrollRun(ctx, runID)
    Repo->>DB: SELECT * FROM payroll_runs WHERE id = $1
    DB-->>Repo: PayrollRun
    Repo-->>Service: PayrollRun

    Service->>Repo: GetEmployeesForRun(ctx, run)
    Repo->>DB: SELECT employees with salary structures
    DB-->>Repo: []EmployeePayrollData
    Repo-->>Service: []EmployeePayrollData

    loop For each employee
        Service->>Engine: CalculateEmployeePayroll(data)
        Engine->>Engine: Compute gross from structure
        Engine->>Nigeria: CalculateCRA(annualGross)
        Nigeria-->>Engine: CRA amount
        Engine->>Pension: Calculate(basic+housing+transport)
        Pension-->>Engine: Employee 8%, Employer 10%
        Engine->>Nigeria: CalculatePAYE(gross, pension, nhf, other)
        Nigeria->>Nigeria: applyTaxBands(taxableIncome)
        Nigeria-->>Engine: PAYEResult {annual, monthly, bands}
        Engine->>Engine: Calculate NHF (2.5% basic)
        Engine->>Engine: Apply other deductions
        Engine->>Engine: Calculate net pay
        Engine->>Engine: Update YTD values
        Engine-->>Service: EmployeePayrollResult
    end

    Service->>Repo: SavePayrollEntries(ctx, entries)
    Repo->>DB: INSERT INTO payroll_entries (batch)
    DB-->>Repo: Saved

    Service->>Repo: UpdateRunTotals(ctx, run, totals)
    Service-->>Handler: PayrollRunResult
```

### 2.2 PAYE Tax Calculation Detail

```mermaid
sequenceDiagram
    participant Engine
    participant Tax as tax.go
    participant Bands as payeTaxBands

    Engine->>Tax: CalculatePAYEDetailed(gross, pension, nhf, other)
    Tax->>Tax: CalculateCRA(annualGross)
    Note over Tax: CRA = max(200000, 0.01*gross) + 0.20*gross
    Tax->>Tax: taxable = gross - CRA - pension - NHF - other
    Note over Tax: If taxable < 0, set to 0

    Tax->>Bands: applyTaxBands(taxable)
    loop For each band
        Note over Bands: Band 1: 300K @ 7%
        Note over Bands: Band 2: 300K @ 11%
        Note over Bands: Band 3: 500K @ 15%
        Note over Bands: Band 4: 500K @ 19%
        Note over Bands: Band 5: 1,600K @ 21%
        Note over Bands: Band 6: Remainder @ 24%
        Bands->>Bands: taxableInBand = min(remaining, bandWidth)
        Bands->>Bands: taxInBand = taxableInBand * rate
        Bands->>Bands: remaining -= taxableInBand
    end
    Bands-->>Tax: (totalTax, []PAYEBandDetail)
    Tax->>Tax: monthlyPAYE = annualPAYE / 12
    Tax-->>Engine: PAYEResult
```

---

## 3. Class Diagrams

### 3.1 Employee Domain Model

```mermaid
classDiagram
    class Employee {
        +UUID ID
        +string TenantID
        +string EmployeeNumber
        +string FirstName
        +string MiddleName
        +string LastName
        +string Email
        +Gender Gender
        +MaritalStatus MaritalStatus
        +EmploymentType EmploymentType
        +EmploymentStatus EmploymentStatus
        +time.Time HireDate
        +UUID DepartmentID
        +UUID PositionID
        +UUID ManagerID
        +Address Address
        +BankDetails BankDetails
        +TaxInfo TaxInfo
        +PensionInfo PensionInfo
        +FullName() string
        +DisplayName() string
        +IsActive() bool
        +IsOnProbation() bool
    }

    class Department {
        +UUID ID
        +string TenantID
        +string Name
        +string Code
        +UUID ParentID
        +UUID HeadID
        +string CostCenter
        +bool IsActive
    }

    class Position {
        +UUID ID
        +string TenantID
        +string Title
        +string Code
        +UUID DepartmentID
        +string GradeLevel
        +float64 MinSalary
        +float64 MaxSalary
    }

    class Address {
        +string Street1
        +string City
        +string State
        +string PostalCode
        +string Country
    }

    class BankDetails {
        +string BankName
        +string BankCode
        +string AccountNumber
        +string AccountName
        +string SwiftCode
        +string IBAN
    }

    class TaxInfo {
        +string TaxID
        +string TaxOffice
        +string TaxStatus
        +bool TaxExempt
    }

    class PensionInfo {
        +string PensionID
        +string PensionProvider
        +string RSAPin
        +float64 EmployeeRate
        +float64 EmployerRate
    }

    Employee *-- Address
    Employee *-- BankDetails
    Employee *-- TaxInfo
    Employee *-- PensionInfo
    Employee --> Department
    Employee --> Position
    Employee --> Employee : ManagerID
    Department --> Department : ParentID
```

### 3.2 Payroll Domain Model

```mermaid
classDiagram
    class PayrollPeriod {
        +UUID ID
        +UUID TenantID
        +UUID CompanyID
        +string PeriodName
        +int PeriodYear
        +int PeriodMonth
        +time.Time StartDate
        +time.Time EndDate
        +time.Time PayDate
        +PayrollPeriodStatus Status
        +bool IsLocked
        +[]PayrollRun Runs
    }

    class PayrollRun {
        +UUID ID
        +UUID PeriodID
        +string RunNumber
        +PayrollRunType RunType
        +PayrollRunStatus Status
        +int TotalEmployees
        +int64 TotalGross
        +int64 TotalNet
        +int64 TotalDeductions
        +int64 TotalPAYE
        +int64 TotalPensionEmployee
        +int64 TotalPensionEmployer
        +int64 TotalNHF
        +string Currency
        +[]PayrollEntry Entries
        +[]PayrollRunApproval Approvals
    }

    class PayrollEntry {
        +UUID ID
        +UUID RunID
        +UUID EmployeeID
        +int64 GrossSalary
        +int64 BasicSalary
        +int64 TotalEarnings
        +int64 TotalDeductions
        +int64 NetPay
        +int64 PAYEAmount
        +int64 PensionEmployee
        +int64 PensionEmployer
        +int64 NHFAmount
        +int64 TaxableIncome
        +int64 CRAAmount
        +bool IsProrated
        +ValidationStatus ValidationStatus
        +[]PayrollEntryItem Items
    }

    class PayrollEntryItem {
        +UUID ID
        +UUID EntryID
        +string ComponentCode
        +string ComponentName
        +ComponentCategory Category
        +int64 Amount
        +bool IsTaxable
        +bool IsPensionable
    }

    PayrollPeriod "1" --> "*" PayrollRun
    PayrollRun "1" --> "*" PayrollEntry
    PayrollEntry "1" --> "*" PayrollEntryItem
```

---

## 4. Database Schema (Key Tables)

### 4.1 Employee Schema

```sql
-- employee.legal_entities
CREATE TABLE employee.legal_entities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    parent_entity_id UUID REFERENCES employee.legal_entities(id),
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    country_code VARCHAR(3) NOT NULL DEFAULT 'NGA',
    currency_code VARCHAR(3) NOT NULL DEFAULT 'NGN',
    is_active BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- employee.departments
CREATE TABLE employee.departments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    legal_entity_id UUID REFERENCES employee.legal_entities(id),
    parent_id UUID REFERENCES employee.departments(id),
    code VARCHAR(50) NOT NULL,
    name VARCHAR(255) NOT NULL,
    head_id UUID,
    cost_center VARCHAR(50),
    is_active BOOLEAN DEFAULT true,
    UNIQUE(tenant_id, code)
);

-- employee.employees (core table)
CREATE TABLE employee.employees (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    employee_number VARCHAR(50) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    department_id UUID REFERENCES employee.departments(id),
    position_id UUID,
    manager_id UUID REFERENCES employee.employees(id),
    employment_type VARCHAR(20) NOT NULL,
    employment_status VARCHAR(20) NOT NULL DEFAULT 'active',
    hire_date DATE NOT NULL,
    bank_details_encrypted BYTEA,
    tax_info_encrypted BYTEA,
    custom_fields JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMPTZ,
    UNIQUE(tenant_id, employee_number),
    UNIQUE(tenant_id, email)
);
```

### 4.2 Payroll Schema

```sql
-- pay_grades
CREATE TABLE pay_grades (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    company_id UUID NOT NULL,
    code VARCHAR(20) NOT NULL,
    name VARCHAR(100) NOT NULL,
    min_salary DECIMAL(18,2) NOT NULL,
    max_salary DECIMAL(18,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'NGN',
    UNIQUE(company_id, code)
);

-- payroll_periods
CREATE TABLE payroll_periods (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    company_id UUID NOT NULL,
    period_year INT NOT NULL,
    period_month INT NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    pay_date DATE NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'open',
    is_locked BOOLEAN DEFAULT false
);

-- payroll_runs
CREATE TABLE payroll_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    period_id UUID NOT NULL REFERENCES payroll_periods(id),
    run_type VARCHAR(20) NOT NULL DEFAULT 'regular',
    status VARCHAR(30) NOT NULL DEFAULT 'draft',
    total_employees INT DEFAULT 0,
    total_gross BIGINT DEFAULT 0,
    total_net BIGINT DEFAULT 0,
    total_paye BIGINT DEFAULT 0,
    total_pension_employee BIGINT DEFAULT 0,
    total_pension_employer BIGINT DEFAULT 0,
    total_nhf BIGINT DEFAULT 0,
    currency VARCHAR(3) DEFAULT 'NGN',
    created_by UUID NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- payroll_entries (one per employee per run)
CREATE TABLE payroll_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    run_id UUID NOT NULL REFERENCES payroll_runs(id),
    employee_id UUID NOT NULL,
    gross_salary BIGINT NOT NULL,
    basic_salary BIGINT NOT NULL,
    total_earnings BIGINT NOT NULL,
    total_deductions BIGINT NOT NULL,
    net_pay BIGINT NOT NULL,
    paye_amount BIGINT DEFAULT 0,
    pension_employee BIGINT DEFAULT 0,
    pension_employer BIGINT DEFAULT 0,
    nhf_amount BIGINT DEFAULT 0,
    taxable_income BIGINT DEFAULT 0,
    cra_amount BIGINT DEFAULT 0,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 5. API Endpoint Specifications

### 5.1 Employee Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/v1/employee` | List employees (paginated) | JWT + Tenant |
| POST | `/v1/employee` | Create employee | JWT + Tenant + HR role |
| GET | `/v1/employee/{id}` | Get employee by ID | JWT + Tenant |
| PUT | `/v1/employee/{id}` | Update employee | JWT + Tenant + HR role |
| DELETE | `/v1/employee/{id}` | Soft-delete employee | JWT + Tenant + Admin |
| POST | `/v1/employee/import` | Bulk import via CSV | JWT + Tenant + HR role |
| GET | `/v1/employee/{id}/history` | Get change history | JWT + Tenant |
| GET | `/v1/employee/statistics` | Get workforce stats | JWT + Tenant |

### 5.2 Payroll Endpoints

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| POST | `/v1/payroll/periods` | Create period | JWT + Payroll role |
| GET | `/v1/payroll/periods` | List periods | JWT + Payroll role |
| POST | `/v1/payroll/runs` | Initiate run | JWT + Payroll role |
| POST | `/v1/payroll/runs/{id}/process` | Process run | JWT + Payroll role |
| POST | `/v1/payroll/runs/{id}/approve` | Approve run | JWT + Approver role |
| GET | `/v1/payroll/runs/{id}/summary` | Get run summary | JWT + Payroll role |
| GET | `/v1/payroll/entries/{id}/payslip` | Get payslip PDF | JWT + Tenant |
| POST | `/v1/payroll/runs/{id}/bank-file` | Generate bank file | JWT + Payroll role |

---

## 6. Error Handling

```mermaid
flowchart TB
    ERR[Error Occurs] --> TYPE{Error Type}
    TYPE -->|Validation| V400[400 Bad Request<br/>{code, message, fields}]
    TYPE -->|Auth| V401[401 Unauthorized<br/>{code, message}]
    TYPE -->|Forbidden| V403[403 Forbidden<br/>{code, message}]
    TYPE -->|Not Found| V404[404 Not Found<br/>{code, message}]
    TYPE -->|Rate Limit| V429[429 Too Many Requests<br/>{retry_after}]
    TYPE -->|Internal| V500[500 Internal Server Error<br/>{code, request_id}]

    V500 --> LOG[Log with zerolog<br/>Stack trace, request_id]
    V500 --> ALERT[Alert via Prometheus<br/>5xx counter increment]
```
