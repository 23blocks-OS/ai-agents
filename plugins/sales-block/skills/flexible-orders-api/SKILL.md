---
name: sales-flexible-orders-api
description: Manage 23blocks Sales flexible orders via REST API. Use when creating flexible orders with custom schemas, managing details, processing payments, handling tips, updating status and logistics, or processing cancellations and refunds.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Flexible Orders API

Complete API reference for 23blocks flexible order management. Flexible orders follow the same structure as standard orders but support custom schemas and flexible data models.

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
curl -X GET "$BLOCKS_API_URL/flexible_orders" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Flexible Order CRUD

### GET /flexible_orders/ - List Flexible Orders

Lists all flexible orders with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/flexible_orders/?page=1&records=20" \
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
      "id": "flex-order-uuid-123",
      "type": "flexible_order",
      "attributes": {
        "unique_id": "flex-order-uuid-123",
        "user_id": "user-uuid-456",
        "total": 250.00,
        "subtotal": 230.00,
        "tax": 20.00,
        "tip": 0.00,
        "currency": "USD",
        "status": "pending",
        "custom_data": {},
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 75
  }
}
```

---

### GET /flexible_orders/:unique_id - Get Flexible Order

Retrieves a single flexible order by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "flex-order-uuid-123",
    "type": "flexible_order",
    "attributes": {
      "unique_id": "flex-order-uuid-123",
      "user_id": "user-uuid-456",
      "total": 250.00,
      "subtotal": 230.00,
      "tax": 20.00,
      "tip": 0.00,
      "currency": "USD",
      "status": "pending",
      "custom_data": {
        "service_type": "consulting",
        "hours": 5
      },
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "details": {
        "data": [
          { "id": "detail-uuid-1", "type": "flexible_order_detail" }
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
- `404 Not Found` - Flexible order not found

---

### POST /flexible_orders/ - Create Flexible Order

Creates a new flexible order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/flexible_orders/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "flexible_order": {
      "currency": "USD",
      "notes": "Custom service order",
      "custom_data": {
        "service_type": "consulting",
        "hours": 5
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `currency` | string | No | Currency code (default: USD) |
| `notes` | string | No | Order notes |
| `custom_data` | object | No | Flexible custom data |
| `metadata` | object | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "flex-order-uuid-new",
    "type": "flexible_order",
    "attributes": {
      "unique_id": "flex-order-uuid-new",
      "total": 0.00,
      "subtotal": 0.00,
      "currency": "USD",
      "status": "pending",
      "custom_data": {
        "service_type": "consulting",
        "hours": 5
      },
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /flexible_orders/:unique_id - Update Flexible Order

Updates an existing flexible order.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "flexible_order": {
      "notes": "Updated notes",
      "custom_data": {
        "service_type": "consulting",
        "hours": 8
      }
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "flex-order-uuid-123",
    "type": "flexible_order",
    "attributes": {
      "unique_id": "flex-order-uuid-123",
      "notes": "Updated notes",
      "custom_data": {
        "service_type": "consulting",
        "hours": 8
      },
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Order Details

### POST /flexible_orders/:unique_id/details - Add Details

Adds line item details to a flexible order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/details" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "detail": {
      "description": "Consulting - 5 hours",
      "quantity": 5,
      "unit_price": 46.00
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "detail-uuid-new",
    "type": "flexible_order_detail",
    "attributes": {
      "unique_id": "detail-uuid-new",
      "description": "Consulting - 5 hours",
      "quantity": 5,
      "unit_price": 46.00,
      "total": 230.00,
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Tips

### POST /flexible_orders/:unique_id/tips/add - Add Tip

Adds a tip to a flexible order.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/tips/add" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tip": {
      "amount": 25.00
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "flex-order-uuid-123",
    "type": "flexible_order",
    "attributes": {
      "tip": 25.00,
      "total": 275.00,
      "updated_at": "2025-01-12T10:35:00Z"
    }
  }
}
```

