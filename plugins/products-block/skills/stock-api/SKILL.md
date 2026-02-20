---
name: products-stock-api
description: Manage 23blocks product stock and inventory via REST API. Use when creating or updating stock entries, managing vendor warehouse stock, updating variation stock levels, searching stock, or evaluating stock availability.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Stock API

Complete API reference for 23blocks product stock and inventory management.

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
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/stock" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Product Stock Endpoints

### GET /products/:product_unique_id/stock - Get Stock

Retrieves stock entries for a product.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/products/product-uuid-123/stock" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "stock-uuid-123",
      "type": "stock",
      "attributes": {
        "unique_id": "stock-uuid-123",
        "product_unique_id": "product-uuid-123",
        "warehouse_unique_id": "warehouse-uuid-001",
        "quantity": 150,
        "reserved": 10,
        "available": 140,
        "min_quantity": 20,
        "max_quantity": 500,
        "status": "in_stock",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /products/:product_unique_id/stock - Create Stock Entry

Creates a new stock entry for a product.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/products/product-uuid-123/stock" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "stock": {
      "warehouse_unique_id": "warehouse-uuid-001",
      "quantity": 100,
      "min_quantity": 10,
      "max_quantity": 500
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `warehouse_unique_id` | uuid | Yes | Warehouse identifier |
| `quantity` | integer | Yes | Stock quantity |
| `min_quantity` | integer | No | Minimum stock threshold |
| `max_quantity` | integer | No | Maximum stock capacity |

**Response 201:**
```json
{
  "data": {
    "id": "new-stock-uuid",
    "type": "stock",
    "attributes": {
      "unique_id": "new-stock-uuid",
      "product_unique_id": "product-uuid-123",
      "warehouse_unique_id": "warehouse-uuid-001",
      "quantity": 100,
      "available": 100,
      "status": "in_stock",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /products/:product_unique_id/stock/:stock_unique_id - Update Stock

Updates a stock entry.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/products/product-uuid-123/stock/stock-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "stock": {
      "quantity": 200,
      "min_quantity": 25
    }
  }'
```

**Response 200:** Updated stock object

---

## Vendor Warehouse Stock

### PUT /vendors/:vendor_unique_id/warehouses/:warehouse_unique_id/products/:product_unique_id/ - Update Vendor Stock

Updates stock for a specific product in a vendor's warehouse.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/vendors/vendor-uuid-001/warehouses/warehouse-uuid-001/products/product-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "stock": {
      "quantity": 75,
      "cost": 25.00
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `quantity` | integer | Yes | Stock quantity |
| `cost` | decimal | No | Vendor cost price |

**Response 200:**
```json
{
  "data": {
    "id": "vendor-stock-uuid",
    "type": "vendor_stock",
    "attributes": {
      "vendor_unique_id": "vendor-uuid-001",
      "warehouse_unique_id": "warehouse-uuid-001",
      "product_unique_id": "product-uuid-123",
      "quantity": 75,
      "cost": 25.00,
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /vendors/:vendor_unique_id/warehouses/:warehouse_unique_id/products/:product_unique_id/variations/:variation_unique_id - Update Variation Stock

Updates stock for a product variation in a vendor's warehouse.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/vendors/vendor-uuid-001/warehouses/warehouse-uuid-001/products/product-uuid-123/variations/variation-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "stock": {
      "quantity": 30,
      "cost": 27.50
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "variation-stock-uuid",
    "type": "variation_stock",
    "attributes": {
      "variation_unique_id": "variation-uuid-001",
      "quantity": 30,
      "cost": 27.50,
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Stock Search & Evaluation

### POST /stocks/search - Search Stock

Searches stock across products and warehouses.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stocks/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
      "warehouse_unique_id": "warehouse-uuid-001",
      "status": "low_stock",
      "min_quantity_below": true
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `warehouse_unique_id` | uuid | No | Filter by warehouse |
| `status` | enum | No | in_stock, low_stock, out_of_stock |
| `min_quantity_below` | boolean | No | Stock below minimum threshold |

**Response 200:**
```json
{
  "data": [
    {
      "id": "stock-uuid-456",
      "type": "stock",
      "attributes": {
        "unique_id": "stock-uuid-456",
        "product_unique_id": "product-uuid-789",
        "warehouse_unique_id": "warehouse-uuid-001",
        "quantity": 5,
        "min_quantity": 20,
        "status": "low_stock"
      }
    }
  ],
  "meta": {
    "total_count": 12
  }
}
```

---

### POST /stocks/:unique_id/eval - Evaluate Stock

Evaluates stock availability and provides a stock assessment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/stocks/stock-uuid-123/eval" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "eval": {
      "requested_quantity": 50
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "stock_evaluation",
    "attributes": {
      "stock_unique_id": "stock-uuid-123",
      "available": 140,
      "requested": 50,
      "fulfillable": true,
      "remaining_after": 90,
      "status": "sufficient"
    }
  }
}
```

---

## Data Models

### Stock
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `product_unique_id` | uuid | Product identifier |
| `warehouse_unique_id` | uuid | Warehouse identifier |
| `quantity` | integer | Total quantity |
| `reserved` | integer | Reserved quantity |
| `available` | integer | Available quantity |
| `min_quantity` | integer | Minimum stock threshold |
| `max_quantity` | integer | Maximum stock capacity |
| `status` | enum | in_stock, low_stock, out_of_stock |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### VendorStock
| Field | Type | Description |
|-------|------|-------------|
| `vendor_unique_id` | uuid | Vendor identifier |
| `warehouse_unique_id` | uuid | Warehouse identifier |
| `product_unique_id` | uuid | Product identifier |
| `quantity` | integer | Stock quantity |
| `cost` | decimal | Vendor cost price |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Quantity must be greater than zero."
  }]
}
```
