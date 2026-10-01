---
name: 23blocks-rewards-loyalties-api
description: "Rewards Block loyalty programs: money, product and event earning rules, details, stats. Use when designing a loyalty program."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Loyalties API

Complete API reference for 23blocks loyalty program management with money, product, and event earning rules.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://rewards.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/loyalties/` | List all loyalty programs |
| GET | `/loyalties/:unique_id` | Get a loyalty program |
| POST | `/loyalties/` | Create a loyalty program |
| PUT | `/loyalties/:unique_id` | Update a loyalty program |
| GET | `/loyalties/:unique_id/stats` | Get program performance stats |
| POST | `/loyalties/:unique_id/details` | Add a program detail |
| PUT | `/loyalties/:unique_id/details/:details_unique_id` | Update a program detail |
| GET | `/loyalties/:unique_id/rules/money` | List money earning rules |
| POST | `/loyalties/:unique_id/rules/money` | Create a money earning rule |
| PUT | `/loyalties/:unique_id/rules/money/:rule_unique_id` | Update a money rule |
| PUT | `/loyalties/:unique_id/rules/money/:rule_unique_id/expirations/` | Set rule expiration |
| DELETE | `/loyalties/:unique_id/rules/money/:rule_unique_id/expirations/:expiration_rule_unique_id` | Remove rule expiration |
| GET | `/loyalties/:unique_id/rules/products` | List product earning rules |
| POST | `/loyalties/:unique_id/rules/products` | Create a product earning rule |
| PUT | `/loyalties/:unique_id/rules/products/:rule_unique_id` | Update a product rule |
| DELETE | `/loyalties/:unique_id/rules/products/:rule_unique_id` | Delete a product rule |
| GET | `/loyalties/:unique_id/rules/events` | List event earning rules |
| POST | `/loyalties/:unique_id/rules/events` | Create an event earning rule |
| PUT | `/loyalties/:unique_id/rules/events/:rule_unique_id` | Update an event rule |
| PUT | `/loyalties/:unique_id/rules/:rule_unique_id/disable` | Disable a rule |
| PUT | `/loyalties/:unique_id/rules/:rule_unique_id/enable` | Enable a rule |

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Loyalty Program Not Found","detail":"The requested loyalty program could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-rewards`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// LoyaltyService — client.rewards.loyalty
client.rewards.loyalty.get(uniqueId: string): Promise<Loyalty>;
client.rewards.loyalty.getByUser(userUniqueId: string): Promise<Loyalty>;
client.rewards.loyalty.addPoints(data: AddPointsRequest): Promise<LoyaltyTransaction>;
client.rewards.loyalty.redeemPoints(data: RedeemPointsRequest): Promise<LoyaltyTransaction>;
client.rewards.loyalty.getHistory(userUniqueId: string, params?: ListTransactionsParams): Promise<PageResult<LoyaltyTransaction>>;
```

### TypeScript Types

```typescript
import type {
  Loyalty,
  LoyaltyTransaction,
  AddPointsRequest,
  RedeemPointsRequest,
  ListTransactionsParams,
  LoyaltyTier,
} from '@23blocks/block-rewards';
```

### React Hook

```typescript
import { useRewardsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useRewardsBlock();
  const result = await client.rewards.loyalty.getByUser('user-unique-id');
}
```
