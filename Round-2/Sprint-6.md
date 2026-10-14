# Sprint 6 — Reports, Admin, Docker, CI/CD & Health Checks

> Project: **DonationHub — Donation Management Platform**
> Duration: **2 weeks (~15 hours/week)**
> Evaluation: **100 points**

## Sprint Goal

Take DonationHub from "it works on my machine" to a system that can be built, tested, containerized, and deployed consistently. Implement the Reports and Admin modules, add health checks, and finalize the project.

## How to use this sprint plan

For every task:

1. Read the **MUST** resources before implementation.
2. Read **DEEP DIVE** only when the concept is unclear or the learner wants deeper understanding.
3. Implement the requested DonationHub change.
4. Verify the behavior manually.
5. Add/update tests when the task naturally requires them.
6. Commit the work in small, meaningful commits.
7. Be able to explain the architectural decision during review.

**Important:** This document does not change the original Sprint scope. It expands the original tasks into an executable checklist.

## Sprint Tasks

### Task 1 — Dashboard & Reports

Implement the Admin dashboard:

- **RPT-01:** Dashboard summary: total raised, number of succeeded donations, number of distinct donors, active/completed/expired campaign counts.
- **RPT-02:** Top N campaigns by raised amount.
- **RPT-03:** Donations over time (per day) for a date range.
- **RPT-04:** Dashboard responses are **cached** (default 5 minutes) and the numbers must match the database.

| Method | Path | Access |
|--------|------|--------|
| GET | `/admin/dashboard/summary` | Admin |
| GET | `/admin/dashboard/top-campaigns?count=5` | Admin |
| GET | `/admin/dashboard/donations-over-time?from=&to=` | Admin |

