# ERP-BI Glossary

| Field | Value |
|---|---|
| Module | ERP-BI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## Terms

| Term | Definition |
|---|---|
| **AIDD** | AI-Driven Development framework that classifies AI actions as autonomous, supervised, or prohibited |
| **Anomaly Detection** | Statistical analysis that identifies data points deviating significantly from expected patterns using Z-Score and IQR methods |
| **BOM** | Bill of Materials - hierarchical structure defining components needed to manufacture a product |
| **CDC** | Change Data Capture - mechanism to detect and deliver data changes from source systems in near real-time |
| **ClickHouse** | Columnar OLAP database engine used for analytical query processing |
| **Cross-filtering** | Interactive feature where selecting an element in one chart filters all other charts on the dashboard |
| **DAX** | Data Analysis Expressions - formula language used by Power BI (equivalent: calculated fields in ERP-BI) |
| **Dimension** | Descriptive attribute used for grouping and filtering (e.g., product name, region, date) |
| **Drill-down** | Navigation from aggregated data to more granular detail levels |
| **Ensemble** | Combining multiple forecasting methods to produce a weighted average prediction |
| **ETL** | Extract, Transform, Load - process of moving data from source systems to the data warehouse |
| **Fact Table** | Central table in a star schema containing quantitative metrics (e.g., revenue, quantity) |
| **Governor** | Query limiter that prevents resource exhaustion by enforcing row limits, timeouts, and concurrency caps |
| **Holt-Winters** | Double exponential smoothing method for time-series forecasting |
| **IQR** | Interquartile Range - statistical measure used for anomaly detection |
| **LookML** | Looker Modeling Language - Looker's proprietary data modeling syntax |
| **MAPE** | Mean Absolute Percentage Error - accuracy metric for forecasting models |
| **Materialized View** | Pre-computed query result stored in ClickHouse, automatically updated on data changes |
| **Measure** | Quantitative value that can be aggregated (SUM, AVG, COUNT, etc.) |
| **MergeTree** | ClickHouse table engine family optimized for analytical workloads |
| **mTLS** | Mutual TLS - bidirectional certificate-based authentication between services |
| **NATS** | Lightweight messaging system used as the ERP event backbone |
| **NLQ** | Natural Language Querying - text input converted to SQL queries via AI |
| **OEE** | Overall Equipment Effectiveness - manufacturing KPI (Availability x Performance x Quality) |
| **OLAP** | Online Analytical Processing - optimized for complex queries across large datasets |
| **Pre-aggregation** | Summary tables computed in advance to speed up common queries |
| **Prisma** | TypeScript ORM used for PostgreSQL schema management and queries |
| **RLS** | Row-Level Security - restricting data visibility based on user attributes |
| **Semantic Layer** | Abstraction that maps physical database columns to business-friendly names |
| **Star Schema** | Data warehouse design with central fact tables connected to dimension tables |
| **Snowflake Schema** | Normalized variant of star schema where dimensions have sub-dimensions |
| **TTL** | Time-To-Live - expiration time for cached data or database partitions |
| **White-labeling** | Customizing the UI appearance (logo, colors, domain) for customer branding |
| **Z-Score** | Number of standard deviations a data point is from the mean, used for anomaly detection |
