---
name: 23blocks-geolocation-locations-api
description: Manage 23blocks locations via REST API. Use when creating, updating, or deleting locations, managing hours, images, slots, taxes, tags, identities, QR codes, geographic hierarchy lookups, and location groups.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Locations API

Complete API reference for 23blocks location management with hours, images, slots, taxes, tags, identities, QR codes, geographic hierarchy, and location groups.

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
| GET | `/locations` | List all locations |
| GET | `/locations/:unique_id/` | Get a single location |
| GET | `/locations/:unique_id/qrcode` | Get QR code for a location |
| POST | `/locations/` | Create a location |
| PUT | `/locations/:unique_id/` | Update a location |
| DELETE | `/locations/:unique_id/` | Delete a location |
| POST | `/locations/search/code` | Search location by code |
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
| GET | `/countries/:country_code/locations` | Locations by country |
| GET | `/states/:code/locations` | Locations by state |
| GET | `/counties/:code/locations` | Locations by county |
| GET | `/cities/:code/locations` | Locations by city |
| GET | `/divisions/:code/locations` | Locations by division |
| GET | `/neighborhoods/:code/locations` | Locations by neighborhood |
| GET | `/buildings/:code/locations` | Locations by building |
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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Location Not Found",
    "detail": "The requested location could not be found."
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

### React Hook

```typescript
import { useGeolocationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useGeolocationBlock();
  const result = await client.geolocation.locations.list();
}
```