### Resources
- **MUST** — [EF Core — Efficient querying](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying)
- **MUST** — [ASP.NET Core — Distributed caching](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed)
- **VIDEO** — [Milan Jovanović — High-Performance Aggregations in EF Core](https://www.youtube.com/watch?v=cM3k0ZqP4f0)

---

### Task 2 — Admin User Management

Implement:

- **ADM-01:** Admin lists all users (paged, filter by role, search by name or email).
- **ADM-02:** Admin activates or deactivates a user. A deactivated user cannot log in.
- **ADM-03:** Admin lists, updates, and deletes staff users. Delete means deactivate. An Admin cannot deactivate themselves, and the last active Admin cannot be removed.

| Method | Path | Access |
|--------|------|--------|
| GET | `/admin/users` | Admin |
| PATCH | `/admin/users/{id}/status` | Admin |
| GET | `/admin/staff` | Admin |
| PUT | `/admin/staff/{id}` | Admin |
| DELETE | `/admin/staff/{id}` | Admin |

### Resources
- **MUST** — [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity)

---

### Task 3 — Health Checks

Implement:

- `/health/live` — the process is up (no dependency checks)
- `/health/ready` — SQL Server, Redis, and write access to the file storage folder
- Responses are JSON with each check's status

### Resources
- **MUST** — [ASP.NET Core — Health checks](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks)
- **VIDEO** — [Nick Chapsas — Health Checks in ASP.NET Core (.NET 8)](https://www.youtube.com/watch?v=kY8c2_J6F9Y)
- **VIDEO** — [Milan Jovanović — Health Checks Monitoring SQL & Redis](https://www.youtube.com/watch?v=d_Z9Q4u0n7A)

---

### Task 4 — Dockerfile

Create a production-oriented, multi-stage Dockerfile:

- Build stage: restore, build, publish
- Runtime stage: minimal image, **non-root user**
- Understand: build context, layers, build vs runtime image, ports, environment configuration

### Resources
- **MUST** — [Docker — Get Started](https://docs.docker.com/get-started/)
- **MUST** — [Docker — Build best practices](https://docs.docker.com/build/building/best-practices/)
- **VIDEO** — [Nick Chapsas — Dockerizing .NET Applications (Multi-Stage & Non-Root)](https://www.youtube.com/watch?v=bb_t_kY6zNo)
- **VIDEO** — [Milan Jovanović — Containerizing .NET 8 Applications with Docker](https://www.youtube.com/watch?v=p7_x1M4n2qA)

---

### Task 5 — Docker Compose

Create `docker-compose.yml` with services:

```text
api          (DonationHub API)
sqlserver    (SQL Server)
redis        (Redis)
mailpit      (Mailpit for development emails)
```

- Use **named volumes** for SQL data and uploaded files
- All settings through **environment variables**
- `docker compose up` must produce a working API with migrations applied and admin seeded
- Health checks and `depends_on` conditions for startup order

### Resources
- **MUST** — [Docker Compose — Getting started](https://docs.docker.com/compose/gettingstarted/)
- **MUST** — [Docker Compose — File reference](https://docs.docker.com/reference/compose-file/)
- **MUST** — [Docker Compose — Environment variables](https://docs.docker.com/compose/how-tos/environment-variables/)
- **VIDEO** — [Milan Jovanović — Docker Compose for .NET Developers](https://www.youtube.com/watch?v=9_Y_j1c8k8A)
- **VIDEO** — [Amichai Mantinband — Docker Compose for ASP.NET Core & Dependencies](https://www.youtube.com/watch?v=W7m_8n4_4wU)

---

### Task 6 — Environment Configuration & Secrets

- Move all environment-specific configuration outside source code:
  - Connection strings, JWT configuration, Redis, Stripe keys, email settings, file storage path
- Secrets are NOT committed to Git
- Use `user-secrets` for local development, environment variables for Docker
- `appsettings.json` contains only placeholders

### Resources
- **MUST** — [ASP.NET Core — Configuration](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)
- **MUST** — [GitHub Actions — Secrets](https://docs.github.com/en/actions/concepts/security/secrets)

---

### Task 7 — GitHub Actions CI

Create a GitHub Actions workflow (`.github/workflows/ci.yml`) that runs on every push and pull request:

```text
Restore
  ↓
Build
  ↓
Unit Tests
  ↓
Integration Tests
```

The CI badge appears at the top of the README.

### Resources
- **MUST** — [GitHub Actions — Building and testing .NET](https://docs.github.com/en/actions/tutorials/build-and-test-code/net)
- **MUST** — [GitHub Actions](https://docs.github.com/en/actions)
- **VIDEO** — [Nick Chapsas — GitHub Actions CI/CD for .NET Developers](https://www.youtube.com/watch?v=3g_t_N4K4hY)
- **VIDEO** — [Milan Jovanović — Automating .NET Builds & Tests with GitHub Actions](https://www.youtube.com/watch?v=k2Z_q6p8v3A)

---

### Task 8 — Docker Build in CI

Extend the CI workflow to build the Docker image:

```text
Build
  ↓
Test
  ↓
Docker Build
  ↓
Image
```

### Resources
- **MUST** — [Docker — Build documentation](https://docs.docker.com/build/)

---

### Task 9 — Local Production Verification

Run DonationHub through Docker Compose and verify:

- API starts and responds
- SQL Server is reachable, migrations applied
- Redis is reachable
- Mailpit receives emails
- Hangfire dashboard accessible
- Configuration injected correctly
- Authentication works
- Important endpoints work

### Resources
- **MUST** — [Docker Compose — Getting started](https://docs.docker.com/compose/gettingstarted/)

---

### Task 10 — Final Refactoring

Review the entire codebase:

- Architecture consistency
- Naming conventions
- Dead code removal
- Unnecessary abstractions
- Error handling completeness
- Business rules enforcement
- Configuration correctness
- Logging quality
- API consistency
- Test coverage of important flows

### Resources
- **DEEP DIVE** — Robert C. Martin — `Book: Clean Architecture`

---

### Task 11 — Final README

Write a comprehensive README documenting:

- Project overview
- Architecture explanation (why this structure)
- Domain model
- Authentication and authorization
- Data access strategy (repository for writes, DbContext for reads)
- Background processing (Hangfire jobs)
- Caching strategy and invalidation
- Payment flow (Stripe)
- Email notifications
- File handling
- Testing strategy
- How to run locally (with and without Docker)
- How to test Stripe locally (Stripe CLI)
- Docker and Docker Compose
- CI/CD
- List of design decisions and trade-offs

### Resources
- **MUST** — [GitHub — README documentation](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)

---

## Sprint Deliverables

- [ ] Dashboard summary, top campaigns, donations over time
- [ ] Dashboard cached
- [ ] Admin user management (list, activate/deactivate)
- [ ] Admin staff management (list, update, delete/deactivate)
- [ ] Self-deactivation and last-admin protection
- [ ] Health checks (liveness + readiness)
- [ ] Production Dockerfile (multi-stage, non-root)
- [ ] Docker Compose (api, sqlserver, redis, mailpit)
- [ ] Named volumes for data persistence
- [ ] Environment configuration (no secrets in repo)
- [ ] GitHub Actions CI workflow (restore, build, test)
- [ ] Docker image built in CI
- [ ] Local production verification through Docker Compose
- [ ] Final refactoring
- [ ] Final comprehensive README

## Definition of Done

- [ ] Dashboard returns correct numbers
- [ ] Dashboard responses cached
- [ ] Admin can list, search, and filter users
- [ ] Admin can activate/deactivate users
- [ ] Admin cannot deactivate themselves
- [ ] Last active Admin cannot be removed
- [ ] Health checks respond correctly
- [ ] Dockerfile builds successfully
- [ ] Docker Compose runs all services
- [ ] `docker compose up` produces a working API
- [ ] Migrations applied automatically
- [ ] Admin seeded automatically
- [ ] Configuration environment-based
- [ ] No secrets in repository
- [ ] GitHub Actions CI passes on tagged commit
- [ ] Docker image built in CI
- [ ] All tests pass with `dotnet test`
- [ ] README is complete and accurate
- [ ] Learner can explain architecture and trade-offs

## Review Rule

The learner should be able to explain not only **what** was implemented, but **why it belongs there, what alternatives existed, and what trade-off was accepted**.