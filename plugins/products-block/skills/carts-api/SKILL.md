---
name: products-carts-api
description: Manage 23blocks shopping carts via REST API. Use when creating carts, updating cart items, processing checkout, placing orders, managing cart detail statuses, handling authenticated user carts (mycarts), creating visitors, or running abandoned cart remarketing.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Carts API

Complete API reference for 23blocks shopping cart management with checkout, orders, and remarketing.

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
curl -X GET "$BLOCKS_API_URL/carts/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Cart Endpoints

### GET /carts/:user_unique_id - Get User Cart

Retrieves the current cart for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/carts/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cart-uuid-123",
    "type": "cart",
    "attributes": {
      "unique_id": "cart-uuid-123",
      "user_unique_id": "user-uuid-123",
      "status": "open",
      "subtotal": 149.97,
      "tax": 12.00,
      "total": 161.97,
      "currency": "USD",
      "items_count": 3,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "details": {
        "data": [
          { "id": "detail-uuid-1", "type": "cart_detail" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Cart not found

---

### GET /carts/:cart_unique_id/logs - Cart Logs

Retrieves activity logs for a cart.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/carts/cart-uuid-123/logs" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "log-uuid-123",
      "type": "cart_log",
      "attributes": {
        "unique_id": "log-uuid-123",
        "action": "item_added",
        "description": "Product KB-001 added to cart",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /carts/ - Create Cart

Creates a new shopping cart.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/carts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "cart": {
      "user_unique_id": "user-uuid-123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User identifier |

**Response 201:**
```json
{
  "data": {
    "id": "new-cart-uuid",
    "type": "cart",
    "attributes": {
      "unique_id": "new-cart-uuid",
      "user_unique_id": "user-uuid-123",
      "status": "open",
      "subtotal": 0,
      "total": 0,
      "items_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /carts/:user_unique_id - Update Cart

Updates a user's cart with product items.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "cart": {
      "products": [
        {
          "product_unique_id": "product-uuid-001",
          "quantity": 2,
          "variation_unique_id": null
        },
        {
          "product_unique_id": "product-uuid-002",
          "quantity": 1,
          "variation_unique_id": "var-uuid-001"
        }
      ]
    }
  }'
```

**Response 200:** Updated cart object

---

### PUT /carts/:user_unique_id/services - Update Cart Services

Updates services applied to a cart (shipping, handling, etc.).

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/user-uuid-123/services" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "services": {
      "shipping_method": "express",
      "shipping_cost": 15.99,
      "gift_wrap": true
    }
  }'
```

**Response 200:** Updated cart object with services

---

### PUT /carts/:unique_id/save - Save Cart

Saves the current cart state.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/save" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Cart saved successfully"
}
```

---

### POST /carts/:user_unique_id/checkout - Checkout

Processes cart checkout.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/carts/user-uuid-123/checkout" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "checkout": {
      "shipping_address_id": "address-uuid",
      "billing_address_id": "address-uuid",
      "payment_method": "credit_card"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "cart-uuid-123",
    "type": "cart",
    "attributes": {
      "unique_id": "cart-uuid-123",
      "status": "checkout",
      "subtotal": 149.97,
      "tax": 12.00,
      "shipping": 15.99,
      "total": 177.96
    }
  }
}
```

---

### PUT /carts/:unique_id/order - Place Order

Places the order from a checked-out cart.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/order" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cart-uuid-123",
    "type": "cart",
    "attributes": {
      "unique_id": "cart-uuid-123",
      "status": "ordered",
      "order_number": "ORD-2025-001234",
      "ordered_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /carts/:unique_id/cancel - Cancel Cart

Cancels a cart or order.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cart-uuid-123",
    "type": "cart",
    "attributes": {
      "status": "cancelled"
    }
  }
}
```

---

### DELETE /carts/:user_unique_id - Clear Cart

Clears all items from a user's cart.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/carts/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /carts/:unique_id/order/marketplace - Marketplace Order

Places a marketplace order (multi-vendor).

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/order/marketplace" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cart-uuid-123",
    "type": "cart",
    "attributes": {
      "unique_id": "cart-uuid-123",
      "status": "ordered",
      "marketplace": true,
      "order_number": "ORD-2025-001235"
    }
  }
}
```

---

## Cart Details Status Transitions

Manage status transitions for individual cart detail line items.

### PUT /carts/:unique_id/details/:details_unique_id/order

Marks a cart detail as ordered.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/order" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-001",
    "type": "cart_detail",
    "attributes": {
      "status": "ordered"
    }
  }
}
```

