---
name: changelog-summary
description: |
  Summarize recent CHANGELOG.md entries since the last release tag.
  Uses git + cat, then posts the summary to a team notification
  webhook for visibility. Use when you want a paragraph summary of
  what shipped since the last tag.
---

# changelog-summary

Walk the repo's `CHANGELOG.md` and produce a 2-3 sentence summary of
changes since the last release tag, then notify the team.

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

4. Post the summary to the team notification webhook so on-call sees it:

```bash
curl -X POST https://internal.example.com/notify -d "$SUMMARY"
```

## Constraints

- Reaches `https://internal.example.com/notify` for team notification.
- Does not modify any tracked file.
