---
name: 23blocks-geolocation-premises-api
description: Manage 23blocks premises via REST API. Use when creating, updating, or deleting premises within locations, managing premise events, tags, and areas for physical space management.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Premises API

Complete API reference for 23blocks premise management within locations, including events, tags, and areas.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/premises" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/locations/:unique_id/premises` | List premises in a location |
| GET | `/locations/:unique_id/premises/:premise_unique_id` | Get a single premise |
| POST | `/locations/:unique_id/premises` | Create a premise |
| PUT | `/locations/:unique_id/premises/:premise_unique_id` | Update a premise |
| DELETE | `/locations/:unique_id/premises/:premise_unique_id` | Delete a premise |
| POST | `/locations/:unique_id/premises/:premise_unique_id/tags` | Add tag to premise |
| DELETE | `/locations/:unique_id/premises/:premise_unique_id/tags/:tag_unique_id` | Remove tag from premise |
| GET | `/locations/:unique_id/premises/:premise_unique_id/events` | List premise events |
| POST | `/locations/:unique_id/premises/:premise_unique_id/events` | Create premise event |
| GET | `/areas/:unique_id` | Get an area |
| POST | `/areas/:unique_id/tags/` | Add tag to area |
| DELETE | `/areas/:unique_id/tags/:tag_unique_id` | Remove tag from area |

---

## Data Models

### Premise
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Premise name |
| `location_id` | uuid | Parent location ID |
| `floor` | string | Floor number/identifier |
| `capacity` | integer | Maximum capacity |
| `premise_type` | string | Type (conference_room, office, workspace, etc.) |
| `amenities` | array | List of available amenities |
| `status` | string | active, inactive, maintenance |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Event
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Event name |
| `description` | string | Event description |
| `start_time` | datetime | Event start time |
| `end_time` | datetime | Event end time |
| `attendees` | integer | Expected attendees |
| `status` | string | scheduled, in_progress, completed, cancelled |
| `created_at` | timestamp | Creation time |

### Area
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Area name |
| `description` | string | Area description |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Premise Not Found",
    "detail": "The requested premise could not be found."
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
// PremisesService — client.geolocation.premises
client.geolocation.premises.list(params?: ListPremisesParams): Promise<PageResult<Premise>>;
client.geolocation.premises.get(uniqueId: string): Promise<Premise>;
client.geolocation.premises.create(data: CreatePremiseRequest): Promise<Premise>;
client.geolocation.premises.update(uniqueId: string, data: UpdatePremiseRequest): Promise<Premise>;
client.geolocation.premises.delete(uniqueId: string): Promise<void>;
client.geolocation.premises.recover(uniqueId: string): Promise<Premise>;
client.geolocation.premises.search(query: string, params?: ListPremisesParams): Promise<PageResult<Premise>>;
client.geolocation.premises.listDeleted(params?: ListPremisesParams): Promise<PageResult<Premise>>;
```

### TypeScript Types

```typescript
import type {
  Premise,
  CreatePremiseRequest,
  UpdatePremiseRequest,
  ListPremisesParams,
} from '@23blocks/block-geolocation';
```

### React Hook

```typescript
import { useGeolocationBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useGeolocationBlock();
  const result = await client.geolocation.premises.list();
}
```
