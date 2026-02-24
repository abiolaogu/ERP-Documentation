# ERP-Commerce -- Implementation Prompts (AIDD Code Generation)

## Document Control

| Field    | Value                                   |
|----------|-----------------------------------------|
| Module   | ERP-Commerce                            |
| Version  | 2.0                                     |
| Date     | 2026-02-23                              |

---

## 1. AIDD Implementation Guide

This document provides structured prompts for AI-Driven Development (AIDD) code generation within ERP-Commerce. All prompts respect the guardrails defined in `erp/aidd.guardrails.yaml`.

---

## 2. Catalog Service Implementation Prompts

### Prompt 2.1: Product Domain Model

> Implement the complete product domain model for the ERP-Commerce catalog-service in Go 1.22. The service lives at `services/catalog-service/`. Create the following package structure:
>
> `internal/domain/models.go`: Define structs for Product (id UUID, tenantId UUID, sku string, name string, description string, categoryId UUID, brandId UUID, status ProductStatus enum, attributes map[string]string, seoMetadata JSONB, createdAt/updatedAt timestamps), ProductVariant (id, productId, variantSku, name, options map, barcode, weight, weightUnit, dimensions JSONB, status), Category (id, tenantId, parentId nullable, name, slug, path materialized, level int, sortOrder), Brand (id, tenantId, name, logo), BrandPolicy (id, tenantId, brandId, categoryId, distributorMOQ, wholesalerMOQ, retailerMOQ, effectiveFrom/To dates), MediaAsset (id, productId, url, mimeType, sortOrder). Include validation methods on each struct. ProductStatus should be draft/active/inactive/archived.
>
> `internal/ports/inbound.go`: Define CatalogService interface with methods: CreateProduct, GetProduct, UpdateProduct, DeleteProduct, ListProducts (with pagination), SearchProducts (with facets), BulkImportProducts, CreateCategory, GetCategoryTree, CreateBrand, SetBrandPolicy, ValidateMOQ.
>
> `internal/ports/outbound.go`: Define ProductRepository, CategoryRepository, BrandRepository interfaces and EventPublisher interface.

### Prompt 2.2: Category Tree with Materialized Path

> Implement the PostgreSQL category tree using materialized path pattern for the catalog-service. In `internal/adapters/repository/postgres.go`, implement the CategoryRepository interface:
>
> CreateCategory: INSERT with auto-generated path (parent_path + '/' + slug). GetCategoryTree: SELECT WHERE path LIKE $parentPath || '%' ORDER BY path. MoveCategory: UPDATE path for subtree using string replacement. DeleteCategory: Cascade delete children via path prefix match. GetAncestors: Parse path string to extract ancestor slugs and batch query. GetBreadcrumb: Return ordered ancestor list for navigation display.
>
> Include migration SQL in `migrations/002_categories.sql` with the categories table, GIN index on path, and unique constraint on (tenant_id, slug).

### Prompt 2.3: Elasticsearch Product Search

> Implement Elasticsearch integration for the catalog-service product search. Create `internal/adapters/search/elasticsearch.go`:
>
> IndexProduct: Index product document with fields (name, description, sku, category_name, brand_name, attributes, price, status). Mapping: name as text with edge_ngram analyzer, sku as keyword, attributes as nested object, price as float.
>
> SearchProducts: Accept SearchRequest (query string, filters: categoryId, brandId, priceRange, attributes, status; sort: relevance/price/name; pagination). Return SearchResult with items, total count, facets (brand counts, category counts, price ranges). Query uses bool/must for text match, filter for structured fields, aggs for facets.
>
> BulkIndex: Batch index up to 1000 products using Elasticsearch bulk API. SyncFromDatabase: Full reindex from PostgreSQL for disaster recovery.

---

## 3. Order Service Implementation Prompts

### Prompt 3.1: Order State Machine

> Implement the order state machine for the order-service in Go. Create `internal/domain/order_state.go`:
>
> Define OrderStatus enum with states: Draft, PendingValidation, CreditCheck, PolicyCheck, MOQGrouping, Approved, Splitting, Allocated, Picking, Packed, Shipped, InTransit, Delivered, Invoiced, Settled, Rejected, Cancelled.
>
> Define transition rules as a map[OrderStatus][]OrderStatus. Valid transitions: Draft->PendingValidation, PendingValidation->CreditCheck/Rejected, CreditCheck->PolicyCheck/Rejected, PolicyCheck->Approved/MOQGrouping, MOQGrouping->Approved, Approved->Splitting/Allocated/Cancelled, Splitting->Allocated, Allocated->Picking, Picking->Packed, Packed->Shipped, Shipped->InTransit, InTransit->Delivered, Delivered->Invoiced, Invoiced->Settled.
>
> Implement CanTransition(from, to) bool and Transition(order, newStatus, reason, actorId) error that validates transition, updates status, records history entry, and returns domain event.

