---
name: 23blocks-geolocation-premises-api
description: "Geolocation Block premises inside a location: events, tags, areas. Use for rooms or spaces within a venue."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Premises API

Complete API reference for 23blocks premise management within locations, including events, tags, and areas.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://geolocation.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Premise Not Found","detail":"The requested premise could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-geolocation`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useGeolocationBlock()` from `@23blocks/react`.

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
