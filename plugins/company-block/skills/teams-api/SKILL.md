---
name: company-teams-api
description: Create and manage 23blocks company teams via REST API. Use when creating, updating, or deleting teams, managing team membership (join, remove, list members), or querying user team associations.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Teams API

Complete API reference for 23blocks company team management with membership.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://company.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Company API base URL | `https://company.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/teams" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /teams - List Teams

Lists all teams with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teams?page=1&records=20" \
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
      "id": "team-uuid-123",
      "type": "team",
      "attributes": {
        "unique_id": "team-uuid-123",
        "name": "Backend Team",
        "description": "Backend services development",
        "department_id": "dept-uuid-123",
        "team_lead_id": "user-uuid-001",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "team-uuid-456",
      "type": "team",
      "attributes": {
        "unique_id": "team-uuid-456",
        "name": "Frontend Team",
        "description": "Frontend development",
        "department_id": "dept-uuid-123",
        "team_lead_id": "user-uuid-002",
        "status": "active",
        "created_at": "2025-01-11T09:00:00Z",
        "updated_at": "2025-01-11T09:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### GET /teams/:unique_id - Get Team

Retrieves a single team by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teams/team-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "team-uuid-123",
    "type": "team",
    "attributes": {
      "unique_id": "team-uuid-123",
      "name": "Backend Team",
      "description": "Backend services development",
      "department_id": "dept-uuid-123",
      "team_lead_id": "user-uuid-001",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "department": {
        "data": { "id": "dept-uuid-123", "type": "department" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Team not found

---

### GET /teams/:unique_id/members - List Team Members

Lists all members of a team.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teams/team-uuid-123/members?page=1&records=20" \
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
      "id": "member-uuid-123",
      "type": "team_member",
      "attributes": {
        "unique_id": "member-uuid-123",
        "user_unique_id": "user-uuid-001",
        "email": "engineer@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "role": "lead",
        "joined_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "member-uuid-456",
      "type": "team_member",
      "attributes": {
        "unique_id": "member-uuid-456",
        "user_unique_id": "user-uuid-002",
        "email": "dev@example.com",
        "first_name": "Jane",
        "last_name": "Smith",
        "role": "member",
        "joined_at": "2025-01-11T08:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 8
  }
}
```

**Errors:**
- `404 Not Found` - Team not found

---

### POST /teams - Create Team

Creates a new team.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teams" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "team": {
      "name": "DevOps Team",
      "description": "Infrastructure and deployment",
      "department_id": "dept-uuid-123",
      "team_lead_id": "user-uuid-003",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Team name |
| `description` | string | No | Team description |
| `department_id` | uuid | No | Associated department ID |
| `team_lead_id` | uuid | No | Team lead user ID |
| `status` | string | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "new-team-uuid",
    "type": "team",
    "attributes": {
      "unique_id": "new-team-uuid",
      "name": "DevOps Team",
      "description": "Infrastructure and deployment",
      "department_id": "dept-uuid-123",
      "team_lead_id": "user-uuid-003",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Team name already exists in department
- `422 Unprocessable Entity` - Validation errors

---

### PUT /teams/:unique_id - Update Team

Updates an existing team.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/teams/team-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "team": {
      "name": "Backend & API Team",
      "description": "Backend services and API development",
      "team_lead_id": "user-uuid-005"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "team-uuid-123",
    "type": "team",
    "attributes": {
      "unique_id": "team-uuid-123",
      "name": "Backend & API Team",
      "description": "Backend services and API development",
      "department_id": "dept-uuid-123",
      "team_lead_id": "user-uuid-005",
      "status": "active",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Team not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /teams/:unique_id - Delete Team

Deletes a team.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/teams/team-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Team not found

---

### POST /teams/:unique_id/join - Join Team

Adds the authenticated user to the team.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teams/team-uuid-123/join" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 201:**
```json
{
  "data": {
    "id": "member-uuid-789",
    "type": "team_member",
    "attributes": {
      "unique_id": "member-uuid-789",
      "user_unique_id": "current-user-uuid",
      "role": "member",
      "joined_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Team not found
- `409 Conflict` - User already a team member

---

### DELETE /teams/:unique_id/members/:user_unique_id - Remove Member

Removes a member from a team.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/teams/team-uuid-123/members/user-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Member not found in team

---

### GET /users/:unique_id/teams - Get User's Teams

Lists all teams a specific user belongs to.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-001/teams?page=1&records=20" \
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
      "id": "team-uuid-123",
      "type": "team",
      "attributes": {
        "unique_id": "team-uuid-123",
        "name": "Backend Team",
        "description": "Backend services development",
        "department_id": "dept-uuid-123",
        "team_lead_id": "user-uuid-001",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "team-uuid-789",
      "type": "team",
      "attributes": {
        "unique_id": "team-uuid-789",
        "name": "Architecture Guild",
        "description": "Cross-team architecture decisions",
        "department_id": null,
        "team_lead_id": "user-uuid-010",
        "status": "active",
        "created_at": "2025-01-08T11:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

## Data Models

### Team
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Team name |
| `description` | string | Team description |
| `department_id` | uuid | Associated department ID |
| `team_lead_id` | uuid | Team lead user ID |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### TeamMember
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Membership identifier |
| `user_unique_id` | uuid | User identifier |
| `email` | string | Member email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `role` | string | Role within team (lead, member) |
| `joined_at` | timestamp | When member joined |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "conflict",
    "title": "Already a Member",
    "detail": "This user is already a member of the team."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-company
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
// Teams — client.company.teams
client.company.teams.list(params?: ListTeamsParams): Promise<PageResult<Team>>;
client.company.teams.get(uniqueId: string): Promise<Team>;
client.company.teams.create(data: CreateTeamRequest): Promise<Team>;
client.company.teams.update(uniqueId: string, data: UpdateTeamRequest): Promise<Team>;
client.company.teams.delete(uniqueId: string): Promise<void>;
client.company.teams.listByDepartment(departmentUniqueId: string): Promise<Team[]>;
```

### TypeScript Types

```typescript
import type {
  Team,
  CreateTeamRequest,
  UpdateTeamRequest,
  ListTeamsParams,
} from '@23blocks/block-company';
```

### React Hook

```typescript
import { useCompanyBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCompanyBlock();
  const result = await client.company.teams.list();
}
```
