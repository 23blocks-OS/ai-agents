---
name: 23blocks-rewards-badges-api
description: "Rewards Block badges: CRUD, categories, award to users. Use for gamification badges."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Badges API

Complete API reference for 23blocks badge management with categories and user awarding for gamification.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://rewards.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"409","code":"already_awarded","title":"Badge Already Awarded","detail":"This badge has already been awarded to the specified customer."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-rewards`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useRewardsBlock()` from `@23blocks/react`.

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
