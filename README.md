# platform-workflows

A collection of GitHub Actions workflows and composite actions built up over time across projects and gathered here in one place.

**Pull from here when adding CI/CD to a repo.** Every workflow is reusable (`workflow_call`), so a repo needs only a short caller file, not copied YAML. Each workflow has a usage page in `docs/` and a ready-to-copy caller in `docs/examples/`. Copy the example you need, change the paths and environments, and you're done. Fixes made here reach every repo that uses the workflows.

When you write something another repo could use, add it here: as a reusable workflow, or as a composite action for a sequence of steps you keep repeating.

---

## Repo layout

```
platform-workflows/
├── README.md
├── actions
│   ├── aws-env
│   │   └── action.yml
│   ├── pr-comment
│   │   └── action.yml
│   └── tf-setup
│       └── action.yml
└── docs
    ├── aws-env-action-usage.md
    ├── build-docker-workflow-usage.md
    ├── build-node-workflow-usage.md
    ├── build-python-workflow-usage.md
    ├── cdk-diff-workflow-usage.md
    ├── examples
    │   ├── auto_assign.yml
    │   ├── build-docker-example.yml
    │   ├── build-node-example.yml
    │   ├── build-python-example.yml
    │   ├── cdk-diff-example.yml
    │   ├── pr-validation-example.yml
    │   ├── terraform-pipeline-example.yml
    │   ├── tf-apply-example.yml
    │   ├── tf-drift-example.yml
    │   ├── tf-plan-example.yml
    │   └── tf-validate-example.yml
    ├── pr-comment-action-usage.md
    ├── pr-validation-workflow-usage.md
    ├── tf-apply-workflow-usage.md
    ├── tf-drift-workflow-usage.md
    ├── tf-plan-workflow-usage.md
    ├── tf-setup-action-usage.md
    └── tf-validate-workflow-usage.md
```

---

## What's here

### Reusable workflows

| Workflow | What it does | Caller must grant | Docs | Example |
|---|---|---|---|---|
| `tf-validate.yml` | `fmt`, `init` (no backend), `validate`, `tflint`. No credentials needed. | `contents: read` | [usage](docs/tf-validate-workflow-usage.md) | [example](docs/examples/tf-validate-example.yml) |
| `tf-plan.yml` | `plan`, posted as a PR comment (failures included) | `id-token: write`, `pull-requests: write` | [usage](docs/tf-plan-workflow-usage.md) | [example](docs/examples/tf-plan-example.yml) |
| `tf-apply.yml` | `plan` + `apply` behind a GitHub environment (approval gate) | `id-token: write` | [usage](docs/tf-apply-workflow-usage.md) | [example](docs/examples/tf-apply-example.yml) |
| `tf-drift.yml` | Scheduled drift check; opens and closes a `drift` issue | `id-token: write`, `issues: write` | [usage](docs/tf-drift-workflow-usage.md) | [example](docs/examples/tf-drift-example.yml) |
| `cdk-diff.yml` | `cdk diff`, posted as a PR comment | `id-token: write`, `pull-requests: write` | [usage](docs/cdk-diff-workflow-usage.md) | [example](docs/examples/cdk-diff-example.yml) |
| `pr-validation.yml` | Semantic PR title check + reviewer auto-assign | `pull-requests: write` | [usage](docs/pr-validation-workflow-usage.md) | [example](docs/examples/pr-validation-example.yml) |
| `build-node.yml` | npm / pnpm / yarn: lint, test, build across a Node matrix | `contents: read` | [usage](docs/build-node-workflow-usage.md) | [example](docs/examples/build-node-example.yml) |
| `build-python.yml` | pip install, ruff, pytest across a Python matrix | `contents: read` | [usage](docs/build-python-workflow-usage.md) | [example](docs/examples/build-python-example.yml) |
| `build-docker.yml` | Buildx multi-arch image; push to GHCR | `packages: write` | [usage](docs/build-docker-workflow-usage.md) | [example](docs/examples/build-docker-example.yml) |

The full validate → plan → apply pipeline across dev and prod is in [docs/examples/terraform-pipeline-example.yml](docs/examples/terraform-pipeline-example.yml).

### Composite actions

Building blocks used by the workflows above. You can also use them directly in your own jobs.

| Action | What it does | Docs |
|---|---|---|
| `tf-setup` | Install Terraform → `aws-env` (optional) → `terraform init` | [usage](docs/tf-setup-action-usage.md) |
| `aws-env` | Export the caller's secrets and vars as env vars, then assume `DEPLOYMENT_ROLE_ARN` via OIDC | [usage](docs/aws-env-action-usage.md) |
| `pr-comment` | Post or update one collapsible PR comment from a file, one per title + environment + directory | [usage](docs/pr-comment-action-usage.md) |

### Internal (this repo only)

