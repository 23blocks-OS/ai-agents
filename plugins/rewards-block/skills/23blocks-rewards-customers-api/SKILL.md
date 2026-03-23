---
name: 23blocks-rewards-customers-api
description: View 23blocks reward customer profiles via REST API. Use when listing customers, checking loyalty status, viewing reward balances, tracking expirations, browsing reward history, or retrieving customer badges, coupons, and offer codes.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Customers API

Complete API reference for 23blocks reward customer profile management, loyalty status, and reward tracking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/customers` | List all reward customers |
| GET | `/customers/:unique_id/` | Get a customer's reward profile |
| GET | `/customers/:unique_id/loyalty` | Get customer loyalty status and tier |
| GET | `/customers/:unique_id/rewards` | Get customer reward balances |
| GET | `/customers/:unique_id/rewards/expirations` | Get expiring rewards |
| GET | `/customers/:unique_id/rewards/history` | Get reward transaction history |
| GET | `/customers/:unique_id/badges` | Get badges earned by customer |
| GET | `/customers/:unique_id/coupons` | Get customer coupons |
| GET | `/customers/:unique_id/offer_codes` | Get customer offer codes |

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
