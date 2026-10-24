# 01 · Vision and scope

> **Status: approved** · Last updated: 2026-10-22
>
> This document only contains decisions made by the project owner. Changes to scope need the owner's decision.

## 1. Vision

Hourlens is a self-hosted time tracking service that replaces paid Clockify. It runs on a personal homelab server and is fully owned by the person who hosts it.

## 2. Users

- **Primary user:** the project owner, hosting it on a personal homelab server.
- **Other users:** a few people the admin adds, such as family members.

### 2.1 Roles

| Role | How it is created | What it can do |
|------|-------------------|----------------|
| **Admin** | One admin account is created during installation | Everything a user can do (including tracking their own time), plus the admin panel. The admin **cannot view** other users' time entries or reports, only manage their accounts. |
| **User** | Created by the admin | Track time, manage their own clients/projects/tasks, view their own reports |

### 2.2 Login and data separation

- Users log in with **local accounts** (username and password).
- There is **no public sign-up**. Only the admin creates accounts.
- **Each user sees only their own data.**
- **Everything is separate per user:** clients, projects, tasks, time entries, rates, currencies, workdays, holidays and time off.

## 3. MVP scope

### 3.1 Clients, projects and tasks

- Work is organized as **Client → Project → Task → Time entry**.
- Creating a client also creates a **default project** for it, named **"Default"**. The user can rename it.
- There are **no default tasks**. A task is created from the **title** the user types when starting a timer or adding an entry:
  - If the title matches an existing task, the entry is logged against that task.
  - If the title is new, a new task is created.
- When starting a timer or adding an entry, the user picks a **client and a project** from a picker. If only one option is available, it is selected by default.
- **Every time entry belongs to a task.** There are no entries without a task.

### 3.2 Time tracking

- **Live timer:** start and stop.
- **Manual entries:** add time without the timer.
- **One running timer at a time** per user in the MVP. The design must allow **several simultaneous timers later** without a rewrite.
- Each time entry is either **billable or non-billable**.

### 3.3 Hourly rates and currency

- Each user has a **default hourly rate**.
- Each client either **uses the user's default rate** or has a **custom rate** entered by the user.
- Rates exist **only** at these two levels. There are no rates per project or per task.
- **Currency** is chosen together with the rate: when setting the default rate, and when setting a client's custom rate.
- **Rate history** applies to both the default rate and client rates:
  - A rate change applies **from a chosen date onward**. Older entries keep the rate that applied when they were logged.
  - *Example:* the rate is 100 PLN/h through December 2026 and changes to 120 PLN/h from January 2027. December 2026 stays at 100 PLN/h.
  - When changing a rate, the user can **optionally apply the new rate to past entries**, starting from a date they pick.
- Totals across clients are shown as **one total per currency**. There is no currency conversion.

### 3.4 Work quota and calendar

**Quota**
- A **quota** (expected working time) is set **per client** and is **optional**.
- A client's quota is **either daily** (for example 7.5 h per workday) **or monthly**, never both.
- A **monthly quota** is spread across the month's workdays, for daily progress and for valuing days off. *Example:* 160 h ÷ 20 workdays = 8 h per workday, so the app can show "8 h expected today".
- **Billable and non-billable hours both count** towards the quota.
- **Time off reduces the required hours** for the period.
- The quota balance is **per month** and **does not carry over** to the next month.

**Workdays**
- Set per user, by the user. Default: **Monday to Friday**.

**Holidays**
- Set per user, by the user.
- **Default: manual.** The user enters holidays year by year (for example 25 December).
- **Optional: external holiday provider.** The user chooses a country, and the app loads that country's public holidays from an external API. Loaded holidays **can be edited manually** afterwards.

**What the app calculates**
- The **required hours** for a period, from quota, workdays, holidays and time off.
- Whether the user is **below, equal to or above quota**, and by how much.
  - *Example:* with a 7.5 h daily quota, 35 h worked in a 5-workday week means 2.5 h short. That shortfall is shown in the month's balance.
- If the user has **several clients**, the balance is shown **per client and as a total**.

### 3.5 Time off (vacation and sick days)

- The user can add **vacation** and **sick days**. These are the only types in the MVP.
- Time off is in **full days only**. Half days or hours may come later.
- Time off is **tied to a client**.
- For each time off entry, the user chooses whether it is **paid** (billable, such as paid holiday) or **unpaid** (it reduces the monthly pay).
- Paid time off can be valued in any of these ways, as the user chooses:
  - a **percentage** of normal pay, for example 80% for sick leave
  - a **fixed number of hours**
  - a **fixed amount** of money
- How many hours a day off is worth, as the base for the percentage:
  - client with a **daily quota**: the daily quota
  - client with a **monthly quota**: monthly quota ÷ workdays in that month
  - client with **no quota**: the user enters the hours

### 3.6 Reports and dashboards

- Time totals **by day, month and year**.
- Period filters: **this week, this month, this year, last week, last month, last year, custom range**.
- **Earnings:** money made per day, month and so on, calculated as hours × rate. Shown per currency.
- **Salary view** (salary means earnings, hours × rate):
  - last month's salary
  - this month's salary so far
  - this month's **predicted salary**, assuming the remaining workdays are worked at quota
- Quota balance and predicted salary are shown **per client and as a total** when there are several clients.

### 3.7 Admin panel

- **User management:** create users, reset passwords, delete users and their data.
- **Email server (SMTP) settings**, used for self-service password reset. *(Added 2026-10-22 during 02 Requirements.)*

### 3.8 General settings

- UI language: **English** only.
- The week starts on **Monday**.
- **Time zone** is detected from the browser on first login, saved in the user's settings, and can be changed by the user. All "today", day, week and month calculations use this saved time zone.

## 4. Out of scope for the MVP

- Mobile app
- Desktop app
- Browser extension
- Integrations with other tools (other than the optional holiday provider)
- Clockify import
- Export of any kind (CSV, PDF, XLSX)
- Several simultaneous timers (the design must allow them later)
- Half-day and hourly time off
- Time off types other than vacation and sick days
- Currency conversion
- Quota balance carry-over between months
- Python services (see [README](README.md))

## 5. Future ideas (not committed)

- Several simultaneous timers
- Half-day and hourly time off

## 6. Hosting

- Deployed with **Docker Compose**.
- Target server: **Ubuntu Linux** homelab server.
- Stack: React, .NET, PostgreSQL.

## 7. Definition of done

The MVP is done when:

1. The app can be installed.
2. An admin account exists after installation, and the admin can create user accounts.
3. The admin can configure the app in the admin panel.
4. Users can log in.
5. Users can define clients, projects and tasks, and log time against tasks.
6. Users can use the live timer and add manual entries.
7. Users can see reports.

## 8. Open questions

None. All questions were answered on 2026-10-22.
