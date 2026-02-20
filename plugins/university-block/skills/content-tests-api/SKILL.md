---
name: university-content-tests-api
description: Manage 23blocks University content tests via REST API. Use when creating tests with questions and options, managing test results and solutions, or when students start, answer, and finish knowledge assessments.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Content Tests API

Complete API reference for 23blocks University content test management with questions, options, results, and student test-taking workflows.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://university.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /tests - List Tests

Lists all content tests.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tests?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "test-uuid-001",
      "type": "test",
      "attributes": {
        "unique_id": "test-uuid-001",
        "name": "Midterm Exam",
        "description": "Comprehensive midterm covering all topics",
        "test_type": "exam",
        "time_limit": 120,
        "max_score": 100,
        "passing_score": 60,
        "status": "active"
      }
    }
  ],
  "meta": {
    "total_count": 15,
    "page": 1,
    "records": 20
  }
}
```

---

### GET /tests/:unique_id - Get Test with Questions

Retrieves a single test with its questions.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tests/test-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "test-uuid-001",
    "type": "test",
    "attributes": {
      "unique_id": "test-uuid-001",
      "name": "Midterm Exam",
      "description": "Comprehensive midterm covering all topics",
      "test_type": "exam",
      "time_limit": 120,
      "max_score": 100,
      "passing_score": 60,
      "status": "active"
    },
    "relationships": {
      "questions": {
        "data": [
          { "id": "question-uuid-001", "type": "question" },
          { "id": "question-uuid-002", "type": "question" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Test not found

---

### POST /tests/ - Create Test

Creates a new content test.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tests/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "test": {
      "name": "Midterm Exam",
      "description": "Comprehensive midterm covering all topics",
      "test_type": "exam",
      "time_limit": 120,
      "max_score": 100,
      "passing_score": 60
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Test name |
| `description` | string | No | Test description |
| `test_type` | string | No | Type: `quiz`, `exam`, `midterm`, `final` |
| `time_limit` | integer | No | Time limit in minutes |
| `max_score` | integer | No | Maximum possible score |
| `passing_score` | integer | No | Minimum score to pass |

**Response 201:**
```json
{
  "data": {
    "id": "test-uuid-001",
    "type": "test",
    "attributes": {
      "unique_id": "test-uuid-001",
      "name": "Midterm Exam",
      "description": "Comprehensive midterm covering all topics",
      "test_type": "exam",
      "time_limit": 120,
      "max_score": 100,
      "passing_score": 60,
      "status": "active",
      "created_at": "2025-09-01T10:00:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /tests/:unique_id - Update Test

Updates an existing test.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tests/test-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "test": {
      "time_limit": 150,
      "passing_score": 55
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "test-uuid-001",
    "type": "test",
    "attributes": {
      "unique_id": "test-uuid-001",
      "name": "Midterm Exam",
      "time_limit": 150,
      "passing_score": 55,
      "status": "active"
    }
  }
}
```

---

### GET /tests/:unique_id/questions/:question_id - Get Question

Retrieves a specific question from a test.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tests/test-uuid-001/questions/question-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "question-uuid-001",
    "type": "question",
    "attributes": {
      "unique_id": "question-uuid-001",
      "text": "What is the time complexity of a hash table lookup?",
      "question_type": "multiple_choice",
      "points": 10
    },
    "relationships": {
      "options": {
        "data": [
          { "id": "option-uuid-001", "type": "option" },
          { "id": "option-uuid-002", "type": "option" },
          { "id": "option-uuid-003", "type": "option" },
          { "id": "option-uuid-004", "type": "option" }
        ]
      }
    }
  }
}
```

---

### POST /tests/:unique_id/questions - Create Question

Creates a new question for a test.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tests/test-uuid-001/questions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "question": {
      "text": "What is the time complexity of a hash table lookup?",
      "question_type": "multiple_choice",
      "points": 10
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | Yes | Question text |
| `question_type` | string | No | Type: `multiple_choice`, `true_false`, `short_answer` |
| `points` | integer | No | Points for correct answer |

**Response 201:**
```json
{
  "data": {
    "id": "question-uuid-001",
    "type": "question",
    "attributes": {
      "unique_id": "question-uuid-001",
      "text": "What is the time complexity of a hash table lookup?",
      "question_type": "multiple_choice",
      "points": 10,
      "created_at": "2025-09-01T10:00:00Z"
    }
  }
}
```

---

### PUT /tests/:unique_id/questions/:question_unique_id - Update Question

Updates an existing question.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tests/test-uuid-001/questions/question-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "question": {
      "text": "What is the average time complexity of a hash table lookup?",
      "points": 15
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "question-uuid-001",
    "type": "question",
    "attributes": {
      "unique_id": "question-uuid-001",
      "text": "What is the average time complexity of a hash table lookup?",
      "points": 15
    }
  }
}
```

---

### GET /tests/questions/options - List Options

Lists all available options across test questions.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/tests/questions/options" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "option-uuid-001",
      "type": "option",
      "attributes": {
        "unique_id": "option-uuid-001",
        "text": "O(1)",
        "is_correct": true
      }
    }
  ]
}
```

---

### POST /tests/options - Create Option

Creates a new answer option.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/tests/options" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "option": {
      "text": "O(1)"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | Yes | Option text |

**Response 201:**
```json
{
  "data": {
    "id": "option-uuid-001",
    "type": "option",
    "attributes": {
      "unique_id": "option-uuid-001",
      "text": "O(1)",
      "created_at": "2025-09-01T10:00:00Z"
    }
  }
}
```

---

### PUT /tests/:unique_id/options/:options_unique_id - Update Option

Updates an existing option.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tests/test-uuid-001/options/option-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "option": {
      "text": "O(1) amortized"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "option-uuid-001",
    "type": "option",
    "attributes": {
      "unique_id": "option-uuid-001",
      "text": "O(1) amortized"
    }
  }
}
```

---

### PUT /tests/:unique_id/questions/:question_id/options - Add Option to Question

Adds an option to a question.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/tests/test-uuid-001/questions/question-uuid-001/options" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "option_unique_id": "option-uuid-001"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `option_unique_id` | uuid | Yes | Option to add |

**Response 200:**
```json
{
  "message": "Option added to question successfully"
}
```

---

### GET /test/:unique_id/results - Test Results

Retrieves results for a test across all students.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/test/test-uuid-001/results" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "result-uuid-001",
      "type": "test_result",
      "attributes": {
        "unique_id": "result-uuid-001",
        "user_unique_id": "student-uuid-123",
        "test_unique_id": "test-uuid-001",
        "score": 85,
        "max_score": 100,
        "passed": true,
        "started_at": "2025-10-15T09:00:00Z",
        "finished_at": "2025-10-15T10:30:00Z"
      }
    }
  ],
  "meta": {
    "total_count": 28,
    "average_score": 72.5,
    "pass_rate": 0.82
  }
}
```

---

### GET /test/:unique_id/solution - Test Solution

Retrieves the solution (correct answers) for a test.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/test/test-uuid-001/solution" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "test-uuid-001",
    "type": "test_solution",
    "attributes": {
      "test_unique_id": "test-uuid-001",
      "questions": [
        {
          "question_unique_id": "question-uuid-001",
          "text": "What is the time complexity of a hash table lookup?",
          "correct_option_id": "option-uuid-001",
          "correct_answer": "O(1)",
          "points": 10
        }
      ]
    }
  }
}
```

---

## Student Test Flow

### GET /users/:unique_id/tests - User Tests

Retrieves all test instances for a student.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/tests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "instance-uuid-201",
      "type": "test_instance",
      "attributes": {
        "unique_id": "instance-uuid-201",
        "test_unique_id": "test-uuid-001",
        "user_unique_id": "student-uuid-123",
        "status": "completed",
        "score": 85,
        "max_score": 100,
        "passed": true,
        "started_at": "2025-10-15T09:00:00Z",
        "finished_at": "2025-10-15T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /users/:unique_id/test/:test_unique_id - Start Test

Starts a test for a student.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/student-uuid-123/test/test-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 201:**
```json
{
  "data": {
    "id": "instance-uuid-201",
    "type": "test_instance",
    "attributes": {
      "unique_id": "instance-uuid-201",
      "test_unique_id": "test-uuid-001",
      "user_unique_id": "student-uuid-123",
      "status": "in_progress",
      "started_at": "2025-10-15T09:00:00Z"
    }
  }
}
```

---

### PUT /users/:unique_id/test/:test_instance_unique_id - Submit Response

Submits a response to a test question.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/test/instance-uuid-201" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "response": {
      "question_unique_id": "question-uuid-001",
      "option_unique_id": "option-uuid-001"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `question_unique_id` | uuid | Yes | Question being answered |
| `option_unique_id` | uuid | Yes | Selected option |

**Response 200:**
```json
{
  "data": {
    "id": "instance-uuid-201",
    "type": "test_instance",
    "attributes": {
      "unique_id": "instance-uuid-201",
      "status": "in_progress",
      "responses_count": 8
    }
  }
}
```

---

### PUT /users/:unique_id/test/:test_instance_unique_id/finish - Finish Test

Finishes a student's test.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/test/instance-uuid-201/finish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "instance-uuid-201",
    "type": "test_instance",
    "attributes": {
      "unique_id": "instance-uuid-201",
      "status": "completed",
      "score": 85,
      "max_score": 100,
      "passed": true,
      "finished_at": "2025-10-15T10:30:00Z"
    }
  }
}
```

