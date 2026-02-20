---
name: products-shopping-lists-api
description: Manage 23blocks user shopping lists via REST API. Use when creating, updating, deleting shopping lists, or adding and removing products from shopping lists.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Shopping Lists API

Complete API reference for 23blocks user shopping list management.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://products.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Products API base URL | `https://products.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /users/:unique_id/shoppinglists - List Shopping Lists

Lists all shopping lists for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "sl-uuid-123",
      "type": "shopping_list",
      "attributes": {
        "unique_id": "sl-uuid-123",
        "user_unique_id": "user-uuid-123",
        "name": "Weekly Groceries",
        "description": "Regular weekly shopping items",
        "products_count": 12,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /users/:unique_id/shoppinglists - Create Shopping List

Creates a new shopping list for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "shopping_list": {
      "name": "Holiday Gift Ideas",
      "description": "Gift ideas for the holiday season"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Shopping list name |
| `description` | string | No | Shopping list description |

**Response 201:**
```json
{
  "data": {
    "id": "new-sl-uuid",
    "type": "shopping_list",
    "attributes": {
      "unique_id": "new-sl-uuid",
      "user_unique_id": "user-uuid-123",
      "name": "Holiday Gift Ideas",
      "description": "Gift ideas for the holiday season",
      "products_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### GET /users/:unique_id/shoppinglists/:sl_unique_id - Get Shopping List

Retrieves a specific shopping list with its products.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists/sl-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "sl-uuid-123",
    "type": "shopping_list",
    "attributes": {
      "unique_id": "sl-uuid-123",
      "user_unique_id": "user-uuid-123",
      "name": "Weekly Groceries",
      "description": "Regular weekly shopping items",
      "products_count": 12,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "products": {
        "data": [
          { "id": "product-uuid-001", "type": "product" },
          { "id": "product-uuid-002", "type": "product" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Shopping list not found

---

### PUT /users/:unique_id/shoppinglists/:sl_unique_id - Update Shopping List

Updates a shopping list.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists/sl-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "shopping_list": {
      "name": "Updated Weekly Groceries",
      "description": "Updated shopping items"
    }
  }'
```

**Response 200:** Updated shopping list object

---

### DELETE /users/:unique_id/shoppinglists/:sl_unique_id - Delete Shopping List

Deletes a shopping list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists/sl-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Shopping List Products

### POST /users/:unique_id/shoppinglists/:sl_unique_id/products - Add Product

Adds a product to a shopping list.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists/sl-uuid-123/products" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "product_unique_id": "product-uuid-001",
    "quantity": 2,
    "notes": "Buy the large size"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `product_unique_id` | uuid | Yes | Product identifier |
| `quantity` | integer | No | Desired quantity (default: 1) |
| `notes` | string | No | Additional notes |

**Response 200:**
```json
{
  "message": "Product added to shopping list successfully"
}
```

---

### DELETE /users/:unique_id/shoppinglists/:sl_unique_id/products - Remove Product

Removes a product from a shopping list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/users/user-uuid-123/shoppinglists/sl-uuid-123/products" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "product_unique_id": "product-uuid-001"
  }'
```

**Response 200:**
```json
{
  "message": "Product removed from shopping list successfully"
}
```

---

## Data Models

### ShoppingList
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_unique_id` | uuid | Owner user ID |
| `name` | string | Shopping list name |
| `description` | string | Shopping list description |
| `products_count` | integer | Number of products |
| `status` | enum | active, archived |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### ShoppingListProduct
| Field | Type | Description |
|-------|------|-------------|
| `product_unique_id` | uuid | Product identifier |
| `quantity` | integer | Desired quantity |
| `notes` | string | Additional notes |
| `added_at` | timestamp | When product was added |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Shopping List Not Found",
    "detail": "The requested shopping list could not be found."
  }]
}
```
