# 🚀 Tawjeh .NET Level 2 — Advanced Backend Master Roadmap
### Comprehensive General Plan, Architecture & Unabridged Sprint Specifications

> **Program:** Tawjeh Mentorship Program — Level 2  
> **Core Project:** **DonationHub** (Donation Management & Charity Crowdfunding Platform)  
> **Program Duration:** 12 Weeks (6 Sprints × 15 Days) + Final Defense Period (23 Oct 2026 → 20 Jan 2027)  
> **Total Effort:** ~180 Hours (~15 Hours / Week)  
> **Evaluation:** 600 Points Total (100 Points / Sprint)  
> **Target Framework:** .NET 10 (`net10.0` with `.slnx` solution format, compatible with .NET 8 / .NET 9)  
> **Target Audience:** Junior to Intermediate .NET Developers transitioning to Production-Ready Backend Engineers  

---

## 📖 Table of Contents

1. [🎯 Level 2 Mission & Educational Philosophy](#-level-2-mission--educational-philosophy)
2. [🏗️ Project Spotlight: DonationHub](#️-project-spotlight-donationhub)
3. [🏛️ Architectural Blueprint & Layering Rules](#️-architectural-blueprint--layering-rules)
4. [👥 Roles & Permissions Matrix](#-roles--permissions-matrix)
5. [📋 Engineering Standards, Decisions & Pinned Dependencies](#-engineering-standards-decisions--pinned-dependencies)
6. [🧭 Official Mentorship Schedule & Live Session Cadence](#-official-mentorship-schedule--live-session-cadence)
7. [🟦 Sprint 1 — Clean Architecture, Domain Modeling, Identity & Categories](#-sprint-1--clean-architecture-domain-modeling-identity--categories)
8. [🟩 Sprint 2 — Campaigns, CQRS, MediatR & Unit Testing](#-sprint-2--campaigns-cqrs-mediatr--unit-testing)
9. [🟨 Sprint 3 — Donations Core, Resource Authorization & Production API Engineering](#-sprint-3--donations-core-resource-authorization--production-api-engineering)
10. [🟧 Sprint 4 — Hangfire, MailKit, QuestPDF, File Security & Structured Logging](#-sprint-4--hangfire-mailkit-questpdf-file-security--structured-logging)
11. [🟥 Sprint 5 — Stripe Checkout, Webhooks, Idempotency & Integration Testing](#-sprint-5--stripe-checkout-webhooks-idempotency--integration-testing)
12. [🟪 Sprint 6 — EF Core Aggregations, Docker, Health Checks, CI/CD & Final Delivery](#-sprint-6--ef-core-aggregations-docker-health-checks-cicd--final-delivery)
13. [🏁 Final Period, Project Defense & Graduation Guidelines](#-final-period-project-defense--graduation-guidelines)
14. [📏 Universal Architectural Rules & Definition of Done](#-universal-architectural-rules--definition-of-done)
15. [📖 Post-Course Reading & Continuous Learning](#-post-course-reading--continuous-learning)

---

## 🎯 Level 2 Mission & Educational Philosophy

Most backend developers get stuck in **"Tutorial Hell"** — they know how to generate an API controller, bind a model with Entity Framework Core, and write basic CRUD operations. However, when faced with real-world enterprise requirements, critical gaps appear:

- *How do we guarantee financial accuracy when multiple donors donate to the same campaign simultaneously?*
- *How do we handle network timeouts during external payment gateway calls without double-charging the donor?*
- *Why is sending emails and generating PDFs inside an HTTP request lifecycle a major production anti-pattern?*
- *How do we organize use cases so modifying one feature never inadvertently breaks another?*
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
Learners will **not** build disconnected toy apps. You will build, evolve, refactor, and defend a single production platform — **DonationHub** — watching it transform from an empty solution into an enterprise-grade backend across six sprints:

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

**DonationHub** is a production-grade charity crowdfunding platform designed to simulate real-world financial, concurrency, and security challenges.

### Core Modules:
1. **Identity & Staff Management:** Authentication, JWT, refresh token rotation, and role-based permissions (`Donor`, `CampaignManager`, `Admin`).
2. **Categories Module:** Bilingual taxonomies with campaign association constraints.
3. **Campaigns Module:** Crowdfunding campaigns with financial goals, deadlines, and optimistic concurrency protection.
4. **Donations & Payments:** Seamless donations via Stripe Checkout, webhook verification, and atomic database settlements.
5. **Background Processing & Receipts:** Asynchronous email receipts and PDF generation via Hangfire and QuestPDF.
6. **Analytics & Administration:** High-performance EF Core aggregations, caching, and audit logging.

---

## 🏛️ Architectural Blueprint & Layering Rules

DonationHub enforces strict **Clean Architecture** with dependency inversion. The layers are structured as follows:

```text
┌─────────────────────────────────────────────────────────────┐
│                      DonationHub.Api                        │
│   • Controllers (Thin)   • Middlewares   • Composition Root │
└──────────────┬───────────────────────────────┬──────────────┘
               │                               │
               ▼                               ▼
┌──────────────────────────────┐ ┌─────────────────────────────┐
│  DonationHub.Infrastructure  │ │   DonationHub.Application   │
│   • EF Core DbContext        │ │   • Vertical Feature Slices │
│   • Repositories & UoW       │ │   • Commands, Queries, DTOs │
│   • Hangfire & MailKit       │ │   • FluentValidation Rules  │
│   • Stripe & QuestPDF        │ │   • Pipeline Behaviors      │
└──────────────┬───────────────┘ └─────────────┬───────────────┘
               │                               │
               └───────────────┬───────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                     DonationHub.Domain                      │
│   • Entities & Value Objects • Invariants • Enums           │
│               (Zero External Dependencies)                  │
└─────────────────────────────────────────────────────────────┘
```

### Layering Rules:
- **`DonationHub.Domain`:** Pure business logic. Zero dependencies on external packages, EF Core, or presentation models.
- **`DonationHub.Application`:** Organizes use cases into **Vertical Feature Slices**. Contains MediatR handlers, DTOs, interfaces, and FluentValidation validators.
- **`DonationHub.Infrastructure`:** Implements interfaces defined by Application/Domain. Contains EF Core, Redis, Hangfire, MailKit, Stripe, and QuestPDF.
- **`DonationHub.Api`:** The presentation layer and composition root. Thin controllers delegate directly to MediatR.

---

## 👥 Roles & Permissions Matrix

| Module / Action | Anonymous | Donor | Campaign Manager | Admin |
|---|:---:|:---:|:---:|:---:|
| **Register & Login** | ✅ | ✅ | ✅ | ✅ |
| **View Categories** | ✅ | ✅ | ✅ | ✅ |
| **Manage Categories (CRUD)** | ❌ | ❌ | ❌ | ✅ |
| **Browse Active Campaigns** | ✅ | ✅ | ✅ | ✅ |
| **Create Campaign** | ❌ | ❌ | ✅ (Own) | ✅ |
| **Update / Delete Campaign** | ❌ | ❌ | ✅ (Own Only) | ✅ (All) |
| **Donate to Campaign** | ✅ (Guest) | ✅ | ✅ | ✅ |
| **Stripe Checkout & Webhooks** | ✅ | ✅ | ✅ | ✅ |
| **Receive Email Receipt** | ✅ | ✅ | ✅ | ✅ |
| **View Analytics Dashboard** | ❌ | ❌ | ❌ | ✅ |
| **Manage Staff Accounts** | ❌ | ❌ | ❌ | ✅ |

---

## 📋 Engineering Standards, Decisions & Pinned Dependencies

To prevent commercial licensing traps and breaking changes, all teams must adhere to the following decisions:

| Area / Library | Pinned Version / Decision | Rationale |
|---|---|---|
| **.NET SDK** | .NET 10 (`net10.0`) | Modern `.slnx` solution format, `Program.cs` root. Built-in OpenAPI + `Scalar.AspNetCore` replaces Swashbuckle. Fully compatible with .NET 8 / 9. |
| **MediatR** | `[12.5.0]` | Pinned to prevent automatic upgrade to commercial v13+. |
| **FluentAssertions** | `[7.0.0]` | Free Apache 2.0 license. v8+ is commercial. (Alternative: Shouldly). |
| **Moq** | `[4.18.4]` | Avoids SponsorLink controversy. (Alternative: NSubstitute). |
| **QuestPDF** | Community License | Set `QuestPDF.Settings.License = LicenseType.Community;` (Free for education, open source, and small businesses). |
| **Hangfire** | Free Core (LGPL v3) | Use `Hangfire.Core`, `Hangfire.AspNetCore`, and `Hangfire.SqlServer`. Avoid paid commercial Pro extensions. |
| **Redis** | Docker `redis:alpine` | Run via `docker run -d --name redis -p 6379:6379 redis:alpine` (or `valkey/valkey:alpine`). |
| **Mailpit** | Docker `axllent/mailpit` | Local SMTP testing container (`docker run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit`). |
| **API Errors** | RFC 9457 `ProblemDetails` | Enforced via `builder.Services.AddProblemDetails();` and `IExceptionHandler`. |
| **Time Abstraction** | `TimeProvider` | Disallow direct `DateTime.UtcNow` in domain and handlers to enable deterministic testing. |

---

## 🧭 Official Mentorship Schedule & Live Session Cadence

> **Program Timeline:** **23 October 2026 → 20 January 2027**  
> **Structure:** 6 Sprints × 15 Days each. 3 live sessions per sprint (Day 5, Day 10, Day 15). Review window on Days 16–18.

### 1. Master Calendar

| Sprint | Day 1 (Task Release) | Day 4 (Reminder) | Day 5 — Stand-up 1 | Day 10 — Stand-up 2 | Day 15 — Closing & Deadline | Review Window (Days 16–18) |
|---|---|---|---|---|---|---|
| **S1** | 23/10/2026 (Fri) | 26/10/2026 (Mon) | 27/10/2026 (Tue) | 01/11/2026 (Sun) | 06/11/2026 (Fri) | 07/11/2026 → 09/11/2026 |
| **S2** | 07/11/2026 (Sat) | 10/11/2026 (Tue) | 11/11/2026 (Wed) | 16/11/2026 (Mon) | 21/11/2026 (Sat) | 22/11/2026 → 24/11/2026 |
| **S3** | 22/11/2026 (Sun) | 25/11/2026 (Wed) | 26/11/2026 (Thu) | 01/12/2026 (Tue) | 06/12/2026 (Sun) | 07/12/2026 → 09/12/2026 |
| **S4** | 07/12/2026 (Mon) | 10/12/2026 (Thu) | 11/12/2026 (Fri) | 16/12/2026 (Wed) | 21/12/2026 (Mon) | 22/12/2026 → 24/12/2026 |
| **S5** | 22/12/2026 (Tue) | 25/12/2026 (Fri) | 26/12/2026 (Sat) | 31/12/2026 (Thu) | 05/01/2027 (Tue) | 06/01/2027 → 08/01/2027 |
| **S6** | 06/01/2027 (Wed) | 09/01/2027 (Sat) | 10/01/2027 (Sun) | 15/01/2027 (Fri) | 20/01/2027 (Wed) | 21/01/2027 → 23/01/2027 |

### 2. Live Session Mentor Rotation
Mentors lead one session per sprint and rotate leading the final deadline closing:

| Sprint | Stand-up 1 (Day 5) | Stand-up 2 (Day 10) | Closing & Deadline (Day 15) |
|---|---|---|---|
| **S1** | Mohamed Atef | Asma ELhadidy | Kareem Zein |
| **S2** | Asma ELhadidy | Kareem Zein | Mohamed Atef |
| **S3** | Kareem Zein | Mohamed Atef | Asma ELhadidy |
| **S4** | Mohamed Atef | Asma ELhadidy | Kareem Zein |
| **S5** | Asma ELhadidy | Kareem Zein | Mohamed Atef |
| **S6** | Kareem Zein | Mohamed Atef | Asma ELhadidy |

### 3. Student Group Distribution
Students are split into 3 groups (**G1, G2, G3 ≈ 33% each**). Mentors swap review groups between S1–S3 and S4–S6 to ensure impartial reviews. **Fares** coordinates, tracks progress, and manages deadlines.

---

## 🟦 Sprint 1 — Clean Architecture, Domain Modeling, Identity & Categories

> **Period:** 23/10/2026 → 06/11/2026 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Git & GitHub PR Workflow:** [freeCodeCamp (1:08:29)](https://youtu.be/RGOj5yH7evk) | [Ahmed Sami [Arabic] (PR at 05:56:20)](https://youtu.be/Q6G-J54vgKc)
2. **Async/Await & ThreadPool:** [#038 Asynchronous Programming — Essam Abdelnabi [Arabic]](https://youtu.be/kDUDX3VJFEc) | [Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
3. **SOLID & DI Lifetimes:** [DIP — Mohamed El-Mehdi [Arabic]](https://youtu.be/xuoT_p4WJKo) | [DI Lifetimes — Julio Casal (at 05:35)](https://youtu.be/KAHxOLmvGZo)
4. **Extension Methods & LINQ IQueryable:** [Extension Methods — El-Mehdi](https://youtu.be/g5lx0l7ikaQ) | [IQueryable vs IEnumerable — Gavin Lon (at 03:02:35)](https://youtu.be/5l2qA3Pc83M)

### 🛠️ Tasks:
- **Task 1 (Solution Setup):** Create 4 projects (`Domain`, `Application`, `Infrastructure`, `Api`) with `.slnx`. Setup vertical slices.
  - ⭐ [Clean Architecture 10,000ft View — Essam Abdelnabi [Arabic]](https://youtu.be/DRSZ6V-eKUY) | 🔄 [Milan Jovanović](https://youtu.be/TQdLgzVk2T8)
- **Task 2 (Domain Modeling):** Model Entities (`ApplicationUser`, `RefreshToken`, `Category`, `Campaign`, `Donation`, `PaymentAttempt`, `IdempotencyRecord`) and Value Objects.
  - ⭐ [Tactical DDD — Mohamed El-Mehdi [Arabic]](https://youtu.be/bHEDV62ATNM) | 🔄 [Value Objects — Milan Jovanović](https://youtu.be/P5CRea21R2E)
- **Task 3 (EF Core Persistence):** `AppDbContext`, Fluent API configs, relationships, and initial migration.
  - ⭐ [#010 EF Migration in Arabic — Essam Abdelnabi](https://youtu.be/WqiGu_MQ-xw) | 🔄 [Migrations — Nick Chapsas](https://youtu.be/StBf3-M5WdM)
- **Task 4 (Generic Repo & UoW):** `IGenericRepository<T>` and `IUnitOfWork`. Direct reads bypass repo with `AsNoTracking`.
  - ⭐1 [Generic Repository — Mohamed El-Helaly [Arabic]](https://youtu.be/7Lc9NRaPMr0) | ⭐2 [UnitOfWork — El-Helaly](https://youtu.be/-72HfsPv9Xs)
- **Task 5 (Soft Delete):** `ISoftDeletable` + `SoftDeleteInterceptor` + `HasQueryFilter`.
  - ⭐ [#019 Interceptors — Essam Abdelnabi](https://youtu.be/hEMCDxqURr8) | 🔄 [Soft Delete — Milan Jovanović](https://youtu.be/zwBaWiu-rF4)
- **Task 6 (Options Pattern):** `JwtOptions`, `DonationOptions`, and `ValidateOnStart()`.
  - ⭐ [#11 Options Pattern — CodingClass [Arabic]](https://youtu.be/S3ygXq94dLo) | 🔄 [Milan Jovanović](https://youtu.be/wxYt0motww0)
- **Task 7 (Identity & JWT):** Register, Login, hashed Refresh Tokens with rotation, and Admin seed.
  - ⭐1 [Identity Tables — El-Helaly](https://youtu.be/zshypyVIVFU) (or [.NET 10 Identity — Remigiusz Zalewski](https://youtu.be/g9YSB4bXhes)) | ⭐2 [JWT Endpoints — El-Helaly](https://youtu.be/B7hQY1YckR0) | ⭐3 [Refresh Tokens — El-Helaly](https://youtu.be/5g0aq4g17yg) | 🔄 [Full Course .NET 9 — Patrick God](https://youtu.be/6EEltKS8AwA)
- **Task 8 (Categories Module):** Bilingual Categories CRUD (Admin manage, public view, restrict delete on linked campaigns).
- **Task 9 (API Docs):** Scalar OpenAPI with JWT Bearer scheme on .NET 10 (or Swashbuckle on .NET 8).
  - ⭐ [No Swagger? Replace with SCALAR! — Israel Quiroz](https://youtu.be/J8i1RWowjwc) | 🔄 [Nick Chapsas](https://youtu.be/8yI4gD1HruY)
- **Task 10 (Project Documentation):** Initial setup guide in `README.md`.

---

## 🟩 Sprint 2 — Campaigns, CQRS, MediatR & Unit Testing

> **Period:** 07/11/2026 → 21/11/2026 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **CQRS & Mediator Mindset:** Separation of write commands and read queries.
2. **Cooperative Cancellation:** CancellationToken propagation to avoid orphaned database tasks.

### 🛠️ Tasks:
- **Task 1 (CQRS with MediatR [12.5.0]):** Refactor all use cases into commands/queries and thin controllers.
  - ⭐ [23 Add mediator pattern with CQRS — Ahmed Mohamady [Arabic]](https://youtu.be/W7_zEf3eRQA) | 🔄 [Patrick God](https://youtu.be/BewUyKLZjtc)
- **Task 2 (CancellationToken & TimeProvider):** Forward cancellation token to EF Core and inject `TimeProvider`.
  - ⭐1 [Cancellation Token — Rahul Nath](https://youtu.be/39gIPekzpjs) | ⭐2 [TimeProvider in .NET 8+ — Nick Chapsas](https://youtu.be/xa6YusjCFF0)
- **Task 3 (Optimistic Concurrency):** `byte[] RowVersion` on `Campaign` to prevent lost updates (409 Conflict).
  - ⭐ [Optimistic Concurrency in .NET — Islam Helmy [Arabic]](https://youtu.be/VhuFcGP6H0E) | 🔄 [Milan Jovanović](https://youtu.be/lLDBVMj5He8)
- **Task 4 (Result Pattern):** Standardize `Result<T>` and `Error` catalogue, eliminating business exceptions.
  - ⭐ [Error & Result Pattern — Loai Saber [Arabic]](https://youtu.be/hfnWW2trOlY) | 🔄 [Milan Jovanović](https://youtu.be/WCCkEe_Hy2Y)
- **Task 5 (Validation Behavior):** MediatR `ValidationBehavior` executing FluentValidation automatically.
  - ⭐ [Clean Arch & CQRS in Action — Essam Abdelnabi](https://youtu.be/v9fRFyg414k) | 🔄 [Milan Jovanović](https://youtu.be/85dxwd8HzEk)
- **Task 6 (Global Exception Handling):** `IExceptionHandler` + RFC 9457 `ProblemDetails`.
  - ⭐ [#9 Exception Handling — CodingClass [Arabic]](https://youtu.be/Efmvo90NuiA) | 🔄 [Milan Jovanović](https://youtu.be/uOEDM0c9BNI)
- **Task 7 (Campaigns Module):** CMP-01 to CMP-06, CMP-10, CMP-11 (Create, update, soft delete, details with `progressPercent` and `daysLeft`).
- **Task 8 (Unit Testing):** At least 15 passing unit tests using xUnit, Moq `[4.18.4]`, and FluentAssertions `[7.0.0]`.
  - ⭐1 [Unit Testing — Essam Abdelnabi](https://youtu.be/cAyROJ84tBs) | ⭐2 [Moq Mocking — Essam](https://youtu.be/JaIeeGqsO9M) | ⭐3 [Fluent Assertions — Coding-Future](https://youtu.be/rr9z92lm2no)

---

## 🟨 Sprint 3 — Donations Core, Resource Authorization & Production API Engineering

> **Period:** 22/11/2026 → 06/12/2026 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Docker Container Basics:** `docker run -d -p`, `docker ps`, `docker logs`, `docker stop`. Spin up Redis locally.
  - ⭐1 [Docker Basics Part 5 — Codographia](https://youtu.be/cB7Qx13YOFM) | ⭐2 [Part 6 — Codographia](https://youtu.be/w_fnmXy9ZEk) | 🔄 [Yehia Tech [Arabic]](https://youtu.be/yRkE-1Lq0qo)

### 🛠️ Tasks:
- **Task 1 (Resource-Based Authorization):** CampaignManager manages own campaigns only; 404 returned for unowned private resources.
  - ⭐1 [Microsoft Learn — Resource Authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased) | ⭐2 [Who Owns This Order? — Code Story](https://youtu.be/29x5WmcUNtM)
- **Task 2 (Donations Core & Transactions):** DON-01 to DON-06. Atomic EF Core transactions for donation settlement.
  - ⭐ [#020 DB Transaction EF Core — Essam Abdelnabi [Arabic]](https://youtu.be/m2RDSxGPuhs) | 🔄 [Milan Jovanović](https://youtu.be/25H84BXcr9M)
- **Task 3 (Pagination, Filtering, Sorting):** `PagedList<T>` with database-level `Skip` and `Take` via `IQueryable`.
  - ⭐1 [Pagination in Web API — Coding-Future](https://youtu.be/XDkUliEyPms) | ⭐2 [Filtering & Sorting — Milan Jovanović](https://youtu.be/X8zRvXbirMU)
- **Task 4 (Distributed Caching with Redis):** `IDistributedCache` for active campaigns, with immediate invalidation on updates.
  - ⭐ [Caching: In-Memory vs Redis — Amr Mohamed [Arabic]](https://youtu.be/AIBOcH_dAX8) | 🔄 [Milan Jovanović](https://youtu.be/Tt5zIKVMMbs)
- **Task 5 (Localization ar-EG / en-US):** `IStringLocalizer` and `.resx` handling client `Accept-Language` headers.
  - ⭐1 [Localization Part 1 — El-Helaly](https://youtu.be/PyCGEO4b6F0) | ⭐2 [Part 2 — El-Helaly](https://youtu.be/YzOarEJDdrY)
- **Task 6 (Versioning & Rate Limiting):** `Asp.Versioning.Mvc` and built-in Rate Limiting returning 429 Too Many Requests.
  - ⭐1 [API Versioning — Mohamad Lawand](https://youtu.be/DkZd-5z0CeI) | ⭐2 [Rate Limiting — Milan Jovanović](https://youtu.be/1tPVVDEDGtE)
- **Task 7 (CORS Policies):** Configured allowed origins, headers, and methods.
  - ⭐ [Enable CORS — Mohamed El-Helaly [Arabic]](https://youtu.be/_YXcxm-O06I)

---

## 🟧 Sprint 4 — Hangfire, MailKit, QuestPDF, File Security & Structured Logging

> **Period:** 07/12/2026 → 21/12/2026 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Local SMTP Container via Mailpit:** `docker run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit`.

### 🛠️ Tasks:
- **Task 1 (Hangfire Infrastructure):** SQL Server job storage, background worker setup, and protected `/hangfire` dashboard.
  - ⭐1-4 [Hangfire Series — Mohamed El-Helaly [Arabic]](https://youtu.be/hnK1qn6ZjKo) | ⭐5 [Dashboard Security Docs](https://docs.hangfire.io/en/latest/configuration/using-dashboard.html) | 🔄 [Coding Droplets](https://youtu.be/lsP5ZjcFtMM)
- **Task 2 (MailKit Email Service):** Transactional email sending directed to local Mailpit container.
  - ⭐1 [Sending Emails in .NET — El-Helaly](https://youtu.be/J23_c3uSyD0) | ⭐2 [Mailpit Docker Docs](https://mailpit.axllent.org/docs/install/docker/) | 🔄 [Milan Jovanović](https://youtu.be/KtCjH-1iCIk)
- **Task 3 (QuestPDF Receipt Generation):** Dynamic PDF receipt layout with Community License.
  - ⭐ [QuestPDF Generator — Claudio Bernasconi](https://youtu.be/T89A_7dz1P8) | 🔄 [M Alvin Maulana](https://youtu.be/eiry9Zli5Rc)
- **Task 4 (File Upload Security & Magic Bytes):** Campaign cover uploads with raw binary file signature validation.
  - ⭐1-2 [Upload Files With Fake Extensions — El-Helaly](https://youtu.be/mh0ZTXfSdBI) | ⭐3 [Microsoft Learn — Magic Bytes](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads)
- **Task 5 (Background Workflows):** Fire-and-forget job for receipt/email upon donation, and recurring daily job for expired campaigns.
- **Task 6 (Structured Logging with Serilog & Seq):** Enrich logs with `TraceId`, `Environment`, and request logging.
  - ⭐ [.NET Logging Using Seq — Bishoy Youssef [Arabic]](https://youtu.be/BwcsVcLgFLw) | 🔄 [Milan Jovanović](https://youtu.be/nVAkSBpsuTk)

---

## 🟥 Sprint 5 — Stripe Checkout, Webhooks, Idempotency & Integration Testing

> **Period:** 22/12/2026 → 05/01/2027 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Docker Daemon Active:** Mandatory for Testcontainers spinning up disposable database containers.
2. **Stripe CLI Installed:** For local webhook forwarding (`stripe listen --forward-to ...`).

### 🛠️ Tasks:
- **Task 1 (Stripe Checkout Session):** PAY-01 to PAY-03. Create session with `client_reference_id`, metadata, and minor units (cents).
  - ⭐ [Stripe API Integration in .NET — Israel Quiroz](https://youtu.be/2sZXrUFvZ3E) | 🔄 [Stripe Official Docs](https://docs.stripe.com/api/checkout/sessions)
- **Task 2 (Webhook HMAC Verification):** PAY-04. Validate `Stripe-Signature` via `EventUtility.ConstructEvent`.
  - ⭐ [Creating Stripe Webhook Endpoint — Mohamed Awadallah [Arabic]](https://youtu.be/WZu0lHvNYjI) | 🔄 [Stripe Docs](https://docs.stripe.com/webhooks)
- **Task 3 (Stripe CLI Testing):** Test local webhooks with `stripe listen` and signing secret verification.
- **Task 4 (Idempotency Safeguards):** Handle `Idempotency-Key` headers and deduplicate duplicate webhook event IDs in database.
  - ⭐ [How To Make Your API Idempotent — Milan Jovanović](https://youtu.be/smXAgcdJzLc)
- **Task 5 (Integration Testing with Testcontainers):** End-to-end user workflows against real SQL Server in Docker via `WebApplicationFactory`.
  - ⭐ [Testcontainers in .NET — Milan Jovanović](https://youtu.be/ssRE0pBNvpE) | 🔄 [Julio Casal](https://youtu.be/RXSPCIrrjHc)

---

## 🟪 Sprint 6 — EF Core Aggregations, Docker, Health Checks, CI/CD & Final Delivery

> **Period:** 06/01/2027 → 20/01/2027 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Docker Compose & Multi-Container Networking:** Service discovery and environment variables.

### 🛠️ Tasks:
- **Task 1 (Admin Analytics & Aggregations):** High-performance reporting (`Sum`, `Count`, `GroupBy`) with `AsNoTracking` and Redis caching.
  - ⭐ [EF Core Query Data Part 2 — Essam Abdelnabi [Arabic]](https://youtu.be/fb8j6CjyWWw) | 🔄 [Microsoft Learn](https://learn.microsoft.com/en-us/ef/core/querying/complex-query-operators)
- **Task 2 (Admin Overrides):** Administrative status controls and campaign cancellations.
- **Task 3 (Multi-Stage Dockerfile):** Lean production runtime image running under non-root user.
  - ⭐ [Dockerize ASP.NET Core — Codographia](https://youtu.be/nINTOS2fhVY) | 🔄 [Julio Casal](https://youtu.be/cWMztQwIQNs)
- **Task 4 (Docker Compose Orchestration):** Spin up API, SQL Server, Redis, and Mailpit together.
  - ⭐ [Complete .NET 10 Dev Environment with Docker Compose — Milan Jovanović](https://youtu.be/svfxvsGfLlU) | 🔄 [Mosh](https://youtu.be/HG6yIjZapSA)
- **Task 5 (Health Checks):** Expose `/health`, `/health/ready`, and `/health/live` validating SQL and Redis connectivity.
  - ⭐ [How to Add Health Checks — Milan Jovanović](https://youtu.be/4abSfjdzqms) | 🔄 [Nick Chapsas](https://youtu.be/p2faw9DCSsY)
- **Task 6 (CI/CD via GitHub Actions):** Automated build, unit tests, and integration tests pipeline on every PR.
  - ⭐ [CI/CD Pipeline with GitHub Actions — Coding-Future](https://youtu.be/Zv-NJBRmoVU) | 🔄 [TechWorld with Nana](https://youtu.be/R8_veQiYBjI)
- **Task 7 (Final Documentation):** Production architecture and setup guide in `README.md`.
  - ⭐ [Professional README.md — Fekra w Code [Arabic]](https://youtu.be/ATVJiw4JrUA)

---

## 🏁 Final Period, Project Defense & Graduation Guidelines

> **Final Defense Window:** **21–23 January 2027**

### Evaluation Criteria:
1. **Architecture & Clean Separation (20%):** Adherence to Clean Architecture dependency rules and vertical slices.
2. **Domain Encapsulation & Financial Invariants (20%):** Robust entity protection, concurrency handling, and ACID transaction safety.
3. **Resilience & Background Jobs (20%):** Asynchronous processing with Hangfire, idempotency, and error handling.
4. **Test Quality & Coverage (20%):** Meaningful unit tests (xUnit, Moq, FluentAssertions) and integration tests (Testcontainers).
5. **DevOps & Production Readiness (20%):** Docker Compose orchestration, Health Checks, CI pipeline, and live verbal defense.

### Defense Golden Rule:
During defense, the learner must not only explain **what** was built, but **why** it was designed that way, what alternatives existed, and what architectural trade-offs were accepted.

---

## 📏 Universal Architectural Rules & Definition of Done

- **Thin Controllers:** Controllers only receive HTTP requests, dispatch through MediatR, and return results. Zero business logic in presentation.
- **DTOs Only:** Never leak EF Core entities through API endpoints.
- **No Secrets in Source Control:** Passwords, API secrets, and connection strings must live in environment variables or user secrets.
- **Fail Fast:** Validate configurations at startup using `.ValidateOnStart()`.
- **Zero Business Exceptions:** Expected domain and validation failures must return `Result<T>` and RFC 9457 `ProblemDetails`.

---

## 📖 Post-Course Reading & Continuous Learning

- **Real-Time Web Apps:** [SignalR Full Course — tutorialsEU](https://youtu.be/LEImpa77irI) | [Mohamad Lawand](https://youtu.be/pvi_ZS_PrSc)
- **Cloud-Native .NET Aspire & Microservices:** [.NET Aspire — Julio Casal](https://youtu.be/pk6FJfHhfq8) | [Microservices — Les Jackson](https://youtu.be/DgVjEo3OGBI)
- **Essential Books:**
  - *Clean Architecture* — Robert C. Martin
  - *Domain-Driven Design* — Eric Evans
  - *Dependency Injection Principles, Practices, and Patterns* — Steven van Deursen & Mark Seemann
  - *Designing Data-Intensive Applications* — Martin Kleppmann
