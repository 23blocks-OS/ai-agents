---
name: 23blocks-conversations-contexts-api
description: "Conversations Block contexts that group conversations and groups by project, department or topic. Use to organize conversations."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Contexts API

Create and manage contexts that organize groups and conversations into logical domains or topics. Contexts provide a higher-level organizational structure for grouping related conversations and teams.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/contexts/` | List contexts |
| GET | `/contexts/:unique_id` | Get context |
| POST | `/contexts/` | Create context |
| PUT | `/contexts/:unique_id` | Update context |
| GET | `/context/:context_unique_id/groups` | List context groups |

---

## Data Model

### Context

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the context |
| name | string | Context name |
| description | string | Context description |
| context_type | string | Type: `project`, `department`, `topic`, `custom` |
| metadata | object | Arbitrary key-value metadata |
| status | string | Context status: `active`, `archived` |
| group_count | integer | Number of groups in this context |
| groups | array | Summary list of associated groups (in detail view) |
| created_at | datetime | Context creation timestamp |
| updated_at | datetime | Last update timestamp |

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

Common status codes: `401` Unauthorized, `404` Not Found, `422` Unprocessable Entity.

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-conversations`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useConversationsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// ContextsService — client.conversations.contexts
list(params?: ListContextsParams): Promise<PageResult<Context>>;
get(uniqueId: string): Promise<Context>;
create(data: CreateContextRequest): Promise<Context>;
update(uniqueId: string, data: UpdateContextRequest): Promise<Context>;
listGroups(contextUniqueId: string): Promise<PageResult<Group>>;
```

### TypeScript Types

```typescript
import type {
  Context,
  CreateContextRequest,
  UpdateContextRequest,
  ListContextsParams,
} from '@23blocks/block-conversations';
```
