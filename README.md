# git-carbon-action

A reusable GitHub Action that installs [git-carbon](https://github.com/mullvad/git-carbon),
syncs all files tracked by `.gitcarbon`, and opens a pull request with the diff — fully automated.

## Permissions

The calling workflow must grant:

```yaml
permissions:
  contents: write
  pull-requests: write
```

## Usage

```yaml
name: Update git-carbon tracked files

on:
  schedule:
    - cron: '0 6 * * 1'  # Every Monday at 06:00 UTC
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: mullvad/git-carbon-action@v1
        with:
          token: ${{ github.token }}
```

> **Note:** `actions/checkout` must run before this action so that the workspace
> contains the repository files (including `.gitcarbon`).

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `token` | yes | `${{ github.token }}` | GitHub token used for downloading releases and creating pull requests. |
| `version` | no | `latest` | git-carbon version to install (e.g. `v1.2.3`) or `latest`. |
| `pr-branch` | no | `git-carbon/update` | Branch name for the update PR. |
| `pr-title` | no | `chore: update git-carbon tracked files` | Title of the pull request. |
| `pr-body` | no | `Automated update of files tracked by git-carbon.` | Body of the pull request. |
| `pr-commit-message` | no | `chore: update git-carbon tracked files` | Commit message for the update commit. |
| `pr-labels` | no | `''` | Labels to apply to the pull request (comma or newline separated). |
| `pr-assignees` | no | `''` | Assignees for the pull request (comma or newline separated). |
| `pr-reviewers` | no | `''` | Reviewers for the pull request (comma or newline separated). |
| `pr-draft` | no | `false` | Create the pull request as a draft. |

## Outputs

These are pass-through outputs from [`peter-evans/create-pull-request@v7`](https://github.com/peter-evans/create-pull-request).

| Output | Description |
|---|---|
| `pull-request-number` | The pull request number. |
| `pull-request-url` | The URL of the pull request. |
| `pull-request-operation` | The pull request operation (`created`, `updated`, `closed`, `none`). |
| `pull-request-head-sha` | The HEAD SHA of the pull request branch. |
| `pull-request-branch` | The pull request branch name. |

## Pinning a version

```yaml
- uses: mullvad/git-carbon-action@v1
  with:
    token: ${{ github.token }}
    version: v1.2.3
```

## Accessing outputs

```yaml
- uses: mullvad/git-carbon-action@v1
  id: gc
  with:
    token: ${{ github.token }}

- name: Print PR URL
  if: steps.gc.outputs.pull-request-url != ''
  run: echo "PR opened at ${{ steps.gc.outputs.pull-request-url }}"
```

## Edge cases

| Scenario | Behaviour |
|---|---|
| No `.gitcarbon` file in the repo | `git carbon update --all` is a no-op; `create-pull-request` sees a clean tree and creates no PR. |
| No file changes after update | `create-pull-request` detects a clean tree and emits empty outputs; no PR is created. |
| PR branch already exists | `create-pull-request` updates the existing PR. |
| Unsupported version string | `go install` fails with an error from the Go toolchain. |
