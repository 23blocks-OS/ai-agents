---
name: subscriptions-api
description: Manage 23blocks newsletter subscriptions via REST API. Use for email list management, subscribe/unsubscribe flows.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Subscriptions API

Complete API reference for 23blocks newsletter subscription management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://forms.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Forms API base URL | `https://forms.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/subscriptions/{form_id}/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /subscriptions/:form_unique_id/instances - List Subscriptions

Lists all subscriptions for a specific form.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscriptions/form-123/instances?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
```

---

### POST /subscriptions/:form_unique_id/instances - Create Subscription

Creates a new subscription.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subscriptions/form-123/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY" \
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
  -H "AppId: $BLOCKS_API_KEY"
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

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-forms
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
