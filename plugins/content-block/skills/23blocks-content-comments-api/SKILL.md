---
name: 23blocks-content-comments-api
description: Create and manage 23blocks comments via REST API. Use when adding comments to posts, creating replies, or managing comment social interactions (likes, dislikes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Comments API

Complete API reference for 23blocks comment management with threading and social interactions.

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
| GET | `/posts/:post_id/comments` | Public | List comments for a post |
| GET | `/posts/:post_id/comments/:unique_id` | Bearer | Get a single comment |
| POST | `/posts/:post_id/comments` | Bearer | Create a comment |
| PUT | `/posts/:post_id/comments/:unique_id` | Bearer | Update a comment |
| DELETE | `/posts/:post_id/comments/:unique_id` | Bearer | Delete a comment |
| POST | `/posts/:post_id/comments/:unique_id/reply` | Bearer | Reply to a comment |
| PUT | `/posts/:post_id/comments/:unique_id/like` | Bearer | Like a comment |
| PUT | `/posts/:post_id/comments/:unique_id/dislike` | Bearer | Dislike a comment |
| PUT | `/posts/:post_id/comments/:unique_id/follow` | Bearer | Follow a comment |
| DELETE | `/posts/:post_id/comments/:unique_id/unfollow` | Bearer | Unfollow a comment |
| PUT | `/posts/:post_id/comments/:unique_id/save` | Bearer | Save a comment |
| DELETE | `/posts/:post_id/comments/:unique_id/unsave` | Bearer | Unsave a comment |
| DELETE | `/posts/:post_id/comments/:unique_id/moderate` | Bearer | Moderate (remove) a comment |

---

## Data Models

### Comment
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `body` | string | Comment content |
| `parent_id` | uuid | Parent comment ID (for replies) |
| `post_id` | uuid | Parent post ID |
| `user_id` | uuid | Author user ID |
| `likes_count` | integer | Number of likes |
| `dislikes_count` | integer | Number of dislikes |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Comment Not Found",
    "detail": "The requested comment could not be found."
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
// CommentsService — client.content.comments
list(postUniqueId: string, params?: ListCommentsParams): Promise<PageResult<Comment>>;
get(postUniqueId: string, uniqueId: string): Promise<Comment>;
create(postUniqueId: string, data: CreateCommentRequest): Promise<Comment>;
update(postUniqueId: string, uniqueId: string, data: UpdateCommentRequest): Promise<Comment>;
delete(postUniqueId: string, uniqueId: string): Promise<void>;
reply(postUniqueId: string, parentCommentUniqueId: string, data: Omit<CreateCommentRequest, 'parentId'>): Promise<Comment>;
like(postUniqueId: string, uniqueId: string): Promise<Comment>;
dislike(postUniqueId: string, uniqueId: string): Promise<Comment>;
save(postUniqueId: string, uniqueId: string): Promise<Comment>;
unsave(postUniqueId: string, uniqueId: string): Promise<Comment>;
follow(postUniqueId: string, uniqueId: string): Promise<Comment>;
unfollow(postUniqueId: string, uniqueId: string): Promise<Comment>;
```

### TypeScript Types

```typescript
import type {
  Comment,
  CreateCommentRequest,
  UpdateCommentRequest,
  ListCommentsParams,
} from '@23blocks/block-content';
```

### React Hook

```typescript
import { useContentBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useContentBlock();

  // Example: list comments for a post
  const result = await client.content.comments.list('post-unique-id');
}
```
