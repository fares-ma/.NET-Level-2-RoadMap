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

Replace the development payment stub with a production-ready **Stripe Checkout** payment gateway. Safely process asynchronous payment notifications using **Stripe Webhooks** with cryptographic HMAC signature verification. Guarantee financial safety against network retries and duplicate webhooks via **API Idempotency** (`Idempotency-Key` header and event deduplication). Validate complete, critical application workflows using real **Integration Tests** powered by `WebApplicationFactory` and **Testcontainers**.

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
Settle Donation & Trigger Background Receipt (Hangfire)
```

---

## 📌 Sprint Prerequisites

### 1. Docker Runtime for Testcontainers
*Integration tests will spin up real, disposable SQL Server database containers dynamically:*
- Ensure Docker Desktop or Docker Engine is running locally.

### 2. Stripe CLI & Test Account Setup
*To receive Stripe webhooks on your local development machine:*
- Sign up for a free Stripe developer account and obtain test API keys (`pk_test_...`, `sk_test_...`).
- Download and install the [Stripe CLI](https://docs.stripe.com/stripe-cli).
- Authenticate via `stripe login`.
- Forward webhooks to your local API:  
  `stripe listen --forward-to https://localhost:7001/api/v1/payments/webhook`

---

## 📋 Engineering Decisions & Financial Invariants

1. **Currency in Minor Units:** Stripe requires amounts in the smallest currency unit (e.g., USD cents: `$10.50` = `1050`). Never pass floating-point numbers directly; calculate integer minor units explicitly.
2. **Never Trust the Client:** The payment amount and campaign association are determined by server state, not client input. The Stripe Checkout session must lock the amount derived from the database record.
3. **At-Least-Once Delivery & Idempotency:** Webhooks may be redelivered multiple times by Stripe during network retries. Every webhook event ID must be recorded and processed exactly once.

---

## 🛠️ Sprint Tasks

### Task 1 — Stripe Checkout Session Integration

Implement the checkout initiation workflow according to SRS §3.6:
- **PAY-01:** `POST /donations/{id}/checkout` creates a Stripe **Checkout Session** (mode: `payment`) for any `Pending` donation and returns the session checkout URL.
- **PAY-02:** The created session must carry:
  - `client_reference_id` = `donation.Id.ToString()`.
  - Metadata dictionary containing `donationId` and `campaignId`.
  - Single line item: Campaign Title as name, amount converted to **minor units** (cents), and currency from configuration.
  - Success and Cancel callback URLs defined in `StripeOptions`.
- **PAY-03:** Create and persist a `PaymentAttempt` entity linking the `DonationId` and `StripeSessionId`. A donor may retry and initiate a new attempt if a prior session was abandoned.

