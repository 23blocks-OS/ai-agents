---
name: 23blocks-campaigns-templates-api
description: "Campaigns Block content templates: details, sections, variables. Use for reusable campaign content."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Templates API

Complete API reference for 23blocks campaign template management with template details and variable substitution.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://campaigns.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"404","code":"not_found","title":"Template Not Found","detail":"The requested template could not be found."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-campaigns`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCampaignsBlock()` from `@23blocks/react`.

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