### Prompt 3.2: Order Splitting Algorithm

> Implement the multi-source order splitting algorithm for the order-service. Create `internal/application/splitting.go`:
>
> SplitOrder(order Order, inventoryClient InventoryService) ([]SubOrder, error):
> 1. For each line item, query inventory-service for available stock across all locations
> 2. Apply splitting strategy (configurable: minimize_shipments, minimize_cost, minimize_time)
> 3. For minimize_shipments: group items by the location that can fulfill the most items
> 4. For minimize_cost: select cheapest fulfillment location per item
> 5. For minimize_time: select nearest location to delivery address
> 6. Create SubOrder per fulfillment location with assigned line items
> 7. Each SubOrder gets its own order_number (parent_order_number + suffix)
> 8. Emit erp.commerce.order.split event with parent and child order IDs

---

## 4. Pricing Service Implementation Prompts

### Prompt 4.1: Price Waterfall Engine

> Implement the pricing waterfall engine for the pricing-service. Create `internal/application/waterfall.go`:
>
> CalculatePrice(req PriceRequest) PriceResult:
> 1. Load base price from product catalog (sync call or cache)
> 2. Apply trade level adjustment (look up rule by trade_level enum)
> 3. Apply volume discount (find matching tier by quantity)
> 4. Apply active promotions (query promotions by product + date range)
> 5. Apply contract pricing override (if customer has active contract)
> 6. Apply geographic adjustment (if location provided)
> 7. Calculate tax (VAT rate by country/region)
> 8. Apply currency rounding rules
> 9. Return PriceResult with unitPrice, totalPrice, discountAmount, taxAmount, appliedRules[]
>
> Use Redis caching with 5-minute TTL for computed prices. Cache key: `price:{tenantId}:{productId}:{variantId}:{tradeLevel}:{customerId}:{quantity}`. Include benchmarks targeting < 50ms for single price calculation and < 500ms for 100-item batch.

---

## 5. Trade Credit Service Implementation Prompts

### Prompt 5.1: Credit Scoring ML Model

> Implement the credit scoring ML model as a Python FastAPI service at `ai-services/credit-scoring/`:
>
> Feature engineering (`features.py`): Compute features from raw data:
> - payment_on_time_rate: (on-time payments / total payments)
> - avg_days_to_pay: mean of (payment_date - due_date) across all invoices
> - order_frequency: orders per month over last 6 months
> - avg_order_value: mean order value over last 6 months
> - total_gmv: cumulative gross merchandise value
> - business_tenure_months: months since first order
> - external_credit_score: normalized bureau score (0-1)
> - network_score: weighted score from supplier references
>
> Model (`credit_model.py`): Gradient boosting classifier (XGBoost) trained on historical data. Output: credit_score (0-1000), risk_category (low/medium/high/critical), recommended_limit, recommended_terms.
>
> API (`routes.py`): POST /score accepts customer_id, returns credit score with full breakdown. Include model versioning in response. Store all scoring decisions for audit.

---

## 6. POS Service Implementation Prompts

### Prompt 6.1: Offline Sync Engine

> Implement the POS offline sync engine in Go. Create `internal/adapters/sync/engine.go`:
>
> Outbound sync:
> 1. Read queued transactions from local SQLite (ordered by created_at)
> 2. Batch up to 50 transactions per sync request
> 3. POST /v1/pos/sync with batch payload
> 4. On success: mark transactions as synced, delete from queue
> 5. On conflict: apply conflict resolution (last-write-wins using vector clocks)
> 6. On failure: retry with exponential backoff (1s, 2s, 4s, 8s, 16s max)
>
> Inbound sync:
> 1. GET /v1/pos/sync/updates?since={last_sync_timestamp}
> 2. Receive updated products, prices, customers, promotions
> 3. Upsert into local SQLite cache
> 4. Update last_sync_timestamp
>
> Conflict resolution: Each transaction has a vector clock (terminal_id, sequence_number). On conflict, the transaction with the higher sequence number wins. Losers are logged in a conflict report for manual review.

---

## 7. Logistics Service Implementation Prompts

