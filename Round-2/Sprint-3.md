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

Advance DonationHub beyond basic CRUD by implementing the **Donations Core** module backed by atomic **EF Core Database Transactions** to protect financial consistency. Enforce fine-grained **Resource-Based Authorization** (ownership rules), and equip the platform with production-grade API features: distributed caching with **Redis** via Docker, bilingual **Localization** (ar-EG / en-US), **API Versioning**, **Rate Limiting** to prevent abuse, and strict **CORS** policies.

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

### Task 1 — Resource-Based Authorization & Ownership Policies

Implement ownership rules and access boundaries according to SRS requirement **CMP-07**:
- `CampaignManager` can create campaigns and modify or delete **only their own campaigns**.
- `Admin` has unrestricted access to manage all campaigns across the system.
- Implement `CampaignOwnershipRequirement : IAuthorizationRequirement`.
- Implement `CampaignOwnershipHandler : AuthorizationHandler<CampaignOwnershipRequirement, Campaign>` utilizing `IAuthorizationService`.
- **Security Rule:** When an authenticated user attempts to fetch or modify a private resource owned by another user, return **404 Not Found** (not 403 Forbidden) to avoid revealing the existence of the resource ID to unauthorized parties.

| Endpoint | Campaign Manager | Admin |
|---|---|---|
| `POST /campaigns` | ✅ Creates owned campaign | ✅ Creates campaigns |
| `PUT /campaigns/{id}` | ✅ Own campaigns only | ✅ All campaigns |
| `DELETE /campaigns/{id}` | ✅ Own campaigns only | ✅ All campaigns |
| `GET /campaigns/mine` | ✅ Fetches own campaigns | — |

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Resource-based authorization — Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased).
  - ⭐2 [Who Owns This Order? — Code Story (01:46)](https://youtu.be/29x5WmcUNtM) (Building handler and invoking `AuthorizeAsync`).
- **🔄 Alternative / Deep Dive:**
  - [ASP.NET Web API 8 Identity — Fullstack Dev (Resource authorization timestamp at 02:19:44)](https://youtu.be/tXUtSGvHMeg)
  - [14- Create Policy Part 1 — Mahmoud Hamza [Arabic]](https://youtu.be/tdMrNKPO39U)
  - [Custom Authorization Handler — Milan Jovanović](https://youtu.be/SZtZuvcMBA0)

---

### Task 2 — Donations Core & Atomic Database Transactions

Implement donation flow requirements from SRS §3.4:
- **DON-01:** Submit donation `POST /campaigns/{id}/donations` with amount, donor display name, optional message, and anonymous flag (`IsAnonymous`). Initial status is `Pending`.
- **DON-02:** Enforce amount boundaries configured in `DonationOptions` (e.g., Min $5, Max $50,000).
- **DON-03:** Reject donations targeting non-active campaigns (`Completed` or `Expired`).
- **DON-04:** Provide a temporary payment stub endpoint `POST /donations/{id}/pay-stub` to simulate successful completion prior to real Stripe integration in Sprint 5.
- **DON-05 & DON-06 (ACID Consistency):** Upon payment success:
  1. Update donation status to `Succeeded` and record payment timestamp.
  2. Increment campaign raised amount: `RaisedAmount += donation.Amount`.
  3. If `RaisedAmount >= GoalAmount`, update campaign status to `Completed`.
  4. Wrap these updates inside an atomic **EF Core Database Transaction** via `await dbContext.Database.BeginTransactionAsync()` to prevent partial writes during runtime failures.

**Resources:**
- **⭐ Required Primary:** [#020 DB Transaction EF Core — Essam Abdelnabi [Arabic] (32:57)](https://youtu.be/m2RDSxGPuhs) (BeginTransaction at 19:15, Rollback at 24:40, Savepoints at 28:50).
- **🔄 Alternative / Deep Dive:**
  - [Transactions in EF Core — Milan Jovanović](https://youtu.be/25H84BXcr9M)
  - [Avoid EF Core Transaction Mistakes — Milan](https://youtu.be/cUszBUYSFzw)

---

### Task 3 — Pagination, Filtering & Sorting

Implement scalable querying across public campaign and donation endpoints:
- Create a reusable `PagedList<T>` model: `Items`, `PageNumber`, `PageSize`, `TotalCount`, `TotalPages`, `HasPreviousPage`, `HasNextPage`.
- Perform pagination at the database level via `Skip()` and `Take()` on `IQueryable` before executing the query.
- Support dynamic filtering: category (`categoryId`), status (`status`), and date/amount ranges.
- Support safe dynamic sorting by created date, goal amount, or raised amount.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [15. Pagination in Web Api Part 1 — Coding-Future (27:51)](https://youtu.be/XDkUliEyPms).
  - ⭐2 [Filtering, Sorting And Pagination — Milan Jovanović (24:02)](https://youtu.be/X8zRvXbirMU) (Filtering with Where at 03:25, Sorting at 08:20, PagedList at 18:00).
- **🔄 Alternative / Deep Dive:**
  - [16. Pagination Part 2 — Coding-Future](https://youtu.be/8vum4WH8PVs)
  - [Paging Math and Implementation — Essam Abdelnabi (Timestamp 56:35)](https://youtu.be/fb8j6CjyWWw)

---

### Task 4 — Distributed Caching with Redis & IDistributedCache

- Run a local **Redis** container in Docker and configure `AddStackExchangeRedisCache`.
- Inject `IDistributedCache` to cache high-frequency read queries:
  - Cache active campaign list and success stories.
  - Cache campaign details `GET /campaigns/{id}` with sliding and absolute expiration.
- **Cache Invalidation:** Evict or refresh cache entries immediately when a campaign is updated or a donation succeeds to guarantee immediate financial accuracy.

**Resources:**
- **⭐ Required Primary:** [Caching Explained: In-Memory vs Redis — Amr Mohamed [Arabic] (21:11)](https://youtu.be/AIBOcH_dAX8).
- **🔄 Alternative / Deep Dive:** [Distributed Caching With Redis in ASP.NET Core — Milan Jovanović](https://youtu.be/Tt5zIKVMMbs)

---

### Task 5 — Localization Support (ar-EG / en-US)

- Configure ASP.NET Core localization using `IStringLocalizer` and `.resx` resource files.
- Automatically resolve client language via `Accept-Language: ar-EG` or `en-US` request header.
- Localize validation and domain error catalogue messages.
- Return appropriate localized fields (`Name`, `Description`) in public DTOs based on the requested culture.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Globalization and Localization Part 1 — Mohamed El-Helaly [Arabic] (08:38)](https://youtu.be/PyCGEO4b6F0).
  - ⭐2 [16. Use Localization In APIs — Mohamed El-Helaly [Arabic] (04:37)](https://youtu.be/YzOarEJDdrY).
- **🔄 Alternative / Deep Dive:** [Microsoft Learn — Localization in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization)

---

### Task 6 — API Versioning & Rate Limiting

- **API Versioning:** Implement URI-based versioning (e.g., `/api/v1/...`) using `Asp.Versioning.Mvc`.
- **Rate Limiting:** Protect sensitive endpoints (donations, authentication) from brute force and denial of service using built-in ASP.NET Core Rate Limiting (e.g., sliding window of 5 requests/minute per client IP).
- Return standard **429 Too Many Requests** when the threshold is exceeded.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [.NET 9: ASP.NET Core Web API Versioning — Mohamad Lawand (12:55)](https://youtu.be/DkZd-5z0CeI).
  - ⭐2 [How To Implement Rate Limiting In .NET — Milan Jovanović (09:42)](https://youtu.be/1tPVVDEDGtE).
- **🔄 Alternative / Deep Dive:** [Rate Limiting Algorithms — Mohamad Lawand](https://youtu.be/bOfOo3Zsfx0)

---

### Task 7 — CORS Policy Configuration

- Configure Cross-Origin Resource Sharing (CORS) in `Program.cs`.
- Define explicit allowed origins, HTTP methods, and headers for prospective frontend applications, avoiding wildcard `AllowAnyOrigin()` in non-development profiles.

**Resources:**
- **⭐ Required Primary:** [6. Enable CORS (Updated) — Mohamed El-Helaly [Arabic] (05:39)](https://youtu.be/_YXcxm-O06I).
- **🔄 Alternative / Deep Dive:** [Microsoft Learn — Enable CORS](https://learn.microsoft.com/en-us/aspnet/core/security/cors)

---

## 📦 Sprint Deliverables

- [ ] Resource-Based Authorization protecting private campaign manager resources (404 on unowned).
- [ ] Donations module implemented with boundary validation and stub payment processing.
- [ ] Database transaction wrapping donation settlement and campaign raised amount updates.
- [ ] Generic `PagedList<T>` with database-level pagination, sorting, and filtering.
- [ ] Redis container integrated with `IDistributedCache` and proactive cache invalidation.
- [ ] Bilingual localization (ar-EG / en-US) for error messages and public responses.
- [ ] API Versioning configured via URL path.
- [ ] Rate limiting active on sensitive endpoints returning HTTP 429 upon violation.
- [ ] Secure CORS policy configured.

---

## ✅ Definition of Done

- [ ] Redis container runs and connects seamlessly with no runtime exceptions.
- [ ] Successful donation increments campaign funds inside an atomic transaction.
- [ ] Unauthenticated requests or unauthorized access to other users' campaigns return 404.
- [ ] Queries do not load entire tables into memory; pagination uses `Skip` and `Take`.
- [ ] Header `Accept-Language: ar-EG` outputs Arabic messages, and `en-US` outputs English.
- [ ] Exceeding request rate triggers HTTP 429 Too Many Requests.