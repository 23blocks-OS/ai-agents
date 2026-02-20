---
name: geolocation-locations-api
description: Manage 23blocks locations via REST API. Use when creating, updating, or deleting locations, managing hours, images, slots, taxes, tags, identities, QR codes, geographic hierarchy lookups, and location groups.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Locations API

Complete API reference for 23blocks location management with hours, images, slots, taxes, tags, identities, QR codes, geographic hierarchy, and location groups.

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
curl -X GET "$BLOCKS_API_URL/locations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## Endpoints

### GET /locations - List Locations

Lists all locations with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `records` | integer | No | Items per page (default: 15) |

**Response 200:**
```json
{
  "data": [
    {
      "id": "loc-uuid-123",
      "type": "location",
      "attributes": {
        "unique_id": "loc-uuid-123",
        "name": "Downtown Office",
        "code": "DT-001",
        "address": "123 Main St, New York, NY 10001",
        "latitude": 40.7128,
        "longitude": -74.0060,
        "phone": "+1-555-0100",
        "email": "downtown@example.com",
        "location_type": "office",
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 5,
    "totalRecords": 72
  }
}
```

---

### GET /locations/:unique_id/ - Get Location

Retrieves a single location by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "loc-uuid-123",
    "type": "location",
    "attributes": {
      "unique_id": "loc-uuid-123",
      "name": "Downtown Office",
      "code": "DT-001",
      "address": "123 Main St, New York, NY 10001",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "phone": "+1-555-0100",
      "email": "downtown@example.com",
      "location_type": "office",
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "hours": {
        "data": [
          { "id": "hour-uuid", "type": "hour" }
        ]
      },
      "tags": {
        "data": [
          { "id": "tag-uuid", "type": "tag" }
        ]
      },
      "images": {
        "data": [
          { "id": "img-uuid", "type": "image" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Location not found

---

### GET /locations/:unique_id/qrcode - Get QR Code

Generates a QR code image for the location.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/qrcode" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "loc-uuid-123",
    "type": "qrcode",
    "attributes": {
      "location_unique_id": "loc-uuid-123",
      "qr_code_url": "https://geolocation.api.us.23blocks.com/qrcodes/loc-uuid-123.png",
      "format": "png"
    }
  }
}
```

---

### POST /locations/ - Create Location

Creates a new location.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location": {
      "name": "Downtown Office",
      "code": "DT-001",
      "address": "123 Main St, New York, NY 10001",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "phone": "+1-555-0100",
      "email": "downtown@example.com",
      "location_type": "office",
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Location name |
| `code` | string | No | Unique location code |
| `address` | string | No | Full address string |
| `latitude` | float | No | Latitude coordinate |
| `longitude` | float | No | Longitude coordinate |
| `phone` | string | No | Contact phone number |
| `email` | string | No | Contact email |
| `location_type` | string | No | Type (office, store, warehouse, etc.) |
| `status` | string | No | Status (active, inactive) |

**Response 201:**
```json
{
  "data": {
    "id": "loc-uuid-123",
    "type": "location",
    "attributes": {
      "unique_id": "loc-uuid-123",
      "name": "Downtown Office",
      "code": "DT-001",
      "address": "123 Main St, New York, NY 10001",
      "latitude": 40.7128,
      "longitude": -74.0060,
      "phone": "+1-555-0100",
      "email": "downtown@example.com",
      "location_type": "office",
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /locations/:unique_id/ - Update Location

Updates an existing location.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location": {
      "name": "Downtown Office - Main",
      "phone": "+1-555-0200"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "loc-uuid-123",
    "type": "location",
    "attributes": {
      "unique_id": "loc-uuid-123",
      "name": "Downtown Office - Main",
      "phone": "+1-555-0200",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Location not found

---

### DELETE /locations/:unique_id/ - Delete Location

Deletes a location.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Location not found

---

### POST /locations/search/code - Search by Code

Searches for a location by its code.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/search/code" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "DT-001"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | Location code to search for |

**Response 200:**
```json
{
  "data": {
    "id": "loc-uuid-123",
    "type": "location",
    "attributes": {
      "unique_id": "loc-uuid-123",
      "name": "Downtown Office",
      "code": "DT-001",
      "address": "123 Main St, New York, NY 10001",
      "status": "active"
    }
  }
}
```

**Errors:**
- `404 Not Found` - No location found with that code

---

## Location Tags

### POST /locations/:unique_id/tags/ - Add Tag

Adds a tag to a location.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/tags/" \
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

### DELETE /locations/:unique_id/tags/:tag_unique_id - Remove Tag

Removes a tag from a location.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/tags/tag-uuid-456" \
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

## Location Hours

### GET /locations/:unique_id/hours/ - List Hours

Lists all operating hours for a location.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/hours/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "hour-uuid-456",
      "type": "hour",
      "attributes": {
        "unique_id": "hour-uuid-456",
        "day_of_week": "monday",
        "open_time": "09:00",
        "close_time": "17:00",
        "is_closed": false,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### GET /locations/:unique_id/hours/:hour_unique_id - Get Hour

Retrieves a specific operating hour entry.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/hours/hour-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "hour-uuid-456",
    "type": "hour",
    "attributes": {
      "unique_id": "hour-uuid-456",
      "day_of_week": "monday",
      "open_time": "09:00",
      "close_time": "17:00",
      "is_closed": false
    }
  }
}
```

---

### POST /locations/:unique_id/hours/ - Create Hour

Creates an operating hour entry for a location.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/hours/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "hour": {
      "day_of_week": "monday",
      "open_time": "09:00",
      "close_time": "17:00",
      "is_closed": false
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `day_of_week` | string | Yes | Day (monday, tuesday, etc.) |
| `open_time` | string | Yes | Opening time (HH:MM) |
| `close_time` | string | Yes | Closing time (HH:MM) |
| `is_closed` | boolean | No | Whether closed on this day |

**Response 201:**
```json
{
  "data": {
    "id": "hour-uuid-456",
    "type": "hour",
    "attributes": {
      "unique_id": "hour-uuid-456",
      "day_of_week": "monday",
      "open_time": "09:00",
      "close_time": "17:00",
      "is_closed": false,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /locations/:unique_id/hours/:hour_unique_id - Update Hour

Updates an existing operating hour entry.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/hours/hour-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "hour": {
      "open_time": "08:00",
      "close_time": "18:00"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "hour-uuid-456",
    "type": "hour",
    "attributes": {
      "unique_id": "hour-uuid-456",
      "day_of_week": "monday",
      "open_time": "08:00",
      "close_time": "18:00",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /locations/:unique_id/hours/:hour_unique_id - Delete Hour

Deletes an operating hour entry.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/hours/hour-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Location Images

### PUT /locations/:unique_id/presign - Presign Upload

Generates a presigned URL for uploading an image to a location.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/presign" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "storefront.jpg",
    "content_type": "image/jpeg"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file_name` | string | Yes | Name of the file |
| `content_type` | string | Yes | MIME type (image/jpeg, image/png) |

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://s3.amazonaws.com/bucket/...",
    "file_key": "locations/loc-uuid-123/storefront.jpg",
    "expires_in": 3600
  }
}
```

---

### POST /locations/:unique_id/images - Upload Image

Registers an uploaded image with a location after presigned upload is complete.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/images" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image": {
      "file_key": "locations/loc-uuid-123/storefront.jpg",
      "caption": "Store entrance",
      "position": 1
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file_key` | string | Yes | S3 file key from presign |
| `caption` | string | No | Image caption |
| `position` | integer | No | Display order position |

**Response 201:**
```json
{
  "data": {
    "id": "img-uuid-789",
    "type": "image",
    "attributes": {
      "unique_id": "img-uuid-789",
      "url": "https://cdn.23blocks.com/locations/loc-uuid-123/storefront.jpg",
      "caption": "Store entrance",
      "position": 1,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /locations/:unique_id/images/:image_unique_id - Delete Image

Deletes an image from a location.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/images/img-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Location Identities

### POST /locations/:unique_id/identities - Add Identity

Associates a user identity with a location.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/identities" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "user-uuid-456",
    "role": "manager"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User to associate |
| `role` | string | No | Role at location (manager, staff, etc.) |

**Response 201:**
```json
{
  "data": {
    "id": "assoc-uuid",
    "type": "location_identity",
    "attributes": {
      "user_unique_id": "user-uuid-456",
      "location_unique_id": "loc-uuid-123",
      "role": "manager",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /locations/:unique_id/identities/:user_unique_id - Remove Identity

Removes a user identity from a location.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/identities/user-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Location Slots

### GET /locations/:unique_id/slots - List Slots

Lists all time slots for a location.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/slots" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "slot-uuid-789",
      "type": "slot",
      "attributes": {
        "unique_id": "slot-uuid-789",
        "name": "Morning Slot",
        "start_time": "09:00",
        "end_time": "12:00",
        "capacity": 20,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

### POST /locations/:unique_id/slots - Create Slot

Creates a time slot for a location.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/slots" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "slot": {
      "name": "Morning Slot",
      "start_time": "09:00",
      "end_time": "12:00",
      "capacity": 20
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Slot name |
| `start_time` | string | Yes | Start time (HH:MM) |
| `end_time` | string | Yes | End time (HH:MM) |
| `capacity` | integer | No | Maximum capacity |

**Response 201:**
```json
{
  "data": {
    "id": "slot-uuid-789",
    "type": "slot",
    "attributes": {
      "unique_id": "slot-uuid-789",
      "name": "Morning Slot",
      "start_time": "09:00",
      "end_time": "12:00",
      "capacity": 20,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /locations/:unique_id/slots/:slot_unique_id - Update Slot

Updates a time slot.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/slots/slot-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "slot": {
      "capacity": 25,
      "end_time": "13:00"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "slot-uuid-789",
    "type": "slot",
    "attributes": {
      "unique_id": "slot-uuid-789",
      "name": "Morning Slot",
      "start_time": "09:00",
      "end_time": "13:00",
      "capacity": 25,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /locations/:unique_id/slots/:slot_unique_id - Delete Slot

Deletes a time slot.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/slots/slot-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Location Taxes

### POST /locations/:unique_id/taxes - Add Tax

Adds a tax configuration to a location.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/taxes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tax": {
      "name": "Sales Tax",
      "rate": 8.875,
      "tax_type": "sales"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Tax name |
| `rate` | float | Yes | Tax rate percentage |
| `tax_type` | string | No | Type of tax (sales, vat, service) |

**Response 201:**
```json
{
  "data": {
    "id": "tax-uuid-101",
    "type": "tax",
    "attributes": {
      "unique_id": "tax-uuid-101",
      "name": "Sales Tax",
      "rate": 8.875,
      "tax_type": "sales",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /locations/:unique_id/taxes/:tax_unique_id - Update Tax

Updates a tax configuration.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/taxes/tax-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tax": {
      "rate": 9.0
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "tax-uuid-101",
    "type": "tax",
    "attributes": {
      "unique_id": "tax-uuid-101",
      "name": "Sales Tax",
      "rate": 9.0,
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

---

### DELETE /locations/:unique_id/taxes/:tax_unique_id - Delete Tax

Deletes a tax configuration from a location.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/taxes/tax-uuid-101" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## Geographic Hierarchy

### GET /countries/:country_code/locations - Locations by Country

Lists locations within a country.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/countries/US/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "loc-uuid-123",
      "type": "location",
      "attributes": {
        "unique_id": "loc-uuid-123",
        "name": "Downtown Office",
        "code": "DT-001",
        "address": "123 Main St, New York, NY 10001",
        "status": "active"
      }
    }
  ],
  "meta": {
    "totalPages": 10,
    "totalRecords": 150
  }
}
```

---

### GET /states/:code/locations - Locations by State

Lists locations within a state.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/states/NY/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as country locations.

---

### GET /counties/:code/locations - Locations by County

Lists locations within a county.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/counties/new-york-county/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as country locations.

---

### GET /cities/:code/locations - Locations by City

Lists locations within a city.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/cities/new-york/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as country locations.

---

### GET /divisions/:code/locations - Locations by Division

Lists locations within an administrative division.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/divisions/manhattan/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as country locations.

---

### GET /neighborhoods/:code/locations - Locations by Neighborhood

Lists locations within a neighborhood.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/neighborhoods/soho/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as country locations.

---

### GET /buildings/:code/locations - Locations by Building

Lists locations within a building.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/buildings/empire-state/locations?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:** Same format as country locations.

---

## Location Groups

### GET /location_groups - List Groups

Lists all location groups.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/location_groups?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "group-uuid-123",
      "type": "location_group",
      "attributes": {
        "unique_id": "group-uuid-123",
        "name": "East Coast Offices",
        "description": "All east coast office locations",
        "location_count": 12,
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 8
  }
}
```

---

### GET /location_groups/:unique_id/ - Get Group

Retrieves a single location group.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/location_groups/group-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "group-uuid-123",
    "type": "location_group",
    "attributes": {
      "unique_id": "group-uuid-123",
      "name": "East Coast Offices",
      "description": "All east coast office locations",
      "location_count": 12,
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "locations": {
        "data": [
          { "id": "loc-uuid-123", "type": "location" }
        ]
      }
    }
  }
}
```

---

### POST /location_groups/ - Create Group

Creates a new location group.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/location_groups/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location_group": {
      "name": "East Coast Offices",
      "description": "All east coast office locations"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Group name |
| `description` | string | No | Group description |

**Response 201:**
```json
{
  "data": {
    "id": "group-uuid-123",
    "type": "location_group",
    "attributes": {
      "unique_id": "group-uuid-123",
      "name": "East Coast Offices",
      "description": "All east coast office locations",
      "location_count": 0,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Data Models

### Location
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Location name |
| `code` | string | Unique location code |
| `address` | string | Full address |
| `latitude` | float | Latitude coordinate |
| `longitude` | float | Longitude coordinate |
| `phone` | string | Contact phone |
| `email` | string | Contact email |
| `location_type` | string | Type (office, store, warehouse, etc.) |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### Hour
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `day_of_week` | string | Day of the week |
| `open_time` | string | Opening time (HH:MM) |
| `close_time` | string | Closing time (HH:MM) |
| `is_closed` | boolean | Whether closed on this day |

### Slot
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Slot name |
| `start_time` | string | Start time (HH:MM) |
| `end_time` | string | End time (HH:MM) |
| `capacity` | integer | Maximum capacity |
| `status` | string | active, inactive |

### Tax
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Tax name |
| `rate` | float | Tax rate percentage |
| `tax_type` | string | Type of tax |

### LocationGroup
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Group name |
| `description` | string | Group description |
| `location_count` | integer | Number of locations |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Location Not Found",
    "detail": "The requested location could not be found."
  }]
}
```
