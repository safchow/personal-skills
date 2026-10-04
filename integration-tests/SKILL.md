---
name: integration-tests
description: >-
  Scaffold and expand black-box integration tests that drive a backend
  service's real external dependencies (sandbox APIs, message brokers, identity
  providers) end-to-end through its public HTTP API using Playwright. Sibling to
  the service-api-tests skill: these cover the real external round-trip that
  service-api tests intentionally skip. Use when adding integration coverage for
  provider-proxying endpoints, writing sandbox round-trip tests, or gating them
  in CI. Not full e2e — no UI, no cross-service orchestration.
disable-model-invocation: true
---

# Integration Tests

Build a Playwright suite that exercises one backend service while driving at least one real external dependency end-to-end (a provider sandbox, a real broker, a real identity provider). Still black-box: you call the service's own HTTP endpoints and verify through follow-up API reads; the external dependency is exercised through the service, never asserted on directly. Still not e2e — no UI, no cross-service orchestration.

## Relationship to service-api-tests

A sibling suite, not a replacement. Service-api tests own everything expressible as pure JSON against the service alone: 401, validation, state conflicts, shape. Integration tests own everything that requires an actual external round-trip: provider happy paths, duplicate/idempotency short-circuits, and authz on real external-backed resources. Add integration coverage for every provider-proxying endpoint whose service-api test only asserts 401/validation. Neither suite replaces the other.

Templates ship as raw text with a `.txt` suffix so no compiler or linter reads them here. On copy into a target repo, remove the trailing `.txt`.

## Rules (non-negotiable)

- Still black-box over our HTTP. Drive the service's own endpoints; verify through follow-up API reads. Exercise the external dependency through the service, never assert on it directly.
- Match gating to credentialing. A required CI check must hard-fail when secrets are missing; an optional check may `test.skip()`. Never a required check that silently self-skips — it goes green without exercising anything. Treat dummy sentinels (e.g. `ci-test`) as absent.
- Fresh external resources per test. No shared `beforeAll` that mints one sandbox resource for the whole file; each test mints its own so a single failure is debuggable without reconstructing shared state. Slower, worth it.
- Mint one-shot tokens fresh per attempt, and use freshness as a regression guard. When testing a duplicate/short-circuit path, use a fresh second token so that if the code regresses and takes the long path the token would actually succeed — making the test fail correctly.
- DB reads are allowed for setup lookups only. Reading a field the DTO intentionally hides (e.g. an external id the endpoint takes) for test setup is in-scope; writes still go through HTTP, and verification is still over HTTP.
- Bump timeouts, allow retries. External calls take seconds (e.g. timeout 60s, retries 2 in CI, workers 1). Retries absorb transient external 5xx — never flaky assertions.
- Prove the round-trip happened. Assert evidence the external call really occurred (a sandbox token prefix, a non-empty shape-conforming list, created:0/updated>0 after an idempotent reconcile), not just a 200.
- Don't over-pin the external contract. Shape checks age better than value checks; avoid asserting specific external IDs unless the provider guarantees them.

## Template files

- templates/playwright.config.ts.txt — 60s timeout, workers:1, retries:2 in CI, shared globalTeardown.
- templates/helpers/externalSandbox.ts.txt — `requireCredentials()` plus an end-to-end `createExternalResource()` that mints and persists via our own HTTP so downstream specs reuse it.
- templates/example-happy-path.spec.ts.txt — mint → persist → verify-via-list.
- templates/example-duplicate-regression.spec.ts.txt — fresh-token-as-regression-guard for a short-circuit path.
- templates/example-authz.spec.ts.txt — owner vs. intruder on a real external-backed resource.
- templates/integration-tests.yml.txt — workflow with pull_request + schedule + dispatch + required secrets.

## Layout (axis-first)

Promote the gating axis (service-contract vs. external round-trip) to the top-level directory so each CI workflow maps 1:1 to one subtree:

- `shared/` — helpers reused by both suites (auth, client, db, assertions, generic fixtures).
- `services/<svc>/` — service-api tests → service-api-tests.yml.
- `integration/<svc>/` — integration tests → integration-tests.yml.

External-dependency-specific helpers (e.g. the sandbox helper) live only in the integration suite's own `helpers/`; shared helpers move to `shared/` when a service appears in both suites. Suffix describe titles with the external dependency so logs distinguish them (e.g. "POST /api/<resource> (sandbox)"). Name scripts `test:services-<svc>` / `test:integration-<svc>`, and keep workflow job names stable — branch protection keys off them.

## Setup workflow

1. Confirm the dependency has a usable test mode (sandbox/stage account, deterministic payloads) and you hold credentials for it. If it does not unlock meaningful coverage, defer (see "When not to add integration tests").
2. Decide gating: required (hard-fail without secrets) or optional (self-skip). This drives the credential helper and the workflow.
3. Copy the templates into `integration/<svc>/`, dropping the `.txt` suffix; put the external helper in `integration/<svc>/helpers/` and reuse `shared/` for auth, client, db, and assertions.
4. Put the credential helper at the top of the suite; treat dummy sentinels as absent.
5. Write the first happy-path spec (mint → persist via our HTTP → verify via a follow-up read), run it with real credentials, and confirm the round-trip evidence.
6. Add the workflow (pull_request + schedule + dispatch) and register its job name as a required status check.

## Scope: what to cover

- Every provider-proxying endpoint whose service-api test only asserts 401/validation gains a happy path plus at least one authz case (401/403 for an intruder on another user's resource).
- Cover duplicate/idempotency short-circuits with the fresh-token regression-guard pattern.
- Prove the round-trip with evidence, not just a 200. Do not over-pin external IDs.

## When not to add integration tests

If a sandbox/test mode does not unlock meaningful coverage, document the gap and defer rather than forcing a bad test. Example: webhook receivers need the external service to POST to a publicly reachable URL (a tunnel); sandbox credentials alone are not enough. List the preconditions (reachable URL, forgeable or deterministic signatures, deterministic payloads) and defer until they are met.

## CI integration

- One workflow file per suite type; integration gets its own, separate from the service-api gate (see the ci-merge-gating skill for the gating mechanics).
- Triggers: pull_request + schedule (nightly, to catch regressions on days with no PR traffic) + workflow_dispatch (to re-run after an external outage).
- Inject real secrets with no dummy fallback when the check is required. The service under test and the test helper must point at the same external account so round-trips succeed.
- Register the job name in the branch ruleset's required status checks (the ruleset API expects the full ruleset body, not a patch). Keep job names stable.

## Expected spillover

Integration tests surface bugs service-contract tests cannot, because the latter pin the handler's self-reported shape against stubs. Expect to fix a handful in the first PR; land them as separate commits so reviewers see the extra scope. Concrete example: a controller that parsed input inside its own try/catch surfaced bad input as a 500 — the service-api test happily asserted 500, while the real round-trip exposed the gap by expecting 400.

## Expanding the suite

- New provider-proxying endpoint: add a spec under `integration/<svc>/`, suffix the describe with the dependency, and cover happy path + authz.
- New external dependency: add its own `helpers/` module (credential gate + an end-to-end create helper); keep shared helpers in `shared/`.
- New gating: wire the job name into branch protection.
