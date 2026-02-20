---
name: auth-identities-api
description: Manage 23blocks Auth user identities via REST API. Use when registering users in the auth block, listing identities, or retrieving identity details.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Identities API

Complete API reference for 23blocks Auth identity registration and management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://auth.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/identities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /identities - List Identities

Lists all registered identities in the auth.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities?page=1&records=20" \
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
      "type": "identity",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "first_name": "John",
        "last_name": "Doe",
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

### GET /identities/:unique_id - Get Identity

Retrieves a single identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Identity not found

---

### POST /identities/:unique_id/register - Register Identity

Registers a new identity in the auth block.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "first_name": "New",
      "last_name": "User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email address |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "first_name": "New",
      "last_name": "User",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Identity already registered
- `422 Unprocessable Entity` - Validation errors

---

## Data Models

### Identity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email address |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `status` | string | Account status (active, inactive) |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Identity Not Found",
    "detail": "The requested identity could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-authentication
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
// UsersService — client.authentication.users
list(params?: ListParams): Promise<PageResult<User>>;
get(uniqueId: string): Promise<User>;
getByUniqueId(uniqueId: string): Promise<User>;
update(uniqueId: string, request: UpdateUserRequest): Promise<User>;
updateProfile(userUniqueId: string, request: UpdateProfileRequest): Promise<User>;
delete(uniqueId: string): Promise<void>;
activate(uniqueId: string): Promise<User>;
deactivate(uniqueId: string): Promise<User>;
changeRole(uniqueId: string, roleUniqueId: string, reason: string, forceReauth?: boolean): Promise<User>;
search(query: string, params?: ListParams): Promise<PageResult<User>>;
searchAdvanced(request: UserSearchRequest, params?: ListParams): Promise<PageResult<User>>;
getProfile(userUniqueId: string): Promise<UserProfileFull>;
createProfile(request: ProfileRequest): Promise<UserProfileFull>;
updateEmail(userUniqueId: string, request: UpdateEmailRequest): Promise<User>;
getDevices(userUniqueId: string, params?: ListParams): Promise<PageResult<UserDeviceFull>>;
addDevice(request: AddDeviceRequest): Promise<UserDeviceFull>;
getCompanies(userUniqueId: string): Promise<Company[]>;
addSubscription(userUniqueId: string, request: AddUserSubscriptionRequest): Promise<UserSubscription>;
updateSubscription(userUniqueId: string, request: AddUserSubscriptionRequest): Promise<UserSubscription>;
resendConfirmationByUniqueId(userUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  User,
  Role,
  Permission,
  UserAvatar,
  UserProfile,
  UserProfileFull,
  ProfileRequest,
  UpdateEmailRequest,
  UserDeviceFull,
  AddDeviceRequest,
  UserSearchRequest,
  AddUserSubscriptionRequest,
  Company,
  UserSubscription,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();
  const result = await client.authentication.users.list();
}
```
