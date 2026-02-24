# ERP-AIOps API Documentation

> **Document ID:** ERP-AIOPS-API-021
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [17-README.md](./17-README.md), [22-Postman-Collection.md](./22-Postman-Collection.md), [23-Webhook-Specifications.md](./23-Webhook-Specifications.md)

---

## Overview

ERP-AIOps exposes two API surfaces:

1. **Rust API** (port 8080): Core platform API handling incidents, anomalies, rules, topology, remediation, cost, and security.
2. **AI Brain API** (port 8081): Python FastAPI service providing ML inference for anomaly detection, root cause analysis, forecasting, and event correlation.

All external traffic is routed through the **Go Gateway** (port 3000), which handles authentication, rate limiting, and routing.

### Base URLs

| Environment | Gateway URL | Rust API (internal) | AI Brain (internal) |
|-------------|-------------|---------------------|---------------------|
| Development | `http://localhost:3000` | `http://localhost:8080` | `http://localhost:8081` |
| Staging | `https://aiops-staging.erp.internal` | Internal only | Internal only |
| Production | `https://aiops.erp.internal` | Internal only | Internal only |

### Authentication

All API requests must include a valid JWT token in the `Authorization` header:

```
Authorization: Bearer <jwt-token>
```

Tokens are issued by Authentik and contain tenant and role claims:

```json
{
  "sub": "user-uuid",
  "tenant_id": "tenant-001",
  "roles": ["aiops_operator"],
  "permissions": ["incidents:read", "incidents:write", "anomalies:read"],
  "exp": 1708905600
}
```

### Common Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer <jwt-token>` |
| `X-Tenant-ID` | Yes | Tenant identifier (must match JWT `tenant_id` claim) |
| `Content-Type` | Yes (for POST/PUT) | `application/json` |
| `X-Request-ID` | No | Client-generated request ID for tracing |
| `Accept` | No | `application/json` (default) |

### Standard Response Envelope

**Success Response:**
```json
{
  "data": { ... },
  "meta": {
    "request_id": "req-abc123",
    "timestamp": "2026-02-24T10:30:00.000Z"
  }
}
```

**Paginated Response:**
```json
{
  "items": [ ... ],
  "total": 150,
  "page": 1,
  "per_page": 20,
  "meta": {
    "request_id": "req-abc123",
    "timestamp": "2026-02-24T10:30:00.000Z"
  }
}
```

**Error Response (RFC 7807):**
```json
{
  "type": "https://aiops.erp.internal/errors/not-found",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Incident with ID 'inc-999' not found for tenant 'tenant-001'",
  "instance": "/api/v1/incidents/inc-999",
  "request_id": "req-abc123"
}
```

### Standard Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number |
| `per_page` | integer | 20 | Items per page (max 100) |
| `sort_by` | string | `created_at` | Sort field |
| `sort_order` | string | `desc` | Sort direction (`asc` or `desc`) |
| `search` | string | _(empty)_ | Full-text search query |

---

## Rust API Routes

### 1. Incidents API (`/api/v1/incidents`)

#### List Incidents

