# CRM Events API — Endpoints

Full endpoint documentation. See [SKILL.md](SKILL.md) for setup, data models, and SDK usage.

---

## Endpoints

### GET /events/ - List Events

Lists all events with pagination.

**Request:**
```bash
curl -X GET "$BLOCKS_API_URL/events/?page=1&records=20" \
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
      "id": "event-uuid-123",
      "type": "event",
      "attributes": {
        "unique_id": "event-uuid-123",
        "title": "Annual Client Conference",
        "event_type": "conference",
        "start_time": "2026-04-15T09:00:00Z",
        "end_time": "2026-04-15T17:00:00Z",
        "location": "Grand Ballroom, Hilton Hotel",
        "participants": 150,
        "status": "scheduled",
        "created_at": "2026-01-10T10:30:00Z",
        "updated_at": "2026-01-10T10:30:00Z"
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

### POST /events/ - Create Event

Creates a new event.

**Request:**
```bash
curl -X POST "$BLOCKS_API_URL/events/" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "title": "Product Launch Webinar",
      "event_type": "webinar",
      "start_time": "2026-05-01T14:00:00Z",
      "end_time": "2026-05-01T15:30:00Z",
      "location": "Online - Zoom"
    }
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `title` | string | Yes | Event title |
| `event_type` | string | No | Event type (conference, webinar, workshop, meetup, etc.) |
| `start_time` | timestamp | Yes | Start time (ISO 8601) |
| `end_time` | timestamp | Yes | End time (ISO 8601) |
| `location` | string | No | Event location |

**Response 201:**
```json
{
  "data": {
    "id": "event-uuid-456",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-456",
      "title": "Product Launch Webinar",
      "event_type": "webinar",
      "start_time": "2026-05-01T14:00:00Z",
      "end_time": "2026-05-01T15:30:00Z",
      "location": "Online - Zoom",
      "participants": 0,
      "status": "scheduled",
      "created_at": "2026-02-15T10:30:00Z"
    }
  }
}
```

**Errors:**
- `422 Unprocessable Entity` - Validation errors

---

### PUT /events/:unique_id - Update Event

Updates an existing event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "event": {
      "title": "Annual Client Conference 2026",
      "status": "in_progress"
    }
  }'
```

**Response 200:**
```json
{
  "data": {
    "id": "event-uuid-123",
    "type": "event",
    "attributes": {
      "unique_id": "event-uuid-123",
      "title": "Annual Client Conference 2026",
      "status": "in_progress",
      "updated_at": "2026-02-15T14:00:00Z"
    }
  }
}
```

**Errors:**
- `404 Not Found` - Event not found

---

### DELETE /events/:unique_id - Delete Event

Deletes an event.

**Request:**
```bash
curl -X DELETE "$BLOCKS_API_URL/events/event-uuid-123" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY"
```

**Response 204:** No content

**Errors:**
- `404 Not Found` - Event not found

---

## Contact Status Management

### PUT /events/:unique_id/contacts/confirmation - Contact Confirmation

Updates contact confirmation status for an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/contacts/confirmation" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "contact-uuid-456",
    "confirmation_status": "confirmed"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | uuid | Yes | Contact ID |
| `confirmation_status` | string | Yes | confirmed, declined, tentative |

**Response 200:**
```json
{
  "message": "Contact confirmation updated successfully"
}
```

---

### PUT /events/:unique_id/contacts/checking - Contact Check-in

Records contact check-in for an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/contacts/checking" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "contact-uuid-456",
    "checked_in_at": "2026-04-15T08:45:00Z"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | uuid | Yes | Contact ID |
| `checked_in_at` | timestamp | No | Check-in time (defaults to current time) |

**Response 200:**
```json
{
  "message": "Contact checked in successfully"
}
```

---

### PUT /events/:unique_id/contacts/checkout - Contact Checkout

Records contact checkout for an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/contacts/checkout" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "contact-uuid-456",
    "checked_out_at": "2026-04-15T16:30:00Z"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | uuid | Yes | Contact ID |
| `checked_out_at` | timestamp | No | Checkout time (defaults to current time) |

**Response 200:**
```json
{
  "message": "Contact checked out successfully"
}
```

---

### PUT /events/:unique_id/contacts/notes - Contact Notes

Adds or updates notes for a contact at an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/contacts/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contact_id": "contact-uuid-456",
    "notes": "Interested in enterprise plan, schedule follow-up demo"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `contact_id` | uuid | Yes | Contact ID |
| `notes` | string | Yes | Notes text |

**Response 200:**
```json
{
  "message": "Contact notes updated successfully"
}
```

---

## Employee Status Management

### PUT /events/:unique_id/employees/confirmation - Employee Confirmation

Updates employee confirmation status for an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/employees/confirmation" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "employee_id": "user-uuid-789",
    "confirmation_status": "confirmed"
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `employee_id` | uuid | Yes | Employee/User ID |
| `confirmation_status` | string | Yes | confirmed, declined, tentative |

**Response 200:**
```json
{
  "message": "Employee confirmation updated successfully"
}
```

---

### PUT /events/:unique_id/employees/checking - Employee Check-in

Records employee check-in for an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/employees/checking" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "employee_id": "user-uuid-789",
    "checked_in_at": "2026-04-15T08:30:00Z"
  }'
```

**Response 200:**
```json
{
  "message": "Employee checked in successfully"
}
```

---

### PUT /events/:unique_id/employees/checkout - Employee Checkout

Records employee checkout for an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/employees/checkout" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "employee_id": "user-uuid-789",
    "checked_out_at": "2026-04-15T17:15:00Z"
  }'
```

**Response 200:**
```json
{
  "message": "Employee checked out successfully"
}
```

---

## Admin Notes

### PUT /events/:unique_id/admin/notes - Admin Notes

Adds or updates admin notes for an event.

**Request:**
```bash
curl -X PUT "$BLOCKS_API_URL/events/event-uuid-123/admin/notes" \
  -H "Authorization: Bearer $BLOCKS_AUTH_TOKEN" \
  -H "AppId: $BLOCKS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "notes": "Venue confirmed. Catering booked for 150 attendees. AV setup at 7am."
  }'
```

**Request Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `notes` | string | Yes | Admin notes text |

**Response 200:**
```json
{
  "message": "Admin notes updated successfully"
}
```
