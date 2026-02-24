# Figma & Make Prompts -- ERP-DBaaS (Database as a Service)
> Version: 1.0 | Last Updated: 2026-02-24 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make (Integromat) automation prompts for the ERP-DBaaS module. ERP-DBaaS is the Database-as-a-Service vertical of the consolidated ERP platform, covering multi-engine database provisioning, instance lifecycle management, backup and restore operations, credential rotation, plugin management, quota enforcement, and engine catalog browsing. All prompts reference real services (dbaas-gateway, dbaas-api, per-engine operators for YugabyteDB, ScyllaDB, DragonflyDB, MongoDB, CouchDB, ClickHouse, TimescaleDB, and QuestDB) and align with the AIDD guardrails defined in `erp/aidd.guardrails.yaml`.

The target frontend stack is **Refine.dev + Ant Design** (React) with Hasura GraphQL federation (port 19108), `dbaas_` table prefix, Go gateway (dbaas-gateway, port 8090), Node.js API (dbaas-api, port 3000), and multi-tenant isolation via `tenant_id` and `X-Tenant-ID` headers.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance on every screen; color contrast ratio >= 4.5:1 for text, >= 3:1 for large text and UI components
- Minimum 44x44px touch targets on all interactive elements (buttons, links, checkboxes, table row actions)
- Clear focus-visible outlines (2px solid, offset 2px) for keyboard navigation
- All form fields include visible labels (never placeholder-only), required field indicators, and inline validation messages
- Plain-language labels (e.g., "Database Instances" not "Service Instance Resource Entities")
- Semantic HTML landmarks: `<nav>`, `<main>`, `<aside>`, `<footer>` reflected in Figma layer naming
- Screen reader annotations for charts (alt-text, data tables), icon-only buttons (aria-label), and live regions
- Engine icon components must include text labels or tooltips; never icon-only identification of database engines

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading via React.lazy/Suspense; initial route bundle < 220 KB gzipped
- Skeleton UIs shown within 200ms; full content within 1s on 4G
- Virtualized tables for datasets > 50 rows (instance lists, backup records, credential rotations)
- Image assets: WebP format, max 80 KB hero, max 20 KB thumbnails; engine icons as inline SVG components
- Prefetch next-likely routes (e.g., instance detail from instance list, backup create from instance show)
- Spinner fallback component (`<Spin size="large" tip="Loading..." />`) for all lazy-loaded page boundaries

### 2.3 Reliability, Trust, And Safety
- Every destructive action (delete instance, overwrite restore, credential rotation) requires a confirmation dialog with explicit typing or checkbox
- Instance deletion requires typing the engine name to confirm (e.g., "Type 'YugabyteDB' to confirm deletion")
- Inline recovery paths on all error states ("Retry", "Go back", "Contact support")
- Optimistic UI for low-risk mutations (backup initiation) with undo toast (5s window)
- Trust signals: instance status badges with real-time WebSocket updates (via graphql-ws live provider), "Last synced" timestamp on dashboard, provisioning progress indicators
- Scaling operations display a warning alert: "Scaling will cause a brief disruption while resources are reallocated"
- Restore operations display a warning alert: "Restoring from a backup will create a new instance or overwrite the target instance with backup data"

### 2.4 Observability And Testability
- Every CTA emits a structured analytics event: `{event, module: "dbaas", entity, action, user_role, timestamp}`
- Error boundaries at page and widget level with correlation_id for support tickets
- Performance instrumentation: LCP, FID, CLS metrics on all page-level components
- Feature flag integration points annotated in Figma (e.g., "Visible when `dbaas_multi_region_enabled` = true")
- All instance status transitions logged for audit trail

### 2.5 Security And Privacy
- Connection strings and credentials are masked by default with a "Show" toggle requiring re-authentication
- All backup encryption toggles default to "Encrypted" (on)
- Tenant isolation enforced at every API call via `X-Tenant-ID` header; cross-tenant data never displayed
- Credential rotation creates new secrets and invalidates old ones atomically; UI shows rotation status in real time
- gRPC plugin endpoints validated server-side before registration is accepted
- Rate limiting per tenant tier (Tier A: 1000 req/min, Tier B: 500 req/min, Tier C: 100 req/min) with user-facing quota warnings at 80% and 90% thresholds

---

## 3. Module-Specific Design Context

### 3.1 Backend Services

| Service | Technology | Port | Purpose |
|---------|-----------|------|---------|
| dbaas-gateway | Go | 8090 | API gateway, routing, auth, rate limiting |
| dbaas-api | Node.js | 3000 | Core business logic, CRUD operations |
| Hasura GraphQL | Hasura | 19108 | GraphQL federation, subscriptions, live queries |
| yugabytedb-operator | Go/K8s | - | YugabyteDB lifecycle management |
| scylladb-operator | Go/K8s | - | ScyllaDB lifecycle management |
| dragonflydb-operator | Go/K8s | - | DragonflyDB lifecycle management |
| mongodb-operator | Go/K8s | - | MongoDB lifecycle management |
| couchdb-operator | Go/K8s | - | CouchDB lifecycle management |
| clickhouse-operator | Go/K8s | - | ClickHouse lifecycle management |
| timescaledb-operator | Go/K8s | - | TimescaleDB lifecycle management |
| questdb-operator | Go/K8s | - | QuestDB lifecycle management |

### 3.2 Personas

| Persona | Role | Primary Tasks |
|---------|------|---------------|
| Platform Engineer | Infrastructure team lead | Provision instances, manage scaling, monitor quotas |
| Database Administrator | DBA / DevOps | Backup/restore, credential rotation, plugin management |
| Application Developer | Consumer of database services | View connection strings, check instance status, browse engines |
| Tenant Admin | Organization administrator | Quota management, instance oversight, cost monitoring |

### 3.3 Key Entities (Hasura GraphQL, `dbaas_` prefix)

| Entity | Table | Description |
|--------|-------|-------------|
| ServiceInstance | `dbaas_service_instances` | Provisioned database instance with engine, version, plan, HA mode, region, status, resource usage |
| BackupRecord | `dbaas_backup_records` | Backup metadata: type (full/incremental/snapshot), status, size, encryption, retention |
| PluginRegistration | `dbaas_plugin_registrations` | Registered operator plugins with engine, gRPC endpoint, capabilities, validation status |
| TenantQuota | `dbaas_tenant_quotas` | Per-tenant resource limits and current usage (instances, CPU, memory, storage) |
| CredentialRotation | `dbaas_credential_rotations` | Credential rotation audit log with status, initiator, timestamps |
| EngineInfo | (derived) | Static engine catalog: 8 engines with versions, features, descriptions, colors |

### 3.4 Database Engines

| Engine | Brand Color | Use Cases |
|--------|------------|-----------|
| YugabyteDB | #FF6B35 | Distributed SQL, ACID transactions, PostgreSQL compatible |
| ScyllaDB | #6C5CE7 | Low latency, high throughput, CQL compatible |
| DragonflyDB | #00B894 | Redis compatible, in-memory, multi-threaded |
| MongoDB | #4DB33D | Document store, flexible schema, aggregation |
| CouchDB | #E43525 | Multi-master replication, RESTful API, MVCC |
| ClickHouse | #FFCC00 | Columnar storage, real-time analytics, SQL support |
| TimescaleDB | #FDB515 | Time-series, PostgreSQL extension, continuous aggregates |
| QuestDB | #4A90D9 | Time-series, high ingest rate, column-oriented |

---

## 4. Figma Make Master Prompt (Copy-Paste Ready)

```
Design the complete ERP-DBaaS (Database as a Service) application for the
consolidated ERP platform. This module allows Platform Engineers, DBAs, and
Application Developers to provision, manage, scale, back up, and monitor
multi-engine database instances across a multi-tenant Kubernetes environment.

TARGET STACK: Refine.dev + Ant Design + React, Hasura GraphQL (port 19108),
Go gateway (port 8090), Node.js API (port 3000), 8 database operators.

BRAND / THEME:
- Primary: #2563eb (Blue)
- Primary Hover: #1d4ed8
- Success: #10b981
- Warning: #f59e0b
- Error: #ef4444
- Info: #3b82f6
- Background: #f5f7fa
- Surface: #ffffff
- Sidebar: #001529 (dark)
- Sidebar Active: #2563eb
- Font: "Plus Jakarta Sans", -apple-system, system-ui, sans-serif
- Border Radius: 10px (cards), 8px (buttons, inputs, tags), 6px (small tags)
- Control Height: 40px
- Table Header: #fafbfc background, #64748b text

SIDEBAR NAVIGATION (260px expanded, collapsible):
- Logo: "DB" badge (32px, gradient #2563eb to #1d4ed8) + "ERP DBaaS" title
- Items: Dashboard, Instances, Backups, Engines, Plugins, Credentials, Quotas
- Icons: DashboardOutlined, DatabaseOutlined, CloudServerOutlined,
  AppstoreOutlined, ApiOutlined, KeyOutlined, PieChartOutlined

PAGES TO DESIGN (14 routes):
1. Dashboard (/) -- KPI cards, engine distribution chart, recent instances table
2. Instance List (/instances) -- Filterable table with search, engine, status filters
3. Instance Create (/instances/new) -- 4-step wizard: Engine > Plan & Size > Config > Review
4. Instance Show (/instances/:id) -- Tabbed detail: Overview, Backups, Credentials
5. Instance Scale (/instances/:id/scale) -- Size preset selector with current vs new comparison
6. Backup List (/backups) -- Filterable table with type, status, encryption indicators
7. Backup Create (/backups/new) -- Form: instance select, type, retention, encryption toggle
8. Restore Create (/backups/restore) -- Form: source backup, optional target instance
9. Engine List (/engines) -- 8-engine catalog grid with versions, features, instance counts
10. Plugin List (/plugins) -- Filterable table with engine, status, capabilities
11. Plugin Create (/plugins/new) -- Form: name, engine, version, gRPC endpoint, capabilities
12. Credential List (/credentials) -- Rotation audit log table with rotate action dropdown
13. Quota Show (/quotas) -- Tier info, 4 progress gauges, detailed quota breakdown
14. Login (/login) -- Authentication page

ENGINE COLOR MAPPING:
- YugabyteDB: #FF6B35, ScyllaDB: #6C5CE7, DragonflyDB: #00B894
- MongoDB: #4DB33D, CouchDB: #E43525, ClickHouse: #FFCC00
- TimescaleDB: #FDB515, QuestDB: #4A90D9

INSTANCE STATUS COLORS:
- provisioning: #3b82f6, running: #10b981, scaling: #f59e0b
- backing_up: #8b5cf6, restoring: #f97316, failed: #ef4444
- decommissioning: #64748b, stopped: #94a3b8

SIZE PRESETS:
- S (Small): 500m CPU, 1Gi RAM
- M (Medium): 2 CPU, 8Gi RAM
- L (Large): 4 CPU, 16Gi RAM
- XL (Extra Large): 8 CPU, 32Gi RAM

PLANS: Starter, Professional, Enterprise
HA MODES: Standalone, Primary-Replica, Multi-Master
REGIONS: US East, US West, EU West, EU Central, AP Southeast, AP Northeast

DATA SOURCE: Hasura GraphQL with live subscriptions (graphql-ws).
MULTI-TENANT: tenant_id on all entities, X-Tenant-ID header.

Apply WCAG 2.1 AA to all screens. Touch targets >= 44px. Skeleton loading states.
Design for 1440px desktop, 1024px tablet, and 390px mobile breakpoints.
```

