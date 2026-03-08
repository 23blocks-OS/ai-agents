# Prompt Comments API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Prompt Comments

### GET /prompts/:id/comments - List Prompt Comments

Lists all comments on a prompt.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "comment-uuid-456",
      "type": "comment",
      "attributes": {
        "unique_id": "comment-uuid-456",
        "body": "Great prompt! Works well for sales emails.",
        "author_id": "user-uuid-789",
        "author_name": "Jane Smith",
        "likes_count": 5,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /prompts/:id/comments/:comment_id - Get Comment

Retrieves a single prompt comment.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Great prompt! Works well for sales emails.",
      "author_id": "user-uuid-789",
      "author_name": "Jane Smith",
      "likes_count": 5,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Comment not found

---

### POST /prompts/:id/comments - Create Comment

Adds a comment to a prompt.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Great prompt! Works well for sales emails."
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `body` | string | Yes | Comment text |

**Response 201:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Great prompt! Works well for sales emails.",
      "likes_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /prompts/:id/comments/:comment_id - Update Comment

Updates an existing comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Updated: Great prompt! Especially effective for formal sales emails."
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Updated: Great prompt! Especially effective for formal sales emails.",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /prompts/:id/comments/:comment_id - Delete Comment

Deletes a comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /prompts/:id/comments/:comment_id/like - Like Comment

Likes a prompt comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment liked successfully"
}
```

---

### DELETE /prompts/:id/comments/:comment_id/dislike - Remove Like

Removes like from a prompt comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/comments/comment-uuid-456/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Like removed successfully"
}
```

---

## Execution Comments

### GET /prompts/:id/executions/:exec_id/comments - List Execution Comments

Lists all comments on a prompt execution.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "comment-uuid-101",
      "type": "comment",
      "attributes": {
        "unique_id": "comment-uuid-101",
        "body": "This execution produced excellent results.",
        "author_id": "user-uuid-789",
        "author_name": "Jane Smith",
        "likes_count": 3,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /prompts/:id/executions/:exec_id/comments - Create Execution Comment

Adds a comment to a prompt execution.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "This execution produced excellent results."
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "comment-uuid-101",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-101",
      "body": "This execution produced excellent results.",
      "likes_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /prompts/:id/executions/:exec_id/comments/:comment_id - Update Execution Comment

Updates an execution comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Updated: Excellent results, especially the tone."
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-101",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-101",
      "body": "Updated: Excellent results, especially the tone.",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /prompts/:id/executions/:exec_id/comments/:comment_id - Delete Execution Comment

Deletes an execution comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /prompts/:id/executions/:exec_id/comments/:comment_id/like - Like Execution Comment

Likes an execution comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment liked successfully"
}
```

---

### DELETE /prompts/:id/executions/:exec_id/comments/:comment_id/dislike - Remove Like

Removes like from an execution comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/executions/exec-uuid-789/comments/comment-uuid-101/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Like removed successfully"
}
```
