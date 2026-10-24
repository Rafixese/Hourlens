# 08 · Reports and analytics

> **Status: approved** · Last updated: 2026-10-24
>
> What each tab of the **Insights** screen ([07](07-ui-ux.md), section 4) shows and how its numbers are produced. Calculation rules come from [03 Domain model](03-domain-model.md), section 4. Endpoints are in [06 API spec](06-api-spec.md), section 9. All decisions below were made by the owner.

## 1. General rules

| # | Rule |
|---|------|
| R14 | Reports are **calculated live** on every request from the stored data. Nothing is pre-computed or cached in the database |
| — | All maths runs on the **server** (06, P9). The frontend formats and draws |
| R3 | The **running timer counts live**: today's hours and earnings include it, and the frontend ticks it up between refreshes |
| R12 | **Money rounding:** minutes are totalled per rate period, multiplied by the hourly rate, and rounded to 2 decimals **once per total**. Individual entries are never rounded to money first |
| — | Money is always shown **per currency**, never converted (03, 4.7) |
| R13 | A client **without a rate** earns 0 and shows a **"Set a rate"** hint linking to its Rates tab |
| R11 | **Hours** are shown clock-style, `125:00`, in all Insights tables, tiles and charts. Decimal hours (`7.5 h`) are used only where decimals read better, such as quota settings |
| R4 | No comparison with previous periods in the MVP |
| — | Archived clients and projects are included (02, US-ORG-03b) |
| — | Weeks start on Monday. Period presets are turned into dates by the frontend in the user's time zone (06, section 9) |

## 2. Overview tab (default)

**Period:** the month picked in the selector, default the current month (R1).

| Element | Contents |
|---------|----------|
| **Today** tile | Hours today vs today's expected hours, with a progress bar. Shown only for the current month |
| **Month quota** tile | Hours counted vs required hours for the month, and the balance so far ("2:30 below quota so far") |
| **Earned** tile | Earnings for the month, one line per currency |
| **Predicted salary** tile | Normal pace, with "if you catch up" below (03, 4.8). **Only for the current month** (R1); hidden for past months |
| **By day** chart | Stacked bars, one segment per client (auto colours, 07 U10), dashed daily quota line. Future days of the current month drawn as outlines (R2) |
| **By client** table | Client, quota, worked, balance, earned, predicted, plus a total row (per currency) |

- **Clicking a bar** opens the **Summary** tab filtered to that day (R2).
- Clicking a client row opens the **Quota** tab for that client.
- Clients without a quota show "No quota" and no balance or prediction (03, 4.8).

## 3. Summary tab

**Filters:** period preset or custom range, client, project, billable / non-billable / all (02, US-REP-02).

**Grouping (R5):** follows the period automatically and can be overridden.

| Period | Default grouping |
|--------|------------------|
| Week, month, custom ≤ 31 days | By day |
| Year, custom > 31 days | By month |
| Several years | By year |

**Breakdown (R6):** none, by client or by project. With a breakdown, the chart is a **stacked bar chart** and the table gets one row per client or project inside each period.

**Table columns (R7):** period, hours, billable, non-billable, time off, earnings (one column per currency).

- Time off appears as its own line (vacation or sick) and is included in the totals (03, 4.6).

## 4. Quota tab

**Filters:** month, client selector with **"All clients"** as the default (R9).

**One row per day of the month (R8):**

| Column | Meaning |
|--------|---------|
| Date | `24.10.2026`, with weekday. Non-workdays and holidays are marked |
| Expected | Expected hours for that day (03, 4.4). 0 on non-workdays, holidays and time off days |
| Worked | Hours counted towards the quota (billable + non-billable) |
| Overtime | Hours on non-workdays and holidays, plus hours above expected on workdays (03, 4.5). **This is the only place overtime is shown** |
| Time off | Vacation or sick, and its value if paid |
| Day balance | Worked − expected |
| Running balance | Sum of day balances from the 1st of the month |

- The footer shows the month totals and the final balance. The balance doesn't carry over to the next month (03, 4.5).
- With "All clients", only clients with a quota are included, and each day's values are summed across them.

## 5. Salary tab

| Element | Contents |
|---------|----------|
| **Last month** | Salary for the previous month, per currency |
| **This month so far** | Salary from the 1st until now, including the running timer |
| **Predicted** | Normal pace and catch-up (03, 4.8) |
| **12-month chart** (R10) | Salary per month for the last 12 months, one series per currency |

- Salary = earnings + value of paid time off (03, 4.7).
- A breakdown table below lists each client's part of each figure.

## 6. Tasks tab

As specified in 02, US-ORG-07:

- Table of tasks: client, project, total hours, billable, non-billable, earnings (per currency), last worked date.
- Filters: period, client, project. Sort by any column, default most hours first.
- Clicking a task shows its entries. Tasks can be renamed from here (merging if the title exists, 02, US-ORG-06).

## 7. Implementation notes

- Calculations live in **Hourlens.Domain** (04, 3.1) as pure functions over entries, rate periods, quota periods, holidays, workdays and time off. They have no database access, so they're easy to unit-test with the examples from 03.
- The Application layer loads the data for the requested range in a few queries (entries by `(user_id, entry_date)`, 05) and passes it to the Domain functions.
- Each Insights tab is one API call (06, section 9). When a SignalR "changed" message arrives (06, section 10), the open tab refetches.

## 8. Details confirmed

Filled in by Claude and confirmed by the owner on 2026-10-24:

- **C1.** Summary grouping thresholds: by day up to 31 days, by month above that, by year for ranges of several years.
- **C2.** Clicking a client row on the Overview opens the Quota tab for that client.
- **C3.** Quota tab with "All clients": only clients with a quota are included, summed per day.
- **C4.** Salary tab: a breakdown table with each client's share below the figures.
