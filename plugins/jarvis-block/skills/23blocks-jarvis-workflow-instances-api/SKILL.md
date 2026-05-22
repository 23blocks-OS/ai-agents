---
name: 23blocks-jarvis-workflow-instances-api
description: Manage 23blocks Jarvis workflow instances via REST API. Use when starting workflow instances, advancing steps, executing steps, managing participants, or monitoring workflow runtime execution.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Workflow Instances API

Complete API reference for 23blocks Jarvis workflow runtime execution with instances, step execution, and participants.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://jarvis.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Instance Not Found",
    "detail": "The requested workflow instance could not be found."
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
