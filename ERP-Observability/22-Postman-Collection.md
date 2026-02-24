# ERP-Observability Postman Collection

> **Document ID:** ERP-OBS-PC-022
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [21-API-Documentation.md](./21-API-Documentation.md)

---

## Overview

This document provides a complete Postman collection for testing the ERP-Observability API. Import the JSON below into Postman, configure the environment variables, and begin testing all endpoints.

---

## Environment Variables

Create a Postman environment with the following variables:

| Variable | Initial Value | Description |
|---|---|---|
| `base_url` | `http://localhost:8090` | Gateway base URL |
| `api_url` | `http://localhost:3000` | Direct API URL (for debugging) |
| `tenant_id` | `dev-tenant` | Tenant identifier |
| `auth_token` | (empty) | JWT bearer token (optional in dev) |
| `alert_rule_id` | (empty) | Populated after creating an alert rule |
| `dashboard_id` | (empty) | Populated after creating a dashboard |
| `trace_id` | (empty) | Set to a known trace ID for testing |

---

## Collection JSON

```json
{
  "info": {
    "name": "ERP-Observability API",
    "description": "Complete API collection for ERP-Observability module v1.0.0",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json",
    "version": "1.0.0"
  },
  "auth": {
    "type": "bearer",
    "bearer": [
      {
        "key": "token",
        "value": "{{auth_token}}",
        "type": "string"
      }
    ]
  },
  "variable": [
    {
      "key": "base_url",
      "value": "http://localhost:8090"
    },
    {
      "key": "tenant_id",
      "value": "dev-tenant"
    }
  ],
  "item": [
    {
      "name": "Gateway",
      "description": "Gateway-level endpoints",
      "item": [
        {
          "name": "Health Check",
          "request": {
            "method": "GET",
            "header": [],
            "url": {
              "raw": "{{base_url}}/healthz",
              "host": ["{{base_url}}"],
              "path": ["healthz"]
            },
            "description": "Gateway health check. Returns module name and healthy status."
          },
          "response": [
            {
              "name": "Healthy",
              "status": "OK",
              "code": 200,
              "body": "{\n  \"status\": \"healthy\",\n  \"module\": \"ERP-Observability\"\n}"
            }
          ]
        },
        {
          "name": "Capabilities",
          "request": {
            "method": "GET",
            "header": [],
            "url": {
              "raw": "{{base_url}}/v1/capabilities",
              "host": ["{{base_url}}"],
              "path": ["v1", "capabilities"]
            },
            "description": "Returns the module's declared capabilities, version, and AIDD governance level."
          },
          "response": [
            {
              "name": "Capabilities Response",
              "status": "OK",
              "code": 200,
              "body": "{\n  \"module\": \"ERP-Observability\",\n  \"version\": \"1.0.0\",\n  \"capabilities\": [\n    \"log_aggregation\",\n    \"metric_collection\",\n    \"distributed_tracing\",\n    \"full_text_search\",\n    \"cross_module_search\",\n    \"alerting\",\n    \"dashboards\",\n    \"audit_trail\",\n    \"slo_monitoring\",\n    \"anomaly_detection\"\n  ],\n  \"integration_mode\": \"platform_service\",\n  \"aidd_governance\": \"ERP_AIDD_STRICT\"\n}"
            }
          ]
        }
      ]
    },
    {
      "name": "Health",
      "item": [
        {
          "name": "Component Health",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/health",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "health"]
            },
            "description": "Returns health status of all observability infrastructure components: Quickwit, Prometheus, Grafana, DragonflyDB, YugabyteDB, OTel Collector, Alertmanager."
          }
        }
      ]
    },
    {
      "name": "Logs",
      "description": "Log querying and streaming endpoints",
      "item": [
        {
          "name": "Query Logs",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/logs?limit=10&sort_order=desc",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "logs"],
              "query": [
                { "key": "limit", "value": "10" },
                { "key": "sort_order", "value": "desc" },
                { "key": "module", "value": "finance", "disabled": true },
                { "key": "level", "value": "error", "disabled": true },
                { "key": "q", "value": "payment failed", "disabled": true },
                { "key": "start", "value": "2026-02-24T00:00:00Z", "disabled": true },
                { "key": "end", "value": "2026-02-24T23:59:59Z", "disabled": true },
                { "key": "offset", "value": "0", "disabled": true }
              ]
            },
            "description": "Query logs from Quickwit erp-logs index. Supports filtering by module, level, time range, and free-text search."
          }
        },
        {
          "name": "Query Logs - By Module and Level",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/logs?module=finance&level=error&limit=20",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "logs"],
              "query": [
                { "key": "module", "value": "finance" },
                { "key": "level", "value": "error" },
                { "key": "limit", "value": "20" }
              ]
            }
          }
        },
        {
          "name": "Query Logs - Full Text Search",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/logs?q=connection+refused&limit=50",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "logs"],
              "query": [
                { "key": "q", "value": "connection refused" },
                { "key": "limit", "value": "50" }
              ]
            }
          }
        },
        {
          "name": "Log Stats",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/logs/stats",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "logs", "stats"],
              "query": [
                { "key": "start", "value": "2026-02-23T00:00:00Z", "disabled": true },
                { "key": "end", "value": "2026-02-24T23:59:59Z", "disabled": true }
              ]
            },
            "description": "Log volume statistics aggregated by level and module."
          }
        }
      ]
    },
    {
      "name": "Metrics",
      "description": "PromQL query proxy endpoints",
      "item": [
        {
          "name": "Instant Query",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/metrics/query?query=up",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "metrics", "query"],
              "query": [
                { "key": "query", "value": "up" },
                { "key": "time", "value": "", "disabled": true },
                { "key": "timeout", "value": "30s", "disabled": true }
              ]
            },
            "description": "Instant PromQL query. Returns the current value of the expression."
          }
        },
        {
          "name": "Range Query",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/metrics/range?query=rate(http_requests_total[5m])&start=2026-02-24T09:00:00Z&end=2026-02-24T10:00:00Z&step=60s",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "metrics", "range"],
              "query": [
                { "key": "query", "value": "rate(http_requests_total[5m])" },
                { "key": "start", "value": "2026-02-24T09:00:00Z" },
                { "key": "end", "value": "2026-02-24T10:00:00Z" },
                { "key": "step", "value": "60s" },
                { "key": "timeout", "value": "30s", "disabled": true }
              ]
            },
            "description": "Range PromQL query for time-series data over a time window."
          }
        },
        {
          "name": "Module Summary",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/metrics/modules",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "metrics", "modules"]
            },
            "description": "Per-module metric summary including request rate, error rate, and latency."
          }
        }
      ]
    },
    {
      "name": "Traces",
      "description": "Distributed tracing endpoints",
      "item": [
        {
          "name": "Query Traces",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/traces?limit=10",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "traces"],
              "query": [
                { "key": "limit", "value": "10" },
                { "key": "module", "value": "finance", "disabled": true },
                { "key": "service", "value": "", "disabled": true },
                { "key": "min_duration_ms", "value": "1000", "disabled": true },
                { "key": "max_duration_ms", "value": "5000", "disabled": true },
                { "key": "has_errors", "value": "true", "disabled": true },
                { "key": "start", "value": "2026-02-24T00:00:00Z", "disabled": true },
                { "key": "end", "value": "2026-02-24T23:59:59Z", "disabled": true }
              ]
            }
          }
        },
        {
          "name": "Get Trace by ID",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/traces/{{trace_id}}",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "traces", "{{trace_id}}"]
            },
            "description": "Get complete trace with all spans, attributes, and events."
          }
        },
        {
          "name": "Trace Timeline",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/traces/{{trace_id}}/timeline",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "traces", "{{trace_id}}", "timeline"]
            },
            "description": "Flattened timeline view suitable for waterfall visualization."
          }
        }
      ]
    },
    {
      "name": "Alerts",
      "description": "Alert management endpoints",
      "item": [
        {
          "name": "List Active Alerts",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/alerts",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "alerts"]
            }
          }
        },
        {
          "name": "List Alert Rules",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/alerts/rules",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "alerts", "rules"],
              "query": [
                { "key": "module", "value": "finance", "disabled": true },
                { "key": "enabled", "value": "true", "disabled": true }
              ]
            }
          }
        },
        {
          "name": "Create Alert Rule",
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "if (pm.response.code === 201) {",
                  "    var jsonData = pm.response.json();",
                  "    pm.collectionVariables.set('alert_rule_id', jsonData.id);",
                  "}"
                ]
              }
            }
          ],
          "request": {
            "method": "POST",
            "header": [
              { "key": "Content-Type", "value": "application/json" },
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Test High Error Rate\",\n  \"description\": \"Alert when error rate exceeds 5%\",\n  \"severity\": \"warning\",\n  \"module\": \"finance\",\n  \"promql_expression\": \"rate(http_requests_total{module=\\\"finance\\\",status=~\\\"5..\\\"}[5m]) / rate(http_requests_total{module=\\\"finance\\\"}[5m]) > 0.05\",\n  \"duration\": \"5m\",\n  \"labels\": { \"team\": \"finance-eng\" },\n  \"annotations\": {\n    \"summary\": \"High error rate for finance module\",\n    \"runbook_url\": \"https://runbooks.erp.io/finance-error-rate\"\n  }\n}"
            },
            "url": {
              "raw": "{{base_url}}/v1/observability/alerts/rules",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "alerts", "rules"]
            },
            "description": "Create a new alert rule. Requires admin role. Auto-saves the created rule ID to collection variable."
          }
        },
        {
          "name": "Update Alert Rule",
          "request": {
            "method": "PUT",
            "header": [
              { "key": "Content-Type", "value": "application/json" },
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"severity\": \"critical\",\n  \"enabled\": true\n}"
            },
            "url": {
              "raw": "{{base_url}}/v1/observability/alerts/rules/{{alert_rule_id}}",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "alerts", "rules", "{{alert_rule_id}}"]
            }
          }
        },
        {
          "name": "Silence Alert",
          "request": {
            "method": "POST",
            "header": [
              { "key": "Content-Type", "value": "application/json" },
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"reason\": \"Known issue, fix in progress\",\n  \"duration_hours\": 4\n}"
            },
            "url": {
              "raw": "{{base_url}}/v1/observability/alerts/{{alert_rule_id}}/silence",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "alerts", "{{alert_rule_id}}", "silence"]
            }
          }
        },
        {
          "name": "Delete Alert Rule",
          "request": {
            "method": "DELETE",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/alerts/rules/{{alert_rule_id}}",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "alerts", "rules", "{{alert_rule_id}}"]
            }
          }
        }
      ]
    },
    {
      "name": "Dashboards",
      "description": "Dashboard CRUD endpoints",
      "item": [
        {
          "name": "List Dashboards",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/dashboards",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "dashboards"]
            }
          }
        },
        {
          "name": "Create Dashboard",
          "event": [
            {
              "listen": "test",
              "script": {
                "exec": [
                  "if (pm.response.code === 201) {",
                  "    var jsonData = pm.response.json();",
                  "    pm.collectionVariables.set('dashboard_id', jsonData.id);",
                  "}"
                ]
              }
            }
          ],
          "request": {
            "method": "POST",
            "header": [
              { "key": "Content-Type", "value": "application/json" },
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Test Dashboard\",\n  \"description\": \"Testing dashboard creation via Postman\",\n  \"panels\": [\n    {\n      \"id\": \"panel-1\",\n      \"title\": \"Request Rate\",\n      \"type\": \"timeseries\",\n      \"query\": \"rate(http_requests_total{module=\\\"finance\\\"}[5m])\",\n      \"datasource\": \"prometheus\",\n      \"position\": { \"x\": 0, \"y\": 0, \"w\": 12, \"h\": 8 }\n    },\n    {\n      \"id\": \"panel-2\",\n      \"title\": \"Error Count\",\n      \"type\": \"stat\",\n      \"query\": \"sum(http_requests_total{module=\\\"finance\\\",status=~\\\"5..\\\"})\",\n      \"datasource\": \"prometheus\",\n      \"position\": { \"x\": 12, \"y\": 0, \"w\": 6, \"h\": 4 }\n    }\n  ],\n  \"tags\": [\"finance\", \"test\"]\n}"
            },
            "url": {
              "raw": "{{base_url}}/v1/observability/dashboards",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "dashboards"]
            }
          }
        },
        {
          "name": "Get Dashboard",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/dashboards/{{dashboard_id}}",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "dashboards", "{{dashboard_id}}"]
            }
          }
        },
        {
          "name": "Update Dashboard",
          "request": {
            "method": "PUT",
            "header": [
              { "key": "Content-Type", "value": "application/json" },
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Updated Test Dashboard\",\n  \"tags\": [\"finance\", \"test\", \"updated\"]\n}"
            },
            "url": {
              "raw": "{{base_url}}/v1/observability/dashboards/{{dashboard_id}}",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "dashboards", "{{dashboard_id}}"]
            }
          }
        }
      ]
    },
    {
      "name": "Search",
      "description": "Cross-module search endpoints",
      "item": [
        {
          "name": "Search",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/search?q=invoice&limit=20",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "search"],
              "query": [
                { "key": "q", "value": "invoice" },
                { "key": "limit", "value": "20" },
                { "key": "modules", "value": "finance,commerce", "disabled": true },
                { "key": "types", "value": "invoice,order", "disabled": true },
                { "key": "sort_by", "value": "relevance", "disabled": true },
                { "key": "offset", "value": "0", "disabled": true }
              ]
            }
          }
        },
        {
          "name": "Search Suggestions",
          "request": {
            "method": "GET",
            "header": [
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "url": {
              "raw": "{{base_url}}/v1/observability/search/suggest?q=inv&limit=5",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "search", "suggest"],
              "query": [
                { "key": "q", "value": "inv" },
                { "key": "limit", "value": "5" }
              ]
            }
          }
        },
        {
          "name": "Trigger Reindex",
          "request": {
            "method": "POST",
            "header": [
              { "key": "Content-Type", "value": "application/json" },
              { "key": "X-Tenant-ID", "value": "{{tenant_id}}" }
            ],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"module\": \"finance\",\n  \"entity_type\": \"invoice\"\n}"
            },
            "url": {
              "raw": "{{base_url}}/v1/observability/search/reindex",
              "host": ["{{base_url}}"],
              "path": ["v1", "observability", "search", "reindex"]
            },
            "description": "Trigger reindex for a module/entity type. Requires admin role."
          }
        }
      ]
    }
  ]
}
```

