# AWS Environment Composite Action

## Overview

The `aws-env` composite action exports the caller's secrets and variables as environment variables for all later steps, then assumes an IAM role through GitHub OIDC. It replaces the "Convert secrets and vars to environment variables" step that used to be copied into each workflow.

## Features

- Exports every secret and variable as `NAME=value` in `GITHUB_ENV`
- Handles multi-line values with a random delimiter, so a value containing `EOF` cannot break the step
- Reads values from JSON environment variables instead of templating them into code
- Skips `github_token` (already available as `github.token`)
- Assumes the role with `aws-actions/configure-aws-credentials`

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `secrets-json` | No | `"{}"` | Pass `toJson(secrets)`. Leave empty to export nothing from secrets. |
| `vars-json` | No | `"{}"` | Pass `toJson(vars)`. |
| `role-arn` | Yes | - | IAM role to assume. |
| `region` | Yes | - | AWS region. |
| `role-session-name` | No | `"github-actions"` | STS session name. |

## Usage

```yaml
steps:
  - uses: YOUR_ORG/shared-workflows/actions/aws-env@v1
    with:
      secrets-json: ${{ toJson(secrets) }}
      vars-json: ${{ toJson(vars) }}
      role-arn: ${{ vars.DEPLOYMENT_ROLE_ARN }}
      region: ${{ vars.AWS_REGION }}
```

> **Security note:** every exported secret is readable by **all later steps in the job**, including third-party actions. Run trusted actions after this step, or pass `secrets-json: "{}"` if the job doesn't need secrets in its environment.
