# Low-Level Design -- FusionCommerce (ERP-eCommerce)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This Low-Level Design document provides implementation-level detail for each FusionCommerce microservice, including class hierarchies, method signatures, data structures, algorithm descriptions, and API endpoint specifications.

## 2. Catalog Service (catalog-service)

### 2.1 Class Diagram

```mermaid
classDiagram
    class CatalogService {
        -repository: CatalogRepository
        -eventBus: EventBus
        -imageStorage: ImageStorage
        +createProduct(input: CreateProductInput): Promise~Product~
        +updateProduct(id: string, input: UpdateProductInput): Promise~Product~
        +getProduct(id: string): Promise~Product~
        +listProducts(filters: ProductFilters): Promise~PaginatedResult~Product~~
        +deleteProduct(id: string): Promise~void~
        +addVariant(productId: string, input: CreateVariantInput): Promise~Variant~
        +uploadImage(productId: string, file: Buffer): Promise~ProductImage~
    }

    class CatalogRepository {
        <<interface>>
        +save(product: Product): Promise~void~
        +findById(id: string): Promise~Product | null~
        +findAll(filters: ProductFilters): Promise~PaginatedResult~Product~~
        +update(id: string, data: Partial~Product~): Promise~Product~
        +delete(id: string): Promise~void~
    }

    class PostgresCatalogRepository {
        -knex: Knex
        +save(product)
        +findById(id)
        +findAll(filters)
        +update(id, data)
        +delete(id)
    }

    class InMemoryCatalogRepository {
        -store: Map~string, Product~
        +save(product)
        +findById(id)
        +findAll(filters)
        +update(id, data)
        +delete(id)
    }

    class Product {
        +id: string
        +tenantId: string
        +sku: string
        +name: string
        +slug: string
        +description: string
        +price: number
        +currency: string
        +status: ProductStatus
        +categoryId: string
        +variants: Variant[]
        +images: ProductImage[]
        +tags: string[]
        +metadata: Record~string, unknown~
        +createdAt: Date
        +updatedAt: Date
    }

    class Variant {
        +id: string
        +productId: string
        +sku: string
        +name: string
        +price: number | null
        +options: Record~string, string~
        +inventoryQuantity: number
    }

    CatalogService --> CatalogRepository
    CatalogRepository <|.. PostgresCatalogRepository
    CatalogRepository <|.. InMemoryCatalogRepository
    CatalogService ..> Product
    Product *-- Variant
```

### 2.2 API Endpoints

| Method | Path | Request | Response | Description |
|--------|------|---------|----------|-------------|
| POST | /v1/products | CreateProductInput JSON | 201 Product | Create product, emit product.created |
| GET | /v1/products | ?page, limit, category, brand, status | 200 PaginatedResult | List products with filtering |
| GET | /v1/products/:id | - | 200 Product | Get product with variants and images |
| PUT | /v1/products/:id | UpdateProductInput JSON | 200 Product | Update product, emit product.updated |
| DELETE | /v1/products/:id | - | 204 No Content | Soft-delete product |
| POST | /v1/products/:id/variants | CreateVariantInput JSON | 201 Variant | Add product variant |
| POST | /v1/products/:id/images | multipart/form-data | 201 ProductImage | Upload image to MinIO |
| GET | /v1/categories | ?parent_id | 200 Category[] | List categories |
| POST | /v1/categories | CreateCategoryInput JSON | 201 Category | Create category |

## 3. Checkout Service (checkout-service)

### 3.1 Checkout State Machine

```mermaid
stateDiagram-v2
    [*] --> CartActive: Create Cart
    CartActive --> ShippingAddress: Add Items + Proceed
    ShippingAddress --> ShippingMethod: Set Address
    ShippingMethod --> PaymentMethod: Select Shipping
    PaymentMethod --> OrderReview: Set Payment
    OrderReview --> Processing: Confirm Order
    Processing --> Completed: Payment Succeeds
    Processing --> PaymentFailed: Payment Fails
    PaymentFailed --> PaymentMethod: Retry
    CartActive --> Abandoned: 30 min inactivity
    Abandoned --> CartActive: Customer Returns
    Completed --> [*]
```

### 3.2 Checkout Class Structure

