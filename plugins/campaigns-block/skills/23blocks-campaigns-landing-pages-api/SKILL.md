---
name: 23blocks-campaigns-landing-pages-api
description: "Campaigns Block landing pages: slugs, audiences, Facebook audiences, templates. Use when building campaign landing pages."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Landing Pages API

Complete API reference for 23blocks landing page management with audience targeting, Facebook audience integration, and landing templates.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://campaigns.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Landing Page Not Found","detail":"The requested landing page could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-campaigns`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCampaignsBlock()` from `@23blocks/react`.

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
