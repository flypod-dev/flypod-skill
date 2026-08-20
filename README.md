# flypod-skill

Agent-skill **packaging** for [flypod](https://flypod.dev).

This repository contains the agent-skill packaging for **flypod**, a hosted
service at [flypod.dev](https://flypod.dev). The files here are instructions that
teach coding agents how to use the public `flypod` CLI on npm. flypod itself is a
hosted product and its source is not published; these skill files are MIT-licensed
so you can copy, fork, and adapt them freely. Bug reports about the service belong
at [flypod.dev](https://flypod.dev), not in this repository's issues.

## Install

```sh
npx skills add flypod-dev/flypod-skill
```

That works for [76+ coding agents](https://github.com/vercel-labs/skills). Target
specific ones with `-a`, or install for every project on the machine with `-g`:

```sh
npx skills add flypod-dev/flypod-skill -a claude-code -a codex -a cursor
npx skills add flypod-dev/flypod-skill -g
```

Or let an agent install it for you — see [`llms-install.md`](llms-install.md).

Already using flypod? The CLI writes the same skill itself, into
`.claude/skills/flypod/` and `.agents/skills/flypod/`:

```sh
npx flypod install skill
```

## What the skill teaches

One command turns a folder of built static files into a live, public URL:

```sh
npx flypod ./dist --json
# → { "ok": true, "url": "https://ab12cd34.flypod.dev", ... }
```

No account, no login, no config, no build pipeline. Deploys are temporary and
expire in 14 days; run `flypod login` and they become yours, with no expiry.
`flypod update` ships a new version to the same URL.

The skill uses progressive disclosure: a short entry file the agent always
loads, plus reference documents it opens only when the task calls for them.
Between them they cover the deploy loop, shipping updates without changing the
URL, publishing Markdown as a rendered site, collecting comments on a preview,
and troubleshooting a deploy that succeeded but looks wrong.

## Contents

```
plugin.json                 Agent Plugins 1.0.0 manifest
.claude-plugin/plugin.json  Claude Code plugin manifest
skills/flypod/
  SKILL.md                  the entry file — always loaded
  reference/
    updates-and-versions.md update, versions, rollback, --site
    publishing-markdown.md  deploying .md as a rendered site
    comments.md             review loop on a deployed preview
    troubleshooting.md      exit codes, warnings, CI, env vars
llms-install.md             agent-executable install instructions
LICENSE                     MIT — covers the files in this repository only
CHANGELOG.md                version history
```

## These files are generated

`skills/flypod/` is generated from the flypod CLI's own skill source, so the
packaged plugin and `npx flypod install skill` can never say different things.
Edit the source in the flypod repository, not the copies here; a test fails the
build if the two drift.

## Scope of the license

The MIT license in this repository covers **the skill and manifest files in this
repository only**. It grants no rights to the flypod service, its backend, its
brand, or any software distributed through the `flypod` npm package. Those are
governed by their own terms.

## Links

- Product: [flypod.dev](https://flypod.dev)
- Documentation: [docs.flypod.dev](https://docs.flypod.dev)
- CLI on npm: [`flypod`](https://www.npmjs.com/package/flypod)

## Support

Questions about the hosted service, deploy failures, accounts, or billing go to
[flypod.dev](https://flypod.dev). This repository's issues are for the skill
files themselves — a wrong command, a stale flag, a broken manifest.
