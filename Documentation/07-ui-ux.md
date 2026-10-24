# 07 · UI/UX

> **Status: approved** · Last updated: 2026-10-24
>
> Screens, navigation and the main flows, built on the requirements in [02](02-requirements.md) and the UI stack in [04](04-architecture.md) (Mantine, Mantine Charts). Mockups: see 9.4.

## 1. Principles

- **Tracking first.** The app opens on the Time tracker, and the timer is visible on every page.
- **Works on a phone** for every feature (02, US-UI-03).
- **Light and dark** themes, chosen by the user (02, US-UI-01).
- English UI (02, NFR-05).

## 2. Navigation

- **Desktop:** a left **sidebar** with the screens below. The **running timer sits in the top bar** on every page.
- **Phone:** the sidebar becomes a **bottom tab bar**. The timer stays visible at the top.
- After login, the app opens the **Time tracker**.

### 2.1 Screens

| Screen | Contents |
|--------|----------|
| **Time tracker** | Timer bar + entries grouped by day, with daily and weekly totals |
| **Insights** | Dashboard and reports merged into one screen (U2), see section 4 |
| **Clients** | List of clients → client page |
| **Time off** | Vacation and sick days across clients |
| **Settings** | Password, time zone, theme, workdays, holidays, default rate |
| **Admin** | Users, email server. Only for the admin |

```
Desktop                                       Phone
┌──────────┬──────────────────────────────┐   ┌──────────────────────┐
│ Hourlens │ ▶ Invoice module · Acme 01:12:05 ■ │ ▶ Invoice… 01:12:05 ■│
│          ├──────────────────────────────┤   ├──────────────────────┤
│ Tracker  │                              │   │                      │
│ Insights │        page content          │   │     page content     │
│          │                              │   │                      │
│ Clients  │                              │   ├──────────────────────┤
│ Time off │                              │   │ Trk  Insights  More  │
│ Settings │                              │   └──────────────────────┘
│ Admin    │                              │
└──────────┴──────────────────────────────┘
```

## 3. Time tracker

```
┌───────────────────────────────────────────────────────────────┐
│ [What are you working on?        ] [Acme / Default ▾] [$] [▶] │
├───────────────────────────────────────────────────────────────┤
│ This week: 31:15                                              │
│ Today · Fri 24.10                                      6:45   │
│   Invoice module      Acme / Default    $  09:00–12:30  3:30 ▶│
│   Code review         Acme / Backend       13:00–16:15  3:15 ▶│
│ Thu 23.10                                              7:30   │
│   …                                                           │
└───────────────────────────────────────────────────────────────┘
```

- **Title field with suggestions from all projects**, shown as "title · client / project". Picking a suggestion fills in the client and project (U5).
- **Client / project picker.** If only one option exists, it is preselected (02, US-TIMER-01).
- **Billable toggle** (`$`), defaulting to the last entry's value (02, US-TIMER-01).
- **Start** needs a title and a project (02, US-TIMER-01).
- While running, the bar shows the elapsed time and a **Stop** button. Title, project and billable can be changed without stopping (06, P16).
- **Manual entry:** a switch on the bar changes it to date + start + end, or date + start + duration (02, US-ENTRY-01).
- **Entry list:** grouped by day with daily totals and the week total.
  - **Desktop:** fields are edited **inline**. **Phone:** tapping an entry opens a **dialog** (U6).
  - Each entry has a **▶ Continue** button that starts a timer with the same title and project (U7).
  - Deleting asks for confirmation.
- **No weekly grid view** (one row per task, one column per day) in the MVP (U8). The daily list is the only way to view and enter time.

## 4. Insights (dashboard + reports)

One screen with tabs (U2):

| Tab | Contents | Source |
|-----|----------|--------|
| **Overview** (default) | Daily and monthly quota progress, earnings today and this month, predictions (normal pace and catch-up), current month split by day | 02, US-REP-01 |
| **Summary** | Totals by day, month or year, with period presets and client / project / billable filters | 02, US-REP-02 |
| **Quota** | Day-by-day quota report with expected, worked, overtime, time off and balance, per client and total | 02, US-CAL-05 |
| **Salary** | Last month, this month so far, predicted | 02, US-REP-02 |
| **Tasks** | Task report | 02, US-ORG-07 |

- The Overview is mainly **charts**. Clicking a chart opens the matching tab, filtered to what was clicked (02, US-REP-01).
- Money is always shown per currency.

