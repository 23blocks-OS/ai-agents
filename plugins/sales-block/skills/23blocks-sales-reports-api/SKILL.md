---
name: 23blocks-sales-reports-api
description: Generate 23blocks Sales reports via REST API. Use when generating order summaries, payment reports, subscription analytics, vendor payment reports, provider reports, or flexible order summaries.
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Reports API

Complete API reference for 23blocks sales reporting including orders, payments, subscriptions, vendors, and providers.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X POST "$BLOCKS_API_URL/reports/orders/summary" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/reports/orders/summary` | Order summary report |
| POST | `/reports/orders/list` | Order list report |
| POST | `/reports/orders/providers/list` | Provider list report |
| POST | `/reports/orders/providers/summary` | Provider summary report |
| POST | `/reports/flexible_orders/summary` | Flexible order summary report |
| POST | `/reports/payments/list` | Payment list report |
| POST | `/reports/payments/summary` | Payment summary report |
| POST | `/reports/vendors/payments/list` | Vendor payment list report |
| POST | `/reports/vendors/payments/summary` | Vendor payment summary report |
| POST | `/reports/users/subscriptions/list` | Subscription list report |
| POST | `/reports/users/subscriptions/summary` | Subscription summary report |

All report endpoints accept: `start_date`, `end_date`, `group_by` (day/week/month), `status`, `currency`.

---

## Error Response Format

```json
{
  "errors": [{
    "status": "422",
    "code": "invalid_parameters",
    "title": "Invalid Report Parameters",
    "detail": "start_date must be before end_date."
  }]
}
```
