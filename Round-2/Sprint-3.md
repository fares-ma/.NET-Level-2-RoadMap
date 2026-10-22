# Sprint 3 — Donations Core, Resource Authorization & Production API Engineering

> **Project:** DonationHub — Donation Management & Crowdfunding Platform  
> **Duration:** 15 Days (22/11/2026 to 06/12/2026)  
> **Meeting Schedule:**  
> - **Stand-up 1 (Day 5):** 26/11/2026 — Session Leader: **Kareem Zein**  
> - **Stand-up 2 (Day 10):** 01/12/2026 — Session Leader: **Mohamed Atef**  
> - **Closing & Deadline (Day 15):** 06/12/2026 — Session Leader: **Asma ELhadidy**  
> - **Code Review & Corrections (Days 16–18):** 07/12/2026 to 09/12/2026  
> **Evaluation:** 100 Points  
> **Target Framework:** .NET 10 (fully compatible with .NET 8 / .NET 9)

---

## 🎯 Sprint Goal

Advance DonationHub beyond basic CRUD by implementing the **Donations Core** module backed by atomic **EF Core Database Transactions** to protect financial consistency. Enforce fine-grained **Resource-Based Authorization** (ownership rules), and equip the platform with production-grade API features: caching (in-memory first, then distributed **Redis** via Docker), bilingual **Localization** (`ar` / `en`), **API Versioning**, **Rate Limiting** to prevent abuse, and strict **CORS** policies.

---

## 📌 Sprint Prerequisites

