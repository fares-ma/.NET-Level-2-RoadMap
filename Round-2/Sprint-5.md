# Sprint 5 — Payments (Stripe), Idempotency & Integration Testing

> Project: **DonationHub — Donation Management Platform**
> Duration: **2 weeks (~15 hours/week)**
> Evaluation: **100 points**

## Sprint Goal

Replace the temporary payment stub with real Stripe Checkout integration, implement idempotency for safe retries, and verify important application flows with integration tests.

## How to use this sprint plan

For every task:

1. Read the **MUST** resources before implementation.
2. Read **DEEP DIVE** only when the concept is unclear or the learner wants deeper understanding.
3. Implement the requested DonationHub change.
4. Verify the behavior manually through Swagger and Stripe test mode.
5. Add/update tests when the task naturally requires them.
6. Commit the work in small, meaningful commits.
7. Be able to explain the architectural decision during review.

**Important:** This document does not change the original Sprint scope. It expands the original tasks into an executable checklist.

## Sprint Tasks

### Task 1 — Stripe Checkout Session

Implement the payment flow using Stripe Checkout:

- **PAY-01:** `POST /donations/{id}/checkout` creates a Stripe **Checkout Session** (mode `payment`) for a `Pending` donation and returns the checkout URL.
- **PAY-02:** The session carries:
  - `client_reference_id` = donation id
  - Metadata with donation id
  - A single line item (campaign title, amount in **minor units/cents**, configured currency)
  - Success and cancel URLs from `StripeOptions`
- **PAY-03:** A `PaymentAttempt` is stored for each created session. A donor may start a new attempt for the same pending donation.

| Method | Path | Access |
|--------|------|--------|
| POST | `/donations/{id}/checkout` | Owner |
| GET | `/donations/{id}/payment-status` | Owner, Admin |

