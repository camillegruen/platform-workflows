# PR Comment Composite Action

## Overview

The `pr-comment` composite action posts a file, such as a Terraform plan or a CDK diff, as a collapsible pull request comment. It updates its own earlier comment instead of adding a new one on each push.

## Features

- One comment per `title` + `key`, so parallel environments don't overwrite each other
- Finds its earlier comment through a hidden HTML marker, checking every comment on the PR (paginated)
- Expanded on success, collapsed on failure
- Truncates at 60,000 characters
- Links back to the workflow run

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `title` | Yes | - | Heading, e.g. "Terraform Plan" or "CDK Diff". |
| `file` | Yes | - | Path to the output file (relative to the workspace). |
| `outcome` | No | `"success"` | Outcome of the step that produced the file (success / failure). |
| `language` | No | `"text"` | Code-fence language (hcl, diff, text...). |
| `key` | No | `""` | Extra identity for the comment, so parallel runs (e.g. one per environment) each keep their own comment. Defaults to environment + working directory.  |

## Usage

```yaml
steps:
  - id: tool
    continue-on-error: true
    run: my-tool 2>&1 | tee output.txt
  - if: always() && github.event_name == 'pull_request'
    uses: YOUR_ORG/shared-workflows/actions/pr-comment@v1
    with:
      title: My Tool
      file: output.txt
      outcome: ${{ steps.tool.outcome }}
      language: text
      key: dev
```

The job needs `permissions: pull-requests: write`.
