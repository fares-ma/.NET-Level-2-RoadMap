# DonationHub: Internal Team Notes (do not share with students)

Companion to `SRS-DonationHub.md` v0.9. This file explains **why** the SRS looks the way it does, what we expect to change, and how the lead and the four mentors should run it.

---

## 1. Decisions already made (by the lead)
| Decision | Value |
|---|---|
| Project | Donation Management, modified from the original SRS |
| Dropped | SignalR, AI |
| Kept | Stripe |
| Required tech | Clean Architecture, Vertical Slice, CQRS, JWT + refresh, UoW, generic repo, Hangfire, FluentValidation, Stripe, rate limiting, idempotency key, caching, API versioning, unit tests, files, localization, health checks, CORS, Docker, Result, Options, MailKit, Serilog, exception handling |
| CQRS library | MediatR (license check below) |
| Plan | 6 sprints of 2 weeks + 9 final days, 8 modules |
| Submission | GitHub link |
| Files | Markdown in English, student SRS plus this internal file |
| Deployment | Optional, never graded |

## 2. Assumptions I made (please confirm or veto)
| # | Assumption | Why | Cost if you reject it |
|---|---|---|---|
| A1 | **Controllers** (not Minimal APIs) | Students know controllers from Round 1; Swagger and versioning examples are plentiful | Low: rewrite 6.8 and the layout |
| A2 | **ASP.NET Core Identity** underneath JWT | Less custom code in a sprint that is already heavy | Medium: Sprint 1 grows if built from scratch |
| A3 | **Stripe Checkout (hosted page)** instead of PaymentIntent | API-only project, no front end to collect cards | Medium: PaymentIntent needs a client |
| A4 | **Single currency** (USD) | Removes currency conversion and edge cases | Low |
| A5 | **SQL Server** | Original SRS used it, Hangfire storage uses the same DB | Medium: Apple Silicon students (see risks) |
| A6 | Hangfire dashboard protected by **basic auth from options** | A JWT cannot be sent by a browser opening `/hangfire` | Low |
| A7 | **Redis in Sprint 3**, memory cache first | Same topic block (caching), Docker comes later but Redis can run from a single `docker run` | Low |
| A8 | **Specification is Bonus** | Lean plan; reads use the DbContext directly | Low, but it is one of the listed techs |
| A9 | Domain Events are **Bonus** | Workload; Core uses explicit job enqueue after commit | Low |
| A10 | **Soft delete** on Campaigns | Teaches global query filters and protects donation history | Low |
| A11 | Late payment on an expired campaign is **accepted** | The donor was already charged | Low |
| A12 | Integration tests use a **real SQL Server** (compose service or Testcontainers) | In-memory providers hide concurrency and rowversion behavior | Decision needed, see 3 |
| A13 | Project name `DonationHub` | Placeholder | None |

## 3. Open decisions
1. **.NET version.** Not decided. .NET 10 is the current LTS and .NET 8 is the older LTS. Pick one before the first resource list is made, because every video and package version depends on it. My recommendation: .NET 10, unless the best videos you have found are for .NET 8 and the students use machines that cannot install the newer SDK.
2. **Integration-test database** (A12). Options: SQL Server from Docker Compose, Testcontainers (Bonus in the SRS), or SQLite. I wrote "a test database" in the SRS. Decide before Sprint 5.
3. **Is MediatR registered with a license key?** See 11.
4. **Cohort identity.** New students or Round 1 graduates? It changes how much of Sprint 1 can be shortened.
5. **Review model.** Who grades which student, and whether the checklist is auto-checked by a CI script.
6. **Does tawjeh.net support** what the plan assumes (checkpoint tags, multi-file task text, bonus points)? I could not read the platform, so please check.

## 4. Changes from the original Donation SRS, and why
| Original | Now | Reason |
|---|---|---|
| OpenIddict authentication server | ASP.NET Core Identity + JWT + refresh tokens, inside the API | A second server doubles the setup and Docker work. The program lists JWT + refresh explicitly. |
| Angular donation portal and admin portal | Removed. API only. | Not backend learning. Students test with Swagger or Postman. |
| External payment page and redirect | Stripe Checkout Session + webhook | Real, common flow; teaches webhooks and idempotency |
| Donation statuses Pending, Successful, Failed | `Pending`, `Succeeded`, `Failed` (+ Bonus `Refunded`) | Aligned with Stripe wording |
| Campaign statuses Active, Completed, Expired | Same, with explicit state machine and rules | The original did not say when each transition happens |
| "Current raised amount based on donations" | Stored `RaisedAmount` updated in the same transaction as the donation, with a concurrency token | Teaches transactions and concurrency; fast reads |
| Campaign Manager (optional) | **Core** | It is the cleanest way to teach resource-based authorization |
| Anonymous donation (extra) | Bonus | Keeps scope small |
| Email receipt on success | Core, via Hangfire and MailKit | Required techs |
| PDF receipt (or email) | Core PDF + email attachment | Needed for the "working with files" requirement |
| Goal reached email (optional) | Core (to the campaign creator) | Good use of fire-and-forget jobs |
| Dashboard and reports (extra) | Core, small, cached | Needed for caching and Sprint 6 content |
| Category CRUD | Same, plus rule "cannot delete if campaigns exist" | Teaches domain rules and conflict errors |
| Arabic and English fields | Same, plus `Accept-Language` localization | Required technique: localization |
| Admin CRUD for admin users | Create in Sprint 1, manage in Sprint 6 | The seeded admin must be able to create a manager for Sprint 3 tests |

