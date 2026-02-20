---
name: jarvis-agent-tests-api
description: Manage 23blocks Jarvis agent tests via REST API. Use when creating agent test cases, running tests, viewing test results, or running all tests for an agent.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Agent Tests API

Complete API reference for 23blocks Jarvis agent testing framework with test cases, execution, and results.

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
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /agents/:id/tests - List Agent Tests

Lists all test cases for an agent.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "test-uuid-456",
      "type": "agent_test",
      "attributes": {
        "unique_id": "test-uuid-456",
        "name": "Password Reset Test",
        "description": "Tests agent response to password reset inquiries",
        "input": "How do I reset my password?",
        "expected_output": "Navigate to Settings > Security",
        "status": "passed",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /agents/:id/tests/:test_id - Get Agent Test

Retrieves a single test case.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/tests/test-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "test-uuid-456",
    "type": "agent_test",
    "attributes": {
      "unique_id": "test-uuid-456",
      "name": "Password Reset Test",
      "description": "Tests agent response to password reset inquiries",
      "input": "How do I reset my password?",
      "expected_output": "Navigate to Settings > Security",
      "evaluation_criteria": "Response must mention Settings and Security sections",
      "status": "passed",
      "last_run_at": "2025-01-12T10:30:00Z",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Test not found

---

### POST /agents/:id/tests - Create Agent Test

Creates a new test case for an agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "test": {
      "name": "Password Reset Test",
      "description": "Tests agent response to password reset inquiries",
      "input": "How do I reset my password?",
      "expected_output": "Navigate to Settings > Security",
      "evaluation_criteria": "Response must mention Settings and Security sections"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Test name |
| `description` | string | No | Test description |
| `input` | string | Yes | Test input message |
| `expected_output` | string | No | Expected agent response |
| `evaluation_criteria` | string | No | Criteria for pass/fail |

**Response 201:**
```json
{
  "data": {
    "id": "test-uuid-456",
    "type": "agent_test",
    "attributes": {
      "unique_id": "test-uuid-456",
      "name": "Password Reset Test",
      "input": "How do I reset my password?",
      "expected_output": "Navigate to Settings > Security",
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /agents/:id/tests/:test_id - Update Agent Test

Updates an existing test case.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/agents/agent-uuid-123/tests/test-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "test": {
      "expected_output": "Go to Settings > Security > Reset Password",
      "evaluation_criteria": "Must include exact navigation path"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "test-uuid-456",
    "type": "agent_test",
    "attributes": {
      "unique_id": "test-uuid-456",
      "name": "Password Reset Test",
      "expected_output": "Go to Settings > Security > Reset Password",
      "evaluation_criteria": "Must include exact navigation path",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /agents/:id/tests/:test_id - Delete Agent Test

Deletes a test case.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/agents/agent-uuid-123/tests/test-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Test Execution

### GET /agents/:id/tests/:test_id/results - Get Test Results

Retrieves results for a specific test.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/agents/agent-uuid-123/tests/test-uuid-456/results" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "result-uuid-789",
      "type": "test_result",
      "attributes": {
        "unique_id": "result-uuid-789",
        "status": "passed",
        "actual_output": "To reset your password, go to Settings > Security > Reset Password.",
        "score": 0.92,
        "tokens_used": 180,
        "duration_ms": 1100,
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /agents/:id/tests/:test_id/run - Run Single Test

Executes a single test case.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/tests/test-uuid-456/run" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "result-uuid-789",
    "type": "test_result",
    "attributes": {
      "unique_id": "result-uuid-789",
      "test_id": "test-uuid-456",
      "status": "passed",
      "actual_output": "To reset your password, go to Settings > Security > Reset Password.",
      "score": 0.92,
      "tokens_used": 180,
      "duration_ms": 1100,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /agents/:id/tests/run_all - Run All Agent Tests

Executes all test cases for an agent.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/agents/agent-uuid-123/tests/run_all" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "test_suite_result",
    "attributes": {
      "total_tests": 10,
      "passed": 8,
      "failed": 2,
      "average_score": 0.87,
      "total_tokens_used": 1850,
      "total_duration_ms": 12500,
      "results": [
        {
          "test_id": "test-uuid-456",
          "name": "Password Reset Test",
          "status": "passed",
          "score": 0.92
        },
        {
          "test_id": "test-uuid-457",
          "name": "Billing Inquiry Test",
          "status": "failed",
          "score": 0.45
        }
      ],
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Data Models

### AgentTest
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Test name |
| `description` | string | Test description |
| `input` | string | Test input message |
| `expected_output` | string | Expected response |
| `evaluation_criteria` | string | Pass/fail criteria |
| `status` | enum | pending, passed, failed |
| `last_run_at` | timestamp | Last execution time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### TestResult
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `test_id` | uuid | Parent test ID |
| `status` | enum | passed, failed |
| `actual_output` | string | Agent's actual response |
| `score` | float | Evaluation score (0-1) |
| `tokens_used` | integer | Tokens consumed |
| `duration_ms` | integer | Execution duration |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Test Not Found",
    "detail": "The requested test could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

**Note:** The Agent Tests API does not have a dedicated SDK service yet. Use raw API calls as documented above, or use the generic HTTP client from the SDK.

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

### React Hook

```typescript
import { useJarvisBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useJarvisBlock();

  // Agent tests are managed via REST API calls.
  // Use the HTTP client or fetch with the environment variables.
}
```
