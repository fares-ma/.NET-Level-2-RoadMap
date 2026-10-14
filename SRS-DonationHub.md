# DonationHub: Software Requirements Specification

| | |
|---|---|
| **Program** | .NET Level 2, Round 2 (tawjeh.net) |
| **Duration** | 20/10/2026 to 20/1/2027 |
| **Document version** | 0.9 (draft for team review) |
| **Audience** | Students (this file). Mentors also have an internal notes file. |
| **System type** | REST API only (no front end) |

> **How to read this document.** Every requirement has an ID and a priority.
> **[CORE]** means mandatory: you must deliver it to pass the sprint.
> **[BONUS]** means optional: it earns extra credit only after all CORE items of the sprint are done and working.
> Never start a BONUS item while a CORE item is missing.

---

## 1. Introduction

### 1.1 Purpose
DonationHub is a back-end platform where donors give money to charity campaigns. Admins and campaign managers create and manage campaigns. Donors browse campaigns, donate through Stripe, receive a receipt, and see their donation history. Admins see dashboards and manage users.

You will build this system step by step over six sprints. Each sprint adds one or two modules and improves what you already built using the sprint's main topics.

### 1.2 Scope
In scope: the REST API, the database, background jobs, email, file handling, Stripe payments, containerization, automated tests.
Out of scope: any web or mobile front end, an external authentication server (no OpenIddict), real money (Stripe **test mode** only), cloud deployment (optional, never graded), multi-currency, subscriptions or recurring donations.

### 1.3 Glossary
| Term | Meaning |
|---|---|
| Campaign | A fundraising goal with a title, description, goal amount, end date and category |
| Donation | One contribution by one donor to one campaign |
| Raised amount | The sum of **Succeeded** donations of a campaign |
| Success story | A campaign whose status is Completed |
| Donor | A registered user who donates |
| Campaign Manager | A staff user who manages only the campaigns they created |
| Checkpoint | A Git tag (`sprint-1` ... `sprint-6`, `final`) marking your submission for a sprint |
| Slice | One feature folder containing everything for one use case (command or query, handler, validator, DTOs) |

### 1.4 Overall rules
1. Submission is a **GitHub repository link** with one tag per sprint.
2. One repository for the whole program. Every sprint builds on the previous one.
3. The code must build and run with the commands in your README.
4. No secrets in the repository. Keys and passwords live in user-secrets or environment variables.
5. Work alone. You may discuss ideas, but copied code is not accepted.

---

## 2. Product Overview

### 2.1 Roles
| Role | Can do |
|---|---|
| **Anonymous** | Browse active campaigns, view campaign details, view success stories, view categories, register, log in |
| **Donor** | Everything Anonymous can, plus donate, view own donations, download own receipts, manage own profile |
| **Campaign Manager** | Everything Donor can, plus create campaigns and edit, close or soft-delete **only their own** campaigns |
| **Admin** | Everything, including categories, all campaigns, all donations, all users, dashboards, staff management |

### 2.2 System context
```
Client (Swagger / Postman)
        |
   DonationHub API  ---- SQL Server   (data + Hangfire storage)
        |        \----- Redis         (distributed cache)
        |         \---- SMTP server   (MailKit; Mailpit in Docker for development)
        |          \--- Stripe        (Checkout + webhooks, test mode)
        \-- File storage (local disk, Docker volume)
```

### 2.3 Assumptions and constraints
- One currency for the whole system, configured through options (default `USD`).
- The amounts are `decimal(18,2)`. Stripe works in minor units (cents), so convert at the boundary only.
- All timestamps are stored in UTC and returned in ISO 8601.
- All entity ids are `Guid`.
- Arabic and English are the two supported languages. The default is English.
- The database is SQL Server, accessed with EF Core code-first migrations.

---

## 3. Technical Requirements

### 3.1 Mandatory technology list [CORE]
| # | Technology / pattern | Where it is used | Sprint |
|---|---|---|---|
| 1 | Clean Architecture | Whole solution structure | 1 |
| 2 | Vertical Slice (feature folders) | Application layer organization | 1 |
| 3 | EF Core + SQL Server | Persistence, migrations, soft delete, concurrency token | 1 |
| 4 | Generic Repository + Unit of Work | Write-side persistence | 1 |
| 5 | Options Pattern | Every configurable setting, validated on startup | 1 |
| 6 | ASP.NET Core Identity, JWT, Refresh Token | Authentication | 1 |
| 7 | CQRS with MediatR | Every use case | 2 |
| 8 | FluentValidation | Request validation as a pipeline behavior | 2 |
| 9 | Result Pattern | Expected failures without exceptions | 2 |
| 10 | Global exception handling + ProblemDetails | Unexpected failures and uniform errors | 2 |
| 11 | Unit testing (xUnit) | Domain and handlers | 2 |
| 12 | Authorization (roles, policies, resource ownership) | Campaign Manager rules | 3 |
| 13 | Pagination, filtering, sorting, searching | Public and admin lists | 3 |
| 14 | Caching (memory, then Redis) | Public reads and dashboard | 3 |
| 15 | Localization (ar / en) | Responses, validation and error messages, emails | 3 |
| 16 | API versioning | `/api/v1/...` with Swagger per version | 3 |
| 17 | CORS | Named policy from options | 3 |
| 18 | Rate limiting | Built-in middleware with named policies | 3 |
| 19 | Hangfire | Fire-and-forget, delayed and recurring jobs | 4 |
| 20 | MailKit | Email sending | 4 |
| 21 | File handling | Banner upload, receipt PDF | 4 |
| 22 | Serilog | Structured logging | 4 |
| 23 | Stripe (Checkout + webhooks) | Payments | 5 |
| 24 | Idempotency key | Safe retries on create endpoints | 5 |
| 25 | Integration testing | `WebApplicationFactory` | 5 |
| 26 | Health checks | Liveness and readiness | 6 |
| 27 | Docker + Docker Compose | Run the whole stack | 6 |
| 28 | GitHub Actions | Build and test on every push | 6 |

