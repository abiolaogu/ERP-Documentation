# Workflows and User Journeys -- FusionCommerce (ERP-eCommerce)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

## 1. Introduction

This document catalogs the end-to-end workflows and user journeys in FusionCommerce, covering both consumer-facing flows and merchant back-office operations. Each workflow includes sequence diagrams, state transitions, and n8n automation integration points.

## 2. Consumer Workflows

### 2.1 Product Discovery and Search

```mermaid
flowchart TD
    START[Consumer Visits Store] --> ENTRY{Entry Point?}
    ENTRY -->|Direct URL| PDP[Product Detail Page]
    ENTRY -->|Category Browse| CAT[Category Listing]
    ENTRY -->|Search Bar| SEARCH[Search Service]
    ENTRY -->|Social Link| SOCIAL[Social Landing Page]

    SEARCH --> NLQ{Query Type?}
    NLQ -->|Text| FTS[Full-Text Search]
    NLQ -->|Image Upload| VIS[Visual Search]
    NLQ -->|Voice| VOICE[Voice-to-Text + Search]

    FTS --> RESULTS[Search Results]
    VIS --> RESULTS
    VOICE --> RESULTS
    CAT --> RESULTS

    RESULTS --> FACET[Apply Facet Filters]
    FACET --> SORT[Sort Results]
    SORT --> PDP
    PDP --> CART{Add to Cart?}
    CART -->|Yes| ADD[Add to Cart]
    CART -->|No| WISH[Add to Wishlist]
    CART -->|Compare| COMP[Product Comparison]
```

### 2.2 Shopping Cart Workflow

```mermaid
sequenceDiagram
    participant C as Consumer
    participant SF as Storefront
    participant CHK as Checkout Service
    participant INV as Inventory Service
    participant SDB as ScyllaDB (Cart State)

    C->>SF: Add Product to Cart
    SF->>CHK: POST /cart/items
    CHK->>INV: Check stock availability
    INV-->>CHK: Stock available (no reservation yet)
    CHK->>SDB: Save cart state
    CHK-->>SF: Cart updated (item added)
    SF-->>C: Show updated cart

    C->>SF: Update quantity
    SF->>CHK: PUT /cart/items/:id
    CHK->>INV: Verify new quantity available
    INV-->>CHK: Available
    CHK->>SDB: Update cart state
    CHK-->>SF: Cart updated
    SF-->>C: Show updated total

    C->>SF: Apply coupon
    SF->>CHK: POST /cart/coupon
    CHK->>CHK: CouponEngine.validate()
    CHK->>SDB: Save discount
    CHK-->>SF: Discount applied
    SF-->>C: Show discounted total
```

### 2.3 Multi-Step Checkout Flow

```mermaid
flowchart TD
    CART[Shopping Cart] --> GUEST{Has Account?}
    GUEST -->|Yes| LOGIN[Login via ERP-IAM]
    GUEST -->|No| GUESTCHK[Guest Checkout<br/>Email Only]
    LOGIN --> ADDR[Shipping Address]
    GUESTCHK --> ADDR

    ADDR --> SAVE{Save Address?}
    SAVE -->|Yes| ADDRBOOK[Add to Address Book]
    SAVE -->|No| CONT[Continue]
    ADDRBOOK --> SHIP
    CONT --> SHIP

    SHIP[Select Shipping Method] --> RATES[Display Carrier Rates]
    RATES --> PAYSEL{Payment Method?}
    PAYSEL -->|Credit Card| CC[Stripe Card Element]
    PAYSEL -->|Apple Pay| AP[Apple Pay Sheet]
    PAYSEL -->|Google Pay| GP[Google Pay Button]
    PAYSEL -->|PayPal| PP[PayPal Redirect]
    PAYSEL -->|BNPL| BNPL[Klarna/Afterpay]

    CC & AP & GP & PP & BNPL --> REVIEW[Order Review]
    REVIEW --> LOYALTY{Use Loyalty Points?}
    LOYALTY -->|Yes| REDEEM[Deduct Points from Wallet]
    LOYALTY -->|No| CONFIRM[Confirm Order]
    REDEEM --> CONFIRM
    CONFIRM --> PROCESS[Processing]
    PROCESS --> SUCCESS[Order Confirmation]
    SUCCESS --> EMAIL[Confirmation Email]
```

