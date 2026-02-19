---
name: jarvis-workflow-bpmn-api
description: Manage 23blocks Jarvis workflow BPMN elements via REST API. Use when creating step conditions, evaluating conditions, or managing step-to-step transitions in workflows.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Workflow BPMN API

Complete API reference for 23blocks Jarvis workflow conditions and step transitions.

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
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid/steps/step-uuid/conditions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Step Conditions

### GET /workflows/:id/steps/:step_id/conditions - List Conditions

Lists all conditions for a workflow step.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/conditions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "cond-uuid-456",
      "type": "condition",
      "attributes": {
        "unique_id": "cond-uuid-456",
        "name": "Quality Score Check",
        "condition_type": "expression",
        "expression": "quality_score >= 0.8",
        "description": "Passes if the quality score meets the threshold",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /workflows/:id/steps/:step_id/conditions - Create Condition

Creates a new condition for a workflow step.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/conditions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "condition": {
      "name": "Quality Score Check",
      "condition_type": "expression",
      "expression": "quality_score >= 0.8",
      "description": "Passes if the quality score meets the threshold"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Condition name |
| `condition_type` | string | Yes | Type (expression, rule, ai_evaluation) |
| `expression` | string | Yes | Condition expression |
| `description` | string | No | Condition description |

**Response 201:**
```json
{
  "data": {
    "id": "cond-uuid-456",
    "type": "condition",
    "attributes": {
      "unique_id": "cond-uuid-456",
      "name": "Quality Score Check",
      "condition_type": "expression",
      "expression": "quality_score >= 0.8",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /workflows/:id/steps/:step_id/conditions/:cond_id - Update Condition

Updates an existing condition.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/conditions/cond-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "condition": {
      "expression": "quality_score >= 0.9",
      "description": "Raised threshold to 0.9"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "cond-uuid-456",
    "type": "condition",
    "attributes": {
      "unique_id": "cond-uuid-456",
      "expression": "quality_score >= 0.9",
      "description": "Raised threshold to 0.9",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /workflows/:id/steps/:step_id/conditions/:cond_id - Delete Condition

Deletes a condition.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/conditions/cond-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### POST /workflows/:id/steps/:step_id/conditions/evaluate - Evaluate Conditions

Evaluates all conditions for a step against provided data.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/steps/step-uuid-1/conditions/evaluate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "data": {
      "quality_score": 0.85,
      "word_count": 500,
      "sentiment": "positive"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `data` | object | Yes | Data to evaluate conditions against |

**Response 200:**
```json
{
  "data": {
    "type": "evaluation_result",
    "attributes": {
      "all_passed": true,
      "results": [
        {
          "condition_id": "cond-uuid-456",
          "name": "Quality Score Check",
          "passed": true,
          "expression": "quality_score >= 0.8",
          "evaluated_value": 0.85
        }
      ]
    }
  }
}
```

---

## Step Transitions

### GET /workflows/:id/transitions - List Transitions

Lists all step transitions in a workflow.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/workflows/workflow-uuid-123/transitions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "trans-uuid-789",
      "type": "transition",
      "attributes": {
        "unique_id": "trans-uuid-789",
        "from_step_id": "step-uuid-1",
        "to_step_id": "step-uuid-2",
        "condition_id": "cond-uuid-456",
        "label": "Quality Passed",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /workflows/:id/transitions - Create Transition

Creates a new step transition.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/workflows/workflow-uuid-123/transitions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transition": {
      "from_step_id": "step-uuid-1",
      "to_step_id": "step-uuid-2",
      "condition_id": "cond-uuid-456",
      "label": "Quality Passed"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `from_step_id` | uuid | Yes | Source step ID |
| `to_step_id` | uuid | Yes | Target step ID |
| `condition_id` | uuid | No | Condition that triggers transition |
| `label` | string | No | Transition label |

**Response 201:**
```json
{
  "data": {
    "id": "trans-uuid-789",
    "type": "transition",
    "attributes": {
      "unique_id": "trans-uuid-789",
      "from_step_id": "step-uuid-1",
      "to_step_id": "step-uuid-2",
      "condition_id": "cond-uuid-456",
      "label": "Quality Passed",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /workflows/:id/transitions/:trans_id - Update Transition

Updates a transition.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/workflows/workflow-uuid-123/transitions/trans-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transition": {
      "label": "High Quality Passed",
      "condition_id": "cond-uuid-new"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "trans-uuid-789",
    "type": "transition",
    "attributes": {
      "unique_id": "trans-uuid-789",
      "label": "High Quality Passed",
      "condition_id": "cond-uuid-new",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /workflows/:id/transitions/:trans_id - Delete Transition

Deletes a transition.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/workflows/workflow-uuid-123/transitions/trans-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### Condition
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Condition name |
| `condition_type` | string | Type (expression, rule, ai_evaluation) |
| `expression` | string | Condition expression |
| `description` | string | Condition description |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Transition
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `from_step_id` | uuid | Source step ID |
| `to_step_id` | uuid | Target step ID |
| `condition_id` | uuid | Triggering condition ID |
| `label` | string | Transition label |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Condition Not Found",
    "detail": "The requested condition could not be found."
  }]
}
```
