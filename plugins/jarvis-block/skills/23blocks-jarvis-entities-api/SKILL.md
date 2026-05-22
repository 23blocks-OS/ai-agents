---
name: 23blocks-jarvis-entities-api
description: Manage 23blocks Jarvis digital twin entities via REST API. Use when creating entities, managing entity contexts, handling entity conversations and messages, streaming responses, or querying entity files with AI.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
---

# Entities API

Complete API reference for 23blocks Jarvis digital twin entity management with contexts, conversations, streaming, and AI file queries.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
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
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Prerequisites

**User identity must be registered** before calling any endpoint in this skill. Without registration, all requests return `404` with code `usr-not-registered`.

```bash
curl -X POST "$BLOCKS_API_URL/identities/$USER_UNIQUE_ID/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "Your Name", "email": "you@example.com" }'
```

> Self-registration (your own JWT) requires no special scope. Registering other users requires `identities:write`.

---

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/entities` | List all entities |
| GET | `/entities/:id` | Get a single entity |
| POST | `/entities` | Create an entity |
| PUT | `/entities/:id` | Update an entity |
| DELETE | `/entities/:id` | Delete an entity |
| GET | `/entities/:id/contexts` | List entity contexts |
| POST | `/entities/:id/contexts` | Create entity context |
| GET | `/entities/:id/conversations` | List entity conversations |
| POST | `/entities/:id/conversations` | Create entity conversation |
| GET | `/entities/:id/conversations/:conv_id/messages` | List entity messages |
| POST | `/entities/:id/conversations/:conv_id/messages` | Send entity message |
| POST | `/entities/:id/conversations/:conv_id/messages/stream` | Stream entity message |
| POST | `/entities/:id/file_query` | Query entity files with AI |

---

## Data Models

### Entity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Entity name |
| `entity_type` | string | Type classification |
| `description` | string | Entity description |
| `instructions` | string | Per-entity AI instructions |
| `status` | enum | active, inactive |
| `created_by` | string | User who created the entity (audit trail) |
| `updated_by` | string | User who last updated the entity (audit trail) |
| `contexts_count` | integer | Number of contexts |
| `conversations_count` | integer | Number of conversations |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Context
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Context name |
| `content` | string | Context content |
| `created_at` | timestamp | Creation time |

### Message
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `role` | enum | user, assistant, system |
| `content` | string | Message content |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Entity Not Found",
    "detail": "The requested entity could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-jarvis
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
// EntitiesService — client.jarvis.entities
list(params?: ListEntitiesParams): Promise<PageResult<Entity>>;
get(uniqueId: string): Promise<Entity>;
register(uniqueId: string, data?: RegisterEntityRequest): Promise<Entity>;
update(uniqueId: string, data: UpdateEntityRequest): Promise<Entity>;
delete(uniqueId: string): Promise<void>;
addPrompt(uniqueId: string, promptUniqueId: string): Promise<void>;
createContext(uniqueId: string, data?: CreateContextRequest): Promise<unknown>;
sendMessage(uniqueId: string, contextUniqueId: string, data: SendMessageRequest): Promise<unknown>;
sendMessageStream(uniqueId: string, contextUniqueId: string, data: SendMessageRequest): Promise<ReadableStream<string>>;
```

### TypeScript Types

```typescript
import type {
  Entity,
  RegisterEntityRequest,
  UpdateEntityRequest,
  ListEntitiesParams,
  CreateContextRequest,
  SendMessageRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();
  const result = await client.jarvis.entities.list();
}
```
