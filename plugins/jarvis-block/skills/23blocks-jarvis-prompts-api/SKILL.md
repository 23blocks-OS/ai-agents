---
name: 23blocks-jarvis-prompts-api
description: "Jarvis prompts: CRUD, render with variables, social actions. Use when authoring prompts; running them is 23blocks-jarvis-prompt-versions-api."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.2"
---

# Prompts API

Complete API reference for 23blocks Jarvis prompt management with rendering and social interactions.

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

## Behaviour notes

- POST and PUT return a `PromptVersion`, not a `Prompt`; the Location header and body both reference the created PromptVersion.
- PUT is a partial update: fields left out of the request (persona, model, temperature, ...) keep their values.

## Endpoints

Request and response detail for each endpoint: [ENDPOINTS.md](ENDPOINTS.md).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/prompts` | List Prompts |
| GET | `/prompts/:id` | Get Prompt |
| POST | `/prompts` | Create Prompt |
| PUT | `/prompts/:id` | Update Prompt |
| DELETE | `/prompts/:id` | Delete Prompt |
| POST | `/prompts/:id/render` | Render Prompt |
| PUT | `/prompts/:id/like` | Like Prompt (Social Actions) |
| DELETE | `/prompts/:id/dislike` | Remove Like (Social Actions) |
| PUT | `/prompts/:id/follow` | Follow Prompt (Social Actions) |
| DELETE | `/prompts/:id/unfollow` | Unfollow Prompt (Social Actions) |
| PUT | `/prompts/:id/save` | Save Prompt (Social Actions) |
| DELETE | `/prompts/:id/unsave` | Unsave Prompt (Social Actions) |

## Data Models

### Prompt
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Prompt name |
| `description` | string | Prompt description |
| `content` | string | Template with {{variables}} |
| `provider` | string | LLM provider (`openai`, `anthropic`, `google`, `mistral`, `perplexity`, `openai_compatible`, `custom`) |
| `model` | string | Model identifier for the provider |
| `status` | enum | draft, published |
| `likes_count` | integer | Number of likes |
| `saves_count` | integer | Number of saves |
| `versions_count` | integer | Number of versions |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### PromptVersion (returned by POST/PUT)
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier of the version |
| `name` | string | Prompt name |
| `description` | string | Prompt description |
| `content` | string | Template with {{variables}} |
| `status` | enum | draft, published |
| `version` | integer | Major version number |
| `revision` | integer | Revision within the version |
| `prompt_unique_id` | uuid | Parent prompt unique ID |
| `user_unique_id` | uuid | Creator user unique ID |
| `user_name` | string | Creator display name |
| `is_published` | boolean | Whether currently published |
| `variables` | array | List of template variables |
| `executions_count` | integer | Number of executions |
| `published_at` | timestamp | Publication time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Prompt Not Found","detail":"The requested prompt could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// PromptsService — client.jarvis.prompts
list(params?: ListPromptsParams): Promise<PageResult<Prompt>>;
get(uniqueId: string): Promise<Prompt>;
create(data: CreatePromptRequest): Promise<PromptVersion>;   // returns PromptVersion
update(uniqueId: string, data: UpdatePromptRequest): Promise<PromptVersion>;   // returns PromptVersion
delete(uniqueId: string): Promise<void>;
execute(uniqueId: string, data: ExecutePromptRequest): Promise<ExecutePromptResponse>;
render(uniqueId: string, data: RenderPromptRequest): Promise<RenderPromptResponse>;
```

### TypeScript Types

```typescript
import type {
  Prompt,
  PromptVersion,
  CreatePromptRequest,
  UpdatePromptRequest,
  ListPromptsParams,
  ExecutePromptRequest,
  ExecutePromptResponse,
  RenderPromptRequest,
  RenderPromptResponse,
  RenderPromptMeta,
  PlaceholderValue,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: render a prompt with variables
  const result = await client.jarvis.prompts.render('prompt-uuid', {
    placeholders: { tone: 'professional', topic: 'quarterly results' },
  });
}
```
