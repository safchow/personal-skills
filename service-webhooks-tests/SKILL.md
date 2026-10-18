---
name: service-webhooks-tests
description: >-
  Write specs in opulus-tests/services/webhooks: signed Plaid webhooks,
  handler behaviour read back from the DB, retries and dead events, and the
  reconcile/replay/fire-webhook scripts. Use when adding or changing a
  webhook handler or script.
disable-model-invocation: true
---

# Webhooks Service Tests

Placement: test-overview skill. Shared helpers and stub controls: test-shared skill.

## Delivering webhooks

- The receiver verifies Plaid's JWT, so sign with the stub's key: `postSignedWebhook(request, event, options)`. `options` breaks the signature on purpose: `key: "expired" | "unknown"`, `forged`, `issuedAt`, `claimedHash`, `bodyAfterSigning`, `rawBody`. The body is sent as a `Buffer`, because Playwright re-quotes a string body that is not JSON.
- To test a handler: `deliverWebhook(request, event)` posts it signed and waits until the worker has recorded one more processed `webhook_event` for that `item_id`. Then read what the worker stored: `readSyncedTransactions`, `readHoldings`, `readStreams`, `readLiabilities`, `readItemStatus`, `readItemSync`, `readWebhookEvents`.
- Set up with `seedUser()` + `seedItemWithAccount(user.id)`. The stub knows the item by its access token (`readItemAccessToken(itemId)`); per-token queues (`setPlaidSyncPages(accessToken, pages)`) mean a retried job from another test can't take your pages. Use `item.providerAccountId` for the account id Plaid refers to.

## Pitfalls

- Identical payloads inside 60 seconds are one job (BullMQ dedup on a fingerprint of the body). Two webhooks for one item in a test need a differing field (`new_transactions`, `environment`). If `deliverWebhook` times out on the second delivery, this is why.
- A permanent failure for a retry/dead-event test is "Plaid always fails the sync": queue several `{ error_code: "INTERNAL_SERVER_ERROR" }` pages. A webhook for an item that does not exist is ignored, not a failure; don't use it.
- "Nothing to give" errors (`PRODUCT_NOT_READY`, `PRODUCTS_NOT_SUPPORTED`...) must end `processed` in one attempt and leave stored data as it was. Assert both.
- For a code a handler doesn't act on, assert `processed`, `attempts: 1`, nothing stored, and (where it matters) that the stub was not asked.

## Scripts and queue

- `reconcile`, `replay`, `fire-webhook` run through `runWebhooksScript(script, args, env)` in `helpers/script.ts`. Pass `NODE_ENV=production` when asserting on a refusal message (the dev logger's output is lost on exit). Quotes in the JSON output are escaped; assert on fragments.
- `helpers/queue.ts` puts an event on the queue directly with chosen job options (short backoff, 1 attempt) for audit-trail tests.
- Syncs run from webhooks and link time, never an API endpoint.
