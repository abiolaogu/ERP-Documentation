# ERP-Observability Integration Patterns

> **Document ID:** ERP-OBS-INT-029
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [01-Technical-Writeup.md](./01-Technical-Writeup.md), [14-Technical-Specifications.md](./14-Technical-Specifications.md), [28-Multi-Tenancy-Guide.md](./28-Multi-Tenancy-Guide.md)

---

## 1. Overview

This document defines the standard integration patterns for connecting ERP modules to the observability platform. Every ERP module instruments its Go microservices and Node.js backends with OpenTelemetry SDKs, emitting metrics, logs, and traces through a unified pipeline. This guide establishes the conventions, naming standards, and configuration templates that ensure consistent telemetry across all 20+ ERP modules.

### Integration Architecture

```mermaid
graph LR
    subgraph "ERP Module (e.g., ERP-CRM)"
        APP["Go Microservice<br/>(OTel SDK)"]
        FE["React Frontend<br/>(OTel JS SDK)"]
        DB["PostgreSQL/YugabyteDB"]
    end

    subgraph "Observability Pipeline"
        OTEL["OTel Collector<br/>:4317 (gRPC) / :4318 (HTTP)"]
    end

    subgraph "Storage & Visualization"
        VM["VictoriaMetrics"]
        QW["Quickwit"]
        GF["Grafana"]
    end

    APP -->|OTLP gRPC| OTEL
    FE -->|OTLP HTTP| OTEL
    APP -->|SQL queries<br/>(auto-instrumented)| DB
    OTEL -->|Remote Write| VM
    OTEL -->|OTLP Export| QW
    VM --> GF
    QW --> GF
```

### Integration Checklist for New Modules

| Step | Description | Owner |
|------|-------------|-------|
| 1 | Add OTel Go SDK dependency | Module team |
| 2 | Initialize tracer, meter, and logger providers | Module team |
| 3 | Add HTTP middleware for automatic span creation | Module team |
| 4 | Add database instrumentation (pgx/gorm OTel plugin) | Module team |
| 5 | Define module-specific business metrics | Module team |
| 6 | Configure OTel Collector pipeline for the module | Observability team |
| 7 | Create Grafana dashboard from template | Observability team |
| 8 | Configure alert rules for the module | Module + Obs team |
| 9 | Add Zabbix host monitoring for infrastructure | Observability team |
| 10 | Validate tenant isolation for module telemetry | QA team |

---

## 2. OTel SDK Instrumentation for Go Services

### 2.1 Standard OTel Initialization

Every Go microservice in the ERP suite uses a shared initialization package:

```go
// pkg/observability/init.go
package observability

import (
    "context"
    "fmt"
    "os"
    "time"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/exporters/otlp/otlpmetric/otlpmetricgrpc"
    "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
    "go.opentelemetry.io/otel/log/global"
    "go.opentelemetry.io/otel/propagation"
    "go.opentelemetry.io/otel/sdk/log"
    "go.opentelemetry.io/otel/sdk/metric"
    "go.opentelemetry.io/otel/sdk/resource"
    "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.21.0"
)

type Config struct {
    ServiceName    string
    ServiceVersion string
    Environment    string
    OTelEndpoint   string  // Default: "otel-collector:4317"
    TenantID       string  // Injected from app context
}

func Init(ctx context.Context, cfg Config) (func(context.Context) error, error) {
    if cfg.OTelEndpoint == "" {
        cfg.OTelEndpoint = os.Getenv("OTEL_EXPORTER_OTLP_ENDPOINT")
        if cfg.OTelEndpoint == "" {
            cfg.OTelEndpoint = "otel-collector:4317"
        }
    }

    // Create resource with standard ERP attributes
    res, err := resource.Merge(
        resource.Default(),
        resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceName(cfg.ServiceName),
            semconv.ServiceVersion(cfg.ServiceVersion),
            semconv.DeploymentEnvironment(cfg.Environment),
            attribute.String("tenant_id", cfg.TenantID),
            attribute.String("erp.module", cfg.ServiceName),
        ),
    )
    if err != nil {
        return nil, fmt.Errorf("failed to create resource: %w", err)
    }

    // Initialize trace exporter
    traceExporter, err := otlptracegrpc.New(ctx,
        otlptracegrpc.WithEndpoint(cfg.OTelEndpoint),
        otlptracegrpc.WithInsecure(),
    )
    if err != nil {
        return nil, fmt.Errorf("failed to create trace exporter: %w", err)
    }

    tracerProvider := trace.NewTracerProvider(
        trace.WithBatcher(traceExporter,
            trace.WithBatchTimeout(5*time.Second),
            trace.WithMaxExportBatchSize(512),
        ),
        trace.WithResource(res),
        trace.WithSampler(trace.ParentBased(trace.TraceIDRatioBased(0.1))), // 10% sampling
    )
    otel.SetTracerProvider(tracerProvider)

    // Initialize metric exporter
    metricExporter, err := otlpmetricgrpc.New(ctx,
        otlpmetricgrpc.WithEndpoint(cfg.OTelEndpoint),
        otlpmetricgrpc.WithInsecure(),
    )
    if err != nil {
        return nil, fmt.Errorf("failed to create metric exporter: %w", err)
    }

    meterProvider := metric.NewMeterProvider(
        metric.WithReader(metric.NewPeriodicReader(metricExporter,
            metric.WithInterval(15*time.Second),
        )),
        metric.WithResource(res),
    )
    otel.SetMeterProvider(meterProvider)

    // Set global propagator (W3C Trace Context + Baggage)
    otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
        propagation.TraceContext{},
        propagation.Baggage{},
    ))

    // Shutdown function
    shutdown := func(ctx context.Context) error {
        var errs []error
        if err := tracerProvider.Shutdown(ctx); err != nil {
            errs = append(errs, err)
        }
        if err := meterProvider.Shutdown(ctx); err != nil {
            errs = append(errs, err)
        }
        if len(errs) > 0 {
            return fmt.Errorf("shutdown errors: %v", errs)
        }
        return nil
    }

    return shutdown, nil
}
```

