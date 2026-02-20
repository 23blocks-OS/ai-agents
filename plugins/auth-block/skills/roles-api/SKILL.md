---
name: auth-roles-api
description: Manage 23blocks roles via REST API. Use when creating, updating, or deleting roles, assigning or removing permissions from roles, or viewing users assigned to a role.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Roles API

Complete API reference for 23blocks role-based access control management.

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
curl -X GET "$BLOCKS_API_URL/roles" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /roles - List Roles

Lists all roles.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/roles?page=1&records=20" \
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
      "id": "role-uuid-123",
      "type": "role",
      "attributes": {
        "unique_id": "role-uuid-123",
        "name": "admin",
        "description": "Full administrative access",
        "permissions_count": 15,
        "users_count": 3,
        "system": false,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
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

### GET /roles/:unique_id - Get Role

Retrieves a single role by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/roles/role-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "role-uuid-123",
    "type": "role",
    "attributes": {
      "unique_id": "role-uuid-123",
      "name": "admin",
      "description": "Full administrative access",
      "permissions_count": 15,
      "users_count": 3,
      "system": false,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "permissions": {
        "data": [
          { "id": "perm-uuid-1", "type": "permission" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Role not found

---

### POST /roles - Create Role

Creates a new role.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/roles" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "role": {
      "name": "editor",
      "description": "Can create and edit content"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Role name |
| `description` | string | No | Role description |

**Response 201:**
```json
{
  "data": {
    "id": "new-role-uuid",
    "type": "role",
    "attributes": {
      "unique_id": "new-role-uuid",
      "name": "editor",
      "description": "Can create and edit content",
      "permissions_count": 0,
      "users_count": 0,
      "system": false,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Role name already exists
- `422 Unprocessable Entity` - Validation errors

---

### PUT /roles/:unique_id - Update Role

Updates an existing role.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/roles/role-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "role": {
      "description": "Updated role description"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "role-uuid-123",
    "type": "role",
    "attributes": {
      "unique_id": "role-uuid-123",
      "name": "admin",
      "description": "Updated role description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /roles/:unique_id - Delete Role

Deletes a role. Cannot delete system roles.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/roles/role-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Cannot delete system roles
- `404 Not Found` - Role not found
- `422 Unprocessable Entity` - Role still has assigned users

---

### GET /roles/:unique_id/permissions - Get Role Permissions

Lists all permissions assigned to a role.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/roles/role-uuid-123/permissions" \
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
        "description": "Read user data",
        "resource_type": "users",
        "action": "read"
      }
    }
  ]
}
```

---

### POST /roles/:unique_id/permissions - Add Permission to Role

Assigns a permission to a role.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/roles/role-uuid-123/permissions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "permission_id": "perm-uuid-2"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `permission_id` | uuid | Yes | Permission to assign |

**Response 200:**
```json
{
  "message": "Permission added to role successfully"
}
```

**Errors:**
- `404 Not Found` - Role or permission not found
- `409 Conflict` - Permission already assigned

---

### DELETE /roles/:unique_id/permissions/:permission_id - Remove Permission from Role

Removes a permission from a role.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/roles/role-uuid-123/permissions/perm-uuid-2" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Role or permission not found

---

### GET /roles/:unique_id/users - Get Users with Role

Lists all users assigned to a specific role.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/roles/role-uuid-123/users?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "user-uuid-123",
      "type": "user",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "admin@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "status": "active"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

## Data Models

### Role
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Role name |
| `description` | string | Role description |
| `permissions_count` | integer | Number of assigned permissions |
| `users_count` | integer | Number of users with this role |
| `system` | boolean | Whether this is a system role |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "conflict",
    "title": "Role Already Exists",
    "detail": "A role with the name 'admin' already exists."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-authentication
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
// RolesService — client.authentication.roles
list(params?: ListParams): Promise<PageResult<Role>>;
get(uniqueId: string): Promise<Role>;
getByCode(code: string): Promise<Role>;
create(request: CreateRoleRequest): Promise<Role>;
update(uniqueId: string, request: UpdateRoleRequest): Promise<Role>;
delete(uniqueId: string): Promise<void>;
getPermissions(roleUniqueId: string): Promise<Permission[]>;
setPermissions(roleUniqueId: string, permissionIds: string[]): Promise<Role>;
addPermission(roleUniqueId: string, permissionUniqueId: string): Promise<Role>;
removePermission(roleUniqueId: string, permissionUniqueId: string): Promise<Role>;
listPermissions(params?: ListParams): Promise<PageResult<Permission>>;
```

### TypeScript Types

```typescript
import type {
  Role,
  Permission,
} from '@23blocks/block-authentication';

// Also exported from the service file:
import type {
  CreateRoleRequest,
  UpdateRoleRequest,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: List all roles
  const result = await client.authentication.roles.list();
}
```
