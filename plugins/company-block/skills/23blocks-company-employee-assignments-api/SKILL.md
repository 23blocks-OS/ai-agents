---
name: 23blocks-company-employee-assignments-api
description: "Company Block employee assignments to positions, departments and teams, with dates. Use when placing people in the org chart."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
  verified-by: 23blocks-api-company
  verified-date: "2026-05-18"
---

# Employee Assignments API

Complete API reference for 23blocks employee assignment management linking users to positions, departments, and teams.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://company.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /employee_assignments - List Assignments

Lists all employee assignments with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/employee_assignments?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "id": "assign-uuid-123",
      "type": "employee_assignment",
      "attributes": {
        "unique_id": "assign-uuid-123",
        "user_unique_id": "user-uuid-001",
        "position_id": "pos-uuid-123",
        "department_id": "dept-uuid-123",
        "team_id": "team-uuid-123",
        "start_date": "2025-01-15",
        "end_date": null,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "assign-uuid-456",
      "type": "employee_assignment",
      "attributes": {
        "unique_id": "assign-uuid-456",
        "user_unique_id": "user-uuid-002",
        "position_id": "pos-uuid-456",
        "department_id": "dept-uuid-456",
        "team_id": "team-uuid-456",
        "start_date": "2025-02-01",
        "end_date": null,
        "status": "active",
        "created_at": "2025-01-20T09:00:00Z",
        "updated_at": "2025-01-20T09:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 87
  },
  "links": {
    "self": "/employee_assignments?page=1&records=20",
    "next": "/employee_assignments?page=2&records=20",
    "prev": null
  }
}
```

---

### GET /employee_assignments/:unique_id - Get Assignment

Retrieves a single employee assignment by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/employee_assignments/assign-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "assign-uuid-123",
    "type": "employee_assignment",
    "attributes": {
      "unique_id": "assign-uuid-123",
      "user_unique_id": "user-uuid-001",
      "position_id": "pos-uuid-123",
      "department_id": "dept-uuid-123",
      "team_id": "team-uuid-123",
      "start_date": "2025-01-15",
      "end_date": null,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "position": {
        "data": { "id": "pos-uuid-123", "type": "position" }
      },
      "department": {
        "data": { "id": "dept-uuid-123", "type": "department" }
      },
      "team": {
        "data": { "id": "team-uuid-123", "type": "team" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Assignment not found

---

### POST /employee_assignments - Create Assignment

Creates a new employee assignment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/employee_assignments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "employee_assignment": {
      "user_unique_id": "user-uuid-003",
      "position_id": "pos-uuid-789",
      "department_id": "dept-uuid-123",
      "team_id": "team-uuid-123",
      "start_date": "2025-02-01",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User being assigned |
| `position_id` | uuid | Yes | Position to assign |
| `department_id` | uuid | No | Department to assign |
| `team_id` | uuid | No | Team to assign |
| `start_date` | date | Yes | Assignment start date (YYYY-MM-DD) |
| `end_date` | date | No | Assignment end date (YYYY-MM-DD) |
| `status` | string | No | active, inactive, pending (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "new-assign-uuid",
    "type": "employee_assignment",
    "attributes": {
      "unique_id": "new-assign-uuid",
      "user_unique_id": "user-uuid-003",
      "position_id": "pos-uuid-789",
      "department_id": "dept-uuid-123",
      "team_id": "team-uuid-123",
      "start_date": "2025-02-01",
      "end_date": null,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User, position, department, or team not found
- `409 Conflict` - User already has an active assignment for this position
- `422 Unprocessable Entity` - Validation errors

---

### PUT /employee_assignments/:unique_id - Update Assignment

Updates an existing employee assignment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/employee_assignments/assign-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "employee_assignment": {
      "end_date": "2025-06-30",
      "status": "inactive"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "assign-uuid-123",
    "type": "employee_assignment",
    "attributes": {
      "unique_id": "assign-uuid-123",
      "user_unique_id": "user-uuid-001",
      "position_id": "pos-uuid-123",
      "department_id": "dept-uuid-123",
      "team_id": "team-uuid-123",
      "start_date": "2025-01-15",
      "end_date": "2025-06-30",
      "status": "inactive",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Assignment not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /employee_assignments/:unique_id - Delete Assignment

Deletes an employee assignment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/employee_assignments/assign-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** Empty body `{}`

**Errors:**
- `404 Not Found` - Assignment not found

---

## Data Models

### EmployeeAssignment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | Assigned user ID |
| `position_id` | uuid | Assigned position ID |
| `department_id` | uuid | Assigned department ID |
| `team_id` | uuid | Assigned team ID |
| `start_date` | date | Assignment start date |
| `end_date` | date | Assignment end date (null if ongoing) |
| `status` | string | active, inactive, pending |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"409","source":{"pointer":"/data/attributes/user_unique_id"},"code":"conflict","title":"Duplicate Assignment","detail":"This user already has an active assignment for this position."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-company`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCompanyBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// EmployeeAssignments — client.company.employeeAssignments
client.company.employeeAssignments.list(params?: ListEmployeeAssignmentsParams): Promise<PageResult<EmployeeAssignment>>;
client.company.employeeAssignments.get(uniqueId: string): Promise<EmployeeAssignment>;
client.company.employeeAssignments.create(data: CreateEmployeeAssignmentRequest): Promise<EmployeeAssignment>;
client.company.employeeAssignments.update(uniqueId: string, data: UpdateEmployeeAssignmentRequest): Promise<EmployeeAssignment>;
client.company.employeeAssignments.delete(uniqueId: string): Promise<void>;
client.company.employeeAssignments.listByUser(userUniqueId: string): Promise<EmployeeAssignment[]>;
client.company.employeeAssignments.listByPosition(positionUniqueId: string): Promise<EmployeeAssignment[]>;
```

### TypeScript Types

```typescript
import type {
  EmployeeAssignment,
  CreateEmployeeAssignmentRequest,
  UpdateEmployeeAssignmentRequest,
  ListEmployeeAssignmentsParams,
} from '@23blocks/block-company';
```
