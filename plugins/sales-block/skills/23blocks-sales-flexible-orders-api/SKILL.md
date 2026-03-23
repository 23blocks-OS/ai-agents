---
name: 23blocks-sales-flexible-orders-api
description: Manage 23blocks Sales flexible orders via REST API. Use when creating flexible orders with custom schemas, managing details, processing payments, handling tips, updating status and logistics, or processing cancellations and refunds.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Flexible Orders API

Complete API reference for 23blocks flexible order management. Flexible orders follow the same structure as standard orders but support custom schemas and flexible data models.

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
| GET | `/flexible_orders/` | List flexible orders with pagination |
| GET | `/flexible_orders/:unique_id` | Get a single flexible order |
| POST | `/flexible_orders/` | Create a new flexible order |
| PUT | `/flexible_orders/:unique_id` | Update a flexible order |
| POST | `/flexible_orders/:unique_id/details` | Add line item details |
| POST | `/flexible_orders/:unique_id/tips/add` | Add tip to order |
| POST | `/flexible_orders/:unique_id/payments/method` | Set payment method |
| POST | `/flexible_orders/:unique_id/payments/` | Create payment |
| PUT | `/flexible_orders/:unique_id/payments/:payment_unique_id/confirm` | Confirm payment |
| PUT | `/flexible_orders/:unique_id/status` | Update order status |
| PUT | `/flexible_orders/:unique_id/details/:details_unique_id/status` | Update detail status |
| DELETE | `/flexible_orders/:unique_id/cancel` | Cancel order |
| POST | `/flexible_orders/:unique_id/refund` | Refund order |
| PUT | `/flexible_orders/:unique_id/logistics` | Update order logistics |
| PUT | `/flexible_orders/:unique_id/details/:details_unique_id/logistics` | Update detail logistics |
| POST | `/reports/flexible_orders/summary` | Flexible order summary report |

---

## Data Models

### FlexibleOrder
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
| `custom_data` | object | Flexible custom data |
| `metadata` | object | Custom metadata |
| `logistics` | object | Shipping information |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### FlexibleOrderDetail
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `description` | string | Item description |
| `quantity` | integer | Quantity |
| `unit_price` | decimal | Unit price |
| `total` | decimal | Line total |
| `status` | enum | pending, processing, shipped, delivered, cancelled |
| `metadata` | object | Custom metadata |
| `logistics` | object | Item shipping info |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Flexible Order Not Found",
    "detail": "The requested flexible order could not be found."
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
// FlexibleOrders — client.sales.flexibleOrders
client.sales.flexibleOrders.list(params?: ListFlexibleOrdersParams): Promise<PageResult<FlexibleOrder>>;
client.sales.flexibleOrders.get(uniqueId: string): Promise<FlexibleOrder>;
client.sales.flexibleOrders.create(data: CreateFlexibleOrderRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.update(uniqueId: string, data: UpdateFlexibleOrderRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.addDetails(uniqueId: string, data: AddFlexibleOrderDetailRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.addTips(uniqueId: string, data: AddFlexibleOrderTipsRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.addPaymentMethod(uniqueId: string, data: AddFlexibleOrderPaymentMethodRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.addPayment(uniqueId: string, data: AddFlexibleOrderPaymentRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.confirmPayment(uniqueId: string, paymentUniqueId: string): Promise<FlexibleOrder>;
client.sales.flexibleOrders.updateStatus(uniqueId: string, status: string): Promise<FlexibleOrder>;
client.sales.flexibleOrders.updateDetailStatus(uniqueId: string, detailUniqueId: string, status: string): Promise<FlexibleOrder>;
client.sales.flexibleOrders.cancel(uniqueId: string): Promise<void>;
client.sales.flexibleOrders.updateLogistics(uniqueId: string, data: UpdateFlexibleOrderLogisticsRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.updateDetailLogistics(uniqueId: string, detailUniqueId: string, data: UpdateFlexibleOrderLogisticsRequest): Promise<FlexibleOrder>;
client.sales.flexibleOrders.getPayments(uniqueId: string): Promise<PageResult<Payment>>;
```

### TypeScript Types

```typescript
import type {
  FlexibleOrder,
  CreateFlexibleOrderRequest,
  UpdateFlexibleOrderRequest,
  AddFlexibleOrderDetailRequest,
  AddFlexibleOrderTipsRequest,
  AddFlexibleOrderPaymentMethodRequest,
  AddFlexibleOrderPaymentRequest,
  UpdateFlexibleOrderLogisticsRequest,
  ListFlexibleOrdersParams,
} from '@23blocks/block-sales';
```

### React Hook

```typescript
import { useSalesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSalesBlock();
  const result = await client.sales.flexibleOrders.list();
}
```