### Prompt 7.1: VRP Route Optimizer

> Implement the VRP route optimizer as a Python FastAPI service at `ai-services/route-optimizer/`:
>
> Using Google OR-Tools RoutingModel:
> 1. Accept delivery jobs with: pickup_location, delivery_location, weight, time_window_start, time_window_end, priority
> 2. Accept vehicles with: capacity_kg, start_location (depot), max_hours, driver_id
> 3. Build distance matrix using Google Maps Distance Matrix API (cache results for 24h)
> 4. Add constraints: capacity per vehicle, time windows per stop, maximum driving hours per driver, break requirements
> 5. Configure search parameters: first_solution_strategy=PATH_CHEAPEST_ARC, local_search_metaheuristic=GUIDED_LOCAL_SEARCH, time_limit=60s
> 6. Return optimized routes: [{driver_id, vehicle_id, stops: [{delivery_id, arrival_time, departure_time, order}], total_distance_km, total_duration_min}]
> 7. Include quality score comparing optimized vs. naive sequential route

---

## 8. Rust Component Implementation Prompts

### Prompt 8.1: EDI X12 Parser

> Implement the EDI X12 parser as a Rust library at `rust-components/edi-parser/`:
>
> Using the `nom` parser combinator library:
> 1. Parse ISA/IEA envelope (interchange header/trailer)
> 2. Parse GS/GE envelope (functional group header/trailer)
> 3. Parse ST/SE envelope (transaction set header/trailer)
> 4. Implement transaction set parsers for: 850 (Purchase Order), 855 (PO Acknowledgment), 856 (Ship Notice), 810 (Invoice), 832 (Price Catalog), 997 (Functional Acknowledgment)
> 5. Output parsed data as serde-serializable Rust structs
> 6. Expose C FFI functions for Go interop: `parse_x12(data: *const c_char, len: usize) -> *mut ParseResult`
> 7. Include comprehensive error reporting with segment/element position
> 8. Benchmark: parse 1000-line X12 850 in < 1ms

### Prompt 8.2: High-Performance Price Calculator

> Implement the bulk price calculator in Rust at `rust-components/price-calculator/`:
>
> 1. Define PriceRequest/PriceResult structs (matching Go DTOs)
> 2. Implement waterfall pipeline: base -> trade_level -> volume -> promo -> contract -> geo -> tax
> 3. Load pricing rules into memory from JSON configuration
> 4. Process batch of 10,000 price calculations in parallel using rayon
> 5. Expose C FFI: `calculate_prices(requests: *const PriceRequest, count: usize, results: *mut PriceResult) -> i32`
> 6. Target: < 1ms per price calculation, < 100ms for 10,000 batch
> 7. Include criterion.rs benchmarks

---

## 9. Portal Frontend Implementation Prompts

### Prompt 9.1: Distributor Dashboard

> Implement the Distributor Dashboard page in Next.js/React/TypeScript at `apps/web/portals/distributor/app/dashboard/page.tsx`:
>
> Layout: 12-column grid with responsive breakpoints.
> Top KPI row (full width): 6 KPICard components showing: Orders Today (count + trend), Revenue Today (amount + trend), Active Deliveries (count), Inventory Health (% score), Credit Utilization (% of portfolio), Fleet Status (active/total drivers).
>
> Main area (8 columns): Order Pipeline as kanban board using `@dnd-kit/core` with columns: New, Confirmed, Picking, Packed, Shipped, Delivered. Each card shows order number, customer, amount, time badge.
>
> Sidebar (4 columns): Live Activity Feed (WebSocket-driven list of recent events), Inventory Alerts (low-stock items with reorder buttons), Route Status (mini map with active driver dots).
>
> Use TanStack Query for data fetching, Zustand for local UI state, ECharts for charts, Mapbox GL for mini map. Include loading skeletons and error boundaries.

---

## 10. Guardrails Compliance

All AIDD code generation must comply with `erp/aidd.guardrails.yaml`:

| Guardrail                         | Implementation Requirement                       |
|-----------------------------------|--------------------------------------------------|
| Tenant isolation                  | Every query must include tenant_id filter         |
| No cross-tenant access            | RLS policies on all tables                        |
| Decision logging                  | All credit/pricing/policy decisions logged         |
| Human-in-the-loop for high-risk   | Bulk deletes, credit overrides require approval   |
| 24-hour rollback window           | All mutations must support undo within 24 hours   |
| No irreversible delete            | Soft delete with retention period                 |