## 5. Additions (not in the original) and why
| Addition | Reason |
|---|---|
| `Idempotency-Key` on donation creation and checkout | Required tech; matches a real payment risk (double click) |
| `ProcessedStripeEvent` table | Webhook duplicates are normal; this is the simplest correct protection |
| `PaymentAttempt` entity | A donation can have several payment tries |
| Pending-timeout job (delayed) | Gives a real use case for a delayed Hangfire job |
| Expire-campaigns job (recurring) | Real use case for a recurring job |
| `mark-succeeded` stub (Sprints 3 and 4) | Lets the system work end to end before Stripe exists. Removed in Sprint 5. |
| Soft delete and global query filter | Protects history, teaches an EF feature |
| `RowVersion` on campaigns | Teaches optimistic concurrency without SignalR |
| Error code catalogue | One shared language for validation, Result and tests |
| Rate-limit policies, CORS policy, versioning | Required tech, kept small and option-driven |
| `PreferredLanguage` on users | Needed to localize emails |

## 6. Removed or deferred
SignalR, AI, Angular, OpenIddict, multi-currency, recurring donations, refunds (Bonus), anonymous donations (Bonus), campaign cancel status (Bonus), Specification (Bonus), Domain Events (Bonus), cloud deployment (optional).

---

## 7. Expected changes during the course, and how to handle them
| Likely change | Handling |
|---|---|
| Students fall behind in Sprint 1 (architecture plus Identity) | Keep Identity on the built-in stores. Make refresh token rotation the first item that can slip to the next sprint, but record it. |
| Sprint 3 is overloaded (8 topics) | Move API versioning and rate limiting to a short "docs only" task. The SRS already lists them as small topics. |
| Stripe test account problems (country, verification) | Offer a **fake payment gateway** behind `IPaymentGateway` as a fallback and still grade the webhook with the signed local payload. Decide before Sprint 5. |
| Students cannot run SQL Server in Docker (low RAM, ARM) | Allow SQL Server LocalDB or a hosted free tier for development, but require Docker in Sprint 6 only on machines that can run it. Mark the Docker task as Core-with-exception. |
| MediatR license warnings | See 11. |
| Package versions change | Pin versions in the template repository and in the first resource post. |
| Team wants to add something | Add as Bonus first. Move it to Core only before the sprint starts and only with an equal removal. |

Rule: **the SRS is frozen per sprint.** Changes are announced in a changelog at the top of the SRS and apply to the next sprint, never to the current one.

---

## 8. Sprint notes for mentors

### Sprint 1: Identity + Categories
- Check the **dependency direction** first (Domain references nothing). It is the most common structural error.
- Common mistakes: refresh tokens stored in plain text, no unique email constraint, entities returned from controllers, options read with `IConfiguration["..."]` instead of option classes.
- Seed admin must come from options, not a hard-coded password.
- Reference solution tag: `ref/sprint-1`.

### Sprint 2: Campaigns + CQRS
- Check that **handlers return `Result`**, not exceptions, for expected failures.
- Common mistakes: validation duplicated in handlers; `DateTime.UtcNow` inside the domain; business rules left in controllers; tests that only test getters.
- Watch for over-engineering: MediatR notifications and generic everything are not required.

### Sprint 3: Donations + API engineering
- This is the heaviest sprint. Expect partial work on localization and cache invalidation.
- Check concurrency: two donations succeeding at once (DON-08). Ask the student to explain how they handled it.
- Check that **cache keys include language and query parameters**, otherwise Arabic users get English data.
- Check ownership: a manager must not edit another manager's campaign (return 404 or 403 per the SRS).
- Common mistake: caching user-specific data under a shared key.

### Sprint 4: Notifications + Files + Hangfire
- Check that **no email or PDF work runs inside the request**.
- Jobs must be idempotent; ask: "what happens if this job runs twice?"
- File upload: verify magic-byte check and random file names. Try uploading a `.exe` renamed `.jpg`.
- Hangfire dashboard must not be open publicly.
- Serilog: verify no secrets are logged.

### Sprint 5: Payments
- The mentor needs a working Stripe test account to review. Prepare a shared test-mode account or a recorded demo.
- Checks: signature verification, raw body, duplicate event ignored, amount mismatch handled, stub removed, idempotency replay returns the same response.
- Common mistake: marking the donation succeeded from the **success URL** instead of the webhook. This is a critical error.
- Common mistake: doing PDF and email work inside the webhook.

### Sprint 6: Reports + operations
- Verify dashboard numbers against a hand-written SQL query.
- `docker compose up` must work from a clean clone. Mentors should literally try it on at least three submissions.
- CI must pass on the tagged commit.