```
GET /api/v1/incidents
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by status: `open`, `investigating`, `identified`, `monitoring`, `resolved`, `closed` |
| `severity` | string | Filter by severity: `critical`, `high`, `medium`, `low`, `info` |
| `source` | string | Filter by source (e.g., `prometheus`, `datadog`, `manual`) |
| `assigned_to` | string | Filter by assigned user ID |
| `tag` | string | Filter by tag (can be repeated for multiple tags) |
| `created_after` | datetime | Filter incidents created after this timestamp |
| `created_before` | datetime | Filter incidents created before this timestamp |

**Response:**
```json
{
  "items": [
    {
      "id": "inc-a1b2c3d4",
      "tenant_id": "tenant-001",
      "title": "High CPU utilization on production API servers",
      "description": "CPU usage exceeded 90% on api-server-01...",
      "severity": "high",
      "status": "investigating",
      "source": "prometheus",
      "assigned_to": "user-uuid-123",
      "tags": ["cpu", "production", "api-servers"],
      "correlation_group_id": "cg-xyz789",
      "remediation_status": "pending",
      "created_at": "2026-02-24T10:15:00.000Z",
      "updated_at": "2026-02-24T10:20:00.000Z",
      "resolved_at": null
    }
  ],
  "total": 45,
  "page": 1,
  "per_page": 20
}
```

#### Get Incident by ID

```
GET /api/v1/incidents/{id}
```

**Response:**
```json
{
  "data": {
    "id": "inc-a1b2c3d4",
    "tenant_id": "tenant-001",
    "title": "High CPU utilization on production API servers",
    "description": "CPU usage exceeded 90% on api-server-01, api-server-02, and api-server-03 for the past 15 minutes. No recent deployments. Load balancer shows even traffic distribution.",
    "severity": "high",
    "status": "investigating",
    "source": "prometheus",
    "assigned_to": "user-uuid-123",
    "tags": ["cpu", "production", "api-servers"],
    "correlation_group_id": "cg-xyz789",
    "remediation_status": "pending",
    "rca_hypotheses": [
      {
        "hypothesis": "Memory leak in API server causing increased GC pressure and CPU usage",
        "confidence": 0.82,
        "evidence": ["memory_usage_trend_increasing", "gc_pause_duration_anomaly"]
      }
    ],
    "timeline": [
      {
        "timestamp": "2026-02-24T10:15:00.000Z",
        "event": "incident_created",
        "actor": "prometheus-alertmanager"
      },
      {
        "timestamp": "2026-02-24T10:16:30.000Z",
        "event": "assigned",
        "actor": "auto-assign-engine",
        "details": "Assigned to on-call engineer"
      }
    ],
    "created_at": "2026-02-24T10:15:00.000Z",
    "updated_at": "2026-02-24T10:20:00.000Z",
    "resolved_at": null
  }
}
```

#### Create Incident

```
POST /api/v1/incidents
```

**Request Body:**
```json
{
  "title": "High CPU utilization on production API servers",
  "description": "CPU usage exceeded 90% on api-server-01, api-server-02, and api-server-03.",
  "severity": "high",
  "source": "prometheus",
  "assigned_to": "user-uuid-123",
  "tags": ["cpu", "production", "api-servers"],
  "metadata": {
    "alert_rule": "HighCpuUsage",
    "threshold": 90,
    "affected_hosts": ["api-server-01", "api-server-02", "api-server-03"]
  }
}
```

**Required Fields:** `title`, `severity`

**Response:** `201 Created` with the created incident object.

#### Update Incident

```
PUT /api/v1/incidents/{id}
```

**Request Body (partial update supported):**
```json
{
  "status": "identified",
  "severity": "critical",
  "description": "Updated: Root cause identified as memory leak in auth middleware.",
  "assigned_to": "user-uuid-456"
}
```

**Response:** `200 OK` with the updated incident object.

#### Resolve Incident

```
POST /api/v1/incidents/{id}/resolve
```

**Request Body:**
```json
{
  "resolution_notes": "Memory leak patched in auth middleware. Deployed fix in v2.3.1.",
  "root_cause": "Memory leak in JWT token parsing cache",
  "resolution_type": "code_fix"
}
```

**Response:** `200 OK` with the resolved incident object (status set to `resolved`, `resolved_at` populated).

#### Request RCA for Incident

```
POST /api/v1/incidents/{id}/rca
```

**Request Body:**
```json
{
  "include_topology": true,
  "include_recent_changes": true,
  "include_similar_incidents": true,
  "max_hypotheses": 5,
  "llm_provider": "auto"
}
```

**Response:**
```json
{
  "data": {
    "incident_id": "inc-a1b2c3d4",
    "analysis_id": "rca-e5f6g7",
    "status": "completed",
    "hypotheses": [
      {
        "rank": 1,
        "hypothesis": "Memory leak in JWT token parsing cache caused by unbounded HashMap growth",
        "confidence": 0.87,
        "reasoning": "1. CPU increase correlates with memory growth trend...",
        "evidence": [
          {
            "type": "metric_anomaly",
            "description": "heap_memory_bytes increasing linearly over 72 hours",
            "relevance": 0.95
          },
          {
            "type": "log_pattern",
            "description": "GC pressure warnings in api-server logs starting 2026-02-21",
            "relevance": 0.88
          }
        ],
        "suggested_remediation": "Implement LRU cache with TTL for JWT token cache"
      }
    ],
    "context_summary": {
      "related_anomalies": 3,
      "topology_services_analyzed": 8,
      "recent_changes": 2,
      "similar_historical_incidents": 1
    },
    "completed_at": "2026-02-24T10:25:00.000Z"
  }
}
```

#### Delete Incident

```
DELETE /api/v1/incidents/{id}
```

**Response:** `204 No Content`

---

### 2. Anomalies API (`/api/v1/anomalies`)

#### List Anomalies

```
GET /api/v1/anomalies
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | `active`, `acknowledged`, `resolved`, `suppressed` |
| `severity` | string | `critical`, `high`, `medium`, `low` |
| `metric_name` | string | Filter by metric name |
| `service` | string | Filter by service name |
| `min_score` | float | Minimum anomaly score (0.0-1.0) |
| `detection_method` | string | `adaptive`, `isolation_forest`, `lstm`, `ensemble` |

