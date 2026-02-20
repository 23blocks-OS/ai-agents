---
name: sales-reports-api
description: Generate 23blocks Sales reports via REST API. Use when generating order summaries, payment reports, subscription analytics, vendor payment reports, provider reports, or flexible order summaries.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Reports API

Complete API reference for 23blocks sales reporting including orders, payments, subscriptions, vendors, and providers.

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
curl -X POST "$BLOCKS_API_URL/reports/orders/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Common Report Parameters

All report endpoints accept a common set of parameters:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | date | No | Report start date (YYYY-MM-DD) |
| `end_date` | date | No | Report end date (YYYY-MM-DD) |
| `group_by` | enum | No | day, week, month |
| `status` | string | No | Filter by status |
| `currency` | string | No | Filter by currency |

---

## Order Reports

### POST /reports/orders/summary - Order Summary Report

Generates a summary report of orders.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/orders/summary" \
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

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "total_orders": 150,
      "total_revenue": 18750.00,
      "average_order_value": 125.00,
      "total_tax": 1875.00,
      "total_tips": 450.00,
      "currency": "USD",
      "period": {
        "start_date": "2025-01-01",
        "end_date": "2025-01-31"
      },
      "by_status": {
        "pending": 12,
        "confirmed": 8,
        "processing": 15,
        "shipped": 20,
        "delivered": 90,
        "cancelled": 5
      },
      "breakdown": [
        {
          "date": "2025-01-01",
          "orders": 5,
          "revenue": 625.00
        }
      ]
    }
  }
}
```

---

### POST /reports/orders/list - Order List Report

Generates a detailed list report of orders.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/orders/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "status": "delivered"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "records": [
        {
          "order_id": "order-uuid-123",
          "user_id": "user-uuid-456",
          "total": 109.99,
          "subtotal": 99.99,
          "tax": 10.00,
          "status": "delivered",
          "payment_method": "stripe",
          "created_at": "2025-01-10T10:30:00Z",
          "delivered_at": "2025-01-14T14:00:00Z"
        }
      ],
      "total_records": 90,
      "total_revenue": 11250.00
    }
  }
}
```

---

## Provider Reports

### POST /reports/orders/providers/list - Provider List Report

Generates a list of provider assignments for orders.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/orders/providers/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "vendor_id": "vendor-uuid-001"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "records": [
        {
          "provider_id": "provider-uuid-002",
          "vendor_id": "vendor-uuid-001",
          "vendor_name": "Vendor Corp",
          "order_id": "order-uuid-123",
          "detail_id": "detail-uuid-1",
          "commission_rate": 15.0,
          "amount": 85.00,
          "status": "confirmed"
        }
      ],
      "total_records": 45
    }
  }
}
```

---

### POST /reports/orders/providers/summary - Provider Summary Report

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/orders/providers/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "group_by": "vendor"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "total_assignments": 90,
      "total_provider_amount": 7650.00,
      "total_commission": 1350.00,
      "currency": "USD",
      "by_vendor": [
        {
          "vendor_id": "vendor-uuid-001",
          "vendor_name": "Vendor Corp",
          "assignments": 45,
          "total_amount": 3825.00,
          "avg_commission_rate": 15.0
        }
      ]
    }
  }
}
```

---

## Flexible Order Reports

### POST /reports/flexible_orders/summary - Flexible Order Summary Report

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/flexible_orders/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "group_by": "week"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "total_orders": 75,
      "total_revenue": 18750.00,
      "average_order_value": 250.00,
      "currency": "USD",
      "period": {
        "start_date": "2025-01-01",
        "end_date": "2025-01-31"
      },
      "by_status": {
        "pending": 5,
        "confirmed": 10,
        "processing": 8,
        "delivered": 48,
        "cancelled": 4
      },
      "breakdown": [
        {
          "week": "2025-W01",
          "orders": 18,
          "revenue": 4500.00
        }
      ]
    }
  }
}
```

---

## Payment Reports

### POST /reports/payments/list - Payment List Report

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
      "payment_method": "stripe"
    }
  }'
```

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
      "total_records": 120,
      "total_amount": 12500.00
    }
  }
}
```

---

### POST /reports/payments/summary - Payment Summary Report

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
      "group_by": "month"
    }
  }'
```

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
      "by_method": {
        "stripe": { "count": 120, "amount": 12500.00 },
        "mercadopago": { "count": 25, "amount": 2750.50 },
        "transfer": { "count": 5, "amount": 500.00 }
      }
    }
  }
}
```

---

## Vendor Payment Reports

### POST /reports/vendors/payments/list - Vendor Payment List

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/vendors/payments/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "status": "paid"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "records": [
        {
          "payment_id": "vendor-payment-uuid-001",
          "vendor_id": "vendor-uuid-001",
          "vendor_name": "Vendor Corp",
          "order_id": "order-uuid-123",
          "amount": 85.00,
          "status": "paid",
          "paid_at": "2025-01-12T15:00:00Z"
        }
      ],
      "total_records": 45,
      "total_amount": 3825.00
    }
  }
}
```

---

### POST /reports/vendors/payments/summary - Vendor Payment Summary

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/vendors/payments/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "group_by": "vendor"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "total_payments": 120,
      "total_amount": 10500.00,
      "paid_amount": 8500.00,
      "pending_amount": 2000.00,
      "currency": "USD",
      "by_vendor": [
        {
          "vendor_id": "vendor-uuid-001",
          "vendor_name": "Vendor Corp",
          "payments": 45,
          "total_amount": 3825.00,
          "paid_amount": 3200.00
        }
      ]
    }
  }
}
```

---

## Subscription Reports

### POST /reports/users/subscriptions/list - Subscription List Report

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/users/subscriptions/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "status": "active"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "records": [
        {
          "user_id": "user-uuid-456",
          "subscription_id": "sub-uuid-001",
          "model_name": "Pro Plan",
          "status": "active",
          "price": 49.99,
          "started_at": "2025-01-01T10:30:00Z"
        }
      ],
      "total_records": 245
    }
  }
}
```

---

### POST /reports/users/subscriptions/summary - Subscription Summary Report

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/users/subscriptions/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-01-31",
      "group_by": "model"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "report",
    "attributes": {
      "total_subscriptions": 245,
      "active_subscriptions": 230,
      "cancelled_subscriptions": 15,
      "new_subscriptions": 32,
      "churned_subscriptions": 8,
      "mrr": 11270.70,
      "currency": "USD",
      "by_model": [
        {
          "model_name": "Pro Plan",
          "subscribers": 180,
          "revenue": 8998.20
        },
        {
          "model_name": "Enterprise Plan",
          "subscribers": 50,
          "revenue": 2272.50
        }
      ],
      "churn_rate": 3.3
    }
  }
}
```

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "invalid_parameters",
    "title": "Invalid Report Parameters",
    "detail": "start_date must be before end_date."
  }]
}
```
