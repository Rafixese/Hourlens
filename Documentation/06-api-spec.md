# 06 · API spec

> **Status: approved** · Last updated: 2026-10-24
>
> The REST API between the React app and the .NET backend. **The generated OpenAPI document is the source of truth** for exact request and response schemas (04, A6). This document lists the endpoints, conventions and key examples.

## 1. Conventions

| # | Rule |
|---|------|
| P2 | Base path **`/api/v1`** |
| P3 | Plural nouns in kebab-case (`/time-entries`), nested where ownership is natural (`/clients/{id}/projects`). JSON fields in **camelCase** |
| P4 | Errors use **Problem Details** (RFC 9457) with per-field validation errors |
| P5 | Dates `"2026-10-24"`, times `"08:00:00"`, UTC moments ISO 8601 with `Z` (`"2026-10-24T06:00:00Z"`). Durations of entries and totals in **seconds** (integers); the frontend formats them as `7:30`. Quota and time off hour amounts stay in **minutes** (05, M7) |
| P6 | Money is a **string** with two decimals (`"100.50"`) plus a separate `currency` field (`"PLN"`) |
| P7 | Lists of entries, time off and holidays are loaded **by date range** (`from`, `to`, inclusive). No page-based pagination |
| P9 | **All calculations happen on the server.** The frontend only displays results |
| P10 | Editable resources return a **`version`** field. Updates and deletes must send it back. If it changed, the server answers **409 Conflict** ("this changed, reload") |
| P11 | The TypeScript client and TanStack Query hooks are generated with **orval** |
| P12, P13 | Interactive API docs with **Scalar**, only in **development** |
| — | Authentication by **session cookie** (04, A7). All endpoints except setup, login and password reset require a logged-in user |
| — | IDs are UUID v7 strings (05, M1) |

### 1.1 Status codes

| Code | When |
|------|------|
| 200 OK | Read or update succeeded, body returned |
| 201 Created | Resource created, body returned |
| 204 No Content | Delete or action succeeded, no body |
| 400 Bad Request | Validation failed (Problem Details with `errors` per field) |
| 401 Unauthorized | Not logged in |
| 403 Forbidden | Logged in but not allowed (e.g. a user calling an admin endpoint) |
| 404 Not Found | Resource doesn't exist, **or belongs to another user** (P14). The API never reveals that another user's ID exists |
| 409 Conflict | `version` mismatch, duplicate name, overlapping period, or a timer already running |

## 2. Setup and authentication

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/setup/status` | `{ "setupRequired": true }` while no account exists | US-SETUP-01 |
| POST | `/setup` | Create the admin (`email`, `password`) and log in. 409 once an account exists | US-SETUP-01 |
| POST | `/auth/login` | `email`, `password`, `rememberMe`. Sets the session cookie | US-AUTH-01, 02 |
| POST | `/auth/logout` | Ends the session on this device | US-AUTH-05 |
| GET | `/auth/me` | Current user: `id`, `email`, `role` | — |
| POST | `/auth/change-password` | `currentPassword`, `newPassword` | US-AUTH-03 |
| POST | `/auth/forgot-password` | `email`. Sends a reset link if SMTP is configured. Always answers 204 so it doesn't reveal which emails exist | US-AUTH-04 |
| POST | `/auth/reset-password` | `email`, `token`, `newPassword` | US-AUTH-04 |

## 3. Admin (role `admin` only)

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/admin/users` | List accounts: `id`, `email`, `role`, `createdAt`. No user data | US-ADMIN-05 |
| POST | `/admin/users` | `email`, `password` | US-ADMIN-01 |
| DELETE | `/admin/users/{id}` | Delete the account and all its data | US-ADMIN-02 |
| POST | `/admin/users/{id}/reset-password` | `newPassword` | US-ADMIN-03 |
| GET | `/admin/smtp` | Current SMTP settings. The password is never returned, only `passwordSet: true/false` | US-ADMIN-04 |
| PUT | `/admin/smtp` | Save SMTP settings | US-ADMIN-04 |
| POST | `/admin/smtp/test` | `to`. Sends a test email | US-ADMIN-04 |

## 4. User settings

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/me/settings` | `timeZone`, `theme`, `workdays`, `holidayProviderCountry`, `version` | US-UI-01, 02, US-CAL-02 |
| PUT | `/me/settings` | Update any of the above | |

## 5. Clients, projects and tasks

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/clients?includeArchived=false` | List clients | |
| POST | `/clients` | `name`, `rateMode`, `roundingMode`, `roundingStepMinutes`. Also creates the "Default" project | US-ORG-01 |
| GET | `/clients/{id}` | One client | |
| PUT | `/clients/{id}` | Rename, change rate mode or rounding | US-ORG-01, US-ENTRY-04 |
| DELETE | `/clients/{id}` | Delete with everything inside | US-ORG-03 |
| POST | `/clients/{id}/archive` · `/unarchive` | Archive or restore | US-ORG-03b |
| GET | `/clients/{id}/projects?includeArchived=false` | List projects of a client | |
| POST | `/clients/{id}/projects` | `name` | US-ORG-02 |
| PUT | `/projects/{id}` | Rename | US-ORG-02 |
| DELETE | `/projects/{id}` | Delete with everything inside | US-ORG-03 |
| POST | `/projects/{id}/archive` · `/unarchive` | Archive or restore | US-ORG-03b |
| GET | `/projects/{id}/tasks?search=` | Task titles for autocomplete in the timer and entry forms | US-ORG-04 |
| PUT | `/tasks/{id}` | `title`. Renames the task; merges if the title already exists in the project | US-ORG-06 |

