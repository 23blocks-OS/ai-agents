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
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/posts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/posts` | List all posts |
| POST | `/posts/query` | Query posts with filters |
| GET | `/posts/:unique_id` | Get a single post |
| POST | `/posts` | Create a post |
| PUT | `/posts/:unique_id` | Update a post |
| PUT | `/posts/:unique_id/replace` | Replace post content entirely |
| DELETE | `/posts/:unique_id` | Delete a post |
| PUT | `/posts/:unique_id/like` | Like a post |
| DELETE | `/posts/:unique_id/dislike` | Remove like from post |
| PUT | `/posts/:unique_id/follow` | Follow a post |
| DELETE | `/posts/:unique_id/unfollow` | Unfollow a post |
| PUT | `/posts/:unique_id/save` | Save a post |
| DELETE | `/posts/:unique_id/unsave` | Unsave a post |
| PUT | `/posts/:unique_id/own` | Transfer post ownership |
| POST | `/posts/:unique_id/versions/:version_id/publish` | Publish a version |

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
