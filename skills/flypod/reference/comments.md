# Comments: feedback on a deployed site

flypod can attach a commenting layer to a site, so a reviewer opens the URL and
leaves comments anchored to the page. You can then read those comments back as
JSON and act on them. That closes the loop: ship a preview, collect feedback,
fix, re-ship, without leaving the terminal.

## Turning comments on

At deploy time:

```sh
npx flypod ./dist --comments --json                     # mode: anonymous
npx flypod ./dist --comments --mode approved --json
```

This needs a signed-in deploy. Without one the site still deploys, and the
receipt says `"comments_enabled": false, "comments_skipped": "not_signed_in"`.
Tell the user comments are off, and that `flypod login` then
`flypod comments enable` turns them on.

On a site that already exists:

```sh
flypod comments enable --site <site_id> [--mode <mode>]
flypod comments mode <mode> --site <site_id> [--passcode <s>]
flypod comments off --site <site_id>        # disable; existing data is preserved
```

## Modes

| Mode | Who can comment |
| --- | --- |
| `anonymous` | anyone with the link, no sign-in; best for quick previews |
| `passcode` | anyone holding the passcode you set with `--passcode` |
| `approved` | anyone with the link, but each comment stays hidden until the owner approves it; best for anything semi-public |

Default when you pass `--comments` with no mode: `anonymous`.

## Reading feedback

```sh
flypod comments list <site_id> --json                 # ND-JSON: one thread per line
flypod comments list <site_id> --open --json          # only unresolved threads
flypod comments list <site_id> --page /pricing.html   # threads on one page
flypod comments show <site_id> <thread_id>            # one thread with its replies
```

`list` writes one JSON object per line, so it streams and pipes. `--json` only
suppresses the human-readable count on stderr; the thread objects go to stdout
either way.

## Replying and closing out

```sh
flypod comments post <site_id> --page <path> --target <json> --body <text>
flypod comments resolve <site_id> <thread_id>
flypod comments reopen  <site_id> <thread_id>
```

## Authorization

Read verbs follow the site's own policy. **Write verbs (`post`, `resolve`,
`reopen`, and the `enable`/`mode`/`off` verbs) need an account session**, and
`enable`/`mode`/`off` only work on a site that account owns. Run `flypod login`,
or set `FLYPOD_TOKEN` to a session token. With no credential they fail with
`not_signed_in` (exit 3). A token starting `fk_` is a retired API key that flypod
no longer accepts for anything: `post`, `resolve` and `reopen` refuse it before
sending (`unauthorized`, exit 3), and a deploy that carries one fails with
`token_rejected`. `reference/troubleshooting.md` has the full exit-code table.

## The review loop

```sh
npx flypod ./dist --comments --json          # ship a preview, share the url
flypod comments list <site_id> --open --json # collect what came back
# … make the fixes …
npx flypod update ./dist                     # same url, updated content
flypod comments resolve <site_id> <thread_id>
```
