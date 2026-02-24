# Figma & Make Prompts -- ERP-SCM
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma Make prompts for the **ERP-SCM** module -- an AI-powered Supply Chain Management platform. ERP-SCM manages procurement, inventory, warehouse operations, logistics, fleet management, manufacturing, demand planning, quality management, and the supplier portal. These prompts generate high-fidelity screens for Supply Chain Directors, Procurement Managers, Warehouse Supervisors, Logistics Coordinators, Fleet Managers, Quality Analysts, and Suppliers.

**Services covered:** demand-planning-service, fleet-management, fleet-service, inventory-service, logistics-service, manufacturing-service, procurement, procurement-service, quality-management, quality-service, supplier-portal-service, warehouse-service

**API surface:** `/v1/procurement`, `/v1/inventory`, `/v1/warehouse`, `/v1/logistics`, `/v1/fleet`, `/v1/manufacturing`, `/v1/demand-planning`, `/v1/quality`, `/v1/supplier-portal`, plus AI endpoints at `/api/ai/*`

**AI Capabilities:** Demand forecasting (Holt-Winters + Random Forest), supplier risk scoring (Isolation Forest), anomaly detection (Z-score), route optimization (NN + 2-opt), business intelligence (health scoring)

---

## 2. AIDD Guardrails (Apply To All Prompts)

Every prompt must comply with these cross-cutting guardrails. Append the relevant subset as context when pasting into Figma Make.

### 2.1 User Experience And Accessibility
- WCAG 2.2 AA minimum; target AAA for critical data displays (inventory levels, quality alerts).
- Keyboard navigable: Tab/Shift-Tab, Enter/Space to activate, Escape to dismiss overlays.
- Command palette via `Cmd+K` / `Ctrl+K`: quick actions including "Create PO", "Check Inventory", "Track Shipment", "View Forecast".
- Role-first layout: Supply Chain Director (dashboards, AI insights), Procurement Manager (PO workflows, supplier scoring), Warehouse Supervisor (WMS floor view, pick/pack), Logistics Coordinator (route maps, fleet tracking), Quality Analyst (QC checklists, inspection reports), Supplier (portal: RFQs, invoices, compliance).
- Progressive disclosure: summary KPIs first, drill into warehouse zones, individual SKUs, shipment legs.
- Skeleton loading for maps, charts, and data-heavy grids; optimistic UI for status updates.
- Color-coding system for inventory levels: Green (healthy), Amber (reorder point), Red (stockout/critical).
- Data density toggle: compact / comfortable / expanded on inventory and order tables.
- Multi-language and multi-currency readiness; RTL-safe layout.
- Touch targets: 44x44px minimum on mobile; 48x48px for warehouse floor tablet UI.

### 2.2 Performance And Frontend Efficiency
- Lighthouse Performance: >= 95.
- Map rendering (logistics/fleet): WebGL-based, < 200ms for 500 markers.
- Inventory table: virtual scroll for 10,000+ SKUs at 60fps.
- Chart rendering: lazy-load forecast and analytics charts below fold.
- First Contentful Paint < 1.2s; Largest Contentful Paint < 2.0s on 4G.
- Bundle budget: < 200KB initial JS (gzipped).

### 2.3 Reliability, Trust, And Safety
- AI confidence badges on ALL AI-generated content: demand forecasts, supplier risk scores, route suggestions, anomaly alerts. Badge levels: Low (amber), Medium (blue), High (green) with tooltip showing model name, training date, and confidence interval.
- Confirmation dialogs for: approving POs > $10,000, releasing manufacturing orders, fleet dispatch, inventory write-offs.
- Undo support: 10-second window for inventory adjustments and order status changes.
- Supplier data isolation: supplier portal shows only their own data; verified via tenant + supplier_id scoping.
- Financial amounts always show currency code (USD, EUR, GBP) and locale-formatted numbers.

### 2.4 Observability And Testability
- `data-testid` on every interactive element: `{page}-{component}-{action}`.
- Analytics events: page_view, po_created, shipment_tracked, forecast_generated, route_optimized, quality_inspection_completed.
- Error states: empty inventory, no suppliers found, forecast unavailable, map load failure, API timeout -- each with illustration and retry CTA.

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```
Create a Figma design-token page titled "ERP-SCM / Tokens" at 1440x3200px.

COLOR PALETTE (light mode):
- Primary: #0F766E (Supply Chain Teal)
- Primary Hover: #0D5D56
- Secondary: #4338CA (Procurement Indigo)
- Accent: #EA580C (Logistics Orange)
- Success: #16A34A
- Warning: #F59E0B
- Danger: #DC2626
- Neutral-50 through Neutral-900 (9-step gray scale)
- Surface: #FFFFFF
- Background: #F3F4F6

INVENTORY STATUS COLORS:
- In Stock: #16A34A (Green-600)
- Low Stock: #F59E0B (Amber-500)
- Reorder Point: #F97316 (Orange-500)
- Out of Stock: #DC2626 (Red-600)
- Overstock: #7C3AED (Violet-600)

QUALITY COLORS:
- Pass: #16A34A
- Conditional Pass: #F59E0B
- Fail: #DC2626
- Pending Inspection: #6B7280

SUPPLIER RISK COLORS:
- Low Risk: #16A34A
- Medium Risk: #F59E0B
- High Risk: #DC2626
- Critical: #7F1D1D (Red-900)

COLOR PALETTE (dark mode):
- Primary: #5EEAD4 (Teal-300)
- Secondary: #818CF8 (Indigo-400)
- Accent: #FB923C (Orange-400)
- Surface: #1F2937
- Background: #111827

TYPOGRAPHY (Inter font family):
- Display: 36px / 700 / 1.1
- H1: 30px / 700 / 1.2
- H2: 24px / 600 / 1.3
- H3: 20px / 600 / 1.3
- H4: 16px / 600 / 1.4
- Body-lg: 16px / 400 / 1.5
- Body: 14px / 400 / 1.5
- Caption: 12px / 400 / 1.4
- Overline: 11px / 600 / 1.4 / uppercase
- Mono: 13px / JetBrains Mono / 400 (SKU codes, PO numbers, tracking IDs)

SPACING: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96px
ELEVATION: shadow-sm, shadow-md, shadow-lg, shadow-xl
BORDER RADIUS: 4px (sm), 8px (md), 12px (lg), 16px (xl)

Show each token as labeled swatch with light/dark side-by-side.
```

#### Prompt F-002: Component Library

