---
name: 23blocks-files-delegations-api
description: Manage file access delegations via REST API. Use when delegating file management permissions between users or managing delegation expiration.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Delegations API

Complete API reference for 23blocks file access delegation management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Files API base URL | `https://files.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://files.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://files.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Overview

Delegations allow users to grant other users the ability to manage their files. This is useful for:
- Assistants managing files on behalf of executives
- Team members collaborating on shared documents
- Temporary access for contractors or partners

### Delegation Terminology

| Term | Description |
|------|-------------|
| **Grantor** | User who delegates access (file owner) |
| **Grantee** | User who receives delegated access |

---

## Endpoints

### GET /users/:unique_id/delegations/granted - List Granted Delegations

Lists all delegations this user has granted to others.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/delegations/granted" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "delegation-123",
      "type": "user_identity_access_delegation",
      "attributes": {
        "unique_id": "delegation-123",
        "grantor_user_unique_id": "user-123",
        "grantee_user_unique_id": "user-456",
        "grantee_name": "John Assistant",
        "grantee_email": "john.assistant@example.com",
        "status": "active",
        "starts_at": "2025-01-10T10:30:00Z",
        "expires_at": "2025-12-31T23:59:59Z",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/delegations/received - List Received Delegations

Lists all delegations this user has received from others.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/delegations/received" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "delegation-789",
      "type": "user_identity_access_delegation",
      "attributes": {
        "unique_id": "delegation-789",
        "grantor_user_unique_id": "user-abc",
        "grantee_user_unique_id": "user-123",
        "grantor_name": "Jane Executive",
        "grantor_email": "jane.executive@example.com",
        "status": "active",
        "starts_at": "2025-01-10T10:30:00Z",
        "expires_at": null,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /users/:unique_id/delegations/:delegation_unique_id - Get Delegation

Retrieves details of a specific delegation.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/$USER_ID/delegations/$DELEGATION_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "delegation-123",
    "type": "user_identity_access_delegation",
    "attributes": {
      "unique_id": "delegation-123",
      "grantor_user_unique_id": "user-123",
      "grantee_user_unique_id": "user-456",
      "grantee_name": "John Assistant",
      "grantee_email": "john.assistant@example.com",
      "status": "active",
      "starts_at": "2025-01-10T10:30:00Z",
      "expires_at": "2025-12-31T23:59:59Z",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Delegation not found

---

### POST /users/:unique_id/delegations - Create Delegation

Creates a new delegation to grant another user access to manage your files.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/$USER_ID/delegations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "delegation": {
      "grantee_user_unique_id": "target-user-id",
      "expires_at": "2025-12-31T23:59:59Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `grantee_user_unique_id` | string | Yes | User to delegate access to |
| `expires_at` | timestamp | No | When delegation expires (null = never) |

**Response 201:**
```json
{
  "data": {
    "id": "new-delegation-id",
    "type": "user_identity_access_delegation",
    "attributes": {
      "unique_id": "new-delegation-id",
      "grantor_user_unique_id": "user-123",
      "grantee_user_unique_id": "target-user-id",
      "status": "active",
      "starts_at": "2025-01-10T10:30:00Z",
      "expires_at": "2025-12-31T23:59:59Z",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### DELETE /users/:unique_id/delegations/:delegation_unique_id - Revoke Delegation

Revokes an existing delegation.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/$USER_ID/delegations/$DELEGATION_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### UserIdentityAccessDelegation
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `grantor_user_unique_id` | uuid | User who granted delegation |
| `grantee_user_unique_id` | uuid | User who received delegation |
| `grantee_name` | string | Name of grantee |
| `grantee_email` | string | Email of grantee |
| `grantor_name` | string | Name of grantor |
| `grantor_email` | string | Email of grantor |
| `status` | enum | active, revoked |
| `starts_at` | timestamp | When delegation starts |
| `expires_at` | timestamp | When delegation expires (null = never) |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## How Delegations Work

When a delegation is active:

1. **File Access**: The grantee can access the grantor's files as if they were their own
2. **Upload**: The grantee can upload files on behalf of the grantor
3. **Manage**: The grantee can update and delete the grantor's files
4. **Access Control**: The grantee can manage access grants for the grantor's files

### Validation

Before any file operation, the system checks:

```ruby
# Is the user accessing their own files?
return true if current_user == file_owner

# Does the user have an active, non-expired delegation?
UserIdentityAccessDelegation.active
  .where(grantee_user_unique_id: current_user)
  .where(grantor_user_unique_id: file_owner)
  .where('expires_at IS NULL OR expires_at > ?', Time.current)
  .exists?
```

---

## Best Practices

### Expiration
- Always set expiration dates for temporary delegations
- Review and clean up expired delegations periodically
- Use short expiration periods for external contractors

### Security
- Only delegate to trusted users
- Monitor delegation activity through audit logs
- Revoke delegations when no longer needed

### Common Patterns

**Assistant Pattern:**
```bash
# Executive delegates to assistant
curl -X POST "$BLOCKS_API_URL/users/executive-id/delegations" \
  -d '{"delegation": {"grantee_user_unique_id": "assistant-id"}}'
```

**Contractor Pattern:**
```bash
# Temporary delegation with 30-day expiration
curl -X POST "$BLOCKS_API_URL/users/user-id/delegations" \
  -d '{
    "delegation": {
      "grantee_user_unique_id": "contractor-id",
      "expires_at": "2025-02-10T23:59:59Z"
    }
  }'
```

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Failed",
    "detail": "Grantee user unique id can't be blank."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-files
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
// DelegationsService — client.files.delegations
list(userUniqueId: string, params?: ListFileDelegationsParams): Promise<PageResult<FileDelegation>>;
get(userUniqueId: string, uniqueId: string): Promise<FileDelegation>;
create(userUniqueId: string, data: CreateFileDelegationRequest): Promise<FileDelegation>;
update(userUniqueId: string, uniqueId: string, data: UpdateFileDelegationRequest): Promise<FileDelegation>;
delete(userUniqueId: string, uniqueId: string): Promise<void>;
listReceivedDelegations(userUniqueId: string): Promise<FileDelegation[]>;
```

### TypeScript Types

```typescript
import type {
  FileDelegation,
  CreateFileDelegationRequest,
  UpdateFileDelegationRequest,
  ListFileDelegationsParams,
} from '@23blocks/block-files';
```

### React Hook

```typescript
import { useFilesBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useFilesBlock();
  const result = await client.files.delegations.list('user-unique-id');
}
```
