---
name: 23blocks-crm-zoom-api
description: Manage 23blocks CRM Zoom integration via REST API. Use when creating Zoom meetings linked to CRM meetings, checking user availability, handling Zoom webhooks, and managing Zoom hosts with availability and allocations.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# CRM Zoom API

Complete API reference for 23blocks CRM Zoom meeting integration with host management, availability tracking, and webhook handling.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | CRM API base URL | `https://crm.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/meetings/meeting-uuid-456/zoom" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/users/:user_unique_id/meetings/:meeting_unique_id/zoom` | Get Zoom meeting |
| POST | `/users/:user_unique_id/meetings/:meeting_unique_id/zoom` | Create Zoom meeting |
| PUT | `/users/:user_unique_id/meetings/:meeting_unique_id/zoom` | Update Zoom meeting |
| DELETE | `/users/:user_unique_id/meetings/:meeting_unique_id/zoom` | Delete Zoom meeting |
| GET | `/users/:user_unique_id/zoom/availability` | Check user Zoom availability |
| POST | `/zoom/webhooks` | Handle Zoom webhook event |
| GET | `/zoom_hosts/available_users` | List available Zoom users |
| GET | `/zoom_hosts/` | List Zoom hosts |
| GET | `/zoom_hosts/:unique_id` | Get Zoom host |
| POST | `/zoom_hosts/` | Create Zoom host |
| PUT | `/zoom_hosts/:unique_id` | Update Zoom host |
| DELETE | `/zoom_hosts/:unique_id` | Delete Zoom host |
| GET | `/zoom_hosts/:unique_id/availability` | Check host availability |
| GET | `/zoom_hosts/:unique_id/allocations` | Get host allocations |

---

## Data Models

### ZoomMeeting
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `meeting_id` | uuid | CRM meeting ID |
| `zoom_meeting_id` | string | Zoom platform meeting ID |
| `join_url` | string | Zoom join URL |
| `host_email` | string | Zoom host email |
| `duration` | integer | Duration in minutes |
| `status` | enum | waiting, started, ended |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### ZoomHost
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `user_id` | uuid | Associated user ID |
| `email` | string | Zoom host email |
| `name` | string | Host name |
| `max_concurrent_meetings` | integer | Max concurrent meetings |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Zoom Meeting Not Found",
    "detail": "The requested Zoom meeting could not be found."
  }]
}
```

## SDK Usage (TypeScript)

> **When building web apps, use the SDK instead of raw API calls.**

### Installation

```bash
npm install @23blocks/block-crm
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
// ZoomMeetingsService — client.crm.zoom
get(userUniqueId: string, meetingUniqueId: string): Promise<ZoomMeeting>;
provision(userUniqueId: string, meetingUniqueId: string, request?: ProvisionZoomMeetingRequest): Promise<ZoomMeeting>;
update(userUniqueId: string, meetingUniqueId: string, request: UpdateZoomMeetingRequest): Promise<ZoomMeeting>;
cancel(userUniqueId: string, meetingUniqueId: string): Promise<void>;
checkAvailability(userUniqueId: string): Promise<ZoomAvailability>;

// ZoomHostsService — client.crm.zoomHosts
list(params?: ListZoomHostsParams): Promise<PageResult<ZoomHost>>;
get(uniqueId: string): Promise<ZoomHost>;
create(data: CreateZoomHostRequest): Promise<ZoomHost>;
update(uniqueId: string, data: UpdateZoomHostRequest): Promise<ZoomHost>;
delete(uniqueId: string): Promise<void>;
getAvailability(uniqueId: string): Promise<ZoomHostAvailability>;
getAllocations(uniqueId: string): Promise<ZoomHostAllocation[]>;
getAvailableUsers(): Promise<AvailableUser[]>;
```

### TypeScript Types

```typescript
import type {
  ZoomMeeting,
  ZoomAvailability,
  ProvisionZoomMeetingRequest,
  UpdateZoomMeetingRequest,
  ZoomHost,
  ZoomHostAvailability,
  ZoomHostAllocation,
  AvailableUser,
  CreateZoomHostRequest,
  UpdateZoomHostRequest,
  ListZoomHostsParams,
} from '@23blocks/block-crm';
```

### React Hook

```typescript
import { useCrmBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCrmBlock();

  // Example: Check Zoom availability for a user
  const result = await client.crm.zoom.checkAvailability('user-unique-id');
}
```