### Final period
Discussion only. Ask each sampled student to explain one decision (for example, why the raised amount is stored and not computed).

---

## 9. Grading skeleton (proposal, not final)
Per sprint, 100 points:
| Layer | Weight | How |
|---|---|---|
| Automated | about 40 | Build, tests pass, tag exists, Core endpoints respond as the API contract says (smoke script run by a mentor) |
| Checklist | about 40 | 8 to 10 yes/no items written by the sprint owner (structure, rules, security, README) |
| Explanation | about 20 | Short question to a sampled subset of students; others get a default grade based on code review of decisions in the README |

Rules:
- Core only counts for the pass mark. Bonus adds up to a capped extra.
- All four mentors grade the **same three submissions** before the first real grading, then compare and adjust the checklist wording (calibration).
- Every sprint owner writes a private reference solution **one sprint ahead** and tags it `ref/sprint-N`.

## 10. Who owns what (proposal)
| Sprint | Owner of resources, tasks, checklist, reference solution |
|---|---|
| 1 | Lead |
| 2 | Mentor 1 |
| 3 | Mentor 2 |
| 4 | Mentor 3 |
| 5 | Mentor 4 |
| 6 and final | Shared |
Each student is followed by one mentor from start to end (6 to 12 students each).

## 11. Packages and licenses to verify before publishing the resource list
| Item | What is known / what to verify |
|---|---|
| **MediatR** | **Verified on the web (October 2026 session).** Version 12.5.0 was the last release under Apache 2.0. From 13.0 (July 2025) it is dual-licensed (RPL-1.5 or a commercial license). A free **Community** edition covers educational use, non-production use, and organizations under USD 5M revenue. Registering for a license key is required (used for auditing; the check logs a warning when the key is missing). Options: use a registered Community key, pin 12.5.0 (no updates), or implement CQRS manually. Tell students which one to use. |
| FluentValidation | Verify the current license of the latest major version. |
| FluentAssertions | Newer major versions changed license terms. Verify, or use plain xUnit assertions or Shouldly. |
| Mocking library | Moq had a trust controversy in 2023; NSubstitute is a safe alternative. Pick one and teach it. |
| Hangfire | Open-source core is LGPL; the dashboard and Pro features differ. Verify. |
| PDF library | QuestPDF has a Community license with a revenue limit. Verify before recommending, or choose another library. |
| Redis | The server license changed in 2024 and again for Redis 8. For a teaching container it is fine; Valkey is a drop-in alternative. Verify. |
| Stripe.net, MailKit, Serilog | Generally permissive; verify at publishing time. |

I only verified MediatR in this session. The others are from memory and must be checked.

## 12. Resource list to prepare (topics, not links)
Rule: **big topic = one video or playlist, small topic = under 45 minutes or official docs**. Each resource must be open and on the program's .NET version. Reuse the Round 2 repo list first.

| Sprint | Big | Small |
|---|---|---|
| 1 | Clean Architecture + Vertical Slice; Identity + JWT + Refresh | Options Pattern; EF Core review; Generic Repo + UoW |
| 2 | CQRS with MediatR; Unit Testing (xUnit) | FluentValidation; Result Pattern; Exception handling + ProblemDetails |
| 3 | Caching (memory + Redis) | Authorization policies; Pagination and filtering; Localization; API versioning, rate limiting, CORS (docs) |
| 4 | Hangfire | MailKit; File upload and PDF; Serilog |
| 5 | Stripe Checkout and webhooks; Integration testing | Idempotency key |
| 6 | Docker and Compose | Health checks; GitHub Actions; README writing |

For each resource record: title, link, language, duration, .NET version, why it was chosen, and the date it was verified. Keep one fallback per big topic.

## 13. Risks
| Risk | Impact | Mitigation |
|---|---|---|
| Sprint 3 overload | High | Docs-only versioning, rate limiting, CORS; Bonus for Specification |
| Stripe account issues | High | Fake gateway fallback; shared test account for mentors |
| Docker on weak machines | Medium | Allow LocalDB for development; relax the Docker requirement per machine |
| Mentor grading differences | High | Calibration, written checklists, reference solutions |
| Dropouts in January (university exams) | High | Freeze content by 11/1; the final period needs no new topic |
| Copied solutions | Medium | Checkpoint tags, commit history, explanation sampling |
| Package license or version drift | Medium | Pin versions in a template repo; license check in section 11 |
| The platform cannot do something the plan needs | Medium | Verify platform features before 20/10 |
| Mentors overloaded by grading | High | 3 tasks per sprint, 8-item checklist, automated smoke script |

## 14. Before 20/10 (checklist)
1. Decide: .NET version, MediatR approach, test database, review model.
2. Confirm or veto assumptions A1 to A13.
3. Finalize the SRS (v1.0) after the team discussion.
4. Build the **template repository** (empty solution with the four projects and the CI file) so every student starts equal.
5. Sprint 1: resource list, three tasks, checklist, reference solution.
6. Write the first platform post: rules, calendar, how to submit, how tags work.
7. Calibration session between mentors on a sample submission.
8. Check the platform for features the plan relies on.
