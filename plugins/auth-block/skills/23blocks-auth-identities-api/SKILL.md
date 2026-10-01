---
name: 23blocks-auth-identities-api
description: "Auth Block identity registration: register, list, get. Use when a user must be registered in the auth block."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks Auth identity registration and management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /identities - List Identities

Lists all registered identities in the auth.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities?page=1&records=20" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Identity Not Found","detail":"The requested identity could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-authentication`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAuthenticationBlock()` from `@23blocks/react`.

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
