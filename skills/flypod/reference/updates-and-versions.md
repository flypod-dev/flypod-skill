# Updates, versions, and rollback

Every command here acts on a site that already exists, so none of them forks it.
All take `--json`.

```sh
flypod update                 # ship this folder as a new version
flypod update ./dist          # same, with an explicit path
flypod versions               # every version, newest first, live one marked
flypod rollback               # put the previous version back
flypod rollback <version>     # roll back to a version id, or a prefix of one
```

## How flypod knows which site

flypod remembers which folder maps to which site, in its own config directory
under your home — never in the repo, so there is nothing to `.gitignore` and
nothing to commit. From the folder you deployed, these need no id and no flags.

`FLYPOD_CONFIG_DIR` relocates that memory, which is how you isolate CI runs.

## When the folder memory isn't there

Stateless, in a different directory, or holding a `site_id` from an earlier
deploy? Target the site:

```sh
flypod update    --site <site_id> ./dist
flypod versions  --site <site_id>
flypod rollback  --site <site_id> <version_id>
flypod forget    [--site <site_id>]   # drop the folder→site memory
```

Sites you own authorize with your login. A **temporary** site acted on from a
machine that does not remember it authorizes with its manage token:

```sh
flypod update --site <site_id> --token <manage_token> ./dist
```

`flypod token [--site <id>]` prints that token on stdout for a site this machine
remembers — the one case that needs the plaintext. Treat it like a password: it
authorizes updates, rollbacks, and claims, and it cannot be rotated.

## Versions are content-addressed

A version id is a hash of the files, so deploying byte-identical content again
is a no-op: same hash, same version, no new entry in the history. A redundant
`flypod update` is free, and needs no guard.

## Typical loop

```sh
npm run build && npx flypod ./dist --json   # first deploy, note the url
# … edit, rebuild …
npm run build && npx flypod update --json   # same url, new content
npx flypod versions --json                  # confirm what's live
npx flypod rollback                         # undo a bad ship
```