---

## 5. Figma Make Design-System Prompt (Component Library)

### Prompt F-001: Core Design Tokens -- DBaaS Theme

```
Create a Figma page titled "DBaaS Design Tokens" containing the complete token
specification for the Database-as-a-Service module.

COLOR PALETTE (Light Theme):
- Primary: #2563EB (Blue -- represents reliability and cloud infrastructure)
- Primary Hover: #1D4ED8
- Primary Light: #DBEAFE
- Secondary: #8B5CF6 (Purple -- used for backup/credential accents)
- Success: #10B981 (Running, Completed, Encrypted)
- Warning: #F59E0B (Scaling, High Usage)
- Danger: #EF4444 (Failed, Critical Quota, Unencrypted)
- Info: #3B82F6 (Provisioning, Informational)
- Neutral-50: #F5F7FA (Page background / layout body)
- Neutral-100: #FAFBFC (Table header background)
- Neutral-200: #F1F5F9 (Progress bar track, dividers)
- Neutral-500: #64748B (Secondary text, table headers)
- Neutral-600: #94A3B8 (Muted icons, placeholder text)
- Neutral-900: #111827 (Primary text)
- Surface: #FFFFFF (Cards, modals, inputs)
- Sidebar Dark: #001529 (Sidebar background)
- Sidebar Active: #2563EB (Active menu item background)
- Sidebar Hover: rgba(37, 99, 235, 0.6)

ENGINE BRAND COLORS (8 engines):
- YugabyteDB: #FF6B35 (Orange)
- ScyllaDB: #6C5CE7 (Deep Purple)
- DragonflyDB: #00B894 (Teal Green)
- MongoDB: #4DB33D (Forest Green)
- CouchDB: #E43525 (Crimson Red)
- ClickHouse: #FFCC00 (Golden Yellow)
- TimescaleDB: #FDB515 (Amber)
- QuestDB: #4A90D9 (Steel Blue)

TYPOGRAPHY (Plus Jakarta Sans):
- Display: 28px/36px, semibold 600 -- page titles ("Dashboard", "Instances")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Provision Instance")
- Heading 2: 20px/28px, semibold 600 -- card titles ("Recent Instances")
- Heading 3: 16px/24px, medium 500 -- subsection headers ("Select Engine")
- Heading 4: 14px/20px, semibold 600 -- small headers, tier display
- Body: 14px/20px, regular 400 -- paragraph text, table cells
- Body Small: 13px/18px, regular 400 -- secondary text, timestamps, subtitles
- Caption: 12px/16px, regular 400 -- helper text, version labels, resource specs
- Label: 14px/20px, medium 500 -- form labels, column headers
- Label Small: 11px/16px, semibold 600, uppercase -- section labels ("VERSIONS", "FEATURES")
- Mono: 12px/18px, system monospace -- endpoints, instance IDs, connection strings, ports

SPACING SCALE (4px base):
- 4px (xs), 8px (sm), 12px (md), 16px (lg), 24px (xl), 32px (2xl), 48px (3xl)
- Card padding: 24px (default), 16px (small cards), 20px (engine cards)
- Section gap: 16px (between cards in a row), 24px (between sections)
- Page margin: 32px (desktop), 16px (mobile)

ELEVATION / SHADOWS:
- Level 0: none (flat elements, bordered=false cards)
- Level 1: 0 1px 3px rgba(0,0,0,0.08) -- cards, dropdowns
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, popovers
- Level 3: 0 8px 24px rgba(0,0,0,0.12) -- dialogs, slide-overs

BORDER RADIUS:
- Small: 6px (small tags)
- Medium: 8px (buttons, inputs, selects, menu items, sidebar logo badge)
- Large: 10px (cards, tables, radio buttons, date pickers)
- Full: 9999px (status dots, circular progress)

GRID:
- Desktop (1440px): 12-column, 24px gutter, 260px sidebar, main content fluid
- Tablet (1024px): 12-column, 16px gutter, collapsed sidebar (72px)
- Mobile (390px): 4-column, 16px gutter, bottom navigation

CONTROL DIMENSIONS:
- Button height: 40px (default), 48px (large)
- Input height: 40px
- Select height: 40px
- Font weight (buttons): 500

Include token swatches, type scale samples, spacing ruler, shadow comparison,
engine color palette grid, and status color reference.
Light theme only (dark theme not currently implemented).
```

### Prompt F-002: Component Library -- DBaaS Components

```
Create a Figma page titled "DBaaS Component Library" with the following reusable
components, each showing all states (default, hover, active, disabled, error,
loading) and light theme.

ENGINE ICON COMPONENT:
- Circular or rounded-square container with engine brand color
- Sizes: 24px, 32px, 44px, 48px, 56px
- Contains engine logo/abbreviation in white
- Variants: standalone icon, EngineTag (icon + label in colored chip)
- Example: YugabyteDB icon (#FF6B35 background, "YB" text), ScyllaDB (#6C5CE7, "Sc")
- All 8 engines: YugabyteDB, ScyllaDB, DragonflyDB, MongoDB, CouchDB, ClickHouse,
  TimescaleDB, QuestDB

KPI CARD COMPONENT:
- 200px min-width, white background, 24px padding, border-radius 10px
- Icon in colored circle (36px), title (Body Small, secondary), value (Display size, bold)
- Loading state: skeleton shimmer replacing value and title
- Examples:
  -- DatabaseOutlined, "Total Instances", "24", color #2563eb
  -- CheckCircleOutlined, "Running", "19", color #10b981
  -- CloudServerOutlined, "Total Backups", "156", color #8b5cf6
  -- WarningOutlined, "Failed", "2", color #ef4444
  -- HddOutlined, "Storage Used", "480 GB", color #f59e0b
  -- AppstoreOutlined, "Active Engines", "6", color #3b82f6

STATUS BADGE COMPONENT:
- Rounded chip with colored dot + text label
- Status mappings:
  -- Instance: provisioning (blue), running (green), scaling (amber),
     backing_up (purple), restoring (orange), failed (red),
     decommissioning (slate), stopped (gray)
  -- Backup: pending (gray), in_progress (blue), completed (green), failed (red)
  -- Plugin: pending_validation (amber), validated (blue), active (green), rejected (red)
  -- Credential: initiated (blue), in_progress (amber), completed (green), failed (red)
  -- Plan: starter (gray), professional (blue), enterprise (gold)
  -- HA Mode: standalone (gray), primary-replica (blue), multi-master (purple)
  -- Backup Type: full (blue), incremental (green), snapshot (purple)

CONNECTION STRING COMPONENT:
- Full-width card with monospace text showing connection URI
- "Copy" icon button on the right that copies to clipboard
- Masked by default ("postgresql://****:****@host:port/db") with "Show" toggle
- Engine-specific URI format variants
- Example: "yugabytedb://admin:****@yb-cluster.us-east-1.dbaas.internal:5433/yugabyte"

PAGE HEADER COMPONENT:
- Breadcrumb trail (Dashboard > Instances > YugabyteDB)
- Title (Heading 1) with optional subtitle (Body Small, secondary)
- Right-aligned action buttons area
- Example: Breadcrumb, "Instances" title, "42 database instances" subtitle,
  "Provision Instance" primary button

ENGINE DISTRIBUTION BAR CHART (Dashboard Widget):
- Horizontal bar chart, each row: EngineTag + progress bar + count
- Progress bar: 8px height, #f1f5f9 track, #2563eb fill, border-radius 4px
- Bar width proportional to instance count relative to total
- Sorted by count descending
- Empty state: "No instances provisioned yet"

PROVISIONING WIZARD STEPPER:
- Ant Design Steps component with 4 steps
- Step icons: DatabaseOutlined, RocketOutlined, SettingOutlined, CheckCircleOutlined
- Step labels: "Engine", "Plan & Size", "Configuration", "Review"
- States: completed (green check), current (blue dot), upcoming (gray dot)
- Navigation: "Cancel"/"Previous" left, "Next"/"Provision Instance" right

QUOTA PROGRESS COMPONENT:
- Card with icon badge (36px, colored background), title, "current / max unit" text
- Full-width progress bar (12px height) with dynamic color:
  -- <= 70%: theme color (varies per metric)
  -- 71-90%: #f59e0b (warning)
  -- > 90%: #ef4444 (critical)
- Warning text appears at 80%+: "Warning: Usage is high"
- Critical text appears at 90%+: "Critical: Approaching limit!"
- Examples:
  -- Instances: 8/10 instances, #2563eb, 80% warning
  -- CPU: 24/32 cores, #8b5cf6
  -- Memory: 96/128 GB, #f59e0b
  -- Storage: 380/500 GB, #10b981

SIZE PRESET SELECTOR:
- Radio button group rendered as cards (4 across on desktop, 2 on mobile)
- Each card: size letter (22px, bold), label, CPU spec, RAM spec
- Selected state: blue border (#2563eb), slightly elevated
- Presets: S (500m/1Gi), M (2/8Gi), L (4/16Gi), XL (8/32Gi)

DATA TABLE (DBaaS variant):
- Ant Design table with sortable columns, horizontal scroll (x: 1100)
- Filter bar above: search input (260px) + filter dropdowns (160px each)
- Pagination: "42 instances" total, page size selector, page navigation
- Row hover: subtle background change, action icons visible
- Action column: icon buttons (view, scale, delete) with tooltips
- Empty state: "No instances found. Provision your first database!"
- Loading: skeleton rows

SIDEBAR NAVIGATION:
- Collapsible sidebar (expanded 260px, collapsed icon-only)
- Dark theme: #001529 background, white text, #2563eb active
- Logo area: 64px height, gradient badge + "ERP DBaaS" title
- 7 menu items with icons, active state with blue background + white text
- Border-bottom on logo area: rgba(255,255,255,0.08)
- Menu item border-radius: 8px

FORM COMPONENTS (DBaaS-specific):
- Instance select dropdown with EngineTag + truncated ID + region
- Backup type select: Full, Incremental, Snapshot
- Encryption toggle: Switch with "Encrypted"/"Unencrypted" labels
- Retention days number input (1-365 range)
- Region select with geographic labels (e.g., "US East (Virginia)")
- gRPC endpoint input with placeholder "grpc://plugin-service:50051"
- Capabilities multi-select tag mode: provision, scale, backup, restore, monitor,
  migrate, ha_failover, connection_pooling
```

