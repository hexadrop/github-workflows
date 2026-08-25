# Hexadrop GitHub Workflows

Reusable GitHub Actions workflows shared across Hexadrop repositories.

## Workflows

### `check.yml`

Runs the project check matrix (lint, typecheck, tests, etc.).

```yaml
jobs:
  check:
    uses: hexadrop/github-workflows/.github/workflows/check.yml@v1
    with:
      commands: '["lint:ci", "typecheck"]'
      build-types: true
      eslint-cache: true
```

| Input | Required | Default | Description |
|---|---|---|---|
| `commands` | no | `["lint:ci", "typecheck"]` | JSON array of commands to run |
| `build-types` | no | `true` | Run `bun build:types` before the commands |
| `eslint-cache` | no | `true` | Enable ESLint cache for `lint:ci` |

### `release.yml`

Publishes stable or beta releases to npm using Changesets and OIDC provenance.

```yaml
jobs:
  release:
    uses: hexadrop/github-workflows/.github/workflows/release.yml@v1
    with:
      snapshot: false
      concurrency-group: release
```

| Input | Required | Default | Description |
|---|---|---|---|
| `snapshot` | no | `false` | Publish a beta snapshot |
| `concurrency-group` | yes | - | Concurrency group name |

| Output | Description |
|---|---|
| `published` | `'true'` if packages were published |

### `release-prepare.yml`

Creates or updates the release pull request from changesets.

```yaml
jobs:
  release-prepare:
    uses: hexadrop/github-workflows/.github/workflows/release-prepare.yml@v1
    with:
      build-types: true
      base-branch: main
```

| Input | Required | Default | Description |
|---|---|---|---|
| `build-types` | no | `true` | Run `bun build:types` before creating the release PR |
| `base-branch` | no | `main` | Base branch for the release PR |

## Versioning

This repository follows semantic versioning via Git tags:

- `v1.0.0` — pinned release
- `v1` — floating alias that receives backward-compatible updates

Use `@v1` for flexibility or `@v1.0.0` for reproducibility.
