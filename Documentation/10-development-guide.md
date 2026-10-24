# 10 · Development guide

> **Status: approved** · Last updated: 2026-10-24
>
> How Hourlens is developed: how we work together, local setup, Git workflow and code conventions. Built by the owner with Claude as a mentor, about 5–10 hours a week (13, RM1).

## 1. How we work together

**Roles.** The owner is the **developer** and writes the code. Claude is the **senior developer and mentor**: explains concepts, plans the work, reviews code and teaches the "why". Claude writes only the trivial parts.

### 1.1 Who writes what (W1)

| Claude does (and explains) | The owner writes |
|----------------------------|------------------|
| Config and plumbing you'd copy from docs: `.editorconfig`, ESLint / Prettier config, `.gitignore`, Dependabot, `.env.example` | Domain logic and calculations (03, section 4) |
| The generated API client (orval output) | Controllers, Application use cases, EF Core models and migrations |
| The demo data script | React screens and components |
| `CLAUDE.md` | Tests |
| | Dockerfile and GitHub Actions workflows, at least the first time |

When in doubt, the owner writes it.

### 1.2 Flow for each issue (W2)

1. **Plan.** Claude explains the concepts the issue needs and proposes a plan: which files, which layers, in what order. It points to the relevant docs (02 stories, 03 rules, 05 tables, 06 endpoints).
2. **Build.** The owner creates a branch and writes the code.
3. **Review.** The owner opens a PR. Claude reviews it **on GitHub** (W4) like a senior would: what's good, what to change, and **why**.
4. **Fix and merge.** The owner addresses the comments. When CI is green and the review is approved, the owner squash-merges.

### 1.3 When stuck (W3)

Help comes in steps, and the owner can ask for the next one at any time:

1. a **guiding question** ("what should happen if the client has no rate?")
2. a **pointer**: the doc section, the official docs page, or a similar place in the code
3. a **small snippet** showing the idea, not the whole solution
4. a **full solution**, only when the owner asks for it

## 2. Local setup

**Environment:** Linux, Visual Studio Code (G1).

### 2.1 Prerequisites

- .NET 10 SDK
- Node.js LTS with **npm** (G3)
- Docker with Docker Compose
- Git, GitHub CLI (`gh`) optional
- VS Code extensions: C# Dev Kit, ESLint, Prettier, Docker, EditorConfig

### 2.2 Running in development (G2)

| Part | How it runs | Notes |
|------|-------------|-------|
| PostgreSQL | `docker compose -f deploy/docker-compose.dev.yml up -d db` | Only the database runs in Docker |
| Backend | `dotnet watch` in `backend/src/Hourlens.Api` | Restarts on code changes. Migrations run on start |
| Frontend | `npm run dev` in `frontend/` | Vite dev server. Forwards `/api` and `/hubs` (including WebSockets) to the backend |

- The full `docker-compose.yml` (app + db) is only used to **test the production image** locally.
- Dev settings (connection string) live in `appsettings.Development.json` and user secrets, never in Git.

### 2.3 Demo data (G4)

- A script fills the dev database with demo data: two users, clients with different currencies and quotas, rate changes, holidays, time off and several months of entries.
- Used to try the reports by hand. **Development only**: it refuses to run against a production database.

## 3. Git workflow

| # | Rule |
|---|------|
| G5 | **Trunk-based:** short-lived branches merged straight into **`main`**. The existing `dev` branch is kept for now but not used in this workflow; what to do with it is decided later |
| G5 | Branch names: **`feature/<issue#>-short-name`**, e.g. `feature/12-setup-screen`. `fix/…` and `docs/…` for fixes and doc-only changes |
| G6 | **Conventional Commits**: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`, `ci:`. Example: `feat(timer): stop running timers at midnight` |
| G7 | PRs need **green CI** and an approved review. **Squash merge**, so each issue is one commit on `main`. The PR links its issue (`Closes #12`) |
| G13 | **Docs change in the same PR** as the code that needs them |
| RM7 | When a milestone is finished, the owner tags `main` (`v0.1.0` … `v1.0.0`, 13) |

Every merge to `main` publishes the `latest` image (04, A19).

## 4. Code conventions

| # | Rule |
|---|------|
| G10 | Code, comments, commit messages and PR texts in **English** |
| G8 | **C#:** `.editorconfig` + `dotnet format`, checked in CI. Nullable reference types on. Warnings as errors in CI |
| G8 | **TypeScript:** strict mode, ESLint + Prettier, checked in CI |
| G9 | **No pre-commit hooks.** CI is the gate |
| — | Backend layering follows 04 (3.1): Domain has no dependencies; Api never talks to the database directly |
| — | Database naming follows 05 (snake_case, UUID v7, minutes, `numeric(12,2)`) |
| — | API shapes follow 06 (camelCase JSON, Problem Details, `version` field) |

## 5. Claude Code in the repo (G11, G12)

A **`CLAUDE.md`** file at the repo root tells every Claude Code session how to behave in this project: the mentoring rules from section 1, the conventions above, and where the docs are. Its content is kept in sync with this guide.

**One issue per session:** start a Claude Code session for one issue, plan it together, write the code, open the PR.
