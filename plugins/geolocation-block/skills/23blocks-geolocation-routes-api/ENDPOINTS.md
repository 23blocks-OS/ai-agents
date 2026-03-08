# Routes API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### GET /routes - List Routes

Lists all routes with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/routes?page=1&records=20" \
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
      "id": "route-uuid-123",
      "type": "route",
      "attributes": {
        "unique_id": "route-uuid-123",
        "name": "Morning Delivery Route",
        "description": "Daily morning delivery stops in downtown area",
        "locations": [
          {
            "location_unique_id": "loc-uuid-001",
            "sequence": 1,
            "name": "Warehouse A"
          },
          {
            "location_unique_id": "loc-uuid-002",
            "sequence": 2,
            "name": "Customer Site B"
          }
        ],
        "distance": 25.4,
        "estimated_time": 45,
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 35
  }
}
```

---

### GET /routes/:unique_id/ - Get Route

Retrieves a single route by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/routes/route-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "route-uuid-123",
    "type": "route",
    "attributes": {
      "unique_id": "route-uuid-123",
      "name": "Morning Delivery Route",
      "description": "Daily morning delivery stops in downtown area",
      "locations": [
        {
          "location_unique_id": "loc-uuid-001",
          "sequence": 1,
          "name": "Warehouse A",
          "address": "100 Industrial Blvd",
          "latitude": 40.7128,
          "longitude": -74.0060
        },
        {
          "location_unique_id": "loc-uuid-002",
          "sequence": 2,
          "name": "Customer Site B",
          "address": "200 Commerce St",
          "latitude": 40.7200,
          "longitude": -74.0100
        }
      ],
      "distance": 25.4,
      "estimated_time": 45,
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
    },
    "relationships": {
      "users": {
        "data": [
          { "id": "user-uuid-456", "type": "user_identity" }
        ]
      }
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route not found

---

### POST /routes/ - Create Route

Creates a new route.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/routes/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "route": {
      "name": "Morning Delivery Route",
      "description": "Daily morning delivery stops in downtown area"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Route name |
| `description` | string | No | Route description |

**Response 201:**
```json
{
  "data": {
    "id": "route-uuid-123",
    "type": "route",
    "attributes": {
      "unique_id": "route-uuid-123",
      "name": "Morning Delivery Route",
      "description": "Daily morning delivery stops in downtown area",
      "locations": [],
      "distance": 0,
      "estimated_time": 0,
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /routes/:unique_id/ - Update Route

Updates an existing route.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/routes/route-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "route": {
      "name": "Morning Delivery Route - Updated",
      "description": "Updated daily morning delivery stops"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "route-uuid-123",
    "type": "route",
    "attributes": {
      "unique_id": "route-uuid-123",
      "name": "Morning Delivery Route - Updated",
      "description": "Updated daily morning delivery stops",
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route not found

---

### DELETE /routes/:unique_id/ - Delete Route

Deletes a route.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/routes/route-uuid-123/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Route not found

---

## Route Locations

### POST /routes/:unique_id/locations - Add Location to Route

Adds a location stop to a route.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/routes/route-uuid-123/locations" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "location_unique_id": "loc-uuid-001",
    "sequence": 1
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `location_unique_id` | uuid | Yes | Location to add as stop |
| `sequence` | integer | No | Order in the route |

**Response 201:**
```json
{
  "data": {
    "id": "route-loc-uuid",
    "type": "route_location",
    "attributes": {
      "route_unique_id": "route-uuid-123",
      "location_unique_id": "loc-uuid-001",
      "sequence": 1,
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route or location not found
- `409 Conflict` - Location already in route

---

## Route Users

### POST /routes/:unique_id/users - Assign User to Route

Assigns a user to a route.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/routes/route-uuid-123/users" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "user_unique_id": "user-uuid-456"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `user_unique_id` | uuid | Yes | User to assign |

**Response 201:**
```json
{
  "data": {
    "id": "route-user-uuid",
    "type": "route_user",
    "attributes": {
      "route_unique_id": "route-uuid-123",
      "user_unique_id": "user-uuid-456",
      "assigned_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Route or user not found
- `409 Conflict` - User already assigned to route

---

### GET /users/:unique_id/routes - User Routes

Retrieves all routes assigned to a user.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/users/user-uuid-456/routes?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "route-uuid-123",
      "type": "route",
      "attributes": {
        "unique_id": "route-uuid-123",
        "name": "Morning Delivery Route",
        "description": "Daily morning delivery stops",
        "distance": 25.4,
        "estimated_time": 45,
        "status": "active"
      }
    }
  ],
  "meta": {
    "totalPages": 1,
    "totalRecords": 3
  }
}
```

---

## Location Tracking

### POST /users/:unique_id/routes/:route_unique_id/tracker/location - Track Location

Reports the current location of a user on a route for real-time tracking.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/users/user-uuid-456/routes/route-uuid-123/tracker/location" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tracker": {
      "latitude": 40.7150,
      "longitude": -74.0080,
      "accuracy": 10.5,
      "speed": 35.2,
      "heading": 90
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `latitude` | float | Yes | Current latitude |
| `longitude` | float | Yes | Current longitude |
| `accuracy` | float | No | GPS accuracy in meters |
| `speed` | float | No | Current speed in km/h |
| `heading` | float | No | Direction in degrees (0-360) |

**Response 200:**
```json
{
  "data": {
    "id": "tracker-uuid",
    "type": "tracker_location",
    "attributes": {
      "user_unique_id": "user-uuid-456",
      "route_unique_id": "route-uuid-123",
      "latitude": 40.7150,
      "longitude": -74.0080,
      "accuracy": 10.5,
      "speed": 35.2,
      "heading": 90,
      "tracked_at": "2025-01-12T14:30:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - User or route not found
- `422 Unprocessable Entity` - Invalid coordinates
