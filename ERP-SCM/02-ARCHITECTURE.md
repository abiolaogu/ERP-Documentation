# ERP-SCM Architecture Document

## 1. Overview

ERP-SCM follows a modular microservices architecture where each supply chain domain (procurement, inventory, warehouse, manufacturing, demand planning, logistics, quality, fleet, supplier portal) is encapsulated as an independent service. All services share a common API gateway, event backbone, and AI inference layer. The system operates in `standalone_plus_suite` mode -- it can run independently or as part of the broader ERP platform.

---

## 2. C4 Context Diagram

```mermaid
C4Context
    title ERP-SCM System Context

    Person(scm_user, "SCM User", "Procurement, warehouse, planning, logistics staff")
    Person(supplier, "Supplier", "External vendor accessing portal")
    Person(exec, "Executive", "VP Supply Chain, COO")

    System(erp_scm, "ERP-SCM", "AI-powered Supply Chain Management Suite")

    System_Ext(erp_iam, "ERP-IAM", "Identity & Access Management")
    System_Ext(erp_platform, "ERP-Platform", "Tenant & Subscription Mgmt")
    System_Ext(erp_finance, "ERP-Finance", "Accounting, AP/AR")
    System_Ext(erp_bi, "ERP-BI", "Business Intelligence")
    System_Ext(erp_commerce, "ERP-Commerce", "Sales & eCommerce")
    System_Ext(carrier_api, "Carrier APIs", "UPS, FedEx, DHL, etc.")
    System_Ext(gps_provider, "GPS Provider", "Fleet telematics")

    Rel(scm_user, erp_scm, "Uses", "HTTPS/WSS")
    Rel(supplier, erp_scm, "Submits POs, ASNs, Invoices", "HTTPS")
    Rel(exec, erp_scm, "Reviews dashboards", "HTTPS")
    Rel(erp_scm, erp_iam, "Authenticates", "OIDC/JWT")
    Rel(erp_scm, erp_platform, "Entitlements", "gRPC")
    Rel(erp_scm, erp_finance, "AP/AR sync", "Events")
    Rel(erp_scm, erp_bi, "Analytics data", "Events")
    Rel(erp_scm, erp_commerce, "Sales orders", "Events")
    Rel(erp_scm, carrier_api, "Rate/Track", "REST")
    Rel(erp_scm, gps_provider, "Telemetry", "MQTT/REST")
```

---

## 3. Container Diagram

```mermaid
flowchart TB
    subgraph "Client Tier"
        WEB["React 18 SPA<br/>TypeScript + Vite + Tailwind"]
        PORTAL["Supplier Portal<br/>React SPA"]
    end

    subgraph "Gateway Tier"
        GW["API Gateway<br/>FastAPI + JWT Middleware"]
        WS["WebSocket Hub<br/>Real-time updates"]
    end

    subgraph "Service Tier"
        S1["procurement-service<br/>FastAPI"]
        S2["inventory-service<br/>FastAPI"]
        S3["warehouse-service<br/>FastAPI"]
        S4["manufacturing-service<br/>FastAPI"]
        S5["demand-planning-service<br/>FastAPI"]
        S6["logistics-service<br/>FastAPI"]
        S7["quality-service<br/>FastAPI"]
        S8["fleet-service<br/>FastAPI"]
        S9["supplier-portal-service<br/>FastAPI"]
    end

    subgraph "AI Tier"
        AI1["DemandForecaster<br/>ES + RF Ensemble"]
        AI2["SupplierRiskAnalyzer<br/>Isolation Forest"]
        AI3["AnomalyDetector<br/>Z-score + IF"]
        AI4["RouteOptimizer<br/>NN + 2-opt + OR-Tools"]
        AI5["InsightsEngine<br/>BI Generator"]
    end

    subgraph "Data Tier"
        PG["PostgreSQL 16<br/>Primary DB"]
        RD["Redis 7<br/>Cache + Sessions"]
        S3S["MinIO / S3<br/>Document Storage"]
    end

    subgraph "Event Tier"
        RP["Redpanda / Kafka<br/>Event Backbone"]
    end

    WEB --> GW
    PORTAL --> GW
    GW --> WS
    GW --> S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9
    S5 --> AI1
    S1 --> AI2
    S2 --> AI3
    S6 --> AI4
    GW --> AI5
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 --> PG
    S1 & S2 & S3 & S4 & S5 & S6 & S7 & S8 & S9 --> RP
    GW --> RD
    S3 & S7 --> S3S
```

