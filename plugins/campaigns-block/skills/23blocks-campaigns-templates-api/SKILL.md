---
name: 23blocks-campaigns-templates-api
description: Manage 23blocks campaign templates via REST API. Use when creating reusable templates, managing template details and sections, or configuring template variables for campaign content.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Templates API

Complete API reference for 23blocks campaign template management with template details and variable substitution.

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
| GET | `/templates` | List all templates |
| GET | `/templates/:unique_id` | Get a single template |
| POST | `/templates` | Create a template |
| PUT | `/templates/:unique_id` | Update a template |
| DELETE | `/templates/:unique_id` | Delete a template |
| GET | `/template_details` | List template details |
| POST | `/template_details` | Create a template detail section |
| PUT | `/template_details/:unique_id` | Update a template detail |
| DELETE | `/template_details/:unique_id` | Delete a template detail |

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