**Resources:**
- **⭐ Required Primary:** [How To Easily Integrate STRIPE Into Your .NET API — Israel Quiroz (38:45)](https://youtu.be/2sZXrUFvZ3E).
- **🔄 Alternative / Deep Dive:** [Stripe Official Checkout Sessions Documentation](https://docs.stripe.com/api/checkout/sessions)

---

### Task 2 — Stripe Webhook Handling & Signature Verification

Implement the secure webhook handler according to SRS §3.6:
- **PAY-04:** Provide `POST /api/v1/payments/webhook` accepting raw payload from Stripe.
- **Cryptographic Verification:** Read the raw request body stream and the `Stripe-Signature` request header. Validate the HMAC signature using the webhook secret:
  ```csharp
  var stripeEvent = EventUtility.ConstructEvent(
      jsonPayload,
      Request.Headers["Stripe-Signature"],
      stripeOptions.WebhookSecret
  );
  ```
  If verification fails, return **400 Bad Request** immediately.
- **Event Handling:** On `checkout.session.completed`, extract the `client_reference_id`, retrieve the donation, and transition it to `Succeeded` within an atomic transaction.

**Resources:**
- **⭐ Required Primary:**
  - ⭐1 [159- Creating Stripe Web Hook Endpoint — Mohamed Awadallah [Arabic] (10:58)](https://youtu.be/WZu0lHvNYjI) (stripe login at 00:15, stripe listen at 01:05, ConstructEvent at 03:05, --forward-to at 05:35).
  - ⭐2 [Stripe Official Webhooks Guide](https://docs.stripe.com/webhooks).

---

### Task 3 — Local Webhook Testing via Stripe CLI

- Run your application locally (e.g., `https://localhost:7001`).
- Execute `stripe listen --forward-to https://localhost:7001/api/v1/payments/webhook`.
- Capture the temporary webhook signing secret displayed by the CLI (`whsec_...`) and update your local `appsettings.Development.json`.
- Trigger test events using `stripe trigger checkout.session.completed` and confirm your API successfully parses and settles the payment.

---

### Task 4 — API & Webhook Idempotency

Implement idempotency safeguards according to SRS §3.7:
- **IDEM-01 (API Request Idempotency):**
  - Support the `Idempotency-Key` header on mutation endpoints (e.g., `POST /campaigns/{id}/donations`).
  - Store incoming keys in an `IdempotentRequests` table (`Key`, `RequestHash`, `ResponseStatusCode`, `ResponseBody`, `CreatedAtUtc`).
  - If a duplicate key is received with the same payload, return the cached response without re-executing business logic.
- **IDEM-02 (Webhook Deduplication):**
  - Store incoming Stripe `event.Id` in a `ProcessedStripeEvents` table.
  - If an incoming event ID already exists in the table, acknowledge receipt immediately with **HTTP 200 OK** and ignore the duplicate event to prevent double-crediting campaign funds.

**Resources:**
- **⭐ Required Primary:** [How To Make Your API Idempotent — Milan Jovanović (14:26)](https://youtu.be/smXAgcdJzLc) (Idempotency-Key header at 01:50, Pipeline behavior at 03:13, database schema at 07:45).
- **🔄 Alternative / Deep Dive:** [Stripe Documentation — Handle duplicate webhook events](https://docs.stripe.com/webhooks#handle-duplicate-events)

---

### Task 5 — Integration Testing with WebApplicationFactory & Testcontainers

Create `DonationHub.Api.IntegrationTests`:
- Configure `WebApplicationFactory<Program>` to bootstrap the API in-memory.
- Use **Testcontainers for .NET** to spin up a genuine Microsoft SQL Server Docker container for the test suite.
- Write automated integration tests covering the critical user journey:
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
- **⭐ Required Primary:** [Stop Mocking Your Database! Use Testcontainers in .NET — Milan Jovanović (15:59)](https://youtu.be/ssRE0pBNvpE).
- **🔄 Alternative / Deep Dive:** [Integration Testing in ASP.NET Core — Julio Casal](https://youtu.be/RXSPCIrrjHc)

---

## 📦 Sprint Deliverables

- [ ] Stripe Checkout session generation endpoint returning valid Stripe redirect URL.
- [ ] Cryptographically verified Stripe webhook endpoint using `EventUtility.ConstructEvent`.
- [ ] Webhook event deduplication via `ProcessedStripeEvents` database tracking.
- [ ] API Idempotency pipeline behavior checking `Idempotency-Key` header.
- [ ] Local testing workflow documented using Stripe CLI.
- [ ] Testcontainers integration test suite validating the complete donation and settlement lifecycle against real SQL Server.

---

## ✅ Definition of Done

- [ ] Creating a checkout session produces a valid URL and stores a `PaymentAttempt`.
- [ ] Tampered or unsigned webhook requests are rejected with HTTP 400.
- [ ] Sending the exact same webhook payload twice does not increment campaign funds a second time.
- [ ] Repeating a request with the same `Idempotency-Key` returns the cached response without creating duplicate entities.
- [ ] `dotnet test` executes all integration tests successfully using Testcontainers without requiring a pre-existing local database.

---

## 💡 Post-Sprint Enrichment

- [Building Resilient Cloud Microservices with Polly (.NET Foundation)](https://youtu.be/L9_fGJOqzbM)
- [Distributed Locks with Redis and MediatR](https://youtu.be/Tt5zIKVMMbs)