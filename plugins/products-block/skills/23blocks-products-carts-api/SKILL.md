---
name: 23blocks-products-carts-api
description: Manage 23blocks shopping carts via REST API. Use when creating carts, updating cart items, processing checkout, placing orders, managing cart detail statuses, handling authenticated user carts (mycarts), creating visitors, or running abandoned cart remarketing.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Carts API

Complete API reference for 23blocks shopping cart management with checkout, orders, and remarketing.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/carts/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
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
| POST | `/visitors` | Create visitor session |
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

// VisitorsService — client.products.visitors
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
