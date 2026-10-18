---
name: unit-tests
description: >-
  Write unit specs in opulus-tests/unit (core, webhooks) for pure logic with
  no service, DB or Redis. Use when adding logic with edge cases (rounding,
  boundaries, enum tables, ordering, nulls) or a mapping table.
disable-model-invocation: true
---

# Unit Tests (`unit/core`, `unit/webhooks`)

Placement: test-overview skill. Conventions (AAA, file doc comment, failing-first) apply.

- Import from `@opulus/core` (needs the core build) or by relative path from `opulus-webhooks/src/...`. No service, DB or Redis.
- `test:unit-core` runs both directories with `LOG_LEVEL=silent` (the logger prints every Plaid error it maps).
- Worth writing when the logic has edge cases an endpoint can't reach cheaply: rounding, boundaries, enum tables, ordering, nulls. Don't unit-test what a service spec already proves through HTTP (field renames, simple DTO mapping).
- Code that reads the DB through a singleton repository can be tested without a DB: replace the repository's methods for the test and restore them in `afterEach` (see `exchangeRateConverter.spec.ts`). Don't mock modules.
- Make a test table-driven when it is a mapping (every Plaid error code → status; every frequency → payments a year).
- A bug found by a unit test goes in the same PR as its fix, with the test failing before the fix (the fingerprint bug, #188).
