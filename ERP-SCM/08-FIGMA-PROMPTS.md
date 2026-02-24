# ERP-SCM Figma Design Prompts

## 1. Overview

This document provides detailed design prompts for generating high-fidelity UI mockups in Figma (or any design tool) for all major ERP-SCM screens. Each prompt specifies layout, components, color palette, interactions, and data visualization requirements. The design system follows a dark-theme enterprise aesthetic consistent with the existing React 18 + Tailwind CSS implementation.

---

## 2. Design System Foundation

**Color Palette**:
- Background: `#0f172a` (slate-900)
- Card Background: `#1e293b` (slate-800)
- Sidebar: `#0f172a` with border `#334155`
- Primary: `#3b82f6` (blue-500)
- Accent: `#8b5cf6` (purple-500)
- Success: `#10b981` (emerald-500)
- Warning: `#f59e0b` (amber-500)
- Danger: `#ef4444` (red-500)
- Cyan: `#06b6d4` (cyan-500)
- Text Primary: `#f8fafc` (slate-50)
- Text Secondary: `#94a3b8` (slate-400)

**Typography**: Inter font family, weights 400/500/600/700

**Component Library**: Cards with rounded-xl corners and subtle border (slate-700), badge pills, icon buttons using Lucide icon set

---

## 3. Screen Prompts

### 3.1 SCM Command Center Dashboard

**Prompt**:

Design a dark-themed enterprise SCM command center dashboard at 1440x900 resolution. The layout has a 256px left sidebar with the SCM AI logo (lightning bolt icon in a blue rounded square), navigation items with Lucide icons (Dashboard, Products, Inventory, Suppliers, Orders, Logistics, AI Center, Alerts), and an "AI Engine Active" status badge at the bottom.

The main content area (right side) contains:

**Top row**: A header with "Dashboard" title, subtitle "AI-powered supply chain overview", and a health score badge on the right showing a shield icon with percentage and letter grade (e.g., "82.5% (B)"). Color the shield green for A/B, yellow for C, red for D/F.

**KPI cards row**: Eight metric cards in a 4-column grid. Each card has a small gray label, a large bold number, and a colored icon in a rounded square. Cards are:
1. Total Products (blue Package icon)
2. Suppliers (purple Users icon)
3. Total Orders (cyan ShoppingCart icon)
4. Active Shipments (green Truck icon)
5. Inventory Value (yellow DollarSign, formatted as currency)
6. Low Stock Items (red AlertTriangle)
7. Revenue (emerald TrendingUp, formatted as currency)
8. Fulfillment Rate (indigo BarChart3, formatted as percentage)

**Middle row**: Three equal-width cards:
1. **Supply Chain Health** - vertical bar chart with four metrics (Inventory Health, Supplier Health, Fulfillment Health, Logistics Health). Each has a colored progress bar (green >= 80, yellow 60-79, red < 60) with percentage label.
2. **Order Status** - donut/pie chart showing Pending vs Delivered orders in blue and purple.
3. **AI Alerts** - scrollable list of alerts with severity icons (red triangle for critical, orange for high, yellow for medium), alert title, and description text.

**Bottom row**: Full-width "AI Insights & Recommendations" card with a 2x2 grid of insight tiles. Each tile has an impact badge (red for high, yellow for medium, blue for low), category label, title, description, and a blue recommendation text.

All cards have dark slate-800 backgrounds with subtle slate-700 borders and rounded-xl corners.

---

### 3.2 Procurement Dashboard

**Prompt**:

Design a procurement management dashboard at 1440x900 with the standard SCM sidebar. The main area contains:

**Header**: "Procurement" title with action buttons: "+ New Requisition" (blue), "+ New RFQ" (outlined), "+ New PO" (outlined).

**KPI strip**: Five mini-cards in a horizontal row:
1. Open Requisitions (count with trend arrow)
2. Active RFQs (count)
3. POs This Month (count with month-over-month delta)
4. Pending 3-Way Matches (count, red if > 10)
5. Avg Vendor Score (0-100 with circular gauge)

**Main content in tabs**: Tabs for "Purchase Orders", "Requisitions", "RFQs", "Contracts", "Vendor Scorecards"

