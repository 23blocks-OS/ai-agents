---
name: 23blocks-assets-identities-api
description: "Assets Block user identities: register, profiles, a user's assets, entities and ownership. Use before a user's first Assets call."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks Assets Block user identity management with asset and entity tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://assets.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /users/ - List Users

Lists all user identities.

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
      "type": "user_identity",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "username": "johndoe",
        "display_name": "John Doe",
        "avatar_url": "https://example.com/avatar.jpg",
        "assets_count": 12,
        "entities_count": 5,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/ - Get User

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
      "username": "johndoe",
      "display_name": "John Doe",
      "avatar_url": "https://example.com/avatar.jpg",
      "assets_count": 12,
      "entities_count": 5,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### GET /users/:unique_id/entities - User Entities

Retrieves all digital entities associated with the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200 (plain JSON, not JSON:API):**
```json
[
  {
    "unique_id": "entity-uuid-456",
    "name": "Software License",
    "entity_type": "digital_license",
    "status": "active",
    "access_level": "private",
    "created_at": "2025-01-10T10:30:00Z"
  }
]
```

---

### GET /users/:unique_id/assets - User Assets

Retrieves all assets assigned to the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/assets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200 (plain JSON, not JSON:API):**
```json
[
  {
    "unique_id": "asset-uuid-789",
    "name": "Laptop Dell XPS 15",
    "serial": "SN-2025-001",
    "status": "active",
    "created_at": "2025-01-10T10:30:00Z"
  }
]
```

---

### GET /users/:unique_id/ownership - User Ownership

Retrieves ownership records for the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/ownership" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200 (plain JSON, not JSON:API):**
```json
[
  {
    "unique_id": "ownership-uuid",
    "asset_unique_id": "asset-uuid-789",
    "asset_name": "Laptop Dell XPS 15",
    "ownership_type": "assigned",
    "assigned_at": "2025-01-10T10:30:00Z",
    "created_at": "2025-01-10T10:30:00Z"
  }
]
```

---

### POST /users/:unique_id/register/ - Register User

Registers a new user in the assets system.

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
      "username": "newuser",
      "display_name": "New User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User unique ID (from the URL path `:unique_id`) — the only required value |
| `email` | string | No | Optional denormalized routing field for notifications; if blank, the block skips the email channel. Duplicates across users are allowed |
| `username` | string | No | Username |
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
      "username": "newuser",
      "display_name": "New User",
      "assets_count": 0,
      "entities_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Missing `user_unique_id`

---

### PUT /users/:unique_id/ - Update User

Updates an existing user profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
      "avatar_url": "https://example.com/new-avatar.jpg"
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
      "avatar_url": "https://example.com/new-avatar.jpg",
      "updated_at": "2025-01-12T14:00:00Z"
    }
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
| `username` | string | Username |
| `display_name` | string | Display name |
| `avatar_url` | string | Avatar image URL |
| `user_type` | enum | `human` or `ai_agent` — distinguishes human users from AI agents |
| `assets_count` | integer | Number of assigned assets |
| `entities_count` | integer | Number of entities |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### OwnershipRecord
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `asset_unique_id` | uuid | Associated asset ID |
| `asset_name` | string | Asset name |
| `ownership_type` | string | Type of ownership (assigned, lent, transferred) |
| `assigned_at` | timestamp | Assignment time |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"User Not Found","detail":"The requested user could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-assets`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAssetsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// AssetsUsersService — client.assets.users
client.assets.users.list(params?: ListAssetsUsersParams): Promise<PageResult<AssetsUser>>;
client.assets.users.get(uniqueId: string): Promise<AssetsUser>;
client.assets.users.register(uniqueId: string, data: RegisterAssetsUserRequest): Promise<AssetsUser>;
client.assets.users.update(uniqueId: string, data: UpdateAssetsUserRequest): Promise<AssetsUser>;
client.assets.users.listEntities(uniqueId: string): Promise<AssetsEntity[]>;
client.assets.users.listAssets(uniqueId: string): Promise<Asset[]>;
client.assets.users.listOwnership(uniqueId: string): Promise<UserOwnership[]>;
```

### TypeScript Types

```typescript
import type {
  AssetsUser,
  RegisterAssetsUserRequest,
  UpdateAssetsUserRequest,
  ListAssetsUsersParams,
  UserOwnership,
} from '@23blocks/block-assets';
```
