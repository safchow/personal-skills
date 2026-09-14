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

Build regression suites that exercise a service through its public REST API only and assert on observable behavior — status codes and response bodies. Everything behind the API is a black box: the framework, the database, caches, queues. The suite never imports app internals and never touches the database directly. Seeding and isolation happen through the API. Playwright's API testing client is the runner in every repo for consistency.

The template files referenced below are stored as raw text with a `.txt` extension so they are not picked up by any compiler or linter in this skills repo. When scaffolding a suite, copy each template into the target repo and rename it back to its real extension (drop the trailing `.txt`).

## Core principles

1. REST black box. Talk to the running service over HTTP only. No app imports, no DB/ORM access, no queue or cache pokes. If you can't observe it through the API, don't assert on it.
2. Seed through the API. Create every piece of setup state by calling the service's own endpoints (signup, create-resource, etc.), never by writing to a store. This keeps the suite portable across any backing database.
3. Isolation without a shared reset. Because the DB is a black box, achieve isolation by giving each test its own uniquely-named data (see the `uniqueId` helper in the client template) and asserting only on what that test created. Never assert on global counts. If — and only if — the service exposes a test-only reset endpoint, call it in a beforeEach hook.
4. Modular fixtures. All setup goes through named helper functions in the fixtures file, never inline API-plumbing inside a test. Adding a resource means adding a builder.
5. Consistency over cleverness. Follow the layout, naming, and assertion helpers in the templates exactly, so every service's suite reads the same way.

## Template files

This skill ships copy-paste templates. Read them when scaffolding; they are the source of truth for structure and are meant to be dropped into a repo and edited. Each is stored with a `.txt` suffix that must be removed on copy.

- templates/playwright.config.ts.txt — runner config; base URL from env, serial workers.
- templates/e2e/helpers/client.ts.txt — base URL, the `uniqueId` generator, and an authed request-context factory.
- templates/e2e/helpers/assertions.ts.txt — the `expectOk`, `expectStatus`, and `expectErrorCode` helpers.
- templates/e2e/helpers/fixtures.ts.txt — example API-driven builders (`createUser`, `loginAs`) to adapt.
- templates/e2e/example.spec.ts.txt — a spec showing the required shape and coverage.

## Directory layout

Drop the suite into the target service repo under a top-level tests directory. It contains the Playwright config plus an e2e directory. Inside e2e, a helpers directory holds the client, assertions, and fixtures modules, and each API surface gets its own spec file. Use one spec file per resource or API surface, and group related endpoints with a describe block named for the HTTP method and path.

## Setup workflow

When introducing the suite to a new repo, work through these steps:

- Add the Playwright test package as a devDependency.
- Copy the templates into the target repo's tests directory, dropping the `.txt` suffix from each file, and set the base-URL environment variable (or edit the default in the client and config).
- Adapt the fixtures module to the service's real create and login endpoints.
- Adapt the assertions module's error-code helper to the service's error envelope shape.
- Add a test script that runs Playwright.
- Write the first spec against the service's auth or entry surface.
- Confirm the suite runs against a running instance and passes.

The service under test is a black box: run it however the repo already does (dev command, container, deployed instance) and point the base-URL environment variable at it. Do not stand up databases or migrations from inside this suite — that belongs to the service, not the tests.

## Spec file conventions

- Import only from the helpers modules; never touch app internals or a database.
- Seed via fixtures (API calls), and namespace created entities with the unique-id generator for isolation. Only add a reset hook if the service exposes a test-only reset endpoint.
- Group by endpoint with a describe block named for the HTTP method and path; nest a further describe for a coherent cluster of cases.
- Name tests as behavior statements: what happens under what condition (for example, rejects a wager that exceeds the balance, or requires authentication).
- Attach auth per-request with a bearer authorization header, or use the authed-context factory for a token-preset context.
- Assert the full outcome: the status code, the body shape, and any side effects verified with a follow-up API read rather than a database peek.

## Coverage matrix per endpoint

For each endpoint, cover this matrix so regressions surface consistently:

- Happy path: valid input returns the right status and body, and a follow-up read reflects the change.
- Validation: malformed or invalid input returns the validation status (typically 422) and the expected error code.
- Boundaries: at-limit values such as exactly equal, one over, and zero or empty.
- Authz and authn: unauthenticated (401) and forbidden or not-owner (403).
- State conflicts: operating on wrong-state resources (409), and unknown IDs (404).
- Idempotency and side effects: repeat and cancel flows leave observable state consistent across follow-up reads.

## Expanding the suite

- New endpoint: add a describe block, or a new spec file for a new resource, and walk the coverage matrix.
- New entity: add an API builder to the fixtures module; never inline the plumbing in a spec.
- New error code: assert it with the error-code helper; keep envelope handling in the helper.
- New auth scheme: adapt the authed-context factory and login helper only.
