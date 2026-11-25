---
name: flypod
description: 'Deploy a folder of static files or Markdown to a live share link anyone with it can open: no account, no config. Use it when the user asks to deploy, publish, or share something built; when a build has just finished and someone else needs to look at it; when a report, research write-up, or set of Markdown notes should become a browsable site instead of a file; or when a screenshot or a local dev server will not do because the recipient must open it themselves. Static output only: flypod serves files, and does not run a server or build them.'
---

# Deploy with flypod

flypod turns a folder into a live share link with one command. You bring the
files; flypod serves them. Anyone with the link can open it; without an account
the link is unlisted (search engines are told not to index it) and expires after
14 days.

## Deploy

1. Build first, if the project has a build: flypod serves files, it does not
   make them. You want a folder whose contents *are* the finished site.
2. Deploy it. **The first time**, `flypod` makes the site:

   ```sh
   npx flypod ./dist --json
   ```

   **Every later deploy** from the same folder updates that site: plain
   `flypod` again (or `flypod update`, the explicit spelling) ships a new
   version to the URL the user already has, and the receipt says
   `"action": "updated"` (see "Shipping a change"). `flypod --new` makes a
   separate site; run it only when the user asks for one.

3. Read `url` from the JSON and give it to the user.

**Done when** the user has the URL, and `warnings` is empty or you have told
them what is in it.

Omit the path and flypod auto-detects a build folder, trying `dist`, `build`,
`out`, `.output/public`, `.vercel/output/static`, `public`, `_site`. Each of
them must hold an `index.html` (or one app folder that does, as Angular's
`dist/<app>/browser`). A folder with an `index.html` and no `package.json` deploys
as it is. Pass the path whenever you know it. `flypod update` with no path
deploys the folder the site was last deployed from.

A single file deploys too: `flypod report.html`. For several, deploy the folder.

No `flypod` binary? `npx flypod ...` downloads and runs it on the fly, and that
is the normal path rather than a fallback. To install once: `npm install -g flypod`.

## The result

```json
{
  "ok": true,
  "action": "created",
  "url": "https://a3f1c2d4e5b60789.flypod.page",
  "site_id": "a3f1c2d4e5b60789",
  "version_id": "…",
  "expires_at": 1782929906088,
  "healed": ["spa-fallback"],
  "warnings": [],
  "link_saved": true,
  "next": { "update": "flypod update", "versions": "flypod versions", "rollback": "flypod rollback", "keep": "flypod login --json" }
}
```

- `action`: `"created"` on a folder's first deploy, `"updated"` when the
  folder already had a site and this run added a version to it.
- `url`: the live site. Give it to the user. Read it from the JSON rather than
  assembling it: every site is served at `https://<site_id>.flypod.page`, on a
  different domain from the API and docs (`flypod.dev`).
- `warnings`: problems that survived the deploy, each with a `code` and a
  `fix_recipe` naming the change to make. A deploy exits 0 and still serves a
  broken site; this array is how you find out before the user does.
- `healed`: problems flypod fixed for you, e.g. `spa-fallback`.
- `site_id`: names the site for `--site <id>` from any other folder.
- `expires_at`: epoch **milliseconds** while the site is temporary, `null` once
  it belongs to an account.
- `next`: the literal commands for what comes after. `next.update` is how a
  change is shipped to this same URL. `next.keep`, while the site is
  temporary, is how it is kept (see below).
- `link_saved`: whether `flypod update` works from this folder with no flags.
  When it is `false` (a read-only config dir, typical of CI), the `next`
  commands carry `--site <id>` and the receipt also has a `manage_token`:
  that is the only copy, so pass it as `--token` on later commands and keep it
  out of chat, commits, logs, and PR descriptions.
- `skipped`: present only when flypod left something out: `.env`,
  `.env.*`, `.git`, `node_modules` and OS litter never upload, nor does
  anything matched by a `.flypodignore` in the folder. Each entry has a
  `path` and a `reason`. If the site needs a file listed here, tell the user.

The receipt carries no credentials otherwise; the CLI has saved what it needs,
keyed by the folder you deployed from. `--json --verbose` adds tokens, file
and byte counts, and timing, for the rare job that needs them.

`-q` prints the URL alone, for a shell variable: `URL=$(npx flypod ./dist -q)`.

## Temporary, then yours

A deploy with no account is **temporary** and expires in **14 days**, right for
previews, demos, and throwaways. `flypod login` makes every temporary site this
machine deployed **yours**: no expiry, listed by `flypod list`.

Because login claims them all at once and not just the last one, the honest
advice is "deploy now, log in whenever you want to keep these." Raise it when
the user will still want the site next week, and let it go otherwise. `flypod
login` is how a site is kept; the tokens above are never the answer.

