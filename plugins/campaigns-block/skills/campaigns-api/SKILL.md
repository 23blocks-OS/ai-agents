---
name: campaigns-campaigns-api
description: Manage 23blocks marketing campaigns via REST API. Use when creating campaigns, managing campaign budgets and schedules, tracking campaign results, configuring markets, locations, and targeting rules.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Campaigns API

Complete API reference for 23blocks campaign management with results tracking, market segments, location targeting, and campaign targets.

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
curl -X GET "$BLOCKS_API_URL/campaigns" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /campaigns - List Campaigns

Lists all campaigns with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaigns?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `status` | string | No | Filter by status (draft, active, paused, completed) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "campaign-uuid-123",
      "type": "campaign",
      "attributes": {
        "unique_id": "campaign-uuid-123",
        "name": "Summer Sale 2026",
        "description": "Annual summer promotional campaign",
        "start_date": "2026-06-01",
        "end_date": "2026-08-31",
        "budget": 50000.00,
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### GET /campaigns/:unique_id - Get Campaign

Retrieves a single campaign by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaigns/campaign-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "campaign-uuid-123",
    "type": "campaign",
    "attributes": {
      "unique_id": "campaign-uuid-123",
      "name": "Summer Sale 2026",
      "description": "Annual summer promotional campaign",
      "start_date": "2026-06-01",
      "end_date": "2026-08-31",
      "budget": 50000.00,
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "campaign_results": {
        "data": [
          { "id": "result-uuid-1", "type": "campaign_result" }
        ]
      },
      "campaign_markets": {
        "data": [
          { "id": "market-uuid-1", "type": "campaign_market" }
        ]
      },
      "campaign_locations": {
        "data": [
          { "id": "location-uuid-1", "type": "campaign_location" }
        ]
      },
      "campaign_targets": {
        "data": [
          { "id": "target-uuid-1", "type": "campaign_target" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Campaign not found

---

### POST /campaigns - Create Campaign

Creates a new marketing campaign.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/campaigns" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign": {
      "name": "Summer Sale 2026",
      "description": "Annual summer promotional campaign",
      "start_date": "2026-06-01",
      "end_date": "2026-08-31",
      "budget": 50000.00,
      "status": "draft"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Campaign name |
| `description` | string | No | Campaign description |
| `start_date` | date | No | Campaign start date (YYYY-MM-DD) |
| `end_date` | date | No | Campaign end date (YYYY-MM-DD) |
| `budget` | decimal | No | Campaign budget |
| `status` | enum | No | draft, active, paused, completed (default: draft) |

**Response 201:**
```json
{
  "data": {
    "id": "campaign-uuid-123",
    "type": "campaign",
    "attributes": {
      "unique_id": "campaign-uuid-123",
      "name": "Summer Sale 2026",
      "description": "Annual summer promotional campaign",
      "start_date": "2026-06-01",
      "end_date": "2026-08-31",
      "budget": 50000.00,
      "status": "draft",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /campaigns/:unique_id - Update Campaign

Updates an existing campaign.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/campaigns/campaign-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign": {
      "name": "Summer Sale 2026 - Extended",
      "end_date": "2026-09-15",
      "budget": 65000.00,
      "status": "active"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "campaign-uuid-123",
    "type": "campaign",
    "attributes": {
      "unique_id": "campaign-uuid-123",
      "name": "Summer Sale 2026 - Extended",
      "end_date": "2026-09-15",
      "budget": 65000.00,
      "status": "active",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Campaign not found

---

### DELETE /campaigns/:unique_id - Delete Campaign

Deletes a campaign.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/campaigns/campaign-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Campaign not found

---

## Campaign Results

### GET /campaigns/:unique_id/campaign_results - Get Campaign Results

Retrieves performance results for a campaign.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_results" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "result-uuid-1",
      "type": "campaign_result",
      "attributes": {
        "unique_id": "result-uuid-1",
        "campaign_id": "campaign-uuid-123",
        "impressions": 150000,
        "clicks": 4500,
        "conversions": 320,
        "spend": 12500.00,
        "revenue": 48000.00,
        "period_start": "2026-06-01",
        "period_end": "2026-06-30",
        "created_at": "2026-07-01T00:00:00Z"
      }
    }
  ]
}
```

---

## Campaign Markets

### GET /campaigns/:unique_id/campaign_markets - List Campaign Markets

Lists market segments assigned to a campaign.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_markets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "market-uuid-1",
      "type": "campaign_market",
      "attributes": {
        "unique_id": "market-uuid-1",
        "campaign_id": "campaign-uuid-123",
        "name": "North America",
        "market_code": "NA",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /campaigns/:unique_id/campaign_markets - Add Campaign Market

Adds a market segment to a campaign.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_markets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_market": {
      "name": "North America",
      "market_code": "NA"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Market name |
| `market_code` | string | Yes | Market code identifier |

**Response 201:**
```json
{
  "data": {
    "id": "market-uuid-1",
    "type": "campaign_market",
    "attributes": {
      "unique_id": "market-uuid-1",
      "campaign_id": "campaign-uuid-123",
      "name": "North America",
      "market_code": "NA",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /campaigns/:unique_id/campaign_markets/:market_id - Remove Campaign Market

Removes a market segment from a campaign.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_markets/market-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Campaign Locations

### GET /campaigns/:unique_id/campaign_locations - List Campaign Locations

Lists geographic locations assigned to a campaign.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_locations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "location-uuid-1",
      "type": "campaign_location",
      "attributes": {
        "unique_id": "location-uuid-1",
        "campaign_id": "campaign-uuid-123",
        "name": "New York",
        "location_type": "city",
        "country_code": "US",
        "region": "NY",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /campaigns/:unique_id/campaign_locations - Add Campaign Location

Adds a geographic location to a campaign.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_locations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_location": {
      "name": "New York",
      "location_type": "city",
      "country_code": "US",
      "region": "NY"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Location name |
| `location_type` | string | Yes | Type (city, state, country, region) |
| `country_code` | string | Yes | ISO country code |
| `region` | string | No | Region or state code |

**Response 201:**
```json
{
  "data": {
    "id": "location-uuid-1",
    "type": "campaign_location",
    "attributes": {
      "unique_id": "location-uuid-1",
      "campaign_id": "campaign-uuid-123",
      "name": "New York",
      "location_type": "city",
      "country_code": "US",
      "region": "NY",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /campaigns/:unique_id/campaign_locations/:location_id - Remove Campaign Location

Removes a location from a campaign.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_locations/location-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Campaign Targets

### GET /campaigns/:unique_id/campaign_targets - List Campaign Targets

Lists targeting rules for a campaign.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_targets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "target-uuid-1",
      "type": "campaign_target",
      "attributes": {
        "unique_id": "target-uuid-1",
        "campaign_id": "campaign-uuid-123",
        "name": "Young Professionals",
        "target_type": "demographic",
        "criteria": {
          "age_min": 25,
          "age_max": 35,
          "interests": ["technology", "fitness"]
        },
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /campaigns/:unique_id/campaign_targets - Add Campaign Target

Adds a targeting rule to a campaign.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_targets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_target": {
      "name": "Young Professionals",
      "target_type": "demographic",
      "criteria": {
        "age_min": 25,
        "age_max": 35,
        "interests": ["technology", "fitness"]
      }
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Target name |
| `target_type` | string | Yes | Target type (demographic, behavioral, geographic) |
| `criteria` | object | No | Targeting criteria |

**Response 201:**
```json
{
  "data": {
    "id": "target-uuid-1",
    "type": "campaign_target",
    "attributes": {
      "unique_id": "target-uuid-1",
      "campaign_id": "campaign-uuid-123",
      "name": "Young Professionals",
      "target_type": "demographic",
      "criteria": {
        "age_min": 25,
        "age_max": 35,
        "interests": ["technology", "fitness"]
      },
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /campaigns/:unique_id/campaign_targets/:target_id - Remove Campaign Target

Removes a targeting rule from a campaign.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/campaigns/campaign-uuid-123/campaign_targets/target-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

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

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Campaign Not Found",
    "detail": "The requested campaign could not be found."
  }]
}
```