---

## 4. Service Inventory

| Service | Responsibility | Port | Database Schema |
|---|---|---|---|
| `procurement-service` | Requisitions, RFQ, POs, vendor scorecards, contracts, 3-way matching | 8001 | `scm_procurement` |
| `inventory-service` | Stock levels, reorder logic, ABC/XYZ, valuation, aging | 8002 | `scm_inventory` |
| `warehouse-service` | Layout, receiving, putaway, picking, packing, shipping, returns | 8003 | `scm_warehouse` |
| `manufacturing-service` | BOM, production orders, work centers, routing, MRP, scheduling | 8004 | `scm_manufacturing` |
| `demand-planning-service` | Forecasting, consensus planning, S&OP, accuracy metrics | 8005 | `scm_demand` |
| `logistics-service` | Carrier management, shipments, route optimization, freight audit | 8006 | `scm_logistics` |
| `quality-service` | Inspections, NCR, CAPA, SPC, ISO compliance | 8007 | `scm_quality` |
| `fleet-service` | Vehicles, drivers, maintenance, fuel, GPS, compliance | 8008 | `scm_fleet` |
| `supplier-portal-service` | External-facing PO ack, ASN, invoices, onboarding | 8009 | `scm_portal` |

---

## 5. Component Diagram (AI Layer)

```mermaid
flowchart LR
    subgraph "AI Inference Layer"
        direction TB
        DF["DemandForecaster"]
        SR["SupplierRiskAnalyzer"]
        AD["AnomalyDetector"]
        RO["RouteOptimizer"]
        IE["InsightsEngine"]
    end

    subgraph "ML Models"
        M1["Exponential Smoothing<br/>(Holt-Winters)"]
        M2["Random Forest<br/>(scikit-learn)"]
        M3["Isolation Forest<br/>(scikit-learn)"]
        M4["Z-Score Analysis<br/>(scipy)"]
        M5["NN + 2-opt<br/>(OR-Tools)"]
    end

    subgraph "Feature Store"
        FS1["Demand History"]
        FS2["Supplier Performance"]
        FS3["Inventory Levels"]
        FS4["Shipment History"]
    end

    DF --> M1 & M2
    SR --> M3
    AD --> M3 & M4
    RO --> M5
    IE --> FS1 & FS2 & FS3 & FS4
    M1 & M2 --> FS1
    M3 --> FS2 & FS3
    M5 --> FS4
```

---

## 6. Data Architecture

### 6.1 Database Strategy

- **Primary**: PostgreSQL 16 with schema-per-service isolation
- **Caching**: Redis 7 for session management, API response caching, and real-time KPI materialization
- **Object Storage**: MinIO (S3-compatible) for documents, labels, COAs, and BOM attachments
- **Search**: Optional Elasticsearch for full-text search across POs, shipments, NCRs

### 6.2 Multi-Tenancy

Each tenant's data is isolated via a `tenant_id` column on every table, enforced at the ORM layer:

```python
class TenantMixin:
    tenant_id = Column(UUID, nullable=False, index=True)

    @declared_attr
    def __table_args__(cls):
        return (
            Index(f'ix_{cls.__tablename__}_tenant', 'tenant_id'),
        )
```

The `X-Tenant-ID` header is required on all business endpoints and validated against the JWT claims.

---

## 7. Event Architecture

### 7.1 Event Convention

All events follow the CloudEvents specification with the naming pattern:

```
erp.scm.<domain>.<entity>.<action>
```

Examples:
- `erp.scm.procurement.po.created`
- `erp.scm.inventory.stock.adjusted`
- `erp.scm.manufacturing.production-order.completed`
- `erp.scm.logistics.shipment.delivered`

### 7.2 Event Flow

```mermaid
sequenceDiagram
    participant Proc as Procurement
    participant Inv as Inventory
    participant WMS as Warehouse
    participant MFG as Manufacturing
    participant Log as Logistics
    participant Bus as Event Bus

    Proc->>Bus: po.approved
    Bus->>WMS: triggers receiving preparation
    Bus->>Inv: reserves expected stock
    WMS->>Bus: goods.received
    Bus->>Inv: stock.adjusted (+)
    Bus->>Proc: receipt.confirmed (3-way match)
    MFG->>Bus: production-order.completed
    Bus->>Inv: stock.adjusted (+ finished goods)
    Bus->>WMS: putaway.triggered
    Bus->>Log: shipment.ready
    Log->>Bus: shipment.dispatched
    Bus->>Inv: stock.adjusted (-)
```