```mermaid
classDiagram
    class CheckoutService {
        -cartRepository: CartRepository
        -couponEngine: CouponEngine
        -taxCalculator: TaxCalculator
        -shippingCalculator: ShippingCalculator
        -eventBus: EventBus
        +createCart(tenantId: string, sessionId: string): Promise~Cart~
        +addItem(cartId: string, item: CartItemInput): Promise~Cart~
        +removeItem(cartId: string, itemId: string): Promise~Cart~
        +updateQuantity(cartId: string, itemId: string, qty: number): Promise~Cart~
        +applyCoupon(cartId: string, code: string): Promise~Cart~
        +setShippingAddress(cartId: string, address: Address): Promise~Cart~
        +selectShippingMethod(cartId: string, methodId: string): Promise~Cart~
        +completeCheckout(cartId: string, paymentToken: string): Promise~Order~
        +detectAbandonment(): Promise~void~
    }

    class CouponEngine {
        -couponRepository: CouponRepository
        +validate(code: string, cart: Cart): Promise~CouponValidation~
        +apply(coupon: Coupon, cart: Cart): Promise~DiscountResult~
        +checkStacking(existingDiscounts: Discount[], newCoupon: Coupon): boolean
    }

    class TaxCalculator {
        +calculate(items: CartItem[], address: Address): Promise~TaxResult~
        -getRate(jurisdiction: string): Promise~number~
    }

    class ShippingCalculator {
        +getRates(items: CartItem[], address: Address): Promise~ShippingRate[]~
        -calculateWeight(items: CartItem[]): number
        -getCarrierRates(weight: number, destination: Address): Promise~Rate[]~
    }

    CheckoutService --> CouponEngine
    CheckoutService --> TaxCalculator
    CheckoutService --> ShippingCalculator
```

### 3.3 Coupon Engine Algorithm

```
FUNCTION validateCoupon(code, cart):
    coupon = couponRepository.findByCode(code)
    IF coupon IS NULL: RETURN { valid: false, reason: "INVALID_CODE" }
    IF coupon.isActive IS false: RETURN { valid: false, reason: "INACTIVE" }
    IF now() < coupon.startsAt: RETURN { valid: false, reason: "NOT_YET_ACTIVE" }
    IF coupon.expiresAt AND now() > coupon.expiresAt: RETURN { valid: false, reason: "EXPIRED" }
    IF coupon.maxUses AND coupon.usedCount >= coupon.maxUses: RETURN { valid: false, reason: "MAX_USES" }
    IF cart.subtotal < coupon.minPurchase: RETURN { valid: false, reason: "MIN_PURCHASE" }
    IF NOT checkStacking(cart.existingDiscounts, coupon): RETURN { valid: false, reason: "STACKING" }
    RETURN { valid: true, discount: calculateDiscount(coupon, cart) }
```

## 4. Search Service (search-service)

### 4.1 Search Pipeline

```mermaid
flowchart TD
    Q[User Query] --> NLP[NLP Processing]
    NLP --> TYPO[Typo Correction]
    TYPO --> SYN[Synonym Expansion]
    SYN --> TOK[Tokenization]
    TOK --> IDX{Query Type?}
    IDX -->|Text| FTS[Full-Text Search<br/>OpenSearch]
    IDX -->|Visual| VIS[Visual Embedding<br/>CLIP Model]
    IDX -->|Voice| ASR[Speech-to-Text<br/>Whisper]
    ASR --> FTS
    VIS --> VEC[Vector Similarity<br/>OpenSearch kNN]
    FTS --> MERGE[Result Merger]
    VEC --> MERGE
    MERGE --> MERCH[Merchandising Rules]
    MERCH --> BOOST[Boost/Bury/Pin]
    BOOST --> FACET[Facet Aggregation]
    FACET --> RANK[Relevance Ranking]
    RANK --> RES[Search Results]
```

### 4.2 Merchandising Rules Engine

```mermaid
classDiagram
    class MerchandisingEngine {
        -rules: MerchandisingRule[]
        +applyRules(results: SearchResult[], context: SearchContext): SearchResult[]
        +addRule(rule: MerchandisingRule): void
        +removeRule(ruleId: string): void
    }

    class MerchandisingRule {
        <<interface>>
        +id: string
        +name: string
        +priority: number
        +condition: RuleCondition
        +action: RuleAction
        +apply(results: SearchResult[]): SearchResult[]
    }

    class BoostRule {
        +boostFactor: number
        +apply(results): SearchResult[]
    }

    class BuryRule {
        +buryFactor: number
        +apply(results): SearchResult[]
    }

    class PinRule {
        +position: number
        +productId: string
        +apply(results): SearchResult[]
    }

    class BannerRule {
        +bannerHtml: string
        +position: number
        +apply(results): SearchResult[]
    }

    MerchandisingEngine *-- MerchandisingRule
    MerchandisingRule <|.. BoostRule
    MerchandisingRule <|.. BuryRule
    MerchandisingRule <|.. PinRule
    MerchandisingRule <|.. BannerRule
```

