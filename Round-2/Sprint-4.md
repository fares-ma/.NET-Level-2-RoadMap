# Sprint 4 — Notifications, Files, Hangfire & Serilog

> Project: **DonationHub — Donation Management Platform**
> Duration: **2 weeks (~15 hours/week)**
> Evaluation: **100 points**

## Sprint Goal

Introduce background processing, email notifications, file handling, and structured logging. After this sprint, the donation success flow will trigger background jobs that generate PDF receipts and send emails — none of this work happens inside the HTTP request.

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

### Task 1 — Hangfire Setup

Configure Hangfire for background job processing:

- Use SQL Server for Hangfire storage (same database or separate)
- Register fire-and-forget, delayed, and recurring job types
- **JOB-05:** Hangfire dashboard is enabled and **protected** (not publicly open). Use basic auth from options.

### Resources
- **MUST** — [Hangfire — Getting started](https://docs.hangfire.io/en/latest/getting-started/)
- **MUST** — [Hangfire — Using SQL Server](https://docs.hangfire.io/en/latest/configuration/using-sql-server.html)
- **VIDEO** — [Code Maze — Hangfire in ASP.NET Core](https://www.youtube.com/watch?v=UGVVpvBYUaI)
- **VIDEO** — [DevCreed (محمد الهلالي) — Background Jobs Using Hangfire in .NET (Arabic)](https://www.youtube.com/watch?v=hnK1qn6ZjKo)
- **VIDEO** — [Milan Jovanović — Background Tasks with Hangfire](https://www.youtube.com/watch?v=m7zC_u6P12o)

---

### Task 2 — Email Service (MailKit)

Implement email sending through MailKit:

- **NTF-01:** Send email through MailKit using SMTP settings from `EmailOptions`.
- **NTF-06:** In development, emails go to the **Mailpit** container (no real email is sent).
- Create an `IEmailService` abstraction in Application, implementation in Infrastructure.
- **NTF-05:** Email text uses the recipient's `PreferredLanguage` (ar or en). Arabic emails render right-to-left.

### Resources
- **MUST** — [MailKit — GitHub](https://github.com/jbogard/MailKit)
- **MUST** — [Mailpit — GitHub](https://github.com/axllent/mailpit)
- **VIDEO** — [Patrick God — Send Emails with MailKit in ASP.NET Core](https://www.youtube.com/watch?v=m53kK7Xk9_k)
- **ARTICLE** — [Code Maze — Send Emails with MailKit](https://code-maze.com/aspnetcore-send-email/)

---

### Task 3 — Donation Receipt PDF

Implement PDF receipt generation:

- **FIL-06:** Generate a PDF receipt for each succeeded donation containing: receipt number, donor name, campaign title (in donor's language), amount, date.
- **JOB-01:** Receipt generation runs in a **Hangfire fire-and-forget job**, not inside the HTTP request.
- **FIL-07:** `GET /donations/{id}/receipt` downloads the PDF, only for the owner or Admin.

| Method | Path | Access |
|--------|------|--------|
| GET | `/donations/{id}/receipt` | Owner, Admin |

### Resources
- **MUST** — [QuestPDF](https://www.questpdf.com/) or another PDF library (verify license)
- **VIDEO** — [Nick Chapsas — Generate PDF Documents Easily in C# with QuestPDF](https://www.youtube.com/watch?v=Y3pU-L1Q7u0)
- **DOCUMENTATION** — [QuestPDF Quick Start](https://www.questpdf.com/quick-start.html)

---

### Task 4 — Notification Jobs

Implement email notifications through Hangfire jobs:

- **NTF-02:** Email receipt to the donor after a successful donation, with the PDF receipt attached.
- **NTF-03:** Email the campaign creator when the campaign reaches its goal.
- **NTF-04:** Emails are sent by Hangfire jobs, never inside the HTTP request. A failed job retries (at least 3 attempts) and logs the failure.

Update the donation success flow (from Sprint 3):
- After the transaction commits, **enqueue** background jobs: generate receipt PDF, email donor
- If the campaign just completed, email the campaign owner

**Important:** Jobs must be **idempotent** — running a job twice must not send two emails or change state twice.

### Resources
- **MUST** — [Hangfire — Fire-and-forget jobs](https://docs.hangfire.io/en/latest/background-methods/calling-methods-with-delay.html)

---

### Task 5 — Stale Donation Timeout

Implement:

- **DON-09:** A `Pending` donation that is not paid within the configured timeout (default 30 minutes) becomes `Failed`.
- **JOB-03:** Schedule a **Hangfire delayed job** when a donation is created. After the timeout, if the donation is still `Pending`, mark it `Failed`.

### Resources
- **MUST** — [Hangfire — Delayed jobs](https://docs.hangfire.io/en/latest/background-methods/calling-methods-with-delay.html)

---

### Task 6 — Campaign Expiration Job

Implement:

- **JOB-04:** A **recurring Hangfire job** (hourly) that finds active campaigns past their `EndDate` and marks them `Expired`.

### Resources
- **MUST** — [Hangfire — Recurring jobs](https://docs.hangfire.io/en/latest/background-methods/performing-recurrent-tasks.html)

---

### Task 7 — Campaign Banner Upload

Implement file upload for campaign banners:

- **FIL-01:** Upload a campaign banner (Admin or owner Campaign Manager). Allowed types: JPG, PNG, WEBP. Max size 2 MB (configurable from `FileStorageOptions`).
- **FIL-02:** Validate **content** (magic bytes), not only the extension.
- **FIL-03:** Store through an `IFileStorage` abstraction (local disk implementation). Save under a **generated random name**, never the user's file name.
- **FIL-04:** Replacing a banner deletes the old file.
- **FIL-05:** The banner is served publicly through a URL returned in the campaign response.

| Method | Path | Access |
|--------|------|--------|
| POST | `/campaigns/{id}/banner` | Admin, owner Manager (multipart/form-data) |
| DELETE | `/campaigns/{id}/banner` | Admin, owner Manager |

### Resources
- **MUST** — [ASP.NET Core — File uploads](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads)
- **MUST** — [ASP.NET Core — Static files](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files)
- **VIDEO** — [Milan Jovanović — Secure File Upload in ASP.NET Core](https://www.youtube.com/watch?v=p3mO2uC2bZ8)

---

### Task 8 — Serilog

Replace the default logger with Serilog:

- Sinks: **console** and **rolling file**
- Enrich every log entry with a **correlation id** (`traceId`) and the **user id** when authenticated
- Request logging middleware with status code and duration
- Use **structured templates** (`"Donation {DonationId} succeeded"`), not string concatenation
- Log levels: Information for business events, Warning for recoverable anomalies, Error for failures
- **Never log** passwords, tokens, Stripe secrets, or full request bodies of auth endpoints

### Resources
- **MUST** — [Serilog — Getting started](https://github.com/serilog/serilog/wiki/Getting-Started)
- **MUST** — [Serilog.AspNetCore](https://github.com/serilog/serilog-aspnetcore)
- **MUST** — [ASP.NET Core — Logging](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/)
- **VIDEO** — [Nick Chapsas — Structured Logging with Serilog in .NET](https://www.youtube.com/watch?v=nVAkSBpsuTk)
- **VIDEO** — [Milan Jovanović — Serilog in ASP.NET Core](https://www.youtube.com/watch?v=0ostnGbzMuo)

---

### Task 9 — Unit Tests for Sprint 4 Features

Add unit tests for:

- Job idempotency (running a job twice does not duplicate effects)
- File validation (magic bytes, size, type)
- Donation timeout logic
- Campaign expiration logic

### Resources
- **MUST** — [xUnit](https://xunit.net/)

---

## Sprint Deliverables

- [ ] Hangfire configured with SQL Server storage
- [ ] Hangfire dashboard protected
- [ ] MailKit email service with `IEmailService` abstraction
- [ ] Mailpit container for development
- [ ] Email localization (ar/en)
- [ ] PDF receipt generation (Hangfire job)
- [ ] Receipt download endpoint
- [ ] Donation receipt email with PDF attachment (Hangfire job)
- [ ] Campaign goal-reached email (Hangfire job)
- [ ] All emails sent by Hangfire, never in HTTP request
- [ ] Jobs idempotent
- [ ] Stale donation timeout (delayed job)
- [ ] Campaign expiration (recurring hourly job)
- [ ] Campaign banner upload with magic-byte validation
- [ ] `IFileStorage` abstraction with local disk implementation
- [ ] Random file names, old file deleted on replace
- [ ] Banner URL in campaign response
- [ ] Serilog with console + rolling file sinks
- [ ] Correlation ID and user ID enrichment
- [ ] Structured logging templates
- [ ] No secrets logged
- [ ] Unit tests for new features

## Definition of Done

- [ ] Hangfire jobs execute correctly
- [ ] Dashboard not publicly accessible
- [ ] Donation receipt PDF generated in background
- [ ] Donor receives email with receipt after donation succeeds
- [ ] Campaign creator notified when goal reached
- [ ] Failed email job retries at least 3 times
- [ ] Stale pending donations auto-fail after timeout
- [ ] Expired campaigns auto-detected hourly
- [ ] Banner upload validates content type by magic bytes
- [ ] File stored with random name
- [ ] Banner served publicly
- [ ] Serilog replaces default logger
- [ ] Correlation ID in every log entry
- [ ] No sensitive data in logs
- [ ] New unit tests pass
- [ ] Learner can explain: "What happens if this job runs twice?"

## Review Rule

The learner should be able to explain not only **what** was implemented, but **why it belongs there, what alternatives existed, and what trade-off was accepted**.