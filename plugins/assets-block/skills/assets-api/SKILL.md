---
name: assets-assets-api
description: Create and manage 23blocks assets via REST API. Use when creating, updating, or deleting assets, managing asset categories, parts, maintenance, lending, ownership transfer, images, and OTP generation.
allowed-tools: Read, Write, Bash, Grep, Glob
user-invocable: true
---

# Assets API

Complete API reference for 23blocks Assets Block asset management with categories, parts, maintenance, lending, ownership transfer, images, and OTP.

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
curl -X GET "$BLOCKS_API_URL/assets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json"
```

---

## CRUD Endpoints

### GET /assets/ - List Assets

Lists all assets with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets?page=1&records=20" \
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
      "id": "asset-uuid-123",
      "type": "asset",
      "attributes": {
        "unique_id": "asset-uuid-123",
        "name": "Laptop Dell XPS 15",
        "description": "Company laptop for engineering team",
        "serial_number": "SN-2025-001",
        "status": "active",
        "condition": "good",
        "purchase_date": "2025-01-05",
        "purchase_price": 1899.99,
        "current_value": 1700.00,
        "warranty_expiry": "2028-01-05",
        "location": "Building A, Floor 3",
        "owner_unique_id": "user-uuid-456",
        "vendor_unique_id": "vendor-uuid-789",
        "warehouse_unique_id": null,
        "category_unique_id": "category-uuid-123",
        "maintenance_status": "none",
        "maintenance_due_at": null,
        "images_count": 2,
        "events_count": 5,
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
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

### GET /assets/:unique_id/ - Get Asset

Retrieves a single asset by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/asset-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "asset-uuid-123",
    "type": "asset",
    "attributes": {
      "unique_id": "asset-uuid-123",
      "name": "Laptop Dell XPS 15",
      "description": "Company laptop for engineering team",
      "serial_number": "SN-2025-001",
      "status": "active",
      "condition": "good",
      "purchase_date": "2025-01-05",
      "purchase_price": 1899.99,
      "current_value": 1700.00,
      "warranty_expiry": "2028-01-05",
      "location": "Building A, Floor 3",
      "owner_unique_id": "user-uuid-456",
      "vendor_unique_id": "vendor-uuid-789",
      "warehouse_unique_id": null,
      "category_unique_id": "category-uuid-123",
      "maintenance_status": "none",
      "maintenance_due_at": null,
      "images_count": 2,
      "events_count": 5,
      "payload": {},
      "created_at": "2025-01-10T10:30:00Z",
      "updated_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "category": {
        "data": { "id": "category-uuid-123", "type": "category" }
      },
      "vendor": {
        "data": { "id": "vendor-uuid-789", "type": "vendor" }
      },
      "tags": {
        "data": [
          { "id": "tag-uuid-001", "type": "tag" }
        ]
      },
      "parts": {
        "data": []
      },
      "images": {
        "data": [
          { "id": "file-uuid-001", "type": "image" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Asset not found

---

### POST /assets/ - Create Asset

Creates a new asset.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "asset": {
      "name": "MacBook Pro 16",
      "description": "Design team workstation",
      "serial_number": "SN-2025-002",
      "status": "active",
      "condition": "new",
      "purchase_date": "2025-01-15",
      "purchase_price": 2499.99,
      "warranty_expiry": "2028-01-15",
      "location": "Building B, Floor 2",
      "vendor_unique_id": "vendor-uuid-apple",
      "category_unique_id": "category-uuid-laptops"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Asset name |
| `description` | string | No | Asset description |
| `serial_number` | string | No | Serial number |
| `status` | enum | No | active, inactive, retired, lost (default: active) |
| `condition` | enum | No | new, good, fair, poor |
| `purchase_date` | date | No | Purchase date (YYYY-MM-DD) |
| `purchase_price` | decimal | No | Original purchase price |
| `current_value` | decimal | No | Current estimated value |
| `warranty_expiry` | date | No | Warranty expiration date |
| `location` | string | No | Physical location |
| `vendor_unique_id` | uuid | No | Vendor ID |
| `warehouse_unique_id` | uuid | No | Warehouse ID |
| `category_unique_id` | uuid | No | Category ID |
| `payload` | json | No | Custom metadata |

**Response 201:**
```json
{
  "data": {
    "id": "new-asset-uuid",
    "type": "asset",
    "attributes": {
      "unique_id": "new-asset-uuid",
      "name": "MacBook Pro 16",
      "serial_number": "SN-2025-002",
      "status": "active",
      "condition": "new",
      "owner_unique_id": "current-user-uuid",
      "created_at": "2025-01-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /assets/:unique_id - Update Asset

Updates an existing asset.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assets/asset-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "asset": {
      "condition": "fair",
      "current_value": 1200.00,
      "location": "Building C, Floor 1"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "asset-uuid-123",
    "type": "asset",
    "attributes": {
      "unique_id": "asset-uuid-123",
      "condition": "fair",
      "current_value": 1200.00,
      "location": "Building C, Floor 1",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Asset not found
- `422 Unprocessable Entity` - Validation errors

---

### DELETE /assets/:unique_id - Delete Asset

Soft-deletes an asset (moves to trash).

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/assets/asset-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Asset not found

---

### GET /assets/trash/show - View Trash

Lists soft-deleted assets.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/assets/trash/show" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "asset-uuid-deleted",
      "type": "asset",
      "attributes": {
        "unique_id": "asset-uuid-deleted",
        "name": "Old Monitor",
        "status": "deleted",
        "deleted_at": "2025-01-10T10:30:00Z"
      }
    }
  ]
}
```

---

## Asset Management

### POST /assets/:unique_id/categories - Assign Category

Assigns a category to an asset.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/categories" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "category_unique_id": "category-uuid-456"
  }'
```

**Response 200:**
```json
{
  "message": "Category assigned successfully"
}
```

---

### PUT /assets/:unique_id/parts - Update Parts

Updates or adds parts/components to an asset.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assets/asset-uuid-123/parts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "parts": [
      {
        "name": "RAM Module",
        "serial_number": "RAM-SN-001",
        "description": "32GB DDR5"
      },
      {
        "name": "SSD",
        "serial_number": "SSD-SN-001",
        "description": "1TB NVMe"
      }
    ]
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "asset-uuid-123",
    "type": "asset",
    "attributes": {
      "unique_id": "asset-uuid-123",
      "parts_count": 2
    },
    "relationships": {
      "parts": {
        "data": [
          { "id": "part-uuid-001", "type": "part" },
          { "id": "part-uuid-002", "type": "part" }
        ]
      }
    }
  }
}
```

---

### DELETE /assets/:unique_id/parts - Remove Parts

Removes parts from an asset.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/assets/asset-uuid-123/parts" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "part_unique_ids": ["part-uuid-001"]
  }'
```

