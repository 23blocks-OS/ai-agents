---
name: 23blocks-assets-vendors-api
description: "Assets Block vendors (suppliers): CRUD. Use when tracking who supplies assets."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Vendors API

Complete API reference for 23blocks Assets Block vendor management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://assets.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Name can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-assets`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useAssetsBlock()` from `@23blocks/react`.

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
