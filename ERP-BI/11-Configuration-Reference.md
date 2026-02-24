# ERP-BI Configuration Reference

| Field | Value |
|---|---|
| Module | ERP-BI |
| Version | 1.0.0 |
| Last Updated | 2026-02-23 |

---

## 1. Environment Configuration

### 1.1 Core Settings

| Variable | Type | Default | Description |
|---|---|---|---|
| PORT | int | 8080 | HTTP server port |
| MODULE_NAME | string | ERP-BI | Module identifier for health checks |
| LOG_LEVEL | string | info | Logging level (debug, info, warn, error) |
| LOG_FORMAT | string | json | Log format (json, text) |
| ENV | string | production | Environment (development, staging, production) |

### 1.2 Database Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| DATABASE_URL | string | - | PostgreSQL connection string |
| CLICKHOUSE_HOST | string | localhost | ClickHouse host |
| CLICKHOUSE_PORT | int | 8123 | ClickHouse HTTP port |
| CLICKHOUSE_DATABASE | string | erp_bi | ClickHouse database name |
| CLICKHOUSE_USER | string | default | ClickHouse username |
| CLICKHOUSE_PASSWORD | string | - | ClickHouse password |
| CLICKHOUSE_CLUSTER | string | - | ClickHouse cluster name |

### 1.3 Cache Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| REDIS_URL | string | - | Redis connection URL |
| REDIS_CLUSTER | bool | false | Enable Redis cluster mode |
| CACHE_L1_TTL | duration | 30s | In-process cache TTL |
| CACHE_L2_TTL | duration | 5m | Redis cache TTL |
| CACHE_ENABLED | bool | true | Enable/disable caching |

### 1.4 NATS Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| NATS_URL | string | - | NATS server URL |
| NATS_CLUSTER_ID | string | erp-nats | NATS cluster ID |
| NATS_CONSUMER_GROUP | string | bi-ingestion | Consumer group name |
| NATS_MAX_PENDING | int | 1000 | Max pending messages |

### 1.5 Integration Configuration

| Variable | Type | Default | Description |
|---|---|---|---|
| IAM_URL | string | - | ERP-IAM service URL |
| AI_URL | string | - | ERP-AI service URL |
| PLATFORM_URL | string | - | ERP-Platform service URL |

---

## 2. Governor Configuration

```yaml
governor:
  max_rows_per_query: 1000000
  max_query_time_ms: 30000
  max_concurrent_queries: 50
  max_query_size_bytes: 10485760
  max_join_depth: 5
  cost_estimation_enabled: true
  cost_threshold: 1000000
  rate_limit:
    free: 60        # requests/min
    professional: 300
    enterprise: 1000
    unlimited: 0     # no limit
```

---

## 3. Alert Configuration

```yaml
alerts:
  evaluation_interval: 60s
  notification:
    email:
      smtp_host: ${SMTP_HOST}
      smtp_port: 587
      from_address: alerts@erp.example.com
      tls_enabled: true
    slack:
      webhook_url: ${SLACK_WEBHOOK}
      default_channel: "#bi-alerts"
    webhook:
      timeout: 10s
      retry_count: 3
  escalation:
    default_timeout: 30m
    max_escalation_levels: 3
```

---

## 4. Report Configuration

```yaml
reports:
  rendering:
    pdf_engine: headless-chrome
    max_pages: 500
    timeout: 120s
  scheduling:
    max_concurrent_executions: 10
    retry_count: 3
    retry_delay: 60s
  storage:
    bucket: ${S3_BUCKET}
    prefix: reports/
    retention_days: 90
  delivery:
    email_max_attachment_size: 25MB
    slack_max_file_size: 10MB
```

---

## 5. ClickHouse Tuning

```yaml
clickhouse:
  max_memory_usage: 10737418240  # 10 GB
  max_execution_time: 30
  max_threads: 8
  max_concurrent_queries: 100
  merge_tree_max_rows_to_use_cache: 10000000
  enable_http_compression: true
  distributed_aggregation_memory_efficient: true
```

---

## 6. Frontend Configuration (Next.js)

From `next.config.js`:

```javascript
module.exports = {
  reactStrictMode: true,
  env: {
    API_URL: process.env.API_URL || 'http://localhost:8080',
    WS_URL: process.env.WS_URL || 'ws://localhost:8080',
  },
};
```

---

## 7. Tailwind Configuration

From `tailwind.config.ts` - custom design tokens for the BI module including manufacturing-specific color palette, responsive breakpoints, and animation utilities.

---

## 8. Prisma Configuration

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```
