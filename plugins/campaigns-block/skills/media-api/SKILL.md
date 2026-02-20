---
name: campaigns-media-api
description: Manage 23blocks media assets via REST API. Use when uploading media, assigning media to campaigns, or tracking media performance results across campaign media assignments.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Media API

Complete API reference for 23blocks media asset management, campaign media assignments, and media performance tracking.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://campaigns.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Campaigns API base URL | `https://campaigns.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/media" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Media Endpoints

### GET /media - List Media

Lists all media assets with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/media?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `media_type` | string | No | Filter by type (image, video, document) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "media-uuid-123",
      "type": "media",
      "attributes": {
        "unique_id": "media-uuid-123",
        "name": "Summer Banner",
        "media_type": "image",
        "url": "https://cdn.example.com/banners/summer-2026.jpg",
        "file_size": 245760,
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 87
  }
}
```

---

### GET /media/:unique_id - Get Media

Retrieves a single media asset by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/media/media-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "media-uuid-123",
    "type": "media",
    "attributes": {
      "unique_id": "media-uuid-123",
      "name": "Summer Banner",
      "media_type": "image",
      "url": "https://cdn.example.com/banners/summer-2026.jpg",
      "file_size": 245760,
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Media not found

---

### POST /media - Create Media

Creates a new media asset.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/media" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "media": {
      "name": "Summer Banner",
      "media_type": "image",
      "url": "https://cdn.example.com/banners/summer-2026.jpg",
      "file_size": 245760
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Media asset name |
| `media_type` | enum | Yes | image, video, document |
| `url` | string | Yes | Media file URL |
| `file_size` | integer | No | File size in bytes |
| `status` | enum | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "media-uuid-123",
    "type": "media",
    "attributes": {
      "unique_id": "media-uuid-123",
      "name": "Summer Banner",
      "media_type": "image",
      "url": "https://cdn.example.com/banners/summer-2026.jpg",
      "file_size": 245760,
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /media/:unique_id - Update Media

Updates an existing media asset.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/media/media-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "media": {
      "name": "Summer Banner - Updated",
      "url": "https://cdn.example.com/banners/summer-2026-v2.jpg"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "media-uuid-123",
    "type": "media",
    "attributes": {
      "unique_id": "media-uuid-123",
      "name": "Summer Banner - Updated",
      "url": "https://cdn.example.com/banners/summer-2026-v2.jpg",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Media not found

---

### DELETE /media/:unique_id - Delete Media

Deletes a media asset.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/media/media-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Media not found

---

## Campaign Media Endpoints

### GET /campaign_media - List Campaign Media

Lists all campaign media assignments with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaign_media?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `campaign_id` | uuid | No | Filter by campaign |
| `media_id` | uuid | No | Filter by media asset |

**Response 200:**
```json
{
  "data": [
    {
      "id": "cm-uuid-123",
      "type": "campaign_media",
      "attributes": {
        "unique_id": "cm-uuid-123",
        "campaign_id": "campaign-uuid-123",
        "media_id": "media-uuid-123",
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z"
      },
      "relationships": {
        "campaign": {
          "data": { "id": "campaign-uuid-123", "type": "campaign" }
        },
        "media": {
          "data": { "id": "media-uuid-123", "type": "media" }
        }
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 24
  }
}
```

---

### POST /campaign_media - Assign Media to Campaign

Assigns a media asset to a campaign.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/campaign_media" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_media": {
      "campaign_id": "campaign-uuid-123",
      "media_id": "media-uuid-456"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `campaign_id` | uuid | Yes | Campaign unique ID |
| `media_id` | uuid | Yes | Media asset unique ID |

**Response 201:**
```json
{
  "data": {
    "id": "cm-uuid-456",
    "type": "campaign_media",
    "attributes": {
      "unique_id": "cm-uuid-456",
      "campaign_id": "campaign-uuid-123",
      "media_id": "media-uuid-456",
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Campaign or media not found
- `409 Conflict` - Media already assigned to campaign
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /campaign_media/:unique_id - Remove Media from Campaign

Removes a media assignment from a campaign.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/campaign_media/cm-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Campaign Media Results

### GET /campaign_media_results - Get Media Performance Results

Retrieves performance results for media across campaigns.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaign_media_results?campaign_id=campaign-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `campaign_id` | uuid | No | Filter by campaign |
| `media_id` | uuid | No | Filter by media asset |
| `campaign_media_id` | uuid | No | Filter by campaign media assignment |

**Response 200:**
```json
{
  "data": [
    {
      "id": "cmr-uuid-123",
      "type": "campaign_media_result",
      "attributes": {
        "unique_id": "cmr-uuid-123",
        "campaign_media_id": "cm-uuid-123",
        "campaign_id": "campaign-uuid-123",
        "media_id": "media-uuid-123",
        "impressions": 75000,
        "clicks": 2250,
        "conversions": 180,
        "click_through_rate": 0.03,
        "conversion_rate": 0.08,
        "spend": 5000.00,
        "period_start": "2026-06-01",
        "period_end": "2026-06-30",
        "created_at": "2026-07-01T00:00:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 8
  }
}
```

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
