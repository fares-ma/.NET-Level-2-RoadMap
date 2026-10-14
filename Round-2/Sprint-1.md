# Sprint 1 — Clean Architecture, Identity & Categories

> Project: **DonationHub — Donation Management Platform**
> Duration: **2 weeks (~15 hours/week)**
> Evaluation: **100 points**

## Sprint Goal

Build the architectural foundation of DonationHub and deliver the first two working modules: Identity and Categories.

The learner should move from:

```text
Domain Model
    ↓
Clean Architecture
    ↓
Vertical Slice Organization
    ↓
EF Core Persistence
    ↓
Working API
```

## How to use this sprint plan

For every task:

1. Read the **MUST** resources before implementation.
2. Read **DEEP DIVE** only when the concept is unclear or the learner wants deeper understanding.
3. Implement the requested DonationHub change.
4. Verify the behavior manually through Swagger.
5. Add/update tests when the task naturally requires them.
6. Commit the work in small, meaningful commits.
7. Be able to explain the architectural decision during review.

**Important:** This document does not change the original Sprint scope. It expands the original tasks into an executable checklist.

## Sprint Tasks

### Task 1 — Solution Structure (Clean Architecture)

Create the four-layer solution:

```text
DonationHub.Domain        (no dependencies)
        ↑
DonationHub.Application   (depends on Domain)
        ↑
DonationHub.Infrastructure (depends on Application)
        ↑
DonationHub.Api           (depends on Application + Infrastructure for composition)
```

Configure dependency direction correctly. Infrastructure implements abstractions defined by inner layers.

Inside `Application`, organize by **feature (vertical slice)**, not by technical type.

**Required output**
- Four projects with correct references
- Composition root in `Api` (DI registration)
- Architecture diagram
- Short explanation of dependency direction and why vertical slices were chosen

