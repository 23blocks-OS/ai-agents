---
name: 23blocks-sales-subscriptions-api
description: Manage 23blocks Sales subscriptions via REST API. Use when creating subscription models, managing user/entity/account subscriptions, tracking consumption, handling subscription lifecycle, or generating subscription reports.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Subscriptions API

Complete API reference for 23blocks subscription management including models, user/entity/account subscriptions, consumption tracking, and reporting.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

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

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/subscription_models/` | List all subscription models |
| GET | `/subscription_models/:unique_id` | Get a subscription model |
| POST | `/subscription_models/` | Create a subscription model |
| PUT | `/subscription_models/:unique_id` | Update a subscription model |
| GET | `/users/:user_unique_id/subscriptions/` | List user subscriptions |
| GET | `/users/:user_unique_id/subscriptions/:subscription_unique_id/` | Get user subscription |
| POST | `/users/:user_unique_id/subscriptions/:subscription_unique_id/` | Create user subscription |
| PUT | `/users/:user_unique_id/subscriptions/:subscription_unique_id/` | Update user subscription |
| POST | `/users/:user_unique_id/subscriptions/:subscription_unique_id/consumption` | Record consumption |
| PUT | `/users/:user_unique_id/subscriptions/:subscription_unique_id/cancel` | Cancel subscription |
| DELETE | `/users/:user_unique_id/subscriptions/:subscription_unique_id/` | Delete subscription |
| POST | `/entities/:unique_id/subscriptions/` | Create entity subscription |
| PUT | `/entities/:unique_id/subscriptions/:subscription_unique_id` | Update entity subscription |
| POST | `/subscriptions` | Create account subscription |
| GET | `/subscriptions/:subscription_unique_id` | Get account subscription |
| DELETE | `/subscriptions/:subscription_unique_id` | Delete account subscription |
| GET | `/subscriptions/:subscription_unique_id/items` | List subscription items |
| POST | `/subscriptions/:subscription_unique_id/items` | Add subscription item |
| DELETE | `/subscriptions/:subscription_unique_id/items/:item_unique_id` | Remove subscription item |
| POST | `/purchases` | Create a one-time purchase |
| POST | `/reports/users/subscriptions/list` | Subscription list report |
| POST | `/reports/users/subscriptions/summary` | Subscription summary report |

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

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-sales
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
// Subscriptions — client.sales.subscriptions
client.sales.subscriptions.list(params?: ListSubscriptionsParams): Promise<PageResult<Subscription>>;
client.sales.subscriptions.get(uniqueId: string): Promise<Subscription>;
client.sales.subscriptions.create(data: CreateSubscriptionRequest): Promise<Subscription>;
client.sales.subscriptions.addItem(subscriptionUniqueId: string, data: CreateSubscriptionItemRequest): Promise<SubscriptionItem>;
```

### TypeScript Types

```typescript
import type {
  Subscription,
  SubscriptionItem,
  CreateSubscriptionRequest,
  CreateSubscriptionItemRequest,
  ListSubscriptionsParams,
} from '@23blocks/block-sales';
```

### React Hook

```typescript
import { useSalesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSalesBlock();
  const result = await client.sales.subscriptions.list();
}
```
