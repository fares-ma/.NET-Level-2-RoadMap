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
1. **Git & GitHub PR Workflow:**
   - ⭐ [Git and GitHub for Beginners — freeCodeCamp (1:08:29)](https://youtu.be/RGOj5yH7evk)
   - 🔄 [Git and GitHub | Ahmed Sami [Arabic] (PR from 05:56:20)](https://youtu.be/Q6G-J54vgKc)
2. **Async/Await & ThreadPool:**
   - ⭐ Asynchronous programming & ThreadPool mechanics.
   - 🔄 [#038 Asynchronous Programming — Essam Abdelnabi [Arabic] (43:48)](https://youtu.be/kDUDX3VJFEc) | [Microsoft Learn — Async in C#](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/) | [ASP.NET Core Performance Best Practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices)
3. **SOLID & DI Lifetimes:**
   - ⭐1 [SOLID 5: Dependency Inversion Principle — Mohamed El-Mehdi [Arabic] (22:32)](https://youtu.be/xuoT_p4WJKo)
   - ⭐2 [Dependency Injection Tutorial — Julio Casal (Lifetimes at 05:35)](https://youtu.be/KAHxOLmvGZo)
   - 🔄 [Solid In Practice Workshop — Essam Abdelnabi [Arabic] (54:21)](https://youtu.be/c_UnR7F900s)
4. **Extension Methods & LINQ IQueryable:**
   - ⭐ [C# Advanced: Extension Methods — Mohamed El-Mehdi [Arabic] (17:58)](https://youtu.be/g5lx0l7ikaQ)
   - 🔄 [#001 Pure Vs. Impure Function #linq — Essam Abdelnabi [Arabic] (27:19)](https://youtu.be/XLIqOBr7Cm0) | [Advanced C# LINQ Tutorial — Gavin Lon (IEnumerable vs IQueryable at 03:02:35)](https://youtu.be/5l2qA3Pc83M)

### 🛠️ Tasks:
- **Task 1 (Clean Architecture Solution Setup):** Create 4 projects (`Domain`, `Application`, `Infrastructure`, `Api`) with `.slnx`. Setup vertical feature slices.
  - ⭐ [Clean Architecture 10,000ft View — Essam Abdelnabi [Arabic] (19:37)](https://youtu.be/DRSZ6V-eKUY) (Layers at 09:18, request journey at 14:30)
  - 🔄 [Beginner's Guide to Clean Architecture — Milan Jovanović](https://youtu.be/TQdLgzVk2T8) | [Clean Architecture & CQRS in Action (.NET 10) — Essam Abdelnabi (09:18 to 20:25)](https://youtu.be/v9fRFyg414k) | [Microsoft Learn — Common Web Application Architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
- **Task 2 (Domain Modeling & Invariants):** Model Entities (`ApplicationUser`, `RefreshToken`, `Category`, `Campaign`, `Donation`, `PaymentAttempt`, `IdempotencyRecord`) and Value Objects.
  - ⭐ [Tactical DDD — Mohamed El-Mehdi [Arabic] (33:40)](https://youtu.be/bHEDV62ATNM) (Entity at 09:00, ValueObject at 19:10, Aggregate Root at 24:55)
  - 🔄 [Value Objects in C# — Milan Jovanović (13:54)](https://youtu.be/P5CRea21R2E) | [What Is An Entity — Milan Jovanović](https://youtu.be/00tCda35Bvk) | [#003 Immutable Vs. Mutable — Essam Abdelnabi](https://youtu.be/bbdmknnwcP8)
- **Task 3 (EF Core Persistence & Migrations):** `AppDbContext`, Fluent API configs (`IEntityTypeConfiguration<T>`), relationships, and initial migration.
  - ⭐ [#010 EF Migration in Arabic — Essam Abdelnabi (46:32)](https://youtu.be/WqiGu_MQ-xw) (AppDbContext at 11:45, Configurations at 20:10, ApplyConfigurations at 30:55, Migrations at 34:45)
  - 🔄 [Correct Way to Run Database Migrations — Nick Chapsas](https://youtu.be/StBf3-M5WdM)
- **Task 4 (Generic Repository & Unit of Work):** `IGenericRepository<T>` and `IUnitOfWork`. Direct reads bypass repo with `AsNoTracking`.
  - ⭐1 [Generic Repository — Mohamed El-Helaly [Arabic] (11:57)](https://youtu.be/7Lc9NRaPMr0)
  - ⭐2 [UnitOfWork — Mohamed El-Helaly [Arabic] (03:37)](https://youtu.be/-72HfsPv9Xs)
  - 🔄 [Transactions & Unit of Work — Milan Jovanović](https://youtu.be/25H84BXcr9M)
- **Task 5 (Soft Delete & Global Query Filter):** `ISoftDeletable` + `SoftDeleteInterceptor` + `HasQueryFilter`.
  - ⭐ [#019 Interceptors EF Core — Essam Abdelnabi [Arabic] (22:02)](https://youtu.be/hEMCDxqURr8) (ISoftDeletable at 05:05, Interceptor at 10:40, HasQueryFilter at 20:10)
  - 🔄 [Soft Deletes With EF Core — Milan Jovanović (12:39)](https://youtu.be/zwBaWiu-rF4) | [Global Query Filters — Milan Jovanović](https://youtu.be/q09fw5OCa_w)
- **Task 6 (Options Pattern & Startup Validation):** `JwtOptions`, `DonationOptions`, and `ValidateOnStart()`.
  - ⭐ [#11 Options Pattern in ASP.NET Core 9 — CodingClass [Arabic] (13:55)](https://youtu.be/S3ygXq94dLo)
  - 🔄 [Options Pattern in ASP.NET Core — Milan Jovanović](https://youtu.be/wxYt0motww0)
- **Task 7 (Identity & Authentication):** Register, Login, hashed Refresh Tokens with rotation, and Admin seed.
  - ⭐1 [Identity Tables — Mohamed El-Helaly [Arabic] (07:13)](https://youtu.be/zshypyVIVFU) (or [.NET 10 Identity — Remigiusz Zalewski (14:50)](https://youtu.be/g9YSB4bXhes))
  - ⭐2 [JWT Endpoints — El-Helaly](https://youtu.be/B7hQY1YckR0) + [Registration Part 1 (12:48)](https://youtu.be/FqQjESC4Gvs) + [Registration Part 2 (06:26)](https://youtu.be/PXKrGK4zjuM)
  - ⭐3 [Refresh Tokens — El-Helaly (10:18)](https://youtu.be/5g0aq4g17yg) + [New JWT with Refresh Token (05:55)](https://youtu.be/jefxQ2xFM8I)
  - 🔄 [Microsoft Learn — Identity API Endpoints](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-api-authorization) | [JWT Full Course .NET 9 — Patrick God (1:13:38)](https://youtu.be/6EEltKS8AwA) | [Master Refresh Tokens — Milan Jovanović (17:19)](https://youtu.be/kR_9gRBeRMQ) | [ASP.NET Web API 8 Identity — Fullstack Dev (2:23:36)](https://youtu.be/tXUtSGvHMeg)
- **Task 8 (Categories Module):** Bilingual Categories CRUD (Admin manage, public view, restrict delete on linked campaigns).
- **Task 9 (API Documentation & Bearer Security):** Scalar OpenAPI with JWT Bearer scheme on .NET 10 (or Swashbuckle on .NET 8).
  - ⭐ [No Swagger? Replace with SCALAR! — Israel Quiroz (12:15)](https://youtu.be/J8i1RWowjwc) (Bearer transformer at 09:50) | [Swagger is Gone! — Nick Chapsas (09:34)](https://youtu.be/8yI4gD1HruY) | [Swagger JWT — El-Helaly (10:44)](https://youtu.be/LGPiZcacG5g)
- **Task 10 (Repository Documentation):** Initial setup guide in `README.md`.

### 💡 Post-Sprint 1 Enrichment:
- [Clean Architecture vs. Vertical Slice — Derek Comartin](https://youtu.be/Az4Z0HJYl-U)
- [Simplest Vertical Slice Architecture — Milan Jovanović](https://youtu.be/dQdXHRkePr8)
- [Project Setup & Clean Architecture Solution Layout](https://youtu.be/tLk4pZZtiDY)
- [DDD Strategic Design 01 — Mohamed El-Mehdi [Arabic]](https://youtu.be/AV3yzj6Ck7k)
- [DDD Bounded Contexts Explained](https://youtu.be/kLLsVT_53bw)
- [Specification Pattern in EF Core — Milan Jovanović](https://youtu.be/bEyBtpnCqVY)
- [High-Performance Data Access with Dapper — Nick Chapsas](https://youtu.be/C763K-VGkfc)
- [Validate Options with FluentValidation — Nick Chapsas](https://youtu.be/I0YPTeCYvrE)
- [What Is JWT and How Does It Work?](https://youtu.be/7Q17ubqLfaM)
- [OAuth 2.0 and OpenID Connect in Plain English](https://youtu.be/t18YB3xDfXI)

---

## 🟩 Sprint 2 — Campaigns, CQRS, MediatR & Unit Testing

> **Period:** 07/11/2026 → 21/11/2026 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **CQRS & Mediator Mindset:** Separation of write commands and read queries.
2. **Cooperative Cancellation:** CancellationToken propagation to avoid orphaned database tasks.

### 🛠️ Tasks:
- **Task 1 (CQRS with MediatR [12.5.0]):** Refactor all use cases into commands/queries and thin controllers.
  - ⭐ [23 Add mediator pattern with CQRS — Ahmed Mohamady [Arabic] (14:05)](https://youtu.be/W7_zEf3eRQA)
  - 🔄 [CQRS & MediatR in ASP.NET Core — Patrick God](https://youtu.be/BewUyKLZjtc) | [Clean Architecture & CQRS in Action (.NET 10) — Essam Abdelnabi (26:25 to 43:20)](https://youtu.be/v9fRFyg414k)
- **Task 2 (CancellationToken & TimeProvider):** Forward cancellation token to EF Core and inject `TimeProvider`.
  - ⭐1 [Cancellation Token in .NET — Rahul Nath (18:42)](https://youtu.be/39gIPekzpjs) (Forwarding RequestAborted at 10:40–18:00)
  - ⭐2 [TimeProvider in .NET 8+ — Nick Chapsas (09:47)](https://youtu.be/xa6YusjCFF0) (DI registration at 04:55, mocking time at 05:25)
  - 🔄 [CancellationTokenSource in C# — Study DotNet [Arabic]](https://youtu.be/2xf6n0SpkNo) | [FakeTimeProvider in .NET — Nick Chapsas](https://youtu.be/acjCxNlWpJQ)
- **Task 3 (Optimistic Concurrency Control):** `byte[] RowVersion` on `Campaign` to prevent lost updates (409 Conflict).
  - ⭐ [Optimistic Concurrency in .NET [Arabic] — Islam Helmy (12:24)](https://youtu.be/VhuFcGP6H0E)
  - 🔄 [Optimistic vs Pessimistic Locking — Milan Jovanović](https://youtu.be/lLDBVMj5He8)
- **Task 4 (Result Pattern & Error Catalogue):** Standardize `Result<T>` and `Error` catalogue, eliminating business exceptions.
  - ⭐ [Handle Errors in C# using Exception and Result Pattern — Loai Saber [Arabic] (18:08)](https://youtu.be/hfnWW2trOlY)
  - 🔄 [Result Pattern in C# — Milan Jovanović](https://youtu.be/WCCkEe_Hy2Y)
- **Task 5 (Validation Behavior):** MediatR `ValidationBehavior` executing FluentValidation automatically.
  - ⭐ [Clean Architecture & CQRS in Action — Essam Abdelnabi (Pipeline at 05:50, Behavior at 21:40, Validator at 43:25)](https://youtu.be/v9fRFyg414k)
  - 🔄 [Validation with MediatR and FluentValidation — Milan Jovanović](https://youtu.be/85dxwd8HzEk)
- **Task 6 (Global Exception Handling):** `IExceptionHandler` + RFC 9457 `ProblemDetails`.
  - ⭐ [#9 Exception Handling in ASP.NET Core 9 — CodingClass [Arabic] (21:35)](https://youtu.be/Efmvo90NuiA)
  - 🔄 [Global Error Handling in .NET 8 — Milan Jovanović](https://youtu.be/uOEDM0c9BNI) | [Exception Handling in ASP.NET Core — Julio Casal](https://youtu.be/HTCb1zXQ6E4)
- **Task 7 (Campaigns Module):** CMP-01 to CMP-06, CMP-10, CMP-11 (Create, update, soft delete, details with `progressPercent` and `daysLeft`).
- **Task 8 (Unit Testing):** At least 15 passing unit tests using xUnit, Moq `[4.18.4]`, and FluentAssertions `[7.0.0]`.
  - ⭐1 [#003 Unit Testing Your Code — Essam Abdelnabi [Arabic] (26:10)](https://youtu.be/cAyROJ84tBs)
  - ⭐2 [#004 Mocking Dependencies (Moq) — Essam Abdelnabi [Arabic] (26:11)](https://youtu.be/JaIeeGqsO9M)
  - ⭐3 [2. Fluent Assertions in xUnit — Coding-Future (38:38)](https://youtu.be/rr9z92lm2no)
  - 🔄 [NSubstitute Mocking Framework — Nick Chapsas](https://youtu.be/LcQYv0cBWk0) | [FluentAssertions v7 vs Shouldly — Nick Chapsas](https://youtu.be/ZFc6jcaM6Ms) | [.NET Unit Testing Tutorial — Julio Casal](https://youtu.be/dsD0CMgPjUk)

### 💡 Post-Sprint 2 Enrichment:
- [#001 Intro to Unit Testing in C# — Essam Abdelnabi [Arabic]](https://youtu.be/HzjMcd1jEio)
- [In-Memory DbContext Testing Pitfalls — Nick Chapsas](https://youtu.be/VM6PIKkEg4s)
- [Code Coverage Metrics in .NET — Essam Abdelnabi](https://youtu.be/W5w-5EK94wc)
- [TDD with xUnit — Mohamad Lawand](https://youtu.be/7YghGdGE7d0)
- [MediatR Commercial Licensing Context — Nick Chapsas](https://youtu.be/bNuQ9xOG9eQ)
- [CQRS Architectural Trade-offs — Mohamad Lawand](https://youtu.be/sUjNZAYTZwI)
- [FluentValidation Deep Dive — Nick Chapsas](https://youtu.be/Ut6mRRFT2vM)

---

## 🟨 Sprint 3 — Donations Core, Resource Authorization & Production API Engineering

> **Period:** 22/11/2026 → 06/12/2026 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Docker Container Basics (Mandatory for running Redis):** `docker run -d -p`, `docker ps`, `docker logs`, `docker stop`. Launch `docker run -d --name redis -p 6379:6379 redis:alpine`.
   - ⭐1 [Basic Commands in Docker Part 5 — Codographia (15:20)](https://youtu.be/cB7Qx13YOFM)
   - ⭐2 [Basic Commands Part 6 — Codographia (20:29)](https://youtu.be/w_fnmXy9ZEk)
   - 🔄 [Docker in 5 Minutes — Yehia Tech [Arabic]](https://youtu.be/yRkE-1Lq0qo) | [Docker For Developers Ep 1 — Ramy Hakam [Arabic]](https://youtu.be/dTZZ-4PibHg)

### 🛠️ Tasks:
- **Task 1 (Resource-Based Authorization):** CampaignManager manages own campaigns only; 404 returned for unowned private resources.
  - ⭐1 [Resource-based authorization — Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased)
  - ⭐2 [Who Owns This Order? — Code Story (01:46)](https://youtu.be/29x5WmcUNtM)
  - 🔄 [ASP.NET Web API 8 Identity — Fullstack Dev (at 02:19:44)](https://youtu.be/tXUtSGvHMeg) | [14- Create Policy Part 1 — Mahmoud Hamza [Arabic]](https://youtu.be/tdMrNKPO39U) | [Custom Authorization Handler — Milan Jovanović](https://youtu.be/SZtZuvcMBA0)
- **Task 2 (Donations Core & Transactions):** DON-01 to DON-06. Atomic EF Core transactions for donation settlement.
  - ⭐ [#020 DB Transaction EF Core — Essam Abdelnabi [Arabic] (32:57)](https://youtu.be/m2RDSxGPuhs) (BeginTransaction at 19:15, Rollback at 24:40, Savepoints at 28:50)
  - 🔄 [Transactions in EF Core — Milan Jovanović](https://youtu.be/25H84BXcr9M) | [Avoid EF Core Transaction Mistakes — Milan](https://youtu.be/cUszBUYSFzw)
- **Task 3 (Pagination, Filtering, Sorting):** `PagedList<T>` with database-level `Skip` and `Take` via `IQueryable`.
  - ⭐1 [15. Pagination in Web Api Part 1 — Coding-Future (27:51)](https://youtu.be/XDkUliEyPms)
  - ⭐2 [Filtering, Sorting And Pagination — Milan Jovanović (24:02)](https://youtu.be/X8zRvXbirMU)
  - 🔄 [16. Pagination Part 2 — Coding-Future](https://youtu.be/8vum4WH8PVs) | [Paging Math and Implementation — Essam Abdelnabi (at 56:35)](https://youtu.be/fb8j6CjyWWw)
- **Task 4 (Distributed Caching with Redis):** `IDistributedCache` for active campaigns, with immediate invalidation on updates.
  - ⭐ [Caching Explained: In-Memory vs Redis — Amr Mohamed [Arabic] (21:11)](https://youtu.be/AIBOcH_dAX8)
  - 🔄 [Distributed Caching With Redis in ASP.NET Core — Milan Jovanović](https://youtu.be/Tt5zIKVMMbs)
- **Task 5 (Localization ar-EG / en-US):** `IStringLocalizer` and `.resx` handling client `Accept-Language` headers.
  - ⭐1 [Globalization and Localization Part 1 — Mohamed El-Helaly [Arabic] (08:38)](https://youtu.be/PyCGEO4b6F0)
  - ⭐2 [16. Use Localization In APIs — Mohamed El-Helaly [Arabic] (04:37)](https://youtu.be/YzOarEJDdrY)
  - 🔄 [Microsoft Learn — Localization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization)
- **Task 6 (Versioning & Rate Limiting):** `Asp.Versioning.Mvc` and built-in Rate Limiting returning 429 Too Many Requests.
  - ⭐1 [.NET 9: ASP.NET Core Web API Versioning — Mohamad Lawand (12:55)](https://youtu.be/DkZd-5z0CeI)
  - ⭐2 [How To Implement Rate Limiting In .NET — Milan Jovanović (09:42)](https://youtu.be/1tPVVDEDGtE)
  - 🔄 [Rate Limiting Algorithms — Mohamad Lawand](https://youtu.be/bOfOo3Zsfx0)
- **Task 7 (CORS Policies):** Configured allowed origins, headers, and methods.
  - ⭐ [6. Enable CORS (Updated) — Mohamed El-Helaly [Arabic] (05:39)](https://youtu.be/_YXcxm-O06I)
  - 🔄 [Microsoft Learn — Enable CORS](https://learn.microsoft.com/en-us/aspnet/core/security/cors)

### 💡 Post-Sprint 3 Enrichment:
- [Cursor-Based Pagination in .NET](https://youtu.be/U3LcKY19z_4)
- [Distributed Caching with Redis — Mohamad Lawand](https://youtu.be/6HZVu3kGOrg)
- [Minimal APIs vs Controllers in ASP.NET Core — Milan Jovanović](https://youtu.be/gsAuFIhXz3g)
- [Minimal API Versioning in ASP.NET Core](https://youtu.be/F9j3X6KuIpw)
- [Building High-Performance APIs with FastEndpoints — Nick Chapsas](https://youtu.be/oqNu0xavAyc)

---

## 🟧 Sprint 4 — Hangfire, MailKit, QuestPDF, File Security & Structured Logging

> **Period:** 07/12/2026 → 21/12/2026 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Local SMTP Container via Mailpit:** `docker run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit` (Web UI at http://localhost:8025).

### 🛠️ Tasks:
- **Task 1 (Hangfire Infrastructure & Dashboard):** SQL Server job storage, background worker setup, and protected `/hangfire` dashboard.
  - ⭐1 [1. Prepare Your Project — Mohamed El-Helaly [Arabic] (10:42)](https://youtu.be/hnK1qn6ZjKo)
  - ⭐2 [2. Enqueue Jobs — El-Helaly (06:07)](https://youtu.be/uqWRHmPm30E)
  - ⭐3 [3. Scheduled Jobs — El-Helaly (04:28)](https://youtu.be/_3C9BVwwM80)
  - ⭐4 [4. Recurring Jobs — El-Helaly (06:19)](https://youtu.be/GjsJrMtNES0)
  - ⭐5 [Hangfire Dashboard Authorization Documentation](https://docs.hangfire.io/en/latest/configuration/using-dashboard.html)
  - 🔄 [Hangfire in .NET Core (Complete Guide) — Coding Droplets (16:17)](https://youtu.be/lsP5ZjcFtMM)
- **Task 2 (MailKit Email Service & Mailpit):** Transactional email sending directed to local Mailpit container.
  - ⭐1 [Sending Emails With .NET Core Part 1 — Mohamed El-Helaly [Arabic] (10:33)](https://youtu.be/J23_c3uSyD0)
  - ⭐2 [Mailpit Docker Documentation](https://mailpit.axllent.org/docs/install/docker/)
  - 🔄 [Send Emails with MailKit — tutorialsEU](https://youtu.be/lCHKwyekbT4) | [Easy Email Verification in .NET with Papercut/Mailpit — Milan Jovanović (04:50)](https://youtu.be/KtCjH-1iCIk)
- **Task 3 (QuestPDF Receipt Generation):** Dynamic PDF receipt layout with Community License.
  - ⭐ [QuestPDF - The BEST PDF Generator for .NET — Claudio Bernasconi (12:07)](https://youtu.be/T89A_7dz1P8)
  - 🔄 [Generate PDF Invoice in ASP.NET Core — M Alvin Maulana](https://youtu.be/eiry9Zli5Rc) | [QuestPDF Official Invoice Tutorial](https://www.questpdf.com/quick-start.html)
- **Task 4 (File Upload Security & Magic Bytes):** Campaign cover uploads with raw binary file signature validation.
  - ⭐1 [Upload Files With Fake Extensions Part 1 — Mohamed El-Helaly [Arabic] (09:38)](https://youtu.be/mh0ZTXfSdBI)
  - ⭐2 [Upload Files Part 2 — Mohamed El-Helaly [Arabic] (11:10)](https://youtu.be/08Ke6nhNWK8)
  - ⭐3 [Microsoft Learn — File signature validation (Magic Bytes)](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads)
  - 🔄 [Design a File Upload Service — Milan Jovanović](https://youtu.be/ka_WCmNdybE)
- **Task 5 (Background Workflows):** Fire-and-forget job for receipt/email upon donation, and recurring daily job for expired campaigns.
- **Task 6 (Structured Logging with Serilog & Seq):** Enrich logs with `TraceId`, `Environment`, and request logging.
  - ⭐ [.NET Logging Using Seq — Bishoy Youssef [Arabic] (20:26)](https://youtu.be/BwcsVcLgFLw)
  - 🔄 [Structured Logging With Serilog — Milan Jovanović](https://youtu.be/nVAkSBpsuTk) | [63. Serilog in Clean Architecture — Coding-Future](https://youtu.be/4Sf156KA4N0)

### 💡 Post-Sprint 4 Enrichment:
- [Transactional Outbox Pattern with Hangfire — Milan Jovanović](https://youtu.be/gytZxzT2IWY)
- [Automatic Logging via MediatR Pipeline Behaviors](https://youtu.be/JVX9MMpO6pE)
- [TickerQ: Modern Background Job Queue in .NET](https://youtu.be/x0dfj95Cj0U)
- [Zero-Allocation File Streaming in ASP.NET Core — Raw Coding](https://youtu.be/l69kUe1fNSc)
- [Logging Crash Course in .NET](https://youtu.be/XrCxy0SNRqM)

---

## 🟥 Sprint 5 — Stripe Checkout, Webhooks, Idempotency & Integration Testing

> **Period:** 22/12/2026 → 05/01/2027 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Docker Daemon Active:** Mandatory for Testcontainers spinning up disposable database containers.
2. **Stripe CLI Installed:** For local webhook forwarding (`stripe listen --forward-to ...`).

### 🛠️ Tasks:
- **Task 1 (Stripe Checkout Session):** PAY-01 to PAY-03. Create session with `client_reference_id`, metadata, and minor units (cents).
  - ⭐1 [How To Easily Integrate STRIPE Into Your .NET API — Israel Quiroz (38:45)](https://youtu.be/2sZXrUFvZ3E)
  - ⭐2 [Stripe Official Checkout Sessions Documentation](https://docs.stripe.com/api/checkout/sessions)
- **Task 2 (Webhook HMAC Verification):** PAY-04. Validate `Stripe-Signature` via `EventUtility.ConstructEvent`.
  - ⭐1 [159- Creating Stripe Web Hook Endpoint — Mohamed Awadallah [Arabic] (10:58)](https://youtu.be/WZu0lHvNYjI)
  - ⭐2 [Stripe Official Webhooks Guide](https://docs.stripe.com/webhooks)
- **Task 3 (Stripe CLI Testing):** Test local webhooks with `stripe listen` and signing secret verification.
- **Task 4 (API & Webhook Idempotency):** Handle `Idempotency-Key` headers and deduplicate duplicate webhook event IDs in database.
  - ⭐1 [How To Make Your API Idempotent — Milan Jovanović (14:26)](https://youtu.be/smXAgcdJzLc)
  - ⭐2 [Stripe Documentation — Handle duplicate webhook events](https://docs.stripe.com/webhooks#handle-duplicate-events)
- **Task 5 (Integration Testing with Testcontainers):** End-to-end user workflows against real SQL Server in Docker via `WebApplicationFactory`.
  - ⭐ [Stop Mocking Your Database! Use Testcontainers in .NET — Milan Jovanović (15:59)](https://youtu.be/ssRE0pBNvpE)
  - 🔄 [Integration Testing in ASP.NET Core — Julio Casal](https://youtu.be/RXSPCIrrjHc)

### 💡 Post-Sprint 5 Enrichment:
- [Building Resilient Cloud Microservices with Polly (.NET Foundation)](https://youtu.be/L9_fGJOqzbM)

---

## 🟪 Sprint 6 — EF Core Aggregations, Docker, Health Checks, CI/CD & Final Delivery

> **Period:** 06/01/2027 → 20/01/2027 | **Evaluation:** 100 Points

### 📌 Prerequisites:
1. **Docker Compose & Multi-Container Networking:** Service discovery and environment variables.

### 🛠️ Tasks:
- **Task 1 (Admin Analytics Dashboard & Aggregations):** High-performance reporting (`Sum`, `Count`, `GroupBy`) with `AsNoTracking` and Redis caching.
  - ⭐1 [EF Core Query Data: Projections & Aggregations — Essam Abdelnabi [Arabic] (1:11:58)](https://youtu.be/fb8j6CjyWWw)
  - ⭐2 [Microsoft Learn — Complex Query Operators (GroupBy in EF Core)](https://learn.microsoft.com/en-us/ef/core/querying/complex-query-operators)
- **Task 2 (Admin Campaign Management & Overrides):** Administrative status controls and campaign cancellations.
- **Task 3 (Production Multi-Stage Dockerfile):** Lean production runtime image running under non-root user.
  - ⭐ [Dockerize ASP.NET Core Application Part 12 — Codographia (27:04)](https://youtu.be/nINTOS2fhVY)
  - 🔄 [Docker Full Course For .NET — Julio Casal](https://youtu.be/cWMztQwIQNs)
- **Task 4 (Full Stack Orchestration via Docker Compose):** Spin up API, SQL Server, Redis, and Mailpit together.
  - ⭐ [Build a Complete .NET 10 Dev Environment with Docker Compose — Milan Jovanović (16:00)](https://youtu.be/svfxvsGfLlU)
  - 🔄 [Docker Compose Tutorial — Mosh Hamedani](https://youtu.be/HG6yIjZapSA)
- **Task 5 (ASP.NET Core Health Checks):** Expose `/health`, `/health/ready`, and `/health/live` validating SQL and Redis connectivity.
  - ⭐ [How to Add Health Checks in ASP.NET Core — Milan Jovanović (11:51)](https://youtu.be/4abSfjdzqms)
  - 🔄 [Best Way to Add Health Checks in .NET — Nick Chapsas](https://youtu.be/p2faw9DCSsY)
- **Task 6 (Automated CI/CD Pipeline via GitHub Actions):** Automated build, unit tests, and integration tests pipeline on every PR.
  - ⭐ [1. CI/CD Pipeline Using GitHub Actions — Coding-Future (23:29)](https://youtu.be/Zv-NJBRmoVU)
  - 🔄 [GitHub Actions Full Tutorial — TechWorld with Nana](https://youtu.be/R8_veQiYBjI) | [GitHub Actions Complete Course — Ahmed El-Fakharany [Arabic]](https://youtu.be/7gJFHjXscr8)
- **Task 7 (Final Documentation & Architecture Guide):** Production architecture and setup guide in `README.md`.
  - ⭐ [How to Write a Professional README.md — Fekra w Code [Arabic] (14:27)](https://youtu.be/ATVJiw4JrUA)
  - 🔄 [Write an Awesome GitHub README — Learn Fast Make Things](https://youtu.be/E6NO0rgFub4)

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
