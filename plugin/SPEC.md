# MemoryMtrls format spec

This file is the single source of truth for the file formats used by MemoryMtrls commands. Both `/memorymtrls:init` and `/memorymtrls:improve` read this before writing anything, so the two commands never drift apart.

The core idea: `CLAUDE.md` is auto-loaded into every agent's context, so it must stay small. Everything else lives in a memory directory, one file per category, read only when relevant.

**The frontmatter of the memory files is the source of truth.** The `CLAUDE.md` router table is always *generated from it* and can be rebuilt at any time.

---

## 1. Names and Casing

Files and directories should make consistent use of one the following casing formats: kebab, snake, camel, pascal.

Examples of each casing format on the string "_error handling_":
- kebab: `error-handling`
- snake: `error_handling`
- camel: `errorHandling`
- pascal: `ErrorHandling`

The casing format used for files does not have to match the format used for directories.

---

## 2. Memory Directory

Context about different parts of the project are stored as memory files in a memory directory. That directory is the root directory for MemoryMtrls.

- By default, the directory name is `memory/`
- If repo uses pascal casing for directory names, the default will be `Memory/`
- Defaults can be overridden for user preference or to avoid name collisions

---

## 3. Memory file format

Each memory file stores context in markdown format file and uses YAML frontmatter for metadata about the memory.

```markdown
---
scope: An example for Auth context
globs:
  - src/auth/**
  - src/middleware/session*
keywords: [login, session, token, oauth, jwt, rbac, permission]
updated: 2026-06-04
---

# Auth

One-line restatement of scope.

## <Section>
Real, specific knowledge about this area drawn from the actual codebase: conventions, gotchas, the "why" behind decisions, key files, commands. Written to be self-contained. An agent should be able to work in this area after reading only this file (plus anything it explicitly cross-references).

## See also
- `<other>.md` — only when there's a genuine dependency. Keep sparse;
  every cross-reference is a potential extra read.
```

### Frontmatter fields

| Field      | Required | Purpose |
| ---------- | -------- | ------- |
| `scope`    | yes      | One-line summary; becomes the basis of the router "read when" trigger. |
| `globs`    | yes      | Path globs this file governs. Used to map changed source files → memory files. Use `[]` for purely conceptual categories with no file home (e.g. `release-process`). |
| `keywords` | yes      | Symptoms / terms that help an agent determine when this file is relevant. Feeds the router trigger. |
| `updated`  | yes      | ISO 8601 datetime of the last update. Use the current date. |

### Body rules

- Be specific to *this* codebase. No generic advice, no placeholders.
- Include this area's conventions and style, not just facts.
- Keep it self-contained per category. Cross-reference sparingly.
- It is fine for a file to be short. A small accurate file beats a padded one.
- **Memories describe what *is*, never what *was*.** A memory file is a
  description of the current state of the system, not a changelog. Change
  narration — "previously", "used to", "no longer", "was removed", "renamed
  from", "migrated to", "now uses" — has no place in a memory file. When the
  reasoning behind a decision matters, state it as a present-tense fact
  ("sessions live in Redis because the app runs on multiple hosts"), not as an
  event ("we moved sessions from cookies to Redis"). Content about removed
  features is deleted outright, unless the *absence* itself is a gotcha worth
  knowing — then describe the absence, in the present tense.

---

## 4. CLAUDE.md as memory router

`CLAUDE.md` holds: 
- a short project header (plus any rules that apply to every task)
- brief description of memory system 
- generated routing table

The routing table lives inside marker comments and is regenerated from frontmatter; everything outside the markers is user-owned and left untouched.

Conventions live in the memory files (§3), not here — they load only when their file is read. The exception is a rule that applies to *every* task; put those in the header prose so they're always loaded.

`CLAUDE.md` uses the following template. Tags `<...>` are replaced with actual values from the project.

```markdown
# <Project name>

<One or two lines on what the project is. Note any rules that apply to every task here too.>

<!-- memorymtrls:start -->

`.notes/` should be used for **personal workflow scratch** and gitignored. Place artifacts produced while working in here, such as plans, specs, exploratory writeups, eg thinking out loud.

- Plans → `.notes/plans/YYYY-MM-DD-<slug>.md` — execution outlines for multi-step work. Stay as historical record after the work lands.
- Specs → `.notes/specs/YYYY-MM-DD-<slug>.md` — designs before implementation.
- Other → `.notes/scratch/`.

## Memory

This project's working context lives in <Memory directory>, with one file per category of context. The table below maps task level concerns to the context file(s) that cover it.

**IMPORTANT: Before starting a task, scan this table. If any rows match the task, read the relevant memory files, and then start the task.** Read only the rows that match. Context window efficiency is gained by not reading irrelevant context; if nothing matches, proceed without reading any.

**IMPORTANT: Before finishing a task that changed how an area works, update that area's memory file.** Keep edits surgical and bump `updated`. Memory files describe the current state of the system — what *is*, never what *was*. Do not narrate the change ("previously", "no longer", "moved from…"); rewrite the affected text as if the current state had always been true. If the work created a new area with no matching file, tell the user rather than inventing a category.

| Memory file | Read this when… |
| ----------- | --------------- |
| `memory/auth.md` | working on login, sessions, tokens, permissions |
| `memory/concurrency.md` | touching async code, locks, the task queue, workers |
| `memory/data-model.md` | changing schemas or migrations, anything under `db/` |

<!-- memorymtrls:end -->
```

### Generating the router table

- Always include both bold instructions (the "Before starting a task…" routing instruction and the "Before finishing a task…" maintenance instruction) verbatim inside the block, except for the `<Memory directory>` tag which should be rendered. The routing instruction is what makes agents consult memory; the maintenance instruction is what keeps memory current as agents work. Never drop either.
- One row per memory file.
- **Read this when…** is written from that file's `scope` + `keywords` (and, where helpful, its `globs`). It should be phrased as the situation/symptoms a fresh agent would recognize, not just the topic noun.
- Rebuild the whole block between the markers from the current frontmatter set. Never hand-edit rows in a way that diverges from frontmatter. Frontmatter is the truth for each file.
- Everything **outside** the markers is user-owned: preserve it untouched.
- If `CLAUDE.md` has no markers yet, insert the block (after the project header if one exists; otherwise create a minimal header).