---

## 6. Make Automation Prompt Pack

### Prompt M-001: Instance Provisioning Pipeline

```
Create a Make (Integromat) scenario titled "DBaaS -- Instance Provisioning Pipeline".

TRIGGER: Webhook from dbaas-api when a new service_instances record is created
with status = "provisioning".
Payload includes: instance_id, tenant_id, engine, version, plan, ha_mode,
region, resource_usage, config.

STEP 1 -- Validate Tenant Quota:
- Call dbaas-api: GET /v1/quotas/{tenant_id}
- Check: current_instances < max_instances
- Check: current_cpu + requested_cpu <= max_cpu
- Check: current_memory + requested_memory <= max_memory
- If any check fails -> update instance status to "failed" with quota_exceeded reason
  and notify tenant admin

STEP 2 -- Select Operator:
- Route to appropriate engine operator based on engine field:
  -- yugabytedb -> yugabytedb-operator gRPC endpoint
  -- scylladb -> scylladb-operator gRPC endpoint
  -- dragonflydb -> dragonflydb-operator gRPC endpoint
  -- mongodb -> mongodb-operator gRPC endpoint
  -- couchdb -> couchdb-operator gRPC endpoint
  -- clickhouse -> clickhouse-operator gRPC endpoint
  -- timescaledb -> timescaledb-operator gRPC endpoint
  -- questdb -> questdb-operator gRPC endpoint

STEP 3 -- Provision Infrastructure (via selected operator):
- Call operator: POST /v1/provision
  Body: { instance_id, version, plan, ha_mode, region, resource_usage, config }
- Operator creates Kubernetes resources (StatefulSet, Service, PVC, ConfigMap)
- Poll for readiness: GET /v1/instances/{id}/status every 10 seconds, max 10 minutes

STEP 4 -- Configure Networking:
- Generate endpoint DNS: {engine}-{short_id}.{region}.dbaas.internal
- Assign port based on engine (YugabyteDB: 5433, ScyllaDB: 9042, MongoDB: 27017, etc.)
- Update service_instances record with endpoint and port

STEP 5 -- Initialize Credentials:
- Generate initial admin credentials via secrets manager
- Store encrypted in credential vault
- Create credential_rotations record with status "completed"
- Generate connection string template

STEP 6 -- Update Quota Usage:
- Call dbaas-api: PATCH /v1/quotas/{tenant_id}
  Increment: current_instances, current_cpu, current_memory, current_storage

STEP 7 -- Finalize:
- Update instance status to "running"
- Publish Hasura event: instance.provisioned
- Notify tenant via in-app notification and email

ERROR HANDLING:
- Retry up to 3 times with exponential backoff on operator calls
- On final failure: set instance status to "failed", rollback quota reservation
- Alert platform-ops@company.com with error details and correlation_id
- Log all steps to audit trail via dbaas-gateway

ESTIMATED RUN TIME: 2-5 minutes (depending on engine and HA mode)
TRIGGER VOLUME: ~20-50/week per tenant
```

### Prompt M-002: Automated Backup Scheduling Pipeline

```
Create a Make scenario titled "DBaaS -- Automated Backup Scheduling".

TRIGGER: Scheduled -- every 6 hours at 00:00, 06:00, 12:00, 18:00 UTC.

STEP 1 -- Gather Running Instances:
- Call dbaas-api: GET /v1/instances?status=running
- For each instance, check backup_policy configuration:
  -- Frequency: 6h, 12h, 24h
  -- Last backup timestamp from backup_records
  -- If (now - last_backup) >= configured_frequency -> add to backup queue

STEP 2 -- Initiate Backups (Parallel, max 10 concurrent):
- For each instance in queue:
  -- Determine backup type: full (if no backup in 7 days), incremental (otherwise)
  -- Call dbaas-api: POST /v1/backups
    Body: { instance_id, type, retention_days: 30, encrypted: true }
  -- Update instance status to "backing_up"

STEP 3 -- Monitor Completion:
- Poll each backup record: GET /v1/backups/{id}/status every 30 seconds
- Timeout: 30 minutes per backup
- On completion: update instance status back to "running"
- On failure: retry once, then alert DBA team

STEP 4 -- Cleanup Expired Backups:
- Query backup_records where completed_at + retention_days < now
- For each expired backup:
  -- Delete storage artifacts via operator
  -- Soft-delete backup_records entry
  -- Log deletion to audit trail

STEP 5 -- Report:
- Generate daily backup summary: total backups, successes, failures, storage consumed
- Email report to platform engineering team
- Update dashboard metrics via Hasura event

ERROR HANDLING:
- Individual backup failures do not block others
- Failed backups create alert in credential_rotations audit log
- Storage cleanup failures are retried on next cycle

ESTIMATED RUN TIME: 5-15 minutes per cycle
```

### Prompt M-003: Credential Rotation Automation

```
Create a Make scenario titled "DBaaS -- Credential Rotation Automation".

TRIGGER: Webhook from dbaas-api when a credential_rotations record is created
with status = "initiated".
Payload: { rotation_id, instance_id, tenant_id, initiated_by }.

STEP 1 -- Fetch Instance Details:
- Call dbaas-api: GET /v1/instances/{instance_id}
- Verify instance status is "running" (reject rotation for non-running instances)
- Get engine type and current connection parameters

STEP 2 -- Generate New Credentials:
- Call secrets manager: POST /v1/secrets/generate
  Body: { instance_id, engine, strength: "strong" }
- Returns: { username, password, connection_string }
- Store in encrypted vault with version tag

STEP 3 -- Apply to Database Instance:
- Route to engine-specific operator:
  -- Call operator: POST /v1/credentials/rotate
    Body: { instance_id, new_username, new_password }
- Operator updates database user credentials atomically
- Verify new credentials work: test connection via operator health check

STEP 4 -- Update Application References:
- Publish Hasura event: credentials.rotated { instance_id, tenant_id, version }
- Applications subscribed via graphql-ws receive real-time notification
- Old credentials remain valid for grace period (5 minutes)

STEP 5 -- Finalize:
- Update credential_rotations record: status = "completed", completed_at = now()
- Invalidate old credentials after grace period
- Notify initiated_by user via in-app notification

ERROR HANDLING:
- If new credentials fail verification: rollback to old credentials
- Update rotation status to "failed" with error details
- Alert security team for manual intervention
- Never log plaintext credentials

ESTIMATED RUN TIME: 30-90 seconds
TRIGGER VOLUME: ~5-10/week per tenant
```

### Prompt M-004: Quota Threshold Alert Pipeline

