# Sprint 2 — Campaigns, CQRS, MediatR & Unit Testing

> Project: **DonationHub — Donation Management Platform**
> Duration: **2 weeks (~15 hours/week)**
> Evaluation: **100 points**

## Sprint Goal

Transform DonationHub from a basic API into a **use-case-oriented application** by introducing CQRS with MediatR, and implement the Campaigns module with proper validation, error handling, and unit tests.

The learner should understand:

```text
HTTP Request
    ↓
Controller (thin)
    ↓
MediatR
    ↓
Command / Query
    ↓
Handler
    ↓
Result (success or expected failure)
    ↓
Domain / Infrastructure
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

### Task 1 — CQRS Conversion

Convert **all existing Sprint 1 use cases** (Identity and Categories) to the CQRS pattern:

```text
Command   (write operations)
Query     (read operations)
Handler   (business logic)
DTO       (response models)
Validator (input validation)
```

Each use case becomes one **slice** (a folder with its command/query, handler, validator, and DTOs).

Keep CQRS as an organizational pattern. Do not introduce distributed CQRS complexity.

### Resources
- **MUST** — [Microsoft — DDD/CQRS architecture guidance](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/)
- **DEEP DIVE** — [Martin Fowler — CQRS](https://martinfowler.com/bliki/CQRS.html)
- **VIDEO** — [Nick Chapsas — CQRS in .NET](https://www.youtube.com/watch?v=2JzQuIvxIqk)
- **ARTICLE** — [CodeWithMukesh — CQRS and MediatR in ASP.NET Core](https://codewithmukesh.com/blog/cqrs-and-mediatr-in-aspnet-core/)

---

### Task 2 — MediatR

Introduce MediatR for all application use cases.

Required:
- `IRequest<T>` / `IRequestHandler<TRequest, TResponse>` for every use case
- Clear dependency flow (controllers → MediatR → handlers)
- Pipeline behaviors registered in DI

Controllers become **thin**: map HTTP to a command/query, send through MediatR, map the `Result` to an HTTP response. No business logic in controllers.

### Resources
- **MUST** — [MediatR — GitHub](https://github.com/jbogard/MediatR)
- **DEEP DIVE** — [Martin Fowler — Mediator](https://martinfowler.com/eaaDev/Mediator.html)
- **VIDEO** — [Derek Comartin (CodeOpinion) — Why Use MediatR?](https://codeopinion.com/why-use-mediatr-3-reasons-why-and-1-reason-not/)
- **VIDEO** — [Nick Chapsas — MediatR Pipeline Behaviors](https://www.youtube.com/watch?v=2JzQuIvxIqk)

---

### Task 3 — Validation Pipeline

Implement automatic validation through a MediatR pipeline behavior using FluentValidation.

- Create validators for every command and query with input
- Validation runs **automatically** in the pipeline behavior — handlers never repeat basic validation
- Separate **input validation** (FluentValidation) from **business rules** (domain/handler)

### Resources
- **MUST** — [FluentValidation documentation](https://docs.fluentvalidation.net/)
- **MUST** — [MediatR — GitHub](https://github.com/jbogard/MediatR)
- **VIDEO** — [Milan Jovanović — Validation with FluentValidation & MediatR](https://www.youtube.com/watch?v=rQ46aI0y9z0)

---

### Task 4 — Result Pattern

Introduce a consistent application result model:

- Success results
- Validation failures (400)
- Business failures (NotFound 404, Conflict 409, UnprocessableEntity 422)
- Error codes from the error catalogue (see SRS §6.1)

Expected failures are returned as a `Result` with an `Error` (`Code`, `Message`, `Type`). **Do not throw exceptions** for expected failures.

Minimum error codes for this sprint:
| Code | HTTP |
|------|------|
| `Validation.Failed` | 400 |
| `Auth.InvalidCredentials` | 401 |
| `Auth.EmailAlreadyExists` | 409 |
| `Auth.RefreshTokenInvalid` | 401 |
| `Auth.AccountDeactivated` | 403 |
| `Category.NotFound` | 404 |
| `Category.HasCampaigns` | 409 |
| `Campaign.NotFound` | 404 |
| `Campaign.NotActive` | 409 |
| `Campaign.GoalBelowRaised` | 422 |
| `Campaign.EndDateInPast` | 422 |
| `Campaign.ConcurrencyConflict` | 409 |

### Resources
- **MUST** — [ASP.NET Core — Handle errors in APIs](https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors)
- **DEEP DIVE** — [Vladimir Khorikov — Functional C# Error Handling](https://enterprisecraftsmanship.com/)
- **VIDEO** — [Milan Jovanović — Functional Error Handling with Result Pattern](https://www.youtube.com/watch?v=WCCkyeekmDo)
- **VIDEO** — [Amichai Mantinband — Clean Architecture Error Handling](https://www.youtube.com/watch?v=4n_sW4hH0Zk)

---

### Task 5 — Global Exception Handling & ProblemDetails

Implement:

- One global handler (`IExceptionHandler`) that catches unexpected exceptions
- Log the exception, return a generic 500 **without leaking details**
- All errors returned as **ProblemDetails** (RFC 9457) with extra fields: `code`, `traceId`, and for validation: `errors` (field → messages)

### Resources
- **MUST** — [ASP.NET Core — Handle errors](https://learn.microsoft.com/en-us/aspnet/core/web-api/handle-errors)
- **VIDEO** — [Nick Chapsas — New Exception Handling in .NET 8 (IExceptionHandler)](https://www.youtube.com/watch?v=4V_X7b9dFMo)
- **VIDEO** — [Milan Jovanović — Global Error Handling in ASP.NET Core](https://www.youtube.com/watch?v=x_u09X_X74s)

---

### Task 6 — Campaigns Module

Implement the Campaigns feature using CQRS + MediatR:

- **CMP-01:** Create campaign (Arabic/English title and description, goal amount, end date, category). Status starts `Active`, `RaisedAmount` starts 0.
- **CMP-02:** Update campaign (titles, descriptions, category, end date, goal), subject to invariants 2, 7, 8.
- **CMP-03:** Soft-delete a campaign. Soft-deleted campaigns never appear in any query (global query filter).
- **CMP-04:** Public list returns active campaigns. Public "success stories" returns completed campaigns.
- **CMP-05:** Campaign details return all fields plus `progressPercent` (raised/goal, capped at 100) and `daysLeft`.
- **CMP-06:** In Sprint 2, only Admin can create and manage campaigns.
- **CMP-10:** Concurrent edits detected using `RowVersion`. Second writer receives a conflict error.
- **CMP-11:** Public responses use both Arabic and English fields (localization comes in Sprint 3).

| Method | Path | Access |
|--------|------|--------|
| GET | `/campaigns` | Anonymous |
| GET | `/campaigns/success-stories` | Anonymous |
| GET | `/campaigns/{id}` | Anonymous |
| POST | `/campaigns` | Admin |
| PUT | `/campaigns/{id}` | Admin |
| DELETE | `/campaigns/{id}` | Admin |

### Resources
- **MUST** — [EF Core — Change tracking](https://learn.microsoft.com/en-us/ef/core/change-tracking/)
- **MUST** — [EF Core — Concurrency](https://learn.microsoft.com/en-us/ef/core/saving/concurrency)
- **VIDEO** — [Milan Jovanović — Optimistic Concurrency in EF Core](https://www.youtube.com/watch?v=bF9EwE0_2H4)
- **VIDEO** — [Nick Chapsas — Concurrency Conflicts in EF Core](https://www.youtube.com/watch?v=pM9mYcEw28Q)

---

### Task 7 — CancellationToken Propagation

Propagate `CancellationToken` through the entire pipeline:

```text
HTTP Request
 ↓
