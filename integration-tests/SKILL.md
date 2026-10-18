---
name: integration-tests
description: >-
  Write integration canaries in opulus-tests/integration that call a real
  third party (Plaid sandbox, breached-password API, Frankfurter) through our
  service. Use only when the stub cannot answer the question: do our code and
  the provider still agree? Never for business logic.
disable-model-invocation: true
---

# Integration Tests (canaries)

Placement: test-overview skill. If the stub could answer it, it is a `services` test.

Canaries only: one test per real call, run against the real Plaid sandbox / Frankfurter, needing credentials or a tunnel. Today: link token create, item create from a sandbox public token, breached-password API, one `SYNC_UPDATES_AVAILABLE` webhook delivered by Plaid, the exchange-rate backfill.

Templates (`templates/`, `.txt` suffix, drop on copy) are generic starting points; where they disagree with this file, this file wins.

## Rules (non-negotiable)

- Business logic never lives here. Each test asserts the call works and the shape is understood, not business results (no exact figures from a provider).
- Adding one needs a reason the stub can't stand in. Everything else about that behaviour goes in `services`.
- Still black-box: drive our own endpoints and verify through follow-up reads. Prove the round-trip happened with evidence (a sandbox token prefix, a non-empty shape-conforming list), not just a 200.
- Match gating to credentialing. A required CI check must hard-fail when secrets are missing; an optional check may `test.skip()`. Never a required check that silently self-skips. Treat dummy sentinels (e.g. `ci-test`) as absent.
- Fresh external resources per test; no shared `beforeAll` minting one sandbox resource. Mint one-shot tokens fresh per attempt.
- DB reads only for setup lookups; writes go through HTTP.
- Plaid's sandbox is flaky: retries are set per config in `defineSuite`. Don't raise them to hide a regression. Retries absorb transient provider 5xx, never flaky assertions.
- Don't over-pin the external contract: shape checks age better than value checks; no specific external ids unless guaranteed.
- Never print real credentials; local sandbox credentials never leave the machine.

## Layout

- `integration/<svc>/`, one CI workflow per suite type (separate from the services gate; see ci-merge-gating). Config via `defineSuite` (test-shared skill).
- Suffix describe titles with the dependency, e.g. "POST /api/<resource> (sandbox)". Keep workflow job names stable; branch protection keys off them.
- Triggers: pull_request + schedule + workflow_dispatch. Service and test helper must point at the same external account.

## When not to add one

If a sandbox doesn't unlock meaningful coverage, document the gap and defer. Webhook receivers need a publicly reachable URL (tunnel); Plaid's JWT can't be forged, which is why webhook handlers are tested in services with the stub's signing key.
