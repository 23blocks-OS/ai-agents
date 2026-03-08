# Placement Tests API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## GET /placements/:unique_id - Get Test with Sections/Rules

Retrieves a placement test with its sections and routing rules.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/placements/placement-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "placement-uuid-001",
    "type": "placement",
    "attributes": {
      "unique_id": "placement-uuid-001",
      "name": "CS Placement Assessment",
      "description": "Determines student starting level",
      "course_id": "course-uuid-456",
      "time_limit": 60,
      "status": "active"
    },
    "relationships": {
      "sections": {
        "data": [
          { "id": "section-uuid-001", "type": "section" },
          { "id": "section-uuid-002", "type": "section" }
        ]
      },
      "rules": {
        "data": [
          { "id": "rule-uuid-001", "type": "rule" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Placement test not found

---

## POST /placements/:unique_id/rules - Create Routing Rule

Creates a routing rule that determines placement based on test scores.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/placements/placement-uuid-001/rules" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "rule": {
      "name": "Beginner Placement",
      "min_score": 0,
      "max_score": 40,
      "target_course_id": "course-uuid-beginner",
      "description": "Place in beginner course if score is below 40"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Rule name |
| `min_score` | integer | Yes | Minimum score threshold |
| `max_score` | integer | Yes | Maximum score threshold |
| `target_course_id` | uuid | Yes | Course to place student in |
| `description` | string | No | Rule description |

**Response 201:**
```json
{
  "data": {
    "id": "rule-uuid-001",
    "type": "rule",
    "attributes": {
      "unique_id": "rule-uuid-001",
      "name": "Beginner Placement",
      "min_score": 0,
      "max_score": 40,
      "target_course_id": "course-uuid-beginner",
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

## GET /placements/:unique_id/sections/:section_id - Get Section

Retrieves a specific section of a placement test.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/placements/placement-uuid-001/sections/section-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "section-uuid-001",
    "type": "section",
    "attributes": {
      "unique_id": "section-uuid-001",
      "name": "Basic Programming Concepts",
      "description": "Tests fundamental programming knowledge",
      "order": 1,
      "placement_unique_id": "placement-uuid-001"
    },
    "relationships": {
      "questions": {
        "data": [
          { "id": "question-uuid-001", "type": "question" }
        ]
      }
    }
  }
}
```

---

## POST /placements/:unique_id/sections - Create Section

Creates a new section in a placement test.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/placements/placement-uuid-001/sections" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "section": {
      "name": "Basic Programming Concepts",
      "description": "Tests fundamental programming knowledge",
      "order": 1
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Section name |
| `description` | string | No | Section description |
| `order` | integer | No | Display order |

**Response 201:**
```json
{
  "data": {
    "id": "section-uuid-001",
    "type": "section",
    "attributes": {
      "unique_id": "section-uuid-001",
      "name": "Basic Programming Concepts",
      "order": 1,
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

## GET /placements/:unique_id/questions/:question_id - Get Question

Retrieves a specific question from a placement test.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/placements/placement-uuid-001/questions/question-uuid-001" \
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
      "text": "What is the time complexity of binary search?",
      "question_type": "multiple_choice",
      "points": 10
    },
    "relationships": {
      "options": {
        "data": [
          { "id": "option-uuid-001", "type": "option" },
          { "id": "option-uuid-002", "type": "option" }
        ]
      }
    }
  }
}
```

---

## POST /placements/:unique_id/questions - Create Question

Creates a new question for a placement test.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/placements/placement-uuid-001/questions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "question": {
      "text": "What is the time complexity of binary search?",
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
      "text": "What is the time complexity of binary search?",
      "question_type": "multiple_choice",
      "points": 10,
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

## PUT /placements/:unique_id/section/:section_id/questions - Add Question to Section

Adds an existing question to a section.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/placements/placement-uuid-001/section/section-uuid-001/questions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "question_unique_id": "question-uuid-001"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `question_unique_id` | uuid | Yes | Question to add |

**Response 200:**
```json
{
  "message": "Question added to section successfully"
}
```

---

## GET /placements/questions/options - List Options

Lists all available options across placement questions.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/placements/questions/options" \
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
        "text": "O(log n)",
        "is_correct": true
      }
    }
  ]
}
```

---

## POST /placements/options - Create Option

Creates a new answer option.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/placements/options" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "option": {
      "text": "O(log n)"
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
      "text": "O(log n)",
      "is_correct": false,
      "created_at": "2025-06-15T10:00:00Z"
    }
  }
}
```

---

## PUT /placements/:unique_id/questions/:question_id/options - Add Option

Adds an option to a question.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/placements/placement-uuid-001/questions/question-uuid-001/options" \
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

## PUT /placements/:unique_id/questions/:question_id/options/:option_id/set-right - Set Correct

Marks an option as the correct answer for a question.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/placements/placement-uuid-001/questions/question-uuid-001/options/option-uuid-001/set-right" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "option-uuid-001",
    "type": "option",
    "attributes": {
      "unique_id": "option-uuid-001",
      "text": "O(log n)",
      "is_correct": true
    }
  }
}
```

---

## DELETE /placements/:unique_id/questions/:question_id/options/:option_id - Remove Option

Removes an option from a question.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/placements/placement-uuid-001/questions/question-uuid-001/options/option-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Option removed successfully"
}
```

