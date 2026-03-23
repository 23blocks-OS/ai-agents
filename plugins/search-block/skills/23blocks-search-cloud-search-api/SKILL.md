---
name: 23blocks-search-cloud-search-api
description: Execute 23blocks AWS CloudSearch queries via REST API. Use when performing structured search queries on AWS CloudSearch with include/exclude filters.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# AWS CloudSearch API

Complete API reference for 23blocks AWS CloudSearch integration.

## Required Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Search API base URL | `https://search.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://search.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://search.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
```

---

## Endpoints

### POST /cloud - Execute CloudSearch Query

Executes a structured search query on the configured AWS CloudSearch domain using include/exclude filters.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/cloud" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "category": "electronics",
      "status": "active"
    },
    "exclude": {
      "brand": "generic"
    },
    "page": 1,
    "records": 20
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `include` | object | No | Fields and values to include in results |
| `exclude` | object | No | Fields and values to exclude from results |
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "result-uuid-123",
      "type": "cloud_search_result",
      "attributes": {
        "entity_unique_id": "entity-uuid-123",
        "entity_type": "product",
        "entity_alias": "Widget Pro",
        "content": {
          "name": "Widget Pro",
          "category": "electronics",
          "status": "active"
        }
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 45,
    "elapsed_time": 0.120
  }
}
```

**Errors:**
- `400 Bad Request` - Invalid query structure
- `422 Unprocessable Entity` - CloudSearch domain not configured

---

## Filter Examples

### Include Only Active Products
```bash
curl -X POST "$BLOCKS_API_URL/cloud" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "entity_type": "product",
      "status": "active"
    }
  }'
```

### Exclude Specific Categories
```bash
curl -X POST "$BLOCKS_API_URL/cloud" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "include": {
      "entity_type": "article"
    },
    "exclude": {
      "category": "archived"
    }
  }'
```

---

## Data Models

### CloudSearchIndex
| Field | Type | Description |
|-------|------|-------------|
| `url` | string | AWS CloudSearch domain endpoint |
| `api_access_key` | string | AWS access key for CloudSearch |
| `status` | string | Index status (active, inactive) |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "configuration_error",
    "title": "CloudSearch Not Configured",
    "detail": "AWS CloudSearch domain is not configured for this tenant."
  }]
}
```