---

## Data Models

### Test
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Test name |
| `description` | string | Test description |
| `test_type` | string | Type: `quiz`, `exam`, `midterm`, `final` |
| `time_limit` | integer | Time limit in minutes |
| `max_score` | integer | Maximum possible score |
| `passing_score` | integer | Minimum score to pass |
| `questions` | array | Test questions |
| `status` | string | Status: `active`, `inactive`, `draft` |

### Question
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `text` | string | Question text |
| `question_type` | string | Type: `multiple_choice`, `true_false`, `short_answer` |
| `points` | integer | Points value |

### Option
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `text` | string | Option text |
| `is_correct` | boolean | Whether this is the correct answer |

### TestInstance
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `test_unique_id` | uuid | Parent test ID |
| `user_unique_id` | uuid | Student ID |
| `status` | string | Status: `in_progress`, `completed` |
| `score` | integer | Final score |
| `max_score` | integer | Maximum possible score |
| `passed` | boolean | Whether student passed |
| `started_at` | timestamp | Start time |
| `finished_at` | timestamp | Completion time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Test Already Started",
    "detail": "Student already has an active test instance."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-university
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
// ContentTestsService — client.university.contentTests
client.university.contentTests.list(params?: ListContentTestsParams): Promise<PageResult<ContentTest>>;
client.university.contentTests.get(uniqueId: string): Promise<ContentTest>;
client.university.contentTests.create(data: CreateContentTestRequest): Promise<ContentTest>;
client.university.contentTests.update(uniqueId: string, data: UpdateContentTestRequest): Promise<ContentTest>;
client.university.contentTests.getResults(uniqueId: string): Promise<unknown[]>;
client.university.contentTests.getSolution(uniqueId: string): Promise<unknown>;
client.university.contentTests.createQuestion(uniqueId: string, data: CreateQuestionRequest): Promise<TestQuestion>;
client.university.contentTests.updateQuestion(uniqueId: string, questionUniqueId: string, data: Partial<CreateQuestionRequest>): Promise<TestQuestion>;
client.university.contentTests.getQuestion(uniqueId: string, questionId: string): Promise<TestQuestion>;
client.university.contentTests.listOptions(): Promise<TestOption[]>;
client.university.contentTests.createOption(data: CreateOptionRequest): Promise<TestOption>;
client.university.contentTests.updateOption(uniqueId: string, optionUniqueId: string, data: Partial<CreateOptionRequest>): Promise<TestOption>;
client.university.contentTests.addOptionToQuestion(uniqueId: string, questionId: string, optionId: string): Promise<TestQuestion>;
```

### TypeScript Types

```typescript
import type {
  ContentTest,
  TestQuestion,
  TestOption,
  CreateContentTestRequest,
  UpdateContentTestRequest,
  CreateQuestionRequest,
  CreateOptionRequest,
  ListContentTestsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.contentTests.list();
}
```
