---
name: 23blocks-jarvis-clusters-api
description: "Jarvis entity clusters: members, prompts, contexts, conversations. Use when several entities act as a group."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Clusters API

Complete API reference for 23blocks Jarvis entity cluster management with members, prompts, contexts, and conversations.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://jarvis.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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
| GET | `/clusters` | List all clusters |
| GET | `/clusters/:id` | Get a single cluster |
| POST | `/clusters` | Create a cluster |
| PUT | `/clusters/:id` | Update a cluster |
| DELETE | `/clusters/:id` | Delete a cluster |
| POST | `/clusters/:id/members/:entity_id` | Add member to cluster |
| DELETE | `/clusters/:id/members/:entity_id` | Remove member from cluster |
| GET | `/clusters/:id/prompts` | List cluster prompts |
| POST | `/clusters/:id/prompts/:prompt_id` | Add prompt to cluster |
| DELETE | `/clusters/:id/prompts/:prompt_id` | Remove prompt from cluster |
| GET | `/clusters/:id/contexts` | List cluster contexts |
| POST | `/clusters/:id/contexts` | Create cluster context |
| GET | `/clusters/:id/conversations` | List cluster conversations |
| POST | `/clusters/:id/conversations` | Create cluster conversation |
| GET | `/clusters/:id/conversations/:conv_id/messages` | List cluster messages |
| POST | `/clusters/:id/conversations/:conv_id/messages` | Send cluster message |

---

## Data Models

### Cluster
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Cluster name |
| `description` | string | Cluster description |
| `members_count` | integer | Number of member entities |
| `prompts_count` | integer | Number of assigned prompts |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Cluster Not Found","detail":"The requested cluster could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// ClustersService — client.jarvis.clusters
list(userUniqueId: string, params?: ListClustersParams): Promise<PageResult<Cluster>>;
get(userUniqueId: string, uniqueId: string): Promise<Cluster>;
create(userUniqueId: string, data: CreateClusterRequest): Promise<Cluster>;
update(userUniqueId: string, uniqueId: string, data: UpdateClusterRequest): Promise<Cluster>;
delete(userUniqueId: string, uniqueId: string): Promise<void>;
addPrompt(userUniqueId: string, uniqueId: string, promptUniqueId: string): Promise<Cluster>;
createContext(userUniqueId: string, uniqueId: string, data?: CreateContextRequest): Promise<unknown>;
sendMessage(userUniqueId: string, uniqueId: string, contextUniqueId: string, data: SendMessageRequest): Promise<unknown>;
```

### TypeScript Types

```typescript
import type {
  Cluster,
  CreateClusterRequest,
  UpdateClusterRequest,
  ListClustersParams,
  CreateContextRequest,
  SendMessageRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();
  const result = await client.jarvis.clusters.list('user-unique-id');
}
```
