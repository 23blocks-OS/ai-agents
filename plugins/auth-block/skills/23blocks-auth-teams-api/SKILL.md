---
name: 23blocks-auth-teams-api
description: "Auth Block teams: CRUD, members, member roles. Use for access-control teams; org-chart teams are 23blocks-company-teams-api."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Teams API

Complete API reference for 23blocks team management with membership.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /teams - List Teams

Lists all teams.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/teams?page=1&records=20" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"409","code":"conflict","title":"Already a Member","detail":"This user is already a member of the team."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-authentication`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAuthenticationBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// TenantUsersService — client.authentication.tenantUsers
current(): Promise<TenantUser>;
get(userUniqueId: string): Promise<TenantUser>;
list(params?: ListParams): Promise<TenantUser[]>;
```

### TypeScript Types

```typescript
import type {
  TenantUser,
  TenantUserFull,
  CreateTenantUserRequest,
} from '@23blocks/block-authentication';
```
