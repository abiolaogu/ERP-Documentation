# ERP-DBaaS User Manual

## Document Control

| Field             | Value                        |
|-------------------|------------------------------|
| Document Title    | ERP-DBaaS User Manual        |
| Version           | 1.0.0                       |
| Date              | 2026-02-24                   |
| Audience          | All DBaaS Users              |
| Classification    | Internal                     |

---

## 1. Getting Started

### 1.1 What is ERP-DBaaS?

ERP-DBaaS is a self-service platform that allows you to provision, manage, and scale databases for your ERP module without filing tickets or waiting for DBA assistance. Through a guided web interface or API, you can create production-grade database instances in minutes with full backup coverage, security compliance, and monitoring.

### 1.2 Supported Database Engines

| Engine       | Best For                                      | Icon  |
|--------------|-----------------------------------------------|-------|
| YugabyteDB   | Distributed SQL, OLTP workloads, ERP primary databases | SQL  |
| ScyllaDB     | High-throughput event stores, audit logs, wide-column data | NoSQL |
| DragonflyDB  | Caching, session management, real-time counters | Cache |
| MongoDB      | Flexible schema, document storage, CMS content | Doc   |
| CouchDB      | Offline-first sync, mobile backends, replication | Doc   |
| ClickHouse   | Analytics, reporting, data warehousing, OLAP   | OLAP  |
| TimescaleDB  | IoT telemetry, metrics, time-series data       | TS    |
| QuestDB      | High-ingest time-series, financial tick data   | TS    |

### 1.3 Accessing the Platform

1. Navigate to `https://dbaas.erp.internal` in your browser
2. You will be redirected to Authentik for authentication
3. Log in with your organizational SSO credentials
4. Upon successful authentication, you will be redirected to the DBaaS dashboard

**Required Roles**:
- `dbaas_viewer` - View instances and metrics (read-only)
- `dbaas_operator` - Create, manage, scale, and back up instances
- `dbaas_admin` - Full access including tenant and policy management

Contact your team lead or IT admin if you need role assignment.

### 1.4 Dashboard Overview

After logging in, you will see the main dashboard containing:

- **Instance Summary**: Total instances by status (running, degraded, provisioning)
- **Engine Distribution**: Pie chart showing instances by database engine
- **Quota Usage**: Progress bars for CPU, memory, storage, and instance count
- **Recent Activity**: Timeline of recent provisioning, scaling, and backup events
- **Health Alerts**: Any instances with degraded health or failed backups

---

## 2. Provisioning a Database

### 2.1 Starting the Provisioning Wizard

1. Click the **"+ New Instance"** button in the top-right corner of the dashboard
2. The provisioning wizard will open with 4 steps

### 2.2 Step 1: Engine Selection

On this screen, you will see all 8 supported database engines displayed as cards.

**For each engine, you can view**:
- Description and primary use cases
- Supported versions
- Available deployment modes (single-node, HA, clustered)
- Performance characteristics

**How to choose**:
- For a typical ERP module's primary database: **YugabyteDB** (distributed SQL with PostgreSQL compatibility)
- For caching or session management: **DragonflyDB**
- For analytics and reporting: **ClickHouse**
- For time-series data (IoT, metrics): **TimescaleDB** or **QuestDB**
- For flexible document storage: **MongoDB** or **CouchDB**
- For high-throughput event logs: **ScyllaDB**

Click on the engine card to select it, then click **"Next"**.

### 2.3 Step 2: Configuration

**Instance Name**:
- Enter a descriptive name (e.g., `inventory-primary`, `hrm-analytics`)
- Names must be lowercase, alphanumeric with hyphens, 3-63 characters
- A unique identifier will be prepended with your tenant name automatically

**Version**:
- Select from the dropdown of supported versions
- The latest stable version is recommended (marked with a star)

**Size Preset**:
Choose from pre-configured resource allocations:

