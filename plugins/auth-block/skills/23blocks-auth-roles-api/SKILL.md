---
name: 23blocks-auth-roles-api
description: "Auth Block roles: CRUD, attach or detach permissions, list a role's users. Use for role-based access control."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Roles API

Complete API reference for 23blocks role-based access control management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /roles - List Roles

Lists all roles.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/roles?page=1&records=20" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"409","code":"conflict","title":"Role Already Exists","detail":"A role with the name 'admin' already exists."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-authentication`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAuthenticationBlock()` from `@23blocks/react`.

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
