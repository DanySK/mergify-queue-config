# mergify-queue-config

This repository stores the Mergify configuration used to automate pull request updates and merge queue behavior.

## Files

- `.mergify.yml`: Mergify rules for pull request maintenance, merge protections, and queue configuration.

## Current behavior

The configuration currently provides:

- automatic branch updates with merge when a pull request has the `auto-update-merge` label;
- automatic rebases when a pull request has the `auto-update-rebase` label;
- an automatic comment asking the author to resolve conflicts when Mergify detects them;
- merge protection that only allows auto-merge for non-draft pull requests with successful checks when they are either:
  - labeled `merge-queue`;
  - opened by `renovate[bot]`;
  - opened by `dependabot[bot]`;
- a merge queue named `squash-queue` with:
  - `batch_size: 5`;
  - `merge_method: squash`;
  - `max_parallel_checks: 2`;
  - reset on every external merge;
  - status comments enabled with outcome reporting.

## Labels in use

- `auto-update-merge`
- `auto-update-rebase`
- `merge-queue`

## Purpose

Keep this repository as the single source of truth for Mergify queue policy so it can be reviewed, versioned, and reused consistently.
