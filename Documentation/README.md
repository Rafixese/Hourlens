# Hourlens documentation

> **Status: proposal.** This index lists the documents planned for the MVP. None of them are written yet. Names, order and scope are open for discussion.

Hourlens is a self-hosted, privacy-friendly time tracker for a homelab: track time on clients, projects and tasks, then turn it into reports and dashboards.

**Stack:** React (web UI) · .NET (API, including reports) · PostgreSQL. Python is deferred, see below.

## Layout

```
Documentation/
├── README.md                     ← this index
├── 01-vision-and-scope.md
├── 02-requirements.md
├── 03-domain-model.md
├── 04-architecture.md
├── 05-data-model.md
├── 06-api-spec.md
├── 07-ui-ux.md
├── 08-reports-and-analytics.md
├── 09-security-and-privacy.md
├── 10-development-guide.md
├── 11-testing-strategy.md
├── 12-deployment-and-operations.md
├── 13-roadmap.md
└── adr/                          ← Architecture Decision Records
    ├── 0000-template.md
    └── NNNN-short-title.md
```

Files are numbered in the order they should be read (and roughly written). Decisions that are hard to reverse go into `adr/` as short, dated records so the reasoning survives.

## Planned documents

### Phase A: before writing code

These pin down *what* we build and the shape of the system. Without them, the code will drift.

| # | Document | Purpose | Key contents |
|---|----------|---------|--------------|
| 01 | [Vision and scope](01-vision-and-scope.md) ✅ approved | Why Hourlens exists and what the MVP is (and is not) | Problem, target users (solo homelabber, small team?), goals, explicit non-goals, MVP feature list, success criteria |
| 02 | [Requirements](02-requirements.md) ✅ approved | What the MVP must do, testably | User stories with acceptance criteria (timer, manual entries, clients/projects/tasks, tags, billable flag, reports, export); non-functional requirements (privacy, runs on modest hardware, backup, performance) |
| 03 | Domain model and glossary | Shared language and business rules | Entities (User, Workspace, Client, Project, Task, TimeEntry, Tag, Rate), relationships, rules (one running timer per user, overlaps, rounding, time zones, archiving vs deleting) |
| 04 | Architecture | How the pieces fit | Component diagram (Mermaid), responsibilities of React / .NET / PostgreSQL, repo layout (monorepo), deployment topology |
| adr/ | Decision records | Record key choices as they are made | First ADRs: stack choice, no Python in the MVP, auth approach, monorepo vs multi-repo, ORM and migrations (EF Core?) |

### Phase B: before building each area

Written just ahead of the feature work that needs them, so they stay accurate.

| # | Document | Purpose | Key contents |
|---|----------|---------|--------------|
| 05 | Data model | PostgreSQL schema | Tables, columns, constraints, indexes, who owns migrations, soft delete, time storage (UTC + user time zone) |
| 06 | API spec | Contract between UI and backend | REST resources and endpoints, auth, pagination/filtering, error format, versioning; generated OpenAPI as the source of truth |
| 07 | UI/UX | Screens and flows | Sitemap, key flows (start/stop timer, edit week, run report), wireframes or sketches, component library choice |
| 08 | Reports and analytics | What reports exist and how they are computed | Time totals and period filters, earnings per currency, quota balance, salary and predicted salary; implemented in .NET with SQL aggregations |
| 09 | Security and privacy | Keep it safe and private | Authentication (local accounts, no public sign-up), authorization (admin vs user, per-user data isolation), secrets, no telemetry, data export and deletion |

### Phase C: alongside implementation

| # | Document | Purpose | Key contents |
|---|----------|---------|--------------|
| 10 | Development guide | How to work in the repo | Local setup (Docker Compose for dev), branching (`main` / `dev` / feature branches), coding conventions per language, commit style, PR checklist |
| 11 | Testing strategy | What gets tested and how | Unit / integration / end-to-end split, tools per stack (xUnit, pytest, Vitest, Playwright), test data, CI gates |
| 12 | Deployment and operations | Running it in a homelab | Docker Compose for production, configuration and env vars, reverse proxy/TLS, backup and restore, upgrades and migrations, logs and health checks |
| 13 | Roadmap | Path from zero to MVP and beyond | Milestones with "done" criteria, MVP cut line, post-MVP ideas (mobile, integrations, invoicing, Clockify import) |

### Later (after the MVP works)

- User guide (end-user docs)
- `CHANGELOG.md` and `CONTRIBUTING.md` at the repo root

## Decision: no Python in the MVP

Agreed 2026-10-22: the MVP uses one backend, **.NET**, for the API, reports and exports. PostgreSQL handles the aggregations, and .NET libraries cover CSV/PDF/XLSX export. One fewer service keeps the homelab deployment simple.

Python stays an option for later if a real need appears (heavier analytics, a data import tool, scheduled jobs). This will be recorded as an ADR.
