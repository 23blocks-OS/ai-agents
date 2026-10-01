---
name: 23blocks-auth-permissions-api
description: "Auth Block permissions: define, grant, revoke, check access, list by resource type. Use for fine-grained access control."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Permissions API

Complete API reference for 23blocks granular permission management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://auth.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /permissions - List Permissions

Lists all permission definitions.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/permissions?page=1&records=20" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Permission Not Found","detail":"The requested permission could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-authentication`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAuthenticationBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// PermissionsService — client.authentication.permissions
list(params?: ListParams): Promise<PageResult<Permission>>;
get(uniqueId: string): Promise<Permission>;
create(request: CreatePermissionRequest): Promise<Permission>;
update(uniqueId: string, request: UpdatePermissionRequest): Promise<Permission>;
delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Permission,
} from '@23blocks/block-authentication';

// Also exported from the service file:
import type {
  CreatePermissionRequest,
  UpdatePermissionRequest,
} from '@23blocks/block-authentication';
```