| File | What it does |
|---|---|
| `_ci.yml` | This repo's PR checks: yamllint, actionlint, PR validation |
| `_bump-terraform.yml` | Weekly: opens a PR when a newer Terraform release is out, bumping the default `terraform-version` everywhere |
| `dependabot.yml` | Weekly PRs to update pinned action SHAs in workflows and `actions/*` |

---

## Using a workflow in your repo

1. **Copy an example** from `docs/examples/` into your repo's `.github/workflows/`. The comment at the top says what to name it and what setup it needs. The matching `docs/*-usage.md` page lists every input.
2. **Edit it** for your repo: `working-directory`, environment names, `init-args` and `plan-args`, language versions, `install-command`.
3. **For AWS workflows, set up GitHub environments.** Under **Settings → Environments**, create one per environment (`dev`, `prod`, …) with these variables:
   - `DEPLOYMENT_ROLE_ARN`: the IAM role GitHub assumes via OIDC
   - `AWS_DEFAULT_REGION`

   Add **Required reviewers** to `prod` to gate applies.
4. **Pass `secrets: inherit`** on jobs that use AWS workflows, so the workflow can export your secrets to its steps.
5. **Grant permissions on the calling job.** A reusable workflow can't have more permissions than its caller grants; see the table above.

Minimal example:

```yaml
jobs:
  plan:
    uses: YOUR_ORG/platform-workflows/.github/workflows/tf-plan.yml@v1
    permissions: { id-token: write, contents: read, pull-requests: write }
    secrets: inherit
    with:
      environment: dev
      working-directory: infra
```

### Python projects

`build-python.yml` and `cdk-diff.yml` use `actions/setup-python` and pip. Set `install-command` to match your project (default `pip install -r requirements.txt`). The pip cache is keyed on `requirements*.txt` and `pyproject.toml`, so the project needs at least one of those files.

### Pinning a version

| Ref | When to use it |
|---|---|
| `@v1` | Recommended. Gets fixes and new features, no breaking changes. |
| `@v1.2.3` | Exact version. Upgrade deliberately. |
| `@<commit-sha>` | Strictest supply-chain control. |
| `@main` | Only for testing changes to this repo. |

### Migrating from the older workflows

| Old | New | What to change in callers |
|---|---|---|
| `terraform-plan.yml` | `tf-plan.yml` | Update the `uses:` path. Inputs are unchanged; `init-args` is new and optional. |
| `_pr-lint.yml` | `pr-validation.yml` | Update the `uses:` path and pass `auto-assign: true` (the default is now `false`). |
| `groups:` input (`cdk-diff`, `build-python`) | `install-command:` | uv was removed; pass a pip command instead, e.g. `pip install -e .[cdk]`. |

---

## Contributing

- **Reusable workflows** go directly in `.github/workflows/`. Group them with a name prefix (`tf-*`, `build-*`), and prefix internal ones with `_`.
- **Repeated step sequences** go in `actions/<name>/action.yml`.
- **Inside reusable workflows, reference composite actions by full path:** `YOUR_ORG/platform-workflows/actions/<name>@v1`. A `./` path resolves against the *caller's* repo and will break.
- **Pin third-party actions to a commit SHA** with a version comment, e.g. `uses: actions/checkout@<sha>  # v6.0.2`. Dependabot updates both together.
- **Never put `${{ inputs.* }}` directly into `run:` or `script:`.** Pass values through `env:` to avoid script injection.
- **For anything new,** add a `docs/<name>-workflow-usage.md` page (or `-action-usage.md`), an example in `docs/examples/<name>-example.yml`, and a row in the tables above. Follow the existing pages: Overview, Features, Inputs, Permissions, Usage.
- **Lint before opening a PR**; `_ci.yml` runs the same checks:

  ```bash
  pip install yamllint && yamllint .   # uses .yamllint
  actionlint                           # https://github.com/rhysd/actionlint
  ```

### Keeping versions current

- **Action SHAs:** Dependabot opens weekly PRs.
- **Default Terraform version:** `_bump-terraform.yml` opens a PR each Monday when a new stable release is out. Run it by hand from the Actions tab to bump now, or to set a specific version.
- A merged bump reaches callers on `@v1` after the next release tag. Repos that need a fixed version should pass `terraform-version` explicitly.

### First-time setup of this repo

1. Replace `YOUR_ORG` with your GitHub org in the workflows, actions, docs and examples.
2. If the repo is private or internal, open **Settings → Actions → General → Access** and allow other repos in the org to use it.
3. Under **Settings → Actions → General → Workflow permissions**, enable **"Allow GitHub Actions to create and approve pull requests"** (needed by `_bump-terraform.yml`).
4. Tag the first release (see below).

### Releasing

```bash
git tag v1.2.3 && git push origin v1.2.3
git tag -f v1 v1.2.3 && git push -f origin v1   # move the major tag
```

For a breaking change (renamed input, removed workflow), release `v2` and leave `v1` in place until callers have moved.