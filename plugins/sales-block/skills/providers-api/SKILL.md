---
name: sales-providers-api
description: Manage 23blocks Sales providers and vendor payments via REST API. Use when assigning providers to sources or order details, managing vendor payments, executing vendor payouts, or generating provider and vendor payment reports.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Providers API

Complete API reference for 23blocks provider assignment and vendor payment management.

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
curl -X POST "$BLOCKS_API_URL/sources/source-uuid/providers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Provider Assignment

### POST /sources/:source_id/providers - Assign Provider to Source

Assigns a provider to a source.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/sources/source-uuid-001/providers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": {
      "vendor_id": "vendor-uuid-001",
      "commission_rate": 15.0,
      "priority": 1
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `vendor_id` | uuid | Yes | Vendor/provider ID |
| `commission_rate` | decimal | No | Commission percentage |
| `priority` | integer | No | Provider priority |

**Response 201:**
```json
{
  "data": {
    "id": "provider-uuid-001",
    "type": "provider",
    "attributes": {
      "unique_id": "provider-uuid-001",
      "source_id": "source-uuid-001",
      "vendor_id": "vendor-uuid-001",
      "commission_rate": 15.0,
      "priority": 1,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /orders/:unique_id/details/:order_details_unique_id/providers - Assign Provider to Detail

Assigns a provider to a specific order detail.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/providers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": {
      "vendor_id": "vendor-uuid-001",
      "commission_rate": 15.0,
      "amount": 85.00
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `vendor_id` | uuid | Yes | Vendor/provider ID |
| `commission_rate` | decimal | No | Commission percentage |
| `amount` | decimal | No | Provider amount |

**Response 201:**
```json
{
  "data": {
    "id": "provider-uuid-002",
    "type": "order_provider",
    "attributes": {
      "unique_id": "provider-uuid-002",
      "order_id": "order-uuid-123",
      "detail_id": "detail-uuid-1",
      "vendor_id": "vendor-uuid-001",
      "commission_rate": 15.0,
      "amount": 85.00,
      "status": "assigned",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id/details/:order_details_unique_id/providers/:provider_unique_id - Update Provider

Updates a provider assignment on an order detail.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/providers/provider-uuid-002" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": {
      "commission_rate": 12.0,
      "amount": 88.00,
      "status": "confirmed"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "provider-uuid-002",
    "type": "order_provider",
    "attributes": {
      "unique_id": "provider-uuid-002",
      "commission_rate": 12.0,
      "amount": 88.00,
      "status": "confirmed",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Vendor Payments

### POST /orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments - Create Vendor Payment

Creates a payment record for a vendor.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/vendors/vendor-uuid-001/payments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "amount": 85.00,
      "currency": "USD",
      "payment_method": "transfer",
      "notes": "Provider payment for order detail"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `amount` | decimal | Yes | Payment amount |
| `currency` | string | No | Currency code |
| `payment_method` | string | No | Payment method |
| `notes` | string | No | Payment notes |

**Response 201:**
```json
{
  "data": {
    "id": "vendor-payment-uuid-001",
    "type": "vendor_payment",
    "attributes": {
      "unique_id": "vendor-payment-uuid-001",
      "order_id": "order-uuid-123",
      "detail_id": "detail-uuid-1",
      "vendor_id": "vendor-uuid-001",
      "amount": 85.00,
      "currency": "USD",
      "status": "pending",
      "payment_method": "transfer",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments/:payment_unique_id - Update Vendor Payment

Updates a vendor payment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/vendors/vendor-uuid-001/payments/vendor-payment-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "amount": 90.00,
      "notes": "Adjusted amount"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "vendor-payment-uuid-001",
    "type": "vendor_payment",
    "attributes": {
      "unique_id": "vendor-payment-uuid-001",
      "amount": 90.00,
      "notes": "Adjusted amount",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments/:payment_unique_id/pay - Execute Vendor Payment

Executes/pays out a vendor payment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/vendors/vendor-uuid-001/payments/vendor-payment-uuid-001/pay" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "reference": "TXN-20250112-001"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reference` | string | No | External payment reference |

**Response 200:**
```json
{
  "data": {
    "id": "vendor-payment-uuid-001",
    "type": "vendor_payment",
    "attributes": {
      "unique_id": "vendor-payment-uuid-001",
      "status": "paid",
      "reference": "TXN-20250112-001",
      "paid_at": "2025-01-12T15:00:00Z"
    }
  }
}
```

---

### DELETE /orders/:unique_id/details/:detail_unique_id/vendors/:vendor_unique_id/payments/:payment_unique_id - Delete Vendor Payment

Deletes a vendor payment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/orders/order-uuid-123/details/detail-uuid-1/vendors/vendor-uuid-001/payments/vendor-payment-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Payables

### GET /payables/:payment_unique_id - Get Payable

Retrieves a payable record.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/payables/vendor-payment-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "vendor-payment-uuid-001",
    "type": "payable",
    "attributes": {
      "unique_id": "vendor-payment-uuid-001",
      "vendor_id": "vendor-uuid-001",
      "vendor_name": "Vendor Corp",
      "order_id": "order-uuid-123",
      "detail_id": "detail-uuid-1",
      "amount": 85.00,
      "currency": "USD",
      "status": "paid",
      "payment_method": "transfer",
      "reference": "TXN-20250112-001",
      "paid_at": "2025-01-12T15:00:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Payable not found

---

## Reports

### POST /reports/vendors/payments/list - Vendor Payment List Report

Generates a detailed list of vendor payments.

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
      "vendor_id": "vendor-uuid-001",
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

### POST /reports/vendors/payments/summary - Vendor Payment Summary Report

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
      "end_date": "2025-01-31"
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
      "total_records": 90
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

## Data Models

### Provider
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `source_id` | uuid | Source ID |
| `vendor_id` | uuid | Vendor ID |
| `commission_rate` | decimal | Commission percentage |
| `priority` | integer | Provider priority |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

### OrderProvider
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `order_id` | uuid | Order ID |
| `detail_id` | uuid | Order detail ID |
| `vendor_id` | uuid | Vendor ID |
| `commission_rate` | decimal | Commission percentage |
| `amount` | decimal | Provider amount |
| `status` | enum | assigned, confirmed, completed |
| `created_at` | timestamp | Creation time |

### VendorPayment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `order_id` | uuid | Order ID |
| `detail_id` | uuid | Order detail ID |
| `vendor_id` | uuid | Vendor ID |
| `amount` | decimal | Payment amount |
| `currency` | string | Currency code |
| `status` | enum | pending, paid, cancelled |
| `payment_method` | string | Payment method |
| `reference` | string | External payment reference |
| `notes` | string | Payment notes |
| `paid_at` | timestamp | Payment execution time |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Provider Not Found",
    "detail": "The requested provider could not be found."
  }]
}
```
