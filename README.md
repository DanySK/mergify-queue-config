# mergify-queue-config

This repository stores the Mergify configuration used to automate pull request updates and merge queue behavior.
Repositories adopt it with:

```yaml
extends: mergify-queue-config
```

Mergify follows `extends` for **one level only**:
this configuration is self-contained and does not extend any other shared configuration.
Adopting repositories can override any rule locally by redefining it with the same `name`.

## Files

- `.mergify.yml`: Mergify rules for pull request maintenance, merge protections, priorities, and queue configuration.

## Two-step CI

Pull requests go through two CI levels:

1. **quick CI**, on every pull request push: cheap checks (e.g., Linux only, JVM only).
   Once green, eligible pull requests are queued automatically.
2. **full CI**, in the merge queue: Mergify groups up to 5 queued pull requests in a draft pull request
   (head branch `mergify/merge-queue/*`) and runs the complete CI on it, including the release dry-run.
   The pull requests are merged only if it passes.

If the full CI fails in the queue, Mergify finds the culprit by splitting the batch,
dequeues it, and labels it `full-ci`.
From then on, that pull request runs the full CI on every push,
and it needs the full CI to be green to re-enter the queue.

## CI contract for adopting repositories

The configuration relies on check names, so the CI of each adopting repository must:

- report a check named **`quick-check`** that runs the quick CI on `pull_request` events;
- report a check named **`ci-cd / success`** that aggregates the full CI
  (e.g., a `success` job in a reusable workflow invoked by a job named `ci-cd`);
- run the **full CI instead of the quick one** when:
  - the head branch starts with `mergify/merge-queue/` (batch draft pull requests);
  - the pull request is labeled `full-ci`
    (also on the `labeled` event, so that the full CI starts as soon as the label is added);
- run the full CI (and the actual release) on pushes to the default branch.

Pull requests from forks follow the same contract: the full CI runs without secrets, as GitHub provides none to forks.

## Current behavior

The configuration currently provides:

- automatic branch updates with merge when a pull request has the `auto-update-merge` label;
- automatic rebases when a pull request has the `auto-update-rebase` label;
- an automatic comment asking the author to resolve conflicts when Mergify detects them;
- a merge protection (`CI success`) requiring, on non-draft pull requests,
  `quick-check` to succeed (or `ci-cd / success`, for pull requests labeled `full-ci`);
- auto-queueing (`auto_merge_conditions: true`) of the non-draft pull requests that:
  - are labeled `merge-queue` or `ancillary`, or are opened by `renovate[bot]` or `dependabot[bot]`;
  - have a green `quick-check` (a green `ci-cd / success`, if labeled `full-ci`);
- a merge queue named `squash-queue` with:
  - `batch_size: 5`;
  - `merge_method: squash`;
  - merge only when `ci-cd / success` passes on the batch;
  - `max_parallel_checks: 3`;
  - reset on every external merge;
  - status comments enabled with outcome reporting;
- queue priorities (first match wins):
  - `low` for bots (authors ending in `[bot]`) and pull requests labeled `ancillary`;
  - `high` for maintainers (`DanySK`);
  - `medium` (Mergify's default) for everyone else;
- the `full-ci` label added to pull requests dequeued because the full CI failed
  (`CHECKS_FAILED`, `CHECKS_TIMEOUT`, or `DROPPED_BY_BISECTION_ELIMINATION`).

## Labels in use

- `auto-update-merge`
- `auto-update-rebase`
- `merge-queue`: queue a pull request from a human author once its CI is green
- `ancillary`: automated updates of ancillary files (e.g., lock files); queued automatically, low priority
- `full-ci`: run the full CI on the pull request (added by Mergify after a failure in the queue; can be added manually)
- `queued` / `dequeued`: managed by Mergify's merge queue

## Purpose

Keep this repository as the single source of truth for Mergify queue policy so it can be reviewed, versioned, and reused consistently.