### 2.2 HTTP Middleware Integration

```go
// pkg/observability/middleware.go
package observability

import (
    "net/http"
    "time"

    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/metric"
)

var (
    httpRequestsTotal   metric.Int64Counter
    httpRequestDuration metric.Float64Histogram
    httpActiveRequests  metric.Int64UpDownCounter
)

func InitHTTPMetrics(moduleName string) {
    meter := otel.Meter(moduleName)

    httpRequestsTotal, _ = meter.Int64Counter(
        fmt.Sprintf("erp_%s_http_requests_total", moduleName),
        metric.WithDescription("Total number of HTTP requests"),
        metric.WithUnit("{request}"),
    )

    httpRequestDuration, _ = meter.Float64Histogram(
        fmt.Sprintf("erp_%s_http_request_duration_seconds", moduleName),
        metric.WithDescription("HTTP request duration in seconds"),
        metric.WithUnit("s"),
        metric.WithExplicitBucketBoundaries(0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10),
    )

    httpActiveRequests, _ = meter.Int64UpDownCounter(
        fmt.Sprintf("erp_%s_http_active_requests", moduleName),
        metric.WithDescription("Number of active HTTP requests"),
        metric.WithUnit("{request}"),
    )
}

// WrapHandler wraps an http.Handler with OTel tracing and metrics
func WrapHandler(handler http.Handler, operation string) http.Handler {
    // OTel HTTP instrumentation (automatic span creation)
    traced := otelhttp.NewHandler(handler, operation)

    // Custom metrics middleware
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()

        attrs := []attribute.KeyValue{
            attribute.String("http.method", r.Method),
            attribute.String("http.route", operation),
            attribute.String("tenant_id", r.Header.Get("X-Scope-OrgID")),
        }

        httpActiveRequests.Add(r.Context(), 1, metric.WithAttributes(attrs...))
        defer httpActiveRequests.Add(r.Context(), -1, metric.WithAttributes(attrs...))

        // Use response writer wrapper to capture status code
        rw := &responseWriter{ResponseWriter: w, statusCode: 200}
        traced.ServeHTTP(rw, r)

        duration := time.Since(start).Seconds()
        statusAttrs := append(attrs, attribute.Int("http.status_code", rw.statusCode))

        httpRequestsTotal.Add(r.Context(), 1, metric.WithAttributes(statusAttrs...))
        httpRequestDuration.Record(r.Context(), duration, metric.WithAttributes(statusAttrs...))
    })
}

type responseWriter struct {
    http.ResponseWriter
    statusCode int
}

func (rw *responseWriter) WriteHeader(code int) {
    rw.statusCode = code
    rw.ResponseWriter.WriteHeader(code)
}
```

### 2.3 Database Instrumentation

```go
// pkg/observability/database.go
package observability

import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5/pgxpool"
    "github.com/exaring/otelpgx"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/metric"
)

var (
    dbQueryTotal    metric.Int64Counter
    dbQueryDuration metric.Float64Histogram
    dbConnections   metric.Int64UpDownCounter
)

func InitDBMetrics(moduleName string) {
    meter := otel.Meter(moduleName)

    dbQueryTotal, _ = meter.Int64Counter(
        fmt.Sprintf("erp_%s_db_queries_total", moduleName),
        metric.WithDescription("Total database queries executed"),
        metric.WithUnit("{query}"),
    )

    dbQueryDuration, _ = meter.Float64Histogram(
        fmt.Sprintf("erp_%s_db_query_duration_seconds", moduleName),
        metric.WithDescription("Database query duration in seconds"),
        metric.WithUnit("s"),
        metric.WithExplicitBucketBoundaries(0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 5),
    )

    dbConnections, _ = meter.Int64UpDownCounter(
        fmt.Sprintf("erp_%s_db_connections", moduleName),
        metric.WithDescription("Active database connections"),
        metric.WithUnit("{connection}"),
    )
}

// NewInstrumentedPool creates a pgxpool with OTel tracing
func NewInstrumentedPool(ctx context.Context, connString string) (*pgxpool.Pool, error) {
    cfg, err := pgxpool.ParseConfig(connString)
    if err != nil {
        return nil, fmt.Errorf("failed to parse connection string: %w", err)
    }

    // Add OTel tracing to pgx
    cfg.ConnConfig.Tracer = otelpgx.NewTracer(
        otelpgx.WithTrimSQLInSpanName(),
        otelpgx.WithIncludeQueryParameters(),
    )

    pool, err := pgxpool.NewWithConfig(ctx, cfg)
    if err != nil {
        return nil, fmt.Errorf("failed to create pool: %w", err)
    }

    return pool, nil
}
```

---

## 3. Standard Metric Naming Conventions

### 3.1 Naming Schema

All ERP module metrics follow this naming pattern:

```
erp_{module}_{subsystem}_{metric_name}_{unit}
```

