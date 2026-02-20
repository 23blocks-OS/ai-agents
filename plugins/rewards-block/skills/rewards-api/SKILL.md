---
name: rewards-rewards-api
description: Manage 23blocks reward transactions via REST API. Use when previewing reward calculations, creating reward transactions (earning points), refunding rewards, processing point expirations, or granting manual rewards to customers.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Rewards API

Complete API reference for 23blocks reward transaction management including earning, refunding, expiring, and granting points.

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
curl -X POST "$BLOCKS_API_URL/rewards/preview" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /rewards/preview - Preview Reward Calculation

Previews how many points would be earned for a transaction without committing.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/rewards/preview" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_unique_id": "customer-uuid-123",
    "amount": 75.00,
    "loyalty_unique_id": "loyalty-uuid-1"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer_unique_id` | uuid | Yes | Customer earning points |
| `amount` | decimal | Yes | Transaction amount |
| `loyalty_unique_id` | uuid | Yes | Loyalty program to apply |

**Response 200:**
```json
{
  "data": {
    "type": "reward_preview",
    "attributes": {
      "customer_unique_id": "customer-uuid-123",
      "loyalty_unique_id": "loyalty-uuid-1",
      "amount": 75.00,
      "points_to_earn": 75,
      "rules_applied": [
        {
          "rule_unique_id": "rule-uuid-1",
          "rule_type": "money",
          "points": 75
        }
      ],
      "current_balance": 1250,
      "projected_balance": 1325
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer or loyalty program not found
- `422 Unprocessable Entity` - Invalid amount or missing parameters

---

### POST /rewards/ - Create Reward (Earn Points)

Creates a reward transaction that awards points to a customer.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/rewards" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "reward": {
      "customer_unique_id": "customer-uuid-123",
      "points": 75,
      "reward_type": "purchase",
      "reference_id": "order-12345",
      "description": "Points earned for order #12345"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer_unique_id` | uuid | Yes | Customer receiving points |
| `points` | integer | Yes | Number of points to award |
| `reward_type` | string | Yes | Type of reward (purchase, event, referral, manual) |
| `reference_id` | string | No | External reference ID (e.g., order number) |
| `description` | string | No | Description of the reward |

**Response 201:**
```json
{
  "data": {
    "id": "reward-uuid-new",
    "type": "reward",
    "attributes": {
      "unique_id": "reward-uuid-new",
      "customer_id": "customer-uuid-123",
      "points": 75,
      "reward_type": "purchase",
      "reference_id": "order-12345",
      "description": "Points earned for order #12345",
      "status": "active",
      "balance_after": 1325,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found
- `422 Unprocessable Entity` - Validation errors

---

### POST /refund/ - Refund Reward

Reverses a previously created reward transaction and deducts points.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/refund" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "reward_unique_id": "reward-uuid-123",
    "reason": "Order cancelled by customer"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reward_unique_id` | uuid | Yes | Reward transaction to refund |
| `reason` | string | No | Reason for refund |

**Response 200:**
```json
{
  "data": {
    "id": "refund-uuid-123",
    "type": "reward_refund",
    "attributes": {
      "unique_id": "refund-uuid-123",
      "reward_unique_id": "reward-uuid-123",
      "points_refunded": 75,
      "reason": "Order cancelled by customer",
      "balance_after": 1250,
      "created_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Reward not found
- `422 Unprocessable Entity` - Reward already refunded or insufficient balance

---

### PUT /customers/:unique_id/expiration - Process Expirations

Processes and expires all eligible points for a customer based on configured expiration rules.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/customers/customer-uuid-123/expiration" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "expiration_result",
    "attributes": {
      "customer_unique_id": "customer-uuid-123",
      "points_expired": 200,
      "transactions_expired": 3,
      "balance_after": 1050,
      "processed_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found

---

### PUT /customers/:unique_id/grant_reward - Grant Manual Reward

Manually grants reward points to a customer outside of normal earning rules.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/customers/customer-uuid-123/grant_reward" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "points": 500,
    "reason": "Customer service goodwill gesture"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `points` | integer | Yes | Number of points to grant |
| `reason` | string | No | Reason for manual grant |

**Response 200:**
```json
{
  "data": {
    "id": "grant-uuid-123",
    "type": "reward_grant",
    "attributes": {
      "unique_id": "grant-uuid-123",
      "customer_unique_id": "customer-uuid-123",
      "points": 500,
      "reason": "Customer service goodwill gesture",
      "balance_after": 1750,
      "granted_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found
- `422 Unprocessable Entity` - Invalid points value

---

## Data Models

### Reward
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `customer_id` | uuid | Customer ID |
| `points` | integer | Points earned |
| `reward_type` | string | purchase, event, referral, manual |
| `reference_id` | string | External reference ID |
| `description` | string | Reward description |
| `status` | enum | active, refunded, expired |
| `balance_after` | integer | Balance after transaction |
| `created_at` | timestamp | Creation time |

### RewardRefund
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `reward_unique_id` | uuid | Original reward ID |
| `points_refunded` | integer | Points deducted |
| `reason` | string | Refund reason |
| `balance_after` | integer | Balance after refund |
| `created_at` | timestamp | Refund time |

### RewardGrant
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `customer_unique_id` | uuid | Customer ID |
| `points` | integer | Points granted |
| `reason` | string | Grant reason |
| `balance_after` | integer | Balance after grant |
| `granted_at` | timestamp | Grant time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "already_refunded",
    "title": "Reward Already Refunded",
    "detail": "This reward transaction has already been refunded."
  }]
}
```
