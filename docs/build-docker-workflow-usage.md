# Build Docker Reusable Workflow

## Overview

The `build-docker.yml` reusable workflow builds a multi-architecture Docker image with Buildx and, when asked, pushes it to GitHub Container Registry (GHCR).

## Features

- Builds multi-arch images (default `linux/amd64,linux/arm64`)
- Pushes to `ghcr.io/<owner>/<repo>` by default, only when `push: true`
- Tags images automatically: branch, PR number, semver (`1.2.3`, `1.2`) and short SHA
- Caches layers with the GitHub Actions cache
- Outputs the image digest

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `context` | string | No | `"."` | Docker build context |
| `dockerfile` | string | No | `"Dockerfile"` | Dockerfile path, relative to `context` |
| `image-name` | string | No | `""` | Defaults to ghcr.io/<owner>/<repo>. |
| `platforms` | string | No | `"linux/amd64,linux/arm64"` | Target platforms |
| `push` | boolean | No | `false` | Push the image to the registry |

## Outputs

| Output | Description |
|--------|-------------|
| `digest` | Image digest. |

## Permissions

The reusable workflow requests the following permissions. The **calling job must grant at least these**, because a reusable workflow can't have more permissions than its caller:

```yaml
permissions:
  contents: read
  packages: write
```

## Image Tags

| Trigger | Example tags |
|---------|--------------|
| Push to `main` | `main`, `sha-abc1234` |
| Pull request #42 | `pr-42`, `sha-abc1234` (not pushed unless `push: true`) |
| Tag `v1.2.3` | `1.2.3`, `1.2`, `sha-abc1234` |

## Usage

See [examples/build-docker-example.yml](examples/build-docker-example.yml) for a complete caller workflow.
