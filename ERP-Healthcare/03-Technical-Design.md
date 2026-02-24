# Technical Design Document - AfriHealth ERP-Healthcare

## 1. Overview

This document details the technical design decisions, patterns, and implementation specifics for the AfriHealth healthcare platform. It covers service design, data modeling patterns, API design conventions, event-driven architecture, and cross-cutting concerns.

---

## 2. Service Design Patterns

### 2.1 Standard Go Microservice Structure

Every Go microservice follows a consistent hexagonal architecture:

```
service-name/
  cmd/
    main.go              # Entry point, dependency injection
  config/
    config.go            # Configuration loading (env vars)
  models/
    <domain>.go          # GORM models + request/response DTOs
  repository/
    <domain>_repository.go  # Database access layer
  service/
    <domain>_service.go     # Business logic
    <domain>_service_test.go
  handlers/
    <domain>_handler.go     # HTTP handlers (Gin)
  events/
    redpanda_consumer.go    # Event consumers
    redpanda_producer.go    # Event producers
  middleware/
    auth.go              # JWT validation
    tenant.go            # Tenant extraction
  tests/
    integration_test.go
  Dockerfile
  main.go               # Simplified entry point
```

### 2.2 Dependency Injection Pattern

```go
func main() {
    // Load configuration
    cfg := config.Load()

    // Initialize database
    db := database.Connect(cfg.DatabaseURL)

    // Initialize Redis
    redis := cache.NewRedisClient(cfg.RedisAddr)

    // Initialize Redpanda producer
    producer := events.NewProducer(cfg.KafkaBrokers)

    // Wire dependencies
    repo := repository.NewPatientRepository(db)
    svc := service.NewPatientService(repo, redis, producer)
    handler := handlers.NewPatientHandler(svc)

    // Setup router
    router := gin.Default()
    handler.RegisterRoutes(router)

    router.Run(fmt.Sprintf(":%s", cfg.Port))
}
```

### 2.3 Multi-Tenant Middleware

```go
func TenantMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        tenantID := c.GetHeader("X-Tenant-ID")
        if tenantID == "" {
            c.AbortWithStatusJSON(400, gin.H{"error": "X-Tenant-ID header required"})
            return
        }

        tenantUUID, err := uuid.Parse(tenantID)
        if err != nil {
            c.AbortWithStatusJSON(400, gin.H{"error": "Invalid tenant ID"})
            return
        }

        c.Set("tenant_id", tenantUUID)
        c.Next()
    }
}
```

---

## 3. Data Access Patterns

### 3.1 Repository Pattern with GORM

All database access is encapsulated in repository structs with tenant-scoped queries:

```mermaid
classDiagram
    class PatientRepository {
        -db *gorm.DB
        +Create(patient *Patient) error
        +FindByID(tenantID uuid, id uuid) (*Patient, error)
        +Search(tenantID uuid, criteria SearchRequest) ([]Patient, error)
        +Update(patient *Patient) error
        +SoftDelete(tenantID uuid, id uuid) error
    }

    class PatientService {
        -repo PatientRepository
        -cache RedisClient
        -producer EventProducer
        +RegisterPatient(req CreatePatientRequest) (*Patient, error)
        +GetPatient(id uuid) (*Patient, error)
        +SearchPatients(criteria SearchRequest) ([]Patient, error)
        +UpdatePatient(id uuid, req UpdatePatientRequest) error
    }

    class PatientHandler {
        -service PatientService
        +CreatePatient(c *gin.Context)
        +GetPatient(c *gin.Context)
        +SearchPatients(c *gin.Context)
        +UpdatePatient(c *gin.Context)
    }

    PatientHandler --> PatientService
    PatientService --> PatientRepository
```

### 3.2 Caching Strategy

```mermaid
graph TD
    A[API Request] --> B{Cache Hit?}
    B -->|Yes| C[Return Cached Data]
    B -->|No| D[Query PostgreSQL]
    D --> E[Store in Redis]
    E --> F[Return Data]

    G[Write Operation] --> H[Update PostgreSQL]
    H --> I[Invalidate Redis Cache]
    I --> J[Publish Event to Redpanda]
```

Cache TTL guidelines:
- Patient demographics: 5 minutes
- Lab results: 1 minute (critical data)
- Drug catalog: 1 hour
- Insurance plans: 30 minutes
- Hospital facility data: 15 minutes

---

## 4. Event-Driven Architecture

### 4.1 Event Schema Standard