There is no "create task" endpoint. Tasks are created from entry titles (03, section 3).

## 6. Timer

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/timer` | The running timer, or 204 if none | US-TIMER-03 |
| POST | `/timer/start` | `projectId`, `title`, `isBillable`. 409 if a timer is already running | US-TIMER-01, 02 |
| POST | `/timer/stop` | `version`. Saves the entry, applies the client's rounding | US-TIMER-04 |
| PUT | `/timer` | Change title, project or billable flag of the running timer without stopping it (P16) | US-TIMER-01 |

## 7. Time entries

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/time-entries?from=&to=` | Entries in a date range, with `clientId`, `projectId`, `taskTitle`, `durationSeconds` | US-ENTRY-05 |
| POST | `/time-entries` | `projectId`, `title`, `date`, `startTime`, `endTime`, `isBillable`. For "start time + duration", the **frontend** calculates `endTime` (P15) | US-ENTRY-01 |
| PUT | `/time-entries/{id}` | Edit any field. Changing `title` moves the entry to another task | US-ENTRY-03, US-ORG-05 |
| DELETE | `/time-entries/{id}?version=` | Delete. Removes the task if it was the last entry | US-ENTRY-03 |
| GET | `/time-entries/defaults` | `{ "isBillable": true }`, the value from the user's last entry | US-TIMER-01 |

## 8. Rates, quotas, holidays and time off

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/rates/default` | The user's default rate periods | US-RATE-01 |
| POST | `/rates/default` | `validFrom`, `hourlyRate`, `currency`. A past `validFrom` re-rates entries from that date | US-RATE-03, 04 |
| GET | `/clients/{id}/rates` | The client's custom rate periods | US-RATE-02 |
| POST | `/clients/{id}/rates` | Same body as above | US-RATE-02, 03, 04 |
| PUT / DELETE | `/rate-periods/{id}` | Edit or remove a period | |
| GET | `/clients/{id}/quotas` | Quota periods | US-CAL-01 |
| POST | `/clients/{id}/quotas` | `validFrom`, `validTo`, `quotaType`, `minutes`. 409 if it overlaps | US-CAL-01 |
| PUT / DELETE | `/quota-periods/{id}` | Edit or remove | |
| GET | `/holidays?year=` | The user's holidays for a year | US-CAL-03 |
| POST | `/holidays` | `date`, `name` | US-CAL-03 |
| PUT / DELETE | `/holidays/{id}` | Edit or remove | US-CAL-04 |
| POST | `/holidays/import` | `year`. Loads public holidays from the provider for the country in settings | US-CAL-04 |
| GET | `/time-off?from=&to=&clientId=` | Time off in a range | US-OFF-01 |
| POST | `/time-off` | `clientId`, `type`, `startDate`, `endDate`, `isPaid`, `payMode`, value field, `baseMinutes`. 409 if it overlaps | US-OFF-01 |
| PUT / DELETE | `/time-off/{id}` | Edit or remove | US-OFF-01 |

## 9. Reports

All read-only, all calculated on the server (P9). Money totals are always grouped **per currency**.

| Method | Path | Purpose | Story |
|--------|------|---------|-------|
| GET | `/reports/dashboard?date=` | Daily and monthly quota progress, earnings today and this month, both predictions, current month split by day | US-REP-01 |
| GET | `/reports/summary?from=&to=&groupBy=&clientId=&projectId=&billable=` | Totals by `day`, `month` or `year`, including time off lines | US-REP-02 |
| GET | `/reports/quota?month=&clientId=` | Day-by-day quota report: expected, worked, **overtime**, time off, balance. Per client and total | US-CAL-05 |
| GET | `/reports/salary?month=` | Last month, this month so far, predicted (normal pace and catch-up) | US-REP-02 |
| GET | `/reports/tasks?from=&to=&clientId=&projectId=&sort=` | Task report | US-ORG-07 |

The period presets (this week, last month and so on) are turned into `from` / `to` dates by the frontend, using the user's time zone and Monday as week start.

## 10. Live updates (SignalR)

- Hub: **`/hubs/updates`**, cookie-authenticated. Each connection joins its user's group.
- After every change (04, A18) the server sends:

```json
{ "entity": "timeEntry", "id": "0192f1c4-…", "action": "updated" }
```

- `entity`: `timer`, `timeEntry`, `task`, `client`, `project`, `ratePeriod`, `quotaPeriod`, `holiday`, `timeOff`, `settings`.
- `action`: `created`, `updated`, `deleted`.
- The frontend invalidates the matching TanStack Query caches and refetches.

Outside the API: `/health` and `/metrics` (04, section 8).

## 11. Examples

**Start the timer**

```http
POST /api/v1/timer/start
{ "projectId": "0192f1c4-…", "title": "Invoice module", "isBillable": true }
```
```http
201 Created
{
  "id": "0192f1d0-…", "projectId": "0192f1c4-…", "taskId": "0192f1cf-…",
  "title": "Invoice module", "date": "2026-10-24", "startTime": "09:00:12",
  "endTime": null, "startedAtUtc": "2026-10-24T07:00:12Z",
  "isBillable": true, "version": "73519"
}
```

**Edit conflict**

```http
409 Conflict
{
  "type": "https://hourlens/errors/version-conflict",
  "title": "This entry was changed on another device. Reload and try again.",
  "status": 409
}
```

**Validation error**

```http
400 Bad Request
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": { "endTime": ["End time must be after start time."] }
}
```

**Money in a report**

```json
{ "earnings": [ { "amount": "12450.00", "currency": "PLN" }, { "amount": "800.00", "currency": "EUR" } ] }
```

## 12. Open questions

None. All questions were answered on 2026-10-24.
