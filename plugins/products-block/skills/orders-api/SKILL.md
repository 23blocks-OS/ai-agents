---
name: products-orders-api
description: Manage 23blocks orders via REST API. Use when updating orders or managing order product statuses (order, ship, deliver, cancel) within carts.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Orders API

Complete API reference for 23blocks order lifecycle management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://products.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### PUT /orders/:unique_id - Update Order

Updates order details.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "order": {
      "shipping_address": "456 Customer St, Miami FL 33101",
      "notes": "Leave at front door",
      "tracking_number": "1Z999AA10123456784"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `shipping_address` | string | No | Shipping address |
| `notes` | string | No | Order notes |
| `tracking_number` | string | No | Shipping tracking number |

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-123",
      "shipping_address": "456 Customer St, Miami FL 33101",
      "notes": "Leave at front door",
      "tracking_number": "1Z999AA10123456784",
      "status": "ordered",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Order not found

---

## Order Product Status Transitions

Manage status transitions for individual products within an order.

### PUT /carts/:unique_id/products/:product_unique_id/order - Order Product

Marks a specific product within a cart as ordered.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/products/product-uuid-001/order" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-001",
    "type": "order_detail",
    "attributes": {
      "product_unique_id": "product-uuid-001",
      "status": "ordered",
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /carts/:unique_id/products/:product_unique_id/ship - Ship Product

Marks a product as shipped.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/products/product-uuid-001/ship" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-001",
    "type": "order_detail",
    "attributes": {
      "product_unique_id": "product-uuid-001",
      "status": "shipped",
      "shipped_at": "2025-01-13T09:00:00Z"
    }
  }
}
```

---

### PUT /carts/:unique_id/products/:product_unique_id/deliver - Deliver Product

Marks a product as delivered.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/products/product-uuid-001/deliver" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-001",
    "type": "order_detail",
    "attributes": {
      "product_unique_id": "product-uuid-001",
      "status": "delivered",
      "delivered_at": "2025-01-15T14:00:00Z"
    }
  }
}
```

---

### PUT /carts/:unique_id/products/:product_unique_id/cancel - Cancel Product

Cancels a product within an order.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/products/product-uuid-001/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-001",
    "type": "order_detail",
    "attributes": {
      "product_unique_id": "product-uuid-001",
      "status": "cancelled",
      "cancelled_at": "2025-01-12T16:00:00Z"
    }
  }
}
```

---

## Data Models

### Order
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `cart_unique_id` | uuid | Associated cart |
| `order_number` | string | Human-readable order number |
| `shipping_address` | string | Shipping address |
| `notes` | string | Order notes |
| `tracking_number` | string | Shipping tracking number |
| `status` | enum | ordered, processing, shipped, delivered, cancelled |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### OrderDetail
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `product_unique_id` | uuid | Product identifier |
| `quantity` | integer | Item quantity |
| `unit_price` | decimal | Price per unit |
| `status` | enum | ordered, shipped, delivered, cancelled |
| `shipped_at` | timestamp | Ship time |
| `delivered_at` | timestamp | Delivery time |
| `cancelled_at` | timestamp | Cancellation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Order Not Found",
    "detail": "The requested order could not be found."
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
```

### TypeScript Types

```typescript
import type {
  Cart,
  CartDetail,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: ship an order detail
  const result = await client.products.cartDetails.ship('cart-uuid', 'detail-uuid');
}
```
