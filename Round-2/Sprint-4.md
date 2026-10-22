# Sprint 4 — Hangfire, MailKit, QuestPDF, File Security & Structured Logging

> **Project:** DonationHub — Donation Management & Crowdfunding Platform  
> **Duration:** 15 Days (07/12/2026 to 21/12/2026)  
> **Meeting Schedule:**  
> - **Stand-up 1 (Day 5):** 11/12/2026 — Session Leader: **Mohamed Atef**  
> - **Stand-up 2 (Day 10):** 16/12/2026 — Session Leader: **Asma ELhadidy**  
> - **Closing & Deadline (Day 15):** 21/12/2026 — Session Leader: **Kareem Zein**  
> - **Code Review & Corrections (Days 16–18):** 22/12/2026 to 24/12/2026  
> **Evaluation:** 100 Points  
> **Target Framework:** .NET 10 (fully compatible with .NET 8 / .NET 9)

---

## 🎯 Sprint Goal

Eliminate heavy, blocking operations from the HTTP request pipeline. When a donation completes, offload receipt generation and email notifications to background workers using **Hangfire**. Generate professional PDF receipts using **QuestPDF**, send transactional emails via **MailKit** tested locally against a **Mailpit** container, enforce security on campaign file uploads via **Magic Bytes verification**, schedule delayed and recurring maintenance jobs, and configure structured logging using **Serilog** (console + rolling file; **Seq** is an optional BONUS sink).

```text
HTTP Donation Completed (success flow transaction)
           ↓
Transaction Commits
           ↓
Enqueue Hangfire Job (Fire-and-Forget) — only AFTER the commit
           ↓
Immediate HTTP 200 OK Returned to Client
           ↓ (Background Worker)
Generate PDF Receipt via QuestPDF
           ↓
Send Email with Attachment via MailKit (Captured in Mailpit)
```

---

## 📌 Sprint Prerequisites

### Local SMTP Testing via Mailpit in Docker
*To test email delivery locally without leaking test emails to real inboxes, learners must run a local Mailpit container:*
- Execute command:  
  `docker run -d --name mailpit -p 8025:8025 -p 1025:1025 axllent/mailpit`
- SMTP Server runs on `localhost:1025` (no TLS in development).
- Web Dashboard accessible at `http://localhost:8025` to inspect rendered emails and attachments.

---

## 📋 Engineering Decisions & Licensing

| Package / Tool | License / Configuration | Notes & Rules |
|---|---|---|
| **QuestPDF** | Community License | `QuestPDF.Settings.License = LicenseType.Community;` in `Program.cs`. Free for individuals, small businesses, education, and open-source projects. |
| **Hangfire** | LGPL v3 Free Core | Use `Hangfire.Core`, `Hangfire.AspNetCore`, and `Hangfire.SqlServer`. Avoid paid commercial Pro extensions. |
| **Mailpit** | MIT | Container `axllent/mailpit` for zero-risk local email interception. |
| **Serilog** | Apache 2.0 | CORE sinks: Console + rolling file. Structured logging enriched with `traceId` and the user id (when authenticated). |
| **Seq** *(BONUS)* | Free developer instance | Optional extra Serilog sink. Not required for CORE. |
| **Secrets** | User secrets / environment variables | SMTP credentials and `HangfireDashboardOptions` credentials must **never** be committed in `appsettings*.json`. |

---

## 🛠️ Sprint Tasks

All tasks below are **CORE** unless marked BONUS.

### Task 1 — Hangfire Infrastructure & Protected Dashboard