### Resources
- **MUST** — [Microsoft — Common web application architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
- **MUST** — [Microsoft — Clean Architecture with ASP.NET Core](https://learn.microsoft.com/en-us/shows/dotnetconf-2022/clean-architecture-with-aspnet-core-7)
- **DEEP DIVE** — Robert C. Martin — `Book: Clean Architecture`
- **VIDEO** — [Milan Jovanović — Clean Architecture from Scratch](https://www.youtube.com/watch?v=yF9SwL0p0Y0)
- **VIDEO** — [Jimmy Bogard — Vertical Slice Architecture](https://www.youtube.com/watch?v=oAoaMlS1PWo)

---

### Task 2 — Domain Model

Create the domain model for DonationHub containing:

- **Entities:** ApplicationUser, RefreshToken, Category, Campaign, Donation, PaymentAttempt, ProcessedStripeEvent, IdempotencyRecord
- **Enums:** CampaignStatus (Active, Completed, Expired), DonationStatus (Pending, Succeeded, Failed), PaymentAttemptStatus
- **Relationships:** Category → Campaigns, Campaign → Donations, User → Donations, User → RefreshTokens, User → Campaigns (creator), Donation → PaymentAttempts
- **Invariants:** GoalAmount > 0, EndDate in future, donation amount within range, state machine rules
- **Value Objects** where appropriate

**Required output**
- Domain model diagram
- Entity/relationship explanation
- Written list of important invariants
- State machines for Campaign and Donation

### Resources
- **MUST** — [Microsoft — DDD and CQRS architecture guidance](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/)
- **DEEP DIVE** — Eric Evans — `Book: Domain-Driven Design: Tackling Complexity in the Heart of Software`
- **VIDEO** — [Milan Jovanović — Domain-Driven Design with C#](https://www.youtube.com/watch?v=F3_l0Tst0tY)

---

### Task 3 — EF Core Persistence

Configure:
- DbContext and DbSets
- Fluent API entity configurations (separate configuration classes)
- Relationships and foreign keys
- Required fields and constraints
- `RowVersion` concurrency token on Campaign
- Soft delete on Campaign with a global query filter
- Important indexes
- Initial migration
- SQL Server connection

**Required output**
- DbContext with all entity configurations
- Fluent configurations (not data annotations)
- SQL Server connection string from configuration
- Initial migration
- Correct FK/unique constraints
- Justification for important indexes

### Resources
- **MUST** — [Microsoft — EF Core](https://learn.microsoft.com/en-us/ef/core/)
- **MUST** — [EF Core — Relationships](https://learn.microsoft.com/en-us/ef/core/modeling/relationships)
- **MUST** — [EF Core — Indexes](https://learn.microsoft.com/en-us/ef/core/modeling/indexes)
- **MUST** — [EF Core — Migrations](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/)
- **MUST** — [EF Core — Global query filters](https://learn.microsoft.com/en-us/ef/core/querying/filters)
- **VIDEO** — [Milan Jovanović — EF Core Fluent API & Relationships](https://www.youtube.com/watch?v=kYJqD2v5bX8)
- **VIDEO** — [Nick Chapsas — Global Query Filters & Soft Delete](https://www.youtube.com/watch?v=Zc2gZ79r_2M)

---

### Task 4 — Generic Repository & Unit of Work

Create a Generic Repository and Unit of Work for write-side persistence.

- `IGenericRepository<T>` in Application layer
- `GenericRepository<T>` implementation in Infrastructure
- `IUnitOfWork` abstraction
- Read queries (queries) may use `DbContext` directly with `AsNoTracking` and projection to DTOs

**Required output**
- Abstractions in Application layer
- Implementations in Infrastructure
- Written explanation of Repository + UoW vs direct DbContext, and why reads bypass the repository

### Resources
- **MUST** — [Microsoft — EF Core documentation](https://learn.microsoft.com/en-us/ef/core/)
- **DEEP DIVE** — [Microsoft — Common web application architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
- **VIDEO** — [DevCreed (محمد الهلالي) — Repository Pattern & Unit of Work (Arabic)](https://youtube.com/playlist?list=PL62tSREI9C-e6nQ47brLj00iSGddiee73)
- **VIDEO** — [Milan Jovanović — Generic Repository & Unit of Work](https://www.youtube.com/watch?v=xdibesoH8zE)

---

### Task 5 — Options Pattern

Create option classes for every configurable setting, validated with data annotations and `ValidateOnStart()`:

- `JwtOptions` (secret, issuer, audience, access token lifetime, refresh token lifetime)
- `SeedAdminOptions` (email, password)
- `DonationOptions` (min amount, max amount, pending timeout, currency)

The application must **fail at startup** with a clear message when configuration is invalid.

**Required output**
- Option classes bound from configuration
- Data annotation validation
- `ValidateOnStart()` registered
- `appsettings.json` with placeholders (no secrets)

### Resources
- **MUST** — [ASP.NET Core — Options pattern](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
- **MUST** — [ASP.NET Core — Configuration](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)
- **VIDEO** — [Nick Chapsas — Options Pattern & ValidateOnStart](https://www.youtube.com/watch?v=b4O2QkM_S7A)

---

### Task 6 — Identity & Authentication

Implement using ASP.NET Core Identity:

- **IDN-01:** Register with email, password, full name, preferred language → Donor role
- **IDN-02:** Unique email (case-insensitive), password policy (8+ chars, upper, lower, digit)
- **IDN-03:** Login returns access token (JWT) + refresh token
- **IDN-04:** Access token: 15 min (configurable), claims: user id, email, roles
- **IDN-05:** Refresh token: 7 days (configurable), stored **hashed**
- **IDN-06:** Refresh endpoint: rotation (old revoked, new issued, linked)
- **IDN-07:** Logout revokes the supplied refresh token
- **IDN-09:** `GET /auth/me` returns current user's profile
- **IDN-10:** Deactivated user cannot log in or refresh
- **IDN-11:** Initial Admin seeded from options at startup
- **IDN-12:** Login failures return the same generic error (no user enumeration)

| Method | Path | Access |
|--------|------|--------|
| POST | `/auth/register` | Anonymous |
| POST | `/auth/login` | Anonymous |
| POST | `/auth/refresh` | Anonymous |
| POST | `/auth/logout` | Any auth |
| GET | `/auth/me` | Any auth |

### Resources
- **MUST** — [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity)
- **MUST** — [ASP.NET Core authentication overview](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/)
- **MUST** — [Configure JWT bearer authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication)
- **VIDEO** — [Milan Jovanović — JWT Authentication & Refresh Tokens](https://www.youtube.com/watch?v=M6Akbba9hnA)
- **VIDEO** — [Khalid Essaadani — ASP.NET Identity in Arabic](https://www.youtube.com/playlist?list=PLwj1YcMhLRN1T3fIb-JDa4xNFfVQoljGI)

---

### Task 7 — Staff User Creation

Implement:
- **IDN-08:** An Admin creates a staff user (role `Admin` or `CampaignManager`) with a temporary password

| Method | Path | Access |
|--------|------|--------|
| POST | `/admin/staff` | Admin |

### Resources
- **MUST** — [ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity)

---

### Task 8 — Categories Module

Implement:
- **CAT-01:** Admin creates, updates, and deletes categories with Arabic and English name and description
- **CAT-02:** Names required in both languages, max 100 characters, unique per language
- **CAT-03:** Anyone can list categories and get one by id
- **CAT-04:** Deleting a category that has campaigns is rejected with a clear error

| Method | Path | Access |
|--------|------|--------|
| GET | `/categories` | Anonymous |
| GET | `/categories/{id}` | Anonymous |
| POST | `/categories` | Admin |
| PUT | `/categories/{id}` | Admin |
| DELETE | `/categories/{id}` | Admin |

### Resources
- **MUST** — [ASP.NET Core Web API fundamentals](https://learn.microsoft.com/en-us/aspnet/core/web-api/)
- **VIDEO** — [Passionate Coders (Mohamed El-Mehdi) — ASP.NET Core Web API (Arabic)](https://www.youtube.com/playlist?list=PLsV97AQt78NQ8E7cEqovH0zLYRJgJahGh)

---

### Task 9 — Swagger & API Documentation

Configure Swagger/OpenAPI with a Bearer authorization button so JWT-protected endpoints can be tested.

**Required output**
- Documented endpoints
- Request/response schemas
- Bearer authorization configured in Swagger
- Useful endpoint descriptions

### Resources
- **MUST** — [Microsoft — Swagger/OpenAPI in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger)
- **VIDEO** — [Milan Jovanović — Swagger with JWT Bearer Authentication](https://www.youtube.com/watch?v=cM3k0ZqP4f0)

---

### Task 10 — README

Write a README with:
- How to build and run the project
- How to apply migrations
- How to use Swagger
- Architecture decisions

### Resources
- **MUST** — [GitHub — README documentation](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)

---

## Sprint Deliverables

- [ ] Four-layer solution (Domain, Application, Infrastructure, Api)
- [ ] Correct dependency direction
- [ ] Vertical slice organization in Application
- [ ] Domain model with entities, enums, relationships, and invariants
- [ ] EF Core persistence with Fluent API configurations
- [ ] SQL Server connection + initial migration
- [ ] Soft delete with global query filter on Campaign
- [ ] `RowVersion` concurrency token on Campaign
- [ ] Generic Repository + Unit of Work
- [ ] Options Pattern with `ValidateOnStart()`
- [ ] Identity: registration, login, JWT, refresh token (hashed + rotation), logout
- [ ] Admin seeded from options
- [ ] Staff user creation
- [ ] Categories CRUD with invariant enforcement
- [ ] Swagger with Bearer authorization
- [ ] README with run steps

## Definition of Done

- [ ] Solution builds with no errors
- [ ] Architecture is correctly separated
- [ ] Dependency direction is correct
- [ ] Domain has no external dependencies
- [ ] Feature folders used in Application
- [ ] EF Core persistence works
- [ ] Migrations work
- [ ] Options validated at startup
- [ ] Registration works
- [ ] Login returns JWT + refresh token
- [ ] Refresh token rotation works
- [ ] Refresh tokens stored hashed
- [ ] Logout revokes token
- [ ] Deactivated user cannot log in
- [ ] Admin seeded at startup
- [ ] Staff creation works
- [ ] Categories CRUD works
- [ ] Category with campaigns cannot be deleted
- [ ] Swagger configured with Bearer auth
- [ ] No secrets in repository
- [ ] README updated

## Review Rule

The learner should be able to explain not only **what** was implemented, but **why it belongs there, what alternatives existed, and what trade-off was accepted**.