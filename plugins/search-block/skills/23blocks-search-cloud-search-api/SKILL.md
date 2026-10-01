---
name: 23blocks-search-cloud-search-api
description: "Search Block AWS CloudSearch with include/exclude filters. Use for CloudSearch-backed queries."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# AWS CloudSearch API

Complete API reference for 23blocks AWS CloudSearch integration.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://search.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

### POST /cloud - Execute CloudSearch Query

Executes a structured search query on the configured AWS CloudSearch domain using include/exclude filters.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/cloud" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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
  -H "X-API-KEY: $BLOCKS_API_KEY" \
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

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"configuration_error","title":"CloudSearch Not Configured","detail":"AWS CloudSearch domain is not configured for this tenant."}]}`.
