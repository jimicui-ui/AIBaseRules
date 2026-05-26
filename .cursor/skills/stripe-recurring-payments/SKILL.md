---
name: stripe-recurring-payments
description: Implements Stripe Subscriptions for monthly recurring billing, including client-side payment method setup, backend subscription creation, updating cards/payment methods, and canceling at period end. Use when the user asks to integrate Stripe via https://docs.stripe.com/, handle monthly recurring charges, or manage subscriptions (pay, update payment details, cancel).
disable-model-invocation: true
---

# Stripe Recurring Payments (Subscriptions)

## Goal
Add Stripe integration that supports:
1) Customer pays via the user page (initial setup + subscription creation)
2) Monthly recurring billing handled by Stripe (backend monitors via webhooks)
3) Client updates card/payment info (update subscription default payment method)
4) Client cancels subscription at period end

## Core Stripe objects to use
- `Customer`: represents your app user in Stripe
- `PaymentMethod`: the card instrument (from the client)
- `SetupIntent`: confirm card details on the client for off-session usage
- `Subscription`: monthly recurring billing container
- Webhook events: keep your database in sync with Stripe state

## Security and correctness rules
- Never expose Stripe secret keys to the browser.
- Always verify webhook signatures using the webhook signing secret.
- Never trust `customerId` / `subscriptionId` provided by the client; derive from the authenticated user.
- Use Stripe idempotency keys for create/attach/update operations that may be retried.

## Required inputs to ask for (before writing code)
1) Your Stripe `price` (or recurring plan) id for monthly billing
2) Whether you use one subscription per user (typical) and whether you allow multiple plans
3) Your app’s user identity mapping:
   - how to find the Stripe `customerId` for the logged-in user
   - how you store `subscriptionId` and current status
4) Webhook endpoint details:
   - the URL your backend exposes
   - the webhook signing secret

## End-to-end flow (recommended)

### 1) User clicks “Pay” on the user page (initial payment setup + subscription creation)
Backend creates a `SetupIntent` for the current user’s Stripe `Customer`:
1. Create or retrieve `Customer` for the authenticated user.
2. Create `SetupIntent` with:
   - `customer`: Stripe customer id
   - `usage`: `off_session` (so future invoices can charge the card)
   - `payment_method_types`: e.g. `card`
3. Return `client_secret` to the frontend.

Frontend confirms the setup with Stripe.js (Angular):
1. Collect card details with Elements (or Payment Element).
2. Call `stripe.confirmCardSetup` using the returned `client_secret`.
3. Send the resulting `payment_method` id (e.g. `paymentMethod.id`) to the backend.

Backend creates the `Subscription`:
1. Create the `Subscription` with:
   - `customer`
   - `items: [{ price: <monthly-price-id> }]`
   - `default_payment_method`: the confirmed `payment_method` id
   - (optional) `payment_settings` depending on your Stripe setup
2. Store `subscriptionId` and initial status (until the first webhook updates it).

### 2) Client updates card info (update payment method on existing subscription)
Typical best practice: reuse the same flow as initial setup, then update subscription defaults.
1. Backend creates a new `SetupIntent` for the existing Stripe `Customer`.
2. Frontend confirms it and returns the new `payment_method` id.
3. Backend:
   - attaches the new `PaymentMethod` to the Stripe customer (if needed)
   - updates the existing `Subscription` to use it as the default payment method
4. If you need to charge immediately for any open invoices, pay them (optional; only if your product requires it).

### 3) Client cancels (cancel at period end)
1. Backend updates the existing `Subscription`:
   - set `cancel_at_period_end = true`
2. Rely on webhook events to update your UI/state:
   - `customer.subscription.updated` (shows next billing/cancellation time)
   - `customer.subscription.deleted` (if Stripe cancels immediately or finalizes)

## API endpoint plan (suggested .NET backend)
Implement endpoints roughly like:

- `POST /api/stripe/create-setup-intent`
  - Auth required
  - Input: none or optional preferences
  - Output: `{ clientSecret, customerId? (server-only), setupIntentId }`

- `POST /api/stripe/subscriptions/create`
  - Auth required
  - Input: `{ paymentMethodId }`
  - Output: `{ subscriptionId, status }`

- `POST /api/stripe/payment-method/update`
  - Auth required
  - Input: `{ paymentMethodId }`
  - Output: `{ subscriptionId, defaultPaymentMethod, status }`

- `POST /api/stripe/subscriptions/cancel-at-period-end`
  - Auth required
  - Input: none
  - Output: `{ subscriptionId, cancelAtPeriodEnd, currentPeriodEnd }`

- `POST /api/stripe/webhook`
  - No app auth required (but verify Stripe signature)
  - Input: raw body + `Stripe-Signature` header
  - Output: 2xx on success

## Webhook handler checklist (keep your DB synced)
Handle these events at minimum:
- `invoice.paid`
  - update your “last paid” timestamp
  - update subscription period info if present
- `invoice.payment_failed`
  - flag billing issue for the user
- `customer.subscription.updated`
  - update status fields like: `cancel_at_period_end`, `current_period_end`, etc.
- `customer.subscription.deleted`
  - mark subscription as canceled/ended in your DB

Helpful extras (depending on your needs):
- `charge.refunded` (if you support refunds in-app)
- `payment_intent.*` (only if it affects your app state)

## Database fields to store (minimum)
- `stripeCustomerId`
- `stripeSubscriptionId`
- `subscriptionStatus` (your normalized status)
- `currentPeriodEnd` (from Stripe)
- `cancelAtPeriodEnd` (from Stripe)
- optional: card display fields (brand/last4) if you want UX improvements

## C# implementation notes (minimal guidance)
- Use your Stripe SDK for .NET.
- When handling webhooks, read the request body as the exact raw bytes required by Stripe signature verification.
- Wrap create/update operations with idempotency keys where retries are possible.

## Example request/response shapes

`POST /api/stripe/create-setup-intent`
Response:
```json
{
  "clientSecret": "seti_...",
  "setupIntentId": "seti_..."
}
```

`POST /api/stripe/subscriptions/create`
Request:
```json
{ "paymentMethodId": "pm_..." }
```
Response:
```json
{ "subscriptionId": "sub_...", "status": "active" }
```

`POST /api/stripe/payment-method/update`
Request:
```json
{ "paymentMethodId": "pm_..." }
```

`POST /api/stripe/subscriptions/cancel-at-period-end`
Response:
```json
{
  "subscriptionId": "sub_...",
  "cancelAtPeriodEnd": true,
  "currentPeriodEnd": 1710000000
}
```

## Notes for the agent when implementing UI/backend coordination
- The client should only handle “collect card details + confirm SetupIntent”. The subscription creation/update should be performed server-side.
- Treat “status” as webhook-driven: the UI should refresh from your DB after webhook processing (or via polling) to avoid race conditions.

