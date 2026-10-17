# Sprint 1 — Clean Architecture, Domain Modeling, Identity & Categories

> **Project:** DonationHub — Donation Management & Crowdfunding Platform  
> **Duration:** 15 Days (23/10/2026 to 06/11/2026)  
> **Meeting Schedule:**  
> - **Stand-up 1 (Day 5):** 27/10/2026 — Session Leader: **Mohamed Atef**  
> - **Stand-up 2 (Day 10):** 01/11/2026 — Session Leader: **Asma ELhadidy**  
> - **Closing & Deadline (Day 15):** 06/11/2026 — Session Leader: **Kareem Zein**  
> - **Code Review & Corrections (Days 16–18):** 07/11/2026 to 09/11/2026  
> **Evaluation:** 100 Points  
> **Target Framework:** .NET 10 (`net10.0` with `.slnx` solution format and `Program.cs` composition root — fully compatible with .NET 8 / .NET 9)

---

## 🎯 Sprint Goal

Establish the rock-solid architectural foundation of DonationHub by implementing Clean Architecture organized by Vertical Slices. Set up EF Core database persistence with migrations and interceptors, and deliver two fully working production modules: **Identity (Registration, Login, JWT & Refresh Token Rotation)** and **Categories (Taxonomy management with domain rules)**.

```text
Domain Model (Entities + Invariants + Value Objects)
        ↓
Clean Architecture (4 Layers + Dependency Inversion)
        ↓
Vertical Slice Organization inside Application Layer
        ↓
EF Core Persistence (Fluent API + Migrations + Interceptors)
        ↓
Secure & Documented API (JWT Bearer + Scalar / Swagger)
```

---

## 📌 Sprint Prerequisites

Before writing code for this sprint, learners must verify and solidify their understanding of the following foundational concepts:

### 1. Git & GitHub Pull Request Workflow (PR Flow)
*Learners must be proficient in forking the repository, cloning, managing branches, and submitting clean Pull Requests for mentor review.*
- **⭐ Required Primary:** [Git and GitHub for Beginners — freeCodeCamp (1:08:29)](https://youtu.be/RGOj5yH7evk)
- **🔄 Alternative / Deep Dive:** [Git and GitHub | Ahmed Sami [Arabic] (PR walkthrough from 05:56:20)](https://youtu.be/Q6G-J54vgKc)

### 2. Async/Await Mechanics & The .NET ThreadPool
*Understand I/O Completion Ports (IOCP) and why wrapping EF Core queries in `Task.Run` is a dangerous anti-pattern in ASP.NET Core.*
- **⭐ Required Primary:** Asynchronous programming and non-blocking I/O concepts.
- **🔄 Alternative / Deep Dive:**
  - [#038 Asynchronous Programming in C# — Essam Abdelnabi [Arabic] (43:48)](https://youtu.be/kDUDX3VJFEc)
  - [Microsoft Learn — Asynchronous programming with async and await](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)
  - [Microsoft Learn — ASP.NET Core Performance Best Practices](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/best-practices)

### 3. SOLID Principles & DI Service Lifetimes
*Master Dependency Inversion Principle (DIP) and accurately select service lifetimes: Transient vs. Scoped vs. Singleton.*
- **⭐ Required Primary:**
  - ⭐1 [SOLID 5: Dependency Inversion Principle — Mohamed El-Mehdi [Arabic] (22:32)](https://youtu.be/xuoT_p4WJKo)
  - ⭐2 [Dependency Injection C# Tutorial — Julio Casal (Service lifetimes compared at 05:35)](https://youtu.be/KAHxOLmvGZo)
- **🔄 Alternative / Deep Dive:** [Solid In Practice Workshop — Essam Abdelnabi [Arabic] (54:21)](https://youtu.be/c_UnR7F900s)

### 4. Extension Methods & LINQ: `IEnumerable` vs. `IQueryable`
*Understand Expression Trees, deferred execution, and why filtering in-memory (`IEnumerable`) instead of in-database (`IQueryable`) ruins backend performance.*
- **⭐ Required Primary:** [C# Advanced: Extension Methods — Mohamed El-Mehdi [Arabic] (17:58)](https://youtu.be/g5lx0l7ikaQ)
- **🔄 Alternative / Deep Dive:**
  - [#001 Pure Vs. Impure Function #linq — Essam Abdelnabi [Arabic] (27:19)](https://youtu.be/XLIqOBr7Cm0)
  - [Advanced C# LINQ Tutorial — Gavin Lon (IEnumerable vs IQueryable at 03:02:35)](https://youtu.be/5l2qA3Pc83M)

---

## 📋 Engineering Decisions & Conventions

| Area | Decision | Details & Commands |
|---|---|---|
| **Target .NET Version** | .NET 10 (`net10.0`) | All configuration is consolidated in `Program.cs`. On .NET 9/10, Swashbuckle is replaced with built-in OpenAPI + `Scalar.AspNetCore` (`builder.Services.AddOpenApi();` and `app.MapOpenApi();`). Backward compatible with .NET 8. |
| **Solution Format** | `.slnx` | Modern solution file format (`dotnet new sln --format slnx` or standard `.sln` on older SDKs). |
| **Domain Modeling** | Rich Domain Model | Anemic models are prohibited. Entity setters are private. Encapsulate business invariants. Value Objects prevent primitive obsession. |
| **Soft Delete** | Interceptor-Driven | Entity implements `ISoftDeletable`. `SoftDeleteInterceptor` intercepts `EntityState.Deleted` and converts it to `IsDeleted = true`. Global query filter `HasQueryFilter(e => !e.IsDeleted)`. |
| **Configuration** | Options Pattern | Strongly-typed options validated with Data Annotations and enforced at startup via `.ValidateOnStart()`. |

---

## 🛠️ Sprint Tasks

### Task 1 — Clean Architecture Solution Setup

Create the four core projects and establish strict, unidirectional dependency flow:

```text
DonationHub.Domain        (Zero external dependencies)
        ↑
DonationHub.Application   (References Domain only)
        ↑
DonationHub.Infrastructure (References Application & Domain to implement abstractions)
        ↑
DonationHub.Api           (Presentation layer & Composition Root, references Application & Infrastructure)
```

- In `DonationHub.Application`, organize code into **feature folders (Vertical Slices)** rather than flat technical categories.
- Create modular DI registration extensions for each layer (`AddApplicationServices`, `AddInfrastructureServices`) called inside `Program.cs`.

**Resources:**
- **⭐ Required Primary:** [Clean Architecture 10,000ft View — Essam Abdelnabi [Arabic] (19:37)](https://youtu.be/DRSZ6V-eKUY) (Layers & dependency rule at 09:18, request journey at 14:30).
- **🔄 Alternative / Deep Dive:**
  - [Beginner's Guide to Clean Architecture — Milan Jovanović](https://youtu.be/TQdLgzVk2T8)
  - [Clean Architecture & CQRS in Action (.NET 10) — Essam Abdelnabi (09:18 to 20:25 for CLI project creation and .slnx)](https://youtu.be/v9fRFyg414k)
  - [Microsoft Learn — Common web application architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)

---

### Task 2 — Domain Modeling & Invariants (Domain Layer)

Model the core domain entities and invariants according to the DonationHub SRS:
- **Entities:** `ApplicationUser`, `RefreshToken`, `Category`, `Campaign`, `Donation`, `PaymentAttempt`, `ProcessedStripeEvent`, `IdempotencyRecord`.
- **Enums:**
  - `CampaignStatus` (Active, Completed, Expired)
  - `DonationStatus` (Pending, Succeeded, Failed)
  - `PaymentAttemptStatus` (Initiated, Succeeded, Failed)
- **Value Objects:**
  - Create a base `ValueObject` with equality comparison.
  - Implement Value Objects where appropriate (e.g., `Money` encapsulating amount and currency).
- **Domain Invariants:**
  - Encapsulate property setters (`private set;`).
  - Business rules: `GoalAmount > 0`, `EndDate` must be in the future, campaign transitions must follow strict status rules.

**Resources:**
- **⭐ Required Primary:** [DDD 06: Tactical Design - Getting Started — Mohamed El-Mehdi [Arabic] (33:40)](https://youtu.be/bHEDV62ATNM) (Entity at 09:00, ValueObject at 19:10, Aggregate Root at 24:55).
- **🔄 Alternative / Deep Dive:**
  - [Value Objects in C# — Milan Jovanović (13:54)](https://youtu.be/P5CRea21R2E) (Base class at 02:40, factory methods at 08:35).
  - [What Is An Entity — Milan Jovanović](https://youtu.be/00tCda35Bvk)
  - [#003 Immutable Vs. Mutable — Essam Abdelnabi](https://youtu.be/bbdmknnwcP8)

---

### Task 3 — EF Core Persistence, Fluent API & Migrations

Configure database persistence in `DonationHub.Infrastructure`:
- Implement `AppDbContext`.
- Configure entities strictly using **Fluent API** via separate configuration classes implementing `IEntityTypeConfiguration<T>`.
- Register configurations dynamically using `ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly)`.
- Establish foreign keys and index strategies:
  - `Category` → `Campaigns` (One-to-Many, `DeleteBehavior.Restrict`).
  - `Campaign` → `Donations` (One-to-Many).
  - `User` → `Donations` and `User` → `Campaigns`.
- Configure `RowVersion` concurrency token (`byte[]`) on `Campaign`.
- Generate the initial migration and configure SQL Server connection string via options.

**Resources:**
- **⭐ Required Primary:** [#010 EF Migration in Arabic — Essam Abdelnabi (46:32)](https://youtu.be/WqiGu_MQ-xw) (AppDbContext setup at 11:45, IEntityTypeConfiguration at 20:10, ApplyConfigurations at 30:55, Migrations at 34:45).
- **🔄 Alternative / Deep Dive:** [Correct Way to Run Database Migrations — Nick Chapsas](https://youtu.be/StBf3-M5WdM)

---

### Task 4 — Generic Repository & Unit of Work

- Define `IGenericRepository<T>` and `IUnitOfWork` interfaces in `DonationHub.Application`.
- Implement them in `DonationHub.Infrastructure` backed by `AppDbContext`.
- The repository handles domain state changes and write operations.
- Direct read queries may project directly to DTOs using `AsNoTracking` for high-throughput queries.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Add The Base (Generic) Repository — Mohamed El-Helaly [Arabic] (11:57)](https://youtu.be/7Lc9NRaPMr0)
  - ⭐2 [Use Complete Method (UnitOfWork) — Mohamed El-Helaly [Arabic] (03:37)](https://youtu.be/-72HfsPv9Xs)
- **🔄 Alternative / Deep Dive:** [Transactions & Unit of Work — Milan Jovanović](https://youtu.be/25H84BXcr9M)

---

### Task 5 — Soft Delete Interceptor & Global Query Filters

- Define `ISoftDeletable` interface (`bool IsDeleted { get; set; }`, `DateTime? DeletedAtUtc { get; set; }`).
- Implement `SoftDeleteInterceptor : SaveChangesInterceptor` to intercept deletions and convert them to soft-deletions automatically.
- Apply `HasQueryFilter(e => !e.IsDeleted)` to exclude deleted records from normal queries, with support for `IgnoreQueryFilters()` when needed.

**Resources:**
- **⭐ Required Primary:** [#019 Interceptors EF Core — Essam Abdelnabi [Arabic] (22:02)](https://youtu.be/hEMCDxqURr8) (ISoftDeletable at 05:05, Interceptor at 10:40, HasQueryFilter at 20:10).
- **🔄 Alternative / Deep Dive:**
  - [Soft Deletes With EF Core — Milan Jovanović (12:39)](https://youtu.be/zwBaWiu-rF4)
  - [Global Query Filters — Milan Jovanović](https://youtu.be/q09fw5OCa_w)

---

### Task 6 — Options Pattern & Startup Validation

Create strongly-typed option classes with Data Annotations and register with `.ValidateOnStart()`:
- `JwtOptions` (SecretKey, Issuer, Audience, ExpiryMinutes, RefreshTokenExpiryDays)
- `SeedAdminOptions` (Email, Password, FullName)
- `DonationOptions` (MinAmount, MaxAmount, TimeoutMinutes, DefaultCurrency)
- Ensure the application **fails fast** on startup if any required configuration is missing or invalid.

**Resources:**
- **⭐ Required Primary:** [#11 Options Pattern in ASP.NET Core 9 — CodingClass [Arabic] (13:55)](https://youtu.be/S3ygXq94dLo).
- **🔄 Alternative / Deep Dive:** [Options Pattern in ASP.NET Core — Milan Jovanović](https://youtu.be/wxYt0motww0)

---

### Task 7 — Authentication & Identity (ASP.NET Core Identity & JWT)

Implement identity and authentication requirements from SRS §3.1:
- **IDN-01:** Registration `POST /auth/register` with Email, FullName, Password, and default `Donor` role.
- **IDN-02:** Strict password policy (8+ chars, upper, lower, digit, special char) and case-insensitive unique email check.
- **IDN-03 & IDN-04:** Login `POST /auth/login` returns a short-lived JWT Access Token (15 min) with claims (UserId, Email, Roles) and a secure Refresh Token.
- **IDN-05 & IDN-06:** Refresh token rotation: refresh tokens must be stored **hashed** in the database. When `/auth/refresh` is called, the old token is revoked and a new linked token is issued.
- **IDN-07:** Logout `POST /auth/logout` revokes the provided refresh token.
- **IDN-09:** Current user profile `GET /auth/me`.
- **IDN-10:** Deactivated users (`IsActive = false`) are prohibited from logging in or refreshing tokens.
- **IDN-11:** Seed initial Admin user at startup from `SeedAdminOptions`.
- **IDN-12:** Return generic error responses on login failures to prevent user enumeration attacks.
- **IDN-08:** Admin endpoint to create staff users `POST /admin/staff` (Roles: `Admin` or `CampaignManager`).

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 Identity Tables: [03 Create Database and Identity Tables — Mohamed El-Helaly [Arabic] (07:13)](https://youtu.be/zshypyVIVFU) (or for .NET 10: [Auth made easy with ASP.NET Core Identity (.NET 10) — Remigiusz Zalewski (14:50)](https://youtu.be/g9YSB4bXhes)).
  - ⭐2 JWT & Endpoints: [05 Add JWT Configuration — El-Helaly (06:35)](https://youtu.be/B7hQY1YckR0) + [06 Prepare Registration EndPoint (12:48)](https://youtu.be/FqQjESC4Gvs) + [07 Registration EndPoint Part 2 (06:26)](https://youtu.be/PXKrGK4zjuM).
  - ⭐3 Refresh Token Rotation: [4. Generate Refresh Token — El-Helaly (10:18)](https://youtu.be/5g0aq4g17yg) + [7. Generate New JWT Using Refresh Token (05:55)](https://youtu.be/jefxQ2xFM8I).
- **🔄 Alternative / Deep Dive:**
  - [JWT Authentication with .NET 9 Full Course — Patrick God (1:13:38)](https://youtu.be/6EEltKS8AwA) (PasswordHasher at 10:01, Scalar at 12:55, Token creation at 19:55, RefreshToken at 59:07).
  - [Master Refresh Tokens in ASP.NET Core — Milan Jovanović (17:19)](https://youtu.be/kR_9gRBeRMQ)
  - [ASP.NET Web API 8 Identity — Fullstack Dev (2:23:36)](https://youtu.be/tXUtSGvHMeg)

---

### Task 8 — Categories Module

Implement category endpoints and invariants from SRS §3.2:
- **CAT-01:** Admin creates, updates, and deletes categories.
- **CAT-02:** Category names are required in both Arabic and English (`NameAr`, `NameEn`), max 100 characters, unique per language, with optional bilingual descriptions.
- **CAT-03:** Anonymous access for listing all categories `GET /categories` and fetching a single category by id `GET /categories/{id}`.
- **CAT-04:** Attempting to delete a category that contains associated campaigns must be rejected with `Category.HasCampaigns` (HTTP 409 Conflict).

| Method | Path | Access |
|---|---|---|
| `GET` | `/categories` | Anonymous |
| `GET` | `/categories/{id}` | Anonymous |
| `POST` | `/categories` | Admin Only |
| `PUT` | `/categories/{id}` | Admin Only |
| `DELETE` | `/categories/{id}` | Admin Only |

---

### Task 9 — API Documentation & Bearer Security (Scalar / OpenAPI / Swagger)

- Set up interactive API documentation with Bearer Token authorization support to test protected endpoints.
- On **.NET 9 / .NET 10**: Install `Scalar.AspNetCore` and configure `BearerSecuritySchemeTransformer`.
- On **.NET 8**: Configure Swashbuckle SwaggerUI with `AddSecurityDefinition` and `AddSecurityRequirement`.

**Resources:**
- **⭐ Required Primary:**
  - On .NET 9/10: [No Swagger in .NET 9? Replace it with SCALAR! — Israel Quiroz (12:15)](https://youtu.be/J8i1RWowjwc) (Bearer transformer at 09:50).
  - Or Swagger fallback: [Swagger is Gone in .NET 9! Replace It With This — Nick Chapsas (09:34)](https://youtu.be/8yI4gD1HruY).
  - On .NET 8: [Swagger JWT Setup — Mohamed El-Helaly [Arabic] (10:44)](https://youtu.be/LGPiZcacG5g).

---

### Task 10 — Repository Documentation & Run Guide

Create an initial `README.md` in the project root documenting:
- Prerequisites (.NET 10 SDK, SQL Server).
- Database migration commands (`dotnet ef database update`).
- Default seeded Admin credentials.
- Instructions on accessing Scalar / Swagger documentation and testing protected endpoints.

---

## 📦 Sprint Deliverables

- [ ] Four-layer solution (`Domain`, `Application`, `Infrastructure`, `Api`) with correct references.
- [ ] Feature-based Vertical Slice folder structure in `Application`.
- [ ] Rich Domain model with encapsulated entities, Value Objects, and invariants.
- [ ] Fluent API configurations, relationships, indexes, and initial migration applied.
- [ ] `ISoftDeletable` with `SoftDeleteInterceptor` and global query filters.
- [ ] Generic Repository and Unit of Work abstractions and implementations.
- [ ] Options pattern configured with `ValidateOnStart()`.
- [ ] Full Identity lifecycle: Register, Login, JWT, hashed Refresh Token with rotation, Logout.
- [ ] Automated initial Admin seeding at application startup.
- [ ] Categories CRUD module with bilingual support and campaign association protection.
- [ ] Interactive API documentation (Scalar / Swagger) configured with JWT Bearer authentication.
- [ ] Clear project setup documentation in `README.md`.

---

## ✅ Definition of Done

- [ ] Solution builds cleanly with zero errors (`dotnet build`).
- [ ] `Domain` has zero dependencies on external packages or database providers.
- [ ] No secrets or passwords committed to Git (`appsettings.json` contains placeholders only).
- [ ] Invalid login attempts return generic messages to prevent user enumeration.
- [ ] Old refresh tokens are invalidated upon use and cannot be replayed.
- [ ] Deleting a category with associated campaigns returns HTTP 409 Conflict.
- [ ] Protected endpoints return 401 Unauthorized when requested without a token.
- [ ] The learner can articulate the rationale behind all architectural decisions, trade-offs, and alternatives during sprint review.

---

## 💡 Post-Sprint Enrichment

- [Clean Architecture vs. Vertical Slice — Derek Comartin](https://youtu.be/Az4Z0HJYl-U)
- [Simplest Vertical Slice Architecture — Milan Jovanović](https://youtu.be/dQdXHRkePr8)
- [DDD Strategic Design: Bounded Contexts — Mohamed El-Mehdi](https://youtu.be/AV3yzj6Ck7k)
- [Specification Pattern in EF Core — Milan Jovanović](https://youtu.be/bEyBtpnCqVY)
- [High-Performance Data Access with Dapper — Nick Chapsas](https://youtu.be/C763K-VGkfc)
- [OAuth 2.0 and OpenID Connect in Plain English](https://youtu.be/t18YB3xDfXI)