# CHANGELOG

## UNDER DEVELOPMENT marginal-action vx.x.x, xxxx-xx-xx

### Breaking Changes

### New Features

### Improvements

### Fixes

### Warnings

## marginal-action v1.1.2, 2026-09-12

### Breaking Changes

### New Features

### Improvements

### Fixes

- The "Post review comment" step no longer fails a forked pull request's
  run just because it never received `anthropic-api-key`/`openai-api-key`
  — GitHub doesn't forward repository secrets to `pull_request` runs
  triggered from a fork, so a `models.reviewer`-configured repo previously
  failed CI on every external contributor's PR. Now detects `marginal
  review`'s dedicated exit code for a missing-credentials failure
  (see `exactml/marginal` ISSUE-60) and, only when the run is on a forked
  PR, posts a `::notice::` and exits 0 instead. A same-repo PR missing its
  key still fails loudly, same as before — this only recognizes the case
  that isn't actually a misconfiguration

  ([exactml/marginal ISSUE-60](https://github.com/exactml/marginal/issues/60),
  [PR-4](https://github.com/exactml/marginal-action/pull/4) by [@exactml](https://github.com/exactml))

### Warnings

## marginal-action v1.1.1, 2026-09-09

### Breaking Changes

### New Features

### Improvements

### Fixes

- Installs `marginal-review[anthropic,openai]` instead of bare
  `marginal-review`. Without the extras, `models.reviewer` could never
  work regardless of API key — the provider SDK itself was never installed,
  failing with "requires its SDK to be installed" instead of running or
  even reaching a credentials check

  ([PR-3](https://github.com/exactml/marginal-action/pull/3) by [@exactml](https://github.com/exactml))

### Warnings

## marginal-action v1.1.0, 2026-09-08

### Breaking Changes

### New Features

- `anthropic-api-key` / `openai-api-key` inputs: forwarded as
  `ANTHROPIC_API_KEY`/`OPENAI_API_KEY` to the `marginal review` step, so
  `marginal`'s `models.reviewer` (see `exactml/marginal` ISSUE-21) actually
  has credentials to run with. Both optional — omit them if
  `.marginal/config.yaml` doesn't configure `models.reviewer`

  ([PR-2](https://github.com/exactml/marginal-action/pull/2) by [@exactml](https://github.com/exactml))

### Improvements

### Fixes

### Warnings

## marginal-action v1.0.0, 2026-09-08

### Breaking Changes

### New Features

- Composite action wiring [`marginal review --comment`](https://github.com/exactml/marginal):
  checks out the consuming repo (so its `.marginal/config.yaml` permissions
  are honored), installs `marginal-review` from PyPI, and posts the PR
  summary as a comment. `repo` and `pr-number` default to the triggering
  workflow's own repository/PR, so most callers only need to pass
  `github-token`

### Improvements

- Self-test CI workflow: runs the action against its own PRs via `uses: ./`,
  verified end-to-end on PR #1

### Fixes

### Warnings
