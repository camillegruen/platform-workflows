# Terraform Drift Detection Reusable Workflow

## Overview

The `tf-drift.yml` reusable workflow runs `terraform plan -detailed-exitcode` to check whether real infrastructure still matches the code. Call it on a schedule. When it finds drift it opens (or updates) a GitHub issue, and it closes the issue once the drift is resolved.

## Features

- Detects drift with `-detailed-exitcode` (exit code 2 means changes)
- Runs without taking a state lock (`-lock=false`), so it never blocks a real apply
- Opens one issue per environment and working directory, labeled `drift` by default
- Updates the open issue with the latest plan on each run
- Closes the issue automatically when no drift remains
- Exposes a `drift` output for follow-up jobs (e.g. a Slack alert)

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `environment` | string | Yes | - | GitHub Environment used for scoped secrets/vars |
| `working-directory` | string | No | `"."` | Directory where commands are executed |
| `role-session-name` | string | No | `{repo-name}-session` | AWS role session name |
| `terraform-version` | string | No | `"1.9.8"` | Terraform CLI version |
| `init-args` | string | No | `""` | Additional arguments passed to `terraform init` |
| `plan-args` | string | No | `""` | Additional arguments passed to `terraform plan` |
| `issue-label` | string | No | `"drift"` | Label applied to drift issues |

## Outputs

| Output | Description |
|--------|-------------|
| `drift` | 'true' if drift was found. |

## Permissions

The reusable workflow requests the following permissions. The **calling job must grant at least these**, because a reusable workflow can't have more permissions than its caller:

```yaml
permissions:
  id-token: write
  contents: read
  issues: write
```

## Required Setup

Create a GitHub Environment in the calling repo (**Settings → Environments**) with the name you pass as `environment`, and add these **variables**:

| Variable | Description |
|----------|-------------|
| `DEPLOYMENT_ROLE_ARN` | IAM role assumed through GitHub OIDC |
| `AWS_REGION` | AWS region |

The calling job must use `secrets: inherit`. All of the caller's secrets and the environment's variables are exported as environment variables for later steps (via the `aws-env` action), so Terraform, CDK and other tools can read them directly.

## Issue Format

Title: `Terraform drift: <environment> (<working-directory>)`

The body links to the run and includes the plan in a collapsed block. When a later run finds no drift, the workflow comments `No drift as of [this run]. Closing.` and closes the issue.

## Usage

See [examples/tf-drift-example.yml](examples/tf-drift-example.yml) for a complete caller workflow.