Implement background job orchestration according to SRS §5.8:
- Configure Hangfire with SQL Server persistent storage: `UseSqlServerStorage`.
- Register the Hangfire server in `Program.cs`: `builder.Services.AddHangfireServer()`.
- **JOB-05 (Protected Dashboard):** Protect the Hangfire dashboard endpoint `/hangfire`. Do not leave it open to the public. A browser cannot send a JWT, so use **basic authentication** with credentials bound from `HangfireDashboardOptions` (validated with `ValidateOnStart()`, values from user secrets / environment variables). An ASP.NET Core policy requiring the `Admin` role is an acceptable alternative.
- **Retries (NTF-04):** Every job retries automatically **at least 3 times** (e.g., `[AutomaticRetry(Attempts = 3)]` or a global filter), and each failure is logged at `Error` level.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [1. Prepare Your Project — Mohamed El-Helaly [Arabic] (10:42)](https://youtu.be/hnK1qn6ZjKo).
  - ⭐2 [2. Enqueue Jobs — El-Helaly (06:07)](https://youtu.be/uqWRHmPm30E).
  - ⭐3 [3. Scheduled Jobs — El-Helaly (04:28)](https://youtu.be/_3C9BVwwM80).
  - ⭐4 [4. Recurring Jobs — El-Helaly (06:19)](https://youtu.be/GjsJrMtNES0).
  - ⭐5 [Hangfire Dashboard Authorization Documentation](https://docs.hangfire.io/en/latest/configuration/using-dashboard.html).
- **🔄 Alternative / Deep Dive:** [Hangfire in .NET Core (Complete Guide with Dashboard Auth at 15:09) — Coding Droplets (16:17)](https://youtu.be/lsP5ZjcFtMM)

---

### Task 2 — Transactional Email Service via MailKit & Mailpit

Implement the notifications module according to SRS §5.6:
- **NTF-01:** Define `IEmailService` in `DonationHub.Application` and implement it in `DonationHub.Infrastructure` using **MailKit** and **MimeKit**, with SMTP settings from `EmailOptions` (Host, Port, SenderEmail, SenderName; credentials from user secrets / environment variables).
- **NTF-02:** Email the receipt to the donor after a successful donation, with the PDF receipt attached (sent by JOB-01).
- **NTF-03:** Email the campaign creator when the campaign reaches its goal (sent by JOB-02).
- **NTF-04:** Emails are sent **only by Hangfire jobs, never inside the HTTP request**. A failed job retries (at least 3 attempts) and logs the failure.
- **NTF-05:** Email text uses the recipient's `PreferredLanguage` (`ar` or `en`). Arabic emails render right-to-left (`<html dir="rtl" lang="ar">`).
- **NTF-06:** In Development, emails go to the local Mailpit container (`localhost:1025`); no real email is sent.
- Send HTML-formatted transactional emails with optional attachments.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Sending Emails With .NET Core Part 1 — Mohamed El-Helaly [Arabic] (10:33)](https://youtu.be/J23_c3uSyD0).
  - ⭐2 [Mailpit Docker Documentation](https://mailpit.axllent.org/docs/install/docker/).
- **🔄 Alternative / Deep Dive:**
  - [Send Emails with MailKit — tutorialsEU](https://youtu.be/lCHKwyekbT4)
  - [Easy Email Verification in .NET: FluentEmail + Papercut — Milan Jovanović (local SMTP concept at 04:50–06:45; uses Papercut, not Mailpit)](https://youtu.be/KtCjH-1iCIk)

---

### Task 3 — Dynamic PDF Receipt Generation via QuestPDF

Implement donor receipt generation according to SRS §5.7 (FIL-06, FIL-07):
- Configure QuestPDF with Community license: `QuestPDF.Settings.License = LicenseType.Community;`.
- **FIL-06:** Generate a PDF receipt for each succeeded donation **inside a Hangfire job** (never in the request). The receipt contains, in the **donor's `PreferredLanguage`**:
  - Receipt Number (`DN-{year}-{6-digit sequence}`, generated in the success flow) and Transaction Date.
  - Campaign Title (in the donor's language) and Category.
  - Donor Name. *(BONUS only, if DON-11 is implemented: show "Anonymous Donor" when `IsAnonymous = true`.)*
  - Formatted Amount and Currency.
  - Organization verification header and footer.
- Generate the receipt as a byte array / stream and store it through `IFileStorage` so it can be downloaded later.
- **FIL-07:** `GET /donations/{id}/receipt` downloads the PDF (`application/pdf`), **only for the donation owner or an Admin**. Another user's donation returns 404.

| Method | Path | Access |
|---|---|---|
| `GET` | `/donations/{id}/receipt` | Owner, Admin |

**Resources:**
- **⭐ Required Primary:** [QuestPDF - The BEST PDF Generator for .NET — Claudio Bernasconi (12:07)](https://youtu.be/T89A_7dz1P8).
- **🔄 Alternative / Deep Dive:**
  - [Generate PDF Invoice in ASP.NET Core — M Alvin Maulana](https://youtu.be/eiry9Zli5Rc)
  - [QuestPDF Official Invoice Tutorial](https://www.questpdf.com/quick-start.html)

---

### Task 4 — Secure File Uploads & Magic Bytes Validation

Implement campaign banner uploads according to SRS §5.7:
- **FIL-01:** Upload a campaign banner via `IFormFile` (`multipart/form-data`). Allowed only for an **Admin or the campaign's owner** (Campaign Manager). Allowed types: JPG, PNG, WEBP. Max size **2 MB**, configurable via `FileStorageOptions`.
- **FIL-02 — Security Invariant (Magic Bytes):** Do not rely on file extension or client `ContentType` headers (these can be easily spoofed). Validate file headers by inspecting the initial raw binary signatures (Magic Bytes) to verify legitimate JPEG, PNG, or WebP files.
- **FIL-03:** Store files through an `IFileStorage` abstraction (defined in Application, local disk implementation in Infrastructure). Save under a generated random name (`Guid`), never the user's file name.
- **FIL-04:** Replacing a banner deletes the old file. Soft-deleting a campaign keeps the file (hard cleanup not required).
- **FIL-05:** The banner is served publicly through a URL returned in the campaign response (e.g., `bannerUrl`).
- **Errors:** wrong type or failed signature check → `File.InvalidType` (400); file over the limit → `File.TooLarge` (413).

| Method | Path | Access |
|---|---|---|
| `POST` | `/campaigns/{id}/banner` | Admin, owner Campaign Manager (multipart/form-data) |
| `DELETE` | `/campaigns/{id}/banner` | Admin, owner Campaign Manager |

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Upload Files With Fake Extensions Part 1 — Mohamed El-Helaly [Arabic] (09:38)](https://youtu.be/mh0ZTXfSdBI).
  - ⭐2 [Upload Files Part 2 — Mohamed El-Helaly [Arabic] (11:10)](https://youtu.be/08Ke6nhNWK8).
  - ⭐3 [Microsoft Learn — File signature validation (Magic Bytes)](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads).
- **🔄 Alternative / Deep Dive:** [Design a File Upload Service — Milan Jovanović](https://youtu.be/ka_WCmNdybE)

---

### Task 5 — Asynchronous Jobs: Donation Processing & Expired Campaigns

Implement the jobs according to SRS §5.8 and the success flow in SRS §5.5:
- **Enqueue after commit:** Jobs are enqueued **only AFTER the success-flow transaction commits**, never inside the transaction. The temporary stub `POST /admin/donations/{id}/mark-succeeded` (Admin, DON-06) is still present in this sprint and triggers the same flow.
- **JOB-01 (Fire-and-forget) — Send donation receipt:** after a donation succeeds:
  `BackgroundJob.Enqueue<IDonationNotificationService>(s => s.SendReceiptAsync(donationId));`
  The job fetches the donation, generates the PDF receipt (FIL-06), and emails it to the donor (NTF-02).
- **JOB-02 (Fire-and-forget) — Send goal-reached email:** if the campaign just became `Completed`, enqueue an email to the campaign creator (NTF-03).
- **JOB-03 (Delayed) — Fail stale pending donation (DON-09):** when a donation is created (`POST /campaigns/{campaignId}/donations`), schedule `BackgroundJob.Schedule(...)` to run after `DonationOptions.PendingTimeout` (default **30 minutes**). If the donation is still `Pending`, it becomes `Failed`; otherwise the job does nothing.
- **JOB-04 (Recurring, hourly):** Register a **Recurring Job** running every hour (`Cron.Hourly()` / `0 * * * *`) that marks all `Active` campaigns past their `EndDate` as `Expired`.
- **Idempotency:** Jobs must be idempotent — running a job twice must not send two emails or change state twice (e.g., check a `ReceiptSentAt` / `GoalEmailSentAt` marker or the current status before acting).
- **Hangfire Best Practice:** Pass only entity IDs (`Guid`/`int`) as job arguments, never full entities or `DbContext` instances.
- Donation amounts remain validated against `DonationOptions` (min 1.00, max 10,000.00).

---

### Task 6 — Structured Logging with Serilog (Seq optional)

Implement logging according to SRS §6.10:
- Replace the default logger with Serilog (`Serilog.AspNetCore`).
- **CORE sinks:** Console and **rolling file** (e.g., `logs/donationhub-.log`, rolling daily). *A **Seq** sink (local or containerized) is a BONUS.*
- Enable request logging middleware (`UseSerilogRequestLogging`) capturing status code and duration.
- Enrich every log entry with the correlation id (`traceId`) and the user id when authenticated.
- Use structured message templates (`"Donation {DonationId} succeeded"`), not string concatenation.
- Log levels: `Information` for business events, `Warning` for recoverable anomalies (e.g., late payment), `Error` for failures (including failed job attempts).
- Never log secrets: passwords, JWT / refresh tokens, SMTP credentials, or card data.

**Resources:**
- **⭐ Required Primary:** [.NET Logging Using Seq — Bishoy Youssef [Arabic] (20:26)](https://youtu.be/BwcsVcLgFLw) (Serilog concepts apply to CORE; the Seq part is for the BONUS sink).
- **🔄 Alternative / Deep Dive:**
  - [Structured Logging With Serilog — Milan Jovanović](https://youtu.be/nVAkSBpsuTk)
  - [63. Serilog in Clean Architecture — Coding-Future](https://youtu.be/4Sf156KA4N0)
  - 📘 [Structured Logging with Serilog in ASP.NET Core — codewithmukesh](https://codewithmukesh.com/blog/structured-logging-with-serilog-in-aspnet-core/) (free written reference with full .NET 10 source code)

---

## 📦 Sprint Deliverables

- [ ] Hangfire integrated with SQL Server persistence and a basic-auth protected dashboard (JOB-05).
- [ ] MailKit email service communicating with local Mailpit SMTP server, localized by `PreferredLanguage` with RTL Arabic (NTF-01..06).
- [ ] QuestPDF receipt generated in a job and downloadable via `GET /donations/{id}/receipt` (FIL-06, FIL-07).
- [ ] Banner upload/delete endpoints with Magic Bytes validation, `IFileStorage`, random names and public URL (FIL-01..05).
- [ ] Receipt and goal-reached emails offloaded to Hangfire, enqueued after commit (JOB-01, JOB-02).
- [ ] Delayed stale-pending job (JOB-03 / DON-09) and hourly recurring expire-campaigns job (JOB-04).
- [ ] Structured Serilog logging (console + rolling file) capturing HTTP requests and background job executions.

---

## ✅ Definition of Done

- [ ] HTTP donation response is fast (<100ms) because PDF and email work is queued asynchronously.
- [ ] Sent emails with attached PDF receipts appear in Mailpit dashboard (`http://localhost:8025`).
- [ ] Hangfire dashboard (`/hangfire`) cannot be accessed anonymously.
- [ ] Renaming an `.exe` file to `.jpg` and uploading it is rejected with `File.InvalidType` (400); a file over the limit returns `File.TooLarge` (413).
- [ ] No email or PDF work runs inside the HTTP request.
- [ ] Jobs are idempotent: running the same job twice sends exactly one email and changes state once.
- [ ] Logs in the console and rolling file show structured properties (`traceId`, user id, request status and duration).
- [ ] Background job failures retry at least 3 times and each failure is logged.
- [ ] Unit tests cover the new behavior (e.g., file signature validation, stale-pending job logic, job idempotency).
- [ ] The `sprint-4` tag exists and points to the final commit.
- [ ] README is updated for the sprint (what was added, how to try it, decisions).
- [ ] No secrets in the repository history (SMTP and dashboard credentials live in user secrets / environment variables).

---

## ⭐ BONUS (optional — counts only after all CORE items pass)

- **NTF-07:** Welcome email after registration.
- **NTF-08:** Email the campaign creator when the campaign expires without reaching its goal.
- **FIL-08:** Admin exports donations to CSV.
- **FIL-09:** Image resizing or thumbnail generation.
- **JOB-06:** Daily summary email to Admin.
- **Seq sink:** Add Seq as an extra Serilog sink (see the Task 6 resources).
- **DON-11 on receipts:** If the anonymous flag is implemented, show "Anonymous Donor" on public displays and the receipt.

---

## 📤 Submission

- Submit your repository URL on **tawjeh.net** before **23:59 on 21/12/2026** (Day 15).
- Tag the final commit and push the tag:
  ```bash
  git tag sprint-4
  git push origin sprint-4
  ```
- Fixes are allowed during the review window (**22/12/2026 to 24/12/2026**). Do not move or force-push `sprint-4`; tag the fixed commit `sprint-4-fix` and push it (`git tag sprint-4-fix && git push origin sprint-4-fix`).

---

## 🧮 Grading (100 Points)

| Layer | Points | How |
|---|---|---|
| Automated | 40 | Build, tests pass, `sprint-4` tag exists, CORE endpoints respond as the API contract says (smoke script run by a mentor). |
| Checklist | 40 | The Definition of Done items above (structure, rules, security, README). |
| Explanation | 20 | Short questions to explain your decisions (e.g., "what happens if this job runs twice?"). |

BONUS items earn capped extra credit only after all CORE items pass.

---

## 💡 Post-Sprint Enrichment

- [Transactional Outbox Pattern with Hangfire — Milan Jovanović](https://youtu.be/gytZxzT2IWY)
- [Automatic Logging via MediatR Pipeline Behaviors](https://youtu.be/JVX9MMpO6pE)
- [TickerQ: Modern Background Job Queue in .NET](https://youtu.be/x0dfj95Cj0U)
- [Zero-Allocation File Streaming in ASP.NET Core — Raw Coding](https://youtu.be/l69kUe1fNSc)
- [Logging Crash Course in .NET](https://youtu.be/XrCxy0SNRqM)
- 📘 [IHostedService vs BackgroundService in .NET — codewithmukesh](https://codewithmukesh.com/blog/ihostedservice-vs-backgroundservice-dotnet/) (what Hangfire gives you over a plain hosted service)