---
name: 23blocks-university-calendar-api
description: "University Block calendar events, including course-specific student events. Use for class schedules."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.0"
---

# Calendar API

Complete API reference for 23blocks university calendar event management with course-specific scheduling.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://university.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

Request and response detail for each endpoint: [ENDPOINTS.md](ENDPOINTS.md).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/events/` | List Events |
| GET | `/events/:unique_id` | Get Event |
| POST | `/events/` | Create Event |
| PUT | `/events/:unique_id` | Update Event |
| DELETE | `/events/:unique_id` | Delete Event |
| GET | `/courses/:unique_id/students/:user_unique_id/events` | Student Course Events (Course-Specific Student Events) |
| GET | `/courses/:unique_id/students/:user_unique_id/events/:event_unique_id` | Specific Student Course Event (Course-Specific Student Events) |
| POST | `/courses/:unique_id/students/:user_unique_id/events` | Create Student Course Event (Course-Specific Student Events) |

## Data Models

### Event
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `title` | string | Event title |
| `description` | string | Event description |
| `event_type` | string | exam, lecture, study_group, office_hours, deadline, other |
| `start_time` | datetime | Start time (ISO 8601) |
| `end_time` | datetime | End time (ISO 8601) |
| `location` | string | Event location |
| `recurrence` | string | daily, weekly, biweekly, monthly, null |
| `attendees` | array | Array of user unique IDs |
| `status` | enum | active, cancelled |
| `created_at` | timestamp | Creation time |

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"422","code":"validation_error","title":"Validation Error","detail":"Start time must be before end time."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-university`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`.

### Available Methods

```typescript
// CalendarsService — client.university.calendars

// Student Availability
client.university.calendars.getStudentAvailability(userUniqueId: string): Promise<Availability[]>;
client.university.calendars.addStudentAvailability(userUniqueId: string, data: CreateAvailabilityRequest): Promise<Availability>;
client.university.calendars.updateStudentAvailability(userUniqueId: string, availabilityUniqueId: string, data: UpdateAvailabilityRequest): Promise<Availability>;
client.university.calendars.updateStudentAvailabilities(userUniqueId: string, data: BulkUpdateAvailabilityRequest): Promise<Availability[]>;
client.university.calendars.deleteStudentAvailability(userUniqueId: string, availabilityUniqueId: string): Promise<void>;
client.university.calendars.deleteAllStudentAvailability(userUniqueId: string): Promise<void>;

// Teacher Availability
client.university.calendars.getTeacherAvailability(teacherUniqueId: string): Promise<Availability[]>;
client.university.calendars.addTeacherAvailability(teacherUniqueId: string, data: CreateAvailabilityRequest): Promise<Availability>;
client.university.calendars.updateTeacherAvailability(teacherUniqueId: string, availabilityUniqueId: string, data: UpdateAvailabilityRequest): Promise<Availability>;
client.university.calendars.deleteTeacherAvailability(teacherUniqueId: string, availabilityUniqueId: string): Promise<void>;
client.university.calendars.deleteAllTeacherAvailability(teacherUniqueId: string): Promise<void>;

// Events
client.university.calendars.listEvents(params?: ListCalendarEventsParams): Promise<PageResult<CalendarEvent>>;
client.university.calendars.getEvent(uniqueId: string): Promise<CalendarEvent>;
client.university.calendars.createEvent(data: CreateCalendarEventRequest): Promise<CalendarEvent>;
client.university.calendars.updateEvent(uniqueId: string, data: UpdateCalendarEventRequest): Promise<CalendarEvent>;
client.university.calendars.deleteEvent(uniqueId: string): Promise<void>;
```

### TypeScript Types

```typescript
import type {
  Availability,
  CalendarEvent,
  CreateAvailabilityRequest,
  UpdateAvailabilityRequest,
  BulkUpdateAvailabilityRequest,
  CreateCalendarEventRequest,
  UpdateCalendarEventRequest,
  ListCalendarEventsParams,
} from '@23blocks/block-university';
```

### React Hook

```typescript
import { useUniversityBlock } from '@23blocks/react';

function MyComponent() {
  const { client } = useUniversityBlock();
  const result = await client.university.calendars.listEvents();
}
```