### 2.4 Express Checkout (Apple Pay / Google Pay)

```mermaid
sequenceDiagram
    participant C as Consumer
    participant SF as Storefront
    participant CHK as Checkout
    participant PAY as Payments
    participant ORD as Orders
    participant STRIPE as Stripe

    C->>SF: Click "Buy Now with Apple Pay"
    SF->>CHK: POST /checkout/express
    CHK->>PAY: Create PaymentIntent
    PAY->>STRIPE: POST /payment_intents
    STRIPE-->>PAY: client_secret
    PAY-->>CHK: PaymentIntent
    CHK-->>SF: Checkout session
    SF->>C: Show Apple Pay sheet
    C->>SF: Authenticate (Touch ID / Face ID)
    SF->>STRIPE: Confirm payment (client-side)
    STRIPE-->>SF: Payment confirmed
    SF->>CHK: POST /checkout/complete
    CHK->>ORD: Create order
    ORD-->>CHK: Order created
    CHK-->>SF: Order confirmation
    SF-->>C: Thank you page
```

### 2.5 Subscription Signup and Management

```mermaid
flowchart TD
    BROWSE[Browse Subscription Products] --> SELECT[Select Plan]
    SELECT --> FREQ{Frequency?}
    FREQ -->|Weekly| W[Weekly Delivery]
    FREQ -->|Biweekly| BW[Every 2 Weeks]
    FREQ -->|Monthly| M[Monthly Delivery]
    FREQ -->|Quarterly| Q[Quarterly Box]

    W & BW & M & Q --> CUSTOM[Customize Box Contents]
    CUSTOM --> PAY[Set Up Payment Method]
    PAY --> CONFIRM[Confirm Subscription]
    CONFIRM --> ACTIVE[Subscription Active]

    ACTIVE --> MANAGE{Manage Subscription}
    MANAGE -->|Skip| SKIP[Skip Next Delivery]
    MANAGE -->|Swap| SWAP[Swap Products]
    MANAGE -->|Pause| PAUSE[Pause Subscription]
    MANAGE -->|Cancel| CANCEL[Cancel Flow]

    SKIP --> SKIPCONF[Skip Confirmed<br/>Next delivery postponed]
    SWAP --> SWAPSEL[Select Replacement Products]
    SWAPSEL --> SWAPCONF[Swap Confirmed]
    PAUSE --> PAUSECONF[Paused<br/>Resume anytime]
    CANCEL --> REASON[Cancellation Reason Survey]
    REASON --> OFFER{Retention Offer?}
    OFFER -->|Accept| ACTIVE
    OFFER -->|Decline| CANCELLED[Subscription Cancelled]
```

## 3. Social Commerce Workflows

### 3.1 Group Buying Campaign

```mermaid
flowchart TD
    CREATE[Merchant Creates Campaign] --> CONFIG[Set Product + Min Participants + Discount + Duration]
    CONFIG --> PUBLISH[Publish Campaign]
    PUBLISH --> SHARE[Participants Share Link]
    SHARE --> JOIN[New Participants Join]
    JOIN --> CHECK{Threshold Met?}
    CHECK -->|No| TIMER{Time Remaining?}
    TIMER -->|Yes| SHARE
    TIMER -->|No| FAIL[Campaign Failed<br/>All Refunded]
    CHECK -->|Yes| SUCCESS[Campaign Success]
    SUCCESS --> ORDERS[Create Orders for All Participants]
    ORDERS --> FULFILL[Process Fulfillment]
```

### 3.2 Livestream Shopping

