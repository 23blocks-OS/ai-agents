---
name: 23blocks-jarvis-workflow-instances-api
description: "Jarvis workflow runtime: start, advance, execute steps, participants, suspend, resume, logs. Use to run a workflow."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Workflow Instances API

Complete API reference for 23blocks Jarvis workflow runtime execution with instances, step execution, and participants.

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
| POST | `/workflows/:id/instances` | Start a workflow instance |
| GET | `/workflows/:id/instances/:inst_id` | Get an instance |
| POST | `/workflows/:id/instances/:inst_id/step` | Advance to next step |
| GET | `/workflows/:id/instances/:inst_id/log` | Get execution log |
| PUT | `/workflows/:id/instances/:inst_id/suspend` | Suspend an instance |
| PUT | `/workflows/:id/instances/:inst_id/resume` | Resume an instance |
| POST | `/workflows/:id/instances/:inst_id/steps/:step_id/execute` | Execute a specific step |
| GET | `/workflows/:id/instances/:inst_id/next_steps` | Get available next steps |
| GET | `/workflows/:id/instances/:inst_id/participants` | List participants |
| POST | `/workflows/:id/instances/:inst_id/participants` | Add a participant |
| DELETE | `/workflows/:id/instances/:inst_id/participants/:part_id` | Remove a participant |

> **Required Scopes:** POST, PUT, and DELETE `/workflows/:id/instances/:inst_id/participants` endpoints require the `workflows:write` scope.

---

## Data Models

### WorkflowInstance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `workflow_id` | uuid | Parent workflow ID |
| `status` | enum | running, suspended, completed, failed |
| `current_step_id` | uuid | Current active step |
| `input` | object | Workflow input data |
| `output` | object | Workflow output data |
| `steps_completed` | integer | Number of completed steps |
| `steps_total` | integer | Total number of steps |
| `started_at` | timestamp | Start time |
| `completed_at` | timestamp | Completion time |
| `created_at` | timestamp | Creation time |

### Participant
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_id` | uuid | User ID |
| `display_name` | string | User display name |
| `role` | string | Participant role |
| `assigned_step_id` | uuid | Assigned step |
| `joined_at` | timestamp | Join time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Instance Not Found","detail":"The requested workflow instance could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// WorkflowInstancesService — client.jarvis.workflowInstances
start(workflowUniqueId: string, data?: StartWorkflowRequest): Promise<WorkflowInstance>;
get(workflowUniqueId: string, instanceUniqueId: string): Promise<WorkflowInstance>;
getDetails(workflowUniqueId: string, instanceUniqueId: string): Promise<WorkflowInstanceDetails>;
step(workflowUniqueId: string, instanceUniqueId: string, data?: StepWorkflowRequest): Promise<WorkflowInstance>;
logStep(workflowUniqueId: string, instanceUniqueId: string, data: LogWorkflowStepRequest): Promise<WorkflowInstance>;
executeStep(workflowUniqueId: string, instanceUniqueId: string, data?: ExecuteStepRequest): Promise<WorkflowInstance>;
executeNextStep(workflowUniqueId: string, instanceUniqueId: string, data?: ExecuteNextStepRequest): Promise<WorkflowInstance>;
suspend(workflowUniqueId: string, instanceUniqueId: string): Promise<WorkflowInstance>;
resume(workflowUniqueId: string, instanceUniqueId: string): Promise<WorkflowInstance>;
```

### TypeScript Types

```typescript
import type {
  WorkflowInstance,
  WorkflowInstanceDetails,
  WorkflowStepLog,
  WorkflowStepStatus,
  StartWorkflowRequest,
  StepWorkflowRequest,
  LogWorkflowStepRequest,
  ExecuteStepRequest,
  ExecuteNextStepRequest,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: start a workflow instance
  const instance = await client.jarvis.workflowInstances.start('workflow-uuid');
}
```