```
Create a Figma component library page titled "ERP-SCM / Components" at 1440x6000px.

Include components with light/dark variants and all states (default, hover, active, focus, disabled):

BUTTONS: Primary (teal), Secondary (indigo), Ghost, Danger, Icon-only. Sizes: sm, md, lg.
INPUTS: Text, Search (Cmd+K), Select, Multi-select, Date picker, Date range, Currency input (with currency code prefix), Quantity input (with unit suffix: pcs, kg, lbs, pallets), Textarea, File upload dropzone.

INVENTORY COMPONENTS:
- SKU Card: SKU code (mono), product name, image thumbnail (48x48), current qty, reorder point, warehouse location code, status badge (In Stock/Low/Reorder/Out of Stock/Overstock).
- Stock Level Bar: horizontal bar showing current qty relative to min/reorder/max thresholds, with colored zones.
- Warehouse Zone Map Tile: grid cell showing zone code, item count, utilization %, colored fill by utilization.

PROCUREMENT COMPONENTS:
- Purchase Order Card: PO number (mono), supplier name + logo, total amount (currency formatted), line items count, status badge (Draft/Submitted/Approved/Received/Cancelled), date.
- RFQ Card: RFQ number, title, closing date, responses count, status.
- Supplier Score Gauge: circular gauge 0-100, colored by risk level, with label.

LOGISTICS COMPONENTS:
- Shipment Tracker: horizontal step indicator (Origin > In Transit > Customs > Destination) with dates, current step highlighted, carrier logo, tracking number (mono).
- Route Map Pin: numbered circle pin (32px) with color by delivery status (on-time green, delayed amber, missed red).
- Fleet Vehicle Card: vehicle ID, type icon (truck/van/ship/plane), status (Available/In Transit/Maintenance), driver name, current location, fuel level bar.

QUALITY COMPONENTS:
- Inspection Badge: Pass/Conditional/Fail circle icon with label.
- QC Checklist Item: checkbox, check point description, result (pass/fail), notes field, photo attachment slot.
- Defect Rate Sparkline: mini line chart showing defect rate trend over 30 days.

AI COMPONENTS:
- AI Confidence Badge: Low (amber), Medium (blue), High (green) pill with "i" tooltip showing model name and confidence interval.
- Forecast Chart Card: line chart with prediction line (solid) and confidence band (shaded), actual values (dots), model name label.
- Anomaly Alert Card: severity icon, description, metric name, expected vs actual values, recommended action, confidence score.
- Health Score Card: letter grade (A-F) in large colored circle, subscore bars (inventory, delivery, quality, fulfillment, supplier).

DATA DISPLAY: Table, Stat card, Badge, Tag, Progress bar, Currency display, Sparkline.
NAVIGATION: Sidebar, Top bar, Breadcrumbs, Tabs.
FEEDBACK: Toast, Modal, Confirmation dialog, Empty state, Error state, Skeleton loader.

Named layers: `{category}/{component}/{variant}/{state}`.
```

---

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: AI Command Center Dashboard (1440px)

```
Design a desktop dashboard titled "ERP-SCM / AI Command Center" at 1440x1200px.

LAYOUT:
- Left sidebar (240px, collapsible): Logo, nav groups: "Command Center" (Dashboard, AI Insights, Anomalies, Alerts), "Procurement" (Purchase Orders, RFQs, Suppliers), "Inventory" (Products, Stock Levels, Warehouses), "Logistics" (Shipments, Fleet, Routes), "Manufacturing" (Production Orders, BOM, Scheduling), "Quality" (Inspections, Reports, Compliance), "Planning" (Demand Forecast, Replenishment). Active: Dashboard.
- Top bar (64px): Breadcrumb "SCM > AI Command Center", search, notification bell "14", tenant "Acme Manufacturing", avatar.
- Content: 24px padding.

CONTENT:
Row 1 -- Supply Chain Health Score (full width, 120px height, gradient teal-to-indigo):
Large centered: Letter grade "B+" in white circle (80px), "Supply Chain Health Score: 82/100" (H2, white), subscores inline: Inventory 88 | Delivery 79 | Quality 91 | Fulfillment 76 | Supplier 84. AI confidence badge "High (0.91)" (white pill).

Row 2 -- KPI Cards (5 across):
- "Total Inventory Value": $12.4M | 34,567 SKUs | sparkline
- "Open POs": 147 | $2.1M pending | amber if > 100
- "On-Time Delivery": 94.2% | Target: 95% | amber trend arrow down
- "Active Shipments": 89 | 23 international | map thumbnail
- "Quality Pass Rate": 97.8% | 12 inspections today | green

Row 3 -- Two panels:
Left (55%): "AI Insights" card list (4 insights generated by insights_engine):
Each insight card: severity icon (teal info, amber warning, red critical), title, body text, affected metric, recommended action, confidence badge, "Take Action" button.
- Info: "Inventory optimization opportunity: 23 SKUs have excess stock worth $180K. Consider markdown or redistribution." Confidence: High (0.88).
- Warning: "Supplier risk increase: Chen Electronics risk score rose from 34 to 62 (Medium). Delivery times +18% this quarter." Confidence: Medium (0.76).
- Warning: "Demand spike predicted: SKU-4521 (Industrial Bearings) forecast shows 40% demand increase in March. Current stock covers only 12 days." Confidence: High (0.85).
- Critical: "Late delivery cluster: 8 shipments from Lagos warehouse are 2+ days late. Carrier: FastFreight. Investigate route disruption." Confidence: Medium (0.72).

Right (45%): "Anomaly Detection" feed:
Table: Timestamp | Type (Inventory/Demand/Delivery) | Description | Severity badge | Status (New/Acknowledged/Resolved). 6 rows with realistic anomalies.

Row 4 -- Two panels:
Left (50%): "Demand Forecast Preview" area chart: 3 product lines, actual data (solid) + forecast (dashed) + confidence band (shaded). X-axis: last 6 months + 3 months forecast. Legend: "Industrial Bearings", "Circuit Boards", "Steel Coils".

Right (50%): "Supplier Risk Matrix" scatter plot: X-axis: Performance Score (0-100), Y-axis: Risk Score (0-100). Each dot = supplier, sized by spend volume. Quadrants labeled: "Strategic" (top-right green), "Leverage" (bottom-right blue), "Bottleneck" (top-left amber), "Non-Critical" (bottom-left gray). 15 supplier dots.

Light and dark mode.
```

#### Prompt F-004: Procurement & Purchase Orders (1440px)

