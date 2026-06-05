---
description: Set up the MemoryMtrls memory/ system for this repo and shrink CLAUDE.md to a router.
---

You are bootstrapping the MemoryMtrls memory system in the current repository.

First, read `${CLAUDE_PLUGIN_ROOT}/SPEC.md` — it defines the exact file formats
you must produce (memory directory + casing rules, memory-file frontmatter, and
the CLAUDE.md router block). Follow it precisely.

Work through these steps. Do not skip the approval checkpoint in Step 3.

## Step 1 — Detect conventions
- Determine the filename casing convention per SPEC.md (kebab / snake / camel /
  Pascal; default kebab for markdown).
- Determine the memory directory name (default `memory/`).
- Briefly report what you detected and why.

## Step 2 — Understand the project
- Inventory the codebase: languages, top-level structure, how to build / test /
  run it, and the main subsystems.
- Read any existing context sources if present: `CLAUDE.md`, `AGENTS.md`,
  `README*`, `docs/`. Treat them as raw material to reorganize — not as gospel,
  and not as something to copy verbatim.

## Step 3 — Propose categories, then WAIT
Getting a taxonomy the user is happy with is the most important part — they will
live with these categories.

- Derive candidate categories from the subsystems (auth, billing, rendering…)
  plus cross-cutting concerns (testing, build/deploy, code style, data model…).
- Present **a few options to choose from** — e.g. a recommended set plus one or
  two alternative groupings (finer-grained vs. coarser, or organized by
  subsystem vs. by concern). For every proposed file show: the filename (in the
  detected casing), its one-line `scope`, and its "read when" trigger.
- Let the user pick a set, mix categories across options, rename, split, or
  merge. **Do not create any files until they approve a final list.**

## Step 4 — Write the memory files
- For each approved category, create `memory/<name>.<ext>` exactly per SPEC.md:
  frontmatter (`scope`, `globs`, `keywords`, `updated`) + body.
- Fill bodies with real, specific knowledge pulled from the actual codebase.
  No placeholders, no generic advice. A short accurate file is fine.

## Step 5 — Write the CLAUDE.md router
- Generate the router table **from the frontmatter** you just wrote, inside the
  `<!-- mtrls:router:start -->` / `<!-- mtrls:router:end -->` markers (SPEC.md §3).
- Preserve any existing CLAUDE.md content outside the markers. If you migrated
  content out of CLAUDE.md into memory files, replace that migrated content with
  the router and keep a short project header.

## Step 6 — Report
Summarize what you created (the files + the router), and tell the user to run
`/mtrls-update-memory` at the end of future sessions to keep memory current.