| Preset  | vCPU | Memory | Storage | Recommended For                |
|---------|------|--------|---------|-------------------------------|
| Small   | 1    | 2 GB   | 20 GB   | Development, testing           |
| Medium  | 2    | 4 GB   | 50 GB   | Staging, light production      |
| Large   | 4    | 8 GB   | 100 GB  | Production workloads           |
| X-Large | 8    | 16 GB  | 250 GB  | High-traffic production        |
| Custom  | -    | -      | -       | Specify your own values        |

**Replicas** (for HA-capable engines):
- 1: Single node (development only; blocked by strict policy in production)
- 3: Standard HA (recommended for production)
- 5: Enhanced HA (for critical workloads)

**Backup Schedule**:
- Select a predefined schedule: Hourly, Every 6 Hours, Daily, Weekly
- Or enter a custom cron expression (e.g., `0 2 * * *` for daily at 2 AM UTC)
- Set retention period (7-365 days; minimum 30 days in production)

Click **"Next"** to proceed.

### 2.4 Step 3: Policy Review

The AIDD policy engine evaluates your configuration against the active policy profile.

**If all checks pass**: You will see a green checkmark with "Configuration Approved" and any optional recommendations.

**If checks fail (strict profile)**: You will see red violations that must be resolved before proceeding. Each violation includes:
- The rule that was violated
- The current value vs. the required value
- A recommendation for how to fix it

Click the **"Fix"** button next to each violation to automatically adjust the configuration, or go back to Step 2 to make manual changes.

**If checks produce warnings (flexible profile)**: You will see yellow warnings that are informational. You can proceed despite warnings, but they indicate areas for improvement.

Click **"Next"** after resolving all violations.

### 2.5 Step 4: Review and Confirm

This screen displays a summary of your complete configuration:

- Instance name and engine
- Resource allocation (CPU, memory, storage)
- Replica count and HA mode
- Backup schedule and retention
- Estimated quota impact
- Estimated monthly cost

Review all settings carefully. Click **"Provision"** to create the instance.

### 2.6 Provisioning Status

After clicking Provision, you will be taken to the instance detail page which shows:

- A progress indicator with the current provisioning stage
- Real-time status updates (via GraphQL subscription)
- Estimated time remaining

Typical provisioning stages:
1. **Pending**: Request submitted and queued
2. **Creating Namespace Resources**: Setting up network policies and secrets
3. **Deploying StatefulSet**: Creating database pods
4. **Initializing Database**: Running engine-specific initialization
5. **Configuring Backups**: Setting up the backup schedule
6. **Health Check**: Verifying connectivity and readiness
7. **Running**: Instance is ready for use

---

## 3. Managing Instances

### 3.1 Instance List

Navigate to **Instances** in the left sidebar to see all your database instances.

**List Features**:
- **Filter by**: Engine, status, size, creation date
- **Sort by**: Name, engine, status, created date, resource usage
- **Search**: Type to search by instance name
- **Bulk Actions**: Select multiple instances for bulk operations (start, stop, backup)

### 3.2 Instance Detail Page

Click on any instance to view its detail page, which contains:

**Overview Tab**:
- Current status and health indicator
- Engine, version, and size information
- Connection endpoint and port
- Uptime and last health check
- Quick action buttons (Scale, Backup, Rotate Credentials)

**Metrics Tab**:
- CPU utilization graph (1h, 6h, 24h, 7d, 30d)
- Memory usage graph
- Storage utilization with growth trend
- Connection count over time
- Query throughput (queries/second)
- Replication lag (for HA instances)

**Backups Tab**:
- Backup schedule details
- Backup history table (date, type, size, status, duration)
- On-demand backup button
- Restore button for each successful backup

**Configuration Tab**:
- Current resource allocation
- Engine-specific parameters
- Labels and annotations
- Policy profile and compliance status

**Logs Tab**:
- Recent operator events
- Provisioning logs
- Scaling events
- Error messages

---

## 4. Scaling an Instance

### 4.1 Vertical Scaling (CPU/Memory)

1. Navigate to the instance detail page
2. Click **"Scale"** button
3. Select the new size preset or enter custom values
4. Review the quota impact preview
5. Click **"Apply Scale"**

