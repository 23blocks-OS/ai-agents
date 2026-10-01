---
name: 23blocks-sales-orders-api
description: "Sales Block orders: details, payments, tips, taxes, status, logistics, cancel, refund. Use for the order lifecycle."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Orders API

Complete API reference for 23blocks order management with details, payments, taxes, logistics, and full lifecycle tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://sales.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Order Not Found","detail":"The requested order could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-sales`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useSalesBlock()` from `@23blocks/react`.

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
