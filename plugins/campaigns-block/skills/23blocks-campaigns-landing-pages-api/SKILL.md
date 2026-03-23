---
name: 23blocks-campaigns-landing-pages-api
description: Manage 23blocks campaign landing pages via REST API. Use when creating landing pages with URL slugs, defining target audiences, creating Facebook audiences for social targeting, or managing landing page templates.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Landing Pages API

Complete API reference for 23blocks landing page management with audience targeting, Facebook audience integration, and landing templates.

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
| GET | `/landing_pages` | List all landing pages |
| GET | `/landing_pages/:unique_id` | Get a single landing page |
| POST | `/landing_pages` | Create a landing page |
| PUT | `/landing_pages/:unique_id` | Update a landing page |
| DELETE | `/landing_pages/:unique_id` | Delete a landing page |
| GET | `/landing_audiences` | List all landing audiences |
| POST | `/landing_audiences` | Create a landing audience |
| PUT | `/landing_audiences/:unique_id` | Update a landing audience |
| DELETE | `/landing_audiences/:unique_id` | Delete a landing audience |
| POST | `/audience/facebook` | Create Facebook audience |
| GET | `/landing_templates` | List landing templates |
| POST | `/landing_templates` | Create a landing template |

---

## Data Models

### LandingPage
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Landing page name |
| `url_slug` | string | URL-friendly slug |
| `template_id` | uuid | Template ID for rendering |
| `audience_id` | uuid | Target audience ID |
| `campaign_id` | uuid | Associated campaign ID |
| `conversion_rate` | decimal | Page conversion rate |
| `status` | enum | draft, active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### LandingAudience
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Audience name |
| `description` | string | Audience description |
| `landing_page_id` | uuid | Associated landing page ID |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### FacebookAudience
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Facebook audience name |
| `landing_audience_id` | uuid | Parent landing audience ID |
| `age_min` | integer | Minimum target age |
| `age_max` | integer | Maximum target age |
| `genders` | array | Target genders |
| `interests` | array | Interest keywords |
| `locations` | array | Target country codes |
| `languages` | array | Target language codes |
| `estimated_reach` | integer | Estimated audience reach |
| `created_at` | timestamp | Creation time |

### LandingTemplate
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Template name |
| `description` | string | Template description |
| `layout` | string | Layout type |
| `content` | string | Template HTML content |
| `preview_url` | string | Preview image URL |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Landing Page Not Found",
    "detail": "The requested landing page could not be found."
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
// LandingPages — client.campaigns.landingPages
client.campaigns.landingPages.list(params?: ListLandingPagesParams): Promise<PageResult<LandingPage>>;
client.campaigns.landingPages.get(uniqueId: string): Promise<LandingPage>;
client.campaigns.landingPages.create(data: CreateLandingPageRequest): Promise<LandingPage>;
client.campaigns.landingPages.update(uniqueId: string, data: UpdateLandingPageRequest): Promise<LandingPage>;
client.campaigns.landingPages.delete(uniqueId: string): Promise<void>;
client.campaigns.landingPages.publish(uniqueId: string): Promise<LandingPage>;
client.campaigns.landingPages.unpublish(uniqueId: string): Promise<LandingPage>;
client.campaigns.landingPages.getBySlug(slug: string): Promise<LandingPage>;
```

### TypeScript Types

```typescript
import type {
  LandingPage,
  CreateLandingPageRequest,
  UpdateLandingPageRequest,
  ListLandingPagesParams,
} from '@23blocks/block-campaigns';
```

### React Hook

```typescript
import { useCampaignsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCampaignsBlock();
  const result = await client.campaigns.landingPages.list();
}
```
