# ERP-Commerce -- Acceptance Criteria

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

## 1. Catalog Service Acceptance Criteria

### AC-CAT-01: Product Creation
- **GIVEN** a manufacturer admin is authenticated
- **WHEN** they create a product with SKU, name, category, and base price
- **THEN** the product is created with status "draft"
- **AND** event `erp.commerce.catalog.created` is emitted
- **AND** the product is visible only to the owning tenant

### AC-CAT-02: Multi-Level Pricing
- **GIVEN** a product exists with base price
- **WHEN** trade-level pricing is configured (distributor: -15%, wholesaler: -10%, retailer: -5%)
- **THEN** each trade level sees their correct price in the catalog
- **AND** pricing waterfall is enforced (no retailer sees distributor price)

### AC-CAT-03: Brand Policy MOQ
- **GIVEN** a brand policy sets category MOQ (distributor: 1000, wholesaler: 100, retailer: 10)
- **WHEN** a retailer attempts to order 5 units
- **THEN** the system rejects or routes to MOQ grouping
- **AND** the policy evaluation is logged for audit

### AC-CAT-04: Product Search
- **GIVEN** 10,000+ products exist in a tenant catalog
- **WHEN** a user searches by keyword with category filter
- **THEN** results are returned within 150ms (p95)
- **AND** results include faceted counts for brands, price ranges, and categories

---

## 2. Order Service Acceptance Criteria

### AC-ORD-01: Order Creation with Credit Check
- **GIVEN** a retailer with credit account (limit: 1,000,000 NGN, balance: 300,000 NGN)
- **WHEN** they place an order for 500,000 NGN
- **THEN** credit check passes (available: 700,000 NGN)
- **AND** credit balance is updated to 800,000 NGN
- **AND** order status becomes "approved"

### AC-ORD-02: Order Rejection on Credit Exceeded
- **GIVEN** a retailer with credit account (limit: 1,000,000 NGN, balance: 900,000 NGN)
- **WHEN** they place an order for 200,000 NGN
- **THEN** credit check fails (available: 100,000 NGN < 200,000 NGN)
- **AND** order status becomes "rejected"
- **AND** retailer receives notification with reason

### AC-ORD-03: Multi-Source Order Splitting
- **GIVEN** an order with items A (available at Warehouse 1) and B (available at Warehouse 2)
- **WHEN** the order is processed
- **THEN** two sub-orders are created (one per warehouse)
- **AND** each sub-order has its own tracking
- **AND** the parent order shows consolidated status

### AC-ORD-04: EDI Order Processing
- **GIVEN** a valid EDI X12 850 document is received via AS2
- **WHEN** the EDI parser processes the document
- **THEN** an internal order is created from the parsed data
- **AND** an EDI 855 acknowledgment is generated
- **AND** the EDI transaction is archived

### AC-ORD-05: Under-MOQ Grouping
- **GIVEN** retailer A orders 5 units and retailer B orders 8 units (MOQ: 10, same region)
- **WHEN** the grouping engine runs
- **THEN** a pooled order of 13 units is created
- **AND** individual delivery assignments are maintained
- **AND** each retailer sees only their portion

---

## 3. Pricing Service Acceptance Criteria

### AC-PRC-01: Price Calculation Performance
- **GIVEN** the pricing engine has 50+ active rules
- **WHEN** a price calculation request is made
- **THEN** the response is returned within 50ms (p95)

### AC-PRC-02: Volume Discount Application
- **GIVEN** volume discount tiers (100+: -3%, 500+: -5%, 1000+: -8%)
- **WHEN** a customer orders 750 units at base price 100 NGN
- **THEN** the 500+ tier is applied (-5%)
- **AND** unit price is 95 NGN, total is 71,250 NGN

### AC-PRC-03: Promotional Pricing
- **GIVEN** a promotion "20% off Category X" valid Feb 1-28, 2026
- **WHEN** a customer orders Category X product on Feb 15
- **THEN** the 20% promotional discount is applied
- **AND** when the same product is ordered on Mar 1, full price applies

