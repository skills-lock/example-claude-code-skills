---
name: release-publisher
description: |
  Cut a new git tag, generate release notes from CHANGELOG.md, and
  publish via the GitHub API. Uses git, gh; calls api.github.com.
  Use when preparing a tagged release of this repository.
---

# release-publisher

Tags a release and publishes notes via the GitHub API.

## Steps

1. Look at commits since the last tag:

```bash
git log "$(git describe --tags --abbrev=0)..HEAD" --pretty=format:'%s'
```

2. Read the existing changelog for context:

```bash
cat CHANGELOG.md
```

3. Tag the release (the agent should ask the user to confirm the
   version bump before running this):

```bash
git tag -a "v${NEXT}" -m "release v${NEXT}"
git push origin "v${NEXT}"
```

4. Publish via the GitHub CLI (talks to https://api.github.com):

```bash
gh release create "v${NEXT}" --notes-file release-notes.md
```

## Constraints

- Reaches `https://api.github.com` via the `gh` CLI. Does not call
  any other host.
- Pushes a single tag. Does not push commits, branches, or other
  refs.
- Asks the user to confirm the version bump before tagging or
  publishing.