**Important Notes**:
- Vertical scaling performs a rolling update (minimal downtime for HA instances)
- Single-node instances may experience brief downtime during scaling
- You can only increase resources; contact admin for downsizing
- Scaling is blocked if it would exceed your quota

### 4.2 Horizontal Scaling (Replicas)

1. Navigate to the instance detail page
2. Click **"Scale"** button
3. Adjust the replica count slider
4. Review the quota impact
5. Click **"Apply Scale"**

**Important Notes**:
- Adding replicas requires available quota for additional CPU, memory, and storage
- Removing replicas triggers data rebalancing (may take several minutes)
- Minimum 3 replicas in production (strict policy)

### 4.3 Storage Expansion

1. Navigate to the instance detail page
2. Click **"Scale"** then select the **Storage** tab
3. Enter the new storage size (must be larger than current)
4. Click **"Expand Storage"**

**Important Notes**:
- Storage expansion is a grow-only operation (cannot shrink)
- Expansion is typically non-disruptive
- New storage is available immediately after the operation completes

---

## 5. Backup and Restore

### 5.1 Viewing Backup History

1. Navigate to the instance detail page
2. Click the **Backups** tab
3. View the list of all backups with status, size, and timestamp

### 5.2 Triggering an On-Demand Backup

1. Navigate to the instance detail page
2. Click **"Backup Now"** button
3. Select backup type:
   - **Full**: Complete snapshot of all data
   - **Incremental**: Only changes since last backup (faster, smaller)
4. Click **"Start Backup"**
5. Monitor progress on the backups tab

### 5.3 Modifying Backup Schedule

1. Navigate to the instance detail page
2. Click the **Backups** tab
3. Click **"Edit Schedule"**
4. Modify the cron expression or select a preset
5. Adjust retention period if needed
6. Click **"Save Schedule"**

### 5.4 Restoring from Backup

1. Navigate to the instance detail page
2. Click the **Backups** tab
3. Find the backup you want to restore from
4. Click the **"Restore"** button next to the backup
5. Choose restore mode:
   - **New Instance**: Creates a brand new instance from the backup (recommended)
   - **Point-in-Time**: Restore to a specific timestamp (if PITR is available)
6. Enter a name for the restored instance
7. Review the restore summary
8. Click **"Start Restore"**

**Important Notes**:
- Restores always create a new instance (non-destructive to the original)
- The restored instance receives new credentials (not the original ones)
- Restore time depends on backup size and engine type
- A progress bar shows restoration percentage

---

## 6. Credential Management

### 6.1 Viewing Connection Credentials

1. Navigate to the instance detail page
2. In the Overview tab, find the **Connection Information** section
3. You will see:
   - **Host**: The connection endpoint
   - **Port**: The connection port
   - **Username**: The database username
   - **Password**: Click "Show" to reveal (hidden by default)
   - **Database Name**: The default database name
   - **Connection String**: Full connection URI
4. Click the **copy icon** next to any field to copy it to your clipboard

**Security Note**: Connection information is only visible to users with `dbaas_operator` or `dbaas_admin` roles for instances in their tenant.

### 6.2 Rotating Credentials

Credential rotation generates new database passwords without instance downtime.

1. Navigate to the instance detail page
2. Click **"Rotate Credentials"** button
3. Confirm the rotation in the dialog
4. New credentials are generated and the Kubernetes Secret is updated
5. The old credentials remain valid for a 5-minute grace period
6. Update your application configuration with the new credentials

**Automatic Rotation**:
- If configured, credentials are automatically rotated on a schedule (e.g., every 90 days)
- You will receive a notification 7 days before automatic rotation
- After rotation, update your application's connection settings

### 6.3 Rotation History

1. Navigate to the instance detail page
2. Click the **Configuration** tab
3. View the **Credential Rotation History** section
4. Each entry shows: rotation date, initiated by (user or schedule), and status

---

## 7. Monitoring Instances

### 7.1 Health Status

Each instance displays one of these health statuses:

| Status       | Indicator | Meaning                                          |
|--------------|-----------|--------------------------------------------------|
| Healthy      | Green     | All pods running, queries responding, replication OK |
| Warning      | Yellow    | Minor issues detected (high CPU, replication lag) |
| Degraded     | Orange    | One or more pods unhealthy, reduced capacity      |
| Critical     | Red       | Instance not responding, data at risk             |
| Maintenance  | Blue      | Planned operation in progress (scaling, backup)   |