**Response 204:** No content

---

### PUT /assets/:unique_id/maintenance - Set Maintenance

Sets the maintenance status and schedule for an asset.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assets/asset-uuid-123/maintenance" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "maintenance": {
      "status": "scheduled",
      "due_at": "2025-04-01T09:00:00Z",
      "notes": "Annual hardware inspection and cleaning"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | enum | Yes | none, scheduled, in_progress, completed |
| `due_at` | timestamp | No | When maintenance is due |
| `notes` | string | No | Maintenance notes |

**Response 200:**
```json
{
  "data": {
    "id": "asset-uuid-123",
    "type": "asset",
    "attributes": {
      "unique_id": "asset-uuid-123",
      "maintenance_status": "scheduled",
      "maintenance_due_at": "2025-04-01T09:00:00Z",
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### POST /assets/:unique_id/lend - Lend Asset

Lends an asset to a user.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/lend" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "lend": {
      "user_unique_id": "user-uuid-789",
      "due_date": "2025-06-01",
      "notes": "Temporary assignment for project Alpha"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User receiving the asset |
| `due_date` | date | No | Expected return date |
| `notes` | string | No | Lending notes |

**Response 200:**
```json
{
  "data": {
    "id": "asset-uuid-123",
    "type": "asset",
    "attributes": {
      "unique_id": "asset-uuid-123",
      "status": "lent",
      "lent_to_unique_id": "user-uuid-789",
      "lend_due_date": "2025-06-01",
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### PUT /assets/:unique_id/transfer - Transfer Ownership

Transfers asset ownership to another user.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assets/asset-uuid-123/transfer" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "transfer": {
      "new_owner_unique_id": "user-uuid-999",
      "notes": "Permanent reassignment to new team lead"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `new_owner_unique_id` | uuid | Yes | New owner user ID |
| `notes` | string | No | Transfer notes |

**Response 200:**
```json
{
  "data": {
    "id": "asset-uuid-123",
    "type": "asset",
    "attributes": {
      "unique_id": "asset-uuid-123",
      "owner_unique_id": "user-uuid-999",
      "updated_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

## Asset Images

### PUT /assets/:unique_id/presign - Presign Image Upload

Generates a presigned URL for uploading an asset image.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/assets/asset-uuid-123/presign" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "filename": "asset-photo.jpg",
      "content_type": "image/jpeg"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filename` | string | Yes | Name of the file |
| `content_type` | string | Yes | MIME type (e.g., image/jpeg, image/png) |

**Response 200:**
```json
{
  "data": {
    "presigned_url": "https://storage.example.com/upload?signature=abc123",
    "file_unique_id": "file-uuid-789",
    "expires_at": "2025-01-12T11:30:00Z"
  }
}
```

---

### POST /assets/:unique_id/images - Upload Image

Confirms an image upload after using the presigned URL.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/images" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file": {
      "file_unique_id": "file-uuid-789"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "file-uuid-789",
    "type": "image",
    "attributes": {
      "unique_id": "file-uuid-789",
      "url": "https://storage.example.com/assets/asset-photo.jpg",
      "filename": "asset-photo.jpg",
      "content_type": "image/jpeg",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

---

### DELETE /assets/:unique_id/images/:unique_file_id - Delete Image

Deletes an asset image.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/assets/asset-uuid-123/images/file-uuid-789" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

---

## OTP

### POST /assets/:unique_id/otp - Generate OTP

Generates a one-time password for asset verification.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/assets/asset-uuid-123/otp" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "otp": "847291",
    "expires_at": "2025-01-12T10:45:00Z",
    "asset_unique_id": "asset-uuid-123"
  }
}
```

---

## Data Models

### Asset
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Asset name |
| `description` | string | Asset description |
| `serial_number` | string | Serial number |
| `status` | enum | active, inactive, retired, lost, lent, deleted |
| `condition` | enum | new, good, fair, poor |
| `purchase_date` | date | Purchase date |
| `purchase_price` | decimal | Original purchase price |
| `current_value` | decimal | Current estimated value |
| `warranty_expiry` | date | Warranty expiration date |
| `location` | string | Physical location |
| `owner_unique_id` | uuid | Owner user ID |
| `vendor_unique_id` | uuid | Vendor ID |
| `warehouse_unique_id` | uuid | Warehouse ID |
| `category_unique_id` | uuid | Category ID |
| `maintenance_status` | enum | none, scheduled, in_progress, completed |
| `maintenance_due_at` | timestamp | Maintenance due date |
| `lent_to_unique_id` | uuid | User the asset is lent to |
| `lend_due_date` | date | Expected return date |
| `images_count` | integer | Number of images |
| `events_count` | integer | Number of events |
| `payload` | jsonb | Custom metadata |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |
| `deleted_at` | timestamp | Soft delete time |

### Part
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Part name |
| `serial_number` | string | Part serial number |
| `description` | string | Part description |
| `asset_unique_id` | uuid | Parent asset ID |
| `created_at` | timestamp | Creation time |

---

## Error Response Format

```json
{
  "errors": [{
    "status": "404",
    "code": "not_found",
    "title": "Asset Not Found",
    "detail": "The requested asset could not be found."
  }]
}
```

| Code | Description |
|------|-------------|
| `401` | Unauthorized - Invalid or missing credentials |
| `404` | Not Found - Asset not found |
| `422` | Unprocessable Entity - Validation errors |
