# personal-skills

Portable [Cursor Agent Skills](https://docs.cursor.com) that can be dropped into any repo.

## Skills

### `service-regression-tests`

Scaffold and expand black-box REST regression tests that target a single backend
service through its public HTTP API using Playwright. The framework, database,
and every other layer are treated as black boxes — the suite talks to the
running service over HTTP only, seeds state through the API, and achieves test
isolation via `uniqueId()` namespacing rather than database truncation.

- `SKILL.md` — codeless guidance for agents; references the templates by path.
- `templates/` — copy-paste starting points stored with a `.txt` suffix so no
  compiler or linter picks them up. Drop the `.txt` when copying into a target
  repo:
  - `playwright.config.ts.txt` — runner config; base URL from `TEST_BASE_URL`.
  - `e2e/example.spec.ts.txt` — coverage-matrix demo using `<resource>` placeholders.
  - `e2e/helpers/client.ts.txt` — `BASE_URL`, `uniqueId()`, `authedContext()`, `bearer()`.
  - `e2e/helpers/assertions.ts.txt` — `expectOk`, `expectStatus`, `expectErrorCode`.
  - `e2e/helpers/fixtures.ts.txt` — API-driven builders (`createUser`, `loginAs`, etc.).

See `service-regression-tests/SKILL.md` for the full workflow and the per-endpoint
coverage matrix (happy path, validation, boundaries, authz/authn, state
conflicts, idempotency/side effects).