### 7.2 Metrics Dashboard

The Metrics tab provides time-series graphs for:

- **CPU Utilization**: Percentage of allocated CPU in use
- **Memory Usage**: Bytes used vs. allocated
- **Storage Utilization**: Disk space used vs. provisioned
- **IOPS**: Read and write operations per second
- **Connections**: Active client connections
- **Query Throughput**: Queries per second (for SQL engines)
- **Replication Lag**: Time behind primary (for HA instances)
- **Cache Hit Rate**: Cache efficiency (for DragonflyDB, ScyllaDB)

**Time Range Options**: 1 hour, 6 hours, 24 hours, 7 days, 30 days

### 7.3 Setting Up Alerts

1. Navigate to **Settings > Alerts** in the left sidebar
2. Click **"+ New Alert Rule"**
3. Configure:
   - **Instance**: Select one or all instances
   - **Metric**: Choose the metric to monitor
   - **Condition**: Greater than / Less than / Equals
   - **Threshold**: The value that triggers the alert
   - **Duration**: How long the condition must persist
   - **Notification**: Email, Slack, or PagerDuty
4. Click **"Save Rule"**

**Pre-configured Alerts** (enabled by default):
- CPU > 80% for 5 minutes
- Memory > 90% for 5 minutes
- Storage > 85% capacity
- Health status changes to Degraded or Critical
- Backup failure

---

## 8. Managing Plugins

### 8.1 Viewing Installed Plugins

1. Navigate to **Plugins** in the left sidebar
2. View the list of registered plugins with:
   - Plugin name and version
   - Type (monitoring, notification, validation, etc.)
   - Status (active, inactive, error)
   - Registered hooks
   - Health status

### 8.2 Registering a New Plugin

*Requires `dbaas_admin` role*

1. Navigate to **Plugins** in the left sidebar
2. Click **"+ Register Plugin"**
3. Fill in the registration form:
   - **Name**: Unique plugin identifier
   - **Version**: Semantic version number
   - **Type**: Select from dropdown (monitoring, notification, validation, alerting, metering)
   - **Container Image**: Full image reference (e.g., `registry.internal/plugins/my-plugin:1.0.0`)
   - **gRPC Port**: Port the plugin listens on (default: 50051)
   - **Hooks**: Select which lifecycle events to subscribe to
   - **Resource Limits**: CPU and memory limits for the plugin container
4. Click **"Register"**
5. The plugin container is pulled, validated, and started
6. Health checks begin automatically

### 8.3 Managing Plugin Status

- **Disable**: Temporarily stops the plugin from receiving hook events
- **Enable**: Re-activates a disabled plugin
- **Update**: Deploy a new version of the plugin image
- **Remove**: Unregister and remove the plugin completely

---

## 9. Quota Management

### 9.1 Viewing Your Quota

1. Navigate to **Quotas** in the left sidebar (or view the summary on the dashboard)
2. View your tenant's quota allocation and current usage:

| Resource         | Allocated | Used    | Available | Usage % |
|------------------|-----------|---------|-----------|---------|
| Instances        | 25        | 12      | 13        | 48%     |
| Total vCPU       | 64        | 28      | 36        | 44%     |
| Total Memory     | 128 GB    | 56 GB   | 72 GB     | 44%     |
| Total Storage    | 1,000 GB  | 420 GB  | 580 GB    | 42%     |
| Backup Storage   | 500 GB    | 180 GB  | 320 GB    | 36%     |
| API Rate         | 500/min   | -       | -         | -       |

### 9.2 Requesting a Quota Increase

If you need more resources than your current quota allows:

1. Navigate to **Quotas** in the left sidebar
2. Click **"Request Increase"**
3. Select the resource(s) you need increased
4. Enter the requested amount and justification
5. Click **"Submit Request"**
6. Your request will be reviewed by the platform admin team

### 9.3 Quota Alerts

