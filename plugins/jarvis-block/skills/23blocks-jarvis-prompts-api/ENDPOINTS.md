# Prompts API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

## Contents

- GET `/prompts`: List Prompts
- GET `/prompts/:id`: Get Prompt
- POST `/prompts`: Create Prompt
- PUT `/prompts/:id`: Update Prompt
- DELETE `/prompts/:id`: Delete Prompt
- POST `/prompts/:id/render`: Render Prompt
- PUT `/prompts/:id/like`: Like Prompt
- DELETE `/prompts/:id/dislike`: Remove Like
- PUT `/prompts/:id/follow`: Follow Prompt
- DELETE `/prompts/:id/unfollow`: Unfollow Prompt
- PUT `/prompts/:id/save`: Save Prompt
- DELETE `/prompts/:id/unsave`: Unsave Prompt

### GET /prompts - List Prompts

Lists all prompts with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "id": "prompt-uuid-123",
      "type": "prompt",
      "attributes": {
        "unique_id": "prompt-uuid-123",
        "name": "Email Generator",
        "description": "Generates professional emails",
        "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
        "status": "published",
        "likes_count": 15,
        "saves_count": 8,
        "versions_count": 3,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /prompts/:id - Get Prompt

Retrieves a single prompt by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "prompt-uuid-123",
    "type": "prompt",
    "attributes": {
      "unique_id": "prompt-uuid-123",
      "name": "Email Generator",
      "description": "Generates professional emails",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
      "status": "published",
      "likes_count": 15,
      "saves_count": 8,
      "versions_count": 3,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "versions": {
        "data": [
          { "id": "version-uuid-1", "type": "prompt_version" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Prompt not found

---

### POST /prompts - Create Prompt

Creates a new prompt.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "name": "Email Generator",
      "description": "Generates professional emails based on tone and topic",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
      "provider": "anthropic",
      "model": "claude-sonnet-4-5-20241022"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Prompt name |
| `description` | string | No | Prompt description |
| `content` | string | Yes | Prompt template with {{variables}} |
| `provider` | string | No | LLM provider: `openai` (default), `anthropic`, `google`, `mistral`, `perplexity`, `openai_compatible`, `custom` |
| `model` | string | No | Model identifier for the provider (e.g., `gpt-4`, `claude-sonnet-4-5-20241022`, `mistral-small-latest`) |

**Response 201:**

> Returns a `PromptVersion` object, not a `Prompt`.

```json
{
  "data": {
    "id": "version-uuid-001",
    "type": "prompt_version",
    "attributes": {
      "unique_id": "version-uuid-001",
      "name": "Email Generator",
      "description": "Generates professional emails based on tone and topic",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}.",
      "status": "draft",
      "version": 1,
      "revision": 0,
      "prompt_unique_id": "prompt-uuid-123",
      "user_unique_id": "user-uuid-456",
      "user_name": "John Doe",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /prompts/:id - Update Prompt

Updates an existing prompt. Only the fields included in the request body are updated; omitted fields are preserved (e.g., persona, model, temperature are no longer wiped when not passed).

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": {
      "name": "Advanced Email Generator",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}. Include a {{call_to_action}}."
    }
  }'
```

**Response 200:**

> Returns a `PromptVersion` object, not a `Prompt`.

```json
{
  "data": {
    "id": "version-uuid-002",
    "type": "prompt_version",
    "attributes": {
      "unique_id": "version-uuid-002",
      "name": "Advanced Email Generator",
      "content": "Write a {{tone}} email about {{topic}} to {{recipient}}. Include a {{call_to_action}}.",
      "version": 2,
      "revision": 0,
      "prompt_unique_id": "prompt-uuid-123",
      "user_unique_id": "user-uuid-456",
      "user_name": "John Doe",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /prompts/:id - Delete Prompt

Deletes a prompt.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Prompt not found

> **Required Scopes:** POST, PUT, and DELETE `/prompts` endpoints require the `prompts:write` scope.

---

### POST /prompts/:id/render - Render Prompt

Renders a prompt template by substituting variables.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/render" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "variables": {
      "tone": "professional",
      "topic": "quarterly results",
      "recipient": "the board of directors"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `variables` | object | Yes | Key-value pairs for template variables |

**Response 200:**
```json
{
  "data": {
    "rendered_content": "Write a professional email about quarterly results to the board of directors.",
    "variables_used": ["tone", "topic", "recipient"],
    "missing_variables": []
  }
}
```

---

## Social Actions

### PUT /prompts/:id/like - Like Prompt

Adds a like to the prompt.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt liked successfully"
}
```

---

### DELETE /prompts/:id/dislike - Remove Like

Removes like from the prompt.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Like removed successfully"
}
```

---

### PUT /prompts/:id/follow - Follow Prompt

Follows a prompt to receive updates.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/follow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt followed successfully"
}
```

---

### DELETE /prompts/:id/unfollow - Unfollow Prompt

Unfollows a prompt.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/unfollow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt unfollowed successfully"
}
```

---

### PUT /prompts/:id/save - Save Prompt

Saves/bookmarks a prompt.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/save" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt saved successfully"
}
```

---

### DELETE /prompts/:id/unsave - Unsave Prompt

Removes prompt from saved list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/unsave" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt unsaved successfully"
}
```

---
