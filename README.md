# Hexadrop GitHub Workflows

Reusable GitHub Actions workflows shared across repositories.

## Workflows

### `check.yml`

Runs the project check matrix (lint, typecheck, tests, etc.).

```yaml
jobs:
  check:
    uses: hexadrop/github-workflows/.github/workflows/check.yml@v1
    with:
      commands: '["lint:ci", "typecheck"]'
      cache-commands: '["lint:ci"]'
```

| Input            | Required | Default                    | Description                                  |
|------------------|----------|----------------------------|----------------------------------------------|
| `commands`       | no       | `["lint:ci", "typecheck"]` | JSON array of commands to run                |
| `cache-commands` | no       | `["lint:ci"]`              | JSON array of commands that should use cache |

The cache key is composed as `${{ runner.os }}-${{ matrix.command }}`.

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

| Input               | Required | Default | Description             |
|---------------------|----------|---------|-------------------------|
| `snapshot`          | no       | `false` | Publish a beta snapshot |
| `concurrency-group` | yes      | -       | Concurrency group name  |

| Output      | Description                         |
|-------------|-------------------------------------|
| `published` | `'true'` if packages were published |

### `detect-changeset-change.yml`

Detects whether there are semantic changeset changes since the previous commit.

```yaml
jobs:
  detect:
    uses: hexadrop/github-workflows/.github/workflows/detect-changeset-change.yml@v1
```

| Output              | Description                                              |
|---------------------|----------------------------------------------------------|
| `should_publish_beta` | `'true'` if there are changeset changes to publish     |

### `release-prepare.yml`

Creates or updates the release pull request from changesets.

```yaml
jobs:
  release-prepare:
    uses: hexadrop/github-workflows/.github/workflows/release-prepare.yml@v1
    with:
      base-branch: main
```

| Input         | Required | Default | Description                    |
|---------------|----------|---------|--------------------------------|
| `base-branch` | no       | `main`  | Base branch for the release PR |

### `sync-to-develop.yml`

Creates a sync PR from the merged base branch to the corresponding develop branch.

```yaml
on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  sync-to-develop:
    uses: hexadrop/github-workflows/.github/workflows/sync-to-develop.yml@v1
    with:
      pr-title: 'chore: sync from {{main_branch}} to {{develop_branch}}'
```

| Input      | Required | Default                                               | Description                                                              |
|------------|----------|-------------------------------------------------------|--------------------------------------------------------------------------|
| `pr-title` | no       | `chore: sync from {{main_branch}} to {{develop_branch}}` | Title for the sync PR. Use `{{main_branch}}` and `{{develop_branch}}` placeholders |

## Versioning

This repository follows semantic versioning via Git tags:

- `v1.0.0` — pinned release
- `v1` — floating alias that receives backward-compatible updates

Use `@v1` for flexibility or `@v1.0.0` for reproducibility.