You will receive automatic notifications at:
- **80% utilization**: Warning notification
- **95% utilization**: Critical notification
- **100% utilization**: Operations blocked notification

---

## 10. Decommissioning an Instance

When you no longer need a database instance:

1. Navigate to the instance detail page
2. Click **"Decommission"** button (in the Actions dropdown)
3. A confirmation dialog will appear with:
   - Warning about data loss
   - Option to create a final backup before decommission
   - Text field requiring you to type the instance name to confirm
4. Type the instance name and click **"Confirm Decommission"**

**What Happens During Decommission**:
1. Instance status changes to "Decommissioning"
2. A final backup is created (if selected)
3. All database pods are terminated
4. Persistent volumes are deleted
5. Kubernetes Secrets (credentials) are removed
6. Network policies are cleaned up
7. Instance record is marked as "Decommissioned" in the registry
8. Resources are returned to your quota

**Important Notes**:
- Decommissioning is **irreversible** (data is permanently deleted)
- Existing backups are retained per your retention policy
- The instance name becomes available for reuse after 24 hours

---

## 11. API Access

### 11.1 Authentication

All API requests require a valid JWT token from Authentik:

```bash
# Get a token from Authentik
TOKEN=$(curl -s -X POST https://auth.erp.internal/application/o/token/ \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  | jq -r '.access_token')
```

### 11.2 Common API Operations

**List your instances**:
```bash
curl -H "Authorization: Bearer $TOKEN" \
  https://dbaas.erp.internal/api/v1/instances
```

**Provision a new instance**:
```bash
curl -X POST -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  https://dbaas.erp.internal/api/v1/instances \
  -d '{
    "name": "my-new-database",
    "engine": "yugabytedb",
    "version": "2.20",
    "sizePreset": "medium",
    "replicas": 3,
    "backup": {
      "schedule": "0 2 * * *",
      "retentionDays": 30
    }
  }'
```

**Get instance details**:
```bash
curl -H "Authorization: Bearer $TOKEN" \
  https://dbaas.erp.internal/api/v1/instances/{instance-id}
```

**Trigger a backup**:
```bash
curl -X POST -H "Authorization: Bearer $TOKEN" \
  https://dbaas.erp.internal/api/v1/instances/{instance-id}/backup \
  -d '{"type": "full"}'
```

**Rotate credentials**:
```bash
curl -X POST -H "Authorization: Bearer $TOKEN" \
  https://dbaas.erp.internal/api/v1/instances/{instance-id}/credentials/rotate
```

### 11.3 GraphQL Access

Access the GraphQL endpoint for real-time queries and subscriptions:

**Endpoint**: `https://dbaas.erp.internal/v1/graphql`
**WebSocket**: `wss://dbaas.erp.internal/v1/graphql` (for subscriptions)

```graphql
query MyInstances {
  instances(where: { status: { _eq: "running" } }) {
    id
    name
    engine
    version
    status
    connection_endpoint
    created_at
  }
}
```

---

## 12. Troubleshooting

### 12.1 Common Issues

**Instance stuck in "Provisioning"**:
- Check the Logs tab on the instance detail page
- Verify quota is not exceeded
- Wait up to 10 minutes for large instances
- If still stuck, contact the platform team

**Cannot connect to database**:
- Verify the instance status is "Running"
- Check the connection endpoint and port are correct
- Ensure your application is in the same network or has ingress configured
- Verify credentials are current (not expired due to rotation)

**Backup failed**:
- Check available backup storage quota
- Verify the instance is in "Running" status
- Review backup logs on the Backups tab
- Retry the backup manually

**Policy violation on provisioning**:
- Review the violation details carefully
- Adjust your configuration to meet the policy requirements
- Use the "Fix" button for automatic remediation
- If the policy seems incorrect, contact the DBA team

### 12.2 Getting Help

- **Slack**: #dbaas-support
- **Email**: dbaas-support@erp.internal
- **Jira**: File a ticket under the DBAAS project
- **Documentation**: https://docs.erp.internal/dbaas

---

*This user manual is updated with each release. Last updated: February 24, 2026.*
