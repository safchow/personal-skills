# personal-skills

Portable [Cursor Agent Skills](https://docs.cursor.com) to drop into any repo.

## Skills

| Skill | What it does |
| --- | --- |
| [`service-api-tests`](service-api-tests/SKILL.md) | Scaffold and expand black-box REST API tests for a backend service using Playwright, verifying behavior only through its public HTTP API. |
| [`integration-tests`](integration-tests/SKILL.md) | Scaffold and expand black-box integration tests that drive a service's real external dependencies (sandbox APIs, brokers) end-to-end over HTTP with Playwright. Sibling to `service-api-tests`. |
| [`ci-merge-gating`](ci-merge-gating/SKILL.md) | Wire a check into CI as a required status check that blocks PR merges on failure, via GitHub Actions and branch protection. |
