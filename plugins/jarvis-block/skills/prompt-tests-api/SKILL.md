---
name: jarvis-prompt-tests-api
description: Manage 23blocks Jarvis prompt tests via REST API. Use when creating prompt test cases, running tests, viewing evaluations, or comparing prompt version performance.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Prompt Tests API

Complete API reference for 23blocks Jarvis prompt testing with evaluations and version comparison.

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
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /prompts/:id/tests - List Prompt Tests

Lists all test cases for a prompt.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "test-uuid-456",
      "type": "prompt_test",
      "attributes": {
        "unique_id": "test-uuid-456",
        "name": "Formal Tone Test",
        "description": "Tests email generation with formal tone",
        "variables": {"tone": "formal", "topic": "budget review"},
        "expected_output": "Should include formal salutation and closing",
        "status": "passed",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /prompts/:id/tests/:test_id - Get Prompt Test

Retrieves a single test case.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/test-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "test-uuid-456",
    "type": "prompt_test",
    "attributes": {
      "unique_id": "test-uuid-456",
      "name": "Formal Tone Test",
      "description": "Tests email generation with formal tone",
      "variables": {"tone": "formal", "topic": "budget review"},
      "expected_output": "Should include formal salutation and closing",
      "evaluation_criteria": "Must contain Dear/Sincerely and formal language",
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

### POST /prompts/:id/tests - Create Prompt Test

Creates a new test case for a prompt.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "test": {
      "name": "Formal Tone Test",
      "description": "Tests email generation with formal tone",
      "variables": {"tone": "formal", "topic": "budget review"},
      "expected_output": "Should include formal salutation and closing",
      "evaluation_criteria": "Must contain Dear/Sincerely and formal language"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Test name |
| `description` | string | No | Test description |
| `variables` | object | Yes | Variable values for the test |
| `expected_output` | string | No | Expected response |
| `evaluation_criteria` | string | No | Pass/fail criteria |

**Response 201:**
```json
{
  "data": {
    "id": "test-uuid-456",
    "type": "prompt_test",
    "attributes": {
      "unique_id": "test-uuid-456",
      "name": "Formal Tone Test",
      "variables": {"tone": "formal", "topic": "budget review"},
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /prompts/:id/tests/:test_id - Update Prompt Test

Updates an existing test case.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/test-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "test": {
      "evaluation_criteria": "Must contain formal greeting, body, and sign-off"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "test-uuid-456",
    "type": "prompt_test",
    "attributes": {
      "unique_id": "test-uuid-456",
      "evaluation_criteria": "Must contain formal greeting, body, and sign-off",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /prompts/:id/tests/:test_id - Delete Prompt Test

Deletes a test case.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/test-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Test Execution

### GET /prompts/:id/tests/:test_id/results - Get Test Results

Retrieves results for a specific test.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/test-uuid-456/results" \
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
        "version_id": "version-uuid-1",
        "status": "passed",
        "actual_output": "Dear Board Members,\n\nI am writing to present the budget review...\n\nSincerely,",
        "score": 0.95,
        "tokens_used": 280,
        "duration_ms": 1800,
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /prompts/:id/tests/:test_id/run - Run Single Test

Executes a single prompt test.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/test-uuid-456/run" \
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
      "actual_output": "Dear Board Members,\n\nI am writing to present...",
      "score": 0.95,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /prompts/:id/tests/run_all - Run All Prompt Tests

Executes all test cases for a prompt.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/run_all" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "type": "test_suite_result",
    "attributes": {
      "total_tests": 8,
      "passed": 7,
      "failed": 1,
      "average_score": 0.89,
      "total_tokens_used": 2200,
      "total_duration_ms": 15000,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Evaluations & Comparison

### GET /prompts/:id/tests/evaluations - List Evaluations

Lists evaluation results across all tests.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/evaluations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "eval-uuid-101",
      "type": "evaluation",
      "attributes": {
        "unique_id": "eval-uuid-101",
        "version_id": "version-uuid-1",
        "total_tests": 8,
        "passed": 7,
        "failed": 1,
        "average_score": 0.89,
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /prompts/:id/tests/compare - Compare Versions

Compares test results between two prompt versions.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/prompts/prompt-uuid-123/tests/compare" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "version_a": "version-uuid-1",
    "version_b": "version-uuid-2"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `version_a` | uuid | Yes | First version to compare |
| `version_b` | uuid | Yes | Second version to compare |

**Response 200:**
```json
{
  "data": {
    "type": "version_comparison",
    "attributes": {
      "version_a": {
        "version_id": "version-uuid-1",
        "average_score": 0.89,
        "passed": 7,
        "failed": 1
      },
      "version_b": {
        "version_id": "version-uuid-2",
        "average_score": 0.93,
        "passed": 8,
        "failed": 0
      },
      "winner": "version-uuid-2",
      "score_improvement": 0.04
    }
  }
}
```

---

## Data Models

### PromptTest
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Test name |
| `description` | string | Test description |
| `variables` | object | Variable values for the test |
| `expected_output` | string | Expected response |
| `evaluation_criteria` | string | Pass/fail criteria |
| `status` | enum | pending, passed, failed |
| `last_run_at` | timestamp | Last execution time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Test Not Found",
    "detail": "The requested prompt test could not be found."
  }]
}
```
