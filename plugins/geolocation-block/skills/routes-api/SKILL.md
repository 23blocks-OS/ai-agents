---
name: geolocation-routes-api
description: Manage 23blocks routes via REST API. Use when creating, updating, or deleting routes, adding location stops, assigning users to routes, retrieving user routes, or tracking user location along routes.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Routes API

Complete API reference for 23blocks route management with location stops, user assignment, and real-time location tracking.

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
curl -X GET "$BLOCKS_API_URL/routes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /routes - List Routes

Lists all routes with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/routes?page=1&records=20" \
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
      "id": "route-uuid-123",
      "type": "route",
      "attributes": {
        "unique_id": "route-uuid-123",
        "name": "Morning Delivery Route",
        "description": "Daily morning delivery stops in downtown area",
        "locations": [
          {
            "location_unique_id": "loc-uuid-001",
            "sequence": 1,
            "name": "Warehouse A"
          },
          {
            "location_unique_id": "loc-uuid-002",
            "sequence": 2,
            "name": "Customer Site B"
          }
        ],
        "distance": 25.4,
        "estimated_time": 45,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
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

### GET /routes/:unique_id/ - Get Route

Retrieves a single route by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/routes/route-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "route-uuid-123",
    "type": "route",
    "attributes": {
      "unique_id": "route-uuid-123",
      "name": "Morning Delivery Route",
      "description": "Daily morning delivery stops in downtown area",
      "locations": [
        {
          "location_unique_id": "loc-uuid-001",
          "sequence": 1,
          "name": "Warehouse A",
          "address": "100 Industrial Blvd",
          "latitude": 40.7128,
          "longitude": -74.0060
        },
        {
          "location_unique_id": "loc-uuid-002",
          "sequence": 2,
          "name": "Customer Site B",
          "address": "200 Commerce St",
          "latitude": 40.7200,
          "longitude": -74.0100
        }
      ],
      "distance": 25.4,
      "estimated_time": 45,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "users": {
        "data": [
          { "id": "user-uuid-456", "type": "user_identity" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route not found

---

### POST /routes/ - Create Route

Creates a new route.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/routes/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "route": {
      "name": "Morning Delivery Route",
      "description": "Daily morning delivery stops in downtown area"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Route name |
| `description` | string | No | Route description |

**Response 201:**
```json
{
  "data": {
    "id": "route-uuid-123",
    "type": "route",
    "attributes": {
      "unique_id": "route-uuid-123",
      "name": "Morning Delivery Route",
      "description": "Daily morning delivery stops in downtown area",
      "locations": [],
      "distance": 0,
      "estimated_time": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /routes/:unique_id/ - Update Route

Updates an existing route.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/routes/route-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "route": {
      "name": "Morning Delivery Route - Updated",
      "description": "Updated daily morning delivery stops"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "route-uuid-123",
    "type": "route",
    "attributes": {
      "unique_id": "route-uuid-123",
      "name": "Morning Delivery Route - Updated",
      "description": "Updated daily morning delivery stops",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route not found

---

### DELETE /routes/:unique_id/ - Delete Route

Deletes a route.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/routes/route-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Route not found

---

## Route Locations

### POST /routes/:unique_id/locations - Add Location to Route

Adds a location stop to a route.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/routes/route-uuid-123/locations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location_unique_id": "loc-uuid-001",
    "sequence": 1
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `location_unique_id` | uuid | Yes | Location to add as stop |
| `sequence` | integer | No | Order in the route |

**Response 201:**
```json
{
  "data": {
    "id": "route-loc-uuid",
    "type": "route_location",
    "attributes": {
      "route_unique_id": "route-uuid-123",
      "location_unique_id": "loc-uuid-001",
      "sequence": 1,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route or location not found
- `409 Conflict` - Location already in route

---

## Route Users

### POST /routes/:unique_id/users - Assign User to Route

Assigns a user to a route.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/routes/route-uuid-123/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "user-uuid-456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User to assign |

**Response 201:**
```json
{
  "data": {
    "id": "route-user-uuid",
    "type": "route_user",
    "attributes": {
      "route_unique_id": "route-uuid-123",
      "user_unique_id": "user-uuid-456",
      "assigned_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route or user not found
- `409 Conflict` - User already assigned to route

---

### GET /users/:unique_id/routes - User Routes

Retrieves all routes assigned to a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/routes?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "route-uuid-123",
      "type": "route",
      "attributes": {
        "unique_id": "route-uuid-123",
        "name": "Morning Delivery Route",
        "description": "Daily morning delivery stops",
        "distance": 25.4,
        "estimated_time": 45,
        "status": "active"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

## Location Tracking

### POST /users/:unique_id/routes/:route_unique_id/tracker/location - Track Location

Reports the current location of a user on a route for real-time tracking.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-456/routes/route-uuid-123/tracker/location" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tracker": {
      "latitude": 40.7150,
      "longitude": -74.0080,
      "accuracy": 10.5,
      "speed": 35.2,
      "heading": 90
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `latitude` | float | Yes | Current latitude |
| `longitude` | float | Yes | Current longitude |
| `accuracy` | float | No | GPS accuracy in meters |
| `speed` | float | No | Current speed in km/h |
| `heading` | float | No | Direction in degrees (0-360) |

**Response 200:**
```json
{
  "data": {
    "id": "tracker-uuid",
    "type": "tracker_location",
    "attributes": {
      "user_unique_id": "user-uuid-456",
      "route_unique_id": "route-uuid-123",
      "latitude": 40.7150,
      "longitude": -74.0080,
      "accuracy": 10.5,
      "speed": 35.2,
      "heading": 90,
      "tracked_at": "2025-01-12T14:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User or route not found
- `422 Unprocessable Entity` - Invalid coordinates

---

## Data Models

### Route
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Route name |
| `description` | string | Route description |
| `locations` | array | Ordered list of location stops |
| `distance` | float | Total distance in km |
| `estimated_time` | integer | Estimated time in minutes |
| `status` | string | active, inactive, completed |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### RouteLocation
| Field | Type | Description |
|-------|------|-------------|
| `route_unique_id` | uuid | Parent route ID |
| `location_unique_id` | uuid | Location stop ID |
| `sequence` | integer | Order in route |

### TrackerLocation
| Field | Type | Description |
|-------|------|-------------|
| `user_unique_id` | uuid | User being tracked |
| `route_unique_id` | uuid | Route being followed |
| `latitude` | float | Current latitude |
| `longitude` | float | Current longitude |
| `accuracy` | float | GPS accuracy in meters |
| `speed` | float | Current speed in km/h |
| `heading` | float | Direction in degrees |
| `tracked_at` | timestamp | Tracking timestamp |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Route Not Found",
    "detail": "The requested route could not be found."
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
// TravelRoutesService — client.geolocation.routes
client.geolocation.routes.list(params?: ListTravelRoutesParams): Promise<PageResult<TravelRoute>>;
client.geolocation.routes.get(uniqueId: string): Promise<TravelRoute>;
client.geolocation.routes.create(data: CreateTravelRouteRequest): Promise<TravelRoute>;
client.geolocation.routes.update(uniqueId: string, data: UpdateTravelRouteRequest): Promise<TravelRoute>;
client.geolocation.routes.delete(uniqueId: string): Promise<void>;
client.geolocation.routes.recover(uniqueId: string): Promise<TravelRoute>;
client.geolocation.routes.search(query: string, params?: ListTravelRoutesParams): Promise<PageResult<TravelRoute>>;
client.geolocation.routes.listDeleted(params?: ListTravelRoutesParams): Promise<PageResult<TravelRoute>>;
```

### TypeScript Types

```typescript
import type {
  TravelRoute,
  CreateTravelRouteRequest,
  UpdateTravelRouteRequest,
  ListTravelRoutesParams,
} from '@23blocks/block-geolocation';
```

### React Hook

```typescript
import { useGeolocationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useGeolocationBlock();
  const result = await client.geolocation.routes.list();
}
```
