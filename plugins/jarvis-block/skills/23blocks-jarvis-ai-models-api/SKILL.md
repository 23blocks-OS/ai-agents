---
name: 23blocks-jarvis-ai-models-api
description: "Jarvis AI models and LLM providers: configure, validate, list vendor models. Use when connecting an LLM."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
---

# AI Models API

Complete API reference for 23blocks Jarvis AI model and LLM provider management.

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
| GET | `/vendors/:vendor/models` | Discover models by vendor name |

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
| `input_token_cost_currency` | string | Currency for input token pricing (e.g., `USD`) |
| `output_token_cost_currency` | string | Currency for output token pricing (e.g., `USD`) |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### LLMProvider
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Provider name |
| `vendor` | string | Vendor: `openai`, `anthropic`, `google`, `mistral` (alias: `mistralai`), `perplexity`, `openai_compatible`, `custom` |
| `api_endpoint` | string | API endpoint URL |
| `status` | enum | active, inactive |
| `models_count` | integer | Number of configured models |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Supported Providers

| Vendor | Accepted Aliases (any case) | Base URL | Default Model | Streaming |
|--------|------------------------------|----------|---------------|-----------|
| `openai` | `gpt`, `openai-gpt`, `openai-gpt4` | `https://api.openai.com/v1` | `gpt-4` | Yes |
| `anthropic` | `claude` | `https://api.anthropic.com` | `claude-sonnet-4-5-20241022` | Yes |
| `google` | `gemini`, `google-gemini`, `google-ai` | `https://generativelanguage.googleapis.com` | `gemini-pro` | Yes |
| `mistral` | `mistralai`, `mistral-ai` | `https://api.mistral.ai` | `mistral-small-latest` | Yes |
| `perplexity` | — | `https://api.perplexity.ai` | `pplx-7b-online` | Yes |
| `openai_compatible` | `openai-compatible`, `custom` | Custom | Varies | Yes |

> Provider names are auto-normalized on CompanyKey creation. All aliases resolve to their canonical uppercase form (e.g. `claude` → `ANTHROPIC`, `gpt` → `OPENAI`).

### Mistral Models

`mistral-large-latest`, `mistral-small-latest`, `ministral-3b-latest`, `ministral-8b-latest`, `open-mistral-nemo`, `codestral-latest`, `pixtral-large-latest`, `pixtral-12b-2409`

> Mistral uses an OpenAI-compatible API (`/v1/chat/completions`). Custom base URLs are supported for self-hosted deployments (Ollama, vLLM) with dynamic model validation fallback.

### Vendor Model Discovery

Use `GET /llm_providers/:id/vendor_models` or `GET /vendors/:vendor/models` to list available models for any provider.

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Model Not Found","detail":"The requested AI model could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useJarvisBlock()` from `@23blocks/react`.

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
