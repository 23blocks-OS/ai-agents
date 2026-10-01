---
name: 23blocks-company-teams-api
description: "Company Block teams: CRUD, join, members, a user's teams. Use for org-chart teams; access-control teams are 23blocks-auth-teams-api."
allowed-tools: Read, Write, Bash, Grep, Glob
metadata:
  author: 23blocks
  version: "1.1"
  verified-by: 23blocks-api-company
  verified-date: "2026-05-18"
---

# Teams API

Complete API reference for 23blocks company team management with membership.

## Setup

Send requests to `$BLOCKS_API_URL` (this block: `https://company.api.us.23blocks.com`) with two headers:

- `X-API-KEY: $BLOCKS_API_KEY`: static tenant routing key (`pk_live_sh_...`) from the company config, the same for every block. It is not the key used to register an agent identity.
- `Authorization: Bearer $BLOCKS_AUTH_TOKEN`: the caller's identity and scopes; it expires. Get it from login (`/auth/sign_in`), from the user, or, for an agent, with `aid-token.sh -a https://auth.api.us.23blocks.com/<tenant> -q` (first-time setup: the `23blocks-auth-agent-identity-api` skill).

## Endpoints

> Full endpoint documentation: [ENDPOINTS.md](ENDPOINTS.md)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/teams` | List teams with pagination |
| GET | `/teams/:unique_id` | Get team by ID |
| POST | `/teams` | Create new team |
| PUT | `/teams/:unique_id` | Update team |
| DELETE | `/teams/:unique_id` | Delete team |
| GET | `/teams/:unique_id/members` | List team members |
| POST | `/teams/:unique_id/join` | Add authenticated user to team |
| DELETE | `/teams/:unique_id/members/:user_unique_id` | Remove member from team |
| GET | `/users/:unique_id/teams` | List all teams a user belongs to |

---

## Data Models

### Team
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Unique identifier |
| `name` | string | Team name |
| `description` | string | Team description |
| `department_id` | uuid | Associated department ID |
| `team_lead_id` | uuid | Team lead user ID |
| `status` | string | active, inactive |
| `created_at` | timestamp | Creation time |
| `updated_at` | timestamp | Last update |

### TeamMember
| Field | Type | Description |
|-------|------|-------------|
| `unique_id` | uuid | Membership identifier |
| `user_unique_id` | uuid | User identifier |
| `email` | string | Member email |
| `first_name` | string | First name |
| `last_name` | string | Last name |
| `role` | string | Role within team (lead, member) |
| `joined_at` | timestamp | When member joined |

---

## Errors

JSON:API error objects, e.g. `{"errors":[{"status":"409","source":{"pointer":"/teams/:unique_id/join"},"code":"conflict","title":"Already a Member","detail":"This user is already a member of the team."}]}`.

## SDK (TypeScript)

For web apps, prefer the SDK to raw calls: `npm install @23blocks/block-company`, then create a client with `create23BlocksClient({ authToken, apiKey, apiUrl })` from `@23blocks/sdk`; in React, `const { client } = useCompanyBlock()` from `@23blocks/react`.

### Available Methods

```typescript
// Teams — client.company.teams
client.company.teams.list(params?: ListTeamsParams): Promise<PageResult<Team>>;
client.company.teams.get(uniqueId: string): Promise<Team>;
client.company.teams.create(data: CreateTeamRequest): Promise<Team>;
client.company.teams.update(uniqueId: string, data: UpdateTeamRequest): Promise<Team>;
client.company.teams.delete(uniqueId: string): Promise<void>;
client.company.teams.listByDepartment(departmentUniqueId: string): Promise<Team[]>;
```

### TypeScript Types

```typescript
import type {
  Team,
  CreateTeamRequest,
  UpdateTeamRequest,
  ListTeamsParams,
} from '@23blocks/block-company';
```
