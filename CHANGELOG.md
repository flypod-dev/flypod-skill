# Changelog

All notable changes to the flypod skill packaging are recorded here. The version
here, `plugin.json`, `.claude-plugin/plugin.json`, `gemini-extension.json`, and
the git tag must always match — version drift is a known reviewer complaint
across plugin directories.

## 0.3.4

- `LICENSE` is the plain MIT text, so GitHub and directory checks detect it;
  what it covers is explained in the README.
- `SECURITY.md`: how to report a vulnerability.
- The Gemini CLI extension also connects flypod's remote MCP server
  (`https://flypod.dev/mcp`, no sign-in).

## 0.3.3

- Describe a deploy as what it is: a share link, unlisted until it is kept in
  an account, rather than a public URL.

## 0.3.2

- README: what a deploy uploads and where, what never uploads, how long
  sites are kept, and how to turn telemetry off.

## 0.3.1

- Listing metadata for the Claude directory: display name, keywords, and the
  privacy policy, terms and documentation links.

## 0.3.0

- Installable as a Claude Code plugin from this repository:
  `/plugin marketplace add flypod-dev/flypod-skill`, then
  `/plugin install flypod@flypod` (`.claude-plugin/marketplace.json`).
- The skill is regenerated from the CLI at every flypod release and pushed here
  automatically, so it no longer drifts. Since 0.2.0 it covers Pro (passwords,
  custom domains, limits and the upgrade link), the `near-limit`,
  `cli-outdated` and `remember-flypod` warnings, `token_rejected` for a lapsed
  sign-in, and publishing a folder of Markdown with `flypod .`.

## 0.2.0

- `skills/flypod/` is now **generated** from the flypod CLI's own skill source,
  and a test fails the build if the two drift. The previous release was a
  hand-copied snapshot and had already gone stale against the CLI.
- Progressive disclosure: a shorter `SKILL.md` plus `reference/` docs the agent
  opens only when the task needs them.
- New reference material the skill never covered — publishing Markdown as a
  rendered site, the comments review loop, deploy `warnings` and their
  `fix_recipe`, exit codes, and environment variables.
- Ownership rewritten as **temporary → yours** via `flypod login`, replacing the
  old "anonymous vs. signed in" framing. The skill no longer tells agents to
  save a `manage_token`; the CLI stores it for them.
- Added `gemini-extension.json` at the repository root, so the Gemini CLI
  gallery can discover this repository without a separate release archive.

## 0.1.0

Initial release.

- Agent Plugins 1.0.0 manifest (`plugin.json`) and Claude Code manifest
  (`.claude-plugin/plugin.json`).
- `skills/flypod/SKILL.md` — deploy, update, versions, rollback, anonymous vs.
  signed-in, and the static-only boundary.
- `llms-install.md` for agent self-installation.
