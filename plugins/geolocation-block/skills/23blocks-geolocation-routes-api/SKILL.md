---
name: 23blocks-geolocation-routes-api
description: "Geolocation Block routes: stops, assigned users, live tracking. Use for delivery or field routes."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Routes API

Complete API reference for 23blocks route management with location stops, user assignment, and real-time location tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://geolocation.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Route Not Found","detail":"The requested route could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-geolocation`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useGeolocationBlock()` from `@23blocks/react`.

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
