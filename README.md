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

Publishes stable or beta releases using Changesets. Two publish strategies are supported:

- **`npm`** (default) — publishes to the npm registry with OIDC provenance (`latest` for stable, `beta` for snapshots).
- **`tags`** — for GitHub Action repos: tags `vX.Y.Z`, force-moves the major tag `vX`, and creates GitHub Releases (latest for stable, prerelease for snapshots). No npm publish.

```yaml
jobs:
  release:
    uses: hexadrop/github-workflows/.github/workflows/release.yml@v1
    with:
      snapshot: false
      concurrency-group: release
```

GitHub Action repo (tags strategy):

```yaml
jobs:
  release:
    uses: hexadrop/github-workflows/.github/workflows/release.yml@v1
    with:
      snapshot: false
      concurrency-group: release
      publish-strategy: tags
      build-command: bun run build
```

| Input               | Required | Default | Description                                                                                          |
|---------------------|----------|---------|------------------------------------------------------------------------------------------------------|
| `snapshot`          | no       | `false` | Publish a beta snapshot                                                                              |
| `concurrency-group` | yes      | -       | Concurrency group name                                                                               |
| `publish-strategy`  | no       | `npm`   | `npm` publishes to the npm registry; `tags` creates git tags and GitHub Releases                     |
| `build-command`     | no       | `''`    | Build command run before tagging (tags strategy only); `dist/index.js` is committed if it changed    |

With the `tags` strategy, stable releases create the tag `vX.Y.Z` (from `package.json`), force-move the major tag `vX`, and create a GitHub Release with `--latest --generate-notes`. Snapshot releases compute a beta version via `bun changeset version --snapshot beta`, restore `package.json`/`CHANGELOG.md` afterwards, tag the snapshot version, and create a GitHub prerelease. Existing tags are skipped, so reruns are safe.

| Output      | Description                                 |
|-------------|---------------------------------------------|
| `published` | `'true'` if packages were published or tags created |

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

| Input             | Required | Default                 | Description                                                                                                  |
|-------------------|----------|-------------------------|--------------------------------------------------------------------------------------------------------------|
| `base-branch`     | no       | `main`                  | Base branch for the release PR                                                                               |
| `version-command` | no       | `bun changeset version` | Command used to version the release. GitHub Action repos can use `bun changeset version && bun run build && git add dist/index.js` to ship the rebuilt bundle in the release PR |

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

After a backward-compatible change merges to `main`, a maintainer tags a new semver release (e.g. `v1.8.0`) and force-moves the floating alias (`git tag -f v1 v1.8.0 && git push -f origin v1`). Only then do `@v1` callers pick up the change; until the alias moves, existing consumers are unaffected.