---

## 4. Inventory Service Acceptance Criteria

### AC-INV-01: Stock Reservation
- **GIVEN** product X has 100 units available at Warehouse A
- **WHEN** Order 1 reserves 30 units and Order 2 reserves 50 units
- **THEN** available quantity shows 20 units
- **AND** reserved quantity shows 80 units

### AC-INV-02: Low Stock Alert
- **GIVEN** product Y has reorder point of 50 units
- **WHEN** available stock drops to 48 units
- **THEN** a low-stock alert is generated
- **AND** event `erp.commerce.inventory.low_stock` is emitted

### AC-INV-03: Lot Tracking with Expiry
- **GIVEN** product Z has lots L1 (expires Mar 1) and L2 (expires Apr 1)
- **WHEN** inventory is allocated for an order
- **THEN** lot L1 (nearest expiry) is allocated first (FEFO)
- **AND** lot information is recorded on the order line item

---

## 5. POS Service Acceptance Criteria

### AC-POS-01: Offline Transaction Processing
- **GIVEN** a POS terminal is offline (no internet connectivity)
- **WHEN** a cashier processes a sale
- **THEN** the transaction completes successfully using local cache
- **AND** the transaction is queued for sync
- **AND** inventory is decremented locally

### AC-POS-02: Offline Sync on Reconnection
- **GIVEN** a POS terminal has 50 queued offline transactions
- **WHEN** internet connectivity is restored
- **THEN** all 50 transactions are synced within 60 seconds
- **AND** server-side inventory is updated
- **AND** no duplicate transactions are created

### AC-POS-03: Shift Reconciliation
- **GIVEN** a shift with opening cash 50,000 NGN
- **AND** cash sales totaling 120,000 NGN and cash refunds of 5,000 NGN
- **WHEN** the shift is closed with closing cash counted at 163,000 NGN
- **THEN** expected cash is 165,000 NGN
- **AND** variance of -2,000 NGN is recorded
- **AND** shift summary report is generated

---

## 6. Trade Credit Service Acceptance Criteria

### AC-TCR-01: AI Credit Score Computation
- **GIVEN** a customer with 12 months transaction history
- **WHEN** a credit score is requested
- **THEN** the AI model returns a score (0-1000) within 30 seconds
- **AND** the score includes factor breakdown
- **AND** the decision is logged with model version

### AC-TCR-02: Collections Escalation
- **GIVEN** an invoice is 31 days overdue
- **WHEN** the collections engine runs
- **THEN** the account is placed on hold
- **AND** new orders from this account are blocked
- **AND** the account holder is notified

---

## 7. Logistics Service Acceptance Criteria

### AC-LOG-01: Route Optimization
- **GIVEN** 50 delivery stops with time windows and 5 vehicles
- **WHEN** route optimization is triggered
- **THEN** routes are generated within 60 seconds
- **AND** all time windows are respected
- **AND** vehicle capacity is not exceeded

### AC-LOG-02: Proof of Delivery
- **GIVEN** a delivery is assigned to a driver
- **WHEN** the driver captures signature POD at the delivery location
- **THEN** the POD is saved with GPS coordinates and timestamp
- **AND** delivery status is updated to "delivered"
- **AND** all parties receive notification

---

## 8. Marketplace Service Acceptance Criteria

### AC-MKT-01: Vendor Onboarding
- **GIVEN** a vendor submits registration with valid KYC/KYB documents
- **WHEN** verification completes successfully
- **THEN** vendor account is activated
- **AND** commission rate is assigned
- **AND** vendor can publish products within 24 hours

### AC-MKT-02: Commission Calculation
- **GIVEN** a vendor with 5% commission rate
- **WHEN** a marketplace order of 100,000 NGN is completed
- **THEN** commission of 5,000 NGN is recorded
- **AND** vendor net payout is 95,000 NGN
- **AND** commission record is visible in vendor dashboard
