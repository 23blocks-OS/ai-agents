---
name: 23blocks-geolocation-locations-api
description: "Geolocation Block locations: hours, images, slots, taxes, tags, QR codes, geographic lookups, groups. Use for places and venues."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Locations API

Complete API reference for 23blocks location management with hours, images, slots, taxes, tags, identities, QR codes, geographic hierarchy, and location groups.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://geolocation.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/locations` | List all locations (public) |
| GET | `/locations/:unique_id/` | Get a single location (public) |
| GET | `/locations/:unique_id/qrcode` | Get QR code for a location (public) |
| POST | `/locations/` | Create a location |
| PUT | `/locations/:unique_id/` | Update a location |
| DELETE | `/locations/:unique_id/` | Delete a location |
| POST | `/locations/search/code` | Search location by code (public) |
| POST | `/locations/:unique_id/tags/` | Add tag to location |
| DELETE | `/locations/:unique_id/tags/:tag_unique_id` | Remove tag from location |
| GET | `/locations/:unique_id/hours/` | List operating hours |
| GET | `/locations/:unique_id/hours/:hour_unique_id` | Get a specific hour entry |
| POST | `/locations/:unique_id/hours/` | Create an hour entry |
| PUT | `/locations/:unique_id/hours/:hour_unique_id` | Update an hour entry |
| DELETE | `/locations/:unique_id/hours/:hour_unique_id` | Delete an hour entry |
| PUT | `/locations/:unique_id/presign` | Presign image upload URL |
| POST | `/locations/:unique_id/images` | Register uploaded image |
| DELETE | `/locations/:unique_id/images/:image_unique_id` | Delete image |
| POST | `/locations/:unique_id/identities` | Add user identity |
| DELETE | `/locations/:unique_id/identities/:user_unique_id` | Remove user identity |
| GET | `/locations/:unique_id/slots` | List time slots |
| POST | `/locations/:unique_id/slots` | Create time slot |
| PUT | `/locations/:unique_id/slots/:slot_unique_id` | Update time slot |
| DELETE | `/locations/:unique_id/slots/:slot_unique_id` | Delete time slot |
| POST | `/locations/:unique_id/taxes` | Add tax configuration |
| PUT | `/locations/:unique_id/taxes/:tax_unique_id` | Update tax configuration |
| DELETE | `/locations/:unique_id/taxes/:tax_unique_id` | Delete tax configuration |
| GET | `/countries/:country_code/locations` | Locations by country (public) |
| GET | `/states/:code/locations` | Locations by state (public) |
| GET | `/counties/:code/locations` | Locations by county (public) |
| GET | `/cities/:code/locations` | Locations by city (public) |
| GET | `/divisions/:code/locations` | Locations by division (public) |
| GET | `/neighborhoods/:code/locations` | Locations by neighborhood (public) |
| GET | `/buildings/:code/locations` | Locations by building (public) |
| GET | `/location_groups` | List location groups |
| GET | `/location_groups/:unique_id/` | Get a location group |
| POST | `/location_groups/` | Create a location group |

---

## Data Models

### Location
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Location name |
| `code` | string | Unique location code |
| `address` | string | Full address |
| `latitude` | float | Latitude coordinate |
| `longitude` | float | Longitude coordinate |
| `phone` | string | Contact phone |
| `email` | string | Contact email |
| `location_type` | string | Type (office, store, warehouse, etc.) |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Hour
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `day_of_week` | string | Day of the week |
| `open_time` | string | Opening time (HH:MM) |
| `close_time` | string | Closing time (HH:MM) |
| `is_closed` | boolean | Whether closed on this day |

### Slot
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Slot name |
| `start_time` | string | Start time (HH:MM) |
| `end_time` | string | End time (HH:MM) |
| `capacity` | integer | Maximum capacity |
| `status` | string | active, inactive |

### Tax
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Tax name |
| `rate` | float | Tax rate percentage |
| `tax_type` | string | Type of tax |

### LocationGroup
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Group name |
| `description` | string | Group description |
| `location_count` | integer | Number of locations |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Location Not Found","detail":"The requested location could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-geolocation`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useGeolocationBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// LocationsService — client.geolocation.locations
client.geolocation.locations.list(params?: ListLocationsParams): Promise<PageResult<Location>>;
client.geolocation.locations.get(uniqueId: string): Promise<Location>;
client.geolocation.locations.create(data: CreateLocationRequest): Promise<Location>;
client.geolocation.locations.update(uniqueId: string, data: UpdateLocationRequest): Promise<Location>;
client.geolocation.locations.delete(uniqueId: string): Promise<void>;
client.geolocation.locations.recover(uniqueId: string): Promise<Location>;
client.geolocation.locations.search(query: string, params?: ListLocationsParams): Promise<PageResult<Location>>;
client.geolocation.locations.listDeleted(params?: ListLocationsParams): Promise<PageResult<Location>>;
client.geolocation.locations.getQRCode(uniqueId: string): Promise<string>;
client.geolocation.locations.searchByCode(code: string): Promise<Location[]>;
client.geolocation.locations.addTag(uniqueId: string, tagUniqueId: string): Promise<Location>;
client.geolocation.locations.removeTag(uniqueId: string, tagUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Location,
  CreateLocationRequest,
  UpdateLocationRequest,
  ListLocationsParams,
} from '@23blocks/block-geolocation';
```
