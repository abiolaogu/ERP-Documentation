# Figma & Make Prompts — ERP-Commerce
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides structured, copy-paste-ready prompts for generating production-grade UI designs in Figma Make and automation workflows in Make (formerly Integromat) for the ERP-Commerce module. The platform is an omnichannel commerce system supporting B2B and B2C operations: product catalogs, multi-warehouse inventory, order management, point-of-sale (POS), marketplace operations, logistics and distribution, pricing engines, trade credit, and self-service portals. All prompts reference the 10 backend microservices (`catalog-service`, `distribution-service`, `inventory-service`, `logistics-service`, `marketplace-service`, `order-service`, `portal-service`, `pos-service`, `pricing-service`, `trade-credit-service`) and are designed for multi-tenant, entitlement-aware deployment via ERP-Platform with ERP-IAM authentication (OIDC/JWT). Design intelligence draws from Amazon, Shopify Plus, Alibaba, Jumia, Coupang, Square, Stripe, and Flexport.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance across all surfaces
- Minimum 44x44px touch targets on all interactive elements
- Clear visual states: default, hover, focus (2px ring), active, disabled, loading, error
- Plain-language labels; commerce terminology with contextual tooltips for B2B jargon (MOQ, RFQ, COD, FOB)
- Color-blind-safe palette; never rely on color alone (stock levels, order status use icons + text + color)
- Keyboard navigation and command palette (Cmd+K) parity for power users
- Screen reader announcements for dynamic content (cart updates, order status changes, stock alerts)

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading for all page modules
- Initial route gzip bundle < 220KB
- Skeleton UIs for every data-dependent component (product grids, order tables, analytics charts)
- Optimistic UI for safe mutations (cart updates, wishlist actions, status changes)
- Autosave for long forms (product creation, order editing)
- Image lazy loading with blur-up placeholders; responsive srcset for product images
- Infinite scroll on product listings with "Back to top" floating button

### 2.3 Reliability, Trust, And Safety
- Recovery paths for all error states (retry button, support link, fallback view)
- Trust signals: payment security badges, seller verification marks, delivery guarantees, buyer protection
- Audit trail badges on financial transactions: "Recorded by [user] on [date]"
- Two-step confirmation for destructive actions (cancel order, void invoice, delete product, issue refund)
- Tenant isolation indicator: organization/merchant name always visible in header
- Human-in-the-loop for AI recommendations (pricing suggestions, restock alerts, fulfillment routing)
- Financial integrity: transactions append-only with correction/reversal workflow

### 2.4 Observability And Testability
- Analytics events on: page view, product view, add-to-cart, checkout step, search query, filter change, POS transaction
- Error events on: API failure, payment failure, inventory conflict, form validation failure
- Performance instrumentation: FCP, LCP, CLS, TTI per route
- Conversion funnel tracking: view > add-to-cart > checkout > payment > confirmation
- Feature flag awareness: components degrade gracefully when features are disabled

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

```text
Create a comprehensive design system for an enterprise omnichannel commerce platform called "ERP-Commerce" that manages catalogs, inventory, orders, POS, marketplace operations, logistics, distribution, pricing, trade credit, and merchant portals for B2B/B2C commerce.

DESIGN TOKENS:

Color System:
- Primary: Deep Navy (#0F172A) for navigation, headers, and primary containers — conveys authority and trust
- Secondary: Commerce Green (#059669) for positive actions, confirmations, purchase CTAs, and revenue indicators
- Accent: Electric Cyan (#06B6D4) for interactive elements, links, and data visualization highlights
- Warning: Rich Amber (#D97706) for alerts, pending states, low-stock warnings, SLA caution
- Danger: Crimson Rose (#E11D48) for destructive actions, out-of-stock, SLA breaches, payment failures
- Info: Sapphire (#0284C7) for informational badges, tooltips, and help content
- Neutral: Slate scale (50-950) for text hierarchy, borders, backgrounds, surfaces
- Surface hierarchy: Base (#F8FAFC), Raised (#FFFFFF with shadow-sm), Overlay (#FFFFFF with shadow-lg)
- Order status colors: Pending (#94A3B8), Confirmed (#3B82F6), Processing (#8B5CF6), Shipped (#06B6D4), Delivered (#059669), Cancelled (#EF4444), Returned (#F59E0B)
- Stock indicator colors: In-Stock (#059669), Low-Stock (#D97706), Out-of-Stock (#EF4444), Pre-Order (#3B82F6)
- Portal accent overrides: Merchant (Navy), Buyer (Green), Admin (Amber), Warehouse (Indigo), Driver (Cyan)
- Dark mode: Slate-950 base, Slate-900 raised, all colors adjusted for dark contrast

Typography:
- Font family: Inter (UI and body) / JetBrains Mono (SKUs, order IDs, prices in tables, API keys)
- Scale: 11px (caption), 12px (label), 13px (body-sm), 14px (body), 16px (subtitle), 20px (title), 24px (heading), 32px (display), 40px (hero price)
- Weight tokens: regular (400), medium (500), semibold (600), bold (700)
- Line-height tokens: tight (1.2), normal (1.5), relaxed (1.75)
- Numeric tabular figures for all prices, quantities, financial data, and tables
- Currency formatting: locale-aware with currency symbol (USD $, NGN ₦, EUR €, GBP £)

Spacing & Layout:
- 4px base grid; spacing scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80
- Breakpoints: mobile (390px), tablet (1024px), desktop (1440px), wide (1920px)
- Max content width: 1440px with 24px gutters (marketplace: 1280px for product grids)
- Sidebar widths: collapsed (64px), expanded (256px)
- Product grid: 4-column (desktop), 3-column (tablet), 2-column (mobile)

Elevation & Motion:
- Shadow tokens: sm (product cards), md (dropdowns/popovers), lg (modals/drawers), xl (command palette)
- Border-radius tokens: none, sm (4px), md (8px), lg (12px), full (9999px)
- Motion tokens: instant (0ms), fast (100ms), normal (200ms), slow (300ms)
- Commerce-specific motion: add-to-cart animation (product thumbnail flies to cart icon), order status transition, stock level countdown

Dark Mode:
- Background: #020617 base, #0F172A raised, #1E293B overlay
- Text: #F8FAFC primary, #CBD5E1 secondary, #64748B tertiary
- Product images: maintain original colors on dark surfaces with subtle border
- Charts/graphs: use lighter tints and increased stroke weights

Deliver as a structured token library with light/dark theme variants, auto-layout primitives, and commerce-specific color semantics.
```

#### Prompt F-002: Component Library

