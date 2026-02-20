---
name: content-posts-api
description: Create and manage 23blocks posts via REST API. Use when creating, updating, publishing posts, or managing post social interactions (likes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Posts API

Complete API reference for 23blocks post management with versioning and social interactions.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://content.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

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

---

## Endpoints

### GET /posts - List Posts

Lists all posts with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/posts?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "post-uuid-123",
      "type": "post",
      "attributes": {
        "unique_id": "post-uuid-123",
        "title": "My First Post",
        "content": "Post content here",
        "status": "published",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### POST /posts/query - Query Posts

Query posts with advanced filters.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/posts/query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "status": "published",
      "tag_id": "tag-uuid"
    }
  }'
```

**Response 200:** Same format as GET /posts

---

### GET /posts/:unique_id - Get Post

Retrieves a single post by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/posts/post-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "post-uuid-123",
    "type": "post",
    "attributes": {
      "unique_id": "post-uuid-123",
      "title": "My First Post",
      "content": "Full post content",
      "status": "published",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "versions": {
        "data": [
          { "id": "version-1", "type": "post_version" }
        ]
      },
      "tags": {
        "data": []
      },
      "categories": {
        "data": []
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Post not found

---

### POST /posts - Create Post

Creates a new post.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/posts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "title": "New Post Title",
      "content": "Post content goes here",
      "status": "draft"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Post title |
| `content` | string | Yes | Post content |
| `status` | enum | No | draft, published (default: draft) |

**Response 201:**
```json
{
  "data": {
    "id": "new-post-uuid",
    "type": "post",
    "attributes": {
      "unique_id": "new-post-uuid",
      "title": "New Post Title",
      "content": "Post content goes here",
      "status": "draft",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /posts/:unique_id - Update Post

Updates an existing post.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "title": "Updated Title",
      "content": "Updated content"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "post-uuid-123",
    "type": "post",
    "attributes": {
      "unique_id": "post-uuid-123",
      "title": "Updated Title",
      "content": "Updated content",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Post not found

---

### PUT /posts/:unique_id/replace - Replace Post

Replaces post content entirely.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/replace" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "post": {
      "title": "Completely New Title",
      "content": "Completely new content"
    }
  }'
```

**Response 200:** Same as PUT /posts/:unique_id

---

### DELETE /posts/:unique_id - Delete Post

Soft-deletes a post.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Post not found

---

## Social Actions

### PUT /posts/:unique_id/like - Like Post

Adds a like to the post.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Post liked successfully"
}
```

---

### DELETE /posts/:unique_id/dislike - Remove Like

Removes like from the post.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Like removed successfully"
}
```

---

### PUT /posts/:unique_id/follow - Follow Post

Follows a post to receive updates.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/follow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Post followed successfully"
}
```

---

### DELETE /posts/:unique_id/unfollow - Unfollow Post

Unfollows a post.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123/unfollow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Post unfollowed successfully"
}
```

---

### PUT /posts/:unique_id/save - Save Post

Saves/bookmarks a post.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/save" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Post saved successfully"
}
```

---

### DELETE /posts/:unique_id/unsave - Unsave Post

Removes post from saved list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123/unsave" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Post unsaved successfully"
}
```

---

## Versioning & Ownership

### PUT /posts/:unique_id/own - Change Owner

Transfers post ownership to another user.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/own" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "owner_id": "new-owner-uuid"
  }'
```

**Response 200:**
```json
{
  "message": "Ownership transferred successfully"
}
```

---

### POST /posts/:unique_id/versions/:version_id/publish - Publish Version

Publishes a specific version of the post.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/posts/post-uuid-123/versions/version-uuid/publish" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "post-uuid-123",
    "type": "post",
    "attributes": {
      "status": "published",
      "published_version_id": "version-uuid"
    }
  }
}
```

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
