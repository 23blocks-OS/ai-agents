---
name: 23blocks-crm-subscribers-api
description: Manage 23blocks CRM subscribers via REST API. Use when creating, updating, or managing subscriber lists and handling communication unsubscribe requests.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Subscribers API

Complete API reference for 23blocks CRM subscriber management and communication preferences.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Crm API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://crm.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /subscribers/ - List Subscribers

Lists all subscribers with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscribers/?page=1&records=20" \
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
      "id": "subscriber-uuid-123",
      "type": "subscriber",
      "attributes": {
        "unique_id": "subscriber-uuid-123",
        "email": "subscriber@example.com",
        "name": "Alice Johnson",
        "source": "website_form",
        "subscribed_at": "2026-01-05T08:00:00Z",
        "status": "active",
        "created_at": "2026-01-05T08:00:00Z",
        "updated_at": "2026-01-05T08:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 142
  }
}
```

---

### GET /subscribers/:unique_id - Get Subscriber

Retrieves a single subscriber by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/subscribers/subscriber-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "subscriber-uuid-123",
    "type": "subscriber",
    "attributes": {
      "unique_id": "subscriber-uuid-123",
      "email": "subscriber@example.com",
      "name": "Alice Johnson",
      "source": "website_form",
      "subscribed_at": "2026-01-05T08:00:00Z",
      "status": "active",
      "created_at": "2026-01-05T08:00:00Z",
      "updated_at": "2026-01-05T08:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Subscriber not found

---

### POST /subscribers/ - Create Subscriber

Creates a new subscriber.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/subscribers/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscriber": {
      "email": "newsubscriber@example.com",
      "name": "Bob Williams",
      "source": "landing_page"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Subscriber email address |
| `name` | string | No | Subscriber name |
| `source` | string | No | Subscription source (website_form, landing_page, import, api) |

**Response 201:**
```json
{
  "data": {
    "id": "subscriber-uuid-456",
    "type": "subscriber",
    "attributes": {
      "unique_id": "subscriber-uuid-456",
      "email": "newsubscriber@example.com",
      "name": "Bob Williams",
      "source": "landing_page",
      "subscribed_at": "2026-02-15T10:30:00Z",
      "status": "active",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (e.g., duplicate email)

---

### PUT /subscribers/:unique_id/ - Update Subscriber

Updates an existing subscriber.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/subscribers/subscriber-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscriber": {
      "name": "Alice M. Johnson",
      "status": "active"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "subscriber-uuid-123",
    "type": "subscriber",
    "attributes": {
      "unique_id": "subscriber-uuid-123",
      "name": "Alice M. Johnson",
      "status": "active",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Subscriber not found

---

### DELETE /subscribers/:unique_id/ - Delete Subscriber

Deletes a subscriber.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/subscribers/subscriber-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Subscriber not found

---

### POST /communications/unsubscribe - Unsubscribe

Processes a communication unsubscribe request.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/communications/unsubscribe" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "subscriber@example.com",
    "reason": "no_longer_interested"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | Email to unsubscribe |
| `reason` | string | No | Unsubscribe reason |

**Response 200:**
```json
{
  "message": "Successfully unsubscribed"
}
```

**Errors:**
- `404 Not Found` - Email not found in subscriber list

---

## Data Models

### Subscriber
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | Subscriber email address |
| `name` | string | Subscriber name |
| `source` | string | Subscription source |
| `subscribed_at` | timestamp | When the subscription started |
| `status` | enum | active, unsubscribed, bounced |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Subscriber Not Found",
    "detail": "The requested subscriber could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-crm
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
// SubscribersService — client.crm.subscribers
list(params?: ListSubscribersParams): Promise<PageResult<Subscriber>>;
get(uniqueId: string): Promise<Subscriber>;
create(data: CreateSubscriberRequest): Promise<Subscriber>;
update(uniqueId: string, data: UpdateSubscriberRequest): Promise<Subscriber>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Subscriber,
  CreateSubscriberRequest,
  UpdateSubscriberRequest,
  ListSubscribersParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: List all subscribers
  const result = await client.crm.subscribers.list();
}
```
