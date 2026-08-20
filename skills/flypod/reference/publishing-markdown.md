# Publishing Markdown with flypod

flypod renders Markdown into a styled, browsable site. You do not need a static
site generator, a theme, or a build step — deploy the `.md` files themselves.

This is the fastest path from "I wrote a report" to "here is a link."

```sh
npx flypod ./report.md --json          # one document
npx flypod ./notes --json              # a folder of documents
```

## When it triggers

Markdown rendering kicks in when **both** are true of the files you deploy:

1. There is **no `index.html`** at the root of the deploy, and
2. at least one file ends in `.md`, `.markdown`, or `.mdx`.

That condition is deliberate: a real static site that happens to ship a stray
`README.md` is never hijacked. If you want files served untouched instead,
include an `index.html`.

## What you get

- **One HTML page per document.** `guide.md` is served at `/guide.html`.
- **A home page.** `README.md` or `index.md` becomes `/`. If you deploy exactly
  one document, that document becomes the home page whatever it is called.
- **Navigation** across every document, with the home doc first and the rest
  alphabetical.
- **A title per page,** taken from the first `#` heading, falling back to a
  title-cased version of the filename.
- **Light and dark themes,** applied before first paint so there is no flash.
- **Sanitized HTML.** Inline `<script>` tags, `on*` event attributes, and
  `javascript:` links are stripped from the rendered output. Interactive content
  belongs in a built static site, deployed as HTML.

## Agents get the Markdown back

The original `.md` sources are preserved and served through HTTP content
negotiation on the *same* URL. A browser gets the styled HTML; a client that
asks for Markdown gets the source:

```sh
curl -H 'Accept: text/markdown' https://ab12cd34.flypod.dev/guide.html
```

Known agent and crawler user-agents also get Markdown when they express no media
preference. An explicit preference for HTML always wins, so a real browser is
never handed raw Markdown.

This matters when you are the one reading the site back later: fetch it with
`Accept: text/markdown` and you get clean source instead of parsing HTML.

## Good uses

- A research write-up or analysis the user asked for, published instead of pasted.
- A changelog, migration guide, or RFC that needs review from someone without a
  checkout.
- Meeting notes or a status report that should have a URL rather than an attachment.
- Documentation for a small project, without adopting a docs framework.

Pair it with comments (`reference/comments.md`) when the reader should be able
to respond in place.
