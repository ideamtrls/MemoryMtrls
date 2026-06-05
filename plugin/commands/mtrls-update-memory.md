---
description: Update memory files with context from current session and regenerate routing in CLAUDE.md.
---

You are updating the MemoryMtrls memory system based on what happened in **this
session**. This command takes no arguments. Use the context of the current
session to decide what changed and which memory files must be updated.

First, read `${CLAUDE_PLUGIN_ROOT}/SPEC.md` for the file formats, then reread
the repo's `CLAUDE.md` router to refresh knowledge of which memory files exist
and what each one covers. Reread individual `memory/*` files as needed.

If there is no `memory/` directory yet, tell the user to run `/mtrls-init-memory`
first, and stop.

## Step 1 — Determine what changed this session
Review what actually happened in this session and extract anything worth
writing down for future sessions. The point is to update knowledge of the
system as it is after the changes. Avoid language that talks about things
that are no longer true, like removed features.

Examples of knowledge worth keeping:
- files created, edited, moved, or deleted;
- new conventions, patterns, or architectural decisions introduced;
- gotchas or surprises discovered;
- build / test / run commands learned;
- anything that makes an existing memory file now inaccurate.

Only capture what this session changed and why. If nothing meaningful changed,
say so and stop.

## Step 2 — Map changes to memory files
- Match the changed paths and topics to memory files using each file's `globs`
  and `keywords` (from frontmatter) and the CLAUDE.md router.
- Produce two lists: (a) existing memory files to update, and (b) any new
  subsystem/concern that has no home.

## Step 3 — Update
- Update each affected memory file: add or correct content and bump `updated` to
  the current date. Keep edits **surgical** — change what's new or stale, don't
  rewrite whole files.
- For a new subsystem/concern with no category: propose a new memory file (name
  in the repo's casing, `scope`, and "read when" trigger) and **confirm with the
  user before creating it** — the user owns the taxonomy.

## Step 4 — Regenerate the router
Rebuild the `<!-- mtrls:router:start -->` … `<!-- mtrls:router:end -->` block in
`CLAUDE.md` from the current frontmatter set (adding rows for new files, removing
rows for deleted ones, refreshing triggers). Leave everything outside the markers
untouched.

## Step 5 — Report
List exactly which files you changed and what you changed in each.
