---
name: rewards-coupons-api
description: Manage 23blocks coupons via REST API. Use when creating coupon configurations, generating single or batch coupons, loading coupon codes, previewing and redeeming coupons, voiding coupons or batches.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Coupons API

Complete API reference for 23blocks coupon management with configuration templates, batch generation, and redemption.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://rewards.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/configurations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Configurations

### GET /configurations/ - List Coupon Configurations

Lists all coupon configuration templates.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/configurations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "config-uuid-123",
      "type": "configuration",
      "attributes": {
        "unique_id": "config-uuid-123",
        "name": "Summer Sale 20%",
        "discount_type": "percentage",
        "discount_value": 20,
        "max_uses": 100,
        "valid_from": "2026-06-01T00:00:00Z",
        "valid_to": "2026-08-31T23:59:59Z",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### GET /configurations/:unique_id - Get Configuration

Retrieves a single coupon configuration.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/configurations/config-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "config-uuid-123",
    "type": "configuration",
    "attributes": {
      "unique_id": "config-uuid-123",
      "name": "Summer Sale 20%",
      "discount_type": "percentage",
      "discount_value": 20,
      "max_uses": 100,
      "valid_from": "2026-06-01T00:00:00Z",
      "valid_to": "2026-08-31T23:59:59Z",
      "status": "active",
      "total_coupons": 50,
      "total_redeemed": 12,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Configuration not found

---

### GET /configurations/:unique_id/coupons - Get Configuration Coupons

Lists all coupons generated from a specific configuration.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/configurations/config-uuid-123/coupons?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "coupon-uuid-1",
      "type": "coupon",
      "attributes": {
        "unique_id": "coupon-uuid-1",
        "code": "SUMMER-ABC123",
        "configuration_id": "config-uuid-123",
        "used_count": 0,
        "max_uses": 1,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 50
  }
}
```

---

### POST /configurations/ - Create Configuration

Creates a new coupon configuration template.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/configurations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "configuration": {
      "name": "Summer Sale 20%",
      "discount_type": "percentage",
      "discount_value": 20,
      "max_uses": 100,
      "valid_from": "2026-06-01T00:00:00Z",
      "valid_to": "2026-08-31T23:59:59Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Configuration name |
| `discount_type` | enum | Yes | percentage, fixed |
| `discount_value` | decimal | Yes | Discount amount or percentage |
| `max_uses` | integer | No | Maximum total uses across all coupons |
| `valid_from` | timestamp | No | Start of validity period |
| `valid_to` | timestamp | No | End of validity period |

**Response 201:**
```json
{
  "data": {
    "id": "new-config-uuid",
    "type": "configuration",
    "attributes": {
      "unique_id": "new-config-uuid",
      "name": "Summer Sale 20%",
      "discount_type": "percentage",
      "discount_value": 20,
      "max_uses": 100,
      "valid_from": "2026-06-01T00:00:00Z",
      "valid_to": "2026-08-31T23:59:59Z",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /configurations/ - Update Configuration

Updates an existing coupon configuration.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/configurations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "configuration": {
      "unique_id": "config-uuid-123",
      "name": "Extended Summer Sale 25%",
      "discount_value": 25,
      "valid_to": "2026-09-30T23:59:59Z"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "config-uuid-123",
    "type": "configuration",
    "attributes": {
      "unique_id": "config-uuid-123",
      "name": "Extended Summer Sale 25%",
      "discount_type": "percentage",
      "discount_value": 25,
      "valid_to": "2026-09-30T23:59:59Z",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /configurations/:unique_id - Delete Configuration

Deletes a coupon configuration.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/configurations/config-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Configuration not found

---

### POST /configurations/:unique_id/one - Generate Single Coupon

Generates a single coupon from a configuration.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/configurations/config-uuid-123/one" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "coupon": {
      "max_uses": 1
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `max_uses` | integer | No | Max uses for this coupon (default: 1) |

**Response 201:**
```json
{
  "data": {
    "id": "coupon-uuid-new",
    "type": "coupon",
    "attributes": {
      "unique_id": "coupon-uuid-new",
      "code": "SUMMER-XYZ789",
      "configuration_id": "config-uuid-123",
      "used_count": 0,
      "max_uses": 1,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /configurations/:unique_id/batch - Generate Batch

Generates multiple coupons from a configuration at once.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/configurations/config-uuid-123/batch" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 50,
    "max_uses": 1
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `quantity` | integer | Yes | Number of coupons to generate |
| `max_uses` | integer | No | Max uses per coupon (default: 1) |

**Response 201:**
```json
{
  "data": {
    "id": "batch-uuid-123",
    "type": "coupon_batch",
    "attributes": {
      "unique_id": "batch-uuid-123",
      "configuration_id": "config-uuid-123",
      "quantity": 50,
      "status": "completed",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /configurations/:unique_id/batches/:batch_id/void - Void Batch

Voids all coupons in a batch.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/configurations/config-uuid-123/batches/batch-uuid-123/void" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "batch-uuid-123",
    "type": "coupon_batch",
    "attributes": {
      "unique_id": "batch-uuid-123",
      "status": "voided",
      "voided_count": 38,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /configurations/:unique_id/load - Load Coupons

Loads pre-existing coupon codes into a configuration.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/configurations/config-uuid-123/load" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "codes": ["PROMO-001", "PROMO-002", "PROMO-003"]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `codes` | array | Yes | Array of coupon code strings |

**Response 201:**
```json
{
  "data": {
    "loaded_count": 3,
    "configuration_id": "config-uuid-123",
    "created_at": "2025-01-12T10:30:00Z"
  }
}
```

---

## Coupons

### GET /coupons - List Coupons

Lists all coupons with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/coupons?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "coupon-uuid-1",
      "type": "coupon",
      "attributes": {
        "unique_id": "coupon-uuid-1",
        "code": "SUMMER-ABC123",
        "configuration_id": "config-uuid-123",
        "used_count": 0,
        "max_uses": 1,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /coupons/:unique_id - Get Coupon

Retrieves a single coupon by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/coupons/coupon-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "coupon-uuid-1",
    "type": "coupon",
    "attributes": {
      "unique_id": "coupon-uuid-1",
      "code": "SUMMER-ABC123",
      "configuration_id": "config-uuid-123",
      "used_count": 0,
      "max_uses": 1,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "configuration": {
        "data": { "id": "config-uuid-123", "type": "configuration" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Coupon not found

---

### POST /coupons/ - Create Coupon

Creates a standalone coupon.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/coupons" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "coupon": {
      "code": "VIP-SPECIAL-100",
      "configuration_id": "config-uuid-123",
      "max_uses": 5
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Unique coupon code |
| `configuration_id` | uuid | Yes | Parent configuration ID |
| `max_uses` | integer | No | Maximum uses (default: 1) |

**Response 201:**
```json
{
  "data": {
    "id": "new-coupon-uuid",
    "type": "coupon",
    "attributes": {
      "unique_id": "new-coupon-uuid",
      "code": "VIP-SPECIAL-100",
      "configuration_id": "config-uuid-123",
      "used_count": 0,
      "max_uses": 5,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Coupon code already exists
- `422 Unprocessable Entity` - Validation errors

---

### PUT /coupons/:unique_id - Update Coupon

Updates an existing coupon.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coupons/coupon-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "coupon": {
      "max_uses": 10
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "coupon-uuid-1",
    "type": "coupon",
    "attributes": {
      "unique_id": "coupon-uuid-1",
      "code": "SUMMER-ABC123",
      "max_uses": 10,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /coupons/:unique_id/void - Void Coupon

Voids a coupon so it can no longer be redeemed.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/coupons/coupon-uuid-1/void" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "coupon-uuid-1",
    "type": "coupon",
    "attributes": {
      "unique_id": "coupon-uuid-1",
      "status": "voided",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /coupons/:unique_id - Delete Coupon

Deletes a coupon.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/coupons/coupon-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Coupon not found

---

### POST /coupon/preview - Preview Redemption

Previews a coupon redemption without committing the transaction.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/coupon/preview" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "SUMMER-ABC123",
    "customer_unique_id": "customer-uuid",
    "order_total": 150.00
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Coupon code to preview |
| `customer_unique_id` | uuid | Yes | Customer applying the coupon |
| `order_total` | decimal | No | Order total for discount calculation |

**Response 200:**
```json
{
  "data": {
    "type": "coupon_preview",
    "attributes": {
      "code": "SUMMER-ABC123",
      "valid": true,
      "discount_type": "percentage",
      "discount_value": 20,
      "discount_amount": 30.00,
      "final_total": 120.00,
      "remaining_uses": 1
    }
  }
}
```

**Errors:**
- `404 Not Found` - Coupon code not found
- `422 Unprocessable Entity` - Coupon expired, voided, or max uses reached

---

### POST /coupon/redeem - Redeem Coupon

Redeems a coupon for a customer.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/coupon/redeem" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "SUMMER-ABC123",
    "customer_unique_id": "customer-uuid",
    "order_total": 150.00
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Coupon code to redeem |
| `customer_unique_id` | uuid | Yes | Customer redeeming the coupon |
| `order_total` | decimal | No | Order total for discount calculation |

**Response 200:**
```json
{
  "data": {
    "type": "coupon_redemption",
    "attributes": {
      "code": "SUMMER-ABC123",
      "discount_amount": 30.00,
      "final_total": 120.00,
      "redeemed_at": "2025-01-12T10:30:00Z",
      "remaining_uses": 0
    }
  }
}
```

**Errors:**
- `404 Not Found` - Coupon code not found
- `422 Unprocessable Entity` - Coupon expired, voided, or max uses reached

---

## Data Models

### Configuration
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Configuration name |
| `discount_type` | enum | percentage, fixed |
| `discount_value` | decimal | Discount amount or percentage |
| `max_uses` | integer | Maximum total uses |
| `valid_from` | timestamp | Start of validity |
| `valid_to` | timestamp | End of validity |
| `status` | enum | active, inactive |
| `total_coupons` | integer | Total coupons generated |
| `total_redeemed` | integer | Total coupons redeemed |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Coupon
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `code` | string | Unique coupon code |
| `configuration_id` | uuid | Parent configuration ID |
| `used_count` | integer | Number of times used |
| `max_uses` | integer | Maximum allowed uses |
| `status` | enum | active, redeemed, voided, expired |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### CouponBatch
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `configuration_id` | uuid | Parent configuration ID |
| `quantity` | integer | Number of coupons in batch |
| `status` | enum | completed, voided |
| `voided_count` | integer | Number of voided coupons |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "coupon_expired",
    "title": "Coupon Expired",
    "detail": "The coupon code has expired and can no longer be redeemed."
  }]
}
```