```
Create a Make scenario titled "DBaaS -- Quota Threshold Alerting".

TRIGGER: Scheduled every 15 minutes.

STEP 1 -- Scan All Tenant Quotas:
- Call dbaas-api: GET /v1/quotas
- For each tenant, calculate utilization percentage for each resource:
  -- Instances: current_instances / max_instances
  -- CPU: current_cpu / max_cpu
  -- Memory: current_memory / max_memory
  -- Storage: current_storage / max_storage

STEP 2 -- Classify Alerts:
- Warning (70-89%): "Your {resource} usage is approaching the limit"
- Critical (90-99%): "Critical: {resource} usage at {percent}%"
- Exhausted (100%): "{resource} quota fully consumed. No new provisioning possible."
- Deduplicate: do not re-alert if same threshold was alerted in last 4 hours

STEP 3 -- Notify:
- Warning: In-app notification only
- Critical: In-app notification + email to tenant admin
- Exhausted: In-app notification + email to tenant admin + Slack/Teams webhook
  to platform-ops channel
- Include: current usage, maximum, recommendation to upgrade tier

STEP 4 -- Update Dashboard Indicators:
- Publish Hasura event: quota.threshold_crossed { tenant_id, resource, percent }
- UI components with live subscriptions auto-update progress bar colors and
  warning/critical text labels

STEP 5 -- Log:
- Record all alerts in audit trail for compliance
- Weekly quota utilization summary report

ESTIMATED RUN TIME: < 30 seconds
```

### Prompt M-005: Instance Health Check and Auto-Recovery

```
Create a Make scenario titled "DBaaS -- Instance Health Check".

TRIGGER: Scheduled every 5 minutes.

STEP 1 -- Gather Active Instances:
- Call dbaas-api: GET /v1/instances?status=running
- Also include status=scaling, status=restoring for monitoring

STEP 2 -- Health Check (Parallel, max 20 concurrent):
- For each instance, call the appropriate operator:
  -- GET /v1/instances/{id}/health
  -- Checks: process running, accepting connections, replication lag (if HA),
     disk space, memory pressure
- Classify: healthy, degraded, unreachable

STEP 3 -- Auto-Recovery for Degraded/Unreachable:
- Degraded: Log warning, attempt restart via operator POST /v1/instances/{id}/restart
- Unreachable (first occurrence): Mark as "under investigation", retry in 5 minutes
- Unreachable (3 consecutive failures): Update status to "failed",
  alert platform engineering team

STEP 4 -- Metrics Collection:
- Collect from each healthy instance: CPU usage, memory usage, storage usage,
  active connections, replication lag
- Update service_instances.resource_usage via dbaas-api PATCH
- Feed into dashboard KPI cards and instance detail views

STEP 5 -- Alert On SLA Violations:
- If any instance has been "failed" > 15 minutes: escalate to on-call
- If replication lag > 30 seconds on HA instances: warn DBA team
- If storage usage > 85%: recommend scaling or cleanup

ERROR HANDLING:
- Health check timeouts (10 seconds) treated as "unreachable"
- Do not auto-restart more than 3 times in 1 hour
- All actions logged to audit trail

ESTIMATED RUN TIME: 30-60 seconds per cycle
```

---

## 7. Page-by-Page Figma Prompts

### Prompt F-003: DBaaS Dashboard -- Command Center (1440px)

```
Design a desktop DBaaS Dashboard page at 1440x900px (scrollable) for the
ERP-DBaaS module. This is the primary landing page for Platform Engineers
and DBAs.

LAYOUT:
- Left: Collapsible sidebar navigation (260px, expanded by default, dark #001529)
- Top: Horizontal header bar with breadcrumb, notification bell, user avatar
- Main content area: fluid width, 32px padding

HEADER:
- PageHeader component: title "Dashboard", subtitle "Welcome back! Here's an
  overview of your database infrastructure."
- No action buttons on dashboard header

CONTENT -- ROW 1 (KPI Cards, 6-column grid, xs:24 sm:12 lg:4):
- Card 1: "Total Instances" -- 24 -- DatabaseOutlined icon, color #2563eb
- Card 2: "Running" -- 19 -- CheckCircleOutlined icon, color #10b981
- Card 3: "Total Backups" -- 156 -- CloudServerOutlined icon, color #8b5cf6
- Card 4: "Failed" -- 2 -- WarningOutlined icon, color #ef4444
- Card 5: "Storage Used" -- "480 GB" -- HddOutlined icon, color #f59e0b
- Card 6: "Active Engines" -- 6 -- AppstoreOutlined icon, color #3b82f6
All cards have loading skeleton state. White background, 10px border-radius.

CONTENT -- ROW 2 (Two-column, 8/16 split):
- Left (lg:8): "Engine Distribution" card with AppstoreOutlined icon in title.
  Horizontal bar chart showing instance count per engine, sorted descending:
  -- YugabyteDB: [======] 8
  -- MongoDB: [====] 5
  -- ScyllaDB: [===] 4
  -- ClickHouse: [===] 3
  -- TimescaleDB: [==] 2
  -- DragonflyDB: [=] 1
  -- QuestDB: [=] 1
  Each row: EngineTag (colored chip with icon + label) + progress bar
  (#2563eb fill, #f1f5f9 track, 8px height, rounded) + count number.
  Empty state: "No instances provisioned yet" centered text.

- Right (lg:16): "Recent Instances" card with DatabaseOutlined icon in title.
  "View All" link with ArrowRightOutlined in card extra area.
  Table with columns: Engine (EngineTag), Version (text), Plan (StatusBadge),
  Status (StatusBadge), Region (secondary text), Created (relative time),
  Arrow link to detail.
  8 most recent rows. No pagination. Size "middle".
  Empty state: "No instances found. Provision your first database!"

  Example rows:
  | YugabyteDB | 2.20 | Professional | running       | us-east-1  | 2 hours ago |
  | ScyllaDB   | 5.4  | Enterprise   | running       | eu-west-1  | 5 hours ago |
  | MongoDB    | 7.0  | Starter      | provisioning  | us-west-2  | 12 min ago  |
  | ClickHouse | 24.1 | Professional | backing_up    | eu-central | 1 day ago   |
  | DragonflyDB| 1.14 | Starter      | running       | ap-southeast| 2 days ago |
  | TimescaleDB| 2.14 | Professional | scaling       | us-east-1  | 3 days ago  |
  | QuestDB    | 7.4  | Starter      | running       | ap-northeast| 4 days ago |
  | CouchDB    | 3.3  | Enterprise   | failed        | eu-west-1  | 5 days ago  |

STATES:
- Loading: Skeleton shimmer on all KPI cards and table
- Empty: "Welcome to DBaaS! Provision your first database instance to get started."
  with "Provision Instance" CTA button linking to /instances/new
- Error: Inline error card with retry for failed data fetches

Include light theme only. Annotate all interactive elements with click targets >= 44px.
```

### Prompt F-004: Instance List Page (1440px)

```
Design a desktop Instance List page at 1440x900px for the dbaas service_instances
resource. Target user: Platform Engineer.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Instances"
- Title: "Instances"
- Subtitle: "42 database instances"
- Action: "Provision Instance" primary button with PlusOutlined icon

FILTER BAR (inside white Card, above table):
- Search input (260px): SearchOutlined prefix, placeholder "Search instances...",
  allowClear. Searches engine, endpoint, region, namespace.
- Engine dropdown (160px): All 8 engine options. Clearable.
- Status dropdown (160px): provisioning, running, scaling, backing_up, restoring,
  failed, decommissioning, stopped. Clearable.
- Filters are applied client-side for search, server-side for dropdowns.

TABLE (Ant Design, horizontal scroll at x:1100):
Columns:
| Engine (160px) | Version (90px) | Plan (120px) | HA Mode (140px) | Status (140px) | Region | Endpoint | Created (120px, sortable) | Actions (120px) |

Engine: EngineTag component (colored chip with icon + label)
Version: plain text, 13px
Plan: StatusBadge (starter=gray, professional=blue, enterprise=gold)
HA Mode: StatusBadge (standalone=gray, primary-replica=blue, multi-master=purple)
Status: StatusBadge with animated dot for active states
Region: secondary text, 13px (e.g., "us-east-1")
Endpoint: monospace code text, 12px, ellipsis overflow (e.g., "yb-a1b2c3d4.us-east-1.dbaas.internal")
  -- Shows "Pending..." for provisioning instances
Created: relative time format ("2 hours ago"), sortable column
Actions: 3 icon buttons:
  -- EyeOutlined (View detail) -> /instances/{id}
  -- ExpandOutlined (Scale) -> /instances/{id}/scale
  -- DeleteOutlined (Delete, danger red) -> confirmation dialog

Example rows:
| YugabyteDB  | 2.20 | Professional | multi-master    | running      | us-east-1    | yb-a1b2c3d4.us-east-1... | 2 hours ago | [eye] [scale] [del] |
| ScyllaDB    | 5.4  | Enterprise   | primary-replica | running      | eu-west-1    | sc-e5f6g7h8.eu-west-1... | 1 day ago   | [eye] [scale] [del] |
| MongoDB     | 7.0  | Starter      | standalone      | provisioning | us-west-2    | Pending...               | 12 min ago  | [eye] [scale] [del] |
| DragonflyDB | 1.14 | Starter      | standalone      | failed       | ap-southeast | Pending...               | 3 hours ago | [eye] [scale] [del] |

Pagination: "42 instances" total display, page size selector (10/20/50),
page number navigation. Default 20 per page.

DELETE CONFIRMATION: Modal with "Type '{engine_name}' to confirm deletion"
input field. "Delete" danger button disabled until correct text entered.

STATES:
- Loading: skeleton rows
- Empty with no filters: "No instances yet. Provision your first database!" + CTA
- Empty with filters: "No instances match your filters." + "Clear filters" link
```

### Prompt F-005: Instance Create -- Provisioning Wizard (1440px)

