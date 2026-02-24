# ERP-AIOps Postman Collection

> **Document ID:** ERP-AIOPS-PC-022
> **Version:** 1.0.0
> **Last Updated:** 2026-02-24
> **Status:** Approved
> **Related Documents:** [21-API-Documentation.md](./21-API-Documentation.md), [17-README.md](./17-README.md)

---

## Overview

This document provides a comprehensive Postman collection for testing all ERP-AIOps API endpoints. The collection is organized by domain (Incidents, Anomalies, Rules, Topology, Remediation, Cost, Security, AI Brain) and includes pre-configured environment variables, authentication setup, and example request/response payloads.

---

## Setup Instructions

### 1. Import the Collection

1. Open Postman.
2. Click **Import** in the top left.
3. Select "Raw Text" and paste the collection JSON from the section below, or import the file from `ERP-AIOps/docs/postman/ERP-AIOps.postman_collection.json`.

### 2. Configure Environment

Create a new Postman Environment named **ERP-AIOps Local** with the following variables:

| Variable | Initial Value | Description |
|----------|--------------|-------------|
| `base_url` | `http://localhost:3000` | Gateway URL |
| `rust_api_url` | `http://localhost:8080` | Direct Rust API URL |
| `ai_brain_url` | `http://localhost:8081` | Direct AI Brain URL |
| `auth_token` | _(empty, auto-populated)_ | JWT Bearer token |
| `tenant_id` | `dev-tenant-001` | Development tenant ID |
| `incident_id` | _(empty, auto-populated)_ | Last created incident ID |
| `anomaly_id` | _(empty, auto-populated)_ | Last created anomaly ID |
| `rule_id` | _(empty, auto-populated)_ | Last created rule ID |
| `node_id` | _(empty, auto-populated)_ | Last created topology node ID |
| `action_id` | _(empty, auto-populated)_ | Last created remediation action ID |
| `scan_id` | _(empty, auto-populated)_ | Last created scan ID |

### 3. Authentication

The collection includes a pre-request script that automatically obtains a JWT token. Alternatively, set the `auth_token` variable manually:

**Pre-Request Script (Collection Level):**

```javascript
// Auto-authenticate if token is missing or expired
const token = pm.environment.get("auth_token");
if (!token || isTokenExpired(token)) {
    pm.sendRequest({
        url: pm.environment.get("base_url") + "/auth/token",
        method: "POST",
        header: { "Content-Type": "application/json" },
        body: {
            mode: "raw",
            raw: JSON.stringify({
                username: "admin@aiops.dev",
                password: "admin123",
                tenant_id: pm.environment.get("tenant_id")
            })
        }
    }, function (err, response) {
        if (!err && response.code === 200) {
            const body = response.json();
            pm.environment.set("auth_token", body.access_token);
        }
    });
}

function isTokenExpired(token) {
    try {
        const payload = JSON.parse(atob(token.split('.')[1]));
        return payload.exp * 1000 < Date.now();
    } catch {
        return true;
    }
}
```

---

## Collection Structure

```
ERP-AIOps API Collection
├── Health & Status
│   ├── Gateway Health
│   ├── Rust API Health
│   ├── AI Brain Health
│   └── Full System Health
├── Incidents
│   ├── List Incidents
│   ├── Create Incident
│   ├── Get Incident by ID
│   ├── Update Incident
│   ├── Resolve Incident
│   ├── Request RCA
│   └── Delete Incident
├── Anomalies
│   ├── List Anomalies
│   ├── Detect Anomalies
│   ├── Get Anomaly by ID
│   ├── Acknowledge Anomaly
│   └── Resolve Anomaly
├── Rules
│   ├── List Rules
│   ├── Create Rule
│   ├── Get Rule by ID
│   ├── Update Rule
│   ├── Enable Rule
│   ├── Disable Rule
│   └── Delete Rule
├── Topology
│   ├── List Nodes
│   ├── Create Node
│   ├── Get Edges
│   ├── Create Edge
│   ├── Impact Analysis
│   └── Delete Node
├── Remediation
│   ├── List Actions
│   ├── Execute Action
│   ├── Execute Action (Dry Run)
│   ├── Get Action Status
│   ├── Approve Action
│   ├── Rollback Action
│   ├── List Policies
│   └── Create Policy
├── Cost
│   ├── List Reports
│   ├── Get Recommendations
│   ├── Get Forecast
│   └── Get Monthly Summary
├── Security
│   ├── List Findings
│   ├── Trigger Scan
│   ├── Get Scan Results
│   ├── Get Posture Score
│   └── Update Finding Status
└── AI Brain (Direct)
    ├── Analyze Anomaly
    ├── Analyze RCA
    ├── Analyze Metrics
    ├── Analyze Logs
    ├── Analyze Cost
    ├── Forecast
    ├── Forecast Cost
    └── Correlate Events
```