```
Design a desktop page titled "ERP-SCM / Purchase Orders" at 1440x1200px.

LAYOUT: Standard sidebar/top bar. Active: Purchase Orders. Breadcrumb: "SCM > Procurement > Purchase Orders".

CONTENT:
Header: "Purchase Orders" (H1). Right: "Create PO" primary button, "Import" ghost, "Export" ghost.

Summary cards (4 across):
- "Open POs": 147 | $2.1M total
- "Awaiting Approval": 23 | $890K
- "In Transit": 64 | ETA spread chart
- "Received This Month": 312 | $4.7M | 96% on-time

Filter bar: Search "Search PO#, supplier, item...", Status (All/Draft/Submitted/Approved/Shipped/Received/Cancelled), Supplier dropdown, Date range, Amount range (min-max), Priority.

Data table (full width):
Columns: Checkbox | PO # (mono) | Supplier (logo + name) | Items | Total Amount (currency) | Status badge | Order Date | Expected Delivery | Received Date | Priority icon | Actions kebab.

15 rows with realistic data:
- PO-2026-0847 | Chen Electronics Ltd | 12 items | $48,750.00 USD | Approved (green) | Feb 18 | Mar 2 | -- | High (orange)
- PO-2026-0842 | Acme Steel Corp | 3 items | $125,000.00 USD | Awaiting Approval (amber) | Feb 17 | Mar 10 | -- | Critical (red)
- PO-2026-0835 | FastParts Inc | 8 items | $12,340.50 USD | Received (blue) | Feb 10 | Feb 20 | Feb 19 | Medium (amber)
- (12 more varied rows including multi-currency: EUR, GBP)

Pagination: "Showing 1-15 of 147" | page size | prev/next.

Bulk actions bar: "5 selected" | "Bulk Approve" | "Export Selected" | "Cancel Selected" (danger).

PO DETAIL (slide-in panel, 560px, shown for PO-2026-0847):
- Header: PO number, supplier name + logo, status badge, created by.
- Tabs: Details | Line Items | Delivery | Documents | Audit.
- Details tab: Supplier info card, payment terms, shipping method, notes.
- Line Items tab: table of items (SKU, description, qty, unit price, total, received qty).
- AI recommendation banner: "AI Suggestion: Consider consolidating with PO-2026-0851 to same supplier for 8% volume discount. Confidence: Medium (0.74)." with "Apply" button.

APPROVAL WORKFLOW: For "Awaiting Approval" POs, show approval chain: Requestor (submitted) > Procurement Manager (pending) > Finance Director (waiting). Each step with avatar, name, status icon, timestamp.

Light and dark mode.
```

#### Prompt F-005: Inventory & Warehouse Dashboard (1440px)

```
Design a desktop page titled "ERP-SCM / Inventory Dashboard" at 1440x1300px.

LAYOUT: Standard sidebar/top bar. Active: Stock Levels. Breadcrumb: "SCM > Inventory > Stock Levels".

CONTENT:
Header: "Inventory Overview" (H1). Right: "Add Product" primary, "Adjust Stock" secondary, "Cycle Count" ghost.

Row 1 -- KPI Cards (5 across):
- "Total SKUs": 34,567 | Active: 28,430
- "Inventory Value": $12.4M | +$340K this month
- "Stockout Items": 23 (red highlight) | Revenue at risk: $89K
- "Reorder Alerts": 147 | 42 critical
- "Dead Stock": $180K | 312 SKUs with zero movement (90d)

Row 2 -- Two panels:
Left (60%): "Inventory by Warehouse" stacked bar chart:
5 warehouses: "Main Distribution (US-East)", "West Coast Hub", "EU Central (Frankfurt)", "APAC (Singapore)", "Lagos Regional". Each bar stacked by category: Raw Materials (blue), WIP (amber), Finished Goods (green). Y-axis: value in $M.

Right (40%): "Stock Status Distribution" donut chart:
- In Stock: 24,120 (70%) green
- Low Stock: 5,430 (16%) amber
- Reorder Point: 3,210 (9%) orange
- Out of Stock: 807 (2%) red
- Overstock: 1,000 (3%) violet
Center: total 34,567 SKUs.

Row 3 -- "Stock Levels" searchable table (full width):
Columns: Image (32px) | SKU (mono) | Product Name | Category | Warehouse | Qty On Hand | Reorder Point | Qty On Order | Status badge | Unit Cost | Total Value | AI Reorder Suggestion | Actions.

12 rows with realistic data:
- [img] | SKU-4521 | "Industrial Bearing 6205-2RS" | Components | Main Distribution | 1,240 | 500 | 0 | In Stock (green) | $4.50 | $5,580 | "Optimal" badge
- [img] | SKU-1893 | "Circuit Board CB-200X" | Electronics | EU Central | 45 | 200 | 500 on PO-2026-0847 | Low Stock (amber) | $28.00 | $1,260 | "Order 300 more" AI badge
- [img] | SKU-7734 | "Steel Coil Grade A (1T)" | Raw Materials | West Coast | 0 | 50 | 0 | Out of Stock (red) | $2,100 | $0 | "URGENT: Reorder" AI badge (red)
- (9 more varied rows)

Row 4 -- "Warehouse Floor Map" (full width, 400px height):
Visual grid representation of "Main Distribution" warehouse:
- Zones: A (Receiving), B (Raw Materials), C (WIP), D (Finished Goods), E (Shipping).
- Each zone shows: zone code, item count, utilization % with color fill (green < 70%, amber 70-90%, red > 90%).
- Zone B highlighted in amber (82% full).
- Interactive: hover shows zone detail tooltip.

Light and dark mode.
```

#### Prompt F-006: Logistics & Fleet Tracking (1440px)

