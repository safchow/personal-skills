---
name: service-scheduler-tests
description: >-
  Write specs in opulus-tests/services/scheduler against the Frankfurter
  stub: run jobs through the jobs API, check stored results, failure and
  recovery. Use when adding or changing a scheduler job.
disable-model-invocation: true
---

# Scheduler Service Tests

Placement: test-overview skill. Shared helpers: test-shared skill.

- The scheduler runs against the Frankfurter stub (`services/scheduler/stub/frankfurter.mjs`, started by the config's `webServer`). Controls: `resetStub`, `failStub`, `stubRequests`.
- Drive a job through the jobs API with `runToEnd(request, job)` (`shared/scheduler.ts`); it returns the finished run. Read what the job stored from the DB. The table is shared, so clear it in `beforeEach`.
- Test the failure and the recovery in the same file: provider down → run `failed`, table unchanged; provider back → next run catches up.
- The `run-job` script runs through `execFile pnpm --filter @opulus/scheduler run-job`. When asserting on a refusal message, pass `NODE_ENV=production`: the dev logger writes from another thread and its output is lost when the script exits right after logging.
- Wait with `runToEnd`, never a fixed sleep.