**Purchase Orders tab (default)**:
- Data table with columns: PO Number (link), Supplier, Status (badge: draft=gray, pending=yellow, confirmed=blue, shipped=cyan, delivered=green, cancelled=red), Total Amount, Expected Delivery, Created Date
- Filter bar: status dropdown, supplier dropdown, date range picker, search box
- Pagination controls

**Side panel (on row click)**: Slide-in panel showing PO detail with:
- PO header information
- Line items table (product, qty, unit price, total)
- Status timeline (vertical with dates)
- 3-way match status indicator
- Action buttons: Approve, Receive, Cancel

Include a mini vendor scorecard radar chart showing 5 axes (Delivery, Quality, Price, Financial, Communication) for the selected PO's supplier.

---

### 3.3 Warehouse Floor Plan

**Prompt**:

Design an interactive warehouse floor plan view at 1440x900 with the standard SCM sidebar.

**Header**: "Warehouse Management" with a warehouse selector dropdown and view toggles (Floor Plan / List / Heatmap).

**Floor Plan view**:
- Large canvas area showing a top-down warehouse layout
- The warehouse is divided into colored zones:
  - Receiving dock area (blue, left side)
  - Bulk storage zone (gray, center)
  - Pick zone A (green, right-center)
  - Pick zone B (yellow, right-center below A)
  - Packing area (purple, right side)
  - Shipping dock (cyan, far right)
  - Returns/QC zone (red-outlined, bottom-left)
- Each zone shows aisles as thin rectangles with rack labels (A1, A2, A3...)
- Bins within aisles shown as small squares, color-coded by occupancy (green = occupied, gray = empty, red = reserved)
- A floating tooltip appears on hover showing: Bin ID, Product, Quantity, Last Activity

**Right sidebar panel** (300px):
- Zone statistics: total bins, occupancy %, active picks
- Active operations list:
  - Receiving in progress (truck icon + PO number)
  - Active pick waves (person icon + wave ID)
  - Packing in progress (box icon + order count)
- Quick actions: Create Pick Wave, Schedule Receiving, Run Slotting Optimization

**Bottom bar**: Real-time activity feed showing recent events ("Bin A1-03-02 putaway completed", "Pick wave W-2026-0234 started") as a scrolling ticker.

---

### 3.4 Production Scheduler Gantt Chart

**Prompt**:

Design a production scheduling Gantt chart interface at 1440x900 with the standard SCM sidebar.

**Header**: "Production Scheduler" with controls:
- Date range selector (week/2-week/month view)
- "Today" button to jump to current date
- View mode toggle: Gantt / Calendar / List
- "+ New Production Order" button (blue)
- "Run MRP" button (outlined, with gear icon)

**Gantt Chart area**:
- Y-axis: Work centers listed vertically (e.g., "CNC Machine 1", "Assembly Line A", "Welding Station 2", "Paint Booth", "QC Station")
- X-axis: Time axis with day columns, hour gridlines within each day
- Work order bars: colored rectangles spanning their scheduled duration
  - Blue: scheduled/not started
  - Green: in progress
  - Orange: behind schedule
  - Purple: completed
  - Red: blocked/waiting
- Each bar shows: Production Order number, Product name, Quantity
- Bars are draggable for rescheduling
- Dependencies shown as arrows between linked operations

**Capacity utilization**: Below each work center row, a thin bar showing capacity utilization (green < 80%, yellow 80-95%, red > 95%)

**Right panel** (320px, collapsible):
- Selected work order details
- Material availability check (green checkmarks for available, red X for short)
- Predecessor/successor operations
- Actual vs. planned progress (percentage bar)
- Actions: Start, Complete, Split, Reschedule

**Bottom**: A "Bottleneck Alert" banner showing work centers that are over-capacity in the current view period, with utilization percentage.

---

### 3.5 Demand Planner Forecast Chart

**Prompt**:

Design a demand planning forecast dashboard at 1440x900 with the standard SCM sidebar.

**Header**: "Demand Planning" with product selector dropdown, forecast horizon selector (30/60/90 days), and "Generate Forecast" button (blue with sparkle/AI icon).