```
Design a desktop Instance Provisioning Wizard at 1440x900px (scrollable).
This is a 4-step wizard inside a single Card component.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Instances > New Instance"
- Title: "Provision Instance"
- Subtitle: "Set up a new database instance"
- Action: "Back" button with ArrowLeftOutlined -> /instances

WIZARD CONTAINER (white Card, bordered=false):
- Steps component at top with 4 steps, 32px margin-bottom:
  Step 1: DatabaseOutlined "Engine"
  Step 2: RocketOutlined "Plan & Size"
  Step 3: SettingOutlined "Configuration"
  Step 4: CheckCircleOutlined "Review"
- Content area: min-height 300px
- Footer: 32px margin-top, 1px solid #f0f0f0 top border,
  flex between "Cancel/Previous" (left) and "Next/Provision Instance" (right)

STEP 1 -- SELECT ENGINE:
- Title: "Select Database Engine" (Heading 5)
- 8 engine cards in 4x2 grid (xs:24 sm:12 md:6):
  Each card: 16px body padding, center-aligned
  - EngineIcon (48px)
  - Engine name (14px, bold)
  - Description (11px, secondary, 2-line ellipsis)
  - Selected state: brand-color border, CheckCircleOutlined (top-right, 18px, brand color)
  - Hover: subtle elevation increase, pointer cursor

  Cards:
  1. YugabyteDB -- "Distributed SQL database for cloud-native..."
  2. ScyllaDB -- "High-performance NoSQL database compatible with..."
  3. DragonflyDB -- "Modern in-memory datastore, fully compatible..."
  4. MongoDB -- "General-purpose document database designed for..."
  5. CouchDB -- "Seamless multi-master sync database with..."
  6. ClickHouse -- "Column-oriented database management system..."
  7. TimescaleDB -- "Time-series database built on PostgreSQL..."
  8. QuestDB -- "High-performance time-series database with..."

- When engine selected, detail card appears below (16px margin-top, small size):
  Left side: "Features:" label + blue Tag chips for each feature
  Right side: "Version:" label + Select dropdown with version options (v2.20, v2.18, etc.)
  Version auto-selects latest on engine change.

- Next button disabled until engine AND version selected.

STEP 2 -- PLAN & SIZE:
- Title: "Select Plan" (Heading 5)
- 3 plan cards in a row (xs:24 sm:8):
  Each card: plan name (Heading 5, colored), description (13px, secondary)
  Selected: blue border, elevated
  -- Starter: basic plan description, neutral color
  -- Professional: recommended plan, blue accent
  -- Enterprise: premium features, gold accent

- Title: "Select Size" (Heading 5, 24px margin-top)
- Radio button group as 4 cards (xs:24 sm:12 md:6):
  Each radio button card: 16px padding, 10px border-radius
  -- S: "Small" / CPU: 500m / RAM: 1Gi
  -- M: "Medium" / CPU: 2 / RAM: 8Gi
  -- L: "Large" / CPU: 4 / RAM: 16Gi
  -- XL: "Extra Large" / CPU: 8 / RAM: 32Gi
  Selected: blue border (#2563eb)
  Size letter: 18px bold, Label: 14px medium 500, Specs: 12px secondary

STEP 3 -- CONFIGURATION:
- Title: "High Availability Mode" (Heading 5)
- 3 HA mode cards (xs:24 sm:8):
  -- Standalone: single node, simplest setup
  -- Primary-Replica: read replicas, failover support
  -- Multi-Master: distributed writes, highest availability
  Selected: blue border

- Title: "Region" (Heading 5, 24px margin-top)
- Select dropdown (100% width, max-width 400px, size large):
  -- US East (Virginia) = us-east-1
  -- US West (Oregon) = us-west-2
  -- EU West (Ireland) = eu-west-1
  -- EU Central (Frankfurt) = eu-central-1
  -- AP Southeast (Singapore) = ap-southeast-1
  -- AP Northeast (Tokyo) = ap-northeast-1

STEP 4 -- REVIEW:
- Title: "Review Configuration" (Heading 5)
- Info Alert: "Instance provisioning will begin immediately after confirmation."
  type=info, showIcon, 16px margin-bottom
- Descriptions component (bordered, 2 columns on sm+, 1 on xs):
  -- Engine: EngineIcon (24px) + Engine name (bold)
  -- Version: blue Tag "v2.20"
  -- Plan: purple Tag "professional"
  -- Size: "M - Medium" bold + "CPU: 2 | RAM: 8Gi" secondary text
  -- HA Mode: Tag "primary-replica"
  -- Region: "US East (Virginia)"
- Action button: "Provision Instance" with RocketOutlined icon, primary, large
  Loading state during creation.

NAVIGATION FOOTER:
- Step 0: "Cancel" (left) + "Next" (right, primary, disabled if no engine/version)
- Steps 1-2: "Previous" (left) + "Next" (right, primary, disabled if incomplete)
- Step 3: "Previous" (left) + "Provision Instance" (right, primary, loading)
```

### Prompt F-006: Instance Show -- Detail Page (1440px)

```
Design a desktop Instance Detail page at 1440x900px for viewing a single
database instance with all its associated data.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Instances > YugabyteDB (a1b2c3d4)"
- No title text (breadcrumb serves as title)
- Actions: "Back" button + "Scale" button with ExpandOutlined icon

INSTANCE HEADER CARD (white, bordered=false, 16px margin-bottom):
- EngineIcon (56px) + content block:
  -- Title (Heading 4): "YugabyteDB" + Tag "v2.20" (blue)
  -- Subtitle: "us-east-1 | professional plan | primary-replica"
  -- Status line: StatusBadge "running" (green) + "Created: 2 hours ago" (12px, secondary)

TABBED CONTENT CARD (white, bordered=false):
Three tabs: "Overview" | "Backups (12)" | "Credentials (3)"

TAB 1 -- OVERVIEW:
Two-column layout (lg: 16 left, 8 right):

Left Column:
- "Instance Details" card (small size, bordered=false):
  Descriptions component (2 columns on sm+, small size):
  -- Engine: EngineTag "YugabyteDB"
  -- Version: Tag "v2.20" (blue)
  -- Plan: StatusBadge "professional"
  -- HA Mode: StatusBadge "primary-replica"
  -- Region: "us-east-1"
  -- Namespace: "dbaas-prod-yb-a1b2c3d4"
  -- Endpoint: monospace code "yb-a1b2c3d4.us-east-1.dbaas.internal"
  -- Port: monospace code "5433"
  -- Created: "2026-02-24 10:30:00 UTC"
  -- Updated: "2 hours ago"

- Connection String card (16px margin-top, small size, bordered=false):
  ConnectionString component showing masked URI:
  "yugabytedb://admin:****@yb-a1b2c3d4.us-east-1.dbaas.internal:5433/yugabyte"
  With "Copy" and "Show/Hide" toggle buttons.

Right Column (vertical stack, 16px gap):
- CPU Cores card: Statistic component, value "2", suffix "cores", color #2563eb
- Memory card: Statistic component, value "8", suffix "GB", color #8b5cf6
- Storage card: Statistic component, value "45", suffix "GB", color #f59e0b
- Connections card: Statistic component, value "127", color #10b981

TAB 2 -- BACKUPS:
- "Create Backup" primary button with CloudServerOutlined icon (links to
  /backups/new?instanceId={id}), 16px margin-bottom
- Table with columns: Type (StatusBadge), Status (StatusBadge), Size (formatted bytes),
  Encrypted (green Tag "Yes" or default Tag "No"), Created (relative time)
- No pagination, middle size
- Empty state: "No backups for this instance"

Example rows:
| full        | completed   | 2.4 GB  | Yes (green, lock icon) | 1 day ago   |
| incremental | completed   | 340 MB  | Yes                    | 6 hours ago |
| snapshot    | in_progress | -       | Yes                    | 5 min ago   |

TAB 3 -- CREDENTIALS:
- "Rotate Credentials" primary button with KeyOutlined icon (links to /credentials),
  16px margin-bottom
- Table with columns: Status (StatusBadge), Initiated By (text), Initiated At
  (datetime), Completed At (datetime or "-")
- Empty state: "No credential rotations"

Example rows:
| completed  | admin   | 2026-02-20 08:00 | 2026-02-20 08:01 |
| completed  | admin   | 2026-01-15 14:30 | 2026-01-15 14:31 |
| failed     | devops  | 2026-01-10 09:00 | -                |

STATES:
- Loading: header card skeleton + tabs skeleton
- Not found: Empty component "Instance not found"
- Error: inline error with retry
```

### Prompt F-007: Instance Scale Page (1440px)

```
Design a desktop Instance Scale page at 1440x900px for adjusting compute
resources of an existing database instance.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Instances > YugabyteDB > Scale"
- Title: "Scale Instance"
- Subtitle: "Adjust compute resources for your database instance"
- Action: "Back" button with ArrowLeftOutlined -> /instances/{id}

CURRENT INSTANCE INFO CARD (white, bordered=false, 16px margin-bottom):
- EngineIcon (48px) + content block:
  -- Title (Heading 5): "YugabyteDB" + Tag "v2.20" (blue)
  -- Status line: StatusBadge "running" + "Current: CPU 2 cores | Memory 8 GB" (13px, secondary)

SCALING CARD (white, bordered=false):
- Warning Alert: "Scaling will cause a brief disruption while resources are reallocated."
  type=warning, showIcon, 24px margin-bottom

- Title: "Select New Size" (Heading 5, 16px margin-bottom)
- Radio button group as 4 cards (xs:24 sm:12 md:6):
  Each card: 20px padding, 10px border-radius, center-aligned
  -- S: "Small" / CPU: 500m / RAM: 1Gi
  -- M: "Medium" / CPU: 2 / RAM: 8Gi (current -- indicated with "Current" badge)
  -- L: "Large" / CPU: 4 / RAM: 16Gi
  -- XL: "Extra Large" / CPU: 8 / RAM: 32Gi
  Size letter: 22px bold
  Label: 14px medium 500
  Specs: 12px secondary, 4px margin-top between CPU and RAM lines
  Selected: blue border (#2563eb)

FOOTER (32px margin-top, 1px top border, flex end, 12px gap):
- "Cancel" button (large) -> /instances/{id}
- "Apply Scaling" primary button (large) with ExpandOutlined icon
  Loading state during mutation. Disabled if selected size equals current size.

SUCCESS: message.success("Scaling operation initiated!") -> navigate to /instances/{id}
ERROR: message.error("Failed to scale instance")
```

