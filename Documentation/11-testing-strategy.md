# 11 · Testing strategy

> **Status: draft for review** · Last updated: 2026-10-24
>
> What is tested, where, and with which tools. All decisions below were made by the owner. Tests are written by the owner as part of each issue ([10](10-development-guide.md), 1.1).

## 1. The pyramid (T1)

| Level | What | Tools | Speed |
|-------|------|-------|-------|
| **Unit: Domain** | Calculation rules and entity behaviour from 03 | xUnit + Shouldly | Milliseconds, the bulk of the suite |
| **Unit: Application** | Use cases with fake repositories and a fake clock | xUnit + Shouldly | Fast |
| **Integration: API** | Real HTTP requests to the app against a real PostgreSQL | xUnit + `WebApplicationFactory` + **Testcontainers** | Seconds |
| **Unit: Frontend** | Components, hooks, formatting helpers | **Vitest** + **React Testing Library** | Fast |
| **End-to-end** | A handful of key flows in a real browser | **Playwright** | Slowest, few tests |

Test projects follow the backend layout in 04 (3.1): `Hourlens.Domain.Tests`, `Hourlens.Application.Tests`, `Hourlens.Api.Tests`.

## 2. Domain tests: the worked examples (T2, T9)

- **Every worked example in 03** becomes a **named unit test**, for example:
  - `DailyQuota_35HoursInFiveWorkdayWeek_Is2Point5HoursShort`
  - `MonthlyQuota_160HoursOver20Workdays_Expects8HoursPerDay`
  - `MonthlyQuota_ContractStartsMidMonth_ProratesTo64Hours`
  - `Overtime_SaturdayHours_IncludedInBalance_Plus5`
  - `TimeOff_FridayToTuesday_CountsThreeDays`
  - `RateChange_FromJanuary_DecemberKeepsOldRate`
  - `Prediction_CatchUp_AddsMissingHours`
- Domain rules are written **test-first** (T9): write the failing test from the 03 example, then the code that makes it pass. This is the main learning loop of the project.
- Edge cases worth their own tests: holidays on weekends, time off over holidays, a rate change mid-month, an entry ending at `24:00`, a month with no quota period, clients in different currencies.

## 3. Time in tests (T8)

- All code that needs "now" takes .NET's **`TimeProvider`**, never `DateTime.Now` / `DateTime.UtcNow` directly.
- Tests use a **fake time provider** set to a fixed moment, so "today", the midnight timer stop and predictions can be tested on any date and time zone.
- The background jobs (04, 3.4) are tested by advancing the fake clock.

## 4. API integration tests (T3, T5)

- Run the real app in memory (`WebApplicationFactory`) against a **throwaway PostgreSQL container** started by **Testcontainers** for the test run. Migrations are applied to it, so constraints from 05 (exclusion constraints, unique indexes, one running timer) are tested for real.
- Each test creates its own users and data, so tests don't depend on each other.
- **Data isolation for every endpoint (T3):** a shared test pattern creates user A's data, logs in as user B, and checks that every read, update and delete answers **404** (06, P14; 09, section 5). A new endpoint isn't done until it's covered.
- Also covered here: authentication and lockout, anti-forgery, `version` conflicts (409), validation errors (400), admin-only endpoints (403).

## 5. Frontend tests (T6)

- **Vitest** + **React Testing Library** for components and hooks: forms and validation, the timer display, number/date/money formatting (07, section 8), period presets → date ranges.
- Tests interact like a user (find by label or role, type, click), not by implementation details.
- API calls are mocked at the network level in component tests.

## 6. End-to-end tests (T7)

A handful of **Playwright** tests against the production Docker image with a fresh database:

1. First run: setup screen → create admin → logged in
2. Admin creates a user → that user logs in and changes password
3. Create a client → start a timer → stop it → the entry appears in the list
4. Add manual entries for a week → Insights shows the right totals
5. Live update: start a timer in one browser tab → it appears in a second tab

E2E tests stay few. A rule that can be tested at a lower level is tested there.

## 7. Coverage and CI (T10, T11)

- **Coverage target:** about **90% for Domain**. No hard target elsewhere.
- Coverage is **reported in CI** but **doesn't block** a merge.
- **All tests run on every PR**, end-to-end included, as long as the full CI run stays **under ~10 minutes**. If it grows beyond that, E2E moves to merges into `main` only.
- A PR can't be merged with failing tests (10, G7). Tests are never skipped or disabled to get green.