### 3.2 Optional technology [BONUS]
Specification pattern for query reuse, Domain Events (in-process notifications instead of direct calls), API v2 of one endpoint, Testcontainers, Seq for logs, refresh token reuse detection, refund flow.

### 3.3 Architecture rules [CORE]
1. **Layers.** `Domain` has no dependency on anything. `Application` depends on `Domain`. `Infrastructure` depends on `Application`. `Api` depends on `Application` and `Infrastructure` (for composition only).
2. **Slices.** Inside `Application`, organize by feature, not by technical type. One slice per use case.
3. **Controllers are thin.** A controller maps HTTP to a command or query, sends it through MediatR, and maps the `Result` to an HTTP response. No business logic in controllers.
4. **Business rules live in the Domain** (entity methods and value objects), not in handlers or controllers.
5. **Writes** go through the Generic Repository and Unit of Work. **Reads** (queries) may use the `DbContext` directly with `AsNoTracking` and projection to DTOs. Explain this choice in your README.
6. **Never return entities from the API.** Return DTOs.
7. **No `DateTime.UtcNow` inside domain or handlers.** Inject a clock abstraction (`TimeProvider`), so tests are deterministic.

### 3.4 Suggested solution layout
```
DonationHub.sln
src/
  DonationHub.Domain/
    Common/               (Entity, AuditableEntity, Error, Result)
    Campaigns/  Categories/  Donations/  Payments/  Identity/
  DonationHub.Application/
    Common/               (Behaviors, Abstractions, Mappings, Options)
    Features/
      Auth/               (Register/, Login/, Refresh/, Logout/ ...)
      Categories/         (Create/, Update/, Delete/, GetById/, List/)
      Campaigns/  Donations/  Payments/  Reports/  Admin/
  DonationHub.Infrastructure/
    Persistence/          (DbContext, Configurations, Migrations, Repositories)
    Identity/  Payments/  Email/  Files/  Jobs/  Caching/
  DonationHub.Api/
    Controllers/  Middleware/  Extensions/  Program.cs  appsettings.json
tests/
  DonationHub.UnitTests/
  DonationHub.IntegrationTests/
docker-compose.yml  Dockerfile  README.md  .github/workflows/ci.yml
```
The names are a guide. You may choose different names, but the dependency rules in 3.3 are mandatory.

---

## 4. Domain Model

### 4.1 Entities
**ApplicationUser** (ASP.NET Core Identity user)
`Id`, `Email`, `UserName`, `FullName`, `PreferredLanguage` (`en` | `ar`, default `en`), `IsActive`, `CreatedAt`. Roles: `Admin`, `CampaignManager`, `Donor`.

**RefreshToken**
`Id`, `UserId`, `TokenHash`, `CreatedAt`, `ExpiresAt`, `RevokedAt?`, `ReplacedByTokenId?`.

**Category**
`Id`, `NameAr`, `NameEn`, `DescriptionAr`, `DescriptionEn`, `CreatedAt`, `UpdatedAt`.

**Campaign**
`Id`, `TitleAr`, `TitleEn`, `DescriptionAr`, `DescriptionEn`, `GoalAmount`, `RaisedAmount`, `EndDate`, `CategoryId`, `BannerFileName?`, `Status`, `CreatedByUserId`, `CreatedAt`, `UpdatedAt`, `IsDeleted`, `RowVersion` (concurrency token).

**Donation**
`Id`, `CampaignId`, `DonorUserId`, `Amount`, `Status`, `CreatedAt`, `CompletedAt?`, `FailureReason?`, `ReceiptNumber?`, `ReceiptFileName?`.
Bonus fields: `IsAnonymous`, `RefundedAt?`.

**PaymentAttempt**
`Id`, `DonationId`, `StripeCheckoutSessionId`, `StripePaymentIntentId?`, `Amount`, `Status`, `CreatedAt`, `UpdatedAt`, `FailureCode?`.

**ProcessedStripeEvent**
`EventId` (key), `Type`, `ProcessedAt`.

**IdempotencyRecord**
`Key`, `UserId`, `RequestHash`, `ResponseStatusCode`, `ResponseBody`, `CreatedAt`, `ExpiresAt`.

