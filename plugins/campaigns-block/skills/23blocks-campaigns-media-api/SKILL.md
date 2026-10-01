---
name: 23blocks-campaigns-media-api
description: "Campaigns Block media: upload, assign to campaigns, performance results. Use for campaign creative assets."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Media API

Complete API reference for 23blocks media asset management, campaign media assignments, and media performance tracking.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://campaigns.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/media` | List all media assets |
| GET | `/media/:unique_id` | Get a single media asset |
| POST | `/media` | Create a media asset |
| PUT | `/media/:unique_id` | Update a media asset |
| DELETE | `/media/:unique_id` | Delete a media asset |
| GET | `/campaign_media` | List campaign media assignments |
| POST | `/campaign_media` | Assign media to campaign |
| DELETE | `/campaign_media/:unique_id` | Remove media from campaign |
| GET | `/campaign_media_results` | Get media performance results |

---

## Data Models

### Media
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Media asset name |
| `media_type` | enum | image, video, document |
| `url` | string | Media file URL |
| `file_size` | integer | File size in bytes |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### CampaignMedia
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `campaign_id` | uuid | Associated campaign ID |
| `media_id` | uuid | Associated media ID |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

### CampaignMediaResult
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `campaign_media_id` | uuid | Campaign media assignment ID |
| `campaign_id` | uuid | Campaign ID |
| `media_id` | uuid | Media asset ID |
| `impressions` | integer | Total impressions |
| `clicks` | integer | Total clicks |
| `conversions` | integer | Total conversions |
| `click_through_rate` | decimal | CTR ratio |
| `conversion_rate` | decimal | Conversion ratio |
| `spend` | decimal | Amount spent |
| `period_start` | date | Reporting period start |
| `period_end` | date | Reporting period end |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Media Not Found","detail":"The requested media asset could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-campaigns`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCampaignsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// Media — client.campaigns.media
client.campaigns.media.list(params?: ListMediaParams): Promise<PageResult<Media>>;
client.campaigns.media.get(uniqueId: string): Promise<Media>;
client.campaigns.media.create(data: CreateMediaRequest): Promise<Media>;
client.campaigns.media.update(uniqueId: string, data: UpdateMediaRequest): Promise<Media>;
client.campaigns.media.delete(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Media,
  CreateMediaRequest,
  UpdateMediaRequest,
  ListMediaParams,
} from '@23blocks/block-campaigns';
```