**Response:**
```json
{
  "items": [
    {
      "id": "anom-x1y2z3",
      "tenant_id": "tenant-001",
      "metric_name": "api_response_time_ms",
      "service": "api-server",
      "status": "active",
      "severity": "high",
      "score": 0.94,
      "detection_method": "adaptive",
      "baseline_value": 120.5,
      "anomalous_value": 485.2,
      "deviation_factor": 3.03,
      "started_at": "2026-02-24T10:04:00.000Z",
      "ended_at": null,
      "incident_id": "inc-a1b2c3d4",
      "created_at": "2026-02-24T10:04:30.000Z"
    }
  ],
  "total": 12,
  "page": 1,
  "per_page": 20
}
```

#### Detect Anomalies (Synchronous)

```
POST /api/v1/anomalies/detect
```

**Request Body:**
```json
{
  "metric_name": "api_response_time_ms",
  "service": "api-server",
  "values": [120, 125, 118, 122, 450, 480, 520, 495, 510],
  "timestamps": [
    "2026-02-24T10:00:00Z",
    "2026-02-24T10:01:00Z",
    "2026-02-24T10:02:00Z",
    "2026-02-24T10:03:00Z",
    "2026-02-24T10:04:00Z",
    "2026-02-24T10:05:00Z",
    "2026-02-24T10:06:00Z",
    "2026-02-24T10:07:00Z",
    "2026-02-24T10:08:00Z"
  ],
  "detection_method": "adaptive",
  "sensitivity": 0.95,
  "create_anomaly_record": true
}
```

**Response:**
```json
{
  "data": {
    "anomalies_detected": 5,
    "detection_method": "adaptive",
    "results": [
      {
        "index": 4,
        "timestamp": "2026-02-24T10:04:00Z",
        "value": 450.0,
        "baseline": 121.25,
        "upper_bound": 135.5,
        "lower_bound": 107.0,
        "score": 0.96,
        "is_anomaly": true
      }
    ],
    "baseline_stats": {
      "mean": 121.25,
      "std_dev": 2.99,
      "seasonal_component": 0.0
    },
    "anomaly_id": "anom-x1y2z3"
  }
}
```

#### Acknowledge Anomaly

```
POST /api/v1/anomalies/{id}/acknowledge
```