### 4.2 Enums
| Enum | Values |
|---|---|
| `CampaignStatus` | `Active`, `Completed`, `Expired` (Bonus: `Cancelled`) |
| `DonationStatus` | `Pending`, `Succeeded`, `Failed` (Bonus: `Refunded`) |
| `PaymentAttemptStatus` | `Created`, `Succeeded`, `Failed`, `Expired` |

### 4.3 Relationships
- Category 1 to many Campaign.
- Campaign 1 to many Donation.
- User 1 to many Donation, 1 to many RefreshToken, 1 to many Campaign (as creator).
- Donation 1 to many PaymentAttempt (a donor may retry payment).

### 4.4 State machines
**Campaign**
```
Active --(RaisedAmount >= GoalAmount)--> Completed
Active --(EndDate passed, goal not reached)--> Expired
Completed, Expired are final (Admin reopening is out of scope)
```
**Donation**
```
Pending --(payment succeeded)--> Succeeded
Pending --(payment failed, expired, or stale timeout)--> Failed
Succeeded, Failed are final (Bonus: Succeeded --refund--> Refunded)
```

### 4.5 Domain invariants [CORE]
Implement these inside the domain model and cover them with unit tests.
1. `GoalAmount > 0`.
2. `EndDate` must be in the future when a campaign is created or its end date is changed.
3. A campaign accepts a new donation only when `Status == Active` and `EndDate` is in the future.
4. `Amount` of a donation is within the configured minimum and maximum (defaults: 1.00 and 10,000.00).
5. `RaisedAmount` changes only when a donation moves to `Succeeded`.
6. When `RaisedAmount >= GoalAmount` after a donation succeeds, the campaign becomes `Completed`.
7. `GoalAmount` cannot be lowered below the current `RaisedAmount`.
8. A final state (Succeeded, Failed, Completed, Expired) cannot change.
9. A campaign that has donations cannot be hard-deleted. Use soft delete.
10. A category that still has campaigns cannot be deleted.

---

## 5. Functional Requirements

Base path: `/api/v1`. Roles in the "Access" column: **A** Anonymous, **D** Donor, **M** Campaign Manager, **Adm** Admin. "Any auth" means any logged-in user.

### 5.1 Module: Identity (Sprint 1)

| ID | Pri | Requirement |
|---|---|---|
| IDN-01 | CORE | A visitor registers with email, password, full name and preferred language. The new user gets the `Donor` role. |
| IDN-02 | CORE | Email is unique (case-insensitive). Password policy: at least 8 characters, upper case, lower case, digit. |
| IDN-03 | CORE | Login with email and password returns an access token (JWT) and a refresh token. |
| IDN-04 | CORE | Access token lifetime is 15 minutes (configurable). Claims: user id, email, roles. |
| IDN-05 | CORE | Refresh token lifetime is 7 days (configurable). It is stored **hashed**, never in plain text. |
| IDN-06 | CORE | `refresh` exchanges a valid refresh token for a new access token and a **new** refresh token (rotation). The old one is revoked and linked to the new one. |
| IDN-07 | CORE | `logout` revokes the supplied refresh token. |
| IDN-08 | CORE | An Admin creates a staff user (role `Admin` or `CampaignManager`) with a temporary password. |
| IDN-09 | CORE | `GET /auth/me` returns the current user's profile. |
| IDN-10 | CORE | A deactivated user (`IsActive = false`) cannot log in or refresh. |
| IDN-11 | CORE | An initial Admin is seeded from options (email, password) at startup. |
| IDN-12 | CORE | Login failures return the same generic error whether the email or the password is wrong (no user enumeration). |
| IDN-13 | BONUS | Refresh token reuse detection: using an already-revoked token revokes the whole token family. |
| IDN-14 | BONUS | Change password and update profile endpoints. |

| Method | Path | Access | Purpose |
|---|---|---|---|
| POST | `/auth/register` | A | Register a donor |
| POST | `/auth/login` | A | Login |
| POST | `/auth/refresh` | A | Rotate tokens |
| POST | `/auth/logout` | Any auth | Revoke refresh token |
| GET | `/auth/me` | Any auth | Current profile |
| POST | `/admin/staff` | Adm | Create Admin or Campaign Manager (managing the rest of staff comes in Sprint 6) |

### 5.2 Module: Categories (Sprint 1)

| ID | Pri | Requirement |
|---|---|---|
| CAT-01 | CORE | Admin creates, updates and deletes categories with Arabic and English name and description. |
| CAT-02 | CORE | Names are required in both languages, max 100 characters, unique per language. |
| CAT-03 | CORE | Anyone can list categories and get one by id. |
| CAT-04 | CORE | Deleting a category that has campaigns is rejected with a clear error (invariant 10). |

| Method | Path | Access |
|---|---|---|
| GET | `/categories` | A |
| GET | `/categories/{id}` | A |
| POST | `/categories` | Adm |
| PUT | `/categories/{id}` | Adm |
| DELETE | `/categories/{id}` | Adm |

### 5.3 Module: Campaigns (Sprint 2, improved in Sprint 3)