```mermaid
sequenceDiagram
    participant HOST as Livestream Host
    participant SOC as Social Commerce
    participant VIEWER as Viewers
    participant CHK as Checkout
    participant KFK as Kafka

    HOST->>SOC: Start Livestream
    SOC->>SOC: Initialize WebRTC Stream
    SOC->>KFK: livestream.started
    VIEWER->>SOC: Join Stream
    HOST->>SOC: Pin Product (real-time)
    SOC->>VIEWER: Show Product Overlay
    VIEWER->>SOC: "Buy Now" on pinned product
    SOC->>CHK: Express checkout (1-click)
    CHK-->>SOC: Order created
    SOC-->>VIEWER: Purchase confirmed
    SOC->>KFK: livestream.purchase
    HOST->>SOC: Pin Next Product
    Note over HOST,VIEWER: Cycle continues
    HOST->>SOC: End Livestream
    SOC->>KFK: livestream.ended (stats)
```

### 3.3 Referral Program

```mermaid
flowchart TD
    REF[Customer Gets Referral Link] --> SHARE[Share via Social/Email/SMS]
    SHARE --> CLICK[Friend Clicks Link]
    CLICK --> COOKIE[Referral Cookie Set (30 days)]
    COOKIE --> SIGNUP[Friend Creates Account]
    SIGNUP --> PURCHASE[Friend Makes First Purchase]
    PURCHASE --> VALIDATE[Validate Referral]
    VALIDATE --> REWARD_REF[Reward Referrer<br/>$10 credit / 500 points]
    VALIDATE --> REWARD_NEW[Reward New Customer<br/>10% off first order]
    REWARD_REF --> NOTIFY_REF[Notify Referrer]
    REWARD_NEW --> NOTIFY_NEW[Notify New Customer]
```

## 4. Merchant Workflows

### 4.1 Product Management

```mermaid
flowchart TD
    START[Merchant Admin] --> ADD{Add Products?}
    ADD -->|Single| FORM[Product Form]
    ADD -->|Bulk| CSV[CSV Import]

    FORM --> DETAILS[Name, Description, Price, SKU]
    DETAILS --> VARIANTS[Add Variants<br/>Size, Color, Material]
    VARIANTS --> IMAGES[Upload Images to MinIO]
    IMAGES --> SEO[SEO Metadata]
    SEO --> CATEGORY[Assign Categories + Tags]
    CATEGORY --> STATUS{Publish?}
    STATUS -->|Draft| DRAFT[Save as Draft]
    STATUS -->|Active| PUBLISH[Publish to Storefront]
    PUBLISH --> SYNC[Sync to Social Channels]
    SYNC --> INDEX[Index in Search]

    CSV --> MAP[Map CSV Columns]
    MAP --> VALIDATE[Validate Data]
    VALIDATE --> IMPORT[Bulk Import]
    IMPORT --> REVIEW[Review Import Results]
```

### 4.2 Order Fulfillment Workflow

```mermaid
flowchart TD
    NEW[New Order Received] --> AUTO{Auto-Assign Warehouse?}
    AUTO -->|Yes| ROUTE[Multi-Warehouse Routing]
    AUTO -->|No| MANUAL[Manual Assignment]
    ROUTE --> PICK[Generate Pick List]
    MANUAL --> PICK

    PICK --> SCAN[Scan Items]
    SCAN --> VERIFY{All Items Scanned?}
    VERIFY -->|No| SCAN
    VERIFY -->|Yes| PACK[Pack Items]
    PACK --> WEIGHT[Record Package Weight]
    WEIGHT --> LABEL[Generate Shipping Label<br/>EasyPost / Shippo]
    LABEL --> PRINT[Print Label + Packing Slip]
    PRINT --> HANDOFF[Hand to Carrier]
    HANDOFF --> TRACK[Tracking Number Assigned]
    TRACK --> NOTIFY[Email Tracking to Customer]
    NOTIFY --> MONITOR[Monitor Delivery Status]
    MONITOR --> DELIVERED[Delivered]
```

### 4.3 Returns and RMA Workflow

