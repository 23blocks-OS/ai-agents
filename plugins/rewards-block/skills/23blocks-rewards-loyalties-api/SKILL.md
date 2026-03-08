---
name: 23blocks-rewards-loyalties-api
description: Manage 23blocks loyalty programs via REST API. Use when creating loyalty programs, configuring money/product/event earning rules, managing rule expirations, adding program details, and viewing program stats.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Loyalties API

Complete API reference for 23blocks loyalty program management with money, product, and event earning rules.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

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
