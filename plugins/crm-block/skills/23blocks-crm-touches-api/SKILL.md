---
name: 23blocks-crm-touches-api
description: "CRM Block touches: calls, emails, meetings and notes with outcomes. Use when logging a customer interaction."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Touches API

Complete API reference for 23blocks CRM touch/interaction logging and tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://crm.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /touches/ - List Touches

Lists all touches with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/touches/?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "id": "touch-uuid-123",
      "type": "touch",
      "attributes": {
        "unique_id": "touch-uuid-123",
        "contact_id": "contact-uuid-456",
        "touch_type": "call",
        "description": "Discussed renewal terms for Q2 contract",
        "touched_at": "2026-02-10T14:00:00Z",
        "outcome": "Follow-up scheduled for next week",
        "created_at": "2026-02-10T14:30:00Z",
        "updated_at": "2026-02-10T14:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 8,
    "totalRecords": 112
  }
}
```

---

### GET /touches/:unique_id - Get Touch

Retrieves a single touch by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/touches/touch-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "touch-uuid-123",
    "type": "touch",
    "attributes": {
      "unique_id": "touch-uuid-123",
      "contact_id": "contact-uuid-456",
      "touch_type": "call",
      "description": "Discussed renewal terms for Q2 contract",
      "touched_at": "2026-02-10T14:00:00Z",
      "outcome": "Follow-up scheduled for next week",
      "created_at": "2026-02-10T14:30:00Z",
      "updated_at": "2026-02-10T14:30:00Z"
    },
    "relationships": {
      "contact": {
        "data": { "id": "contact-uuid-456", "type": "contact" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Touch not found

---

### POST /touches/ - Create Touch

Creates a new touch/interaction record.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/touches/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "touch": {
      "contact_id": "contact-uuid-456",
      "touch_type": "email",
      "description": "Sent follow-up proposal with updated pricing",
      "touched_at": "2026-02-15T10:00:00Z",
      "outcome": "Awaiting response"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | uuid | Yes | Associated contact ID |
| `touch_type` | enum | Yes | call, email, meeting, note |
| `description` | string | No | Description of the interaction |
| `touched_at` | timestamp | No | When the interaction occurred |
| `outcome` | string | No | Interaction outcome |

**Response 201:**
```json
{
  "data": {
    "id": "touch-uuid-456",
    "type": "touch",
    "attributes": {
      "unique_id": "touch-uuid-456",
      "contact_id": "contact-uuid-456",
      "touch_type": "email",
      "description": "Sent follow-up proposal with updated pricing",
      "touched_at": "2026-02-15T10:00:00Z",
      "outcome": "Awaiting response",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /touches/:unique_id/ - Update Touch

Updates an existing touch record.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/touches/touch-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "touch": {
      "outcome": "Client agreed to renewal at proposed terms"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "touch-uuid-123",
    "type": "touch",
    "attributes": {
      "unique_id": "touch-uuid-123",
      "outcome": "Client agreed to renewal at proposed terms",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Touch not found

---

### DELETE /touches/:unique_id/ - Delete Touch

Deletes a touch record.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/touches/touch-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Touch not found

---

## Data Models

### Touch
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `contact_id` | uuid | Associated contact ID |
| `touch_type` | enum | call, email, meeting, note |
| `description` | string | Description of the interaction |
| `touched_at` | timestamp | When the interaction occurred |
| `outcome` | string | Interaction outcome |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Touch Not Found","detail":"The requested touch record could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-crm`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCrmBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// TouchesService — client.crm.touches
list(params?: ListTouchesParams): Promise<PageResult<Touch>>;
get(uniqueId: string): Promise<Touch>;
create(data: CreateTouchRequest): Promise<Touch>;
update(uniqueId: string, data: UpdateTouchRequest): Promise<Touch>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Touch,
  CreateTouchRequest,
  UpdateTouchRequest,
  ListTouchesParams,
} from '@23blocks/block-crm';
```
