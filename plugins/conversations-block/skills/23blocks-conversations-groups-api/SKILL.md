---
name: 23blocks-conversations-groups-api
description: "Conversations Block groups: CRUD, members, group conversations, contexts. Use for group chats; invite codes are 23blocks-conversations-group-invites-api."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Groups API

Create and manage groups of users. Supports member management, group conversations, invite codes with QR generation, and context-based grouping.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://realtime.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/:unique_id/groups` | List user groups |
| GET | `/groups/:group_unique_id/members` | Get group members |
| POST | `/groups/` | Create group |
| PUT | `/groups/:unique_id/` | Update group |
| GET | `/groups/:group_unique_id/conversations/:conversation_unique_id` | Get group conversation |
| PUT | `/groups/:unique_id/users/:user_unique_id` | Add member to group |
| DELETE | `/groups/:unique_id/users/:user_unique_id` | Remove member from group |
| POST | `/groups/conversations/contexts` | Create group with context |
| GET | `/users/:unique_id/context/:context_unique_id/groups` | List context groups |
| GET | `/groups/:group_unique_id/invites` | List group invites |
| POST | `/groups/:group_unique_id/invites` | Create group invite |
| DELETE | `/groups/:group_unique_id/invites/:code` | Revoke group invite |
| GET | `/groups/:group_unique_id/invites/:code/qr` | Get invite QR code |
| POST | `/groups/join/:code` | Join group via invite code |

> **Invite management:** For full invite lifecycle management (expiration, QR generation, rate limits, user params on join), see the dedicated **23blocks-conversations-group-invites-api** skill.

---

## Data Model

### Group

| Field | Type | Description |
|-------|------|-------------|
| unique_id | string | Unique identifier for the group |
| name | string | Group name |
| description | string | Group description |
| members | array | List of member user unique IDs |
| owner_id | string | User unique ID of the group owner |
| group_type | string | Group type identifier |
| context_id | string | Associated context ID |
| status | string | Group status: `active`, `archived`, `deleted` |
| member_count | integer | Current number of members |
| metadata | object | Arbitrary key-value metadata |
| created_at | datetime | Group creation timestamp |
| updated_at | datetime | Last update timestamp |

### GroupUser (Membership)

| Field | Type | Description |
|-------|------|-------------|
| user_unique_id | string | Member's user unique ID |
| group_unique_id | string | Parent group ID |
| role | string | Member role within the group (e.g., `owner`, `admin`, `member`, `moderator`) |
| joined_at | datetime | When the user joined the group |

### Invite

| Field | Type | Description |
|-------|------|-------------|
| code | string | URL-safe invite code (22 chars, 132 bits entropy) |
| group_unique_id | string | Associated group ID |
| name | string | Optional invite label |
| created_by | string | User who created the invite |
| status | string | Status: `active`, `revoked`, `expired` |
| max_uses | integer | Maximum uses allowed (null for unlimited) |
| use_count | integer | Number of times the code has been used |
| expires_at | datetime | Expiration timestamp (null for never) |
| created_at | datetime | Invite creation timestamp |

> For detailed invite management including `expires_in_hours`, user params on join, QR generation, and rate limiting, see the **23blocks-conversations-group-invites-api** skill.

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

Common status codes: `401` Unauthorized, `403` Forbidden, `404` Not Found, `410` Gone (expired invite), `422` Unprocessable Entity.

---

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-conversations`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useConversationsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// GroupsService — client.conversations.groups
list(params?: ListGroupsParams): Promise<PageResult<Group>>;
get(uniqueId: string): Promise<Group>;
create(data: CreateGroupRequest): Promise<Group>;
update(uniqueId: string, data: UpdateGroupRequest): Promise<Group>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Group>;
search(query: string, params?: ListGroupsParams): Promise<PageResult<Group>>;
listDeleted(params?: ListGroupsParams): Promise<PageResult<Group>>;
addMember(uniqueId: string, memberId: string): Promise<Group>;
removeMember(uniqueId: string, memberId: string): Promise<Group>;
```

### TypeScript Types

```typescript
import type {
  Group,
  CreateGroupRequest,
  UpdateGroupRequest,
  ListGroupsParams,
} from '@23blocks/block-conversations';
```
