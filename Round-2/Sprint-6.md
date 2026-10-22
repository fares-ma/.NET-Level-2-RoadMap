# Sprint 6 — EF Core Aggregations, Docker, Health Checks, CI/CD & Final Delivery

> **Project:** DonationHub — Donation Management & Crowdfunding Platform  
> **Duration:** 15 Days (06/01/2027 to 20/01/2027)  
> **Meeting Schedule:**  
> - **Stand-up 1 (Day 5):** 10/01/2027 — Session Leader: **Kareem Zein**  
> - **Stand-up 2 (Day 10):** 15/01/2027 — Session Leader: **Mohamed Atef**  
> - **Closing & Deadline (Day 15):** 20/01/2027 — Session Leader: **Asma ELhadidy**  
> - **Code Review & Final Corrections (Days 16–18):** 21/01/2027 to 23/01/2027  
> **Evaluation:** 100 Points  
> **Target Framework:** .NET 10 (fully compatible with .NET 8 / .NET 9)

---

## 🎯 Sprint Goal

Take DonationHub from "it works on my machine" to an automated, containerized, and production-ready system. Implement the **Admin Analytics & Reports** dashboard using optimized **EF Core Aggregations and Projections**, containerize the application using an efficient **Multi-stage Dockerfile**, orchestrate the entire environment (API, SQL Server, Redis, Mailpit) using **Docker Compose**, configure **Health Checks**, build a continuous integration pipeline via **GitHub Actions**, and craft comprehensive engineering documentation for final defense.

```text
Admin Dashboard Analytics (Optimized GroupBy & Projections)
                    ↓
Multi-Stage Containerization (Build SDK → Lean Runtime)
                    ↓
Docker Compose Orchestration (API + SQL + Redis + Mailpit)
                    ↓
Liveness & Readiness Health Checks (/health/live, /health/ready)
                    ↓
GitHub Actions CI Pipeline (Automated Build & Unit/Integration Tests)
                    ↓
Production Architecture Documentation & Final Project Defense
```

---

## 📌 Sprint Prerequisites

### Multi-Container Orchestration & CI Basics
*Learners must be familiar with Docker networking concepts and YAML syntax:*
- Understand how Docker networks allow containers to communicate by service name (e.g., `Server=sqlserver` instead of `localhost`).
- Basic familiarity with GitHub repository settings (Repository Secrets, GitHub Actions runners).

---

## 🛠️ Sprint Tasks

All tasks below are **CORE** unless marked BONUS.

### Task 1 — Admin Analytics Dashboard & EF Core Aggregations

Implement reporting and aggregate queries according to SRS §5.10:
- **RPT-01 (Summary Metrics):**
  - Total funds raised across all `Succeeded` donations (`Sum`).
  - Total count of `Succeeded` donations (`Count`).
  - Total count of distinct donors.
  - Campaign counts grouped by status (`Active`, `Completed`, `Expired`).
- **RPT-02 (Top Performing Campaigns):**
  - Top N campaigns ranked by `RaisedAmount` (`OrderByDescending` + `Take`).
- **RPT-03 (Donations Over Time):**
  - Daily aggregated totals within a date range (`GroupBy(d => d.CreatedAt.Date)` translated directly to SQL `GROUP BY`).
- **Performance Rules:**
  - Enforce `AsNoTracking()` on all reporting queries.
  - Project directly into response DTOs via `.Select()` to fetch only required columns.
  - Use `.AsSplitQuery()` where appropriate to avoid Cartesian product explosion on joined collections.
  - **RPT-04:** Cache the dashboard responses in Redis via `IDistributedCache` (default 5-minute expiration, configured through options). The numbers must match the database: verify them against a hand-written SQL query and document that query in your README.

