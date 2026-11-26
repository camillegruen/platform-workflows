# Terraform Apply Reusable Workflow

## Overview

The `tf-apply.yml` reusable workflow runs `terraform plan` and applies that exact plan file in a single job. Because the job runs in a GitHub Environment, adding **Required reviewers** to the environment pauses the run until someone approves it.

## Features

- Plans and applies the saved plan file, so what was planned is what gets applied
- Manual approval gate through GitHub Environment protection rules
- Only one apply per environment and working directory at a time; queued applies wait, and running ones are never cancelled
- Waits up to 5 minutes for the state lock

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
```

## Required Setup

Create a GitHub Environment in the calling repo (**Settings → Environments**) with the name you pass as `environment`, and add these **variables**:

| Variable | Description |
|----------|-------------|
| `DEPLOYMENT_ROLE_ARN` | IAM role assumed through GitHub OIDC |
| `AWS_REGION` | AWS region |

The calling job must use `secrets: inherit`. All of the caller's secrets and the environment's variables are exported as environment variables for later steps (via the `aws-env` action), so Terraform, CDK and other tools can read them directly.

## Approval Gate

To require approval before apply runs:

1. Go to **Settings → Environments → `<environment>`**.
2. Enable **Required reviewers** and add people or teams.
3. Optionally, under **Deployment branches**, restrict deployments to `main`.

The run then waits in the Actions UI until a reviewer approves it.

## Usage

See [examples/tf-apply-example.yml](examples/tf-apply-example.yml) for a complete caller workflow.

For a full validate → plan → apply pipeline across environments, see [examples/terraform-pipeline-example.yml](examples/terraform-pipeline-example.yml).
