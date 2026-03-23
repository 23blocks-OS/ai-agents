---
name: 23blocks-auth-user-profiles-api
description: Manage 23blocks user profiles via REST API. Use when creating, reading, or updating user profile data including personal information, social links, and preferences.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# User Profiles API

Complete API reference for 23blocks dedicated user profile CRUD operations including personal information, social links, and preferences.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Auth API base URL | `https://auth.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://auth.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://auth.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### GET /users/:user_unique_id/profile - Show Profile

Retrieves the profile for a specific user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/{user_unique_id}/profile" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "profile-uuid-123",
    "type": "user_profile",
    "attributes": {
      "unique_id": "profile-uuid-123",
      "user_unique_id": "user-uuid-123",
      "first_name": "John",
      "middle_name": "M",
      "last_name": "Doe",
      "email": "john@example.com",
      "phone_number": "+1234567890",
      "gender": "male",
      "birthdate": "1990-01-15",
      "zipcode": "10001",
      "preferred_language": "en",
      "time_zone": "America/New_York",
      "web_site": "https://johndoe.com",
      "twitter": "@johndoe",
      "linkedin": "johndoe",
      "status": "updated",
      "created_at": "2026-01-01T00:00:00Z",
      "updated_at": "2026-03-13T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` (PROFILE_NOT_FOUND) - Profile does not exist for this user
- `404 Not Found` - User not found

---

### POST /users/profile - Create Profile

Creates a profile for the currently authenticated user (from token). Uses upsert pattern: creates if missing, updates if exists.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/profile" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profile": {
      "first_name": "John",
      "last_name": "Doe",
      "phone_number": "+1234567890",
      "gender": "male",
      "birthdate": "1990-01-15",
      "zipcode": "10001",
      "preferred_language": "en",
      "time_zone": "America/New_York"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `profile.first_name` | string | No | First name |
| `profile.middle_name` | string | No | Middle name |
| `profile.last_name` | string | No | Last name |
| `profile.gender` | string | No | Gender |
| `profile.ethnicity` | string | No | Ethnicity |
| `profile.zipcode` | string | No | Zip code |
| `profile.marital_status` | string | No | Marital status |
| `profile.birthdate` | date | No | Date of birth |
| `profile.hhi` | string | No | Household income |
| `profile.children` | string | No | Number of children |
| `profile.source` | string | No | Registration source |
| `profile.preferred_language` | string | No | Preferred language |
| `profile.phone_number` | string | No | Phone number |
| `profile.email` | string | No | Contact email |
| `profile.preferred_device` | string | No | Preferred device |
| `profile.web_site` | string | No | Personal website |
| `profile.twitter` | string | No | Twitter handle |
| `profile.fb` | string | No | Facebook profile |
| `profile.instagram` | string | No | Instagram handle |
| `profile.linkedin` | string | No | LinkedIn profile |
| `profile.youtube` | string | No | YouTube channel |
| `profile.blog` | string | No | Blog URL |
| `profile.network_a` | string | No | Custom social network A |
| `profile.network_b` | string | No | Custom social network B |
| `profile.time_zone` | string | No | Time zone |

**Response 201 (created) / 200 (updated):**
```json
{
  "data": {
    "id": "profile-uuid-123",
    "type": "user_profile",
    "attributes": {
      "unique_id": "profile-uuid-123",
      "first_name": "John",
      "last_name": "Doe",
      "status": "created",
      "created_at": "2026-03-13T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` (USER_NOT_FOUND) - Authenticated user not found

---

### PUT /users/:user_unique_id/profile - Update Profile

Updates the profile for a specific user. Uses upsert pattern: creates profile if it doesn't exist, updates if it does.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/users/{user_unique_id}/profile" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profile": {
      "first_name": "John",
      "last_name": "Doe",
      "phone_number": "+1234567890",
      "preferred_language": "es",
      "time_zone": "America/Los_Angeles"
    }
  }'
```

**Response 200 (updated) / 201 (created):**
```json
{
  "data": {
    "id": "profile-uuid-123",
    "type": "user_profile",
    "attributes": {
      "unique_id": "profile-uuid-123",
      "first_name": "John",
      "last_name": "Doe",
      "phone_number": "+1234567890",
      "preferred_language": "es",
      "time_zone": "America/Los_Angeles",
      "status": "updated"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "source": "UserProfiles",
    "code": "PROFILE_NOT_FOUND",
    "title": "Profile Not Found",
    "detail": "Profile not found for this user."
  }]
}
```

---

## Important Notes

- **Upsert pattern**: Both `POST /users/profile` and `PUT /users/:user_unique_id/profile` use upsert logic. If no profile exists, one is created; if it exists, it is updated.
- **User name sync**: When both `first_name` and `last_name` are provided, the parent user's `name` field is automatically updated to `"first_name last_name"`.
- **Payload support**: Custom JSON payloads can be attached via the `payload` field for extensible profile data.
- **Event publishing**: Profile create and update operations publish events for downstream services.
- **Status field**: Profile status is set to `created` on first creation and `updated` on subsequent updates.
