---
name: 23blocks-geolocation-identities-api
description: "Geolocation Block user identities: register, profiles, current location. Use before a user's first Geolocation call."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Identities API

Complete API reference for 23blocks geolocation user identity management and user location tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://geolocation.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /identities/ - List Identities

Lists all user identities registered in the geolocation system.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "user-uuid-123",
      "type": "UserIdentity",
      "attributes": {
        "unique_id": "user-uuid-123",
        "email": "user@example.com",
        "username": "johndoe",
        "display_name": "John Doe",
        "avatar_url": "https://example.com/avatar.jpg",
        "latitude": 40.7128,
        "longitude": -74.0060,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /identities/:unique_id/ - Get Identity

Retrieves a single user identity by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/identities/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "UserIdentity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "email": "user@example.com",
      "username": "johndoe",
      "display_name": "John Doe",
      "avatar_url": "https://example.com/avatar.jpg",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### POST /identities/:unique_id/register/ - Register Identity

Registers a new user in the geolocation system. Each block is autonomous; users must register their identity before using private endpoints.

> **Note:** Block identity records are notification routing caches, not identity models. The canonical user record lives in the Auth (Gateway) block. `email`/`phone` here are optional denormalized routing fields; duplicates across users are allowed.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/identities/user-uuid-123/register/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
| `user_unique_id` | uuid | Yes | User unique ID (from the URL path `:unique_id`) — the only required value |
| `email` | string | No | Optional denormalized routing field for notifications; if blank, the block skips the email channel. Duplicates across users are allowed |
| `username` | string | No | Username |
| `display_name` | string | No | Display name |

**Response 201:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "UserIdentity",
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

**Errors:**
- `409 Conflict` - This `user_unique_id` is already registered
- `422 Unprocessable Entity` - Missing `user_unique_id`

---

### PUT /identities/:unique_id/ - Update Identity

Updates an existing user profile.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/identities/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user": {
      "display_name": "John D.",
      "avatar_url": "https://example.com/new-avatar.jpg"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "UserIdentity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "display_name": "John D.",
      "avatar_url": "https://example.com/new-avatar.jpg",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found

---

### DELETE /identities/:unique_id/ - Delete Identity

Deletes a user identity from the geolocation system.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/identities/user-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - User not found

---

## User Location

### POST /users/:unique_id/location - Set User Location

Sets or updates the current geographic location for a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-123/location" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location": {
      "latitude": 40.7128,
      "longitude": -74.0060
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `latitude` | float | Yes | Latitude coordinate |
| `longitude` | float | Yes | Longitude coordinate |

**Response 200:**
```json
{
  "data": {
    "id": "user-uuid-123",
    "type": "UserIdentity",
    "attributes": {
      "unique_id": "user-uuid-123",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "location_updated_at": "2025-01-12T14:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User not found
- `422 Unprocessable Entity` - Invalid coordinates

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
| `latitude` | float | Current latitude |
| `longitude` | float | Current longitude |
| `location_updated_at` | timestamp | Last location update time |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"User Not Found","detail":"The requested user could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-geolocation`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useGeolocationBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// GeoIdentitiesService — client.geolocation.geoIdentities
client.geolocation.geoIdentities.list(params?: ListGeoIdentitiesParams): Promise<PageResult<GeoIdentity>>;
client.geolocation.geoIdentities.get(uniqueId: string): Promise<GeoIdentity>;
client.geolocation.geoIdentities.register(uniqueId: string, data: RegisterGeoIdentityRequest): Promise<GeoIdentity>;
client.geolocation.geoIdentities.update(uniqueId: string, data: UpdateGeoIdentityRequest): Promise<GeoIdentity>;
client.geolocation.geoIdentities.delete(uniqueId: string): Promise<void>;
client.geolocation.geoIdentities.addToLocation(locationUniqueId: string, data: LocationIdentityRequest): Promise<void>;
client.geolocation.geoIdentities.removeFromLocation(locationUniqueId: string, userUniqueId: string): Promise<void>;
client.geolocation.geoIdentities.updateLocation(userUniqueId: string, data: UserLocationRequest): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  GeoIdentity,
  RegisterGeoIdentityRequest,
  UpdateGeoIdentityRequest,
  ListGeoIdentitiesParams,
  LocationIdentityRequest,
  UserLocationRequest,
} from '@23blocks/block-geolocation';
```
