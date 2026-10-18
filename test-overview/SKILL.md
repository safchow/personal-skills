---
name: test-overview
description: >-
  Decide which opulus-tests directory a test belongs in (unit, services,
  deployment, integration) and apply the conventions every suite shares. Read
  first whenever tests are required or new business logic is added.
disable-model-invocation: true
---

# Test Overview

Entry point for `opulus-tests`. Pick the directory, then read that directory's skill.

## Where a test goes

Ask what the test needs, in this order:

1. Pure function, no I/O → `unit/` → unit-tests skill.
2. Our service's behaviour, third party stubbed → `services/<service>/` → service-api-tests (backend), service-webhooks-tests, service-scheduler-tests.
3. Does the process start, stop, refuse bad config, survive a dead dependency, register its schedules? → `deployment/` → deployment-tests skill.
4. Only a real third party can answer it (do our code and Plaid still agree?) → `integration/` → integration-tests skill. Canary only: one test per real call. If the stub could answer it, it is a `services` test.

Business logic is tested in `services` and `unit`. `integration` never carries business logic.

Route protections (401, two-factor, 404 for another user's item, rate limit) are tested once, in `services/backend/routes/protections.spec.ts` and the rate-limit specs. A default-deny guard (`requireApiSession`) enforces them, so a per-endpoint copy only duplicates it.

Shared fixtures, scenarios, stubs and config: test-shared skill. CI gating: ci-merge-gating skill.

## Conventions (every suite)

- AAA with `// Arrange`, `// Act`, `// Assert` comments. Behaviour in the test name, not the function name.
- File-level doc comment: what the file is about, then `Test cases:` listing each test in one line. Reviewers read it to see what is proven.
- Prove a new spec can fail: remove the wiring (or revert the fix), confirm the spec fails, and say so in the PR.
- Before pushing: `type-check` (no unused imports), and run the affected suite for real with a throwaway Postgres and Redis, never the dev ones.
- Output of a script or log is data to assert on, not to print. Never print real credentials; local sandbox credentials never leave the machine.
- A flaky wait is a bug: wait on a condition (`waitForWebhookEvents`, `runToEnd`), never on a fixed sleep.
- PR shape: one concern per PR, test-case list at the top of each spec, no long review sections in the body (guidance goes in inline comments).
- Don't write a test that two endpoints return the same data; it ties two contracts together.
