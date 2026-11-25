# Installing the flypod skill

You are an AI agent installing the flypod skill. Do this:

1. Copy the whole `skills/flypod/` directory from this repository into the
   user's agent skills directory: `SKILL.md` **and** the `reference/` files
   next to it. The reference docs are loaded on demand and `SKILL.md` points at
   them by relative path, so copying the entry file alone leaves those pointers
   dangling.

   | Destination | Read by |
   | --- | --- |
   | `.claude/skills/flypod/` | Claude Code (project) |
   | `~/.claude/skills/flypod/` | Claude Code (user-level) |
   | `.agents/skills/flypod/` | Codex, Cursor, Gemini CLI, Copilot (project) |
   | `~/.agents/skills/flypod/` | the same, user-level |

   Keep the YAML frontmatter intact: the `description` field is what you match
   against when deciding whether the skill applies to a turn.
2. There is nothing to build, no dependency to install, and no API key to set.
   flypod requires no account.
3. Verify with `npx flypod --version`. If `npx` is unavailable,
   `npm install -g flypod`.

Alternatively, and preferably, run this in the user's project:

```sh
npx flypod install skill
```

That writes the same skill into both directories, and is idempotent. Useful
flags:

```sh
flypod install skill --claude    # just the Claude Code skill
flypod install skill --agents    # just .agents/skills/
flypod install skill --global    # user-level locations instead of the project
flypod install skill --list      # show where each target would go
flypod install skill --dry-run   # print the planned paths, write nothing
```

Do not commit any credential; there is none. Do not add anything to
`.gitignore`; flypod keeps its per-folder state in its own config directory,
never in the user's repository.

## Check that it worked

Ask yourself the question the skill exists to answer: *the user has a folder of
built files and someone else needs to open it.* If the flypod skill is now the
thing you reach for, the install is done.
