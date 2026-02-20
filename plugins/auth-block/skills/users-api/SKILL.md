---
name: auth-users-api
description: Manage 23blocks users via REST API. Use when creating, updating, or deleting user accounts, managing profiles, changing passwords, resetting passwords, verifying emails, or activating/deactivating users.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Users API

Complete API reference for 23blocks user account management.

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
curl -X GET "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /users - List Users

Lists all users with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users?page=1&records=20" \
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
      "type": "user",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "status": "active",
        "email_verified": true,
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

### GET /users/:unique_id - Get User

Retrieves a single user by unique ID.

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
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "status": "active",
      "email_verified": true,
      "last_login_at": "2025-01-12T08:00:00Z",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "roles": {
        "data": [
          { "id": "role-uuid", "type": "role" }
        ]
      },
      "organization": {
        "data": { "id": "org-uuid", "type": "organization" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users - Create User

Creates a new user account.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "password": "secure_password_123",
      "first_name": "Jane",
      "last_name": "Smith"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
| `password` | string | Yes | User password |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |

**Response 201:**
```json
{
  "data": {
    "id": "new-user-uuid",
    "type": "user",
    "attributes": {
      "unique_id": "new-user-uuid",
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "status": "active",
      "email_verified": false,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Email already taken
- `422 Unprocessable Entity` - Validation errors

---

### PUT /users/:unique_id - Update User

Updates an existing user account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "first_name": "Jonathan",
      "last_name": "Doe"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "first_name": "Jonathan",
      "last_name": "Doe",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### DELETE /users/:unique_id - Delete User

Soft-deletes a user account.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - User not found

---

### GET /users/me - Get Current Profile

Retrieves the current authenticated user's profile.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/me" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "current-user-uuid",
    "type": "user",
    "attributes": {
      "unique_id": "current-user-uuid",
      "email": "me@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "status": "active",
      "email_verified": true,
      "last_login_at": "2025-01-12T08:00:00Z",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /users/me - Update Current Profile

Updates the current authenticated user's profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/me" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "first_name": "John",
      "last_name": "Updated"
    }
  }'
```

**Response 200:** Same format as GET /users/me

---

### PUT /users/:unique_id/change_password - Change Password

Changes a user's password.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/change_password" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "current_password": "old_password",
      "new_password": "new_secure_password",
      "new_password_confirmation": "new_secure_password"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `current_password` | string | Yes | Current password |
| `new_password` | string | Yes | New password |
| `new_password_confirmation` | string | Yes | Password confirmation |

**Response 200:**
```json
{
  "message": "Password changed successfully"
}
```

**Errors:**
- `401 Unauthorized` - Current password incorrect
- `422 Unprocessable Entity` - Password does not meet requirements

---

### POST /users/reset_password - Request Password Reset

Sends a password reset email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/reset_password" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "user@example.com"
    }
  }'
```

**Response 200:**
```json
{
  "message": "Password reset instructions sent"
}
```

---

### POST /users/verify_email - Verify Email

Verifies a user's email address using the verification token.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/verify_email" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "email-verification-token"
  }'
```

**Response 200:**
```json
{
  "message": "Email verified successfully"
}
```

**Errors:**
- `422 Unprocessable Entity` - Invalid or expired token

---

### PUT /users/:unique_id/activate - Activate User

Activates a deactivated user account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/activate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### PUT /users/:unique_id/deactivate - Deactivate User

Deactivates a user account.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/deactivate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user",
    "attributes": {
      "unique_id": "user-uuid-123",
      "status": "inactive",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /users/search - Search Users

Searches users with filters.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/search" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "email": "example.com",
      "status": "active"
    },
    "page": 1,
    "records": 20
  }'
```

**Response 200:** Same format as GET /users

---

## Data Models

### User
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `status` | enum | active, inactive, suspended |
| `email_verified` | boolean | Email verification status |
| `last_login_at` | timestamp | Last login time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Failed",
    "detail": "Email has already been taken."
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
  UserProfile,
  UserAvatar,
  Role,
  Permission,
  Company,
  UserSubscription,
  UserProfileFull,
  ProfileRequest,
  UpdateEmailRequest,
  UserDeviceFull,
  AddDeviceRequest,
  UserSearchRequest,
  AddUserSubscriptionRequest,
} from '@23blocks/block-authentication';

// Also exported from the service file:
import type {
  UpdateUserRequest,
  UpdateProfileRequest,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: List users with pagination
  const result = await client.authentication.users.list({ page: 1, perPage: 20 });
}
```
