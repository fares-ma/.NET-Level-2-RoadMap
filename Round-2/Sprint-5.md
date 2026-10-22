# Sprint 5 — Stripe Checkout, Webhooks, Idempotency & Integration Testing

> **Project:** DonationHub — Donation Management & Crowdfunding Platform  
> **Duration:** 15 Days (22/12/2026 to 05/01/2027)  
> **Meeting Schedule:**  
> - **Stand-up 1 (Day 5):** 26/12/2026 — Session Leader: **Asma ELhadidy**  
> - **Stand-up 2 (Day 10):** 31/12/2026 — Session Leader: **Kareem Zein**  
> - **Closing & Deadline (Day 15):** 05/01/2027 — Session Leader: **Mohamed Atef**  
> - **Code Review & Corrections (Days 16–18):** 06/01/2027 to 08/01/2027  
> **Evaluation:** 100 Points  
> **Target Framework:** .NET 10 (fully compatible with .NET 8 / .NET 9)

---

## 🎯 Sprint Goal

Replace the development payment stub with a production-ready **Stripe Checkout** payment gateway. Safely process asynchronous payment notifications using **Stripe Webhooks** with cryptographic HMAC signature verification. Guarantee financial safety against network retries and duplicate webhooks via **API Idempotency** (`Idempotency-Key` header and event deduplication). Validate complete, critical application workflows using real **Integration Tests** powered by `WebApplicationFactory` against a real SQL Server test database (**Testcontainers** is a BONUS option).

```text
Donor clicks Donate
        ↓
POST /donations/{id}/checkout
        ↓
Stripe Checkout Session Created (Minor Units / Cents)
        ↓
Donor Pays on Hosted Stripe Page
        ↓
Stripe Webhook (checkout.session.completed)
        ↓
Verify Stripe-Signature Header (EventUtility.ConstructEvent)
        ↓
Deduplicate Webhook Event (Idempotency Check)
        ↓
Verify Amount & Currency → Settle Donation (one transaction)
        ↓
Return 200 quickly → Enqueue Receipt PDF & Email After Commit (Hangfire)
```

---

## 📌 Sprint Prerequisites

### 1. SQL Server Test Database
*Integration tests run against a real SQL Server test database (never EF Core InMemory):*
- Use a SQL Server service from Docker Compose or SQL Server LocalDB, with a dedicated test database.
- **BONUS:** Testcontainers spins up a disposable SQL Server container instead; this requires Docker Desktop or Docker Engine running locally.

