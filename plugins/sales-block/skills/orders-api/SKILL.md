---
name: sales-orders-api
description: Manage 23blocks Sales orders via REST API. Use when creating orders, managing order details, processing payments, handling tips and taxes, updating order status and logistics, or processing cancellations and refunds.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Orders API

Complete API reference for 23blocks order management with details, payments, taxes, logistics, and full lifecycle tracking.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://sales.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/orders" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Order CRUD

### GET /orders/ - List Orders

Lists all orders with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/orders/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `status` | string | No | Filter by status |

**Response 200:**
```json
{
  "data": [
    {
      "id": "order-uuid-123",
      "type": "order",
      "attributes": {
        "unique_id": "order-uuid-123",
        "user_id": "user-uuid-456",
        "total": 109.99,
        "subtotal": 99.99,
        "tax": 10.00,
        "tip": 0.00,
        "currency": "USD",
        "status": "pending",
        "notes": "Customer order",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 150
  }
}
```

---

### GET /orders/:unique_id - Get Order

Retrieves a single order by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/orders/order-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-123",
      "user_id": "user-uuid-456",
      "total": 109.99,
      "subtotal": 99.99,
      "tax": 10.00,
      "tip": 0.00,
      "currency": "USD",
      "status": "pending",
      "notes": "Customer order",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "details": {
        "data": [
          { "id": "detail-uuid-1", "type": "order_detail" }
        ]
      },
      "payments": {
        "data": []
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Order not found

---

### GET /orders/:unique_id/payments - Get Order Payments

Retrieves all payments for an order.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/orders/order-uuid-123/payments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "payment-uuid-001",
      "type": "payment",
      "attributes": {
        "unique_id": "payment-uuid-001",
        "amount": 109.99,
        "currency": "USD",
        "status": "confirmed",
        "payment_method": "stripe",
        "created_at": "2025-01-10T10:35:00Z"
      }
    }
  ]
}
```

---

### POST /orders/ - Create Order

Creates a new order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "order": {
      "currency": "USD",
      "notes": "New customer order",
      "metadata": {
        "source": "web",
        "campaign": "summer-sale"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `currency` | string | No | Currency code (default: USD) |
| `notes` | string | No | Order notes |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "order-uuid-new",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-new",
      "total": 0.00,
      "subtotal": 0.00,
      "tax": 0.00,
      "currency": "USD",
      "status": "pending",
      "notes": "New customer order",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id - Update Order

Updates an existing order.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "order": {
      "notes": "Updated order notes",
      "metadata": {
        "priority": "high"
      }
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-123",
      "notes": "Updated order notes",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Order Details

### POST /orders/:unique_id/details - Add Order Details

Adds line item details to an order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/details" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "detail": {
      "description": "Premium Plan - Annual",
      "quantity": 1,
      "unit_price": 99.99,
      "sku": "PLAN-PREMIUM-ANNUAL",
      "metadata": {
        "plan_type": "annual"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `description` | string | Yes | Item description |
| `quantity` | integer | Yes | Quantity |
| `unit_price` | decimal | Yes | Unit price |
| `sku` | string | No | Product SKU |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "detail-uuid-new",
    "type": "order_detail",
    "attributes": {
      "unique_id": "detail-uuid-new",
      "description": "Premium Plan - Annual",
      "quantity": 1,
      "unit_price": 99.99,
      "total": 99.99,
      "sku": "PLAN-PREMIUM-ANNUAL",
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Tips

### POST /orders/:unique_id/tips/add - Add Tip

Adds a tip/gratuity to an order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/tips/add" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tip": {
      "amount": 15.00
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | decimal | Yes | Tip amount |

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-123",
      "tip": 15.00,
      "total": 124.99,
      "updated_at": "2025-01-12T10:35:00Z"
    }
  }
}
```

---

## Payments

### POST /orders/:unique_id/payments/method - Set Payment Method