---

## Request Examples

### Health & Status

#### Gateway Health

```
GET {{base_url}}/health

Headers:
  (none required)

Expected Response (200 OK):
{
  "status": "healthy",
  "backends": {
    "rust_api": "up",
    "ai_brain": "up"
  },
  "version": "1.0.0",
  "uptime_seconds": 3600
}
```

#### Full System Health

```
GET {{base_url}}/health/all

Headers:
  Authorization: Bearer {{auth_token}}

Expected Response (200 OK):
{
  "gateway": { "status": "healthy", "version": "1.0.0" },
  "rust_api": { "status": "healthy", "version": "1.0.0", "db": "connected", "cache": "connected" },
  "ai_brain": { "status": "healthy", "models_loaded": 4, "gpu_available": false },
  "yugabytedb": { "status": "healthy", "connections_active": 12, "connections_max": 100 },
  "dragonflydb": { "status": "healthy", "memory_used_mb": 45, "keys": 1250 },
  "rustfs": { "status": "healthy", "buckets": 3, "objects": 28 }
}
```

---

### Incidents

#### Create Incident

```
POST {{base_url}}/api/v1/incidents

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "title": "Database connection pool exhaustion on order-service",
  "description": "The order-service is experiencing connection pool exhaustion. Active connections have reached the maximum limit of 50. New requests are being rejected with 'connection pool timeout' errors. Error rate has spiked to 15% over the past 10 minutes.",
  "severity": "critical",
  "source": "prometheus",
  "tags": ["database", "connection-pool", "order-service", "production"],
  "metadata": {
    "alert_rule": "ConnectionPoolExhaustion",
    "active_connections": 50,
    "max_connections": 50,
    "wait_queue_length": 127,
    "affected_service": "order-service"
  }
}

Test Script:
  pm.test("Status code is 201", () => pm.response.to.have.status(201));
  const body = pm.response.json();
  pm.environment.set("incident_id", body.data.id);
  pm.test("Incident has correct severity", () => pm.expect(body.data.severity).to.equal("critical"));
  pm.test("Incident status is open", () => pm.expect(body.data.status).to.equal("open"));

Expected Response (201 Created):
{
  "data": {
    "id": "inc-d4e5f6g7",
    "tenant_id": "dev-tenant-001",
    "title": "Database connection pool exhaustion on order-service",
    "severity": "critical",
    "status": "open",
    "source": "prometheus",
    "tags": ["database", "connection-pool", "order-service", "production"],
    "created_at": "2026-02-24T10:30:00.000Z",
    "updated_at": "2026-02-24T10:30:00.000Z"
  }
}
```

#### List Incidents (Filtered)

```
GET {{base_url}}/api/v1/incidents?status=open&severity=critical&sort_by=created_at&sort_order=desc&per_page=10

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Returns items array", () => pm.expect(body.items).to.be.an("array"));
  pm.test("All items are critical", () => {
    body.items.forEach(item => pm.expect(item.severity).to.equal("critical"));
  });
```

#### Request RCA

```
POST {{base_url}}/api/v1/incidents/{{incident_id}}/rca

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "include_topology": true,
  "include_recent_changes": true,
  "include_similar_incidents": true,
  "max_hypotheses": 5,
  "llm_provider": "auto"
}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Has hypotheses", () => pm.expect(body.data.hypotheses).to.be.an("array").that.is.not.empty);
  pm.test("Hypotheses have confidence scores", () => {
    body.data.hypotheses.forEach(h => {
      pm.expect(h.confidence).to.be.a("number").and.to.be.at.least(0).and.to.be.at.most(1);
    });
  });
```

#### Resolve Incident

