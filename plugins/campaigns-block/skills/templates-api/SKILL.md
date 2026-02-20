---
name: campaigns-templates-api
description: Manage 23blocks campaign templates via REST API. Use when creating reusable templates, managing template details and sections, or configuring template variables for campaign content.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Templates API

Complete API reference for 23blocks campaign template management with template details and variable substitution.

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
curl -X GET "$BLOCKS_API_URL/templates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Template Endpoints

### GET /templates - List Templates

Lists all templates with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/templates?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `template_type` | string | No | Filter by template type |
| `status` | string | No | Filter by status (active, inactive) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "template-uuid-123",
      "type": "template",
      "attributes": {
        "unique_id": "template-uuid-123",
        "name": "Promo Landing Template",
        "template_type": "landing_page",
        "content": "<html>{{hero_section}}{{product_grid}}{{cta_section}}</html>",
        "variables": ["hero_section", "product_grid", "cta_section"],
        "status": "active",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 18
  }
}
```

---

### GET /templates/:unique_id - Get Template

Retrieves a single template by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/templates/template-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "name": "Promo Landing Template",
      "template_type": "landing_page",
      "content": "<html>{{hero_section}}{{product_grid}}{{cta_section}}</html>",
      "variables": ["hero_section", "product_grid", "cta_section"],
      "status": "active",
      "created_at": "2026-01-10T10:30:00Z",
      "updated_at": "2026-01-10T10:30:00Z"
    },
    "relationships": {
      "template_details": {
        "data": [
          { "id": "detail-uuid-1", "type": "template_detail" },
          { "id": "detail-uuid-2", "type": "template_detail" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Template not found

---

### POST /templates - Create Template

Creates a new template.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/templates" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "name": "Promo Landing Template",
      "template_type": "landing_page",
      "content": "<html>{{hero_section}}{{product_grid}}{{cta_section}}</html>",
      "variables": ["hero_section", "product_grid", "cta_section"],
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Template name |
| `template_type` | string | Yes | Template type (landing_page, email, banner, social) |
| `content` | string | Yes | Template content with variable placeholders |
| `variables` | array | No | List of template variable names |
| `status` | enum | No | active, inactive (default: active) |

**Response 201:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "name": "Promo Landing Template",
      "template_type": "landing_page",
      "content": "<html>{{hero_section}}{{product_grid}}{{cta_section}}</html>",
      "variables": ["hero_section", "product_grid", "cta_section"],
      "status": "active",
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /templates/:unique_id - Update Template

Updates an existing template.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/templates/template-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "name": "Promo Landing Template v2",
      "content": "<html>{{hero}}{{products}}{{testimonials}}{{cta}}</html>",
      "variables": ["hero", "products", "testimonials", "cta"]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "template-uuid-123",
    "type": "template",
    "attributes": {
      "unique_id": "template-uuid-123",
      "name": "Promo Landing Template v2",
      "content": "<html>{{hero}}{{products}}{{testimonials}}{{cta}}</html>",
      "variables": ["hero", "products", "testimonials", "cta"],
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Template not found

---

### DELETE /templates/:unique_id - Delete Template

Deletes a template.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/templates/template-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Template not found

---

## Template Detail Endpoints

### GET /template_details - List Template Details

Lists all template details with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/template_details?template_id=template-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |
| `template_id` | uuid | No | Filter by parent template |

**Response 200:**
```json
{
  "data": [
    {
      "id": "detail-uuid-1",
      "type": "template_detail",
      "attributes": {
        "unique_id": "detail-uuid-1",
        "template_id": "template-uuid-123",
        "section_name": "hero_section",
        "content": "<div class=\"hero\">{{headline}}{{subheadline}}</div>",
        "position": 1,
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

### POST /template_details - Create Template Detail

Creates a new template detail section.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/template_details" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template_detail": {
      "template_id": "template-uuid-123",
      "section_name": "hero_section",
      "content": "<div class=\"hero\">{{headline}}{{subheadline}}</div>",
      "position": 1
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `template_id` | uuid | Yes | Parent template ID |
| `section_name` | string | Yes | Section identifier name |
| `content` | string | Yes | Section content with variable placeholders |
| `position` | integer | No | Display order position |

**Response 201:**
```json
{
  "data": {
    "id": "detail-uuid-1",
    "type": "template_detail",
    "attributes": {
      "unique_id": "detail-uuid-1",
      "template_id": "template-uuid-123",
      "section_name": "hero_section",
      "content": "<div class=\"hero\">{{headline}}{{subheadline}}</div>",
      "position": 1,
      "created_at": "2026-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Parent template not found
- `422 Unprocessable Entity` - Validation errors

---

### PUT /template_details/:unique_id - Update Template Detail

Updates an existing template detail.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/template_details/detail-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template_detail": {
      "content": "<div class=\"hero-v2\">{{headline}}{{subheadline}}{{cta_button}}</div>",
      "position": 1
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "detail-uuid-1",
    "type": "template_detail",
    "attributes": {
      "unique_id": "detail-uuid-1",
      "content": "<div class=\"hero-v2\">{{headline}}{{subheadline}}{{cta_button}}</div>",
      "position": 1,
      "updated_at": "2026-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Template detail not found

---

### DELETE /template_details/:unique_id - Delete Template Detail

Deletes a template detail.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/template_details/detail-uuid-1" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Template detail not found

---

## Data Models

### Template
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Template name |
| `template_type` | string | landing_page, email, banner, social |
| `content` | string | Template content with variable placeholders |
| `variables` | array | List of variable names used in content |
| `status` | enum | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### TemplateDetail
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `template_id` | uuid | Parent template ID |
| `section_name` | string | Section identifier name |
| `content` | string | Section content with variable placeholders |
| `position` | integer | Display order position |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Template Not Found",
    "detail": "The requested template could not be found."
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
// CampaignTemplates — client.campaigns.campaignTemplates
client.campaigns.campaignTemplates.list(params?: ListCampaignTemplatesParams): Promise<PageResult<CampaignTemplate>>;
client.campaigns.campaignTemplates.get(uniqueId: string): Promise<CampaignTemplate>;
client.campaigns.campaignTemplates.create(data: CreateCampaignTemplateRequest): Promise<CampaignTemplate>;
client.campaigns.campaignTemplates.update(uniqueId: string, data: UpdateCampaignTemplateRequest): Promise<CampaignTemplate>;
client.campaigns.campaignTemplates.delete(uniqueId: string): Promise<void>;
client.campaigns.campaignTemplates.listDetails(params?: ListTemplateDetailsParams): Promise<PageResult<TemplateDetail>>;
client.campaigns.campaignTemplates.getDetail(uniqueId: string): Promise<TemplateDetail>;
client.campaigns.campaignTemplates.createDetail(data: CreateTemplateDetailRequest): Promise<TemplateDetail>;
client.campaigns.campaignTemplates.updateDetail(uniqueId: string, data: UpdateTemplateDetailRequest): Promise<TemplateDetail>;
client.campaigns.campaignTemplates.deleteDetail(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  CampaignTemplate,
  CreateCampaignTemplateRequest,
  UpdateCampaignTemplateRequest,
  ListCampaignTemplatesParams,
  TemplateDetail,
  CreateTemplateDetailRequest,
  UpdateTemplateDetailRequest,
  ListTemplateDetailsParams,
} from '@23blocks/block-campaigns';
```

### React Hook

```typescript
import { useCampaignsBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useCampaignsBlock();
  const result = await client.campaigns.campaignTemplates.list();
}
```
