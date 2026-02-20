---
name: sales-subscriptions-api
description: Manage 23blocks Sales subscriptions via REST API. Use when creating subscription models, managing user/entity/account subscriptions, tracking consumption, handling subscription lifecycle, or generating subscription reports.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Subscriptions API

Complete API reference for 23blocks subscription management including models, user/entity/account subscriptions, consumption tracking, and reporting.

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
curl -X GET "$BLOCKS_API_URL/subscription_models" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Subscription Models

### GET /subscription_models/ - List Models

Lists all subscription models.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscription_models/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "model-uuid-123",
      "type": "subscription_model",
      "attributes": {
        "unique_id": "model-uuid-123",
        "name": "Pro Plan",
        "description": "Professional tier with advanced features",
        "price": 49.99,
        "currency": "USD",
        "billing_period": "monthly",
        "trial_days": 14,
        "features": ["unlimited_projects", "priority_support", "analytics"],
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 8
  }
}
```

---

### GET /subscription_models/:unique_id - Get Model

Retrieves a specific subscription model.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscription_models/model-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "model-uuid-123",
    "type": "subscription_model",
    "attributes": {
      "unique_id": "model-uuid-123",
      "name": "Pro Plan",
      "description": "Professional tier with advanced features",
      "price": 49.99,
      "currency": "USD",
      "billing_period": "monthly",
      "trial_days": 14,
      "features": ["unlimited_projects", "priority_support", "analytics"],
      "usage_limits": {
        "api_calls": 100000,
        "storage_gb": 50
      },
      "status": "active",
      "subscribers_count": 245,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Model not found

---

### POST /subscription_models/ - Create Model

Creates a new subscription model.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subscription_models/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription_model": {
      "name": "Enterprise Plan",
      "description": "Full-featured enterprise plan",
      "price": 199.99,
      "currency": "USD",
      "billing_period": "monthly",
      "trial_days": 30,
      "features": ["unlimited_everything", "sla", "dedicated_support"]
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Plan name |
| `description` | string | No | Plan description |
| `price` | decimal | Yes | Plan price |
| `currency` | string | No | Currency code (default: USD) |
| `billing_period` | enum | Yes | monthly, yearly, weekly |
| `trial_days` | integer | No | Trial period in days |
| `features` | array | No | List of feature slugs |
| `usage_limits` | object | No | Usage limit definitions |

**Response 201:**
```json
{
  "data": {
    "id": "model-uuid-new",
    "type": "subscription_model",
    "attributes": {
      "unique_id": "model-uuid-new",
      "name": "Enterprise Plan",
      "price": 199.99,
      "currency": "USD",
      "billing_period": "monthly",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /subscription_models/:unique_id - Update Model

Updates an existing subscription model.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/subscription_models/model-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription_model": {
      "price": 59.99,
      "features": ["unlimited_projects", "priority_support", "analytics", "custom_reports"]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "model-uuid-123",
    "type": "subscription_model",
    "attributes": {
      "unique_id": "model-uuid-123",
      "name": "Pro Plan",
      "price": 59.99,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## User Subscriptions

### GET /users/:user_unique_id/subscriptions/ - List User Subscriptions

Lists all subscriptions for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/subscriptions/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "sub-uuid-001",
      "type": "subscription",
      "attributes": {
        "unique_id": "sub-uuid-001",
        "model_id": "model-uuid-123",
        "model_name": "Pro Plan",
        "status": "active",
        "current_period_start": "2025-01-01T00:00:00Z",
        "current_period_end": "2025-02-01T00:00:00Z",
        "trial_end": null,
        "cancelled_at": null,
        "created_at": "2025-01-01T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:user_unique_id/subscriptions/:subscription_unique_id/ - Get User Subscription

Retrieves a specific user subscription.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/subscriptions/sub-uuid-001/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "sub-uuid-001",
    "type": "subscription",
    "attributes": {
      "unique_id": "sub-uuid-001",
      "model_id": "model-uuid-123",
      "model_name": "Pro Plan",
      "status": "active",
      "price": 49.99,
      "currency": "USD",
      "billing_period": "monthly",
      "current_period_start": "2025-01-01T00:00:00Z",
      "current_period_end": "2025-02-01T00:00:00Z",
      "usage": {
        "api_calls": 45000,
        "storage_gb": 12
      },
      "created_at": "2025-01-01T10:30:00Z"
    }
  }
}
```

---

### POST /users/:user_unique_id/subscriptions/:subscription_unique_id/ - Create User Subscription

Creates a new subscription for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-456/subscriptions/sub-uuid-new/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "model_id": "model-uuid-123",
      "payment_method": "stripe",
      "start_date": "2025-01-15T00:00:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model_id` | uuid | Yes | Subscription model ID |
| `payment_method` | string | No | Payment method |
| `start_date` | timestamp | No | Subscription start date |

**Response 201:**
```json
{
  "data": {
    "id": "sub-uuid-new",
    "type": "subscription",
    "attributes": {
      "unique_id": "sub-uuid-new",
      "model_id": "model-uuid-123",
      "status": "active",
      "current_period_start": "2025-01-15T00:00:00Z",
      "current_period_end": "2025-02-15T00:00:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /users/:user_unique_id/subscriptions/:subscription_unique_id/ - Update User Subscription

Updates a user subscription.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-456/subscriptions/sub-uuid-001/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "model_id": "model-uuid-enterprise"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "sub-uuid-001",
    "type": "subscription",
    "attributes": {
      "model_id": "model-uuid-enterprise",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /users/:user_unique_id/subscriptions/:subscription_unique_id/consumption - Record Consumption

Records usage consumption for a subscription.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-456/subscriptions/sub-uuid-001/consumption" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "consumption": {
      "metric": "api_calls",
      "quantity": 500,
      "timestamp": "2025-01-12T10:30:00Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `metric` | string | Yes | Consumption metric name |
| `quantity` | integer | Yes | Amount consumed |
| `timestamp` | timestamp | No | Consumption time |

**Response 201:**
```json
{
  "data": {
    "id": "consumption-uuid-001",
    "type": "consumption",
    "attributes": {
      "unique_id": "consumption-uuid-001",
      "metric": "api_calls",
      "quantity": 500,
      "total_used": 45500,
      "limit": 100000,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /users/:user_unique_id/subscriptions/:subscription_unique_id/cancel - Cancel Subscription

Cancels a user subscription.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-456/subscriptions/sub-uuid-001/cancel" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "cancellation": {
      "reason": "Switching to different plan",
      "cancel_at_period_end": true
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | No | Cancellation reason |
| `cancel_at_period_end` | boolean | No | Cancel at end of billing period |

**Response 200:**
```json
{
  "data": {
    "id": "sub-uuid-001",
    "type": "subscription",
    "attributes": {
      "status": "cancelling",
      "cancel_at_period_end": true,
      "cancelled_at": "2025-01-12T15:00:00Z"
    }
  }
}
```

---

### DELETE /users/:user_unique_id/subscriptions/:subscription_unique_id/ - Delete Subscription

Deletes a user subscription immediately.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-456/subscriptions/sub-uuid-001/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Entity Subscriptions

### POST /entities/:unique_id/subscriptions/ - Create Entity Subscription

Creates a subscription for an entity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-456/subscriptions/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "model_id": "model-uuid-123",
      "payment_method": "stripe"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "sub-uuid-entity-001",
    "type": "subscription",
    "attributes": {
      "unique_id": "sub-uuid-entity-001",
      "model_id": "model-uuid-123",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /entities/:unique_id/subscriptions/:subscription_unique_id - Update Entity Subscription

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/entities/entity-uuid-456/subscriptions/sub-uuid-entity-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "model_id": "model-uuid-enterprise"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "sub-uuid-entity-001",
    "type": "subscription",
    "attributes": {
      "model_id": "model-uuid-enterprise",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Account Subscriptions

### POST /subscriptions - Create Account Subscription

Creates an account-level subscription.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subscriptions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "model_id": "model-uuid-123",
      "name": "Company Subscription"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "acct-sub-uuid-001",
    "type": "subscription",
    "attributes": {
      "unique_id": "acct-sub-uuid-001",
      "model_id": "model-uuid-123",
      "name": "Company Subscription",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /subscriptions/:subscription_unique_id - Get Account Subscription

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscriptions/acct-sub-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "acct-sub-uuid-001",
    "type": "subscription",
    "attributes": {
      "unique_id": "acct-sub-uuid-001",
      "model_id": "model-uuid-123",
      "name": "Company Subscription",
      "status": "active",
      "items_count": 3,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /subscriptions/:subscription_unique_id - Delete Account Subscription

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/subscriptions/acct-sub-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### GET /subscriptions/:subscription_unique_id/items - List Subscription Items

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscriptions/acct-sub-uuid-001/items" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "item-uuid-001",
      "type": "subscription_item",
      "attributes": {
        "unique_id": "item-uuid-001",
        "name": "Additional Seat",
        "quantity": 5,
        "unit_price": 10.00,
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /subscriptions/:subscription_unique_id/items - Add Subscription Item

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subscriptions/acct-sub-uuid-001/items" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "item": {
      "name": "Additional Seat",
      "quantity": 5,
      "unit_price": 10.00
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "item-uuid-new",
    "type": "subscription_item",
    "attributes": {
      "unique_id": "item-uuid-new",
      "name": "Additional Seat",
      "quantity": 5,
      "unit_price": 10.00,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /subscriptions/:subscription_unique_id/items/:item_unique_id - Remove Item

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/subscriptions/acct-sub-uuid-001/items/item-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Purchases

### POST /purchases - Create Purchase

Creates a one-time purchase.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/purchases" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "purchase": {
      "model_id": "model-uuid-123",
      "payment_method": "stripe",
      "amount": 199.99
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "purchase-uuid-001",
    "type": "purchase",
    "attributes": {
      "unique_id": "purchase-uuid-001",
      "model_id": "model-uuid-123",
      "amount": 199.99,
      "status": "completed",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Reports

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
      ]
    }
  }
}
```

---

## Data Models

### SubscriptionModel
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Plan name |
| `description` | string | Plan description |
| `price` | decimal | Plan price |
| `currency` | string | Currency code |
| `billing_period` | enum | monthly, yearly, weekly |
| `trial_days` | integer | Trial period in days |
| `features` | array | Feature list |
| `usage_limits` | object | Usage limit definitions |
| `status` | enum | active, inactive |
| `subscribers_count` | integer | Number of subscribers |
| `created_at` | timestamp | Creation time |

### Subscription
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `model_id` | uuid | Subscription model ID |
| `status` | enum | active, cancelling, cancelled, expired |
| `price` | decimal | Subscription price |
| `currency` | string | Currency code |
| `billing_period` | string | Billing period |
| `current_period_start` | timestamp | Current period start |
| `current_period_end` | timestamp | Current period end |
| `trial_end` | timestamp | Trial end date |
| `cancel_at_period_end` | boolean | Cancel at period end |
| `cancelled_at` | timestamp | Cancellation time |
| `usage` | object | Current usage metrics |
| `created_at` | timestamp | Creation time |

### SubscriptionItem
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Item name |
| `quantity` | integer | Item quantity |
| `unit_price` | decimal | Unit price |
| `created_at` | timestamp | Creation time |

### Consumption
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `metric` | string | Consumption metric |
| `quantity` | integer | Amount consumed |
| `total_used` | integer | Total usage so far |
| `limit` | integer | Usage limit |
| `created_at` | timestamp | Consumption time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Subscription Not Found",
    "detail": "The requested subscription could not be found."
  }]
}
```
