---
name: 23blocks-content-identities-api
description: Manage 23blocks user identities via REST API. Use when registering users, managing user profiles, handling social following, or retrieving user contributions and activities.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks user identity management with social following and activity tracking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Content API base URL | `https://content.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://content.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://content.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/identities` | List all identities |
| GET | `/identities/:unique_id` | Get a single identity |
| POST | `/identities/:unique_id/register` | Register a user |
| PUT | `/identities/:unique_id` | Update user profile |
| POST | `/identities/:unique_id/tags` | Add tag to user |
| DELETE | `/identities/:unique_id/tags/:tag_id` | Remove tag from user |
| GET | `/identities/:unique_id/drafts` | Get user drafts |
| GET | `/identities/:unique_id/posts` | Get user posts |
| GET | `/identities/:unique_id/comments` | Get user comments |
| GET | `/identities/:unique_id/followers` | Get user followers |
| GET | `/identities/:unique_id/following` | Get users being followed |
| POST | `/identities/:unique_id/follows/:user_id` | Follow a user |
| DELETE | `/identities/:unique_id/unfollows/:user_id` | Unfollow a user |
| GET | `/identities/:unique_id/activities` | Get user activities |

---

## Data Models

### UserIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `username` | string | Username |
| `display_name` | string | Display name |
| `avatar_url` | string | Avatar image URL |
| `bio` | string | User biography |
| `posts_count` | integer | Number of posts |
| `comments_count` | integer | Number of comments |
| `followers_count` | integer | Number of followers |
| `following_count` | integer | Number of users following |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Activity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `activity_type` | string | Type of activity |
| `description` | string | Activity description |
| `target_id` | uuid | Related entity ID |
| `target_type` | string | Related entity type |
| `created_at` | timestamp | Activity time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "User Not Found",
    "detail": "The requested user could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-content
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
// ContentUsersService — client.content.users
list(params?: ListContentUsersParams): Promise<PageResult<ContentUser>>;
get(uniqueId: string): Promise<ContentUser>;
register(uniqueId: string, data: RegisterContentUserRequest): Promise<ContentUser>;
update(uniqueId: string, data: UpdateContentUserRequest): Promise<ContentUser>;
getDrafts(uniqueId: string): Promise<Post[]>;
getPosts(uniqueId: string): Promise<Post[]>;
getComments(uniqueId: string): Promise<Comment[]>;
getActivities(uniqueId: string): Promise<UserActivity[]>;
addTag(uniqueId: string, tagUniqueId: string): Promise<ContentUser>;
removeTag(uniqueId: string, tagUniqueId: string): Promise<void>;
getFollowers(uniqueId: string): Promise<Following[]>;
getFollowing(uniqueId: string): Promise<Following[]>;
followUser(uniqueId: string, targetUserUniqueId: string): Promise<void>;
unfollowUser(uniqueId: string, targetUserUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  ContentUser,
  Following,
  RegisterContentUserRequest,
  UpdateContentUserRequest,
  ListContentUsersParams,
  UserActivity,
} from '@23blocks/block-content';
```

### React Hook

```typescript
import { useContentBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useContentBlock();

  // Example: get a user's profile
  const result = await client.content.users.get('user-unique-id');
}
```