## 5. Clients

- **Client list:** active clients, with a switch to show archived ones.
- **Client page** with tabs (U9):
  - **Projects:** list, add, rename, archive, delete
  - **Rates:** default or custom rate, rate history, add a rate change "from date", optional re-rate of past entries
  - **Quota:** quota periods (daily or monthly, from/to dates)
  - **Rounding:** mode and step
  - **Time off:** this client's vacation and sick days
- Deleting a client or project shows what will be deleted and asks for confirmation (02, US-ORG-03).
- Client colours in charts are **assigned automatically** (U10).

## 6. Settings and admin

- **Settings:** change password, time zone, theme, workdays, holidays (manual list per year, or provider with a country, plus edits), default rate with history.
- **Admin:** users (create, reset password, delete with confirmation), email server (settings + test email).

## 7. First use

- **Setup screen** on first launch creates the admin (02, US-SETUP-01).
- **First-login guide** (U15) for every new user, short and skippable:
  1. default hourly rate and currency
  2. workdays (preset Monday–Friday)
  3. first client (with its "Default" project)
  4. optional quota for that client

## 8. Formats

| What | Format |
|------|--------|
| Clock | **24-hour**, `09:00` |
| Money | **Polish style**: `1 234,50 PLN` |
| Durations | Clock style everywhere: `7:30`, `125:00`. Decimal hours (`7.5 h`) only where decimals read better, such as quota settings (U13, refined in 08 R11) |
| Dates | `24.10.2026` (U12) |
| Week start | Monday (02, NFR-06) |

Not in the MVP: keyboard shortcuts (U14).

## 9. Visual identity

Chosen by the owner on 2026-10-24 from the [brand options board](https://claude.ai/artifact/R7W4yweUsxyFocdyf588tb). Feel: **strategic, premium, enterprise**. **Dark mode is the main look**; light mode is fully supported.

### 9.1 Logo: "Ring & hand" (option A)

- **Icon + wordmark.** The icon is a lens (a thick blue ring with a faint inner ring) with a single clock hand pointing to about 1 o'clock and a centre dot.
- **Wordmark:** "Hourlens" in **Sora SemiBold**, tight letter spacing (−0.03em).
- Small sizes (32 px and below) drop the inner ring and centre dot and use thicker strokes.
- Icon source (48 × 48, dark background):

```svg
<svg viewBox="0 0 48 48" xmlns="http://www.w3.org/2000/svg">
  <circle cx="24" cy="24" r="19" fill="none" stroke="#3B82F6" stroke-width="4"/>
  <circle cx="24" cy="24" r="11" fill="none" stroke="#3B82F6" stroke-opacity="0.4" stroke-width="2"/>
  <line x1="24" y1="24" x2="31" y2="15" stroke="#E6ECF8" stroke-width="3" stroke-linecap="round"/>
  <circle cx="24" cy="24" r="2.6" fill="#E6ECF8"/>
</svg>
```

On light backgrounds the ring is `#2563EB` and the hand and dot are `#0F172A`.

### 9.2 Palette: "Midnight Cobalt" (option 1)

| Token | Dark (main) | Light |
|-------|-------------|-------|
| Background | `#0B1020` | `#F7F9FC` |
| Surface (cards, bars) | `#121A2E` | `#FFFFFF` |
| Border | `#22304D` | `#E2E8F0` |
| Text | `#E6ECF8` | `#0F172A` |
| Muted text | `#8B9BB8` | `#5B6B85` |
| Primary (buttons, white text on it) | `#2563EB` | `#2563EB` |
| Accent (links, highlights) | `#60A5FA` | `#2563EB` |
| Chart 1 | `#3B82F6` | `#2563EB` |
| Chart 2 | `#22D3EE` | `#0891B2` |

- Mantine is themed with these tokens: `primaryColor` built around `#2563EB`, `defaultColorScheme: 'dark'`.
- More chart colours for many clients are derived from this palette when implementing.

### 9.3 Typography

- **Sora** for the wordmark and large headings.
- **Inter** for all interface text. Numbers in timers and tables use tabular figures.

### 9.4 Mockups

Desktop mockups of the **Time tracker** and the **Insights overview**, in the chosen logo and palette, are on the [design board](https://claude.ai/artifact/R7W4yweUsxyFocdyf588tb) next to the logo and palette options. The data in them is sample data. Approved by the owner on 2026-10-24.

## 10. Open questions

None. All questions were answered on 2026-10-24.
