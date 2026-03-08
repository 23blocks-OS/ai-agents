# Posts API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

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
