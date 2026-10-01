---
name: 23blocks-files-categories-api
description: "Files Block file categories: create, list, update. Use when categorizing files."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Categories API

Complete API reference for 23blocks file category management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://files.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /categories - List Categories

Lists all categories for the tenant.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "cat-123",
      "type": "category",
      "attributes": {
        "unique_id": "cat-123",
        "name": "Contracts",
        "code": "CONTRACTS",
        "description": "Legal contracts and agreements",
        "status": "active",
        "display_order": 1,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "cat-456",
      "type": "category",
      "attributes": {
        "unique_id": "cat-456",
        "name": "Reports",
        "code": "REPORTS",
        "description": "Monthly and annual reports",
        "status": "active",
        "display_order": 2,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /categories/:unique_id - Get Category

Retrieves a single category by ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories/$CATEGORY_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "cat-123",
    "type": "category",
    "attributes": {
      "unique_id": "cat-123",
      "name": "Contracts",
      "code": "CONTRACTS",
      "description": "Legal contracts and agreements",
      "status": "active",
      "display_order": 1,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Category not found

---

### POST /categories - Create Category

Creates a new category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Invoices",
      "code": "INVOICES",
      "description": "Customer and vendor invoices",
      "display_order": 3
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Category display name |
| `code` | string | Yes | Unique category code (uppercase) |
| `description` | string | No | Category description |
| `display_order` | integer | No | Sort order for display |

**Response 201:**
```json
{
  "data": {
    "id": "new-cat-id",
    "type": "category",
    "attributes": {
      "unique_id": "new-cat-id",
      "name": "Invoices",
      "code": "INVOICES",
      "description": "Customer and vendor invoices",
      "status": "active",
      "display_order": 3,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

---

### PUT /categories/:unique_id - Update Category

Updates an existing category.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/$CATEGORY_ID" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Customer Invoices",
      "description": "Updated description",
      "display_order": 5
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "cat-id",
    "type": "category",
    "attributes": {
      "name": "Customer Invoices",
      "description": "Updated description",
      "display_order": 5,
      "updated_at": "2025-01-10T14:00:00Z"
    }
  }
}
```

---

## Data Models

### Category
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Display name |
| `code` | string | Unique code (uppercase) |
| `description` | string | Description |
| `status` | enum | active, inactive |
| `display_order` | integer | Sort order |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Default Category

If no category is specified when creating a file, the system uses the default "Uncategorized" category:

```json
{
  "name": "Uncategorized",
  "code": "UNCATEGORIZED",
  "description": "Default category for files without a specific category",
  "display_order": 0
}
```

This category is auto-created if it doesn't exist (self-healing for legacy tenants).

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Failed","detail":"Name can't be blank."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-files`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useFilesBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// FileCategoriesService — client.files.fileCategories
list(params?: ListFileCategoriesParams): Promise<PageResult<FileCategory>>;
get(uniqueId: string): Promise<FileCategory>;
create(data: CreateFileCategoryRequest): Promise<FileCategory>;
update(uniqueId: string, data: UpdateFileCategoryRequest): Promise<FileCategory>;
delete(uniqueId: string): Promise<void>;
listChildren(parentUniqueId: string): Promise<FileCategory[]>;
```

### TypeScript Types

```typescript
import type {
  FileCategory,
  CreateFileCategoryRequest,
  UpdateFileCategoryRequest,
  ListFileCategoriesParams,
} from '@23blocks/block-files';
```
