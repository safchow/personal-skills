---
name: service-regression-tests
description: >-
  Scaffold and expand black-box REST regression tests that target a single
  backend service through its public HTTP API using Playwright, independent of
  the framework, database, and every other layer (all treated as black boxes).
  Use when adding a regression suite to a repo, writing new endpoint tests, or
  standardizing how REST integration tests are structured, seeded, and isolated.
disable-model-invocation: true
---

# Service Regression Tests

Purpose: build a Playwright regression suite that exercises one backend service through its public REST API only. Everything behind the API is a black box (framework, database, cache, queue). Assert on observable behavior: status codes, response bodies, and side effects confirmed by follow-up API reads.

Templates ship as raw text with a `.txt` suffix so no compiler or linter reads them here. On copy into a target repo, remove the trailing `.txt` from each file.

## Rules (non-negotiable)

- REST only. Talk to the running service over HTTP. No app imports, no DB/ORM access, no queue or cache pokes. If it is not observable through the API, do not assert on it.
- Seed through the API. Create all setup state by calling the service's own endpoints. Never write to a store directly.
- Isolate with unique names, not resets. Give each test uniquely-named data via the `uniqueId` helper; assert only on what that test created. Never assert on global counts.
- Reset only if offered. Call a reset endpoint in `beforeEach` only if the service exposes a test-only one; otherwise rely on `uniqueId` namespacing.
- Setup lives in fixtures. All setup goes through named helper functions in the fixtures file. Never inline API plumbing in a test. Adding a resource means adding a builder.
- Match the templates. Follow their layout, naming, and assertion helpers exactly so every suite reads the same.

## Template files (read before scaffolding)

- templates/playwright.config.ts.txt — runner config; base URL from env, serial workers.
- templates/e2e/helpers/client.ts.txt — base URL, the `uniqueId` generator, authed request-context factory.
- templates/e2e/helpers/assertions.ts.txt — the `expectOk`, `expectStatus`, and `expectErrorCode` helpers.
- templates/e2e/helpers/fixtures.ts.txt — example API-driven builders (`createUser`, `loginAs`) to adapt.
- templates/e2e/example.spec.ts.txt — spec showing the required shape and coverage.

## Layout

- Put the suite under a top-level tests directory in the target service repo.
- It holds the Playwright config plus an e2e directory.
- Inside e2e: a helpers directory (client, assertions, fixtures) plus one spec file per resource or API surface.
- Group related endpoints in a describe block named for the HTTP method and path.

## Setup workflow

1. Add the Playwright test package as a devDependency.
2. Copy the templates into the target repo's tests directory, dropping the `.txt` suffix; set the base-URL environment variable (or edit the default in client and config).
3. Adapt the fixtures module to the service's real create and login endpoints.
4. Adapt the assertions module's error-code helper to the service's error envelope shape.
5. Add a test script that runs Playwright.
6. Write the first spec against the service's auth or entry surface.
7. Run the suite against a running instance and confirm it passes.

The service under test is a black box: run it however the repo already does (dev command, container, deployed instance) and point the base-URL variable at it. Do not stand up databases or migrations inside this suite — that is the service's job.

## Spec conventions

- Import only from the helpers modules. Never touch app internals or a database.
- Seed via fixtures (API calls); namespace created entities with `uniqueId` for isolation. Add a reset hook only if the service exposes a test-only reset endpoint.
- Group by endpoint with a describe named for the HTTP method and path; nest a further describe for a coherent cluster of cases.
- Name tests as behavior statements: what happens under what condition (e.g. rejects a wager that exceeds the balance; requires authentication).
- Attach auth per-request with a bearer authorization header, or use the authed-context factory for a token-preset context.
- Assert the full outcome: status code, body shape, and any side effect verified via a follow-up API read (never a database peek).

## Coverage matrix (per endpoint)

- Happy path: valid input returns the right status and body; a follow-up read reflects the change.
- Validation: malformed or invalid input returns the validation status (typically 422) and the expected error code.
- Boundaries: at-limit values — exactly equal, one over, zero or empty.
- Authz/authn: unauthenticated (401) and forbidden or not-owner (403).
- State conflicts: wrong-state resources (409) and unknown IDs (404).
- Idempotency/side effects: repeat and cancel flows leave observable state consistent across follow-up reads.

## CI integration

- Run the suite as a required status check on pull requests to the protected branch; block merge on failure.
- Point the base-URL variable at the service instance the pipeline brings up.
- Bringing up the service and its dependencies (database, migrations, containers, seed data) is the service's responsibility, invoked from the pipeline — never from inside the suite. Mirror whatever the repo already does to boot the service.
- Keep the black-box contract in CI too: the pipeline may provision the service's backing store, but the suite still talks to the service over HTTP only and never touches that store.
- For the concrete pipeline, branch-protection, and merge-gating mechanics, use the ci-merge-gating skill.

## Expanding the suite

- New endpoint: add a describe block, or a new spec file for a new resource, and walk the coverage matrix.
- New entity: add an API builder to the fixtures module; never inline the plumbing in a spec.
- New error code: assert it with the error-code helper; keep envelope handling in the helper.
- New auth scheme: adapt the authed-context factory and login helper only.
