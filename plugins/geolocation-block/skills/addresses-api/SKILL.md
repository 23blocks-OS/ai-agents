---
name: geolocation-addresses-api
description: Manage 23blocks addresses via REST API. Use when creating, updating, or deleting addresses, managing user and contact addresses, setting default addresses, or organizing addresses with tags.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Addresses API

Complete API reference for 23blocks address management with user/contact associations, default address support, and tags.

## Required Environment Variables

**BEFORE making ANY API call**, verify these environment variables are set:

```bash
# Pre-flight check - Run this FIRST
if [ -z "$BLOCKS_API_URL" ] || [ -z "$BLOCKS_AUTH_TOKEN" ] || [ -z "$BLOCKS_API_KEY" ]; then
  echo "ERROR: Missing required environment variables"
  echo "Please set:"
  echo "  BLOCKS_API_URL     - API base URL (e.g., https://geolocation.api.us.23blocks.com)"
  echo "  BLOCKS_AUTH_TOKEN  - Your authentication token"
  echo "  BLOCKS_API_KEY     - Your API key (AppId)"
  exit 1
fi
```

| Variable | Description | Example |
|----------|-------------|---------|
| `BLOCKS_API_URL` | Geolocation API base URL | `https://geolocation.api.us.23blocks.com` |
| `BLOCKS_AUTH_TOKEN` | Bearer token | `eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...` |
| `BLOCKS_API_KEY` | API key (AppId) | `pk_live_sh_f2b5ab3c7203d29b6d2937e2` |

## Authentication
```bash
curl -X GET "$BLOCKS_API_URL/addresses/addr-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /addresses/:unique_id/ - Get Address

Retrieves a single address by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/addresses/addr-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "latitude": 33.9425,
      "longitude": -118.2551,
      "address_type": "home",
      "is_default": true,
      "owner_type": "user",
      "owner_id": "user-uuid-123",
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "tags": {
        "data": [
          { "id": "tag-uuid", "type": "tag" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Address not found

---

### GET /addresses/owned_by/:unique_id - Get Addresses by Owner

Retrieves all addresses owned by a specific entity.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/addresses/owned_by/user-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "addr-uuid-123",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-123",
        "street": "456 Oak Ave",
        "city": "Los Angeles",
        "state": "CA",
        "zip_code": "90001",
        "country": "US",
        "address_type": "home",
        "is_default": true,
        "owner_type": "user",
        "owner_id": "user-uuid-123"
      }
    }
  ]
}
```

---

### POST /addresses/ - Create Address

Creates a new address.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/addresses/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "address": {
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "latitude": 33.9425,
      "longitude": -118.2551,
      "address_type": "home",
      "is_default": true,
      "owner_type": "user",
      "owner_id": "user-uuid-123"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `street` | string | Yes | Street address |
| `city` | string | Yes | City name |
| `state` | string | Yes | State/province code |
| `zip_code` | string | Yes | Postal/zip code |
| `country` | string | Yes | Country code (ISO 3166) |
| `latitude` | float | No | Latitude coordinate |
| `longitude` | float | No | Longitude coordinate |
| `address_type` | string | No | Type (home, work, billing, shipping) |
| `is_default` | boolean | No | Whether this is the default address |
| `owner_type` | string | Yes | Owner entity type (user, contact) |
| `owner_id` | uuid | Yes | Owner entity unique ID |

**Response 201:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "latitude": 33.9425,
      "longitude": -118.2551,
      "address_type": "home",
      "is_default": true,
      "owner_type": "user",
      "owner_id": "user-uuid-123",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /addresses/:unique_id - Update Address

Updates an existing address.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/addresses/addr-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "address": {
      "street": "789 Pine St",
      "zip_code": "90002",
      "is_default": false
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "789 Pine St",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90002",
      "country": "US",
      "is_default": false,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Address not found

---

### DELETE /addresses/:unique_id - Delete Address

Deletes an address.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/addresses/addr-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Address not found

---

## Address Tags

### POST /addresses/:unique_id/tags/ - Add Tag

Adds a tag to an address.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/addresses/addr-uuid-123/tags/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag_unique_id": "tag-uuid-456"
  }'
```

**Response 200:**
```json
{
  "message": "Tag added successfully"
}
```

---

### DELETE /addresses/:unique_id/tags/:tag_unique_id - Remove Tag

Removes a tag from an address.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/addresses/addr-uuid-123/tags/tag-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Tag removed successfully"
}
```

---

## User Addresses

### GET /users/:unique_id/addresses - User Addresses

Retrieves all addresses for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/addresses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "addr-uuid-123",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-123",
        "street": "456 Oak Ave",
        "city": "Los Angeles",
        "state": "CA",
        "zip_code": "90001",
        "country": "US",
        "address_type": "home",
        "is_default": true
      }
    },
    {
      "id": "addr-uuid-456",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-456",
        "street": "100 Corporate Blvd",
        "city": "Los Angeles",
        "state": "CA",
        "zip_code": "90010",
        "country": "US",
        "address_type": "work",
        "is_default": false
      }
    }
  ]
}
```

---

### GET /users/:unique_id/default - User Default Address

Retrieves the default address for a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-123/default" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "addr-uuid-123",
    "type": "address",
    "attributes": {
      "unique_id": "addr-uuid-123",
      "street": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zip_code": "90001",
      "country": "US",
      "address_type": "home",
      "is_default": true
    }
  }
}
```

**Errors:**
- `404 Not Found` - No default address set

---

## Contact Addresses

### GET /contacts/:unique_id/addresses - Contact Addresses

Retrieves all addresses for a contact.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/contacts/contact-uuid-789/addresses" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "addr-uuid-789",
      "type": "address",
      "attributes": {
        "unique_id": "addr-uuid-789",
        "street": "200 Business Park Dr",
        "city": "San Francisco",
        "state": "CA",
        "zip_code": "94105",
        "country": "US",
        "address_type": "work",
        "is_default": true
      }
    }
  ]
}
```

---

## Data Models

### Address
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `street` | string | Street address |
| `city` | string | City name |
| `state` | string | State/province code |
| `zip_code` | string | Postal/zip code |
| `country` | string | Country code (ISO 3166) |
| `latitude` | float | Latitude coordinate |
| `longitude` | float | Longitude coordinate |
| `address_type` | string | Type (home, work, billing, shipping) |
| `is_default` | boolean | Whether this is the default address |
| `owner_type` | string | Owner entity type (user, contact) |
| `owner_id` | uuid | Owner entity unique ID |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Address Not Found",
    "detail": "The requested address could not be found."
  }]
}
```
