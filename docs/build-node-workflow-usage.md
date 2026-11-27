# Build Node Reusable Workflow

## Overview

The `build-node.yml` reusable workflow installs dependencies, then lints, tests and builds a Node.js project across a matrix of Node versions.

## Features

- Supports npm, pnpm and yarn (pnpm and yarn through Corepack)
- Runs one job per Node version (`fail-fast: false`)
- Uses frozen lockfile installs
- Runs `lint` and `build` only if those scripts exist (`--if-present`)
- Caches npm packages (pnpm and yarn are not cached)

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `node-versions` | string | No | `'["20", "22"]'` | JSON array of Node versions. |
| `working-directory` | string | No | `"."` | Directory where commands are executed |
| `package-manager` | string | No | `"npm"` | npm, pnpm or yarn. |
| `run-lint` | boolean | No | `true` | Run the lint step |
| `run-build` | boolean | No | `true` | Run the build step |

## Permissions

The reusable workflow requests the following permissions. The **calling job must grant at least these**, because a reusable workflow can't have more permissions than its caller:

```yaml
permissions:
  contents: read
```

## Usage

See [examples/build-node-example.yml](examples/build-node-example.yml) for a complete caller workflow.