## 5. Loyalty Service (loyalty-service)

### 5.1 Points Calculation Engine

```mermaid
flowchart TD
    ORD[order.created event] --> CALC[Calculate Base Points]
    CALC --> TIER{Customer Tier?}
    TIER -->|Bronze 1x| B[Base Points x 1.0]
    TIER -->|Silver 1.25x| S[Base Points x 1.25]
    TIER -->|Gold 1.5x| G[Base Points x 1.5]
    TIER -->|Platinum 2x| P[Base Points x 2.0]
    B & S & G & P --> BONUS{Active Bonus?}
    BONUS -->|Double Points Day| DBL[Points x 2]
    BONUS -->|Category Bonus| CAT[Points x 1.5 for category]
    BONUS -->|No Bonus| STD[Standard Points]
    DBL & CAT & STD --> CREDIT[Credit to Wallet]
    CREDIT --> NOTIFY[Notify Customer]
    CREDIT --> EVAL[Evaluate Tier Upgrade]
```

### 5.2 Tier Evaluation Algorithm

```
FUNCTION evaluateTier(customerId):
    account = loyaltyRepository.findByCustomer(customerId)
    spend12m = orderRepository.sumSpendLast12Months(customerId)

    newTier = CASE
        WHEN spend12m >= 5000: 'platinum'
        WHEN spend12m >= 2000: 'gold'
        WHEN spend12m >= 500:  'silver'
        ELSE: 'bronze'

    IF newTier > account.currentTier:
        account.currentTier = newTier
        account.tierUpgradedAt = now()
        eventBus.publish('loyalty.tier_upgraded', { customerId, newTier })
    ELSE IF newTier < account.currentTier:
        IF monthsSince(account.tierUpgradedAt) > 3:  // Grace period
            account.currentTier = newTier
            eventBus.publish('loyalty.tier_downgraded', { customerId, newTier })

    loyaltyRepository.save(account)
```

## 6. Subscription Commerce Service (subscription-commerce-service)

### 6.1 Subscription Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> Active: Customer Subscribes
    Active --> Paused: Customer Pauses
    Active --> Skipped: Customer Skips Next
    Active --> SwapPending: Customer Swaps Product
    Skipped --> Active: Next Cycle
    SwapPending --> Active: Swap Confirmed
    Paused --> Active: Customer Resumes
    Active --> PaymentFailed: Renewal Fails
    PaymentFailed --> Active: Retry Succeeds
    PaymentFailed --> Suspended: 3 Retries Failed
    Suspended --> Active: Payment Updated
    Suspended --> Cancelled: 30 Days No Action
    Active --> Cancelled: Customer Cancels
    Paused --> Cancelled: Customer Cancels
    Cancelled --> [*]
```

### 6.2 Renewal Processing

```mermaid
sequenceDiagram
    participant CRON as Scheduler
    participant SUB as Subscription Service
    participant PAY as Payments Service
    participant ORD as Orders Service
    participant INV as Inventory Service
    participant KFK as Kafka

    CRON->>SUB: Check renewals due today
    SUB->>SUB: Get active subscriptions due
    loop For each subscription
        SUB->>PAY: Charge saved payment method
        alt Payment succeeds
            PAY-->>SUB: Payment confirmed
            SUB->>ORD: Create renewal order
            ORD->>KFK: order.created
            KFK->>INV: Reserve stock
            SUB->>SUB: Set next renewal date
            SUB->>KFK: subscription.renewed
        else Payment fails
            PAY-->>SUB: Payment failed
            SUB->>SUB: Increment retry count
            alt Retries < 3
                SUB->>SUB: Schedule retry (2 days)
                SUB->>KFK: subscription.payment_retry
            else Retries = 3
                SUB->>SUB: Suspend subscription
                SUB->>KFK: subscription.suspended
            end
        end
    end