### Docker Container Fundamentals (Mandatory for running Redis)
*Learners must be able to spin up, monitor, inspect, and stop Docker containers locally before starting Sprint 3 tasks:*
- Master commands: `docker run -d -p`, `docker ps`, `docker logs`, `docker stop`.
- Launch local Redis container:  
  `docker run -d --name redis -p 6379:6379 redis:alpine` (or open-source alternative `valkey/valkey:alpine`).

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Basic Commands in Docker Part 5 — Codographia (15:20)](https://youtu.be/cB7Qx13YOFM) (pull, run, ps, stop).
  - ⭐2 [Basic Commands Part 6 — Codographia (20:29)](https://youtu.be/w_fnmXy9ZEk) (detached mode -d, port mapping -p, viewing logs).
- **🔄 Alternative / Deep Dive:**
  - [Docker in 5 Minutes — Yehia Tech [Arabic]](https://youtu.be/yRkE-1Lq0qo)
  - [Docker For Developers Ep 1 — Ramy Hakam [Arabic]](https://youtu.be/dTZZ-4PibHg)

---

## 🛠️ Sprint Tasks

All tasks below are **CORE** unless marked BONUS.

> 📘 **Written references:** Links marked 📘 point to the free [.NET Web API Zero to Hero](https://codewithmukesh.com/courses/dotnet-webapi-zero-to-hero/) course (.NET 10 articles + source code). Its samples use **PostgreSQL** and **Minimal APIs** — apply the ideas to our **SQL Server** + **thin Controllers** setup.

### Task 1 — Resource-Based Authorization & Ownership Policies

Implement ownership rules and access boundaries according to SRS requirement **CMP-07**:
- `CampaignManager` can create campaigns and modify or delete **only their own campaigns**.
- `Admin` has unrestricted access to manage all campaigns across the system.
- Implement `CampaignOwnershipRequirement : IAuthorizationRequirement`.
- Implement `CampaignOwnershipHandler : AuthorizationHandler<CampaignOwnershipRequirement, Campaign>` utilizing `IAuthorizationService`.
- **Security Rule:** When an authenticated user attempts to fetch or modify a private resource owned by another user, return **404 Not Found** (`Campaign.NotFound` / `Donation.NotFound`, not 403 Forbidden) to avoid revealing the existence of the resource ID to unauthorized parties. Unauthenticated requests to protected endpoints return **401**.
- **Defense in depth (SRS §6.3):** Enforce authorization in the API (policies/roles) **and** check ownership again inside the handler for owned resources (campaigns and donations).
- `GET /campaigns/mine` returns the manager's own campaigns in **all statuses** (paged), with both-language fields.

| Endpoint | Campaign Manager | Admin |
|---|---|---|
| `POST /campaigns` | ✅ Creates owned campaign | ✅ Creates campaigns |
| `PUT /campaigns/{id}` | ✅ Own campaigns only | ✅ All campaigns |
| `DELETE /campaigns/{id}` | ✅ Own campaigns only | ✅ All campaigns |
| `GET /campaigns/mine` | ✅ Fetches own campaigns (all statuses) | — |

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Resource-based authorization — Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased).
  - ⭐2 [Who Owns This Order? — Code Story (01:46)](https://youtu.be/29x5WmcUNtM) (Building handler and invoking `AuthorizeAsync`).
- **🔄 Alternative / Deep Dive:**
  - [ASP.NET Web API 8 Identity — Fullstack Dev (Resource authorization timestamp at 02:19:44)](https://youtu.be/tXUtSGvHMeg)
  - [14- Create Policy Part 1 — Mahmoud Hamza [Arabic]](https://youtu.be/tdMrNKPO39U)
  - [Custom Authorization Handler — Milan Jovanović](https://youtu.be/SZtZuvcMBA0)
  - 📘 [Policy-Based Authorization in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/policy-based-authorization-in-aspnet-core/)
  - 📘 [Claims-Based Authorization in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/claims-based-authorization-in-aspnet-core/)

---

### Task 2 — Donations Core & Atomic Database Transactions

Implement donation flow requirements from SRS §5.4:
- **DON-01:** A logged-in user (Donor, Campaign Manager or Admin) creates a donation for an active campaign via `POST /campaigns/{campaignId}/donations` with body `{ "amount": 25.00 }`. The donation starts as `Pending`. Return **201 Created** with a `Location` header.
- **DON-02:** Reject the donation when the campaign is not `Active` or its `EndDate` has passed (`Campaign.NotActive`, 409), or when the amount is outside the range configured in `DonationOptions` (defaults: min **1.00**, max **10,000.00**) (`Donation.AmountOutOfRange`, 422). These are domain invariants 3 and 4.
- **DON-03:** `GET /donations/mine` — the current user lists **their own** donations (paged) with amount, date, status and campaign title.
- **DON-04:** `GET /donations/{id}` — the owner (or an Admin) gets one donation by id. Accessing someone else's donation returns **404** `Donation.NotFound` (do not reveal that it exists).
- **DON-05:** `GET /admin/donations` (Admin only) — list all donations, paged, with filters: `status`, `campaignId`, `donorId`, and date range (`from`, `to`).
- **DON-06 (Temporary stub, Sprints 3–4):** `POST /admin/donations/{id}/mark-succeeded` (**Admin only**) simulates a successful payment so the full flow can be tested before Stripe. A donation that is not `Pending` returns `Donation.InvalidState` (409). **This endpoint is removed in Sprint 5.**
- **DON-07 (Success flow in one transaction — SRS §5.5):** Marking a donation succeeded runs, inside **one database transaction** (e.g. `await dbContext.Database.BeginTransactionAsync()` or a single `SaveChangesAsync` through the Unit of Work):
  1. Set donation status to `Succeeded`, set `CompletedAt` (from `TimeProvider`), and generate `ReceiptNumber` in the format `DN-{year}-{6-digit sequence}` (e.g. `DN-2026-000042`). The sequence must be **unique under concurrent success flows** (database sequence, or a unique index with retry — SRS §9.4).
  2. Add the amount to the campaign's `RaisedAmount`.
  3. If `RaisedAmount >= GoalAmount`, set the campaign status to `Completed`.
  4. Commit. (From Sprint 4, background jobs are enqueued only **after** the commit, never inside the transaction.)
  - **Late-payment edge case:** If a payment succeeds after the campaign already expired or completed, **accept the money and record the donation as `Succeeded`** (the donor was already charged) and log a warning with `ILogger`. Refunds are BONUS.
- **DON-08 (Concurrency-safe `RaisedAmount`):** Two donations succeeding at the same time must **both** be counted. Use the campaign `RowVersion` with a retry on `DbUpdateConcurrencyException`, or an atomic database update (`ExecuteUpdateAsync(... RaisedAmount + amount ...)`). Be ready to explain your choice.
- **Unit tests (SRS §6.13):** Add unit tests for the new donation invariants (amount range, campaign must be active, final states cannot change, campaign completes when goal is reached) and for the donation handlers (create, get own / 404 for others, mark-succeeded), with the repository and `TimeProvider` faked.

| Method | Path | Access |
|---|---|---|
| `POST` | `/campaigns/{campaignId}/donations` | Donor, Campaign Manager, Admin |
| `GET` | `/donations/mine` | Any authenticated user |
| `GET` | `/donations/{id}` | Owner, Admin |
| `GET` | `/admin/donations` | Admin |
| `POST` | `/admin/donations/{id}/mark-succeeded` | Admin (temporary, removed in Sprint 5) |

**Resources:**
- **⭐ Required Primary:** [#020 DB Transaction EF Core — Essam Abdelnabi [Arabic] (32:57)](https://youtu.be/m2RDSxGPuhs) (BeginTransaction at 19:15, Rollback at 24:40, Savepoints at 28:50).
- **🔄 Alternative / Deep Dive:**
  - [Transactions in EF Core — Milan Jovanović](https://youtu.be/25H84BXcr9M)
  - [Avoid EF Core Transaction Mistakes — Milan](https://youtu.be/cUszBUYSFzw)

---

### Task 3 — Pagination, Filtering & Sorting

Implement scalable querying across public and admin list endpoints (`GET /campaigns`, `GET /campaigns/success-stories`, `GET /campaigns/mine`, `GET /donations/mine`, `GET /admin/donations`):
- Create a reusable `PagedList<T>` returned with the SRS §6.8 shape: `{ "items": [], "page": 1, "pageSize": 20, "totalCount": 0, "totalPages": 0 }`. Default page size **20**, maximum **50**.
- Perform pagination at the database level via `Skip()` and `Take()` on `IQueryable` before executing the query.
- **CMP-08 (public campaign list):**
  - Search in the campaign title in the **current request language** (`search` → `TitleAr` or `TitleEn`).
  - Filter by category (`categoryId`) and status (`status`).
  - Safe dynamic sorting (whitelist only) by **end date**, **raised amount**, or **creation date**, ascending or descending.
- Admin donation filters (DON-05): `status`, `campaignId`, `donorId`, `from`, `to`.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [15. Pagination in Web Api Part 1 — Coding-Future (27:51)](https://youtu.be/XDkUliEyPms).
  - ⭐2 [Filtering, Sorting And Pagination — Milan Jovanović (24:02)](https://youtu.be/X8zRvXbirMU) (Filtering with Where at 03:25, Sorting at 08:20, PagedList at 18:00).
- **🔄 Alternative / Deep Dive:**
  - [16. Pagination Part 2 — Coding-Future](https://youtu.be/8vum4WH8PVs)
  - [Paging Math and Implementation — Essam Abdelnabi (Timestamp 56:35)](https://youtu.be/fb8j6CjyWWw)
  - 📘 [Pagination, Sorting & Searching in ASP.NET Core Web API — codewithmukesh](https://codewithmukesh.com/blog/pagination-sorting-searching-aspnet-core-webapi/)

---

### Task 4 — Caching: In-Memory First, Then Redis & IDistributedCache (CMP-09)

- **Step 1 — In-memory:** Implement the cache-aside pattern with `IMemoryCache` behind your own caching abstraction in Application.
- **Step 2 — Distributed:** Switch the implementation to `IDistributedCache` backed by a local **Redis** container (`AddStackExchangeRedisCache`), with durations from `CacheOptions`.
- Cache high-frequency public reads (SRS §6.6):
  - Categories list.
  - Public campaign list pages and success stories.
  - Campaign details `GET /campaigns/{id}` with sliding and absolute expiration.
- **Key naming:** Keys must include the **language** and the **query parameters** (e.g. `campaigns:list:en:page=1:size=20:cat=…:status=…:search=…:sort=…`), otherwise Arabic users receive English data.
- **Cache Invalidation (CMP-09):** Evict related keys immediately when a **campaign** changes (create, update, delete, status change), when a **category** changes, and when a donation succeeds (raised amount changed). Document your invalidation strategy in the README.
- **Never** cache user-specific data (e.g. `GET /donations/mine`) under a shared key.

**Resources:**
- **⭐ Required Primary:** [Caching Explained: In-Memory vs Redis — Amr Mohamed [Arabic] (21:11)](https://youtu.be/AIBOcH_dAX8).
- **🔄 Alternative / Deep Dive:**
  - [Distributed Caching With Redis in ASP.NET Core — Milan Jovanović](https://youtu.be/Tt5zIKVMMbs)
  - 📘 [In-Memory Caching in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/in-memory-caching-in-aspnet-core/) (Step 1)
  - 📘 [Distributed Caching in ASP.NET Core with Redis — codewithmukesh](https://codewithmukesh.com/blog/distributed-caching-in-aspnet-core-with-redis/) (Step 2)

---

### Task 5 — Localization Support (ar / en)

- Configure ASP.NET Core localization using `IStringLocalizer` and `.resx` resource files. Texts live in resource files, not hard-coded strings in handlers.
- Supported cultures: `en` (**default**) and `ar`, resolved from the `Accept-Language: ar` or `Accept-Language: en` request header, with fallback to `en`.
- Localize validation messages and error catalogue messages (the `Message` of each `Error`).
- **CMP-11:** Public campaign and category responses return the localized fields (`Title`, `Name`, `Description`) for the requested culture. Admin and manager endpoints return both languages. Stored data stays bilingual (Ar/En columns) and is never auto-translated.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Globalization and Localization Part 1 — Mohamed El-Helaly [Arabic] (08:38)](https://youtu.be/PyCGEO4b6F0).
  - ⭐2 [16. Use Localization In APIs — Mohamed El-Helaly [Arabic] (04:37)](https://youtu.be/YzOarEJDdrY).
- **🔄 Alternative / Deep Dive:** [Microsoft Learn — Localization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization)

---

### Task 6 — API Versioning & Rate Limiting

- **API Versioning:** Implement URL-segment versioning (`/api/v1/...`) using `Asp.Versioning.Mvc`. The OpenAPI documentation (Scalar) shows **one document per API version**.
- **Rate Limiting (SRS §6.4):** Use the built-in ASP.NET Core rate limiter with **named policies** whose limits come from `RateLimitOptions`:

| Policy | Applies to | Default |
|---|---|---|
| `global` | All endpoints, partitioned per client IP | 100 requests / minute |
| `auth` | `login`, `register`, `refresh` | 10 requests / minute per IP |
| `donations` | Donation creation (and checkout from Sprint 5) | 20 requests / minute per user |

- The Stripe webhook endpoint (Sprint 5) is **excluded** from rate limiting.
- Rejected requests return **429 Too Many Requests** with a `Retry-After` header and a ProblemDetails body (`code: Common.RateLimited`).

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [.NET 9: ASP.NET Core Web API Versioning — Mohamad Lawand (12:55)](https://youtu.be/DkZd-5z0CeI).
  - ⭐2 [How To Implement Rate Limiting In .NET — Milan Jovanović (09:42)](https://youtu.be/1tPVVDEDGtE).
- **🔄 Alternative / Deep Dive:**
  - [Rate Limiting Algorithms — Mohamad Lawand](https://youtu.be/bOfOo3Zsfx0)
  - 📘 [API Versioning in ASP.NET Core (.NET 10, Asp.Versioning + Scalar) — codewithmukesh](https://codewithmukesh.com/blog/api-versioning-in-aspnet-core/)
  - 📘 [Rate Limiting in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/rate-limiting-aspnet-core/)

---

### Task 7 — CORS Policy Configuration

- Configure Cross-Origin Resource Sharing (CORS) in `Program.cs` with **one named policy**.
- Allowed origins come from `CorsOptions` (validated on startup), together with explicit HTTP methods and headers for prospective frontend applications.
- **Never** combine `AllowAnyOrigin()` with `AllowCredentials()`.

**Resources:**
- **⭐ Required Primary:** [6. Enable CORS (Updated) — Mohamed El-Helaly [Arabic] (05:39)](https://youtu.be/_YXcxm-O06I).
- **🔄 Alternative / Deep Dive:**
  - [Microsoft Learn — Enable CORS](https://learn.microsoft.com/en-us/aspnet/core/security/cors)
  - 📘 [CORS in ASP.NET Core (.NET 10) — Cross-Origin Done Right — codewithmukesh](https://codewithmukesh.com/blog/cors-in-aspnet-core/) (preflight, config-driven origins, and the `AllowAnyOrigin` + `AllowCredentials` trap)

---

## 📦 Sprint Deliverables

- [ ] Resource-Based Authorization protecting private campaign manager resources (CMP-07, 404 on unowned) and `GET /campaigns/mine`.
- [ ] Donations module DON-01 to DON-08 with boundary validation, own/admin lists, and the Admin-only `mark-succeeded` stub.
- [ ] Database transaction wrapping the success flow (status, `ReceiptNumber`, raised amount, campaign completion), safe under concurrency.
- [ ] Generic `PagedList<T>` with database-level pagination, search, sorting, and filtering (CMP-08) on public and admin lists.
- [ ] In-memory cache first, then Redis via `IDistributedCache`, with language-aware keys and invalidation on campaign/category changes (CMP-09).
- [ ] Bilingual localization (`ar` / `en`) for validation messages, error messages and public responses.
- [ ] API Versioning configured via URL path, with one OpenAPI document per version.
- [ ] Named rate limiting policies (`global`, `auth`, `donations`) from `RateLimitOptions`, returning HTTP 429 with `Retry-After`.
- [ ] Named CORS policy with origins from `CorsOptions`.
- [ ] Unit tests for the new donation invariants and handlers.

---

## ✅ Definition of Done

- [ ] Redis container runs and connects seamlessly with no runtime exceptions.
- [ ] Successful donation increments campaign funds inside an atomic transaction, and two concurrent successes are both counted.
- [ ] Each succeeded donation has a unique `ReceiptNumber` in the format `DN-{year}-{6 digits}`.
- [ ] Unauthenticated requests to protected endpoints return **401**; requesting another user's resource (a donation, or another manager's campaign) returns **404**.
- [ ] Queries do not load entire tables into memory; pagination uses `Skip` and `Take` and returns the `{ items, page, pageSize, totalCount, totalPages }` shape.
- [ ] Header `Accept-Language: ar` outputs Arabic messages and fields, `Accept-Language: en` (or no header) outputs English.
- [ ] Exceeding a request rate triggers HTTP 429 Too Many Requests with a `Retry-After` header.
- [ ] Unit tests for the new donation invariants and handlers pass with `dotnet test`.
- [ ] The checkpoint tag `sprint-3` exists and points to the final commit.
- [ ] The README is updated for the sprint (what was added, how to try it, decisions), including the **cache invalidation strategy**.
- [ ] No secrets in the repository history (connection strings and keys live in user-secrets or environment variables, not `appsettings`).

---

## ⭐ BONUS (optional — counts only after all CORE items pass)

- **DON-11:** Anonymous donation flag (`IsAnonymous`) — the donor's name is hidden from public displays.
- **DON-12:** Public list of recent donors of a campaign (respecting the anonymous flag).
- **CMP-12:** Campaign `Cancelled` status set by Admin or the owner.
- **CMP-13:** Specification pattern for the campaign list filters.
- **API v2 of one endpoint** (SRS §6.8), shown as a separate OpenAPI document.

---

## 📤 Submission

- Submit your repository URL on **tawjeh.net** before **23:59 on Day 15 (06/12/2026)**.
- Tag the final commit and push the tag:
  ```bash
  git tag sprint-3
  git push origin sprint-3
  ```
- Fixes are allowed during the **Code Review & Corrections window (07/12/2026 to 09/12/2026)**. Do not move or force-push `sprint-3`; tag the fixed commit `sprint-3-fix` and push it (`git tag sprint-3-fix && git push origin sprint-3-fix`).

---

## 🧮 Grading (100 Points)

| Part | Points | How |
|---|---|---|
| Automated | 40 | Build succeeds, `dotnet test` passes, the `sprint-3` tag exists, CORE endpoints respond as specified (smoke check by the mentor) |
| Checklist | 40 | The Definition of Done above (structure, rules, security, README) |
| Explanation | 20 | A short question on one of your design decisions (e.g. how you made `RaisedAmount` concurrency-safe), or a code review of the decisions written in your README |

BONUS items add capped extra credit on top of the 100 points, only when all CORE items pass.

---

## 💡 Post-Sprint Enrichment

- [Cursor-Based Pagination in .NET](https://youtu.be/U3LcKY19z_4)
- [Distributed Caching with Redis — Mohamad Lawand](https://youtu.be/6HZVu3kGOrg)
- [Minimal APIs vs Controllers in ASP.NET Core — Milan Jovanović](https://youtu.be/gsAuFIhXz3g)
- [Minimal API Versioning in ASP.NET Core](https://youtu.be/F9j3X6KuIpw)
- [Building High-Performance APIs with FastEndpoints — Nick Chapsas](https://youtu.be/oqNu0xavAyc)
- 📘 [HybridCache in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/hybridcache-in-aspnet-core/)
- 📘 [Permission-Based Authorization in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/permission-based-authorization-in-aspnet-core/)
- 📘 [Filtering by a Large List — Contains and the 2,100 Parameter Wall — codewithmukesh](https://codewithmukesh.com/blog/ef-core-contains-large-list/)