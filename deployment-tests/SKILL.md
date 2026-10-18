---
name: deployment-tests
description: >-
  Write specs in opulus-tests/deployment (config, images, migrations) that
  start the real service and check boot, shutdown, bad config, dead
  dependencies and schedule registration. Use when changing startup, env
  validation, Dockerfiles or migrations.
disable-model-invocation: true
---

# Deployment Tests

Placement: test-overview skill.

- Specs start the real service from source on a port of their own (`startService` in `deployment/helpers/service.ts`) and stop it. Pick an unused port per spec file.
- They need Postgres and Redis, and a migrated database, except for the migrations suite.
- Stop a service with `SIGTERM` and wait for exit, never `SIGKILL`: `tsx` runs the service as a child, and killing the wrapper leaves the child holding the port, so the next spec's service fails to start with no clear error. `startService`/`stopService` kill the whole process group; never SIGKILL the child directly.
- `deployment/config`: production-config checks through a booted process. The service-by-variable table is a unit test.
- `deployment/images`: builds and runs each Docker image.
- `deployment/migrations`: migrations apply to an empty database. It lists 0 tests unless `DATABASE_URL` is set; that is normal.
- These gate first in CI. Keep them fast and independent of Plaid and third parties.
- Every new migration gets its own PR; code PRs stack on it.
- The image workflow rebuilds only what a change can affect. If you add a file an image copies, add its path to the "shared inputs" list in `deployment-images.yml`.
- A new root `.dockerignore` entry can silently break an image build (excluding the frontend by mistake once). Build every image you could affect. The root `.dockerignore` decides what any image can see; the frontend must stay un-ignored.