```

## 7. Fulfillment Service (fulfillment-service)

### 7.1 Multi-Warehouse Routing Algorithm

```mermaid
flowchart TD
    ORD[New Order] --> ITEMS[Extract Line Items]
    ITEMS --> WH[Query Warehouse Inventory]
    WH --> ALG{Routing Algorithm}
    ALG -->|Single Warehouse| SW[All items from nearest warehouse with full stock]
    ALG -->|Split Shipment| SS[Minimize total shipping distance across warehouses]
    ALG -->|Dropship| DS[Route to supplier warehouse]
    SW --> PICK[Create Pick List]
    SS --> PICK
    DS --> THREPL[Send to 3PL/Dropship Vendor]
    PICK --> PACK[Pack Items]
    PACK --> LABEL[Generate Shipping Label]
    LABEL --> SHIP[Hand to Carrier]
    SHIP --> TRACK[Tracking Number Created]
```

### 7.2 Warehouse Selection Algorithm

```
FUNCTION selectWarehouse(items, shippingAddress):
    warehouses = warehouseRepository.findAll()
    candidates = []

    FOR EACH warehouse IN warehouses:
        availability = checkAllItemsAvailable(warehouse.id, items)
        IF availability.allAvailable:
            distance = calculateDistance(warehouse.location, shippingAddress)
            shippingCost = estimateShippingCost(distance, items.totalWeight)
            candidates.push({ warehouse, distance, shippingCost, type: 'single' })

    IF candidates.length > 0:
        RETURN candidates.sortBy(c => c.shippingCost).first()

    // No single warehouse can fulfill -- try split shipment
    splitPlan = optimizeSplitShipment(warehouses, items, shippingAddress)
    IF splitPlan.feasible:
        RETURN splitPlan

    // Check dropship vendors
    dropshipPlan = checkDropshipAvailability(items)
    IF dropshipPlan.feasible:
        RETURN dropshipPlan

    THROW InsufficientInventoryError
```

## 8. Social Commerce Service (social-commerce-service)

### 8.1 Platform Integration Architecture

```mermaid
classDiagram
    class SocialCommerceService {
        -adapters: Map~string, PlatformAdapter~
        +syncCatalog(platform: string, products: Product[]): Promise~SyncResult~
        +handleOrder(platform: string, order: SocialOrder): Promise~Order~
        +startLivestream(config: LivestreamConfig): Promise~LivestreamSession~
        +createReferralLink(customerId: string, productId: string): Promise~string~
    }

    class PlatformAdapter {
        <<interface>>
        +syncProducts(products: Product[]): Promise~SyncResult~
        +processOrder(order: PlatformOrder): Promise~Order~
        +getAnalytics(): Promise~PlatformAnalytics~
    }

    class InstagramAdapter {
        -apiClient: InstagramGraphAPI
        +syncProducts(products)
        +processOrder(order)
        +getAnalytics()
    }

    class TikTokAdapter {
        -apiClient: TikTokShopAPI
        +syncProducts(products)
        +processOrder(order)
        +getAnalytics()
    }

    class FacebookAdapter {
        -apiClient: FacebookCommerceAPI
        +syncProducts(products)
        +processOrder(order)
        +getAnalytics()
    }

    SocialCommerceService *-- PlatformAdapter
    PlatformAdapter <|.. InstagramAdapter
    PlatformAdapter <|.. TikTokAdapter
    PlatformAdapter <|.. FacebookAdapter
```

## 9. Analytics Service (analytics-service)

### 9.1 Druid Data Model

| Datasource | Dimensions | Metrics | Granularity |
|-----------|------------|---------|-------------|
| sales_events | tenant_id, product_id, category, channel, country | revenue, quantity, order_count | MINUTE |
| funnel_events | tenant_id, step, device, source | event_count, unique_users | MINUTE |
| search_events | tenant_id, query, has_results, clicked | search_count, click_count, ctr | MINUTE |
| cart_events | tenant_id, action, product_id | cart_adds, cart_removes, abandonment | MINUTE |
| loyalty_events | tenant_id, tier, action_type | points_earned, points_redeemed, members | HOUR |

### 9.2 Funnel Analysis Query Pattern

```mermaid
flowchart LR
    V[Visit<br/>100%] --> S[Search<br/>45%]
    S --> P[Product View<br/>30%]
    P --> A[Add to Cart<br/>15%]
    A --> C[Checkout Start<br/>8%]
    C --> O[Order Complete<br/>3.5%]
```

```sql
SELECT
  step,
  COUNT(DISTINCT session_id) as unique_users,
  COUNT(DISTINCT session_id) * 100.0 /
    FIRST_VALUE(COUNT(DISTINCT session_id)) OVER (ORDER BY step_order) as conversion_pct
FROM funnel_events
WHERE tenant_id = ? AND __time >= CURRENT_TIMESTAMP - INTERVAL '7' DAY
GROUP BY step, step_order
ORDER BY step_order
```
