---
name: 23blocks-sales-identities-api
description: "Sales Block identities for users, entities and customers. Use before their first Sales call."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks Sales identity management including users, entities, and customers.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://sales.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/` | List all users |
| GET | `/users/:unique_id/` | Get a single user |
| POST | `/users/:unique_id/register/` | Register a new user |
| PUT | `/users/:unique_id/` | Update a user profile |
| GET | `/entities/` | List all entities |
| POST | `/entities/:unique_id/register/` | Register a new entity |
| GET | `/customers/:unique_id/` | Get a customer |
| POST | `/customers/:unique_id/register/` | Register a new customer |

---

## Data Models

### User
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `display_name` | string | Display name |
| `phone` | string | Phone number |
| `status` | enum | active, inactive |
| `orders_count` | integer | Number of orders |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Entity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Entity name |
| `entity_type` | string | Type classification |
| `email` | string | Entity email |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

### Customer
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | Customer email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `company_name` | string | Company name |
| `phone` | string | Phone number |
| `status` | enum | active, inactive |
| `orders_count` | integer | Number of orders |
| `total_spent` | decimal | Total amount spent |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"User Not Found","detail":"The requested user could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-sales`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useSalesBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// SalesUsers — client.sales.users
client.sales.users.list(params?: ListSalesUsersParams): Promise<PageResult<SalesUser>>;
client.sales.users.get(uniqueId: string): Promise<SalesUser>;
client.sales.users.register(uniqueId: string, data?: RegisterSalesUserRequest): Promise<SalesUser>;
client.sales.users.update(uniqueId: string, data: UpdateSalesUserRequest): Promise<SalesUser>;
client.sales.users.listOrders(uniqueId: string, params?: { page?: number; perPage?: number }): Promise<PageResult<Order>>;
client.sales.users.getOrder(uniqueId: string, orderUniqueId: string): Promise<Order>;
client.sales.users.listSubscriptions(uniqueId: string, params?: ListUserSubscriptionsParams): Promise<PageResult<UserSubscription>>;
client.sales.users.getSubscription(uniqueId: string, subscriptionUniqueId: string): Promise<UserSubscription>;
client.sales.users.createSubscription(uniqueId: string, subscriptionUniqueId: string, data: CreateUserSubscriptionRequest): Promise<UserSubscription>;
client.sales.users.updateSubscription(uniqueId: string, subscriptionUniqueId: string, data: UpdateUserSubscriptionRequest): Promise<UserSubscription>;
client.sales.users.addConsumption(uniqueId: string, subscriptionUniqueId: string, data: AddSubscriptionConsumptionRequest): Promise<UserSubscription>;
client.sales.users.cancelSubscription(uniqueId: string, subscriptionUniqueId: string): Promise<UserSubscription>;
client.sales.users.deleteSubscription(uniqueId: string, subscriptionUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  SalesUser,
  RegisterSalesUserRequest,
  UpdateSalesUserRequest,
  ListSalesUsersParams,
  UserSubscription,
  CreateUserSubscriptionRequest,
  UpdateUserSubscriptionRequest,
  AddSubscriptionConsumptionRequest,
  ListUserSubscriptionsParams,
} from '@23blocks/block-sales';
```
