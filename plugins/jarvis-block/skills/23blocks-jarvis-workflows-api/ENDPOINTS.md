# Workflows API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

### GET /workflows - List Workflows

Lists all workflows with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows?page=1&records=20" \
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
      "id": "workflow-uuid-123",
      "type": "workflow",
      "attributes": {
        "unique_id": "workflow-uuid-123",
        "name": "Content Review Pipeline",
        "description": "Multi-step content review process",
        "status": "active",
        "steps_count": 4,
        "instances_count": 15,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### GET /workflows/:id - Get Workflow

Retrieves a single workflow by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "workflow-uuid-123",
    "type": "workflow",
    "attributes": {
      "unique_id": "workflow-uuid-123",
      "name": "Content Review Pipeline",
      "description": "Multi-step content review process",
      "status": "active",
      "steps_count": 4,
      "instances_count": 15,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "steps": {
        "data": [
          { "id": "step-uuid-1", "type": "workflow_step" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Workflow not found

---

### POST /workflows - Create Workflow

Creates a new workflow.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workflow": {
      "name": "Content Review Pipeline",
      "description": "Multi-step content review process"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Workflow name |
| `description` | string | No | Workflow description |

**Response 201:**
```json
{
  "data": {
    "id": "workflow-uuid-123",
    "type": "workflow",
    "attributes": {
      "unique_id": "workflow-uuid-123",
      "name": "Content Review Pipeline",
      "description": "Multi-step content review process",
      "status": "draft",
      "steps_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /workflows/:id - Update Workflow

Updates an existing workflow.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/workflows/workflow-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "workflow": {
      "name": "Advanced Content Review",
      "status": "active"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "workflow-uuid-123",
    "type": "workflow",
    "attributes": {
      "unique_id": "workflow-uuid-123",
      "name": "Advanced Content Review",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /workflows/:id - Delete Workflow

Deletes a workflow.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/workflows/workflow-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Workflow Steps

### GET /workflows/:id/steps - List Steps

Lists all steps in a workflow.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "step-uuid-1",
      "type": "workflow_step",
      "attributes": {
        "unique_id": "step-uuid-1",
        "name": "Draft Review",
        "step_type": "ai_review",
        "position": 1,
        "description": "AI reviews the draft for quality",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "step-uuid-2",
      "type": "workflow_step",
      "attributes": {
        "unique_id": "step-uuid-2",
        "name": "Manager Approval",
        "step_type": "human_review",
        "position": 2,
        "description": "Manager reviews and approves",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /workflows/:id/steps/:step_id - Get Step

Retrieves a single workflow step.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "step-uuid-1",
    "type": "workflow_step",
    "attributes": {
      "unique_id": "step-uuid-1",
      "name": "Draft Review",
      "step_type": "ai_review",
      "position": 1,
      "description": "AI reviews the draft for quality",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "prompts": {
        "data": [{ "id": "prompt-uuid-1", "type": "prompt" }]
      },
      "agents": {
        "data": [{ "id": "agent-uuid-1", "type": "agent" }]
      }
    }
  }
}
```

---

### POST /workflows/:id/steps - Create Step

Creates a new workflow step.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "step": {
      "name": "Draft Review",
      "step_type": "ai_review",
      "position": 1,
      "description": "AI reviews the draft for quality"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Step name |
| `step_type` | string | Yes | Step type (ai_review, human_review, automated) |
| `position` | integer | Yes | Step position in workflow |
| `description` | string | No | Step description |

**Response 201:**
```json
{
  "data": {
    "id": "step-uuid-1",
    "type": "workflow_step",
    "attributes": {
      "unique_id": "step-uuid-1",
      "name": "Draft Review",
      "step_type": "ai_review",
      "position": 1,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /workflows/:id/steps/:step_id - Update Step

Updates a workflow step.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "step": {
      "name": "Enhanced Draft Review",
      "position": 1
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "step-uuid-1",
    "type": "workflow_step",
    "attributes": {
      "unique_id": "step-uuid-1",
      "name": "Enhanced Draft Review",
      "position": 1,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /workflows/:id/steps/:step_id - Delete Step

Deletes a workflow step.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Step Assignments

### POST /workflows/:id/steps/:step_id/prompts/:prompt_id - Add Prompt to Step

Assigns a prompt to a workflow step.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/prompts/prompt-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt added to step successfully"
}
```

---

### DELETE /workflows/:id/steps/:step_id/prompts/:prompt_id - Remove Prompt from Step

Removes a prompt from a workflow step.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/prompts/prompt-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Prompt removed from step successfully"
}
```

---

### POST /workflows/:id/steps/:step_id/agents/:agent_id - Add Agent to Step

Assigns an agent to a workflow step.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/agents/agent-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Agent added to step successfully"
}
```

---

### DELETE /workflows/:id/steps/:step_id/agents/:agent_id - Remove Agent from Step

Removes an agent from a workflow step.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/agents/agent-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Agent removed from step successfully"
}
```
