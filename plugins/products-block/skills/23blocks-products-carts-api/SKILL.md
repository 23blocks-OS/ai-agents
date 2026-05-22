---
name: 23blocks-products-carts-api
description: Manage 23blocks shopping carts via REST API. Use when creating carts, updating cart items, processing checkout, placing orders, managing cart detail statuses, handling authenticated user carts (mycarts), creating guests (or legacy visitors), or running abandoned cart remarketing.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Carts API

Complete API reference for 23blocks shopping cart management with checkout, orders, and remarketing.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://products.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://products.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/carts/:user_unique_id` | Get user's current cart |
| GET | `/carts/:cart_unique_id/logs` | Get cart activity logs |
| POST | `/carts/` | Create a new cart |
| PUT | `/carts/:user_unique_id` | Update cart with product items |
| PUT | `/carts/:user_unique_id/services` | Update cart services (shipping, etc.) |
| PUT | `/carts/:unique_id/save` | Save current cart state |
| POST | `/carts/:user_unique_id/checkout` | Process cart checkout |
| PUT | `/carts/:unique_id/order` | Place order from cart |
| PUT | `/carts/:unique_id/cancel` | Cancel a cart or order |
| DELETE | `/carts/:user_unique_id` | Clear all items from cart |
| PUT | `/carts/:unique_id/order/marketplace` | Place a marketplace order |
| PUT | `/carts/:unique_id/details/:details_unique_id/order` | Mark detail as ordered |
| PUT | `/carts/:unique_id/details/:details_unique_id/accepted` | Mark detail as accepted |
| PUT | `/carts/:unique_id/details/:details_unique_id/start` | Mark detail as started |
| PUT | `/carts/:unique_id/details/:details_unique_id/processing` | Mark detail as processing |
| PUT | `/carts/:unique_id/details/:details_unique_id/ready` | Mark detail as ready |
| PUT | `/carts/:unique_id/details/:details_unique_id/ship` | Mark detail as shipped |
| PUT | `/carts/:unique_id/details/:details_unique_id/deliver` | Mark detail as delivered |
| PUT | `/carts/:unique_id/details/:details_unique_id/cancel` | Cancel a cart detail |
| PUT | `/carts/:unique_id/details/:details_unique_id/return` | Mark detail as returned |
| GET | `/mycarts/:unique_id` | Get authenticated user's cart |
| POST | `/mycarts/` | Create authenticated user's cart |
| PUT | `/mycarts/:unique_id` | Update authenticated user's cart |
| POST | `/mycarts/:unique_id/checkout` | Checkout authenticated user's cart |
| PUT | `/mycarts/:unique_id/order` | Place authenticated user's order |
| PUT | `/mycarts/:unique_id/cancel` | Cancel authenticated user's cart |
| DELETE | `/mycarts/:unique_id` | Delete authenticated user's cart |
| POST | `/guests/` | Create guest session (preferred) |
| GET | `/guests/:user_unique_id` | Get guest |
| PUT | `/guests/:user_unique_id` | Update guest |
| PUT | `/guests/:unique_id/convert` | Convert guest to user |
| POST | `/guests/:unique_id/auth` | Authenticate guest |
| POST | `/visitors` | Create visitor session (**legacy alias** for `POST /guests/`) |
| POST | `/tools/remarketing/carts/abandoned` | Get abandoned carts for remarketing |

---

## Data Models

### Cart
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User identifier |
| `status` | enum | open, checkout, ordered, cancelled, abandoned |
| `subtotal` | decimal | Cart subtotal |
| `tax` | decimal | Tax amount |
| `shipping` | decimal | Shipping cost |
| `total` | decimal | Cart total |
| `currency` | string | Currency code |
| `items_count` | integer | Number of items |
| `order_number` | string | Order number (after ordering) |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### CartDetail
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `product_unique_id` | uuid | Product identifier |
| `variation_unique_id` | uuid | Variation identifier (optional) |
| `quantity` | integer | Item quantity |
| `unit_price` | decimal | Price per unit |
| `total` | decimal | Line total |
| `status` | enum | ordered, accepted, started, processing, ready, shipped, delivered, cancelled, returned |

### Visitor
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `session_id` | string | Browser session ID |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Cart Not Found",
    "detail": "The requested cart could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-products
```

### Setup

```typescript
import { create23BlocksClient } from '@23blocks/sdk';

const client = create23BlocksClient({
  authToken: process.env.BLOCKS_AUTH_TOKEN!,
  apiKey: process.env.BLOCKS_API_KEY!,
  apiUrl: process.env.BLOCKS_API_URL!,
});
```

### Available Methods

```typescript
// CartService — client.products.cart
get(userUniqueId: string): Promise<Cart>;
getOrCreate(userUniqueId: string): Promise<Cart>;
update(userUniqueId: string, data: UpdateCartRequest): Promise<Cart>;
delete(userUniqueId: string): Promise<void>;
addItem(userUniqueId: string, item: AddToCartRequest): Promise<Cart>;
updateItem(userUniqueId: string, productUniqueId: string, data: UpdateCartItemRequest): Promise<Cart>;
removeItem(userUniqueId: string, productUniqueId: string): Promise<Cart>;
getItems(userUniqueId: string): Promise<CartDetail[]>;
checkout(userUniqueId: string, data?: CheckoutRequest): Promise<Cart>;
order(userUniqueId: string): Promise<{ cart: Cart; orderUniqueId: string }>;
orderItem(userUniqueId: string, productUniqueId: string): Promise<{ cart: Cart; orderUniqueId: string }>;
cancel(userUniqueId: string): Promise<Cart>;
cancelItem(userUniqueId: string, productUniqueId: string): Promise<Cart>;

// CartDetailsService — client.products.cartDetails
order(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
accept(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
startProcessing(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
processing(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
ready(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
ship(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
deliver(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
cancel(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;
return(cartUniqueId: string, detailUniqueId: string): Promise<CartDetail>;

// MyCartsService — client.products.myCarts
get(uniqueId: string): Promise<Cart>;
create(): Promise<Cart>;
update(uniqueId: string, data: UpdateMyCartRequest): Promise<Cart>;
addToCart(data: AddToMyCartRequest): Promise<Cart>;
checkout(uniqueId: string): Promise<Cart>;
orderAll(uniqueId: string): Promise<Cart>;
cancelAll(uniqueId: string): Promise<Cart>;
delete(uniqueId: string): Promise<void>;

// GuestsService — client.products.guests (preferred)
create(data: CreateGuestRequest): Promise<Guest>;
get(userUniqueId: string): Promise<Guest>;
update(userUniqueId: string, data: UpdateGuestRequest): Promise<Guest>;
convert(uniqueId: string): Promise<User>;
auth(uniqueId: string): Promise<AuthToken>;

// VisitorsService — client.products.visitors (legacy alias for guests)
create(data: CreateVisitorRequest): Promise<Visitor>;

// RemarketingService — client.products.remarketing
getAbandonedCarts(params?: AbandonedCartsParams): Promise<{ carts: AbandonedCart[]; total: number }>;
```

### TypeScript Types

```typescript
import type {
  Cart,
  CartDetail,
  AddToCartRequest,
  UpdateCartItemRequest,
  UpdateCartRequest,
  CheckoutRequest,
  CartResponse,
  OrderCartResponse,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: get or create a cart for a user
  const cart = await client.products.cart.getOrCreate('user-uuid-123');
}
```
