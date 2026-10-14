# 🚀 Tawjeh .NET Level 2 — Advanced Backend Master Roadmap
### Comprehensive General Plan, Architecture & Unabridged Sprint Specifications

> **Program:** Tawjeh Mentorship Program — Level 2  
> **Core Project:** **DonationHub** (Donation Management & Charity Crowdfunding Platform)  
> **Duration:** 12 Weeks (6 Sprints × 2 Weeks) + Final Defense Period (9 Days)  
> **Total Effort:** ~180 Hours (~15 Hours / Week)  
> **Evaluation:** 600 Points Total (100 Points / Sprint)  
> **Target Audience:** Junior to Intermediate .NET Developers transitioning to Production-Ready Backend Engineers  

---

## 📖 Table of Contents

1. [🎯 Level 2 Mission & Educational Philosophy](#-level-2-mission--educational-philosophy)
2. [🏗️ Project Spotlight: DonationHub](#️-project-spotlight-donationhub)
3. [👥 Roles & Permissions Matrix](#-roles--permissions-matrix)
4. [🏛️ Architectural Blueprint & Layering Rules](#️-architectural-blueprint--layering-rules)
5. [🧭 Mentorship Schedule & Timeline](#-mentorship-schedule--timeline)
6. [🟦 Sprint 1 — Clean Architecture, Identity & Categories](#-sprint-1--clean-architecture-identity--categories)
7. [🟩 Sprint 2 — Campaigns, CQRS, MediatR & Unit Testing](#-sprint-2--campaigns-cqrs-mediatr--unit-testing)
8. [🟨 Sprint 3 — Donations, Authorization & API Engineering](#-sprint-3--donations-authorization--api-engineering)
9. [🟧 Sprint 4 — Notifications, Files, Hangfire & Serilog](#-sprint-4--notifications-files-hangfire--serilog)
10. [🟥 Sprint 5 — Payments (Stripe), Idempotency & Integration Testing](#-sprint-5--payments-stripe-idempotency--integration-testing)
11. [🟪 Sprint 6 — Reports, Admin, Docker, CI/CD & Health Checks](#-sprint-6--reports-admin-docker-cicd--health-checks)
12. [🏁 Final Period, Defense & Graduation Guidelines](#-final-period-defense--graduation-guidelines)
13. [📏 Universal Architectural Rules & Definition of Done](#-universal-architectural-rules--definition-of-done)
14. [🚫 Explicitly Out of Scope](#-explicitly-out-of-scope)
15. [📂 Quick Navigation & Documentation Links](#-quick-navigation--documentation-links)

---

## 🎯 Level 2 Mission & Educational Philosophy

Most backend developers get stuck in **"Tutorial Hell"** — they know how to generate an API controller, bind a model with Entity Framework Core, and write basic CRUD operations. However, when faced with real-world enterprise requirements, critical gaps appear:

- *How do we ensure financial accuracy when multiple donors donate to the same campaign simultaneously?*
- *How do we handle network timeouts during external payment gateway calls without double-charging?*
- *Why is sending emails and generating PDFs inside an HTTP request lifecycle an anti-pattern?*
- *How do we isolate use cases so modifying one feature never inadvertently breaks another?*
- *How do we design automated tests that verify critical domain behaviors rather than testing EF Core's built-in methods?*

**Level 2 bridges this exact gap.** It transforms learners from junior CRUD developers into confident, production-grade backend engineers.

```text
Junior CRUD Developer                          Production Backend Engineer
┌───────────────────────┐                     ┌──────────────────────────────┐
│ • Models = DB Tables  │                     │ • Rich Domain Invariants     │
│ • Fat Controllers     │     Tawjeh Level 2  │ • Clean Arch + CQRS Slices   │
│ • Untested Code       │  ────────────────►  │ • Idempotent Stripe Payments │
│ • Heavy Work in HTTP  │                     │ • Background Job Workers     │
│ • "Works on Machine"  │                     │ • Containerized CI/CD System │
└───────────────────────┘                     └──────────────────────────────┘
```

### The Production Engineering Cycle:
```text
Understand  ➔  Model  ➔  Design  ➔  Implement  ➔  Debug  ➔  Refactor  ➔  Test  ➔  Deploy  ➔  Explain
```

### The Single Evolving Project Principle
Learners will **not** build six disconnected toys. You will build, evolve, refactor, and defend a single production platform — **DonationHub** — watching it transform from an empty solution into an enterprise-grade backend across six sprints:

```text
Sprint 1: Architecture, Rich Invariants, Persistence & Identity
    ↓
Sprint 2: CQRS Slices, MediatR Pipeline, Validation, Result Pattern & Unit Tests
    ↓
Sprint 3: Core Donation Flows, Resource Ownership, Distributed Caching & Localization
    ↓
Sprint 4: Asynchronous Processing, Hangfire, MailKit, QuestPDF & Serilog
    ↓
Sprint 5: Stripe Checkout, Webhook Verification, Idempotency & Integration Testing
    ↓
Sprint 6: Analytics Dashboard, Admin Management, Docker, Health Checks & CI/CD
```

---

## 🏗️ Project Spotlight: DonationHub

**DonationHub** is a financial donation and charity crowdfunding platform. It empowers charitable organizations and campaign managers to launch fundraising campaigns transparently, while enabling donors worldwide to contribute securely via credit cards and obtain certified tax receipts.

### Why DonationHub?
Unlike simple task managers or blogs, a donation platform introduces real financial and architectural stakes:
1. **Financial Integrity:** Money cannot be lost, double-counted, or corrupted by race conditions.
2. **Third-Party Payment Gateways:** Real integration with **Stripe Checkout**, handling asynchronous webhooks and payment failures.
3. **Strict State Transitions:** A campaign moves through well-defined states (`Active` ➔ `Completed` or `Expired`), and donations transition from `Pending` ➔ `Succeeded` or `Failed`.
4. **Decoupled Asynchronous Workloads:** Generating PDF receipts and sending transactional emails must happen reliably in background workers (**Hangfire**), completely decoupled from HTTP responses.
5. **Idempotency:** Network retries must never result in duplicate charges or duplicate donation records.

---

## 👥 Roles & Permissions Matrix

DonationHub enforces strict, real-world authorization policies:

| Feature / Action | Anonymous | Donor | Campaign Manager | System Admin |
|---|:---:|:---:|:---:|:---:|
| Browse active campaigns & success stories | ✅ | ✅ | ✅ | ✅ |
| View campaign details & categories | ✅ | ✅ | ✅ | ✅ |
| Register, Log in, Refresh token | ✅ | ✅ | ✅ | ✅ |
| Make a donation via Stripe | ❌ | ✅ | ✅ | ✅ |
| View own donation history & download own PDF receipts | ❌ | ✅ | ✅ | ✅ |
| Create new campaigns | ❌ | ❌ | ✅ | ✅ |
| Edit, close, or delete campaigns | ❌ | ❌ | ✅ *(Own campaigns only)* | ✅ *(All campaigns)* |
| Upload campaign banner image | ❌ | ❌ | ✅ *(Own campaigns only)* | ✅ *(All campaigns)* |
| Manage categories (Create, Update, Delete) | ❌ | ❌ | ❌ | ✅ |
| View all platform donations & export reports | ❌ | ❌ | ❌ | ✅ |
| Access Admin Analytics Dashboard | ❌ | ❌ | ❌ | ✅ |
| Manage Users & Staff Accounts | ❌ | ❌ | ❌ | ✅ |

> [!NOTE]
> **Security Rule:** When a user attempts to access a private resource belonging to someone else (e.g., another donor's receipt or another manager's unpublished campaign), the API must return **`404 Not Found`** instead of `403 Forbidden` to prevent resource enumeration attacks.

---

## 🏛️ Architectural Blueprint & Layering Rules

The platform implements **Clean Architecture** with a **Vertical Slice** feature organization inside the Application layer:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                          DonationHub.Api                               │
│  Controllers (Thin) │ Middleware (Idempotency, Rate Limiter) │ Swagger │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Depends on
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                      DonationHub.Application                           │
│  Organized by Feature Slices (e.g., Features/Donations/CreateDonation) │
│  Commands │ Queries │ MediatR Handlers │ FluentValidation │ DTOs       │
└───────────────────┬────────────────────────────────┬───────────────────┘
                    │ Depends on                     │ Implemented by
                    ▼                                ▼
┌──────────────────────────────────────┐  ┌──────────────────────────────┐
│         DonationHub.Domain           │  │  DonationHub.Infrastructure  │
│  Entities │ Invariants │ Value Objects│  │  EF Core │ SQL Server │ Redis│
│  Enums │ State Machines              │  │  MailKit │ Hangfire │ Stripe │
│  (Zero External Dependencies)        │  │  QuestPDF │ File Storage     │
└──────────────────────────────────────┘  └──────────────────────────────┘
```

### Core Architecture Rules:
1. **Domain has ZERO dependencies:** No EF Core packages, no ASP.NET packages, no third-party libraries.
2. **Application defines abstractions:** Interfaces for repositories, email service, file storage, payment gateway, and current user reside in Application.
3. **Infrastructure implements abstractions:** All external concerns (databases, email, Stripe, PDF, background jobs) are isolated in Infrastructure.
4. **Thin Controllers:** Controllers only accept HTTP requests, delegate to MediatR via `ISender`, and translate the resulting `Result<T>` into HTTP responses.
5. **No Entity Leakage:** Domain entities are **never** returned directly by API endpoints. DTOs are mandatory.

---

## 🧭 Mentorship Schedule & Timeline

| Phase | Sprint / Milestone | Start Date | End Date | Focus & Core Deliverables |
|:---:|---|:---:|:---:|---|
| **Phase 1** | **Sprint 1: Architecture, Identity & Categories** | 20 Oct 2026 | 02 Nov 2026 | Clean Arch, Domain Models, EF Core, Options, JWT & Refresh Tokens, Categories CRUD |
| **Phase 1** | **Sprint 2: CQRS, MediatR & Unit Testing** | 03 Nov 2026 | 16 Nov 2026 | CQRS slices, MediatR, FluentValidation, Result pattern, ProblemDetails, Campaigns, Unit Tests |
| **Phase 2** | **Sprint 3: Donations, AuthZ & API Standards** | 17 Nov 2026 | 30 Nov 2026 | Ownership AuthZ, Donations module, Concurrency, Caching (Redis), Localization, Versioning, Rate Limiting |
| **Phase 2** | **Sprint 4: Hangfire, Files & Serilog** | 01 Dec 2026 | 14 Dec 2026 | Hangfire jobs, MailKit email, QuestPDF receipts, Magic-byte uploads, Serilog structured logging |
| **Phase 3** | **Sprint 5: Stripe Payments & Idempotency** | 15 Dec 2026 | 28 Dec 2026 | Stripe Checkout, Webhooks, Idempotency-Key middleware, WebApplicationFactory integration tests |
| **Phase 3** | **Sprint 6: Docker, CI/CD & Final Polish** | 29 Dec 2026 | 11 Jan 2027 | Admin analytics, Multi-stage Docker, Docker Compose stack, Health checks, GitHub Actions CI |
| **Defense** | **Final Period: Hardening & Oral Defense** | 12 Jan 2027 | 20 Jan 2027 | Architectural review, production readiness inspection, live demo & oral defense |

---

## 🟦 Sprint 1 — Clean Architecture, Identity & Categories

* **Timeline:** 20 Oct 2026 → 02 Nov 2026 | **Effort:** ~30 Hours | **Evaluation:** 100 Points
* **Sprint Goal:** Establish the architectural foundation of DonationHub and deliver the first two production modules: Identity (with secure JWT & refresh token rotation) and Categories.

### Detailed Sprint Tasks

#### Task 1 — Solution Structure (Clean Architecture)
Create the four-layer solution adhering strictly to dependency inversion:
* `DonationHub.Domain` (Zero dependencies)
* `DonationHub.Application` (Depends only on Domain)
* `DonationHub.Infrastructure` (Depends on Application and Domain)
* `DonationHub.Api` (Depends on Application and Infrastructure as the Composition Root)
* Inside `Application`, organize code by **feature slices** (`Features/Categories`, `Features/Auth`), not technical folders.
* **Resources:**
  * **MUST:** [Microsoft — Common web application architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
  * **MUST:** [Microsoft — Clean Architecture with ASP.NET Core](https://learn.microsoft.com/en-us/shows/dotnetconf-2022/clean-architecture-with-aspnet-core-7)
  * **VIDEO:** [Milan Jovanović — Clean Architecture from Scratch](https://www.youtube.com/watch?v=yF9SwL0p0Y0)
  * **VIDEO:** [Jimmy Bogard — Vertical Slice Architecture](https://www.youtube.com/watch?v=oAoaMlS1PWo)

#### Task 2 — Domain Model
Implement the rich domain model with business invariants and state machines:
* **Entities:** `ApplicationUser`, `RefreshToken`, `Category`, `Campaign`, `Donation`, `PaymentAttempt`, `ProcessedStripeEvent`, `IdempotencyRecord`.
* **Enums:** `CampaignStatus` (`Active`, `Completed`, `Expired`), `DonationStatus` (`Pending`, `Succeeded`, `Failed`), `PaymentAttemptStatus`.
* **Invariants:** `GoalAmount > 0`, `EndDate` strictly in future at creation, non-empty bilingual names, valid state machine transitions.
* **Resources:**
  * **MUST:** [Microsoft — DDD and CQRS architecture guidance](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/)
  * **VIDEO:** [Milan Jovanović — Domain-Driven Design with C#](https://www.youtube.com/watch?v=F3_l0Tst0tY)

#### Task 3 — EF Core Persistence
Configure persistence in Infrastructure:
* `DonationHubDbContext` with separate `IEntityTypeConfiguration<T>` classes.
* Concurrency token: `RowVersion` (`byte[]`) configured on `Campaign`.
* Soft delete on `Campaign` (`IsDeleted`) with a Global Query Filter (`builder.Entity<Campaign>().HasQueryFilter(c => !c.IsDeleted)`).
* Indexes on frequently filtered fields: `Campaign.CategoryId`, `Campaign.Status`, `Donation.CampaignId`, `Donation.DonorId`.
* Initial migration generated and applied to SQL Server.
* **Resources:**
  * **MUST:** [EF Core Relationships](https://learn.microsoft.com/en-us/ef/core/modeling/relationships) | [Indexes](https://learn.microsoft.com/en-us/ef/core/modeling/indexes) | [Global query filters](https://learn.microsoft.com/en-us/ef/core/querying/filters)
  * **VIDEO:** [Milan Jovanović — EF Core Fluent API & Relationships](https://www.youtube.com/watch?v=kYJqD2v5bX8)
  * **VIDEO:** [Nick Chapsas — Global Query Filters & Soft Delete](https://www.youtube.com/watch?v=Zc2gZ79r_2M)

#### Task 4 — Generic Repository & Unit of Work
Implement persistence patterns:
* `IGenericRepository<T>` and `IUnitOfWork` in Application; implemented in Infrastructure.
* **Architectural Rule:** Write operations go through the Repository and Unit of Work. Read queries bypass repositories and query `DbContext` directly with `AsNoTracking()` and projection to DTOs for maximum performance.
* **Resources:**
  * **VIDEO:** [DevCreed (محمد الهلالي) — Repository Pattern & Unit of Work (Arabic)](https://youtube.com/playlist?list=PL62tSREI9C-e6nQ47brLj00iSGddiee73)
  * **VIDEO:** [Milan Jovanović — Generic Repository & Unit of Work](https://www.youtube.com/watch?v=xdibesoH8zE)

#### Task 5 — Options Pattern with Validation
* Create strongly-typed options: `JwtOptions`, `SeedAdminOptions`, `DonationOptions`.
* Register with `.ValidateDataAnnotations().ValidateOnStart()`. The application must fail immediately at startup if configuration is invalid or missing.
* **Resources:**
  * **MUST:** [ASP.NET Core — Options pattern](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
  * **VIDEO:** [Nick Chapsas — Options Pattern & ValidateOnStart](https://www.youtube.com/watch?v=b4O2QkM_S7A)

#### Task 6 — Identity & Authentication
Implement full user authentication with ASP.NET Core Identity:
* **IDN-01:** Donor registration (Email, Password, Full Name, Preferred Language).
* **IDN-02:** Unique email (case-insensitive), password complexity policy (8+ chars, upper, lower, digit).
* **IDN-03:** Login endpoint returning short-lived JWT access token + refresh token.
* **IDN-04:** JWT Access Token: 15-minute lifetime with claims: `sub` (User ID), `email`, `role`.
* **IDN-05:** Refresh Token: 7-day lifetime, stored **cryptographically hashed** (SHA-256) in database.
* **IDN-06:** Refresh Token Rotation: on `/auth/refresh`, revoke old token, issue linked new token.
* **IDN-07:** Logout endpoint revokes current refresh token immediately.
* **IDN-09:** `GET /auth/me` returns authenticated user profile.
* **IDN-10:** Deactivated user cannot log in or refresh tokens.
* **IDN-11:** Automatic Admin user seeding at startup from configuration options.
* **IDN-12:** Unified generic error message for invalid login credentials (anti-enumeration).
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | POST | `/auth/register` | Anonymous |
  | POST | `/auth/login` | Anonymous |
  | POST | `/auth/refresh` | Anonymous |
  | POST | `/auth/logout` | Authenticated |
  | GET | `/auth/me` | Authenticated |
* **Resources:**
  * **MUST:** [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity) | [Configure JWT bearer](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication)
  * **VIDEO:** [Milan Jovanović — JWT Authentication & Refresh Tokens](https://www.youtube.com/watch?v=M6Akbba9hnA)
  * **VIDEO:** [Khalid Essaadani — ASP.NET Identity in Arabic](https://www.youtube.com/playlist?list=PLwj1YcMhLRN1T3fIb-JDa4xNFfVQoljGI)

#### Task 7 — Staff User Creation
* **IDN-08:** `POST /admin/staff` allows Admin to create staff users (`Admin` or `CampaignManager`) with temporary passwords.
* **Resources:**
  * **MUST:** [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity)

#### Task 8 — Categories Module
Implement complete CRUD for Categories with domain validation:
* **CAT-01:** Admin creates, updates, and deletes categories with Arabic and English names/descriptions.
* **CAT-02:** Names required in both languages, max 100 characters, unique per language.
* **CAT-03:** Public listing and single category lookup.
* **CAT-04:** Deleting a category that has linked campaigns is rejected (`Category.HasCampaigns`).
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | GET | `/categories` | Anonymous |
  | GET | `/categories/{id}` | Anonymous |
  | POST | `/categories` | Admin |
  | PUT | `/categories/{id}` | Admin |
  | DELETE | `/categories/{id}` | Admin |
* **Resources:**
  * **VIDEO:** [Passionate Coders (Mohamed El-Mehdi) — ASP.NET Core Web API (Arabic)](https://www.youtube.com/playlist?list=PLsV97AQt78NQ8E7cEqovH0zLYRJgJahGh)

#### Task 9 — Swagger & API Documentation
Configure Swashbuckle with JWT Bearer Authentication schemes so protected endpoints can be tested interactively.
* **Resources:**
  * **VIDEO:** [Milan Jovanović — Swagger with JWT Bearer Authentication](https://www.youtube.com/watch?v=cM3k0ZqP4f0)

#### Task 10 — Repository Documentation
Add developer setup guide: connection strings, running migrations via CLI, test credentials, and architecture summary.

---

### Sprint 1 Deliverables & Definition of Done
- [ ] 4-layer Clean Architecture solution compiled without warnings.
- [ ] Rich domain entities with invariants and state machines.
- [ ] EF Core Fluent API mappings, indexes, soft delete filter, and migrations.
- [ ] Options pattern configured with `ValidateOnStart()`.
- [ ] Identity auth: register, login, refresh rotation, revocation, profile.
- [ ] Category CRUD with campaign association validation.
- [ ] Swagger Bearer auth enabled.
- [ ] Evaluation Rubric (100 Points): Architecture (25) + Identity (30) + Categories (20) + Data Persistence (15) + Code Quality (10).

---

## 🟩 Sprint 2 — Campaigns, CQRS, MediatR & Unit Testing

* **Timeline:** 03 Nov 2026 → 16 Nov 2026 | **Effort:** ~30 Hours | **Evaluation:** 100 Points
* **Sprint Goal:** Convert all use cases to CQRS vertical slices using MediatR, implement automatic FluentValidation pipelines, standard RFC 9457 ProblemDetails, the Campaigns module, and a comprehensive unit test suite.

### Detailed Sprint Tasks

#### Task 1 — CQRS Conversion
Refactor all existing use cases (Identity and Categories) into distinct Command and Query feature slices:
* `Features/<FeatureName>/<UseCase>/<UseCaseCommand|Query>.cs`
* `Features/<FeatureName>/<UseCase>/<UseCaseHandler>.cs`
* `Features/<FeatureName>/<UseCase>/<UseCaseValidator>.cs`
* `Features/<FeatureName>/<UseCase>/<UseCaseDto>.cs`
* **Resources:**
  * **MUST:** [Microsoft — DDD/CQRS architecture guidance](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/)
  * **VIDEO:** [Nick Chapsas — CQRS in .NET](https://www.youtube.com/watch?v=2JzQuIvxIqk)
  * **ARTICLE:** [CodeWithMukesh — CQRS and MediatR in ASP.NET Core](https://codewithmukesh.com/blog/cqrs-and-mediatr-in-aspnet-core/)

#### Task 2 — MediatR Dispatching & Thin Controllers
Introduce MediatR for all requests:
* Controllers contain zero business logic. They simply construct the command/query, dispatch via `ISender.Send()`, and translate the result into HTTP responses.
* **Resources:**
  * **VIDEO:** [Derek Comartin (CodeOpinion) — Why Use MediatR?](https://codeopinion.com/why-use-mediatr-3-reasons-why-and-1-reason-not/)
  * **VIDEO:** [Nick Chapsas — MediatR Pipeline Behaviors](https://www.youtube.com/watch?v=2JzQuIvxIqk)

#### Task 3 — Automatic Validation Pipeline Behavior
Implement `ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>`:
* Collects all registered `IValidator<TRequest>` for the request.
* Runs validation asynchronously. If failures exist, short-circuits the pipeline and returns a structured validation error before the handler is ever invoked.
* **Resources:**
  * **MUST:** [FluentValidation Documentation](https://docs.fluentvalidation.net/)
  * **VIDEO:** [Milan Jovanović — Validation with FluentValidation & MediatR](https://www.youtube.com/watch?v=rQ46aI0y9z0)

#### Task 4 — Result Pattern & Standard Error Catalogue
Replace exception-based control flow with functional results:
* Implement `Result` and `Result<T>` with `Error` types (`Validation`, `NotFound`, `Conflict`, `Forbidden`).
* Map standard error codes: `Validation.Failed` (400), `Auth.InvalidCredentials` (401), `Auth.EmailAlreadyExists` (409), `Category.NotFound` (404), `Category.HasCampaigns` (409), `Campaign.NotFound` (404), `Campaign.GoalBelowRaised` (422), `Campaign.EndDateInPast` (422), `Campaign.ConcurrencyConflict` (409).
* **Resources:**
  * **VIDEO:** [Milan Jovanović — Functional Error Handling with Result Pattern](https://www.youtube.com/watch?v=WCCkyeekmDo)
  * **VIDEO:** [Amichai Mantinband — Clean Architecture Error Handling](https://www.youtube.com/watch?v=4n_sW4hH0Zk)

#### Task 5 — Global Exception Handling (IExceptionHandler & ProblemDetails)
* Register an `IExceptionHandler` in .NET 8 returning RFC 9457 `ProblemDetails`.
* Catches unexpected 500 exceptions, logs the full exception stack trace with a unique `traceId`, and returns a safe sanitized response to clients.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — New Exception Handling in .NET 8 (IExceptionHandler)](https://www.youtube.com/watch?v=4V_X7b9dFMo)
  * **VIDEO:** [Milan Jovanović — Global Error Handling in ASP.NET Core](https://www.youtube.com/watch?v=x_u09X_X74s)

#### Task 6 — Campaigns Module
Implement the complete Campaigns feature slice:
* **CMP-01:** Create campaign (Arabic & English title/description, goal amount, end date, category id). Initial status is `Active`, `RaisedAmount = 0`.
* **CMP-02:** Update campaign, verifying invariants: goal cannot be set lower than currently raised amount, end date must remain in future.
* **CMP-03:** Soft-delete campaign (`IsDeleted = true`), automatically hidden by global query filter.
* **CMP-04:** Public active campaign list and public "success stories" list (`Completed` status).
* **CMP-05:** Campaign details returning calculated fields: `progressPercent` (capped at 100) and `daysLeft`.
* **CMP-06:** In Sprint 2, Admin creates and manages campaigns (Campaign Manager ownership arrives in Sprint 3).
* **CMP-10:** Optimistic concurrency conflict detection using `RowVersion`.
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | GET | `/campaigns` | Anonymous |
  | GET | `/campaigns/success-stories` | Anonymous |
  | GET | `/campaigns/{id}` | Anonymous |
  | POST | `/campaigns` | Admin |
  | PUT | `/campaigns/{id}` | Admin |
  | DELETE | `/campaigns/{id}` | Admin |
* **Resources:**
  * **VIDEO:** [Milan Jovanović — Optimistic Concurrency in EF Core](https://www.youtube.com/watch?v=bF9EwE0_2H4)
  * **VIDEO:** [Nick Chapsas — Concurrency Conflicts in EF Core](https://www.youtube.com/watch?v=pM9mYcEw28Q)

#### Task 7 — CancellationToken Propagation
Ensure all asynchronous methods from Controller → MediatR → Handler → EF Core accept and propagate `CancellationToken`.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — How to Cancel Tasks with CancellationToken in .NET](https://www.youtube.com/watch?v=8V-w6Gj9w-A)

#### Task 8 — TimeProvider Abstraction
Eliminate all direct calls to `DateTime.UtcNow`. Inject `TimeProvider` across domain logic, handlers, and validation so time-dependent tests can be deterministically simulated.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — Stop Using DateTime.UtcNow in .NET 8 (TimeProvider)](https://www.youtube.com/watch?v=S8L3kK_s7hA)

#### Task 9 — Unit Testing Suite (15+ Tests)
Create a dedicated test project `DonationHub.Application.UnitTests` using xUnit, FluentAssertions, and NSubstitute:
* Test domain invariants: goal amount positive, end date in future, state transitions.
* Test business rules: cannot delete category with active campaigns, cannot lower goal below raised amount.
* Test MediatR handlers with mocked repositories and fixed `TimeProvider`.
* Test FluentValidation rules.
* **Resources:**
  * **VIDEO:** [Teddy Smith — xUnit Full Course for Beginners](https://www.youtube.com/watch?v=fVVNOkbLcPw)
  * **VIDEO:** [Mohamed El-Mehdi — Unit Testing in C# (Arabic)](https://www.youtube.com/watch?v=lY4Q8fZYSN8)
  * **VIDEO:** [Nick Chapsas — Effective Testing Workshop](https://www.youtube.com/watch?v=kJFB8o78FBs)

---

### Sprint 2 Deliverables & Definition of Done
- [ ] All use cases refactored into CQRS slices with MediatR.
- [ ] Validation pipeline behavior active; zero manual validation in handlers.
- [ ] Result pattern and ProblemDetails returned across all endpoints.
- [ ] Campaigns CRUD with optimistic concurrency control (`RowVersion`).
- [ ] TimeProvider and CancellationToken propagated everywhere.
- [ ] At least 15 passing unit tests.
- [ ] Evaluation Rubric (100 Points): CQRS & MediatR (25) + Campaigns Module (25) + Error Handling & Validation (20) + Unit Testing (20) + Clean Code & Timing (10).

---

## 🟨 Sprint 3 — Donations, Authorization & API Engineering

* **Timeline:** 17 Nov 2026 → 30 Nov 2026 | **Effort:** ~30 Hours | **Evaluation:** 100 Points
* **Sprint Goal:** Deliver the Donations module with transactional integrity, enforce resource-based authorization, implement distributed caching with Redis, bilingual localization, API versioning, rate limiting, and CORS.

### Detailed Sprint Tasks

#### Task 1 — Authorization Policies & Resource Ownership
* **CMP-07:** Implement ASP.NET Core Resource-Based Authorization:
  * Campaign Managers can create campaigns and modify/delete **only campaigns they created**.
  * Admins can modify and delete all campaigns.
* **Security Anti-Enumeration Rule:** When a user requests a private resource belonging to someone else, return **`404 Not Found`** (not 403) so the requester cannot discover whether the resource exists.
* **Endpoints:**
  | Endpoint | Campaign Manager | Admin |
  |---|---|---|
  | `POST /campaigns` | ✅ Create own | ✅ Create |
  | `PUT /campaigns/{id}` | ✅ Own only | ✅ All |
  | `DELETE /campaigns/{id}` | ✅ Own only | ✅ All |
  | `GET /campaigns/mine` | ✅ Own campaigns | N/A |
* **Resources:**
  * **MUST:** [Resource-based authorization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resource-based)
  * **VIDEO:** [Milan Jovanović — Resource-Based Authorization in ASP.NET Core](https://www.youtube.com/watch?v=24r7UmuRh0A)
  * **VIDEO:** [Raw Coding — Resource-Based Authorization](https://www.youtube.com/watch?v=680Qp1r4ZFc)

#### Task 2 — Donations Module
Implement the core donation lifecycle:
* **DON-01:** Authenticated user creates a donation for an active campaign. Starts in `Pending` status.
* **DON-02:** Rejections: campaign is inactive, end date has passed, or amount is outside allowed bounds (configured in `DonationOptions`).
* **DON-03:** Donor lists their own donations with pagination (`/donations/mine`).
* **DON-04:** Donor views single donation details by ID. Accessing another donor's record yields 404.
* **DON-05:** Admin lists all donations across the platform with multi-parameter filtering (status, campaign, donor, dates).
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | POST | `/donations` | Donor, Manager, Admin |
  | GET | `/donations/mine` | Authenticated |
  | GET | `/donations/{id}` | Owner, Admin |
  | GET | `/admin/donations` | Admin |
* **Resources:**
  * **MUST:** [EF Core Querying & Relationships](https://learn.microsoft.com/en-us/ef/core/querying/)

#### Task 3 — Transactional Donation Success Flow (Temporary Stub)
* **DON-06:** `POST /admin/donations/{id}/mark-succeeded` simulates payment success (**Temporary stub to be removed in Sprint 5**).
* **DON-07:** Atomic transaction flow:
  1. Set donation status to `Succeeded`, record `CompletedAt`, generate sequential `ReceiptNumber` (`DN-{yyyy}-{000000}`).
  2. Increment campaign `RaisedAmount` by donation amount.
  3. If `RaisedAmount >= GoalAmount`, update campaign status to `Completed`.
  4. Commit transaction.
* **DON-08:** Concurrency-safe raised amount update: handle simultaneous donations without lost updates using optimistic concurrency (`RowVersion`) or atomic SQL increment.
* **Resources:**
  * **MUST:** [EF Core Transactions](https://learn.microsoft.com/en-us/ef/core/saving/transactions)
  * **VIDEO:** [Nick Chapsas — Concurrency Conflicts in EF Core](https://www.youtube.com/watch?v=pM9mYcEw28Q)

#### Task 4 — Pagination, Filtering, Sorting & Searching
* Standardize query responses: `{ "items": [...], "page": 1, "pageSize": 20, "totalCount": 100, "totalPages": 5 }`.
* Bounded page sizes: default 20, maximum 50.
* Deterministic ordering by primary key to prevent duplicate items across pages.
* Safe sorting whitelist: only allowed fields (`endDate`, `raisedAmount`, `createdAt`) can be sorted.
* **Resources:**
  * **VIDEO:** [Milan Jovanović — Adding Filtering, Sorting, and Pagination](https://www.youtube.com/watch?v=X8zRvXbirMU)
  * **ARTICLE:** [Code Maze — Pagination in ASP.NET Core Web API](https://code-maze.com/paging-aspnet-core-webapi/)

#### Task 5 — Distributed Caching with Redis (Cache-Aside Pattern)
* Implement Cache-Aside with `IDistributedCache` backed by **Redis**:
  * Cache public categories list, public campaign list pages, and campaign details.
  * Cache keys include language and serialized query parameters: `campaigns:p1:s20:cat2:lang_en`.
  * **Cache Invalidation:** Invalidate cached keys upon campaign creation, update, deletion, or donation completion.
  * **Security Rule:** Never cache user-specific data (e.g., personal donations) under shared keys.
* **Resources:**
  * **MUST:** [ASP.NET Core Distributed Caching](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed)
  * **VIDEO:** [Nick Chapsas — Distributed Caching with Redis](https://www.youtube.com/watch?v=7_E9q4hZp8w)
  * **VIDEO:** [Milan Jovanović — Cache-Aside Pattern with Redis](https://www.youtube.com/watch?v=sW2wN1n3z6U)

#### Task 6 — Bilingual Localization (Arabic & English)
* **CMP-11:** Read `Accept-Language` header (`ar` or `en`, fallback `en`).
* Public API responses return titles and descriptions in the requested language. Admin endpoints return both.
* Validation messages and business error messages are localized using `.resx` resource files.
* **Resources:**
  * **VIDEO:** [Mohamad Lawand — Localization and Globalization in Web API](https://www.youtube.com/watch?v=aG49R_s4y_U)
  * **ARTICLE:** [Code Maze — ASP.NET Core Web API Localization](https://code-maze.com/aspnetcore-web-api-localization/)

#### Task 7 — API Versioning
Implement URL-segment API versioning (`/api/v1/...`) using `Asp.Versioning.Mvc`.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — API Versioning in ASP.NET Core (.NET 8)](https://www.youtube.com/watch?v=2wOCkNm1_zY)

#### Task 8 — Built-in Rate Limiting Middleware
Configure ASP.NET Core Rate Limiter with named policies:
* `global`: 100 req/min per IP.
* `auth`: 10 req/min per IP (login, register, refresh).
* `donations`: 20 req/min per User ID.
* Returns `429 Too Many Requests` with `Retry-After` header. Webhooks are explicitly excluded.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — Built-in Rate Limiting in .NET](https://www.youtube.com/watch?v=9_p_H6c0zXQ)
  * **VIDEO:** [Milan Jovanović — Rate Limiting in ASP.NET Core](https://www.youtube.com/watch?v=6l7Z1c2k4n8)

#### Task 9 — CORS Configuration
Named CORS policy configured from `CorsOptions` with explicit allowed origins. Credentials are never enabled with wildcard origins.

#### Task 10 — Unit Tests for Donations
Add automated tests for donation business invariants, atomic raised amount updates, and ownership rules.

---

### Sprint 3 Deliverables & Definition of Done
- [ ] Resource-based authorization enforcing campaign ownership.
- [ ] Donations module with atomic transactional completion.
- [ ] Deterministic pagination, filtering, and safe sorting.
- [ ] Redis distributed caching with cache invalidation.
- [ ] Localization via `Accept-Language` header.
- [ ] API versioning (`/api/v1/`), rate limiting, and CORS active.
- [ ] Evaluation Rubric (100 Points): Donations Business Logic (25) + Authorization & Ownership (20) + Caching & Performance (20) + API Standards (20) + Testing & Documentation (15).

---

## 🟧 Sprint 4 — Notifications, Files, Hangfire & Serilog

* **Timeline:** 01 Dec 2026 → 14 Dec 2026 | **Effort:** ~30 Hours | **Evaluation:** 100 Points
* **Sprint Goal:** Offload heavy and asynchronous workloads to Hangfire background queues, generate dynamic PDF receipts with QuestPDF, send localized transactional emails with MailKit, validate file uploads by magic bytes, and configure Serilog structured logging.

### Detailed Sprint Tasks

#### Task 1 — Hangfire Background Job Processing
* Configure Hangfire with SQL Server persistent storage.
* Enable fire-and-forget, delayed, and recurring job capabilities.
* **JOB-05:** Hangfire dashboard is enabled and secured using basic authentication from `HangfireOptions`.
* **Resources:**
  * **MUST:** [Hangfire — Getting started](https://docs.hangfire.io/en/latest/getting-started/) | [Using SQL Server](https://docs.hangfire.io/en/latest/configuration/using-sql-server.html)
  * **VIDEO:** [DevCreed (محمد الهلالي) — Background Jobs Using Hangfire in .NET (Arabic)](https://www.youtube.com/watch?v=hnK1qn6ZjKo)
  * **VIDEO:** [Milan Jovanović — Background Tasks with Hangfire](https://www.youtube.com/watch?v=m7zC_u6P12o)

#### Task 2 — Transactional Email Service (MailKit & Mailpit)
* **NTF-01:** Implement `IEmailService` in Application, backed by MailKit in Infrastructure.
* **NTF-06:** In development, SMTP connects to local **Mailpit** container (zero external emails sent).
* **NTF-05:** Emails formatted in donor's `PreferredLanguage` (Arabic emails support RTL directionality).
* **Resources:**
  * **VIDEO:** [Patrick God — Send Emails with MailKit in ASP.NET Core](https://www.youtube.com/watch?v=m53kK7Xk9_k)
  * **ARTICLE:** [Code Maze — Send Emails with MailKit](https://code-maze.com/aspnetcore-send-email/)

#### Task 3 — Dynamic PDF Receipt Generation (QuestPDF)
* **FIL-06:** Generate certified PDF receipts containing receipt number, donor name, campaign title, amount, and date.
* **JOB-01:** Receipt generation executes asynchronously in a **Hangfire fire-and-forget job**.
* **FIL-07:** `GET /donations/{id}/receipt` allows donor or Admin to download the generated PDF.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — Generate PDF Documents Easily in C# with QuestPDF](https://www.youtube.com/watch?v=Y3pU-L1Q7u0)
  * **DOCUMENTATION:** [QuestPDF Quick Start](https://www.questpdf.com/quick-start.html)

#### Task 4 — Asynchronous Notification Jobs
* **NTF-02:** Email receipt to donor with PDF receipt attached.
* **NTF-03:** Email campaign manager when campaign reaches its financial target.
* **NTF-04:** All emails sent via Hangfire jobs with automatic retry (minimum 3 attempts).
* **Idempotency Rule:** Background jobs must be idempotent — retrying a job must never generate duplicate emails or duplicate receipts.

#### Task 5 — Stale Donation Timeout (Delayed Job)
* **DON-09 & JOB-03:** When a pending donation is created, schedule a **Hangfire delayed job** (default 30-minute timeout). If the donation remains in `Pending` status after the timeout, mark it `Failed`.
* **Resources:**
  * **MUST:** [Hangfire — Delayed jobs](https://docs.hangfire.io/en/latest/background-methods/calling-methods-with-delay.html)

#### Task 6 — Recurring Campaign Expiration Job
* **JOB-04:** Register a **Hangfire recurring job** running hourly to transition active campaigns past their `EndDate` to `Expired`.
* **Resources:**
  * **MUST:** [Hangfire — Recurring jobs](https://docs.hangfire.io/en/latest/background-methods/performing-recurrent-tasks.html)

#### Task 7 — Secure Campaign Banner Upload
* **FIL-01:** Upload banner image for campaigns (Admin or campaign owner). Allowed types: JPG, PNG, WEBP. Max size 2 MB.
* **FIL-02:** Validate file headers (**magic bytes**), rejecting disguised executable files.
* **FIL-03:** Store files via `IFileStorage` abstraction with unique randomized GUID filenames.
* **FIL-04:** Replacing an existing banner permanently deletes the obsolete file.
* **FIL-05:** Static file middleware serves banners publicly via generated relative URLs.
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | POST | `/campaigns/{id}/banner` | Admin, Campaign Owner (multipart/form-data) |
  | DELETE | `/campaigns/{id}/banner` | Admin, Campaign Owner |
* **Resources:**
  * **VIDEO:** [Milan Jovanović — Secure File Upload in ASP.NET Core](https://www.youtube.com/watch?v=p3mO2uC2bZ8)

#### Task 8 — Enterprise Structured Logging (Serilog)
* Configure Serilog with Console and Rolling File sinks.
* Enrich logs with correlation ID (`traceId`) and authenticated `UserId`.
* Use message templates: `Log.Information("Donation {DonationId} succeeded for {Amount}", id, amount)`.
* **Security Rule:** Never log passwords, tokens, full auth request bodies, or Stripe webhook secrets.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — Structured Logging with Serilog in .NET](https://www.youtube.com/watch?v=nVAkSBpsuTk)
  * **VIDEO:** [Milan Jovanović — Serilog in ASP.NET Core](https://www.youtube.com/watch?v=0ostnGbzMuo)

#### Task 9 — Unit Tests for Asynchronous Workloads
Write unit tests for job idempotency, magic byte validation, and expiration logic.

---

### Sprint 4 Deliverables & Definition of Done
- [ ] Hangfire operational with SQL Server storage and basic auth dashboard.
- [ ] MailKit email service with Mailpit development support.
- [ ] QuestPDF receipt generation in background job.
- [ ] Delayed job for donation timeouts and recurring job for campaign expiration.
- [ ] File upload with magic bytes inspection and random storage naming.
- [ ] Serilog structured logging with correlation IDs.
- [ ] Evaluation Rubric (100 Points): Background Jobs & Hangfire (30) + Email & PDF Services (25) + File Upload & Validation (20) + Structured Logging (15) + Testing (10).

---

## 🟥 Sprint 5 — Payments (Stripe), Idempotency & Integration Testing

* **Timeline:** 15 Dec 2026 → 28 Dec 2026 | **Effort:** ~30 Hours | **Evaluation:** 100 Points
* **Sprint Goal:** Replace the temporary payment stub with production-ready Stripe Checkout Sessions, build a secure webhook receiver with HMAC signature verification, enforce client idempotency keys, and build an automated integration testing suite.

### Detailed Sprint Tasks

#### Task 1 — Stripe Checkout Session Creation
* **PAY-01:** `POST /donations/{id}/checkout` creates a hosted Stripe Checkout Session (mode `payment`) and returns session URL.
* **PAY-02:** Session payload configuration:
  * `client_reference_id` = Donation ID
  * Metadata containing Donation ID
  * Single line item with campaign title, amount in **minor units (cents)**, and configured currency
  * Success and Cancel redirect URLs from `StripeOptions`
* **PAY-03:** Record a new `PaymentAttempt` for each created session.
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | POST | `/donations/{id}/checkout` | Donation Owner |
  | GET | `/donations/{id}/payment-status` | Owner, Admin |
* **Resources:**
  * **MUST:** [Stripe Checkout Sessions](https://docs.stripe.com/api/checkout/sessions) | [Accept a payment](https://docs.stripe.com/payments/accept-a-payment)
  * **VIDEO:** [Stripe Developers — Accept Payments with Stripe Checkout in .NET](https://www.youtube.com/watch?v=mI_y_vC0g9Y)
  * **VIDEO:** [Patrick God — Stripe Payments with ASP.NET Core Web API](https://www.youtube.com/watch?v=F3aO-4U4rK0)
  * **ARTICLE:** [Code Maze — Stripe Payments in ASP.NET Core](https://code-maze.com/stripe-payments-aspnetcore/)

#### Task 2 — Webhook Receiver Endpoint
* **PAY-04:** `POST /payments/webhook` receives Stripe asynchronous events.
  * Publicly accessible, reads the **raw HTTP request body**, and **verifies HMAC signature** using the webhook secret.
  * Invalid signatures return `400 Bad Request`.
* **PAY-08:** Webhook responds with `200 OK` immediately. Heavy background tasks (PDF, emails) are enqueued to Hangfire.
* **Resources:**
  * **MUST:** [Stripe Webhooks](https://docs.stripe.com/webhooks) | [Webhook signature verification](https://docs.stripe.com/webhooks/signature)
  * **VIDEO:** [Stripe Developers — Listen for Webhooks in .NET](https://www.youtube.com/watch?v=o04A_Bsmh8s)
  * **VIDEO:** [Milan Jovanović — Stripe Webhooks in ASP.NET Core](https://www.youtube.com/watch?v=4xR1E7aZ6Ys)

#### Task 3 — Webhook Event Dispatching & Verification
Handle essential Stripe events:
* **PAY-05:**
  * `checkout.session.completed` ➔ Run atomic success flow: update donation status, increment raised amount, check campaign completion, enqueue PDF & email jobs.
  * `checkout.session.expired` ➔ Mark `PaymentAttempt` as `Expired`.
  * `payment_intent.payment_failed` ➔ Mark `PaymentAttempt` and `Donation` as `Failed`.
* **PAY-07:** Verify paid amount and currency match donation record exactly. Mismatches are flagged and rejected.
* **Edge Case:** If payment succeeds for an already completed or expired campaign, accept the funds, log an administrative warning, and process the receipt.

#### Task 4 — Webhook Idempotency (ProcessedStripeEvent)
* **PAY-06:** Persist incoming Stripe `EventId` in `ProcessedStripeEvent`.
* If a duplicate event arrives, acknowledge immediately with `200 OK` without re-executing any business actions.
* **Resources:**
  * **MUST:** [Stripe Webhooks Best Practices](https://docs.stripe.com/webhooks#best-practices)

#### Task 5 — Idempotency-Key Header Middleware
* **DON-10:** Require `Idempotency-Key: <unique-key>` on `POST /donations` and `POST /donations/{id}/checkout`. Missing header returns 400.
* Store key, user ID, request body hash, and cached response for 24 hours.
* Same key + same body ➔ Replay cached response without repeating side effects.
* Same key + different body ➔ Return `422 Unprocessable Entity` (`Idempotency.KeyReusedWithDifferentRequest`).
* **Resources:**
  * **VIDEO:** [Milan Jovanović — Building Idempotent APIs with ASP.NET Core](https://www.youtube.com/watch?v=b4O6u9B2y6A)
  * **VIDEO:** [Derek Comartin (CodeOpinion) — What is Idempotency?](https://www.youtube.com/watch?v=T_T5Z0Y2K68)

#### Task 6 — Elimination of Temporary Payment Stub
* **PAY-09:** Permanently delete `POST /admin/donations/{id}/mark-succeeded`. All donation completions now exclusively flow through verified Stripe webhooks.

#### Task 7 — Stripe Test Mode & Local Forwarding
* Use Stripe test API keys in `user-secrets`.
* Forward local webhooks using Stripe CLI: `stripe listen --forward-to https://localhost:<port>/api/v1/payments/webhook`.
* **Resources:**
  * **MUST:** [Stripe Test Mode](https://docs.stripe.com/test-mode) | [Stripe CLI](https://docs.stripe.com/stripe-cli)

#### Task 8 & 9 — Integration Testing Suite (WebApplicationFactory)
Create `DonationHub.Api.IntegrationTests` covering critical end-to-end flows:
* Registration ➔ Login ➔ Token Refresh flow.
* Campaign creation ➔ Donation checkout initialization.
* Webhook success flow with **locally synthesized HMAC-SHA256 signatures** (no external HTTP calls).
* Duplicate webhook replay verifying idempotency.
* Rejection of invalid webhook signatures.
* Idempotency-Key caching and 422 mismatch validation.
* **Resources:**
  * **MUST:** [ASP.NET Core Integration Tests](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)
  * **VIDEO:** [Nick Chapsas — Master ASP.NET Core Integration Testing](https://www.youtube.com/watch?v=zaRM0iIhJvs)
  * **VIDEO:** [Milan Jovanović — Integration Testing with WebApplicationFactory](https://www.youtube.com/watch?v=tj5ZCtvgLoQ)

---

### Sprint 5 Deliverables & Definition of Done
- [ ] Stripe Checkout session generation with line items and minor units.
- [ ] Webhook receiver with HMAC signature verification.
- [ ] Webhook deduplication via `ProcessedStripeEvent`.
- [ ] `Idempotency-Key` middleware active on write endpoints.
- [ ] Temporary payment stub deleted.
- [ ] Integration tests verifying auth, checkout, webhooks, and idempotency.
- [ ] Evaluation Rubric (100 Points): Stripe Checkout & Webhook Handling (30) + Idempotency & Concurrency (25) + Integration Testing Suite (25) + Security & Secret Management (10) + Code Quality & Defense (10).

---

## 🟪 Sprint 6 — Reports, Admin, Docker, CI/CD & Health Checks

* **Timeline:** 29 Dec 2026 → 11 Jan 2027 | **Effort:** ~30 Hours | **Evaluation:** 100 Points
* **Sprint Goal:** Deliver executive analytics reports, staff administrative controls, comprehensive health checks, a production multi-stage Dockerfile, a multi-container Docker Compose environment, and automated GitHub Actions CI.

### Detailed Sprint Tasks

#### Task 1 — Analytics Dashboard & Reports
Implement high-performance aggregation queries:
* **RPT-01:** Dashboard overview: total raised funds, count of succeeded donations, count of distinct donors, active/completed/expired campaign breakdown.
* **RPT-02:** Top N campaigns ranked by raised funds.
* **RPT-03:** Daily donation trend analytics over a custom date range.
* **RPT-04:** Cache dashboard responses in Redis (default 5-minute sliding expiration).
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | GET | `/admin/dashboard/summary` | Admin |
  | GET | `/admin/dashboard/top-campaigns?count=5` | Admin |
  | GET | `/admin/dashboard/donations-over-time?from=&to=` | Admin |
* **Resources:**
  * **VIDEO:** [Milan Jovanović — High-Performance Aggregations in EF Core](https://www.youtube.com/watch?v=cM3k0ZqP4f0)

#### Task 2 — Admin User & Staff Management
* **ADM-01:** Admin lists all registered users (paged, role filtering, name/email search).
* **ADM-02:** Admin activates or deactivates user accounts (deactivated accounts are blocked from login).
* **ADM-03:** Admin manages staff accounts (`Admin`, `CampaignManager`). Deleting a staff account performs deactivation.
* **Critical Safety Rules:**
  * Admins cannot deactivate or delete their own account.
  * The last remaining active Admin cannot be deactivated or deleted.
* **Endpoints:**
  | Method | Path | Access |
  |---|---|---|
  | GET | `/admin/users` | Admin |
  | PATCH | `/admin/users/{id}/status` | Admin |
  | GET | `/admin/staff` | Admin |
  | PUT | `/admin/staff/{id}` | Admin |
  | DELETE | `/admin/staff/{id}` | Admin |
* **Resources:**
  * **MUST:** [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity)

#### Task 3 — System Health Checks
Implement ASP.NET Core Health Checks:
* `/health/live`: Liveness probe indicating the process is running.
* `/health/ready`: Readiness probe verifying SQL Server connectivity, Redis cache availability, and file storage write permissions.
* Structured JSON responses including component latency and status.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — Health Checks in ASP.NET Core (.NET 8)](https://www.youtube.com/watch?v=kY8c2_J6F9Y)
  * **VIDEO:** [Milan Jovanović — Health Checks Monitoring SQL & Redis](https://www.youtube.com/watch?v=d_Z9Q4u0n7A)

#### Task 4 — Production Multi-Stage Dockerfile
Create an optimized `Dockerfile`:
* Build stage: restore dependencies, build release binary, publish artifacts.
* Runtime stage: minimal `mcr.microsoft.com/dotnet/aspnet:8.0` image.
* **Security Hardening:** Execute application under an unprivileged **non-root user** (`USER app`).
* **Resources:**
  * **VIDEO:** [Nick Chapsas — Dockerizing .NET Applications (Multi-Stage & Non-Root)](https://www.youtube.com/watch?v=bb_t_kY6zNo)
  * **VIDEO:** [Milan Jovanović — Containerizing .NET 8 Applications with Docker](https://www.youtube.com/watch?v=p7_x1M4n2qA)

#### Task 5 — Multi-Container Docker Compose Stack
Create `docker-compose.yml` orchestrating the entire platform:
* `api`: DonationHub Web API.
* `sqlserver`: Microsoft SQL Server 2022.
* `redis`: Redis distributed cache.
* `mailpit`: Local SMTP testing server and web UI.
* Named volumes for SQL data persistence and banner uploads.
* Health check dependencies (`depends_on: condition: service_healthy`).
* Running `docker compose up` starts the entire system with migrations applied and admin seeded automatically.
* **Resources:**
  * **VIDEO:** [Milan Jovanović — Docker Compose for .NET Developers](https://www.youtube.com/watch?v=9_Y_j1c8k8A)
  * **VIDEO:** [Amichai Mantinband — Docker Compose for ASP.NET Core & Dependencies](https://www.youtube.com/watch?v=W7m_8n4_4wU)

#### Task 6 — Environment Configuration & Secret Management
* Extract all sensitive settings into environment variables for container execution.
* Ensure no production keys, passwords, or secrets are committed to source control.

#### Task 7 & 8 — GitHub Actions CI Automation
Create `.github/workflows/ci.yml`:
* Triggers on every push and pull request to `main`.
* Steps: Checkout ➔ Setup .NET ➔ Restore ➔ Build (`--configuration Release`) ➔ Run Unit Tests ➔ Run Integration Tests ➔ Build Docker image.
* **Resources:**
  * **VIDEO:** [Nick Chapsas — GitHub Actions CI/CD for .NET Developers](https://www.youtube.com/watch?v=3g_t_N4K4hY)
  * **VIDEO:** [Milan Jovanović — Automating .NET Builds & Tests with GitHub Actions](https://www.youtube.com/watch?v=k2Z_q6p8v3A)

#### Task 9, 10 & 11 — Verification, Code Cleanup & Final Master Documentation
* Verify containerized production stack locally.
* Clean up dead code, eliminate unused abstractions, verify zero compiler warnings.
* Update comprehensive repository documentation.

---

### Sprint 6 Deliverables & Definition of Done
- [ ] Admin analytics dashboard with high-performance aggregations and Redis caching.
- [ ] Admin user & staff management with self-deactivation protection.
- [ ] `/health/live` and `/health/ready` endpoints returning structured JSON.
- [ ] Multi-stage Dockerfile running as non-root user.
- [ ] `docker-compose.yml` running API, SQL Server, Redis, and Mailpit with named volumes.
- [ ] Passing GitHub Actions CI pipeline on GitHub.
- [ ] Evaluation Rubric (100 Points): Reports & Admin Management (25) + Health Checks & Monitoring (15) + Docker & Docker Compose (25) + CI/CD Automation (20) + Production Hardening (15).

---

## 🏁 Final Period, Defense & Graduation Guidelines

* **Timeline:** 12 Jan 2027 → 20 Jan 2027 (9 Days)
* **Objective:** Comprehensive bug fixing, system hardening, and individual oral defense of architectural decisions.

### The Oral Defense (Final Review)
Each learner presents their live system and defends architectural decisions against technical questions:

1. *"Why did you bypass the generic repository for query operations in CQRS?"*  
   **Model Answer:** Read queries have no entity lifecycle or change-tracking needs. Bypassing repositories and querying `DbContext` directly with `.AsNoTracking()` and `.Select()` projection generates optimal SQL, avoids allocating entity objects, and dramatically improves read throughput.
2. *"Why do we verify Stripe webhooks via raw request body signatures instead of redirecting the user to a success page?"*  
   **Model Answer:** Client redirects can be intercepted, spoofed, or lost if the user closes their browser before returning. Stripe Webhooks are server-to-server cryptographic notifications signed by Stripe's private secret, ensuring undeniable proof of payment regardless of client state.
3. *"Why did you select Hangfire over ASP.NET Core BackgroundService for PDF generation and emails?"*  
   **Model Answer:** `BackgroundService` runs in-memory. If the application restarts, crashes, or scales out, in-memory queues are lost and lack retry logic. Hangfire persists jobs in SQL Server, providing guaranteed execution, exponential backoff retries, and distributed worker capabilities.
4. *"How does your system prevent double donation records when a network timeout occurs?"*  
   **Model Answer:** Through the `Idempotency-Key` header middleware. The client passes a unique UUID. The server caches the result of the first execution for 24 hours. If a retry with the exact same payload arrives, the cached response is replayed without executing the payment or donation logic a second time.

### Graduation Criteria:
* Total Score: **480+ / 600 Points (80%+)** across the 6 sprints.
* Zero critical security vulnerabilities (no SQL injection, no unhashed tokens, no secrets in Git).
* Successful containerized deployment via Docker Compose.
* Confident defense of architectural design decisions.

---

## 📏 Universal Architectural Rules & Definition of Done

The following rules apply universally to every commit and pull request:

### Universal Definition of Done (DoD)
- [ ] The solution compiles with **zero errors** and **zero warnings**.
- [ ] Clean Architecture layer boundaries are strictly maintained (Domain has zero references).
- [ ] No domain entities are exposed across API controllers (DTOs only).
- [ ] Controllers contain zero business logic (thin MediatR dispatchers).
- [ ] No `DateTime.UtcNow` calls in domain or handlers (injected `TimeProvider` used).
- [ ] All async methods accept and propagate a `CancellationToken`.
- [ ] All inputs validated via FluentValidation; expected business failures return `Result`.
- [ ] All errors conform to RFC 9457 `ProblemDetails` with correlation IDs (`traceId`).
- [ ] No secrets, keys, or connection strings exist in source code or Git history.
- [ ] All automated unit and integration tests pass cleanly with `dotnet test`.
- [ ] System boots cleanly from cold start via `docker compose up`.

---

## 🚫 Explicitly Out of Scope

To prevent scope creep and maintain absolute focus on advanced backend engineering mastery, the following are **strictly prohibited**:

* **Front-End Development:** No React, Angular, Vue, Blazor, or MVC views. Pure REST API.
* **External Identity Providers:** No OpenIddict, Duende IdentityServer, or Keycloak (ASP.NET Core Identity + JWT is used).
* **Distributed Microservices:** No service meshes, gRPC inter-service topologies, or multiple deployable backend services.
* **Message Brokers:** No Kafka, RabbitMQ, or MassTransit (Hangfire handles all asynchronous background workloads).
* **AI / LLM / Vector Databases:** No OpenAI APIs, LangChain, or vector search integrations.

---

## 📂 Quick Navigation & Documentation Links

| Document / Specification | Link | Description |
|---|---|---|
| **Software Requirements Specification** | [📄 SRS-DonationHub.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/SRS-DonationHub.md) | Official functional & non-functional requirements |
| **Architectural Decisions & Trade-offs** | [📄 TEAM-NOTES-DonationHub.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/TEAM-NOTES-DonationHub.md) | Architectural agreements and design rationales |
| **Master General Roadmap** | [🧭 Round 2 RoadMap.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/General%20RoadMap/Round%202%20RoadMap.md) | Detailed technical blueprint and sprint guides |
| **Mentorship Timeline** | [⏱️ Round2 Timeline.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/General%20RoadMap/Round2%20Timeline.md) | Official schedule, sync dates, and milestones |
| **Sprint 1 Specification** | [🟦 Sprint-1.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/Sprint-1.md) | Architecture, EF Core, Identity & Categories |
| **Sprint 2 Specification** | [🟩 Sprint-2.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/Sprint-2.md) | Campaigns, CQRS, MediatR & Unit Testing |
| **Sprint 3 Specification** | [🟨 Sprint-3.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/Sprint-3.md) | Donations, Authorization, Caching & API Standards |
| **Sprint 4 Specification** | [🟧 Sprint-4.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/Sprint-4.md) | Notifications, QuestPDF, Hangfire & Serilog |
| **Sprint 5 Specification** | [🟥 Sprint-5.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/Sprint-5.md) | Stripe Payments, Idempotency & Integration Tests |
| **Sprint 6 Specification** | [🟪 Sprint-6.md](file:///c:/Users/Fares/Desktop/.NET-Level-2-RoadMap/Round-2/Sprint-6.md) | Analytics, Admin, Docker, Compose & CI/CD |

---

*Tawjeh .NET Mentorship Program — Level 2 Backend Engineering Roadmap.*
