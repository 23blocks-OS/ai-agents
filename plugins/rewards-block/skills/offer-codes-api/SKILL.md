---
name: rewards-offer-codes-api
description: Manage 23blocks offer codes via REST API. Use when creating offer codes, sending codes via email, generating bulk codes, or distributing codes through API-to-API integrations.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Offer Codes API

Complete API reference for 23blocks offer code management, distribution, and API-to-API delivery.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://rewards.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Rewards API base URL | `https://rewards.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/offer_codes/WELCOME50" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /offer_codes/:code - Get Offer Code

Retrieves an offer code by its code value.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/offer_codes/WELCOME50" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "offer-uuid-123",
    "type": "offer_code",
    "attributes": {
      "unique_id": "offer-uuid-123",
      "code": "WELCOME50",
      "offer_type": "discount",
      "discount_value": 50,
      "max_uses": 1000,
      "used_count": 245,
      "valid_from": "2025-01-01T00:00:00Z",
      "valid_to": "2026-12-31T23:59:59Z",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Offer code not found

---

### POST /offer_codes/ - Create Offer Code

Creates a new offer code.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/offer_codes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "offer_code": {
      "code": "WELCOME50",
      "offer_type": "discount",
      "discount_value": 50,
      "max_uses": 1000,
      "valid_from": "2025-01-01T00:00:00Z",
      "valid_to": "2026-12-31T23:59:59Z"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Unique offer code |
| `offer_type` | string | Yes | Type of offer (discount, points, freebie) |
| `discount_value` | decimal | Yes | Discount or points value |
| `max_uses` | integer | No | Maximum total uses |
| `valid_from` | timestamp | No | Start of validity period |
| `valid_to` | timestamp | No | End of validity period |

**Response 201:**
```json
{
  "data": {
    "id": "new-offer-uuid",
    "type": "offer_code",
    "attributes": {
      "unique_id": "new-offer-uuid",
      "code": "WELCOME50",
      "offer_type": "discount",
      "discount_value": 50,
      "max_uses": 1000,
      "used_count": 0,
      "valid_from": "2025-01-01T00:00:00Z",
      "valid_to": "2026-12-31T23:59:59Z",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `409 Conflict` - Offer code already exists
- `422 Unprocessable Entity` - Validation errors

---

### POST /offer_codes/send - Send Offer Code

Sends an offer code to a recipient via email.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/offer_codes/send" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "WELCOME50",
    "recipient_email": "jane@example.com",
    "recipient_name": "Jane Smith",
    "message": "Welcome! Use this code for your first purchase."
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Offer code to send |
| `recipient_email` | string | Yes | Recipient email address |
| `recipient_name` | string | No | Recipient name |
| `message` | string | No | Custom message to include |

**Response 200:**
```json
{
  "data": {
    "type": "offer_code_delivery",
    "attributes": {
      "code": "WELCOME50",
      "recipient_email": "jane@example.com",
      "delivery_status": "sent",
      "sent_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Offer code not found
- `422 Unprocessable Entity` - Invalid email or offer code expired

---

### POST /codes/ - Generate Codes

Generates multiple offer codes in bulk.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/codes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "quantity": 100,
    "prefix": "SPRING",
    "offer_type": "discount",
    "discount_value": 15,
    "max_uses": 1,
    "valid_from": "2026-03-01T00:00:00Z",
    "valid_to": "2026-05-31T23:59:59Z"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `quantity` | integer | Yes | Number of codes to generate |
| `prefix` | string | No | Code prefix (e.g., SPRING-XXXX) |
| `offer_type` | string | Yes | Type of offer |
| `discount_value` | decimal | Yes | Discount or points value |
| `max_uses` | integer | No | Max uses per code (default: 1) |
| `valid_from` | timestamp | No | Start of validity |
| `valid_to` | timestamp | No | End of validity |

**Response 201:**
```json
{
  "data": {
    "type": "code_generation",
    "attributes": {
      "generated_count": 100,
      "prefix": "SPRING",
      "offer_type": "discount",
      "discount_value": 15,
      "sample_codes": ["SPRING-A1B2C3", "SPRING-D4E5F6", "SPRING-G7H8I9"],
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /:url_id/offer_codes/send - API-to-API Send

Sends an offer code through an API-to-API integration between tenants.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/acme-corp/offer_codes/send" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "PARTNER-DEAL-25",
    "recipient_email": "jane@example.com",
    "source_tenant": "partner-company"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Offer code to send |
| `recipient_email` | string | Yes | Recipient email |
| `source_tenant` | string | No | Source tenant identifier |

**Response 200:**
```json
{
  "data": {
    "type": "offer_code_delivery",
    "attributes": {
      "code": "PARTNER-DEAL-25",
      "recipient_email": "jane@example.com",
      "target_tenant": "acme-corp",
      "delivery_status": "sent",
      "sent_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `403 Forbidden` - Not authorized for cross-tenant delivery
- `404 Not Found` - Tenant or offer code not found

---

## Data Models

### OfferCode
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `code` | string | Unique offer code |
| `offer_type` | string | discount, points, freebie |
| `discount_value` | decimal | Discount or points value |
| `max_uses` | integer | Maximum total uses |
| `used_count` | integer | Number of times used |
| `valid_from` | timestamp | Start of validity |
| `valid_to` | timestamp | End of validity |
| `status` | enum | active, expired, depleted |
| `created_at` | timestamp | Creation time |

### OfferCodeDelivery
| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Offer code sent |
| `recipient_email` | string | Recipient email address |
| `delivery_status` | string | sent, failed |
| `sent_at` | timestamp | Delivery time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "409",
    "code": "duplicate_code",
    "title": "Offer Code Already Exists",
    "detail": "An offer code with this code value already exists."
  }]
}
```
