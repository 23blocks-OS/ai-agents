---
name: 23blocks-sales-payments-api
description: "Sales Block payments: a user's history and details, payment reports. Use when looking up payments."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Payments API

Complete API reference for 23blocks payment tracking and reporting.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://sales.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /users/:unique_id/payments - List User Payments

Lists all payments for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/payments?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "id": "payment-uuid-001",
      "type": "payment",
      "attributes": {
        "unique_id": "payment-uuid-001",
        "order_id": "order-uuid-123",
        "amount": 109.99,
        "currency": "USD",
        "status": "confirmed",
        "payment_method": "stripe",
        "stripe_payment_id": "pi_3N1234567890",
        "confirmed_at": "2025-01-10T10:35:00Z",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "payment-uuid-002",
      "type": "payment",
      "attributes": {
        "unique_id": "payment-uuid-002",
        "order_id": "order-uuid-456",
        "amount": 49.99,
        "currency": "USD",
        "status": "confirmed",
        "payment_method": "mercadopago",
        "confirmed_at": "2025-01-11T14:20:00Z",
        "created_at": "2025-01-11T14:15:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 68
  }
}
```

---

### GET /users/:unique_id/payments/:payment_unique_id - Get Payment

Retrieves a specific payment for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/payments/payment-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "payment-uuid-001",
    "type": "payment",
    "attributes": {
      "unique_id": "payment-uuid-001",
      "order_id": "order-uuid-123",
      "amount": 109.99,
      "currency": "USD",
      "status": "confirmed",
      "payment_method": "stripe",
      "stripe_payment_id": "pi_3N1234567890",
      "transaction_id": "txn_1234567890",
      "confirmed_at": "2025-01-10T10:35:00Z",
      "metadata": {
        "source": "checkout",
        "ip_address": "192.168.1.1"
      },
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "order": {
        "data": { "id": "order-uuid-123", "type": "order" }
      },
      "user": {
        "data": { "id": "user-uuid-456", "type": "user" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Payment not found

---

## Reports

### POST /reports/payments/list - Payment List Report

Generates a detailed list report of payments.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/payments/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "status": "confirmed",
      "payment_method": "stripe"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | date | No | Report start date |
| `end_date` | date | No | Report end date |
| `status` | string | No | Filter by payment status |
| `payment_method` | string | No | Filter by payment method |

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "records": [
        {
          "payment_id": "payment-uuid-001",
          "order_id": "order-uuid-123",
          "user_id": "user-uuid-456",
          "amount": 109.99,
          "currency": "USD",
          "status": "confirmed",
          "payment_method": "stripe",
          "created_at": "2025-01-10T10:30:00Z"
        }
      ],
      "total_records": 150,
      "total_amount": 15750.50
    }
  }
}
```

---

### POST /reports/payments/summary - Payment Summary Report

Generates a summary report of payments.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/payments/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "group_by": "day"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | date | No | Report start date |
| `end_date` | date | No | Report end date |
| `group_by` | enum | No | day, week, month |

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "total_payments": 150,
      "total_amount": 15750.50,
      "confirmed_amount": 15200.00,
      "pending_amount": 400.50,
      "failed_amount": 150.00,
      "currency": "USD",
      "period": {
        "start_date": "2025-01-01",
        "end_date": "2025-01-31"
      },
      "by_method": {
        "stripe": { "count": 120, "amount": 12500.00 },
        "mercadopago": { "count": 25, "amount": 2750.50 },
        "transfer": { "count": 5, "amount": 500.00 }
      },
      "breakdown": [
        {
          "date": "2025-01-01",
          "payments": 5,
          "amount": 520.00
        }
      ]
    }
  }
}
```

---

## Data Models

### Payment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `order_id` | uuid | Associated order ID |
| `amount` | decimal | Payment amount |
| `currency` | string | Currency code |
| `status` | enum | pending, confirmed, failed, refunded |
| `payment_method` | string | Payment method (stripe, mercadopago, cash, transfer) |
| `stripe_payment_id` | string | Stripe payment intent ID |
| `transaction_id` | string | External transaction ID |
| `confirmed_at` | timestamp | Confirmation time |
| `metadata` | object | Custom metadata |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Payment Not Found","detail":"The requested payment could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-sales`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useSalesBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// Payments — client.sales.payments
client.sales.payments.list(params?: ListPaymentsParams): Promise<PageResult<Payment>>;
client.sales.payments.get(uniqueId: string): Promise<Payment>;
client.sales.payments.create(orderUniqueId: string, data: CreatePaymentRequest): Promise<Payment>;
client.sales.payments.createPaymentMethod(orderUniqueId: string, data: CreatePaymentMethodRequest): Promise<Payment>;
client.sales.payments.listByOrder(orderUniqueId: string): Promise<Payment[]>;
```

### TypeScript Types

```typescript
import type {
  Payment,
  CreatePaymentRequest,
  CreatePaymentMethodRequest,
  ListPaymentsParams,
} from '@23blocks/block-sales';
```
