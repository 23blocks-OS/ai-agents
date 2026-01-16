---
name: content-series-api
description: Create and manage 23blocks series via REST API. Use when grouping related posts into series, managing post ordering, or handling series social interactions (likes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Series API

Complete API reference for 23blocks series management - grouping related posts (chapters, episodes, multi-part stories) with ordering and social interactions.

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
curl -X GET "$BLOCKS_API_URL/series" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## CRUD Endpoints

### GET /series - List Series

Lists all series with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/series?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `search` | string | No | Search by title |

**Response 200:**
```json
{
  "data": [
    {
      "id": "series-uuid-123",
      "type": "series",
      "attributes": {
        "unique_id": "series-uuid-123",
        "title": "Climate Change Explained",
        "description": "A comprehensive multi-part series",
        "slug": "climate-change-explained",
        "thumbnail_url": "https://example.com/thumb.jpg",
        "image_url": "https://example.com/image.jpg",
        "status": "active",
        "enabled": "true",
        "visibility": "public",
        "completion_status": "ongoing",
        "user_unique_id": "user-uuid",
        "user_name": "John Doe",
        "user_alias": "johndoe",
        "user_avatar_url": "https://example.com/avatar.jpg",
        "posts_count": 5,
        "likes": 42,
        "dislikes": 2,
        "followers": 156,
        "savers": 89,
        "ai_generated": false,
        "moderated": false,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "posts": { "data": [] },
        "tags": { "data": [] },
        "categories": { "data": [] }
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

### POST /series/query - Query Series

Query series with advanced filters.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/series/query" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "search": {
      "visibility": "public",
      "completion_status": "ongoing",
      "search": "climate"
    }
  }'
```

**Query Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `visibility` | string | public, private, unlisted |
| `completion_status` | string | ongoing, completed, hiatus, cancelled |
| `search` | string | Search in title |

**Response 200:** Same format as GET /series

---

### GET /series/:unique_id - Get Series

Retrieves a single series by unique ID or slug.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/series/series-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "series-uuid-123",
    "type": "series",
    "attributes": {
      "unique_id": "series-uuid-123",
      "title": "Climate Change Explained",
      "description": "A comprehensive multi-part series on climate science",
      "slug": "climate-change-explained",
      "thumbnail_url": "https://example.com/thumb.jpg",
      "image_url": "https://example.com/image.jpg",
      "status": "active",
      "enabled": "true",
      "visibility": "public",
      "completion_status": "ongoing",
      "user_unique_id": "user-uuid",
      "user_name": "John Doe",
      "user_alias": "johndoe",
      "user_avatar_url": "https://example.com/avatar.jpg",
      "posts_count": 5,
      "likes": 42,
      "dislikes": 2,
      "followers": 156,
      "savers": 89,
      "payload": {},
      "ai_generated": false,
      "ai_model": null,
      "moderated": false,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "posts": {
        "data": [
          { "id": "post-1", "type": "post" },
          { "id": "post-2", "type": "post" }
        ]
      },
      "tags": { "data": [] },
      "categories": { "data": [] }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Series not found

---

### POST /series - Create Series

Creates a new series.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/series" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "series": {
      "title": "Climate Change Explained",
      "description": "A comprehensive multi-part series on climate science",
      "visibility": "public",
      "completion_status": "ongoing",
      "thumbnail_url": "https://example.com/thumb.jpg",
      "image_url": "https://example.com/image.jpg"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Series title |
| `description` | text | No | Series description |
| `visibility` | enum | No | public, private, unlisted (default: public) |
| `completion_status` | enum | No | ongoing, completed, hiatus, cancelled (default: ongoing) |
| `thumbnail_url` | string | No | Thumbnail image URL |
| `image_url` | string | No | Full image URL |
| `payload` | json | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "new-series-uuid",
    "type": "series",
    "attributes": {
      "unique_id": "new-series-uuid",
      "title": "Climate Change Explained",
      "slug": "climate-change-explained-1705123456-ab12cd34",
      "status": "active",
      "visibility": "public",
      "completion_status": "ongoing",
      "user_unique_id": "current-user-uuid",
      "user_name": "John Doe",
      "posts_count": 0,
      "likes": 0,
      "followers": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `401 Unauthorized` - Missing or invalid credentials
- `422 Unprocessable Entity` - Validation errors

---

### PUT /series/:unique_id - Update Series

Updates an existing series. Only the series owner or admin can update.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/series/series-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "series": {
      "title": "Updated Series Title",
      "description": "Updated description",
      "completion_status": "completed"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "series-uuid-123",
    "type": "series",
    "attributes": {
      "unique_id": "series-uuid-123",
      "title": "Updated Series Title",
      "description": "Updated description",
      "completion_status": "completed",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Not the owner or admin
- `404 Not Found` - Series not found

---

### DELETE /series/:unique_id - Delete Series

Soft-deletes a series. Only the series owner or admin can delete.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/series/series-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Not the owner or admin
- `404 Not Found` - Series not found

---

## Social Actions

### PUT /series/:unique_id/like - Toggle Like

Toggles like on a series. If already liked, removes the like.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/series/series-uuid-123/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /series/:unique_id/dislike - Toggle Dislike

Toggles dislike on a series. If already disliked, removes the dislike.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/series/series-uuid-123/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /series/:unique_id/follow - Follow Series

Follows a series to receive updates.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/series/series-uuid-123/follow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### DELETE /series/:unique_id/unfollow - Unfollow Series

Unfollows a series.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/series/series-uuid-123/unfollow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### PUT /series/:unique_id/save - Save Series

Saves/bookmarks a series.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/series/series-uuid-123/save" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

### DELETE /series/:unique_id/unsave - Unsave Series

Removes series from saved list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/series/series-uuid-123/unsave" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Post Management

### GET /series/:unique_id/posts - List Series Posts

Lists all posts in a series, ordered by sequence.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/series/series-uuid-123/posts?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "post-uuid-1",
      "type": "post",
      "attributes": {
        "unique_id": "post-uuid-1",
        "title": "Chapter 1: Introduction",
        "series_sequence": 1,
        "series_unique_id": "series-uuid-123"
      }
    },
    {
      "id": "post-uuid-2",
      "type": "post",
      "attributes": {
        "unique_id": "post-uuid-2",
        "title": "Chapter 2: The Science",
        "series_sequence": 2,
        "series_unique_id": "series-uuid-123"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 2
  }
}
```

---

### POST /series/:unique_id/posts/:post_id - Add Post to Series

Adds a post to the series. Only the series owner or admin can add posts.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/series/series-uuid-123/posts/post-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"sequence": 3}'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sequence` | integer | No | Position in series (auto-assigns next if not provided) |

**Response 200:**
```json
{
  "data": {
    "id": "post-uuid-456",
    "type": "post",
    "attributes": {
      "unique_id": "post-uuid-456",
      "series_id": 123,
      "series_unique_id": "series-uuid-123",
      "series_sequence": 3
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Not the series owner or admin
- `404 Not Found` - Series or post not found

---

### DELETE /series/:unique_id/posts/:post_id - Remove Post from Series

Removes a post from the series. Only the series owner or admin can remove posts.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/series/series-uuid-123/posts/post-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Not the series owner or admin
- `404 Not Found` - Series or post not found

---

### PUT /series/:unique_id/reorder - Reorder Posts

Reorders posts within the series. Only the series owner or admin can reorder.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/series/series-uuid-123/reorder" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "posts": [
      {"post_unique_id": "post-uuid-3", "sequence": 1},
      {"post_unique_id": "post-uuid-1", "sequence": 2},
      {"post_unique_id": "post-uuid-2", "sequence": 3}
    ]
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `posts` | array | Yes | Array of post reorder objects |
| `posts[].post_unique_id` | string | Yes | Post unique ID |
| `posts[].sequence` | integer | Yes | New sequence number |

**Response 200:**
```json
{
  "message": "Posts reordered successfully"
}
```

**Errors:**
- `403 Forbidden` - Not the series owner or admin

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
| `user_alias` | string | Creator's username |
| `user_avatar_url` | string | Creator's avatar URL |
| `posts_count` | integer | Number of posts in series |
| `likes` | integer | Like count |
| `dislikes` | integer | Dislike count |
| `followers` | integer | Follower count |
| `savers` | integer | Save count |
| `payload` | jsonb | Custom metadata |
| `ai_generated` | boolean | AI generation flag |
| `ai_model` | string | AI model used if generated |
| `moderated` | boolean | Moderation flag |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Visibility Options
| Value | Description |
|-------|-------------|
| `public` | Visible to everyone |
| `private` | Only visible to owner |
| `unlisted` | Not listed but accessible via direct link |

### Completion Status Options
| Value | Description |
|-------|-------------|
| `ongoing` | Series is actively being updated |
| `completed` | Series is finished |
| `hiatus` | Series is on temporary break |
| `cancelled` | Series has been cancelled |

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

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `403` | Forbidden - Not the owner or admin |
| `404` | Not Found - Resource not found |
| `422` | Unprocessable Entity - Validation errors |
