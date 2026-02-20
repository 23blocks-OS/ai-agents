---
name: assets-reports-api
description: Generate 23blocks asset reports via REST API. Use when creating reports for events lists, events summaries, and operations summaries across assets.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Reports API

Complete API reference for 23blocks Assets Block report generation.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://assets.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Assets API base URL | `https://assets.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X POST "$BLOCKS_API_URL/reports/events/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### POST /reports/events/list - List Events Report

Generates a detailed events report across assets with filtering.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/events/list" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-03-31",
      "event_types": ["inspection", "repair"],
      "asset_unique_ids": [],
      "page": 1,
      "records": 50
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | date | No | Report start date (YYYY-MM-DD) |
| `end_date` | date | No | Report end date (YYYY-MM-DD) |
| `event_types` | array | No | Filter by event types |
| `asset_unique_ids` | array | No | Filter by specific asset IDs (empty for all) |
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 50) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "event-uuid-001",
      "type": "report_event",
      "attributes": {
        "event_unique_id": "event-uuid-001",
        "asset_unique_id": "asset-uuid-123",
        "asset_name": "Laptop Dell XPS 15",
        "event_type": "inspection",
        "description": "Quarterly hardware inspection",
        "occurred_at": "2025-03-15T10:00:00Z",
        "recorded_by_name": "John Doe"
      }
    },
    {
      "id": "event-uuid-002",
      "type": "report_event",
      "attributes": {
        "event_unique_id": "event-uuid-002",
        "asset_unique_id": "asset-uuid-456",
        "asset_name": "Monitor Samsung 27",
        "event_type": "repair",
        "description": "Replaced power cable",
        "occurred_at": "2025-02-20T14:00:00Z",
        "recorded_by_name": "Jane Smith"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 128,
    "start_date": "2025-01-01",
    "end_date": "2025-03-31"
  }
}
```

---

### POST /reports/events/summary - Events Summary

Generates an aggregated summary of events across assets.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/events/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-03-31",
      "group_by": "event_type"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | date | No | Report start date (YYYY-MM-DD) |
| `end_date` | date | No | Report end date (YYYY-MM-DD) |
| `group_by` | string | No | Grouping dimension: event_type, asset, month (default: event_type) |

**Response 200:**
```json
{
  "data": {
    "type": "events_summary",
    "attributes": {
      "total_events": 128,
      "date_range": {
        "start_date": "2025-01-01",
        "end_date": "2025-03-31"
      },
      "by_type": [
        {
          "event_type": "inspection",
          "count": 45,
          "percentage": 35.2
        },
        {
          "event_type": "repair",
          "count": 32,
          "percentage": 25.0
        },
        {
          "event_type": "relocation",
          "count": 28,
          "percentage": 21.9
        },
        {
          "event_type": "incident",
          "count": 23,
          "percentage": 17.9
        }
      ],
      "top_assets": [
        {
          "asset_unique_id": "asset-uuid-123",
          "asset_name": "Laptop Dell XPS 15",
          "events_count": 12
        },
        {
          "asset_unique_id": "asset-uuid-456",
          "asset_name": "Server Rack A1",
          "events_count": 9
        }
      ],
      "monthly_trend": [
        { "month": "2025-01", "count": 38 },
        { "month": "2025-02", "count": 42 },
        { "month": "2025-03", "count": 48 }
      ]
    }
  }
}
```

---

### POST /reports/operations/summary - Operations Summary

Generates an aggregated summary of operations across assets.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/reports/operations/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "report": {
      "start_date": "2025-01-01",
      "end_date": "2025-03-31",
      "group_by": "operation_type"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_date` | date | No | Report start date (YYYY-MM-DD) |
| `end_date` | date | No | Report end date (YYYY-MM-DD) |
| `group_by` | string | No | Grouping dimension: operation_type, asset, month (default: operation_type) |

**Response 200:**
```json
{
  "data": {
    "type": "operations_summary",
    "attributes": {
      "total_operations": 256,
      "date_range": {
        "start_date": "2025-01-01",
        "end_date": "2025-03-31"
      },
      "by_type": [
        {
          "operation_type": "checkout",
          "count": 89,
          "percentage": 34.8
        },
        {
          "operation_type": "checkin",
          "count": 85,
          "percentage": 33.2
        },
        {
          "operation_type": "transfer",
          "count": 42,
          "percentage": 16.4
        },
        {
          "operation_type": "deployment",
          "count": 25,
          "percentage": 9.8
        },
        {
          "operation_type": "decommission",
          "count": 15,
          "percentage": 5.8
        }
      ],
      "top_assets": [
        {
          "asset_unique_id": "asset-uuid-123",
          "asset_name": "Laptop Dell XPS 15",
          "operations_count": 18
        },
        {
          "asset_unique_id": "asset-uuid-789",
          "asset_name": "Projector Epson EB",
          "operations_count": 14
        }
      ],
      "monthly_trend": [
        { "month": "2025-01", "count": 78 },
        { "month": "2025-02", "count": 85 },
        { "month": "2025-03", "count": 93 }
      ]
    }
  }
}
```

---

## Data Models

### EventsReport
| Field | Type | Description |
|-------|------|-------------|
| `total_events` | integer | Total events in date range |
| `date_range` | object | Start and end dates |
| `by_type` | array | Breakdown by event type with counts and percentages |
| `top_assets` | array | Assets with most events |
| `monthly_trend` | array | Monthly event counts |

### OperationsReport
| Field | Type | Description |
|-------|------|-------------|
| `total_operations` | integer | Total operations in date range |
| `date_range` | object | Start and end dates |
| `by_type` | array | Breakdown by operation type with counts and percentages |
| `top_assets` | array | Assets with most operations |
| `monthly_trend` | array | Monthly operation counts |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "validation_error",
    "title": "Validation Error",
    "detail": "start_date must be before end_date."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `422` | Unprocessable Entity - Invalid date range or parameters |