**Request Body:**
```json
{
  "reason": "Known maintenance window",
  "suppress_until": "2026-02-24T12:00:00Z"
}
```

#### Get Anomaly by ID

```
GET /api/v1/anomalies/{id}
```

#### Resolve Anomaly

```
POST /api/v1/anomalies/{id}/resolve
```

---

### 3. Rules API (`/api/v1/rules`)

#### List Rules

```
GET /api/v1/rules
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `enabled` | boolean | Filter by enabled/disabled status |
| `type` | string | `threshold`, `anomaly`, `pattern`, `composite` |
| `category` | string | `incident`, `anomaly`, `remediation`, `notification` |

**Response:**
```json
{
  "items": [
    {
      "id": "rule-abc123",
      "tenant_id": "tenant-001",
      "name": "High Error Rate Alert",
      "description": "Trigger incident when error rate exceeds adaptive threshold",
      "type": "anomaly",
      "category": "incident",
      "enabled": true,
      "condition": {
        "metric": "http_errors_5xx_rate",
        "detection_method": "adaptive",
        "sensitivity": 0.9,
        "min_duration_seconds": 300,
        "min_score": 0.8
      },
      "actions": [
        {
          "type": "create_incident",
          "params": {
            "severity": "high",
            "title_template": "High error rate on {{service}}: {{value}}%",
            "tags": ["error-rate", "auto-generated"]
          }
        },
        {
          "type": "notify",
          "params": {
            "channel": "slack",
            "target": "#oncall-alerts"
          }
        }
      ],
      "cooldown_seconds": 3600,
      "last_triggered_at": "2026-02-24T09:00:00.000Z",
      "created_at": "2026-02-01T00:00:00.000Z",
      "updated_at": "2026-02-20T10:00:00.000Z"
    }
  ],
  "total": 15,
  "page": 1,
  "per_page": 20
}
```

#### Create Rule

```
POST /api/v1/rules
```

**Request Body:**
```json
{
  "name": "High Error Rate Alert",
  "description": "Trigger incident when error rate exceeds adaptive threshold",
  "type": "anomaly",
  "category": "incident",
  "enabled": true,
  "condition": {
    "metric": "http_errors_5xx_rate",
    "detection_method": "adaptive",
    "sensitivity": 0.9,
    "min_duration_seconds": 300,
    "min_score": 0.8
  },
  "actions": [
    {
      "type": "create_incident",
      "params": {
        "severity": "high",
        "title_template": "High error rate on {{service}}: {{value}}%",
        "tags": ["error-rate", "auto-generated"]
      }
    }
  ],
  "cooldown_seconds": 3600
}
```

#### Update Rule

```
PUT /api/v1/rules/{id}
```

#### Delete Rule

```
DELETE /api/v1/rules/{id}
```

#### Enable/Disable Rule

```
POST /api/v1/rules/{id}/enable
POST /api/v1/rules/{id}/disable
```

---

### 4. Topology API (`/api/v1/topology`)

#### List Topology Nodes

```
GET /api/v1/topology/nodes
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `type` | string | `service`, `database`, `cache`, `queue`, `external` |
| `status` | string | `healthy`, `degraded`, `unhealthy`, `unknown` |
| `tag` | string | Filter by tag |

**Response:**
```json
{
  "items": [
    {
      "id": "node-svc-api",
      "tenant_id": "tenant-001",
      "name": "api-server",
      "type": "service",
      "status": "healthy",
      "metadata": {
        "version": "2.3.0",
        "replicas": 3,
        "namespace": "production",
        "language": "go"
      },
      "health_score": 0.98,
      "last_seen_at": "2026-02-24T10:29:55.000Z",
      "dependencies": ["node-db-primary", "node-cache-redis", "node-svc-auth"],
      "dependents": ["node-svc-frontend", "node-svc-mobile-api"],
      "created_at": "2026-01-15T00:00:00.000Z",
      "updated_at": "2026-02-24T10:29:55.000Z"
    }
  ],
  "total": 42,
  "page": 1,
  "per_page": 50
}
```