---

## Payments

### POST /flexible_orders/:unique_id/payments/method - Set Payment Method

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/payments/method" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_method": {
      "method": "transfer",
      "reference": "TXN-20250112-001"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "flex-order-uuid-123",
    "type": "flexible_order",
    "attributes": {
      "payment_method": "transfer",
      "updated_at": "2025-01-12T10:36:00Z"
    }
  }
}
```

---

### POST /flexible_orders/:unique_id/payments/ - Create Payment

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/payments/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "payment": {
      "amount": 275.00,
      "currency": "USD"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "payment-uuid-new",
    "type": "payment",
    "attributes": {
      "unique_id": "payment-uuid-new",
      "amount": 275.00,
      "currency": "USD",
      "status": "pending",
      "created_at": "2025-01-12T10:37:00Z"
    }
  }
}
```

---

### PUT /flexible_orders/:unique_id/payments/:payment_unique_id/confirm - Confirm Payment

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/payments/payment-uuid-001/confirm" \
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
      "status": "confirmed",
      "confirmed_at": "2025-01-12T10:38:00Z"
    }
  }
}
```

---

## Status Management

### PUT /flexible_orders/:unique_id/status - Update Status

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/status" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "flexible_order": {
      "status": "confirmed"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "flex-order-uuid-123",
    "type": "flexible_order",
    "attributes": {
      "status": "confirmed",
      "updated_at": "2025-01-12T11:00:00Z"
    }
  }
}
```

---

### PUT /flexible_orders/:unique_id/details/:details_unique_id/status - Update Detail Status

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/details/detail-uuid-1/status" \
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
    "type": "flexible_order_detail",
    "attributes": {
      "status": "processing",
      "updated_at": "2025-01-12T11:05:00Z"
    }
  }
}
```

---

## Cancel & Refund

### DELETE /flexible_orders/:unique_id/cancel - Cancel

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "flex-order-uuid-123",
    "type": "flexible_order",
    "attributes": {
      "status": "cancelled",
      "cancelled_at": "2025-01-12T12:00:00Z"
    }
  }
}
```

---

### POST /flexible_orders/:unique_id/refund - Refund

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/refund" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "refund": {
      "amount": 275.00,
      "reason": "Service not completed"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "refund-uuid-001",
    "type": "refund",
    "attributes": {
      "unique_id": "refund-uuid-001",
      "amount": 275.00,
      "reason": "Service not completed",
      "status": "processed",
      "created_at": "2025-01-12T12:05:00Z"
    }
  }
}
```

---

## Logistics

### PUT /flexible_orders/:unique_id/logistics - Update Logistics

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/logistics" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "logistics": {
      "carrier": "DHL",
      "tracking_number": "DHL123456789",
      "estimated_delivery": "2025-01-18T10:00:00Z"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "flex-order-uuid-123",
    "type": "flexible_order",
    "attributes": {
      "logistics": {
        "carrier": "DHL",
        "tracking_number": "DHL123456789",
        "estimated_delivery": "2025-01-18T10:00:00Z"
      },
      "updated_at": "2025-01-12T13:00:00Z"
    }
  }
}
```

---

### PUT /flexible_orders/:unique_id/details/:details_unique_id/logistics - Update Detail Logistics

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/flexible_orders/flex-order-uuid-123/details/detail-uuid-1/logistics" \
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
    "type": "flexible_order_detail",
    "attributes": {
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

## Reports

### POST /reports/flexible_orders/summary - Flexible Order Summary Report

Generates a summary report for flexible orders.

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
      "total_orders": 75,
      "total_revenue": 18750.00,
      "average_order_value": 250.00,
      "currency": "USD",
      "period": {
        "start_date": "2025-01-01",
        "end_date": "2025-01-31"
      },
      "breakdown": [
        {
          "date": "2025-01-01",
          "orders": 3,
          "revenue": 750.00
        }
      ]
    }
  }
}
```

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
