# Campaigns API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

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
