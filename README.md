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

### `renovate-changesets.yml`

Adds a Changesets entry to dependency pull requests. The commit is pushed with a GitHub App token (`hexabot`) instead of `GITHUB_TOKEN`, so the checks of the new commit start without maintainer approval and every event stays attributed to the app.

```yaml
on:
  pull_request_target:
    types: [opened, synchronize, reopened, labeled]

permissions: {}

jobs:
  changeset:
    uses: hexadrop/github-workflows/.github/workflows/renovate-changesets.yml@v1
    secrets:
      app-private-key: ${{ secrets.HEXABOT_PRIVATE_KEY }}
```

| Input   | Required | Default        | Description                               |
|---------|----------|----------------|-------------------------------------------|
| `label` | no       | `dependencies` | Label that marks dependency pull requests |

| Secret            | Required | Description                             |
|-------------------|----------|-----------------------------------------|
| `app-private-key` | yes      | Private key of the `hexabot` GitHub App |

The app id is read from the `HEXABOT_APP_ID` variable. The workflow does not use `GITHUB_TOKEN` at all: it is scoped to `permissions: {}` and every API call goes through the app installation token, limited to the current repository with `contents: write` and `pull-requests: write`.

### `release.yml`

Publishes stable or beta releases using Changesets. The workflow dispatches to one of four conditional jobs based on the `target` and `snapshot` inputs:

- `target: registry` + `snapshot: false` → stable npm release.
- `target: registry` + `snapshot: true` → beta npm snapshot.
- `target: tags` + `snapshot: false` → stable git tags + GitHub Release.
- `target: tags` + `snapshot: true` → beta git tag + GitHub prerelease.

Registry targets publish to npm with OIDC provenance. Tags targets are intended for GitHub Action repos and do not publish to npm.

```yaml
jobs:
  release:
    uses: hexadrop/github-workflows/.github/workflows/release.yml@v1
    with:
      snapshot: false
      concurrency-group: release
```

GitHub Action repo (tags target):

```yaml
jobs:
  release:
    uses: hexadrop/github-workflows/.github/workflows/release.yml@v1
    with:
      snapshot: false
      concurrency-group: release
      target: tags
```

| Input               | Required | Default      | Description                                                                                             |
|---------------------|----------|--------------|---------------------------------------------------------------------------------------------------------|
| `snapshot`          | no       | `false`      | Publish a beta snapshot                                                                                 |
| `concurrency-group` | yes      | -            | Concurrency group name                                                                                  |
| `target`            | no       | `'registry'` | `registry` publishes to the npm registry; `tags` creates git tags and GitHub Releases                   |
| `publish-strategy`  | no       | `'npm'`      | **Deprecated.** Use `target` instead. Accepts `npm`/`registry` or `tags`.                               |
| `package-json-path` | no       | `package.json` | Manifest that holds the version. Set it in monorepos, where the root manifest has no version.         |

With the `tags` target, stable releases create the tag `vX.Y.Z` (from `package-json-path`), force-move the major tag `vX`, and create a GitHub Release with `--latest --generate-notes`. Snapshot releases compute a beta version via `bun changeset version --snapshot beta`, restore `package.json`/`CHANGELOG.md` afterwards, tag the snapshot version, and create a GitHub prerelease. Existing tags are skipped, so reruns are safe.

| Output      | Description                                              |
|-------------|----------------------------------------------------------|
| `published` | `'true'` if packages were published or tags created      |

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
