---
name: 23blocks-content-series-api
description: "Content Block series: ordered groups of posts (chapters, episodes), reorder, social actions. Use for multi-part content."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Series API

Complete API reference for 23blocks series management - grouping related posts (chapters, episodes, multi-part stories) with ordering and social interactions.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://content.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/series` | Public | List all series |
| POST | `/series/query` | Public | Query series with filters |
| GET | `/series/:unique_id` | Public | Get a single series |
| POST | `/series` | Bearer | Create a series |
| PUT | `/series/:unique_id` | Bearer | Update a series |
| DELETE | `/series/:unique_id` | Bearer | Delete a series |
| PUT | `/series/:unique_id/like` | Bearer | Toggle like |
| PUT | `/series/:unique_id/dislike` | Bearer | Toggle dislike |
| PUT | `/series/:unique_id/follow` | Bearer | Follow a series |
| DELETE | `/series/:unique_id/unfollow` | Bearer | Unfollow a series |
| PUT | `/series/:unique_id/save` | Bearer | Save a series |
| DELETE | `/series/:unique_id/unsave` | Bearer | Unsave a series |
| GET | `/series/:unique_id/posts` | Bearer | List series posts |
| POST | `/series/:unique_id/posts/:post_id` | Bearer | Add post to series |
| DELETE | `/series/:unique_id/posts/:post_id` | Bearer | Remove post from series |
| PUT | `/series/:unique_id/reorder` | Bearer | Reorder posts |

---

## Data Models

### Series
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Series title |
| `description` | text | Series description |
| `slug` | string | SEO-friendly URL slug |
| `thumbnail_url` | string | Thumbnail image URL |
| `image_url` | string | Full image URL |
| `status` | enum | draft, active, archived, deleted |
| `enabled` | string | 'true' or 'false' (soft delete) |
| `visibility` | enum | public, private, unlisted |
| `completion_status` | enum | ongoing, completed, hiatus, cancelled |
| `user_unique_id` | uuid | Creator's user ID |
| `user_name` | string | Creator's display name |
| `posts_count` | integer | Number of posts in series |
| `likes` | integer | Like count |
| `dislikes` | integer | Dislike count |
| `followers` | integer | Follower count |
| `savers` | integer | Save count |
| `payload` | jsonb | Custom metadata |
| `ai_generated` | boolean | AI generation flag |
| `moderated` | boolean | Moderation flag |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Series Not Found","detail":"The requested series could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-content`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// SeriesService — client.content.series
list(params?: ListSeriesParams): Promise<PageResult<Series>>;
query(params: QuerySeriesParams): Promise<PageResult<Series>>;
get(uniqueId: string): Promise<Series>;
create(data: CreateSeriesRequest): Promise<Series>;
update(uniqueId: string, data: UpdateSeriesRequest): Promise<Series>;
delete(uniqueId: string): Promise<void>;
like(uniqueId: string): Promise<Series>;
dislike(uniqueId: string): Promise<Series>;
follow(uniqueId: string): Promise<Series>;
unfollow(uniqueId: string): Promise<void>;
save(uniqueId: string): Promise<Series>;
unsave(uniqueId: string): Promise<void>;
getPosts(uniqueId: string): Promise<Post[]>;
addPost(seriesUniqueId: string, postUniqueId: string, sequence?: number): Promise<void>;
removePost(seriesUniqueId: string, postUniqueId: string): Promise<void>;
reorderPosts(uniqueId: string, data: ReorderPostsRequest): Promise<Series>;
```

### TypeScript Types

```typescript
import type {
  Series,
  CreateSeriesRequest,
  UpdateSeriesRequest,
  ListSeriesParams,
  QuerySeriesParams,
  ReorderPostsRequest,
  SeriesVisibility,
  SeriesCompletionStatus,
} from '@23blocks/block-content';
```

### React Hook

```typescript
import { useContentBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useContentBlock();

  // Example: list all series with pagination
  const result = await client.content.series.list({ page: 1, perPage: 20 });
}
```