| Component | Convention | Examples |
|-----------|-----------|----------|
| **Prefix** | Always `erp_` | `erp_` |
| **Module** | Lowercase module name, no hyphens | `crm`, `iam`, `accounting`, `inventory`, `hrm` |
| **Subsystem** | Functional area within the module | `http`, `db`, `grpc`, `cache`, `queue`, `business` |
| **Metric Name** | Descriptive, snake_case | `requests_total`, `query_duration`, `items_processed` |
| **Unit** | Appended when not obvious | `_seconds`, `_bytes`, `_ratio` |

### 3.2 Standard Metrics Per Module

Every ERP module MUST expose these standard metrics:

#### HTTP Metrics

| Metric Name | Type | Labels | Description |
|-------------|------|--------|-------------|
| `erp_{module}_http_requests_total` | Counter | method, route, status_code, tenant_id | Total HTTP requests |
| `erp_{module}_http_request_duration_seconds` | Histogram | method, route, tenant_id | Request latency |
| `erp_{module}_http_active_requests` | Gauge | method, route, tenant_id | In-flight requests |
| `erp_{module}_http_request_size_bytes` | Histogram | method, route, tenant_id | Request body size |
| `erp_{module}_http_response_size_bytes` | Histogram | method, route, tenant_id | Response body size |

#### Database Metrics

| Metric Name | Type | Labels | Description |
|-------------|------|--------|-------------|
| `erp_{module}_db_queries_total` | Counter | operation, table, tenant_id | Total DB queries |
| `erp_{module}_db_query_duration_seconds` | Histogram | operation, table, tenant_id | Query latency |
| `erp_{module}_db_connections` | Gauge | state (active/idle), tenant_id | Connection pool state |
| `erp_{module}_db_errors_total` | Counter | operation, error_type, tenant_id | Database errors |

#### Business Metrics (Module-Specific Examples)

| Module | Metric Name | Type | Description |
|--------|-------------|------|-------------|
| CRM | `erp_crm_contacts_created_total` | Counter | New contacts created |
| CRM | `erp_crm_deals_value_total` | Counter | Total deal value processed |
| IAM | `erp_iam_auth_attempts_total` | Counter | Authentication attempts |
| IAM | `erp_iam_active_sessions` | Gauge | Current active sessions |
| Accounting | `erp_accounting_transactions_total` | Counter | Financial transactions |
| Accounting | `erp_accounting_journal_entries_total` | Counter | Journal entries posted |
| Inventory | `erp_inventory_items_count` | Gauge | Current inventory level |
| Inventory | `erp_inventory_stockouts_total` | Counter | Stockout events |
| HRM | `erp_hrm_employees_active` | Gauge | Active employee count |
| HRM | `erp_hrm_payroll_runs_total` | Counter | Payroll processing runs |
| Procurement | `erp_procurement_orders_total` | Counter | Purchase orders created |
| Manufacturing | `erp_manufacturing_production_rate` | Gauge | Units per hour |
| Logistics | `erp_logistics_shipments_total` | Counter | Shipments dispatched |

#### Runtime Metrics

| Metric Name | Type | Labels | Description |
|-------------|------|--------|-------------|
| `erp_{module}_go_goroutines` | Gauge | | Active goroutines |
| `erp_{module}_go_gc_duration_seconds` | Summary | | GC pause duration |
| `erp_{module}_go_memstats_alloc_bytes` | Gauge | | Allocated heap memory |
| `erp_{module}_process_cpu_seconds_total` | Counter | | CPU time consumed |
| `erp_{module}_process_resident_memory_bytes` | Gauge | | RSS memory |

### 3.3 Label Standards

| Label | Required | Description | Example Values |
|-------|----------|-------------|----------------|
| `tenant_id` | Yes | Tenant identifier | `tenant-acme`, `tenant-globex` |
| `service_name` | Yes (auto) | OTel resource attribute | `erp-crm`, `erp-iam` |
| `deployment_environment` | Yes (auto) | Environment | `production`, `staging`, `development` |
| `instance` | Yes (auto) | Pod/container instance | `erp-crm-7d4f8b9-abc12` |
| `http_method` | For HTTP metrics | HTTP method | `GET`, `POST`, `PUT`, `DELETE` |
| `http_route` | For HTTP metrics | Route pattern (not path) | `/api/v1/contacts`, `/api/v1/deals/{id}` |
| `http_status_code` | For HTTP metrics | Response status | `200`, `404`, `500` |
| `db_operation` | For DB metrics | SQL operation | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| `db_table` | For DB metrics | Target table | `contacts`, `deals`, `invoices` |

**Important:** Use route patterns (e.g., `/api/v1/contacts/{id}`) not actual paths (e.g., `/api/v1/contacts/123`) to prevent label cardinality explosion.

---

## 4. Standard Log Attributes

### 4.1 Required Log Fields

Every log entry emitted by an ERP module MUST include these attributes:

```json
{
  "timestamp": "2026-02-24T10:30:00.123Z",
  "severity": "INFO",
  "body": "Contact created successfully",
  "resource": {
    "service.name": "erp-crm",
    "service.version": "1.2.0",
    "deployment.environment": "production",
    "tenant_id": "tenant-acme"
  },
  "attributes": {
    "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
    "span_id": "00f067aa0ba902b7",
    "module": "crm",
    "component": "contact-service",
    "user_id": "user-12345",
    "request_id": "req-abc-123",
    "operation": "create_contact"
  }
}
```

