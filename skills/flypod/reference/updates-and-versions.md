# Updates, versions, and rollback

Every command here acts on a site that already exists, so none of them forks it.
All take `--json`, and under `--json` every failure is `{"ok":false,"code":…}`
on stdout (see `reference/troubleshooting.md` for the codes). If the deploy
receipt said `"link_saved": false`, add `--site <site_id>` to each of these:
the `next` block in that receipt already has the exact commands.

```sh
flypod update                 # ship this folder as a new version
flypod update ./dist          # same, with an explicit path
flypod versions               # every version, newest first, live one marked
flypod rollback               # put the previous version back
flypod rollback <version>     # roll back to a version id, or a prefix of one
```

## How flypod knows which site

flypod remembers which folder maps to which site, in its own config directory
under your home, never in the repo, so there is nothing to `.gitignore` and
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
remembers: the one case that needs the plaintext. Treat it like a password: it
authorizes updates, rollbacks, and claims, and it cannot be rotated.

## Versions are content-addressed

A version id is a hash of the files, so deploying byte-identical content again
is a no-op: same hash, same version, no new entry in the history. A redundant
`flypod update` is free, and needs no guard.

## How long versions last

The live version stays as long as the site does. Any other version is deleted
seven days after it was deployed, so `flypod rollback` can reach back about a
week; a site also keeps at most 5 versions. On flypod Pro a site keeps 24, and
for 30 days. `flypod
versions` shows what is still there. Versions share their files, so an update
stores, and on a big site sends, only what changed.

## Free plan limits

flypod.dev is for hobby use. Per person (the signed-in account, or the network
an anonymous deploy comes from): 300 deploys a month (new sites and updates;
any one site at most 300 updates a month), 250 MB of storage (every site and
every kept version, each distinct file once), 10 deploys a minute, and 30 updates an hour to any one
site. There is no limit on how many
sites: notes, Markdown and single pages take a few KB. Each site may serve
5,000 requests a day, then shows a "back at midnight UTC" page. Months and days
are UTC.

Do not redeploy on a timer or use a site as a live data feed: every update
spends the month's deploys. Batch your changes and deploy when they are done.
A spent limit is `quota_exceeded` (exit 1). Do not retry it: tell the human
which limit (`limit`) and when it resets (`resets_at`). For
`limit: "storage"` there is no reset date: suggest smaller images or video,
or deleting sites they no longer need at flypod.dev/account. When the error
carries `upgrade_url`, also show the human that link: a bigger plan raises the
limit (`upgrade_plan` says which), they pay on that page themselves (about 30
seconds), and then you run the same command again. Never try to pay for them.

flypod Pro is $5 a month (or $50 a year): custom domains, password-protected
sites, 2 GB of storage, 1,000 deploys a month and 60 updates per site an hour.
flypod Studio is $12 a month (or $120 a year): everything in Pro with 25 GB of
storage, 5,000 deploys a month and 120 updates per site an hour, for people
shipping many sites; on Pro the link is a page where the person confirms paying
only the difference. `flypod upgrade --json` (`--studio` for Studio) returns the
payment link (`url`) to show the human whenever they ask for more room;
it charges nothing by itself.

## Private sites and custom domains (Pro)

If the human wants the site private, deploy with `--password`: the receipt's
`protection` has `password`, `share_url` (opens it without typing) and
`curl`. Give the human the share link (or the password), and check the deploy
with the curl line (`curl -u ':PASSWORD' URL`). `flypod password --json` shows
it again; `--new` changes it, `--off` makes the site public.

For their own domain: `flypod domain www.example.com --json`. Relay `say` to the
human as it is (it names their DNS provider, the one record to add, and the
page to add it on), then run `flypod domain --json` every `check_after_seconds`
until `status` is `active`; `url` is then live. If `problem` is set, its
`fix` is exactly what to change. Prefer `www.` over a bare domain; a 400
with `suggestion` means the provider cannot point the bare one, so run the
command again with `suggestion` (after asking the human).

Either one without Pro fails with `pro_required`: show the human
`upgrade_url`, and once they say they have paid, run the same command again.

## Typical loop

```sh
npm run build && npx flypod ./dist --json   # first deploy, note the url
# … edit, rebuild …
npm run build && npx flypod update --json   # same url, new content
npx flypod versions --json                  # confirm what's live
npx flypod rollback                         # undo a bad ship
```
