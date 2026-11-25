# Troubleshooting and environment

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | success |
| `1` | runtime error you can act on: no build folder, site gone, upload rejected or blocked, a free-plan limit spent (`quota_exceeded`) |
| `2` | usage error: unknown flag or bad arguments, or a login that needs a browser |
| `3` | auth: not signed in, credential refused (`unauthorized`, `token_rejected`), no manage token held (`no_token`), sign-in denied |
| `4` | retryable: rate limited, server 5xx, network failure (including a proxy refusing the request), sign-in still pending |

The exit code follows the failure's `code`, on every verb. With `--json`, every
failure prints one line on stdout and nothing on stderr:

```json
{"ok": false, "code": "rate_limited", "error": "…", "fix": "Wait a minute and retry the same command.", "status": 429, "retry_after_s": 30}
```

Branch on `code` (stable), never on `error` (prose). `fix` is the literal next
step when there is one; `status` is the HTTP status when a server answered. Codes you will meet: `no_build_dir`, `no_files`,
`no_site`, `not_signed_in`, `unauthorized`, `token_rejected`, `site_gone`,
`invalid_upload`, `too_large`, `blocked`, `challenge_required`,
`rate_limited`, `quota_exceeded`, `pro_required`, `domain_taken`, `server_error`, `network`, `unknown_flag`, `usage`.

`pro_required` means the feature (`--password`, `flypod domain`) needs flypod Pro:
show the human `upgrade_url`, and run the same command again once they have paid.
`domain_taken`: the domain is connected to another flypod site; tell the human.

`rate_limited` means wait a minute and retry. `quota_exceeded` is different: a
monthly free-plan limit is spent, and the same command fails until `resets_at`.
Do not retry it; tell the human the `limit` and the date, and the
`upgrade_url` when there is one.

`flypod update` has already retried a `server_error` or `network` failure once
before reporting it, so do not loop on it: wait a minute before one more try.
When flypod itself answered, the JSON carries a `request_id` to quote.

`usage` includes a value flag given no value (`--site needs a value`) and a
`--mode` that is not `anonymous`, `passcode` or `approved`. `unknown_flag`
names the closest real flag when there is one. `no_build_dir` also covers a
path that does not exist; its `fix` is the command to run, such as
"Run `npm run build`, then `flypod ./dist`" for a project that is not built.
`too_large` is checked before uploading to flypod.dev
(at most 200 files, 10 MB of files, 12 MB as an upload) and names the largest files.
A program or installer in the folder (`.exe`, `.apk`, `.dmg`, `.ps1`, …)
fails with `invalid_upload` naming the file: flypod does not host them, so
leave it out (`.flypodignore`) and link to a release page instead.
For `too_large`, read its `fix`: when the folder is a project root, or most of it is source maps,
video or images, it says so and names the folder or the `.flypodignore` lines to use.

`no_site` carries a `reason`: `never_linked` (run `flypod` to make the site),
`expired` (the temporary site ran out: run `flypod` for a new one, then
`flypod login --json` to keep it), `gone` (the site no longer exists: run `flypod`), or
`other_folder` (a folder above or below this one is linked: the `fix` gives the
`--site <id>` command). `unauthorized` on a version verb says what to do with the
credential this machine holds; when the site was kept by the account you are
signed in to, flypod retries as that account by itself. Plain `flypod` in a
folder that already has a site is an update, so it fails the same ways; its
`fix` then also names `flypod --new`, for when a separate site is what the
user wants.

Rarer ones: `unknown_command` (a verb that does not exist: `flypod verisons`
fails this way with `"fix": "flypod versions"` instead of deploying a folder
called `verisons`; or a `flypod comments` verb that does not exist),
`no_such_version` (nothing to roll back to), `no_token` (`flypod token` on a
site this machine holds no manage token for), `already_claimed`, `not_found`,
`bad_request` (any other 4xx), `unexpected`, and from the login flow
`login_denied` (exit 3), `login_expired` (exit 1, start again) and
`login_timeout` (exit 4).

