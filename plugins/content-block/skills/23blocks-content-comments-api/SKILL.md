---
name: 23blocks-content-comments-api
description: "Content Block comments on posts: create, reply, moderate, like, follow, save. Use for post discussions."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Comments API

Complete API reference for 23blocks comment management with threading and social interactions.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://content.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Comment Not Found","detail":"The requested comment could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-content`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
