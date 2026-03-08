---
name: 23blocks-assets-entities-api
description: Create and manage 23blocks digital entities with access control via REST API. Use when creating entities, managing access permissions (public, private, request-based), approving or denying access requests, and revoking access.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Entities API

Complete API reference for 23blocks Assets Block digital entity management with granular access control.

## Required Environment Variables

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

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/entities/` | List all entities |
| GET | `/entities/:unique_id/` | Get a single entity |
| POST | `/entities/` | Create an entity |
| PUT | `/entities/:unique_id` | Update an entity |
| DELETE | `/entities/:unique_id` | Delete an entity |
| GET | `/entities/:unique_id/accesses` | List users with access |
| POST | `/entities/:unique_id/requests/access` | Request access to entity |
| POST | `/entities/:unique_id/access/make_public` | Make entity public |
| GET | `/entities/:unique_id/access` | Get access list |
| DELETE | `/entities/:unique_id/access/:access_unique_id/revoke` | Revoke user access |
| GET | `/entities/:unique_id/access/requests` | List access requests |
| PUT | `/entities/:unique_id/access/requests/:request_unique_id/approve` | Approve access request |
| DELETE | `/entities/:unique_id/access/requests/:request_unique_id/deny` | Deny access request |

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
