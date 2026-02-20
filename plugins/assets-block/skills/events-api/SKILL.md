---
name: assets-events-api
description: Create and manage 23blocks asset events via REST API. Use when recording, listing, or updating events on assets, including event image management with presigned URLs.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Events API

Complete API reference for 23blocks Assets Block event management on assets with image support.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://assets.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /assets/:unique_id/events - List Events

Lists all events for an asset.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "event-uuid-001",
      "type": "asset_event",
      "attributes": {
        "unique_id": "event-uuid-001",
        "asset_unique_id": "asset-uuid-123",
        "event_type": "inspection",
        "description": "Quarterly hardware inspection",
        "occurred_at": "2025-03-15T10:00:00Z",
        "recorded_by_unique_id": "user-uuid-456",
        "recorded_by_name": "John Doe",
        "images_count": 2,
        "payload": {},
        "created_at": "2025-03-15T10:30:00Z",
        "updated_at": "2025-03-15T10:30:00Z"
      }
    },
    {
      "id": "event-uuid-002",
      "type": "asset_event",
      "attributes": {
        "unique_id": "event-uuid-002",
        "asset_unique_id": "asset-uuid-123",
        "event_type": "repair",
        "description": "Replaced faulty keyboard",
        "occurred_at": "2025-02-20T14:00:00Z",
        "recorded_by_unique_id": "user-uuid-456",
        "recorded_by_name": "John Doe",
        "images_count": 1,
        "payload": {},
        "created_at": "2025-02-20T14:30:00Z",
        "updated_at": "2025-02-20T14:30:00Z"
      }
    }
  ]
}
```

---

### GET /assets/:unique_id/events/:event_unique_id - Get Event

Retrieves a single event by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123/events/event-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "event-uuid-001",
    "type": "asset_event",
    "attributes": {
      "unique_id": "event-uuid-001",
      "asset_unique_id": "asset-uuid-123",
      "event_type": "inspection",
      "description": "Quarterly hardware inspection",
      "occurred_at": "2025-03-15T10:00:00Z",
      "recorded_by_unique_id": "user-uuid-456",
      "recorded_by_name": "John Doe",
      "images_count": 2,
      "payload": {},
      "created_at": "2025-03-15T10:30:00Z",
      "updated_at": "2025-03-15T10:30:00Z"
    },
    "relationships": {
      "images": {
        "data": [
          { "id": "file-uuid-001", "type": "image" },
          { "id": "file-uuid-002", "type": "image" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Event not found

---

### POST /assets/:unique_id/events - Create Event

Creates a new event on an asset.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "event_type": "inspection",
      "description": "Annual safety and hardware inspection",
      "occurred_at": "2025-03-15T10:00:00Z",
      "payload": {
        "inspector": "Quality Team",
        "result": "passed"
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `event_type` | string | Yes | Type of event (e.g., inspection, repair, relocation, incident) |
| `description` | string | No | Event description |
| `occurred_at` | timestamp | No | When the event occurred (defaults to now) |
| `payload` | json | No | Custom event data |

**Response 201:**
```json
{
  "data": {
    "id": "new-event-uuid",
    "type": "asset_event",
    "attributes": {
      "unique_id": "new-event-uuid",
      "asset_unique_id": "asset-uuid-123",
      "event_type": "inspection",
      "description": "Annual safety and hardware inspection",
      "occurred_at": "2025-03-15T10:00:00Z",
      "recorded_by_unique_id": "current-user-uuid",
      "images_count": 0,
      "created_at": "2025-03-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Asset not found
- `422 Unprocessable Entity` - Validation errors

---

### PUT /assets/:unique_id/events/:event_unique_id - Update Event

Updates an existing event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assets/asset-uuid-123/events/event-uuid-001" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "description": "Updated inspection notes - minor wear detected",
      "payload": {
        "inspector": "Quality Team",
        "result": "passed_with_notes",
        "notes": "Minor wear on keyboard"
      }
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "event-uuid-001",
    "type": "asset_event",
    "attributes": {
      "unique_id": "event-uuid-001",
      "description": "Updated inspection notes - minor wear detected",
      "updated_at": "2025-03-15T12:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Event not found

---

## Event Images

### PUT /assets/:unique_id/events/:event_unique_id/presign - Presign Event Image

Generates a presigned URL for uploading an event image.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assets/asset-uuid-123/events/event-uuid-001/presign" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "inspection-photo.jpg",
      "content_type": "image/jpeg"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of the file |
| `content_type` | string | Yes | MIME type |

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://storage.example.com/upload?signature=xyz789",
    "file_unique_id": "file-uuid-003",
    "expires_at": "2025-03-15T11:30:00Z"
  }
}
```

---

### POST /assets/:unique_id/events/:event_unique_id/images - Upload Event Image

Confirms an event image upload after using the presigned URL.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/events/event-uuid-001/images" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "file_unique_id": "file-uuid-003"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "file-uuid-003",
    "type": "image",
    "attributes": {
      "unique_id": "file-uuid-003",
      "url": "https://storage.example.com/events/inspection-photo.jpg",
      "filename": "inspection-photo.jpg",
      "content_type": "image/jpeg",
      "created_at": "2025-03-15T10:30:00Z"
    }
  }
}
```

---

### DELETE /assets/:unique_id/events/:event_unique_id/images/:unique_file_id - Delete Event Image

Deletes an event image.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/assets/asset-uuid-123/events/event-uuid-001/images/file-uuid-003" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Data Models

### AssetEvent
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `asset_unique_id` | uuid | Parent asset ID |
| `event_type` | string | Type of event (inspection, repair, relocation, incident, etc.) |
| `description` | string | Event description |
| `occurred_at` | timestamp | When the event occurred |
| `recorded_by_unique_id` | uuid | User who recorded the event |
| `recorded_by_name` | string | Name of the recorder |
| `images_count` | integer | Number of attached images |
| `payload` | jsonb | Custom event data |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Event Not Found",
    "detail": "The requested event could not be found."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `404` | Not Found - Asset or event not found |
| `422` | Unprocessable Entity - Validation errors |

---

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-assets
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
// AssetEventsService — client.assets.assetEvents
client.assets.assetEvents.list(assetUniqueId: string, params?: ListAssetEventsParams): Promise<PageResult<AssetEvent>>;
client.assets.assetEvents.get(assetUniqueId: string, eventUniqueId: string): Promise<AssetEvent>;
client.assets.assetEvents.create(assetUniqueId: string, data: CreateAssetEventRequest): Promise<AssetEvent>;
client.assets.assetEvents.update(assetUniqueId: string, eventUniqueId: string, data: UpdateAssetEventRequest): Promise<AssetEvent>;
client.assets.assetEvents.reportList(params: EventReportParams): Promise<EventReportList>;
client.assets.assetEvents.reportSummary(params: EventReportParams): Promise<EventReportSummary>;
client.assets.assetEvents.presignImage(assetUniqueId: string, eventUniqueId: string): Promise<EventImagePresignResponse>;
client.assets.assetEvents.createImage(assetUniqueId: string, eventUniqueId: string, data: CreateEventImageRequest): Promise<EventImage>;
client.assets.assetEvents.deleteImage(assetUniqueId: string, eventUniqueId: string, imageUniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  AssetEvent,
  AssetEventType,
  CreateAssetEventRequest,
  UpdateAssetEventRequest,
  ListAssetEventsParams,
  EventImagePresignResponse,
  CreateEventImageRequest,
  EventImage,
} from '@23blocks/block-assets';

import type {
  EventReportParams,
  EventReportSummary,
  EventReportList,
} from '@23blocks/block-assets';
```

### React Hook

```typescript
import { useAssetsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useAssetsBlock();
  const result = await client.assets.assetEvents.list('asset-unique-id');
}
```
