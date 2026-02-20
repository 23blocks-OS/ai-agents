---
name: assets-entities-api
description: Create and manage 23blocks digital entities with access control via REST API. Use when creating entities, managing access permissions (public, private, request-based), approving or denying access requests, and revoking access.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Entities API

Complete API reference for 23blocks Assets Block digital entity management with granular access control.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://assets.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## CRUD Endpoints

### GET /entities/ - List Entities

Lists all digital entities.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "entity-uuid-123",
      "type": "entity",
      "attributes": {
        "unique_id": "entity-uuid-123",
        "name": "Software License - Adobe CC",
        "description": "Adobe Creative Cloud enterprise license",
        "entity_type": "digital_license",
        "status": "active",
        "access_level": "private",
        "owner_unique_id": "user-uuid-456",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /entities/:unique_id/ - Get Entity

Retrieves a single entity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "name": "Software License - Adobe CC",
      "description": "Adobe Creative Cloud enterprise license",
      "entity_type": "digital_license",
      "status": "active",
      "access_level": "private",
      "owner_unique_id": "user-uuid-456",
      "access_count": 3,
      "pending_requests_count": 1,
      "payload": {},
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Entity not found

---

### POST /entities/ - Create Entity

Creates a new digital entity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "name": "API Key - Production",
      "description": "Production API key for payment gateway",
      "entity_type": "api_credential",
      "access_level": "private",
      "payload": {
        "provider": "stripe",
        "environment": "production"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Entity name |
| `description` | string | No | Entity description |
| `entity_type` | string | No | Type classification (e.g., digital_license, api_credential, certificate) |
| `access_level` | enum | No | private, public (default: private) |
| `payload` | json | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "new-entity-uuid",
    "type": "entity",
    "attributes": {
      "unique_id": "new-entity-uuid",
      "name": "API Key - Production",
      "description": "Production API key for payment gateway",
      "entity_type": "api_credential",
      "status": "active",
      "access_level": "private",
      "owner_unique_id": "current-user-uuid",
      "access_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /entities/:unique_id - Update Entity

Updates an existing entity.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "entity": {
      "name": "Updated Entity Name",
      "description": "Updated description"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "name": "Updated Entity Name",
      "description": "Updated description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Not the owner
- `404 Not Found` - Entity not found

---

### DELETE /entities/:unique_id - Delete Entity

Deletes a digital entity.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/entities/entity-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Not the owner
- `404 Not Found` - Entity not found

---

## Access Control

### GET /entities/:unique_id/accesses - List Accesses

Lists all users who have access to the entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123/accesses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "access-uuid-001",
      "type": "entity_access",
      "attributes": {
        "unique_id": "access-uuid-001",
        "user_unique_id": "user-uuid-789",
        "user_name": "Alice Smith",
        "user_email": "alice@example.com",
        "access_type": "granted",
        "granted_at": "2025-01-10T10:30:00Z",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /entities/:unique_id/requests/access - Request Access

Requests access to a private entity.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/requests/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "request": {
      "reason": "Need access for Q1 project implementation"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reason` | string | No | Reason for access request |

**Response 201:**
```json
{
  "data": {
    "id": "request-uuid-001",
    "type": "access_request",
    "attributes": {
      "unique_id": "request-uuid-001",
      "entity_unique_id": "entity-uuid-123",
      "user_unique_id": "requesting-user-uuid",
      "reason": "Need access for Q1 project implementation",
      "status": "pending",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /entities/:unique_id/access/make_public - Make Public

Makes an entity publicly accessible to all users.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/entities/entity-uuid-123/access/make_public" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "entity-uuid-123",
    "type": "entity",
    "attributes": {
      "unique_id": "entity-uuid-123",
      "access_level": "public",
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### GET /entities/:unique_id/access - Get Access List

Gets the full access list for an entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123/access" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "access-uuid-001",
      "type": "entity_access",
      "attributes": {
        "unique_id": "access-uuid-001",
        "user_unique_id": "user-uuid-789",
        "user_name": "Alice Smith",
        "access_type": "granted",
        "granted_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### DELETE /entities/:unique_id/access/:access_unique_id/revoke - Revoke Access

Revokes a user's access to the entity.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/entities/entity-uuid-123/access/access-uuid-001/revoke" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Not the entity owner
- `404 Not Found` - Access record not found

---

## Access Requests

### GET /entities/:unique_id/access/requests - List Access Requests

Lists pending access requests for the entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/entities/entity-uuid-123/access/requests" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "request-uuid-001",
      "type": "access_request",
      "attributes": {
        "unique_id": "request-uuid-001",
        "user_unique_id": "user-uuid-999",
        "user_name": "Bob Johnson",
        "user_email": "bob@example.com",
        "reason": "Need access for Q1 project",
        "status": "pending",
        "created_at": "2025-01-12T10:30:00Z"
      }
    }
  ]
}
```

---

### PUT /entities/:unique_id/access/requests/:request_unique_id/approve - Approve Request

Approves a pending access request.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/entities/entity-uuid-123/access/requests/request-uuid-001/approve" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "request-uuid-001",
    "type": "access_request",
    "attributes": {
      "unique_id": "request-uuid-001",
      "status": "approved",
      "approved_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Not the entity owner
- `404 Not Found` - Request not found

---

### DELETE /entities/:unique_id/access/requests/:request_unique_id/deny - Deny Request

Denies a pending access request.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/entities/entity-uuid-123/access/requests/request-uuid-001/deny" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Not the entity owner
- `404 Not Found` - Request not found

---

## Data Models

### Entity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Entity name |
| `description` | string | Entity description |
| `entity_type` | string | Type classification |
| `status` | enum | active, inactive, archived |
| `access_level` | enum | private, public |
| `owner_unique_id` | uuid | Owner user ID |
| `access_count` | integer | Number of users with access |
| `pending_requests_count` | integer | Pending access requests |
| `payload` | jsonb | Custom metadata |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### EntityAccess
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | User with access |
| `user_name` | string | User display name |
| `user_email` | string | User email |
| `access_type` | string | Type of access (granted, owner) |
| `granted_at` | timestamp | When access was granted |
| `created_at` | timestamp | Creation time |

### AccessRequest
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `entity_unique_id` | uuid | Requested entity ID |
| `user_unique_id` | uuid | Requesting user ID |
| `user_name` | string | Requesting user name |
| `reason` | string | Request reason |
| `status` | enum | pending, approved, denied |
| `approved_at` | timestamp | Approval time |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Access Denied",
    "detail": "You do not have permission to manage access for this entity."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `403` | Forbidden - Not the entity owner |
| `404` | Not Found - Entity or request not found |
| `422` | Unprocessable Entity - Validation errors |

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-assets
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
// AssetsEntitiesService — client.assets.entities
client.assets.entities.list(params?: ListAssetsEntitiesParams): Promise<PageResult<AssetsEntity>>;
client.assets.entities.get(uniqueId: string): Promise<AssetsEntity>;
client.assets.entities.create(data: CreateAssetsEntityRequest): Promise<AssetsEntity>;
client.assets.entities.update(uniqueId: string, data: UpdateAssetsEntityRequest): Promise<AssetsEntity>;
client.assets.entities.delete(uniqueId: string): Promise<void>;

// Access Management
client.assets.entities.listAccesses(uniqueId: string): Promise<EntityAccess[]>;
client.assets.entities.getAccess(uniqueId: string): Promise<EntityAccess>;
client.assets.entities.makePublic(uniqueId: string): Promise<void>;
client.assets.entities.revokeAccess(uniqueId: string, accessUniqueId: string): Promise<void>;

// Access Requests
client.assets.entities.requestAccess(uniqueId: string, data: CreateAccessRequestRequest): Promise<AccessRequest>;
client.assets.entities.listAccessRequests(uniqueId: string): Promise<AccessRequest[]>;
client.assets.entities.approveAccessRequest(uniqueId: string, requestUniqueId: string): Promise<AccessRequest>;
client.assets.entities.denyAccessRequest(uniqueId: string, requestUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  AssetsEntity,
  CreateAssetsEntityRequest,
  UpdateAssetsEntityRequest,
  ListAssetsEntitiesParams,
  EntityAccess,
  AccessRequest,
  CreateAccessRequestRequest,
} from '@23blocks/block-assets';
```

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.entities.list();
}
```
