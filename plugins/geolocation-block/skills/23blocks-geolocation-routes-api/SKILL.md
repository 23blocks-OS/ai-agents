---
name: 23blocks-geolocation-routes-api
description: Manage 23blocks routes via REST API. Use when creating, updating, or deleting routes, adding location stops, assigning users to routes, retrieving user routes, or tracking user location along routes.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Routes API

Complete API reference for 23blocks route management with location stops, user assignment, and real-time location tracking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://geolocation.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://geolocation.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/routes` | List all routes |
| GET | `/routes/:unique_id/` | Get a single route |
| POST | `/routes/` | Create a route |
| PUT | `/routes/:unique_id/` | Update a route |
| DELETE | `/routes/:unique_id/` | Delete a route |
| POST | `/routes/:unique_id/locations` | Add location stop to route |
| POST | `/routes/:unique_id/users` | Assign user to route |
| GET | `/users/:unique_id/routes` | Get routes assigned to a user |
| POST | `/users/:unique_id/routes/:route_unique_id/tracker/location` | Report user location on route |

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
