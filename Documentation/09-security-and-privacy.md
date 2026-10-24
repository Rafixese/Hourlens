# 09 · Security and privacy

> **Status: draft, two questions open** · Last updated: 2026-10-24
>
> How Hourlens protects accounts and data. Builds on authentication in [04 Architecture](04-architecture.md) (3.5) and the API in [06](06-api-spec.md). Items marked **⏳ S#** wait on an open question, see [Open questions](#8-open-questions).

## 1. Threat model (short)

Hourlens runs on a homelab server for 1–2 users, behind Nginx with HTTPS, possibly reachable from the internet. The main risks:

- someone guessing or brute-forcing a password
- another website making the browser act on Hourlens while the user is logged in (CSRF)
- injected scripts (XSS) stealing data
- one user seeing another user's data
- sensitive data (client names, titles, earnings) leaking into logs or to third parties

## 2. First run

- The setup screen (02, US-SETUP-01) is open to whoever reaches the app first while no account exists. **No setup code** is required (S1).
- **Operational rule:** finish setup before making the app reachable from the internet. This is stated in 12 Deployment.
- Once an account exists, `POST /setup` always answers 409 (06, section 2).

## 3. Accounts and login

| # | Control |
|---|---------|
| — | Passwords hashed by **ASP.NET Core Identity** (PBKDF2) |
| S4 | Password rules: **8–128 characters**, no forced symbols, and the **most common passwords are rejected** (a built-in list) |
| S2 | **Lockout:** 5 failed logins → account locked for **15 minutes** |
| S3 | **Rate limiting per IP and per account** on `/auth/login`, `/auth/forgot-password` and `/auth/reset-password`, e.g. 10 attempts per minute. Over the limit: 429 Too Many Requests |
| — | Login errors never say whether the email exists. Forgot password always answers 204 (06, section 2) |
| — | Password reset tokens come from Identity, are single-use and expire after **1 hour** |
| S5 | **No two-factor login in the MVP.** Planned after the MVP (authenticator app) |

## 4. Sessions

| # | Control |
|---|---------|
| — | Session cookie: `HttpOnly`, `Secure`, **`SameSite=Lax`**. 30 days with "remember me", browser session without (02, US-AUTH-02) |
| S6 | **Changing or resetting a password logs out all other devices** (Identity security stamp) |
| S7 | **Admin actions take effect immediately:** a deleted user or a user whose password was reset is logged out at once |
| S8 | **CSRF:** `SameSite=Lax` cookie **plus** ASP.NET Core **anti-forgery token**, sent in a header with every request that changes data |
| — | SignalR uses the same cookie. A connection is closed when its user is logged out |

## 5. Data isolation

- Every user-owned table has `user_id`, and every query filters by the logged-in user (05, conventions).
- Asking for another user's data by ID answers **404**, as if it didn't exist (06, P14).
- The admin role manages accounts and email settings only. It **cannot read** other users' data (01, 2.1).
- Automated tests check isolation for every endpoint (11 Testing strategy).

## 6. Transport and browser protections

| # | Control |
|---|---------|
| — | **HTTPS is terminated by Nginx.** Nginx sets **HSTS** |
| S9 | The app trusts `X-Forwarded-For` / `X-Forwarded-Proto` **only from the reverse proxy**. ⏳ S9 (Nginx on another server) |
| S10 | Security headers sent by the app: **Content-Security-Policy** (only own scripts, styles, fonts and the SignalR connection; no inline scripts), `X-Content-Type-Options: nosniff`, `frame-ancestors 'none'` (no framing), `Referrer-Policy: same-origin` |
| — | React escapes all output by default. No raw HTML from user input is ever rendered |

## 7. Privacy

| # | Rule |
|---|------|
| S11 | **No outbound calls by default.** Fonts (Sora, Inter) and all assets are **bundled in the app**, never loaded from Google Fonts or a CDN |
| — | The only outbound calls are **Nager.Date** (when the user turns on the holiday provider) and **the admin's SMTP server** (when configured) |
| — | **No telemetry**, no analytics, no update checks |
| S12 | **Logs never contain** entry titles, task titles, client or project names, amounts, emails in request bodies, passwords or tokens. Logs record IDs, routes, status codes and timings |
| S13 | **Admin actions are logged** as structured log lines: user created, user deleted, password reset by admin, SMTP settings changed. No separate audit table |
| S14 | **Encryption at rest** is left to the server's disk encryption. The app encrypts only the **SMTP password** (05, M14) |
| — | `/metrics` exposes counts and timings only, no user data, and isn't proxied by Nginx (04, 6.1) |
| — | Deleting a user deletes all their data (02, US-ADMIN-02). Backups made by the server may still contain it until they rotate |

### 7.1 Secrets

| # | Rule |
|---|------|
| S15 | Secrets (database password) live in **`.env`** with permissions **`600`**, owned by the user that runs Docker. `.env` is never committed. An `.env.example` without secrets is in the repo |
| — | ASP.NET Data Protection keys (cookies, SMTP password) are stored in a **Docker volume** so they survive restarts. Losing them logs everyone out and requires re-entering the SMTP password |

### 7.2 Supply chain

| # | Rule |
|---|------|
| S16 | **Dependabot** checks NuGet, npm, Docker and GitHub Actions dependencies and opens PRs for updates and known vulnerabilities |
| — | Docker images are built from pinned base images (.NET runtime, PostgreSQL) |

## 8. Open questions

- **S9.** Nginx might run on another server. Proposal: two `.env` settings.
  - **`HOURLENS_BIND`**: the address the app port is published on. Default `127.0.0.1` (Nginx on the same server). Set it to the server's LAN IP when Nginx runs elsewhere.
  - **`HOURLENS_TRUSTED_PROXIES`**: the IP addresses or networks allowed to send forwarded headers. Default `127.0.0.1`. Set it to the Nginx server's IP otherwise.
  
  This also changes 04 (6.1), which currently says the app is always bound to `127.0.0.1`.
- **S17.** Details filled in by Claude, to confirm: password reset links expire after **1 hour**; the rate limit is **10 attempts per minute** per IP and per account; `Referrer-Policy: same-origin`.
