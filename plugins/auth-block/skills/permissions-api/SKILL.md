---
name: auth-permissions-api
description: Manage 23blocks permissions via REST API. Use when creating or managing permission definitions, granting or revoking permissions, checking user access, or viewing resource permissions.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Permissions API

Complete API reference for 23blocks granular permission management.

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
curl -X GET "$BLOCKS_API_URL/permissions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /permissions - List Permissions

Lists all permission definitions.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/permissions?page=1&records=20" \
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
      "id": "perm-uuid-123",
      "type": "permission",
      "attributes": {
        "unique_id": "perm-uuid-123",
        "name": "users.read",
        "description": "Read user data",
        "resource_type": "users",
        "action": "read",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### GET /permissions/:unique_id - Get Permission

Retrieves a single permission by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/permissions/perm-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "perm-uuid-123",
    "type": "permission",
    "attributes": {
      "unique_id": "perm-uuid-123",
      "name": "users.read",
      "description": "Read user data",
      "resource_type": "users",
      "action": "read",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Permission not found

---

### POST /permissions - Create Permission

Creates a new permission definition.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/permissions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "permission": {
      "name": "posts.write",
      "description": "Create and edit posts",
      "resource_type": "posts",
      "action": "write"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Permission name (resource.action) |
| `description` | string | No | Human-readable description |
| `resource_type` | string | Yes | Resource this applies to |
| `action` | string | Yes | Action type (read, write, delete, admin) |

**Response 201:**
```json
{
  "data": {
    "id": "new-perm-uuid",
    "type": "permission",
    "attributes": {
      "unique_id": "new-perm-uuid",
      "name": "posts.write",
      "description": "Create and edit posts",
      "resource_type": "posts",
      "action": "write",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Permission name already exists
- `422 Unprocessable Entity` - Validation errors

---

### PUT /permissions/:unique_id - Update Permission

Updates an existing permission.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/permissions/perm-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "permission": {
      "description": "Updated permission description"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "perm-uuid-123",
    "type": "permission",
    "attributes": {
      "unique_id": "perm-uuid-123",
      "description": "Updated permission description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /permissions/:unique_id - Delete Permission

Deletes a permission definition.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/permissions/perm-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Permission not found
- `422 Unprocessable Entity` - Permission still assigned to roles

---

### GET /permissions/resources/:resource_type - Get Resource Permissions

Lists all permissions for a specific resource type.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/permissions/resources/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "perm-uuid-1",
      "type": "permission",
      "attributes": {
        "unique_id": "perm-uuid-1",
        "name": "users.read",
        "resource_type": "users",
        "action": "read"
      }
    },
    {
      "id": "perm-uuid-2",
      "type": "permission",
      "attributes": {
        "unique_id": "perm-uuid-2",
        "name": "users.write",
        "resource_type": "users",
        "action": "write"
      }
    }
  ]
}
```

---

### POST /permissions/grant - Grant Permission

Grants a permission to a user or role.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/permissions/grant" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "grant": {
      "permission_id": "perm-uuid-123",
      "grantee_id": "user-uuid-456",
      "grantee_type": "user"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `permission_id` | uuid | Yes | Permission to grant |
| `grantee_id` | uuid | Yes | User or role ID |
| `grantee_type` | string | Yes | "user" or "role" |

**Response 200:**
```json
{
  "message": "Permission granted successfully"
}
```

**Errors:**
- `404 Not Found` - Permission or grantee not found
- `409 Conflict` - Permission already granted

---

### DELETE /permissions/revoke - Revoke Permission

Revokes a permission from a user or role.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/permissions/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "revoke": {
      "permission_id": "perm-uuid-123",
      "grantee_id": "user-uuid-456",
      "grantee_type": "user"
    }
  }'
```

**Response 200:**
```json
{
  "message": "Permission revoked successfully"
}
```

---

### POST /permissions/check - Check Permission

Checks if a user has a specific permission.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/permissions/check" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "check": {
      "user_id": "user-uuid-456",
      "permission": "users.write",
      "resource_id": "user-uuid-789"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_id` | uuid | Yes | User to check |
| `permission` | string | Yes | Permission name |
| `resource_id` | uuid | No | Specific resource ID |

**Response 200:**
```json
{
  "data": {
    "allowed": true,
    "permission": "users.write",
    "source": "role:admin"
  }
}
```

---

## Data Models

### Permission
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Permission name (resource.action) |
| `description` | string | Human-readable description |
| `resource_type` | string | Resource type |
| `action` | string | Action (read, write, delete, admin) |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Permission Not Found",
    "detail": "The requested permission could not be found."
  }]
}
```
