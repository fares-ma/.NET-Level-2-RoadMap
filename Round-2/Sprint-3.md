# Sprint 3 — Donations, Authorization & API Engineering

> Project: **DonationHub — Donation Management Platform**
> Duration: **2 weeks (~15 hours/week)**
> Evaluation: **100 points**

## Sprint Goal

Move DonationHub beyond basic CRUD by implementing the Donations module, enforcing authorization and ownership rules, and adding production-quality API features: caching, localization, pagination, filtering, sorting, versioning, rate limiting, and CORS.

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

### Task 1 — Authorization Policies & Resource Ownership

Implement project-level authorization:

- **CMP-07:** Campaign Manager can create campaigns and manage **only the ones they created**. Admin manages all.
- Return **404** (not 403) when a user asks for another user's private resource (do not reveal it exists).
- Use ASP.NET Core authorization policies and resource-based authorization.

| Endpoint | Campaign Manager | Admin |
|----------|-----------------|-------|
| `POST /campaigns` | ✅ Create own | ✅ Create |
| `PUT /campaigns/{id}` | ✅ Own only | ✅ All |
| `DELETE /campaigns/{id}` | ✅ Own only | ✅ All |
| `GET /campaigns/mine` | ✅ Own campaigns | N/A |

### Resources
- **MUST** — [ASP.NET Core authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction)
- **MUST** — [Resource-based authorization](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resource-based)
- **VIDEO** — [Milan Jovanović — Resource-Based Authorization in ASP.NET Core](https://www.youtube.com/watch?v=24r7UmuRh0A)
- **VIDEO** — [Raw Coding — Resource-Based Authorization](https://www.youtube.com/watch?v=680Qp1r4ZFc)

---

### Task 2 — Donations Module

Implement the Donations feature:

- **DON-01:** A logged-in user creates a donation for an active campaign with an amount. Status starts `Pending`.
- **DON-02:** Reject if campaign is not active, end date passed, or amount outside allowed range (configured min/max from `DonationOptions`).
- **DON-03:** A donor lists their own donations (paged) with amount, date, status, and campaign title.
- **DON-04:** A donor gets one of their own donations by id. Accessing someone else's donation returns 404.
- **DON-05:** Admin lists all donations with filters: status, campaign, donor, date range. Paged.

| Method | Path | Access |
|--------|------|--------|
| POST | `/donations` | Donor, Manager, Admin |
| GET | `/donations/mine` | Any auth |
| GET | `/donations/{id}` | Owner, Admin |
| GET | `/admin/donations` | Admin |

### Resources
- **MUST** — [EF Core — Querying](https://learn.microsoft.com/en-us/ef/core/querying/)
- **MUST** — [EF Core — Relationships](https://learn.microsoft.com/en-us/ef/core/modeling/relationships)

---

### Task 3 — Donation Success Flow (Temporary Stub)

Implement a temporary admin endpoint to simulate a successful payment:

- **DON-06:** `POST /admin/donations/{id}/mark-succeeded` simulates a successful payment. **This must be removed in Sprint 5.**

- **DON-07:** The success flow runs in **one database transaction:**
  1. Set donation status to `Succeeded`, set `CompletedAt`, generate `ReceiptNumber` (format `DN-{year}-{6-digit sequence}`)
  2. Add the amount to the campaign's `RaisedAmount`
  3. If `RaisedAmount >= GoalAmount`, set the campaign to `Completed`
  4. Commit

- **DON-08:** Raised amount updates are safe under concurrency — two donations succeeding at the same time must both be counted (use `RowVersion` with retry or atomic update).

| Method | Path | Access |
|--------|------|--------|
| POST | `/admin/donations/{id}/mark-succeeded` | Admin (temporary) |

### Resources
- **MUST** — [EF Core — Saving data](https://learn.microsoft.com/en-us/ef/core/saving/)
- **MUST** — [EF Core — Concurrency](https://learn.microsoft.com/en-us/ef/core/saving/concurrency)
- **MUST** — [EF Core — Transactions](https://learn.microsoft.com/en-us/ef/core/saving/transactions)
- **VIDEO** — [Nick Chapsas — Concurrency Conflicts in EF Core](https://www.youtube.com/watch?v=pM9mYcEw28Q)
- **VIDEO** — [Milan Jovanović — Optimistic Concurrency in EF Core](https://www.youtube.com/watch?v=bF9EwE0_2H4)

---

### Task 4 — Pagination, Filtering, Sorting & Searching

Implement reusable API querying features:

- **CMP-08:** Public campaign list supports pagination, filtering by category and status, search in title (current language), and sorting by end date, raised amount, or creation date.
- Admin donation list supports pagination, filtering by status, campaign, donor, and date range.
- **Paged response shape:** `{ "items": [], "page": 1, "pageSize": 20, "totalCount": 0, "totalPages": 0 }`
- Default page size 20, maximum 50.
- Deterministic ordering required.
- Do not allow arbitrary SQL/order fragments from user input.

### Resources
- **MUST** — [EF Core — Efficient querying](https://learn.microsoft.com/en-us/ef/core/performance/efficient-querying)
- **VIDEO** — [Milan Jovanović — Adding Filtering, Sorting, and Pagination](https://www.youtube.com/watch?v=X8zRvXbirMU)
- **ARTICLE** — [Code Maze — Pagination in ASP.NET Core Web API](https://code-maze.com/paging-aspnet-core-webapi/)

---

### Task 5 — Caching

Implement caching using the cache-aside pattern:

- **CMP-09:** Public reads are cached and the cache is invalidated when a campaign or category changes.
- Start with **in-memory cache**, then switch to `IDistributedCache` backed by **Redis**.
- Cache: categories list, public campaign list pages, campaign details.
- Cache keys must include **language and query parameters**.
- Invalidate related keys when data changes.
- **Never cache user-specific data** (own donations) under a shared key.
- Document the invalidation strategy in the README.

### Resources
- **MUST** — [ASP.NET Core — In-memory caching](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory)
- **MUST** — [ASP.NET Core — Distributed caching](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed)
- **MUST** — [Redis documentation](https://redis.io/docs/latest/)
- **VIDEO** — [Nick Chapsas — Distributed Caching with Redis](https://www.youtube.com/watch?v=7_E9q4hZp8w)
- **VIDEO** — [Milan Jovanović — Cache-Aside Pattern with Redis](https://www.youtube.com/watch?v=sW2wN1n3z6U)

---

### Task 6 — Localization

Implement bilingual support:

- **CMP-11:** Public responses use the language of the `Accept-Language` header (`ar` or `en`), fallback `en`. Admin and manager endpoints return both languages.
- Localize: validation messages, error messages.
- Texts live in **resource files**, not hard-coded strings.
- Data fields stay bilingual (Ar and En columns). Do not translate stored data automatically.

### Resources
- **MUST** — [ASP.NET Core — Globalization and localization](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization)
- **VIDEO** — [Mohamad Lawand — Localization and Globalization in Web API](https://www.youtube.com/watch?v=aG49R_s4y_U)
- **ARTICLE** — [Code Maze — ASP.NET Core Web API Localization](https://code-maze.com/aspnetcore-web-api-localization/)

---

### Task 7 — API Versioning

Implement URL-segment versioning:

- All routes under `/api/v1/...`
- Swagger shows one document per version

### Resources
- **MUST** — [ASP.NET Core API versioning](https://learn.microsoft.com/en-us/aspnet/core/web-api/advanced/versioning)
- **VIDEO** — [Nick Chapsas — API Versioning in ASP.NET Core (.NET 8)](https://www.youtube.com/watch?v=2wOCkNm1_zY)

---

### Task 8 — Rate Limiting

Use the built-in ASP.NET Core rate limiter with named policies configured through options:

| Policy | Applies to | Default |
|--------|-----------|---------|
| `global` | All endpoints, per IP | 100 requests / minute |
| `auth` | Login, register, refresh | 10 requests / minute per IP |
| `donations` | Donation creation and checkout | 20 requests / minute per user |

- Webhook endpoint is **excluded** from rate limiting.
- Rejected requests return 429 with `Retry-After` header and ProblemDetails body.

### Resources
- **MUST** — [ASP.NET Core — Rate limiting middleware](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit)
- **VIDEO** — [Nick Chapsas — Built-in Rate Limiting in .NET](https://www.youtube.com/watch?v=9_p_H6c0zXQ)
- **VIDEO** — [Milan Jovanović — Rate Limiting in ASP.NET Core](https://www.youtube.com/watch?v=6l7Z1c2k4n8)

---

### Task 9 — CORS

Implement a named CORS policy:

- Allowed origins come from options (`CorsOptions`)
- **Never** `AllowAnyOrigin` together with credentials

### Resources
- **MUST** — [ASP.NET Core — Enable CORS](https://learn.microsoft.com/en-us/aspnet/core/security/cors)

---

### Task 10 — Unit Tests for Donations

Add unit tests for:

- Donation creation invariants (amount range, campaign must be active, end date not passed)
- Success flow (raised amount updated, campaign completed when goal reached)
- Concurrency handling
- Ownership rules (donor cannot access another donor's donation)

### Resources
- **MUST** — [xUnit](https://xunit.net/)
- **MUST** — [Microsoft — Testing in .NET](https://learn.microsoft.com/en-us/dotnet/core/testing/)
- **VIDEO** — [Teddy Smith — Unit Testing in ASP.NET Core](https://www.youtube.com/watch?v=fVVNOkbLcPw)

---

## Sprint Deliverables

- [ ] Authorization policies with resource ownership
- [ ] Campaign Manager can manage only own campaigns
- [ ] Donations module (DON-01 to DON-08)
- [ ] Temporary `mark-succeeded` stub
- [ ] Success flow in one transaction
- [ ] Concurrency-safe raised amount update
- [ ] Pagination, filtering, sorting, searching on public and admin lists
- [ ] In-memory cache then Redis caching with invalidation
- [ ] Localization (ar/en) for public responses, validation, and errors
- [ ] API versioning (`/api/v1/`)
- [ ] Rate limiting policies
- [ ] CORS configuration
- [ ] Unit tests for donation business rules
- [ ] Cache invalidation strategy documented in README

## Definition of Done

- [ ] Campaign Manager ownership enforced
- [ ] 404 returned for unauthorized private resources
- [ ] Donation creation works with invariant checks
- [ ] Own donations listed and accessible
- [ ] Admin donation list with filters works
- [ ] Mark-succeeded stub works (temporary)
- [ ] Success flow transactional and concurrency-safe
- [ ] Receipt number generated
- [ ] Pagination works with deterministic ordering
- [ ] Filtering works
- [ ] Sorting works
- [ ] Searching works
- [ ] Cache-aside implemented
- [ ] Cache keys include language and query params
- [ ] Cache invalidated on writes
- [ ] Public responses localized by Accept-Language
- [ ] Validation messages localized
- [ ] API versioned at `/api/v1/`
- [ ] Rate limiting enforced
- [ ] CORS configured from options
- [ ] New unit tests added for donations
- [ ] Learner can explain caching, authorization, and concurrency decisions

## Review Rule

The learner should be able to explain not only **what** was implemented, but **why it belongs there, what alternatives existed, and what trade-off was accepted**.