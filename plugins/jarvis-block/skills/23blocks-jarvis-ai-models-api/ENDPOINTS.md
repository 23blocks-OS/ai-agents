# AI Models API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## AI Models

### GET /ai_models - List AI Models

Lists all configured AI models with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/ai_models?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "model-uuid-123",
      "type": "ai_model",
      "attributes": {
        "unique_id": "model-uuid-123",
        "name": "GPT-4 Production",
        "model_id": "gpt-4",
        "provider_id": "provider-uuid-456",
        "provider_name": "OpenAI",
        "max_tokens": 8192,
        "temperature": 0.7,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 8
  }
}
```

---

### GET /ai_models/:id - Get AI Model

Retrieves a single AI model by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/ai_models/model-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "model-uuid-123",
    "type": "ai_model",
    "attributes": {
      "unique_id": "model-uuid-123",
      "name": "GPT-4 Production",
      "model_id": "gpt-4",
      "provider_id": "provider-uuid-456",
      "provider_name": "OpenAI",
      "max_tokens": 8192,
      "temperature": 0.7,
      "top_p": 1.0,
      "frequency_penalty": 0.0,
      "presence_penalty": 0.0,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Model not found

---

### POST /ai_models - Create AI Model

Creates a new AI model configuration.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/ai_models" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "ai_model": {
      "name": "GPT-4 Production",
      "model_id": "gpt-4",
      "provider_id": "provider-uuid-456",
      "max_tokens": 8192,
      "temperature": 0.7
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Model configuration name |
| `model_id` | string | Yes | Vendor model identifier |
| `provider_id` | uuid | Yes | LLM provider ID |
| `max_tokens` | integer | No | Maximum tokens |
| `temperature` | float | No | Temperature (0-2) |
| `top_p` | float | No | Top-p sampling (0-1) |
| `frequency_penalty` | float | No | Frequency penalty (0-2) |
| `presence_penalty` | float | No | Presence penalty (0-2) |

**Response 201:**
```json
{
  "data": {
    "id": "model-uuid-123",
    "type": "ai_model",
    "attributes": {
      "unique_id": "model-uuid-123",
      "name": "GPT-4 Production",
      "model_id": "gpt-4",
      "provider_id": "provider-uuid-456",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /ai_models/:id - Update AI Model

Updates an AI model configuration.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/ai_models/model-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "ai_model": {
      "temperature": 0.5,
      "max_tokens": 4096
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "model-uuid-123",
    "type": "ai_model",
    "attributes": {
      "unique_id": "model-uuid-123",
      "temperature": 0.5,
      "max_tokens": 4096,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /ai_models/:id - Delete AI Model

Deletes an AI model configuration.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/ai_models/model-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## LLM Providers

### GET /llm_providers - List LLM Providers

Lists all configured LLM providers.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/llm_providers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "provider-uuid-456",
      "type": "llm_provider",
      "attributes": {
        "unique_id": "provider-uuid-456",
        "name": "OpenAI Production",
        "vendor": "openai",
        "api_endpoint": "https://api.openai.com/v1",
        "status": "active",
        "models_count": 3,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /llm_providers - Create LLM Provider

Creates a new LLM provider configuration.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/llm_providers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "llm_provider": {
      "name": "OpenAI Production",
      "vendor": "openai",
      "api_key": "sk-...",
      "api_endpoint": "https://api.openai.com/v1"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Provider name |
| `vendor` | string | Yes | Vendor (openai, anthropic, google, etc.) |
| `api_key` | string | Yes | Provider API key |
| `api_endpoint` | string | No | Custom API endpoint |

**Response 201:**
```json
{
  "data": {
    "id": "provider-uuid-456",
    "type": "llm_provider",
    "attributes": {
      "unique_id": "provider-uuid-456",
      "name": "OpenAI Production",
      "vendor": "openai",
      "api_endpoint": "https://api.openai.com/v1",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /llm_providers/:id - Update LLM Provider

Updates a provider configuration.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/llm_providers/provider-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "llm_provider": {
      "name": "OpenAI Production v2",
      "api_key": "sk-new-key..."
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "provider-uuid-456",
    "type": "llm_provider",
    "attributes": {
      "unique_id": "provider-uuid-456",
      "name": "OpenAI Production v2",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /llm_providers/:id - Delete LLM Provider

Deletes a provider configuration.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/llm_providers/provider-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /llm_providers/:id/validate - Validate Provider

Validates the provider connection and API key.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/llm_providers/provider-uuid-456/validate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "validation_result",
    "attributes": {
      "valid": true,
      "vendor": "openai",
      "message": "Provider connection validated successfully",
      "available_models": ["gpt-4", "gpt-4-turbo", "gpt-3.5-turbo"],
      "validated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /llm_providers/:id/vendor_models - Get Vendor Models

Lists available models from the vendor.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/llm_providers/provider-uuid-456/vendor_models" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "gpt-4",
      "type": "vendor_model",
      "attributes": {
        "model_id": "gpt-4",
        "name": "GPT-4",
        "max_tokens": 8192,
        "supports_streaming": true,
        "supports_function_calling": true
      }
    },
    {
      "id": "gpt-4-turbo",
      "type": "vendor_model",
      "attributes": {
        "model_id": "gpt-4-turbo",
        "name": "GPT-4 Turbo",
        "max_tokens": 128000,
        "supports_streaming": true,
        "supports_function_calling": true
      }
    }
  ]
}
```
