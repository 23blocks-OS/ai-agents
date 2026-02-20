---
name: rewards-customers-api
description: View 23blocks reward customer profiles via REST API. Use when listing customers, checking loyalty status, viewing reward balances, tracking expirations, browsing reward history, or retrieving customer badges, coupons, and offer codes.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Customers API

Complete API reference for 23blocks reward customer profile management, loyalty status, and reward tracking.

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
curl -X GET "$BLOCKS_API_URL/customers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /customers - List Customers

Lists all reward customers with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers?page=1&records=20" \
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
      "id": "customer-uuid-123",
      "type": "customer",
      "attributes": {
        "unique_id": "customer-uuid-123",
        "name": "Jane Smith",
        "email": "jane@example.com",
        "total_points": 1250,
        "tier": "gold",
        "lifetime_points": 5400,
        "joined_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 148
  }
}
```

---

### GET /customers/:unique_id/ - Get Customer

Retrieves a single customer reward profile.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "customer-uuid-123",
    "type": "customer",
    "attributes": {
      "unique_id": "customer-uuid-123",
      "name": "Jane Smith",
      "email": "jane@example.com",
      "total_points": 1250,
      "tier": "gold",
      "lifetime_points": 5400,
      "joined_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found

---

### GET /customers/:unique_id/loyalty - Customer Loyalty Status

Retrieves the customer's current loyalty program status and tier information.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/loyalty" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "customer-uuid-123",
    "type": "customer_loyalty",
    "attributes": {
      "customer_unique_id": "customer-uuid-123",
      "loyalty_unique_id": "loyalty-uuid-1",
      "loyalty_name": "Gold Rewards",
      "current_tier": "gold",
      "total_points": 1250,
      "points_to_next_tier": 750,
      "next_tier": "platinum",
      "tier_expiry": "2026-12-31T23:59:59Z",
      "enrolled_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found or not enrolled in loyalty program

---

### GET /customers/:unique_id/rewards - Customer Rewards

Retrieves the customer's current reward balances.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/rewards?page=1&records=20" \
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
      "id": "reward-uuid-1",
      "type": "reward",
      "attributes": {
        "unique_id": "reward-uuid-1",
        "customer_id": "customer-uuid-123",
        "points": 75,
        "reward_type": "purchase",
        "reference_id": "order-12345",
        "description": "Points earned for order #12345",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
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

### GET /customers/:unique_id/rewards/expirations - Expiring Rewards

Retrieves the customer's rewards that are approaching expiration.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/rewards/expirations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "reward-uuid-2",
      "type": "reward_expiration",
      "attributes": {
        "unique_id": "reward-uuid-2",
        "points": 200,
        "expires_at": "2026-03-15T23:59:59Z",
        "days_remaining": 24,
        "reward_type": "purchase",
        "description": "Points from holiday purchases"
      }
    }
  ]
}
```

---

### GET /customers/:unique_id/rewards/history - Reward History

