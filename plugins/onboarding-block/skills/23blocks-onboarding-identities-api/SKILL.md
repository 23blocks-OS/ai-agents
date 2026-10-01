---
name: 23blocks-onboarding-identities-api
description: "Onboarding Block user identities: register (optionally starting an onboarding), profiles, journeys. Use before a user's first call."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks Onboarding user identity management with journey tracking and registration.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://onboarding.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /users/ - List Identities

Lists all user identities in the onboarding system with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users?page=1&records=20" \
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
      "id": "user-uuid-123",
      "type": "user_identity",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "display_name": "John Doe",
        "status": "active",
        "journeys_count": 3,
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

### GET /users/:unique_id/ - Get Identity

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
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "display_name": "John Doe",
      "status": "active",
      "journeys_count": 3,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users/:unique_id/register/ - Register Identity

Registers a new user identity in the onboarding system. The `user_unique_id` (the `:unique_id` path parameter) is the only required value.

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
      "display_name": "New User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | No | Optional denormalized routing field; if blank, the block skips email notifications |
| `display_name` | string | No | Display name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "display_name": "New User",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Missing `user_unique_id`

---

### POST /users/:unique_id/register/:onboarding_id - Register and Start Onboarding

Registers a new user identity and automatically starts the specified onboarding journey.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register/onboarding-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "display_name": "New User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | No | Optional denormalized routing field; if blank, the block skips email notifications |
| `display_name` | string | No | Display name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "display_name": "New User",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    },
    "relationships": {
      "journey": {
        "data": {
          "id": "journey-uuid-789",
          "type": "journey"
        }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Onboarding not found
- `422 Unprocessable Entity` - Missing `user_unique_id`

---

### PUT /users/:unique_id/ - Update Identity

Updates an existing user identity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
      "metadata": {"preferred_language": "en"}
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "display_name": "John D.",
      "metadata": {"preferred_language": "en"},
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### GET /users/:unique_id/journey - Get User Journey (Admin)

Retrieves the current active journey for a user. Admin endpoint.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/journey" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "journey-uuid-789",
    "type": "journey",
    "attributes": {
      "unique_id": "journey-uuid-789",
      "onboarding_id": "onboarding-uuid-456",
      "user_unique_id": "user-uuid-123",
      "current_step": 2,
      "status": "active",
      "started_at": "2025-01-10T10:30:00Z",
      "completed_at": null
    }
  }
}
```

**Errors:**
- `404 Not Found` - User or journey not found

---

### GET /users/:unique_id/journeys - Get All User Journeys

Retrieves all journeys for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/journeys?page=1&records=20" \
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
      "id": "journey-uuid-789",
      "type": "journey",
      "attributes": {
        "unique_id": "journey-uuid-789",
        "onboarding_id": "onboarding-uuid-456",
        "user_unique_id": "user-uuid-123",
        "current_step": 3,
        "status": "completed",
        "started_at": "2025-01-10T10:30:00Z",
        "completed_at": "2025-01-11T15:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

### GET /journeys/ - Get All Journeys

Lists all journeys across all users with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/journeys?page=1&records=20" \
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
      "id": "journey-uuid-789",
      "type": "journey",
      "attributes": {
        "unique_id": "journey-uuid-789",
        "onboarding_id": "onboarding-uuid-456",
        "user_unique_id": "user-uuid-123",
        "current_step": 2,
        "status": "active",
        "started_at": "2025-01-10T10:30:00Z",
        "completed_at": null
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

## Data Models

### UserIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `display_name` | string | Display name |
| `status` | enum | active, inactive |
| `metadata` | object | Custom metadata |
| `journeys_count` | integer | Number of journeys |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Journey (summary)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `onboarding_id` | uuid | Associated onboarding |
| `user_unique_id` | uuid | Associated user |
| `current_step` | integer | Current step number |
| `status` | enum | active, completed, suspended |
| `started_at` | timestamp | Journey start time |
| `completed_at` | timestamp | Journey completion time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"User Not Found","detail":"The requested user could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-onboarding`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// UserIdentitiesService — client.onboarding.userIdentities
client.onboarding.userIdentities.list(params?: ListUserIdentitiesParams): Promise<PageResult<UserIdentity>>;
client.onboarding.userIdentities.get(uniqueId: string): Promise<UserIdentity>;
client.onboarding.userIdentities.create(data: CreateUserIdentityRequest): Promise<UserIdentity>;
client.onboarding.userIdentities.verify(uniqueId: string, data: VerifyUserIdentityRequest): Promise<UserIdentity>;
client.onboarding.userIdentities.delete(uniqueId: string): Promise<void>;
client.onboarding.userIdentities.listByUser(userUniqueId: string): Promise<UserIdentity[]>;
```

### TypeScript Types

```typescript
import type {
  UserIdentity,
  CreateUserIdentityRequest,
  VerifyUserIdentityRequest,
  ListUserIdentitiesParams,
} from '@23blocks/block-onboarding';
```

### React Hook

```typescript
import { useOnboardingBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useOnboardingBlock();
  const result = await client.onboarding.userIdentities.list({ page: 1, perPage: 20 });
}
```
