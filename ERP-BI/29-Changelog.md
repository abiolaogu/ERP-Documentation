# ERP-BI Changelog

| Field | Value |
|---|---|
| Module | ERP-BI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## Version 1.0.0 (2026-02-23) -- Initial Release

### Added

#### Dashboard Builder
- Drag-and-drop widget placement with flexible grid layout
- 30+ chart types (bar, line, area, pie, donut, scatter, heatmap, treemap, funnel, gauge, geo-map, waterfall, sankey, table, pivot, KPI, sparkline, etc.)
- Real-time data refresh via WebSocket subscriptions
- Multi-level drill-down with breadcrumb navigation
- Cross-filtering between widgets
- Embeddable dashboards via iframe with JWT authentication
- Full white-labeling (logo, colors, fonts, custom domain)
- Dashboard templates library
- Mobile-responsive layout

#### Report Builder
- Paginated report designer with WYSIWYG editor
- Tabular, matrix/pivot, and free-form report layouts
- Sub-report embedding with parameter passing
- Dynamic parameters with cascading filters
- Report scheduling (cron-based and event-driven)
- Delivery via email, Slack, and webhook
- Export to PDF, Excel, CSV, and PowerPoint

#### Data Modeling
- Semantic layer with business-friendly naming
- Calculated fields with expression builder
- Measures (SUM, AVG, COUNT, DISTINCT, etc.)
- Dimensions with hierarchy support
- Data blending across multiple data sources
- Row-Level Security integrated with ERP-IAM

#### Query Engine
- ClickHouse OLAP backend for sub-second aggregations
- Star and snowflake schema support
- Materialized view management
- Pre-aggregation tables with automatic refresh
- Multi-tier query caching (L1 in-process, L2 Redis)
- Governor limits (max rows, max query time, concurrency)

#### Data Warehouse
- Automated ETL from all ERP modules via CDC
- Schema management with migration tracking
- Full data lineage graph (source to dashboard)
- Data quality monitoring with anomaly scoring

#### Alerts
- Threshold-based alerts on any metric
- AI anomaly detection alerts
- Trend-based alerts
- Multi-channel notifications (email, Slack, webhook, in-app)
- Escalation policies

#### NLQ (Natural Language Querying)
- Natural language input with intent parsing
- Text to SQL generation via Claude
- Automatic chart type suggestion
- SQL validation and injection prevention
- Query history

#### AI Engines
- Demand forecasting (Holt-Winters, WMA, Linear Regression, Ensemble)
- Anomaly detection (Z-Score, IQR, multi-metric scoring)
- Predictive maintenance (degradation modeling)
- Quality prediction (historical pattern matching)
- Capacity optimization (constraint solver)

#### Infrastructure
- 7 Go microservices (dashboard, report, data-modeling, query-engine, data-warehouse, alert, nlq)
- Next.js 14 frontend with TypeScript and Tailwind CSS
- PostgreSQL metadata store with Prisma ORM
- ClickHouse OLAP cluster
- Redis caching layer
- NATS JetStream event backbone
- Kubernetes deployment with Helm
- Full observability (OpenTelemetry, Prometheus, Loki)

---

## Planned: Version 1.1.0

### Planned Features
- Dashboard versioning and change history
- Report bursting (per-tenant, per-department)
- Model versioning and promotion workflow
- Additional 20 chart types
- Collaborative dashboard editing
- Mobile-native app
