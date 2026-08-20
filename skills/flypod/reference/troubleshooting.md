# Troubleshooting and environment

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | success |
| `1` | runtime error — no build folder, network failure, deploy rejected |
| `2` | usage error — unknown flag or bad arguments |
| `3` | `flypod comments` only: not signed in, or the token cannot write comments |
| `4` | `flypod comments` only: server error (5xx), so a retry is reasonable |

With `--json`, failures print `{"ok": false, "error": "…"}` on stdout, so you can
branch on the payload instead of scraping stderr.

## "No build output folder found"

flypod looked for `dist`, `build`, `out`, `.output/public`,
`.vercel/output/static`, `public`, `_site` — each of which must contain an
`index.html` — and found none. Either the project has not been built yet, or the
output lives somewhere else. Build it, then pass the path explicitly:

```sh
npm run build && npx flypod ./path/to/output --json
```

## The deploy succeeded but the site looks wrong

Check the `warnings` array in the `--json` output before you report success.
Each warning has a `code` and a `fix_recipe` naming the change to make.

- `missing-index` — nothing is served at `/`. Name the entry file `index.html`.
- `not-spa` — the page ships scripts but also has visible body content, so
  client-side routing fallback stays off and deep links will 404 on refresh.

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

## Getting the full command surface

`flypod --help` is authoritative and current; so is `--help` on any subcommand
(`update`, `versions`, `rollback`, `forget`, `token`, `login`). Read it rather
than guessing at a flag.
