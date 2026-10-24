# 02 · Requirements

> **Status: approved** · Last updated: 2026-10-24
>
> Turns the approved [01 Vision and scope](01-vision-and-scope.md) into user stories with acceptance criteria. Only the project owner's decisions are written here.

## 1. Conventions

- Each requirement is a **user story**: *As a ⟨role⟩, I want ⟨goal⟩, so that ⟨reason⟩.*
- Acceptance criteria use **Given / When / Then**.
- IDs: `US-<area>-<nn>` (for example `US-TIME-01`). Non-functional requirements use `NFR-<nn>`.
- Roles: **Admin** and **User**, as defined in 01. "User" also covers the admin when they track their own time.

## 2. Installation and first run (SETUP)

**US-SETUP-01: Create the admin on first run**
*As the person installing Hourlens, I want a setup screen on first launch, so that I can create the admin account.*
- Given a fresh installation with no accounts, when I open the app, then I see a setup screen instead of the login page.
- When I enter an email and a password (8+ characters) and submit, then the admin account is created and I am logged in.
- Given an admin already exists, when anyone opens the setup URL, then the setup screen is not available.

## 3. Login and account (AUTH)

**US-AUTH-01: Log in with email and password**
*As a user, I want to log in with my email and password, so that I can reach my data.*
- Given valid credentials, when I log in, then I see my dashboard.
- Given wrong credentials, when I log in, then I see an error and stay on the login page.
- There is no sign-up page.

**US-AUTH-02: Remember me**
*As a user, I want a "remember me" option, so that I don't have to log in every time.*
- Given "remember me" is checked, when I close and reopen the browser, then I am still logged in for **30 days**.
- Given it is not checked, when I close the browser, then my session ends.

**US-AUTH-03: Change my password**
*As a user, I want to change my own password, so that I can keep my account secure.*
- When I enter my current password and a new one (8+ characters), then the password is changed.
- Users cannot change their own email or display name.

**US-AUTH-04: Forgotten password**
*As a user who forgot my password, I want a way back into my account.*
- The admin can always reset a user's password (US-ADMIN-03).
- If the admin has configured an email server, a **self-service "forgot password"** reset by email is available. Without an email server, only the admin can reset passwords.

**US-AUTH-05: Log out**
- When I log out, then my session ends on this device.

## 4. Admin panel (ADMIN)

**US-ADMIN-01: Create a user**
- Given I am the admin, when I enter an email and an initial password (8+ characters), then the account is created and the user can log in.

**US-ADMIN-02: Delete a user**
- Given I am the admin, when I delete a user and confirm, then the account and **all its data** are deleted.

**US-ADMIN-03: Reset a user's password**
- Given I am the admin, when I set a new password for a user, then they can log in with it.

**US-ADMIN-04: Configure the email server**
- Given I am the admin, I can enter the email (SMTP) server settings and send a test email.
- In the MVP, email is used **only** for self-service password reset.

**US-ADMIN-05: Admin privacy**
- The admin **cannot see** other users' clients, entries, time off or reports.

## 5. Clients, projects and tasks (ORG)

**US-ORG-01: Create a client**
- When I create a client with a **name**, then it is created together with a project named **"Default"**.
- A client has only a name in the MVP, plus its rate, currency and quota settings (sections 8 and 9).

**US-ORG-02: Create, rename and delete projects**
- A project has only a **name**.
- A project belongs to one client and **cannot be moved** to another client.
- I can rename any project, including "Default".

**US-ORG-03: Delete a client or project**
- I can delete a client or project even if it has time entries.
- Before deleting, I must **confirm**. Deleting removes **everything inside it** (projects, tasks, entries, and for a client also its time off, rates and quota).

**US-ORG-03b: Archive a client or project**
- I can **archive** a client or project to hide it while keeping its data.
- Archived clients and projects **disappear from the pickers** (timer, new entry).
- Their data **still appears in reports and the dashboard**.
- I can **unarchive** them at any time.

**US-ORG-04: Tasks are created from titles**
- When I start a timer or add an entry with a title, then the app looks for a task with the same title **in the selected project only**, ignoring letter case.
- If one exists, the entry is added to it. If not, a new task is created.

**US-ORG-05: Renaming an entry**
- When I change an entry's title, then the entry moves to the task with that title in the same project (ignoring letter case). If no such task exists, a **new task** is created.
- When a task has no entries left, then it is **deleted automatically**.

**US-ORG-06: Rename a task**
- I can rename a task directly. All its entries then show the new title.
- If I rename a task to a title that already exists in the same project (ignoring letter case), the two tasks are **merged** into one.
- I can also rename any single entry of a task, which moves it to a new or existing task (US-ORG-05).

**US-ORG-07: Task report**
*Proposed by Claude at the owner's request and confirmed by the owner.*
- A table of tasks with, for each task: **client**, **project**, **total hours**, **billable hours**, **non-billable hours**, **earnings** (per currency) and **last worked date**.
- Filters: **period** (same presets as other reports), **client** and **project**.
- Sorting by any column. Default: most hours first.
- Clicking a task shows its **entries**.
- From the report I can **rename** a task (US-ORG-06).

## 6. Live timer (TIMER)

**US-TIMER-01: Start a timer**
- To start, I must enter a **title** and pick a **client and project**. The timer cannot start without them.
- In the picker, if only one client or project is available, it is selected by default.
- The billable flag defaults to the value I chose on my **last entry**.

**US-TIMER-02: Only one timer**
- Given a timer is running, I cannot start a second one. (The design must allow several later, see 01.)

