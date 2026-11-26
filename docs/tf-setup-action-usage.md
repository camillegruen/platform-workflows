# Terraform Setup Composite Action

## Overview

The `tf-setup` composite action installs Terraform, optionally runs `aws-env` to export secrets and vars and assume the AWS role, then runs `terraform init`. The `tf-*` workflows use it, and you can also use it in your own jobs.

## Features

- Installs the requested Terraform version (wrapper disabled, so exit codes pass through)
- Optional AWS authentication (`cloud-auth: "false"` for offline checks)
- Three init modes: `backend`, `no-backend` or `none`
- Extra `terraform init` arguments, e.g. `-backend-config=backend.hcl`

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `working-directory` | No | `"."` | Terraform root module. |
| `terraform-version` | No | `"1.9.8"` | Terraform version. |
| `cloud-auth` | No | `"true"` | Export secrets/vars and assume the AWS role. Set "false" for offline checks. |
| `secrets-json` | No | `"{}"` | Pass `toJson(secrets)`. |
| `vars-json` | No | `"{}"` | Pass `toJson(vars)`. |
| `role-arn` | No | `""` | IAM role to assume (usually vars.DEPLOYMENT_ROLE_ARN). |
| `region` | No | `""` | AWS region (usually vars.AWS_REGION). |
| `role-session-name` | No | `"github-actions"` | STS session name. |
| `init` | No | `"backend"` | Init mode: "backend" (normal), "no-backend" (validate only) or "none". |
| `init-args` | No | `""` | Extra arguments for terraform init (e.g. -backend-config=backend.hcl). |

## Usage

```yaml
steps:
  - uses: actions/checkout@<sha>  # v6.0.2
  - uses: YOUR_ORG/shared-workflows/actions/tf-setup@v1
    with:
      working-directory: infra
      secrets-json: ${{ toJson(secrets) }}
      vars-json: ${{ toJson(vars) }}
      role-arn: ${{ vars.DEPLOYMENT_ROLE_ARN }}
      region: ${{ vars.AWS_REGION }}
      init-args: -backend-config=backend-dev.hcl
  - run: terraform output
    working-directory: infra
```

The job needs `permissions: id-token: write` when `cloud-auth` is on.
