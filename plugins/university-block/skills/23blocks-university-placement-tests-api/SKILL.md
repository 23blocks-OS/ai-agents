---
name: 23blocks-university-placement-tests-api
description: "University Block placement tests: sections, questions, routing rules, students taking them. Use for level placement."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Placement Tests API

Complete API reference for 23blocks University placement test management with sections, questions, options, routing rules, and student test-taking workflows.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Test Already Started","detail":"Student already has an active placement test instance."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
