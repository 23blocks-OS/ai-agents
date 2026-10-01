---
name: 23blocks-sales-subscriptions-api
description: "Sales Block subscriptions: models, user/entity/account subscriptions, items, consumption. Use for recurring billing."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Subscriptions API

Complete API reference for 23blocks subscription management including models, user/entity/account subscriptions, consumption tracking, and reporting.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://sales.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Subscription Not Found","detail":"The requested subscription could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-sales`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useSalesBlock()` from `@23blocks/react`.

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
