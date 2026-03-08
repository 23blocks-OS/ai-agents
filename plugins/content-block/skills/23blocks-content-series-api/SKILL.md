---
name: 23blocks-content-series-api
description: Create and manage 23blocks series via REST API. Use when grouping related posts into series, managing post ordering, or handling series social interactions (likes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Series API

Complete API reference for 23blocks series management - grouping related posts (chapters, episodes, multi-part stories) with ordering and social interactions.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Content API base URL | `https://content.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/series" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/series` | List all series |
| POST | `/series/query` | Query series with filters |
| GET | `/series/:unique_id` | Get a single series |
| POST | `/series` | Create a series |
| PUT | `/series/:unique_id` | Update a series |
| DELETE | `/series/:unique_id` | Delete a series |
| PUT | `/series/:unique_id/like` | Toggle like |
| PUT | `/series/:unique_id/dislike` | Toggle dislike |
| PUT | `/series/:unique_id/follow` | Follow a series |
| DELETE | `/series/:unique_id/unfollow` | Unfollow a series |
| PUT | `/series/:unique_id/save` | Save a series |
| DELETE | `/series/:unique_id/unsave` | Unsave a series |
| GET | `/series/:unique_id/posts` | List series posts |
| POST | `/series/:unique_id/posts/:post_id` | Add post to series |
| DELETE | `/series/:unique_id/posts/:post_id` | Remove post from series |
| PUT | `/series/:unique_id/reorder` | Reorder posts |

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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Series Not Found",
    "detail": "The requested series could not be found."
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