---

## Import Instructions

### Postman Desktop

1. Open Postman.
2. Click **Import** in the top-left.
3. Select **Raw Text** and paste the JSON above.
4. Click **Import**.
5. Create an environment with the variables listed above.
6. Select the environment from the environment dropdown.

### Postman CLI (Newman)

```bash
# Save the collection to a file
# (copy the JSON above to erp-observability.postman_collection.json)

# Run all requests
newman run erp-observability.postman_collection.json \
  --env-var "base_url=http://localhost:8090" \
  --env-var "tenant_id=dev-tenant"

# Run a specific folder
newman run erp-observability.postman_collection.json \
  --folder "Logs" \
  --env-var "base_url=http://localhost:8090" \
  --env-var "tenant_id=dev-tenant"
```

---

## Request Flow Examples

### Example 1: Full Alert Rule Lifecycle

1. **Create** alert rule (POST `/v1/observability/alerts/rules`) -- saves `alert_rule_id`
2. **List** alert rules (GET `/v1/observability/alerts/rules`) -- verify creation
3. **Update** alert rule (PUT `/v1/observability/alerts/rules/:id`) -- change severity
4. **Silence** alert (POST `/v1/observability/alerts/:id/silence`) -- create silence
5. **Delete** alert rule (DELETE `/v1/observability/alerts/rules/:id`) -- cleanup

