---
name: rewards-badges-api
description: Manage 23blocks badges via REST API. Use when creating, updating, or deleting badges, organizing badges into categories, or awarding badges to users for gamification.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Badges API

Complete API reference for 23blocks badge management with categories and user awarding for gamification.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://rewards.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/badges" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /badges - List Badges

Lists all badges with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/badges?page=1&records=20" \
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
      "id": "badge-uuid-123",
      "type": "badge",
      "attributes": {
        "unique_id": "badge-uuid-123",
        "name": "First Purchase",
        "description": "Awarded after first purchase",
        "image_url": "https://cdn.example.com/badges/first-purchase.png",
        "criteria": "Complete first transaction",
        "points_value": 50,
        "category_id": "category-uuid-1",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 24
  }
}
```

---

### GET /badges/:unique_id - Get Badge

Retrieves a single badge by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/badges/badge-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "badge-uuid-123",
    "type": "badge",
    "attributes": {
      "unique_id": "badge-uuid-123",
      "name": "First Purchase",
      "description": "Awarded after first purchase",
      "image_url": "https://cdn.example.com/badges/first-purchase.png",
      "criteria": "Complete first transaction",
      "points_value": 50,
      "category_id": "category-uuid-1",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "category": {
        "data": { "id": "category-uuid-1", "type": "category" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Badge not found

---

### POST /badges/ - Create Badge

Creates a new badge.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/badges" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "badge": {
      "name": "First Purchase",
      "description": "Awarded after first purchase",
      "image_url": "https://cdn.example.com/badges/first-purchase.png",
      "criteria": "Complete first transaction",
      "points_value": 50,
      "category_id": "category-uuid-1"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Badge name |
| `description` | string | No | Badge description |
| `image_url` | string | No | URL to badge image |
| `criteria` | string | No | Earning criteria description |
| `points_value` | integer | No | Points awarded with badge |
| `category_id` | uuid | No | Badge category ID |

**Response 201:**
```json
{
  "data": {
    "id": "new-badge-uuid",
    "type": "badge",
    "attributes": {
      "unique_id": "new-badge-uuid",
      "name": "First Purchase",
      "description": "Awarded after first purchase",
      "image_url": "https://cdn.example.com/badges/first-purchase.png",
      "criteria": "Complete first transaction",
      "points_value": 50,
      "category_id": "category-uuid-1",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /badges/ - Update Badge

Updates an existing badge.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/badges" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "badge": {
      "unique_id": "badge-uuid-123",
      "name": "Super Shopper",
      "points_value": 100
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "badge-uuid-123",
    "type": "badge",
    "attributes": {
      "unique_id": "badge-uuid-123",
      "name": "Super Shopper",
      "points_value": 100,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Badge not found

---

### DELETE /badges/ - Delete Badge

Deletes a badge.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/badges" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "badge": {
      "unique_id": "badge-uuid-123"
    }
  }'
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Badge not found

---

### POST /badges/:unique_id/categories - Assign Category

Assigns a category to a badge.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/badges/badge-uuid-123/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category_unique_id": "category-uuid-1"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `category_unique_id` | uuid | Yes | Category to assign |

**Response 200:**
```json
{
  "data": {
    "id": "badge-uuid-123",
    "type": "badge",
    "attributes": {
      "unique_id": "badge-uuid-123",
      "category_id": "category-uuid-1",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### POST /badge/ - Award Badge to User

Awards a badge to a specific user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/badge" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "customer_unique_id": "customer-uuid",
    "badge_unique_id": "badge-uuid-123"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `customer_unique_id` | uuid | Yes | Customer to award badge to |
| `badge_unique_id` | uuid | Yes | Badge to award |

**Response 201:**
```json
{
  "data": {
    "id": "award-uuid-123",
    "type": "badge_award",
    "attributes": {
      "unique_id": "award-uuid-123",
      "customer_unique_id": "customer-uuid",
      "badge_unique_id": "badge-uuid-123",
      "points_awarded": 50,
      "awarded_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Customer or badge not found
- `409 Conflict` - Badge already awarded to customer

---

## Badge Categories

### GET /categories/ - List Badge Categories

Lists all badge categories.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
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
    }
  ]
}
```

---

### POST /categories/ - Create Category

Creates a new badge category.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
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
- `422 Unprocessable Entity` - Validation errors

---

## Data Models

### Badge
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Badge name |
| `description` | string | Badge description |
| `image_url` | string | URL to badge image |
| `criteria` | string | Earning criteria description |
| `points_value` | integer | Points awarded with badge |
| `category_id` | uuid | Badge category ID |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Category
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Category name |
| `description` | string | Category description |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

### BadgeAward
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `customer_unique_id` | uuid | Customer ID |
| `badge_unique_id` | uuid | Badge ID |
| `points_awarded` | integer | Points given with award |
| `awarded_at` | timestamp | Award time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "already_awarded",
    "title": "Badge Already Awarded",
    "detail": "This badge has already been awarded to the specified customer."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-rewards
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
// BadgesService — client.rewards.badges
client.rewards.badges.list(params?: ListBadgesParams): Promise<PageResult<Badge>>;
client.rewards.badges.get(uniqueId: string): Promise<Badge>;
client.rewards.badges.create(data: CreateBadgeRequest): Promise<Badge>;
client.rewards.badges.update(uniqueId: string, data: UpdateBadgeRequest): Promise<Badge>;
client.rewards.badges.delete(uniqueId: string): Promise<void>;
client.rewards.badges.award(data: AwardBadgeRequest): Promise<UserBadge>;
client.rewards.badges.listByUser(userUniqueId: string, params?: ListUserBadgesParams): Promise<PageResult<UserBadge>>;
```

### TypeScript Types

```typescript
import type {
  Badge,
  UserBadge,
  CreateBadgeRequest,
  UpdateBadgeRequest,
  ListBadgesParams,
  AwardBadgeRequest,
  ListUserBadgesParams,
} from '@23blocks/block-rewards';
```

### React Hook

```typescript
import { useRewardsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useRewardsBlock();
  const result = await client.rewards.badges.list();
}
```