### 4.2 Attribute Reference

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `timestamp` | datetime | Yes | RFC 3339 timestamp with millisecond precision |
| `severity` | string | Yes | One of: TRACE, DEBUG, INFO, WARN, ERROR, FATAL |
| `body` | string | Yes | Human-readable log message |
| `service.name` | string | Yes | OTel resource: `erp-{module}` |
| `service.version` | string | Yes | Semantic version of the module |
| `deployment.environment` | string | Yes | `production`, `staging`, `development` |
| `tenant_id` | string | Yes | Tenant identifier |
| `trace_id` | string | Conditional | W3C trace ID (present when in request context) |
| `span_id` | string | Conditional | W3C span ID (present when in request context) |
| `module` | string | Yes | Short module name: `crm`, `iam`, `accounting` |
| `component` | string | Recommended | Sub-component: `contact-service`, `auth-handler` |
| `user_id` | string | Recommended | Acting user identifier |
| `request_id` | string | Recommended | Unique request correlation ID |
| `operation` | string | Recommended | Business operation: `create_contact`, `process_payment` |
| `error.type` | string | For errors | Error class: `ValidationError`, `DatabaseError` |
| `error.message` | string | For errors | Error message text |
| `error.stack_trace` | string | For errors | Stack trace (truncated to 4KB) |

### 4.3 Structured Logging Implementation

```go
// pkg/observability/logging.go
package observability

import (
    "context"
    "log/slog"
    "os"

    "go.opentelemetry.io/otel/trace"
)

// NewLogger creates a structured logger with standard ERP attributes
func NewLogger(moduleName, version, environment string) *slog.Logger {
    handler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
        Level: slog.LevelInfo,
    })

    return slog.New(&ContextHandler{
        handler:     handler,
        moduleName:  moduleName,
        version:     version,
        environment: environment,
    })
}

// ContextHandler enriches log entries with trace context and tenant info
type ContextHandler struct {
    handler     slog.Handler
    moduleName  string
    version     string
    environment string
}

func (h *ContextHandler) Handle(ctx context.Context, r slog.Record) error {
    // Add standard resource attributes
    r.AddAttrs(
        slog.String("service.name", "erp-"+h.moduleName),
        slog.String("service.version", h.version),
        slog.String("deployment.environment", h.environment),
        slog.String("module", h.moduleName),
    )

    // Add tenant_id from context
    if tenantID, ok := ctx.Value(TenantIDKey).(string); ok {
        r.AddAttrs(slog.String("tenant_id", tenantID))
    }

    // Add trace context if present
    spanCtx := trace.SpanFromContext(ctx).SpanContext()
    if spanCtx.HasTraceID() {
        r.AddAttrs(
            slog.String("trace_id", spanCtx.TraceID().String()),
            slog.String("span_id", spanCtx.SpanID().String()),
        )
    }

    // Add request_id from context
    if reqID, ok := ctx.Value("request_id").(string); ok {
        r.AddAttrs(slog.String("request_id", reqID))
    }

    return h.handler.Handle(ctx, r)
}

func (h *ContextHandler) Enabled(ctx context.Context, level slog.Level) bool {
    return h.handler.Enabled(ctx, level)
}

func (h *ContextHandler) WithAttrs(attrs []slog.Attr) slog.Handler {
    return &ContextHandler{
        handler:     h.handler.WithAttrs(attrs),
        moduleName:  h.moduleName,
        version:     h.version,
        environment: h.environment,
    }
}

func (h *ContextHandler) WithGroup(name string) slog.Handler {
    return &ContextHandler{
        handler:     h.handler.WithGroup(name),
        moduleName:  h.moduleName,
        version:     h.version,
        environment: h.environment,
    }
}
```

---

## 5. Alert Routing Per Module

### 5.1 Module Alert Configuration

Each ERP module defines its alert rules in a standardized format. Alert rules are stored in VictoriaMetrics (via vmalert) and route through Alertmanager:

```yaml
# alert-rules/erp-crm.yml
groups:
  - name: erp-crm-alerts
    interval: 30s
    rules:
      # Availability: High error rate
      - alert: CRMHighErrorRate
        expr: |
          sum(rate(erp_crm_http_requests_total{status_code=~"5.."}[5m])) by (tenant_id)
          /
          sum(rate(erp_crm_http_requests_total[5m])) by (tenant_id)
          > 0.05
        for: 5m
        labels:
          severity: critical
          module: crm
          category: availability
        annotations:
          summary: "CRM error rate above 5% for tenant {{ $labels.tenant_id }}"
          description: "Error rate: {{ $value | humanizePercentage }}"
          runbook_url: "https://docs.erp.internal/runbooks/crm-high-error-rate"

      # Latency: Slow API responses
      - alert: CRMHighLatency
        expr: |
          histogram_quantile(0.99,
            sum(rate(erp_crm_http_request_duration_seconds_bucket[5m])) by (le, tenant_id)
          ) > 2
        for: 5m
        labels:
          severity: warning
          module: crm
          category: latency
        annotations:
          summary: "CRM p99 latency above 2s for tenant {{ $labels.tenant_id }}"
          description: "p99 latency: {{ $value | humanizeDuration }}"

      # Saturation: Database connection pool exhaustion
      - alert: CRMDBConnectionPoolExhausted
        expr: |
          erp_crm_db_connections{state="active"}
          /
          erp_crm_db_connections{state="idle"} + erp_crm_db_connections{state="active"}
          > 0.9
        for: 5m
        labels:
          severity: warning
          module: crm
          category: saturation
        annotations:
          summary: "CRM DB connection pool >90% utilized"

      # Business: Unusual deal creation patterns
      - alert: CRMDealCreationAnomaly
        expr: |
          abs(
            sum(rate(erp_crm_deals_value_total[1h])) by (tenant_id)
            -
            sum(rate(erp_crm_deals_value_total[1h] offset 7d)) by (tenant_id)
          )
          /
          sum(rate(erp_crm_deals_value_total[1h] offset 7d)) by (tenant_id)
          > 3
        for: 15m
        labels:
          severity: info
          module: crm
          category: business
        annotations:
          summary: "CRM deal creation rate 3x different from last week"
```

