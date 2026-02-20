---
name: auth-teams-api
description: Manage 23blocks teams via REST API. Use when creating, updating, or deleting teams, managing team membership, or updating team member roles.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Teams API

Complete API reference for 23blocks team management with membership.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://auth.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
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

Lists all teams.

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
        "name": "Engineering",
        "description": "Engineering team",
        "members_count": 8,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "organization": {
          "data": { "id": "org-uuid", "type": "organization" }
        }
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 5
  }
}
```

---

### GET /teams/:unique_id - Get Team

Retrieves a single team by ID.

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
      "name": "Engineering",
      "description": "Engineering team",
      "members_count": 8,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "organization": {
        "data": { "id": "org-uuid", "type": "organization" }
      }
    }
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
      "name": "Design",
      "description": "Design and UX team",
      "organization_id": "org-uuid-123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Team name |
| `description` | string | No | Team description |
| `organization_id` | uuid | Yes | Parent organization |

**Response 201:**
```json
{
  "data": {
    "id": "new-team-uuid",
    "type": "team",
    "attributes": {
      "unique_id": "new-team-uuid",
      "name": "Design",
      "description": "Design and UX team",
      "members_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Team name already exists in organization
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
      "name": "Engineering & Platform",
      "description": "Updated team description"
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
      "name": "Engineering & Platform",
      "description": "Updated team description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

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

### GET /teams/:unique_id/members - List Team Members

Lists all members of a team.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teams/team-uuid-123/members?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "member-uuid-123",
      "type": "team_member",
      "attributes": {
        "unique_id": "member-uuid-123",
        "user_id": "user-uuid-123",
        "email": "engineer@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "role": "lead",
        "joined_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 8
  }
}
```

---

### POST /teams/:unique_id/members - Add Team Member

Adds a member to a team.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/teams/team-uuid-123/members" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "member": {
      "user_id": "user-uuid-456",
      "role": "member"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | uuid | Yes | User to add |
| `role` | string | No | Role in team (default: member) |

**Response 201:**
```json
{
  "data": {
    "id": "member-uuid-456",
    "type": "team_member",
    "attributes": {
      "unique_id": "member-uuid-456",
      "user_id": "user-uuid-456",
      "role": "member",
      "joined_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User or team not found
- `409 Conflict` - User already a team member

---

### DELETE /teams/:unique_id/members/:user_id - Remove Team Member

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

### PUT /teams/:unique_id/members/:user_id - Update Team Member Role

Updates a team member's role.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/teams/team-uuid-123/members/user-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "member": {
      "role": "lead"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "member-uuid-456",
    "type": "team_member",
    "attributes": {
      "unique_id": "member-uuid-456",
      "user_id": "user-uuid-456",
      "role": "lead",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## Data Models

### Team
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Team name |
| `description` | string | Team description |
| `organization_id` | uuid | Parent organization |
| `members_count` | integer | Number of members |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### TeamMember
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Membership identifier |
| `user_id` | uuid | User identifier |
| `email` | string | Member email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `role` | string | Role within team |
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
