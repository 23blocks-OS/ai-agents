---
name: 23blocks-rewards-categories-api
description: "Rewards Block badge categories: list, get, create. Use when grouping badges."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Categories API

Complete API reference for 23blocks badge category management to organize badges into logical groups.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://rewards.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /categories/ - List Categories

Lists all badge categories.

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
      "id": "category-uuid-1",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-1",
        "name": "Shopping",
        "description": "Badges earned through purchases",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "category-uuid-2",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-2",
        "name": "Engagement",
        "description": "Badges earned through community participation",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    },
    {
      "id": "category-uuid-3",
      "type": "category",
      "attributes": {
        "unique_id": "category-uuid-3",
        "name": "Milestones",
        "description": "Badges for reaching account milestones",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /categories/:unique_id/ - Get Category

Retrieves a single category by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories/category-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "category-uuid-1",
    "type": "category",
    "attributes": {
      "unique_id": "category-uuid-1",
      "name": "Shopping",
      "description": "Badges earned through purchases",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "badges": {
        "data": [
          { "id": "badge-uuid-1", "type": "badge" },
          { "id": "badge-uuid-2", "type": "badge" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Category not found

---

### POST /categories/ - Create Category

Creates a new badge category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category": {
      "name": "Shopping",
      "description": "Badges earned through purchases"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Category name |
| `description` | string | No | Category description |

**Response 201:**
```json
{
  "data": {
    "id": "new-category-uuid",
    "type": "category",
    "attributes": {
      "unique_id": "new-category-uuid",
      "name": "Shopping",
      "description": "Badges earned through purchases",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Category name already exists
- `422 Unprocessable Entity` - Validation errors

---

## Data Models

### Category
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Category name |
| `description` | string | Category description |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"409","code":"duplicate_name","title":"Category Already Exists","detail":"A category with this name already exists."}]}`.
