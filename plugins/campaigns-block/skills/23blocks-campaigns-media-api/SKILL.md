---
name: 23blocks-campaigns-media-api
description: Manage 23blocks media assets via REST API. Use when uploading media, assigning media to campaigns, or tracking media performance results across campaign media assignments.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Media API

Complete API reference for 23blocks media asset management, campaign media assignments, and media performance tracking.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Campaigns API base URL | `https://campaigns.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://campaigns.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://campaigns.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```


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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Media Not Found",
    "detail": "The requested media asset could not be found."
  }]
}
```

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-campaigns
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

### React Hook

```typescript
import { useCampaignsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCampaignsBlock();
  const result = await client.campaigns.media.list();
}
```