```mermaid
flowchart TD
    REQ[Customer Requests Return] --> RMA[Generate RMA Number]
    RMA --> REASON[Select Return Reason]
    REASON --> APPROVE{Auto-Approve?}
    APPROVE -->|Policy Met| AUTO[Auto-Approve]
    APPROVE -->|Review Needed| REVIEW[Manual Review]
    REVIEW --> DECISION{Approved?}
    DECISION -->|No| DENY[Deny with Reason]
    DECISION -->|Yes| AUTO
    AUTO --> LABEL[Generate Return Label]
    LABEL --> SHIP_BACK[Customer Ships Item]
    SHIP_BACK --> RECEIVE[Warehouse Receives Return]
    RECEIVE --> INSPECT[Inspect Condition]
    INSPECT --> RESTOCK{Restockable?}
    RESTOCK -->|Yes| INVENTORY[Return to Inventory]
    RESTOCK -->|No| DISPOSE[Dispose / Donate]
    INVENTORY --> REFUND[Process Refund]
    DISPOSE --> REFUND
    REFUND --> NOTIFY_CUST[Notify Customer of Refund]
```

## 5. n8n Workflow Automations

### 5.1 Registered Workflows

| Workflow | Kafka Trigger | Actions | Purpose |
|----------|--------------|---------|---------|
| Order Processing | order.created | Fraud check -> Reserve inventory -> Process payment -> Create fulfillment | Core order pipeline |
| Cart Abandonment | cart.abandoned | Wait 1h -> Email 10% off -> Wait 24h -> Email 15% off -> Wait 48h -> SMS | Revenue recovery |
| Low Stock Alert | inventory.low_stock | Check threshold -> Email merchant -> Create PO suggestion | Inventory management |
| Review Moderation | review.submitted | AI content check -> Auto-approve or flag -> Notify merchant | Quality control |
| Subscription Renewal | subscription.due | Charge payment -> Create order -> Update next date | Recurring billing |
| Loyalty Tier Evaluation | order.completed | Calculate 12m spend -> Evaluate tier -> Upgrade/downgrade | Customer retention |
| Social Sync | product.created | Format for platform -> Sync to Instagram -> Sync to Facebook -> Sync to TikTok | Multi-channel |
| Refund Processing | refund.approved | Process via Stripe -> Update order status -> Email customer -> Restore points | Financial ops |

### 5.2 Cart Abandonment Recovery Workflow Detail

```mermaid
flowchart TD
    TRIGGER[Kafka: cart.abandoned] --> WAIT1[Wait 1 Hour]
    WAIT1 --> CHECK1{Cart Still Abandoned?}
    CHECK1 -->|No - Converted| END[End Workflow]
    CHECK1 -->|Yes| EMAIL1[Send Email<br/>'You left items in your cart'<br/>Include 10% coupon]
    EMAIL1 --> WAIT2[Wait 24 Hours]
    WAIT2 --> CHECK2{Cart Still Abandoned?}
    CHECK2 -->|No| END
    CHECK2 -->|Yes| EMAIL2[Send Email<br/>'Still interested?'<br/>Include 15% coupon]
    EMAIL2 --> WAIT3[Wait 48 Hours]
    WAIT3 --> CHECK3{Cart Still Abandoned?}
    CHECK3 -->|No| END
    CHECK3 -->|Yes| SMS[Send SMS<br/>'Complete your purchase'<br/>Direct link to cart]
    SMS --> ANALYTICS[Record Recovery Attempt]
    ANALYTICS --> END
```

## 6. Loyalty Journey

```mermaid
journey
    title Customer Loyalty Journey
    section Join
      Make first purchase: 5: Customer
      Auto-enrolled as Bronze: 4: System
      Earn first points: 5: System
    section Engage
      Receive points notification: 4: Customer
      Redeem points for discount: 5: Customer
      Daily check-in bonus: 3: Customer
    section Grow
      Reach Silver tier ($500 spend): 4: Customer
      Unlock Silver benefits: 5: System
      Birthday bonus points: 4: System
    section Advocate
      Share referral link: 4: Customer
      Friend makes purchase: 3: Friend
      Earn referral bonus: 5: System
    section Retain
      Approach Gold tier: 4: Customer
      Exclusive Gold member sale: 5: System
      VIP early access: 5: Customer
```
