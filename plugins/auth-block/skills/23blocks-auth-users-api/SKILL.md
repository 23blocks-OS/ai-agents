---
name: 23blocks-auth-users-api
description: Manage 23blocks users via REST API. Use when creating, updating, or deleting user accounts, managing profiles, changing passwords, resetting passwords, verifying emails, or activating/deactivating users.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Users API

Complete API reference for 23blocks user account management.

## Required Environment Variables

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

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users` | List users with pagination |
| GET | `/users/:unique_id` | Get user by ID |
| POST | `/users` | Create new user account |
| PUT | `/users/:unique_id` | Update user account |
| DELETE | `/users/:unique_id` | Soft-delete user |
| GET | `/users/me` | Get current authenticated user's profile |
| PUT | `/users/me` | Update current user's profile |
| PUT | `/users/:unique_id/change_password` | Change user password |
| POST | `/users/reset_password` | Request password reset email |
| POST | `/users/verify_email` | Verify email with token |
| PUT | `/users/:unique_id/activate` | Activate deactivated user |
| PUT | `/users/:unique_id/deactivate` | Deactivate user account |
| POST | `/users/search` | Search users with filters |

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