---

## 8. Security Architecture

### 8.1 Authentication

- OIDC/JWT tokens issued by ERP-IAM
- Token refresh via sliding window
- Service-to-service: mTLS with short-lived certificates

### 8.2 Authorization

RBAC with the following role hierarchy:

```mermaid
graph TD
    ADMIN["scm:admin"]
    PROC_MGR["scm:procurement:manager"]
    PROC_USER["scm:procurement:user"]
    WMS_MGR["scm:warehouse:manager"]
    WMS_OP["scm:warehouse:operator"]
    MFG_MGR["scm:manufacturing:manager"]
    MFG_OP["scm:manufacturing:operator"]
    QMS_MGR["scm:quality:manager"]
    FLT_MGR["scm:fleet:manager"]
    SUPPLIER["scm:supplier:external"]

    ADMIN --> PROC_MGR & WMS_MGR & MFG_MGR & QMS_MGR & FLT_MGR
    PROC_MGR --> PROC_USER
    WMS_MGR --> WMS_OP
    MFG_MGR --> MFG_OP
```

### 8.3 Data Protection

- Field-level encryption for PII (supplier contacts, driver PII)
- TLS 1.3 in transit
- AES-256 at rest (database, object storage)
- Audit log for all write operations (immutable append-only)

---

## 9. Deployment Architecture

```mermaid
flowchart TB
    subgraph "Kubernetes Cluster"
        subgraph "Ingress"
            IG["NGINX Ingress<br/>TLS Termination"]
        end
        subgraph "Application Pods"
            GW["Gateway Pod (x3)"]
            P1["Procurement Pod (x2)"]
            P2["Inventory Pod (x2)"]
            P3["Warehouse Pod (x3)"]
            P4["Manufacturing Pod (x2)"]
            P5["Demand Pod (x2)"]
            P6["Logistics Pod (x2)"]
            P7["Quality Pod (x2)"]
            P8["Fleet Pod (x2)"]
            P9["Portal Pod (x2)"]
        end
        subgraph "Data Pods"
            PG["PostgreSQL (HA)"]
            RD["Redis Sentinel"]
            RP["Redpanda Cluster"]
        end
    end

    IG --> GW
    GW --> P1 & P2 & P3 & P4 & P5 & P6 & P7 & P8 & P9
    P1 & P2 & P3 & P4 & P5 & P6 & P7 & P8 & P9 --> PG & RD & RP
```

---

## 10. Technology Stack Summary

| Layer | Technology | Version |
|---|---|---|
| Frontend | React, TypeScript, Vite, Tailwind CSS, Recharts | 18.x, 5.x |
| Backend | Python, FastAPI, SQLAlchemy, Pydantic | 3.11, 0.109, 2.0, 2.5 |
| AI/ML | scikit-learn, statsmodels, NumPy, pandas, SciPy, OR-Tools | 1.4, 0.14, 1.26, 2.2, 1.12, 9.8 |
| Database | PostgreSQL, Redis | 16, 7 |
| Messaging | Redpanda (Kafka-compatible) | Latest |
| Infrastructure | Docker, Kubernetes, Nginx | Latest |
| CI/CD | GitHub Actions | N/A |
| Observability | OpenTelemetry, Prometheus, Grafana | Latest |

---

## 11. Cross-Cutting Concerns

### 11.1 Observability

- **Distributed tracing**: OpenTelemetry spans across all services
- **Metrics**: Prometheus exporters per service (request latency, error rates, queue depth)
- **Logging**: Structured JSON logs, correlation IDs propagated via headers
- **Dashboards**: Grafana dashboards for SRE and business KPIs

### 11.2 Resilience

- Circuit breakers on all inter-service calls (exponential backoff)
- Dead letter queues for failed event processing
- Graceful degradation: AI features degrade to rule-based fallbacks if ML models are unavailable
- Health checks: `/healthz` (liveness) and `/readyz` (readiness) on every service

### 11.3 Configuration

- Environment variables for runtime configuration
- `pydantic-settings` for typed configuration with `.env` file support
- Feature flags for gradual rollout of new capabilities
