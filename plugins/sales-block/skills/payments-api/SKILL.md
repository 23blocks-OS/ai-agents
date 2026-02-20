---
name: sales-payments-api
description: Manage 23blocks Sales payments via REST API. Use when viewing user payment history, retrieving payment details, or generating payment reports.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Payments API

Complete API reference for 23blocks payment tracking and reporting.

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
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/payments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /users/:unique_id/payments - List User Payments

Lists all payments for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/payments?page=1&records=20" \
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
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY" \
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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Payment Not Found",
    "detail": "The requested payment could not be found."
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

### React Hook

```typescript
import { useSalesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSalesBlock();
  const result = await client.sales.payments.list();
}
```