```
Design a desktop page titled "ERP-SCM / Logistics & Fleet" at 1440x1200px.

LAYOUT: Standard sidebar/top bar. Active: Shipments. Breadcrumb: "SCM > Logistics > Active Shipments".

CONTENT:
Header: "Logistics Command" (H1). Right: "Create Shipment" primary, "Optimize Routes" secondary (AI icon), "Fleet Status" ghost.

Row 1 -- KPI Cards (4 across):
- "Active Shipments": 89 | 23 international, 66 domestic
- "On-Time Rate": 94.2% | Target: 95% | amber trend
- "Fleet Utilization": 78% | 42/54 vehicles active
- "Avg Delivery Time": 3.2 days | -0.4d vs last month | green

Row 2 -- Shipment Map (full width, 450px height):
World map (or regional map) showing:
- Origin markers (blue circles): warehouse locations (5 markers).
- Destination markers (teal pins): delivery locations (20+ markers).
- Route lines (curved, animated dots showing direction):
  - Green lines: on-time shipments.
  - Amber lines: delayed shipments.
  - Red lines: critical/overdue shipments.
- Vehicle icons on routes showing current position.
- Side panel overlay (320px, left): Shipment quick-list, scrollable: 8 shipments with tracking#, origin > destination, status, ETA. Click to highlight on map.
- Map controls: zoom, layers toggle (traffic, weather), filter by carrier/status.

Row 3 -- Two panels:
Left (55%): "Shipment Tracking Table":
Columns: Tracking # (mono) | Origin | Destination | Carrier (logo + name) | Status (step indicator: Picked Up > In Transit > Out for Delivery > Delivered) | Ship Date | ETA | Actual Delivery | Delay (days, red if > 0) | Actions.
8 rows with realistic data:
- SHP-89012 | Main Distribution, NJ | Chicago, IL | FastFreight | In Transit (step 2/4) | Feb 20 | Feb 24 | -- | 0d
- SHP-88934 | EU Central, DE | London, UK | DHL Express | Out for Delivery (step 3/4) | Feb 19 | Feb 22 | -- | +1d (amber)
- (6 more)

Right (45%): "Fleet Status" grid:
Vehicle cards (3 columns, 4 rows):
Each card: Vehicle ID, type icon (truck emoji-free: use "TRK", "VAN", "SHIP"), status badge (Available green/In Transit blue/Maintenance amber), driver name, current location (city), fuel/charge level bar.
12 vehicles.

Row 4 -- "AI Route Optimization" panel (full width):
Banner: "Route optimization available for 5 pending shipments. Estimated savings: 12% fuel cost, 8% delivery time." AI confidence: High (0.87). "Run Optimization" primary button.

When expanded (show detail frame):
Before/After comparison:
- Left: "Current Routes" map with 5 routes (total: 2,400 km, est 18h).
- Right: "Optimized Routes" map with consolidated routes (total: 2,080 km, est 15.5h).
- Savings summary: Distance: -320km (-13%), Time: -2.5h (-14%), Fuel: -$480 (-12%).
- "Apply Optimized Routes" primary CTA, "View Details" secondary.

Light and dark mode.
```

#### Prompt F-007: Supplier Portal (1440px)

```
Design a desktop page titled "ERP-SCM / Supplier Portal" at 1440x1100px.

NOTE: This is the SUPPLIER-FACING portal (not internal SCM user). Different header/branding. Supplier sees only their own data.

LAYOUT:
- Top bar (64px): "ERP Supplier Portal" logo, supplier company name "Chen Electronics Ltd", notification bell "3", contact avatar "Li Wei", help icon.
- Left sidebar (200px): Dashboard, Purchase Orders, RFQs, Invoices, Quality Reports, Documents, Profile.
- Content area.

CONTENT (Supplier Dashboard):
Welcome: "Welcome back, Li Wei" (H2), "Chen Electronics Ltd" (subtitle), "Verified Supplier" green badge.

Row 1 -- KPI Cards (4 across):
- "Active POs": 12 | $348K total value
- "Pending RFQs": 3 | Closing in 2-7 days
- "On-Time Delivery Score": 92% | Target: 95% | amber
- "Quality Score": 96% | 0 rejections this month | green

Row 2 -- Two panels:
Left (50%): "Open Purchase Orders" table:
Columns: PO # | Buyer | Items | Value | Status | Due Date | Actions.
5 rows:
- PO-2026-0847 | Acme Manufacturing | 12 items | $48,750 | Confirmed (blue) | Mar 2 | "Ship" / "View"
- PO-2026-0823 | Globex Corp | 4 items | $22,100 | Awaiting Confirmation (amber) | Mar 8 | "Confirm" / "Reject"
- (3 more)

Right (50%): "Pending RFQs" cards:
3 RFQ cards:
- RFQ-2026-0234 | "Electronic Components Bulk" | 15 line items | Closes: Feb 28 | "Submit Bid" primary CTA
- RFQ-2026-0229 | "Cable Assemblies Q2" | 8 items | Closes: Mar 5 | "Submit Bid"
- RFQ-2026-0221 | "PCB Manufacturing Lot" | 3 items | Closes: Mar 1 | "Draft in Progress" amber badge

Row 3 -- Two panels:
Left (50%): "Performance Scorecard" (from buyer's perspective):
Radar chart with 5 axes: Delivery Timeliness (92%), Quality (96%), Price Competitiveness (78%), Communication (88%), Financial Stability (90%).
Overall score: 88/100 (B+). Trend: "Improving" green badge.

Right (50%): "Recent Quality Reports":
4 items: Inspection date, PO reference, items inspected, result badge (Pass/Conditional/Fail), defects found.

Row 4 -- "Action Items" banner:
"You have 2 POs requiring confirmation and 3 RFQs open for bidding. 1 invoice is overdue for submission." with inline action links.

Light and dark mode.
```

#### Prompt F-008: Quality Management Dashboard (1440px)

```
Design a desktop page titled "ERP-SCM / Quality Dashboard" at 1440x1100px.

LAYOUT: Standard sidebar/top bar. Active: Inspections. Breadcrumb: "SCM > Quality > Dashboard".

CONTENT:
Header: "Quality Management" (H1). Right: "New Inspection" primary, "Reports" secondary.

Row 1 -- KPI Cards (4 across):
- "Pass Rate": 97.8% | Target: 98% | amber (just below target)
- "Inspections Today": 12 | 8 completed, 4 pending
- "Open NCRs": 7 | Non-Conformance Reports | 2 critical
- "CAPA Actions": 14 active | 3 overdue (red)

Row 2 -- Two panels:
Left (55%): "Inspection Results" table:
Columns: Inspection # (mono) | PO / Production Order | Supplier/Line | Inspector | Items | Pass/Fail/Conditional | Defect Count | Date | Actions.
8 rows with realistic data.

Right (45%): "Quality Trend" line chart:
3 lines over 12 months: Pass Rate (green), Conditional Rate (amber), Fail Rate (red). Y-axis: percentage. Target line at 98%.

Row 3 -- Two panels:
Left (50%): "Defect Pareto" bar chart:
Top 8 defect categories: "Dimensional tolerance" 34%, "Surface finish" 22%, "Assembly error" 15%, "Material defect" 12%, "Packaging damage" 8%, "Color mismatch" 5%, "Labeling error" 3%, "Other" 1%. Cumulative line overlay.

Right (50%): "Supplier Quality Ranking":
Ranked list: Supplier name, inspections count, pass rate, defect rate, trend arrow. Color-coded rows by quality tier.

Include QC inspection detail modal (720px):
- Inspection #, PO reference, date, inspector, location.
- Checklist: 10 check points with Pass/Fail radio, notes, photo attachment thumbnails.
- Summary: overall result, defects found, corrective actions.
- Signatures section.

Light and dark mode.
```

---

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile SCM Dashboard (390px)

