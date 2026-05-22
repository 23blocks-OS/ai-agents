---
name: 23blocks-content-posts-api
description: Create and manage 23blocks posts via REST API. Use when creating, updating, publishing posts, or managing post social interactions (likes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Posts API

Complete API reference for 23blocks post management with versioning and social interactions.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Content API base URL | `https://content.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token — your identity & scopes (from login or AID token exchange) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | Tenant routing key (X-API-KEY header) — static, from company config | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

**These two credentials serve different purposes and come from different sources:**

| Credential | Purpose | Source | Changes? |
|------------|---------|--------|----------|
| `BLOCKS_API_KEY` | **Tenant routing** — identifies which company/app | Company config (static `pk_live_sh_...` key) | No — same key for all blocks |
| `BLOCKS_AUTH_TOKEN` | **Identity & authorization** — who you are + what you can do | Login (`/auth/sign_in`), AID token exchange, or human-provided | Yes — expires, must be refreshed |

> The API key used during AID registration is NOT the same as `BLOCKS_API_KEY`. The registration key authenticates the agent with the Auth API; `BLOCKS_API_KEY` routes requests to the correct tenant across all blocks.

Two methods to obtain the Bearer token:

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

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/posts` | Public | List all posts |
| POST | `/posts/query` | Public | Query posts with filters |
| GET | `/posts/:unique_id` | Public | Get a single post |
| POST | `/posts` | Bearer | Create a post |
| PUT | `/posts/:unique_id` | Bearer | Update a post |
| PUT | `/posts/:unique_id/replace` | Bearer | Replace post content entirely |
| DELETE | `/posts/:unique_id` | Bearer | Delete a post |
| PUT | `/posts/:unique_id/like` | Bearer | Like a post |
| DELETE | `/posts/:unique_id/dislike` | Bearer | Remove like from post |
| PUT | `/posts/:unique_id/follow` | Bearer | Follow a post |
| DELETE | `/posts/:unique_id/unfollow` | Bearer | Unfollow a post |
| PUT | `/posts/:unique_id/save` | Bearer | Save a post |
| DELETE | `/posts/:unique_id/unsave` | Bearer | Unsave a post |
| PUT | `/posts/:unique_id/own` | Bearer | Transfer post ownership |
| POST | `/posts/:unique_id/versions/:version_id/publish` | Bearer | Publish a version |
| GET | `/posts/:post_unique_id/attachments` | Bearer | List post attachments |
| POST | `/posts/:post_unique_id/attachments` | Bearer | Add attachment to post |
| GET | `/posts/:post_unique_id/attachments/:unique_id` | Bearer | Get attachment |
| PUT | `/posts/:post_unique_id/attachments/:unique_id` | Bearer | Update attachment |
| DELETE | `/posts/:post_unique_id/attachments/:unique_id` | Bearer | Delete attachment |
| PUT | `/posts/:post_unique_id/attachments/reorder` | Bearer | Reorder attachments |

---

## Data Models

### Post
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Post title |
| `content` | string | Post content |
| `status` | enum | draft, published |
| `owner_id` | uuid | Owner user ID |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### PostVersion
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Version identifier |
| `post_id` | uuid | Parent post ID |
| `content` | string | Version content |
| `version_number` | integer | Sequential version number |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Post Not Found",
    "detail": "The requested post could not be found."
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
// PostsService — client.content.posts
list(params?: ListPostsParams): Promise<PageResult<Post>>;
query(params: ListPostsParams): Promise<PageResult<Post>>;
get(uniqueId: string): Promise<Post>;
create(data: CreatePostRequest): Promise<Post>;
update(uniqueId: string, data: UpdatePostRequest): Promise<Post>;
replace(uniqueId: string, data: UpdatePostRequest): Promise<Post>;
delete(uniqueId: string): Promise<void>;
recover(uniqueId: string): Promise<Post>;
search(query: string, params?: ListPostsParams): Promise<PageResult<Post>>;
listDeleted(params?: ListPostsParams): Promise<PageResult<Post>>;
changeOwner(uniqueId: string, newOwnerUniqueId: string): Promise<Post>;
publishVersion(uniqueId: string, versionUniqueId: string): Promise<Post>;
like(uniqueId: string): Promise<Post>;
dislike(uniqueId: string): Promise<Post>;
save(uniqueId: string): Promise<Post>;
unsave(uniqueId: string): Promise<Post>;
follow(uniqueId: string): Promise<Post>;
unfollow(uniqueId: string): Promise<Post>;
validate(uniqueId: string, templateUniqueId: string): Promise<PostValidationResult>;
```

### TypeScript Types

```typescript
import type {
  Post,
  CreatePostRequest,
  UpdatePostRequest,
  ListPostsParams,
} from '@23blocks/block-content';
```

### React Hook

```typescript
import { useContentBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useContentBlock();

  // Example: list all posts with pagination
  const result = await client.content.posts.list({ page: 1, perPage: 20 });
}
```
