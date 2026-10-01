---
name: 23blocks-auth-users-api
description: "Auth Block user accounts: CRUD, search, me, change or reset password, verify email, activate. Use for account management."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Users API

Complete API reference for 23blocks user account management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Email has already been taken."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-authentication`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
