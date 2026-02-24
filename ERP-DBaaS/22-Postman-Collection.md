# ERP-DBaaS Postman Collection

This document contains a complete Postman collection for testing all ERP-DBaaS API endpoints, including environment variables, pre-request scripts, and test assertions.

---

## Table of Contents

1. [Environment Setup](#environment-setup)
2. [Collection Variables](#collection-variables)
3. [Pre-Request Scripts](#pre-request-scripts)
4. [Endpoints](#endpoints)
5. [Test Scripts](#test-scripts)
6. [Import Instructions](#import-instructions)

---

## Environment Setup

### Development Environment

```json
{
  "name": "ERP-DBaaS - Development",
  "values": [
    {
      "key": "baseUrl",
      "value": "http://localhost:8090",
      "type": "default",
      "enabled": true
    },
    {
      "key": "apiBaseUrl",
      "value": "http://localhost:3000",
      "type": "default",
      "enabled": true
    },
    {
      "key": "tenantId",
      "value": "dev-tenant-001",
      "type": "default",
      "enabled": true
    },
    {
      "key": "userId",
      "value": "dev-user-001",
      "type": "default",
      "enabled": true
    },
    {
      "key": "profile",
      "value": "erp-aidd-strict",
      "type": "default",
      "enabled": true
    },
    {
      "key": "jwtToken",
      "value": "",
      "type": "secret",
      "enabled": true
    },
    {
      "key": "instanceId",
      "value": "",
      "type": "default",
      "enabled": true
    },
    {
      "key": "backupId",
      "value": "",
      "type": "default",
      "enabled": true
    },
    {
      "key": "pluginId",
      "value": "",
      "type": "default",
      "enabled": true
    },
    {
      "key": "rotationId",
      "value": "",
      "type": "default",
      "enabled": true
    }
  ]
}
```

### Staging Environment

```json
{
  "name": "ERP-DBaaS - Staging",
  "values": [
    {
      "key": "baseUrl",
      "value": "https://dbaas-staging.erp.businessactivation.cloud",
      "type": "default",
      "enabled": true
    },
    {
      "key": "tenantId",
      "value": "staging-tenant-001",
      "type": "default",
      "enabled": true
    },
    {
      "key": "jwtToken",
      "value": "",
      "type": "secret",
      "enabled": true
    }
  ]
}
```

---

## Collection Variables

| Variable | Description | Example |
|---|---|---|
| `baseUrl` | Gateway base URL | `http://localhost:8090` |
| `apiBaseUrl` | Direct API base URL | `http://localhost:3000` |
| `tenantId` | Tenant ID for dev auth bypass | `dev-tenant-001` |
| `userId` | User ID for dev auth bypass | `dev-user-001` |
| `profile` | Policy profile | `erp-aidd-strict` |
| `jwtToken` | Authentik JWT token (production) | `eyJhbGciOiJSUz...` |
| `instanceId` | Auto-populated instance ID | Set by provisioning request |
| `backupId` | Auto-populated backup ID | Set by backup request |
| `pluginId` | Auto-populated plugin ID | Set by plugin registration |
| `rotationId` | Auto-populated rotation ID | Set by rotation request |

---

## Pre-Request Scripts

### Collection-Level Pre-Request Script

This script runs before every request in the collection. It sets authentication headers based on the environment.

```javascript
// Collection-level pre-request script
const jwtToken = pm.environment.get("jwtToken");

if (jwtToken) {
    // Production: Use JWT Bearer token
    pm.request.headers.add({
        key: "Authorization",
        value: "Bearer " + jwtToken
    });
} else {
    // Development: Use dev auth bypass headers
    pm.request.headers.add({
        key: "X-Dev-Tenant-ID",
        value: pm.environment.get("tenantId") || "dev-tenant-001"
    });
    pm.request.headers.add({
        key: "X-Dev-User-ID",
        value: pm.environment.get("userId") || "dev-user-001"
    });
    pm.request.headers.add({
        key: "X-Dev-Profile",
        value: pm.environment.get("profile") || "erp-aidd-strict"
    });
}

// Set content type for requests with body
if (pm.request.body && pm.request.body.raw) {
    pm.request.headers.add({
        key: "Content-Type",
        value: "application/json"
    });
}

// Generate a unique request ID
pm.request.headers.add({
    key: "X-Request-ID",
    value: "postman-" + Date.now() + "-" + Math.random().toString(36).substr(2, 9)
});
```

---

## Endpoints

### 1. Health and Capabilities

#### 1.1 Gateway Health Check

```
GET {{baseUrl}}/healthz
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Status is healthy", () => {
    const json = pm.response.json();
    pm.expect(json.status).to.equal("healthy");
    pm.expect(json.module).to.equal("ERP-DBaaS");
});
```

#### 1.2 API Health Check

```
GET {{apiBaseUrl}}/healthz
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Service is dbaas-api", () => {
    const json = pm.response.json();
    pm.expect(json.service).to.equal("dbaas-api");
});
```

#### 1.3 API Readiness Check

```
GET {{apiBaseUrl}}/readyz
```

#### 1.4 Module Capabilities

```
GET {{baseUrl}}/v1/capabilities
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Module is ERP-DBaaS", () => {
    const json = pm.response.json();
    pm.expect(json.module).to.equal("ERP-DBaaS");
    pm.expect(json.capabilities).to.include("database_provisioning");
    pm.expect(json.aidd_governance).to.equal("ERP_AIDD_STRICT");
});
```

---

### 2. Instances

#### 2.1 Provision Instance

```
POST {{baseUrl}}/v1/dbaas/instances
```

**Body (raw JSON)**:
```json
{
    "engine": "yugabytedb",
    "version": "2.21",
    "plan": "M",
    "haMode": "standalone",
    "region": "africa-west"
}
```

**Test Script**:
```javascript
pm.test("Status 201", () => pm.response.to.have.status(201));
pm.test("Instance created", () => {
    const json = pm.response.json();
    pm.expect(json.success).to.be.true;
    pm.expect(json.data.status).to.equal("provisioning");
    pm.expect(json.data.engine).to.equal("yugabytedb");
    // Save instance ID for subsequent requests
    pm.environment.set("instanceId", json.data.id);
});
```

#### 2.2 Provision ScyllaDB Instance

```
POST {{baseUrl}}/v1/dbaas/instances
```

**Body (raw JSON)**:
```json
{
    "engine": "scylladb",
    "version": "6.1",
    "plan": "L",
    "haMode": "ha",
    "region": "africa-west"
}
```

#### 2.3 Provision ClickHouse Instance

```
POST {{baseUrl}}/v1/dbaas/instances
```

**Body (raw JSON)**:
```json
{
    "engine": "clickhouse",
    "version": "24.12",
    "plan": "XL",
    "haMode": "ha",
    "region": "europe-west"
}
```

#### 2.4 Provision Denied Engine (Negative Test)

```
POST {{baseUrl}}/v1/dbaas/instances
```

**Body (raw JSON)**:
```json
{
    "engine": "postgresql",
    "version": "16",
    "plan": "M",
    "haMode": "standalone"
}
```

**Test Script**:
```javascript
pm.test("Status 403 - Policy Violation", () => pm.response.to.have.status(403));
pm.test("Error is policy_violation", () => {
    const json = pm.response.json();
    pm.expect(json.success).to.be.false;
    pm.expect(json.error).to.equal("policy_violation");
    pm.expect(json.message).to.include("DENIED");
});
```

#### 2.5 List Instances

```
GET {{baseUrl}}/v1/dbaas/instances?page=1&pageSize=20
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Returns paginated data", () => {
    const json = pm.response.json();
    pm.expect(json.success).to.be.true;
    pm.expect(json.data).to.be.an("array");
    pm.expect(json).to.have.property("total");
    pm.expect(json).to.have.property("page");
    pm.expect(json).to.have.property("pageSize");
});
```

#### 2.6 List Instances (Filtered by Engine)

```
GET {{baseUrl}}/v1/dbaas/instances?engine=yugabytedb&status=running
```

#### 2.7 Get Instance Details

```
GET {{baseUrl}}/v1/dbaas/instances/{{instanceId}}
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Returns instance data", () => {
    const json = pm.response.json();
    pm.expect(json.success).to.be.true;
    pm.expect(json.data.id).to.equal(pm.environment.get("instanceId"));
});
```

#### 2.8 Get Instance Metrics

```
GET {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/metrics
```

#### 2.9 Scale Instance

```
PUT {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/scale
```

**Body (raw JSON)**:
```json
{
    "plan": "L",
    "replicas": 3
}
```

**Test Script**:
```javascript
pm.test("Status 200 or 409", () => {
    pm.expect(pm.response.code).to.be.oneOf([200, 409]);
});
```

#### 2.10 Update Instance Config

```
PUT {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/config
```

**Body (raw JSON)**:
```json
{
    "config": {
        "ysql_max_connections": 300,
        "log_min_duration_statement": 1000
    },
    "restartRequired": true
}
```

#### 2.11 Rolling Restart

```
POST {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/restart
```

#### 2.12 Decommission Instance

```
DELETE {{baseUrl}}/v1/dbaas/instances/{{instanceId}}
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Decommissioning initiated", () => {
    const json = pm.response.json();
    pm.expect(json.data.status).to.equal("decommissioning");
});
```

---

### 3. Backups

#### 3.1 Trigger Backup

```
POST {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/backup
```

**Body (raw JSON)**:
```json
{
    "type": "full",
    "retentionDays": 30
}
```

**Test Script**:
```javascript
pm.test("Status 201", () => pm.response.to.have.status(201));
pm.test("Backup initiated", () => {
    const json = pm.response.json();
    pm.expect(json.success).to.be.true;
    pm.expect(json.data.encrypted).to.be.true;
    pm.environment.set("backupId", json.data.id);
});
```

#### 3.2 Trigger Incremental Backup

```
POST {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/backup
```

**Body (raw JSON)**:
```json
{
    "type": "incremental",
    "retentionDays": 14
}
```

#### 3.3 List Backups

```
GET {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/backups?page=1&pageSize=10
```

#### 3.4 Restore from Backup

```
POST {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/restore
```

**Body (raw JSON)**:
```json
{
    "backupId": "{{backupId}}"
}
```

**Test Script**:
```javascript
pm.test("Status 202 or 404", () => {
    pm.expect(pm.response.code).to.be.oneOf([202, 404]);
});
```

---

### 4. Credentials

#### 4.1 Get Credentials

```
GET {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/credentials
```

**Test Script**:
```javascript
pm.test("Returns credential info", () => {
    if (pm.response.code === 200) {
        const json = pm.response.json();
        pm.expect(json.data).to.have.property("host");
        pm.expect(json.data).to.have.property("port");
        pm.expect(json.data).to.have.property("connectionString");
    }
});
```

#### 4.2 Rotate Credentials

```
POST {{baseUrl}}/v1/dbaas/instances/{{instanceId}}/credentials/rotate
```

**Test Script**:
```javascript
pm.test("Status 202 or 409", () => {
    pm.expect(pm.response.code).to.be.oneOf([202, 409]);
    if (pm.response.code === 202) {
        const json = pm.response.json();
        pm.environment.set("rotationId", json.data.rotationId);
    }
});
```

---

### 5. Engines, Plans, and Profiles

#### 5.1 List Engines

```
GET {{baseUrl}}/v1/dbaas/engines
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Returns 8 AIDD engines", () => {
    const json = pm.response.json();
    pm.expect(json.data.length).to.be.at.least(8);
    const engines = json.data.map(e => e.engine);
    pm.expect(engines).to.include("yugabytedb");
    pm.expect(engines).to.include("scylladb");
    pm.expect(engines).to.include("dragonfly");
    pm.expect(engines).to.include("clickhouse");
});
```

#### 5.2 List Plans

```
GET {{baseUrl}}/v1/dbaas/plans
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Returns 4 plans", () => {
    const json = pm.response.json();
    pm.expect(json.data.length).to.equal(4);
    const sizes = json.data.map(p => p.size);
    pm.expect(sizes).to.include.members(["S", "M", "L", "XL"]);
});
```

#### 5.3 List Profiles

```
GET {{baseUrl}}/v1/dbaas/profiles
```

**Test Script**:
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Returns 2 profiles", () => {
    const json = pm.response.json();
    pm.expect(json.data.length).to.equal(2);
    const names = json.data.map(p => p.name);
    pm.expect(names).to.include("erp-aidd-strict");
    pm.expect(names).to.include("commercial-flexible");
});
```

---

### 6. Plugins

#### 6.1 List Plugins

```
GET {{baseUrl}}/v1/dbaas/plugins?page=1&pageSize=20
```

#### 6.2 List Plugins (Filtered by Status)

```
GET {{baseUrl}}/v1/dbaas/plugins?status=active&engine=neo4j
```

#### 6.3 Register Plugin

```
POST {{baseUrl}}/v1/dbaas/plugins
```

**Body (raw JSON)**:
```json
{
    "name": "neo4j-custom-plugin",
    "engine": "neo4j",
    "version": "1.0.0",
    "grpcEndpoint": "http://neo4j-plugin.dbaas-plugins.svc.cluster.local:50051",
    "capabilities": ["provision", "scale", "backup", "restore", "health"],
    "securityContext": {
        "serviceAccount": "neo4j-plugin-sa",
        "namespace": "dbaas-plugins"
    }
}
```

**Test Script**:
```javascript
pm.test("Status 201 or 409", () => {
    pm.expect(pm.response.code).to.be.oneOf([201, 409]);
    if (pm.response.code === 201) {
        const json = pm.response.json();
        pm.expect(json.data.status).to.equal("pending_validation");
        pm.environment.set("pluginId", json.data.id);
    }
});
```

---

## Test Scripts

### Collection-Level Test Script

This script runs after every request and validates common response properties.

```javascript
// Collection-level test script

// Verify response time is reasonable
pm.test("Response time < 5000ms", () => {
    pm.expect(pm.response.responseTime).to.be.below(5000);
});

// Verify response has correct content type
if (pm.response.code !== 204) {
    pm.test("Content-Type is JSON", () => {
        pm.response.to.have.header("Content-Type");
        pm.expect(pm.response.headers.get("Content-Type")).to.include("application/json");
    });
}

// Verify security headers from gateway
if (pm.request.url.toString().includes(pm.environment.get("baseUrl"))) {
    pm.test("Has X-Request-ID header", () => {
        pm.response.to.have.header("X-Request-ID");
    });
    pm.test("Has X-Content-Type-Options header", () => {
        pm.response.to.have.header("X-Content-Type-Options");
    });
}

// Verify rate limit headers on API routes
if (pm.request.url.toString().includes("/v1/dbaas/")) {
    pm.test("Has rate limit headers", () => {
        if (pm.response.headers.has("X-RateLimit-Limit")) {
            pm.expect(parseInt(pm.response.headers.get("X-RateLimit-Limit"))).to.be.above(0);
        }
    });
}
```

---

## Import Instructions

### Method 1: Import as Postman Collection JSON

1. Open Postman.
2. Click **Import** in the top-left.
3. Select **Raw text** and paste the collection JSON below.
4. Click **Import**.
5. Import the environment JSON separately under **Environments**.

### Method 2: Manual Setup

1. Create a new Collection named "ERP-DBaaS".
2. Add the collection-level pre-request script and test script.
3. Create folders: Health, Instances, Backups, Credentials, Engines, Plugins.
4. Add each request as described above.
5. Create environments (Development, Staging) with the variables listed.

### Running the Collection

```bash
# Using Newman (Postman CLI)
npm install -g newman

# Run against development environment
newman run erp-dbaas-collection.json \
  -e erp-dbaas-dev-environment.json \
  --reporters cli,json \
  --reporter-json-export results.json

# Run with specific folder
newman run erp-dbaas-collection.json \
  -e erp-dbaas-dev-environment.json \
  --folder "Instances"
```

### Recommended Test Execution Order

For end-to-end testing, execute requests in this order:

1. Gateway Health Check
2. API Health Check
3. Module Capabilities
4. List Engines
5. List Plans
6. List Profiles
7. Provision Instance (saves `instanceId`)
8. List Instances
9. Get Instance Details
10. Get Instance Metrics
11. Update Instance Config
12. Scale Instance
13. Rolling Restart
14. Trigger Backup (saves `backupId`)
15. List Backups
16. Get Credentials
17. Rotate Credentials
18. Register Plugin
19. List Plugins
20. Restore from Backup
21. Decommission Instance
