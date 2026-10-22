# Sprint 2 — Campaigns, CQRS, MediatR & Unit Testing

> **Project:** DonationHub — Donation Management & Crowdfunding Platform  
> **Duration:** 15 Days (07/11/2026 to 21/11/2026)  
> **Meeting Schedule:**  
> - **Stand-up 1 (Day 5):** 11/11/2026 — Session Leader: **Asma ELhadidy**  
> - **Stand-up 2 (Day 10):** 16/11/2026 — Session Leader: **Kareem Zein**  
> - **Closing & Deadline (Day 15):** 21/11/2026 — Session Leader: **Mohamed Atef**  
> - **Code Review & Corrections (Days 16–18):** 22/11/2026 to 24/11/2026  
> **Evaluation:** 100 Points  
> **Target Framework:** .NET 10 (fully compatible with .NET 8 / .NET 9)

---

## 🎯 Sprint Goal

Transform DonationHub into a use-case-driven system by migrating all endpoints to the **CQRS** pattern using **MediatR**. Implement the **Campaigns** module with optimistic concurrency control, replace exception-driven business flow with the **Result Pattern**, enforce automatic validation via MediatR Pipeline Behaviors, format all API errors using **RFC 9457 ProblemDetails** with `IExceptionHandler`, and establish a robust test suite of at least **15 Unit Tests** using xUnit, Moq, and FluentAssertions.

```text
HTTP Request
     ↓
Thin Controller (Maps incoming HTTP request to Command or Query)
     ↓
MediatR Send
     ↓
Pipeline Behaviors (ValidationBehavior via FluentValidation)
     ↓
Command / Query Handler
     ↓
Domain Invariants & Concurrency Check (RowVersion)
     ↓
Result<T> (Success or Expected Domain Error from Catalogue)
     ↓
HTTP Response (or ProblemDetails on Failure)
```

---

## 📌 Sprint Prerequisites

Before starting this sprint, learners must solidify their understanding of:

### 1. The CQRS Architectural Mindset & Vertical Slices
*Understand the fundamental difference between Commands (state mutations with side effects) and Queries (pure reads without side effects). Understand why grouping related artifacts (Command + Handler + DTO + Validator) inside a single feature slice is superior to bloated monolithic service classes.*

### 2. Cooperative Cancellation via CancellationToken
*Understand how ASP.NET Core signals client disconnection (`HttpContext.RequestAborted`) and how passing this token down to database calls prevents orphaned background query execution.*

---

## 📋 Engineering Decisions & Pinned Package Versions

To ensure reproducibility, prevent commercial licensing pitfalls, and avoid breaking changes, all teams must adhere to the following pinned versions:

| Package / Tool | Pinned Version | Rationale & Guidance |
|---|---|---|
| **MediatR** | `[12.5.0]` | `<PackageReference Include="MediatR" Version="[12.5.0]" />` — Brackets prevent automatic upgrade to commercial v13+. |
| **FluentAssertions** | `[7.0.0]` | Apache 2.0 license. v8+ is commercial. (Acceptable alternative: Shouldly). |
| **Moq** | `[4.18.4]` | Avoids SponsorLink controversy present in 4.20. (Acceptable alternative: NSubstitute). |
| **RFC 9457 ProblemDetails** | Built-in | `builder.Services.AddProblemDetails();` paired with `IExceptionHandler`. |
| **Time Provider** | `TimeProvider` | Disallow direct `DateTime.UtcNow` in Handlers and Domain to enable deterministic unit testing. |

---

## 🛠️ Sprint Tasks

All tasks below are **CORE** unless marked BONUS.

