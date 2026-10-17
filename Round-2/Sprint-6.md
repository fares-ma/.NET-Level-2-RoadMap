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
Liveness & Readiness Health Checks (/health)
                    ↓
GitHub Actions CI Pipeline (Automated Build & Unit/Integration Tests)
                    ↓
Production Architecture Documentation & Final Project Defense
```

---

## 📌 Sprint Prerequisites

### Multi-Container Orchestration & CI Basics
*Learners must be familiar with Docker networking concepts and YAML syntax:*
- Understand how Docker networks allow containers to communicate by service name (e.g., `Server=sql-server` instead of `localhost`).
- Basic familiarity with GitHub repository settings (Repository Secrets, GitHub Actions runners).

---

## 🛠️ Sprint Tasks

### Task 1 — Admin Analytics Dashboard & EF Core Aggregations

Implement reporting and aggregate queries according to SRS §3.8:
- **RPT-01 (Summary Metrics):**
  - Total funds raised across all successful donations (`Sum`).
  - Total count of successful donations (`Count`).
  - Total count of distinct donors.
  - Campaign counts grouped by status (`Active`, `Completed`, `Expired`).
- **RPT-02 (Top Performing Campaigns):**
  - Top N campaigns ranked by `RaisedAmount` (`OrderByDescending` + `Take`).
- **RPT-03 (Donations Over Time):**
  - Daily aggregated totals within a date range (`GroupBy(d => d.CreatedAtUtc.Date)` translated directly to SQL `GROUP BY`).
- **Performance Rules:**
  - Enforce `AsNoTracking()` on all reporting queries.
  - Project directly into response DTOs via `.Select()` to fetch only required columns.
  - Use `.AsSplitQuery()` where appropriate to avoid Cartesian product explosion on joined collections.
  - **RPT-04:** Cache the summary dashboard response in Redis with a 5-minute expiration window.

| Method | Path | Access |
|---|---|---|
| `GET` | `/admin/dashboard/summary` | Admin Only |
| `GET` | `/admin/dashboard/top-campaigns?count=5` | Admin Only |
| `GET` | `/admin/dashboard/donations-over-time?from=&to=` | Admin Only |

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [EF Core Query Data: Projections & Aggregations — Essam Abdelnabi [Arabic] (1:11:58)](https://youtu.be/fb8j6CjyWWw) (Projections & AsNoTracking at 08:13, AsSplitQuery at 13:38, GroupBy in SQL at 44:17).
  - ⭐2 [Microsoft Learn — Complex Query Operators (GroupBy in EF Core)](https://learn.microsoft.com/en-us/ef/core/querying/complex-query-operators).

---

### Task 2 — Admin Campaign Management & Status Overrides

Implement administrative campaign controls according to SRS §3.8:
- **ADM-01:** Admin can manually change campaign status (e.g., cancel a fraudulent campaign, mark as completed early).
- **ADM-02:** When an Admin closes a campaign, validate that pending payment attempts are handled appropriately.

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

Create `docker-compose.yml` to launch the complete DonationHub ecosystem with a single command (`docker compose up -d`):
1. **`donationhub-api`:** Builds from the local `Dockerfile`, mounts secrets via environment variables, depends on database and cache.
2. **`sql-server`:** Containerized Microsoft SQL Server (`mcr.microsoft.com/mssql/server`) with health check and volume persistence.
3. **`redis`:** Containerized Redis cache (`redis:alpine`).
4. **`mailpit`:** Containerized local SMTP server (`axllent/mailpit`) exposing port 8025 for web UI and 1025 for SMTP.

**Resources:**
- **⭐ Required Primary:** [Build a Complete .NET 10 Dev Environment with Docker Compose — Milan Jovanović (16:00)](https://youtu.be/svfxvsGfLlU).
- **🔄 Alternative / Deep Dive:** [Docker Compose Tutorial — Mosh Hamedani](https://youtu.be/HG6yIjZapSA)

---

### Task 5 — ASP.NET Core Health Checks

Implement health monitoring endpoints according to production standards:
- Install `AspNetCore.HealthChecks.SqlServer` and `AspNetCore.HealthChecks.Redis`.
- Expose `/health` returning overall system status:
  - Checks database connectivity.
  - Checks Redis connectivity.
  - Checks local storage availability.
- Configure dedicated endpoints for container orchestrators:
  - `/health/ready` (Readiness probe verifying all dependencies are ready).
  - `/health/live` (Liveness probe verifying API process is responsive).

**Resources:**
- **⭐ Required Primary:** [How to Add Health Checks in ASP.NET Core — Milan Jovanović (11:51)](https://youtu.be/4abSfjdzqms).
- **🔄 Alternative / Deep Dive:** [Best Way to Add Health Checks in .NET — Nick Chapsas](https://youtu.be/p2faw9DCSsY)

---

### Task 6 — Automated CI/CD Pipeline via GitHub Actions

Create `.github/workflows/ci.yml`:
- Trigger on every `push` and `pull_request` targeting `main` or `develop`.
- Job Steps:
  1. Check out repository code.
  2. Setup .NET 10 SDK.
  3. Restore NuGet dependencies (`dotnet restore`).
  4. Build solution in Release configuration (`dotnet build --no-restore -c Release`).
  5. Run all Unit Tests (`dotnet test --no-build -c Release`).
  6. Launch Docker service containers and run Integration Tests.
  7. Fail the pipeline if any test fails or compilation warnings violate quality gates.

**Resources:**
- **⭐ Required Primary:** [1. CI/CD Pipeline Using GitHub Actions — Coding-Future (23:29)](https://youtu.be/Zv-NJBRmoVU).
- **🔄 Alternative / Deep Dive:**
  - [GitHub Actions Full Tutorial — TechWorld with Nana](https://youtu.be/R8_veQiYBjI)
  - [GitHub Actions Complete Course — Ahmed El-Fakharany [Arabic]](https://youtu.be/7gJFHjXscr8)

---

### Task 7 — Professional Architecture Documentation & Final README.md

Write a comprehensive, publication-ready `README.md` for DonationHub documenting:
- Architectural diagram and layer responsibilities.
- Technology stack, libraries, and design patterns used.
- One-line quickstart command with Docker Compose.
- Environment variables and options configuration table.
- API endpoint summary and authentication guide.
- Testing and CI pipeline badges.

**Resources:**
- **⭐ Required Primary:** [How to Write a Professional README.md — Fekra w Code [Arabic] (14:27)](https://youtu.be/ATVJiw4JrUA).
- **🔄 Alternative / Deep Dive:** [Write an Awesome GitHub README — Learn Fast Make Things](https://youtu.be/E6NO0rgFub4)

---

## 📦 Sprint Deliverables

- [ ] Admin reporting endpoints executing optimized aggregate queries (`Sum`, `Count`, `GroupBy`).
- [ ] Caching layer for dashboard metrics in Redis with 5-minute invalidation.
- [ ] Multi-stage `Dockerfile` generating a lean, non-root runtime image.
- [ ] `docker-compose.yml` orchestrating API, SQL Server, Redis, and Mailpit.
- [ ] Health checks endpoint `/health` validating SQL Server and Redis connectivity.
- [ ] Working GitHub Actions CI workflow running automated builds and tests on every PR.
- [ ] Professional, complete `README.md` documenting the entire project.

---

## ✅ Definition of Done

- [ ] Running `docker compose up --build` spins up all 4 services without errors.
- [ ] Opening `http://localhost:8080/health` returns `Healthy` status in JSON.
- [ ] Reporting endpoints translate to efficient SQL `GROUP BY` without client-side evaluation.
- [ ] Pull requests trigger the GitHub Actions workflow and pass all test suites.
- [ ] The learner is prepared for the Final Graduation Defense to walk through the architecture, design patterns, and justify technical trade-offs.

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

### 3. Recommended Books
- *Clean Architecture: A Craftsman's Guide to Software Structure and Design* — Robert C. Martin
- *Dependency Injection Principles, Practices, and Patterns* — Steven van Deursen & Mark Seemann
- *Domain-Driven Design: Tackling Complexity in the Heart of Software* — Eric Evans
- *Software Architecture with C# 12 and .NET 8* — Gabriel Baptista & Francesco Abbruzzese
- *Designing Data-Intensive Applications* — Martin Kleppmann