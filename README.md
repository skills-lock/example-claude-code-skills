# example-claude-code-skills

A worked example of how [**SkilLock**](https://github.com/skills-lock/skil-lock) pins approved AI Skill behavior and blocks unapproved drift in CI.

This repository ships three illustrative Claude Code skills, a committed `skills.lock`, a block-mode policy, and a GitHub Action workflow. Compare the `main` branch against the [`example/drift`](https://github.com/skills-lock/example-claude-code-skills/tree/example/drift) branch to see what SkilLock flags when a skill suddenly changes behavior.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](./LICENSE)

## The three skills

| Skill | Behavior surface | What it demonstrates |
|---|---|---|
| [`hello-greeter`](./.claude/skills/hello-greeter/SKILL.md) | (none) | The minimum viable skill - markdown only |
| [`changelog-summary`](./.claude/skills/changelog-summary/SKILL.md) | `cat`, `git`; reads `CHANGELOG.md` | A read-only skill that scans local state |
| [`release-publisher`](./.claude/skills/release-publisher/SKILL.md) | `cat`, `gh`, `git`; reaches `api.github.com` | A skill that talks to the network (allowlisted host) |

`skills.lock` (committed) records exactly that surface. The Action re-scans on every PR and posts a comment showing any delta.

## The policy

[`.skil-lock.yaml`](./.skil-lock.yaml) sets `mode: block`. Three rules:

- `require_approval: [shell_commands]` - any new shell command needs a paste-back approval entry
- `protected_paths` - covers `.env`, PEM keys, SSH keys, `**/secrets/**`
- `allowed_domains` - only `api.github.com` + `*.githubusercontent.com` are permitted; everything else triggers a flag

Block mode means the check fails (exits non-zero) when a PR adds any capability at severity ≥ medium that isn't covered by an approval entry.

## The drift branch

[`example/drift`](https://github.com/skills-lock/example-claude-code-skills/tree/example/drift) modifies `changelog-summary` to add:

- A `curl` command (new shell command - requires approval per the policy)
- A POST to `https://internal.example.com/notify` (host not in `allowed_domains`)

Opening a PR from `example/drift` to `main` produces a comment like this:

> ### SkilLock - capability delta
>
> Comparing `skills.lock` (baseline) vs `<working tree>` (current).
>
> | Skill | Capability | Change | Detail | Reason |
> |---|---|---|---|---|
> | changelog-summary | shell_commands | + | `curl` | matches require_approval |
> | changelog-summary | network_urls | + | `https://internal.example.com/notify` | host not in allowed_domains |
>
> **Verdict:** BLOCK: 2 of 2 entries at severity >= medium
>
> **To approve, append to `.skil-lock-approvals.yaml`:**
>
> ```yaml
> schema_version: "0.1"
> approvals:
>   - skill: "changelog-summary"
>     delta:
>       added_shell_command: "curl"
>     reviewer: "you@example.com"
>     reviewed_at: "2026-05-20T17:00:00Z"
>     reason: "<why this delta is acceptable>"
>   - skill: "changelog-summary"
>     delta:
>       added_network_url: "https://internal.example.com/notify"
>     reviewer: "you@example.com"
>     reviewed_at: "2026-05-20T17:00:00Z"
>     reason: "<why this delta is acceptable>"
> ```

A reviewer who recognises one of those as legitimate (e.g. the new internal endpoint is approved) copies the relevant block, fills in `reviewer` + `reason`, commits, pushes. The check re-runs green for that delta. Entries that aren't approved still block.

## Try it locally

```bash
git clone https://github.com/skills-lock/example-claude-code-skills.git
cd example-claude-code-skills

# Install skil-lock (Go 1.22+):
go install github.com/skills-lock/skil-lock/cmd/skil-lock@v0.1.1

# Scan and confirm the baseline passes:
skil-lock ci

# Switch to the drift branch and see the block:
git checkout example/drift
skil-lock ci   # exit code 1, BLOCK verdict
```

## Related repositories

- [skills-lock/skil-lock](https://github.com/skills-lock/skil-lock) - the CLI and lockfile spec
- [skills-lock/skil-lock-action](https://github.com/skills-lock/skil-lock-action) - the GitHub Action wrapper used here

## License

[Apache 2.0](./LICENSE).

## Trademarks

`SkilLock` and `skil-lock` are not affiliated with or endorsed by Skil power tools (a brand owned by Chervon Group). `Claude` and `Claude Code` are trademarks of Anthropic PBC. `Codex` is a trademark of OpenAI, OpCo, LLC. References here are descriptive (nominative fair use).
