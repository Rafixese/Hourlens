# 13 · Roadmap

> **Status: approved** · Last updated: 2026-10-24
>
> The path from an empty repo to the MVP defined in [01](01-vision-and-scope.md). Built by the owner with Claude Code, about **5–10 hours a week**. The roadmap gives the **order** only, no dates (RM2).

## 1. Approach

| # | Decision |
|---|----------|
| RM3 | Each milestone is a **working vertical slice**: UI, API, database and tests for its features, usable end to end |
| RM5 | The owner starts using Hourlens **after the MVP is complete** (M7). Until then the database can be reset freely, and migrations may be squashed into one initial migration before 1.0 |
| RM6 | Each **task** below becomes a **GitHub issue**, a unit of work for one or a few sessions. User story IDs from 02 are referenced in each issue |
| RM7 | Each finished milestone gets a **git tag**, `v0.1.0` (M0) to `v1.0.0` (M7). Images are still only published as `latest` (04, A19) |
| RM9 | No separate `adr/` folder. Decisions live in the tables in 04–09. An ADR is written only when an approved decision changes later |

**Definition of done for every task:** code + tests (11 Testing strategy) pass in CI, the relevant doc is updated if behaviour changed, and the PR is merged into `main`.

## 2. Milestones

### M0 · Foundation → `v0.1.0`

**Goal:** an empty but real app: you can install it, set it up, log in, and the admin can manage users.

Stories: US-SETUP-01, US-AUTH-01 to 05, US-ADMIN-01 to 03, US-ADMIN-05, US-UI-01, US-UI-02, NFR-01 to 06.

Tasks:
1. Monorepo skeleton: `backend/`, `frontend/`, `deploy/`, solution with Domain / Application / Infrastructure / Api projects and test projects (04, 3.1)
2. Frontend skeleton: Vite + React + TypeScript, Mantine themed with Midnight Cobalt (07, 9.2), bundled Sora and Inter fonts (09, S11), React Router, TanStack Query
3. EF Core + PostgreSQL, snake_case naming, migrations on start, Identity tables (05, 3.1)
4. OpenAPI + Scalar (dev only) + orval client generation (06)
5. Dockerfile (React build served by .NET) and `docker-compose.yml` with `app` + `db`, `.env.example` (04, 6)
6. GitHub Actions: build and test on every PR; publish `latest` image to GHCR on merge to `main`; Dependabot (04, A14/A19; 09, S16)
7. Serilog JSON logs, `/health`, `/metrics` (04, 8)
8. Setup screen and first admin (02, US-SETUP-01)
9. Login, logout, remember me, cookie + anti-forgery, lockout, rate limiting (09, 3–4)
10. Change password (02, US-AUTH-03)
11. Admin: list, create, delete users, reset passwords (02, US-ADMIN-01 to 03)
12. App shell: sidebar / bottom tabs, top bar placeholder, light/dark theme, logo (07, 2 and 9)
13. User settings: time zone detected on first login, theme (05, 3.2)
14. Security headers, forwarded headers with `HOURLENS_BIND` / `HOURLENS_TRUSTED_PROXIES` (09, 6)

**Done when:** `docker compose up` on a clean server gives a setup screen; the admin can log in, create a user, and that user can log in and change their password.

### M1 · Tracking → `v0.2.0`

**Goal:** track time: clients, projects, tasks, timer and manual entries.

Stories: US-ORG-01 to 06, US-TIMER-01 to 05, US-ENTRY-01 to 05.

Tasks:
1. Clients: create (with "Default" project), rename, archive/unarchive, delete with confirmation (02, US-ORG-01/03)
2. Projects: create, rename, archive/unarchive, delete (02, US-ORG-02/03)
3. Tasks from titles: case-insensitive match per project, auto-delete when empty, rename and merge (02, US-ORG-04 to 06)
4. Timer API: start, stop, edit running timer, one-timer rule (06, section 6)
5. Midnight auto-stop background job (04, 3.4)
6. Manual entries: create (start/end or start/duration), edit, delete, overlap allowed (02, US-ENTRY-01 to 03)
7. Rounding per client, applied on save (02, US-ENTRY-04)
8. Billable flag with "last used" default (02, US-TIMER-01)
9. Time tracker screen: timer bar with title suggestions from all projects, client/project picker, entries by day with totals, inline edit (desktop) / dialog (phone), continue button (07, 3)
10. Timer in the top bar on every page (07, 2)
11. SignalR hub and live updates for every change (04, 3.3; 06, section 10)
12. Optimistic concurrency with `version` and 409 handling in the UI (06, P10)
13. Client page shell with Projects tab (07, 5)

**Done when:** you can track a full week with the timer and manual entries, on desktop and phone, and see changes on a second device instantly.

### M2 · Money → `v0.3.0`

**Goal:** know what the time is worth.

