---
name: auth-org-members-api
description: Manage 23blocks organization members via REST API. Use when listing organization members, adding or removing members, updating member roles, or transferring organization ownership.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Organization Members API

Complete API reference for 23blocks organization membership management.

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
curl -X GET "$BLOCKS_API_URL/organizations/{org_id}/members" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /organizations/:org_id/members - List Members

Lists all members of an organization.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/organizations/org-uuid-123/members?page=1&records=20" \
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
      "type": "org_member",
      "attributes": {
        "unique_id": "member-uuid-123",
        "user_id": "user-uuid-123",
        "email": "member@example.com",
        "first_name": "John",
        "last_name": "Doe",
        "role": "admin",
        "status": "active",
        "joined_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 25
  }
}
```

---

### POST /organizations/:org_id/members - Add Member

Adds a user as a member to the organization.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/organizations/org-uuid-123/members" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
| `role` | string | No | Role in org (default: member) |

**Response 201:**
```json
{
  "data": {
    "id": "member-uuid-456",
    "type": "org_member",
    "attributes": {
      "unique_id": "member-uuid-456",
      "user_id": "user-uuid-456",
      "role": "member",
      "status": "active",
      "joined_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User or organization not found
- `409 Conflict` - User is already a member
- `422 Unprocessable Entity` - Validation errors

---

### PUT /organizations/:org_id/members/:user_id - Update Member Role

Updates a member's role within the organization.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/organizations/org-uuid-123/members/user-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "member": {
      "role": "admin"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "member-uuid-456",
    "type": "org_member",
    "attributes": {
      "unique_id": "member-uuid-456",
      "user_id": "user-uuid-456",
      "role": "admin",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Member not found

---

### DELETE /organizations/:org_id/members/:user_id - Remove Member

Removes a member from the organization.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/organizations/org-uuid-123/members/user-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Cannot remove the owner
- `404 Not Found` - Member not found

---

### POST /organizations/:org_id/transfer_ownership - Transfer Ownership

Transfers organization ownership to another member.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/organizations/org-uuid-123/transfer_ownership" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "new_owner_id": "user-uuid-456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `new_owner_id` | uuid | Yes | New owner's user ID |

**Response 200:**
```json
{
  "data": {
    "id": "org-uuid-123",
    "type": "organization",
    "attributes": {
      "unique_id": "org-uuid-123",
      "name": "Acme Corp"
    },
    "relationships": {
      "owner": {
        "data": { "id": "user-uuid-456", "type": "user" }
      }
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Only the current owner can transfer
- `404 Not Found` - New owner not a member
- `422 Unprocessable Entity` - Cannot transfer to self

---

## Data Models

### OrgMember
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Membership identifier |
| `user_id` | uuid | User identifier |
| `email` | string | Member email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `role` | string | Role within organization |
| `status` | enum | active, inactive |
| `joined_at` | timestamp | When member joined |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "403",
    "code": "forbidden",
    "title": "Insufficient Permissions",
    "detail": "You do not have permission to manage members in this organization."
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
// TenantUsersService — client.authentication.tenantUsers
current(): Promise<TenantUser>;
get(userUniqueId: string): Promise<TenantUser>;
list(params?: ListParams): Promise<TenantUser[]>;

// TenantsService (member management) — client.authentication.tenants
createTenantUser(userUniqueId: string, request: CreateTenantUserRequest): Promise<TenantUserFull>;
updateOnboarding(userUniqueId: string, urlId: string, request: UpdateTenantUserOnboardingRequest): Promise<TenantUserFull>;
updateSales(userUniqueId: string, urlId: string, request: UpdateTenantUserSalesRequest): Promise<TenantUserFull>;
```

### TypeScript Types

```typescript
import type {
  TenantUser,
  TenantUserFull,
  CreateTenantUserRequest,
  UpdateTenantUserOnboardingRequest,
  UpdateTenantUserSalesRequest,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: Get current tenant user context
  const tenantUser = await client.authentication.tenantUsers.current();
}
```
