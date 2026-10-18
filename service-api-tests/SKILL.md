---
name: service-api-tests
description: >-
  Write black-box REST API specs for the backend in opulus-tests/services/backend,
  driven over HTTP with Playwright against the running backend and the Plaid
  stub. Use when adding or changing an endpoint or business logic that has an
  HTTP surface. Webhooks and scheduler have their own skills.
disable-model-invocation: true
---

# Backend Service API Tests

Black-box tests of one service over HTTP, with the Plaid stub (`services/backend/stub/plaid.mjs`) as Plaid. No test calls real Plaid; the real round-trip is an integration canary (integration-tests skill). Placement of tests: test-overview skill. Shared builders, fixtures, stub controls and typing: test-shared skill.

Templates (`templates/`, raw text with a `.txt` suffix, drop the suffix on copy) are generic starting points for a new suite elsewhere. In opulus-tests, the rules below win over them: the older flat "service" layout, DB seeding as the default, and real-Plaid calls no longer apply.

## Rules (non-negotiable)

- Verify over HTTP. Read the DB only for what has no read endpoint (transactions' internals, holdings, webhook events, item sync state), through the readers in `shared/fixtures`. Never write to the DB to force a state a request or the stub can create. Seed with fixtures only what has no create endpoint.
- Make a user and a linked item with one call: `createAuthedUser`, `createLinkedUser`, `linkInstitution` (`shared/scenarios/plaid.ts`). Describe what Plaid should say with the scenario builders and the stub controls (`setPlaidItem`, `setPlaidHoldings`, `setPlaidRecurring`, `setPlaidLiabilities`, `setPlaidSyncPages`, `failAccounts`, `failItemRemoval`).
- Call `resetPlaidStub()` in `beforeEach` of any spec that sets the stub.
- Isolate by `uniqueId()` data, not resets. Give each user a fresh `uniqueIp()` so per-IP rate limits don't leak between specs.
- Check response bodies with `expectMatchesSchema(res, SomeResponseSchema)` (the DTO schema from core): it also fails on fields outside the schema. Borrow input types from the source of truth; assert outputs against expected shapes, not the service's own DTOs.
- Don't add a 401/403/two-factor/404-for-others test to a new endpoint spec; the default-deny guard is tested once in `routes/protections.spec.ts`. Add the route there only if it is public or exempt from two-factor, a deliberate list in the backend.
- New routes: the Plaid rate-limit lint rule (#201) fails the build for a non-GET route without `plaidRateLimit`. A route that never calls Plaid needs a disable comment giving the reason (see #43).
- Before adding a stub control, check for an existing one; keep stub changes minimal and typed against the SDK.
- Don't write a test that two endpoints return the same data (a closed PR; it ties two contracts together).
- One endpoint per spec file, named after it; a top-level describe "<METHOD> <path>"; one test per behavior. Single call to the endpoint under test per Act (a follow-up read to confirm a side effect is fine).
- Setup goes through named helpers; never inline plumbing in a spec.

## Coverage matrix (per endpoint)

Cover the rows that apply: happy path (and follow-up read), validation (typically 422 + error code), boundaries (at-limit, one over, zero/empty), state conflicts (409) and unknown ids (404), idempotency/side effects. Authn/authz is covered by the protections spec, not per endpoint. Descriptive test titles are the coverage record.

## Scope

- Product behavior only. Skip health/readiness and framework defaults.
- Provider-proxying endpoints are tested against the stub, including the happy path. Only "do we and Plaid still agree" goes to integration, as a canary.
- Syncs run from webhooks and link time, never an API endpoint; don't add a sync endpoint or test.
