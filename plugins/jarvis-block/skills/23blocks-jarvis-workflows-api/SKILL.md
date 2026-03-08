---
name: 23blocks-jarvis-workflows-api
description: Manage 23blocks Jarvis workflows via REST API. Use when creating BPMN workflows, defining workflow steps, or assigning prompts and agents to workflow steps.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Workflows API

Complete API reference for 23blocks Jarvis workflow management with steps, prompt assignments, and agent assignments.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

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

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/workflows` | List all workflows |
| GET | `/workflows/:id` | Get a single workflow |
| POST | `/workflows` | Create a workflow |
| PUT | `/workflows/:id` | Update a workflow |
| DELETE | `/workflows/:id` | Delete a workflow |
| GET | `/workflows/:id/steps` | List workflow steps |
| GET | `/workflows/:id/steps/:step_id` | Get a workflow step |
| POST | `/workflows/:id/steps` | Create a workflow step |
| PUT | `/workflows/:id/steps/:step_id` | Update a workflow step |
| DELETE | `/workflows/:id/steps/:step_id` | Delete a workflow step |
| POST | `/workflows/:id/steps/:step_id/prompts/:prompt_id` | Add prompt to step |
| DELETE | `/workflows/:id/steps/:step_id/prompts/:prompt_id` | Remove prompt from step |
| POST | `/workflows/:id/steps/:step_id/agents/:agent_id` | Add agent to step |
| DELETE | `/workflows/:id/steps/:step_id/agents/:agent_id` | Remove agent from step |

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