All events follow a standardized envelope:

```json
{
    "event_id": "uuid-v4",
    "event_type": "patient.created",
    "event_version": "1.0",
    "tenant_id": "uuid-v4",
    "source": "patient-service",
    "timestamp": "2026-02-23T10:30:00Z",
    "correlation_id": "uuid-v4",
    "data": {
        "patient_id": "uuid-v4",
        "first_name": "Amara",
        "last_name": "Okafor"
    },
    "metadata": {
        "user_id": "uuid-v4",
        "ip_address": "10.0.1.50"
    }
}
```

### 4.2 Topic Naming Convention

```
{domain}.{entity}.{action}

Examples:
- clinical.patient.created
- clinical.patient.updated
- clinical.lab.order.created
- clinical.lab.result.critical
- financial.payment.completed
- financial.claim.submitted
- ai.tb.detection.completed
- ai.crisis.detected
- platform.notification.sent
```

### 4.3 Redpanda Consumer Pattern

```go
// Event consumer with dead-letter queue
func (c *Consumer) ProcessEvent(msg *kafka.Message) error {
    var event EventEnvelope
    if err := json.Unmarshal(msg.Value, &event); err != nil {
        return c.sendToDeadLetter(msg, err)
    }

    switch event.EventType {
    case "clinical.lab.result.critical":
        return c.handleCriticalLabResult(event)
    case "clinical.patient.created":
        return c.handlePatientCreated(event)
    default:
        log.Warnf("Unknown event type: %s", event.EventType)
        return nil
    }
}
```

---

## 5. AI/ML Service Integration

### 5.1 AI Service Communication Flow

```mermaid
sequenceDiagram
    participant Provider as Provider Portal
    participant GW as API Gateway
    participant AID as AI Diagnosis Service (Go)
    participant CAI as Clinical AI (Python/FastAPI)
    participant IMG as Imaging AI (Python/TF)

    Provider->>GW: POST /api/v1/ai/analyze-xray
    GW->>AID: Forward with auth context
    AID->>IMG: gRPC: AnalyzeImage(dicom_bytes)
    IMG->>IMG: Preprocess (CLAHE + resize)
    IMG->>IMG: EfficientNetB4 inference
    IMG->>IMG: Generate Grad-CAM
    IMG-->>AID: TBDetectionResult
    AID->>CAI: POST /ai/clinical/differential-diagnosis
    CAI-->>AID: Differential diagnosis + recommendations
    AID-->>GW: Combined AI analysis
    GW-->>Provider: AI-assisted diagnosis
```

### 5.2 Model Serving Architecture

```mermaid
graph TB
    subgraph "Model Registry"
        MR[MLflow Model Registry]
    end

    subgraph "Serving Layer"
        TFS[TensorFlow Serving<br/>TB Detection Model]
        TS[Triton Inference Server<br/>Multi-model]
        FA[FastAPI Services<br/>Clinical AI + Mental Health]
    end

    subgraph "Monitoring"
        DD[Data Drift Detection]
        PM[Performance Monitoring]
        AB[A/B Testing Framework]
    end

    MR --> TFS & TS & FA
    TFS --> DD
    TS --> DD
    FA --> PM
    DD --> AB
```

---

## 6. Blockchain Integration Design

### 6.1 Hyperledger Fabric Network Topology

```mermaid
graph TB
    subgraph "Hyperledger Fabric Network"
        subgraph "Organization 1: AfriHealth"
            P1[Peer 0]
            P2[Peer 1]
            CA1[Certificate Authority]
        end

        subgraph "Organization 2: Hospital Network"
            P3[Peer 0]
            P4[Peer 1]
            CA2[Certificate Authority]
        end

        subgraph "Ordering Service"
            O1[Orderer 0 - Raft]
            O2[Orderer 1 - Raft]
            O3[Orderer 2 - Raft]
        end

        subgraph "Chaincodes"
            CC1[Consent Chaincode<br/>Grant/Revoke/Verify]
            CC2[Drug Supply Chain<br/>Register/Transfer/Verify]
            CC3[Medical Credentials<br/>Issue/Verify/Revoke]
        end
    end

    P1 & P2 --> CC1 & CC2 & CC3
    P3 & P4 --> CC1 & CC2 & CC3
    P1 & P2 & P3 & P4 --> O1 & O2 & O3
```

### 6.2 Consent Management Flow

