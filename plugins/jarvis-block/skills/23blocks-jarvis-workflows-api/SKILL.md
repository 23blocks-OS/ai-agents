---
name: 23blocks-jarvis-workflows-api
description: "Jarvis BPMN workflow definitions: steps, prompts and agents on steps. Use when designing a workflow."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
---

# Workflows API

Complete API reference for 23blocks Jarvis workflow management with steps, prompt assignments, and agent assignments.

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
| `step_type` | string | Step type: `task`, `gateway`, `event`, etc. |
| `gateway_type` | string | Gateway type: `exclusive`, `parallel`, `inclusive`, `event_based` (only for gateway steps) |
| `is_entry_point` | boolean | Whether this step is the workflow entry point |
| `is_exit_point` | boolean | Whether this step is the workflow exit point |
| `position` | integer | Step position |
| `description` | string | Step description |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Workflow Not Found","detail":"The requested workflow could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useJarvisBlock()` from `@23blocks/react`.

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
