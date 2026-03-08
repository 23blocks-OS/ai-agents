# Identities API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### GET /identities - List Identities

Lists all user identities.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "user-uuid-123",
      "type": "user_identity",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "username": "johndoe",
        "display_name": "John Doe",
        "avatar_url": "https://example.com/avatar.jpg",
        "bio": "Software developer",
        "posts_count": 15,
        "comments_count": 42,
        "followers_count": 120,
        "following_count": 85,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /identities/:unique_id - Get Identity

Retrieves a single user identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "username": "johndoe",
      "display_name": "John Doe",
      "avatar_url": "https://example.com/avatar.jpg",
      "bio": "Software developer",
      "posts_count": 15,
      "comments_count": 42,
      "followers_count": 120,
      "following_count": 85,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "tags": {
        "data": [
          { "id": "tag-uuid", "type": "tag" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /identities/:unique_id/register - Register User

Registers a new user in the content system.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/register" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "email": "newuser@example.com",
      "username": "newuser",
      "display_name": "New User"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `email` | string | Yes | User email |
| `username` | string | No | Username |
| `display_name` | string | No | Display name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "newuser@example.com",
      "username": "newuser",
      "display_name": "New User",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /identities/:unique_id - Update User

Updates an existing user profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/identities/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
      "bio": "Senior software developer",
      "avatar_url": "https://example.com/new-avatar.jpg"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "user_identity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "display_name": "John D.",
      "bio": "Senior software developer",
      "avatar_url": "https://example.com/new-avatar.jpg",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

## User Tags

### POST /identities/:unique_id/tags - Add Tag to User

Adds a tag to a user's profile.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag_unique_id": "tag-uuid-456"
  }'
```

**Response 200:**
```json
{
  "message": "Tag added successfully"
}
```

---

### DELETE /identities/:unique_id/tags/:tag_id - Remove Tag from User

Removes a tag from a user's profile.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/identities/user-uuid-123/tags/tag-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Tag removed successfully"
}
```

---

## User Contributions

### GET /identities/:unique_id/drafts - Get User Drafts

Retrieves all draft posts by the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/drafts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "post-uuid",
      "type": "post",
      "attributes": {
        "unique_id": "post-uuid",
        "title": "Draft Post",
        "status": "draft",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /identities/:unique_id/posts - Get User Posts

Retrieves all published posts by the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/posts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "post-uuid",
      "type": "post",
      "attributes": {
        "unique_id": "post-uuid",
        "title": "Published Post",
        "status": "published",
        "likes_count": 25,
        "comments_count": 8,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /identities/:unique_id/comments - Get User Comments

Retrieves all comments by the user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/comments" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "comment-uuid",
      "type": "comment",
      "attributes": {
        "unique_id": "comment-uuid",
        "body": "Great article!",
        "likes_count": 5,
        "created_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "post": {
          "data": { "id": "post-uuid", "type": "post" }
        }
      }
    }
  ]
}
```

---

## Social Following

### GET /identities/:unique_id/followers - Get User Followers

Retrieves all users following this user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/followers" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "follower-uuid",
      "type": "user_identity",
      "attributes": {
        "unique_id": "follower-uuid",
        "username": "follower1",
        "display_name": "Follower One",
        "avatar_url": "https://example.com/follower.jpg"
      }
    }
  ],
  "meta": {
    "total_count": 120
  }
}
```

---

### GET /identities/:unique_id/following - Get Following

Retrieves all users this user is following.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/following" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "following-uuid",
      "type": "user_identity",
      "attributes": {
        "unique_id": "following-uuid",
        "username": "followed_user",
        "display_name": "Followed User",
        "avatar_url": "https://example.com/followed.jpg"
      }
    }
  ],
  "meta": {
    "total_count": 85
  }
}
```

---

### POST /identities/:unique_id/follows/:user_id - Follow User

Follows another user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/follows/target-user-uuid" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "User followed successfully"
}
```

---

### DELETE /identities/:unique_id/unfollows/:user_id - Unfollow User

Unfollows a user.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/identities/user-uuid-123/unfollows/target-user-uuid" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "User unfollowed successfully"
}
```

---

## Activity

### GET /identities/:unique_id/activities - Get User Activities

Retrieves the user's activity feed.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/activities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "activity-uuid",
      "type": "activity",
      "attributes": {
        "unique_id": "activity-uuid",
        "activity_type": "post_created",
        "description": "Created a new post",
        "created_at": "2025-01-10T10:30:00Z"
      },
      "relationships": {
        "target": {
          "data": { "id": "post-uuid", "type": "post" }
        }
      }
    }
  ]
}
```
