# Workflow Instances API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

### POST /workflows/:id/instances - Start Instance

Starts a new workflow instance.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "instance": {
      "input": {
        "content": "Article text to review...",
        "author": "John Doe"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input` | object | Yes | Input data for the workflow |

**Response 201:**
```json
{
  "data": {
    "id": "inst-uuid-789",
    "type": "workflow_instance",
    "attributes": {
      "unique_id": "inst-uuid-789",
      "workflow_id": "workflow-uuid-123",
      "status": "running",
      "current_step_id": "step-uuid-1",
      "input": {"content": "Article text to review...", "author": "John Doe"},
      "started_at": "2025-01-12T10:30:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /workflows/:id/instances/:inst_id - Get Instance

Retrieves a workflow instance.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "inst-uuid-789",
    "type": "workflow_instance",
    "attributes": {
      "unique_id": "inst-uuid-789",
      "workflow_id": "workflow-uuid-123",
      "status": "running",
      "current_step_id": "step-uuid-2",
      "input": {"content": "Article text to review..."},
      "output": null,
      "steps_completed": 1,
      "steps_total": 4,
      "started_at": "2025-01-12T10:30:00Z",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Instance not found

---

### POST /workflows/:id/instances/:inst_id/step - Advance Step

Advances the workflow instance to the next step.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/step" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "step_output": {
      "quality_score": 0.92,
      "feedback": "Content meets quality standards"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "inst-uuid-789",
    "type": "workflow_instance",
    "attributes": {
      "unique_id": "inst-uuid-789",
      "status": "running",
      "current_step_id": "step-uuid-3",
      "steps_completed": 2,
      "updated_at": "2025-01-12T11:00:00Z"
    }
  }
}
```

---

### GET /workflows/:id/instances/:inst_id/log - Get Execution Log

Retrieves the execution log for an instance.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/log" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "log-uuid-1",
      "type": "execution_log",
      "attributes": {
        "step_id": "step-uuid-1",
        "step_name": "Draft Review",
        "status": "completed",
        "input": {"content": "Article text..."},
        "output": {"quality_score": 0.92},
        "duration_ms": 3500,
        "started_at": "2025-01-12T10:30:00Z",
        "completed_at": "2025-01-12T10:30:04Z"
      }
    }
  ]
}
```

---

### PUT /workflows/:id/instances/:inst_id/suspend - Suspend Instance

Suspends a running workflow instance.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/suspend" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "inst-uuid-789",
    "type": "workflow_instance",
    "attributes": {
      "unique_id": "inst-uuid-789",
      "status": "suspended",
      "updated_at": "2025-01-12T12:00:00Z"
    }
  }
}
```

---

### PUT /workflows/:id/instances/:inst_id/resume - Resume Instance

Resumes a suspended workflow instance.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/resume" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "inst-uuid-789",
    "type": "workflow_instance",
    "attributes": {
      "unique_id": "inst-uuid-789",
      "status": "running",
      "updated_at": "2025-01-12T13:00:00Z"
    }
  }
}
```

---

### POST /workflows/:id/instances/:inst_id/steps/:step_id/execute - Execute Step

Executes a specific step in the workflow instance.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/steps/step-uuid-2/execute" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "content": "Content to process in this step"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "type": "step_execution",
    "attributes": {
      "step_id": "step-uuid-2",
      "status": "completed",
      "output": {
        "result": "Step processed successfully",
        "score": 0.88
      },
      "duration_ms": 2800,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /workflows/:id/instances/:inst_id/next_steps - Get Next Steps

Retrieves the available next steps for the current instance state.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/next_steps" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "step-uuid-3",
      "type": "workflow_step",
      "attributes": {
        "unique_id": "step-uuid-3",
        "name": "Final Review",
        "step_type": "human_review",
        "position": 3,
        "transition_label": "Quality Passed"
      }
    }
  ]
}
```

---

## Instance Participants

### GET /workflows/:id/instances/:inst_id/participants - List Participants

Lists all participants in a workflow instance.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/participants" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "part-uuid-101",
      "type": "participant",
      "attributes": {
        "unique_id": "part-uuid-101",
        "user_id": "user-uuid-456",
        "display_name": "Jane Smith",
        "role": "reviewer",
        "assigned_step_id": "step-uuid-2",
        "joined_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /workflows/:id/instances/:inst_id/participants - Add Participant

Adds a participant to a workflow instance.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/participants" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "participant": {
      "user_id": "user-uuid-456",
      "role": "reviewer",
      "assigned_step_id": "step-uuid-2"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | uuid | Yes | User ID |
| `role` | string | Yes | Participant role |
| `assigned_step_id` | uuid | No | Step assignment |

**Response 201:**
```json
{
  "data": {
    "id": "part-uuid-101",
    "type": "participant",
    "attributes": {
      "unique_id": "part-uuid-101",
      "user_id": "user-uuid-456",
      "role": "reviewer",
      "assigned_step_id": "step-uuid-2",
      "joined_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /workflows/:id/instances/:inst_id/participants/:part_id - Remove Participant

Removes a participant from a workflow instance.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/workflows/workflow-uuid-123/instances/inst-uuid-789/participants/part-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content