```text
Create a production-ready component library for ERP-Commerce with these domain-specific components:

NAVIGATION:
- Admin sidebar with sections: Dashboard, Orders, Catalog, Inventory, POS, Marketplace, Logistics, Distribution, Pricing, Trade Credit, Customers, Analytics, Settings
- Each item: icon + label + optional count badge (pending orders, low-stock alerts)
- Collapsed (64px) and expanded (256px) states
- Active state: left 3px accent bar + background highlight
- Top bar: merchant/org name, global search ("Search products, orders, customers..."), notification bell, cart icon (for buyer portal), user avatar with role badge
- Breadcrumb trail with clickable segments
- Portal switcher: Admin | Merchant | Buyer | Warehouse | POS
- Command palette (Cmd+K) for fuzzy search across products, orders, customers, and actions

DATA DISPLAY:
- Data table with sortable headers, sticky header, row selection, bulk actions, inline edit, column resize, pagination, CSV/Excel export
- Product card: image, name, price (original/discounted), rating stars, seller badge, stock indicator, "Add to Cart" button
- Order card: order ID, date, customer, item count, total, payment status badge, fulfillment status badge, actions
- Inventory card: SKU, product thumbnail, warehouse location, available/reserved/incoming quantities, reorder point indicator
- KPI stat card: metric name, value, delta %, trend sparkline, period selector
- Price display: original price (strikethrough), sale price, savings badge, bulk tier table
- Stock indicator badge: in-stock (green dot), low-stock (amber dot + "Only X left"), out-of-stock (red dot), pre-order (blue dot)
- Order status stepper: Placed > Confirmed > Processing > Shipped > Delivered (horizontal stepper with dates and status per step)

FORMS & INPUT:
- Text input, textarea, number input with increment/decrement, currency input with symbol
- Select, multi-select, combobox with search
- Date picker, date range picker
- File upload with drag-and-drop (product images, bulk CSV import)
- Toggle, checkbox, radio group
- Quantity stepper (min/max constraints, MOQ indicator)
- Barcode/SKU scanner input
- Address input with autocomplete
- Rich text editor (product descriptions)
- Price matrix editor (variant x option grid with per-cell price/stock)

FEEDBACK & OVERLAY:
- Toast notifications: success, error, warning, info with auto-dismiss and action link
- Modal dialog (sm, md, lg, full) for product creation, order review, payment processing
- Drawer (right-slide for order detail, bottom-sheet on mobile for filters)
- Confirmation dialog with destructive variant (type-to-confirm for refunds, cancellations)
- Progress stepper for multi-step flows (checkout, product creation, bulk import)
- Skeleton loaders matching each component shape
- Empty state with illustration and CTA (empty cart, no search results, no products)
- Error state with retry action and support link

COMMERCE-SPECIFIC:
- Cart item row: product thumbnail, name, variant, seller, quantity stepper, unit price, line total, remove, save-for-later
- Checkout step indicator: Address > Shipping > Payment > Review > Confirm
- Fulfillment badge: STANDARD, EXPRESS, PICKUP, SAME-DAY with estimated delivery date
- Seller trust badge: verified mark, fulfillment rate %, return rate %, response time
- MOQ compliance indicator: Met (green check), Partial (amber), Below (red warning)
- Trade credit gauge: circular gauge showing used/available/limit amounts
- POS terminal layout: product grid, cart panel, payment buttons, receipt preview
- Delivery tracking map: map with driver pin, route polyline, ETA countdown
- Bulk pricing tier table: quantity range, unit price, savings vs. base price
- Inventory heatmap: warehouse grid with color-coded stock levels
- Distribution route card: origin > waypoints > destination with distance, time, status

AIDD COMPONENTS:
- AI recommendation card: suggestion text, confidence %, reasoning, data sources, Approve/Edit/Reject buttons
- Explainability panel (collapsible): model inputs and decision factors
- Human-in-the-loop toggle: "Auto-execute" vs. "Require approval"
- Confidence meter (low/medium/high) with color coding
- Audit trail row: actor, action, entity, timestamp, AI-assisted badge

ACCESSIBILITY:
- All interactive elements: visible focus rings (2px offset, primary color)
- Minimum contrast 4.5:1 for text, 3:1 for UI elements
- All icons paired with labels or aria-labels
- Form inputs with associated labels, helper text, and error messages
- Screen reader announcements for dynamic content (cart count, stock alerts)
- Keyboard navigation for all patterns

Deliver as a component library with variant properties, auto-layout, design token references, light and dark mode variants.
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Commerce Command Dashboard (1440px)

```text
Design an omnichannel commerce command dashboard for ERP-Commerce at 1440px desktop width.

TOP BAR:
- Left: ERP-Commerce logo, merchant name "NovaMart Trading Ltd"
- Center: global search bar "Search products, orders, customers, SKUs..."
- Right: notification bell with "14" badge, cart icon (for testing), user avatar "AO" with name "Abiola Ogunsakin" and role "Admin"

LEFT SIDEBAR (expanded, 256px):
- Dashboard (active)
- Orders (badge: "23 new")
- Catalog (badge: "1,847 products")
- Inventory (badge: "12 low stock")
- POS
- Marketplace
- Logistics (badge: "8 in transit")
- Distribution
- Pricing
- Trade Credit
- Customers
- Analytics
- Settings

MAIN CONTENT:

Row 1 — Morning Briefing:
- "Good morning, Abiola. 23 new orders overnight, 12 items need restocking."
- Quick actions: "Process Orders" (primary green), "View Low Stock" (amber), "Create Product" (secondary), "POS Mode" (secondary)

Row 2 — KPI Cards (6 columns):
- Revenue Today: $12,450 (up 15% vs. yesterday, green sparkline)
- Orders Today: 47 (up 8%, trend sparkline)
- GMV (MTD): $387,200 (on track vs. $400K target)
- Avg Order Value: $264.89 (down 3%, amber)
- Fulfillment SLA: 94.2% (above 90% target, green)
- Inventory Turnover: 8.4x (annualized, above 7x target, green)

