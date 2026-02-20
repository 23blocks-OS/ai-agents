---
name: auth-organizations-api
description: Manage 23blocks organizations via REST API. Use when creating, updating, or deleting organizations, managing organization settings, billing configuration, or viewing organization statistics.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Organizations API

Complete API reference for 23blocks organization management with settings and billing.

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
curl -X GET "$BLOCKS_API_URL/organizations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /organizations - List Organizations

Lists all organizations the user has access to.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/organizations?page=1&records=20" \
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
      "id": "org-uuid-123",
      "type": "organization",
      "attributes": {
        "unique_id": "org-uuid-123",
        "name": "Acme Corp",
        "slug": "acme-corp",
        "plan": "pro",
        "members_count": 25,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 2
  }
}
```

---

### GET /organizations/:unique_id - Get Organization

Retrieves a single organization by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/organizations/org-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "org-uuid-123",
    "type": "organization",
    "attributes": {
      "unique_id": "org-uuid-123",
      "name": "Acme Corp",
      "slug": "acme-corp",
      "description": "Leading technology company",
      "logo_url": "https://example.com/logo.png",
      "plan": "pro",
      "members_count": 25,
      "teams_count": 5,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "owner": {
        "data": { "id": "user-uuid", "type": "user" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Organization not found

---

### POST /organizations - Create Organization

Creates a new organization.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/organizations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "organization": {
      "name": "New Corp",
      "slug": "new-corp",
      "description": "A new organization"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Organization name |
| `slug` | string | No | URL-friendly slug |
| `description` | string | No | Description |

**Response 201:**
```json
{
  "data": {
    "id": "new-org-uuid",
    "type": "organization",
    "attributes": {
      "unique_id": "new-org-uuid",
      "name": "New Corp",
      "slug": "new-corp",
      "description": "A new organization",
      "plan": "free",
      "members_count": 1,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Slug already taken
- `422 Unprocessable Entity` - Validation errors

---

### PUT /organizations/:unique_id - Update Organization

Updates an existing organization.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/organizations/org-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "organization": {
      "name": "Acme Corporation",
      "description": "Updated description"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "org-uuid-123",
    "type": "organization",
    "attributes": {
      "unique_id": "org-uuid-123",
      "name": "Acme Corporation",
      "description": "Updated description",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /organizations/:unique_id - Delete Organization

Deletes an organization. Only the owner can delete.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/organizations/org-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Only the owner can delete
- `404 Not Found` - Organization not found

---

### GET /organizations/:unique_id/settings - Get Organization Settings

Retrieves organization settings.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/organizations/org-uuid-123/settings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "org-uuid-123",
    "type": "organization_settings",
    "attributes": {
      "allow_public_signup": false,
      "require_mfa": true,
      "default_role": "member",
      "session_timeout_minutes": 60,
      "allowed_email_domains": ["acme.com"],
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /organizations/:unique_id/settings - Update Organization Settings

Updates organization settings.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/organizations/org-uuid-123/settings" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "settings": {
      "require_mfa": true,
      "session_timeout_minutes": 30
    }
  }'
```

**Response 200:** Same format as GET settings

---

### GET /organizations/:unique_id/billing - Get Billing Info

Retrieves organization billing information.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/organizations/org-uuid-123/billing" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "org-uuid-123",
    "type": "organization_billing",
    "attributes": {
      "plan": "pro",
      "billing_email": "billing@acme.com",
      "billing_cycle": "monthly",
      "current_period_start": "2025-01-01T00:00:00Z",
      "current_period_end": "2025-02-01T00:00:00Z",
      "seats_used": 25,
      "seats_limit": 50
    }
  }
}
```

---

### PUT /organizations/:unique_id/billing - Update Billing Info

Updates organization billing configuration.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/organizations/org-uuid-123/billing" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "billing": {
      "billing_email": "finance@acme.com",
      "billing_cycle": "annual"
    }
  }'
```

**Response 200:** Same format as GET billing

---

### GET /organizations/:unique_id/stats - Get Organization Statistics

Retrieves usage statistics for an organization.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/organizations/org-uuid-123/stats" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "org-uuid-123",
    "type": "organization_stats",
    "attributes": {
      "total_members": 25,
      "active_members": 22,
      "total_teams": 5,
      "total_api_keys": 8,
      "total_invitations_pending": 3,
      "api_requests_this_month": 150000
    }
  }
}
```

---

## Data Models

### Organization
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Organization name |
| `slug` | string | URL-friendly slug |
| `description` | string | Description |
| `logo_url` | string | Logo image URL |
| `plan` | enum | free, pro, enterprise |
| `members_count` | integer | Total members |
| `teams_count` | integer | Total teams |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### OrganizationSettings
| Field | Type | Description |
|-------|------|-------------|
| `allow_public_signup` | boolean | Allow public registration |
| `require_mfa` | boolean | Require MFA for all users |
| `default_role` | string | Default role for new members |
| `session_timeout_minutes` | integer | Session timeout |
| `allowed_email_domains` | array | Whitelisted email domains |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "conflict",
    "title": "Slug Already Taken",
    "detail": "The organization slug 'acme-corp' is already in use."
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
// TenantsService — client.authentication.tenants
listChildren(params?: ListParams): Promise<PageResult<Company>>;
validateCode(request: ValidateTenantCodeRequest): Promise<ValidateTenantCodeResponse>;
searchByName(request: SearchTenantRequest): Promise<Company>;
searchByCode(request: SearchTenantRequest): Promise<Company>;
createTenantUser(userUniqueId: string, request: CreateTenantUserRequest): Promise<TenantUserFull>;
updateOnboarding(userUniqueId: string, urlId: string, request: UpdateTenantUserOnboardingRequest): Promise<TenantUserFull>;
updateSales(userUniqueId: string, urlId: string, request: UpdateTenantUserSalesRequest): Promise<TenantUserFull>;
```

### TypeScript Types

```typescript
import type {
  Company,
  CompanyDetail,
  CompanyBlock,
  CompanyKey,
  Tenant,
  TenantUserFull,
  CreateTenantUserRequest,
  ValidateTenantCodeRequest,
  ValidateTenantCodeResponse,
  SearchTenantRequest,
  UpdateTenantUserOnboardingRequest,
  UpdateTenantUserSalesRequest,
} from '@23blocks/block-authentication';
```

### React Hook

```typescript
import { useAuthenticationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAuthenticationBlock();

  // Example: List child organizations (tenants)
  const result = await client.authentication.tenants.listChildren({ page: 1, perPage: 20 });
}
```