#### Get Topology Edges

```
GET /api/v1/topology/edges
```

**Response:**
```json
{
  "items": [
    {
      "id": "edge-api-to-db",
      "source_node_id": "node-svc-api",
      "target_node_id": "node-db-primary",
      "relationship": "depends_on",
      "protocol": "postgresql",
      "metrics": {
        "avg_latency_ms": 4.2,
        "p99_latency_ms": 12.8,
        "requests_per_second": 1250,
        "error_rate": 0.001
      },
      "last_observed_at": "2026-02-24T10:29:55.000Z"
    }
  ]
}
```

#### Impact Analysis

```
GET /api/v1/topology/impact/{node_id}
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `direction` | string | `upstream`, `downstream`, `both` (default: `downstream`) |
| `max_depth` | integer | Maximum traversal depth (default: 5) |

**Response:**
```json
{
  "data": {
    "source_node": "node-db-primary",
    "direction": "downstream",
    "max_depth": 5,
    "impacted_nodes": [
      {
        "node_id": "node-svc-api",
        "name": "api-server",
        "depth": 1,
        "impact_probability": 0.95,
        "impact_type": "direct_dependency"
      },
      {
        "node_id": "node-svc-frontend",
        "name": "frontend",
        "depth": 2,
        "impact_probability": 0.85,
        "impact_type": "transitive_dependency"
      }
    ],
    "total_impacted": 8,
    "blast_radius_score": 0.72
  }
}
```

#### Add/Update Topology Node

```
POST /api/v1/topology/nodes
PUT /api/v1/topology/nodes/{id}
```

#### Add/Remove Edge

```
POST /api/v1/topology/edges
DELETE /api/v1/topology/edges/{id}
```

---

### 5. Remediation API (`/api/v1/remediation`)

#### List Remediation Actions

```
GET /api/v1/remediation/actions
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | `pending`, `approved`, `executing`, `completed`, `failed`, `rolled_back` |
| `incident_id` | string | Filter by incident |
| `action_type` | string | `restart`, `scale`, `traffic_shift`, `cache_flush`, `dns_failover`, `rollback` |

**Response:**
```json
{
  "items": [
    {
      "id": "rem-abc123",
      "tenant_id": "tenant-001",
      "incident_id": "inc-a1b2c3d4",
      "action_type": "restart",
      "target_service": "api-server",
      "status": "completed",
      "dry_run": false,
      "approval_required": false,
      "approved_by": null,
      "parameters": {
        "strategy": "rolling",
        "max_unavailable": 1,
        "timeout_seconds": 300
      },
      "result": {
        "success": true,
        "message": "Successfully restarted 3/3 replicas",
        "duration_seconds": 45,
        "before_state": {"replicas": 3, "ready": 1},
        "after_state": {"replicas": 3, "ready": 3}
      },
      "executed_at": "2026-02-24T10:22:00.000Z",
      "completed_at": "2026-02-24T10:22:45.000Z",
      "created_at": "2026-02-24T10:21:00.000Z"
    }
  ],
  "total": 8,
  "page": 1,
  "per_page": 20
}
```

#### Execute Remediation Action

```
POST /api/v1/remediation/execute
```

**Request Body:**
```json
{
  "incident_id": "inc-a1b2c3d4",
  "action_type": "restart",
  "target_service": "api-server",
  "dry_run": false,
  "parameters": {
    "strategy": "rolling",
    "max_unavailable": 1,
    "timeout_seconds": 300
  }
}
```

**Response:**
```json
{
  "data": {
    "action_id": "rem-abc123",
    "status": "executing",
    "message": "Remediation action initiated. Track progress via GET /api/v1/remediation/actions/rem-abc123"
  }
}
```

#### List Remediation Policies

```
GET /api/v1/remediation/policies
```

