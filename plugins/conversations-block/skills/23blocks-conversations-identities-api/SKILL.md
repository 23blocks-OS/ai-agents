---
name: 23blocks-conversations-identities-api
description: "Conversations Block user identities: register, status and presence, WebSocket tokens. Use before a user's first Conversations call."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Conversations Identities API

Manage user identities within the Conversations Block. Users must register their identity before accessing private endpoints. This skill also handles user status management and WebSocket token generation for real-time connectivity.

> **Note:** Block identity records are notification routing caches, not identity models. The canonical user record lives in the Auth (Gateway) block. `email`/`phone` here are optional denormalized routing fields; duplicates across users are allowed. The only validated field at registration is `user_unique_id`.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/` | List users |
| GET | `/users/:unique_id/` | Get user |
| POST | `/users/:unique_id/register/` | Register user |
| PUT | `/users/:unique_id/` | Update user |
| GET | `/users/:unique_id/status` | Get user status |
| POST | `/users/status` | Set user status |
| DELETE | `/users/status` | Clear user status |
| POST | `/ws-tokens` | Generate WebSocket token |

---

## Data Model

### User

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the user |
| email | string | Optional denormalized routing field — if blank, the email notification channel is skipped |
| display_name | string | User display name |
| avatar_url | string | URL to user avatar image |
| status | string | Current status: `online`, `away`, `busy`, `offline` |
| status_message | string | Custom status message |
| status_emoji | string | Emoji code for status display |
| metadata | object | Arbitrary key-value metadata |
| last_seen_at | datetime | Timestamp of last activity |
| created_at | datetime | Account creation timestamp |
| updated_at | datetime | Last update timestamp |

### WebSocket Token

| Field | Type | Description |
|-------|------|-------------|
| token | string | JWT token for WebSocket authentication |
| ws_url | string | WebSocket server URL |
| user_unique_id | string | Associated user ID |
| device_id | string | Device identifier |
| expires_at | datetime | Token expiration time |
| created_at | datetime | Token creation timestamp |

---

## Error Response Format

```json
{
  "errors": [
    {
      "status": "401",
      "title": "Unauthorized",
      "detail": "Invalid or missing authentication token"
    }
  ]
}
```

Common status codes: `401` Unauthorized, `404` Not Found, `409` Conflict (duplicate `user_unique_id` — user already registered), `422` Unprocessable Entity (missing `user_unique_id`).

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-conversations`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useConversationsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// UsersService — client.conversations.users
list(params?: ListUsersParams): Promise<PageResult<ConversationsUser>>;
get(uniqueId: string): Promise<ConversationsUser>;
register(uniqueId: string, data?: RegisterUserRequest): Promise<ConversationsUser>;
update(uniqueId: string, data: UpdateUserRequest): Promise<ConversationsUser>;
listGroups(uniqueId: string): Promise<PageResult<Group>>;
listConversations(uniqueId: string, params?: { page?: number; perPage?: number }): Promise<PageResult<Conversation>>;
listGroupConversations(uniqueId: string, params?: { page?: number; perPage?: number }): Promise<PageResult<Conversation>>;
listContextGroups(uniqueId: string, contextUniqueId: string): Promise<PageResult<Group>>;
```

### TypeScript Types

```typescript
import type {
  ConversationsUser,
  RegisterUserRequest,
  UpdateUserRequest,
  ListUsersParams,
} from '@23blocks/block-conversations';
```
