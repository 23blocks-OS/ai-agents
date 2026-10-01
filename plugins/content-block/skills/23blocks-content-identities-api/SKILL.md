---
name: 23blocks-content-identities-api
description: "Content Block user identities: register, profiles, follows, a user's posts, drafts, comments, activity. Use for author and reader profiles."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks user identity management with social following and activity tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://content.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/identities` | List all identities |
| GET | `/identities/:unique_id` | Get a single identity |
| POST | `/identities/:unique_id/register` | Register a user |
| PUT | `/identities/:unique_id` | Update user profile |
| POST | `/identities/:unique_id/tags` | Add tag to user |
| DELETE | `/identities/:unique_id/tags/:tag_id` | Remove tag from user |
| GET | `/identities/:unique_id/drafts` | Get user drafts |
| GET | `/identities/:unique_id/posts` | Get user posts |
| GET | `/identities/:unique_id/comments` | Get user comments |
| GET | `/identities/:unique_id/followers` | Get user followers |
| GET | `/identities/:unique_id/following` | Get users being followed |
| POST | `/identities/:unique_id/follows/:user_id` | Follow a user |
| DELETE | `/identities/:unique_id/unfollows/:user_id` | Unfollow a user |
| GET | `/identities/:unique_id/activities` | Get user activities |

---

## Data Models

### UserIdentity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `email` | string | User email |
| `username` | string | Username |
| `display_name` | string | Display name |
| `avatar_url` | string | Avatar image URL |
| `bio` | string | User biography |
| `posts_count` | integer | Number of posts |
| `comments_count` | integer | Number of comments |
| `followers_count` | integer | Number of followers |
| `following_count` | integer | Number of users following |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Activity
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `activity_type` | string | Type of activity |
| `description` | string | Activity description |
| `target_id` | uuid | Related entity ID |
| `target_type` | string | Related entity type |
| `created_at` | timestamp | Activity time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"User Not Found","detail":"The requested user could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-content`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// ContentUsersService — client.content.users
list(params?: ListContentUsersParams): Promise<PageResult<ContentUser>>;
get(uniqueId: string): Promise<ContentUser>;
register(uniqueId: string, data: RegisterContentUserRequest): Promise<ContentUser>;
update(uniqueId: string, data: UpdateContentUserRequest): Promise<ContentUser>;
getDrafts(uniqueId: string): Promise<Post[]>;
getPosts(uniqueId: string): Promise<Post[]>;
getComments(uniqueId: string): Promise<Comment[]>;
getActivities(uniqueId: string): Promise<UserActivity[]>;
addTag(uniqueId: string, tagUniqueId: string): Promise<ContentUser>;
removeTag(uniqueId: string, tagUniqueId: string): Promise<void>;
getFollowers(uniqueId: string): Promise<Following[]>;
getFollowing(uniqueId: string): Promise<Following[]>;
followUser(uniqueId: string, targetUserUniqueId: string): Promise<void>;
unfollowUser(uniqueId: string, targetUserUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  ContentUser,
  Following,
  RegisterContentUserRequest,
  UpdateContentUserRequest,
  ListContentUsersParams,
  UserActivity,
} from '@23blocks/block-content';
```

### React Hook

```typescript
import { useContentBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useContentBlock();

  // Example: get a user's profile
  const result = await client.content.users.get('user-unique-id');
}
```