#### Create Remediation Policy

```
POST /api/v1/remediation/policies
```

**Request Body:**
```json
{
  "name": "Auto-restart on crash loop",
  "description": "Automatically restart service when crash loop is detected",
  "enabled": true,
  "trigger": {
    "type": "incident",
    "conditions": {
      "source": "kubernetes",
      "tags_include": ["CrashLoopBackOff"],
      "severity_min": "medium"
    }
  },
  "action": {
    "type": "restart",
    "parameters": {
      "strategy": "rolling",
      "max_unavailable": 1
    }
  },
  "approval_required": false,
  "cooldown_seconds": 1800,
  "max_executions_per_hour": 3
}
```

#### Approve Remediation Action

```
POST /api/v1/remediation/actions/{id}/approve
```

#### Rollback Remediation Action

```
POST /api/v1/remediation/actions/{id}/rollback
```

---

### 6. Cost API (`/api/v1/cost`)

#### List Cost Reports

```
GET /api/v1/cost/reports
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `period` | string | `daily`, `weekly`, `monthly` |
| `start_date` | date | Start of reporting period |
| `end_date` | date | End of reporting period |
| `group_by` | string | `service`, `department`, `environment`, `tag` |

**Response:**
```json
{
  "items": [
    {
      "id": "cost-rpt-202602",
      "tenant_id": "tenant-001",
      "period": "monthly",
      "start_date": "2026-02-01",
      "end_date": "2026-02-28",
      "total_cost": 45230.50,
      "currency": "USD",
      "breakdown": [
        {"category": "compute", "cost": 28500.00, "percentage": 63.0},
        {"category": "storage", "cost": 8200.50, "percentage": 18.1},
        {"category": "network", "cost": 5100.00, "percentage": 11.3},
        {"category": "database", "cost": 3430.00, "percentage": 7.6}
      ],
      "cost_trend": "increasing",
      "trend_percentage": 8.5,
      "anomalies_detected": 2,
      "created_at": "2026-02-24T00:00:00.000Z"
    }
  ]
}
```

#### Get Cost Recommendations

```
GET /api/v1/cost/recommendations
```

**Response:**
```json
{
  "items": [
    {
      "id": "rec-001",
      "tenant_id": "tenant-001",
      "type": "rightsizing",
      "title": "Downsize api-server instances",
      "description": "api-server instances are utilizing only 35% of allocated CPU. Recommended to downsize from c5.2xlarge to c5.xlarge.",
      "estimated_monthly_savings": 1250.00,
      "confidence": 0.92,
      "impact": "low",
      "status": "pending",
      "resource": {
        "type": "compute",
        "name": "api-server",
        "current_spec": "c5.2xlarge",
        "recommended_spec": "c5.xlarge"
      },
      "evidence": {
        "avg_cpu_utilization": 0.35,
        "peak_cpu_utilization": 0.62,
        "observation_period_days": 30
      },
      "created_at": "2026-02-24T00:00:00.000Z"
    }
  ]
}
```

#### Get Cost Forecast

```
GET /api/v1/cost/forecast
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `horizon_days` | integer | Forecast horizon: 30, 60, or 90 |
| `group_by` | string | Optional grouping dimension |

---

### 7. Security API (`/api/v1/security`)

#### List Security Findings