| ID | Pri | Requirement |
|---|---|---|
| CMP-01 | CORE | Create a campaign with Arabic/English title and description, goal amount, end date, category. Status starts as `Active`, `RaisedAmount` as 0. |
| CMP-02 | CORE | Update a campaign's titles, descriptions, category, end date and goal, subject to invariants 2, 7 and 8. |
| CMP-03 | CORE | Soft delete a campaign. Soft-deleted campaigns never appear in any query (use a global query filter). |
| CMP-04 | CORE | Public list returns active campaigns. Public "success stories" returns completed campaigns. |
| CMP-05 | CORE | Campaign details return all fields plus `progressPercent` (raised / goal, capped at 100) and `daysLeft`. |
| CMP-06 | CORE | In Sprint 2 only Admin can create and manage campaigns. |
| CMP-07 | CORE | From Sprint 3: Campaign Manager can create campaigns and manage **only the ones they created**. Admin manages all. |
| CMP-08 | CORE | From Sprint 3: the public list supports pagination, filtering by category and status, search in title (current language), and sorting by end date, raised amount or creation date. |
| CMP-09 | CORE | From Sprint 3: public reads are cached and the cache is invalidated when a campaign or category changes. |
| CMP-10 | CORE | Concurrent edits of the same campaign are detected using `RowVersion`. The second writer receives a conflict error. |
| CMP-11 | CORE | Public responses use the language of the `Accept-Language` header (`ar` or `en`), fallback `en`. Admin and manager endpoints return both languages. |
| CMP-12 | BONUS | Campaign `Cancelled` status set by Admin or the owner. |
| CMP-13 | BONUS | Specification pattern for the list filters. |

| Method | Path | Access |
|---|---|---|
| GET | `/campaigns` | A |
| GET | `/campaigns/success-stories` | A |
| GET | `/campaigns/{id}` | A |
| POST | `/campaigns` | Adm (Sprint 2), Adm and M (Sprint 3 onward) |
| PUT | `/campaigns/{id}` | Adm, owner M |
| DELETE | `/campaigns/{id}` | Adm, owner M |
| GET | `/campaigns/mine` | M (own campaigns, including all statuses) |

### 5.4 Module: Donations (Sprint 3, improved in Sprints 4 and 5)

| ID | Pri | Requirement |
|---|---|---|
| DON-01 | CORE | A logged-in user creates a donation for an active campaign with an amount. The donation starts as `Pending`. |
| DON-02 | CORE | Reject a donation if the campaign is not active, its end date passed, or the amount is outside the allowed range (invariants 3 and 4). |
| DON-03 | CORE | A donor lists their own donations (paged) with amount, date, status and campaign title. |
| DON-04 | CORE | A donor gets one of their own donations by id. Accessing someone else's donation returns 404 (do not reveal it exists). |
| DON-05 | CORE | Admin lists all donations with filters: status, campaign, donor, date range. Paged. |
| DON-06 | CORE | **Temporary stub (Sprints 3 and 4):** `POST /admin/donations/{id}/mark-succeeded` simulates a successful payment so you can test the full flow without Stripe. **It must be removed in Sprint 5.** |
| DON-07 | CORE | Marking a donation succeeded runs the success flow described in 5.5 (raise amount, complete campaign if goal reached) inside **one database transaction**. |
| DON-08 | CORE | Raised amount updates are safe under concurrency: two donations succeeding at the same time must both be counted (use `RowVersion` with retry, or an atomic update). |
| DON-09 | CORE | A `Pending` donation that is not paid within the configured timeout (default 30 minutes) becomes `Failed`. (Implemented with Hangfire in Sprint 4.) |
| DON-10 | CORE | From Sprint 5: `POST /donations` and `POST /donations/{id}/checkout` accept an `Idempotency-Key` header (see 6.5). |
| DON-11 | BONUS | Anonymous donation flag: the donor's name is hidden from public displays. |
| DON-12 | BONUS | Public list of recent donors of a campaign (respecting the anonymous flag). |
| DON-13 | BONUS | Refund of a succeeded donation (Admin), with amount subtracted from the raised amount. |

| Method | Path | Access |
|---|---|---|
| POST | `/donations` | D, M, Adm |
| GET | `/donations/mine` | Any auth |
| GET | `/donations/{id}` | Owner, Adm |
| GET | `/admin/donations` | Adm |
| POST | `/admin/donations/{id}/mark-succeeded` | Adm (temporary, removed in Sprint 5) |
| GET | `/donations/{id}/receipt` | Owner, Adm (Sprint 4) |

### 5.5 Success flow (shared by the stub and Stripe)
When a donation becomes `Succeeded`, do the following in **one transaction**:
1. Set donation status `Succeeded`, set `CompletedAt`, generate `ReceiptNumber` (format `DN-{year}-{6-digit sequence}`).
2. Add the amount to the campaign's `RaisedAmount`.
3. If `RaisedAmount >= GoalAmount`, set the campaign to `Completed`.
4. Commit.

After the commit (never inside the transaction), enqueue background jobs (Sprint 4 onward): generate the receipt PDF, email the donor, and, if the campaign just completed, email the campaign owner.

Edge case: if a payment succeeds after the campaign already expired or completed, **accept the money and record the donation as Succeeded** (the donor was charged already). Log a warning. Refunds are Bonus.

### 5.6 Module: Notifications (Sprint 4)

