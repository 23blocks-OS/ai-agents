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

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Sales API base URL | `https://sales.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token (human or AID) | `eyJhbGciOiJSUzI1NiJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication

Two methods are supported. The Bearer token works the same either way.

**Method 1: Agent Identity (AID)** -- For AI agents with AMP identity:
```bash
export BLOCKS_AUTH_TOKEN=$(aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q)
export BLOCKS_API_URL="https://sales.api.us.23blocks.com"
export BLOCKS_API_KEY="<your-api-key>"
```
> First time? See the `23blocks-auth-agent-identity-api` skill for setup.

**Method 2: User Token** -- For human-provided credentials:
```bash
export BLOCKS_API_URL="https://sales.api.us.23blocks.com"
export BLOCKS_AUTH_TOKEN="<your-bearer-token>"
export BLOCKS_API_KEY="<your-api-key>"
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
