---
name: 23blocks-jarvis-clusters-api
description: Manage 23blocks Jarvis entity clusters via REST API. Use when grouping entities, managing cluster members, assigning prompts, or handling cluster contexts and conversations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Clusters API

Complete API reference for 23blocks Jarvis entity cluster management with members, prompts, contexts, and conversations.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Cluster Not Found",
    "detail": "The requested cluster could not be found."
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