```
Design a mobile dashboard titled "ERP-SCM / Mobile Dashboard" at 390x844px.

LAYOUT:
- Bottom tab bar (56px): Dashboard (active), Inventory, Shipments, Scan, More.
- Top bar (56px): "Supply Chain" title, search icon, bell "14", avatar.
- Content: scrollable, 16px padding.

CONTENT:
Health score banner (full width, gradient teal):
"Supply Chain Health: B+ (82/100)" (H3, white). AI confidence: "High" green pill.

KPI horizontal scroll (4 cards, 140px each):
- Inventory: $12.4M | 23 stockouts
- Open POs: 147 | $2.1M
- Shipments: 89 active | 94.2% on-time
- Quality: 97.8% pass rate

"AI Alerts" section (3 cards):
Each: severity stripe (left border), title (1 line), description (2 lines, truncated), confidence badge, timestamp. Tappable for detail.
- Amber: "Demand spike predicted for SKU-4521"
- Red: "8 shipments delayed from Lagos warehouse"
- Teal: "Route optimization available: save 12% fuel"

"Active Shipments" compact list (4 items):
Tracking # | Route (origin > dest) | Status badge | ETA.
"View All" link.

"Low Stock Alerts" compact list (4 items):
SKU + name | Qty | Status badge (amber/red).
"View All" link.

Light and dark mode.
```

#### Prompt F-010: Mobile Inventory Scanner (390px)

```
Design a mobile page titled "ERP-SCM / Mobile Scanner" at 390x844px.

LAYOUT: Bottom tab bar with "Scan" active. Top bar: "Inventory Scanner" title, flashlight toggle icon.

CONTENT:
Camera viewfinder area (full width, 300px height):
- Barcode/QR scan frame (centered rectangle with corner brackets).
- "Point camera at barcode or QR code" instruction text.
- Manual entry link: "Enter SKU manually".

Below viewfinder -- "Last Scanned" result card (appears after scan):
- SKU: "SKU-4521" (mono, large)
- Product: "Industrial Bearing 6205-2RS"
- Image thumbnail (64px)
- Location: "Zone B, Rack 12, Shelf 3"
- Qty On Hand: 1,240
- Status: "In Stock" green badge
- Reorder Point: 500
- AI Suggestion: "Stock level optimal" green text

Action buttons (full width):
- "Adjust Quantity" primary (opens number pad bottom sheet)
- "Move Item" secondary
- "View Full Details" ghost

Quantity adjustment bottom sheet (second frame):
- Product name + SKU
- Current qty: 1,240
- Adjustment type: "Add" / "Remove" / "Set" segmented control
- Number pad with large digits (48px buttons)
- Reason dropdown: "Received", "Returned", "Damaged", "Cycle Count", "Other"
- Notes field
- "Confirm Adjustment" primary button

Scan history (scrollable below last result):
5 recent scans: timestamp, SKU, product name, action taken.

Light and dark mode.
```

#### Prompt F-011: Mobile Shipment Tracking (390px)

```
Design a mobile page titled "ERP-SCM / Mobile Shipment" at 390x844px.

LAYOUT: Bottom tab bar with "Shipments" active. Top bar: "Active Shipments" title, search icon, filter icon.

CONTENT:
Filter chips (horizontal scroll): All | In Transit | Out for Delivery | Delayed | Delivered.

Shipment cards (vertical list):
Each card:
- Row 1: Tracking # (mono, H4) + Carrier logo (24px, right)
- Row 2: Route: "New Jersey" arrow-right "Chicago, IL"
- Row 3: Status step indicator (4 steps, horizontal, current highlighted): Picked Up > In Transit > Out for Delivery > Delivered.
- Row 4: "ETA: Feb 24" + delay badge if applicable: "+1 day" (amber)
- Row 5: Items count + value (muted caption)

6 shipment cards with varied statuses.

Tapping a card shows shipment detail (second frame, 390x844px):
- Header: Tracking #, carrier, status badge.
- Map (200px height): route line with current position marker, origin/destination pins.
- Timeline (vertical): each tracking event as a step:
  - Feb 20 09:00 "Picked up from Main Distribution, NJ"
  - Feb 20 14:30 "Departed facility"
  - Feb 21 08:00 "In transit -- Columbus, OH hub"
  - Feb 22 06:00 "In transit -- Gary, IN"
  - Feb 23 (estimated) "Arrive Chicago, IL"
- Items list: SKU, name, qty for each item in shipment.
- Actions: "Report Issue" | "Contact Carrier" | "Share Tracking"

Light and dark mode.
```

#### Prompt F-012: Mobile Purchase Order (390px)

```
Design a mobile page titled "ERP-SCM / Mobile PO Detail" at 390x844px.

LAYOUT: Top bar: back arrow, "PO-2026-0847" title, kebab menu. No bottom tab bar.

CONTENT:
Status banner (full width): "Approved" green background, "Awaiting Shipment" subtitle.

Supplier card:
- Logo (48px) + "Chen Electronics Ltd" (H4) + "Verified" badge
- Contact: "Li Wei" | phone icon | email icon

Details section:
- Order Date: Feb 18, 2026
- Expected Delivery: Mar 2, 2026
- Payment Terms: Net 30
- Shipping: DHL Express
- Priority: High (orange badge)

Line Items section (expandable):
"12 items | $48,750.00 USD" header.
Compact list:
- "Circuit Board CB-200X" | Qty: 500 | $28.00 ea | $14,000
- "Capacitor Array CA-50" | Qty: 2,000 | $0.85 ea | $1,700
- (expand to see all 12)

Totals:
- Subtotal: $45,000.00
- Tax (8%): $3,600.00
- Shipping: $150.00
- Total: $48,750.00

Approval chain (vertical timeline):
- Requestor: Sarah Chen (submitted Feb 17) -- checkmark
- Procurement Mgr: James Okonkwo (approved Feb 18) -- checkmark
- Finance: Auto-approved (< $50K threshold) -- checkmark

Actions (sticky bottom): "Track Shipment" primary | "Edit" secondary | "Cancel PO" danger ghost.

AI recommendation (collapsible):
"Consolidation opportunity: PO-2026-0851 also from Chen Electronics. Combining may save 8% ($3,900). Confidence: Medium (0.74)."

Light and dark mode.
```

#### Prompt F-013: Mobile Fleet Tracking (390px)

