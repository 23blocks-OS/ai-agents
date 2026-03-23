---
name: 23blocks-sales-orders-api
description: Manage 23blocks Sales orders via REST API. Use when creating orders, managing order details, processing payments, handling tips and taxes, updating order status and logistics, or processing cancellations and refunds.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Orders API

Complete API reference for 23blocks order management with details, payments, taxes, logistics, and full lifecycle tracking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://sales.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://sales.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/orders/` | List orders with pagination |
| GET | `/orders/:unique_id` | Get a single order |
| GET | `/orders/:unique_id/payments` | Get payments for an order |
| POST | `/orders/` | Create a new order |
| PUT | `/orders/:unique_id` | Update an order |
| POST | `/orders/:unique_id/details` | Add line item details |
| POST | `/orders/:unique_id/tips/add` | Add tip to an order |
| POST | `/orders/:unique_id/payments/method` | Set payment method |
| POST | `/orders/:unique_id/payments/` | Create payment |
| PUT | `/orders/:unique_id/payments/:payment_unique_id/confirm` | Confirm payment |
| PUT | `/orders/:unique_id/status` | Update order status |
| PUT | `/orders/:unique_id/details/:details_unique_id/status` | Update detail status |
| DELETE | `/orders/:unique_id/cancel` | Cancel an order |
| POST | `/orders/:unique_id/refund` | Refund an order |
| PUT | `/orders/:unique_id/logistics` | Update order logistics |
| PUT | `/orders/:unique_id/details/:details_unique_id/logistics` | Update detail logistics |
| POST | `/orders/:unique_id/taxes` | Add tax to order |
| PUT | `/orders/:unique_id/taxes/:tax_unique_id` | Update a tax |
| DELETE | `/orders/:unique_id/taxes/:tax_unique_id` | Delete a tax |
| GET | `/users/:unique_id/orders` | List orders for a user |
| GET | `/users/:unique_id/orders/:order_unique_id` | Get a user's order |

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