Retrieves the customer's complete reward transaction history.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/rewards/history?page=1&records=20" \
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
      "id": "history-uuid-1",
      "type": "reward_history",
      "attributes": {
        "unique_id": "history-uuid-1",
        "action": "earn",
        "points": 75,
        "balance_after": 1250,
        "reward_type": "purchase",
        "reference_id": "order-12345",
        "description": "Points earned for order #12345",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "history-uuid-2",
      "type": "reward_history",
      "attributes": {
        "unique_id": "history-uuid-2",
        "action": "redeem",
        "points": -500,
        "balance_after": 1175,
        "reward_type": "redemption",
        "reference_id": "redemption-456",
        "description": "Points redeemed for gift card",
        "created_at": "2025-01-08T14:00:00Z"
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

### GET /customers/:unique_id/badges - Customer Badges

Retrieves all badges earned by the customer.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/badges" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "award-uuid-1",
      "type": "badge_award",
      "attributes": {
        "unique_id": "award-uuid-1",
        "badge_unique_id": "badge-uuid-123",
        "badge_name": "First Purchase",
        "badge_description": "Awarded after first purchase",
        "badge_image_url": "https://cdn.example.com/badges/first-purchase.png",
        "points_awarded": 50,
        "awarded_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /customers/:unique_id/coupons - Customer Coupons

Retrieves all coupons assigned to or redeemed by the customer.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/coupons" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "coupon-uuid-1",
      "type": "customer_coupon",
      "attributes": {
        "unique_id": "coupon-uuid-1",
        "code": "SUMMER-ABC123",
        "discount_type": "percentage",
        "discount_value": 20,
        "status": "redeemed",
        "redeemed_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /customers/:unique_id/offer_codes - Customer Offer Codes

Retrieves all offer codes used by the customer.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-123/offer_codes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "offer-uuid-1",
      "type": "customer_offer_code",
      "attributes": {
        "unique_id": "offer-uuid-1",
        "code": "WELCOME50",
        "offer_type": "discount",
        "discount_value": 50,
        "status": "redeemed",
        "redeemed_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

## Data Models

### Customer
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Customer name |
| `email` | string | Customer email |
| `total_points` | integer | Current point balance |
| `tier` | string | Current loyalty tier |
| `lifetime_points` | integer | Total points ever earned |
| `joined_at` | timestamp | Date joined rewards program |

### CustomerLoyalty
| Field | Type | Description |
|-------|------|-------------|
| `customer_unique_id` | uuid | Customer ID |
| `loyalty_unique_id` | uuid | Loyalty program ID |
| `loyalty_name` | string | Loyalty program name |
| `current_tier` | string | Current tier level |
| `total_points` | integer | Current point balance |
| `points_to_next_tier` | integer | Points needed for next tier |
| `next_tier` | string | Next tier name |
| `tier_expiry` | timestamp | When current tier expires |
| `enrolled_at` | timestamp | Enrollment date |

### RewardHistory
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `action` | enum | earn, redeem, expire, refund, grant |
| `points` | integer | Points added or deducted |
| `balance_after` | integer | Balance after transaction |
| `reward_type` | string | Type of reward transaction |
| `reference_id` | string | External reference ID |
| `description` | string | Transaction description |
| `created_at` | timestamp | Transaction time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Customer Not Found",
    "detail": "The requested customer could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-rewards
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
// RewardsCustomersService — client.rewards.rewardsCustomers
client.rewards.rewardsCustomers.list(params?: ListRewardsCustomersParams): Promise<PageResult<RewardsCustomer>>;
client.rewards.rewardsCustomers.get(uniqueId: string): Promise<RewardsCustomer>;
client.rewards.rewardsCustomers.getLoyaltyTier(uniqueId: string): Promise<Loyalty>;
client.rewards.rewardsCustomers.getRewards(uniqueId: string): Promise<unknown>;
client.rewards.rewardsCustomers.getRewardExpirations(uniqueId: string): Promise<CustomerRewardExpiration[]>;
client.rewards.rewardsCustomers.getRewardHistory(uniqueId: string): Promise<CustomerRewardHistory[]>;
client.rewards.rewardsCustomers.getBadges(uniqueId: string): Promise<Badge[]>;
client.rewards.rewardsCustomers.getCoupons(uniqueId: string): Promise<Coupon[]>;
client.rewards.rewardsCustomers.getOfferCodes(uniqueId: string): Promise<OfferCode[]>;
client.rewards.rewardsCustomers.grantReward(uniqueId: string, points: number, reason?: string): Promise<RewardsCustomer>;
client.rewards.rewardsCustomers.updateExpiration(uniqueId: string, expirationDate: Date): Promise<RewardsCustomer>;
```

### TypeScript Types

```typescript
import type {
  RewardsCustomer,
  CustomerRewardExpiration,
  CustomerRewardHistory,
  ListRewardsCustomersParams,
} from '@23blocks/block-rewards';
```

### React Hook

```typescript
import { useRewardsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useRewardsBlock();
  const result = await client.rewards.rewardsCustomers.list();
}
```