```
Design a mobile page titled "ERP-SCM / Mobile Fleet" at 390x844px.

LAYOUT: Accessed from "More" tab. Top bar: back arrow, "Fleet Tracking" title, map/list toggle.

MAP VIEW (default):
Full-screen map (below top bar, above action bar):
- Vehicle markers: 12 vehicles shown as colored dots with vehicle ID labels.
  - Green dots: 8 available/idle vehicles at depot.
  - Blue dots: 3 in-transit vehicles on routes.
  - Amber dot: 1 vehicle in maintenance.
- Depot marker: warehouse icon.
- Route lines for in-transit vehicles.
- Bottom sheet (peek: 120px, swipe up for full): "42 Active | 8 Maintenance | 4 Idle"

Tapping a vehicle marker shows bottom sheet detail:
- Vehicle: "TRK-2847" | Type: Heavy Truck | Status: "In Transit" blue
- Driver: "Marcus Johnson" | avatar | call button
- Route: "NJ Depot > Chicago, IL" | progress: 65%
- Fuel: 72% | Mileage: 142,800 km
- Current speed: 65 mph | ETA: 4.5 hours
- Last ping: "2 min ago"
- "View Full Details" link.

LIST VIEW (toggle, second frame):
Vehicle cards in vertical list:
- ID + Type | Driver name | Status badge | Location | Fuel bar.
8 vehicles.

Light and dark mode.
```

#### Prompt F-014: Mobile Quality Inspection (390px)

```
Design a mobile page titled "ERP-SCM / Mobile QC Inspection" at 390x844px.

LAYOUT: Top bar: back arrow, "Inspection INS-2026-0489" title, save icon. No bottom tab bar.

CONTENT:
Header card:
- PO: "PO-2026-0847" link
- Supplier: "Chen Electronics Ltd"
- Items: "500x Circuit Board CB-200X"
- Inspector: "Alex Kim"
- Date: "Feb 23, 2026"

Inspection checklist (scrollable):
10 check points, each:
- Check point name (Body, semibold)
- Pass / Fail toggle buttons (44px height, green/red)
- Notes field (collapsed, expand on tap)
- Camera icon to attach photo (opens camera)
- Specification reference (caption, muted)

Check points:
1. "Visual inspection -- no visible defects" | [Pass] [Fail]
2. "Dimensional accuracy (within +/- 0.05mm)" | [Pass] [Fail]
3. "Solder joint quality" | [Pass] [Fail]
4. "Component placement accuracy" | [Pass] [Fail]
5. "Electrical continuity test" | [Pass] [Fail]
6. "Surface finish quality" | [Pass] [Fail]
7. "Labeling correctness" | [Pass] [Fail]
8. "Packaging integrity" | [Pass] [Fail]
9. "Quantity verification (sample: 50/500)" | [Pass] [Fail]
10. "Documentation completeness" | [Pass] [Fail]

5 checks marked Pass, 1 marked Fail (highlighted red background), 4 pending.

Summary bar (sticky bottom, 80px):
"6/10 checked | 5 Pass | 1 Fail | 4 Remaining"
"Submit Inspection" primary button (disabled until all checked).

Photo attachments section (above summary):
Thumbnail grid of captured photos with captions.

Light and dark mode.
```

---

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-015: Tablet SCM Dashboard (1024px)

```
Design a tablet dashboard titled "ERP-SCM / Tablet Dashboard" at 1024x768px.

LAYOUT: Icon-only sidebar (64px), top bar (56px).

CONTENT:
Health score banner: compact, single row.
KPI cards: 5 across (narrower).

AI Insights: 2-column card grid instead of list.
Anomaly feed: below insights, full width, 4 rows.

Shipment map: full width, 300px height with overlay list panel.

Touch-optimized: all targets 44px+, card actions via swipe or tap.

Light and dark mode.
```

#### Prompt F-016: Tablet Warehouse Floor View (1024px)

```
Design a tablet page titled "ERP-SCM / Tablet Warehouse Floor" at 1024x768px.

PURPOSE: This is designed for warehouse supervisors using a tablet on the floor.

LAYOUT: No sidebar. Full-screen warehouse view with top bar (48px): warehouse name, zone selector tabs, alert bell, worker avatar.

CONTENT:
Warehouse floor map (full width, 500px):
- Grid layout of warehouse zones (A through E).
- Each zone: large tappable area (min 100x100px), zone code, item count, utilization bar.
- Active pickers shown as animated dots with worker name labels.
- Highlighted zones: amber for > 80% utilization, red for active alerts.

Below map:
Tab bar: Pick Queue | Receiving | Shipping | Inventory Alerts.

Pick Queue tab (active):
List of pick orders:
- Order # | Priority (colored dot) | Items count | Zone(s) | Assigned picker | Status (Pending/In Progress/Complete).
8 rows. Touch-friendly row height (56px).

"Assign to Me" and "Complete Pick" action buttons per row.

Large text mode toggle: increases all text by 20% for readability in warehouse lighting.

Light mode only (warehouse environment). High-contrast variant available.
```

#### Prompt F-017: Tablet Supplier Portal (1024px)

```
Design a tablet supplier portal titled "ERP-SCM / Tablet Supplier Portal" at 1024x768px.

LAYOUT: Collapsible sidebar (200px, icon-only on collapse), top bar with supplier branding.

CONTENT:
Same as desktop F-007 supplier portal but with:
- 3-column grid for cards (instead of 4).
- Tables show 6 key columns; secondary columns available via horizontal scroll.
- RFQ cards at full width.
- Performance radar chart at 300px square.
- Touch-optimized action buttons (44px height).

Light and dark mode.
```

---

## 4. Make Automation Prompts

#### Prompt M-001: Procurement Lifecycle Automation

```
Create a Make (Integromat) scenario titled "ERP-SCM: Purchase Order Lifecycle".

TRIGGER: Webhook -- CloudEvent "erp.scm.procurement.*".

FLOW:
1. Router by action:
   a. "created" (PO submitted):
      - HTTP GET `/v1/procurement/{po_id}` for full PO details.
      - If total_amount > $10,000: route to approval queue.
        - HTTP POST notification to procurement manager: "PO {{po_number}} for ${{amount}} requires approval."
      - If total_amount <= $10,000: auto-approve.
        - HTTP PUT `/v1/procurement/{po_id}` status = "approved".
        - HTTP POST notification to supplier via supplier-portal: "New PO received."
      - HTTP POST to audit trail.

   b. "updated" (status change to "shipped"):
      - HTTP POST to logistics-service `/v1/logistics` to create shipment tracking entry.
      - HTTP POST to inventory-service `/v1/inventory` to create "expected receipt" record.
      - HTTP POST notification to warehouse: "Incoming shipment for PO {{po_number}}, ETA {{eta}}."
      - HTTP POST to demand-planning-service: update lead time actuals.

   c. "updated" (status change to "received"):
      - HTTP PUT to inventory-service: add received quantities to stock.
      - HTTP POST to quality-service `/v1/quality` to trigger incoming inspection.
      - HTTP POST to supplier-portal: "PO {{po_number}} received. Quality inspection scheduled."
      - HTTP PUT to procurement: mark PO as "received".

2. For all transitions: HTTP POST audit log with PO lifecycle event.
ERROR HANDLING: 3x retry, then alert procurement manager.
```

