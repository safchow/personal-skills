---
name: test-shared
description: >-
  Rules for opulus-tests/shared: fixtures, scenario builders, Plaid stub
  controls, typing, and suite config (defineSuite). Use when adding a builder,
  fixture, stub endpoint or suite config, or when a helper is about to be
  copied between specs.
disable-model-invocation: true
---

# Shared Test Infrastructure

Applies to `opulus-tests/shared/`, used by every suite. Placement of tests: test-overview skill.

## Typing

- The tests package is type-checked (`pnpm --filter @opulus/tests type-check`, in the Type Check workflow).
- Control payloads use `Plaid<T>` (a partial of an SDK response type, enums read as strings): a field the SDK doesn't have fails the check.
- The stub's complete answers use `Wire<T>` with `@satisfies` in the `.mjs` JSDoc. A new stub endpoint gets its SDK response type the same way; a new control gets a typed function in `shared/plaid-stub.ts`.

## Scenarios

- Builders live in `shared/scenarios/`, shared by every suite that needs them. Builders: `plaidAccount`, `creditCardAccount`, `loanAccount`, `liabilitiesFor`, `holding`/`apple`/`cash`, `stream`/`payroll`, `plaidTransaction`.
- A builder that three specs would copy is a scenario; a one-off stays in the spec.
- Account/user helpers in `shared/scenarios/plaid.ts`: `createAuthedUser`, `createLinkedUser`, `linkInstitution`.

## Fixtures

- `shared/fixtures/` holds resource-scoped files re-exported by `index.ts`: a seeder (`seedX`) for what has no create endpoint, a reader (`readX`) for what has no read endpoint.
- Readers go through the isolated `TEST_DATABASE_URL`, never the dev DB.
- Never write to the DB to force a state a request or the stub can create.

## Stub controls

`setPlaidItem`, `setPlaidHoldings`, `setPlaidRecurring`, `setPlaidLiabilities`, `setPlaidSyncPages`, `failAccounts`, `failItemRemoval`, `resetPlaidStub()` in `shared/plaid-stub.ts`.

- Keep the stub minimal (only fields our code reads), deterministic, and resettable (`/__control/reset`). A stub that grows its own logic is a second product.
- Stub drift (Plaid changing its API) is defended by the SDK pin, the typed stub, and the integration canaries. Don't add more machinery.

## Config

Every suite's `playwright.config.ts` is `defineSuite({ project, timeout, ciRetries, port, webServer?, parallel? })` from `shared/suite.ts`. Teardown is the shared `global-teardown.ts`. Don't hand-write a config.
