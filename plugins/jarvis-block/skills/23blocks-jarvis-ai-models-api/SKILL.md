---
name: 23blocks-jarvis-ai-models-api
description: Manage 23blocks Jarvis AI models and LLM providers via REST API. Use when configuring AI models, managing LLM provider connections, validating providers, or browsing available vendor models.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# AI Models API

Complete API reference for 23blocks Jarvis AI model and LLM provider management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/ai_models" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/ai_models` | List all AI models |
| GET | `/ai_models/:id` | Get a single AI model |
| POST | `/ai_models` | Create an AI model |
| PUT | `/ai_models/:id` | Update an AI model |
| DELETE | `/ai_models/:id` | Delete an AI model |
| GET | `/llm_providers` | List all LLM providers |
| POST | `/llm_providers` | Create an LLM provider |
| PUT | `/llm_providers/:id` | Update an LLM provider |
| DELETE | `/llm_providers/:id` | Delete an LLM provider |
| POST | `/llm_providers/:id/validate` | Validate a provider connection |
| GET | `/llm_providers/:id/vendor_models` | List vendor models |

---

## Data Models

### AIModel
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Model configuration name |
| `model_id` | string | Vendor model identifier |
| `provider_id` | uuid | LLM provider ID |
| `provider_name` | string | Provider display name |
| `max_tokens` | integer | Maximum tokens |
| `temperature` | float | Temperature setting |
| `top_p` | float | Top-p sampling |
| `frequency_penalty` | float | Frequency penalty |
| `presence_penalty` | float | Presence penalty |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### LLMProvider
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Provider name |
| `vendor` | string | Vendor (openai, anthropic, google) |
| `api_endpoint` | string | API endpoint URL |
| `status` | enum | active, inactive |
| `models_count` | integer | Number of configured models |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Model Not Found",
    "detail": "The requested AI model could not be found."
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
// AIModelsService — client.jarvis.aiModels
list(params?: ListAIModelsParams): Promise<PageResult<AIModel>>;
get(uniqueId: string): Promise<AIModel>;
create(data: CreateAIModelRequest): Promise<AIModel>;
update(uniqueId: string, data: UpdateAIModelRequest): Promise<AIModel>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  AIModel,
  CreateAIModelRequest,
  UpdateAIModelRequest,
  ListAIModelsParams,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();
  const result = await client.jarvis.aiModels.list();
}
```
