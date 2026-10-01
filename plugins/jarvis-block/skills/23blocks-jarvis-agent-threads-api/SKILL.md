---
name: 23blocks-jarvis-agent-threads-api
description: "Jarvis agent runtime: threads, messages, streaming, runs. Use to talk to a Jarvis agent."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Agent Threads API

Complete API reference for 23blocks Jarvis agent runtime — threads, messages, streaming, runs, and executions.

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

> Context `unique_id` must be a valid UUID; other values return 400.

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Thread Not Found","detail":"The requested thread could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-jarvis`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
