---
name: 23blocks-jarvis-agents-api
description: "Jarvis agent definitions: CRUD, settings, prompts, entity bindings, supervisor handoffs. Use when creating or configuring an agent."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.2"
---

# Agents API

Complete API reference for 23blocks Jarvis AI agent management with prompt assignments, entity bindings, and supervisor handoffs.

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

## Endpoints

Request and response detail for each endpoint: [ENDPOINTS.md](ENDPOINTS.md).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/agents` | List Agents |
| GET | `/agents/:id` | Get Agent |
| POST | `/agents` | Create Agent |
| PUT | `/agents/:id` | Update Agent |
| DELETE | `/agents/:id` | Delete Agent |
| POST | `/agents/:id/prompts/:prompt_id` | Add Prompt to Agent (Agent Prompts) |
| DELETE | `/agents/:id/prompts/:prompt_id` | Remove Prompt from Agent (Agent Prompts) |
| POST | `/agents/:id/entities/:entity_id` | Add Entity to Agent (Agent Entities) |
| DELETE | `/agents/:id/entities/:entity_id` | Remove Entity from Agent (Agent Entities) |
| POST | `/agents/:id/context/:context_id/handoff` | Create Handoff (Supervisor Handoff) |
| GET | `/agents/:id/context/:context_id/handoff` | List Handoffs (Supervisor Handoff) |
| DELETE | `/agents/:id/context/:context_id/handoff/:delegation_id` | Revoke Handoff (Supervisor Handoff) |

## Context Creation Behavior

When creating agent contexts, if no `members` array is provided, Jarvis auto-populates it from the JWT token (user_unique_id + user_email). The `members` parameter (array, optional) can be explicitly passed during context creation to override this default behavior.

> Context `unique_id` must be a valid UUID; other values return 400.

## Data Models

### Agent
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Agent name |
| `code` | string | Agent code/identifier |
| `description` | string | Agent description |
| `system_prompt` | string | System prompt for behavior |
| `provider` | string | LLM provider (`openai`, `anthropic`, `google`, `mistral`, `perplexity`, `openai_compatible`, `custom`) |
| `model` | string | Model identifier for the provider |
| `supervisor_user_uid` | uuid | User UID assigned as agent supervisor |
| `status` | enum | active, inactive |
| `prompts_count` | integer | Number of assigned prompts |
| `entities_count` | integer | Number of bound entities |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Agent Not Found","detail":"The requested agent could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useJarvisBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// AgentsService — client.jarvis.agents
list(params?: ListAgentsParams): Promise<PageResult<Agent>>;
get(uniqueId: string): Promise<Agent>;
create(data: CreateAgentRequest): Promise<Agent>;
update(uniqueId: string, data: UpdateAgentRequest): Promise<Agent>;
delete(uniqueId: string): Promise<void>;
addPrompt(uniqueId: string, data: AddAgentPromptRequest): Promise<Agent>;
addEntity(uniqueId: string, data: AddAgentEntityRequest): Promise<Agent>;
removeEntity(uniqueId: string, data: AddAgentEntityRequest): Promise<Agent>;
```

### TypeScript Types

```typescript
import type {
  Agent,
  CreateAgentRequest,
  UpdateAgentRequest,
  ListAgentsParams,
  AddAgentPromptRequest,
  AddAgentEntityRequest,
} from '@23blocks/block-jarvis';
```