### 5.2 Alertmanager Routing by Module

```yaml
# alertmanager.yml - Module-specific routing
route:
  receiver: 'default'
  group_by: ['alertname', 'module', 'tenant_id']
  routes:
    # CRM module alerts
    - match:
        module: 'crm'
      receiver: 'crm-team'
      routes:
        - match:
            severity: 'critical'
          receiver: 'crm-oncall'

    # IAM module alerts
    - match:
        module: 'iam'
      receiver: 'iam-team'
      routes:
        - match:
            severity: 'critical'
          receiver: 'security-oncall'

    # Accounting module alerts
    - match:
        module: 'accounting'
      receiver: 'accounting-team'
      routes:
        - match:
            severity: 'critical'
          receiver: 'finance-oncall'

    # Inventory module alerts
    - match:
        module: 'inventory'
      receiver: 'inventory-team'

    # HRM module alerts
    - match:
        module: 'hrm'
      receiver: 'hr-team'

    # Observability self-monitoring
    - match:
        module: 'observability'
      receiver: 'platform-oncall'

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://observability-api:3000/api/v1/alerts/default'

  - name: 'crm-team'
    slack_configs:
      - api_url_file: '/secrets/slack-crm-webhook'
        channel: '#crm-alerts'
        title: '[{{ .Status | toUpper }}] {{ .GroupLabels.alertname }}'
        text: '{{ .CommonAnnotations.summary }}'

  - name: 'crm-oncall'
    pagerduty_configs:
      - routing_key_file: '/secrets/pagerduty-crm-key'
        severity: '{{ .CommonLabels.severity }}'

  - name: 'iam-team'
    slack_configs:
      - api_url_file: '/secrets/slack-iam-webhook'
        channel: '#iam-alerts'

  - name: 'security-oncall'
    pagerduty_configs:
      - routing_key_file: '/secrets/pagerduty-security-key'

  - name: 'accounting-team'
    slack_configs:
      - api_url_file: '/secrets/slack-accounting-webhook'
        channel: '#accounting-alerts'

  - name: 'finance-oncall'
    pagerduty_configs:
      - routing_key_file: '/secrets/pagerduty-finance-key'

  - name: 'inventory-team'
    slack_configs:
      - api_url_file: '/secrets/slack-inventory-webhook'
        channel: '#inventory-alerts'

  - name: 'hr-team'
    email_configs:
      - to: 'hr-ops@opensase.io'

  - name: 'platform-oncall'
    pagerduty_configs:
      - routing_key_file: '/secrets/pagerduty-platform-key'
    slack_configs:
      - api_url_file: '/secrets/slack-platform-webhook'
        channel: '#platform-alerts'
```

### 5.3 Standard Alert Categories

Every module should define alerts in these four categories (based on the RED and USE methods):

| Category | Focus | Example Alerts |
|----------|-------|----------------|
| **Availability** | Is the service responding? | High error rate, endpoint down, health check failure |
| **Latency** | How fast is the service? | p99 above SLO, slow queries, upstream timeout |
| **Saturation** | Is the service overloaded? | CPU > 90%, memory > 85%, connection pool exhausted, queue depth high |
| **Business** | Are business KPIs normal? | Unusual transaction volume, anomalous user behavior, SLA breach |

---

## 6. Dashboard Templates Per Module Type

### 6.1 Standard Module Dashboard

Every ERP module receives a provisioned Grafana dashboard with these standard panels:

