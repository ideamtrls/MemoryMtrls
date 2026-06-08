---
description: Setup the memory system & commands provided by MemoryMtrls.
---

You are bootstrapping the MemoryMtrls memory system in the current repository.

First, read `${CLAUDE_PLUGIN_ROOT}/SPEC.md`. It defines the exact file formats
you must produce (memory directory + casing rules, memory-file frontmatter, and
the CLAUDE.md router block). Follow it precisely.

Work through these steps. Do not skip the approval checkpoint in Step 3.

## Step 1 — Detect format settings & mode

**Format settings** (per SPEC.md):
- Filename casing for category files (kebab / snake / camel / Pascal; default
  kebab for markdown).
- Memory directory name (default `memory/`; if one already exists, keep its name
  and casing — don't rename it).

**Mode** — decide how much to (re)generate. This is the most important early
call, because regenerating over good hand-written memory destroys real work:
- **Migrate** — a memory system already exists: a memory directory with category
  files, and/or a `CLAUDE.md` that already routes to per-area files, and/or
  memory files carrying *non-SPEC* frontmatter (any keys other than
  `scope`/`globs`/`keywords`/`updated` — e.g. `file:` / `read-when:`). Treat the
  existing content as **valuable and authoritative**: your job is to bring it to
  SPEC format and verify it, NOT to rewrite it or reorganize its taxonomy.
- **Greenfield** — no memory directory (or an empty one). Build from scratch.

Report the detected casing, directory name, and mode (with the evidence that
picked it).

## Step 2 — Understand the project

Scale the effort to the repo — don't spin up agents for a small one:
- **Small / single-subsystem** → inventory inline: languages, top-level
  structure, how to build / test / run, and the main subsystems.
- **Large / many subsystems** → dispatch **parallel read-only survey agents**
  (one per top-level subsystem, plus one for existing context docs), each
  returning a structured summary: `{subsystem, key files, what it does, public
  types / entry points, build·test·run commands, gotchas}`. Tell the user how
  many you're dispatching and why *before* you do it. (Use the Task/Agent tool,
  or a Workflow when available.) Merge the summaries.

Read existing context if present — `CLAUDE.md`, `AGENTS.md`, `README*`, `docs/`.
In **Greenfield** mode treat these as raw material to reorganize, not gospel, and
not to copy verbatim. In **Migrate** mode the existing memory files + router ARE
the primary material: read them to learn the established taxonomy and content,
and preserve it.

## Step 3 — Propose categories, then WAIT

Getting a taxonomy the user is happy with is the most important part — they will
live with these categories.

- **Migrate mode:** the existing categories ARE the recommended proposal.
  Present them as-is (filename, one-line `scope`, "read when" trigger) and
  **default to keeping them** — do not reorganize a working, human-authored
  taxonomy. You may note a genuinely missing area or a real overlap as an
  *optional* suggestion the user can decline.
- **Greenfield mode:** derive candidate categories from the subsystems (auth,
  billing, rendering…) plus cross-cutting concerns (testing, build/deploy, code
  style, data model…). Present **a few options**: a recommended set plus 2–3
  alternative groupings (finer vs. coarser, by-subsystem vs. by-concern). For a
  large/complex repo you may generate the candidates with a short **taxonomy
  panel** — a few agents each drafting a taxonomy under a different organizing
  principle, then synthesize the best — but this is optional.

For every proposed file show: the filename (in the detected casing), its one-line
`scope`, and its "read when" trigger. Let the user pick a set, mix options across
groupings, rename, split, or merge. **Do not create or modify any files until
they approve a final list.**

## Step 4 — Write the files

- **Migrate mode:** keep each file's body. Convert its frontmatter to the SPEC
  set (`scope`, `globs`, `keywords`, `updated`), deriving `globs` / `keywords`
  from the content and the source tree. Only edit body text that is now **stale
  or wrong**. Keep the diff minimal — this is a reformat, not a rewrite.
- **Greenfield mode:** for each approved category, create the file per SPEC.md
  (frontmatter + body) with real, specific knowledge pulled from the actual
  codebase — no placeholders, no generic advice; a short accurate file is fine.
  For a large repo, dispatch **parallel authoring agents**, one per category,
  each deep-reading its globbed files before writing.

## Step 5 — Verify

Independently check the written files against reality — a subtly wrong memory
file is worse than none:
- Every claim matches the actual source (spot-check the key facts in code).
- Cross-references resolve, and the `globs` **partition** the source tree — flag
  gaps (source with no memory home) and overlaps.
For a large file set, fan out one checker agent per file in parallel. Apply
targeted fixes for whatever the check surfaces.

## Step 6 — Write the CLAUDE.md router

Generate the router table **from the frontmatter** you just wrote or migrated,
inside the `<!-- memorymtrls:start -->` / `<!-- memorymtrls:end -->` markers
(SPEC.md §4). This is deterministic — do it in one context, no agents. Preserve
all CLAUDE.md content outside the markers. If you migrated content out of
CLAUDE.md into memory files, replace that migrated block with the router and keep
a short project header.

## Step 7 — Report

Summarize what you created or migrated (the files + the router), note the mode
you used and any agents you dispatched, and tell the user to run
`/memorymtrls:update` at the end of future sessions to keep memory current.

## Step 8 — Clean up historical data

Tell the user the memory system is created, and the last step is to clean up the
historical workflow data.

- Figure out which directory holds the workflow scratch — plans, specs,
  exploratory writeups, task lists; working notes, not published docs.
- Tell the user MemoryMtrls keeps this as `.notes/` and adds it to `.gitignore`,
  to separate the temporary from the permanent, then move it there. (If there is
  no such directory, just create `.notes/` and gitignore it.)
