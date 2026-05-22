---
name: 23blocks-assets-vendors-api
description: Create and manage 23blocks asset vendors via REST API. Use when creating, listing, updating, or deleting vendors (suppliers) for asset tracking.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Vendors API

Complete API reference for 23blocks Assets Block vendor management.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://assets.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://assets.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /vendors/ - List Vendors

Lists all vendors.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/vendors" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "vendor-uuid-123",
      "type": "vendor",
      "attributes": {
        "unique_id": "vendor-uuid-123",
        "name": "Dell Technologies",
        "description": "Computer hardware supplier",
        "contact_name": "Jane Smith",
        "contact_email": "jane@dell.example.com",
        "contact_phone": "+1-555-0100",
        "address": "1 Dell Way, Round Rock, TX",
        "website": "https://dell.example.com",
        "assets_count": 35,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /vendors/:unique_id/ - Get Vendor

Retrieves a single vendor by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/vendors/vendor-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "vendor-uuid-123",
    "type": "vendor",
    "attributes": {
      "unique_id": "vendor-uuid-123",
      "name": "Dell Technologies",
      "description": "Computer hardware supplier",
      "contact_name": "Jane Smith",
      "contact_email": "jane@dell.example.com",
      "contact_phone": "+1-555-0100",
      "address": "1 Dell Way, Round Rock, TX",
      "website": "https://dell.example.com",
      "assets_count": 35,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Vendor not found

---

### POST /vendors/ - Create Vendor

Creates a new vendor.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/vendors" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vendor": {
      "code": "HP",
      "name": "HP Inc.",
      "description": "Printers and computing hardware",
      "contact_name": "Bob Jones",
      "contact_email": "bob@hp.example.com",
      "contact_phone": "+1-555-0200",
      "address": "1501 Page Mill Road, Palo Alto, CA",
      "website": "https://hp.example.com"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Vendor code (unique identifier) |
| `name` | string | Yes | Vendor name |
| `description` | string | No | Vendor description |
| `contact_name` | string | No | Primary contact name |
| `contact_email` | string | No | Contact email |
| `contact_phone` | string | No | Contact phone number |
| `address` | string | No | Vendor address |
| `website` | string | No | Vendor website URL |

**Response 201:**
```json
{
  "data": {
    "id": "new-vendor-uuid",
    "type": "vendor",
    "attributes": {
      "unique_id": "new-vendor-uuid",
      "name": "HP Inc.",
      "description": "Printers and computing hardware",
      "contact_name": "Bob Jones",
      "contact_email": "bob@hp.example.com",
      "assets_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /vendors/:unique_id - Update Vendor

Updates an existing vendor.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/vendors/vendor-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vendor": {
      "contact_name": "Updated Contact",
      "contact_email": "updated@dell.example.com"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "vendor-uuid-123",
    "type": "vendor",
    "attributes": {
      "unique_id": "vendor-uuid-123",
      "name": "Dell Technologies",
      "contact_name": "Updated Contact",
      "contact_email": "updated@dell.example.com",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Vendor not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /vendors/:unique_id - Delete Vendor

Deletes a vendor.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/vendors/vendor-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** Returns `{}` with status 204

**Errors:**
- `404 Not Found` - Vendor not found

---

## Data Models

### Vendor
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `code` | string | Vendor code (unique identifier) |
| `name` | string | Vendor name |
| `description` | string | Vendor description |
| `contact_name` | string | Primary contact name |
| `contact_email` | string | Contact email |
| `contact_phone` | string | Contact phone |
| `address` | string | Vendor address |
| `website` | string | Vendor website URL |
| `assets_count` | integer | Number of assets from this vendor |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "Name can't be blank."
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
// VendorsService — client.assets.vendors
client.assets.vendors.list(params?: ListVendorsParams): Promise<PageResult<Vendor>>;
client.assets.vendors.get(uniqueId: string): Promise<Vendor>;
client.assets.vendors.create(data: CreateVendorRequest): Promise<Vendor>;
client.assets.vendors.update(uniqueId: string, data: UpdateVendorRequest): Promise<Vendor>;
client.assets.vendors.delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Vendor,
  CreateVendorRequest,
  UpdateVendorRequest,
  ListVendorsParams,
} from '@23blocks/block-assets';
```

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.vendors.list();
}
```
