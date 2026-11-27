# Build Python Reusable Workflow

## Overview

The `build-python.yml` reusable workflow installs a Python project with pip, runs Ruff, and runs the tests across a matrix of Python versions.

## Features

- Installs dependencies with a configurable command (default `pip install -r requirements.txt`)
- Runs `ruff check` and `ruff format --check`
- Runs a configurable test command (default `pytest -q`)
- Runs one job per Python version (`fail-fast: false`)
- Caches pip downloads, keyed on `requirements*.txt` and `pyproject.toml`

## Inputs

| Input | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `python-versions` | string | No | `'["3.12"]'` | JSON array of Python versions. |
| `working-directory` | string | No | `"."` | Directory where commands are executed |
| `install-command` | string | No | `"pip install -r requirements.txt"` | Command that installs the project and its test dependencies |
| `run-lint` | boolean | No | `true` | Run the lint step |
| `test-command` | string | No | `"pytest -q"` | Test command, run through the project environment |

## Permissions

The reusable workflow requests the following permissions. The **calling job must grant at least these**, because a reusable workflow can't have more permissions than its caller:

```yaml
permissions:
  contents: read
```

## Install Command Examples

| Project layout | `install-command` |
|----------------|-------------------|
| `requirements.txt` | `pip install -r requirements.txt` (default) |
| `pyproject.toml` with a `dev` extra | `pip install -e .[dev]` |
| Separate dev requirements | `pip install -r requirements.txt -r requirements-dev.txt` |

`ruff` and `pytest` are always installed, so they don't need to be in your requirements.

## Usage

See [examples/build-python-example.yml](examples/build-python-example.yml) for a complete caller workflow.