```json
{
  "dashboard": {
    "title": "ERP-${MODULE} Overview",
    "tags": ["erp", "${module}", "auto-provisioned"],
    "templating": {
      "list": [
        {
          "name": "tenant_id",
          "type": "query",
          "query": "label_values(erp_${module}_http_requests_total, tenant_id)",
          "datasource": "VictoriaMetrics",
          "multi": true,
          "includeAll": true
        },
        {
          "name": "instance",
          "type": "query",
          "query": "label_values(erp_${module}_http_requests_total{tenant_id=~\"$tenant_id\"}, instance)",
          "datasource": "VictoriaMetrics",
          "multi": true,
          "includeAll": true
        }
      ]
    },
    "panels": [
      {
        "title": "Request Rate",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0},
        "targets": [{
          "expr": "sum(rate(erp_${module}_http_requests_total{tenant_id=~\"$tenant_id\"}[5m])) by (status_code)",
          "legendFormat": "{{status_code}}"
        }]
      },
      {
        "title": "Request Latency (p50, p95, p99)",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0},
        "targets": [
          {
            "expr": "histogram_quantile(0.50, sum(rate(erp_${module}_http_request_duration_seconds_bucket{tenant_id=~\"$tenant_id\"}[5m])) by (le))",
            "legendFormat": "p50"
          },
          {
            "expr": "histogram_quantile(0.95, sum(rate(erp_${module}_http_request_duration_seconds_bucket{tenant_id=~\"$tenant_id\"}[5m])) by (le))",
            "legendFormat": "p95"
          },
          {
            "expr": "histogram_quantile(0.99, sum(rate(erp_${module}_http_request_duration_seconds_bucket{tenant_id=~\"$tenant_id\"}[5m])) by (le))",
            "legendFormat": "p99"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "stat",
        "gridPos": {"h": 4, "w": 6, "x": 0, "y": 8},
        "targets": [{
          "expr": "sum(rate(erp_${module}_http_requests_total{status_code=~\"5..\",tenant_id=~\"$tenant_id\"}[5m])) / sum(rate(erp_${module}_http_requests_total{tenant_id=~\"$tenant_id\"}[5m])) * 100",
          "legendFormat": ""
        }],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 1},
                {"color": "red", "value": 5}
              ]
            }
          }
        }
      },
      {
        "title": "Active Requests",
        "type": "stat",
        "gridPos": {"h": 4, "w": 6, "x": 6, "y": 8},
        "targets": [{
          "expr": "sum(erp_${module}_http_active_requests{tenant_id=~\"$tenant_id\"})"
        }]
      },
      {
        "title": "DB Query Duration (p99)",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 12},
        "targets": [{
          "expr": "histogram_quantile(0.99, sum(rate(erp_${module}_db_query_duration_seconds_bucket{tenant_id=~\"$tenant_id\"}[5m])) by (le, operation))",
          "legendFormat": "{{operation}}"
        }]
      },
      {
        "title": "DB Connections",
        "type": "timeseries",
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 12},
        "targets": [{
          "expr": "erp_${module}_db_connections{tenant_id=~\"$tenant_id\"}",
          "legendFormat": "{{state}}"
        }]
      },
      {
        "title": "Recent Logs",
        "type": "logs",
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 20},
        "datasource": "Quickwit Logs",
        "targets": [{
          "query": "service_name:erp-${module} AND severity:(ERROR OR WARN)",
          "limit": 50
        }]
      }
    ]
  }
}
```

### 6.2 Module-Specific Dashboard Extensions

In addition to the standard dashboard, each module type gets specialized panels:

| Module Type | Additional Panels |
|-------------|-------------------|
| **CRM** | Contacts created/day, Deal pipeline value, Lead conversion funnel |
| **IAM** | Auth success/failure rate, Active sessions gauge, Token issuance rate |
| **Accounting** | Transaction volume, Journal entries/hour, GL balance summary |
| **Inventory** | Stock level gauges, Stockout count, Reorder alerts |
| **HRM** | Employee count, Payroll processing time, Leave request rate |
| **Procurement** | PO count, Approval pending queue, Vendor response time |
| **Manufacturing** | Production rate, Defect rate, Equipment uptime |
| **Logistics** | Shipment count, Delivery SLA %, Transit time distribution |

---

## 7. Grafana Provisioning for New Modules

### 7.1 Automated Dashboard Provisioning

When a new ERP module is registered, the observability platform automatically provisions:

```go
// internal/provisioner/module_provisioner.go
package provisioner

import (
    "context"
    "fmt"
    "text/template"
    "bytes"
)

type ModuleProvisioner struct {
    grafanaClient    *GrafanaClient
    alertRuleManager *AlertRuleManager
    templates        map[string]*template.Template
}

type ModuleConfig struct {
    ModuleName       string   // e.g., "crm"
    DisplayName      string   // e.g., "ERP-CRM"
    ServiceName      string   // e.g., "erp-crm"
    Port             int      // e.g., 8091
    HasDatabase      bool     // Whether to include DB panels
    CustomMetrics    []CustomMetric
    AlertRecipients  AlertRecipients
    TenantIDs        []string // Tenants using this module
}

type CustomMetric struct {
    Name        string
    Type        string // counter, gauge, histogram
    Description string
    PanelType   string // timeseries, stat, gauge, bar
}

func (p *ModuleProvisioner) ProvisionModule(ctx context.Context, cfg ModuleConfig) error {
    // 1. Generate and apply standard dashboard
    dashboardJSON, err := p.generateDashboard(cfg)
    if err != nil {
        return fmt.Errorf("dashboard generation failed: %w", err)
    }

    for _, tenantID := range cfg.TenantIDs {
        orgID, err := p.grafanaClient.GetOrgIDForTenant(tenantID)
        if err != nil {
            continue
        }
        err = p.grafanaClient.CreateDashboard(orgID, dashboardJSON)
        if err != nil {
            return fmt.Errorf("dashboard creation failed for tenant %s: %w", tenantID, err)
        }
    }

    // 2. Generate and apply alert rules
    alertRules, err := p.generateAlertRules(cfg)
    if err != nil {
        return fmt.Errorf("alert rule generation failed: %w", err)
    }
    err = p.alertRuleManager.Apply(ctx, alertRules)
    if err != nil {
        return fmt.Errorf("alert rule application failed: %w", err)
    }

    // 3. Configure Alertmanager routing for the module
    err = p.configureAlertRouting(ctx, cfg)
    if err != nil {
        return fmt.Errorf("alert routing configuration failed: %w", err)
    }

    // 4. Create Zabbix host group for the module's infrastructure
    err = p.createZabbixHostGroup(ctx, cfg)
    if err != nil {
        return fmt.Errorf("Zabbix provisioning failed: %w", err)
    }

    return nil
}
```

### 7.2 Grafana Folder Structure

