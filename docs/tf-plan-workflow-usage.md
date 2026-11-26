# Terraform Plan Reusable Workflow

## Overview

The `tf-plan.yml` reusable workflow runs `terraform init` and `terraform plan`, then posts the rendered plan output as a pull request comment. This gives reviewers a clear view of infrastructure changes before apply.

> Previously named `terraform-plan.yml`. The inputs are unchanged, plus a new optional `init-args`; update the `uses:` path in callers.

## Features

- Runs Terraform plan in a configurable working directory
- Supports optional extra `terraform init` and `terraform plan` arguments
- Posts plan output to pull requests, including the error output when the plan fails
- Keeps one comment per environment and working directory, updated on each push instead of duplicated
- Truncates large output to stay within GitHub comment limits
- Cancels superseded plan runs for the same branch

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `environment` | string | Yes | - | GitHub Environment used for scoped secrets/vars |
| `working-directory` | string | No | `"."` | Directory where commands are executed |
| `role-session-name` | string | No | `{repo-name}-session` | AWS role session name |
| `terraform-version` | string | No | `"1.9.8"` | Terraform CLI version |
| `init-args` | string | No | `""` | Additional arguments passed to `terraform init` |
| `plan-args` | string | No | `""` | Additional arguments passed to `terraform plan` |

## Permissions

The reusable workflow requests the following permissions. The **calling job must grant at least these**, because a reusable workflow can't have more permissions than its caller:

```yaml
permissions:
  id-token: write
  contents: read
  pull-requests: write
```

## Required Setup

Create a GitHub Environment in the calling repo (**Settings → Environments**) with the name you pass as `environment`, and add these **variables**:

| Variable | Description |
|----------|-------------|
| `DEPLOYMENT_ROLE_ARN` | IAM role assumed through GitHub OIDC |
| `AWS_REGION` | AWS region |

The calling job must use `secrets: inherit`. All of the caller's secrets and the environment's variables are exported as environment variables for later steps (via the `aws-env` action), so Terraform, CDK and other tools can read them directly.

## PR Comment Format

The workflow posts one comment per environment and working directory and updates it on every push:

````text
#### Terraform Plan: `success` — `dev:infra`

<details open><summary>Show Terraform Plan</summary>

```hcl
[terraform plan output]
```

</details>

*Pusher: @octocat, Action: `pull_request`, Workflow: `Terraform plan`, [Run](...)*
````

If the terraform plan step fails, the comment shows `failure` with the error output (collapsed), and the job then fails.

Output longer than 60,000 characters is truncated to stay under GitHub's 65,536-character comment limit. The full output is always in the run log.

## Usage

See [examples/tf-plan-example.yml](examples/tf-plan-example.yml) for a complete caller workflow.

For a full validate → plan → apply pipeline across environments, see [examples/terraform-pipeline-example.yml](examples/terraform-pipeline-example.yml).