**US-TIMER-03: Timer runs on the server**
- The timer keeps running when I close the browser.
- I see the same running timer on every device where I'm logged in.

**US-TIMER-04: Stop a timer**
- When I stop the timer, then a time entry is saved with the start and end time.

**US-TIMER-05: Timer at midnight**
- A running timer **stops automatically at midnight** (in my saved time zone). No new entry is created for the next day. To keep working after midnight, I start a new timer.

## 7. Manual time entries (ENTRY)

**US-ENTRY-01: Add an entry**
- I enter a title, client, project and billable flag, plus either:
  - a **start and end time**, or
  - a **start time and duration**, and the end time is filled in for me.

**US-ENTRY-02: Overlapping entries**
- Entries **may overlap** in time.

**US-ENTRY-03: Edit and delete**
- I can edit or delete any of my entries, of any age.

**US-ENTRY-04: Rounding**
- Rounding is set **per client**. A new client starts with **no rounding**.
- Rounding mode: **none, down, nearest or up**.
- Rounding step: **1 minute, 15 minutes, 30 minutes or 1 hour**.
- Rounding is applied to **each entry** separately, not to daily totals.
- Rounding **changes the stored end time**. The start time is kept as entered.
  - *Example:* rounding up to 15 minutes, an entry 09:00–09:07 is saved as 09:00–09:15.
- If rounding would leave the entry **zero length or shorter**, the entry is **saved unrounded**.
  - *Example:* rounding down to 15 minutes, an entry 09:00–09:07 stays 09:00–09:07.
- Rounding is applied **when an entry is saved**: when the timer stops, or when an entry is added or edited.
- Changing a client's rounding setting **does not change entries already saved**.

**US-ENTRY-05: Entry list**
- I see my entries grouped by day, with daily and weekly totals. *(From 01, 3.2.)*

## 8. Rates and currency (RATE)

**US-RATE-01: Default rate**
- I set a default hourly rate with its currency.

**US-RATE-02: Client rate**
- For each client I choose **"use my default rate"** or a **custom rate** with its own currency.

**US-RATE-03: Change a rate from a date**
- When I change a rate (default or client), I pick the date it applies from. Entries before that date keep their old rate.

**US-RATE-04: Re-rate past entries**
- When changing a rate, I can optionally apply it to past entries starting from a date I pick.

## 9. Quota, workdays and holidays (CAL)

**US-CAL-01: Client quota**
- For a client, I can set **no quota**, a **daily quota** or a **monthly quota**.
- A quota can **change over time**, with the same rule as rates: a change applies **from a chosen date**, and I can optionally recalculate past months from a date I pick.
- A quota can have a **start and end date** (for example a contract starting mid-month).

**US-CAL-02: Workdays**
- I choose my workdays. Default: Monday to Friday.

**US-CAL-03: Holidays, manual**
- I add holidays by date, year by year.

**US-CAL-04: Holidays, external provider**
- I can switch on the external provider and choose a **country**. Its public holidays are loaded and I can edit them.

**US-CAL-05: Required hours and balance**
- For each client with a quota, the app calculates required hours from quota, workdays, holidays and time off.
- Billable and non-billable hours both count.
- A monthly quota is spread evenly across the month's workdays.
- Hours worked on a **holiday or non-workday**, and hours above the day's expected hours on a workday, count as **overtime**, paid at the **normal rate**.
- Overtime is shown **only in the monthly day-by-day quota report**, as hours above the quota. Time entries and total reports show the hours as worked. *(Clarified in 03.)*
- The balance is per month and does not carry over.

## 10. Time off (OFF)

**US-OFF-01: Add time off**
- I add **vacation** or **sick** days for a client, in **full days**.
- I choose **paid** or **unpaid**. If paid: a **percentage**, **fixed hours** or **fixed amount**.
- Time off reduces required hours for that client.
- I can add a **date range**. Only **workdays** in the range count as days off. Non-workdays and holidays inside the range are skipped: they don't reduce required hours and aren't paid.
  - *Example:* vacation Friday to Tuesday = 3 days off (Fri, Mon, Tue).

## 11. Reports and dashboard (REP)

**US-REP-01: Dashboard**
- The dashboard shows:
  - my **daily and monthly quota** progress
  - **current and predicted earnings** for the day and the month
  - a **current month report split by day**
- It is mainly **charts**. Clicking a chart opens the matching **table**.

**US-REP-02: Reports**
- Time totals by day, month and year.
- Period filters: this week, this month, this year, last week, last month, last year, custom range.
- Further filters: **client**, **project** and **billable**. No filter by task.
- Earnings are shown per currency. Salary view: last month, this month so far, predicted. *(From 01, 3.6.)*

## 12. User settings and UI (UI)

**US-UI-01: Theme**
- I choose a **light or dark** theme.

**US-UI-02: Time zone**
- Detected from my browser on first login, saved in my settings, changeable. *(From 01.)*

**US-UI-03: Mobile**
- The UI works well in a **phone browser**. **Every feature** works on a phone, not only a subset.

## 13. Non-functional requirements

- **NFR-01:** Runs with Docker Compose on an Ubuntu server.
- **NFR-02:** Runs behind a reverse proxy, most likely **Nginx**, which handles HTTPS.
- **NFR-03:** Designed for **1–2 users** per installation and **many years** of data.
- **NFR-04:** Backups are handled by the **server**, outside the app.
- **NFR-05:** UI in **English** only.
- **NFR-06:** Week starts on **Monday**.

## 14. Open questions

None. All questions were answered on 2026-10-22.
