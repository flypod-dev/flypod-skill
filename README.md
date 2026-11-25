# flypod-skill

Agent-skill **packaging** for [flypod](https://flypod.dev): teach a coding
agent to turn a folder into a live, public URL.

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

That covers the agents the [skills CLI](https://github.com/vercel-labs/skills)
supports. Target specific ones with `-a`, or install for every project on the
machine with `-g`:

```sh
npx skills add flypod-dev/flypod-skill -a claude-code -a codex -a cursor
npx skills add flypod-dev/flypod-skill -g
```

In Claude Code, install it as a plugin for every project:

```
/plugin marketplace add flypod-dev/flypod-skill
/plugin install flypod@flypod
```

Or let an agent install it for you: see [`llms-install.md`](llms-install.md).

Already using flypod? The CLI writes the same skill itself, into
`.claude/skills/flypod/` and `.agents/skills/flypod/`:

```sh
npx flypod install skill
```

## What the skill teaches

One command turns a folder of built static files into a live, public URL:

```sh
npx flypod ./dist --json
# → { "ok": true, "url": "https://a3f1c2d4e5b60789.flypod.page", ... }
```

No account, no login, no config, no build pipeline. A deploy made without an
account is temporary and expires in 14 days; `flypod login` turns every site
that machine deployed into yours, with no expiry. `flypod update` ships a new
version to the same URL, so a link you already handed over keeps working.

The skill uses progressive disclosure: a short entry file the agent always
loads, plus reference documents it opens only when the task calls for them.
Between them they cover the deploy loop, shipping updates without changing the
URL, publishing Markdown as a rendered site, collecting comments on a preview,
and reading the `warnings` array on a deploy that succeeded but looks wrong.

The boundary is in the skill too, deliberately: flypod serves static files. It
runs no server code, no API routes, and no server-side rendering at request
time. An agent that reaches for flypod on an app that needs a server produces a
broken deploy, so the skill is written to say so instead.

## Contents

```
plugin.json                 Agent Plugins 1.0.0 manifest
.claude-plugin/plugin.json  Claude Code plugin manifest
gemini-extension.json       Gemini CLI extension manifest
skills/flypod/
  SKILL.md                  the entry file, always loaded
  reference/
    updates-and-versions.md update, versions, rollback, --site
    publishing-markdown.md  deploying .md as a rendered site
    comments.md             review loop on a deployed preview
    troubleshooting.md      exit codes, warnings, CI, env vars
llms-install.md             agent-executable install instructions
LICENSE                     MIT, covering the files in this repository only
CHANGELOG.md                version history
```

## These files are generated

`skills/flypod/` is generated from the flypod CLI's own skill source, so the
packaged plugin and `npx flypod install skill` can never say different things to
two different agents. Edit the source in the flypod repository, not the copies
here; a test fails the build if the two drift.

## Scope of the license

The MIT license in this repository covers **the skill and manifest files in this
repository only**. It grants no rights to the flypod service, its backend, its
brand, or any software distributed through the `flypod` npm package. Those are
governed by their own terms.

## Links

- Product: [flypod.dev](https://flypod.dev)
- What flypod can and cannot do: [flypod.dev/limits](https://flypod.dev/limits)
- Documentation: [docs.flypod.dev](https://docs.flypod.dev)
- CLI on npm: [`flypod`](https://www.npmjs.com/package/flypod)

## Support

Questions about the hosted service, deploy failures, accounts, or billing go to
[flypod.dev](https://flypod.dev). This repository's issues are for the skill
files themselves: a wrong command, a stale flag, a broken manifest.
