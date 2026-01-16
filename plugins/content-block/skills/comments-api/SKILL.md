---
name: content-comments-api
description: Create and manage 23blocks comments via REST API. Use when adding comments to posts, creating replies, or managing comment social interactions (likes, dislikes, follows, saves).
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Comments API

Complete API reference for 23blocks comment management with threading and social interactions.

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
curl -X GET "$BLOCKS_API_URL/posts/{post_id}/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /posts/:post_id/comments - List Comments

Lists all comments for a post.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/posts/post-uuid-123/comments?page=1&records=20" \
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
      "id": "comment-uuid-123",
      "type": "comment",
      "attributes": {
        "unique_id": "comment-uuid-123",
        "body": "Great post!",
        "parent_id": null,
        "likes_count": 5,
        "dislikes_count": 0,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "user": {
          "data": { "id": "user-uuid", "type": "user_identity" }
        },
        "replies": {
          "data": []
        }
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 25
  }
}
```

---

### GET /posts/:post_id/comments/:unique_id - Get Comment

Retrieves a single comment by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Great post!",
      "parent_id": null,
      "likes_count": 5,
      "dislikes_count": 0,
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Comment not found

---

### POST /posts/:post_id/comments - Create Comment

Creates a new comment on a post.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/posts/post-uuid-123/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "This is a great article!"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `body` | string | Yes | Comment content |

**Response 201:**
```json
{
  "data": {
    "id": "new-comment-uuid",
    "type": "comment",
    "attributes": {
      "unique_id": "new-comment-uuid",
      "body": "This is a great article!",
      "parent_id": null,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors
- `404 Not Found` - Post not found

---

### PUT /posts/:post_id/comments/:unique_id - Update Comment

Updates an existing comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Updated comment text"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "comment-uuid-456",
    "type": "comment",
    "attributes": {
      "unique_id": "comment-uuid-456",
      "body": "Updated comment text",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Comment not found

---

### DELETE /posts/:post_id/comments/:unique_id - Delete Comment

Deletes a comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Comment not found

---

### POST /posts/:post_id/comments/:unique_id/reply - Reply to Comment

Creates a reply to an existing comment.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/reply" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "comment": {
      "body": "Thanks for your feedback!"
    }
  }'
```

**Response 201:**
```json
{
  "data": {
    "id": "reply-comment-uuid",
    "type": "comment",
    "attributes": {
      "unique_id": "reply-comment-uuid",
      "body": "Thanks for your feedback!",
      "parent_id": "comment-uuid-456",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Social Actions

### PUT /posts/:post_id/comments/:unique_id/like - Like Comment

Adds a like to the comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/like" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment liked successfully"
}
```

---

### PUT /posts/:post_id/comments/:unique_id/dislike - Dislike Comment

Adds a dislike to the comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/dislike" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment disliked successfully"
}
```

---

### PUT /posts/:post_id/comments/:unique_id/follow - Follow Comment

Follows a comment to receive updates on replies.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/follow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment followed successfully"
}
```

---

### DELETE /posts/:post_id/comments/:unique_id/unfollow - Unfollow Comment

Unfollows a comment.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/unfollow" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment unfollowed successfully"
}
```

---

### PUT /posts/:post_id/comments/:unique_id/save - Save Comment

Saves/bookmarks a comment.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/save" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment saved successfully"
}
```

---

### DELETE /posts/:post_id/comments/:unique_id/unsave - Unsave Comment

Removes comment from saved list.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/unsave" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Comment unsaved successfully"
}
```

---

## Moderation

### DELETE /posts/:post_id/comments/:unique_id/moderate - Moderate Comment

Removes a comment as a moderator.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/posts/post-uuid-123/comments/comment-uuid-456/moderate" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Comment not found

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
