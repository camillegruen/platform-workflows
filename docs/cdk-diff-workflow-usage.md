# CDK Diff Reusable Workflow

## Overview

The `cdk-diff.yml` reusable workflow installs the CDK app's Python dependencies with pip and the AWS CDK CLI, runs `cdk diff`, and posts the output as a pull request comment, so reviewers can see stack changes before deploy.

## Features

- Installs Python dependencies with a configurable command (default `pip install -r requirements.txt`), with pip caching
- Installs a configurable AWS CDK CLI version
- Runs the shared `version` setup action before the diff
- Captures the diff from both stdout and stderr (`cdk diff` writes to stderr)
- Posts the diff to pull requests, including the error output when it fails
- Keeps one comment per environment and working directory

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `environment` | string | Yes | - | GitHub Environment used for scoped secrets/vars |
| `role-session-name` | string | No | `{repo-name}-session` | AWS role session name |
| `working-directory` | string | No | `"."` | Directory where commands are executed |
| `python-version` | string | No | `"3.12"` | Python version |
| `install-command` | string | No | `"pip install -r requirements.txt"` | Command that installs the CDK app's Python dependencies |
| `cdk-version` | string | No | `"latest"` | AWS CDK CLI version (npm version or tag) |

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
#### CDK Diff: `success` — `dev:infra`

<details open><summary>Show CDK Diff</summary>

```diff
[cdk diff output]
```

</details>

*Pusher: @octocat, Action: `pull_request`, Workflow: `CDK`, [Run](...)*
````

If the cdk diff step fails, the comment shows `failure` with the error output (collapsed), and the job then fails.

Output longer than 60,000 characters is truncated to stay under GitHub's 65,536-character comment limit. The full output is always in the run log.

## Usage

See [examples/cdk-diff-example.yml](examples/cdk-diff-example.yml) for a complete caller workflow.
