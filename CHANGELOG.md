# Changelog

## v0.1.0 — 2026-05-20

Initial release. Three example skills:

- `hello-greeter` — minimal Skill with no behavior surface
- `changelog-summary` — reads files + uses git
- `release-publisher` — uses git + gh; reaches `api.github.com`

Committed `skills.lock` baselines all three. Block-mode policy fails
any PR that adds new shell commands or reaches a non-allowlisted
network host.