### 2. Stripe CLI & Test Account Setup
*To receive Stripe webhooks on your local development machine:*
- Sign up for a free Stripe developer account and obtain **test mode** API keys only (`pk_test_...`, `sk_test_...`).
- Store the secret key and webhook secret with `dotnet user-secrets` (or environment variables). **Never** put them in `appsettings*.json`.
- If a Stripe test account cannot be created in your country, tell your mentor **before Stand-up 1 (26/12/2026)**.
- Download and install the [Stripe CLI](https://docs.stripe.com/stripe-cli).
- Authenticate via `stripe login`.
- Forward webhooks to your local API:  
  `stripe listen --forward-to https://localhost:7001/api/v1/payments/webhook`

---

## 📋 Engineering Decisions & Financial Invariants

1. **Currency in Minor Units:** Stripe requires amounts in the smallest currency unit (e.g., USD cents: `$10.50` = `1050`). Never pass floating-point numbers directly; calculate integer minor units explicitly.
2. **Never Trust the Client:** The payment amount and campaign association are determined by server state, not client input. The Stripe Checkout session must lock the amount derived from the database record.
3. **At-Least-Once Delivery & Idempotency:** Webhooks may be redelivered multiple times by Stripe during network retries. Every webhook event ID must be recorded and processed exactly once.
4. **The Webhook Is the Only Source of Truth:** A donation becomes `Succeeded` only from a verified webhook event, never from the success URL redirect (the client can open that URL without paying).

---

## 🛠️ Sprint Tasks

All tasks below are **CORE** unless marked BONUS.

### Task 1 — Stripe Checkout Session Integration

Implement the checkout initiation workflow according to SRS §5.9:
- **PAY-01:** `POST /donations/{id}/checkout` (Owner only) creates a Stripe **Checkout Session** (mode: `payment`) for a `Pending` donation and returns the session checkout URL. Donations are created beforehand with `POST /campaigns/{campaignId}/donations`.
- **PAY-02:** The created session must carry:
  - `client_reference_id` = `donation.Id.ToString()`.
  - Metadata dictionary containing `donationId` and `campaignId`.
  - Single line item: Campaign Title as name, amount converted to **minor units** (cents), and currency from configuration.
  - Success and Cancel callback URLs defined in `StripeOptions`.
- **PAY-03:** Create and persist a `PaymentAttempt` for each created session (`DonationId`, `StripeCheckoutSessionId`, `StripePaymentIntentId?`, `Amount`, `Status`, `CreatedAt`, `UpdatedAt`, `FailureCode?`). Status values: `Created`, `Succeeded`, `Failed`, `Expired`. A donor may start a new attempt for the same pending donation if a prior session was abandoned.
- **Payment status:** `GET /donations/{id}/payment-status` (Owner, Admin) returns the donation status and its latest payment attempt status. Another user's donation returns 404.
- **PAY-10:** Use Stripe **test mode** keys only. `StripeOptions` (secret key, webhook secret, currency, success/cancel URLs) is validated with `ValidateOnStart()`; the secret values come from `dotnet user-secrets` or environment variables, never from `appsettings*.json`.
- **PAY-09:** Remove the temporary stub endpoint `POST /admin/donations/{id}/mark-succeeded` (DON-06) and its handler. From now on, only the webhook can complete a donation.

**Resources:**
- **⭐ Required Primary:** [How To Easily Integrate STRIPE Into Your .NET API — Israel Quiroz (38:45)](https://youtu.be/2sZXrUFvZ3E).
- **🔄 Alternative / Deep Dive:** [Stripe Official Checkout Sessions Documentation](https://docs.stripe.com/api/checkout/sessions)

---

### Task 2 — Stripe Webhook Handling & Signature Verification

Implement the secure webhook handler according to SRS §5.9:
- **PAY-04:** Provide `POST /api/v1/payments/webhook` accepting the raw payload from Stripe. The endpoint is anonymous (the signature is the authentication) and is **excluded from rate limiting** (SRS §6.4).
- **Cryptographic Verification:** Read the raw request body stream and the `Stripe-Signature` request header. Validate the HMAC signature using the webhook secret:
  ```csharp
  var stripeEvent = EventUtility.ConstructEvent(
      jsonPayload,
      Request.Headers["Stripe-Signature"],
      stripeOptions.WebhookSecret
  );
  ```
  If verification fails, return **400 Bad Request** immediately.
- **PAY-05 (Event Handling):**
  - `checkout.session.completed`: extract the `client_reference_id`, retrieve the donation and its attempt, and run the SRS §5.5 success flow in **one transaction** (donation `Succeeded` + `CompletedAt` + `ReceiptNumber`, attempt `Succeeded`, add to `RaisedAmount`, campaign `Completed` when the goal is reached). A payment that arrives after the campaign expired or completed is still accepted and logged as a warning.
  - `checkout.session.expired`: mark the attempt `Expired`.
  - `payment_intent.payment_failed`: mark the attempt `Failed` (store `FailureCode`); mark the donation `Failed` when no other attempt can still succeed.
  - Unknown sessions or unhandled event types are logged and answered with 200.
- **PAY-07 (Amount Verification):** Verify that the paid amount (minor units) and currency match the donation. A mismatch is logged as an **error** and does **not** complete the donation.
- **PAY-08 (Fast Response):** The webhook returns 200 quickly. Heavy work (receipt PDF, emails) is **enqueued with Hangfire after the commit**, never executed inside the webhook request.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [159- Creating Stripe Web Hook Endpoint — Mohamed Awadallah [Arabic] (10:58)](https://youtu.be/WZu0lHvNYjI) (stripe login at 00:15, stripe listen at 01:05, ConstructEvent at 03:05, --forward-to at 05:35).
  - ⭐2 [Stripe Official Webhooks Guide](https://docs.stripe.com/webhooks).

---

### Task 3 — Local Webhook Testing via Stripe CLI

- Run your application locally (e.g., `https://localhost:7001`).
- Execute `stripe listen --forward-to https://localhost:7001/api/v1/payments/webhook`.
- Capture the temporary webhook signing secret displayed by the CLI (`whsec_...`) and store it with `dotnet user-secrets set "Stripe:WebhookSecret" "whsec_..."` (or an environment variable). **Never** commit it to `appsettings.Development.json` or any other tracked file.
- Trigger test events using `stripe trigger checkout.session.completed`, `stripe trigger checkout.session.expired` and `stripe trigger payment_intent.payment_failed`, and confirm your API handles each one.
- Document these steps in the README section "How to test Stripe locally".

---

### Task 4 — API & Webhook Idempotency

Implement idempotency safeguards according to SRS §6.5:
- **DON-10 / §6.5 (API Request Idempotency):**
  - `POST /campaigns/{campaignId}/donations` and `POST /donations/{id}/checkout` require an `Idempotency-Key` header. A missing key returns **400**.
  - Store each key in an `IdempotencyRecord` (`Key`, `UserId`, `RequestHash`, `ResponseStatusCode`, `ResponseBody`, `CreatedAt`, `ExpiresAt`).
  - Same key + same user + same body: return the stored response without repeating the action.
  - Same key + different body: return **422** with error code `Idempotency.KeyReusedWithDifferentRequest`.
  - Keys expire after **24 hours**.
  - Two simultaneous requests with the same key must not create two donations (e.g., a unique index on `Key` + `UserId`, inserted before the action runs).
- **PAY-06 (Webhook Deduplication):**
  - Store each processed Stripe event in a `ProcessedStripeEvent` record (`EventId` as key, `Type`, `ProcessedAt`), saved in the same transaction as the event's changes.
  - If an incoming event ID already exists, acknowledge it immediately with **HTTP 200 OK** and ignore the duplicate to prevent double-crediting campaign funds.
- Errors follow the shared Error catalogue (`Code`, `Message`, `Type`) and are returned as ProblemDetails including `code`, `traceId` and, for validation, `errors`.

**Resources:**
- **⭐ Required Primary:** [How To Make Your API Idempotent — Milan Jovanović (14:26)](https://youtu.be/smXAgcdJzLc) (Idempotency-Key header at 01:50, Pipeline behavior at 03:13, database schema at 07:45).
- **🔄 Alternative / Deep Dive:** [Stripe Documentation — Handle duplicate webhook events](https://docs.stripe.com/webhooks#handle-duplicate-events)

---

### Task 5 — Integration Testing with WebApplicationFactory (Testcontainers as BONUS)

Create `DonationHub.Api.IntegrationTests` (SRS §6.13):
- Configure `WebApplicationFactory<Program>` to bootstrap the API in-memory.
- Run the tests against a **real SQL Server test database** (a Docker Compose service or LocalDB). Never use the EF Core InMemory provider.
- **BONUS:** use **Testcontainers for .NET** to spin up a disposable SQL Server container for the test suite.
- Never call real Stripe or real SMTP in tests: build webhook payloads locally and sign them with a test webhook secret, and replace the email sender with a fake.
- Cover at least these scenarios:
  1. Register and login.
  2. Create a campaign and a donation.
  3. Webhook success flow with a locally signed payload (donation `Succeeded`, `RaisedAmount` updated).
  4. Duplicate webhook event (funds credited once, second call returns 200).
  5. Invalid signature (400, nothing changed).
  6. Idempotency key replay (same response, no second donation).
- The critical user journey:
  ```text
  Register Donor
       ↓
  Authenticate & Receive JWT
       ↓
  Create Campaign as Admin
       ↓
  Create Donation as Donor (Pending)
       ↓
  Simulate Valid Stripe Webhook Delivery
       ↓
  Assert Donation is Succeeded & Campaign RaisedAmount is Updated
  ```

**Resources:**
- **⭐ Required Primary:** [Stop Mocking Your Database! Use Testcontainers in .NET — Milan Jovanović (15:59)](https://youtu.be/ssRE0pBNvpE) (`WebApplicationFactory` setup applies to the CORE task; the Testcontainers part is for the BONUS option).
- **🔄 Alternative / Deep Dive:** [Integration Testing in ASP.NET Core — Julio Casal](https://youtu.be/RXSPCIrrjHc)

---

## 📦 Sprint Deliverables

- [ ] Stripe Checkout session generation endpoint returning valid Stripe redirect URL (PAY-01 to PAY-03).
- [ ] `GET /donations/{id}/payment-status` for the owner and Admin.
- [ ] Cryptographically verified Stripe webhook endpoint using `EventUtility.ConstructEvent`, handling the three PAY-05 events with amount/currency verification (PAY-07).
- [ ] Webhook event deduplication via `ProcessedStripeEvent` records (PAY-06).
- [ ] `Idempotency-Key` handling with `IdempotencyRecord` on donation creation and checkout (DON-10, §6.5).
- [ ] The `mark-succeeded` stub removed (PAY-09).
- [ ] Local testing workflow documented in the README using Stripe CLI.
- [ ] Integration test suite (`WebApplicationFactory` + real SQL Server test database) covering the six SRS §6.13 scenarios.

---

## ✅ Definition of Done

- [ ] Creating a checkout session produces a valid URL and stores a `PaymentAttempt` with status `Created`.
- [ ] Tampered or unsigned webhook requests are rejected with HTTP 400.
- [ ] Sending the exact same webhook payload twice does not increment campaign funds a second time; the duplicate is answered with 200.
- [ ] `checkout.session.expired` marks the attempt `Expired`, and `payment_intent.payment_failed` marks it `Failed`.
- [ ] An amount or currency mismatch is logged as an error and does not complete the donation.
- [ ] The donation is marked `Succeeded` only by the webhook, never by the success URL.
- [ ] The webhook does no PDF or email work inline; these jobs are enqueued after the commit.
- [ ] The stub endpoint `POST /admin/donations/{id}/mark-succeeded` no longer exists.
- [ ] Repeating a request with the same `Idempotency-Key` returns the stored response without creating duplicate entities; a missing key returns 400 and a reused key with a different body returns 422.
- [ ] `dotnet test` runs all integration tests against a real SQL Server test database, without calling real Stripe or SMTP.
- [ ] The README is updated for the sprint, including **how to test Stripe locally**.
- [ ] No secrets (Stripe keys, webhook secret) exist in the repository history.
- [ ] The `sprint-5` tag exists and points to the final commit.

---

## ⭐ BONUS (optional — counts only after all CORE items pass)

- **PAY-11:** Admin refund of a succeeded donation (see DON-13).
- **PAY-12:** Handle the `charge.refunded` webhook event.
- **DON-13:** Refund of a succeeded donation (Admin), with the amount subtracted from the campaign's `RaisedAmount`.
- **Testcontainers:** run the integration tests against a disposable SQL Server container.
- **IDN-13:** Refresh token reuse detection (if not done in an earlier sprint): using an already-revoked token revokes the whole token family.

---

## 📤 Submission

- Submit your repository URL on **tawjeh.net** before **23:59 on Day 15 (05/01/2027)**.
- Tag the final commit and push the tag:
  ```bash
  git tag sprint-5
  git push origin sprint-5
  ```
- Fixes are allowed during the review window (**06/01/2027 to 08/01/2027**). Do not move or force-push `sprint-5`; tag the fixed commit `sprint-5-fix` and push it (`git tag sprint-5-fix && git push origin sprint-5-fix`).
- If a Stripe test account cannot be created in your country, tell your mentor **before Stand-up 1 (26/12/2026)**.

---

## 🧮 Grading (100 Points)

| Part | Points | How |
|---|---|---|
| Automated | 40 | Build succeeds, `dotnet test` passes, the `sprint-5` tag exists, CORE endpoints respond as specified (smoke check by the mentor) |
| Checklist | 40 | The Definition of Done above (structure, rules, security, README) |
| Explanation | 20 | A short question on one of your design decisions, or a code review of the decisions written in your README |

BONUS items add extra credit (capped by the mentors) on top of the 100 points, and only when all CORE items pass.

---

## 💡 Post-Sprint Enrichment

- [Building Resilient Cloud Microservices with Polly (.NET Foundation)](https://youtu.be/L9_fGJOqzbM)