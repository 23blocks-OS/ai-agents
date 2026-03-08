---
name: 23blocks-sales-identities-api
description: Manage 23blocks Sales user, entity, and customer identities via REST API. Use when registering users, entities, or customers in the sales system, or retrieving identity information.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks Sales identity management including users, entities, and customers.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "User Not Found",
    "detail": "The requested user could not be found."
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

### React Hook

```typescript
import { useSalesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useSalesBlock();
  const result = await client.sales.users.list();
}
```
