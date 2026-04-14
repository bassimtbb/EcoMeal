# EcoMeal — Design Workflow (Steps 1–3)
> **Course:** Conception d'Application & Architecture N-Tier — Prof. Addi  
> **Project:** EcoMeal — Reducing food waste by connecting local businesses with users to sell unsold food at a discount.

---

## Table of Contents

1. [Step 1 — Raw Functionality List](#step-1--raw-functionality-list)
2. [Step 2 — Grouping by Business Domain (DDD)](#step-2--grouping-by-business-domain-ddd)
3. [Step 3 — Business Entities per Module](#step-3--business-entities-per-module)

---

## Step 1 — Raw Functionality List

> **Method:** Exhaustive, unsorted list. We are in "functional pure" mode — *what* the app does, not *how* it's built. Three perspectives: Consumer, Business Partner, and System Admin.

### 👤 Consumer Perspective

- Register a new user account (email + password)
- Log in / Log out
- Reset forgotten password via email
- View and edit personal profile (name, address, dietary preferences)
- Browse available meal offers on a map or list view
- Filter offers by category (bakery, restaurant, sushi…), distance, price, or dietary tag (vegan, gluten-free…)
- Search for offers by business name or food type
- View a detailed offer page (photos, description, quantity, pickup time window, original price, discounted price)
- Add an offer to a shopping cart
- Remove an item from the shopping cart
- Apply a discount or promo code
- Place an order and pay online
- Receive an order confirmation (in-app + email)
- View current and past orders
- Cancel an order (within a time limit)
- Generate and display a QR code for in-store pickup
- Rate and review a business after pickup
- Save favourite businesses
- Receive push/email notifications for new offers nearby
- Manage notification preferences (frequency, type)
- Report a problem with an order or a business
- Request a refund or open a dispute

### 🏪 Business Partner Perspective

- Register a business account (name, address, SIRET/registration number, type)
- Upload business logo and photos
- Submit account for admin verification/approval
- Log in to a business dashboard
- Create a new meal offer (title, description, photos, category, quantity, pickup window, original price, discount price, expiry time)
- Edit an existing offer
- Deactivate or delete an offer
- Mark an offer as "sold out"
- View real-time stock count for active offers
- View incoming orders for the day
- Scan a consumer's QR code to validate pickup
- Mark an order as picked up / completed
- View business analytics (number of meals saved, revenue, average rating, waste reduction stats)
- Receive payouts for completed orders
- View payout history and invoices
- Respond to consumer reviews
- Manage business opening hours
- Set geographic service radius or pickup location
- Receive notifications when an offer is running low on stock
- Receive notification of a new order

### 🛡️ System Admin Perspective

- View and manage all user accounts (consumers + businesses)
- Approve or reject business registration requests
- Suspend or ban an account (consumer or business)
- Moderate reported content (reviews, offers)
- Manage platform-wide discount codes and promotions
- View platform analytics dashboard (total meals saved, total orders, active users, revenue)
- Manage categories and food tags
- Configure commission rates per business type
- Process refund requests and disputes
- Send system-wide announcements or notifications
- Monitor system health (API uptime, error rates)
- Manage content (FAQs, terms of service, privacy policy)

---

## Step 2 — Grouping by Business Domain (DDD)

> **Principle applied:** High Cohesion within each module (all features share the same business reason to change), Low Coupling between modules (each module exposes clean interfaces, does not depend on internals of another).

After analysing the feature list, six **Bounded Contexts** emerge naturally:

| # | Module / Bounded Context | Core Responsibility | Key Trigger for Separation |
|---|---|---|---|
| 1 | **UserIdentity** | Who are you? Authentication & profile | Auth/security changes independently of everything else |
| 2 | **Catalog** | What is available? Offer lifecycle management | Businesses publish/edit offers; changes frequently |
| 3 | **Order** | What was purchased? Order lifecycle & QR validation | Payment, fulfillment, and dispute rules are complex and distinct |
| 4 | **Payment** | How is money handled? Charges, payouts, refunds | Financial logic is regulated and completely isolated |
| 5 | **Notification** | How are users informed? Alerts, emails, push | Communication channel logic changes independently |
| 6 | **Administration** | Who runs the platform? Moderation & governance | Admin workflows are internal and never exposed to end users |

---

### Module 1 — `UserIdentity`

**Cohesion rationale:** All features here have one reason to change — how users are authenticated and how their personal profile is managed. Sending an email to a user is *not* this module's job (that belongs to `Notification`).

| Feature | Origin |
|---|---|
| Register a new consumer account | Consumer |
| Log in / Log out | Consumer |
| Reset forgotten password | Consumer |
| View and edit personal profile | Consumer |
| Manage notification preferences | Consumer |
| Register a business account | Business |
| Submit business for admin approval | Business |
| Log in to business dashboard | Business |
| Manage user / business accounts (admin) | Admin |
| Approve / reject / suspend accounts | Admin |

---

### Module 2 — `Catalog`

**Cohesion rationale:** Everything related to *what food is available* lives here. The offer is the central artifact: its creation, visibility, stock, and discovery. Geographic search is also here because it directly queries offers. Pricing rules (discounts, commissions) are deferred to `Payment` and `Administration`.

| Feature | Origin |
|---|---|
| Create a new meal offer | Business |
| Edit an existing offer | Business |
| Deactivate / delete an offer | Business |
| Mark an offer as "sold out" | Business |
| View real-time stock count | Business |
| Manage categories and food tags | Admin |
| Browse offers (map / list view) | Consumer |
| Filter offers (category, distance, price, dietary tag) | Consumer |
| Search offers by name or type | Consumer |
| View detailed offer page | Consumer |
| Save favourite businesses | Consumer |
| Upload business logo and photos | Business |
| Manage business opening hours | Business |
| Set pickup location / service radius | Business |

---

### Module 3 — `Order`

**Cohesion rationale:** All features share the same business lifecycle — from cart to pickup confirmation. QR validation belongs here because it is a step in the order fulfillment workflow, not a communication event. Disputes are initiated here but *resolved* in `Payment`.

| Feature | Origin |
|---|---|
| Add / remove item from cart | Consumer |
| Apply promo code | Consumer |
| Place an order | Consumer |
| Receive order confirmation | Consumer |
| View current and past orders | Consumer |
| Cancel an order | Consumer |
| Generate QR code for pickup | Consumer |
| Scan QR code to validate pickup | Business |
| Mark an order as picked up / completed | Business |
| View incoming orders for the day | Business |
| Report a problem with an order | Consumer |
| View orders and disputes (admin) | Admin |

---

### Module 4 — `Payment`

**Cohesion rationale:** All money movement is isolated here. This module integrates with external payment providers (Stripe, etc.) and is subject to financial regulation — it must be able to evolve (or be replaced) without touching `Order` or `Catalog`.

| Feature | Origin |
|---|---|
| Process online payment at checkout | Consumer |
| Process refund request | Consumer |
| Open and resolve a dispute | Consumer |
| Receive payouts for completed orders | Business |
| View payout history and invoices | Business |
| Configure commission rates | Admin |
| Process refund and dispute resolution | Admin |
| Manage platform-wide promo codes | Admin |

---

### Module 5 — `Notification`

**Cohesion rationale:** All communication events are centralised here regardless of their channel (email, push, in-app). Other modules emit *Domain Events* (e.g., `OrderPlaced`, `OfferLowStock`); this module *listens* and decides how to communicate. This is a classic Observer pattern — the emitter has zero knowledge of the notification channel.

| Feature | Origin |
|---|---|
| Send order confirmation (in-app + email) | System |
| Send push/email alerts for new offers nearby | Consumer |
| Notify business of new order | Business |
| Notify business when offer stock is low | Business |
| Send system-wide announcements | Admin |
| Manage notification preferences (consumer) | Consumer |

---

### Module 6 — `Administration`

**Cohesion rationale:** Governance, moderation, and platform-level configuration are internal operations that no consumer or business can access. These features change at the pace of business policy — independently of product features.

| Feature | Origin |
|---|---|
| Moderate reported content (reviews, offers) | Admin |
| View platform analytics dashboard | Admin |
| Monitor system health | Admin |
| Manage content (FAQs, ToS, privacy policy) | Admin |
| View business analytics dashboard | Business (read-only view) |
| Rate and review a business | Consumer |
| Respond to consumer reviews | Business |

---

### Module Dependency Map

```
                  ┌─────────────────┐
                  │  UserIdentity   │
                  └────────┬────────┘
                           │ (auth token / user identity)
          ┌────────────────┼──────────────────┐
          ▼                ▼                  ▼
   ┌─────────────┐  ┌─────────────┐  ┌──────────────┐
   │   Catalog   │  │    Order    │  │ Administration│
   └──────┬──────┘  └──────┬──────┘  └──────────────┘
          │                │
          │  (offer ref)   │  (order events)
          └────────┬───────┘
                   ▼
           ┌──────────────┐
           │   Payment    │
           └──────┬───────┘
                  │
                  │ (domain events: OrderPlaced,
                  │  PaymentConfirmed, OfferLowStock…)
                  ▼
          ┌───────────────┐
          │  Notification │
          └───────────────┘
```

> **Low Coupling enforced:** `Notification` never imports from `Order` or `Catalog` directly. It only reacts to published Domain Events. `Catalog` never knows `Payment` exists — it simply stores a price. `Order` calls `Payment` through an interface (Dependency Inversion Principle).

---

## Step 3 — Business Entities per Module

> **Vocabulary:**
> - **Entity** — Has a unique identity, is mutable over time (e.g., `Order` changes status)
> - **Value Object** — No identity, immutable, defined entirely by its values (e.g., `Money(8.50, EUR)`)
> - **Aggregate Root** — The single entry point to a cluster of related entities; external modules only hold a reference to the root's ID

---

### Module 1 — `UserIdentity`

#### Entity: `User`
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `email` | String | Registration |
| `passwordHash` | String | Registration |
| `role` | Enum: `CONSUMER, BUSINESS, ADMIN` | Registration type |
| `status` | Enum: `PENDING, ACTIVE, SUSPENDED` | Admin approval |
| `createdAt` | DateTime | System |

#### Entity: `ConsumerProfile` *(owned by User)*
| Attribute | Type | Derived from |
|---|---|---|
| `userId` | UUID | FK to User |
| `firstName` | String | Profile edit |
| `lastName` | String | Profile edit |
| `dietaryPreferences` | List\<Tag\> | Profile edit |
| `notificationPreferences` | NotificationPreferences | Settings |

#### Entity: `BusinessProfile` *(owned by User)*
| Attribute | Type | Derived from |
|---|---|---|
| `userId` | UUID | FK to User |
| `businessName` | String | Registration |
| `registrationNumber` | String | Registration (SIRET) |
| `businessType` | Enum: `RESTAURANT, BAKERY, SUSHI, ...` | Registration |
| `logoUrl` | String | Upload |
| `approvalStatus` | Enum: `PENDING, APPROVED, REJECTED` | Admin action |

#### Value Object: `NotificationPreferences`
| Attribute | Type |
|---|---|
| `emailEnabled` | Boolean |
| `pushEnabled` | Boolean |
| `nearbyOffersAlerts` | Boolean |
| `orderUpdatesEnabled` | Boolean |

---

### Module 2 — `Catalog`

#### Aggregate Root: `Offer`
> An `Offer` is the central artifact of the entire platform. Its lifecycle (DRAFT → ACTIVE → SOLD_OUT → EXPIRED) drives many downstream events.

| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `businessId` | UUID | FK to BusinessProfile |
| `title` | String | Feature: Create offer |
| `description` | String | Feature: Create offer |
| `photoUrls` | List\<String\> | Feature: Upload photos |
| `category` | Enum: `BAKERY, RESTAURANT, SUSHI, ...` | Feature: Tag filtering |
| `dietaryTags` | List\<Tag\> | Feature: Filter by diet |
| `originalPrice` | Money | Feature: Show original price |
| `discountedPrice` | Money | Feature: Discounted selling price |
| `totalQuantity` | Integer | Feature: Set stock |
| `remainingQuantity` | Integer | Feature: Real-time stock |
| `pickupWindow` | TimeWindow | Feature: Pickup time |
| `expiresAt` | DateTime | Feature: Offer expiry |
| `status` | Enum: `DRAFT, ACTIVE, SOLD_OUT, EXPIRED, DELETED` | Lifecycle |
| `pickupLocation` | Address | Feature: Set location |

#### Entity: `Business` *(read-only projection from UserIdentity)*
> `Catalog` only holds a `businessId` reference. Full business data lives in `UserIdentity`. This enforces Low Coupling.

#### Value Object: `Money`
| Attribute | Type |
|---|---|
| `amount` | BigDecimal |
| `currency` | Enum: `EUR, USD, ...` |

#### Value Object: `TimeWindow`
| Attribute | Type |
|---|---|
| `startTime` | LocalTime |
| `endTime` | LocalTime |
| `date` | LocalDate |

#### Value Object: `Address`
| Attribute | Type |
|---|---|
| `street` | String |
| `city` | String |
| `postalCode` | String |
| `latitude` | Double |
| `longitude` | Double |

#### Entity: `Category`
| Attribute | Type |
|---|---|
| `id` | UUID |
| `name` | String |
| `iconUrl` | String |

#### Entity: `Tag` *(dietary / food label)*
| Attribute | Type |
|---|---|
| `id` | UUID |
| `label` | String (e.g., "vegan", "gluten-free") |

---

### Module 3 — `Order`

#### Aggregate Root: `Order`
> The `Order` aggregate encapsulates the full purchase lifecycle. The `Cart` is a transient pre-order state — it becomes an `Order` upon payment confirmation. No external module modifies an `Order` directly; all changes go through the aggregate root.

| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `consumerId` | UUID | FK to UserIdentity |
| `businessId` | UUID | FK to UserIdentity |
| `items` | List\<OrderItem\> | Feature: Add to cart |
| `subtotal` | Money | Computed |
| `discountApplied` | Money | Feature: Promo code |
| `totalAmount` | Money | Computed |
| `promoCode` | String (nullable) | Feature: Apply promo |
| `status` | Enum: `PENDING, CONFIRMED, READY, PICKED_UP, CANCELLED, DISPUTED` | Lifecycle |
| `pickupCode` | String (QR payload) | Feature: Generate QR |
| `placedAt` | DateTime | System |
| `confirmedAt` | DateTime | Payment event |
| `pickedUpAt` | DateTime | Business scan |

#### Entity: `OrderItem` *(part of Order aggregate)*
| Attribute | Type | Derived from |
|---|---|---|
| `offerId` | UUID | FK to Catalog.Offer |
| `offerTitle` | String | Snapshot at order time |
| `unitPrice` | Money | Snapshot at order time |
| `quantity` | Integer | Feature: Add to cart |

> **Design note:** `offerTitle` and `unitPrice` are *snapshotted* at order creation. This is critical — if a business edits or deletes an offer, historical orders must remain intact. This is the **Immutable Snapshot** pattern.

#### Entity: `Dispute`
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `orderId` | UUID | FK to Order |
| `consumerId` | UUID | FK to UserIdentity |
| `reason` | String | Feature: Report problem |
| `status` | Enum: `OPEN, UNDER_REVIEW, RESOLVED, REJECTED` | Admin action |
| `resolution` | String (nullable) | Admin action |
| `createdAt` | DateTime | System |

---

### Module 4 — `Payment`

#### Aggregate Root: `Payment`
> All financial records are immutable once created (append-only). A `Refund` is a new record, never a mutation of the original `Payment`. This aligns with financial audit requirements and is a foundation for Event Sourcing if needed.

| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `orderId` | UUID | FK to Order |
| `consumerId` | UUID | FK to UserIdentity |
| `amount` | Money | Order total |
| `status` | Enum: `PENDING, SUCCEEDED, FAILED, REFUNDED` | Payment provider |
| `externalTransactionId` | String | Stripe / provider |
| `createdAt` | DateTime | System |

#### Entity: `Payout`
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `businessId` | UUID | FK to UserIdentity |
| `amount` | Money | Orders completed in period |
| `commissionDeducted` | Money | Feature: Commission config |
| `status` | Enum: `SCHEDULED, PROCESSED, FAILED` | Batch job |
| `periodStart` | LocalDate | Payout schedule |
| `periodEnd` | LocalDate | Payout schedule |

#### Entity: `Refund`
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `paymentId` | UUID | FK to Payment |
| `disputeId` | UUID (nullable) | FK to Dispute |
| `amount` | Money | Refund amount |
| `reason` | String | Admin / consumer |
| `status` | Enum: `PENDING, PROCESSED, FAILED` | Provider |

#### Value Object: `CommissionRate`
| Attribute | Type |
|---|---|
| `businessType` | Enum |
| `rate` | BigDecimal (e.g., 0.15 = 15%) |

---

### Module 5 — `Notification`

> This module contains **no business entities** in the DDD sense — it is purely reactive. It subscribes to Domain Events emitted by other modules and converts them into communications.

#### Entity: `NotificationLog` *(audit trail)*
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `recipientId` | UUID | Target user |
| `channel` | Enum: `EMAIL, PUSH, IN_APP` | Preferences |
| `type` | Enum: `ORDER_CONFIRMED, NEW_OFFER, LOW_STOCK, ...` | Domain Event |
| `payload` | JSON | Event data |
| `sentAt` | DateTime | System |
| `status` | Enum: `SENT, FAILED, SKIPPED` | Delivery result |

#### Domain Events consumed (from other modules):
| Event | Emitted by | Action |
|---|---|---|
| `OrderPlaced` | Order | Notify consumer (confirmation) + business (new order) |
| `OrderPickedUp` | Order | Notify consumer (receipt) |
| `OrderCancelled` | Order | Notify business |
| `OfferLowStock` | Catalog | Notify business |
| `BusinessApproved` | UserIdentity | Notify business |
| `DisputeResolved` | Payment | Notify consumer |

---

### Module 6 — `Administration`

#### Entity: `Review`
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `consumerId` | UUID | FK to UserIdentity |
| `businessId` | UUID | FK to UserIdentity |
| `orderId` | UUID | FK to Order |
| `rating` | Integer (1–5) | Feature: Rate business |
| `comment` | String | Feature: Write review |
| `businessReply` | String (nullable) | Feature: Business response |
| `status` | Enum: `VISIBLE, FLAGGED, REMOVED` | Moderation |
| `createdAt` | DateTime | System |

#### Entity: `PromoCode`
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `code` | String | Admin creates |
| `discountType` | Enum: `PERCENTAGE, FIXED` | Feature |
| `discountValue` | BigDecimal | Feature |
| `validFrom` | DateTime | Feature |
| `validUntil` | DateTime | Feature |
| `maxUses` | Integer | Feature |
| `currentUses` | Integer | Computed |
| `status` | Enum: `ACTIVE, EXPIRED, DISABLED` | Lifecycle |

#### Entity: `ModerationReport`
| Attribute | Type | Derived from |
|---|---|---|
| `id` | UUID | System-generated |
| `reportedBy` | UUID | FK to UserIdentity |
| `targetType` | Enum: `REVIEW, OFFER, USER` | Feature: Report |
| `targetId` | UUID | Target entity |
| `reason` | String | Feature: Report |
| `status` | Enum: `PENDING, REVIEWED, ACTIONED` | Admin |

---

## Design Summary — 4-Question Checklist

As taught by Prof. Addi, every feature must answer these four questions:

| Module | What module? | What data? | What logic? | What interaction? |
|---|---|---|---|---|
| UserIdentity | User lifecycle | User, ConsumerProfile, BusinessProfile | Password hashing, role assignment, approval flow | REST (POST /register, POST /login), Admin dashboard |
| Catalog | Offer availability | Offer, Category, Tag | Stock decrement, expiry check, geo-search | REST (GET /offers, POST /offers), Cron (expire offers) |
| Order | Purchase flow | Order, OrderItem, Dispute | Promo validation, QR generation, snapshot prices | REST (POST /orders, PATCH /orders/{id}/cancel), QR scan event |
| Payment | Money movement | Payment, Payout, Refund | Commission deduction, refund eligibility | Stripe webhook, REST (GET /payouts), Cron (weekly payout) |
| Notification | Communication | NotificationLog | Channel routing, preference filtering | Event listener (Kafka / Observer), Email/Push provider |
| Administration | Platform governance | Review, PromoCode, ModerationReport | Moderation rules, promo validity | Admin REST API, internal dashboard |

---

*Document produced as part of the EcoMeal TP — Design Workflow Steps 1–3.*  
*Next step: Step 4 — Deriving technical components and writing ADRs.*
