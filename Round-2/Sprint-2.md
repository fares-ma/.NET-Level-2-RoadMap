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

### Task 1 — CQRS Migration with MediatR ([12.5.0])

Migrate all previous endpoints (Identity and Categories) and implement upcoming Campaign endpoints using CQRS:
- Create `IRequest<Result<T>>` definitions for all commands and queries.
- Implement isolated `IRequestHandler<TRequest, TResponse>` classes for each individual use case.
- Keep all controllers extremely thin: their sole responsibility is binding HTTP inputs, dispatching through `_mediator.Send()`, and mapping `Result<T>` to HTTP status codes (`Ok`, `Created`, `NotFound`, `Conflict`, `BadRequest`).

**Resources:**
- **⭐ Required Primary:** [23 Add mediator pattern with CQRS — Ahmed Mohamady [Arabic] (14:05)](https://youtu.be/W7_zEf3eRQA).
- **🔄 Alternative / Deep Dive:**
  - [CQRS & MediatR in ASP.NET Core — Patrick God](https://youtu.be/BewUyKLZjtc)
  - [Clean Architecture & CQRS in Action (.NET 10) — Essam Abdelnabi (26:25 to 43:20)](https://youtu.be/v9fRFyg414k)

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
- **🔄 Alternative / Deep Dive:** [Optimistic vs Pessimistic Locking — Milan Jovanović](https://youtu.be/lLDBVMj5He8)

---

### Task 4 — Result Pattern & Error Catalogue

Eliminate exception throwing for expected business and validation failures:
- Implement:
  - `Error` record (`Code`, `Description`, `ErrorType`).
  - `Result` and `Result<T>` with factory methods (`Success`, `Failure`).
- Align all errors with the DonationHub Error Catalogue (SRS §6.1):

| Error Code | HTTP Status | Context |
|---|---|---|
| `Validation.Failed` | 400 Bad Request | Input validation failure |
| `Auth.InvalidCredentials` | 401 Unauthorized | Incorrect login credentials |
| `Category.NotFound` | 404 Not Found | Specified category does not exist |
| `Category.HasCampaigns` | 409 Conflict | Cannot delete category with linked campaigns |
| `Campaign.NotFound` | 404 Not Found | Campaign ID not found |
| `Campaign.NotActive` | 409 Conflict | Campaign is completed or expired |
| `Campaign.GoalBelowRaised` | 422 Unprocessable | Goal amount cannot be lower than raised amount |
| `Campaign.EndDateInPast` | 422 Unprocessable | End date must be in the future |
| `Campaign.ConcurrencyConflict` | 409 Conflict | Concurrent update conflict detected |

**Resources:**
- **⭐ Required Primary:** [Handle Errors in C# using Exception and Result Pattern — Loai Saber [Arabic] (18:08)](https://youtu.be/hfnWW2trOlY).
- **🔄 Alternative / Deep Dive:** [Result Pattern in C# — Milan Jovanović](https://youtu.be/WCCkEe_Hy2Y)

---

### Task 5 — Automatic Validation via MediatR Pipeline Behavior

- Implement `ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>`.
- Collect all `IValidator<TRequest>` registered for the command/query and execute them before reaching the handler.
- If validation fails, short-circuit the pipeline immediately and return `Result.Failure` with code `Validation.Failed` and a dictionary of property-level errors.

**Resources:**
- **⭐ Required Primary:** [Clean Architecture & CQRS in Action — Essam Abdelnabi (Timestamps: Pipeline concept at 05:50, ValidationBehavior at 21:40, Validator at 43:25)](https://youtu.be/v9fRFyg414k).
- **🔄 Alternative / Deep Dive:** [Validation with MediatR and FluentValidation — Milan Jovanović](https://youtu.be/85dxwd8HzEk)

---

### Task 6 — Global Exception Handling & RFC 9457 ProblemDetails

- Implement `IExceptionHandler` (.NET 8+) to catch unhandled server crashes (500 Internal Server Error).
- Log the exception with full stack trace securely while returning a sanitized RFC 9457 `ProblemDetails` response containing `traceId`, `title`, and generic message to prevent leaking internal infrastructure details.

**Resources:**
- **⭐ Required Primary:** [#9 Exception Handling in ASP.NET Core 9 — CodingClass [Arabic] (21:35)](https://youtu.be/Efmvo90NuiA).
- **🔄 Alternative / Deep Dive:**
  - [Global Error Handling in .NET 8 (IExceptionHandler) — Milan Jovanović](https://youtu.be/uOEDM0c9BNI)
  - [Exception Handling in ASP.NET Core — Julio Casal](https://youtu.be/HTCb1zXQ6E4)

---

### Task 7 — Campaigns Module Implementation

Implement campaign features according to SRS §3.3:
- **CMP-01:** Create Campaign `POST /campaigns` (Bilingual title & description, `GoalAmount`, `EndDate`, `CategoryId`). Initial status is `Active` and `RaisedAmount = 0`.
- **CMP-02:** Update Campaign `PUT /campaigns/{id}` enforcing invariant checks: cannot reduce goal below raised amount, end date must be future.
- **CMP-03:** Soft-delete Campaign `DELETE /campaigns/{id}`.
- **CMP-04:** Public list of active campaigns `GET /campaigns`, and public success stories `GET /campaigns/success-stories` (status `Completed`).
- **CMP-05:** Campaign details `GET /campaigns/{id}` including computed fields: `progressPercent` (min(100, (Raised/Goal)*100)) and `daysLeft`.
- **CMP-06:** In Sprint 2, campaign creation and management is restricted to `Admin` (CampaignManager authorization comes in Sprint 3).
- **CMP-11:** Return bilingual fields in response DTOs.

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

Create `DonationHub.Application.UnitTests` and `DonationHub.Domain.UnitTests`. Write **at least 15 comprehensive unit tests** following the **Arrange-Act-Assert (AAA)** pattern:
1. Domain invariant: Goal amount must be greater than zero.
2. Domain invariant: End date cannot be in the past (verified using mocked `TimeProvider`).
3. Domain invariant: Goal cannot be updated below raised amount.
4. Calculation: `progressPercent` capped at 100%.
5. Calculation: `daysLeft` computed accurately relative to `TimeProvider`.
6. State transition: Campaign automatically transitions to `Completed` when goal is reached.
7. Category rule: Deleting a category with active campaigns returns conflict.
8. FluentValidation tests for invalid command inputs.
9. Handler unit tests mocking repositories via Moq `[4.18.4]`.
10. FluentAssertions `[7.0.0]` assertion validation for readable test failures.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [#003 Unit Testing Your Code — Essam Abdelnabi [Arabic] (26:10)](https://youtu.be/cAyROJ84tBs).
  - ⭐2 [#004 Mocking Dependencies (Moq) — Essam Abdelnabi [Arabic] (26:11)](https://youtu.be/JaIeeGqsO9M).
  - ⭐3 [2. Fluent Assertions in xUnit — Coding-Future (38:38)](https://youtu.be/rr9z92lm2no).
- **🔄 Alternative / Deep Dive:**
  - [NSubstitute Mocking Framework — Nick Chapsas](https://youtu.be/LcQYv0cBWk0)
  - [FluentAssertions v7 vs Shouldly — Nick Chapsas](https://youtu.be/ZFc6jcaM6Ms)

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

---

## ✅ Definition of Done

- [ ] Solution builds cleanly and all unit tests pass (`dotnet test`).
- [ ] Controllers contain zero business logic or validation code.
- [ ] No domain entities are exposed through API endpoints (DTOs only).
- [ ] Concurrency conflict triggers HTTP 409 with ProblemDetails error.
- [ ] Validation failures return HTTP 400 with detailed field-level errors.
- [ ] Learner can explain CQRS, MediatR pipeline behaviors, and Result Pattern trade-offs during review.

---

## 💡 Post-Sprint Enrichment

- [In-Memory DbContext Testing Pitfalls — Nick Chapsas](https://youtu.be/VM6PIKkEg4s)
- [Code Coverage Metrics in .NET — Essam Abdelnabi](https://youtu.be/W5w-5EK94wc)
- [TDD with xUnit — Mohamad Lawand](https://youtu.be/7YghGdGE7d0)
- [MediatR Commercial Licensing Context — Nick Chapsas](https://youtu.be/bNuQ9xOG9eQ)
- [CQRS Architectural Trade-offs — Mohamad Lawand](https://youtu.be/sUjNZAYTZwI)