Row 3 — Two Panels:
- Left (55%): "Revenue Trend" — area chart, last 30 days
  - Revenue line (green filled area)
  - Order count line (cyan overlay)
  - X-axis: dates, Y-axis dual (revenue $, order count #)
  - Hover tooltip with daily breakdown
  - Compare toggle: "vs. Previous 30 Days"

- Right (45%): "Order Status Distribution" — horizontal stacked bar chart
  - Pending: 23 (gray)
  - Confirmed: 15 (blue)
  - Processing: 12 (purple)
  - Shipped: 8 (cyan)
  - Delivered: 187 (green)
  - Returns: 3 (amber)

Row 4 — Three Panels:
- Left (33%): "Top Products" — mini leaderboard
  - Rank, product thumbnail, name, units sold, revenue
  - 5 products: "Organic Shea Butter 500ml" (#1, 234 units, $4,680), "Premium Rice 25kg" (#2, 189 units, $7,560), etc.

- Center (33%): "Inventory Alerts" — alert list
  - "Premium Rice 25kg" — 12 remaining (red, below reorder point)
  - "Coconut Oil 1L" — 28 remaining (amber, approaching reorder)
  - "Vitamin C 1000mg" — 45 remaining (amber)
  - "Baby Formula 400g" — Out of Stock (red, 3 orders pending)
  - "Restock All" button (primary)

- Right (33%): "Cash Flow" — mini card
  - Received Today: $11,200 (green)
  - Pending Settlement: $23,450 (amber)
  - Trade Credit Outstanding: $45,000 (blue)
  - Refunds Processed: $890 (red)
  - "View Finance" link

Row 5 — Two Panels:
- Left (50%): "Recent Orders" — table
  - Columns: Order #, Customer, Items, Total, Payment (badge), Fulfillment (badge), Time
  - 6 sample orders with varied statuses
  - "View All Orders" link

- Right (50%): "Active Deliveries" — mini map + list
  - Map showing 4 delivery driver pins with route lines
  - Below: list of active deliveries with driver name, destination, ETA
  - "View Fleet" link

Row 6 — AI Insights Panel:
- "Demand Prediction: 'Premium Rice 25kg' sales trending up 23% — consider ordering 500 units to avoid stockout by March 5."
- Confidence: 87% | Based on: 90-day sales data, seasonal pattern, current stock
- Buttons: "Create Purchase Order" (primary), "View Analysis" (secondary), "Dismiss"

Include light and dark mode variants.
```

#### Prompt F-004: Order Management Center (1440px)

```text
Design an order management page for ERP-Commerce at 1440px desktop width, powered by order-service.

PAGE HEADER:
- Title: "Orders" with subtitle "1,247 orders this month | $387,200 GMV"
- Actions: "Create Order" (primary green), "Bulk Process" (secondary), "Export" (icon), "Import" (icon)
- View toggle: List (active) | Board (Kanban by status)

FILTER BAR:
- Status: All | Pending (23) | Confirmed (15) | Processing (12) | Shipped (8) | Delivered (187) | Cancelled (4) | Returned (3)
- Payment: All | Paid | Partial | Unpaid | Refunded
- Fulfillment: All | Unfulfilled | Partial | Fulfilled
- Channel: All | Marketplace | POS | B2B Portal | API
- Date range picker
- Customer search
- Amount range: min-max slider
- "Clear Filters"

ORDER TABLE:
- Columns: Checkbox, Order # (e.g., #ORD-2026-0847), Date/Time, Customer (avatar + name), Channel (badge icon), Items (count), Total ($), Payment (badge), Fulfillment (badge), Shipping Method, Actions (view, process, more)
- 10 sample orders:
  1. #ORD-2026-0847 | Feb 23, 09:15 | "TechMart Retail" | B2B Portal | 12 items | $4,280.00 | Paid (green) | Unfulfilled (gray) | Express
  2. #ORD-2026-0846 | Feb 23, 08:42 | "Sarah Okafor" | Marketplace | 3 items | $127.50 | Paid | Processing (purple) | Standard
  3. #ORD-2026-0845 | Feb 23, 08:30 | "Walk-in Customer" | POS | 5 items | $89.00 | Paid | Fulfilled (green) | In-Store Pickup
  4-10: Varied orders including B2B bulk, returns, partial payments

- Hover: row highlight with quick action icons
- Bulk action bar: "Confirm Selected", "Print Labels", "Mark Shipped", "Export", "Cancel"

ORDER DETAIL (click order — split view or full page):

Left Panel (60%):
- Header: order #, date, status stepper (Placed > Confirmed > Processing > Shipped > Delivered), channel badge
- Items table: product thumbnail, name, SKU, variant, quantity, unit price, line total, fulfillment status
- Order actions: "Confirm Order" (primary), "Hold" (amber), "Cancel" (red with confirmation), "Split Order" (secondary)
- Notes section: internal team notes with add note composer

Right Panel (40%):
- Customer card: name, email, phone, total orders, LTV, trade credit status
- Payment card: method, transaction ID, amount paid, status, "Issue Refund" button (with confirmation modal showing refund amount input, reason selector, and impact warning)
- Shipping card: method, carrier, tracking number input, estimated delivery, "Print Label" button, "Mark Shipped" button
- Fulfillment Intelligence card (from logistics-service):
  - Recommended fulfillment source: "Warehouse A — Lagos"
  - Reason: "Closest to delivery address, all items in stock"
  - SLA confidence: 96%
  - Alternative: "Warehouse B — Abuja" (2 items available, 1 requires transfer)
  - Buttons: "Accept Route" (primary), "Override" (secondary)
- Timeline: all order events with timestamps, actors, and status changes

Include light and dark mode variants.
```

#### Prompt F-005: Product Catalog Manager (1440px)

```text
Design a product catalog management page for ERP-Commerce at 1440px desktop width, powered by catalog-service.

PAGE HEADER:
- Title: "Catalog" with subtitle "1,847 products | 24 categories"
- Actions: "Add Product" (primary), "Bulk Import" (secondary), "Export Catalog" (icon), "Category Manager" (icon)
- View toggle: Grid (active) | List | Tree (by category)

FILTER BAR:
- Search: "Search products, SKUs, brands..."
- Category: hierarchical dropdown tree (Groceries > Beverages > Soft Drinks)
- Status: All | Active | Draft | Archived | Out of Stock
- Brand: multi-select
- Price range: slider
- Stock level: All | In Stock | Low Stock | Out of Stock
- "Clear Filters"

PRODUCT GRID VIEW (4 columns):
- Product cards (per product):
  - Product image (200x200px, with hover: show secondary image)
  - Product name (2 lines max, truncated)
  - SKU: "SKU-RIC-025-NG" (monospace, small text)
  - Category badge: "Groceries"
  - Price: "$12.50" (if discounted: original strikethrough + sale price in green)
  - Stock indicator: "In Stock — 234 units" (green) or "Low Stock — 12 units" (amber) or "Out of Stock" (red)
  - Rating: 4.5 stars (128 reviews)
  - Status badge: Active (green dot) | Draft (gray dot) | Archived (blue dot)
  - Quick actions on hover: Edit, Duplicate, Archive, View Analytics
- Pagination: "Showing 1-24 of 1,847 products" with page navigation

PRODUCT LIST VIEW:
- Table: Checkbox, Image (40px thumbnail), Name, SKU, Category, Price, Compare Price, Stock, Status, Sales (last 30d), Actions
- Sortable columns, inline status editing

PRODUCT CREATION/EDIT (multi-step form or tabbed page):
- Tab 1 — General:
  - Product name, brand (dropdown), description (rich text editor)
  - Category selector (tree with search)
  - Tags (multi-select with create)
  - Product type: Physical / Digital / Service
  - Status: Active / Draft

- Tab 2 — Variants:
  - Option groups: Size, Color, Pack Size (add custom)
  - Variant matrix: auto-generated grid with per-variant: SKU, price, stock, weight, barcode
  - Image assignment per variant

- Tab 3 — Media:
  - Drag-and-drop image uploader (up to 10 images)
  - Reorder images, set primary, add alt text
  - Video upload / YouTube link
  - 360-degree view upload

- Tab 4 — Pricing (from pricing-service):
  - Base price, compare-at price
  - Cost price, margin calculator (auto-calculated)
  - Currency selector (multi-currency support)
  - Volume/bulk pricing tiers table:
    - 1-9 units: $12.50
    - 10-49 units: $11.25 (10% off)
    - 50-99 units: $10.00 (20% off)
    - 100+ units: $8.75 (30% off)
  - MOQ (Minimum Order Quantity): number input
  - Customer segment pricing: dropdown segment + price per segment
  - Promotional pricing: start/end date, promo price, promo badge text

- Tab 5 — Inventory (from inventory-service):
  - Per-warehouse stock table: Warehouse Name, Available, Reserved, Incoming, Reorder Point, Lead Time
  - Total across warehouses
  - Reorder point alert threshold
  - "Track Inventory" toggle
  - Batch/lot tracking toggle with batch number, expiry date fields

- Tab 6 — SEO & Marketplace:
  - URL slug, meta title, meta description, preview
  - Marketplace listing status: toggle per marketplace channel
  - Search keywords
  - Related products selector

Include light and dark mode variants.
```

#### Prompt F-006: Inventory & Warehouse Dashboard (1440px)

```text
Design an inventory and warehouse management dashboard for ERP-Commerce at 1440px desktop width, powered by inventory-service.

PAGE HEADER:
- Title: "Inventory" with subtitle "3 warehouses | 1,847 SKUs | $2.4M stock value"
- Actions: "Stock Adjustment" (primary), "Transfer Stock" (secondary), "Cycle Count" (secondary), "Import" (icon)

TAB BAR:
- Overview (active) | Stock List | Transfers | Adjustments | Cycle Counts | Warehouse Map

OVERVIEW TAB:

Row 1 — Warehouse Summary Cards (3 columns, one per warehouse):
- Warehouse A — Lagos: 1,247 SKUs | $1.6M value | 78% utilization | 3 critical alerts
- Warehouse B — Abuja: 842 SKUs | $580K value | 52% utilization | 1 alert
- Warehouse C — Port Harcourt: 564 SKUs | $220K value | 34% utilization | 0 alerts
- Each card: name, location, SKU count, stock value, utilization bar (color-coded), alert count

Row 2 — KPI Row (5 cards):
- Total Stock Value: $2,400,000 (up 5%)
- Stock Turnover: 8.4x (annualized, vs. 7x target, green)
- Fill Rate: 96.2% (vs. 95% target, green)
- Days of Supply: 42 (vs. 45 target, green)
- Shrinkage Rate: 0.8% (vs. 1% target, green)

Row 3 — Two Panels:
- Left (60%): "Stock Level Distribution" — horizontal bar chart
  - Healthy (green): 1,523 SKUs (82%)
  - Low Stock (amber): 187 SKUs (10%)
  - Critical/Out of Stock (red): 48 SKUs (3%)
  - Overstock (blue): 89 SKUs (5%)
  - Click each bar to filter stock list

- Right (40%): "Inventory Movement" — area chart, last 30 days
  - Inbound (green area)
  - Outbound (red area)
  - Net change line (blue)

Row 4 — Critical Alerts Table:
- "Stock Alerts — 12 items need attention"
- Columns: Priority (icon), Product (thumbnail + name), SKU, Warehouse, Available, Reorder Point, Days Until Stockout, Suggested Action, Action Button
- Sample alerts:
  1. CRITICAL | "Premium Rice 25kg" | SKU-RIC-025 | Lagos | 12 | 50 | 3 days | "Order 500 units" | [Reorder] button
  2. CRITICAL | "Baby Formula 400g" | SKU-BAB-400 | Lagos | 0 | 20 | Stockout | "Emergency reorder" | [Reorder] button
  3. WARNING | "Coconut Oil 1L" | SKU-COC-001 | Abuja | 28 | 40 | 8 days | "Order 200 units" | [Reorder]
  4. INFO | "Organic Shea Butter" | SKU-SHE-500 | Lagos | 456 | 100 | Overstock | "Consider promotion" | [View]

Row 5 — Stock Transfer Panel:
- Active transfers: 3 in transit
- Transfer cards: source warehouse > destination warehouse, items count, status (Pending/In Transit/Received), estimated arrival
- "Create Transfer" button

STOCK LIST VIEW (from tab):
- Full inventory table: Checkbox, Image, Product Name, SKU, Category, Warehouse (tabs or filter), Available, Reserved, Incoming, Reorder Point, Last Received, Status, Actions
- Bulk actions: "Adjust Stock", "Transfer", "Export", "Print Labels"
- Inline quantity editing

Include light and dark mode variants.
```

#### Prompt F-007: POS Terminal Interface (1440px)

```text
Design a point-of-sale (POS) terminal interface for ERP-Commerce at 1440px desktop width, powered by pos-service. This is optimized for touchscreen operation in retail environments.

LAYOUT: Two-panel design (60% products / 40% cart)

LEFT PANEL — PRODUCT SELECTION (60%):

Top Section:
- Search bar: "Scan barcode or search product..." with barcode scanner icon
- Category quick-filter chips (horizontal scroll): "All", "Groceries", "Beverages", "Health", "Personal Care", "Household", "Electronics"
- View toggle: Grid | List

Product Grid (4 columns, large touch-friendly cards):
- Product cards (120x120px):
  - Product image
  - Name (1 line, truncated)
  - Price: "$12.50" (large, bold)
  - Stock indicator: green dot (in stock) or red dot (out of stock, dimmed card)
  - Tap to add to cart (single tap = add 1 unit)
  - Long press = enter custom quantity
- Quick-scroll vertical (touch-optimized)
- "Frequently Sold" section at top (4 items based on POS history)

Barcode Scan Result (overlay):
- Product found: show product image, name, price, stock, "Add to Cart" button
- Not found: "Product not found. Create new?" link

RIGHT PANEL — CART & CHECKOUT (40%):

Cart Header:
- "Current Sale" title
- Customer: "Walk-in" (tappable to search/assign customer) or "TechMart Retail — B2B" (with trade credit info)
- "Clear Cart" link (with confirmation)

Cart Items List:
- Each item: product name, quantity stepper (large +/- buttons), unit price, line total, remove (X) button
- If discount applied: original price strikethrough, discounted price, savings badge
- Scrollable if many items

Cart Summary:
- Subtotal: $127.50
- Discount: -$12.75 (if applied)
- Tax (VAT 7.5%): $8.61
- Total: $123.36 (large, bold)
- Items: 5

Action Buttons (large, full-width, touch-optimized):
- "Apply Discount" (amber button) — opens discount modal: percentage, fixed amount, or coupon code
- "Hold Order" (secondary) — save cart for later retrieval
- "Pay" (primary green, extra large) — opens payment modal

Payment Modal (full-screen overlay):
- Amount Due: $123.36 (large display)
- Payment Method Tiles (large, 2x3 grid):
  - Cash (with amount tendered input and change calculator)
  - Card (integrated POS terminal trigger)
  - Mobile Money (MTN/Airtel/Glo selector)
  - Bank Transfer (shows account details + QR code)
  - Wallet (customer wallet balance)
  - Split Payment (divide across methods)
- Cash: preset amount buttons ($50, $100, $150, $200, "Exact"), tendered input, change due display
- "Complete Payment" button

Receipt Panel (after payment):
- Order # and timestamp
- Items summary
- Payment method and amount
- "Print Receipt" button (large)
- "Email Receipt" button
- "SMS Receipt" button
- "New Sale" button (primary, returns to empty cart)

HELD ORDERS DRAWER (accessible via "Held" tab):
- List of held orders with customer, time, items count, total
- Tap to restore to active cart

Include light and dark mode variants. Optimize all touch targets for 48px minimum on this POS interface.
```

#### Prompt F-008: Marketplace & Trade Credit Dashboard (1440px)

```text
Design a marketplace operations and trade credit management page for ERP-Commerce at 1440px desktop width, powered by marketplace-service and trade-credit-service.

PAGE HEADER:
- Title: "Marketplace & Credit"
- Actions: "New Listing" (primary), "Credit Application" (secondary), "Export" (icon)

TAB BAR:
- Marketplace Overview (active) | Seller Management | Trade Credit | Credit Applications

MARKETPLACE OVERVIEW:

Row 1 — KPI Cards (5 columns):
- Total Listings: 1,847 (12 pending review)
- Active Sellers: 124 (8 new this month)
- Marketplace GMV (MTD): $287,500 (up 18%)
- Average Seller Rating: 4.3/5.0
- Dispute Rate: 1.2% (below 2% target, green)

Row 2 — Two Panels:
- Left (50%): "GMV by Seller Tier" — bar chart
  - Platinum sellers: $125,000 (12 sellers)
  - Gold sellers: $87,500 (24 sellers)
  - Silver sellers: $52,000 (38 sellers)
  - Bronze sellers: $23,000 (50 sellers)

- Right (50%): "Category Performance" — treemap chart
  - Groceries: $98,000 (34%)
  - Electronics: $58,000 (20%)
  - Health & Beauty: $43,000 (15%)
  - Fashion: $37,000 (13%)
  - Home & Living: $28,750 (10%)
  - Other: $22,750 (8%)

Row 3 — Seller Management Table:
- Columns: Seller (avatar + name), Tier (badge), Products, Orders (MTD), Revenue (MTD), Rating, Response Time, Fulfillment Rate, Status, Actions
- 6 sample sellers with diverse performance levels
- "View Store", "Performance Report", "Suspend" actions

TRADE CREDIT TAB:

Row 1 — Credit Portfolio KPIs (4 columns):
- Total Credit Extended: $450,000
- Credit Utilized: $312,000 (69%, gauge visual)
- Available Credit: $138,000
- Overdue Amount: $18,500 (amber, 4.1% of utilized)

Row 2 — Credit Accounts Table:
- Columns: Customer (avatar + name), Credit Limit, Utilized, Available, Overdue, Credit Score (gauge badge), Payment History (mini sparkline), Days Past Due, Status (Active/Suspended/Under Review), Actions
- 8 sample accounts:
  1. "TechMart Retail" | $50,000 | $32,000 | $18,000 | $0 | Score: 85 (green) | Good | 0 | Active
  2. "QuickShop Nigeria" | $25,000 | $23,500 | $1,500 | $3,200 | Score: 62 (amber) | Declining | 15 | Under Review
  3-8: Varied accounts

Row 3 — Credit Application Queue:
- Applications pending review: 5
- Table: Applicant, Business Type, Requested Limit, Credit Score, Revenue (12mo), Time in Business, Recommendation (AI), Actions (Approve/Reject/Request Info)
- AI recommendation card per application:
  - "Recommended: Approve with $15,000 limit (requested: $25,000)"
  - Reasoning: "Revenue supports $15K. New customer, limited history. Suggest 90-day review."
  - Confidence: 78%
  - Buttons: "Approve $15K" | "Approve Full" | "Counter-Offer" | "Reject"

Row 4 — Aging Analysis:
- Horizontal stacked bar chart:
  - Current: $245,000 (green)
  - 1-30 days: $48,500 (light amber)
  - 31-60 days: $12,000 (amber)
  - 61-90 days: $4,500 (orange)
  - 90+ days: $2,000 (red)
- Table below with per-customer aging breakdown

Include light and dark mode variants.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Commerce Mobile Dashboard (390px)

```text
Design a mobile commerce dashboard for ERP-Commerce at 390px width (iPhone 14 frame, 390x844px).

STATUS BAR: standard iOS status bar

TOP BAR:
- Left: hamburger menu icon
- Center: "ERP-Commerce" logo
- Right: notification bell with "14" badge, user avatar "AO"

MAIN CONTENT (scrollable):

Greeting Section:
- "Good morning, Abiola"
- "23 new orders | 12 items need restocking"

KPI Row (horizontal scroll, 2 visible + peek):
- Revenue Today: $12,450 (up 15%)
- Orders: 47 (up 8%)
- GMV MTD: $387,200
- Fulfillment SLA: 94.2%
- Inventory Alerts: 12

Quick Actions Grid (2x2):
- Process Orders (green icon, badge "23")
- Scan Product (blue icon)
- POS Mode (purple icon)
- Low Stock (amber icon, badge "12")

Recent Orders Card:
- 3 most recent orders with order #, customer, total, status badge
- Swipe actions: confirm (green), hold (amber)
- "View All Orders" link

Inventory Alerts Card:
- "12 items need attention"
- Top 3 critical items: product name, stock remaining, "Reorder" button
- "View All Alerts" link

Revenue Chart Card:
- Mini area chart: last 7 days revenue trend
- Today's total prominent

AI Insight Card:
- "'Premium Rice 25kg' will stockout in 3 days. Order 500 units?"
- "Create PO" (primary), "Dismiss"

BOTTOM NAVIGATION BAR (5 items):
- Dashboard (active)
- Orders (badge "23")
- Catalog
- POS
- More

Include light and dark mode variants.
```

#### Prompt F-010: Mobile Order Detail (390px)

```text
Design a mobile order detail screen for ERP-Commerce at 390px width (iPhone 14 frame).

TOP BAR:
- Back arrow
- Title: "Order #ORD-2026-0847"
- Share icon, More (three-dot menu)

ORDER STATUS STEPPER (horizontal, full width):
- Placed (green check) > Confirmed (green check) > Processing (current, blue pulse) > Shipped (gray) > Delivered (gray)
- "Confirmed on Feb 23, 2026 at 09:30 AM"

CUSTOMER CARD:
- "TechMart Retail" (B2B badge)
- Contact: David Okafor
- Phone: +234 805 234 5678 (tap to call)
- Email: david@techmart.ng
- Trade Credit: $18,000 available (green gauge)

ORDER ITEMS LIST (scrollable):
- Item 1: Thumbnail | "Premium Rice 25kg" | SKU: RIC-025 | Qty: 20 | $12.50 x 20 = $250.00
- Item 2: Thumbnail | "Coconut Oil 1L" | SKU: COC-001 | Qty: 50 | $8.00 x 50 = $400.00
- Item 3: Thumbnail | "Organic Shea Butter 500ml" | SKU: SHE-500 | Qty: 100 | $20.00 x 100 = $2,000.00
- Subtotal, tax, shipping, total displayed below

ORDER SUMMARY CARD:
- Subtotal: $3,650.00
- Shipping: $180.00
- Tax (7.5%): $287.25
- Discount: -$365.00 (10% B2B)
- Total: $3,752.25
- Payment: Trade Credit (green badge)

SHIPPING CARD:
- Method: Express Delivery
- Carrier: "NovaMart Logistics"
- Tracking: (not yet shipped)
- Estimated Delivery: Feb 26, 2026
- Warehouse: "Warehouse A — Lagos"
- "Generate Shipping Label" button

FULFILLMENT CARD:
- AI Recommendation: "Ship from Lagos warehouse — all items in stock"
- Confidence: 96%
- "Accept" (primary), "Override" (secondary)

ACTION BUTTONS (sticky bottom):
- "Process Order" (primary green, full width)
- Row: "Hold" (amber) | "Cancel" (red)

ACTIVITY TIMELINE (expandable section):
- Feb 23, 09:15 — Order placed by David Okafor (B2B Portal)
- Feb 23, 09:20 — Payment confirmed (Trade Credit)
- Feb 23, 09:30 — Order auto-confirmed

Include light and dark mode variants.
```

#### Prompt F-011: Mobile POS (390px)

```text
Design a mobile POS screen for ERP-Commerce at 390px width (iPhone 14 frame), optimized for smartphone-based retail.

SCREEN 1 — POS HOME:

TOP BAR:
- Back arrow
- Title: "POS Terminal"
- Settings gear icon

Search Bar (full width):
- "Scan barcode or search..." with camera/barcode icon button

Category Tabs (horizontal scroll):
- All | Groceries | Beverages | Health | Personal Care | Household

Product Grid (2 columns, large cards for touch):
- Each card (170x170px):
  - Product image
  - Name (1 line)
  - Price (large, bold)
  - Stock badge if low
  - Tap to add to cart (haptic feedback)
- Vertical scroll through products

Cart Summary Bar (sticky bottom, expandable):
- Collapsed: "5 items | $123.36 | Pay" (green button)
- Tap to expand: full cart view

SCREEN 2 — EXPANDED CART (bottom sheet, 80% height):
- Drag handle at top
- Cart items list:
  - Product name + quantity stepper + line total + remove (X)
  - Each item on one line, compact
- Customer field: "Walk-in" (tappable to assign customer)
- Subtotal, Tax (7.5%), Total
- "Apply Discount" button (amber)
- "Pay $123.36" button (large, green, full width)

SCREEN 3 — PAYMENT:
- Amount: $123.36 (large display)
- Payment method cards (2x2 grid, large touch targets):
  - Cash
  - Card
  - Mobile Money
  - Bank Transfer
- Cash selected: amount tendered input with preset buttons ($100, $150, $200), change display
- "Complete Payment" button

SCREEN 4 — RECEIPT:
- Green checkmark animation
- "Sale Complete!"
- Order #POS-2026-0847
- Items summary (collapsible)
- Total: $123.36
- Payment: Cash — Tendered: $150.00 — Change: $26.64
- Buttons: "Print" | "Email" | "SMS" | "WhatsApp"
- "New Sale" (primary, full width)

Include light and dark mode variants. All touch targets minimum 48px.
```

#### Prompt F-012: Mobile Inventory Check (390px)

```text
Design a mobile inventory check screen for ERP-Commerce at 390px width (iPhone 14 frame).

SCREEN 1 — INVENTORY SEARCH:

TOP BAR:
- Back arrow
- Title: "Inventory"
- Barcode scan icon

Search Bar (full width):
- "Search product or scan barcode..."

Quick Filters (horizontal scroll chips):
- All | Low Stock (badge "12") | Out of Stock (badge "4") | Overstock

Inventory List:
- Cards (full width, compact):
  - Product thumbnail (48px) | Product name | SKU
  - Stock bar: Available / Reserved / Incoming (stacked horizontal mini-bar)
  - Available: "234 units" (green text) or "12 units" (amber) or "0 units" (red)
  - Warehouse: "Lagos" (badge)
  - Tap to view detail

SCREEN 2 — PRODUCT STOCK DETAIL (from card tap):

TOP BAR:
- Back arrow
- Title: product name (truncated)
- Edit icon

Product Header:
- Large image (120px)
- Name: "Premium Rice 25kg"
- SKU: "SKU-RIC-025-NG"
- Category: "Groceries"
- Price: $12.50

Stock by Warehouse (expandable cards):
- Warehouse A — Lagos:
  - Available: 12 (RED — below reorder point of 50)
  - Reserved: 8 (for pending orders)
  - Incoming: 200 (PO #PO-2026-0045, ETA: Feb 28)
  - Reorder Point: 50
  - "Adjust Stock" button

- Warehouse B — Abuja:
  - Available: 45 (amber)
  - Reserved: 3
  - Incoming: 0
  - Reorder Point: 30
  - "Transfer from Lagos" button (disabled — Lagos low)

- Warehouse C — Port Harcourt:
  - Available: 8 (red)
  - Reserved: 0
  - Incoming: 0
  - Reorder Point: 20

Total Across Warehouses: 65 available

Quick Actions (bottom buttons):
- "Create Purchase Order" (primary green)
- "Transfer Stock" (secondary)
- "Adjust Stock" (secondary)
- "View Movement History" (text link)

SCREEN 3 — QUICK STOCK ADJUSTMENT:
- Product: "Premium Rice 25kg" (shown)
- Warehouse: dropdown (default: current)
- Adjustment Type: "Add" / "Remove" (toggle)
- Quantity: number input with +/- stepper
- Reason: dropdown (Received, Damaged, Count Correction, Returned, Other)
- Notes: text area
- "Submit Adjustment" button (primary)
- "Requires Approval" note if quantity > threshold

Include light and dark mode variants.
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Commerce Dashboard Tablet (1024px)

```text
Design the commerce command dashboard adapted for 1024px tablet width.

LAYOUT ADAPTATIONS FROM 1440px DESKTOP:
- Sidebar: collapsed to 64px (icon-only with tooltips), expandable as overlay on tap
- Top bar: search becomes icon trigger opening full-width overlay
- Page padding: 16px

Row 1 — Morning Briefing:
- Full-width card, quick action buttons in a single row (4 buttons, icon + label)

Row 2 — KPI Cards:
- Change from 6 columns to 3x2 grid (2 rows of 3 cards)

Row 3 — Charts:
- Stacked vertically: revenue trend full width, then order status full width below

Row 4 — Three Panels:
- Change to 2-column (top products + inventory alerts side by side), cash flow below as full width

Row 5 — Tables:
- Stacked vertically: recent orders full width, then active deliveries full width
- Tables hide less critical columns (Channel, Shipping Method), available via scroll

Row 6 — AI Insights:
- Full-width card, same content

INTERACTIONS:
- All touch targets 44px minimum
- Tables support horizontal swipe for hidden columns
- Charts are tap-interactive
- Pull-to-refresh
- KPI cards tap to show detailed breakdown

Include both light and dark mode variants.
```

#### Prompt F-014: POS Terminal Tablet (1024px)

```text
Design the POS terminal interface adapted for 1024px tablet width, optimized for countertop tablet use in retail.

LAYOUT:
- Two-panel split: Left (55%) product grid, Right (45%) cart — similar to desktop but adjusted proportions
- No sidebar navigation (POS is a focused mode)
- Minimal top bar: "POS Terminal" title, customer selector, "Exit POS" link

LEFT PANEL — Products:
- Search bar with barcode scanner icon
- Category chips (horizontal scroll)
- Product grid: 3 columns (vs. 4 on desktop)
- Cards slightly larger than desktop for better touch targeting (140x140px)
- Frequently sold items pinned at top

RIGHT PANEL — Cart:
- Customer assignment field
- Cart items with larger touch targets on quantity steppers (44px buttons)
- Discount application area
- Payment method selector (grid of large buttons)
- Total display (extra large font for visibility from distance)
- "Pay" button: full width, 64px height, green

PAYMENT FLOW:
- Full-screen overlay (same as desktop but touch-optimized)
- Payment method grid: 2x3, each tile 48px minimum
- Cash: large numpad for tendered amount entry
- Card: "Waiting for card..." status with animation
- Success: large checkmark, receipt options

RECEIPT:
- Large print area matching thermal printer width
- Print, email, SMS, WhatsApp buttons (large, touch-friendly)

Include both light and dark mode variants. All interactive elements minimum 48px for POS context.
```

---

## 4. Make Automation Prompts

### Prompt M-001: Order Processing & Fulfillment Automation

```text
Create a Make (Integromat) scenario for ERP-Commerce order processing and intelligent fulfillment routing:

Trigger: Webhook — receives POST from order-service when a new order is placed
Payload: { orderId, customerId, customerType, items: [{ productId, sku, quantity, warehousePreference }], totalAmount, paymentMethod, shippingMethod, deliveryAddress, channel, tenantId }

Step 1: HTTP Module — POST /v1/order/{orderId}/validate via order-service
  - Validate order: stock availability, pricing, customer status
  - If validation fails: notify customer and operations team

Step 2: Router — Branch by customerType
  - Branch A: B2B order (trade credit, MOQ validation)
  - Branch B: B2C order (standard consumer)
  - Branch C: POS order (immediate fulfillment)

Branch A — B2B Order:
  Step 3A: HTTP Module — GET /v1/trade-credit/{customerId} via trade-credit-service
    - Verify trade credit limit and available balance
    - If insufficient: hold order, notify customer and credit team
  Step 4A: HTTP Module — POST /v1/pricing/validate-moq via pricing-service
    - Validate MOQ compliance for each item
    - If non-compliant: notify customer with required quantities

Branch B — B2C Order:
  Step 3B: HTTP Module — POST /v1/order/{orderId}/confirm via order-service
    - Auto-confirm if payment received

Branch C — POS Order:
  Step 3C: Auto-confirm, skip to fulfillment

Step 5 (all branches): HTTP Module — POST /v1/logistics/route via logistics-service
  - Request optimal fulfillment routing:
    - Input: items with quantities, delivery address, shipping method
    - Output: recommended warehouse, estimated delivery, shipping cost, confidence score
  - If multi-warehouse split needed: create split order instructions

Step 6: HTTP Module — POST /v1/inventory/reserve via inventory-service
  - Reserve stock for order items at assigned warehouse(s)
  - If reservation fails (stock changed): alert operations, suggest alternatives

Step 7: HTTP Module — POST /v1/distribution/shipment via distribution-service
  - Create shipment record with warehouse, carrier, tracking details

Step 8: Router — Notification by channel
  - Email: order confirmation with items, total, tracking link
  - SMS: "Order #{orderId} confirmed! Estimated delivery: {date}"
  - WhatsApp: rich message with order summary
  - Push notification: "Your order is confirmed"

Step 9: Google Sheets — Log order in "Order Tracking" sheet
  - Row: Order ID, Date, Customer, Channel, Total, Payment, Fulfillment Route, Warehouse, ETA

Error Handler: Log to error tracking, notify operations team, attempt auto-retry for transient failures
```

### Prompt M-002: Inventory Restock & Purchase Order Automation

```text
Create a Make (Integromat) scenario for ERP-Commerce automated inventory restocking:

Trigger: Scheduled — runs every 4 hours (6 times daily)

Step 1: HTTP Module — GET /v1/inventory/below-reorder-point via inventory-service
  - Fetch all products below reorder point across all warehouses
  - Include: productId, sku, productName, warehouseId, available, reorderPoint, leadTime, avgDailySales

Step 2: Iterator — For each low-stock product

Step 3: Data Transform — Calculate recommended order quantity
  - Formula: (avgDailySales * leadTime * 1.5) - available + safetyStock
  - Minimum: MOQ from pricing-service
  - Factor in seasonal adjustment if applicable

Step 4: Router — Branch by urgency
  - Branch A: Critical (available < 3 days of supply) — immediate action
  - Branch B: Warning (available < 7 days of supply) — scheduled action
  - Branch C: Preventive (approaching reorder point) — queue for review

Branch A — Critical:
  Step 5A: HTTP Module — POST /v1/inventory/purchase-order via inventory-service
    - Auto-create purchase order with recommended quantity
    - Set priority: "Urgent"
  Step 6A: Slack/Teams — Send urgent alert to procurement channel
    - "CRITICAL STOCK ALERT: {productName} ({sku}) at {warehouseName} — {available} units remaining ({daysOfSupply} days). PO #{poNumber} created for {orderQty} units."
  Step 7A: Email — Notify warehouse manager and procurement team
  Step 8A: SMS — Alert to operations lead

Branch B — Warning:
  Step 5B: HTTP Module — POST /v1/inventory/purchase-order
    - Create purchase order with standard priority
  Step 6B: Email — Notify procurement team with order details

Branch C — Preventive:
  Step 5C: HTTP Module — POST /v1/inventory/restock-queue
    - Add to review queue for weekly procurement meeting

Step 9: Aggregator — Compile daily restock summary
  - Total products below reorder: X
  - POs auto-created: Y
  - Estimated restock cost: $Z

Step 10: Google Sheets — Update "Inventory Restock Log"
  - Row per product: Date, Product, SKU, Warehouse, Available, Reorder Point, Order Qty, PO Number, Priority, ETA

Error Handler: Continue on individual product failure, compile error summary
```

### Prompt M-003: Trade Credit Monitoring & Collection

```text
Create a Make (Integromat) scenario for ERP-Commerce trade credit monitoring and automated collection:

Trigger: Scheduled — runs daily at 7:00 AM

Step 1: HTTP Module — GET /v1/trade-credit/accounts?status=active via trade-credit-service
  - Fetch all active trade credit accounts with balances

Step 2: Iterator — For each account

Step 3: HTTP Module — GET /v1/trade-credit/{accountId}/aging via trade-credit-service
  - Fetch aging breakdown: current, 1-30, 31-60, 61-90, 90+

Step 4: Router — Branch by aging status
  - Branch A: Current (no overdue) — skip
  - Branch B: 1-15 days overdue — gentle reminder
  - Branch C: 16-30 days overdue — firm reminder
  - Branch D: 31-60 days overdue — warning with credit freeze threat
  - Branch E: 61+ days overdue — credit freeze + escalation

Branch B — Gentle Reminder (1-15 days):
  Step 5B: Email — Send payment reminder
    - "Dear {customerName}, your payment of {amount} was due on {dueDate}. Please arrange payment at your earliest convenience. Current balance: {totalOutstanding}."
  Step 6B: WhatsApp — Send reminder via communication channel

Branch C — Firm Reminder (16-30 days):
  Step 5C: Email — Send formal notice
    - "PAYMENT OVERDUE: {customerName}, your account has {amount} overdue by {days} days. Please pay immediately to maintain your credit standing."
  Step 6C: SMS — Send to primary contact
  Step 7C: HTTP Module — PATCH /v1/trade-credit/{accountId} via trade-credit-service
    - Reduce credit limit by 25%
    - Log adjustment with reason

Branch D — Warning (31-60 days):
  Step 5D: Email — Send formal warning
    - "FINAL WARNING: Your credit account will be frozen in 7 days if payment of {amount} is not received."
  Step 6D: HTTP Module — PATCH /v1/trade-credit/{accountId}
    - Reduce credit limit by 50%
    - Flag account for review
  Step 7D: Email — Notify finance team for personal follow-up

Branch E — Credit Freeze (61+ days):
  Step 5E: HTTP Module — PATCH /v1/trade-credit/{accountId}
    - Freeze account (status: suspended)
    - Block new B2B orders
  Step 6E: Email — Formal account suspension notice
  Step 7E: Email — Notify finance director and legal team
  Step 8E: HTTP Module — POST /v1/order/block-customer/{customerId}
    - Block new orders via order-service until payment received

Step 9: Aggregator — Daily credit health summary
  - Total outstanding, total overdue, aging distribution
  - Accounts actioned today

Step 10: Email — Daily summary to finance team
Step 11: Google Sheets — Update "Credit Monitoring Log"

Error Handler: Log failures, notify finance admin
```

### Prompt M-004: POS End-of-Day Reconciliation

```text
Create a Make (Integromat) scenario for ERP-Commerce POS end-of-day reconciliation:

Trigger: Scheduled — runs daily at 11:00 PM (after store close)

Step 1: HTTP Module — GET /v1/pos/sessions?date=today&status=closed via pos-service
  - Fetch all closed POS sessions for the day

Step 2: Iterator — For each POS session

Step 3: HTTP Module — GET /v1/pos/session/{sessionId}/summary via pos-service
  - Fetch session summary:
    - Total sales count and amount
    - Payment breakdown: cash, card, mobile money, bank transfer
    - Discounts applied
    - Refunds/voids
    - Expected cash in drawer

Step 4: HTTP Module — GET /v1/pos/session/{sessionId}/transactions via pos-service
  - Fetch all individual transactions for reconciliation

Step 5: Data Transform — Calculate reconciliation
  - Expected drawer cash = opening float + cash sales - cash refunds
  - Compare with actual drawer count (from session close)
  - Calculate variance: over/short
  - Flag variance > $10 for review

Step 6: HTTP Module — POST /v1/inventory/adjust-batch via inventory-service
  - Batch-adjust inventory for all POS transactions (decrement sold items)
  - This ensures POS sales are reflected in real-time inventory

Step 7: Router — Branch by variance
  - Branch A: Within tolerance ($0-$10) — auto-reconcile
  - Branch B: Minor variance ($10-$50) — flag for manager review
  - Branch C: Major variance (>$50) — escalate

Branch A:
  Step 8A: HTTP Module — POST /v1/pos/session/{sessionId}/reconcile
    - Mark session as reconciled

Branch B:
  Step 8B: Email — Notify store manager with variance details
  Step 9B: HTTP Module — POST /v1/pos/session/{sessionId}/flag
    - Flag for review with variance amount

Branch C:
  Step 8C: Email — Notify store manager, operations director, and finance
  Step 9C: HTTP Module — POST /v1/pos/session/{sessionId}/escalate
    - Escalate with mandatory investigation requirement

Step 10: Aggregator — Daily POS summary across all terminals
  - Total revenue, transaction count, payment method breakdown
  - Total variance (over/short)
  - Terminals reconciled vs. flagged

Step 11: Email — Send daily POS report to management
Step 12: Google Sheets — Archive daily POS data

Error Handler: Log reconciliation failures, alert operations
```

### Prompt M-005: Marketplace Seller Performance Monitoring

```text
Create a Make (Integromat) scenario for ERP-Commerce marketplace seller performance monitoring:

Trigger: Scheduled — runs daily at 6:00 AM

Step 1: HTTP Module — GET /v1/marketplace/sellers?status=active via marketplace-service
  - Fetch all active sellers

Step 2: Iterator — For each seller

Step 3: HTTP Module — GET /v1/marketplace/seller/{sellerId}/metrics?period=last_30_days via marketplace-service
  - Fetch performance metrics:
    - Order fulfillment rate
    - On-time shipping rate
    - Customer rating (average)
    - Response time to inquiries
    - Return/dispute rate
    - Revenue contribution

Step 4: Data Transform — Calculate seller health score
  - Weighted score: fulfillment rate (30%) + on-time rate (25%) + rating (20%) + response time (15%) + dispute rate (10%)
  - Score out of 100

Step 5: Router — Branch by seller health
  - Branch A: Excellent (90-100) — recognition
  - Branch B: Good (70-89) — standard operation
  - Branch C: Warning (50-69) — improvement needed
  - Branch D: Critical (<50) — suspension risk

Branch A — Excellent:
  Step 6A: HTTP Module — PATCH /v1/marketplace/seller/{sellerId}
    - Promote to higher tier if eligible
  Step 7A: Monthly: Email — Send recognition badge and benefits notification

Branch C — Warning:
  Step 6C: Email — Send performance improvement notice
    - "Your seller performance score has dropped to {score}/100. Areas needing improvement: {metrics}. You have 30 days to improve or risk account suspension."
  Step 7C: HTTP Module — PATCH /v1/marketplace/seller/{sellerId}
    - Add "Performance Warning" flag
    - Reduce marketplace visibility (lower search ranking)

Branch D — Critical:
  Step 6D: Email — Send final warning
    - "Your seller account is at risk of suspension due to performance below acceptable levels."
  Step 7D: HTTP Module — PATCH /v1/marketplace/seller/{sellerId}
    - Suspend new listing creation
    - Reduce visibility to minimum
    - Flag for manual review
  Step 8D: Email — Notify marketplace operations team

Step 9: Aggregator — Daily seller performance summary
  - Sellers by tier: Excellent/Good/Warning/Critical counts
  - Top 5 and bottom 5 performers
  - New sellers onboarded

Step 10: Email — Daily summary to marketplace operations
Step 11: Google Sheets — Update "Seller Performance Tracker"

Error Handler: Continue on individual seller failure, compile error summary
```

### Prompt M-006: Logistics & Delivery SLA Monitoring

```text
Create a Make (Integromat) scenario for ERP-Commerce logistics and delivery SLA monitoring:

Trigger: Scheduled — runs every 30 minutes

Step 1: HTTP Module — GET /v1/logistics/shipments?status=in_transit via logistics-service
  - Fetch all active shipments with tracking data

Step 2: Iterator — For each shipment

Step 3: HTTP Module — GET /v1/logistics/shipment/{shipmentId}/tracking via logistics-service
  - Fetch current tracking status, location, estimated delivery time

Step 4: Data Transform — Calculate SLA status
  - Compare estimated delivery vs. promised delivery date
  - Calculate: on-time probability, hours remaining, distance remaining

Step 5: Router — Branch by SLA status
  - Branch A: On track (delivery within SLA) — update tracking
  - Branch B: At risk (delivery may miss SLA by < 4 hours) — alert
  - Branch C: SLA breach (delivery will miss promised date) — escalate
  - Branch D: Delivered — confirm and close

Branch A — On Track:
  Step 6A: HTTP Module — PATCH /v1/order/{orderId}/tracking via order-service
    - Update order with latest tracking information
  (no alerts needed)

Branch B — At Risk:
  Step 6B: HTTP Module — GET /v1/logistics/alternatives?shipmentId={shipmentId}
    - Check if alternative delivery route exists
  Step 7B: Slack/Teams — Alert logistics channel
    - "SLA AT RISK: Shipment #{shipmentId} for Order #{orderId} — ETA: {eta}, Promised: {promised}. {hoursLate} hours late."
  Step 8B: Email — Notify logistics coordinator
  Step 9B: HTTP Module — PATCH /v1/order/{orderId}/sla-risk
    - Flag order as SLA-at-risk

Branch C — SLA Breach:
  Step 6C: HTTP Module — PATCH /v1/order/{orderId}/sla-breach via order-service
    - Mark order as SLA breached
  Step 7C: Email — Notify customer proactively
    - "We apologize — your order #{orderId} is experiencing a delivery delay. New estimated delivery: {newEta}. We are working to expedite."
  Step 8C: Email — Escalate to logistics manager and operations director
  Step 9C: HTTP Module — POST /v1/logistics/escalation via logistics-service
    - Create escalation case with shipment details and suggested remediation
  Step 10C: Router — If high-value order or VIP customer
    - Offer compensation: discount code for next order

Branch D — Delivered:
  Step 6D: HTTP Module — PATCH /v1/order/{orderId}/status via order-service
    - Update order status to "Delivered"
  Step 7D: Email — Send delivery confirmation and review request to customer
    - "Your order has been delivered! We'd love your feedback."
  Step 8D: HTTP Module — POST /v1/inventory/sold-confirm via inventory-service
    - Confirm inventory deduction for delivered items

Step 9: Aggregator — Hourly logistics summary
  - Shipments in transit, on-track %, at-risk %, breached %
  - Average delivery time vs. SLA

Step 10: Google Sheets — Update "Logistics SLA Tracker"

Error Handler: Log tracking failures, alert logistics team for manual check
```

---

## 5. Prompt Usage Guidelines

### 5.1 Execution Order
1. Run **F-001** (Design Tokens) and **F-002** (Component Library) first to establish the foundation
2. Run desktop page prompts (**F-003** through **F-008**) for primary layouts
3. Run mobile prompts (**F-009** through **F-012**) for mobile-specific designs
4. Run tablet prompts (**F-013**, **F-014**) for responsive adaptations
5. Make automation prompts (**M-001** through **M-006**) can be executed independently

### 5.2 Customization
- Replace "NovaMart Trading Ltd" with your merchant/organization name
- Adjust currency symbols and formats for regional deployment (USD, NGN, EUR, GBP, KES)
- Modify product categories and sample data to match your industry vertical
- Adjust warehouse names and locations to match your distribution network
- Configure payment methods for your supported gateways (Paystack, Flutterwave, Stripe, etc.)

### 5.3 Portal Variants
The design system supports multiple portal personas. When generating pages, specify the portal context:
- **Admin Portal**: full access, all modules visible
- **Merchant Portal**: catalog, orders, inventory, analytics, payments
- **Buyer Portal**: marketplace, cart, orders, account, trade credit
- **Warehouse Portal**: inventory, shipments, cycle counts, receiving
- **POS Terminal**: product grid, cart, payment, receipt (focused mode)

### 5.4 Design Quality Checks
- Every page must have: loading (skeleton), empty, populated, and error states
- Every interactive element must show: default, hover, focus, active, disabled states
- All forms must show: valid, invalid, and submitting states
- POS interface must be tested for touch-optimized use (48px minimum targets)
- Dark mode must be tested for contrast compliance on every page

---

## 6. Output Packaging Convention

| Artifact | Format | Naming Convention |
|----------|--------|-------------------|
| Design tokens | Figma Variables / JSON | `erp-commerce-tokens-v{version}` |
| Component library | Figma Library | `ERP-Commerce Components v{version}` |
| Desktop pages | Figma Pages | `Desktop — {PageName}` |
| Mobile pages | Figma Pages | `Mobile — {PageName}` |
| Tablet pages | Figma Pages | `Tablet — {PageName}` |
| POS layouts | Figma Pages | `POS — {DeviceType}` |
| Prototype flows | Figma Prototype | `Flow — {JourneyName}` |
| Make blueprints | Make JSON export | `make-commerce-{scenario-name}-v{version}.json` |
| Design specs | Figma Dev Mode | Linked to each page |

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Lighthouse Performance | >= 95 | Per-route audit |
| Lighthouse Accessibility | >= 95 | Per-route audit |
| First Contentful Paint | < 1.2s | Lab + RUM |
| Largest Contentful Paint | < 2.5s | Lab + RUM |
| Cumulative Layout Shift | < 0.1 | Lab + RUM |
| Time to Interactive | < 3.0s | Lab measurement |
| Initial route bundle (gzip) | < 220KB | Build analysis |
| Product image load (p95) | < 1.5s | CDN + RUM |
| POS scan-to-cart | < 0.5s | End-to-end measurement |
| Checkout completion | < 5 steps | UX measurement |
| API response (p95) | < 500ms | Backend monitoring |
| Search results display | < 800ms | End-to-end measurement |

---

## 8. AIDD Handoff Gate Template

| Gate | Check | Status |
|------|-------|--------|
| G-01 | All breakpoints designed (1440, 1024, 390) | Pending |
| G-02 | Light and dark mode variants complete | Pending |
| G-03 | All states covered (loading, empty, populated, error) | Pending |
| G-04 | WCAG 2.1 AA contrast ratios verified | Pending |
| G-05 | Touch targets >= 44px (48px for POS) | Pending |
| G-06 | Skeleton loaders designed for all data-dependent areas | Pending |
| G-07 | AI recommendations show Approve/Edit/Reject actions | Pending |
| G-08 | Destructive actions have two-step confirmation | Pending |
| G-09 | Audit trail badges on financial transactions | Pending |
| G-10 | Tenant/merchant isolation indicator visible | Pending |
| G-11 | Multi-currency display tested | Pending |
| G-12 | POS interface tested for touch-first operation | Pending |
| G-13 | Analytics and conversion tracking events documented | Pending |
| G-14 | Error recovery paths with retry and support link | Pending |
| G-15 | Design tokens reference Figma Variables (not hard-coded) | Pending |
| G-16 | Component auto-layout verified at all breakpoints | Pending |
| G-17 | Make automation scenarios tested with sample payloads | Pending |

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial Figma and Make design prompts for ERP-Commerce covering 10 services: catalog, distribution, inventory, logistics, marketplace, order, portal, pos, pricing, trade-credit. Includes design system, 6 desktop pages, 4 mobile screens, 2 tablet adaptations, and 6 Make automations. Design intelligence drawn from Amazon, Shopify Plus, Alibaba, Jumia, Coupang, Square, Stripe, and Flexport. |
