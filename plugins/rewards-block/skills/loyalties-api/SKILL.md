---
name: rewards-loyalties-api
description: Manage 23blocks loyalty programs via REST API. Use when creating loyalty programs, configuring money/product/event earning rules, managing rule expirations, adding program details, and viewing program stats.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Loyalties API

Complete API reference for 23blocks loyalty program management with money, product, and event earning rules.

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
curl -X GET "$BLOCKS_API_URL/loyalties" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /loyalties/ - List Loyalty Programs

Lists all loyalty programs with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/loyalties?page=1&records=20" \
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
      "id": "loyalty-uuid-123",
      "type": "loyalty",
      "attributes": {
        "unique_id": "loyalty-uuid-123",
        "name": "Gold Rewards",
        "description": "Earn points on every purchase",
        "points_name": "Stars",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### GET /loyalties/:unique_id - Get Loyalty Program

Retrieves a single loyalty program by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/loyalties/loyalty-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "loyalty-uuid-123",
    "type": "loyalty",
    "attributes": {
      "unique_id": "loyalty-uuid-123",
      "name": "Gold Rewards",
      "description": "Earn points on every purchase",
      "points_name": "Stars",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "rules": {
        "data": [
          { "id": "rule-uuid-1", "type": "rule" }
        ]
      },
      "details": {
        "data": []
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Loyalty program not found

---

### POST /loyalties/ - Create Loyalty Program

Creates a new loyalty program.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/loyalties" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "loyalty": {
      "name": "Gold Rewards",
      "description": "Earn points on every purchase",
      "points_name": "Stars",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Program name |
| `description` | string | No | Program description |
| `points_name` | string | No | Display name for points (e.g., Stars, Coins) |
| `status` | enum | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "new-loyalty-uuid",
    "type": "loyalty",
    "attributes": {
      "unique_id": "new-loyalty-uuid",
      "name": "Gold Rewards",
      "description": "Earn points on every purchase",
      "points_name": "Stars",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /loyalties/:unique_id - Update Loyalty Program

Updates an existing loyalty program.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "loyalty": {
      "name": "Platinum Rewards",
      "description": "Updated rewards program"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "loyalty-uuid-123",
    "type": "loyalty",
    "attributes": {
      "unique_id": "loyalty-uuid-123",
      "name": "Platinum Rewards",
      "description": "Updated rewards program",
      "points_name": "Stars",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Loyalty program not found

---

### GET /loyalties/:unique_id/stats - Get Program Stats

Retrieves performance statistics for a loyalty program.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/stats" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "loyalty-uuid-123",
    "type": "loyalty_stats",
    "attributes": {
      "total_members": 1250,
      "total_points_issued": 450000,
      "total_points_redeemed": 125000,
      "total_points_expired": 30000,
      "active_rules": 5,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Loyalty program not found

---

### POST /loyalties/:unique_id/details - Add Detail

Adds supplementary detail information to a loyalty program.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/details" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "detail": {
      "key": "terms_url",
      "value": "https://example.com/terms"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `key` | string | Yes | Detail key name |
| `value` | string | Yes | Detail value |

**Response 201:**
```json
{
  "data": {
    "id": "detail-uuid-123",
    "type": "loyalty_detail",
    "attributes": {
      "unique_id": "detail-uuid-123",
      "key": "terms_url",
      "value": "https://example.com/terms",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /loyalties/:unique_id/details/:details_unique_id - Update Detail

Updates an existing loyalty program detail.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/details/detail-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "detail": {
      "value": "https://example.com/updated-terms"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-123",
    "type": "loyalty_detail",
    "attributes": {
      "unique_id": "detail-uuid-123",
      "key": "terms_url",
      "value": "https://example.com/updated-terms",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Rules - Money

### GET /loyalties/:unique_id/rules/money - List Money Rules

Lists all money-based earning rules for a loyalty program.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/money" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "rule-uuid-1",
      "type": "rule",
      "attributes": {
        "unique_id": "rule-uuid-1",
        "rule_type": "money",
        "points_per_unit": 1,
        "min_amount": 5.00,
        "max_points": 500,
        "enabled": true,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /loyalties/:unique_id/rules/money - Create Money Rule

Creates a money-based earning rule.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/money" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "points_per_unit": 1,
      "min_amount": 5.00,
      "max_points": 500
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `points_per_unit` | integer | Yes | Points earned per currency unit |
| `min_amount` | decimal | No | Minimum transaction amount to qualify |
| `max_points` | integer | No | Maximum points per transaction |

**Response 201:**
```json
{
  "data": {
    "id": "new-rule-uuid",
    "type": "rule",
    "attributes": {
      "unique_id": "new-rule-uuid",
      "rule_type": "money",
      "points_per_unit": 1,
      "min_amount": 5.00,
      "max_points": 500,
      "enabled": true,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /loyalties/:unique_id/rules/money/:rule_unique_id - Update Money Rule

Updates an existing money rule.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/money/rule-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "points_per_unit": 2,
      "max_points": 1000
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "rule-uuid-1",
    "type": "rule",
    "attributes": {
      "unique_id": "rule-uuid-1",
      "rule_type": "money",
      "points_per_unit": 2,
      "min_amount": 5.00,
      "max_points": 1000,
      "enabled": true,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /loyalties/:unique_id/rules/money/:rule_unique_id/expirations/ - Set Rule Expiration

Attaches an expiration rule to a money earning rule.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/money/rule-uuid-1/expirations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "expiration_rule_unique_id": "expiration-uuid-1"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `expiration_rule_unique_id` | uuid | Yes | Expiration rule to attach |

**Response 200:**
```json
{
  "data": {
    "id": "rule-uuid-1",
    "type": "rule",
    "attributes": {
      "unique_id": "rule-uuid-1",
      "rule_type": "money",
      "expiration_rule_unique_id": "expiration-uuid-1",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /loyalties/:unique_id/rules/money/:rule_unique_id/expirations/:expiration_rule_unique_id - Remove Rule Expiration

Removes an expiration rule from a money earning rule.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/money/rule-uuid-1/expirations/expiration-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Rules - Products

### GET /loyalties/:unique_id/rules/products - List Product Rules

Lists all product-based earning rules for a loyalty program.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/products" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "rule-uuid-2",
      "type": "rule",
      "attributes": {
        "unique_id": "rule-uuid-2",
        "rule_type": "products",
        "points_per_unit": 10,
        "min_amount": 1,
        "max_points": 100,
        "enabled": true,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /loyalties/:unique_id/rules/products - Create Product Rule

Creates a product-based earning rule.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/products" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "points_per_unit": 10,
      "min_amount": 1,
      "max_points": 100
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `points_per_unit` | integer | Yes | Points earned per product unit |
| `min_amount` | integer | No | Minimum quantity to qualify |
| `max_points` | integer | No | Maximum points per transaction |

**Response 201:**
```json
{
  "data": {
    "id": "new-rule-uuid",
    "type": "rule",
    "attributes": {
      "unique_id": "new-rule-uuid",
      "rule_type": "products",
      "points_per_unit": 10,
      "min_amount": 1,
      "max_points": 100,
      "enabled": true,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /loyalties/:unique_id/rules/products/:rule_unique_id - Update Product Rule

Updates an existing product rule.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/products/rule-uuid-2" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "points_per_unit": 15,
      "max_points": 200
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "rule-uuid-2",
    "type": "rule",
    "attributes": {
      "unique_id": "rule-uuid-2",
      "rule_type": "products",
      "points_per_unit": 15,
      "min_amount": 1,
      "max_points": 200,
      "enabled": true,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /loyalties/:unique_id/rules/products/:rule_unique_id - Delete Product Rule

Deletes a product earning rule.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/products/rule-uuid-2" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Rule not found

---

## Rules - Events

### GET /loyalties/:unique_id/rules/events - List Event Rules

Lists all event-based earning rules for a loyalty program.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "rule-uuid-3",
      "type": "rule",
      "attributes": {
        "unique_id": "rule-uuid-3",
        "rule_type": "events",
        "points_per_unit": 25,
        "min_amount": null,
        "max_points": 25,
        "enabled": true,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /loyalties/:unique_id/rules/events - Create Event Rule

Creates an event-based earning rule (e.g., sign-ups, referrals, reviews).

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "points_per_unit": 25,
      "max_points": 25
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `points_per_unit` | integer | Yes | Points earned per event occurrence |
| `max_points` | integer | No | Maximum points per event |

**Response 201:**
```json
{
  "data": {
    "id": "new-rule-uuid",
    "type": "rule",
    "attributes": {
      "unique_id": "new-rule-uuid",
      "rule_type": "events",
      "points_per_unit": 25,
      "max_points": 25,
      "enabled": true,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /loyalties/:unique_id/rules/events/:rule_unique_id - Update Event Rule

Updates an existing event rule.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/events/rule-uuid-3" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "points_per_unit": 50,
      "max_points": 50
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "rule-uuid-3",
    "type": "rule",
    "attributes": {
      "unique_id": "rule-uuid-3",
      "rule_type": "events",
      "points_per_unit": 50,
      "max_points": 50,
      "enabled": true,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Rule Management

### PUT /loyalties/:unique_id/rules/:rule_unique_id/disable - Disable Rule

Disables an earning rule without deleting it.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/rule-uuid-1/disable" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "rule-uuid-1",
    "type": "rule",
    "attributes": {
      "unique_id": "rule-uuid-1",
      "enabled": false,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /loyalties/:unique_id/rules/:rule_unique_id/enable - Enable Rule

Re-enables a previously disabled earning rule.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/loyalties/loyalty-uuid-123/rules/rule-uuid-1/enable" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "rule-uuid-1",
    "type": "rule",
    "attributes": {
      "unique_id": "rule-uuid-1",
      "enabled": true,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Data Models

### Loyalty
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Program name |
| `description` | string | Program description |
| `points_name` | string | Display name for points |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Rule
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `rule_type` | enum | money, products, events |
| `points_per_unit` | integer | Points earned per unit |
| `min_amount` | decimal | Minimum amount to qualify |
| `max_points` | integer | Maximum points per transaction |
| `enabled` | boolean | Whether the rule is active |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### LoyaltyDetail
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `key` | string | Detail key name |
| `value` | string | Detail value |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Loyalty Program Not Found",
    "detail": "The requested loyalty program could not be found."
  }]
}
```
