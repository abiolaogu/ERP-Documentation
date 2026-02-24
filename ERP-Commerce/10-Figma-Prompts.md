# ERP-Commerce -- Figma Design Prompts

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

## 1. Design System Foundation

### Global Design Tokens

- **Primary Palette**: Deep Commerce Blue (#1A365D), Trade Green (#2E7D32), Alert Amber (#F57C00)
- **Typography**: Inter for UI, JetBrains Mono for data/codes
- **Spacing Grid**: 8px base unit
- **Border Radius**: 8px cards, 4px inputs, 12px modals
- **Shadows**: Elevation system (sm: 0 1px 2px, md: 0 4px 6px, lg: 0 10px 15px)
- **Responsive Breakpoints**: Mobile 375px, Tablet 768px, Desktop 1280px, Wide 1440px

---

## 2. Trade Command Center Dashboard

### Figma Prompt

> Design a comprehensive **Trade Command Center** dashboard for ERP-Commerce that serves as the executive overview for the entire multi-party trade network. Layout: full-width 1440px desktop view with responsive breakpoints.
>
> **Top Navigation Bar**: Logo "ERP-Commerce" at left, global search bar (Cmd+K), notification bell with badge count, user avatar with role dropdown, tenant switcher.
>
> **KPI Strip** (full-width, horizontal scrolling on mobile): 8 metric cards showing: (1) Total GMV with sparkline, (2) Active Orders count with status breakdown, (3) Inventory Health score as circular gauge, (4) Credit Utilization percentage bar, (5) Delivery SLA compliance percentage, (6) Active Trade Partners count, (7) POS Transactions today, (8) Marketplace Commission Revenue.
>
> **Main Grid** (12-column): Left 8 columns contain: (a) Order Volume chart (area chart, 30-day trend, segmented by order type B2B/B2B2C), (b) Geographic Heat Map showing order density by state/region with drill-down capability, (c) Top Products table (rank, SKU, name, units sold, revenue, trend arrow). Right 4 columns contain: (d) Live Activity Feed (real-time order/delivery/POS events with timestamps), (e) Alerts Panel (critical: stockouts, credit limits, SLA breaches; warning: approaching limits; info: new vendors, promotions), (f) Quick Actions grid (New Order, Adjust Price, Check Credit, Plan Route).
>
> **Color coding**: Use green for positive trends, amber for warnings, red for critical alerts. All charts should use the commerce blue palette. Include dark mode variant. Use Ant Design component patterns for consistency.

---

## 3. 13 Role-Specific Portal Prompts

### 3.1 Manufacturer Portal

> Design a **Manufacturer Portal** dashboard. Full desktop layout at 1440px. Left sidebar navigation with icons: Dashboard, Catalog, Pricing, Distribution, Analytics, Partners, Settings.
>
> **Dashboard**: (1) Production-to-Sell-Through funnel visualization showing units produced vs. distributed vs. sold at retail, (2) Distributor Performance scorecards in a 4-column grid showing each distributor's order volume, coverage, and payment timeliness, (3) Price Compliance heatmap showing actual selling prices vs. MSRP across regions, (4) Category Performance treemap showing revenue by product category sized by volume, (5) Coverage Map with territory boundaries showing distributor reach, (6) Pending Approvals queue for pricing overrides and new distributor requests.
>
> Include tabs for: Catalog Management (product grid with inline editing), Pricing Console (waterfall configurator), Distribution Map (interactive territory map), and Analytics (customizable report builder).

### 3.2 Distributor Portal

> Design a **Distributor Portal** for regional distributors managing B2B fulfillment. Left sidebar: Dashboard, Orders, Inventory, Routes, Fleet, Customers, Van Sales, Reports.
>
> **Dashboard**: (1) Today's Operations summary card showing orders to fulfill, deliveries in transit, drivers active, (2) Order Pipeline kanban board (New > Confirmed > Picking > Packed > Shipped > Delivered), (3) Warehouse Capacity utilization bar chart per location, (4) Route Map showing today's delivery routes with GPS dots for active drivers, (5) Top Customers by revenue table with credit utilization badges, (6) Inventory Alerts panel showing low-stock and expiring items.
>
> **Order Fulfillment View**: Split screen with order details on left (line items, quantities, pricing) and warehouse location map on right showing pick paths. Include pack slip printing action and carrier assignment dropdown.

### 3.3 Wholesaler Portal

> Design a **Wholesaler Portal** focused on bulk purchasing and trade credit management. Navigation: Dashboard, Catalog Browse, Orders, Credit Account, Inventory, Analytics.
>
> **Dashboard**: (1) Credit Summary card showing limit, utilized, available with circular progress, (2) Pending Orders with estimated delivery dates, (3) Price Comparison table showing prices from multiple distributors for the same product, (4) Reorder Suggestions AI-powered list with one-click reorder, (5) Order History with filtering and export.
>
> **Catalog Browse**: Grid/List toggle for products with faceted search. Each product card shows: image, name, SKU, wholesaler price, stock availability indicator (green/amber/red), MOQ badge, add-to-cart button with quantity stepper. Cart sidebar slides in from right.

### 3.4 Retailer Portal

> Design a **Retailer Portal** optimized for small shop owners, mobile-first with desktop support. Bottom navigation on mobile: Home, Order, POS, Credit, Profile.
>
> **Home**: (1) Quick Reorder section with last 5 orders as one-tap reorder cards, (2) Promotional Banner carousel showing active deals, (3) Order Status tracker for pending orders with step visualization, (4) Credit Balance card with payment due date countdown, (5) Notifications list.
>
> **Order Flow**: Category grid with large touch targets, product cards with quantity stepper, cart summary with credit/cash toggle, delivery date selector, confirm button. Simple 3-step checkout: Cart > Delivery > Confirm. Use large fonts (16px+ body) for readability on small screens.

### 3.5 Supermarket Portal

> Design a **Supermarket Portal** for large-format retail chain management. Navigation: Dashboard, Planogram, Promotions, Inventory, Orders, Analytics, Merchandising.
>
> **Dashboard**: (1) Store Performance comparison table (all locations), (2) Promotional Calendar timeline view showing active and upcoming promotions, (3) Planogram Compliance scores per category with thumbnails, (4) Fast-Moving Items leaderboard, (5) Shrinkage/Wastage tracker, (6) Vendor Delivery Performance scorecards.
>
> **Planogram View**: Drag-and-drop shelf layout editor with product blocks. Split view showing defined planogram vs. actual shelf photo. Compliance score overlay with color-coded indicators.

### 3.6 Warehouse Portal

> Design a **Warehouse Portal** for warehouse managers and operators. Navigation: Dashboard, Receiving, Putaway, Picking, Packing, Shipping, Inventory, Zones.
>
> **Dashboard**: (1) Today's Activity summary (items to receive, pick, ship), (2) Picking Queue with priority sorting and wave assignment, (3) Zone Capacity visualization showing fill levels per warehouse zone as a floor plan, (4) Dock Schedule showing incoming/outgoing shipments on timeline, (5) Worker Productivity metrics (picks per hour, accuracy rate).
>
> **Picking View**: Optimized pick path visualization on warehouse floor plan. Pick list with barcode scan confirmation. Progress bar showing completed vs. remaining items.

### 3.7 Delivery Company Portal

> Design a **Delivery Company Portal** for 3PL logistics providers. Navigation: Dashboard, Deliveries, Routes, Fleet, Drivers, Performance, Billing.
>
> **Dashboard**: (1) Live Delivery Map with all active drivers as pins, color-coded by status (on-route: blue, delivering: green, returning: amber), (2) Today's Metrics cards (deliveries completed, on-time %, failed attempts, average delivery time), (3) Route Efficiency chart comparing planned vs. actual times, (4) Driver Leaderboard, (5) Pending Assignments queue.
>
> **Route Planner**: Interactive map with drag-to-reorder stops. Vehicle capacity indicator. Time window visualization per stop. One-click route optimization button.

### 3.8 Driver Portal (Mobile-First)

> Design a **Driver Mobile App** optimized for Android (Sunmi devices, standard smartphones). Bottom navigation: Route, Deliveries, Earnings, Profile.
>
> **Route View**: Full-screen map with turn-by-turn navigation. Next stop highlighted with large address text and estimated arrival time. Swipe-up bottom sheet showing stop details (customer name, items, special instructions). Large "Navigate" button linking to maps app.
>
> **Delivery View**: Customer info card, item checklist with scan verification, Payment collection interface (cash/mobile money), POD capture buttons (Signature pad, Camera for photo, OTP input field). Large "Complete Delivery" action button (minimum 56px height). All interactions designed for one-handed operation.

### 3.9 Agent Portal

> Design an **Agent Portal** for marketplace and trade agents who acquire customers and earn commissions. Navigation: Dashboard, Customers, Commissions, Tasks, Leaderboard.
>
> **Dashboard**: (1) Commission Summary with current month earnings and payout schedule, (2) Customer Pipeline funnel (Lead > Onboarding > Active > Retained), (3) Task List with priority badges, (4) Earnings Chart (monthly trend), (5) Referral Link with share buttons (WhatsApp, SMS, Copy).
>
> **Commission Detail**: Transaction-level breakdown showing order, commission rate, amount, status (pending/approved/paid).

### 3.10 Brand Manager Portal

> Design a **Brand Manager Portal** for brand/product managers overseeing market performance. Navigation: Dashboard, Products, Pricing, Competitors, Distribution, Campaigns.
>
> **Dashboard**: (1) Brand Health Index composite score (distribution, pricing compliance, share of shelf), (2) Sales Performance by channel and region (multi-series chart), (3) Competitive Positioning matrix (price vs. market share scatter plot), (4) Distribution Coverage map with brand-specific overlay, (5) Price Exception Report table showing off-policy prices.
>
> **Competitor Monitor**: Side-by-side product comparison cards showing own price vs. competitor prices with trend indicators. Alert configuration for price changes.

### 3.11 Merchandiser Portal (Mobile-First)

> Design a **Merchandiser Mobile App** for in-store merchandising operations. Bottom navigation: Tasks, Audit, Reports, Profile.
>
> **Audit Flow**: (1) Select store from today's visit list, (2) Camera opens for shelf photo capture, (3) AI overlay shows planogram compliance with green/red highlights, (4) Non-compliance items listed with corrective actions, (5) Competitor product presence checklist, (6) Share-of-shelf measurement by brand, (7) Submit audit with GPS timestamp.
>
> **Task List**: Cards showing store name, address, task type (audit, setup, promotion check), due time, priority badge. One-tap navigation to store location.

### 3.12 Field Sales Portal (Mobile-First)

> Design a **Field Sales Mobile App** for sales representatives covering territories. Bottom navigation: Route, Customers, Orders, Dashboard.
>
> **Route View**: Today's beat plan shown on map with customer pins. Visited/unvisited status indicators. Optimized route line. Check-in button at each stop with GPS verification.
>
> **Customer Visit**: Customer profile card (last order, credit status, outstanding balance). Quick Order entry with product search and quantity input. Previous order history for quick reorder. Notes and follow-up task creation. Photo capture for shelf/display conditions.
>
> **Order Capture**: Streamlined product selection with recent/frequent items at top. Cart with running total. Submit order with delivery date selection. Offline-capable with queue indicator.

### 3.13 Trade Marketing Portal

> Design a **Trade Marketing Portal** for campaign management and promotion analytics. Navigation: Dashboard, Campaigns, Promotions, Budget, Analytics, Calendar.
>
> **Dashboard**: (1) Campaign Performance overview cards (active campaigns, total budget, spend rate, ROI), (2) Promotion Redemption funnel (distributed > redeemed > revenue generated), (3) Regional Performance heatmap showing campaign effectiveness, (4) Budget Tracker stacked bar chart (allocated vs. spent by category), (5) Top Performing Promotions table with ROI sorting.
>
> **Campaign Builder**: Step-by-step wizard: (a) Define campaign (name, objective, dates), (b) Select products/categories, (c) Configure promotion mechanics (discount, BOGO, bundle), (d) Set targeting rules (regions, store types, customer segments), (e) Allocate budget, (f) Review and launch.

---

## 4. POS Checkout Interface

### Figma Prompt

> Design a **POS Checkout Interface** for touch-screen terminals (10-inch Sunmi T2 and 14-inch desktop displays). Split layout: Left 60% for product/cart area, Right 40% for payment and actions.
>
> **Left Panel (Product/Cart)**: (a) Search bar with barcode scan icon at top, (b) Category quick-select ribbon (horizontal scroll of category pills), (c) Product grid (4x3 on desktop, 3x3 on 10-inch) with product image, name, price, and tap-to-add. (d) Cart area below product grid showing line items with quantity +/- buttons, line total, and swipe-to-remove. (e) Cart footer showing subtotal, tax, discounts, and bold total.
>
> **Right Panel (Payment)**: (a) Numpad for manual price/quantity entry, (b) Payment method buttons (Cash, Card, Mobile Money, Credit, Split), (c) Cash payment: amount tendered input and change calculation, (d) Card payment: "Tap/Insert/Swipe" instruction with terminal status, (e) Action buttons row: Hold Order, Discount, Void, Print, New Sale. (f) Customer lookup with loyalty points display.
>
> **Receipt Preview**: Slide-up panel showing receipt with store header, items, totals, payment method, barcode, and footer. Print and email/SMS send buttons.
>
> **Offline Indicator**: Top banner showing connection status. Green "Online" or amber "Offline - X transactions queued". Must be highly visible without obstructing workflow.

---

## 5. Order Orchestration Dashboard

### Figma Prompt

> Design an **Order Orchestration Dashboard** showing the real-time flow of multi-party orders through the commerce platform. Full 1440px width.
>
> **Pipeline View** (top half): Horizontal swim lanes representing order stages (Draft > Policy Check > Credit Check > Approved > Splitting > Allocated > Picking > Packed > Shipped > Delivered > Settled). Each order represented as a card in its current lane. Cards show order number, customer, total amount, and time-in-stage indicator. Color-coded urgency (green: on track, amber: approaching SLA, red: SLA breach). Drag capability to manually advance orders.
>
> **Detail Panel** (bottom half, appears on order card click): Order timeline visualization showing each stage transition with timestamps. Sub-order breakdown if split. Fulfillment source map showing warehouse and delivery route. Action buttons: Approve, Escalate, Cancel, Add Note.
>
> **Filters**: Date range, order type (B2B/B2B2C), channel, status, customer, seller. Quick filter chips for common views (SLA Breached, Pending Approval, High Value).

---

## 6. Inventory Map Visualization

### Figma Prompt

> Design an **Inventory Map** interface showing real-time stock levels across all locations (warehouses, stores, vans, consignment locations). Full 1440px width.
>
> **Map View**: Interactive map showing all inventory locations as circles sized by total stock value. Color-coded: green (healthy stock), amber (below reorder point), red (stockout). Click location to see stock detail panel.
>
> **Location Detail Panel** (slides in from right): Location name, type, address. Stock level table: Product, Variant, On-Hand, Reserved, Available, Reorder Point, Status. Lot/Serial detail expandable rows. Transfer action button. Replenishment order action.
>
> **Dashboard Strip** (above map): Total locations count, aggregate stock value, items below reorder, upcoming expiries, pending transfers count.
>
> **List View** (toggle from map): Full-width data table with all stock records, sortable and filterable. Export to CSV button.

---

## 7. Trade Credit Manager Interface

### Figma Prompt

> Design a **Trade Credit Manager** interface for credit officers to manage trade credit accounts across the platform. Full 1440px width.
>
> **Portfolio Overview** (top section): (1) Total credit exposure gauge chart, (2) Portfolio health donut chart (current/30day/60day/90day aging), (3) Credit utilization histogram, (4) Default rate trend line chart (12 months).
>
> **Account Table** (main section): Searchable, sortable table showing: Customer Name, Credit Limit, Utilized, Available, Payment Terms, Risk Category (badge: Low/Medium/High/Critical), Last Payment Date, Days Outstanding, AI Score (0-1000 with color bar), Actions (Review, Adjust Limit, Hold Account).
>
> **Account Detail** (slide-out panel on row click): Customer profile summary, Credit score breakdown (radar chart showing each scoring factor), Transaction history timeline, Collections action history, Related orders list, Adjustment form (new limit, new terms, notes).
>
> **Alerts Panel** (right sidebar): Accounts approaching limit (>80% utilization), Overdue accounts sorted by days overdue, Pending credit applications queue, Recent AI score changes.

---

## 8. Route Planner Interface

### Figma Prompt

> Design a **Route Planner** interface for logistics operations, optimizing delivery routes for the day. Full 1440px width.
>
> **Map Area** (left 65%): Interactive map showing planned routes as colored polylines (one color per driver/vehicle). Delivery stops as numbered pins. Depot/warehouse as star icon. Real-time driver location as pulsing dot. Click stop to see delivery details popup.
>
> **Control Panel** (right 35%): (a) Route Summary showing: total stops, total distance, estimated duration, vehicle capacity utilization bar, (b) Stop List in delivery order with drag-to-reorder, each showing: stop number, customer name, address, time window, items count, weight, status icon, (c) Optimization Controls: "Optimize Route" button with options (minimize distance, minimize time, balance workload), (d) Vehicle/Driver assignment dropdowns, (e) Add Stop button with address search.
>
> **Timeline View** (bottom, collapsible): Gantt-style view showing each driver's schedule with time blocks for driving, delivery, and breaks. Time window markers for each stop.

---

## 9. B2B Marketplace Interface

### Figma Prompt

> Design a **B2B Marketplace** interface for the vendor-facing and buyer-facing marketplace experience.
>
> **Buyer View**: (a) Marketplace homepage with category grid, featured vendors carousel, promotional banners, (b) Vendor directory with search, filters (category, rating, location, minimum order), vendor cards showing logo, name, rating stars, product count, commission rate badge, (c) Vendor store page with vendor profile header, product grid, reviews section, (d) Multi-vendor cart with items grouped by vendor, separate shipping per vendor.
>
> **Vendor Dashboard**: (a) Sales Overview cards (GMV, orders, commission, payout), (b) Product Listing Manager with status indicators, (c) Order Management table, (d) Commission Statement with downloadable invoices, (e) Dispute Queue with response deadline countdown, (f) Analytics: traffic, conversion, average order value.
>
> **Admin View**: Vendor Verification Queue showing pending applications with document preview, approve/reject actions, KYC status checklist.
