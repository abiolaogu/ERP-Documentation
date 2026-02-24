# ERP-SCM Changelog

All notable changes to the ERP-SCM module are documented in this file. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this project adheres to [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

### Planned
- S&OP (Sales & Operations Planning) workflow
- Prophet and ARIMA forecasting model integration
- Advanced slotting optimization for warehouse
- Digital twin simulation for supply chain scenarios
- Generative AI recommendations engine

---

## [1.0.0] - 2026-02-23

### Added

#### Core Platform
- FastAPI backend with Python 3.11, SQLAlchemy 2.0, Pydantic 2.5
- React 18 + TypeScript + Vite frontend with Tailwind CSS
- Docker Compose deployment configuration
- JWT-based authentication with role-based access control
- Health check endpoints (`/api/health`, `/healthz`)

#### Procurement
- Purchase order lifecycle management (create, approve, receive, close, cancel)
- Supplier CRUD with multi-factor scoring profiles
- Supplier-product mapping with preferred supplier designation
- Supplier performance tracking (on-time, complete, defect rate, cost variance)

#### Inventory Management
- Multi-warehouse inventory tracking
- Product catalog with category management
- Reorder point and reorder quantity management (manual + AI-optimized)
- Stock level monitoring with low-stock and overstock detection
- Inventory valuation (weighted average)

#### AI/ML Capabilities
- **Demand Forecasting**: Exponential Smoothing (Holt-Winters) + Random Forest ensemble
  - Feature engineering: day-of-week, month, lags (1/7/14/30), rolling averages, trend
  - Weighted model averaging (40% ES + 60% RF)
  - 95% confidence intervals with upper/lower bounds
  - Optimal reorder point and EOQ calculation
- **Supplier Risk Analysis**: Multi-factor weighted risk scoring
  - Factor weights: delivery (30%), quality (25%), price (20%), financial (15%), communication (10%)
  - Risk classification: low, moderate, elevated, high, critical
  - Trend analysis (improving/stable/declining)
  - Isolation Forest anomaly detection for supplier behavior
- **Anomaly Detection**: Multi-strategy detection engine
  - Inventory anomalies (low stock, overstock)
  - Demand anomalies (Z-score based, threshold 2.5 sigma)
  - Delivery anomalies (late order detection)
  - Automated alert generation with severity classification
- **Route Optimization**: NN + 2-opt heuristic
  - Haversine distance calculation
  - Multi-stop route optimization
  - Delivery time estimation with weight factors
  - Shipment consolidation suggestions
- **Business Intelligence**: InsightsEngine
  - Supply chain health scoring (A-F grading)
  - Dead stock detection
  - Supplier concentration risk analysis
  - Demand trend identification
  - Fulfillment rate tracking
  - Dashboard KPI materialization

#### Logistics
- Shipment creation and lifecycle management
- Multi-carrier support
- Tracking event recording
- AI-optimized route suggestions
- AI-estimated delivery predictions

#### Dashboard & UI
- SCM Command Center with 8 KPI cards
- Supply Chain Health Score visualization (A-F grade)
- Order status pie chart
- AI Alerts panel with severity indicators
- AI Insights & Recommendations grid
- Products management page
- Inventory management with AI reorder optimization
- Supplier management with risk radar charts
- Order management with lifecycle tracking
- Logistics tracking page
- AI Center with anomaly detection and forecast triggers
- Alerts monitoring page

#### Microservices Architecture
- 12 service definitions for future decomposition:
  - demand-planning-service, fleet-management, fleet-service
  - inventory-service, logistics-service, manufacturing-service
  - procurement, procurement-service, quality-management
  - quality-service, supplier-portal-service, warehouse-service
- Event-driven architecture with CloudEvents convention
- Event topics for all nine domains

#### DevOps
- Docker containerization (backend + frontend)
- Nginx reverse proxy for frontend
- Demo data seeder (`seed_data.py`)
- pytest test framework configuration

### Dependencies
- FastAPI 0.109.0, SQLAlchemy 2.0.25, Pydantic 2.5.3
- scikit-learn 1.4.0, statsmodels 0.14.1, NumPy 1.26.3, pandas 2.2.0
- OR-Tools 9.8.3296
- React 18, TypeScript, Vite, Tailwind CSS, Recharts, Lucide Icons

---

## [0.1.0] - 2026-02-23

### Added
- Initial project scaffolding
- Basic data model design
- Proof-of-concept AI demand forecaster
- Project README and architecture documentation

---

## Version History

| Version | Date | Description |
|---|---|---|
| 1.0.0 | 2026-02-23 | GA release with full SCM suite |
| 0.1.0 | 2026-02-23 | Initial scaffolding and PoC |