### Prompt F-008: Backup List Page (1440px)

```
Design a desktop Backup List page at 1440x900px for the dbaas backup_records
resource. Target user: DBA.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Backups"
- Title: "Backups"
- Subtitle: "156 backup records"
- Action: "Create Backup" primary button with PlusOutlined icon -> /backups/new

FILTER BAR (inside white Card):
- Search input (260px): SearchOutlined prefix, placeholder "Search by instance ID...",
  allowClear
- Type dropdown (140px): Full, Incremental, Snapshot. Clearable.
- Status dropdown (140px): Pending, In Progress, Completed, Failed. Clearable.

TABLE (horizontal scroll x:1100):
Columns:
| Instance ID (120px) | Type (120px) | Status (130px) | Size (100px) | Encrypted (130px) | Retention (100px) | Created (130px, sortable) | Completed (130px) | Actions (100px) |

Instance ID: monospace code text, truncated "a1b2c3d4..."
Type: StatusBadge (full=blue, incremental=green, snapshot=purple)
Status: StatusBadge (pending=gray, in_progress=blue, completed=green, failed=red)
Size: formatted bytes (e.g., "2.4 GB", "340 MB")
Encrypted: green Tag with LockOutlined "Encrypted" or default Tag "Unencrypted"
Retention: "30 days"
Created: relative time, sortable
Completed: relative time or "-"
Actions: ReloadOutlined restore button -> /backups/restore?backupId={id}

Example rows:
| a1b2c3d4... | full        | completed   | 2.4 GB | Encrypted   | 30 days | 1 day ago   | 23 hours ago | [restore] |
| e5f6g7h8... | incremental | completed   | 340 MB | Encrypted   | 14 days | 6 hours ago | 5 hours ago  | [restore] |
| i9j0k1l2... | snapshot    | in_progress | -      | Encrypted   | 7 days  | 5 min ago   | -            | [restore] |
| m3n4o5p6... | full        | failed      | -      | Unencrypted | 30 days | 2 days ago  | -            | [restore] |

Pagination: "156 backups" total, page size selector, page navigation. Default 20.
```

### Prompt F-009: Backup Create Page (1440px)

```
Design a desktop Backup Create page at 1440x900px for initiating a new backup.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Backups > New Backup"
- Title: "Create Backup"
- Action: "Back" button -> /backups

FORM CARD (white, bordered=false, max-width 600px):
Form layout: vertical, requiredMark="optional"

Fields:
1. Instance (required):
   Select dropdown, showSearch, optionFilterProp="label"
   Options: running instances only, formatted as:
   "YugabyteDB - a1b2c3d4... (us-east-1)"
   "ScyllaDB - e5f6g7h8... (eu-west-1)"
   Pre-selected if query param instanceId is present.

2. Backup Type + Retention (2-column row, gutter 16):
   Left (sm:12): Backup Type select (required):
   -- Full, Incremental, Snapshot
   Default: "full"
   Right (sm:12): Retention (days) number input (required):
   min 1, max 365, default 30

3. Encrypt Backup:
   Switch component with "Encrypted"/"Unencrypted" labels
   Default: on (true)

ACTIONS (16px margin-top):
- "Start Backup" primary button with CloudServerOutlined icon, large, loading state
- "Cancel" button, large -> /backups

SUCCESS: message.success("Backup initiated successfully!") -> /backups
ERROR: message.error("Failed to initiate backup")
```

### Prompt F-010: Restore Create Page (1440px)

```
Design a desktop Restore page at 1440x900px for restoring from a backup.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Backups > Restore"
- Title: "Restore from Backup"
- Action: "Back" button -> /backups

FORM CARD (white, bordered=false, max-width 600px):
- Warning Alert at top: "Restoring from a backup will create a new instance or
  overwrite the target instance with backup data."
  type=warning, showIcon, 24px margin-bottom

Form layout: vertical, requiredMark="optional"

Fields:
1. Source Backup (required):
   Select dropdown, showSearch, optionFilterProp="label"
   Only completed backups shown. Formatted as:
   "full backup - a1b2c3d4... (2.4 GB, 1 day ago)"
   "incremental backup - e5f6g7h8... (340 MB, 6 hours ago)"
   Pre-selected if query param backupId is present.

2. Target Instance (optional):
   Select dropdown, showSearch, allowClear, optionFilterProp="label"
   Placeholder: "Select target instance (or leave blank for new)"
   All instances shown with status. Formatted as:
   "YugabyteDB - a1b2c3d4... (running)"
   "ScyllaDB - e5f6g7h8... (stopped)"
   Help text: "Leave blank to create a new instance from the backup."

ACTIONS (16px margin-top):
- "Start Restore" primary button with ReloadOutlined icon, large, loading state
- "Cancel" button, large -> /backups

SUCCESS: message.success("Restore operation initiated!") -> /instances
ERROR: message.error("Failed to initiate restore")
```

### Prompt F-011: Engine Catalog Page (1440px)

```
Design a desktop Engine Catalog page at 1440x900px showing all 8 supported
database engines as an informational grid.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Engines"
- Title: "Database Engines"
- Subtitle: "Supported database engines and their capabilities"
- No action buttons

ENGINE GRID (4x2 on lg, 2x4 on sm, 1x8 on xs):
- Row gutter: [16, 16]
- Each engine card: white, bordered=false, full height, 3px colored top border
  using engine brand color, 20px body padding

Card content (vertical stack, 12px gap):
1. Header row (space-between):
   - EngineIcon (44px)
   - Instance count Tag (blue): "8 instances" (only if > 0)

2. Name + Description:
   - Engine name (Heading 5, margin 0)
   - Description (13px, secondary, 2-line ellipsis)

3. Versions section:
   - "VERSIONS" label (11px, secondary, semibold, uppercase)
   - Wrap of version Tags: "v2.20" "v2.18" "v2.16" (11px font)

4. Features section:
   - "FEATURES" label (11px, secondary, semibold, uppercase)
   - Vertical list of features, each with CheckCircleOutlined (12px, engine color) + text (12px)

ENGINE CARDS (all 8):

1. YugabyteDB (top border: #FF6B35):
   "Distributed SQL database for cloud-native applications with PostgreSQL compatibility."
   Versions: v2.20, v2.18, v2.16
   Features: Distributed SQL, ACID Transactions, Horizontal Scaling, PostgreSQL Compatible

2. ScyllaDB (top border: #6C5CE7):
   "High-performance NoSQL database compatible with Apache Cassandra."
   Versions: v5.4, v5.2, v5.1
   Features: Low Latency, High Throughput, CQL Compatible, Shard-per-Core

3. DragonflyDB (top border: #00B894):
   "Modern in-memory datastore, fully compatible with Redis and Memcached APIs."
   Versions: v1.14, v1.12, v1.10
   Features: Redis Compatible, In-Memory, Multi-Threaded, Snapshots

4. MongoDB (top border: #4DB33D):
   "General-purpose document database designed for ease of development and scaling."
   Versions: v7.0, v6.0, v5.0
   Features: Document Store, Flexible Schema, Aggregation, Replication

5. CouchDB (top border: #E43525):
   "Seamless multi-master sync database with a document-oriented NoSQL approach."
   Versions: v3.3, v3.2, v3.1
   Features: Multi-Master Replication, RESTful API, MVCC, MapReduce

6. ClickHouse (top border: #FFCC00):
   "Column-oriented database management system for online analytical processing."
   Versions: v24.1, v23.12, v23.8
   Features: Columnar Storage, Real-Time Analytics, SQL Support, Compression

7. TimescaleDB (top border: #FDB515):
   "Time-series database built on PostgreSQL for fast ingest and complex queries."
   Versions: v2.14, v2.12, v2.10
   Features: Time-Series, PostgreSQL Extension, Continuous Aggregates, Compression

8. QuestDB (top border: #4A90D9):
   "High-performance time-series database with SQL support and fast ingestion."
   Versions: v7.4, v7.3, v7.2
   Features: Time-Series, SQL Support, High Ingest Rate, Column-Oriented
```

### Prompt F-012: Plugin List Page (1440px)

