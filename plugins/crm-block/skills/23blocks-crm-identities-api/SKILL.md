---
name: 23blocks-crm-identities-api
description: "CRM Block user identities: register, profiles, a user's contacts and meetings. Use before a user's first CRM call."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Identities API

Complete API reference for 23blocks CRM user identity management, registration, and associated data retrieval.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://crm.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /users - List Users

Lists all user identities in the CRM system.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

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
        "role": "agent",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 5
  }
}
```

---

### GET /users/:unique_id - Get User

Retrieves a single user identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "role": "agent",
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users/:unique_id/register - Register User

Registers a new user in the CRM system. Each block is autonomous; users must register their identity before using private endpoints.

> **Note:** Block identity records are notification routing caches, not identity models. The canonical user record lives in the Auth (Gateway) block. `email`/`phone` here are optional denormalized routing fields; duplicates across users are allowed.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User unique ID (from the URL path `:unique_id`) — the only required value |
| `email` | string | No | Optional denormalized routing field for notifications; if blank, the block skips the email channel. Duplicates across users are allowed |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |

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
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Missing `user_unique_id`

---

### DELETE /users/:unique_id - Delete User

Deletes a user identity from the CRM system.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - User not found

---

### GET /users/:unique_id/contacts - User Contacts

Retrieves all contacts associated with a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/contacts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "contact-uuid-456",
      "type": "contact",
      "attributes": {
        "unique_id": "contact-uuid-456",
        "first_name": "Alice",
        "last_name": "Johnson",
        "email": "alice@example.com",
        "phone": "+1-555-0101",
        "title": "CEO",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalRecords": 12
  }
}
```

---

### GET /users/:unique_id/meetings - User Meetings

Retrieves all meetings associated with a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/meetings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "meeting-uuid-789",
      "type": "meeting",
      "attributes": {
        "unique_id": "meeting-uuid-789",
        "title": "Weekly Standup",
        "start_time": "2026-02-20T14:00:00Z",
        "end_time": "2026-02-20T14:30:00Z",
        "location": "Conference Room A",
        "meeting_type": "in_person",
        "status": "scheduled",
        "created_at": "2026-01-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalRecords": 8
  }
}
```

---

## Data Models

### User
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email address |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `role` | string | User role (e.g., agent, admin) |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"User Not Found","detail":"The requested user could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-crm`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCrmBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// CrmUsersService — client.crm.users
list(params?: ListCrmUsersParams): Promise<PageResult<CrmUser>>;
get(uniqueId: string): Promise<CrmUser>;
register(uniqueId: string, data: RegisterCrmUserRequest): Promise<CrmUser>;
delete(uniqueId: string): Promise<void>;
getContacts(uniqueId: string): Promise<Contact[]>;
getMeetings(uniqueId: string): Promise<Meeting[]>;
```

### TypeScript Types

```typescript
import type {
  CrmUser,
  RegisterCrmUserRequest,
  ListCrmUsersParams,
} from '@23blocks/block-crm';
```
