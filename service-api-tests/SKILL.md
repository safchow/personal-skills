---
name: service-api-tests
description: >-
  Scaffold and expand black-box REST API tests that target a single
  backend service and verify its behavior only through its public HTTP API
  using Playwright, treating the framework and internals as a black box. Use
  when adding a black-box API suite to a repo, writing new endpoint tests, or
  standardizing how service-level HTTP tests are structured, seeded, and
  isolated. These are single-service, black-box tests — not cross-system e2e.
disable-model-invocation: true
---

# Service API Tests

Build a Playwright suite of black-box API tests for one backend service. The test boundary is that single service: drive it over HTTP and verify only what the API exposes — never call a real third party, drive a UI, or coordinate other services (that would be cross-system e2e). "Regression" is a role these play in CI, not their scope.

Templates ship as raw text with a `.txt` suffix so no compiler or linter reads them here. On copy into a target repo, remove the trailing `.txt`.

## Rules (non-negotiable)

- Verify only over HTTP. Assert on status codes, response bodies, and side effects confirmed by a follow-up API read. No app imports, no store peeks in assertions.
- Seed through the service's endpoints. When a resource has no create endpoint, seed it directly into an isolated test database via the db helper — prefer building the row through the service's own model/factory so it matches production shape — and test its real creation path (webhook or callback) as that endpoint's own spec. Never add a test-only backdoor.
- Isolate with `uniqueId`, not resets. Give each test uniquely-named data and assert only on what it created; never on global counts. Use a `beforeEach` reset only if the service exposes a test-only reset endpoint.
- One endpoint per spec. One spec file per endpoint, named after it; a single top-level describe named "<METHOD> <path>"; one test per coverage-matrix row, named as a behavior statement. Each test body is Arrange / Act / Assert with a single call to the endpoint under test (a follow-up read to confirm a side effect is fine).
- Keep setup in fixtures. All state creation goes through named helpers; never inline plumbing in a spec. Store access is confined to the db helper and used only to seed.
- Match the templates. Follow their layout, naming, and assertion helpers so every suite reads the same.

## Template files

- templates/playwright.config.ts.txt — runner config; base URL from env, serial workers, optional `webServer`.
- templates/helpers/client.ts.txt — base URL, the `uniqueId` generator, authed request-context factory.
- templates/helpers/assertions.ts.txt — `expectOk`, `expectStatus`, `expectErrorCode`.
- templates/helpers/fixtures.ts.txt — example builders (`createUser`, `loginAs`, and a db-seed example).
- templates/helpers/db.ts.txt — OPTIONAL isolated-test-DB access; only for seeding resources with no create endpoint.
- templates/example-create.spec.ts.txt — create endpoint spec; carries the canonical structure doc.
- templates/example-detail.spec.ts.txt — item read endpoint spec (authz, unknown-id).
- templates/example-delete.spec.ts.txt — item delete endpoint spec (side-effect read, idempotency).

## Layout

- Put the suite under a top-level tests directory in the target repo: the Playwright config, a helpers directory, and one spec file per endpoint.
- Name each spec after its endpoint (e.g. resource-create.spec.ts, resource-detail.spec.ts).

## Setup workflow

1. Add the Playwright test package as a devDependency and a script that runs it.
2. Copy the templates in, dropping the `.txt` suffix, and set the base-URL env var (or edit the default).
3. Adapt the fixtures to the service's real create/login endpoints and the error-code helper to its envelope shape (service-specific — e.g. body.error.code vs body.code). Only if a resource has no create endpoint, copy the db helper and point TEST_DATABASE_URL at an isolated test database.
4. Write the first spec against the auth or entry surface, then run it against a running instance and confirm it passes.

Run the service however the repo already does and point the base-URL variable at it; do not stand up databases or migrations inside the suite. For local runs, prefer booting via the config's `webServer` block — and when it injects a test env (e.g. a test DATABASE_URL), set those vars before the service's own dotenv loader runs, or a dev `.env` can leak in and point the suite at the wrong store.

## Coverage matrix (per endpoint)

Cover the rows that apply to the endpoint; not all apply to each (a collection POST has no 404; an item GET has no 422).

- Happy path: valid input returns the right status and body; a follow-up read reflects the change.
- Validation: malformed input returns the validation status (typically 422) and the expected error code.
- Boundaries: at-limit values — exactly equal, one over, zero or empty.
- Authz/authn: unauthenticated (401) and forbidden or not-owner (403).
- State conflicts: wrong-state resources (409) and unknown IDs (404).
- Idempotency/side effects: repeat and cancel flows leave observable state consistent across follow-up reads.

## CI integration

- Run the suite as a required PR check that blocks merge on failure — see the ci-merge-gating skill for the mechanics.
- Point the base-URL at the instance the pipeline boots; provisioning the service and its store is the pipeline's job, not the suite's.
- Verify only over HTTP in CI; if a spec seeds directly, point the db helper at that same isolated store.

## Expanding the suite

- New endpoint: add a spec file named for it and walk the coverage matrix.
- New entity: add a fixture builder (API-driven, or a db seed if it has no create endpoint).
- New error code or auth scheme: adapt the relevant helper only.