| Method | Path | Access |
|---|---|---|
| `GET` | `/admin/dashboard/summary` | Admin Only |
| `GET` | `/admin/dashboard/top-campaigns?count=5` | Admin Only |
| `GET` | `/admin/dashboard/donations-over-time?from=&to=` | Admin Only |

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [EF Core Query Data: Projections & Aggregations — Essam Abdelnabi [Arabic] (1:11:58)](https://youtu.be/fb8j6CjyWWw) (Projections & AsNoTracking at 08:13, AsSplitQuery at 13:38, GroupBy in SQL at 44:17).
  - ⭐2 [Microsoft Learn — Complex Query Operators (GroupBy in EF Core)](https://learn.microsoft.com/en-us/ef/core/querying/complex-query-operators).
- **🔄 Alternative / Deep Dive:** 📘 [Tracking vs No-Tracking Queries in EF Core — codewithmukesh](https://codewithmukesh.com/blog/tracking-vs-no-tracking-queries-efcore/) (written reference, .NET 10 samples on PostgreSQL)

---

### Task 2 — Admin User & Staff Management

Implement user and staff administration according to SRS §5.10:
- **ADM-01:** Admin lists all users — paged, filter by role, search by name or email.
- **ADM-02:** Admin activates or deactivates a user. A deactivated user cannot log in (and cannot refresh tokens, IDN-10).
- **ADM-03:** Admin lists, updates and deletes staff users (`Admin`, `CampaignManager`). **Delete means deactivate** (no hard delete). Business rules:
  - An Admin cannot deactivate themself.
  - The last active Admin cannot be removed or deactivated.
- Expected failures return `Result` errors mapped to `ProblemDetails` (no exceptions for business rules).
- Add unit tests for the admin rules (self-deactivation blocked, last active Admin protected, deactivated user cannot log in).

| Method | Path | Access |
|---|---|---|
| `GET` | `/admin/users?page=&role=&search=` | Admin Only |
| `PATCH` | `/admin/users/{id}/status` | Admin Only |
| `GET` | `/admin/staff` | Admin Only |
| `PUT` | `/admin/staff/{id}` | Admin Only |
| `DELETE` | `/admin/staff/{id}` | Admin Only |

---

### Task 3 — Production Multi-Stage Dockerfile

Create an optimized `Dockerfile` in the repository root:
- **Stage 1 (Build):** Use the official .NET 10 SDK image (`mcr.microsoft.com/dotnet/sdk:10.0`), copy project files, restore dependencies (`dotnet restore`), copy source code, and publish optimized binaries (`dotnet publish -c Release -o /app/publish`).
- **Stage 2 (Runtime):** Use the lean ASP.NET runtime image (`mcr.microsoft.com/dotnet/aspnet:10.0`), copy only the published artifacts from Stage 1, expose port 8080, and run under a secure non-root user.
- Add a `.dockerignore` file excluding `bin/`, `obj/`, `.git/`, and local test results.

**Resources:**
- **⭐ Required Primary:** [Dockerize ASP.NET Core Application Part 12 — Codographia (27:04)](https://youtu.be/nINTOS2fhVY).
- **🔄 Alternative / Deep Dive:** [Docker Full Course For .NET — Julio Casal](https://youtu.be/cWMztQwIQNs)

---

### Task 4 — Full Stack Orchestration via Docker Compose

Create `docker-compose.yml` to launch the complete DonationHub ecosystem with a single command (`docker compose up -d`), according to SRS §6.12:
1. **`api`:** Builds from the local `Dockerfile`. All settings come from environment variables. Starts only after its dependencies are healthy (`depends_on` with `condition: service_healthy`).
2. **`sqlserver`:** Containerized Microsoft SQL Server (`mcr.microsoft.com/mssql/server`) with a health check and a named volume for SQL data.
3. **`redis`:** Containerized Redis cache (`redis:alpine`) with a health check.
4. **`mailpit`:** Containerized local SMTP server (`axllent/mailpit`) exposing port 8025 for web UI and 1025 for SMTP.
- **Named volumes:** one for SQL Server data and one for uploaded files (campaign banners and receipts).
- **No secrets in the repository:** values are supplied through environment variables loaded from a `.env` file. `.env` is git-ignored; commit a `.env.example` with placeholder values only. Never put secrets in `appsettings*.json`.
- **Working from a clean clone:** `docker compose up` must produce a working API with EF Core migrations applied and the admin user seeded on startup (from `SeedAdminOptions`).

**Resources:**
- **⭐ Required Primary:** [Build a Complete .NET 10 Dev Environment with Docker Compose — Milan Jovanović (16:00)](https://youtu.be/svfxvsGfLlU).
- **🔄 Alternative / Deep Dive:** [Docker Compose Tutorial — Mosh Hamedani](https://youtu.be/HG6yIjZapSA)

---

### Task 5 — ASP.NET Core Health Checks

Implement health monitoring endpoints according to SRS §6.11:
- Install `AspNetCore.HealthChecks.SqlServer` and `AspNetCore.HealthChecks.Redis`.
- Configure dedicated endpoints for container orchestrators:
  - `/health/live` (Liveness probe): the process is up — **no dependency checks**.
  - `/health/ready` (Readiness probe): checks SQL Server connectivity, Redis connectivity, and **write access to the file storage folder** (custom health check).
- Responses are JSON and include each check's name and status.
- Use `/health/ready` as the `api` container health check in Docker Compose.

**Resources:**
- **⭐ Required Primary:** [How to Add Health Checks in ASP.NET Core — Milan Jovanović (11:51)](https://youtu.be/4abSfjdzqms).
- **🔄 Alternative / Deep Dive:**
  - [Best Way to Add Health Checks in .NET — Nick Chapsas](https://youtu.be/p2faw9DCSsY)
  - 📘 [Health Checks in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/health-checks-in-aspnet-core/) (written reference; samples check PostgreSQL — use the SQL Server check for DonationHub)

---

### Task 6 — Automated CI/CD Pipeline via GitHub Actions

Create `.github/workflows/ci.yml` according to SRS §6.14:
- Trigger on **every** `push` and `pull_request` (all branches).
- Job Steps:
  1. Check out repository code.
  2. Setup .NET 10 SDK.
  3. Restore NuGet dependencies (`dotnet restore`).
  4. Build solution in Release configuration (`dotnet build --no-restore -c Release`).
  5. Run all tests (`dotnet test --no-build -c Release`) — unit tests and the Sprint 5 integration tests (e.g., against a SQL Server service container). Tests must not call real Stripe or real SMTP.
  6. Fail the pipeline if any test fails or compilation fails.
- Add the CI status badge at the top of `README.md`.
- The pipeline must pass on the commit tagged `sprint-6`.

**Resources:**
- **⭐ Required Primary:** [1. CI/CD Pipeline Using GitHub Actions — Coding-Future (23:29)](https://youtu.be/Zv-NJBRmoVU).
- **🔄 Alternative / Deep Dive:**
  - [GitHub Actions Full Tutorial — TechWorld with Nana](https://youtu.be/R8_veQiYBjI)
  - [GitHub Actions Complete Course — Ahmed El-Fakharany [Arabic]](https://youtu.be/7gJFHjXscr8)

---

### Task 7 — Professional Architecture Documentation & Final README.md

Write a comprehensive, publication-ready `README.md` for DonationHub documenting:
- CI status badge at the top.
- **Architecture explanation:** diagram and layer responsibilities (Domain, Application, Infrastructure, Api), vertical slices, and the read/write split (repository + UoW for writes, `AsNoTracking` projections for reads).
- Technology stack, libraries, and design patterns used.
- **How to run locally:** prerequisites, user-secrets, migrations, and the Redis/Mailpit containers.
- **How to run with Docker:** `cp .env.example .env`, then `docker compose up`.
- Environment variables and options configuration table.
- **How to test Stripe locally:** Stripe CLI (`stripe listen --forward-to ...`), the webhook signing secret via user-secrets/env vars, and `stripe trigger` commands.
- **Design decisions:** a list of the key decisions and trade-offs (e.g., why `RaisedAmount` is stored, Result vs. exceptions, idempotency strategy, cache invalidation, soft delete).
- API endpoint summary and authentication guide, plus the hand-written SQL used to verify dashboard numbers (RPT-04).

**Resources:**
- **⭐ Required Primary:** [How to Write a Professional README.md — Fekra w Code [Arabic] (14:27)](https://youtu.be/ATVJiw4JrUA).
- **🔄 Alternative / Deep Dive:** [Write an Awesome GitHub README — Learn Fast Make Things](https://youtu.be/E6NO0rgFub4)

---

## 📦 Sprint Deliverables

- [ ] Admin reporting endpoints (RPT-01 to RPT-03) executing optimized aggregate queries (`Sum`, `Count`, `GroupBy`).
- [ ] Dashboard responses cached in Redis (default 5 minutes) with numbers matching the database (RPT-04).
- [ ] Admin user and staff management endpoints (ADM-01 to ADM-03) with their business rules and unit tests.
- [ ] Multi-stage `Dockerfile` generating a lean, non-root runtime image.
- [ ] `docker-compose.yml` orchestrating `api`, `sqlserver`, `redis`, and `mailpit` with named volumes, env-var configuration (`.env.example` committed), health checks and `depends_on` conditions.
- [ ] Health checks `/health/live` and `/health/ready` (SQL Server, Redis, file storage write access) with JSON responses.
- [ ] Working GitHub Actions CI workflow running restore, build and tests on every push and pull request.
- [ ] Professional, complete `README.md` documenting the entire project (architecture, local and Docker run, Stripe local testing, design decisions).

---

## ✅ Definition of Done

- [ ] Running `docker compose up --build` spins up all 4 services without errors.
- [ ] `docker compose up` from a clean clone gives a working API with migrations applied and the admin seeded.
- [ ] `/health/live` and `/health/ready` respond with JSON status for each check.
- [ ] Reporting endpoints translate to efficient SQL `GROUP BY` without client-side evaluation.
- [ ] Dashboard numbers match a hand-written SQL query.
- [ ] Unit tests cover the new admin rules (self-deactivation, last active Admin, deactivated user login).
- [ ] Every push and pull request triggers the GitHub Actions workflow, and CI passes on the tagged commit.
- [ ] The `sprint-6` tag exists and points to the final commit.
- [ ] No secrets in the repository history (`.env` git-ignored, only `.env.example` committed).
- [ ] The learner is prepared for the Final Graduation Defense to walk through the architecture, design patterns, and justify technical trade-offs.

---

## ⭐ BONUS (optional — counts only after all CORE items pass)

- **RPT-05:** Donations by category and by month.
- **CMP-12 — Admin Campaign Management & Status Overrides:** Admin (or the owner) can set a campaign to `Cancelled` (e.g., cancel a fraudulent campaign). When a campaign is closed by an Admin, validate that pending payment attempts are handled appropriately.
- **Cloud deployment:** optional and never graded.
- **Testcontainers in CI:** run the integration tests in the pipeline using Testcontainers instead of a pre-provisioned service container.

---

## 📤 Submission

- Submit your repository URL on **tawjeh.net** before **23:59 on Day 15 (20/01/2027)**.
- Tag the final commit and push the tag:
  ```bash
  git tag sprint-6
  git push origin sprint-6
  ```
- Fixes are allowed within the review window (**21/01/2027 to 23/01/2027**). Do not move or force-push `sprint-6`.
- Fixes for the final defense are tagged `final` (`git tag final && git push origin final`). The **final defense** takes place on **21–23/01/2027**.

---

## 🧮 Grading (100 Points)

| Part | Points | How |
|---|---|---|
| Automated | 40 | Build succeeds, `dotnet test` passes, the `sprint-6` tag exists, CORE endpoints respond as specified (smoke check by the mentor) |
| Checklist | 40 | The Definition of Done above (structure, rules, security, README) |
| Explanation | 20 | A short question on one of your design decisions, or a code review of the decisions written in your README |

- BONUS items add capped extra credit on top of the 100 points, only after all CORE items pass.
- The final defense is evaluated separately using the 5 × 20% criteria in the Round 2 RoadMap.

---

## 📖 Post-Course Reading (Continuous Learning)

### 1. Real-Time Communication with SignalR
- [SignalR Full Course — tutorialsEU](https://youtu.be/LEImpa77irI)
- [Real-Time Web Apps with SignalR — Mohamad Lawand](https://youtu.be/pvi_ZS_PrSc)
- [SignalR in ASP.NET Core Playlist — Ahmed Mohamady [Arabic]](https://youtube.com/playlist?list=PL62tSREI9C-e6nQ47brLj00iSGddiee73)

### 2. Cloud Native .NET Aspire & Microservices
- [.NET Aspire Crash Course — Julio Casal](https://youtu.be/pk6FJfHhfq8)
- [.NET Microservices Full Course — Les Jackson](https://youtu.be/DgVjEo3OGBI)
- [Microservices Architecture with .NET — Julio Casal](https://youtu.be/ByYyk8eMG6c)
- [Microservices with .NET — DotNetMastery](https://youtu.be/Nw4AZs1kLAs)
- 📘 [When to Use Microservices (And When Not To) — codewithmukesh](https://codewithmukesh.com/blog/when-to-use-microservices/)
- 📘 [Containerize .NET Without a Dockerfile — codewithmukesh](https://codewithmukesh.com/blog/containerize-dotnet-without-dockerfile/)
- 📘 [Anti-Patterns to Avoid in .NET APIs — codewithmukesh](https://codewithmukesh.com/blog/anti-patterns-to-avoid-dotnet-apis/)
- 📘 [.NET Web API Zero to Hero — full course (new lessons on testing, observability & CI/CD are added regularly)](https://codewithmukesh.com/courses/dotnet-webapi-zero-to-hero/)

### 3. Recommended Books
- *Clean Architecture: A Craftsman's Guide to Software Structure and Design* — Robert C. Martin
- *Dependency Injection Principles, Practices, and Patterns* — Steven van Deursen & Mark Seemann
- *Domain-Driven Design: Tackling Complexity in the Heart of Software* — Eric Evans
- *Software Architecture with C# 12 and .NET 8* — Gabriel Baptista & Francesco Abbruzzese
- *Designing Data-Intensive Applications* — Martin Kleppmann