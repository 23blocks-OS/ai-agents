# Teams API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

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
