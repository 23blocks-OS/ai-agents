---
name: 23blocks-rewards-customers-api
description: "Rewards Block customer view: loyalty status, balances, expirations, history, badges, coupons, offer codes. Use to look up a customer's rewards."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Customers API

Complete API reference for 23blocks reward customer profile management, loyalty status, and reward tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://rewards.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

> **Note:** Block identity records are notification routing caches, not identity models. The canonical user record lives in the Auth (Gateway) block. `email`/`phone` here are optional denormalized routing fields; duplicates across users are allowed. Customer registration in this block is keyed by `user_unique_id` only.

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Customer Not Found","detail":"The requested customer could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-rewards`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useRewardsBlock()` from `@23blocks/react`.

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
