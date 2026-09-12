# marginal-action

[![self-test](https://github.com/exactml/marginal-action/actions/workflows/self-test.yml/badge.svg)](https://github.com/exactml/marginal-action/actions/workflows/self-test.yml)

A GitHub Action that runs [`marginal review --comment`](https://github.com/exactml/marginal)
on a pull request and posts the result as a PR comment — no local install,
no CI script of your own to maintain.

This is a thin wrapper: it installs [`marginal-review`](https://pypi.org/project/marginal-review/)
from PyPI and runs the CLI. There's no analysis here beyond what `marginal
review` itself does — see the [marginal](https://github.com/exactml/marginal)
repo for the actual review logic and its roadmap.

## Usage

```yaml
# .github/workflows/marginal.yml
name: marginal

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: exactml/marginal-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

`repo` and `pr-number` default to the repository and pull request that
triggered the workflow, so most consumers only need to pass `github-token`
(plus a provider key, if `.marginal/config.yaml` configures `models.reviewer`
— see below).

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `github-token` | yes | — | Token used to read the PR and post the comment. `secrets.GITHUB_TOKEN` works as long as the job has `permissions: pull-requests: write`. |
| `repo` | no | current repository | GitHub repository as `owner/name`. |
| `pr-number` | no | current PR number | Pull request number. |
| `marginal-version` | no | latest | Pin a specific `marginal-review` version instead of always installing latest. |
| `python-version` | no | `3.11` | Python version used to run `marginal`. |
| `anthropic-api-key` | no | — | `ANTHROPIC_API_KEY`. Required only if `.marginal/config.yaml` configures `models.reviewer` with `provider: anthropic`. |
| `openai-api-key` | no | — | `OPENAI_API_KEY`. Required only if `.marginal/config.yaml` configures `models.reviewer` with `provider: openai`. |

If `models.reviewer` is configured but its matching key isn't supplied here,
`marginal review` exits non-zero with a clean `MissingCredentialsError`
message and posts nothing — it does not fall back to a metadata-only
summary. Leave `models.reviewer` unset entirely in `.marginal/config.yaml`
to skip finding generation and keep the metadata-only summary instead.

The one exception is a pull request from a fork: GitHub never forwards
repository secrets to a fork-triggered `pull_request` run, so a missing key
there doesn't mean the workflow is misconfigured — it's expected. This
action detects that case and skips the review with a `::notice::` instead
of failing the run; a same-repo PR missing its key still fails loudly as
above.

## Permissions

Posting a PR comment needs `permissions: pull-requests: write` on the job
(or equivalent repository/organization default). Without it, GitHub itself
will reject the API call before `marginal` ever gets a chance to run.

## Related

- [exactml/marginal](https://github.com/exactml/marginal) — the CLI and
  library this action wraps
- [ADR 0002](https://github.com/exactml/marginal/blob/master/docs/adr/0002-marginal-action-composite.md) —
  why this is a composite action rather than a Docker action