**Main chart area** (full width, 400px height):
- Combined line chart with:
  - Solid blue line: Historical actual demand (past 90 days)
  - Dashed blue line: AI forecast (next 30 days)
  - Light blue shaded area: 95% confidence interval (upper/lower bounds)
  - Orange dots: Consensus plan adjustments (if any)
  - Green dots: Promotional events marked on timeline
- X-axis: Dates (daily ticks)
- Y-axis: Quantity (auto-scaled)
- Interactive crosshair tooltip showing date, actual/forecast value, confidence bounds
- Legend at the top showing all series

**Below the chart, two cards side by side**:

**Left card**: "Forecast Accuracy Metrics" with:
- MAPE gauge (semicircular gauge, green < 15%, yellow 15-25%, red > 25%)
- MAD value
- Bias indicator (arrow up/down showing over/under forecast)
- Model used badge (e.g., "ensemble_es_rf")
- Trend chart showing MAPE over last 6 months

**Right card**: "Reorder Recommendations" with:
- Current vs AI-recommended reorder point (before/after comparison)
- Current vs AI-recommended EOQ
- Safety stock calculation details
- Service level target (95%)
- "Apply AI Recommendations" button

**Bottom**: "Product Forecast Summary" table with columns: Product, Current MAPE, Forecast Trend (up/down arrow), 30-Day Forecast Total, Recommended Action, Last Updated. Sortable columns.

---

### 3.6 Logistics Tracker Map

**Prompt**:

Design a logistics shipment tracking dashboard at 1440x900 with the standard SCM sidebar.

**Header**: "Logistics & Transportation" with tabs: "Active Shipments", "Route Optimizer", "Carriers", "Freight Audit".

**Active Shipments tab** (default):

**Left panel** (400px):
- Shipment list with:
  - Tracking number (bold)
  - Status badge (color-coded: pending=gray, picked_up=blue, in_transit=cyan, out_for_delivery=yellow, delivered=green, returned=red)
  - Carrier icon and name
  - Origin -> Destination
  - ETA with countdown
- Search and filter controls at top
- Sort by: status, ETA, carrier

**Right panel** (map):
- Full interactive map (dark theme, Mapbox/Google Maps dark style)
- Shipment locations shown as colored pins matching their status
- Selected shipment shows:
  - Route polyline from origin to destination
  - Current position marker (animated pulsing dot)
  - Origin marker (circle) and destination marker (flag)
  - Estimated remaining route in dashed line
- Vehicle icon for fleet-tracked shipments

**Bottom drawer** (expandable, 200px):
- Shipment detail for selected item:
  - Timeline of tracking events (vertical timeline with location, timestamp, status)
  - Carrier information
  - Weight, cost, Incoterms
  - AI-optimized route comparison (original distance vs. optimized)
  - ETA prediction confidence

**Route Optimizer tab**:
- Map with draggable stop pins
- Sidebar showing stop list with reorder handles
- "Optimize" button that animates the route recalculation
- Before/After comparison: distance, time, fuel cost savings

---

### 3.7 Quality Management Dashboard

**Prompt**:

Design a quality management dashboard at 1440x900 with the standard SCM sidebar.

**Header**: "Quality Management" with tabs: "Dashboard", "Inspections", "NCRs", "CAPAs", "SPC Charts".

**Dashboard tab**:

**KPI row** (4 cards):
1. Incoming Inspection Pass Rate (percentage with trend, green gauge)
2. Open NCRs (count, red if > 5)
3. CAPA On-Time Closure Rate (percentage with trend)
4. Supplier Quality Index (score 0-100)

**Middle row** (3 cards):

**Left**: "Inspection Summary" - stacked bar chart showing pass/fail/hold counts over last 12 weeks. Bars colored green (pass), red (fail), yellow (hold). X-axis: weeks, Y-axis: count.

**Center**: "NCR Severity Distribution" - horizontal bar chart or pie chart showing critical/major/minor NCR counts. Color: red/orange/yellow.

**Right**: "CAPA Aging" - horizontal bars showing CAPAs by age bucket (< 30 days = green, 30-60 days = yellow, 60-90 days = orange, > 90 days = red). Count in each bucket.

