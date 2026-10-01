---
name: 23blocks-campaigns-api
description: "Campaigns Block campaigns: CRUD, budgets, schedules, results, markets, locations, targets. Use when running a marketing campaign."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Campaigns API

Complete API reference for 23blocks campaign management with results tracking, market segments, location targeting, and campaign targets.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://campaigns.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/campaigns` | List all campaigns |
| GET | `/campaigns/:unique_id` | Get a single campaign |
| POST | `/campaigns` | Create a campaign |
| PUT | `/campaigns/:unique_id` | Update a campaign |
| DELETE | `/campaigns/:unique_id` | Delete a campaign |
| GET | `/campaigns/:unique_id/campaign_results` | Get campaign performance results |
| GET | `/campaigns/:unique_id/campaign_markets` | List campaign markets |
| POST | `/campaigns/:unique_id/campaign_markets` | Add market to campaign |
| DELETE | `/campaigns/:unique_id/campaign_markets/:market_id` | Remove market from campaign |
| GET | `/campaigns/:unique_id/campaign_locations` | List campaign locations |
| POST | `/campaigns/:unique_id/campaign_locations` | Add location to campaign |
| DELETE | `/campaigns/:unique_id/campaign_locations/:location_id` | Remove location from campaign |
| GET | `/campaigns/:unique_id/campaign_targets` | List campaign targets |
| POST | `/campaigns/:unique_id/campaign_targets` | Add target to campaign |
| DELETE | `/campaigns/:unique_id/campaign_targets/:target_id` | Remove target from campaign |

---

## Data Models

### Campaign
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Campaign name |
| `description` | string | Campaign description |
| `start_date` | date | Campaign start date |
| `end_date` | date | Campaign end date |
| `budget` | decimal | Campaign budget |
| `status` | enum | draft, active, paused, completed |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### CampaignResult
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `campaign_id` | uuid | Parent campaign ID |
| `impressions` | integer | Total impressions |
| `clicks` | integer | Total clicks |
| `conversions` | integer | Total conversions |
| `spend` | decimal | Amount spent |
| `revenue` | decimal | Revenue generated |
| `period_start` | date | Reporting period start |
| `period_end` | date | Reporting period end |
| `created_at` | timestamp | Creation time |

### CampaignMarket
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `campaign_id` | uuid | Parent campaign ID |
| `name` | string | Market name |
| `market_code` | string | Market code identifier |
| `created_at` | timestamp | Creation time |

### CampaignLocation
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `campaign_id` | uuid | Parent campaign ID |
| `name` | string | Location name |
| `location_type` | string | city, state, country, region |
| `country_code` | string | ISO country code |
| `region` | string | Region or state code |
| `created_at` | timestamp | Creation time |

### CampaignTarget
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `campaign_id` | uuid | Parent campaign ID |
| `name` | string | Target name |
| `target_type` | string | demographic, behavioral, geographic |
| `criteria` | object | Targeting criteria |
| `created_at` | timestamp | Creation time |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Campaign Not Found","detail":"The requested campaign could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-campaigns`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCampaignsBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// Campaigns — client.campaigns.campaigns
client.campaigns.campaigns.list(params?: ListCampaignsParams): Promise<PageResult<Campaign>>;
client.campaigns.campaigns.get(uniqueId: string): Promise<Campaign>;
client.campaigns.campaigns.create(data: CreateCampaignRequest): Promise<Campaign>;
client.campaigns.campaigns.update(uniqueId: string, data: UpdateCampaignRequest): Promise<Campaign>;
client.campaigns.campaigns.delete(uniqueId: string): Promise<void>;
client.campaigns.campaigns.start(uniqueId: string): Promise<Campaign>;
client.campaigns.campaigns.pause(uniqueId: string): Promise<Campaign>;
client.campaigns.campaigns.stop(uniqueId: string): Promise<Campaign>;
client.campaigns.campaigns.getResults(uniqueId: string): Promise<CampaignResults>;
```

### TypeScript Types

```typescript
import type {
  Campaign,
  CreateCampaignRequest,
  UpdateCampaignRequest,
  ListCampaignsParams,
  CampaignResults,
} from '@23blocks/block-campaigns';
```