```
Design a desktop Plugin List page at 1440x900px for the dbaas
plugin_registrations resource.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Plugins"
- Title: "Plugins"
- Subtitle: "18 registered plugins"
- Action: "Register Plugin" primary button with PlusOutlined icon -> /plugins/new

FILTER BAR (inside white Card):
- Search input (260px): placeholder "Search plugins...", searches name and gRPC endpoint
- Engine dropdown (160px): All 8 engine options. Clearable.
- Status dropdown (160px): Pending Validation, Validated, Active, Rejected. Clearable.

TABLE (horizontal scroll x:900):
Columns:
| Name | Engine (160px) | Version (90px) | gRPC Endpoint | Status (160px) | Capabilities | Registered (130px) |

Name: bold text
Engine: EngineTag component
Version: Tag "v2.20"
gRPC Endpoint: monospace code text, 12px, ellipsis (e.g., "grpc://yugabyte-op:50051")
Status: StatusBadge (pending_validation=amber, validated=blue, active=green, rejected=red)
Capabilities: Tag chips showing first 3, "+N" overflow Tag if more than 3
  -- Examples: "provision", "scale", "backup", "restore", "monitor"
Registered: relative time

Example rows:
| yugabyte-operator  | YugabyteDB | v2.20 | grpc://yb-op:50051   | active              | provision, scale, backup +2 | 30 days ago |
| scylla-operator    | ScyllaDB   | v5.4  | grpc://sc-op:50051   | active              | provision, scale, backup    | 28 days ago |
| dragonfly-operator | DragonflyDB| v1.14 | grpc://df-op:50051   | validated           | provision, scale            | 7 days ago  |
| custom-mongo-ext   | MongoDB    | v7.0  | grpc://mongo-ext:50052| pending_validation | monitor, migrate            | 1 day ago   |
| rejected-plugin    | CouchDB    | v3.3  | grpc://bad-plugin:50053| rejected           | provision                  | 3 days ago  |

Pagination: "18 plugins" total, default 20 per page.
```

### Prompt F-013: Plugin Create Page (1440px)

```
Design a desktop Plugin Registration page at 1440x900px for registering a new
database operator plugin.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Plugins > Register Plugin"
- Title: "Register Plugin"
- Action: "Back" button -> /plugins

FORM CARD (white, bordered=false, max-width 600px):
Form layout: vertical, requiredMark="optional"

Fields:
1. Plugin Name (required):
   Input with ApiOutlined prefix icon
   Placeholder: "my-database-plugin"
   Validation: required, min 3 characters

2. Engine + Version (2-column row, gutter 16):
   Left (sm:12): Engine select (required):
   All 8 engine options.
   Right (sm:12): Version select (required):
   Dynamically populated based on selected engine.
   Disabled until engine is selected.
   Options formatted as "v2.20", "v2.18", etc.

3. gRPC Endpoint (required):
   Input with placeholder "grpc://plugin-service:50051"
   Validation: required, must start with "grpc://"

4. Capabilities (optional):
   Multi-select in "tags" mode with predefined options:
   Provision, Scale, Backup, Restore, Monitor, Migrate, HA Failover, Connection Pooling
   Users can also type custom capability names and press enter.
   Placeholder: "Add capabilities (press enter to create)"

ACTIONS (16px margin-top):
- "Register Plugin" primary button with SaveOutlined icon, large, loading state
- "Cancel" button, large -> /plugins

SUCCESS: message.success("Plugin registered successfully!") -> /plugins
ERROR: message.error("Failed to register plugin")

NOTE: Plugin starts in "pending_validation" status. Backend validates the gRPC
endpoint is reachable before transitioning to "validated" then "active".
```

### Prompt F-014: Credential List Page (1440px)

```
Design a desktop Credential Rotation page at 1440x900px for the dbaas
credential_rotations resource.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Credentials"
- Title: "Credentials"
- Subtitle: "34 credential rotation records"
- Action: Instance select dropdown (320px) for initiating rotation:
  Placeholder "Rotate credentials for..."
  Options: running instances formatted as "YugabyteDB - a1b2c3d4..."
  On selection: immediately creates a new credential_rotations record
  with status "initiated" and triggers rotation automation.
  Loading state while mutation is in progress.

FILTER BAR (inside white Card):
- Search input (260px): placeholder "Search by instance ID...",
  searches instanceId and initiatedBy
- Status dropdown (160px): Initiated, In Progress, Completed, Failed. Clearable.

TABLE (horizontal scroll x:800):
Columns:
| Instance ID (130px) | Status (140px) | Initiated By | Initiated At | Completed At |

Instance ID: monospace code text, truncated "a1b2c3d4..."
Status: StatusBadge (initiated=blue, in_progress=amber, completed=green, failed=red)
Initiated By: plain text (e.g., "admin", "devops", "system")
Initiated At: full datetime format (e.g., "2026-02-24 10:30:00")
Completed At: full datetime or "-"

Example rows:
| a1b2c3d4... | completed   | admin  | 2026-02-24 10:30 | 2026-02-24 10:31 |
| e5f6g7h8... | in_progress | devops | 2026-02-24 09:15 | -                |
| i9j0k1l2... | completed   | admin  | 2026-02-20 14:00 | 2026-02-20 14:01 |
| m3n4o5p6... | failed      | system | 2026-02-18 08:00 | -                |

Pagination: "34 rotations" total, default 20 per page.

SECURITY NOTE: No credential values (passwords, connection strings) are displayed
on this page. This is purely an audit log of rotation events.
```

### Prompt F-015: Quota Show Page (1440px)

```
Design a desktop Quota Usage page at 1440x900px for displaying tenant resource
quotas and current utilization.

HEADER:
- PageHeader: Breadcrumb "Dashboard > Quotas"
- Title: "Quota Usage"
- Subtitle: "Monitor your tenant resource quotas and usage"
- No action buttons

TIER INFO CARD (white, bordered=false, 16px margin-bottom):
Row layout with 24px gutter, vertically centered:
- Col 1: "CURRENT TIER" label (12px, secondary, semibold, uppercase)
  "Tier B" (Heading 2, #2563eb, margin 0) + Tag "Professional" (blue)
- Col 2: Statistic: "Rate Limit" title, "500" value, "req/min" suffix, #2563eb color
- Col 3: Statistic: "Tenant ID" title, "tenant-acme-corp" value, 16px font size

QUOTA PROGRESS CARDS (2x2 grid, xs:24 sm:12):
Each card: white, bordered=false
- Header row: icon badge (36px, rounded 8px, 15% opacity background) + title (14px, bold)
  + "current / max unit" text (13px, secondary) right-aligned
- Progress bar (12px height, full width):
  -- Track: #f1f5f9
  -- Fill color: normal = metric color, 71-90% = #f59e0b, >90% = #ef4444
  -- Rounded ends
- Warning text (conditional):
  -- 80-90%: "Warning: Usage is high" (12px, #f59e0b, medium weight)
  -- >90%: "Critical: Approaching limit!" (12px, #ef4444, medium weight)

Cards:
1. Instances: DatabaseOutlined, color #2563eb
   "8 / 10 instances" -> 80% -> warning amber
2. CPU: ThunderboltOutlined, color #8b5cf6
   "24 / 32 cores" -> 75% -> purple fill
3. Memory: CloudOutlined, color #f59e0b
   "96 / 128 GB" -> 75% -> amber fill
4. Storage: HddOutlined, color #10b981
   "380 / 500 GB" -> 76% -> green fill

QUOTA DETAILS CARD (white, bordered=false, 16px margin-top):
Title: "Quota Details"
Descriptions component (bordered, 3 columns on md+, 2 on sm, 1 on xs):
| Max Instances   | Current Instances | Instance Utilization (Progress bar, 120px) |
| Max CPU         | Current CPU       | CPU Utilization (Progress bar)              |
| Max Memory      | Current Memory    | Memory Utilization (Progress bar)           |
| Max Storage     | Current Storage   | Storage Utilization (Progress bar)          |
| Rate Limit      | Tier              |                                             |

Values:
- Max Instances: 10, Current: 8, Utilization: 80%
- Max CPU: 32 cores, Current: 24 cores, Utilization: 75%
- Max Memory: 128 GB, Current: 96 GB, Utilization: 75%
- Max Storage: 500 GB, Current: 380 GB, Utilization: 76%
- Rate Limit: 500 requests/min
- Tier: Tag "Tier B" (blue)

STATES:
- Loading: skeleton on tier card + all quota progress cards + details
- No data: "No quota information available for this tenant"
```

---

## 8. Mobile & Responsive Prompt

### Prompt F-016: Mobile DBaaS Dashboard (390px)

```
Design a mobile DBaaS Dashboard at 390x844px for the ERP-DBaaS module.
Target: Platform Engineer on-the-go.

NAVIGATION:
- Bottom tab bar (5 tabs): Home (Dashboard), Instances, Backups, Engines, More
- "More" tab opens a menu with: Plugins, Credentials, Quotas, Settings, Logout
- Top status bar with "DB" logo badge (left), notification bell (right)

HOME TAB:
- Greeting area: "Database Infrastructure" title + current date
- KPI Cards (2x3 grid, each ~170x80px):
  -- Total Instances: 24
  -- Running: 19
  -- Backups: 156
  -- Failed: 2
  -- Storage: 480 GB
  -- Engines: 6

- "Engine Distribution" section (full width card):
  Compact horizontal bars for top 4 engines, "View All" link

- "Recent Instances" section (scrollable list):
  Each item: EngineTag + version + status badge + region + relative time
  Tap to navigate to instance detail
  Max 5 items with "View All" link

INSTANCES TAB:
- Search bar at top (full width)
- Engine filter chips (horizontal scroll): All, YugabyteDB, ScyllaDB, etc.
- Status filter chips (horizontal scroll): All, Running, Failed, etc.
- Instance cards (full width, ~100px tall each):
  -- EngineIcon (32px) + Engine name + version
  -- Status badge + plan badge
  -- Region + created time
  -- Tap to view detail, swipe left for quick actions (Scale, Delete)
- "Provision Instance" floating action button (56px, bottom-right)

INSTANCE DETAIL (full screen):
- Engine header: icon + name + version + status
- Tabbed sections: Overview, Backups, Credentials
- Overview: details list (label: value pairs, full width)
- Connection string with copy button (masked by default)
- Resource usage: 4 mini stat cards (CPU, Memory, Storage, Connections)

BACKUP TAB:
- Backup list as cards with: instance ID, type, status, size, created
- "Create Backup" button at top
- Pull-to-refresh

All touch targets >= 44px. Pull-to-refresh on all lists. Skeleton loading states.
Bottom sheet for quick actions instead of full page navigation.
```

