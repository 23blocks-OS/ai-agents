# Premises API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### GET /locations/:unique_id/premises - List Premises

Lists all premises within a location.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/premises?page=1&records=20" \
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
      "id": "premise-uuid-456",
      "type": "premise",
      "attributes": {
        "unique_id": "premise-uuid-456",
        "name": "Conference Room A",
        "location_id": "loc-uuid-123",
        "floor": "3",
        "capacity": 20,
        "premise_type": "conference_room",
        "amenities": ["projector", "whiteboard", "video_conferencing"],
        "status": "active",
        "created_at": "2025-01-10T10:30:00Z",
        "updated_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 2,
    "totalRecords": 15
  }
}
```

---

### GET /locations/:unique_id/premises/:premise_unique_id - Get Premise

Retrieves a single premise by unique ID.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "premise-uuid-456",
    "type": "premise",
    "attributes": {
      "unique_id": "premise-uuid-456",
      "name": "Conference Room A",
      "location_id": "loc-uuid-123",
      "floor": "3",
      "capacity": 20,
      "premise_type": "conference_room",
      "amenities": ["projector", "whiteboard", "video_conferencing"],
      "status": "active",
      "created_at": "2025-01-10T10:30:00Z"
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
- `404 Not Found` - Premise not found

---

### POST /locations/:unique_id/premises - Create Premise

Creates a new premise within a location.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/premises" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "premise": {
      "name": "Conference Room A",
      "floor": "3",
      "capacity": 20,
      "premise_type": "conference_room",
      "amenities": ["projector", "whiteboard", "video_conferencing"],
      "status": "active"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Premise name |
| `floor` | string | No | Floor number/identifier |
| `capacity` | integer | No | Maximum capacity |
| `premise_type` | string | No | Type (conference_room, office, workspace, etc.) |
| `amenities` | array | No | List of available amenities |
| `status` | string | No | Status (active, inactive, maintenance) |

**Response 201:**
```json
{
  "data": {
    "id": "premise-uuid-456",
    "type": "premise",
    "attributes": {
      "unique_id": "premise-uuid-456",
      "name": "Conference Room A",
      "location_id": "loc-uuid-123",
      "floor": "3",
      "capacity": 20,
      "premise_type": "conference_room",
      "amenities": ["projector", "whiteboard", "video_conferencing"],
      "status": "active",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /locations/:unique_id/premises/:premise_unique_id - Update Premise

Updates an existing premise.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "premise": {
      "capacity": 25,
      "amenities": ["projector", "whiteboard", "video_conferencing", "phone"]
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "premise-uuid-456",
    "type": "premise",
    "attributes": {
      "unique_id": "premise-uuid-456",
      "name": "Conference Room A",
      "capacity": 25,
      "amenities": ["projector", "whiteboard", "video_conferencing", "phone"],
      "updated_at": "2025-01-12T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Premise not found

---

### DELETE /locations/:unique_id/premises/:premise_unique_id - Delete Premise

Deletes a premise.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Premise not found

---

## Premise Tags

### POST /locations/:unique_id/premises/:premise_unique_id/tags - Add Tag

Adds a tag to a premise.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/tags" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "tag_unique_id": "tag-uuid-789"
  }'
```

**Response 200:**
```json
{
  "message": "Tag added successfully"
}
```

---

### DELETE /locations/:unique_id/premises/:premise_unique_id/tags/:tag_unique_id - Remove Tag

Removes a tag from a premise.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/tags/tag-uuid-789" \
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

## Premise Events

### GET /locations/:unique_id/premises/:premise_unique_id/events - List Events

Lists all events scheduled at a premise.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/events?page=1&records=20" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": [
    {
      "id": "event-uuid-101",
      "type": "event",
      "attributes": {
        "unique_id": "event-uuid-101",
        "name": "All Hands Meeting",
        "description": "Monthly all-hands company meeting",
        "start_time": "2025-02-15T14:00:00Z",
        "end_time": "2025-02-15T15:30:00Z",
        "attendees": 50,
        "status": "scheduled",
        "created_at": "2025-01-10T10:30:00Z"
      }
    }
  ],
  "meta": {
    "totalPages": 3,
    "totalRecords": 42
  }
}
```

---

### POST /locations/:unique_id/premises/:premise_unique_id/events - Create Event

Creates a new event at a premise.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/locations/loc-uuid-123/premises/premise-uuid-456/events" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "name": "All Hands Meeting",
      "description": "Monthly all-hands company meeting",
      "start_time": "2025-02-15T14:00:00Z",
      "end_time": "2025-02-15T15:30:00Z",
      "attendees": 50
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Event name |
| `description` | string | No | Event description |
| `start_time` | datetime | Yes | Event start time (ISO 8601) |
| `end_time` | datetime | Yes | Event end time (ISO 8601) |
| `attendees` | integer | No | Expected number of attendees |

**Response 201:**
```json
{
  "data": {
    "id": "event-uuid-101",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-101",
      "name": "All Hands Meeting",
      "description": "Monthly all-hands company meeting",
      "start_time": "2025-02-15T14:00:00Z",
      "end_time": "2025-02-15T15:30:00Z",
      "attendees": 50,
      "status": "scheduled",
      "created_at": "2025-01-12T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors (time conflicts, capacity exceeded)

---

## Areas

### GET /areas/:unique_id - Get Area

Retrieves a specific area.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/areas/area-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "data": {
    "id": "area-uuid-123",
    "type": "area",
    "attributes": {
      "unique_id": "area-uuid-123",
      "name": "Reception Area",
      "description": "Main entrance reception area",
      "created_at": "2025-01-10T10:30:00Z"
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
- `404 Not Found` - Area not found

---

### POST /areas/:unique_id/tags/ - Add Tag to Area

Adds a tag to an area.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/areas/area-uuid-123/tags/" \
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

### DELETE /areas/:unique_id/tags/:tag_unique_id - Remove Tag from Area

Removes a tag from an area.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/areas/area-uuid-123/tags/tag-uuid-456" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 200:**
```json
{
  "message": "Tag removed successfully"
}
```
