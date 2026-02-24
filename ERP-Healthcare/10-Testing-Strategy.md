# Testing Strategy Document - AfriHealth ERP-Healthcare

## 1. Overview

AfriHealth employs a comprehensive multi-level testing strategy covering unit tests, integration tests, end-to-end tests, performance tests, security tests, and AI model validation tests.

---

## 2. Testing Pyramid

```mermaid
graph TB
    subgraph "Testing Pyramid"
        E2E[End-to-End Tests<br/>~50 tests<br/>Cypress + Playwright]
        INT[Integration Tests<br/>~200 tests<br/>Go test + testcontainers]
        UNIT[Unit Tests<br/>~1000+ tests<br/>Go test + pytest]
    end

    subgraph "Specialized Tests"
        PERF[Performance Tests<br/>k6 + Locust]
        SEC[Security Tests<br/>OWASP ZAP + Trivy]
        AI[AI Model Validation<br/>pytest + custom metrics]
        COMP[Compliance Tests<br/>HIPAA/GDPR checklist automation]
    end

    E2E --> INT --> UNIT
```

---

## 3. Unit Testing

### 3.1 Go Services (go test)

```go
// Example: appointment_service_test.go
func TestCreateAppointment_Success(t *testing.T) {
    mockRepo := new(MockAppointmentRepository)
    mockRepo.On("Create", mock.Anything).Return(nil)

    svc := NewAppointmentService(mockRepo, nil, nil)
    req := CreateAppointmentRequest{
        PatientID:       "patient-uuid",
        DoctorID:        "doctor-uuid",
        AppointmentDate: time.Now().Add(24 * time.Hour),
        Duration:        30,
        Type:            "consultation",
        Reason:          "Follow-up",
    }

    apt, err := svc.CreateAppointment(tenantID, req)
    assert.NoError(t, err)
    assert.Equal(t, "scheduled", apt.Status)
    mockRepo.AssertExpectations(t)
}
```

### 3.2 Python AI Services (pytest)

```python
# Example: test_tb_detection.py
def test_tb_detection_normal_xray():
    detector = ChestXRayTBDetector()
    result = detector.detect_tb("test_data/normal_xray.png")
    assert result.prediction == TBPrediction.NORMAL
    assert result.confidence > 0.8
    assert result.tb_probability < 0.2

def test_tb_detection_active_tb():
    detector = ChestXRayTBDetector()
    result = detector.detect_tb("test_data/active_tb.png")
    assert result.prediction == TBPrediction.TB_ACTIVE
    assert result.confidence > 0.85
    assert result.severity is not None
```

---

## 4. Integration Testing

```mermaid
sequenceDiagram
    participant Test as Test Suite
    participant TC as Testcontainers
    participant PG as PostgreSQL Container
    participant RD as Redis Container
    participant SVC as Service Under Test

    Test->>TC: Start PostgreSQL container
    TC-->>Test: PostgreSQL ready on random port
    Test->>TC: Start Redis container
    TC-->>Test: Redis ready on random port
    Test->>PG: Run migrations
    Test->>SVC: Initialize service with test config
    Test->>SVC: Execute test scenarios
    SVC->>PG: Database operations
    SVC->>RD: Cache operations
    SVC-->>Test: Assert responses
    Test->>TC: Cleanup containers
```

---

## 5. AI Model Validation

```mermaid
flowchart TB
    subgraph "Model Validation Pipeline"
        A[Test Dataset<br/>1000 labeled X-rays]
        B[Run Inference]
        C[Calculate Metrics]
        D{Meets Thresholds?}
        E[Deploy to Production]
        F[Block + Alert Team]
    end

    A --> B --> C --> D
    D -->|Yes| E
    D -->|No| F

    subgraph "Metrics"
        M1[Sensitivity >= 96.8%]
        M2[Specificity >= 94.2%]
        M3[AUC >= 0.987]
        M4[F1 Score >= 0.95]
    end

    C --> M1 & M2 & M3 & M4
```

---

## 6. Coverage Targets

| Component | Coverage Target | Current |
|-----------|----------------|---------|
| Go Services (business logic) | 80% | Tracked |
| Go Handlers | 70% | Tracked |
| Python AI Services | 85% | Tracked |
| React Components | 75% | Tracked |
| Database Migrations | 100% (all run successfully) | Tracked |
| API Endpoints | 100% (all have integration tests) | Tracked |