> 📘 **Written references:** Links marked 📘 point to the free [.NET Web API Zero to Hero](https://codewithmukesh.com/courses/dotnet-webapi-zero-to-hero/) course (.NET 10 articles + source code). Its samples use **PostgreSQL** and **Minimal APIs** — apply the ideas to our **SQL Server** + **thin Controllers** setup, and always keep the pinned versions above (MediatR `[12.5.0]`, FluentAssertions `[7.0.0]`, Moq `[4.18.4]`) even if an article installs a newer one.

### Task 1 — CQRS Migration with MediatR ([12.5.0])

Migrate all previous endpoints (Identity and Categories) and implement upcoming Campaign endpoints using CQRS:
- Rewrite **every Sprint 1 use case** (IDN-01 to IDN-12, CAT-01 to CAT-04) as a command or query, each in its own feature slice (command/query + handler + validator + DTOs).
- Create `IRequest<Result<T>>` definitions for all commands and queries.
- Implement isolated `IRequestHandler<TRequest, TResponse>` classes for each individual use case.
- Keep all controllers extremely thin: their sole responsibility is binding HTTP inputs, dispatching through `_mediator.Send()`, and mapping `Result<T>` to HTTP status codes (`Ok`, `Created` with `Location`, `NoContent`) or to ProblemDetails on failure (see Task 6).
- **Read/write split (SRS 3.3 rule 5):** Commands persist through the Generic Repository and Unit of Work. Queries may read directly from the `DbContext` with `AsNoTracking()` and project to DTOs with `.Select()`. **Explain this choice in your README** (why writes go through Repository/UoW while reads bypass it).

**Resources:**
- **⭐ Required Primary:** [23 Add mediator pattern with CQRS — Ahmed Mohamady [Arabic] (14:05)](https://youtu.be/W7_zEf3eRQA).
- **🔄 Alternative / Deep Dive:**
  - [CQRS & MediatR in ASP.NET Core — Patrick God](https://youtu.be/BewUyKLZjtc)
  - [Clean Architecture & CQRS in Action (.NET 10) — Essam Abdelnabi (26:25 to 43:20)](https://youtu.be/v9fRFyg414k)
  - 📘 [CQRS and MediatR in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/cqrs-and-mediatr-in-aspnet-core/) (⚠️ the article installs MediatR 14.x, which is commercial — install `[12.5.0]` instead; the concepts are the same)
  - 📘 [Tracking vs No-Tracking Queries in EF Core — codewithmukesh](https://codewithmukesh.com/blog/tracking-vs-no-tracking-queries-efcore/) (supports the read/write split)

---

### Task 2 — CancellationToken Propagation & TimeProvider Abstraction

- **CancellationToken:** Accept `CancellationToken` in every controller action and forward it through `_mediator.Send()` into handlers, repositories, and EF Core asynchronous methods (`ToListAsync`, `SaveChangesAsync`).
- **TimeProvider:** Inject `TimeProvider` into domain services and handlers. Replace all instances of `DateTime.UtcNow` with `timeProvider.GetUtcNow()` to allow testing date-dependent logic (such as campaign expiry) deterministically.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Cancellation Token in .NET — Rahul Nath (18:42)](https://youtu.be/39gIPekzpjs) (Forwarding RequestAborted to EF Core at 10:40–18:00).
  - ⭐2 [TimeProvider in .NET 8+ — Nick Chapsas (09:47)](https://youtu.be/xa6YusjCFF0) (DI registration at 04:55, mocking time in unit tests at 05:25).
- **🔄 Alternative / Deep Dive:**
  - [CancellationTokenSource in C# — Study DotNet [Arabic]](https://youtu.be/2xf6n0SpkNo)
  - [FakeTimeProvider in .NET — Nick Chapsas](https://youtu.be/acjCxNlWpJQ)

---

### Task 3 — Optimistic Concurrency Control (RowVersion on Campaign)

Implement SRS requirement **CMP-10**:
- Prevent lost updates during concurrent edits on the same campaign.
- Add `byte[] RowVersion` on `Campaign` and configure it as a concurrency token via Fluent API:  
  `builder.Property(c => c.RowVersion).IsRowVersion();`
- When concurrent edits occur, EF Core throws `DbUpdateConcurrencyException`. Catch this conflict and return `Campaign.ConcurrencyConflict` (HTTP 409 Conflict) from the error catalogue.

**Resources:**
- **⭐ Required Primary:** [Optimistic Concurrency in .NET [Arabic] — Islam Helmy (12:24)](https://youtu.be/VhuFcGP6H0E).
- **🔄 Alternative / Deep Dive:**
  - [Optimistic vs Pessimistic Locking — Milan Jovanović](https://youtu.be/lLDBVMj5He8)
  - 📘 [Concurrency Control & Optimistic Locking in EF Core — codewithmukesh](https://codewithmukesh.com/blog/concurrency-control-optimistic-locking-efcore/) (covers both PostgreSQL `xmin` and SQL Server `IsRowVersion()` — we use `IsRowVersion()`)

---

### Task 4 — Result Pattern & Error Catalogue

Eliminate exception throwing for expected business and validation failures:
- Implement:
  - `Error` record (`Code`, `Message`, `Type`), where `Type` is an `ErrorType` enum.
  - `Result` and `Result<T>` with factory methods (`Success`, `Failure`).
- Map every `ErrorType` to an HTTP status code (SRS §6.1):

| `ErrorType` | HTTP Status |
|---|---|
| `Validation` | 400 Bad Request |
| `Unauthorized` | 401 Unauthorized |
| `Forbidden` | 403 Forbidden |
| `NotFound` | 404 Not Found |
| `Conflict` | 409 Conflict |
| `PayloadTooLarge` | 413 Payload Too Large (used by `File.TooLarge` from Sprint 4) |
| `UnprocessableEntity` | 422 Unprocessable Entity |
| `TooManyRequests` | 429 Too Many Requests |
| `Failure` | 500 Internal Server Error |

- Define the **full** DonationHub Error Catalogue (SRS §6.1) as static errors now. Codes marked with a later sprint are declared now and used when that module arrives:

| Error Code | HTTP Status | Context | Used from |
|---|---|---|---|
| `Validation.Failed` | 400 Bad Request | Input validation failure | Sprint 2 |
| `Auth.InvalidCredentials` | 401 Unauthorized | Wrong email or password (generic, IDN-12) | Sprint 2 (Identity rewrite) |
| `Auth.EmailAlreadyExists` | 409 Conflict | Registration with an existing email | Sprint 2 (Identity rewrite) |
| `Auth.RefreshTokenInvalid` | 401 Unauthorized | Refresh token unknown, expired or revoked | Sprint 2 (Identity rewrite) |
| `Auth.AccountDeactivated` | 403 Forbidden | Deactivated user tries to log in or refresh (IDN-10) | Sprint 2 (Identity rewrite) |
| `Category.NotFound` | 404 Not Found | Specified category does not exist | Sprint 2 |
| `Category.HasCampaigns` | 409 Conflict | Cannot delete category with linked campaigns | Sprint 2 |
| `Campaign.NotFound` | 404 Not Found | Campaign ID not found | Sprint 2 |
| `Campaign.NotActive` | 409 Conflict | Campaign is completed or expired | Sprint 2 |
| `Campaign.GoalBelowRaised` | 422 Unprocessable | Goal amount cannot be lower than raised amount | Sprint 2 |
| `Campaign.EndDateInPast` | 422 Unprocessable | End date must be in the future | Sprint 2 |
| `Campaign.ConcurrencyConflict` | 409 Conflict | Concurrent update conflict detected | Sprint 2 |
| `Donation.NotFound` | 404 Not Found | Donation not found or not owned | Sprint 3 |
| `Donation.AmountOutOfRange` | 422 Unprocessable | Amount outside `DonationOptions` range | Sprint 3 |
| `Donation.InvalidState` | 409 Conflict | Donation already in a final state | Sprint 3 |
| `Common.RateLimited` | 429 Too Many Requests | Rate limit exceeded | Sprint 3 |
| `File.InvalidType` | 400 Bad Request | File signature not allowed | Sprint 4 |
| `File.TooLarge` | 413 Payload Too Large | File exceeds max size | Sprint 4 |
| `Payment.InvalidSignature` | 400 Bad Request | Stripe webhook signature invalid | Sprint 5 |
| `Idempotency.KeyReusedWithDifferentRequest` | 422 Unprocessable | Same `Idempotency-Key` with a different body | Sprint 5 |

**Resources:**
- **⭐ Required Primary:** [Handle Errors in C# using Exception and Result Pattern — Loai Saber [Arabic] (18:08)](https://youtu.be/hfnWW2trOlY).
- **🔄 Alternative / Deep Dive:** [Result Pattern in C# — Milan Jovanović](https://youtu.be/WCCkEe_Hy2Y)

---

### Task 5 — Automatic Validation via MediatR Pipeline Behavior

- Implement `ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>`.
- Collect all `IValidator<TRequest>` registered for the command/query and execute them before reaching the handler.
- If validation fails, short-circuit the pipeline immediately and return `Result.Failure` with code `Validation.Failed` and a dictionary of property-level errors (surfaced as the ProblemDetails `errors` field, see Task 6).
- Write a validator for **every** command and query that has input (SRS §6.2). Handlers never repeat basic input validation; business rules stay in the domain.

**Resources:**
- **⭐ Required Primary:** [Clean Architecture & CQRS in Action — Essam Abdelnabi (Timestamps: Pipeline concept at 05:50, ValidationBehavior at 21:40, Validator at 43:25)](https://youtu.be/v9fRFyg414k).
- **🔄 Alternative / Deep Dive:**
  - [Validation with MediatR and FluentValidation — Milan Jovanović](https://youtu.be/85dxwd8HzEk)
  - 📘 [Validation with MediatR Pipeline Behavior and FluentValidation — codewithmukesh](https://codewithmukesh.com/blog/validation-with-mediatr-pipeline-behavior-and-fluentvalidation/)
  - 📘 [FluentValidation in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/fluentvalidation-in-aspnet-core/)

---

### Task 6 — Global Exception Handling & RFC 9457 ProblemDetails

- Implement `IExceptionHandler` (.NET 8+) to catch unhandled server crashes (500 Internal Server Error).
- Log the exception with full stack trace securely while returning a sanitized RFC 9457 `ProblemDetails` response containing `traceId`, `title`, and generic message to prevent leaking internal infrastructure details.
- **All** errors (expected `Result` failures and unexpected exceptions) are returned as ProblemDetails (`application/problem+json`) with the extra fields required by SRS §6.1:
  - `code` — the catalogue code (e.g. `Campaign.NotFound`).
  - `traceId` — the request correlation id.
  - `errors` — for validation failures only: a field → messages dictionary.
- The HTTP status comes from the `ErrorType` mapping in Task 4. Example:
  ```json
  {
    "type": "https://httpstatuses.io/422",
    "title": "Goal amount cannot be lower than the raised amount.",
    "status": 422,
    "code": "Campaign.GoalBelowRaised",
    "traceId": "00-9f2d…"
  }
  ```

**Resources:**
- **⭐ Required Primary:** [#9 Exception Handling in ASP.NET Core 9 — CodingClass [Arabic] (21:35)](https://youtu.be/Efmvo90NuiA).
- **🔄 Alternative / Deep Dive:**
  - [Global Error Handling in .NET 8 (IExceptionHandler) — Milan Jovanović](https://youtu.be/uOEDM0c9BNI)
  - [Exception Handling in ASP.NET Core — Julio Casal](https://youtu.be/HTCb1zXQ6E4)
  - 📘 [Global Exception Handling in ASP.NET Core (IExceptionHandler + ProblemDetails) — codewithmukesh](https://codewithmukesh.com/blog/global-exception-handling-in-aspnet-core/)

---

### Task 7 — Campaigns Module Implementation

Implement campaign features according to SRS §5.3:
- **CMP-01:** Create Campaign `POST /campaigns` (Bilingual title & description, `GoalAmount`, `EndDate`, `CategoryId`). Initial status is `Active` and `RaisedAmount = 0`. Returns `201 Created` with a `Location` header. Unknown category → `Category.NotFound`; `GoalAmount <= 0` → `Validation.Failed`; past end date → `Campaign.EndDateInPast`.
- **CMP-02:** Update Campaign `PUT /campaigns/{id}` (titles, descriptions, category, end date, goal) enforcing domain invariants 2, 7 and 8: end date must be in the future (`Campaign.EndDateInPast`), goal cannot be lowered below `RaisedAmount` (`Campaign.GoalBelowRaised`), and a campaign in a final state (`Completed`, `Expired`) cannot be changed (`Campaign.NotActive`). The client sends the last `RowVersion` it received (see CMP-10).
- **CMP-03:** Soft-delete Campaign `DELETE /campaigns/{id}` (`204 No Content`). Soft-deleted campaigns never appear in any query (global query filter from Sprint 1). A campaign with donations is never hard-deleted (invariant 9).
- **CMP-04:** Public list of active campaigns `GET /campaigns`, and public success stories `GET /campaigns/success-stories` (status `Completed`). (Pagination, filtering and search come in Sprint 3 with CMP-08.)
- **CMP-05:** Campaign details `GET /campaigns/{id}` including all fields plus computed fields: `progressPercent` (min(100, (Raised/Goal)*100)) and `daysLeft` (computed from `TimeProvider`, never negative). Unknown id → `Campaign.NotFound`.
- **CMP-06:** In Sprint 2, campaign creation and management is restricted to `Admin` (CampaignManager authorization comes in Sprint 3).
- **CMP-10:** Concurrent edits are detected with `RowVersion` (Task 3); the second writer receives `Campaign.ConcurrencyConflict` (409).
- **CMP-11:** Public endpoints (`GET /campaigns`, `GET /campaigns/success-stories`, `GET /campaigns/{id}`) return the title and description in the language of the `Accept-Language` header (`en` or `ar`), with fallback `en` for any other or missing value. Admin endpoints return **both** languages (`TitleAr`, `TitleEn`, `DescriptionAr`, `DescriptionEn`). Resource-file localization (`IStringLocalizer`, `.resx`) for messages arrives in Sprint 3, but selecting the data fields by language is required now.
- Model the campaign as a rich entity: invariants live in entity methods (e.g. `Campaign.Create(...)`, `Campaign.Update(...)`), not in handlers. Implement the campaign-related invariants from SRS §4.5 (1, 2, 6, 7, 8, 9).
- Enums are returned as strings in JSON (e.g. `"status": "Active"`).

| Method | Path | Access |
|---|---|---|
| `GET` | `/campaigns` | Anonymous |
| `GET` | `/campaigns/success-stories` | Anonymous |
| `GET` | `/campaigns/{id}` | Anonymous |
| `POST` | `/campaigns` | Admin |
| `PUT` | `/campaigns/{id}` | Admin |
| `DELETE` | `/campaigns/{id}` | Admin |

---

### Task 8 — Unit Testing (xUnit, Moq [4.18.4] & FluentAssertions [7.0.0])

Create a unit test project (`DonationHub.UnitTests`, or split into `DonationHub.Domain.UnitTests` and `DonationHub.Application.UnitTests`). Write **at least 15 comprehensive unit tests** following the **Arrange-Act-Assert (AAA)** pattern. Use Moq `[4.18.4]` to fake repositories, the Unit of Work and `TimeProvider` (or `FakeTimeProvider`), and FluentAssertions `[7.0.0]` for readable assertions. Do not test getters or EF Core itself. Suggested coverage:
1. Domain invariant: Goal amount must be greater than zero.
2. Domain invariant: End date cannot be in the past (verified using a faked `TimeProvider`).
3. Domain invariant: Goal cannot be updated below raised amount (`Campaign.GoalBelowRaised`).
4. Domain invariant: A campaign in a final state (`Completed` / `Expired`) cannot be updated.
5. Calculation: `progressPercent` capped at 100%.
6. Calculation: `daysLeft` computed accurately relative to `TimeProvider` and never negative.
7. State transition: Campaign automatically transitions to `Completed` when goal is reached.
8. Category rule: Deleting a category that has campaigns returns `Category.HasCampaigns`.
9. Validator: `CreateCampaignCommandValidator` rejects missing bilingual titles, `GoalAmount <= 0` and an empty `CategoryId`.
10. Validator: register command rejects a password that breaks the policy (8+ chars, upper, lower, digit).
11. Handler: `CreateCampaignCommandHandler` returns `Category.NotFound` when the category does not exist (repository mocked) and never calls `SaveChangesAsync`.
12. Handler: `CreateCampaignCommandHandler` adds the campaign and calls `IUnitOfWork.SaveChangesAsync` once on success.
13. Handler: `UpdateCampaignCommandHandler` returns `Campaign.NotFound` for an unknown id.
14. Handler: `UpdateCampaignCommandHandler` maps a `DbUpdateConcurrencyException` to `Campaign.ConcurrencyConflict`.
15. Pipeline: `ValidationBehavior` short-circuits with `Validation.Failed` and never invokes the handler when a validator fails.
16. Handler: login returns the same `Auth.InvalidCredentials` error for an unknown email and a wrong password (IDN-12).

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [#003 Unit Testing Your Code — Essam Abdelnabi [Arabic] (26:10)](https://youtu.be/cAyROJ84tBs).
  - ⭐2 [#004 Mocking Dependencies (Moq) — Essam Abdelnabi [Arabic] (26:11)](https://youtu.be/JaIeeGqsO9M).
  - ⭐3 [2. Fluent Assertions in xUnit — Coding-Future (38:38)](https://youtu.be/rr9z92lm2no).
- **🔄 Alternative / Deep Dive:**
  - [NSubstitute Mocking Framework — Nick Chapsas](https://youtu.be/LcQYv0cBWk0)
  - [FluentAssertions v7 vs Shouldly — Nick Chapsas](https://youtu.be/ZFc6jcaM6Ms)
  - [.NET Unit Testing Tutorial — Julio Casal](https://youtu.be/dsD0CMgPjUk)

---

## 📦 Sprint Deliverables

- [ ] All use cases refactored to CQRS with MediatR `[12.5.0]`.
- [ ] Automatic validation pipeline via `ValidationBehavior` and FluentValidation.
- [ ] Result Pattern implemented with zero business exceptions thrown.
- [ ] Centralized exception handling via `IExceptionHandler` returning RFC 9457 ProblemDetails.
- [ ] Campaigns module fully implemented according to SRS specifications.
- [ ] `CancellationToken` forwarded across all async boundaries.
- [ ] `TimeProvider` injected and used for all date calculations.
- [ ] Optimistic concurrency control working on `Campaign` via `RowVersion`.
- [ ] At least 15 passing unit tests covering domain rules, validators, and handlers.
- [ ] Full error catalogue (20 codes) and `ErrorType` → HTTP mapping implemented.
- [ ] Public campaign responses localized by `Accept-Language` (`en` / `ar`, fallback `en`); admin responses bilingual.
- [ ] README updated: what was added, how to try it, and the read/write split decision (Repository/UoW for writes, `DbContext` + `AsNoTracking` for reads).

---

## ✅ Definition of Done

- [ ] Solution builds cleanly and all unit tests pass (`dotnet test`).
- [ ] Controllers contain zero business logic or validation code.
- [ ] No domain entities are exposed through API endpoints (DTOs only).
- [ ] Concurrency conflict triggers HTTP 409 with ProblemDetails error (`code: Campaign.ConcurrencyConflict`).
- [ ] Validation failures return HTTP 400 with `code: Validation.Failed` and field-level `errors`.
- [ ] Every error response is ProblemDetails containing `code` and `traceId`; unexpected exceptions return a generic 500 without internal details.
- [ ] No `DateTime.UtcNow` in domain or handlers (`TimeProvider` only).
- [ ] `GET /campaigns/{id}` with `Accept-Language: ar` returns Arabic title/description; any other value falls back to English.
- [ ] Every CORE requirement of the sprint (CMP-01 to CMP-06, CMP-10, CMP-11, and the Sprint 1 use cases rewritten as CQRS) works through Scalar or Postman.
- [ ] New behavior is covered by unit tests.
- [ ] The `sprint-2` tag exists and points to the final commit.
- [ ] README updated (what was added, how to try it, decisions — including the read/write split).
- [ ] No secrets in the repository history (keys and passwords in user-secrets or environment variables only).
- [ ] Learner can explain CQRS, MediatR pipeline behaviors, and Result Pattern trade-offs during review.

---

## ⭐ BONUS (optional — counts only after all CORE items pass)

- **CMP-12:** Campaign `Cancelled` status, set by Admin (or the owner from Sprint 3), with its own domain rules and tests.
- **CMP-13:** Specification pattern for campaign list filters.
- **Domain Events (SRS §3.2):** In-process notifications (e.g. `CampaignCompletedEvent`) instead of direct calls.
- **IDN-13 / IDN-14 (carried over from Sprint 1):** Refresh token reuse detection; change password and update profile endpoints, implemented as CQRS slices.

---

## 📤 Submission

- Submit your repository URL on **tawjeh.net** before **23:59 on 21/11/2026** (Day 15).
- Tag the final commit and push the tag:
  ```bash
  git tag sprint-2
  git push origin sprint-2
  ```
- Fixes are allowed during the review window **22/11/2026 to 24/11/2026** (Days 16–18). Do not move or force-push `sprint-2`; tag the fixed commit `sprint-2-fix` and push it (`git tag sprint-2-fix && git push origin sprint-2-fix`).

---

## 🧮 Grading (100 Points)

| Part | Points | How |
|---|---|---|
| Automated | 40 | Build succeeds, `dotnet test` passes, the `sprint-2` tag exists, CORE endpoints respond as specified (smoke check by the mentor) |
| Checklist | 40 | The Definition of Done items above |
| Explanation | 20 | One design-decision question (e.g. why reads bypass the repository, why Result instead of exceptions), or review of the decisions written in your README |

BONUS items add capped extra credit on top of the 100 points, only when all CORE items pass.

---

## 💡 Post-Sprint Enrichment

- [#001 Intro to Unit Testing in C# — Essam Abdelnabi [Arabic]](https://youtu.be/HzjMcd1jEio)
- [In-Memory DbContext Testing Pitfalls — Nick Chapsas](https://youtu.be/VM6PIKkEg4s)
- [Code Coverage Metrics in .NET — Essam Abdelnabi](https://youtu.be/W5w-5EK94wc)
- [TDD with xUnit — Mohamad Lawand](https://youtu.be/7YghGdGE7d0)
- [MediatR Commercial Licensing Context — Nick Chapsas](https://youtu.be/bNuQ9xOG9eQ)
- [CQRS Architectural Trade-offs — Mohamad Lawand](https://youtu.be/sUjNZAYTZwI)
- [FluentValidation Deep Dive — Nick Chapsas](https://youtu.be/Ut6mRRFT2vM)
- 📘 [CQRS Without MediatR (Custom Dispatcher) — codewithmukesh](https://codewithmukesh.com/blog/cqrs-without-mediatr/) (useful context for the MediatR licensing decision)
- 📘 [Filters in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/filters-in-aspnet-core/)