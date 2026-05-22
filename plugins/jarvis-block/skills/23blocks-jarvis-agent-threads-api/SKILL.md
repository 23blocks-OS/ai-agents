---
name: 23blocks-jarvis-agent-threads-api
description: Manage 23blocks Jarvis agent threads, messages, and runs via REST API. Use when creating agent conversation threads, sending messages, streaming responses, or managing agent execution runs.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Agent Threads API

Complete API reference for 23blocks Jarvis agent runtime — threads, messages, streaming, runs, and executions.

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
| GET | `/agents/:id/context` | Get agent context |
| GET | `/agents/:id/threads` | List agent threads |
| GET | `/agents/:id/threads/:thread_id` | Get a thread |
| POST | `/agents/:id/threads` | Create a thread |
| DELETE | `/agents/:id/threads/:thread_id` | Delete a thread |
| GET | `/agents/:id/threads/:thread_id/messages` | List thread messages |
| POST | `/agents/:id/threads/:thread_id/messages` | Send a message |
| POST | `/agents/:id/threads/:thread_id/messages/stream` | Stream a message |
| POST | `/agents/:id/threads/:thread_id/runs` | Create a run |
| GET | `/agents/:id/threads/:thread_id/runs` | List runs |
| GET | `/agents/:id/threads/:thread_id/runs/:run_id` | Get a run |
| GET | `/agents/:id/threads/:thread_id/runs/:run_id/executions` | List run executions |

---

## Context Creation Behavior

When creating contexts (via `GET /agents/:id/context` or context creation endpoints), if no `members` array is provided, Jarvis auto-populates it from the JWT token (user_unique_id + user_email).

The `members` parameter (array, optional) can be explicitly passed during context creation to override this default behavior.

---

## Data Models

### Thread
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Thread title |
| `messages_count` | integer | Number of messages |
| `runs_count` | integer | Number of runs |
| `status` | enum | active, archived |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Run
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `status` | enum | queued, running, completed, failed |
| `input_message` | string | User input |
| `output_message` | string | Agent response |
| `tokens_used` | integer | Total tokens consumed |
| `duration_ms` | integer | Execution time in ms |
| `model` | string | LLM model used |
| `created_at` | timestamp | Creation time |
| `completed_at` | timestamp | Completion time |

### Execution
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `step` | string | Execution step type |
| `status` | enum | pending, running, completed, failed |
| `input` | string | Step input |
| `output` | string | Step output |
| `tokens_used` | integer | Tokens consumed |
| `duration_ms` | integer | Step duration in ms |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Thread Not Found",
    "detail": "The requested thread could not be found."
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
// AgentRuntimeService — client.jarvis.agentRuntime
getContext(agentUniqueId: string, contextUniqueId: string): Promise<AgentContext>;
createContext(agentUniqueId: string, data?: CreateAgentContextRequest): Promise<AgentContext>;
getConversation(agentUniqueId: string, contextUniqueId: string): Promise<{ messages: AgentMessage[] }>;
getThread(agentUniqueId: string, threadId: string): Promise<AgentThread>;
createThread(agentUniqueId: string, data?: CreateAgentThreadRequest): Promise<AgentThread>;
sendMessage(agentUniqueId: string, threadId: string, data: SendAgentMessageRequest): Promise<unknown>;
sendMessageStream(agentUniqueId: string, threadId: string, data: SendAgentMessageRequest): Promise<ReadableStream<string>>;
getMessages(agentUniqueId: string, threadId: string): Promise<AgentMessage[]>;
listExecutions(agentUniqueId: string, params?: ListAgentRunExecutionsParams): Promise<PageResult<AgentRunExecution>>;
getExecution(agentUniqueId: string, executionUniqueId: string): Promise<AgentRunExecution>;
```

### TypeScript Types

```typescript
import type {
  AgentThread,
  AgentMessage,
  AgentMessageContent,
  AgentContext,
  CreateAgentThreadRequest,
  CreateAgentContextRequest,
  SendAgentMessageRequest,
  AgentRunExecution,
  ListAgentRunExecutionsParams,
} from '@23blocks/block-jarvis';
```

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Example: create a thread and send a message
  const thread = await client.jarvis.agentRuntime.createThread('agent-uuid');
  const response = await client.jarvis.agentRuntime.sendMessage('agent-uuid', thread.threadId, { content: 'Hello!' });
}
```