```
Grafana Dashboards/
├── Platform/
│   ├── Observability Overview
│   ├── Infrastructure Health
│   ├── Cross-Module Latency
│   └── Tenant Usage Summary
├── ERP-CRM/
│   ├── CRM Overview (auto-provisioned)
│   ├── CRM Business Metrics
│   └── CRM Custom
├── ERP-IAM/
│   ├── IAM Overview (auto-provisioned)
│   ├── IAM Security Dashboard
│   └── IAM Custom
├── ERP-Accounting/
│   ├── Accounting Overview (auto-provisioned)
│   ├── Accounting Financial Dashboard
│   └── Accounting Custom
└── ... (one folder per module)
```

### 7.3 Datasource Provisioning via YAML

```yaml
# grafana/provisioning/datasources/erp-modules.yml
apiVersion: 1
datasources:
  - name: VictoriaMetrics
    type: prometheus
    access: proxy
    url: http://vmselect:8481/select/0/prometheus
    isDefault: true
    jsonData:
      timeInterval: "15s"
      httpMethod: POST
      manageAlerts: true
      prometheusType: VictoriaMetrics
      cacheLevel: "High"

  - name: Quickwit Logs
    type: quickwit-quickwit-datasource
    access: proxy
    url: http://quickwit-searcher:7280/api/v1
    jsonData:
      index: erp-logs

  - name: Quickwit Traces
    type: quickwit-quickwit-datasource
    access: proxy
    url: http://quickwit-searcher:7280/api/v1
    jsonData:
      index: erp-traces

  - name: Alertmanager
    type: alertmanager
    access: proxy
    url: http://alertmanager:9093
    jsonData:
      implementation: prometheus
```

---

## 8. Frontend (React) Instrumentation

### 8.1 OTel Browser SDK Setup

```typescript
// src/observability/init.ts
import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { ZoneContextManager } from '@opentelemetry/context-zone';
import { registerInstrumentations } from '@opentelemetry/instrumentation';
import { getWebAutoInstrumentations } from '@opentelemetry/auto-instrumentations-web';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';

export function initObservability(moduleName: string, tenantId: string) {
  const resource = new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: `erp-${moduleName}-frontend`,
    [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]:
      import.meta.env.VITE_ENVIRONMENT || 'development',
    'tenant_id': tenantId,
  });

  const exporter = new OTLPTraceExporter({
    url: `${import.meta.env.VITE_OTEL_ENDPOINT || 'http://localhost:4318'}/v1/traces`,
    headers: {
      'X-Scope-OrgID': tenantId,
    },
  });

  const provider = new WebTracerProvider({
    resource,
    spanProcessors: [
      new BatchSpanProcessor(exporter, {
        maxQueueSize: 100,
        maxExportBatchSize: 10,
        scheduledDelayMillis: 5000,
      }),
    ],
  });

  provider.register({
    contextManager: new ZoneContextManager(),
  });

  registerInstrumentations({
    instrumentations: [
      getWebAutoInstrumentations({
        '@opentelemetry/instrumentation-document-load': {},
        '@opentelemetry/instrumentation-fetch': {
          propagateTraceHeaderCorsUrls: [
            new RegExp(`${import.meta.env.VITE_API_URL}`),
          ],
          clearTimingResources: true,
        },
        '@opentelemetry/instrumentation-xml-http-request': {
          propagateTraceHeaderCorsUrls: [
            new RegExp(`${import.meta.env.VITE_API_URL}`),
          ],
        },
        '@opentelemetry/instrumentation-user-interaction': {
          eventNames: ['click', 'submit'],
        },
      }),
    ],
  });

  return provider;
}
```

### 8.2 React Error Boundary with Telemetry

```typescript
// src/observability/ErrorBoundary.tsx
import React from 'react';
import { trace, SpanStatusCode } from '@opentelemetry/api';

interface Props {
  moduleName: string;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ObservabilityErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    const tracer = trace.getTracer(this.props.moduleName);
    const span = tracer.startSpan('react.error_boundary');

    span.setAttributes({
      'error.type': error.name,
      'error.message': error.message,
      'error.stack': error.stack?.substring(0, 4096) || '',
      'react.component_stack': errorInfo.componentStack?.substring(0, 4096) || '',
      'module': this.props.moduleName,
    });
    span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
    span.end();
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div style={{ padding: 24, textAlign: 'center' }}>
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
        </div>
      );
    }
    return this.props.children;
  }
}
```

---

## 9. Integration Testing

### 9.1 Telemetry Validation Test Suite