Controller
 ↓
MediatR Handler
 ↓
Repository / EF Core
```

All async operations must accept and pass `CancellationToken`.

### Resources
- **MUST** — [Microsoft — Cancellation in managed threads](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads)
- **MUST** — [EF Core async APIs](https://learn.microsoft.com/en-us/ef/core/querying/async)
- **VIDEO** — [Nick Chapsas — How to Cancel Tasks with CancellationToken in .NET](https://www.youtube.com/watch?v=8V-w6Gj9w-A)

---

### Task 8 — TimeProvider Abstraction

Inject `TimeProvider` instead of using `DateTime.UtcNow` directly in domain or handlers, so tests are deterministic.

### Resources
- **MUST** — [Microsoft — TimeProvider](https://learn.microsoft.com/en-us/dotnet/api/system.timeprovider)
- **VIDEO** — [Nick Chapsas — Stop Using DateTime.UtcNow in .NET 8 (TimeProvider)](https://www.youtube.com/watch?v=S8L3kK_s7hA)

---

### Task 9 — Unit Tests

Write **at least 15 unit tests** covering:

- Campaign domain invariants (GoalAmount > 0, EndDate in future, goal cannot go below raised, state transitions)
- Category deletion rule (cannot delete if has campaigns)
- Handlers (with repository and clock faked)
- Validators

Do not chase arbitrary coverage percentages. Focus on behavior and edge cases.

### Resources
- **MUST** — [xUnit](https://xunit.net/)
- **MUST** — [Microsoft — Testing in .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- **DEEP DIVE** — [Vladimir Khorikov — Unit Testing Principles, Practices, and Patterns](Book)
- **VIDEO** — [Teddy Smith — xUnit Full Course for Beginners](https://www.youtube.com/watch?v=fVVNOkbLcPw)
- **VIDEO** — [Mohamed El-Mehdi — Unit Testing in C# (Arabic)](https://www.youtube.com/watch?v=lY4Q8fZYSN8)
- **VIDEO** — [Nick Chapsas — Effective Testing Workshop](https://www.youtube.com/watch?v=kJFB8o78FBs)

---

## Sprint Deliverables

- [ ] All Sprint 1 use cases rewritten as commands and queries
- [ ] MediatR integrated with pipeline behaviors
- [ ] FluentValidation pipeline behavior
- [ ] Result Pattern with error catalogue
- [ ] Global exception handler with ProblemDetails
- [ ] Campaigns module (CMP-01 to CMP-06, CMP-10, CMP-11)
- [ ] CancellationToken propagated
- [ ] TimeProvider abstraction (no `DateTime.UtcNow` in domain/handlers)
- [ ] At least 15 unit tests
- [ ] Request-flow explanation (Controller → MediatR → Handler → Domain → Infrastructure)

## Definition of Done

- [ ] Every use case uses CQRS (command or query + handler)
- [ ] MediatR correctly integrated
- [ ] Validation pipeline works automatically
- [ ] Result Pattern is consistent across all use cases
- [ ] ProblemDetails returned for all errors
- [ ] Global exception handler catches unexpected errors
- [ ] Campaign creation works
- [ ] Campaign update works with invariant checks
- [ ] Campaign soft-delete works
- [ ] Public campaign list and success stories work
- [ ] Campaign details include progressPercent and daysLeft
- [ ] Concurrency conflict detected on Campaign
- [ ] CancellationToken propagated
- [ ] No `DateTime.UtcNow` in domain or handlers
- [ ] At least 15 unit tests pass
- [ ] Controllers are thin
- [ ] No entities returned from API (DTOs only)
- [ ] Learner can explain CQRS and Mediator decisions

## Review Rule

The learner should be able to explain not only **what** was implemented, but **why it belongs there, what alternatives existed, and what trade-off was accepted**.