---
name: sales-identities-api
description: Manage 23blocks Sales user, entity, and customer identities via REST API. Use when registering users, entities, or customers in the sales system, or retrieving identity information.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks Sales identity management including users, entities, and customers.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://sales.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

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

---

## User Endpoints

### GET /users/ - List Users

Lists all registered users in the sales system.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/?page=1&records=20" \
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
      "id": "user-uuid-123",
      "type": "user",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "display_name": "John Doe",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /users/:unique_id/ - Get User

Retrieves a single user by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "display_name": "John Doe",
      "phone": "+1234567890",
      "status": "active",
      "orders_count": 12,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users/:unique_id/register/ - Register User

Registers a new user in the sales system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "display_name": "Jane Smith"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `display_name` | string | No | Display name |
| `phone` | string | No | Phone number |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "display_name": "Jane Smith",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /users/:unique_id/ - Update User

Updates an existing user profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "Jane S.",
      "phone": "+1987654321"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "display_name": "Jane S.",
      "phone": "+1987654321",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Entity Endpoints

### GET /entities/ - List Entities

Lists all registered entities.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "entity-uuid-456",
      "type": "entity",
      "attributes": {
        "unique_id": "entity-uuid-456",
        "name": "Acme Corp",
        "entity_type": "organization",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 25
  }
}
```

---

### POST /entities/:unique_id/register/ - Register Entity

Registers a new entity in the sales system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-456/register/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "name": "Acme Corp",
      "entity_type": "organization",
      "email": "billing@acme.com"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Entity name |
| `entity_type` | string | No | Type classification |
| `email` | string | No | Entity email |

**Response 201:**
```json
{
  "data": {
    "id": "entity-uuid-456",
    "type": "entity",
    "attributes": {
      "unique_id": "entity-uuid-456",
      "name": "Acme Corp",
      "entity_type": "organization",
      "email": "billing@acme.com",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Customer Endpoints

### GET /customers/:unique_id/ - Get Customer

Retrieves a customer profile.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/customers/customer-uuid-789/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "customer-uuid-789",
    "type": "customer",
    "attributes": {
      "unique_id": "customer-uuid-789",
      "email": "customer@example.com",
      "first_name": "Alice",
      "last_name": "Johnson",
      "company_name": "Johnson LLC",
      "phone": "+1555000111",
      "status": "active",
      "orders_count": 8,
      "total_spent": 2450.00,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer not found

---

### POST /customers/:unique_id/register/ - Register Customer

Registers a new customer in the sales system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/customers/customer-uuid-789/register/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customer": {
      "email": "customer@example.com",
      "first_name": "Alice",
      "last_name": "Johnson",
      "company_name": "Johnson LLC",
      "phone": "+1555000111"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Customer email |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `company_name` | string | No | Company name |
| `phone` | string | No | Phone number |

**Response 201:**
```json
{
  "data": {
    "id": "customer-uuid-789",
    "type": "customer",
    "attributes": {
      "unique_id": "customer-uuid-789",
      "email": "customer@example.com",
      "first_name": "Alice",
      "last_name": "Johnson",
      "company_name": "Johnson LLC",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

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
