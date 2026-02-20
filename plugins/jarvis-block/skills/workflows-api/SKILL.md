---
name: jarvis-workflows-api
description: Manage 23blocks Jarvis workflows via REST API. Use when creating BPMN workflows, defining workflow steps, or assigning prompts and agents to workflow steps.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Workflows API

Complete API reference for 23blocks Jarvis workflow management with steps, prompt assignments, and agent assignments.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://jarvis.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/workflows" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

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

---

## Data Models

### Workflow
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Workflow name |
| `description` | string | Workflow description |
| `status` | enum | draft, active, archived |
| `steps_count` | integer | Number of steps |
| `instances_count` | integer | Number of instances |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### WorkflowStep
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Step name |
| `step_type` | string | Step type |
| `position` | integer | Step position |
| `description` | string | Step description |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Workflow Not Found",
    "detail": "The requested workflow could not be found."
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
// WorkflowsService — client.jarvis.workflows
list(params?: ListWorkflowsParams): Promise<PageResult<Workflow>>;
get(uniqueId: string): Promise<Workflow>;
create(data: CreateWorkflowRequest): Promise<Workflow>;
update(uniqueId: string, data: UpdateWorkflowRequest): Promise<Workflow>;
delete(uniqueId: string): Promise<void>;
addStep(uniqueId: string, data: AddWorkflowStepRequest): Promise<WorkflowStep>;
updateStep(uniqueId: string, stepUniqueId: string, data: UpdateWorkflowStepRequest): Promise<WorkflowStep>;
removeStep(uniqueId: string, stepUniqueId: string): Promise<void>;

// WorkflowStepsService — client.jarvis.workflowSteps
get(workflowUniqueId: string, stepUniqueId: string): Promise<WorkflowStep>;
add(workflowUniqueId: string, data: AddWorkflowStepRequest): Promise<WorkflowStep>;
update(workflowUniqueId: string, stepUniqueId: string, data: UpdateWorkflowStepRequest): Promise<WorkflowStep>;
remove(workflowUniqueId: string, stepUniqueId: string): Promise<void>;
addPrompt(stepUniqueId: string, data: AddStepPromptRequest): Promise<void>;
addAgent(stepUniqueId: string, data: AddStepAgentRequest): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Workflow,
  CreateWorkflowRequest,
  UpdateWorkflowRequest,
  ListWorkflowsParams,
  WorkflowStep,
  AddWorkflowStepRequest,
  UpdateWorkflowStepRequest,
  AddStepPromptRequest,
  AddStepAgentRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: list all workflows
  const result = await client.jarvis.workflows.list();
}
```