### Prompt F-017: Tablet DBaaS Dashboard (1024px)

```
Design a tablet-adapted DBaaS Dashboard at 1024x768px for the ERP-DBaaS module.

ADAPTATIONS FROM DESKTOP (1440px):
- Sidebar collapsed to icon-only (72px) by default, expandable via hamburger
- KPI cards: 2x3 grid instead of 6-column row (2 per row, 3 rows)
- Engine Distribution + Recent Instances: stacked vertically (full width each)
  instead of side-by-side 8/16 split
- Recent Instances table: hide "Endpoint" column, show on row expand
- Touch-optimized: all clickable areas >= 44px, increased spacing between action buttons

HEADER:
- Hamburger menu (left), "Dashboard" title (center), search icon + notification bell + avatar (right)
- Breadcrumbs hidden; replaced by title

SPECIFIC LAYOUT:
Row 1: 2x3 KPI grid (2 per row: Total Instances + Running, Backups + Failed,
  Storage + Engines)
Row 2: Engine Distribution chart (full width)
Row 3: Recent Instances table (full width, 6 rows max)

PROVISIONING WIZARD (tablet):
- Steps component switches to vertical orientation
- Engine selection: 2x4 grid instead of 4x2
- Size selection: 2x2 grid
- Form inputs: full width

Support landscape (1024x768) and portrait (768x1024) orientations.
In portrait, KPI cards become single column and wizard steps stack vertically.
```

---

## 9. Validation Checklist

```markdown
## Design Handoff Checklist -- ERP-DBaaS

### Visual Completeness
- [ ] All 14 pages designed for 1440px, 1024px, and 390px breakpoints
- [ ] Light theme fully specified (dark theme not in scope)
- [ ] All component states documented (default, hover, active, disabled, error, loading, empty)
- [ ] Realistic data used -- no "Lorem ipsum" or "John Doe"; use actual engine names,
      realistic instance IDs, accurate resource values
- [ ] All 8 engine brand colors represented correctly
- [ ] Status badge colors match STATUS_COLORS mapping from code

### Accessibility
- [ ] Color contrast ratios verified (>= 4.5:1 text, >= 3:1 UI components)
- [ ] Touch targets >= 44x44px verified on all interactive elements
- [ ] Focus order annotated for keyboard navigation
- [ ] Screen reader annotations for all engine icons (aria-label with engine name)
- [ ] Screen reader annotations for progress bars (aria-valuenow, aria-valuemax)
- [ ] Chart accessibility: engine distribution chart has data table alternative
- [ ] Connection string component has aria-label for copy/show toggle actions

### Developer Handoff
- [ ] Design tokens exported in Style Dictionary JSON format
- [ ] Component specs include spacing, sizing, and behavioral notes
- [ ] API data mapping documented:
      -- Dashboard KPIs -> service_instances aggregate queries
      -- Instance list -> service_instances with pagination/filters
      -- Backup list -> backup_records with pagination/filters
      -- Quota gauges -> tenant_quotas single record
- [ ] Interaction specifications:
      -- 4-step wizard navigation with validation gates
      -- Confirmation dialogs for destructive actions
      -- Toast messages for success/error mutations
      -- Real-time status updates via graphql-ws subscriptions
- [ ] Error states and empty states designed for every data-dependent component
- [ ] Loading states (skeleton shimmer) designed for every data-dependent component

### AIDD Guardrails
- [ ] No cross-tenant data access possible in any design
- [ ] Instance deletion requires confirmation dialog with engine name typing
- [ ] Scaling operations show disruption warning
- [ ] Restore operations show overwrite warning
- [ ] Credential values never displayed in cleartext by default
- [ ] Quota exhaustion prevents new provisioning with clear messaging
- [ ] All mutations (create, delete, scale, rotate, restore) have confirmation or loading states
- [ ] Feature flag annotation points marked for:
      -- `dbaas_multi_region_enabled`: region selector in provisioning wizard
      -- `dbaas_enterprise_ha_enabled`: multi-master HA mode option
      -- `dbaas_pitr_enabled`: point-in-time recovery in restore flow
- [ ] Human-in-the-loop gates marked for: instance deletion, bulk operations, restore to existing instance

### Performance Acceptance Targets
| Metric | Target | Measurement |
|--------|--------|-------------|
| Initial route bundle size | < 220 KB gzipped | Webpack bundle analyzer |
| Largest Contentful Paint (LCP) | < 1.5s | Lighthouse CI |
| First Input Delay (FID) | < 100ms | Web Vitals |
| Cumulative Layout Shift (CLS) | < 0.1 | Web Vitals |
| Time to Interactive (TTI) | < 2.5s | Lighthouse CI |
| API read latency (p99) | < 200ms | perf/targets.yaml SLO |
| API write latency (p99) | < 500ms | perf/targets.yaml SLO |
| Availability | >= 99.95% | perf/targets.yaml SLO |
| Table render (1000 rows) | < 300ms | Custom perf mark |
| Instance provisioning UI feedback | < 200ms | Optimistic UI start |
```

---

## 10. Output Packaging Convention

```
Deliverable naming:
  DBaaS-{PromptID}-{Breakpoint}-{Theme}-v{Version}.fig
  Example: DBaaS-F003-1440-light-v1.fig

Layer naming:
  {Page}/{Section}/{Component}-{State}
  Example: Dashboard/KPICards/TotalInstancesCard-Default
  Example: InstanceCreate/Step1-Engine/YugabyteDBCard-Selected
  Example: QuotaShow/ProgressCards/InstancesQuota-Warning

Asset export:
  /exports/dbaas/{breakpoint}/{page}/
  Example: /exports/dbaas/1440/dashboard/kpi-total-instances.png
  Example: /exports/dbaas/1440/engines/yugabytedb-card.png
  Example: /exports/dbaas/390/dashboard/mobile-kpi-grid.png

Engine icon exports:
  /exports/dbaas/icons/{engine}-{size}.svg
  Example: /exports/dbaas/icons/yugabytedb-48.svg

Handoff tokens:
  /tokens/dbaas-tokens.json (Style Dictionary format)

Automation exports:
  /automations/dbaas-{ScenarioID}.json (Make scenario export)
  Example: /automations/dbaas-M001-provisioning-pipeline.json
```

---

## 11. Prompt Usage Guidelines

| Step | Action | Tool |
|------|--------|------|
| 1 | Copy the prompt text from Section 5, 6, or 7 | Clipboard |
| 2 | Open Figma and activate the Make Design / AI plugin | Figma |
| 3 | Paste the prompt into the plugin's input field | Plugin UI |
| 4 | Review the generated output and adjust spacing, alignment, and token references | Manual |
| 5 | Connect components to the shared DBaaS Design System library | Figma Libraries |
| 6 | Export developer-ready assets using the naming convention in Section 10 | Figma Export |

**Prompt modification guidelines:**
- Replace placeholder data with real tenant data when available
- Adjust engine list if customer deployment supports a subset of the 8 engines
- Adjust region list to match actual deployment regions
- Adjust quota limits to match tenant tier configuration
- Keep all AIDD guardrail annotations intact during modifications

---

## 12. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-24 | AIDD System | Initial comprehensive prompt set covering dashboard, instance lifecycle (list/create/show/scale), backup management (list/create/restore), engine catalog, plugin management (list/create), credential rotation, quota monitoring, mobile views, tablet adaptations, 5 Make automation workflows, design system tokens, and component library |

---

## 13. Round 03 Implementation Handoff (Web)

### 13.1 Delivered UI Surfaces
- `web/`: multi-engine DBaaS operations console for Platform Engineers, DBAs, and Application Developers.
- 14 route-level pages covering the complete database instance lifecycle.
- Real-time status updates via Hasura GraphQL subscriptions (graphql-ws).

### 13.2 Design Token Mapping
- Tokens are implemented in `web/src/theme.ts` and consumed by all Ant Design components.
- Palette: primary `#2563eb`, success `#10b981`, warning `#f59e0b`, danger `#ef4444`.
- Typography baseline: Plus Jakarta Sans (all text), system monospace (code/endpoints).
- Border radius: 10px (cards/tables), 8px (buttons/inputs/menu items), 6px (small tags).

### 13.3 Figma Make Build Prompt (Round 03)
```
Build ERP-DBaaS GA-ready UI kit and application flows using the existing token system.

Deliverables:
1. Desktop web ops console with pages:
   - Dashboard, Instance List/Create/Show/Scale, Backup List/Create, Restore,
     Engine Catalog, Plugin List/Create, Credential Rotations, Quota Usage.
2. Mobile-responsive adaptations with:
   - Bottom navigation, engine filter chips, instance card list, floating action buttons.
3. Component library with:
   - EngineIcon/EngineTag (8 engines), KPICard, StatusBadge, ConnectionString,
     QuotaProgress, SizePresetSelector, ProvisioningWizardStepper.

Interaction constraints:
- p95 interaction latency <= 120ms for primary workflows.
- Provisioning wizard completion achievable within 4 clicks after engine selection.
- Preserve AIDD confirmation patterns for destructive actions (delete, restore overwrite).
- Instance status changes reflect in real-time via WebSocket subscriptions.
```

### 13.4 Commercial UX Guardrails
- No dead-end states: every error/empty state includes clear recovery actions.
- Role-first navigation: Platform Engineers can provision an instance within 2 clicks from dashboard.
- KPI-first information architecture: dashboard shows infrastructure health before raw tables.
- Engine-first mental model: every instance, backup, and plugin is contextually linked to its engine.
