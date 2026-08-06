# bump

[![Release](https://img.shields.io/github/v/release/libnudget/bump?logo=github&label=latest)](https://github.com/libnudget/bump/releases)

Reusable GitHub Action for version bump automation after merged pull requests.

It is intended for repositories that already keep their versioning logic in repo-local files and scripts,
while wanting the PR orchestration, tagging, and release flow to live in one reusable action.

## What it does

- determines bump type from PR labels with patch as the fallback
- skips bumping when the merged PR is itself a version bump PR
- runs a repository-local version bump script
- opens a version bump PR on a `version-bump-X.Y.Z` branch
- creates the release tag when the version bump PR is merged
- creates a GitHub release and dispatches a release workflow

## Expected repository shape

This action assumes the target repository provides:

- a version bump script, default `./scripts/version.sh`
- a version file, default `VERSION`
- a pubspec file to stage in the bump PR, default `pubspec.yaml`
- a whats-new JSON file to stage in the bump PR, default `assets/whats_new.json`
- a release workflow that accepts `inputs.tag`, default `release.yml`

These paths are configurable through inputs.

## Example

```yaml
name: Version Bump

on:
  pull_request:
    types: [closed]
    branches: [main]

permissions:
  contents: write
  pull-requests: write
  actions: write

jobs:
  bump-version:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          fetch-tags: true
          token: ${{ secrets.GITHUB_TOKEN }}
      - uses: libnudget/bump@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr-number: ${{ github.event.pull_request.number }}
          tag-prefix: desktop/app
```

## Inputs

| Name | Required | Default | Notes |
| --- | --- | --- | --- |
| `token` | yes | | GitHub token with repo write access |
| `pr-number` | yes | | Merged PR number |
| `tag-prefix` | no | `desktop/app` | Release tag prefix |
| `base-branch` | no | `main` | Default branch |
| `version-script` | no | `./scripts/version.sh` | Repo-local version bump script |
| `version-file` | no | `VERSION` | Repo-local version file |
| `pubspec-file` | no | `pubspec.yaml` | File staged in version bump PR |
| `whats-new-file` | no | `assets/whats_new.json` | File staged in version bump PR |
| `release-workflow` | no | `release.yml` | Workflow filename or ID for dispatch |
| `git-user-name` | no | `github-actions[bot]` | Commit author name |
| `git-user-email` | no | bot noreply email | Commit author email |

## Notes

- The target repository remains responsible for its own version-file semantics.
- This action is meant to orchestrate the flow, not replace repository-specific version math.
- The bump label precedence is `major > minor > patch`, with `patch` as the default.

## License

MIT