Sets the payment method for an order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/payments/method" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_method": {
      "method": "stripe",
      "token": "tok_visa"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `method` | string | Yes | Payment method (stripe, mercadopago, cash, transfer) |
| `token` | string | No | Payment token (for card payments) |

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "payment_method": "stripe",
      "updated_at": "2025-01-12T10:36:00Z"
    }
  }
}
```

---

### POST /orders/:unique_id/payments/ - Create Payment

Creates a payment for the order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/payments/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "amount": 124.99,
      "currency": "USD"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | decimal | Yes | Payment amount |
| `currency` | string | No | Currency code |

**Response 201:**
```json
{
  "data": {
    "id": "payment-uuid-new",
    "type": "payment",
    "attributes": {
      "unique_id": "payment-uuid-new",
      "amount": 124.99,
      "currency": "USD",
      "status": "pending",
      "created_at": "2025-01-12T10:37:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id/payments/:payment_unique_id/confirm - Confirm Payment

Confirms a pending payment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/payments/payment-uuid-001/confirm" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "payment-uuid-001",
    "type": "payment",
    "attributes": {
      "unique_id": "payment-uuid-001",
      "status": "confirmed",
      "confirmed_at": "2025-01-12T10:38:00Z"
    }
  }
}
```

---

## Status Management

### PUT /orders/:unique_id/status - Update Order Status

Updates the order status.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "order": {
      "status": "confirmed"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | enum | Yes | pending, confirmed, processing, shipped, delivered, cancelled |

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-123",
      "status": "confirmed",
      "updated_at": "2025-01-12T11:00:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id/details/:details_unique_id/status - Update Detail Status

Updates the status of a specific order detail.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "detail": {
      "status": "processing"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-1",
    "type": "order_detail",
    "attributes": {
      "unique_id": "detail-uuid-1",
      "status": "processing",
      "updated_at": "2025-01-12T11:05:00Z"
    }
  }
}
```

---

## Cancel & Refund

### DELETE /orders/:unique_id/cancel - Cancel Order

Cancels an order.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/orders/order-uuid-123/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-123",
      "status": "cancelled",
      "cancelled_at": "2025-01-12T12:00:00Z"
    }
  }
}
```

---

### POST /orders/:unique_id/refund - Refund Order

Processes a refund for an order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/refund" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "refund": {
      "amount": 124.99,
      "reason": "Customer request"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | decimal | No | Refund amount (full refund if omitted) |
| `reason` | string | No | Refund reason |

**Response 200:**
```json
{
  "data": {
    "id": "refund-uuid-001",
    "type": "refund",
    "attributes": {
      "unique_id": "refund-uuid-001",
      "order_id": "order-uuid-123",
      "amount": 124.99,
      "reason": "Customer request",
      "status": "processed",
      "created_at": "2025-01-12T12:05:00Z"
    }
  }
}
```

---

## Logistics

### PUT /orders/:unique_id/logistics - Update Order Logistics

Updates shipping/logistics information for the order.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/logistics" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "logistics": {
      "carrier": "FedEx",
      "tracking_number": "FX123456789",
      "shipping_address": "123 Main St, City, ST 12345",
      "estimated_delivery": "2025-01-15T10:00:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `carrier` | string | No | Shipping carrier |
| `tracking_number` | string | No | Tracking number |
| `shipping_address` | string | No | Delivery address |
| `estimated_delivery` | timestamp | No | Estimated delivery date |

**Response 200:**
```json
{
  "data": {
    "id": "order-uuid-123",
    "type": "order",
    "attributes": {
      "unique_id": "order-uuid-123",
      "logistics": {
        "carrier": "FedEx",
        "tracking_number": "FX123456789",
        "shipping_address": "123 Main St, City, ST 12345",
        "estimated_delivery": "2025-01-15T10:00:00Z"
      },
      "updated_at": "2025-01-12T13:00:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id/details/:details_unique_id/logistics - Update Detail Logistics

Updates logistics for a specific order detail.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/logistics" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "logistics": {
      "carrier": "UPS",
      "tracking_number": "1Z999AA10123456784"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-1",
    "type": "order_detail",
    "attributes": {
      "unique_id": "detail-uuid-1",
      "logistics": {
        "carrier": "UPS",
        "tracking_number": "1Z999AA10123456784"
      },
      "updated_at": "2025-01-12T13:05:00Z"
    }
  }
}
```

---

## Taxes

### POST /orders/:unique_id/taxes - Add Tax

Adds a tax to the order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/taxes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tax": {
      "name": "Sales Tax",
      "rate": 10.0,
      "amount": 10.00
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Tax name |
| `rate` | decimal | No | Tax rate percentage |
| `amount` | decimal | Yes | Tax amount |

**Response 201:**
```json
{
  "data": {
    "id": "tax-uuid-001",
    "type": "order_tax",
    "attributes": {
      "unique_id": "tax-uuid-001",
      "name": "Sales Tax",
      "rate": 10.0,
      "amount": 10.00,
      "created_at": "2025-01-12T10:31:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id/taxes/:tax_unique_id - Update Tax

Updates an existing tax on the order.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/taxes/tax-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tax": {
      "rate": 12.0,
      "amount": 12.00
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tax-uuid-001",
    "type": "order_tax",
    "attributes": {
      "unique_id": "tax-uuid-001",
      "name": "Sales Tax",
      "rate": 12.0,
      "amount": 12.00,
      "updated_at": "2025-01-12T10:32:00Z"
    }
  }
}
```

---

### DELETE /orders/:unique_id/taxes/:tax_unique_id - Delete Tax

Removes a tax from the order.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/orders/order-uuid-123/taxes/tax-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## User Orders

### GET /users/:unique_id/orders - List User Orders

Lists all orders for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/orders?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "order-uuid-123",
      "type": "order",
      "attributes": {
        "unique_id": "order-uuid-123",
        "total": 109.99,
        "status": "delivered",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 12
  }
}
```

---

### GET /users/:unique_id/orders/:order_unique_id - Get User Order

Retrieves a specific order for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/orders/order-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as GET /orders/:unique_id

---

## Data Models

### Order
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_id` | uuid | Associated user ID |
| `total` | decimal | Order total |
| `subtotal` | decimal | Subtotal before tax/tip |
| `tax` | decimal | Tax amount |
| `tip` | decimal | Tip amount |
| `currency` | string | Currency code |
| `status` | enum | pending, confirmed, processing, shipped, delivered, cancelled |
| `notes` | string | Order notes |
| `metadata` | object | Custom metadata |
| `logistics` | object | Shipping information |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### OrderDetail
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `description` | string | Item description |
| `quantity` | integer | Quantity |
| `unit_price` | decimal | Unit price |
| `total` | decimal | Line total |
| `sku` | string | Product SKU |
| `status` | enum | pending, processing, shipped, delivered, cancelled |
| `metadata` | object | Custom metadata |
| `logistics` | object | Item shipping info |
| `created_at` | timestamp | Creation time |

### Payment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `amount` | decimal | Payment amount |
| `currency` | string | Currency code |
| `status` | enum | pending, confirmed, failed, refunded |
| `payment_method` | string | Payment method used |
| `confirmed_at` | timestamp | Confirmation time |
| `created_at` | timestamp | Creation time |

### OrderTax
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Tax name |
| `rate` | decimal | Tax rate percentage |
| `amount` | decimal | Tax amount |
| `created_at` | timestamp | Creation time |

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
npm install @23blocks/block-sales
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
// Orders — client.sales.orders
client.sales.orders.list(params?: ListOrdersParams): Promise<PageResult<Order>>;
client.sales.orders.get(uniqueId: string): Promise<Order>;
client.sales.orders.create(data: CreateOrderRequest): Promise<Order>;
client.sales.orders.update(uniqueId: string, data: UpdateOrderRequest): Promise<Order>;
client.sales.orders.updateStatus(uniqueId: string, data: UpdateOrderStatusRequest): Promise<Order>;
client.sales.orders.updateLogistics(uniqueId: string, data: UpdateOrderLogisticsRequest): Promise<Order>;
client.sales.orders.addTips(uniqueId: string, data: AddOrderTipsRequest): Promise<Order>;
client.sales.orders.listByCustomer(customerUniqueId: string, params?: ListOrdersParams): Promise<PageResult<Order>>;
```

### TypeScript Types

```typescript
import type {
  Order,
  CreateOrderRequest,
  UpdateOrderRequest,
  UpdateOrderStatusRequest,
  UpdateOrderLogisticsRequest,
  AddOrderTipsRequest,
  ListOrdersParams,
} from '@23blocks/block-sales';
```

### React Hook

```typescript
import { useSalesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSalesBlock();
  const result = await client.sales.orders.list();
}
```
