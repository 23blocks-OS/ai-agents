---
name: 23blocks-products-identities-api
description: "Products Block user identities: register, profiles, a user's reviews. Use before a user's first Products call."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks product user identity management.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://products.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /users/:unique_id/ - Get User

Retrieves a user profile by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "UserIdentity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "display_name": "John Doe",
      "phone": "+1234567890",
      "avatar_url": "https://example.com/avatar.jpg",
      "status": "active",
      "reviews_count": 12,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /users/:unique_id/register - Register User

Registers a new user in the products system. Each block is autonomous; users must register before using private endpoints. The `user_unique_id` (the `:unique_id` path parameter) is the only required value.

> **Note:** Block identity records are notification routing caches, not identity models. The canonical user record lives in the Auth (Gateway) block. `email`/`phone` here are optional denormalized routing fields; duplicates across users are allowed.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "display_name": "Jane Smith"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | No | Optional denormalized routing field; if blank, the block skips email notifications |
| `first_name` | string | No | First name |
| `last_name` | string | No | Last name |
| `display_name` | string | No | Display name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "UserIdentity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "first_name": "Jane",
      "last_name": "Smith",
      "display_name": "Jane Smith",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Missing `user_unique_id`

---

### PUT /users/:unique_id/ - Update User

Updates an existing user profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
      "phone": "+1987654321",
      "avatar_url": "https://example.com/new-avatar.jpg"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "UserIdentity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "display_name": "John D.",
      "phone": "+1987654321",
      "avatar_url": "https://example.com/new-avatar.jpg",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### GET /users/:user_id/reviews - Get User Reviews

Retrieves all reviews submitted by the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/reviews" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "review-uuid-456",
      "type": "Review",
      "attributes": {
        "unique_id": "review-uuid-456",
        "rating": 5,
        "title": "Excellent product",
        "body": "Really happy with this purchase.",
        "status": "published",
        "created_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "product": {
          "data": { "id": "product-uuid", "type": "Product" }
        }
      }
    }
  ],
  "meta": {
    "total_count": 12
  }
}
```

---

## Data Models

### User
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `display_name` | string | Display name |
| `phone` | string | Phone number |
| `avatar_url` | string | Avatar image URL |
| `status` | string | active, inactive |
| `reviews_count` | integer | Number of reviews |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"User Not Found","detail":"The requested user could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-products`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// GuestsService — client.products.guests (preferred)
create(data: CreateGuestRequest): Promise<Guest>;
get(userUniqueId: string): Promise<Guest>;
update(userUniqueId: string, data: UpdateGuestRequest): Promise<Guest>;
convert(uniqueId: string): Promise<User>;
auth(uniqueId: string): Promise<AuthToken>;

// VisitorsService — client.products.visitors (legacy alias for guests)
create(data: CreateVisitorRequest): Promise<Visitor>;

// ProductReviewsService — client.products.reviews (user reviews)
listByUser(userUniqueId: string, page?: number, perPage?: number): Promise<PageResult<ProductReview>>;
```

### TypeScript Types

```typescript
import type {
  ProductReview,
} from '@23blocks/block-products';
```

### React Hook

```typescript
import { useProductsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useProductsBlock();

  // Example: create a guest session (preferred over legacy visitors)
  const guest = await client.products.guests.create({ email: 'guest@example.com', name: 'Jane Doe' });
}
```