| ID | Pri | Requirement |
|---|---|---|
| NTF-01 | CORE | Send an email through MailKit using SMTP settings from options. |
| NTF-02 | CORE | Email receipt to the donor after a successful donation, with the PDF receipt attached. |
| NTF-03 | CORE | Email the campaign creator when the campaign reaches its goal. |
| NTF-04 | CORE | Emails are sent by Hangfire jobs, never inside the HTTP request. A failed job retries (at least 3 attempts) and logs the failure. |
| NTF-05 | CORE | Email text uses the recipient's `PreferredLanguage` (ar or en). Arabic emails render right-to-left. |
| NTF-06 | CORE | In development, emails go to the Mailpit container (no real email is sent). |
| NTF-07 | BONUS | Welcome email after registration. |
| NTF-08 | BONUS | Email the campaign creator when the campaign expires without reaching its goal. |

### 5.7 Module: Files and Receipts (Sprint 4)

| ID | Pri | Requirement |
|---|---|---|
| FIL-01 | CORE | Upload a campaign banner (Admin or owner). Allowed types: JPG, PNG, WEBP. Max size 2 MB (configurable). |
| FIL-02 | CORE | Validate content, not only the extension: check the file signature (magic bytes). |
| FIL-03 | CORE | Store files through an `IFileStorage` abstraction (local disk implementation). Save under a generated random name, never the user's file name. |
| FIL-04 | CORE | Replacing a banner deletes the old file. Deleting a campaign keeps the file until hard cleanup (not required). |
| FIL-05 | CORE | The banner is served publicly through a URL returned in the campaign response. |
| FIL-06 | CORE | Generate a PDF receipt for each succeeded donation (receipt number, donor name, campaign title in the donor's language, amount, date). Generation runs in a Hangfire job. |
| FIL-07 | CORE | `GET /donations/{id}/receipt` downloads the PDF, only for the owner or an Admin. |
| FIL-08 | BONUS | Admin exports donations to CSV. |
| FIL-09 | BONUS | Image resizing or thumbnail generation. |

| Method | Path | Access |
|---|---|---|
| POST | `/campaigns/{id}/banner` | Adm, owner M (multipart/form-data) |
| DELETE | `/campaigns/{id}/banner` | Adm, owner M |
| GET | `/donations/{id}/receipt` | Owner, Adm |

### 5.8 Background jobs (Sprint 4)

| ID | Pri | Job | Type | Behavior |
|---|---|---|---|---|
| JOB-01 | CORE | Send donation receipt | Fire-and-forget | Generate PDF then email it |
| JOB-02 | CORE | Send goal-reached email | Fire-and-forget | To the campaign creator |
| JOB-03 | CORE | Fail stale pending donation | Delayed | Scheduled when a donation is created, runs after the pending timeout, fails the donation if it is still `Pending` |
| JOB-04 | CORE | Expire campaigns | Recurring (hourly) | Active campaigns past `EndDate` become `Expired` |
| JOB-05 | CORE | The Hangfire dashboard is enabled and protected (not publicly open). |
| JOB-06 | BONUS | Daily summary email to Admin |

Jobs must be **idempotent**: running a job twice must not send two emails or change state twice.

### 5.9 Module: Payments with Stripe (Sprint 5)

| ID | Pri | Requirement |
|---|---|---|
| PAY-01 | CORE | `POST /donations/{id}/checkout` creates a Stripe **Checkout Session** (mode `payment`) for a `Pending` donation and returns the checkout URL. |
| PAY-02 | CORE | The session carries the donation id in `client_reference_id` and metadata, a single line item (campaign title, amount in minor units, configured currency), and success and cancel URLs from options. |
| PAY-03 | CORE | A `PaymentAttempt` is stored for each created session. A donor may start a new attempt for the same pending donation. |
| PAY-04 | CORE | `POST /payments/webhook` receives Stripe events. It is anonymous, reads the **raw body**, and **verifies the signature** with the webhook secret. An invalid signature returns 400. |
| PAY-05 | CORE | Handle `checkout.session.completed` (run the success flow in 5.5), `checkout.session.expired` (attempt `Expired`) and `payment_intent.payment_failed` (attempt `Failed`, donation `Failed` when no other attempt can still succeed). |
| PAY-06 | CORE | Webhooks are **idempotent**: store each processed `EventId` and ignore duplicates, answering 200. |
| PAY-07 | CORE | Verify that the paid amount and currency match the donation. A mismatch is logged as an error and does not complete the donation. |
| PAY-08 | CORE | The webhook returns 200 quickly. Heavy work (PDF, email) is enqueued, not done inside the webhook. |
| PAY-09 | CORE | Remove the temporary `mark-succeeded` stub (DON-06). |
| PAY-10 | CORE | Use Stripe **test mode** keys only. Test locally with the Stripe CLI (`stripe listen`, `stripe trigger`). |
| PAY-11 | BONUS | Admin refund (see DON-13). |
| PAY-12 | BONUS | Handle `charge.refunded`. |

| Method | Path | Access |
|---|---|---|
| POST | `/donations/{id}/checkout` | Owner |
| GET | `/donations/{id}/payment-status` | Owner, Adm |
| POST | `/payments/webhook` | A (signature required) |

### 5.10 Module: Reports and Admin (Sprint 6)

| ID | Pri | Requirement |
|---|---|---|
| RPT-01 | CORE | Dashboard summary: total raised, number of succeeded donations, number of distinct donors, active, completed and expired campaign counts. |
| RPT-02 | CORE | Top N campaigns by raised amount. |
| RPT-03 | CORE | Donations over time (per day) for a date range. |
| RPT-04 | CORE | Dashboard responses are cached (default 5 minutes) and the numbers must match the database. |
| ADM-01 | CORE | Admin lists all users (paged, filter by role, search by name or email). |
| ADM-02 | CORE | Admin activates or deactivates a user. A deactivated user cannot log in. |
| ADM-03 | CORE | Admin lists, updates and deletes staff users. Delete means deactivate. An Admin cannot deactivate themself, and the last active Admin cannot be removed. |
| RPT-05 | BONUS | Donations by category and by month. |

| Method | Path | Access |
|---|---|---|
| GET | `/admin/dashboard/summary` | Adm |
| GET | `/admin/dashboard/top-campaigns?count=5` | Adm |
| GET | `/admin/dashboard/donations-over-time?from=&to=` | Adm |
| GET | `/admin/users` | Adm |
| PATCH | `/admin/users/{id}/status` | Adm |
| GET | `/admin/staff` | Adm |
| PUT | `/admin/staff/{id}` | Adm |
| DELETE | `/admin/staff/{id}` | Adm |

---

## 6. Cross-Cutting Requirements

### 6.1 Error handling [CORE, Sprint 2]
- Expected failures (not found, validation, business-rule violations, conflicts) are returned as a `Result` with an `Error` (`Code`, `Message`, `Type`). Do not throw exceptions for them.
- Unexpected failures are caught by one global handler (`IExceptionHandler`) that logs the exception and returns a generic 500 without leaking details.
- All errors are returned as **ProblemDetails** (RFC 9457) with extra fields `code`, `traceId` and, for validation, `errors` (field to messages).
- Error `Type` maps to HTTP: Validation 400, Unauthorized 401, Forbidden 403, NotFound 404, Conflict 409, UnprocessableEntity 422, TooManyRequests 429, Failure 500.

Minimum error code catalogue:
| Code | HTTP |
|---|---|
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
| `Donation.NotFound` | 404 |
| `Donation.AmountOutOfRange` | 422 |
| `Donation.InvalidState` | 409 |
| `Payment.InvalidSignature` | 400 |
| `Idempotency.KeyReusedWithDifferentRequest` | 422 |
| `File.InvalidType` | 400 |
| `File.TooLarge` | 413 |
| `Common.RateLimited` | 429 |

### 6.2 Validation [CORE, Sprint 2]
- FluentValidation validators for every command and query with input.
- Validation runs automatically in a MediatR pipeline behavior. Handlers never repeat basic validation.
- Validation messages are localized (6.7).

### 6.3 Security [CORE]
- Passwords are hashed by Identity. Refresh tokens are stored hashed.
- Never log passwords, tokens, Stripe secrets or full request bodies of auth endpoints.
- Secrets come from user-secrets or environment variables. The repository contains only `appsettings.json` with placeholders.
- Authorization is enforced in the API **and** checked again in handlers when a resource has an owner.
- Return 404 (not 403) when a user asks for another user's private resource.

### 6.4 Rate limiting [CORE, Sprint 3]
Use the built-in ASP.NET Core rate limiter with named policies configured through options.
| Policy | Applies to | Default |
|---|---|---|
| `global` | all endpoints, per IP | 100 requests / minute |
| `auth` | login, register, refresh | 10 requests / minute per IP |
| `donations` | donation creation and checkout | 20 requests / minute per user |
The webhook endpoint is **excluded** from rate limiting. Rejected requests return 429 with a `Retry-After` header and a ProblemDetails body.

### 6.5 Idempotency key [CORE, Sprint 5]
- Applies to `POST /donations` and `POST /donations/{id}/checkout`.
- The client sends `Idempotency-Key: <unique string>`. The key is required on these endpoints (400 if missing).
- First request: process and store the key, user id, a hash of the request body and the response.
- Same key, same user, same body: return the stored response without repeating the action.
- Same key, different body: return 422 `Idempotency.KeyReusedWithDifferentRequest`.
- Keys expire after 24 hours.
- Two simultaneous requests with the same key must not create two donations.

### 6.6 Caching [CORE, Sprint 3]
- Cache: categories list, public campaign list pages, campaign details, dashboard summary.
- Start with in-memory cache, then switch to `IDistributedCache` backed by Redis (the Redis container is part of Docker Compose).
- Use the cache-aside pattern. Define clear key names that include language and query parameters.
- Invalidate related keys when data changes. Document your invalidation strategy in the README.
- Never cache user-specific data (own donations) under a shared key.

### 6.7 Localization [CORE, Sprint 3]
- Supported cultures: `en` (default) and `ar`, selected by `Accept-Language`.
- Localize: validation messages, error messages, emails, the localized fields of public campaign and category responses.
- Texts live in resource files, not hard-coded strings in handlers.
- Data fields stay bilingual (Ar and En columns). Do not translate stored data automatically.

### 6.8 API conventions [CORE]
- **Versioning:** URL segment (`/api/v1`). Swagger shows one document per version. (Bonus: v2 of one endpoint.)
- **Paged response shape:** `{ "items": [], "page": 1, "pageSize": 20, "totalCount": 0, "totalPages": 0 }`. Default page size 20, maximum 50.
- **Status codes:** 200 read, 201 create (with `Location`), 204 no content, 400/401/403/404/409/422/429 as in 6.1.
- **Naming:** plural nouns, kebab-case paths, camelCase JSON. Enums are returned as strings.
- **CORS:** one named policy whose allowed origins come from options. Never `AllowAnyOrigin` together with credentials.

### 6.9 Configuration (Options pattern) [CORE, Sprint 1 onward]
Each option class is bound from configuration, validated with data annotations and `ValidateOnStart()`. The app must fail at startup with a clear message when configuration is invalid.
`JwtOptions`, `SeedAdminOptions`, `DonationOptions` (min, max, pending timeout, currency), `StripeOptions`, `EmailOptions`, `FileStorageOptions`, `CorsOptions`, `RateLimitOptions`, `HangfireDashboardOptions`, `CacheOptions`.

### 6.10 Logging [CORE, Sprint 4]
- Serilog replaces the default logger. Sinks: console and rolling file.
- Enrich every log entry with a correlation id (`traceId`), and the user id when authenticated.
- Request logging middleware with status code and duration.
- Use structured templates (`"Donation {DonationId} succeeded"`), not string concatenation.
- Log levels: Information for business events, Warning for recoverable anomalies (late payment, unknown webhook session), Error for failures.

### 6.11 Health checks [CORE, Sprint 6]
- `/health/live`: the process is up (no dependency checks).
- `/health/ready`: SQL Server, Redis, and write access to the file storage folder.
- Responses are JSON with each check's status.

### 6.12 Docker [CORE, Sprint 6]
- Multi-stage `Dockerfile` for the API (non-root user).
- `docker-compose.yml` with services: `api`, `sqlserver`, `redis`, `mailpit`. Use named volumes for SQL data and uploaded files.
- All settings through environment variables. `docker compose up` must produce a working API with migrations applied and the admin seeded.
- Health checks and `depends_on` conditions for startup order.

### 6.13 Testing [CORE]
- **Unit tests (Sprint 2 onward):** domain invariants, handlers (with the repository and clock faked), validators. Minimum: 15 tests by Sprint 2, and tests added for every new module afterward.
- **Integration tests (Sprint 5):** use `WebApplicationFactory` and a test database. Cover at least: register and login, create campaign and donation, the webhook success flow with a locally signed payload, duplicate webhook event, invalid signature, idempotency key replay.
- Tests must run with `dotnet test` and in the CI pipeline. Do not call real Stripe or real SMTP in tests.

### 6.14 CI [CORE, Sprint 6]
A GitHub Actions workflow runs restore, build and `dotnet test` on every push and pull request. The badge appears at the top of the README.

---

## 7. Sprint Plan

Each sprint ends with a checkpoint tag. The new modules are what you build; the improvement is what you apply to earlier modules using the sprint's topics. Individual tasks and the grading checklist for each sprint are published on the platform at the start of the sprint.

| Sprint | Dates | New modules | Topics | Improvements to earlier work |
|---|---|---|---|---|
| **1** | 20/10 to 2/11 | Identity, Categories | Clean Architecture + Vertical Slice, EF Core, Generic Repo + UoW, Options, Identity + JWT + Refresh | Categories protected by the Admin role |
| **2** | 3/11 to 16/11 | Campaigns | CQRS with MediatR, FluentValidation, Result, exception handling, unit testing | Identity and Categories move to CQRS with validation and Result |
| **3** | 17/11 to 30/11 | Donations (with the temporary stub) | Caching, authorization policies, pagination and filters, localization, API versioning, rate limiting, CORS | Campaign Manager ownership, cached and localized public reads |
| **4** | 1/12 to 14/12 | Notifications, Files and Receipts | Hangfire, MailKit, file handling, Serilog | Receipt flow and background jobs on Donations, Serilog everywhere |
| **5** | 15/12 to 28/12 | Payments | Stripe, webhooks, idempotency, integration testing | Stub removed, donations completed by webhook, idempotency on donations |
| **6** | 29/12 to 11/1 | Reports and Admin | Health checks, Docker, GitHub Actions | Dashboard caching, full containerization, CI |
| **Final** | 12/1 to 20/1 | none | none | Fixes, documentation, discussion |

Note: from Sprint 1 to Sprint 3, use the built-in `ILogger`. Serilog arrives in Sprint 4.

### 7.1 Deliverables per sprint (CORE)

**Sprint 1.** Working solution with the four-layer structure. Identity (IDN-01 to IDN-12) and Categories (CAT-01 to CAT-04). Migrations committed. Options classes validated at startup. Swagger with a Bearer authorization button. README with run steps.

**Sprint 2.** Campaigns (CMP-01 to CMP-06, CMP-10, CMP-11 for the basic list). All Sprint 1 use cases rewritten as commands and queries. Validators and the pipeline behavior. Result pattern and global exception handler with the error catalogue. At least 15 unit tests including the campaign invariants.

**Sprint 3.** Donations (DON-01 to DON-08). CMP-07 to CMP-09. Policies and resource ownership. Pagination, filtering, search and sorting on public and admin lists. Memory then Redis caching with invalidation. Localization for validation, errors and public responses. Versioned routes and Swagger per version. CORS and rate limiting policies.

**Sprint 4.** Notifications (NTF-01 to NTF-06), Files (FIL-01 to FIL-07), Jobs (JOB-01 to JOB-05), DON-09. Serilog configured. Mailpit used for development.

**Sprint 5.** Payments (PAY-01 to PAY-10). DON-10 and the idempotency behavior. Integration tests (see 6.13). Removal of the stub.

**Sprint 6.** Reports and Admin (RPT-01 to RPT-04, ADM-01 to ADM-03). Health checks. Dockerfile and Compose. CI workflow. Final README: architecture explanation, how to run (local and Docker), how to test Stripe locally, list of design decisions.

**Final.** Fix defects found in review. Bonus items. Prepare to explain your design decisions.

### 7.2 Definition of Done for every sprint
1. The solution builds with no warnings treated as errors, and runs with the README steps.
2. Every CORE requirement of the sprint works through Swagger.
3. Validation and error responses follow section 6.1 and 6.2 (from Sprint 2).
4. New behavior is covered by tests (from Sprint 2).
5. The checkpoint tag exists and points to the final commit.
6. The README is updated for the sprint (what was added, how to try it, decisions).
7. No secrets in the repository history.

---

## 8. Submission and Evaluation

- **Submit:** the repository URL on tawjeh.net before the sprint deadline (Monday 23:59). Tag the commit `sprint-N`.
- **Reviewed against:** the sprint checklist published with the tasks. It checks the requirements above, code structure, tests, and your README.
- **Bonus** items count only when all CORE items of that sprint pass.
- **Late work and fixes:** you can fix and resubmit within the review window announced on the platform. Work after the deadline is not counted unless the sprint rules say so.
- **Discussion:** mentors may ask you to explain parts of your code. Be ready to explain why, not only what.

---

## 9. Appendix

### 9.1 Seed data (development)
- Roles: Admin, CampaignManager, Donor.
- One Admin from options; one Campaign Manager and one Donor for testing (development only).
- Five categories (for example: Health, Education, Relief, Orphans, Environment) with Arabic and English names.
- Optional: sample campaigns in different statuses to test lists.

### 9.2 Typical request and response samples

Create a donation:
```http
POST /api/v1/donations
Authorization: Bearer <access-token>
Idempotency-Key: 6f1b0c1e-8a53-4c3a-9a9c-0d6f9d0c2f11
Content-Type: application/json

{ "campaignId": "…", "amount": 25.00 }
```
```http
201 Created
{ "id": "…", "campaignId": "…", "amount": 25.00, "status": "Pending", "createdAt": "2026-11-20T10:15:00Z" }
```

Error sample:
```http
422 Unprocessable Entity
Content-Type: application/problem+json

{
  "type": "https://httpstatuses.io/422",
  "title": "Donation amount is out of range.",
  "status": 422,
  "code": "Donation.AmountOutOfRange",
  "traceId": "00-9f2d…"
}
```

### 9.3 Local Stripe testing (Sprint 5)
1. Create a free Stripe account and stay in **test mode**.
2. Put the test secret key and webhook secret in user-secrets (never commit them).
3. Run `stripe listen --forward-to https://localhost:<port>/api/v1/payments/webhook` and copy the signing secret it prints into your configuration.
4. Use Stripe's test card numbers on the hosted Checkout page.
5. In automated tests, build the event payload yourself and sign it with a test secret; never call Stripe from tests.

### 9.4 Receipt numbering
`DN-{year}-{sequence padded to 6 digits}`. The sequence must be unique even under concurrent success flows (use a database sequence or a unique index with retry).

### 9.5 Requirement traceability (summary)
| Module | Requirements | Sprint |
|---|---|---|
| Identity | IDN-01 to IDN-12 | 1 |
| Categories | CAT-01 to CAT-04 | 1 |
| Campaigns | CMP-01 to CMP-06, CMP-10, CMP-11 | 2 |
| Campaigns (improved) | CMP-07 to CMP-09 | 3 |
| Donations | DON-01 to DON-08 | 3 |
| Donations (improved) | DON-09 | 4 |
| Notifications | NTF-01 to NTF-06 | 4 |
| Files and Receipts | FIL-01 to FIL-07 | 4 |
| Jobs | JOB-01 to JOB-05 | 4 |
| Payments | PAY-01 to PAY-10, DON-10 | 5 |
| Reports and Admin | RPT-01 to RPT-04, ADM-01 to ADM-03 | 6 |
| Cross-cutting | Section 6 | as listed |

### 9.6 Out of scope (do not build)
Front end, external identity server, social login, multi-currency, recurring donations, real payments, cloud deployment, message brokers, microservices.
