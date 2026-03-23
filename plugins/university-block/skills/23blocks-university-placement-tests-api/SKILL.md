---
name: 23blocks-university-placement-tests-api
description: Manage 23blocks University placement tests via REST API. Use when creating placement tests with sections, questions, options, routing rules, or when students start, answer, and finish placement assessments.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Placement Tests API

Complete API reference for 23blocks University placement test management with sections, questions, options, routing rules, and student test-taking workflows.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | University API base URL | `https://university.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://university.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://university.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/placements/:unique_id` | Get placement test with sections and rules |
| POST | `/placements/:unique_id/rules` | Create routing rule |
| GET | `/placements/:unique_id/sections/:section_id` | Get section |
| POST | `/placements/:unique_id/sections` | Create section |
| GET | `/placements/:unique_id/questions/:question_id` | Get question |
| POST | `/placements/:unique_id/questions` | Create question |
| PUT | `/placements/:unique_id/section/:section_id/questions` | Add question to section |
| GET | `/placements/questions/options` | List options |
| POST | `/placements/options` | Create option |
| PUT | `/placements/:unique_id/questions/:question_id/options` | Add option to question |
| PUT | `/placements/:unique_id/questions/:question_id/options/:option_id/set-right` | Set correct answer |
| DELETE | `/placements/:unique_id/questions/:question_id/options/:option_id` | Remove option |
| PUT | `/placements/:placement_instance_unique_id/finish` | Complete test (admin) |
| GET | `/users/:unique_id/placement` | Get student's placement results |
| POST | `/users/:unique_id/placement/:placement_unique_id` | Start placement test |
| PUT | `/users/:unique_id/placement/:placement_instance_unique_id` | Submit response |
| PUT | `/users/:unique_id/placement/:placement_instance_unique_id/finish` | Finish placement test |

---

## Data Models

### Placement
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Placement test name |
| `description` | string | Test description |
| `course_id` | uuid | Associated course |
| `sections` | array | Test sections |
| `rules` | array | Routing rules |
| `time_limit` | integer | Time limit in minutes |
| `status` | string | Status: `active`, `inactive`, `draft` |

### Section
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Section name |
| `description` | string | Section description |
| `order` | integer | Display order |

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

### Rule
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Rule name |
| `min_score` | integer | Minimum score threshold |
| `max_score` | integer | Maximum score threshold |
| `target_course_id` | uuid | Course to place student in |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Test Already Started",
    "detail": "Student already has an active placement test instance."
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
// PlacementsService — client.university.placements

// Placement Tests
client.university.placements.get(uniqueId: string): Promise<PlacementTest>;
client.university.placements.listByCourse(courseUniqueId: string, params?: ListPlacementsParams): Promise<PageResult<PlacementTest>>;
client.university.placements.create(courseUniqueId: string, data: CreatePlacementRequest): Promise<PlacementTest>;

// Sections
client.university.placements.getSection(placementUniqueId: string, sectionId: string): Promise<PlacementSection>;
client.university.placements.createSection(placementUniqueId: string, data: CreatePlacementSectionRequest): Promise<PlacementSection>;

// Questions
client.university.placements.getQuestion(placementUniqueId: string, questionId: string): Promise<PlacementQuestion>;
client.university.placements.createQuestion(placementUniqueId: string, data: CreatePlacementQuestionRequest): Promise<PlacementQuestion>;
client.university.placements.addQuestionToSection(placementUniqueId: string, sectionId: string, questionId: string): Promise<void>;

// Options
client.university.placements.listOptions(): Promise<PlacementOption[]>;
client.university.placements.createOption(data: CreatePlacementOptionRequest): Promise<PlacementOption>;
client.university.placements.addOptionToQuestion(placementUniqueId: string, questionId: string, optionId: string): Promise<void>;
client.university.placements.setRightOption(placementUniqueId: string, questionId: string, optionId: string): Promise<void>;
client.university.placements.removeOption(placementUniqueId: string, questionId: string, optionId: string): Promise<void>;

// Rules
client.university.placements.createRule(placementUniqueId: string, data: CreatePlacementRuleRequest): Promise<PlacementRule>;

// User Placements
client.university.placements.getUserPlacement(userUniqueId: string): Promise<PlacementInstance | null>;
client.university.placements.startPlacement(userUniqueId: string, placementUniqueId: string): Promise<PlacementInstance>;
client.university.placements.submitResponse(userUniqueId: string, instanceUniqueId: string, responses: PlacementResponse[]): Promise<PlacementInstance>;
client.university.placements.finishPlacement(userUniqueId: string, instanceUniqueId: string): Promise<PlacementInstance>;
```

### TypeScript Types

```typescript
import type {
  PlacementTest,
  PlacementSection,
  PlacementQuestion,
  PlacementOption,
  PlacementRule,
  PlacementInstance,
  CreatePlacementRequest,
  CreatePlacementSectionRequest,
  CreatePlacementQuestionRequest,
  CreatePlacementOptionRequest,
  CreatePlacementRuleRequest,
  PlacementResponse,
  ListPlacementsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.placements.listByCourse('course-unique-id');
}
```
