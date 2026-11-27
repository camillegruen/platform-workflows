# PR Validation Reusable Workflow

## Overview

The `pr-validation.yml` reusable workflow checks that pull request titles follow [Conventional Commits](https://www.conventionalcommits.org/) and can auto-assign reviewers.

> Replaces both `pr-validation.yml` and `_pr-lint.yml`. They were identical apart from the `auto-assign` default, which is now `false`. Callers of `_pr-lint.yml` should switch to this workflow and pass `auto-assign: true`.

## Features

- Validates PR titles with `amannn/action-semantic-pull-request` (e.g. `feat: add drift check`)
- Optionally auto-assigns reviewers and assignees with `kentaro-m/auto-assign-action`
- Each check can be turned on or off with an input

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `validate-title` | boolean | No | `true` | Validate PR title with semantic-pull-request |
| `auto-assign` | boolean | No | `false` | Auto-assign reviewers (needs .github/auto_assign.yml in the calling repo) |

## Permissions

The reusable workflow requests the following permissions. The **calling job must grant at least these**, because a reusable workflow can't have more permissions than its caller:

```yaml
permissions:
  contents: read
  pull-requests: write
```

## Auto-assign Config

Auto-assign reads `.github/auto_assign.yml` in the **calling** repo. A starter config is in [examples/auto_assign.yml](examples/auto_assign.yml).

## Recommended Trigger

Include `edited` so the title check re-runs when someone fixes the title:

```yaml
on:
  pull_request:
    types: [opened, edited, synchronize, reopened, ready_for_review]
```

## Usage

See [examples/pr-validation-example.yml](examples/pr-validation-example.yml) for a complete caller workflow.