```
GET /api/v1/security/findings
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `severity` | string | `critical`, `high`, `medium`, `low`, `informational` |
| `status` | string | `open`, `acknowledged`, `remediated`, `false_positive`, `risk_accepted` |
| `category` | string | `vulnerability`, `misconfiguration`, `compliance`, `access_control` |
| `scan_id` | string | Filter by scan ID |

**Response:**
```json
{
  "items": [
    {
      "id": "finding-sec001",
      "tenant_id": "tenant-001",
      "scan_id": "scan-20260224",
      "title": "Container running as root",
      "description": "The api-server container is running as root user, which violates the principle of least privilege.",
      "severity": "high",
      "status": "open",
      "category": "misconfiguration",
      "resource": {
        "type": "container",
        "name": "api-server",
        "namespace": "production"
      },
      "compliance": {
        "frameworks": ["CIS-Kubernetes-1.8", "SOC2-CC6.1"],
        "controls": ["5.2.6"]
      },
      "remediation_guidance": "Set securityContext.runAsNonRoot: true in the Pod spec.",
      "cve_id": null,
      "cvss_score": null,
      "first_detected_at": "2026-02-20T00:00:00.000Z",
      "created_at": "2026-02-24T06:00:00.000Z"
    }
  ],
  "total": 23,
  "page": 1,
  "per_page": 20
}
```

#### Trigger Security Scan

```
POST /api/v1/security/scans
```

**Request Body:**
```json
{
  "scan_type": "full",
  "targets": ["infrastructure", "containers", "dependencies", "access_control"],
  "compliance_frameworks": ["SOC2", "ISO27001"]
}
```

#### Get Security Posture Score

```
GET /api/v1/security/posture
```

**Response:**
```json
{
  "data": {
    "tenant_id": "tenant-001",
    "overall_score": 78,
    "grade": "B",
    "category_scores": {
      "vulnerability_management": 85,
      "configuration_security": 72,
      "access_control": 80,
      "compliance": 75,
      "data_protection": 78
    },
    "findings_summary": {
      "critical": 0,
      "high": 3,
      "medium": 8,
      "low": 12,
      "informational": 15
    },
    "trend": "improving",
    "trend_delta": 5,
    "last_scan_at": "2026-02-24T06:00:00.000Z"
  }
}
```

#### Update Finding Status

```
PUT /api/v1/security/findings/{id}
```

#### Get Scan Results

```
GET /api/v1/security/scans/{scan_id}
```

---

## AI Brain API Routes

The AI Brain is a Python FastAPI service providing ML inference. These endpoints are typically called by the Rust API internally, but can also be accessed directly for testing.

### 1. Anomaly Analysis

```
POST /analyze/anomaly
```

**Request Body:**
```json
{
  "metric_name": "api_response_time_ms",
  "values": [120, 125, 118, 122, 450, 480, 520],
  "timestamps": ["2026-02-24T10:00:00Z", "..."],
  "method": "isolation_forest",
  "sensitivity": 0.95,
  "tenant_id": "tenant-001"
}
```

**Response:**
```json
{
  "anomalies": [
    {
      "index": 4,
      "timestamp": "2026-02-24T10:04:00Z",
      "value": 450.0,
      "score": 0.96,
      "is_anomaly": true,
      "method": "isolation_forest"
    }
  ],
  "model_version": "anomaly-detector-v1.2.0",
  "inference_time_ms": 12.5
}
```

### 2. Root Cause Analysis

```
POST /analyze/rca
```

**Request Body:**
```json
{
  "incident": {
    "id": "inc-a1b2c3d4",
    "title": "High CPU utilization",
    "description": "...",
    "severity": "high",
    "timeline": [...]
  },
  "related_anomalies": [...],
  "topology_context": {...},
  "recent_changes": [...],
  "similar_incidents": [...],
  "max_hypotheses": 5,
  "llm_provider": "auto",
  "tenant_id": "tenant-001"
}
```

**Response:**
```json
{
  "hypotheses": [
    {
      "rank": 1,
      "hypothesis": "Memory leak in JWT token parsing cache...",
      "confidence": 0.87,
      "reasoning": "Step-by-step analysis...",
      "evidence_links": [...],
      "suggested_remediation": "Implement LRU cache with TTL..."
    }
  ],
  "model_version": "rca-v1.0.0",
  "llm_provider_used": "llama-3-70b",
  "inference_time_ms": 3200
}
```

### 3. Metrics Analysis

```
POST /analyze/metrics
```

### 4. Log Analysis

```
POST /analyze/logs
```

### 5. Cost Analysis

```
POST /analyze/cost
```

**Request Body:**
```json
{
  "cost_data": [...],
  "historical_data": [...],
  "analysis_type": "anomaly_detection",
  "tenant_id": "tenant-001"
}
```

### 6. Forecasting

```
POST /forecast
```

**Request Body:**
```json
{
  "metric_name": "daily_cost_usd",
  "historical_values": [...],
  "historical_timestamps": [...],
  "horizon_steps": 30,
  "method": "prophet",
  "confidence_level": 0.95,
  "tenant_id": "tenant-001"
}
```

**Response:**
```json
{
  "predictions": [
    {
      "timestamp": "2026-03-01T00:00:00Z",
      "predicted_value": 1520.50,
      "lower_bound": 1380.20,
      "upper_bound": 1660.80
    }
  ],
  "model_version": "forecaster-prophet-v1.1.0",
  "method": "prophet",
  "confidence_level": 0.95,
  "inference_time_ms": 850
}
```

### 7. Cost Forecasting

```
POST /forecast/cost
```

### 8. Event Correlation

```
POST /correlate
```

**Request Body:**
```json
{
  "events": [
    {
      "id": "evt-001",
      "service": "api-server",
      "type": "error_rate_spike",
      "timestamp": "2026-02-24T10:04:00Z",
      "severity": "high"
    },
    {
      "id": "evt-002",
      "service": "database",
      "type": "connection_pool_exhaustion",
      "timestamp": "2026-02-24T10:03:30Z",
      "severity": "critical"
    }
  ],
  "topology": {...},
  "correlation_method": "gnn",
  "tenant_id": "tenant-001"
}
```

**Response:**
```json
{
  "correlation_groups": [
    {
      "group_id": "cg-xyz789",
      "primary_event": "evt-002",
      "contributing_events": ["evt-001"],
      "correlation_score": 0.93,
      "root_cause_hypothesis": "Database connection pool exhaustion caused cascading API errors",
      "correlation_method": "gnn"
    }
  ],
  "model_version": "correlator-gnn-v1.0.0",
  "inference_time_ms": 45
}
```

### 9. Health Check

```
GET /health
```

**Response:**
```json
{
  "status": "healthy",
  "models_loaded": [
    "anomaly-detector-v1.2.0",
    "rca-v1.0.0",
    "forecaster-prophet-v1.1.0",
    "correlator-gnn-v1.0.0"
  ],
  "gpu_available": false,
  "uptime_seconds": 86400,
  "version": "1.0.0"
}
```

---

## Rate Limits

Rate limits are enforced per tenant at the Go Gateway:

| Tier | Requests/Second | Burst |
|------|----------------|-------|
| Free | 10 | 20 |
| Standard | 100 | 200 |
| Enterprise | 1,000 | 2,000 |
| Unlimited | No limit | No limit |

Rate limit headers are included in all responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1708905600
```

---

## HTTP Status Codes

| Code | Meaning | When Used |
|------|---------|-----------|
| 200 | OK | Successful GET, PUT, POST (non-creation) |
| 201 | Created | Successful resource creation |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid request body or parameters |
| 401 | Unauthorized | Missing or invalid JWT token |
| 403 | Forbidden | Valid token but insufficient permissions |
| 404 | Not Found | Resource does not exist for the given tenant |
| 409 | Conflict | Resource state conflict (e.g., resolving already-resolved incident) |
| 422 | Unprocessable Entity | Valid JSON but semantic validation failure |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Unexpected server error |
| 502 | Bad Gateway | Upstream service (AI Brain) unavailable |
| 503 | Service Unavailable | Service is starting up or shutting down |

---

*For interactive API testing, see [22-Postman-Collection.md](./22-Postman-Collection.md). For webhook event schemas, see [23-Webhook-Specifications.md](./23-Webhook-Specifications.md).*
