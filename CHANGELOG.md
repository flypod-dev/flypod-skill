# Changelog

All notable changes to the flypod skill packaging are recorded here. The version
here, `plugin.json`, `.claude-plugin/plugin.json`, and the git tag must always
match — version drift is a known reviewer complaint across plugin directories.

## 0.2.0

- `skills/flypod/` is now **generated** from the flypod CLI's own skill source
  (`bun scripts/build-skill-package.ts` in the flypod repo), and a test fails
  the build if the two drift. The previous release was a hand-copied snapshot
  and had already gone stale against the CLI.
- Progressive disclosure: a shorter `SKILL.md` plus `reference/` docs the agent
  opens only when the task needs them.
- New reference material the skill never covered — publishing Markdown as a
  rendered site, the comments review loop, deploy `warnings` and their
  `fix_recipe`, exit codes, and environment variables.
- Ownership rewritten as **temporary → yours** via `flypod login`, replacing the
  old "anonymous vs. signed in" framing. The skill no longer tells agents to
  save a `manage_token`; the CLI stores it for them.

## 0.1.0

Initial release.

- Agent Plugins 1.0.0 manifest (`plugin.json`) and Claude Code manifest
  (`.claude-plugin/plugin.json`).
- `skills/flypod/SKILL.md` — deploy, update, versions, rollback, anonymous vs.
  signed-in, and the static-only boundary.
- `llms-install.md` for agent self-installation.