#### Prompt M-002: AI Demand Forecast Pipeline

```
Create a Make scenario titled "ERP-SCM: Demand Forecast Pipeline".

TRIGGER: Scheduled -- daily at 02:00 UTC (off-peak).

FLOW:
1. HTTP GET `/v1/inventory?low_stock=true&include_history=true` to fetch products needing forecast.
2. For each product (batch of 50):
   a. HTTP POST `/api/ai/forecast/{product_id}` with parameters: horizon=90, confidence=0.85.
   b. Parse response: predicted_demand[], confidence_intervals[], eoq, reorder_date.
   c. If forecast.confidence >= 0.85:
      - HTTP PUT `/v1/demand-planning/{product_id}` with forecast data.
      - If reorder_date <= today + 7 days:
        - HTTP POST notification to procurement: "Reorder recommended for {{product_name}} (SKU: {{sku}}). Suggested qty: {{eoq}}. AI Confidence: {{confidence}}."
   d. If forecast.confidence < 0.70:
      - Log low-confidence forecast for human review.
      - HTTP POST notification to demand planner: "Low confidence forecast for {{product_name}}. Manual review recommended."
3. HTTP POST `/api/ai/anomalies/detect` to run anomaly detection across all products.
4. For each detected anomaly:
   - HTTP POST `/v1/quality` or `/v1/inventory` alert depending on anomaly type.
   - HTTP POST notification with severity-based routing.
5. Aggregate results: forecasts generated, reorders triggered, anomalies detected.
6. HTTP POST audit log with pipeline summary.

ERROR HANDLING: Per-product isolation. Skip failed products, continue batch. Alert on > 10% failure rate.
```

#### Prompt M-003: Supplier Risk Monitoring

```
Create a Make scenario titled "ERP-SCM: Supplier Risk Monitor".

TRIGGER: Scheduled -- weekly on Monday at 06:00 UTC.

FLOW:
1. HTTP GET `/api/suppliers/ai/rankings` to fetch current AI-generated supplier rankings.
2. HTTP GET previous week's rankings from data store.
3. For each supplier:
   a. Compare current risk_score vs previous risk_score.
   b. If risk_score increased by > 10 points:
      - HTTP POST notification to procurement manager: "Supplier risk increase: {{supplier_name}} score changed from {{old}} to {{new}}. Key factors: {{risk_factors}}."
   c. If risk_score > 70 (High Risk):
      - HTTP POST escalation to supply chain director.
      - HTTP GET `/v1/procurement?supplier_id={{id}}&status=active` to find active POs.
      - If active POs exist: HTTP POST notification: "{{count}} active POs worth ${{total}} with high-risk supplier {{name}}. Consider mitigation."
   d. If trend = "declining" for 3+ consecutive weeks:
      - HTTP POST to supplier-portal: "Performance alert: Your scores have declined for 3 weeks. Schedule a review meeting."
4. Generate weekly supplier risk report.
5. HTTP POST to audit trail: weekly risk assessment summary.

ERROR HANDLING: Full supplier list processed regardless of individual failures. Alert on API timeout.
```

#### Prompt M-004: Inventory Reorder Automation

```
Create a Make scenario titled "ERP-SCM: Smart Reorder Engine".

TRIGGER: Webhook -- CloudEvent "erp.scm.inventory.updated" OR scheduled every 4 hours.

FLOW:
1. HTTP GET `/v1/inventory?quantity_below_reorder=true` to fetch items at or below reorder point.
2. For each item:
   a. HTTP POST `/api/inventory/{id}/optimize-reorder` to get AI-optimized reorder quantity (EOQ).
   b. HTTP GET `/v1/procurement?sku={{sku}}&status=open` to check if PO already exists.
   c. If no open PO:
      - HTTP GET preferred supplier from supplier rankings.
      - If auto_reorder_enabled AND confidence >= 0.82 AND amount < $10,000:
        - HTTP POST `/v1/procurement` to auto-create PO draft.
        - HTTP POST notification to procurement: "Auto-reorder PO draft created for {{product}}, qty {{eoq}}, supplier {{supplier}}. Review and approve."
      - Else:
        - HTTP POST notification to procurement: "Reorder recommended for {{product}}. Suggested qty: {{eoq}}, estimated cost: ${{cost}}. Manual PO required (above threshold or low confidence)."
   d. If open PO exists: skip, log "PO already pending."
3. For out-of-stock items (qty = 0):
   - HTTP POST urgent notification to procurement AND supply chain director: "STOCKOUT: {{product}} (SKU: {{sku}}). Revenue at risk: ${{daily_revenue}} per day."
4. HTTP POST audit log: reorder engine run summary.

ERROR HANDLING: Per-item isolation. Circuit breaker if > 5 PO creation failures in sequence.
```

#### Prompt M-005: Quality Non-Conformance Workflow

```
Create a Make scenario titled "ERP-SCM: NCR Workflow Automation".

TRIGGER: Webhook -- CloudEvent "erp.scm.quality.created" where type = "inspection" AND result = "fail" OR "conditional".

FLOW:
1. Parse event: extract inspection_id, po_id, supplier_id, items_failed, defect_types[], severity.
2. HTTP POST `/v1/quality/ncr` to create Non-Conformance Report (NCR):
   - Include inspection details, defect types, affected quantities.
3. Router by severity:
   a. "critical" (> 10% defect rate OR safety-related):
      - HTTP PUT `/v1/procurement/{po_id}` to hold PO (prevent further processing).
      - HTTP POST notification to quality manager + supply chain director: "Critical NCR raised for PO {{po_number}}. Supplier: {{supplier_name}}. Defect rate: {{rate}}%. PO placed on hold."
      - HTTP POST to supplier-portal: "Quality hold on PO {{po_number}}. Immediate corrective action required. Upload CAPA within 48h."
   b. "major" (5-10% defect rate):
      - HTTP POST notification to quality manager: "Major NCR. CAPA required within 5 business days."
      - HTTP POST to supplier-portal: "Quality issue on PO {{po_number}}. CAPA required."
   c. "minor" (< 5% defect rate):
      - HTTP POST notification to quality analyst: "Minor NCR logged. Review during next supplier assessment."
4. Create CAPA tracking entry: HTTP POST `/v1/quality/capa` with due dates based on severity.
5. Set SLA timer: critical=48h, major=5d, minor=15d.
6. On SLA expiry: escalate to next management level.
7. HTTP POST audit log with full NCR trail.

ERROR HANDLING: NCR creation is idempotent. Duplicate inspections with same ID are ignored.
```