Stories: US-RATE-01 to 04, US-REP-02 (partly).

Tasks:
1. Default rate with history and currency (02, US-RATE-01/03)
2. Client rate mode: default or custom, with history (02, US-RATE-02/03)
3. Re-rate past entries from a date (02, US-RATE-04; 03, 4.2)
4. Domain: earnings calculation with money rounding once per total (03, 4.7; 08, R12)
5. Insights screen shell with tabs (07, 4)
6. Summary tab: period presets, grouping, breakdown, filters, table and stacked chart (08, 3)
7. "Set a rate" hint for clients without a rate (08, R13)
8. Client page: Rates and Rounding tabs (07, 5)

**Done when:** a month of entries shows correct earnings per currency in Summary, and a rate change from a past date re-rates entries correctly.

### M3 · Calendar and quota → `v0.4.0`

**Goal:** see whether you're on track.

Stories: US-CAL-01 to 03, US-CAL-05.

Tasks:
1. Workdays setting (02, US-CAL-02)
2. Manual holidays per year (02, US-CAL-03)
3. Quota periods per client: daily or monthly, from/to dates, no overlaps (02, US-CAL-01; 05, 5.2)
4. Domain: quota workdays, expected hours, monthly spread and proration, required hours, balance, overtime (03, 4.3–4.5)
5. Quota tab: day-by-day table with running balance and overtime, client selector (08, 4)
6. Client page: Quota tab (07, 5)

**Done when:** the Quota tab matches the worked examples in 03 for daily and monthly quotas, holidays and a mid-month contract start.

### M4 · Time off → `v0.5.0`

**Goal:** vacations and sick days count correctly.

Stories: US-OFF-01.

Tasks:
1. Time off CRUD per client, full days, no overlaps (02, US-OFF-01; 05, 5.4)
2. Paid modes: percentage, fixed hours, fixed amount; base hours for clients without quota (03, 4.6)
3. Domain: time off reduces required hours, is valued for salary, skips non-workdays and holidays (03, 4.3/4.6)
4. Time off screen and client page tab (07, 2 and 5)
5. Time off lines in Summary and Quota tabs (08, 3–4)

**Done when:** a vacation week reduces required hours and pays the chosen amount, and a range over a weekend counts only workdays.

### M5 · Insights → `v0.6.0`

**Goal:** the full dashboard and salary picture.

Stories: US-REP-01, US-REP-02, US-ORG-07.

Tasks:
1. Domain: salary and both predictions (normal pace, catch-up), including today and the running timer (03, 4.8; 08, R3)
2. Overview tab: tiles, stacked by-day chart with quota line, by-client table, click-through to Summary and Quota (08, 2)
3. Salary tab: last month, this month, predicted, 12-month chart, client breakdown (08, 5)
4. Tasks tab: table, filters, sorting, entries, rename (08, 6)
5. Past-month view without predictions (08, R1)

**Done when:** the Overview for a sample month matches hand-calculated figures, including predictions.

### M6 · Finishing → `v0.7.0`

**Goal:** the remaining MVP features and a smooth first use.

Stories: US-CAL-04, US-AUTH-04, US-ADMIN-04, US-UI-03.

Tasks:
1. Holiday provider: country setting, Nager.Date import, yearly refresh job, editable results (02, US-CAL-04; 04, 3.4)
2. SMTP settings in the admin panel with encrypted password and test email (02, US-ADMIN-04; 05, 3.3)
3. Self-service "forgot password" by email (02, US-AUTH-04)
4. First-login guide (07, 7)
5. Phone pass over every screen (02, US-UI-03)
6. Empty states and error messages across the app

**Done when:** a new user goes from first login to tracked time with the guide, and every screen works on a phone.

### M7 · Release 1.0 → `v1.0.0`

**Goal:** ready for daily use on the homelab.

Tasks:
1. Security review against 09 (isolation tests for every endpoint, headers, rate limits, lockout)
2. Performance check with several years of sample data (02, NFR-03)
3. Squash migrations into one initial migration (RM5)
4. Write 12 Deployment and operations: install, Nginx config (including WebSockets), `.env`, updates, backup and restore
5. Backup and restore test on a real server
6. Final pass over all docs so they match the code

**Done when:** the definition of done in 01 (section 7) is met on the owner's server, and the owner starts using Hourlens daily.

## 3. After the MVP (not ordered)

Candidates, to be prioritised later (RM8): several simultaneous timers, two-factor login, SSO (OIDC), CSV/PDF/XLSX export, half-day and hourly time off, weekly grid view, comparison with previous periods, keyboard shortcuts, Clockify import, mobile app, more time off types.

## 4. Tracking on GitHub

- **RM10.** Issues are grouped into **GitHub Milestones M0–M7**, one per milestone above.
- **RM11.** The issues are created from the task lists above in [Rafixese/Hourlens](https://github.com/Rafixese/Hourlens).
