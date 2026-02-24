# ERP-DBaaS Figma Design Prompts

## Document Control

| Field             | Value                                  |
|-------------------|----------------------------------------|
| Document Title    | ERP-DBaaS Figma Design Prompts         |
| Version           | 1.0.0                                 |
| Date              | 2026-02-24                             |
| Classification    | Internal - Design                      |
| Author            | Platform Engineering Team              |

---

## Table of Contents

1. [Design System Foundations](#1-design-system-foundations)
2. [Dashboard with KPI Cards](#2-dashboard-with-kpi-cards)
3. [Instance List View](#3-instance-list-view)
4. [Instance Creation Wizard (4-Step)](#4-instance-creation-wizard-4-step)
5. [Instance Detail View](#5-instance-detail-view)
6. [Backup Management Table](#6-backup-management-table)
7. [Engine Catalog Card Grid](#7-engine-catalog-card-grid)
8. [Plugin Registry Table](#8-plugin-registry-table)
9. [Credential Rotation History](#9-credential-rotation-history)
10. [Quota Usage Dashboard](#10-quota-usage-dashboard)

---

## 1. Design System Foundations

### Color Palette

| Token                | Hex       | Usage                              |
|----------------------|-----------|------------------------------------|
| Primary              | #2563EB   | Buttons, links, active states      |
| Primary Hover        | #1D4ED8   | Button hover states                |
| Primary Light        | #EFF6FF   | Backgrounds, selected rows         |
| Success              | #16A34A   | Running status, healthy indicators |
| Warning              | #D97706   | Scaling, pending states            |
| Error                | #DC2626   | Failed status, error messages      |
| Neutral 900          | #111827   | Primary text                       |
| Neutral 500          | #6B7280   | Secondary text                     |
| Neutral 100          | #F3F4F6   | Page background                    |
| White                | #FFFFFF   | Card backgrounds                   |

### Typography

- **Headings**: Inter, Semi-Bold, sizes 24/20/16px
- **Body**: Inter, Regular, 14px, line-height 1.5
- **Code/Monospace**: JetBrains Mono, 13px

### Component Library

All designs use Ant Design components (v5.x) as the base component library, consistent with the Refine.dev framework used in the frontend.

---

## 2. Dashboard with KPI Cards

### Figma Make Prompt

> Design a database management dashboard page using Ant Design components with a blue primary color (#2563EB). The page has a left sidebar navigation with the active item "Dashboard" highlighted. The main content area contains:
>
> **Top Row - 4 KPI Statistic Cards in a horizontal row:**
> 1. "Total Instances" showing "47" with a database icon, a green up arrow with "+3 this week"
> 2. "Active Backups" showing "156" with a cloud-upload icon, subtitle "12 scheduled today"
> 3. "Storage Used" showing "2.4 TB" with a hard-drive icon, a progress bar at 62% utilization
> 4. "Active Engines" showing "6 / 8" with a server icon, subtitle "2 engines in maintenance"
>
> **Middle Row - Two charts side by side:**
> - Left: An area chart titled "Instance Provisioning (Last 30 Days)" showing daily provisioning count with a blue gradient fill
> - Right: A donut chart titled "Instances by Engine" with 8 segments colored differently, showing YugabyteDB as the largest segment
>
> **Bottom Row - A table titled "Recent Activity" with columns:**
> - Timestamp, Action (with colored tags: "Created" green, "Scaled" blue, "Backed Up" purple, "Deleted" red), Instance Name, Engine, Actor
> - Show 5 rows of sample data
>
> Background color: #F3F4F6. Cards have white backgrounds with subtle shadows. Use 24px grid gaps.

---

## 3. Instance List View

### Figma Make Prompt

> Design a database instance list page using Ant Design Table component with blue primary (#2563EB). The page includes:
>
> **Header section:**
> - Page title "Database Instances" (24px, semi-bold) with a subtitle "Manage your provisioned database instances"
> - A row of filters: engine type dropdown (showing all 8 engine icons), status dropdown (Running/Scaling/Stopped/Failed), size dropdown (S/M/L/XL), and a search input
> - A primary blue button "Create Instance" with a plus icon on the right side
>
> **Table with columns:**
> 1. Instance Name - bold text with a small engine icon to the left (YugabyteDB = blue elephant, DragonflyDB = red dragonfly, ClickHouse = yellow arrow, etc.)
> 2. Engine - text with version number in gray below
> 3. Size - badge showing S/M/L/XL
> 4. Status - colored badge: "Running" (green), "Scaling" (yellow), "Provisioning" (blue pulse), "Failed" (red), "Stopped" (gray)
> 5. CPU / Memory - two mini progress bars stacked vertically showing utilization percentage
> 6. Storage - text like "45 / 100 GiB" with a thin progress bar
> 7. Created - relative time like "2 hours ago"
> 8. Actions - three-dot menu with options: View, Scale, Backup Now, Restart, Delete
>
> Show 8 rows of diverse sample data. Include pagination showing "1-8 of 47 instances". Table rows have hover highlight in #EFF6FF.

---

## 4. Instance Creation Wizard (4-Step)

### Figma Make Prompt

> Design a 4-step database instance creation wizard using Ant Design Steps component with blue primary (#2563EB). Show all 4 steps in separate frames:
>
> **Step 1 - Select Engine:**
> - Steps indicator at top: "Select Engine" (active blue), "Configure", "Resources", "Review"
> - A grid of 8 engine cards (2 rows of 4), each card containing: engine logo/icon, engine name, short description (e.g., "Distributed SQL" for YugabyteDB), supported use case tags (e.g., "OLTP", "HA")
> - One card is selected with a blue border and checkmark
> - "Next" button at bottom right (blue primary)
>
> **Step 2 - Configure:**
> - Form fields: Instance Name (text input with validation), Database Name (text input), Engine Version (dropdown), Topology (radio group: "Standalone" or "High Availability"), Replication Factor (number input, visible only when HA selected)
> - "Back" ghost button on left, "Next" primary button on right
>
> **Step 3 - Resources:**
> - Size preset selector: 4 cards in a row (S/M/L/XL), each showing vCPU, Memory, Storage, and monthly cost estimate. Selected card has blue border
> - Storage size slider (20GB to 1TB) with current value displayed
> - Backup configuration: toggle switch "Enable Automated Backups", schedule dropdown (Daily/Hourly), retention days input
> - Estimated monthly cost summary card on the right side
>
> **Step 4 - Review:**
> - A summary card showing all selected options in a description list format
> - Connection string preview (grayed out, shown as template)
> - Checkbox "I understand this will consume X vCPU and Y GiB from my quota"
> - "Back" ghost button, "Create Instance" primary button (blue)

---

## 5. Instance Detail View

### Figma Make Prompt

> Design a database instance detail page using Ant Design components with blue primary (#2563EB). The page contains:
>
> **Header:**
> - Breadcrumb: "Instances > yugabyte-finance-prod"
> - Instance name (24px bold) with a green "Running" status badge next to it
> - Engine icon and "YugabyteDB v2.20.1" in gray text
> - Action buttons row: "Scale" (outlined), "Backup Now" (outlined), "Restart" (outlined), "Delete" (red outlined)
>
> **Connection Info Card:**
> - A card with a dark code-block background showing the connection string: `postgresql://user:****@yugabyte-finance-prod.ns.svc:5433/finance_db`
> - Copy button with clipboard icon
> - Toggle to reveal/hide password
> - Port, host, database name as individual copyable fields
>
> **Tabs section with 5 tabs:**
> 1. **Overview** (active): 4 mini KPI cards (CPU %, Memory %, Storage %, Connections), a time-series line chart showing the last 24h of CPU and memory usage
> 2. **Backups**: Table of recent backups (covered in separate prompt)
> 3. **Plugins**: List of installed plugins with version and install date
> 4. **Credentials**: Current credential info with rotation history
> 5. **Logs**: Scrollable log viewer with severity coloring (INFO blue, WARN yellow, ERROR red)
>
> Show the Overview tab as active with sample metrics data.

---

## 6. Backup Management Table

### Figma Make Prompt

> Design a backup management page using Ant Design Table with blue primary (#2563EB). The page includes:
>
> **Header:**
> - Title "Backups" with subtitle "View and manage database backups across all instances"
> - Filters row: instance dropdown, backup type dropdown (Full/Incremental/WAL), status dropdown (Completed/In Progress/Failed), date range picker
> - "Create Backup" primary button
>
> **Table columns:**
> 1. Backup ID - monospace font, truncated UUID with copy icon
> 2. Instance - instance name with engine icon
> 3. Type - tag: "Full" (blue), "Incremental" (cyan), "WAL" (purple)
> 4. Status - badge: "Completed" (green check), "In Progress" (blue spinner), "Failed" (red X)
> 5. Size - formatted like "2.4 GiB"
> 6. Duration - formatted like "4m 32s"
> 7. Encryption - lock icon (green if encrypted, gray if not)
> 8. Created At - datetime with relative time tooltip
> 9. Expires At - datetime with days remaining
> 10. Actions - "Restore" button (blue link), "Download" button (gray link), "Delete" button (red link)
>
> Show 6 rows of sample data. Include a "Restore Backup" modal frame showing: target instance selector, restore point confirmation, warning message about data overwrite, "Cancel" and "Restore" buttons.

---

## 7. Engine Catalog Card Grid

### Figma Make Prompt

> Design an engine catalog page using Ant Design Card components with blue primary (#2563EB). The page shows:
>
> **Header:**
> - Title "Engine Catalog" with subtitle "Available database engines for provisioning"
> - Search input and category filter tabs: "All", "Relational", "Key-Value", "Time-Series", "Document", "Columnar"
>
> **Card Grid (2 rows of 4 cards):**
> Each card contains:
> - Engine logo/icon at top (64x64)
> - Engine name (bold, 18px)
> - Category tag below the name (e.g., "Distributed SQL" in blue tag)
> - Short description (2 lines max, 14px gray text)
> - Version badge: "v2.20.x" in a small outlined tag
> - Status indicator: green dot + "Available" or yellow dot + "Maintenance"
> - Bottom section with 3 small stats: "Instances: 12", "Avg Uptime: 99.98%", "Plugins: 8"
> - "Provision" primary button at bottom of card
>
> The 8 engines are: YugabyteDB, DragonflyDB, ClickHouse, Tembo (PostgreSQL), SurrealDB, QuestDB, Apache Doris, InfluxDB. Two cards should show "Maintenance" status with a yellow indicator and a disabled "Provision" button.

---

## 8. Plugin Registry Table

### Figma Make Prompt

> Design a plugin registry page using Ant Design Table with blue primary (#2563EB). The page includes:
>
> **Header:**
> - Title "Plugin Registry" with subtitle "Browse and manage database engine plugins and extensions"
> - Filters: engine compatibility dropdown, category dropdown (Search, Analytics, Security, Utility), status dropdown (Available/Installed/Deprecated)
> - Search input for plugin name
>
> **Table columns:**
> 1. Plugin Name - bold text with a small package icon
> 2. Version - text like "1.6.0"
> 3. Compatible Engines - row of small engine icon badges
> 4. Category - colored tag: "Search" (blue), "Analytics" (purple), "Security" (green), "Utility" (gray)
> 5. Description - truncated text (max 60 chars) with tooltip for full text
> 6. Installed Count - number like "23 instances"
> 7. Status - "Available" (green), "Installed" (blue), "Deprecated" (orange)
> 8. Actions - "Install" button (primary, for available), "Uninstall" button (danger, for installed), "Update" button (outlined, when update available)
>
> Show 8 rows including plugins like pg_trgm, pg_stat_statements, postgis, timescaledb, clickhouse-dictionaries, etc. Include an "Install Plugin" modal with target instance multi-select, version selector, and compatibility warning.

---

## 9. Credential Rotation History

### Figma Make Prompt

> Design a credential management page using Ant Design components with blue primary (#2563EB). The page includes:
>
> **Header:**
> - Title "Credential Management" with subtitle "Manage database credentials and view rotation history"
> - "Rotate Now" primary button with a refresh icon
>
> **Current Credentials Card:**
> - A card with sections: Username (copyable), Password (masked with reveal toggle and copy), Created date, Expires date with countdown (e.g., "67 days remaining"), Rotation policy text (e.g., "Auto-rotate every 90 days")
> - A yellow warning banner if rotation is due within 7 days
>
> **Rotation History Table:**
> - Columns: Rotation ID (monospace), Triggered By (tag: "Scheduled" blue, "Manual" green, "Emergency" red), Old Key ID (truncated), New Key ID (truncated), Grace Period (e.g., "24h"), Status (completed green, in-progress blue, failed red), Rotated At (datetime), Completed At (datetime)
> - Show 6 rows of history data
>
> **Rotation Policy Card:**
> - Form showing: Rotation Interval (dropdown: 30/60/90/180 days), Grace Period (input with hours), Password Policy (checkboxes: min length, special chars, uppercase), Auto-rotation toggle
> - "Save Policy" button

---

## 10. Quota Usage Dashboard

### Figma Make Prompt

> Design a quota usage dashboard page using Ant Design components with blue primary (#2563EB). The page includes:
>
> **Header:**
> - Title "Quota Usage" with subtitle "Monitor your resource consumption against tier limits"
> - Current tier badge: "Tier B - Professional" in a blue outlined tag
> - "Upgrade Tier" button (outlined, gold/yellow accent)
>
> **Quota Progress Cards (2x3 grid):**
> Each card shows:
> 1. **Instances**: Progress bar at 65% (13/20), green color. Text "13 of 20 instances used"
> 2. **CPU**: Progress bar at 45% (36/80 vCPU), green color. Text "36 of 80 vCPU allocated"
> 3. **Memory**: Progress bar at 72% (184/256 GiB), yellow/warning color. Text "184 of 256 GiB allocated"
> 4. **Storage**: Progress bar at 38% (1.9/5 TiB), green color. Text "1.9 of 5.0 TiB used"
> 5. **Backup Storage**: Progress bar at 85% (1.7/2 TiB), red/danger color with warning icon. Text "1.7 of 2.0 TiB used"
> 6. **Network**: Progress bar at 22%, green color. Text "Monthly transfer: 440 GiB"
>
> Each card has a thin colored bar at the top matching the severity (green < 70%, yellow 70-85%, red > 85%).
>
> **Usage Trend Chart:**
> - Below the cards, a stacked area chart showing "Resource Usage Trend (Last 30 Days)" with CPU, Memory, and Storage as separate colored areas
>
> **Per-Instance Breakdown Table:**
> - Columns: Instance Name, Engine, Size, CPU Used, Memory Used, Storage Used, % of Quota
> - Show 5 rows sorted by highest resource consumption
> - Rows consuming > 20% of total quota are highlighted with a subtle yellow background

---

## Appendix: Responsive Breakpoints

| Breakpoint | Width     | Layout Adjustments                          |
|------------|-----------|----------------------------------------------|
| Desktop    | >= 1440px | Full layout, 4-column card grids             |
| Laptop     | >= 1024px | 3-column card grids, condensed table columns |
| Tablet     | >= 768px  | 2-column card grids, horizontal scroll tables|
| Mobile     | < 768px   | Single column, stacked cards, drawer nav     |