```
POST {{base_url}}/api/v1/incidents/{{incident_id}}/resolve

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "resolution_notes": "Increased connection pool size from 50 to 100 and identified leaking connections in the batch processing module. Applied fix in v3.2.1.",
  "root_cause": "Connection leak in batch order processing - connections were not returned to pool on error paths",
  "resolution_type": "code_fix"
}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Status is resolved", () => pm.expect(body.data.status).to.equal("resolved"));
  pm.test("Has resolved_at timestamp", () => pm.expect(body.data.resolved_at).to.not.be.null);
```

---

### Anomalies

#### Detect Anomalies

```
POST {{base_url}}/api/v1/anomalies/detect

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "metric_name": "order_service_response_time_ms",
  "service": "order-service",
  "values": [45, 48, 42, 47, 44, 46, 43, 290, 310, 285, 305, 295, 50, 47, 44],
  "timestamps": [
    "2026-02-24T10:00:00Z", "2026-02-24T10:01:00Z", "2026-02-24T10:02:00Z",
    "2026-02-24T10:03:00Z", "2026-02-24T10:04:00Z", "2026-02-24T10:05:00Z",
    "2026-02-24T10:06:00Z", "2026-02-24T10:07:00Z", "2026-02-24T10:08:00Z",
    "2026-02-24T10:09:00Z", "2026-02-24T10:10:00Z", "2026-02-24T10:11:00Z",
    "2026-02-24T10:12:00Z", "2026-02-24T10:13:00Z", "2026-02-24T10:14:00Z"
  ],
  "detection_method": "ensemble",
  "sensitivity": 0.9,
  "create_anomaly_record": true
}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Anomalies detected", () => pm.expect(body.data.anomalies_detected).to.be.above(0));
  if (body.data.anomaly_id) {
    pm.environment.set("anomaly_id", body.data.anomaly_id);
  }
```

---

### Topology

#### Create Topology Node

```
POST {{base_url}}/api/v1/topology/nodes

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "name": "order-service",
  "type": "service",
  "metadata": {
    "version": "3.2.1",
    "replicas": 3,
    "namespace": "production",
    "language": "go",
    "team": "commerce"
  },
  "tags": ["commerce", "critical-path", "production"]
}

Test Script:
  pm.test("Status code is 201", () => pm.response.to.have.status(201));
  pm.environment.set("node_id", pm.response.json().data.id);
```

#### Impact Analysis

```
GET {{base_url}}/api/v1/topology/impact/{{node_id}}?direction=downstream&max_depth=3

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Has impacted nodes", () => pm.expect(body.data.impacted_nodes).to.be.an("array"));
  pm.test("Has blast radius score", () => {
    pm.expect(body.data.blast_radius_score).to.be.a("number");
  });
```

---

### Remediation

#### Execute Remediation (Dry Run)

```
POST {{base_url}}/api/v1/remediation/execute

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "incident_id": "{{incident_id}}",
  "action_type": "scale",
  "target_service": "order-service",
  "dry_run": true,
  "parameters": {
    "direction": "up",
    "target_replicas": 5,
    "current_replicas": 3,
    "timeout_seconds": 300
  }
}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Action is in dry_run mode", () => pm.expect(body.data.dry_run).to.be.true);
  pm.environment.set("action_id", body.data.action_id);
```

#### Execute Remediation (Live)

```
POST {{base_url}}/api/v1/remediation/execute

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "incident_id": "{{incident_id}}",
  "action_type": "restart",
  "target_service": "order-service",
  "dry_run": false,
  "parameters": {
    "strategy": "rolling",
    "max_unavailable": 1,
    "timeout_seconds": 300
  }
}

Test Script:
  pm.test("Status code is 200 or 202", () => {
    pm.expect(pm.response.code).to.be.oneOf([200, 202]);
  });
  pm.environment.set("action_id", pm.response.json().data.action_id);
```

---

### Cost

#### Get Cost Recommendations

```
GET {{base_url}}/api/v1/cost/recommendations?status=pending&sort_by=estimated_monthly_savings&sort_order=desc

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Recommendations have savings estimates", () => {
    body.items.forEach(rec => {
      pm.expect(rec.estimated_monthly_savings).to.be.a("number").and.to.be.above(0);
    });
  });
```

---

### Security

#### Trigger Security Scan

