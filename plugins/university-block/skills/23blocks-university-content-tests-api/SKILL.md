---
name: 23blocks-university-content-tests-api
description: Manage 23blocks University content tests via REST API. Use when creating tests with questions and options, managing test results and solutions, or when students start, answer, and finish knowledge assessments.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Content Tests API

Complete API reference for 23blocks University content test management with questions, options, results, and student test-taking workflows.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

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

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/tests` | List content tests |
| GET | `/tests/:unique_id` | Get test with questions |
| POST | `/tests/` | Create test |
| PUT | `/tests/:unique_id` | Update test |
| GET | `/tests/:unique_id/questions/:question_id` | Get question |
| POST | `/tests/:unique_id/questions` | Create question |
| PUT | `/tests/:unique_id/questions/:question_unique_id` | Update question |
| GET | `/tests/questions/options` | List options |
| POST | `/tests/options` | Create option |
| PUT | `/tests/:unique_id/options/:options_unique_id` | Update option |
| PUT | `/tests/:unique_id/questions/:question_id/options` | Add option to question |
| GET | `/test/:unique_id/results` | Get test results |
| GET | `/test/:unique_id/solution` | Get test solution |
| GET | `/users/:unique_id/tests` | Get student's test instances |
| POST | `/users/:unique_id/test/:test_unique_id` | Start test |
| PUT | `/users/:unique_id/test/:test_instance_unique_id` | Submit response |
| PUT | `/users/:unique_id/test/:test_instance_unique_id/finish` | Finish test |

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