A `FLYPOD_TOKEN` the server does not accept fails the deploy with
`token_rejected` (exit 3). It never falls back to an anonymous deploy, so an
exit 0 with a token set always means the site is owned. A saved sign-in that has
lapsed (30 days without use) gives `token_rejected` on any command, `flypod
list` included: ask the user to run `flypod login` again, and do not retry
the command first; it fails the same way until they do.

## "No build output folder found"

flypod looked for `dist`, `build`, `out`, `.output/public`,
`.vercel/output/static`, `public`, `_site` (each of which must contain an
`index.html`) and found none. Either the project has not been built yet, or the
output lives somewhere else. Build it, then pass the path explicitly:

```sh
npm run build && npx flypod ./path/to/output --json
```

## The deploy succeeded but the site looks wrong

Check the `warnings` array in the `--json` output before you report success.
Each warning has a `code` and a `fix_recipe` naming the change to make.

- `missing-index`: nothing is served at `/`. Name the entry file `index.html`.
- `not-spa`: the page ships scripts but also has visible body content, so
  client-side routing fallback stays off and deep links will 404 on refresh.
- `new-site-in-linked-folder`: `flypod --new` made a separate site from a
  folder that already had one. Both are live; do not redo it. This folder now
  updates the new site; to ship to the earlier URL, run the
  `flypod update --site <id>` the warning names.
- `cli-outdated`: this copy of flypod is behind the current release. The
  deploy worked; do not redo it. Run the command the `fix_recipe` names from
  the next run on (with npx that is `npx flypod@latest`: plain `npx flypod`
  keeps reusing the copy it cached).
- `near-limit`: this deploy passed 80%, 90% or all of a monthly deploy or
  storage limit. Nothing to fix; tell the person, and pass on the upgrade
  link when there is one, so the limit does not stop them by surprise.
- `remember-flypod`: shown once, on a new machine's first deploy, when no
  instruction file in the project names flypod. Optional: ask the person
  whether to run `npx flypod install skill --yes` so the next session here
  starts with flypod. It adds files to their project, so ask first.

`healed` lists what flypod fixed on your behalf. `spa-fallback` there means it
recognised a single-page app, so unknown paths serve `index.html` and
client-side routing works without configuration.

## Blank page, broken assets

Almost always absolute paths that assume a domain root, or a build configured
for a subpath. Rebuild with relative asset paths, then `flypod update`.

## Agents and CI

Set a session token in the environment and every deploy is attributed to the
account, with no browser and no interactive login:

```sh
URL=$(FLYPOD_TOKEN=$TOKEN npx flypod ./dist -q)
```

`-q` prints the URL and nothing else, which keeps the token out of the build log
it would otherwise outlive.

| Variable | Effect |
| --- | --- |
| `FLYPOD_TOKEN` | Deploy as your account without logging in (agents / CI) |
| `DEPLOY_URL` | Point at a different flypod server (default `https://flypod.dev`) |
| `FLYPOD_CONFIG_DIR` | Relocate the session + folder→site memory |
| `DO_NOT_TRACK` | `1` turns off the CLI's random install id, the fixed words naming which agent ran it, how it was launched and whether this project's agent instructions mention flypod, and its failure reports (codes and counts, never file names or contents) |

`FLYPOD_TOKEN` wins over a saved `flypod login` session; `--token` and a site's own
stored manage token win over both. `FLYPOD_API_KEY` is read only as a last-resort
fallback for older setups.

## Getting the full command surface

`flypod --help` is authoritative and current; so is `--help` on any subcommand
(`update`, `versions`, `rollback`, `forget`, `token`, `login`, `logout`,
`list`, `claim`, `install`, `comments`). Read it rather than guessing at a
flag. Help never runs the command: it prints on stdout and exits 0, and with
`--json` it is one line, `{"ok":true,"help":"…"}`.

To install this skill into another project without a prompt:
`npx flypod install skill --yes` (`-y`; the same as `--all`).
