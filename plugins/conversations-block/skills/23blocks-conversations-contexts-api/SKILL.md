---
name: 23blocks-conversations-contexts-api
description: Organize groups and conversations into logical domains via contexts. Use when creating contexts for projects, departments, or topics, or managing hierarchical conversation organization.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Contexts API

Create and manage contexts that organize groups and conversations into logical domains or topics. Contexts provide a higher-level organizational structure for grouping related conversations and teams.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Conversations API base URL | `https://conversations.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

```bash
curl -X GET "$BLOCKS_API_URL/contexts/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

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

### React Hook

```typescript
import { useConversationsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useConversationsBlock();
  const result = await client.conversations.contexts.list();
}
```