For agents and CI, set a session token and every deploy is attributed to the
account from the start, with no browser:

```sh
FLYPOD_TOKEN=<session-token> npx flypod ./dist --json
```

When the user wants to keep the site and you have no browser, run the
receipt's `next.keep`, which is `flypod login` with `--json` (without it, in
a terminal, login waits for a browser):

```sh
flypod login --json
# {"ok":false,"code":"interactive_login_required","error":"…","fix":"…","verification_uri":"https://…","verification_uri_complete":"https://…","user_code":"ABCD-1234","device_code":"…","expires_in":600,"interval":5}
```

It exits at once (code 2). Give the user `verification_uri_complete` and the
`user_code` and ask them to confirm. That is all; there is nothing to poll.
Once they have confirmed, your next flypod command of any kind (a deploy,
`flypod list`) finishes signing in by itself, keeps every temporary site this
machine deployed, and runs signed in.

To finish at once instead (the user has just said they confirmed), pass the
`device_code` (not the `user_code`) from that output:

```sh
flypod login --device-code <device_code> --json
# {"ok":true,"logged_in":true,"attached":[…],"skipped":[]}   or
# {"ok":false,"code":"authorization_pending",…}             exit 4
```

It waits up to 25 seconds for the confirmation. `authorization_pending` means
the user has not confirmed yet: carry on with the work, and the next command
finishes it once they do. `login_expired` means the code is spent (expired,
or already used): if an earlier run printed `"ok":true` you are signed in,
otherwise start again with `flypod login --json`.

A successful login also attaches every temporary site this machine deployed;
`attached` lists them.

## Shipping a change

In a folder that already has a site, plain `flypod` adds a version to it and
the URL holds. `flypod update` does the same and says so outright (it also
takes `--site <id>` from any other folder):

```sh
flypod update ./dist
```

`flypod --new` **forks** the site: a separate site on a new URL, while the link
you already gave the user stays on the earlier one. Run it only when the user
asks for a second site; the receipt then carries `replaced_link` and a
`new-site-in-linked-folder` warning.

### Let a reviewer compare versions

```sh
flypod update --timeline              # with the next deploy
flypod versions --timeline --json     # now, without deploying
flypod versions --no-timeline         # off again
```

`--timeline` (also on `flypod`) puts a History chip on the site's pages that lists
its versions and opens any of them, plus a page that compares two side by
side (`<site url>/_flypod/history`), without changing what is live. Offer it when the user is
choosing between iterations. It is off by default because it makes replaced
versions public: do not turn it on for a site where an earlier version held
something that was removed on purpose. With it on, `--json` carries a
`history_url` to give the user (two versions side by side), and
`flypod versions --json` a `url` for each version. A version that is not live is
deleted seven days after it was deployed, so the timeline shows about a week.

## Where sites live

Every deployed site, temporary or claimed, is served at
`https://<site_id>.flypod.page`. The `flypod.dev` domain is the API, the docs,
the marketing site, and account pages, and it never serves uploaded content.
Splitting them is deliberate: deploys take no account, so anything anyone
uploads can only affect the reputation of a domain flypod does not depend on.
Claiming a site does not move it: the URL you were given is the URL you keep.

A site id is 16 hex characters and flypod picks it; there are no custom or
chosen subdomain names, and no custom domains. Site URLs handed out before
2026-09-14 used `<site_id>.flypod.dev`; those redirect to the same id on
`flypod.page` until 2026-12-13.

## What flypod serves

Static files: HTML, CSS, browser JavaScript, assets, Markdown. An app that needs
a server at request time (API routes, server-side rendering, a database call)
belongs somewhere else, and saying so beats shipping a build that breaks on
arrival. Most frameworks have a static export that deploys fine.

Paths resolve the way a static export expects: `/about` serves
`about.html`, `/blog/` serves `blog/index.html`, `/blog` redirects there,
and a `404.html` in the folder is used for missing pages. Nothing to configure.

Every site also answers `/llms.txt` with a list of its pages, generated from
the deploy unless the folder ships its own. Another agent handed the URL can
start there; you do not need to write the file.

## Deeper reference

Open these when the task calls for them.

| File | Read it when |
| --- | --- |
| `reference/updates-and-versions.md` | shipping a change without forking the site, or listing and rolling back versions |
| `reference/publishing-markdown.md` | publishing Markdown: a report, research notes, a changelog, docs |
| `reference/comments.md` | collecting feedback on a preview, or reading reviewer comments back |
| `reference/troubleshooting.md` | a deploy failed or the site looks wrong, or you need exit codes, CI setup, env vars |
