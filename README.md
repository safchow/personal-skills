# personal-skills

Agent Skills for the Opulus test suite (`opulus-tests`) and CI. Start with `test-overview`.

## Skills

| Skill | What it does |
| --- | --- |
| [`test-overview`](test-overview/SKILL.md) | Which directory a test belongs in (unit, services, deployment, integration) and the conventions every suite shares. |
| [`test-shared`](test-shared/SKILL.md) | `shared/`: scenario builders, fixtures, Plaid stub controls, typing, `defineSuite` config. |
| [`unit-tests`](unit-tests/SKILL.md) | `unit/core`, `unit/webhooks`: pure logic, no service or DB. |
| [`service-api-tests`](service-api-tests/SKILL.md) | `services/backend`: black-box HTTP specs against the backend and the Plaid stub. |
| [`service-webhooks-tests`](service-webhooks-tests/SKILL.md) | `services/webhooks`: signed webhooks, handlers, retries, scripts. |
| [`service-scheduler-tests`](service-scheduler-tests/SKILL.md) | `services/scheduler`: jobs against the Frankfurter stub. |
| [`deployment-tests`](deployment-tests/SKILL.md) | `deployment/`: boot, shutdown, config, images, migrations. |
| [`integration-tests`](integration-tests/SKILL.md) | `integration/`: real-provider canaries only. |
| [`ci-merge-gating`](ci-merge-gating/SKILL.md) | Wire a check into CI as a required status check that blocks PR merges. |
