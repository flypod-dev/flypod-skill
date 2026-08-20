---
name: flypod
description: 'Deploy a folder of static files or Markdown to a live, public URL anyone can open — no account, no config. Use it when the user asks to deploy, publish, or share something built; when a build has just finished and someone else needs to look at it; when a report, research write-up, or set of Markdown notes should become a browsable site instead of a file; or when a screenshot or a local dev server will not do because the recipient must open it themselves. Static output only — flypod serves files, and does not run a server or build them.'
---

# Deploy with flypod

flypod turns a folder into a live, public URL with one command. You bring the
files; flypod serves them.

## Deploy

1. Build first, if the project has a build — flypod serves files, it does not
   make them. You want a folder whose contents *are* the finished site.
2. Deploy it:

   ```sh
   npx flypod ./dist --json
   ```

3. Read `url` from the JSON and give it to the user.

**Done when** the user has the URL, and `warnings` is empty or you have told
them what is in it.

Omit the path and flypod auto-detects a build folder, trying `dist`, `build`,
`out`, `.output/public`, `.vercel/output/static`, `public`, `_site` — each of
which must hold an `index.html`. Pass the path whenever you know it.

A single file deploys too: `flypod report.html`. For several, deploy the folder.

No `flypod` binary? `npx flypod ...` downloads and runs it on the fly, and that
is the normal path rather than a fallback. To install once: `npm install -g flypod`.

## The result

```json
{
  "ok": true,
  "url": "https://ab12cd34.flypod.dev",
  "site_id": "ab12cd34",
  "version_id": "…",
  "expires_at": 1782929906088,
  "files": 12,
  "bytes": 1234567,
  "elapsed_ms": 812,
  "healed": ["spa-fallback"],
  "warnings": []
}
```

- `url` — the live site. Give it to the user.
- `warnings` — problems that survived the deploy, each with a `code` and a
  `fix_recipe` naming the change to make. A deploy exits 0 and still serves a
  broken site; this array is how you find out before the user does.
- `healed` — problems flypod fixed for you, e.g. `spa-fallback`.
- `site_id` — names the site for `--site <id>` from any other folder.
- `expires_at` — epoch **milliseconds** while the site is temporary, `null` once
  it belongs to an account.
- `manage_token` / `claim_token` — secrets, on a first temporary deploy. The CLI
  has already saved what it needs, keyed by the folder you deployed from, so
  these want no action from you. Keep them out of chat, commits, logs, and PR
  descriptions.

`-q` prints the URL alone, for a shell variable: `URL=$(npx flypod ./dist -q)`.

## Temporary, then yours

A deploy with no account is **temporary** and expires in **14 days** — right for
previews, demos, and throwaways. `flypod login` makes every temporary site this
machine deployed **yours**: no expiry, listed by `flypod list`.

Because login claims them all at once and not just the last one, the honest
advice is "deploy now, log in whenever you want to keep these." Raise it when
the user will still want the site next week, and let it go otherwise. `flypod
login` is how a site is kept — the tokens above are never the answer.

For agents and CI, set a session token and every deploy is attributed to the
account from the start, with no browser:

```sh
FLYPOD_TOKEN=<session-token> npx flypod ./dist --json
```

## Shipping a change

A second `flypod` **forks** the site: a new site on a new URL, while the link
you already gave the user quietly goes stale. `flypod update` adds a version to
the site you already made, and the URL holds.

```sh
flypod update ./dist
```

## What flypod serves

Static files: HTML, CSS, browser JavaScript, assets, Markdown. An app that needs
a server at request time — API routes, server-side rendering, a database call —
belongs somewhere else, and saying so beats shipping a build that breaks on
arrival. Most frameworks have a static export that deploys fine.

## Deeper reference

Open these when the task calls for them.

| File | Read it when |
| --- | --- |
| `reference/updates-and-versions.md` | shipping a change without forking the site, or listing and rolling back versions |
| `reference/publishing-markdown.md` | publishing Markdown — a report, research notes, a changelog, docs |
| `reference/comments.md` | collecting feedback on a preview, or reading reviewer comments back |
| `reference/troubleshooting.md` | a deploy failed or the site looks wrong, or you need exit codes, CI setup, env vars |