```mermaid
sequenceDiagram
    participant Patient
    participant App as Mobile App
    participant BS as Blockchain Service
    participant HF as Hyperledger Fabric
    participant Provider as Provider Service

    Patient->>App: Grant consent for data sharing
    App->>BS: POST /api/v1/blockchain/consent/grant
    BS->>HF: GrantConsent(patientID, providerID, purpose, dataTypes, expiry)
    HF->>HF: Create consent record on ledger
    HF-->>BS: Transaction ID + Hash
    BS-->>App: Consent granted (with blockchain proof)

    Provider->>BS: VerifyConsent(patientID, providerID, purpose)
    BS->>HF: VerifyConsent query
    HF-->>BS: Valid/Invalid + consent details
    BS-->>Provider: Access decision
```

---

## 7. IoT Medical Device Integration

```mermaid
graph LR
    subgraph "Medical Devices"
        BP[Blood Pressure Monitor]
        GLU[Glucose Meter]
        OXI[Pulse Oximeter]
        ECG[ECG Monitor]
        VENT[Ventilator]
    end

    subgraph "IoT Gateway"
        MQTT[MQTT Broker]
        PROT[Protocol Adapter<br/>HL7/FHIR/Custom]
    end

    subgraph "IoT Service"
        ING[Ingestion Pipeline]
        VAL[Validation & Normalization]
        ALT[Alert Engine<br/>Threshold Rules]
        TSS[Time-Series Storage<br/>TimescaleDB]
    end

    subgraph "Downstream"
        NOT[Notification Service]
        EHR[EHR Service]
        AI[AI Diagnosis Service]
    end

    BP & GLU & OXI & ECG & VENT --> MQTT
    MQTT --> PROT --> ING --> VAL --> ALT
    VAL --> TSS
    ALT --> NOT
    TSS --> EHR & AI
```

---

## 8. Performance Optimization Strategies

### 8.1 Database Optimization
- **Materialized views**: `patient_summary` and `mv_patient_mental_health_summary` for dashboard queries
- **Full-text search**: PostgreSQL tsvector with weighted columns (name = A, MRN = B, phone = C)
- **Partial indexes**: `WHERE is_active = true`, `WHERE status = 'active'`
- **Connection pooling**: PgBouncer with 100 connections per service
- **Read replicas**: For analytics and reporting queries

### 8.2 Application-Level Optimization
- **Redis caching**: Multi-level cache with TTL-based invalidation
- **Batch processing**: Lab results processing in batches of 100
- **Pagination**: Cursor-based pagination for large result sets
- **Compression**: gzip response compression for API responses > 1KB
- **Connection reuse**: HTTP/2 connection multiplexing between services

### 8.3 AI/ML Optimization
- **Model quantization**: INT8 quantization for TB detection model (2x inference speedup)
- **Batch inference**: Process multiple X-rays per GPU batch
- **Model caching**: Keep models warm in GPU memory
- **Async processing**: Non-blocking AI analysis with callback notifications

---

## 9. Error Handling and Resilience

### 9.1 Circuit Breaker Pattern

```mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open: Failure threshold exceeded
    Open --> HalfOpen: Timeout elapsed
    HalfOpen --> Closed: Success
    HalfOpen --> Open: Failure
```

### 9.2 Retry Strategy
- **API calls**: Exponential backoff (100ms, 200ms, 400ms, max 3 retries)
- **Event processing**: Dead-letter queue after 5 failed attempts
- **Database operations**: Immediate retry once, then fail
- **External payments**: Idempotency keys to prevent duplicate charges

---

## 10. Monitoring and Observability

```mermaid
graph TB
    subgraph "Data Collection"
        M[Metrics<br/>Prometheus]
        L[Logs<br/>Fluentd]
        T[Traces<br/>Jaeger/OpenTelemetry]
    end

    subgraph "Storage"
        PM[Prometheus TSDB]
        ES[Elasticsearch]
        JA[Jaeger Backend]
    end

    subgraph "Visualization"
        GR[Grafana Dashboards]
        KI[Kibana]
        JU[Jaeger UI]
    end

    subgraph "Alerting"
        AM[AlertManager]
        PD[PagerDuty]
        SL[Slack]
    end

    M --> PM --> GR
    L --> ES --> KI
    T --> JA --> JU
    PM --> AM --> PD & SL
```

Key metrics monitored:
- Request latency (p50, p95, p99) per service
- Error rate per endpoint
- Database query duration
- Redpanda consumer lag
- AI model inference time
- Cache hit/miss ratio
- Active WebSocket connections (telemedicine)
