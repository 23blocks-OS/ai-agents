---
name: 23blocks-university-categories-api
description: "University Block hierarchical categories with parent-child links. Use when organizing university content into categories."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Categories API

Complete API reference for 23blocks university category management with hierarchical organization.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /categories/ - List Categories

Lists all categories.

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
      "id": "category-uuid-123",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-123",
        "name": "STEM",
        "description": "Science, Technology, Engineering, and Mathematics",
        "parent_id": null,
        "order": 1,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "category-uuid-456",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-456",
        "name": "Computer Science",
        "description": "Programming, algorithms, and systems",
        "parent_id": "category-uuid-123",
        "order": 1,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /categories/:unique_id/ - Get Category with Children

Retrieves a single category by unique ID, including child categories.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories/category-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "category-uuid-123",
    "type": "category",
    "attributes": {
      "unique_id": "category-uuid-123",
      "name": "STEM",
      "description": "Science, Technology, Engineering, and Mathematics",
      "parent_id": null,
      "order": 1,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "children": {
        "data": [
          { "id": "category-uuid-456", "type": "category" },
          { "id": "category-uuid-789", "type": "category" }
        ]
      }
    }
  },
  "included": [
    {
      "id": "category-uuid-456",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-456",
        "name": "Computer Science",
        "description": "Programming, algorithms, and systems",
        "parent_id": "category-uuid-123",
        "order": 1,
        "status": "active"
      }
    },
    {
      "id": "category-uuid-789",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-789",
        "name": "Mathematics",
        "description": "Pure and applied mathematics",
        "parent_id": "category-uuid-123",
        "order": 2,
        "status": "active"
      }
    }
  ]
}
```

**Errors:**
- `404 Not Found` - Category not found

---

### POST /categories/ - Create Category

Creates a new category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Humanities",
      "description": "Arts, literature, history, and philosophy",
      "parent_id": null,
      "order": 2,
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Category name |
| `description` | string | No | Category description |
| `parent_id` | uuid | No | Parent category ID (null for root) |
| `order` | integer | No | Display order within parent |
| `status` | enum | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "new-category-uuid",
    "type": "category",
    "attributes": {
      "unique_id": "new-category-uuid",
      "name": "Humanities",
      "description": "Arts, literature, history, and philosophy",
      "parent_id": null,
      "order": 2,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### Creating Nested Categories

To create a child category under an existing parent:

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Data Science",
      "description": "Data analysis, machine learning, and AI",
      "parent_id": "category-uuid-123",
      "order": 3
    }
  }'
```

---

### PUT /categories/:unique_id/ - Update Category

Updates an existing category.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/categories/category-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Computer Science & Engineering",
      "description": "Updated description for CS department",
      "order": 1
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "category-uuid-456",
    "type": "category",
    "attributes": {
      "unique_id": "category-uuid-456",
      "name": "Computer Science & Engineering",
      "description": "Updated description for CS department",
      "parent_id": "category-uuid-123",
      "order": 1,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Category not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /categories/:unique_id/ - Delete Category

Deletes a category.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/categories/category-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Category not found
- `422 Unprocessable Entity` - Cannot delete category with children

---

## Data Models

### Category
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Category name |
| `description` | string | Category description |
| `parent_id` | uuid | Parent category ID (null for root) |
| `order` | integer | Display order within parent |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Name can't be blank."}]}`.
