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

Eliminate heavy, blocking operations from the HTTP request pipeline. When a donation completes, offload receipt generation and email notifications to background workers using **Hangfire**. Generate professional PDF receipts using **QuestPDF**, send transactional emails via **MailKit** tested locally against a **Mailpit** container, enforce security on campaign file uploads via **Magic Bytes verification**, schedule daily maintenance jobs, and configure structured logging using **Serilog** and **Seq**.

```text
HTTP Donation Completed
           ↓
Enqueue Hangfire Job (Fire-and-Forget)
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
| **Serilog & Seq** | Free developer instance | Structured logging enriched with `TraceId`, `Environment`, and `MachineName`. |

---

## 🛠️ Sprint Tasks

### Task 1 — Hangfire Infrastructure & Protected Dashboard

Implement background job orchestration according to SRS §3.5:
- Configure Hangfire with SQL Server persistent storage: `UseSqlServerStorage`.
- Register the Hangfire server in `Program.cs`: `builder.Services.AddHangfireServer()`.
- **JOB-05 (Protected Dashboard):** Protect the Hangfire dashboard endpoint `/hangfire`. Do not leave it open to the public. Implement an authorization filter (e.g., `BasicAuthAuthorizationFilter` or ASP.NET Core Policy requirement checking `Admin` role).

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

- Implement `IEmailService` in `DonationHub.Application` and concrete implementation in `DonationHub.Infrastructure` using **MailKit** and **MimeKit**.
- Configure `MailOptions` (Host, Port, SenderEmail, SenderName).
- Connect to local Mailpit container (`localhost:1025`) in Development environment.
- Send HTML-formatted transactional emails with optional attachments.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Sending Emails With .NET Core Part 1 — Mohamed El-Helaly [Arabic] (10:33)](https://youtu.be/J23_c3uSyD0).
  - ⭐2 [Mailpit Docker Documentation](https://mailpit.axllent.org/docs/install/docker/).
- **🔄 Alternative / Deep Dive:**
  - [Send Emails with MailKit — tutorialsEU](https://youtu.be/lCHKwyekbT4)
  - [Easy Email Verification in .NET with Papercut/Mailpit — Milan Jovanović (04:50)](https://youtu.be/KtCjH-1iCIk)

---

### Task 3 — Dynamic PDF Receipt Generation via QuestPDF

Implement tax-deductible / donor receipt generation:
- Configure QuestPDF with Community license: `QuestPDF.Settings.License = LicenseType.Community;`.
- Design a clean, branded PDF receipt containing:
  - Donation Reference ID and Transaction Date.
  - Campaign Title and Category.
  - Donor Name (or "Anonymous Donor" if `IsAnonymous = true`).
  - Formatted Amount and Currency.
  - Organization verification header and footer.
- Generate receipt as byte array / stream in-memory without disk overhead.

**Resources:**
- **⭐ Required Primary:** [QuestPDF - The BEST PDF Generator for .NET — Claudio Bernasconi (12:07)](https://youtu.be/T89A_7dz1P8).
- **🔄 Alternative / Deep Dive:**
  - [Generate PDF Invoice in ASP.NET Core — M Alvin Maulana](https://youtu.be/eiry9Zli5Rc)
  - [QuestPDF Official Invoice Tutorial](https://www.questpdf.com/quick-start.html)

---

### Task 4 — Secure File Uploads & Magic Bytes Validation

Implement image uploads for campaigns according to SRS §3.3:
- Accept campaign cover images via `IFormFile`.
- **Security Invariant (Magic Bytes):** Do not rely on file extension or client `ContentType` headers (these can be easily spoofed). Validate file headers by inspecting the initial raw binary signatures (Magic Bytes) to verify legitimate JPEG, PNG, or WebP files.
- Enforce size restrictions (e.g., max 2 MB).
- Generate cryptographically random unique filenames (`Guid`) before saving to storage.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [Upload Files With Fake Extensions Part 1 — Mohamed El-Helaly [Arabic] (09:38)](https://youtu.be/mh0ZTXfSdBI).
  - ⭐2 [Upload Files Part 2 — Mohamed El-Helaly [Arabic] (11:10)](https://youtu.be/08Ke6nhNWK8).
  - ⭐3 [Microsoft Learn — File signature validation (Magic Bytes)](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads).
- **🔄 Alternative / Deep Dive:** [Design a File Upload Service — Milan Jovanović](https://youtu.be/ka_WCmNdybE)

---

### Task 5 — Asynchronous Jobs: Donation Processing & Expired Campaigns

- **JOB-01 & JOB-02:** Upon donation settlement, enqueue a background job:
  `BackgroundJob.Enqueue<IDonationNotificationService>(s => s.SendReceiptAsync(donationId));`
  The job fetches the donation, generates the PDF receipt, and emails it to the donor.
- **JOB-03 & JOB-04:** Register a **Recurring Job** running daily at midnight via Cron expression (`0 0 * * *`) that queries all active campaigns past their `EndDate` and marks them as `Expired`.
- **Hangfire Best Practice:** Pass only entity IDs (`Guid`/`int`) as job arguments, never full entities or `DbContext` instances.

---

### Task 6 — Structured Logging with Serilog & Seq

- Remove default console logger and install `Serilog.AspNetCore`.
- Configure sinks for Console and **Seq** (local or containerized).
- Enable request logging middleware (`UseSerilogRequestLogging`).
- Enrich logs with contextual metadata: `TraceId`, `Environment`, `UserId`, `MachineName`.
- Ensure sensitive data (passwords, JWT tokens, credit card data) is strictly sanitized and omitted from log payloads.

**Resources:**
- **⭐ Required Primary:** [.NET Logging Using Seq — Bishoy Youssef [Arabic] (20:26)](https://youtu.be/BwcsVcLgFLw).
- **🔄 Alternative / Deep Dive:**
  - [Structured Logging With Serilog — Milan Jovanović](https://youtu.be/nVAkSBpsuTk)
  - [63. Serilog in Clean Architecture — Coding-Future](https://youtu.be/4Sf156KA4N0)

---

## 📦 Sprint Deliverables

- [ ] Hangfire integrated with SQL Server persistence and protected dashboard.
- [ ] MailKit email service communicating with local Mailpit SMTP server.
- [ ] QuestPDF receipt generator producing formatted, valid PDF files.
- [ ] Image upload endpoint with raw Magic Bytes file header inspection.
- [ ] Asynchronous receipt and email dispatch offloaded to Hangfire background queue.
- [ ] Daily recurring job marking overdue campaigns as `Expired`.
- [ ] Structured Serilog logging capturing HTTP requests and background job executions.

---

## ✅ Definition of Done

- [ ] HTTP donation response is fast (<100ms) because PDF and email work is queued asynchronously.
- [ ] Sent emails with attached PDF receipts appear in Mailpit dashboard (`http://localhost:8025`).
- [ ] Hangfire dashboard (`/hangfire`) cannot be accessed anonymously.
- [ ] Renaming an `.exe` file to `.jpg` and uploading it is rejected with a validation error.
- [ ] Logs in Seq or Console show structured JSON properties (`TraceId`, `Properties`).
- [ ] Background job failures retry automatically according to configured policies.

---

## 💡 Post-Sprint Enrichment

- [Transactional Outbox Pattern with Hangfire — Milan Jovanović](https://youtu.be/gytZxzT2IWY)
- [Automatic Logging via MediatR Pipeline Behaviors](https://youtu.be/JVX9MMpO6pE)
- [TickerQ: Modern Background Job Queue in .NET](https://youtu.be/x0dfj95Cj0U)
- [Zero-Allocation File Streaming in ASP.NET Core — Raw Coding](https://youtu.be/l69kUe1fNSc)
- [Logging Crash Course in .NET](https://youtu.be/XrCxy0SNRqM)