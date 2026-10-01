---
name: 23blocks-geolocation-regions-api
description: "Geolocation Block regions: CRUD, their locations, tags. Use for grouping locations geographically."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Regions API

Complete API reference for 23blocks geographic region management with location associations and tags.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://geolocation.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### GET /regions/:unique_id/locations - Region Locations

Lists all locations within a specific region.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/regions/region-uuid-123/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
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
      "id": "loc-uuid-123",
      "type": "Location",
      "attributes": {
        "unique_id": "loc-uuid-123",
        "name": "Downtown Office",
        "code": "DT-001",
        "address": "123 Main St, New York, NY 10001",
        "latitude": 40.7128,
        "longitude": -74.0060,
        "location_type": "office",
        "status": "active"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 35
  }
}
```

---

### GET /regions/:unique_id - Get Region

Retrieves a single region by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/regions/region-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "region-uuid-123",
    "type": "Region",
    "attributes": {
      "unique_id": "region-uuid-123",
      "name": "Northeast Region",
      "description": "Covers all northeast US locations",
      "boundary": {
        "type": "Polygon",
        "coordinates": [
          [
            [-74.2591, 40.4774],
            [-73.7004, 40.4774],
            [-73.7004, 40.9176],
            [-74.2591, 40.9176],
            [-74.2591, 40.4774]
          ]
        ]
      },
      "location_count": 35,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "tags": {
        "data": [
          { "id": "tag-uuid", "type": "Tag" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Region not found

---

### POST /regions - Create Region

Creates a new geographic region.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/regions" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "region": {
      "name": "Northeast Region",
      "description": "Covers all northeast US locations",
      "boundary": {
        "type": "Polygon",
        "coordinates": [
          [
            [-74.2591, 40.4774],
            [-73.7004, 40.4774],
            [-73.7004, 40.9176],
            [-74.2591, 40.9176],
            [-74.2591, 40.4774]
          ]
        ]
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Region name |
| `description` | string | No | Region description |
| `boundary` | object | No | GeoJSON polygon boundary |

**Response 201:**
```json
{
  "data": {
    "id": "region-uuid-123",
    "type": "Region",
    "attributes": {
      "unique_id": "region-uuid-123",
      "name": "Northeast Region",
      "description": "Covers all northeast US locations",
      "boundary": {
        "type": "Polygon",
        "coordinates": [
          [
            [-74.2591, 40.4774],
            [-73.7004, 40.4774],
            [-73.7004, 40.9176],
            [-74.2591, 40.9176],
            [-74.2591, 40.4774]
          ]
        ]
      },
      "location_count": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (invalid boundary geometry)

---

### PUT /regions/:unique_id - Update Region

Updates an existing region.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/regions/region-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "region": {
      "name": "Northeast Region - Extended",
      "description": "Expanded to include additional northeast locations"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "region-uuid-123",
    "type": "Region",
    "attributes": {
      "unique_id": "region-uuid-123",
      "name": "Northeast Region - Extended",
      "description": "Expanded to include additional northeast locations",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Region not found

---

### DELETE /regions/:unique_id - Delete Region

Deletes a region.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/regions/region-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Region not found

---

## Region Tags

### POST /regions/:unique_id/tags/ - Add Tag

Adds a tag to a region.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/regions/region-uuid-123/tags/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag": {
      "unique_id": "tag-uuid-456"
    }
  }'
```

**Response 200:**
```json
{
  "message": "Tag added successfully"
}
```

---

### DELETE /regions/:unique_id/tags/:tag_unique_id - Remove Tag

Removes a tag from a region.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/regions/region-uuid-123/tags/tag-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Tag removed successfully"
}
```

---

## Data Models

### Region
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Region name |
| `description` | string | Region description |
| `boundary` | object | GeoJSON polygon boundary |
| `location_count` | integer | Number of locations in region |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Region Not Found","detail":"The requested region could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-geolocation`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useGeolocationBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// RegionsService — client.geolocation.regions
client.geolocation.regions.list(params?: ListRegionsParams): Promise<PageResult<Region>>;
client.geolocation.regions.get(uniqueId: string): Promise<Region>;
client.geolocation.regions.create(data: CreateRegionRequest): Promise<Region>;
client.geolocation.regions.update(uniqueId: string, data: UpdateRegionRequest): Promise<Region>;
client.geolocation.regions.delete(uniqueId: string): Promise<void>;
client.geolocation.regions.recover(uniqueId: string): Promise<Region>;
client.geolocation.regions.search(query: string, params?: ListRegionsParams): Promise<PageResult<Region>>;
client.geolocation.regions.listDeleted(params?: ListRegionsParams): Promise<PageResult<Region>>;
```

### TypeScript Types

```typescript
import type {
  Region,
  CreateRegionRequest,
  UpdateRegionRequest,
  ListRegionsParams,
} from '@23blocks/block-geolocation';
```