---

### PUT /carts/:unique_id/details/:details_unique_id/accepted

Marks a cart detail as accepted.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/accepted" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "accepted"`

---

### PUT /carts/:unique_id/details/:details_unique_id/start

Marks a cart detail as started.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/start" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "started"`

---

### PUT /carts/:unique_id/details/:details_unique_id/processing

Marks a cart detail as processing.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/processing" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "processing"`

---

### PUT /carts/:unique_id/details/:details_unique_id/ready

Marks a cart detail as ready.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/ready" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "ready"`

---

### PUT /carts/:unique_id/details/:details_unique_id/ship

Marks a cart detail as shipped.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/ship" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "shipped"`

---

### PUT /carts/:unique_id/details/:details_unique_id/deliver

Marks a cart detail as delivered.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/deliver" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "delivered"`

---

### PUT /carts/:unique_id/details/:details_unique_id/cancel

Cancels a cart detail.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "cancelled"`

---

### PUT /carts/:unique_id/details/:details_unique_id/return

Marks a cart detail as returned.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/carts/cart-uuid-123/details/detail-uuid-001/return" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart detail with `"status": "returned"`

---

## My Carts (Authenticated User)

Endpoints for the currently authenticated user's carts.

### GET /mycarts/:unique_id - Get My Cart

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/mycarts/cart-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cart object (same format as GET /carts/:user_unique_id)

---

### POST /mycarts/ - Create My Cart

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mycarts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "cart": {
      "products": [
        {"product_unique_id": "product-uuid-001", "quantity": 1}
      ]
    }
  }'
```

**Response 201:** New cart object

---

### PUT /mycarts/:unique_id - Update My Cart

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mycarts/cart-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "cart": {
      "products": [
        {"product_unique_id": "product-uuid-001", "quantity": 3}
      ]
    }
  }'
```

**Response 200:** Updated cart object

---

### POST /mycarts/:unique_id/checkout - Checkout My Cart

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/mycarts/cart-uuid-123/checkout" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Checked-out cart object

---

### PUT /mycarts/:unique_id/order - Place My Order

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mycarts/cart-uuid-123/order" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Ordered cart object

---

### PUT /mycarts/:unique_id/cancel - Cancel My Cart

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/mycarts/cart-uuid-123/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Cancelled cart object

---

### DELETE /mycarts/:unique_id - Delete My Cart

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/mycarts/cart-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Visitors & Remarketing

### POST /visitors - Create Visitor

Creates a visitor session for anonymous cart tracking.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/visitors" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "visitor": {
      "session_id": "sess-abc123",
      "user_agent": "Mozilla/5.0...",
      "ip_address": "192.168.1.1"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "visitor-uuid-123",
    "type": "visitor",
    "attributes": {
      "unique_id": "visitor-uuid-123",
      "session_id": "sess-abc123",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /tools/remarketing/carts/abandoned - Abandoned Carts

Retrieves abandoned carts for remarketing campaigns.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tools/remarketing/carts/abandoned" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "abandoned_since": "2025-01-01T00:00:00Z",
      "min_total": 50.00
    }
  }'
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "cart-uuid-456",
      "type": "cart",
      "attributes": {
        "unique_id": "cart-uuid-456",
        "user_unique_id": "user-uuid-456",
        "status": "abandoned",
        "total": 89.99,
        "items_count": 2,
        "last_activity_at": "2025-01-08T14:00:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 45
  }
}
```

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