#### Prompt M-006: Fleet Maintenance Scheduler

```
Create a Make scenario titled "ERP-SCM: Fleet Maintenance Automation".

TRIGGER: Scheduled -- daily at 05:00 UTC AND webhook on "erp.scm.fleet.updated".

FLOW:
1. HTTP GET `/v1/fleet?include_maintenance_schedule=true` to fetch all vehicles.
2. For each vehicle:
   a. Check mileage vs last service: if (current_mileage - last_service_mileage) > service_interval:
      - HTTP POST `/v1/fleet/{id}/maintenance` to schedule preventive maintenance.
      - HTTP POST notification to fleet manager: "Preventive maintenance due for {{vehicle_id}}. Mileage: {{mileage}}km since last service."
   b. Check days since last inspection: if > 90 days:
      - HTTP POST notification: "Inspection overdue for {{vehicle_id}}."
   c. Check fuel/charge level from webhook: if < 15%:
      - HTTP POST alert: "Low fuel/charge on {{vehicle_id}} at {{location}}. Current level: {{level}}%."
   d. Check tire pressure, engine temp from IoT telemetry (if available):
      - Anomaly detection: if values outside normal range, alert fleet manager.
3. Aggregate: vehicles needing service, total maintenance cost forecast.
4. Weekly summary (Monday): fleet utilization, maintenance costs, compliance status.
5. HTTP POST audit log.

ERROR HANDLING: Per-vehicle processing. Skip individual vehicle failures.
```

---

## 5. Prompt Usage Guidelines

### How to Use These Prompts

1. **Figma Make**: Paste prompt content (without backticks) into Figma's "Make a Design" feature.
2. **Iteration order**: Generate F-001 (tokens) and F-002 (components) first. They define the SCM-specific color language (inventory status, quality, supplier risk).
3. **Map components**: For F-006 (logistics), generate the map and data table as separate components, then compose.
4. **Warehouse floor**: F-016 (tablet warehouse) is designed for on-floor tablet use. Prioritize large touch targets and high contrast.
5. **Supplier portal**: F-007 is a separate user context. Generate with distinct branding/header.
6. **Make scenarios**: Import as blueprints. Configure webhook URLs, API endpoints, notification channels per environment.

### Prompt Customization Points

| Variable | Default | Description |
|----------|---------|-------------|
| `{{primary_color}}` | #0F766E | Brand primary (teal) |
| `{{font_family}}` | Inter | System font |
| `{{currency}}` | USD | Default currency display |
| `{{warehouse_name}}` | Main Distribution | Default warehouse for mockups |
| `{{supplier_name}}` | Chen Electronics Ltd | Sample supplier |
| `{{api_base_url}}` | `/v1` | API prefix |
| `{{ai_api_base}}` | `/api/ai` | AI endpoint prefix |

---

## 6. Output Packaging Convention

```
ERP-SCM-Design/
  tokens/
    light-theme.json
    dark-theme.json
  components/
    buttons.fig
    inputs.fig
    inventory-components.fig
    procurement-components.fig
    logistics-components.fig
    quality-components.fig
    ai-components.fig
    data-display.fig
    navigation.fig
    feedback.fig
  pages/
    desktop/
      ai-command-center.fig
      purchase-orders.fig
      inventory-dashboard.fig
      logistics-fleet.fig
      supplier-portal.fig
      quality-dashboard.fig
    tablet/
      dashboard.fig
      warehouse-floor.fig
      supplier-portal.fig
    mobile/
      dashboard.fig
      inventory-scanner.fig
      shipment-tracking.fig
      po-detail.fig
      fleet-tracking.fig
      quality-inspection.fig
  make-scenarios/
    po-lifecycle.json
    demand-forecast.json
    supplier-risk.json
    smart-reorder.json
    ncr-workflow.json
    fleet-maintenance.json
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Lighthouse Performance | >= 95 | Chrome DevTools |
| Lighthouse Accessibility | >= 95 | Chrome DevTools |
| First Contentful Paint | < 1.2s | WebPageTest, 4G |
| Largest Contentful Paint | < 2.0s | WebPageTest, 4G |
| Cumulative Layout Shift | < 0.1 | Chrome UX Report |
| Time to Interactive | < 3.0s | WebPageTest |
| Map render (500 markers) | < 200ms | Custom benchmark |
| Inventory table (10K rows) | < 300ms virtual scroll init | Custom benchmark |
| Barcode scan to result | < 500ms | Device test |
| AI forecast generation | < 3s per product | API SLO |
| Initial JS Bundle (gzip) | < 200 KB | Build output |
| API Read p99 | < 200ms | SLO dashboard |
| API Write p99 | < 500ms | SLO dashboard |
| Error rate | < 0.1% | Observability |
| Availability | >= 99.95% | Uptime monitor |

---

## 8. AIDD Handoff Gate Template

```
AIDD Design Handoff Checklist -- ERP-SCM
=========================================
[ ] WCAG 2.2 AA contrast verified (inventory status colors pass on all backgrounds)
[ ] Keyboard navigation documented for all pages
[ ] All interactive states: default, hover, active, focus, disabled, loading, error, empty
[ ] Light and dark mode complete (except warehouse floor view -- light/high-contrast only)
[ ] Desktop (1440px), tablet (1024px), mobile (390px) breakpoints delivered
[ ] Design tokens match exported JSON (including SCM-specific palette: inventory, quality, supplier risk)
[ ] Component naming: {category}/{component}/{variant}/{state}
[ ] data-testid attributes documented
[ ] AI confidence badges present on ALL AI-generated content (forecasts, risk scores, route suggestions, insights)
[ ] Currency formatting verified (USD, EUR, GBP with locale)
[ ] Map interactions documented (zoom, pan, marker click, route highlight)
[ ] Barcode scanner UX flow documented (scan, result, action, error)
[ ] Warehouse floor view optimized for tablet (large targets, high contrast)
[ ] Supplier portal isolation verified (supplier sees only own data)
[ ] Performance budget annotations (map lazy-load, virtual scroll boundaries, chart below-fold)
[ ] Analytics annotations (po_created, shipment_tracked, forecast_generated, scan_completed)
[ ] API endpoint mapping per data region
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial creation. 17 Figma prompts, 6 Make prompts covering all ERP-SCM services including AI capabilities. |
