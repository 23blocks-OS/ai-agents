---
name: 23blocks-conversations-identities-api
description: Manage user identities, registration, status, and real-time WebSocket connectivity. Use when registering users, updating user status, generating WebSocket tokens, or managing user presence.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Conversations Identities API

Manage user identities within the Conversations Block. Users must register their identity before accessing private endpoints. This skill also handles user status management and WebSocket token generation for real-time connectivity.

> **Note:** Block identity records are notification routing caches, not identity models. The canonical user record lives in the Auth (Gateway) block. `email`/`phone` here are optional denormalized routing fields; duplicates across users are allowed. The only validated field at registration is `user_unique_id`.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://realtime.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://realtime.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://realtime.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


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

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-conversations
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

### React Hook

```typescript
import { useConversationsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useConversationsBlock();
  const result = await client.conversations.users.list();
}
```
