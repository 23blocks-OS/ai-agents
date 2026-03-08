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

**BEFORE making ANY API call**, verify these environment variables are set:

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Jarvis API base URL | `https://jarvis.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid/threads" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

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
