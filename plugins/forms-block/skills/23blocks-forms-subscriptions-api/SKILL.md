---
name: 23blocks-forms-subscriptions-api
description: "Forms Block newsletter subscriptions: subscribe, unsubscribe, lists. Use for email signup forms."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Subscriptions API

Complete API reference for 23blocks newsletter subscription management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://forms.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /subscriptions/:form_unique_id/instances - List Subscriptions

Lists all subscriptions for a specific form.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscriptions/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "sub-123",
      "type": "subscription",
      "attributes": {
        "unique_id": "sub-123",
        "email": "subscriber@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "status": "active",
        "source": "website",
        "subscribed_at": "2025-01-10T10:30:00Z",
        "unsubscribed_at": null,
        "preferences": {
          "newsletter": true,
          "product_updates": true,
          "promotions": false
        },
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 150
  }
}
```

---

### GET /subscriptions/:form_unique_id/instances/:unique_id - Get Subscription

Retrieves a specific subscription.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscriptions/form-123/instances/sub-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

---

### POST /subscriptions/:form_unique_id/instances - Create Subscription

Creates a new subscription.

> **Public Endpoint:** This endpoint does NOT require authentication. The `Authorization` and `X-API-KEY` headers are optional. Public submissions are accepted without auth.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subscriptions/form-123/instances" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "email": "newsubscriber@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "source": "footer_form",
      "preferences": {
        "newsletter": true,
        "product_updates": true,
        "promotions": false
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Subscriber email |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `source` | string | No | Subscription source |
| `preferences` | object | No | Email preferences |

---

### PUT /subscriptions/:form_unique_id/instances/:unique_id - Update Subscription

Updates a subscription.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/subscriptions/form-123/instances/sub-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscription": {
      "status": "unsubscribed",
      "preferences": {
        "newsletter": false
      }
    }
  }'
```

---

### DELETE /subscriptions/:form_unique_id/instances/:unique_id - Delete Subscription

Soft-deletes a subscription.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/subscriptions/form-123/instances/sub-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Subscription
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | Subscriber email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `status` | enum | active, unsubscribed |
| `source` | string | Subscription source |
| `preferences` | object | Email preferences |
| `subscribed_at` | timestamp | Subscription time |
| `unsubscribed_at` | timestamp | Unsubscribe time |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Email can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-forms`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// SubscriptionsService — client.forms.subscriptions
list(formUniqueId: string, params?: ListSubscriptionsParams): Promise<PageResult<Subscription>>;
get(formUniqueId: string, uniqueId: string): Promise<Subscription>;
submit(formUniqueId: string, data: CreateSubscriptionRequest): Promise<Subscription>;
update(formUniqueId: string, uniqueId: string, data: UpdateSubscriptionRequest): Promise<Subscription>;
delete(formUniqueId: string, uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Subscription,
  CreateSubscriptionRequest,
  UpdateSubscriptionRequest,
  ListSubscriptionsParams,
} from '@23blocks/block-forms';
```

### React Hook

```typescript
import { useFormsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFormsBlock();

  // Example: list subscriptions for a form
  const result = await client.forms.subscriptions.list('form-unique-id');
}
```
