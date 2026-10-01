---
name: 23blocks-content-posts-api
description: "Content Block posts: CRUD, query, versions and publish, attachments, likes, follows, saves. Use when writing or publishing content."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Posts API

Complete API reference for 23blocks post management with versioning and social interactions.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://content.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Post Not Found","detail":"The requested post could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-content`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

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
