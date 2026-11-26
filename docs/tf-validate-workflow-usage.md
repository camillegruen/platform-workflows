# Terraform Validate Reusable Workflow

## Overview

The `tf-validate.yml` reusable workflow runs static checks on Terraform code: `terraform fmt`, `terraform init` without a backend, `terraform validate`, and TFLint. It needs no cloud credentials, so it is safe to run on every pull request, including PRs from forks.

## Features

- Fails if any file is not `terraform fmt` formatted (shows the diff)
- Initializes providers without touching remote state (`-backend=false`)
- Runs `terraform validate`
- Runs TFLint recursively, with an optional shared config file
- No AWS credentials or GitHub Environment required

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `working-directory` | string | No | `"."` | Directory where commands are executed |
| `terraform-version` | string | No | `"1.9.8"` | Terraform CLI version |
| `tflint-version` | string | No | `"v0.66.0"` | TFLint version |
| `tflint-config` | string | No | `""` | Path to .tflint.hcl relative to the repo root (empty = look in working-directory). |

## Permissions

The reusable workflow requests the following permissions. The **calling job must grant at least these**, because a reusable workflow can't have more permissions than its caller:

```yaml
permissions:
  contents: read
```

## Usage

See [examples/tf-validate-example.yml](examples/tf-validate-example.yml) for a complete caller workflow.

For a full validate → plan → apply pipeline across environments, see [examples/terraform-pipeline-example.yml](examples/terraform-pipeline-example.yml).