---

## PUT /placements/:placement_instance_unique_id/finish - Complete Test

Completes a placement test instance (admin/system level).

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/placements/instance-uuid-101/finish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "instance-uuid-101",
    "type": "placement_instance",
    "attributes": {
      "unique_id": "instance-uuid-101",
      "status": "completed",
      "total_score": 75,
      "placement_result": "intermediate",
      "finished_at": "2025-09-05T10:45:00Z"
    }
  }
}
```

---

## Student Placement Flow

### GET /users/:unique_id/placement - Get User Placement

Retrieves a student's placement test results.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/student-uuid-123/placement" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "instance-uuid-101",
      "type": "placement_instance",
      "attributes": {
        "unique_id": "instance-uuid-101",
        "placement_unique_id": "placement-uuid-001",
        "user_unique_id": "student-uuid-123",
        "status": "completed",
        "total_score": 75,
        "placement_result": "intermediate",
        "started_at": "2025-09-05T10:00:00Z",
        "finished_at": "2025-09-05T10:45:00Z"
      }
    }
  ]
}
```

---

### POST /users/:unique_id/placement/:placement_unique_id - Start Placement

Starts a placement test for a student.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/student-uuid-123/placement/placement-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 201:**
```json
{
  "data": {
    "id": "instance-uuid-101",
    "type": "placement_instance",
    "attributes": {
      "unique_id": "instance-uuid-101",
      "placement_unique_id": "placement-uuid-001",
      "user_unique_id": "student-uuid-123",
      "status": "in_progress",
      "started_at": "2025-09-05T10:00:00Z"
    }
  }
}
```

---

### PUT /users/:unique_id/placement/:placement_instance_unique_id - Submit Response

Submits a response to a placement test question.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/placement/instance-uuid-101" \
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
    "id": "instance-uuid-101",
    "type": "placement_instance",
    "attributes": {
      "unique_id": "instance-uuid-101",
      "status": "in_progress",
      "responses_count": 5
    }
  }
}
```

---

### PUT /users/:unique_id/placement/:placement_instance_unique_id/finish - Finish

Finishes a student's placement test.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/student-uuid-123/placement/instance-uuid-101/finish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "instance-uuid-101",
    "type": "placement_instance",
    "attributes": {
      "unique_id": "instance-uuid-101",
      "status": "completed",
      "total_score": 75,
      "max_score": 100,
      "placement_result": "intermediate",
      "target_course_id": "course-uuid-intermediate",
      "finished_at": "2025-09-05T10:45:00Z"
    }
  }
}
```