```
POST {{base_url}}/api/v1/security/scans

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}
  Content-Type: application/json

Body:
{
  "scan_type": "full",
  "targets": ["infrastructure", "containers", "dependencies", "access_control"],
  "compliance_frameworks": ["SOC2", "ISO27001", "CIS-Kubernetes"]
}

Test Script:
  pm.test("Status code is 202", () => pm.response.to.have.status(202));
  pm.environment.set("scan_id", pm.response.json().data.scan_id);
```

#### Get Security Posture

```
GET {{base_url}}/api/v1/security/posture

Headers:
  Authorization: Bearer {{auth_token}}
  X-Tenant-ID: {{tenant_id}}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Score is between 0 and 100", () => {
    pm.expect(body.data.overall_score).to.be.at.least(0).and.at.most(100);
  });
  pm.test("Has grade", () => pm.expect(body.data.grade).to.be.oneOf(["A", "B", "C", "D", "F"]));
```

---

### AI Brain (Direct)

#### Analyze Anomaly (Direct to AI Brain)

```
POST {{ai_brain_url}}/analyze/anomaly

Headers:
  Content-Type: application/json
  X-Tenant-ID: {{tenant_id}}

Body:
{
  "metric_name": "cpu_utilization_percent",
  "values": [35, 38, 36, 37, 35, 34, 36, 92, 95, 94, 91, 93],
  "timestamps": [
    "2026-02-24T10:00:00Z", "2026-02-24T10:05:00Z", "2026-02-24T10:10:00Z",
    "2026-02-24T10:15:00Z", "2026-02-24T10:20:00Z", "2026-02-24T10:25:00Z",
    "2026-02-24T10:30:00Z", "2026-02-24T10:35:00Z", "2026-02-24T10:40:00Z",
    "2026-02-24T10:45:00Z", "2026-02-24T10:50:00Z", "2026-02-24T10:55:00Z"
  ],
  "method": "ensemble",
  "sensitivity": 0.95,
  "tenant_id": "dev-tenant-001"
}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  pm.test("Has anomalies array", () => pm.expect(pm.response.json().anomalies).to.be.an("array"));
```

#### Forecast (Direct to AI Brain)

```
POST {{ai_brain_url}}/forecast

Headers:
  Content-Type: application/json
  X-Tenant-ID: {{tenant_id}}

Body:
{
  "metric_name": "daily_active_users",
  "historical_values": [1200, 1250, 1180, 1300, 1350, 1280, 1400, 1450, 1380, 1500, 1550, 1480, 1600, 1650, 1580, 1700, 1750, 1680, 1800, 1850, 1780, 1900, 1950, 1880, 2000, 2050, 1980, 2100, 2150, 2080],
  "historical_timestamps": [
    "2026-01-25T00:00:00Z", "2026-01-26T00:00:00Z", "2026-01-27T00:00:00Z",
    "2026-01-28T00:00:00Z", "2026-01-29T00:00:00Z", "2026-01-30T00:00:00Z",
    "2026-01-31T00:00:00Z", "2026-02-01T00:00:00Z", "2026-02-02T00:00:00Z",
    "2026-02-03T00:00:00Z", "2026-02-04T00:00:00Z", "2026-02-05T00:00:00Z",
    "2026-02-06T00:00:00Z", "2026-02-07T00:00:00Z", "2026-02-08T00:00:00Z",
    "2026-02-09T00:00:00Z", "2026-02-10T00:00:00Z", "2026-02-11T00:00:00Z",
    "2026-02-12T00:00:00Z", "2026-02-13T00:00:00Z", "2026-02-14T00:00:00Z",
    "2026-02-15T00:00:00Z", "2026-02-16T00:00:00Z", "2026-02-17T00:00:00Z",
    "2026-02-18T00:00:00Z", "2026-02-19T00:00:00Z", "2026-02-20T00:00:00Z",
    "2026-02-21T00:00:00Z", "2026-02-22T00:00:00Z", "2026-02-23T00:00:00Z"
  ],
  "horizon_steps": 14,
  "method": "prophet",
  "confidence_level": 0.95,
  "tenant_id": "dev-tenant-001"
}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Has 14 predictions", () => pm.expect(body.predictions).to.have.length(14));
  pm.test("Predictions have bounds", () => {
    body.predictions.forEach(p => {
      pm.expect(p.lower_bound).to.be.below(p.predicted_value);
      pm.expect(p.upper_bound).to.be.above(p.predicted_value);
    });
  });
```

