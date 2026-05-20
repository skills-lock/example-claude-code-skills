---
name: changelog-summary
description: |
  Summarize recent CHANGELOG.md entries since the last release tag.
  Uses git + cat. Does not modify files or call external services.
  Use when you want a paragraph summary of what shipped since the
  last tag, suitable for pasting into a Slack post or a release
  email.
---

# changelog-summary

Walk the repo's `CHANGELOG.md` and produce a 2-3 sentence summary of
changes since the last release tag.

## Steps

1. Find the last release tag:

```bash
git describe --tags --abbrev=0
```

2. Read the changelog:

```bash
cat CHANGELOG.md
```

3. Identify the section for the most recent release and summarize it
   in 2-3 sentences. Highlight breaking changes first, then notable
   features, then bug fixes.

## Constraints

- Read-only. Do not push, do not modify files, do not call any
  network endpoint.
- If `CHANGELOG.md` is missing, report that and stop.
