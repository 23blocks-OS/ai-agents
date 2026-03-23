---
name: 23blocks-rewards-badges-api
description: Manage 23blocks badges via REST API. Use when creating, updating, or deleting badges, organizing badges into categories, or awarding badges to users for gamification.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Badges API

Complete API reference for 23blocks badge management with categories and user awarding for gamification.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://rewards.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/badges` | List all badges |
| GET | `/badges/:unique_id` | Get a badge |
| POST | `/badges/` | Create a new badge |
| PUT | `/badges/` | Update an existing badge |
| DELETE | `/badges/` | Delete a badge |
| POST | `/badges/:unique_id/categories` | Assign category to badge |
| POST | `/badge/` | Award a badge to a user |
| GET | `/categories/` | List badge categories |
| POST | `/categories/` | Create a badge category |

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
