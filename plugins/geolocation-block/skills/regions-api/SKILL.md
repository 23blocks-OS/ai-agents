---
name: geolocation-regions-api
description: Manage 23blocks geographic regions via REST API. Use when creating, updating, or deleting regions, viewing locations within regions, or organizing regions with tags.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Regions API

Complete API reference for 23blocks geographic region management with location associations and tags.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://geolocation.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/regions/region-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /regions/:unique_id/locations - Region Locations

Lists all locations within a specific region.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/regions/region-uuid-123/locations?page=1&records=20" \
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
      "id": "loc-uuid-123",
      "type": "location",
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
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "region-uuid-123",
    "type": "region",
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
          { "id": "tag-uuid", "type": "tag" }
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
  -H "AppId: $BLOCKS_API_KEY" \
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
    "type": "region",
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
  -H "AppId: $BLOCKS_API_KEY" \
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
    "type": "region",
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
  -H "AppId: $BLOCKS_API_KEY"
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

### DELETE /regions/:unique_id/tags/:tag_unique_id - Remove Tag

Removes a tag from a region.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/regions/region-uuid-123/tags/tag-uuid-456" \
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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Region Not Found",
    "detail": "The requested region could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-geolocation
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

### React Hook

```typescript
import { useGeolocationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useGeolocationBlock();
  const result = await client.geolocation.regions.list();
}
```
