---
name: ci-merge-gating
description: >-
  Wire a test suite or check into CI as a required status check that blocks
  pull-request merges when it fails, using GitHub Actions and branch protection.
  Use when setting up merge gating, adding a required check, provisioning a
  service and its dependencies in a pipeline, or standardizing how CI prevents
  bad merges.
disable-model-invocation: true
---

# CI Merge Gating

Purpose: run a check (tests, typecheck, lint) on every pull request and prevent merge when it fails. Requires two independent parts: a CI workflow that runs the check, and branch protection that marks that workflow's job as required.

A workflow that only runs does not gate merges. Gating takes effect only once the job is a required status check in branch protection. Both parts are mandatory.

Templates ship as raw text with a `.txt` suffix so no tool runs them here. On copy into a target repo, remove the trailing `.txt`.

## Rules (non-negotiable)

- One check, one job. Each required check gets its own job with a stable, descriptive name. Branch protection matches jobs by name; renaming a job breaks the gate silently.
- Provision in the pipeline. The pipeline stands up any service and dependencies the check needs (database, migrations, containers, seed data). The check consumes them; it never provisions its own.
- Fail loud, fail fast. A non-zero exit from the check step fails the job. Never swallow errors. Never continue-on-error a gating step.
- Deterministic installs. Use frozen/locked installs so CI matches the committed lockfile.
- Least privilege. Grant read-only workflow permissions unless a step needs more.
- Bound the run. Set a timeout so a hung service or test cannot pin a runner.

## Template files (read before scaffolding)

- templates/required-check.yml.txt — GitHub Actions workflow that optionally provisions dependencies, runs a check command, and exposes it as a required-by-name job. Placeholders mark the repo-specific steps.

## Setup workflow

1. Add the workflow file under the repo's .github/workflows directory, dropping the `.txt` suffix.
2. Fill the placeholders: dependency install, any dependency provisioning (database, migrations, service boot), and the check command.
3. Confirm the check trigger targets pull requests to the protected branch.
4. Push the workflow and open a pull request so the job registers a status check by name.
5. In repository settings, enable branch protection on the target branch and mark that job's name as a required status check. Optionally require branches to be up to date before merging.
6. Verify: a failing check blocks the merge button; a passing check unblocks it.

## Provisioning conventions

- Bring up dependencies (e.g. a database service container) and run migrations as pipeline steps before the check. Mirror how the repo boots the service locally.
- If the check needs a running service, boot it in the pipeline (or let the check's runner boot it) and point the check at that instance.
- Keep provisioning in the pipeline, never in the check itself.
- Pass secrets and connection details via CI environment/secrets, never committed files.

## Branch protection conventions

- Require the check's job name as a status check on the protected branch.
- Keep job names stable. Changing a name silently removes the gate until protection is updated.
- Require each gating check by name when multiple checks gate merges (tests, typecheck, lint).
- Add pull-request review and an up-to-date/linear branch requirement per the repo's policy.

## Expanding

- New required check: add a job (or a new workflow), then add its job name to branch protection.
- New dependency: add a provisioning step before the check; do not push it into the check itself.
- New protected branch: replicate the branch-protection required-check settings for that branch.