### Example 2: Log Investigation Workflow

1. **Log Stats** (GET `/v1/observability/logs/stats`) -- identify error spike
2. **Query Logs** by error level (GET `/v1/observability/logs?level=error`)
3. **Search** for specific error message (GET `/v1/observability/search?q=connection refused`)
4. **Get Trace** linked from log entry (GET `/v1/observability/traces/:traceId`)
5. **View Timeline** for root cause (GET `/v1/observability/traces/:traceId/timeline`)

### Example 3: Dashboard Workflow

1. **Create** dashboard (POST `/v1/observability/dashboards`) -- saves `dashboard_id`
2. **Get** dashboard (GET `/v1/observability/dashboards/:id`) -- verify panels
3. **Update** dashboard (PUT `/v1/observability/dashboards/:id`) -- add panels
4. **List** dashboards (GET `/v1/observability/dashboards`) -- browse all

---

## cURL Equivalents

For quick testing without Postman:

```bash
# Health
curl -s http://localhost:8090/healthz | jq .

# Logs
curl -s "http://localhost:8090/v1/observability/logs?limit=5" \
  -H "X-Tenant-ID: dev-tenant" | jq .

# Metrics
curl -s "http://localhost:8090/v1/observability/metrics/query?query=up" \
  -H "X-Tenant-ID: dev-tenant" | jq .

# Search
curl -s "http://localhost:8090/v1/observability/search?q=test&limit=10" \
  -H "X-Tenant-ID: dev-tenant" | jq .

# Create Alert Rule
curl -X POST http://localhost:8090/v1/observability/alerts/rules \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: dev-tenant" \
  -d '{
    "name": "Test Alert",
    "severity": "warning",
    "module": "finance",
    "promql_expression": "up{module=\"finance\"} == 0",
    "duration": "5m"
  }' | jq .

# Create Dashboard
curl -X POST http://localhost:8090/v1/observability/dashboards \
  -H "Content-Type: application/json" \
  -H "X-Tenant-ID: dev-tenant" \
  -d '{
    "name": "Quick Test",
    "panels": [],
    "tags": ["test"]
  }' | jq .
```