### Resources
- **MUST** — [Stripe — Checkout Sessions](https://docs.stripe.com/api/checkout/sessions)
- **MUST** — [Stripe — Accept a payment](https://docs.stripe.com/payments/accept-a-payment)
- **MUST** — [Stripe.net — GitHub](https://github.com/stripe/stripe-dotnet)
- **VIDEO** — [Stripe Developers — Accept Payments with Stripe Checkout in .NET](https://www.youtube.com/watch?v=mI_y_vC0g9Y)
- **VIDEO** — [Patrick God — Stripe Payments with ASP.NET Core Web API](https://www.youtube.com/watch?v=F3aO-4U4rK0)
- **ARTICLE** — [Code Maze — Stripe Payments in ASP.NET Core](https://code-maze.com/stripe-payments-aspnetcore/)

---

### Task 2 — Webhook Endpoint

Implement the Stripe webhook receiver:

- **PAY-04:** `POST /payments/webhook` receives Stripe events. It is **anonymous**, reads the **raw body**, and **verifies the signature** with the webhook secret from `StripeOptions`. Invalid signature returns 400.
- **PAY-08:** The webhook returns 200 quickly. Heavy work (PDF, email) is **enqueued** via Hangfire, not done inside the webhook.

| Method | Path | Access |
|--------|------|--------|
| POST | `/payments/webhook` | Anonymous (signature required) |

### Resources
- **MUST** — [Stripe — Webhooks](https://docs.stripe.com/webhooks)
- **MUST** — [Stripe — Webhook signature verification](https://docs.stripe.com/webhooks/signature)
- **VIDEO** — [Stripe Developers — Listen for Webhooks in .NET](https://www.youtube.com/watch?v=o04A_Bsmh8s)
- **VIDEO** — [Milan Jovanović — Stripe Webhooks in ASP.NET Core](https://www.youtube.com/watch?v=4xR1E7aZ6Ys)

---

### Task 3 — Webhook Event Handling

Handle the following Stripe events:

- **PAY-05:**
  - `checkout.session.completed` → Run the success flow (§5.5 in SRS): update donation status, raised amount, campaign completion, enqueue PDF + email jobs
  - `checkout.session.expired` → Mark the PaymentAttempt as `Expired`
  - `payment_intent.payment_failed` → Mark PaymentAttempt as `Failed`; mark Donation as `Failed` when no other attempt can still succeed
- **PAY-07:** Verify that the paid amount and currency **match** the donation. A mismatch is logged as an error and does not complete the donation.

Edge case: if a payment succeeds after the campaign already expired or completed, **accept the money** (the donor was charged already). Log a warning.

### Resources
- **MUST** — [Stripe — Checkout events](https://docs.stripe.com/api/events/types)

---

### Task 4 — Webhook Idempotency

Implement duplicate event protection:

- **PAY-06:** Store each processed `EventId` in `ProcessedStripeEvent`. Ignore duplicates, answering 200.
- Webhooks are **idempotent**: processing the same event twice must not create duplicate donations, send duplicate emails, or change state twice.

### Resources
- **MUST** — [Stripe — Webhooks best practices](https://docs.stripe.com/webhooks#best-practices)

---

### Task 5 — Idempotency Key

Implement the `Idempotency-Key` header:

- **DON-10:** Applies to `POST /donations` and `POST /donations/{id}/checkout`.
- The client sends `Idempotency-Key: <unique string>`. The key is **required** on these endpoints (400 if missing).
- First request: process and store the key, user id, hash of request body, and response.
- Same key, same user, same body: return the stored response without repeating the action.
- Same key, different body: return 422 `Idempotency.KeyReusedWithDifferentRequest`.
- Keys expire after 24 hours.
- Two simultaneous requests with the same key must not create two donations.

### Resources
- **MUST** — [Stripe — Idempotent requests](https://docs.stripe.com/api/idempotent_requests)
- **DEEP DIVE** — [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- **VIDEO** — [Milan Jovanović — Building Idempotent APIs with ASP.NET Core](https://www.youtube.com/watch?v=b4O6u9B2y6A)
- **VIDEO** — [Derek Comartin (CodeOpinion) — What is Idempotency?](https://www.youtube.com/watch?v=T_T5Z0Y2K68)

---

### Task 6 — Remove Temporary Stub

- **PAY-09:** Remove the `POST /admin/donations/{id}/mark-succeeded` endpoint. All donation completion now goes through Stripe webhooks only.

---

### Task 7 — Stripe Test Mode Setup

- **PAY-10:** Use Stripe **test mode** keys only.
- Test locally with Stripe CLI: `stripe listen --forward-to https://localhost:<port>/api/v1/payments/webhook`
- Use Stripe's test card numbers on the hosted Checkout page.
- Stripe keys go in **user-secrets**, never committed to the repository.

### Resources
- **MUST** — [Stripe — Test mode](https://docs.stripe.com/test-mode)
- **MUST** — [Stripe CLI](https://docs.stripe.com/stripe-cli)

---

### Task 8 — Integration Test Infrastructure

Create the infrastructure required to run integration tests:

- Use `WebApplicationFactory<Program>` to create a test server
- Configure a **test database** (SQL Server from Docker Compose or another strategy)
- Set up test authentication (generate tokens for test users)
- Ensure test isolation (clean database between test runs)

### Resources
- **MUST** — [ASP.NET Core — Integration tests](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)
- **RECOMMENDED** — [Testcontainers for .NET](https://dotnet.testcontainers.org/)
- **VIDEO** — [Nick Chapsas — Master ASP.NET Core Integration Testing](https://www.youtube.com/watch?v=zaRM0iIhJvs)
- **VIDEO** — [Milan Jovanović — Integration Testing with WebApplicationFactory](https://www.youtube.com/watch?v=tj5ZCtvgLoQ)

---

### Task 9 — Critical Integration Tests

Write integration tests covering at least:

- Register → Login (full auth flow)
- Create campaign → Create donation
- Webhook success flow with a **locally signed payload** (do not call real Stripe)
- Duplicate webhook event (verify idempotency)
- Invalid webhook signature (verify rejection)
- Idempotency key replay (same key, same body → same response)
- Idempotency key reuse with different body (→ 422)

Do not call real Stripe or real SMTP in tests.

### Resources
- **MUST** — [ASP.NET Core — Integration tests](https://learn.microsoft.com/en-us/aspnet/core/test/integration-tests)

---

## Sprint Deliverables

- [ ] Stripe Checkout Session creation
- [ ] PaymentAttempt tracking
- [ ] Webhook endpoint with raw body and signature verification
- [ ] Handle `checkout.session.completed`, `checkout.session.expired`, `payment_intent.payment_failed`
- [ ] Amount/currency verification
- [ ] Webhook idempotency (ProcessedStripeEvent)
- [ ] Idempotency-Key on donation creation and checkout
- [ ] Temporary stub removed
- [ ] Stripe test mode configured
- [ ] Integration test infrastructure (WebApplicationFactory)
- [ ] At least 7 integration tests covering critical flows
- [ ] No real Stripe or SMTP calls in tests

## Definition of Done

- [ ] Checkout creates a Stripe session and returns the URL
- [ ] PaymentAttempt created for each session
- [ ] Webhook verifies signature (invalid → 400)
- [ ] Successful payment triggers the full success flow
- [ ] Duplicate webhook ignored (200 returned)
- [ ] Amount mismatch logged and donation not completed
- [ ] Late payment on expired campaign accepted with warning
- [ ] Stub endpoint removed
- [ ] Idempotency-Key required on donation and checkout endpoints
- [ ] Same key + same body → stored response replayed
- [ ] Same key + different body → 422
- [ ] Integration tests pass with `dotnet test`
- [ ] Auth flow tested end-to-end
- [ ] Webhook flow tested with local signature
- [ ] Idempotency tested
- [ ] No secrets in repository
- [ ] Learner can explain: "Why webhook, not success URL?"

## Review Rule

The learner should be able to explain not only **what** was implemented, but **why it belongs there, what alternatives existed, and what trade-off was accepted**.