```go
// tests/integration/telemetry_validation_test.go
package integration

import (
    "testing"
    "time"
    "net/http"
    "encoding/json"
    "fmt"
)

// TestModuleMetricsPresence verifies that a module emits all required metrics
func TestModuleMetricsPresence(t *testing.T) {
    modules := []string{"crm", "iam", "accounting", "inventory", "hrm"}

    requiredMetrics := []string{
        "erp_%s_http_requests_total",
        "erp_%s_http_request_duration_seconds",
        "erp_%s_http_active_requests",
        "erp_%s_db_queries_total",
        "erp_%s_db_query_duration_seconds",
    }

    for _, module := range modules {
        t.Run(module, func(t *testing.T) {
            for _, metricTpl := range requiredMetrics {
                metricName := fmt.Sprintf(metricTpl, module)

                // Query VictoriaMetrics for the metric
                resp, err := http.Get(fmt.Sprintf(
                    "http://vmselect:8481/select/0/prometheus/api/v1/query?query=%s",
                    metricName,
                ))
                if err != nil {
                    t.Fatalf("failed to query VM: %v", err)
                }
                defer resp.Body.Close()

                var result struct {
                    Data struct {
                        Result []interface{} `json:"result"`
                    } `json:"data"`
                }
                json.NewDecoder(resp.Body).Decode(&result)

                if len(result.Data.Result) == 0 {
                    t.Errorf("metric %s not found for module %s", metricName, module)
                }
            }
        })
    }
}

// TestModuleLogAttributes verifies that logs have required attributes
func TestModuleLogAttributes(t *testing.T) {
    requiredAttributes := []string{
        "service.name",
        "tenant_id",
        "severity",
        "timestamp",
    }

    // Search recent logs for each module
    modules := []string{"erp-crm", "erp-iam", "erp-accounting"}

    for _, module := range modules {
        t.Run(module, func(t *testing.T) {
            resp, err := http.Post(
                "http://quickwit-searcher:7280/api/v1/erp-logs/search",
                "application/json",
                strings.NewReader(fmt.Sprintf(
                    `{"query":"service_name:%s","max_hits":1}`, module)),
            )
            if err != nil {
                t.Fatalf("failed to search logs: %v", err)
            }
            defer resp.Body.Close()

            var result struct {
                Hits []map[string]interface{} `json:"hits"`
            }
            json.NewDecoder(resp.Body).Decode(&result)

            if len(result.Hits) == 0 {
                t.Fatalf("no logs found for module %s", module)
            }

            log := result.Hits[0]
            for _, attr := range requiredAttributes {
                if _, exists := log[attr]; !exists {
                    t.Errorf("required attribute %s missing from %s logs", attr, module)
                }
            }
        })
    }
}

// TestTraceCorrelation verifies that traces span multiple modules
func TestTraceCorrelation(t *testing.T) {
    // Make a request that spans CRM -> IAM (auth check) -> DB
    resp, err := http.Get("http://erp-gateway:8090/api/v1/crm/contacts?limit=1")
    if err != nil {
        t.Fatalf("request failed: %v", err)
    }
    traceID := resp.Header.Get("X-Trace-ID")

    // Wait for trace to be indexed
    time.Sleep(10 * time.Second)

    // Verify trace has spans from multiple services
    searchResp, err := http.Post(
        "http://quickwit-searcher:7280/api/v1/erp-traces/search",
        "application/json",
        strings.NewReader(fmt.Sprintf(
            `{"query":"trace_id:%s","max_hits":100}`, traceID)),
    )
    if err != nil {
        t.Fatalf("trace search failed: %v", err)
    }
    defer searchResp.Body.Close()

    var traceResult struct {
        Hits []map[string]interface{} `json:"hits"`
    }
    json.NewDecoder(searchResp.Body).Decode(&traceResult)

    services := make(map[string]bool)
    for _, span := range traceResult.Hits {
        if svc, ok := span["service_name"].(string); ok {
            services[svc] = true
        }
    }

    if len(services) < 2 {
        t.Errorf("expected trace to span multiple services, got: %v", services)
    }
}
```

---

## 10. Troubleshooting Integration Issues

### 10.1 Common Issues and Solutions

| Issue | Symptom | Diagnosis | Solution |
|-------|---------|-----------|----------|
| Missing metrics | Module metrics not in VictoriaMetrics | Check OTel Collector logs for export errors | Verify endpoint URL and network connectivity |
| Missing tenant_id label | Queries return data but without tenant scoping | Check OTel resource attributes | Ensure tenant_id is set in OTel SDK resource |
| No traces for a module | Trace search returns empty | Check sampling configuration | Increase sample rate or check OTLP endpoint |
| Log attributes missing | Logs present but missing fields | Check log formatter configuration | Ensure structured logger with ContextHandler |
| High cardinality | VictoriaMetrics slow queries | Check label values for user IDs or paths | Use route patterns, not actual paths |
| Dashboard empty | Grafana panels show "No data" | Check datasource, tenant filter, time range | Verify datasource proxy headers include tenant_id |
| Alert not firing | Expected alert not appearing | Check vmalert config, query correctness | Test query in Grafana Explore first |
| Cross-module traces broken | Trace shows single service only | Check propagator configuration | Ensure W3C TraceContext propagator is set |

### 10.2 Diagnostic Commands

```bash
# Check if OTel Collector is receiving data from a module
curl -s http://otel-collector:8888/metrics | grep "otelcol_receiver_accepted" | grep "erp-crm"

# Check VictoriaMetrics for a module's metrics
curl -s "http://vmselect:8481/select/0/prometheus/api/v1/label/__name__/values" \
  | jq '.data[] | select(startswith("erp_crm"))'

# Check Quickwit for a module's logs
curl -s -X POST "http://quickwit-searcher:7280/api/v1/erp-logs/search" \
  -H "Content-Type: application/json" \
  -d '{"query":"service_name:erp-crm","max_hits":5}' | jq '.hits[]'

# Check Alertmanager for active alerts from a module
curl -s "http://alertmanager:9093/api/v2/alerts?filter=module%3Dcrm" | jq '.'

# Verify OTel Collector pipeline health
curl -s http://otel-collector:13133/health | jq '.'

# Check Grafana datasource connectivity
curl -s -H "Authorization: Bearer ${GRAFANA_API_KEY}" \
  "http://grafana:3000/api/datasources/proxy/1/api/v1/query?query=up" | jq '.status'
```