**Bottom**: "SPC Control Chart" (full width) showing:
- Line chart with individual measurements as dots
- Center line (CL) as solid green horizontal line
- Upper Control Limit (UCL) as dashed red line
- Lower Control Limit (LCL) as dashed red line
- Out-of-control points highlighted as red dots with alert icon
- X-axis: measurement sequence or time
- Y-axis: measurement value
- Product and characteristic selectors above the chart

---

### 3.8 Fleet Management Monitor

**Prompt**:

Design a fleet management monitoring dashboard at 1440x900 with the standard SCM sidebar.

**Header**: "Fleet Management" with tabs: "Live Map", "Vehicles", "Drivers", "Maintenance", "Fuel", "Compliance".

**Live Map tab**:

**Main area**: Full-width dark-themed map showing:
- Vehicle markers at real-time GPS positions
  - Green marker: moving/on route
  - Blue marker: idle/parked
  - Orange marker: maintenance needed
  - Red marker: alert condition (speeding, breakdown)
- Vehicle labels showing: registration, driver name, current speed
- Route traces for active trips (colored polylines)
- Geofence boundaries for depot/warehouse locations (dashed outlines)

**Right panel** (350px):
- "Fleet Summary" card:
  - Total vehicles: X
  - Active: Y (green number)
  - Idle: Z (blue number)
  - In Maintenance: W (orange number)
- "Active Trips" list:
  - Each item shows: vehicle, driver, from -> to, ETA
  - Progress bar showing trip completion percentage
- "Alerts" list:
  - Speeding alert (vehicle, speed, limit)
  - Maintenance overdue (vehicle, days overdue)
  - License expiring (driver, days until expiry)

**Bottom bar**: Fleet KPIs strip:
- Avg Fuel Efficiency (L/100km)
- Fleet Utilization Rate (%)
- On-Time Delivery Rate (%)
- Safety Score (0-100)
- Total Distance Today (km)

**Vehicles tab**: Data table with columns: Registration, Make/Model, Type, Status, Odometer, Next Service, Insurance Expiry, Assigned Driver, Utilization %.

---

### 3.9 Supplier Portal

**Prompt**:

Design a supplier-facing portal at 1440x900 with a simplified navigation. The portal has a different color scheme: white/light background with blue accent (professional, clean).

**Header bar**: Company logo placeholder, supplier company name, notification bell (with badge count), user avatar with dropdown.

**Left navigation** (200px, light gray):
- Dashboard (home icon)
- Purchase Orders (document icon)
- ASN (truck icon)
- Invoices (receipt icon)
- Payments (dollar icon)
- Documents (folder icon)
- Profile (settings icon)

**Dashboard view**:

**Welcome banner**: "Welcome, [Supplier Name]" with a summary sentence about open items.

**Quick stats row** (4 cards, white with colored left border):
1. Open POs: count (blue left border)
2. Pending ASNs: count (orange left border)
3. Invoices Awaiting Payment: count and total amount (green left border)
4. Next Payment: date and amount (purple left border)

**Two-column layout below**:

**Left**: "Recent Purchase Orders" table showing: PO Number, Date, Items, Total, Status (New/Acknowledged/Shipped), Action button (Acknowledge/View)

**Right**: "Payment Schedule" showing upcoming payment dates as a timeline or table: Invoice Number, Amount, Due Date, Status (Pending/Scheduled/Paid with green checkmark)

**Action cards at bottom**:
- "Submit ASN" card with truck illustration and "+ New ASN" button
- "Submit Invoice" card with receipt illustration and "+ New Invoice" button
- "Upload Documents" card with folder illustration and "Upload" button

**PO Detail view** (on PO click):
- PO header: PO number, date, delivery address, Incoterms
- Line items table: item, description, qty, unit price, total
- Status timeline: Created -> Acknowledged -> Shipped -> Received
- Action buttons: Acknowledge, Create ASN, Create Invoice
- Chat/notes section for communication with buyer

Use a clean, professional design with ample white space, subtle shadows, and rounded corners. No dark theme for the supplier portal.