#### Correlate Events (Direct to AI Brain)

```
POST {{ai_brain_url}}/correlate

Headers:
  Content-Type: application/json
  X-Tenant-ID: {{tenant_id}}

Body:
{
  "events": [
    {
      "id": "evt-001",
      "service": "order-service",
      "type": "error_rate_spike",
      "timestamp": "2026-02-24T10:05:00Z",
      "severity": "high",
      "description": "5xx error rate increased from 0.1% to 15%"
    },
    {
      "id": "evt-002",
      "service": "payment-service",
      "type": "latency_spike",
      "timestamp": "2026-02-24T10:04:30Z",
      "severity": "high",
      "description": "P99 latency increased from 200ms to 2500ms"
    },
    {
      "id": "evt-003",
      "service": "database-primary",
      "type": "connection_saturation",
      "timestamp": "2026-02-24T10:03:00Z",
      "severity": "critical",
      "description": "Connection pool at 100% capacity"
    },
    {
      "id": "evt-004",
      "service": "inventory-service",
      "type": "timeout_errors",
      "timestamp": "2026-02-24T10:05:30Z",
      "severity": "medium",
      "description": "Database query timeouts increasing"
    }
  ],
  "topology": {
    "nodes": ["order-service", "payment-service", "inventory-service", "database-primary"],
    "edges": [
      {"from": "order-service", "to": "database-primary"},
      {"from": "payment-service", "to": "database-primary"},
      {"from": "inventory-service", "to": "database-primary"},
      {"from": "order-service", "to": "payment-service"}
    ]
  },
  "correlation_method": "gnn",
  "tenant_id": "dev-tenant-001"
}

Test Script:
  pm.test("Status code is 200", () => pm.response.to.have.status(200));
  const body = pm.response.json();
  pm.test("Has correlation groups", () => pm.expect(body.correlation_groups).to.be.an("array").that.is.not.empty);
  pm.test("Database event is primary", () => {
    pm.expect(body.correlation_groups[0].primary_event).to.equal("evt-003");
  });
```

---

## Collection Runner Workflows

### Workflow 1: Complete Incident Lifecycle

Run these requests in order to test the full incident lifecycle:

1. **Create Incident** -- Creates a new incident
2. **Get Incident by ID** -- Verifies creation
3. **Request RCA** -- Triggers root cause analysis
4. **Execute Remediation (Dry Run)** -- Simulates a fix
5. **Execute Remediation (Live)** -- Applies the fix
6. **Resolve Incident** -- Closes the incident
7. **Get Incident by ID** -- Verifies resolution

### Workflow 2: Anomaly Detection Pipeline

1. **Detect Anomalies** -- Submits metric data
2. **List Anomalies** -- Verifies anomaly was created
3. **Get Anomaly by ID** -- Checks anomaly details
4. **Acknowledge Anomaly** -- Acknowledges the anomaly
5. **Resolve Anomaly** -- Resolves after investigation

### Workflow 3: Security Assessment

1. **Trigger Security Scan** -- Initiates a full scan
2. **Get Scan Results** -- Retrieves scan findings
3. **List Findings** -- Reviews all findings
4. **Get Posture Score** -- Checks overall security score
5. **Update Finding Status** -- Triages findings

---

## Environment Templates

### Staging Environment

| Variable | Value |
|----------|-------|
| `base_url` | `https://aiops-staging.erp.internal` |
| `tenant_id` | `staging-tenant-001` |

### Production Environment (Read-Only)

| Variable | Value |
|----------|-------|
| `base_url` | `https://aiops.erp.internal` |
| `tenant_id` | `prod-tenant-001` |

**Warning**: The production environment collection should be configured with read-only requests only. Do not execute write operations (POST, PUT, DELETE) against production.

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| 401 Unauthorized | Token expired; re-run authentication request or clear `auth_token` variable |
| 403 Forbidden | User lacks required permissions; check role assignments |
| Connection refused | Verify services are running: `docker compose ps` |
| 502 Bad Gateway | AI Brain may be down; check: `curl http://localhost:8081/health` |
| Timeout | Increase Postman timeout in Settings; AI Brain RCA can take 5-10 seconds |

---

*For complete API documentation, see [21-API-Documentation.md](./21-API-Documentation.md). For webhook payloads, see [23-Webhook-Specifications.md](./23-Webhook-Specifications.md).*
