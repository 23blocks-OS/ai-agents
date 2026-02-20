---
name: assets-identities-api
description: Manage 23blocks user identities in the Assets Block via REST API. Use when registering users, managing user profiles, or retrieving user assets, entities, and ownership records.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks Assets Block user identity management with asset and entity tracking.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://assets.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
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

## Endpoints

### GET /users/ - List Users

Lists all user identities.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
  -H "AppId: $BLOCKS_API_KEY"
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
        "name": "Software License",
        "entity_type": "digital_license",
        "status": "active",
        "access_level": "private",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/assets - User Assets

Retrieves all assets assigned to the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/assets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "asset-uuid-789",
      "type": "asset",
      "attributes": {
        "unique_id": "asset-uuid-789",
        "name": "Laptop Dell XPS 15",
        "serial_number": "SN-2025-001",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/ownership - User Ownership

Retrieves ownership records for the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/ownership" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "ownership-uuid",
      "type": "ownership_record",
      "attributes": {
        "unique_id": "ownership-uuid",
        "asset_unique_id": "asset-uuid-789",
        "asset_name": "Laptop Dell XPS 15",
        "ownership_type": "assigned",
        "assigned_at": "2025-01-10T10:30:00Z",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /users/:unique_id/register/ - Register User

Registers a new user in the assets system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
| `email` | string | Yes | User email |
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

---

### PUT /users/:unique_id/ - Update User

Updates an existing user profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
npm install @23blocks/block-assets
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

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.users.list();
}
```
