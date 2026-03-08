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
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/posts/{post_id}/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/posts/:post_id/comments` | List comments for a post |
| GET | `/posts/:post_id/comments/:unique_id` | Get a single comment |
| POST | `/posts/:post_id/comments` | Create a comment |
| PUT | `/posts/:post_id/comments/:unique_id` | Update a comment |
| DELETE | `/posts/:post_id/comments/:unique_id` | Delete a comment |
| POST | `/posts/:post_id/comments/:unique_id/reply` | Reply to a comment |
| PUT | `/posts/:post_id/comments/:unique_id/like` | Like a comment |
| PUT | `/posts/:post_id/comments/:unique_id/dislike` | Dislike a comment |
| PUT | `/posts/:post_id/comments/:unique_id/follow` | Follow a comment |
| DELETE | `/posts/:post_id/comments/:unique_id/unfollow` | Unfollow a comment |
| PUT | `/posts/:post_id/comments/:unique_id/save` | Save a comment |
| DELETE | `/posts/:post_id/comments/:unique_id/unsave` | Unsave a comment |
| DELETE | `/posts/:post_id/comments/:unique_id/moderate` | Moderate (remove) a comment |

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
