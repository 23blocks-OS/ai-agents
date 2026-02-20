---
name: campaigns-landing-pages-api
description: Manage 23blocks campaign landing pages via REST API. Use when creating landing pages with URL slugs, defining target audiences, creating Facebook audiences for social targeting, or managing landing page templates.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Landing Pages API

Complete API reference for 23blocks landing page management with audience targeting, Facebook audience integration, and landing templates.

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
curl -X GET "$BLOCKS_API_URL/landing_pages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Landing Page Endpoints

### GET /landing_pages - List Landing Pages

Lists all landing pages with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/landing_pages?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `campaign_id` | uuid | No | Filter by campaign |
| `status` | string | No | Filter by status (draft, active, inactive) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "lp-uuid-123",
      "type": "landing_page",
      "attributes": {
        "unique_id": "lp-uuid-123",
        "name": "Summer Sale Landing",
        "url_slug": "summer-sale-2026",
        "template_id": "template-uuid-123",
        "audience_id": "audience-uuid-123",
        "campaign_id": "campaign-uuid-123",
        "conversion_rate": 0.045,
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 15
  }
}
```

---

### GET /landing_pages/:unique_id - Get Landing Page

Retrieves a single landing page by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/landing_pages/lp-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "lp-uuid-123",
    "type": "landing_page",
    "attributes": {
      "unique_id": "lp-uuid-123",
      "name": "Summer Sale Landing",
      "url_slug": "summer-sale-2026",
      "template_id": "template-uuid-123",
      "audience_id": "audience-uuid-123",
      "campaign_id": "campaign-uuid-123",
      "conversion_rate": 0.045,
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "template": {
        "data": { "id": "template-uuid-123", "type": "template" }
      },
      "audience": {
        "data": { "id": "audience-uuid-123", "type": "landing_audience" }
      },
      "campaign": {
        "data": { "id": "campaign-uuid-123", "type": "campaign" }
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Landing page not found

---

### POST /landing_pages - Create Landing Page

Creates a new landing page.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/landing_pages" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "landing_page": {
      "name": "Summer Sale Landing",
      "url_slug": "summer-sale-2026",
      "campaign_id": "campaign-uuid-123",
      "template_id": "template-uuid-123",
      "status": "draft"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Landing page name |
| `url_slug` | string | Yes | URL-friendly slug (must be unique) |
| `campaign_id` | uuid | No | Associated campaign ID |
| `template_id` | uuid | No | Template to use for rendering |
| `audience_id` | uuid | No | Target audience ID |
| `status` | enum | No | draft, active, inactive (default: draft) |

**Response 201:**
```json
{
  "data": {
    "id": "lp-uuid-123",
    "type": "landing_page",
    "attributes": {
      "unique_id": "lp-uuid-123",
      "name": "Summer Sale Landing",
      "url_slug": "summer-sale-2026",
      "campaign_id": "campaign-uuid-123",
      "template_id": "template-uuid-123",
      "conversion_rate": 0.0,
      "status": "draft",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - URL slug already exists
- `422 Unprocessable Entity` - Validation errors

---

### PUT /landing_pages/:unique_id - Update Landing Page

Updates an existing landing page.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/landing_pages/lp-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "landing_page": {
      "name": "Summer Sale Landing - Updated",
      "audience_id": "audience-uuid-456",
      "status": "active"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "lp-uuid-123",
    "type": "landing_page",
    "attributes": {
      "unique_id": "lp-uuid-123",
      "name": "Summer Sale Landing - Updated",
      "audience_id": "audience-uuid-456",
      "status": "active",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Landing page not found

---

### DELETE /landing_pages/:unique_id - Delete Landing Page

Deletes a landing page.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/landing_pages/lp-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Landing page not found

---

## Landing Audience Endpoints

### GET /landing_audiences - List Audiences

Lists all landing audiences with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/landing_audiences?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "audience-uuid-123",
      "type": "landing_audience",
      "attributes": {
        "unique_id": "audience-uuid-123",
        "name": "Young Professionals 25-35",
        "description": "Target demographic for summer promotions",
        "landing_page_id": "lp-uuid-123",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
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

### POST /landing_audiences - Create Audience

Creates a new landing audience.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/landing_audiences" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "landing_audience": {
      "name": "Young Professionals 25-35",
      "description": "Target demographic for summer promotions",
      "landing_page_id": "lp-uuid-123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Audience name |
| `description` | string | No | Audience description |
| `landing_page_id` | uuid | No | Associated landing page ID |

**Response 201:**
```json
{
  "data": {
    "id": "audience-uuid-123",
    "type": "landing_audience",
    "attributes": {
      "unique_id": "audience-uuid-123",
      "name": "Young Professionals 25-35",
      "description": "Target demographic for summer promotions",
      "landing_page_id": "lp-uuid-123",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /landing_audiences/:unique_id - Update Audience

Updates an existing audience.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/landing_audiences/audience-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "landing_audience": {
      "name": "Young Professionals 25-40",
      "description": "Updated age range for broader targeting"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "audience-uuid-123",
    "type": "landing_audience",
    "attributes": {
      "unique_id": "audience-uuid-123",
      "name": "Young Professionals 25-40",
      "description": "Updated age range for broader targeting",
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Audience not found

---

### DELETE /landing_audiences/:unique_id - Delete Audience

Deletes an audience.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/landing_audiences/audience-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Audience not found

---

## Facebook Audience

### POST /audience/facebook - Create Facebook Audience

Creates a Facebook-specific audience for social media targeting.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/audience/facebook" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "facebook_audience": {
      "name": "Summer Sale FB Audience",
      "landing_audience_id": "audience-uuid-123",
      "age_min": 25,
      "age_max": 35,
      "genders": ["male", "female"],
      "interests": ["shopping", "fashion", "deals"],
      "locations": ["US", "CA"],
      "languages": ["en"]
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Facebook audience name |
| `landing_audience_id` | uuid | Yes | Parent landing audience ID |
| `age_min` | integer | No | Minimum age (13-65) |
| `age_max` | integer | No | Maximum age (13-65) |
| `genders` | array | No | Target genders |
| `interests` | array | No | Interest keywords |
| `locations` | array | No | Country codes for targeting |
| `languages` | array | No | Language codes |

**Response 201:**
```json
{
  "data": {
    "id": "fb-audience-uuid-123",
    "type": "facebook_audience",
    "attributes": {
      "unique_id": "fb-audience-uuid-123",
      "name": "Summer Sale FB Audience",
      "landing_audience_id": "audience-uuid-123",
      "age_min": 25,
      "age_max": 35,
      "genders": ["male", "female"],
      "interests": ["shopping", "fashion", "deals"],
      "locations": ["US", "CA"],
      "languages": ["en"],
      "estimated_reach": 2500000,
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Landing audience not found
- `422 Unprocessable Entity` - Validation errors

---

## Landing Template Endpoints

### GET /landing_templates - List Landing Templates

Lists all landing page templates.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/landing_templates?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "lt-uuid-123",
      "type": "landing_template",
      "attributes": {
        "unique_id": "lt-uuid-123",
        "name": "Product Showcase",
        "description": "Template for product-focused landing pages",
        "layout": "single_column",
        "preview_url": "https://cdn.example.com/templates/product-showcase-preview.jpg",
        "created_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 6
  }
}
```

---

### POST /landing_templates - Create Landing Template

Creates a new landing page template.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/landing_templates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "landing_template": {
      "name": "Product Showcase",
      "description": "Template for product-focused landing pages",
      "layout": "single_column",
      "content": "<html>{{header}}{{hero}}{{products}}{{footer}}</html>"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Template name |
| `description` | string | No | Template description |
| `layout` | string | No | Layout type (single_column, two_column, hero) |
| `content` | string | Yes | Template HTML content |

**Response 201:**
```json
{
  "data": {
    "id": "lt-uuid-123",
    "type": "landing_template",
    "attributes": {
      "unique_id": "lt-uuid-123",
      "name": "Product Showcase",
      "description": "Template for product-focused landing pages",
      "layout": "single_column",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

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
