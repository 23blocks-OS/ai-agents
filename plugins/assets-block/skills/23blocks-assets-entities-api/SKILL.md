---
name: 23blocks-assets-entities-api
description: "Assets Block digital entities and their access: public/private/request-based, approve, deny, revoke. Use when sharing entities."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Entities API

Complete API reference for 23blocks Assets Block digital entity management with granular access control.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://assets.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"403","code":"forbidden","title":"Access Denied","detail":"You do not have permission to manage access for this entity."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-assets`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAssetsBlock()` from `@23blocks/react`.

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
